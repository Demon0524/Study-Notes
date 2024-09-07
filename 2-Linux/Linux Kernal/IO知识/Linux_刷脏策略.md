 #Linux  #Linux_kernal #IO
# 脏页的由来
1. 在多级存储系统中，上一级高速设备会成为下一级低速设备的缓存。相较之内存，磁盘是一个低速设备，因此Linux中会通过一种叫“磁盘高速缓存”的软件机制来允许将磁盘上的一些数据保留在内存中，以加快访问速度。页高速缓存page cache就是Linux内核所使用的主要磁盘高速缓存。几乎所有的文件读写都依赖磁盘高速缓存，除非你设置了O_DIRECT标志；

2. Linux中文件系统主要分成三类：磁盘文件系统(例如ext系列)、基于内存的特殊文件系统(例如devtmpfs、bdev、proc、sys等)和网络文件系统(例如nfs)，你可以通过mount指令不带参数查看，或者查看/proc/mounts，就能看到已经安装的文件系统及其对应的挂载点，IO设备中块设备会涉及三个文件系统——设备文件系统devtmpfs、磁盘文件系统例如ext、块设备文件系统bdev；

3. 在Linux kernel 2.4.10以前有两种磁盘高速缓存，一种是文件缓存page cache，一种是设备缓冲buffer，都在struct page结构中，前者是address_space(实际上是指向inode->address_space)，后者是buffer_head；在Linux kernel 2.4.10以后，设备缓冲和文件缓存都放在inode->address_space中，但是会通过不同的文件系统来做区别，page cache继续放在磁盘文件系统的inode中(例如ext系列)，buffer虽然也是通过address_space中的radix tree来组织，不过是放在块设备文件系统bdev->inode中；

4. address_space中通过基树数radix tree来组织page cache，这棵基树数中的叶子节点其实是一个个page对象，struct page中有一个unsigned long flags，其中有一个bit是PG_dirty的标志，如果该标志被置位，则表示这是一个脏页；

5. 其实也就是说，如果一个page cache中的页的数据被修改了，那么该页和磁盘上的数据就不一致了，就需要把修改后的数据刷盘；

6. 但是内存是易失性的，断电后数据就丢了，所以脏页的存在其实是一件危险的事情，因此本文就要讨论两个问题：

> 引入脏页的原因
> 通过刷脏页的机制来尽力保证数据安全

总结一下，脏页是page cache中被设置了PG_dirty标志的页面。但这只是解释了Linux系统中哪些是脏页哪些不是脏页，为什么这么设计，请接着往下看。

![](https://img2023.cnblogs.com/blog/1755541/202304/1755541-20230418185329812-2057016111.png)

![](https://img2023.cnblogs.com/blog/1755541/202304/1755541-20230418185338123-1389432795.png)

![](https://img2023.cnblogs.com/blog/1755541/202304/1755541-20230418185348014-216595905.png)

## 二、引入脏页的原因

其实并没有什么特别的地方，追根揭底就是一个原因——性能。

Linux在设计IO模块的时候，认为读写操作的紧迫性是不一样的，读操作的紧迫性要高于写操作。所以Linux的写默认是延迟的，因为这样不仅仅可以显著的提高效率：

写操作合并：多次写操作才会真正的发起一次写物理磁盘，大幅提升IO性能；

读操作比写操作更紧迫：

通常延迟写物理磁盘不会导致进程挂起，写到page cache然后修改该页的PG_dirty标志即可返回(如果open的时候设置了sync标志或者调用了fsync、sync函数，那么会将写page cache的内容封装成一个IO request放到IO调度层的request_queue中，然后由调度层的调度算法来写到物理磁盘)；

但是如果读操作读的时候page cache中没有缓存数据，读操作就需要去读物理磁盘，此时延迟读就会让用户感知，这就很糟糕了，用户就会感觉OS卡顿，Linux作为桌面系统的时候，他还是要优先保证和用户良好的交互性；

你还可以从IO scheduler的调度算法deadline的默认配置感受一下，读延迟是5ms，写延迟是5s，这就很明显的感受到Linux对读写操作紧迫性的区别对待；

也正因为写操作默认是延迟写的，因此内存中就会存在大量的脏页，这时候的风险是：

写操作返回时已经向用户应答写成功，但是其实只是写了内存页page cache，数据并没有落盘，如果此时发生机器掉电那么数据就会丢失；

除此之外，脏页会占用大量内存，这也是一个缺点；

page cache脏页，在提高效率的同时也引入了风险。为了降低风险，Linux引入了一些刷脏页相关的机制：

pdflush：

数量维度：如果脏页太多，达到了一定的量，那么就需要刷脏页到磁盘；

时间维度：如果脏页没有达到刷盘的量，但是这一小部分脏页在内存中存活了很长时间，超过了允许的时间阈值，也要刷盘；

立即落盘：

在open file的时候，设置flag的sync标志；

如果在open file的时候没有设置，那么通过sync()/fsync()等系统调用也能刷盘；

而pdflush刷脏页的内核线程就是本文要谈的第二个点。

## 三、pdflush

早期Linux是通过bdflush内核线程从数量维度扫描需要刷盘的脏页，通过kupdate内核线程从时间维度来保证不会有脏页存在太长时间。在Linux kernel 2.6以后，通过pdflush内核线程替代了这两个内核线程。

在pdflush中有两个主要的函数：

background_writeout():从数量维度出发，当脏页达到一定数量，就唤醒pdflush调用该方法将脏页刷盘；

wb_kupdate():从时间维度出发，周期性的唤醒pdflush调用该方法扫描，如果脏页存活时间超过配置，则刷盘；

四个配置项，你可以通过sysctl来修改，也可以通过/proc/sys/vm/dirty_background_ratio等来修改：

dirty_background_ratio：默认是10%，如果脏页占比超过这个值，那么就唤醒pdflush刷脏页；

dirty_ratio：默认是40%，dirty_background_ratio虽然唤醒了pdflush刷脏页，但是进程一样可以继续写操作，当写的速度超过pdflush刷盘的速度，那么脏页会继续增多，这时候产生的危险是可能会导致内存耗尽；所以为了安全，当超过了这个比例，那么就会发生IO长暂停然后各个相关的进程都会阻塞直到把自己的脏页刷盘；

```
dirty_expire_seconds & dirty_writeback_centisecs：默认都是1/100(即时间间隔为每秒100次)，根据dirty_write_centisecs的配置周期性的唤醒pdflush，pdflush会检查脏页的存活时间是否超过dirty_expire_seconds，如果超过则刷盘；

dirty_background_ratio

Contains, as a percentage of total available memory that contains free pages

and reclaimable pages, the number of pages at which the background kernel

flusher threads will start writing out dirty data.

The total available memory is not equal to total system memory.

dirty_ratio

Contains, as a percentage of total available memory that contains free pages

and reclaimable pages, the number of pages at which a process which is

generating disk writes will itself start writing out dirty data.

The total available memory is not equal to total system memory.

dirtytime_expire_seconds

When a lazytime inode is constantly having its pages dirtied, the inode with

an updated timestamp will never get chance to be written out. And, if the

only thing that has happened on the file system is a dirtytime inode caused

by an atime update, a worker will be scheduled to make sure that inode

eventually gets pushed out to disk. This tunable is used to define when dirty

inode is old enough to be eligible for writeback by the kernel flusher threads.

And, it is also used as the interval to wakeup dirtytime_writeback thread.

==============================================================

dirty_writeback_centisecs

The kernel flusher threads will periodically wake up and write `old' data

out to disk. This tunable expresses the interval between those wakeups, in

100'ths of a second.

Setting this to zero disables periodic writeback altogether.
```
参考资料：

《understanding the linux kernel 3rd》