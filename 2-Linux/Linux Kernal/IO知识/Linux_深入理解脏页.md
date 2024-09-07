 #Linux  #Linux_kernal #IO
# 前言  
Linux 内核Page Cache 和Buffer Cache 关系及演化历史 一文中讲过Linux 2.4之后将Page Cache和Buffer Cache 进行了融合，在buffer_head 中添加了b_page，很容易就能找到缓存的Page Cache，而buffer_head 的存在就是能够快速确定页中的一个块在磁盘中的地址。

Linux内核由于存在page cache, 一般修改的文件数据并不会马上同步到磁盘，会缓存在内存的page cache中，我们把这种和磁盘数据不一致的页称为脏页，脏页会在合适的时机同步到磁盘。为了回写page cache中的脏页，需要标记页为脏（dirty）。

脏页跟踪是指内核如何在合适的时机记录文件页为脏，以便内核在进行脏页回写时，知道将哪些页面回写到磁盘。匿名页不需要跟踪脏页，因为不需要同步到磁盘；私有文件页也不需要跟踪脏页，因为映射的时候，可写页会映射为只读，写访问会发生写时复制，转变为匿名页；所以只有共享的文件页需要跟踪脏页。跟踪有两个层面：一个是页表项记录，一个是页描述符记录。

访问文件页有两种方式：

一种是通过mmap映射文件；  
一种是通过文件系统的write接口操作文件；  
本文将对这两种方式进行讲解。在Linux内核中，因为跟踪脏页会涉及到文件回写、缺页异常、反向映射等技术，所以本文也重点讲解在Linux内核中如何跟踪脏页。

# 引入脏页提高性能  
Linux在设计IO模块的时候，认为读写操作的紧迫性是不一样的，读操作的紧迫性要高于写操作。所以Linux的写默认是延迟的，因为这样不仅仅可以显著的提高效率：

写操作合并：多次写操作才会真正的发起一次写物理磁盘，大幅提升IO性能；  
IO栈中还有通用块层和IO调度层，在具体的文件系统层有文件在文件系统中的逻辑块映射，也就是文件的文件块在磁盘上对应的磁盘块的地址，这时候在通用块层会把这些信息封装成BIO对象，然后提交到gendisk->request_queue，在提交的过程中，如果BIO和已有的request中的BIO在物理地址上是连续的，那么这些BIO就会合并成一个request。在IO调度层，一次request就意味着一次IO，因为这时候磁盘需要重新寻址，所以这个合并其实是在io调度层入队列的时候完成的；  
你可以通过iostat查看物理盘IO情况的时候看到对应的监控指标：rrqm/s、wrqm/s等；  
读操作比写操作更紧迫：  
通常延迟写物理磁盘不会导致进程挂起，写到page cache然后修改该页的PG_dirty标志即可返回（如果open的时候设置了sync标志或者调用了fsync、sync函数，那么会将写page cache的内容封装成一个IO request放到IO调度层的request_queue中，然后由调度层的调度算法来写到物理磁盘）；  
但是如果读操作读的时候page cache中没有缓存数据，读操作就需要去读物理磁盘，此时延迟读就会让用户感知，这就很糟糕了，用户就会感觉OS卡顿，Linux作为桌面系统的时候，他还是要优先保证和用户良好的交互性；  
你还可以从IO scheduler的调度算法deadline的默认配置感受一下，读延迟是500ms，写延迟是5s，这就很明显的感受到Linux对读写操作紧迫性的区别对待；  
# mmap 映射的文件页  
基本过程如下：

step1，通过mmap映射共享文件；  
step2，第一次访问文件页时，发生缺页后读文件页到page cache，如果是写访问则设置相应进程的页表项为脏、可写；  
step3，脏页回写时，会通过反向映射机制，查找映射这个页的每一个vma, 设置相应进程的页表项为只读，清脏标记；  
step4，假如第二次写访问这个文件页时，脏页的处理有两种情况：  
page cache中的文件页还未回写到磁盘（step3 之前）， 此刻，这个文件页依然是脏页。因为相应进程的页表项为脏、可写，所以可以直接写这个页；  
page cache中的文件页已经回写到磁盘（step3 之后）， 此刻，这个文件页不再是脏页。因为页表项为只读，所以写访问会发生写时复制缺页异常，异常处理中将处理共享文件页映射，重新将相应进程的页表项为设置为脏、可写  
## 第一次写访问文件页


```
/mm/memory.c
 
handle_pte_fault
    ->do_fault
        ->do_shared_fault
            |->__do_fault //读文件页到page cache
            |->do_page_mkwrite
            |    ->vmf->vma->vm_ops->page_mkwrite
            |        ->filemap_page_mkwrite /mm/filemap.c
            |            ->set_page_dirty
            |                ->__set_page_dirty_buffers
            |                    ->TestSetPageDirty  //设置页描述符脏标记
            |                    ->__set_page_dirty  //page cache中标记页为脏 
            |->finish_fault  //设置页表项
                ->alloc_set_pte
                    ->maybe_mkwrite  //设置页表项脏、可写
```

## 2.2 脏页回写

```
/mm/page_writeback.c

write_cache_pages
    ->clear_page_dirty_for_io  //对于回写的每一个页
        |->page_mkclean  //清脏标记  mm/rmap.c
        |   ->page_mkclean_one  //反向映射查找这个页的每个vma，调用清脏标记和写保护处理
        |        ->entry = pmdp_invalidate(vma, address, pmd);
        |          entry = pmd_wrprotect(entry);  //写保护处理，设置只读
        |          entry = pmd_mkclean(entry);  //清脏标记 set_pte_at(vma->vm_mm, address, pte, entry) //设置到页表项中
        |->set_page_dirty
```

## 第二次写访问文件页  
1. 脏页还没有回写时（确切的说是调用clear_page_dirty_for_io之前），页描述符已经设置了脏标记，页表项已经设置了脏标记、可写。

这时可以直接写访问文件页，不会发生缺页。

2. 脏页已经回写时（确切的说是调用clear_page_dirty_for_io之后），页描述符已经清除了脏标记，页表项已经清除了脏标记，且只读。

这时写访问文件页会发生写时复制缺页异常（访问权限错误缺页）。

调用链如下：
 ```
/mm/memory.c

handle_pte_fault

    ->if (vmf->flags & FAULT_FLAG_WRITE) {  //vma可写
          if (!pte_write(entry))  //页表项没有可写属性
              do_wp_page(vmf)  //写时复制缺页异常处理
                  ->if (unlikely((vma->vm_flags & (VM_WRITE|VM_SHARED))  //是共享可写的文件映射vma
                        wp_page_shared
                            |->do_page_mkwrite
                            |    ->vmf->vma->vm_ops->page_mkwrite
                            |        ->filemap_page_mkwrite /mm/filemap.c
                            |            ->set_page_dirty
                            |               ->__set_page_dirty_buffers
                            |                    ->TestSetPageDirty  //设置页描述符脏标记
                            |                    ->__set_page_dirty  //page cache中标记页为脏
                            |->finish_mkwrite_fault
                                 ->wp_page_reuse
                                     ->entry = maybe_mkwrite(pte_mkdirty(entry), vma);  //重新设置页表项脏、可写
```
3. 再次写访问  
重复上面步骤

# write 接口操作的文件页  
由于通过write接口访问文件页时，会读取文件页到page cache,不会映射到任何进程地址空间，所有这种方式跟踪脏页是通过设置/清除页描述符脏标记来实现。

## 第一次写访问文件页  
会首先读文件页到page cache，然后将用户空间写缓冲区数据写到page cache，调用链如下：
```
/fs/ext4/file.c

ext4_file_write_iter

    ->__generic_file_write_iter
        ->generic_perform_write
            ->a_ops->write_begin   //写之前处理 分配page cache页
                ->__block_commit_write
                    ->mark_buffer_dirty
                        ->__set_page_dirty  //设置页为脏（设置页描述符脏标记）
            ->iov_iter_copy_from_user_atomic  //用户空间写缓冲区数据写到page cache页
```

## 3.2 脏页回写

```
write_cache_pages  //mm/page-writeback.c
	->clear_page_dirty_for_io 
        ->TestClearPageDirty(page) //清除页描述符脏标记
```
## 第二次写访问文件页  
脏页回写之前，页描述符脏标志位依然被置位，等待回写, 不需要设置页描述符脏标志位。

脏页回写之后，页描述符脏标志位是清零的，文件写页调用链会设置页描述符脏标志位。

# 总结  
1. 对于mmap映射的共享文件页，因为这个文件页可能会被多个进程共享到多个vma中，所以通过页表项的脏标志位来跟踪脏页：第一次写访问发生缺页异常会读文件页到page cache中并设置进程的页表项的脏标志，回写之前（clear_page_dirty_for_io完成之前），页表项的脏标志是置位的，回写的时候（clear_page_dirty_for_io的调用）会通过反向映射机制将所有映射这个页的页表项的脏标志位清零并设置只读权限，回写之后（clear_page_dirty_for_io完成之后），再次的写访问会发生写时复制缺页异常，再次设置页表项的脏标志位，如此重复，从而跟踪了脏页。

2. 对于直接通过write接口访问的文件页，因为这个文件页只会被读取到page cache中，并没有映射到任何进程地址空间，进程写访问是通过copy_from_user的方式，所以通过页描述符记录脏页。回写之前（clear_page_dirty_for_io完成之前），写文件的时候通过文件系统的写文件的调用链会设置页描述符脏标志位，回写的时候（clear_page_dirty_for_io的调用）会清除页描述符脏标志位，回写之后（clear_page_dirty_for_io完成之后），再次通过write接口写访问时，再次通过文件系统的写文件的调用链会再次设置页描述符脏标志位，如此重复，从而跟踪了脏页。