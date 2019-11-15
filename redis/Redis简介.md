## 基本数据类型及其应用场景：

类型	| 简介	| 特性|	场景
-|-|-|-
String(字符串)|	二进制安全	|可以包含任何数据,比如jpg图片或者序列化的对象,一个键最大能存储512M	|---
Hash(字典)|	键值对集合,即编程语言中的Map类型	|适合存储对象,并且可以像数据库中update一个属性一样只修改某一项属性值(Memcached中需要取出整个字符串反序列化成对象修改完再序列化存回去)	|存储、读取、修改用户属性
List(列表)|	链表(双向链表)	|增删快,提供了操作某一段元素的API	|1,最新消息排行等功能(比如朋友圈的时间线) 2,消息队列
Set(集合)|	哈希表实现,元素不重复|	1、添加、删除,查找的复杂度都是O(1) 2、为集合提供了求交集、并集、差集等操作 |	1、共同好友 2、利用唯一性,统计访问网站的所有独立ip 3、好友推荐时,根据tag求交集,大于某个阈值就可以推荐
Sorted Set(有序集合)|	将Set中的元素增加一个权重参数score,元素按score有序排列	 | 数据插入集合时,已经进行天然排序|1、排行榜 2、带权重的消息队列


## 配置说明
redis.conf 配置项说明如下：

配置项	| 说明
-|-
daemonize no	| Redis 默认不是以守护进程的方式运行，可以通过该配置项修改，使用 yes 启用守护进程（Windows 不支持守护线程的配置为 no ）
pidfile /var/run/redis.pid |	当 Redis 以守护进程方式运行时，Redis 默认会把 pid 写入 /var/run/redis.pid 文件，可以通过 pidfile 指定
port 6379|	指定 Redis 监听端口，默认端口为 6379，作者在自己的一篇博文中解释了为什么选用 6379 作为默认端口，因为 6379 在手机按键上 MERZ 对应的号码，而 MERZ 取自意大利歌女 Alessia Merz 的名字
bind 127.0.0.1	|绑定的主机地址
timeout 300	|当客户端闲置多长时间后关闭连接，如果指定为 0，表示关闭该功能	
loglevel notice	|指定日志记录级别，Redis 总共支持四个级别：debug、verbose、notice、warning，默认为 notice	
logfile stdout	|日志记录方式，默认为标准输出，如果配置 Redis 为守护进程方式运行，而这里又配置为日志记录方式为标准输出，则日志将会发送给 /dev/null	
databases 16	|设置数据库的数量，默认数据库为0，可以使用SELECT 命令在连接上指定数据库id	
save <seconds> <changes> | Redis 默认配置文件中提供了三个条件：save 900 1  ，  save 300 10  ， save 60 10000    分别表示 900 秒（15 分钟）内有 1 个更改，300 秒（5 分钟）内有 10 个更改以及 60 秒内有 10000 个更改。 指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可以多个条件配合
rdbcompression yes	|指定存储至本地数据库时是否压缩数据，默认为 yes，Redis 采用 LZF 压缩，如果为了节省 CPU 时间，可以关闭该选项，但会导致数据库文件变的巨大
dbfilename dump.rdb|	指定本地数据库文件名，默认值为 dump.rdb	
dir ./	|指定本地数据库存放目录	
slaveof <masterip> <masterport>	|设置当本机为 slav 服务时，设置 master 服务的 IP 地址及端口，在 Redis 启动时，它会自动从 master 进行数据同步
masterauth <master-password>|	当 master 服务设置了密码保护时，slav 服务连接 master 的密码
requirepass foobared	|设置 Redis 连接密码，如果配置了连接密码，客户端在连接 Redis 时需要通过 AUTH <password> 命令提供密码，默认关闭	
 maxclients 128	|设置同一时间最大客户端连接数，默认无限制，Redis 可以同时打开的客户端连接数为 Redis 进程可以打开的最大文件描述符数，如果设置 maxclients 0，表示不作限制。当客户端连接数到达限制时，Redis 会关闭新的连接并向客户端返回 max number of clients reached 错误信息
maxmemory <bytes>	|指定 Redis 最大内存限制，Redis 在启动时会把数据加载到内存中，达到最大内存后，Redis 会先尝试清除已到期或即将到期的 Key，当此方法处理 后，仍然到达最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作。Redis 新的 vm 机制，会把 Key 存放内存，Value 会存放在 swap 区
appendonly no	|指定是否在每次更新操作后进行日志记录，Redis 在默认情况下是异步的把数据写入磁盘，如果不开启，可能会在断电时导致一段时间内的数据丢失。因为 redis 本身同步数据文件是按上面 save 条件来同步的，所以有的数据会在一段时间内只存在于内存中。默认为 no	
appendfilename appendonly.aof	|指定更新日志文件名，默认为 appendonly.aof	
appendfsync everysec	|指定更新日志条件，共有 3 个可选值：no：表示等操作系统进行数据缓存同步到磁盘（快）always：表示每次更新操作后手动调用 fsync() 将数据写到磁盘（慢，安全）everysec：表示每秒同步一次（折中，默认值）	
vm-enabled no|	指定是否启用虚拟内存机制，默认值为 no，简单的介绍一下，VM 机制将数据分页存放，由 Redis 将访问量较少的页即冷数据 swap 到磁盘上，访问多的页面由磁盘自动换出到内存中（在后面的文章我会仔细分析 Redis 的 VM 机制）
vm-swap-file /tmp/redis.swap	|虚拟内存文件路径，默认值为 /tmp/redis.swap，不可多个 Redis 实例共享
vm-max-memory 0	|将所有大于 vm-max-memory 的数据存入虚拟内存，无论 vm-max-memory 设置多小，所有索引数据都是内存存储的(Redis 的索引数据 就是 keys)，也就是说，当 vm-max-memory 设置为 0 的时候，其实是所有 value 都存在于磁盘。默认值为 0
vm-page-size 32	|Redis swap 文件分成了很多的 page，一个对象可以保存在多个 page 上面，但一个 page 上不能被多个对象共享，vm-page-size 是要根据存储的 数据大小来设定的，作者建议如果存储很多小对象，page 大小最好设置为 32 或者 64bytes；如果存储很大大对象，则可以使用更大的 page，如果不确定，就使用默认值
vm-pages 134217728|	设置 swap 文件中的 page 数量，由于页表（一种表示页面空闲或使用的 bitmap）是在放在内存中的，，在磁盘上每 8 个 pages 将消耗 1byte 的内存。
vm-max-threads 4|	设置访问swap文件的线程数,最好不要超过机器的核数,如果设置为0,那么所有对swap文件的操作都是串行的，可能会造成比较长时间的延迟。默认值为4
glueoutputbuf yes|	设置在向客户端应答时，是否把较小的包合并为一个包发送，默认为开启
hash-max-zipmap-entries 64  ，  hash-max-zipmap-value 512	|指定在超过一定的数量或者最大的元素超过某一临界值时，采用一种特殊的哈希算法
activerehashing yes|指定是否激活重置哈希，默认为开启（后面在介绍 Redis 的哈希算法时具体介绍）
include /path/to/local.conf|	指定包含其它的配置文件，可以在同一主机上多个Redis实例之间使用同一份配置文件，而同时各个实例又拥有自己的


## Redis 命令简介：
参考： https://www.redis.net.cn/order/
### Redis 键(key) 命令
命令|	描述
-|-
Redis Type 命令	 |返回 key 所储存的值的类型。
Redis PEXPIREAT 命令	 |设置 key 的过期时间亿以毫秒计。
Redis PEXPIREAT 命令	 |设置 key 过期时间的时间戳(unix timestamp) 以毫秒计
Redis Rename 命令	 |修改 key 的名称
Redis PERSIST 命令	 |移除 key 的过期时间，key 将持久保持。
Redis Move 命令	 |将当前数据库的 key 移动到给定的数据库 db 当中。
Redis RANDOMKEY 命令	 |从当前数据库中随机返回一个 key 。
Redis Dump 命令	 |序列化给定 key ，并返回被序列化的值。
Redis TTL 命令	 |以秒为单位，返回给定 key 的剩余生存时间(TTL, time to live)。
Redis Expire 命令	 |seconds 为给定 key 设置过期时间。
Redis DEL 命令	 |该命令用于在 key 存在是删除 key。
Redis Pttl 命令	 |以毫秒为单位返回 key 的剩余的过期时间。
Redis Renamenx 命令	 |仅当 newkey 不存在时，将 key 改名为 newkey 。
Redis EXISTS 命令	 |检查给定 key 是否存在。
Redis Expireat 命令	 |EXPIREAT 的作用和 EXPIRE 类似，都用于为 key 设置过期时间。 不同在于 EXPIREAT 命令接受的时间参数是 UNIX 时间戳(unix timestamp)。
Redis Keys 命令	 |查找所有符合给定模式( pattern)的 key 。
### Redis 字符串(String) 命令
命令|	描述
-|-
Redis Setnx 命令	 |只有在 key 不存在时设置 key 的值。
Redis Getrange 命令	 |返回 key 中字符串值的子字符
Redis Mset 命令	 |同时设置一个或多个 key-value 对。
Redis Setex 命令	 |将值 value 关联到 key ，并将 key 的过期时间设为 seconds (以秒为单位)。
Redis SET 命令	 |设置指定 key 的值
Redis Get 命令	 |获取指定 key 的值。
Redis Getbit 命令	 |对 key 所储存的字符串值，获取指定偏移量上的位(bit)。
Redis Setbit 命令	 |对 key 所储存的字符串值，设置或清除指定偏移量上的位(bit)。
Redis Decr 命令	 |将 key 中储存的数字值减一。
Redis Decrby 命令	 |key 所储存的值减去给定的减量值（decrement） 。
Redis Strlen 命令	 |返回 key 所储存的字符串值的长度。
Redis Msetnx 命令	 |同时设置一个或多个 key-value 对，当且仅当所有给定 key 都不存在。
Redis Incrby 命令	 |将 key 所储存的值加上给定的增量值（increment） 。
Redis Incrbyfloat 命令	 |将 key 所储存的值加上给定的浮点增量值（increment） 。
Redis Setrange 命令	 |用 value 参数覆写给定 key 所储存的字符串值，从偏移量 offset 开始。
Redis Psetex 命令	 |这个命令和 SETEX 命令相似，但它以毫秒为单位设置 key 的生存时间，而不是像 SETEX 命令那样，以秒为单位。
Redis Append 命令	 |如果 key 已经存在并且是一个字符串， APPEND 命令将 value 追加到 key 原来的值的末尾。
Redis Getset 命令	 |将给定 key 的值设为 value ，并返回 key 的旧值(old value)。
Redis Mget 命令	 |获取所有(一个或多个)给定 key 的值。
Redis Incr 命令	 |将 key 中储存的数字值增一。
### Redis 列表(List) 命令
命令|	描述
-|-
Redis Lindex 命令	 |通过索引获取列表中的元素
Redis Rpush 命令	 |在列表中添加一个或多个值
Redis Lrange 命令	 |获取列表指定范围内的元素
Redis Rpoplpush 命令	 |移除列表的最后一个元素，并将该元素添加到另一个列表并返回
Redis Blpop 命令	 |移出并获取列表的第一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。
Redis Brpop 命令	 |移出并获取列表的最后一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。
Redis Brpoplpush 命令	 |从列表中弹出一个值，将弹出的元素插入到另外一个列表中并返回它； 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。
Redis Lrem 命令	 |移除列表元素
Redis Llen 命令	 |获取列表长度
Redis Ltrim 命令	 |对一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除。
Redis Lpop 命令	 |移出并获取列表的第一个元素
Redis Lpushx 命令	 |将一个或多个值插入到已存在的列表头部
Redis Linsert 命令	 |在列表的元素前或者后插入元素
Redis Rpop 命令	 |移除并获取列表最后一个元素
Redis Lset 命令	 |通过索引设置列表元素的值
Redis Lpush 命令	 |将一个或多个值插入到列表头部
Redis Rpushx 命令	 |为已存在的列表添加值
### Redis 哈希(Hash) 命令
命令|	描述
-|-
Redis Hmset 命令	 |同时将多个 field-value (域-值)对设置到哈希表 key 中。
Redis Hmget 命令	 |获取所有给定字段的值
Redis Hset 命令	 |将哈希表 key 中的字段 field 的值设为 value 。
Redis Hgetall 命令	 |获取在哈希表中指定 key 的所有字段和值
Redis Hget 命令	 |获取存储在哈希表中指定字段的值/td>
Redis Hexists 命令	 |查看哈希表 key 中，指定的字段是否存在。
Redis Hincrby 命令	 |为哈希表 key 中的指定字段的整数值加上增量 increment 。
Redis Hlen 命令	 |获取哈希表中字段的数量
Redis Hdel 命令	 |删除一个或多个哈希表字段
Redis Hvals 命令	 |获取哈希表中所有值
Redis Hincrbyfloat 命令	 |为哈希表 key 中的指定字段的浮点数值加上增量 increment 。
Redis Hkeys 命令	 |获取所有哈希表中的字段
Redis Hsetnx 命令	 |只有在字段 field 不存在时，设置哈希表字段的值。
### Redis 集合(Set) 命令
命令|	描述
-|-
Redis Sunion 命令	 |返回所有给定集合的并集
Redis Scard 命令	 |获取集合的成员数
Redis Srandmember 命令	 |返回集合中一个或多个随机数
Redis Smembers 命令	 |返回集合中的所有成员
Redis Sinter 命令	 |返回给定所有集合的交集
Redis Srem 命令	 |移除集合中一个或多个成员
Redis Smove 命令	 |将 member 元素从 source 集合移动到 destination 集合
Redis Sadd 命令	 |向集合添加一个或多个成员
Redis Sismember 命令	 |判断 member 元素是否是集合 key 的成员
Redis Sdiffstore 命令	 |返回给定所有集合的差集并存储在 destination 中
Redis Sdiff 命令	 |返回给定所有集合的差集
Redis Sscan 命令	 |迭代集合中的元素
Redis Sinterstore 命令	 |返回给定所有集合的交集并存储在 destination 中
Redis Sunionstore 命令	 |所有给定集合的并集存储在 destination 集合中
Redis Spop 命令	 |移除并返回集合中的一个随机元素
### Redis 有序集合(sorted set) 命令
命令|	描述
-|-
Redis Zrevrank 命令	 |返回有序集合中指定成员的排名，有序集成员按分数值递减(从大到小)排序
Redis Zlexcount 命令	 |在有序集合中计算指定字典区间内成员数量
Redis Zunionstore 命令	 |计算给定的一个或多个有序集的并集，并存储在新的 key 中
Redis Zremrangebyrank 命令	 |移除有序集合中给定的排名区间的所有成员
Redis Zcard 命令	 |获取有序集合的成员数
Redis Zrem 命令	 |移除有序集合中的一个或多个成员
Redis Zinterstore 命令	 |计算给定的一个或多个有序集的交集并将结果集存储在新的有序集合 key 中
Redis Zrank 命令	 |返回有序集合中指定成员的索引
Redis Zincrby 命令	 |有序集合中对指定成员的分数加上增量 increment
Redis Zrangebyscore 命令	 |通过分数返回有序集合指定区间内的成员
Redis Zrangebylex 命令	 |通过字典区间返回有序集合的成员
Redis Zscore 命令	 |返回有序集中，成员的分数值
Redis Zremrangebyscore 命令	 |移除有序集合中给定的分数区间的所有成员
Redis Zscan 命令	 |迭代有序集合中的元素（包括元素成员和元素分值）
Redis Zrevrangebyscore 命令	 |返回有序集中指定分数区间内的成员，分数从高到低排序
Redis Zremrangebylex 命令	 |移除有序集合中给定的字典区间的所有成员
Redis Zrevrange 命令	 |返回有序集中指定区间内的成员，通过索引，分数从高到底
Redis Zrange 命令	 |通过索引区间返回有序集合成指定区间内的成员
Redis Zcount 命令	 |计算在有序集合中指定区间分数的成员数
Redis Zadd 命令	 |向有序集合添加一个或多个成员，或者更新已存在成员的分数
### Redis 连接 命令
命令|	描述
-|-
Redis Echo 命令	 |打印字符串
Redis Select 命令	 |切换到指定的数据库
Redis Ping 命令	 |查看服务是否运行
Redis Quit 命令	 |关闭当前连接
Redis Auth 命令	 |验证密码是否正确
### Redis 服务器 命令
命令|	描述
-|-
Redis Client Pause 命令	 |在指定时间内终止运行来自客户端的命令
Redis Debug Object 命令	 |获取 key 的调试信息
Redis Flushdb 命令	 |删除当前数据库的所有key
Redis Save 命令	 |异步保存数据到硬盘
Redis Showlog 命令	 |管理 redis 的慢日志
Redis Lastsave 命令	 |返回最近一次 Redis 成功将数据保存到磁盘上的时间，以 UNIX 时间戳格式表示
Redis Config Get 命令	 |获取指定配置参数的值
Redis Command 命令	 |获取 Redis 命令详情数组
Redis Slaveof 命令	 |将当前服务器转变为指定服务器的从属服务器(slave server)
Redis Debug Segfault 命令	 |让 Redis 服务崩溃
Redis Flushall 命令	 |删除所有数据库的所有key
Redis Dbsize 命令	 |返回当前数据库的 key 的数量
Redis Bgrewriteaof 命令	 |异步执行一个 AOF（AppendOnly File） 文件重写操作
Redis Cluster Slots 命令	 |获取集群节点的映射数组
Redis Config Set 命令	 |修改 redis 配置参数，无需重启
Redis Command Info 命令	 |获取指定 Redis 命令描述的数组
Redis Shutdown 命令	 |异步保存数据到硬盘，并关闭服务器
Redis Sync 命令	 |用于复制功能(replication)的内部命令
Redis Client Kill 命令	 |关闭客户端连接
Redis Role 命令	 |返回主从实例所属的角色
Redis Monitor 命令	 |实时打印出 Redis 服务器接收到的命令，调试用
Redis Command Getkeys 命令	 |获取给定命令的所有键
Redis Client Getname 命令	 |获取连接的名称
Redis Config Resetstat 命令	 |重置 INFO 命令中的某些统计数据
Redis Command Count 命令	 |获取 Redis 命令总数
Redis Time 命令	 |返回当前服务器时间
Redis Info 命令	 |获取 Redis 服务器的各种信息和统计数值
Redis Config rewrite 命令	 |对启动 Redis 服务器时所指定的 redis.conf 配置文件进行改写
Redis Client List 命令	 |获取连接到服务器的客户端连接列表
Redis Client Setname 命令	 |设置当前连接的名称
Redis Bgsave 命令	 |在后台异步保存当前数据库的数据到磁盘
### Redis 脚本 命令
命令|	描述
-|-
Redis Script kill 命令	 |杀死当前正在运行的 Lua 脚本。
Redis Script Load 命令	 |将脚本 script 添加到脚本缓存中，但并不立即执行这个脚本。
Redis Eval 命令	 |执行 Lua 脚本。
Redis Evalsha 命令	 |执行 Lua 脚本。
Redis Script Exists 命令	 |查看指定的脚本是否已经被保存在缓存当中。
Redis Script Flush 命令	 |从脚本缓存中移除所有脚本。
### Redis 事务 命令
命令|	描述
-|-
Redis Exec 命令	 |执行所有事务块内的命令。
Redis Watch 命令	 |监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。
Redis Discard 命令	 |取消事务，放弃执行事务块内的所有命令。
Redis Unwatch 命令	 |取消 WATCH 命令对所有 key 的监视。
Redis Multi 命令	 |标记一个事务块的开始。
### Redis HyperLogLog 命令
命令|	描述
-|-
Redis Pgmerge 命令	 |将多个 HyperLogLog 合并为一个 HyperLogLog
Redis Pfadd 命令	 |添加指定元素到 HyperLogLog 中。
Redis Pfcount 命令	 |返回给定 HyperLogLog 的基数估算值。
### Redis 发布订阅 命令
命令|	描述
-|-
Redis Unsubscribe 命令	 |指退订给定的频道。
Redis Subscribe 命令	 |订阅给定的一个或多个频道的信息。
Redis Pubsub 命令	 |查看订阅与发布系统状态。
Redis Punsubscribe 命令	 |退订所有给定模式的频道。
Redis Publish 命令	 |将信息发送到指定的频道。
Redis Psubscribe 命令	 |订阅一个或多个符合给定模式的频道。
### Redis 地理位置(geo) 命令
命令|	描述
-|-
Redis GEOHASH 命令	 |返回一个或多个位置元素的 Geohash 表示
Redis GEOPOS 命令	 |从key里返回所有给定位置元素的位置（经度和纬度）
Redis GEODIST 命令	 |返回两个给定位置之间的距离
Redis GEORADIUS 命令	 |以给定的经纬度为中心， 找出某一半径内的元素
Redis GEOADD 命令	 |将指定的地理空间位置（纬度、经度、名称）添加到指定的key中
Redis GEORADIUSBYMEMBER 命令	 |找出位于指定范围内的元素，中心点是由给定的位置元素决定