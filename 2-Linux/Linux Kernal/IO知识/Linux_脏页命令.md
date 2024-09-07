 #Linux  #Linux_kernal #IO


> 在 /proc/vmstat 中查看页面缓存的统计信息：

```bash
cat /proc/vmstat | egrep "dirty|writeback"

nr_dirty 878   
nr_writeback 0   
nr_writeback_temp 0

# 有878个脏页等待写入磁盘。
```




