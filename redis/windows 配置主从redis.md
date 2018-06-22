#### windows 配置主从redis

##### 1，运行两个redis实例

* 在redis的安装目录下，复制redis.windows.conf 为 redis.windows6380.conf
* 修改redis.windows6380.conf 目录中的端口问 port 6380
* 修改 logfile "server_log6380.txt"
* 可选：把 redis-server.exe redis.windows6380.conf 配置为服务 



#####2 ，更改**从redis**的配置文件：redis.windows6380.conf 

* 更改**主从配置**的参数：

*  slaveof <masterip> <masterport>  把#去掉:slaveof 127.0.0.1 6379  注意，slaveof前不能有空格

  其中6379 为主redis端口

* 一切更改完成后，启动从redis服务


这时，在主redis中修改 key value，会自动同步到从redis上



##### 3 windows一台主机上多实例运行 查看redis信息

* 用客户端连接查看信息

  ~~~shel
  redis-cli.exe -h 127.0.0.1 -p 6379
  ~~~

* 查看该redis服务的所有信息

  ~~~shell
  info

  127.0.0.1:6379> info
  # Server
  redis_version:3.2.100
  redis_git_sha1:00000000
  redis_git_dirty:0
  redis_build_id:dd26f1f93c5130ee
  redis_mode:standalone
  os:Windows
  arch_bits:64
  multiplexing_api:WinSock_IOCP
  process_id:9224
  run_id:d513d7a9dae5d9bde1b5c8895458440692ca243b
  tcp_port:6379
  uptime_in_seconds:475
  uptime_in_days:0
  hz:10
  lru_clock:2904092
  executable:D:\Program Files\Redis\"D:\Program Files\Redis\redis-server.exe"
  config_file:D:\Program Files\Redis\redis.windows-service.conf

  # Clients
  connected_clients:8
  client_longest_output_list:0
  client_biggest_input_buf:0
  blocked_clients:0

  # Memory
  used_memory:836584
  used_memory_human:816.98K
  used_memory_rss:798680
  used_memory_rss_human:779.96K
  used_memory_peak:998944
  used_memory_peak_human:975.53K
  total_system_memory:0
  total_system_memory_human:0B
  used_memory_lua:37888
  used_memory_lua_human:37.00K
  maxmemory:0
  maxmemory_human:0B
  maxmemory_policy:noeviction
  mem_fragmentation_ratio:0.95
  mem_allocator:jemalloc-3.6.0

  # Persistence
  loading:0
  rdb_changes_since_last_save:0
  rdb_bgsave_in_progress:0
  rdb_last_save_time:1529630273
  rdb_last_bgsave_status:ok
  rdb_last_bgsave_time_sec:-1
  rdb_current_bgsave_time_sec:-1
  aof_enabled:0
  aof_rewrite_in_progress:0
  aof_rewrite_scheduled:0
  aof_last_rewrite_time_sec:-1
  aof_current_rewrite_time_sec:-1
  aof_last_bgrewrite_status:ok
  aof_last_write_status:ok

  # Stats
  total_connections_received:7
  total_commands_processed:2916
  instantaneous_ops_per_sec:5
  total_net_input_bytes:204397
  total_net_output_bytes:6818043
  instantaneous_input_kbps:0.44
  instantaneous_output_kbps:1.28
  rejected_connections:0
  sync_full:0
  sync_partial_ok:0
  sync_partial_err:0
  expired_keys:0
  evicted_keys:0
  keyspace_hits:0
  keyspace_misses:0
  pubsub_channels:1
  pubsub_patterns:0
  latest_fork_usec:0
  migrate_cached_sockets:0

  # Replication
  role:slave
  master_host:127.0.0.1
  master_port:6381
  master_link_status:up
  master_last_io_seconds_ago:0
  master_sync_in_progress:0
  slave_repl_offset:3378361
  slave_priority:100
  slave_read_only:1
  connected_slaves:0
  master_repl_offset:0
  repl_backlog_active:0
  repl_backlog_size:1048576
  repl_backlog_first_byte_offset:0
  repl_backlog_histlen:0

  # CPU
  used_cpu_sys:0.36
  used_cpu_user:0.14
  used_cpu_sys_children:0.00
  used_cpu_user_children:0.00

  # Cluster
  cluster_enabled:0

  # Keyspace
  db0:keys=4,expires=0,avg_ttl=0
  127.0.0.1:6379>
  ~~~

  可以看到redis的服务内存情况、连接数、主从信息等

* 单独查看主从信息

  ~~~shell
  info Replication

  127.0.0.1:6381> info Replication
  # Replication
  role:master
  connected_slaves:2
  slave0:ip=127.0.0.1,port=6380,state=online,offset=3369926,lag=0
  slave1:ip=127.0.0.1,port=6379,state=online,offset=3369926,lag=1
  master_repl_offset:3369926
  repl_backlog_active:1
  repl_backlog_size:1048576
  repl_backlog_first_byte_offset:2321351
  repl_backlog_histlen:1048576
  ~~~

  ​







