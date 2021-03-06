## Postgres参数
- postgres.conf文件配置
```conf
max_connections = 300       # (change requires restart)   
unix_socket_directories = '.'   # comma-separated list of directories
shared_buffers = 194GB       # 尽量用数据库管理内存，减少双重缓存，提高使用效率
huge_pages = on           # on, off, or try  ，使用大页
work_mem = 256MB # min 64kB  ， 减少外部文件排序的可能，提高效率
maintenance_work_mem = 2GB  # min 1MB  ， 加速建立索引
autovacuum_work_mem = 2GB   # min 1MB, or -1 to use maintenance_work_mem  ， 加速垃圾回收
dynamic_shared_memory_type = mmap      # the default is the first option
vacuum_cost_delay = 0      # 0-100 milliseconds   ， 垃圾回收不妥协，极限压力下，减少膨胀可能性
bgwriter_delay = 10ms       # 10-10000ms between rounds    ， 刷shared buffer脏页的进程调度间隔，尽量高频调度，减少用户进程申请不到内存而需要主动刷脏页的可能（导致RT升高）。
bgwriter_lru_maxpages = 1000   # 0-1000 max buffers written/round ,  一次最多刷多少脏页
bgwriter_lru_multiplier = 10.0          # 0-10.0 multipler on buffers scanned/round  一次扫描多少个块，上次刷出脏页数量的倍数
effective_io_concurrency = 2           # 1-1000; 0 disables prefetching ， 执行节点为bitmap heap scan时，预读的块数。从而
wal_level = minimal         # minimal, archive, hot_standby, or logical ， 如果现实环境，建议开启归档。
synchronous_commit = off    # synchronization level;    ， 异步提交
wal_sync_method = open_sync    # the default is the first option  ， 因为没有standby，所以写xlog选择一个支持O_DIRECT的fsync方法。
full_page_writes = off      # recover from partial page writes  ， 生产中，如果有增量备份和归档，可以关闭，提高性能。
wal_buffers = 1GB           # min 32kB, -1 sets based on shared_buffers  ，wal buffer大小，如果大量写wal buffer等待，则可以加大。
wal_writer_delay = 10ms         # 1-10000 milliseconds  wal buffer调度间隔，和bg writer delay类似。
commit_delay = 20           # range 0-100000, in microseconds  ，分组提交的等待时间
commit_siblings = 9        # range 1-1000  , 有多少个事务同时进入提交阶段时，就触发分组提交。
checkpoint_timeout = 55min  # range 30s-1h  时间控制的检查点间隔。
max_wal_size = 320GB    #   2个检查点之间最多允许产生多少个XLOG文件
checkpoint_completion_target = 0.99     # checkpoint target duration, 0.0 - 1.0  ，平滑调度间隔，假设上一个检查点到现在这个检查点之间产生了100个XLOG，则这次检查点需要在产生100*checkpoint_completion_target个XLOG文件的过程中完成。PG会根据这些值来调度平滑检查点。
random_page_cost = 1.0     # same scale as above  , 离散扫描的成本因子，本例使用的SSD IO能力足够好
effective_cache_size = 240GB  # 可用的OS CACHE
log_destination = 'csvlog'  # Valid values are combinations of
logging_collector = on          # Enable capturing of stderr and csvlog
log_truncate_on_rotation = on           # If on, an existing log file with the
update_process_title = off
track_activities = off
autovacuum = on    # Enable autovacuum subprocess?  'on'
autovacuum_max_workers = 4 # max number of autovacuum subprocesses    ，允许同时有多少个垃圾回收工作进程。
autovacuum_naptime = 6s  # time between autovacuum runs   ， 自动垃圾回收探测进程的唤醒间隔
autovacuum_vacuum_cost_delay = 0    # default vacuum cost delay for  ， 垃圾回收不妥协
```