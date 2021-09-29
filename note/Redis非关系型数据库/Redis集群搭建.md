### 一、Windows 环境下搭建 Redis 集群

#### 1 采用 Ruby 搭建集群

`注：针对 Redis 5 之前的版本`

##### 1.1 部署 Redis 服务

**集群模式：** Redis Cluster

**三主三从：** master：7000	    slave：6000

​				   master：7001        slave：6001

​				   master：7002        slave：6002

**（1）下载 Redis-x64-3.2.100.zip 压缩包，解压缩 Redis-6001 文件夹下，复制出其余 5 个文件夹**

https://github.com/MicrosoftArchive/redis/releases  →  Redis-x64-3.2.100.zip

**（2）修改 6 个文件夹下的 redis.windows.conf 配置文件**

```properties
# 绑定端口
portal 6000

# 开启集群
cluster-enabled yes

# 集群自动生成配置文件
cluster-config-file nodes-6379.conf

# 集群节点超时时限，节点超过这个时间未响应就断定是宕机
cluster-node-timeout 15000

# 将写指令记录到appendonly.aof文件
appendonly yes
```

**（3）依次启动 6 个 Redis 服务，在文件夹下打开 CMD 命令行窗口，执行以下指令**

```shell
# 修改CMD命令行窗口标题 
title Redis6000

# 启动Redis服务器
redis-server.exe redis.windows.conf
```

**注意：**

​         **重新启动 Redis 集群，为了方便集群搭建，需删除 Redis 文件夹下的 appendonly.aof（AOF 增量指令持久化文件）、dump.rdb（RDB 全量快照持久化文件）和 nodes-6001.conf （集群节点信息配置文件）**

##### 1.2 搭建 Ruby 环境

**（1）安装 Ruby**

https://rubyinstaller.org/downloads/	→	WITHOUT DEVKIT	→	Ruby 2.4.10-1(x64)

**（2）安装 RubyGems**

https://rubygems.org/pages/download	→	ZIP

在 RubyGems 文件夹下打开 CMD 命令行窗口，执行指令 `ruby setup.rb`，再执行指令 `gem install redis`

在 Ruby24-x64\lib\ruby\gems\2.4.0\gems 文件夹下， 查看 Redis 版本为 redis-4.2.5

##### 1.3 下载集群脚本redis-trib.rb

https://raw.githubusercontent.com/antirez/redis/4.0/src/redis-trib.rb 复制内容给保存在 Redis-6000 文件夹下的 redis-trib.rb 中，可使用以下指令

```shell
# 将网络文件下载到本地指定路径下
curl -L https://raw.githubusercontent.com/antirez/redis/4.0/src/redis-trib.rb -o F:\RedisClusterRuby\redis-trib.rb
```

##### 1.4 创建集群

在 redis-trib.rb 文件所在的文件夹下打开 CMD 命令行窗口，再执行以下指令

```shell
redis-trib.rb create --replicas 1 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:6000 127.0.0.1:6001 127.0.0.1:6002 
```

创建过程

```shell
Can I set the above configuration? (type 'yes' to accept): yes
```

注：redis-trib.rb 脚本应与使用 RubyGems 安装的 Redis 版本对应，否则会出现警告 redis-trib.rb is not longer available!

#### 2 采用 redis-cli 搭建集群

`注：针对 Redis 5 之后的版本`

##### 2.1 部署 Redis 服务

**集群模式：** Redis Cluster

**三主三从：** master：7100		slave：6100

​					master：7101        slave：6101

​					master：7102        slave：6102

**（1）下载 Redis-x64-5.0.10.zip 压缩包，解压缩 Redis-6100 文件夹下，复制出其余 5 个文件夹**

https://github.com/tporadowski/redis/releases	→	Redis-x64-5.0.10.zip

**（2）修改 6 个文件夹下的 redis.windows.conf 配置文件**

```properties
# 绑定端口
portal 6000

# 开启集群
cluster-enabled yes

# 集群自动生成配置文件
cluster-config-file nodes-6379.conf

# 集群节点超时时限，节点超过这个时间未响应就断定是宕机
cluster-node-timeout 15000

# 将写指令记录到appendonly.aof文件
appendonly yes
```

**（3）依次启动 6 个 Redis 服务，在所在文件夹下打开 CMD 命令行窗口，执行以下指令**

```shell
# 修改CMD命令行窗口标题 
title Redis6100

# 启动Redis服务器
redis-server.exe redis.windows.conf
```

##### 2.2 创建集群

在任意节点 redis-cli.exe 所在文件夹下打开 CMD 命令行窗口，执行下列指令创建集群

```shell
redis-cli --cluster create --cluster-replicas 1 127.0.0.1:7100 127.0.0.1:7101 127.0.0.1:7102 127.0.0.1:6100 127.0.0.1:6101 127.0.0.1:6102
```

创建过程

```shell
Can I set the above configuration? (type 'yes' to accept): yes
```

使用下列指令

```shell
redis-cli --cluster check 127.0.0.1:6100
```

查看集群节点信息

```shell
S: 040dde4aeca5650fc4ad2be15126a59973b71041 127.0.0.1:6100
   slots: (0 slots) slave
   replicates ec1cd366e2bdc76ee46b42c40c2d2a42101d5694
M: ec1cd366e2bdc76ee46b42c40c2d2a42101d5694 127.0.0.1:7100
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
M: 56ae60154a1ecc35d9bfbce845740b2d3d07dd5b 127.0.0.1:7101
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: 5c0c5b7133dd395bd18572cc58c5a4ab359f311b 127.0.0.1:6102
   slots: (0 slots) slave
   replicates 4b26074aac7e0be04e78ec6f130f0eb79e62c3e0
M: 4b26074aac7e0be04e78ec6f130f0eb79e62c3e0 127.0.0.1:7102
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
S: 27203d9ac3b736efa85cb52216af9dd8262b9822 127.0.0.1:6101
   slots: (0 slots) slave
   replicates 56ae60154a1ecc35d9bfbce845740b2d3d07dd5b
```

查看指令格式提示

```shell
redis-cli --cluster help
```

### 二、集群高可用测试

#### 1 数据读写测试

连接 Redis 集群服务器，使用 Redis 指令进行读写操作

```shell
# 连接Redis客户端
redis-cli -c -h 127.0.0.1 -p 7000

# 查看集群节点信息
cluster nodes

# 退出当前客户端
exit
```

#### 2 节点故障测试

在 redis-trib.rb 所在文件夹下打开 CMD 命令行窗口，执行以下指令

```shell
redis-trib.rb check 127.0.0.1:6000
```

或

```shell
redis-cli --cluster check 127.0.0.1:6100
```

查看集群节点信息

```shell
S: 3891d29233105176daff094728214ce833c372d6 127.0.0.1:6000
   slots: (0 slots) slave
   replicates fbf894e0a0ca5d38cc44ad8a53960e3dbb2392c4
S: 5c686ff0e5bc2d02c971f58b762786d7a1aef105 127.0.0.1:6002
   slots: (0 slots) slave
   replicates cc2d1db8cbd233f60e55791b329d4be08ee0942e
S: 741951e2d08e52942580f2311e2d479ddd0ec618 127.0.0.1:6001
   slots: (0 slots) slave
   replicates 6a470a11f205dbfa140ec35ac20fa029763bbaff
M: 6a470a11f205dbfa140ec35ac20fa029763bbaff 127.0.0.1:7001
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
M: fbf894e0a0ca5d38cc44ad8a53960e3dbb2392c4 127.0.0.1:7000
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
M: cc2d1db8cbd233f60e55791b329d4be08ee0942e 127.0.0.1:7002
   slots:10923-16383 (5461 slots) master
   1 additional replica(s)
```

##### 2.1 主节点故障

关闭 127.0.0.1:7001 主节点服务器（删除对应 CMD 命令行窗口），查看集群节点信息

```shell
[ERR] Not all 16384 slots are covered by nodes.
```

集群在**极短时间**内，将原主节点对应的从节点服务器 127.0.0.1:6001 替代成为主节点

```shell
S: 3891d29233105176daff094728214ce833c372d6 127.0.0.1:6000
   slots: (0 slots) slave
   replicates fbf894e0a0ca5d38cc44ad8a53960e3dbb2392c4
S: 5c686ff0e5bc2d02c971f58b762786d7a1aef105 127.0.0.1:6002
   slots: (0 slots) slave
   replicates cc2d1db8cbd233f60e55791b329d4be08ee0942e
M: 741951e2d08e52942580f2311e2d479ddd0ec618 127.0.0.1:6001
   slots:5461-10922 (5462 slots) master
   0 additional replica(s)
M: fbf894e0a0ca5d38cc44ad8a53960e3dbb2392c4 127.0.0.1:7000
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
M: cc2d1db8cbd233f60e55791b329d4be08ee0942e 127.0.0.1:7002
   slots:10923-16383 (5461 slots) master
   1 additional replica(s)
```

##### 2.2 主节点重启

重新启动 127.0.0.1:7001 节点服务器

```shell
title Redis7001
redis-server.exe redis.windows.conf
```

查看集群节点信息，127.0.0.1:7001 节点成为 127.0.0.1:6001 节点的从节点

```shell
S: 3891d29233105176daff094728214ce833c372d6 127.0.0.1:6000
   slots: (0 slots) slave
   replicates fbf894e0a0ca5d38cc44ad8a53960e3dbb2392c4
S: 5c686ff0e5bc2d02c971f58b762786d7a1aef105 127.0.0.1:6002
   slots: (0 slots) slave
   replicates cc2d1db8cbd233f60e55791b329d4be08ee0942e
M: 741951e2d08e52942580f2311e2d479ddd0ec618 127.0.0.1:6001
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
S: 6a470a11f205dbfa140ec35ac20fa029763bbaff 127.0.0.1:7001
   slots: (0 slots) slave
   replicates 741951e2d08e52942580f2311e2d479ddd0ec618
M: fbf894e0a0ca5d38cc44ad8a53960e3dbb2392c4 127.0.0.1:7000
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
M: cc2d1db8cbd233f60e55791b329d4be08ee0942e 127.0.0.1:7002
   slots:10923-16383 (5461 slots) master
   1 additional replica(s)
```

##### 2.3 从节点故障

关闭 127.0.0.1:6002 从节点服务器（删除对应 CMD 命令行窗口），集群不会有任何影响，查看集群节点信息

```shell
S: 3891d29233105176daff094728214ce833c372d6 127.0.0.1:6000
   slots: (0 slots) slave
   replicates fbf894e0a0ca5d38cc44ad8a53960e3dbb2392c4
M: 741951e2d08e52942580f2311e2d479ddd0ec618 127.0.0.1:6001
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
S: 6a470a11f205dbfa140ec35ac20fa029763bbaff 127.0.0.1:7001
   slots: (0 slots) slave
   replicates 741951e2d08e52942580f2311e2d479ddd0ec618
M: fbf894e0a0ca5d38cc44ad8a53960e3dbb2392c4 127.0.0.1:7000
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
M: cc2d1db8cbd233f60e55791b329d4be08ee0942e 127.0.0.1:7002
   slots:10923-16383 (5461 slots) master
   0 additional replica(s)
```

##### 2.4 从节点重启

重启 127.0.0.1:6002 从节点服务器，集群不会有任何影响，查看集群节点信息

```shell
S: 3891d29233105176daff094728214ce833c372d6 127.0.0.1:6000
   slots: (0 slots) slave
   replicates fbf894e0a0ca5d38cc44ad8a53960e3dbb2392c4
S: 5c686ff0e5bc2d02c971f58b762786d7a1aef105 127.0.0.1:6002
   slots: (0 slots) slave
   replicates cc2d1db8cbd233f60e55791b329d4be08ee0942e
M: 741951e2d08e52942580f2311e2d479ddd0ec618 127.0.0.1:6001
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
S: 6a470a11f205dbfa140ec35ac20fa029763bbaff 127.0.0.1:7001
   slots: (0 slots) slave
   replicates 741951e2d08e52942580f2311e2d479ddd0ec618
M: fbf894e0a0ca5d38cc44ad8a53960e3dbb2392c4 127.0.0.1:7000
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
M: cc2d1db8cbd233f60e55791b329d4be08ee0942e 127.0.0.1:7002
   slots:10923-16383 (5461 slots) master
   1 additional replica(s)
```

#### 3 集群节点扩容

##### 3.1 添加主节点

启动待添加的主节点服务器

在 redis-trib.rb 所在文件夹下 CMD 命令行窗口，执行下列指令

```shell
redis-trib.rb add-node 127.0.0.1:7003 127.0.0.1:7000
```

或

```shell
redis-cli --cluster add-node 127.0.0.1:7103 127.0.0.1:7100
```

127.0.0.1:7103 节点已加入主节点，但没有分配哈希槽，查看集群节点信息

```shell
S: 9bc23f82e9b5dd85fd272005651aeae4ff6e5743 127.0.0.1:6100
   slots: (0 slots) slave
   replicates 4dea3c445b9b1630f35224dd2d068531b9ae1145
M: e296087c3ca7cfa5dc2fc0e4ac9699b92942491f 127.0.0.1:7103
   slots: (0 slots) master
M: 6551176c418c6798b46a68308a0192d5104eebbd 127.0.0.1:7101
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: ee93b89b902f6fec25e8b16c397504fedb2740e6 127.0.0.1:6102
   slots: (0 slots) slave
   replicates 6551176c418c6798b46a68308a0192d5104eebbd
M: 4dea3c445b9b1630f35224dd2d068531b9ae1145 127.0.0.1:7102
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
S: c164815256f54c9ca0398b45222ea6687bf582ef 127.0.0.1:6101
   slots: (0 slots) slave
   replicates 49a8acf8614bc8cb7c782bcff003c403063c46e9
M: 49a8acf8614bc8cb7c782bcff003c403063c46e9 127.0.0.1:7100
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
```

分配哈希槽，任意一个主节点服务器都可，执行下列指令

```shell
redis-trib.rb reshard 127.0.0.1:7000
```

或

```shell
redis-cli --cluster reshard 127.0.0.1:7103
```

分配哈希槽过程

```shell
# 每个主节点对应的哈希槽数量
How many slots do you want to move <from 1 to 16384>? 4096

# 待添加的主节点ID
What is the receiving node ID? e296087c3ca7cfa5dc2fc0e4ac9699b92942491f

Please enter all the source node IDs.
	Type 'all' to use all the nodes as source nodes for the hash slots.
	Type 'done' once you entered all the source nodes IDs.
# 从其他节点抽取4000个哈希槽
Source node #1:all

# 执行上述分配计划
Do you want to proceed with the proposed reshard plan (yes/no)?yes
```

查看集群节点信息

```shell
S: 9bc23f82e9b5dd85fd272005651aeae4ff6e5743 127.0.0.1:6100
   slots: (0 slots) slave
   replicates 4dea3c445b9b1630f35224dd2d068531b9ae1145
M: e296087c3ca7cfa5dc2fc0e4ac9699b92942491f 127.0.0.1:7103
   slots:[0-1364],[5461-6826],[10923-12287] (4096 slots) master
M: 6551176c418c6798b46a68308a0192d5104eebbd 127.0.0.1:7101
   slots:[6827-10922] (4096 slots) master
   1 additional replica(s)
S: ee93b89b902f6fec25e8b16c397504fedb2740e6 127.0.0.1:6102
   slots: (0 slots) slave
   replicates 6551176c418c6798b46a68308a0192d5104eebbd
M: 4dea3c445b9b1630f35224dd2d068531b9ae1145 127.0.0.1:7102
   slots:[12288-16383] (4096 slots) master
   1 additional replica(s)
S: c164815256f54c9ca0398b45222ea6687bf582ef 127.0.0.1:6101
   slots: (0 slots) slave
   replicates 49a8acf8614bc8cb7c782bcff003c403063c46e9
M: 49a8acf8614bc8cb7c782bcff003c403063c46e9 127.0.0.1:7100
   slots:[1365-5460] (4096 slots) master
   1 additional replica(s)
```

##### 3.2 添加从节点

查看集群信息

```shell
S: 9bc23f82e9b5dd85fd272005651aeae4ff6e5743 127.0.0.1:6100
   slots: (0 slots) slave
   replicates 4dea3c445b9b1630f35224dd2d068531b9ae1145
M: e296087c3ca7cfa5dc2fc0e4ac9699b92942491f 127.0.0.1:7103
   slots:[0-1364],[5461-6826],[10923-12287] (4096 slots) master
M: 6551176c418c6798b46a68308a0192d5104eebbd 127.0.0.1:7101
   slots:[6827-10922] (4096 slots) master
   1 additional replica(s)
S: ee93b89b902f6fec25e8b16c397504fedb2740e6 127.0.0.1:6102
   slots: (0 slots) slave
   replicates 6551176c418c6798b46a68308a0192d5104eebbd
M: 4dea3c445b9b1630f35224dd2d068531b9ae1145 127.0.0.1:7102
   slots:[12288-16383] (4096 slots) master
   1 additional replica(s)
S: c164815256f54c9ca0398b45222ea6687bf582ef 127.0.0.1:6101
   slots: (0 slots) slave
   replicates 49a8acf8614bc8cb7c782bcff003c403063c46e9
M: 49a8acf8614bc8cb7c782bcff003c403063c46e9 127.0.0.1:7100
   slots:[1365-5460] (4096 slots) master
   1 additional replica(s)
```

启动待添加的从节点服务器

在 redis-trib.rb 所在文件夹下 CMD 命令行窗口，执行下列指令

```shell
redis-trib.rb add-node --slave 127.0.0.1:6003 127.0.0.1:7003
```

或

```shell
redis-cli --cluster add-node 127.0.0.1:6103 127.0.0.1:7100 --cluster-slave --cluster-master-id e296087c3ca7cfa5dc2fc0e4ac9699b92942491f
```

查看集群信息

```shell
S: 9bc23f82e9b5dd85fd272005651aeae4ff6e5743 127.0.0.1:6100
   slots: (0 slots) slave
   replicates 4dea3c445b9b1630f35224dd2d068531b9ae1145
M: e296087c3ca7cfa5dc2fc0e4ac9699b92942491f 127.0.0.1:7103
   slots:[0-1364],[5461-6826],[10923-12287] (4096 slots) master
   1 additional replica(s)
M: 6551176c418c6798b46a68308a0192d5104eebbd 127.0.0.1:7101
   slots:[6827-10922] (4096 slots) master
   1 additional replica(s)
S: ee93b89b902f6fec25e8b16c397504fedb2740e6 127.0.0.1:6102
   slots: (0 slots) slave
   replicates 6551176c418c6798b46a68308a0192d5104eebbd
M: 4dea3c445b9b1630f35224dd2d068531b9ae1145 127.0.0.1:7102
   slots:[12288-16383] (4096 slots) master
   1 additional replica(s)
S: c164815256f54c9ca0398b45222ea6687bf582ef 127.0.0.1:6101
   slots: (0 slots) slave
   replicates 49a8acf8614bc8cb7c782bcff003c403063c46e9
M: 49a8acf8614bc8cb7c782bcff003c403063c46e9 127.0.0.1:7100
   slots:[1365-5460] (4096 slots) master
   1 additional replica(s)
S: 18a6e254743befd394877e2b96a054d3209d1f8f 127.0.0.1:6103
   slots: (0 slots) slave
   replicates e296087c3ca7cfa5dc2fc0e4ac9699b92942491f
```

##### 3.3 移除主节点

**（1）迁移哈希槽**

迁移前对应的集群节点信息

```shell
S: 9bc23f82e9b5dd85fd272005651aeae4ff6e5743 127.0.0.1:6100
   slots: (0 slots) slave
   replicates 4dea3c445b9b1630f35224dd2d068531b9ae1145
M: e296087c3ca7cfa5dc2fc0e4ac9699b92942491f 127.0.0.1:7103
   slots:[0-1364],[5461-6826],[10923-12287] (4096 slots) master
   1 additional replica(s)
M: 6551176c418c6798b46a68308a0192d5104eebbd 127.0.0.1:7101
   slots:[6827-10922] (4096 slots) master
   1 additional replica(s)
S: ee93b89b902f6fec25e8b16c397504fedb2740e6 127.0.0.1:6102
   slots: (0 slots) slave
   replicates 6551176c418c6798b46a68308a0192d5104eebbd
M: 4dea3c445b9b1630f35224dd2d068531b9ae1145 127.0.0.1:7102
   slots:[12288-16383] (4096 slots) master
   1 additional replica(s)
S: c164815256f54c9ca0398b45222ea6687bf582ef 127.0.0.1:6101
   slots: (0 slots) slave
   replicates 49a8acf8614bc8cb7c782bcff003c403063c46e9
M: 49a8acf8614bc8cb7c782bcff003c403063c46e9 127.0.0.1:7100
   slots:[1365-5460] (4096 slots) master
   1 additional replica(s)
S: 18a6e254743befd394877e2b96a054d3209d1f8f 127.0.0.1:6103
   slots: (0 slots) slave
   replicates e296087c3ca7cfa5dc2fc0e4ac9699b92942491f
```

将 127.0.0.1:7103 主节点的哈希槽迁移到 127.0.0.1:7101主节点

```shell
# 要移除的主节点ip 端口号
redis-trib.rb reshard 127.0.0.1:7003  
```


迁移过程

```shell
# 移除的slot数
How many slots do you want to move (from 1 to 16384)? 4000  
# 接受slot的主节点ID
What is the receiving node ID? 339a50df26f4722f14faba2a8fe3cad508059e88 
# 待迁移的主节点ID 
Source node #1:d33aba61b519cbc0b5d7f33b825863d8f78a8925
Source node #2:done  
Do you want to proceed with the proposed reshard plan (yes/no)? yes
```

或

``` shell
redis-cli --cluster reshard 127.0.0.1:7103 --cluster-from e296087c3ca7cfa5dc2fc0e4ac9699b92942491f --cluster-to 6551176c418c6798b46a68308a0192d5104eebbd --cluster-slots 4096
```

主节点哈希槽迁移后，其对应的从节点也会转移到 127.0.0.1:7101 主节点，查看集群信息

```shell
S: 9bc23f82e9b5dd85fd272005651aeae4ff6e5743 127.0.0.1:6100
   slots: (0 slots) slave
   replicates 4dea3c445b9b1630f35224dd2d068531b9ae1145
M: e296087c3ca7cfa5dc2fc0e4ac9699b92942491f 127.0.0.1:7103
   slots: (0 slots) master
M: 6551176c418c6798b46a68308a0192d5104eebbd 127.0.0.1:7101
   slots:[0-1364],[5461-12287] (8192 slots) master
   2 additional replica(s)
S: ee93b89b902f6fec25e8b16c397504fedb2740e6 127.0.0.1:6102
   slots: (0 slots) slave
   replicates 6551176c418c6798b46a68308a0192d5104eebbd
M: 4dea3c445b9b1630f35224dd2d068531b9ae1145 127.0.0.1:7102
   slots:[12288-16383] (4096 slots) master
   1 additional replica(s)
S: c164815256f54c9ca0398b45222ea6687bf582ef 127.0.0.1:6101
   slots: (0 slots) slave
   replicates 49a8acf8614bc8cb7c782bcff003c403063c46e9
M: 49a8acf8614bc8cb7c782bcff003c403063c46e9 127.0.0.1:7100
   slots:[1365-5460] (4096 slots) master
   1 additional replica(s)
S: 18a6e254743befd394877e2b96a054d3209d1f8f 127.0.0.1:6103
   slots: (0 slots) slave
   replicates 6551176c418c6798b46a68308a0192d5104eebbd
```

**（2）移除主节点**

```shell
redis-trib.rb del-node 127.0.0.1:7003 d33aba61b519cbc0b5d7f33b825863d8f78a8925
```

或

```shell
redis-cli --cluster del-node 127.0.0.1:7103 e296087c3ca7cfa5dc2fc0e4ac9699b92942491f
```

查看集群节点信息

```shell
S: 9bc23f82e9b5dd85fd272005651aeae4ff6e5743 127.0.0.1:6100
   slots: (0 slots) slave
   replicates 4dea3c445b9b1630f35224dd2d068531b9ae1145
M: 6551176c418c6798b46a68308a0192d5104eebbd 127.0.0.1:7101
   slots:[0-1364],[5461-12287] (8192 slots) master
   2 additional replica(s)
S: ee93b89b902f6fec25e8b16c397504fedb2740e6 127.0.0.1:6102
   slots: (0 slots) slave
   replicates 6551176c418c6798b46a68308a0192d5104eebbd
M: 4dea3c445b9b1630f35224dd2d068531b9ae1145 127.0.0.1:7102
   slots:[12288-16383] (4096 slots) master
   1 additional replica(s)
S: c164815256f54c9ca0398b45222ea6687bf582ef 127.0.0.1:6101
   slots: (0 slots) slave
   replicates 49a8acf8614bc8cb7c782bcff003c403063c46e9
M: 49a8acf8614bc8cb7c782bcff003c403063c46e9 127.0.0.1:7100
   slots:[1365-5460] (4096 slots) master
   1 additional replica(s)
S: 18a6e254743befd394877e2b96a054d3209d1f8f 127.0.0.1:6103
   slots: (0 slots) slave
   replicates 6551176c418c6798b46a68308a0192d5104eebbd
```

##### 3.4 移除从节点

移除从节点前，查看集群节点信息

```shell
S: 3891d29233105176daff094728214ce833c372d6 127.0.0.1:6000
   slots: (0 slots) slave
   replicates fbf894e0a0ca5d38cc44ad8a53960e3dbb2392c4
M: bfbdd28c914e77e156fb8711420869db6e132fb7 127.0.0.1:7003
   slots:0-1360,5461-7075,10923-12284 (4338 slots) master
   0 additional replica(s)
S: 5c686ff0e5bc2d02c971f58b762786d7a1aef105 127.0.0.1:6002
   slots: (0 slots) slave
   replicates cc2d1db8cbd233f60e55791b329d4be08ee0942e
M: 741951e2d08e52942580f2311e2d479ddd0ec618 127.0.0.1:6001
   slots:7076-10922 (3847 slots) master
   1 additional replica(s)
S: 6a470a11f205dbfa140ec35ac20fa029763bbaff 127.0.0.1:7001
   slots: (0 slots) slave
   replicates 741951e2d08e52942580f2311e2d479ddd0ec618
M: fbf894e0a0ca5d38cc44ad8a53960e3dbb2392c4 127.0.0.1:7000
   slots:1361-5460 (4100 slots) master
   1 additional replica(s)
M: cc2d1db8cbd233f60e55791b329d4be08ee0942e 127.0.0.1:7002
   slots:12285-16383 (4099 slots) master
   1 additional replica(s)
```

移除从节点，执行下列指令

```shell
redis-trib.rb del-node 127.0.0.1:7001 6a470a11f205dbfa140ec35ac20fa029763bbaff
```

或

```shell
redis-cli --cluster del-node 127.0.0.1:6103 18a6e254743befd394877e2b96a054d3209d1f8f
```

移除从节点后，查看集群节点信息

```shell
S: 3891d29233105176daff094728214ce833c372d6 127.0.0.1:6000
   slots: (0 slots) slave
   replicates fbf894e0a0ca5d38cc44ad8a53960e3dbb2392c4
M: bfbdd28c914e77e156fb8711420869db6e132fb7 127.0.0.1:7003
   slots:0-1360,5461-7075,10923-12284 (4338 slots) master
   0 additional replica(s)
S: 5c686ff0e5bc2d02c971f58b762786d7a1aef105 127.0.0.1:6002
   slots: (0 slots) slave
   replicates cc2d1db8cbd233f60e55791b329d4be08ee0942e
M: 741951e2d08e52942580f2311e2d479ddd0ec618 127.0.0.1:6001
   slots:7076-10922 (3847 slots) master
   0 additional replica(s)
M: fbf894e0a0ca5d38cc44ad8a53960e3dbb2392c4 127.0.0.1:7000
   slots:1361-5460 (4100 slots) master
   1 additional replica(s)
M: cc2d1db8cbd233f60e55791b329d4be08ee0942e 127.0.0.1:7002
   slots:12285-16383 (4099 slots) master
   1 additional replica(s)
```

### 三、SpringBoot 整合 Redis 集群

**两种客户端：lettuce 和 jedis**

#### 1 Lettuce客户端

**（1）引入 jar 包**

```xml
<!-- spring-boot-starter-data-redis:2.4.1 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<!-- 对象池：2.9.0 -->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency>
```

**（2）配置文件**

```yaml
spring:
  redis:
    cluster:
      nodes:
        - 127.0.0.1:6100
        - 127.0.0.1:6101
        - 127.0.0.1:6102
        - 127.0.0.1:7100
        - 127.0.0.1:7101
        - 127.0.0.1:7102
      # 客户端最大重定向次数
      max-redirects: 3
    # 注入commons-pool2依赖后，才可使用lettuce标签名: spring.redis.lettuce.pool
    # 未注入commons-pool2依赖后，不可使用lettuce标签名，无法启动服务: spring.redis.pool
    lettuce:
      pool:
        # 连接池最大连接数
        max-active: 50
        # 连接池最大空闲连接数
        max-idle: 10
        # 连接池最小空闲连接数
        min-idle: 5
    # 连接超时时间（ms）
    timeout: 1000
```

#### 2 Jedis 客户端

**（1）引入 jar 包**

```xml
<!-- spring-boot-starter-data-redis:2.4.1 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <exclusions>
        <exclusion>
            <groupId>io.lettuce</groupId>
            <artifactId>lettuce-core</artifactId>
        </exclusion>
    </exclusions>
</dependency>

<!-- jedis客户端：3.3.0 -->
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
</dependency>
```

**（2）配置文件**

```yaml
spring:
  redis:
    cluster:
      nodes:
        - 127.0.0.1:6100
        - 127.0.0.1:6101
        - 127.0.0.1:6102
        - 127.0.0.1:7100
        - 127.0.0.1:7101
        - 127.0.0.1:7102
      # 客户端最大重定向次数
      max-redirects: 3
    jedis:
      pool:
        # 连接池最大连接数
        max-active: 50
        # 连接池最大空闲连接数
        max-idle: 10
        # 连接池最小空闲连接数
        min-idle: 5
    # 连接超时时间（ms）
    timeout: 1000
```
