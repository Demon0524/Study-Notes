 #Linux  #Linux_kernal #IO
```bash
# 系统内核文件位置
/etc/sysctl.conf

$ sysctl -a | grep dirty
>
vm.dirty_background_bytes = 0
vm.dirty_background_ratio = 10
# 内存中存在的脏页数量百分比，当达到这个百分比是通过异步IO操作将脏数据写入磁盘。
vm.dirty_bytes = 0
vm.dirty_ratio = 20
# 内存中存在最大脏页数量百分比，当达到该百分比时通过同步IO操作将脏数据写入磁盘，此时所有新IO将被阻塞，直到脏数据全部写入磁盘。

vm.dirty_writeback_centisecs = 500 # 单位：1/100
# 唤醒一次 pdflush/flush/kmdflush 程序的时间间隔；该程序旨在保证内存和磁盘数据保持一致。
vm.dirty_expire_centisecs = 3000 # 单位：1/100
# 脏数据可存活时间；当pdflush程序运行时会检查脏数据存活是否超过该时间，当超过是将进行异步IO操作进行同步。
vm.dirtytime_expire_seconds = 43200
# 指定时间查看inode是否存在时间戳的更新
```



