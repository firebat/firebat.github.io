+++
date = '2016-12-07T00:00:00-08:00'
title = '存储的发展'
tags = ["存储"]
+++
## 文件系统
### Professor Eggert's Dumb File System (~1974)
![img](./images/eggertfs.png)

### FAT - File Allocation Table (~1977)
![img](./images/fat.png)

### 
![img](./images/unix.jpg)

### 
![img](./images/pdp-11-40.jpg)

### Unix File System (~1973)
![img](./images/ufs.png)

### Lions' Commentary on UNIX 6th Edition (1976)
![img](./images/lions.jpg)

### Unix Version 7 (1979)
- 支持x86架构
- 最后一个可自由散布的版本
- 不可对学生提供原始码

### Berkeley Software Distribution (BSD)
- 1977 1BSD ex
- 1978 2BSD vi, csh, berknet  
- 1979 3BSD VAX虚拟内存
- 1980 4BSD job, TCP/IP
- 1981 4.1a rcp, rsh..

![img](./images/bsd.jpg)

### 
![img](./images/billjoy.jpg)

Bill Joy - BSD, Sun co-founder, vi, csh, tcp/ip, NFS

### Berkeley Fast File System (~1980 4.1b)
![img](./images/bffs.png)

### 
![img](./images/EricAllmanAndMarshallKirkMcKusick.jpg)

Eric Allman & Marshall Kirk McKusick and their last son, Tyson
   
### 
There is some sort of perverse pleasure in knowing that it's basically impossible to send a piece of hate mail through the Internet without its being touched by a gay program. That's kind of funny. —Eric Allman

### 
![img](./images/richard.jpg)

GNU Opearting System (1984)

### 
![img](./images/andrew.jpg)

Minix Operating System (1984 ~ 1987)

### 
![img](./images/minix15-floppies-scr-01.jpg)

### 
![img](./images/linus-torvalds.jpg)

### 

gcc on minix-386 doesn't optimize? (1991/3/29)
Hello everybody,
I've had minix for a week now, and have upgraded to 386-minix (nice), and duly downloaded gcc for minix. Yes, it works.

PS and Minix-386 (1991/4/1)
RTFSC (Read the F**ing Source Code :-) - It is heavily commented and the solution should be obvious (take that with a grain of salt, it certainly stumped me for a while :-). Simply change the 4 define-lines at the
start (well, I'd guess about line 40) that read

#define kernel "/usr/src/kernel/kernel" ...

What would you like to see most in minix? (1991/8/26)
Hello everybody out there using minix -
I'm doing a (free) operating system (just a hobby, won't be big and professional like gnu) for 386(486) AT clones.  This has been brewing since april, and is starting to get ready.  I'd like any feedback on things people like/dislike in minix, as my OS resembles it somewhat (same physical layout of the file-system (due to practical reasons) among other things).

### 
![img](./images/vfs.gif)

## Network File System
Network File System (NFS) is a distributed file system protocol originally developed by Sun Microsystems in 1984, allowing a user on a client computer to access files over a computer network much like local storage is accessed. NFS, like many other protocols, builds on the Open Network Computing Remote Procedure Call (ONC RPC) system. The NFS is an open standard defined in Request for Comments (RFC), allowing anyone to implement the protocol.
 
### 
![img](./images/NAS.gif)

### MooseFS
MooseFS is a fault tolerant, network distributed file system. It spreads data over several physical servers which are visible to the user as one resource. For standard file operations MooseFS acts as other Unix-alike file systems

- Master server - a single machine managing the whole filesystem, storing metadata for every file.
- Chunk servers -  any number of commodity servers storing files data and synchronizing it among themselves.
- Client -  any number of machines using `mfsmount` process to communicate with the managing server and with chunkservers.

### 
![img](./images/mfs_write.png)

### 
![img](./images/mfs_read.png)


## 分布式系统
### CAP
- Consistence 一致性

  all nodes see the same data at the same time
  
- Availability 可用性

  Reads and writes always succeed
  
- Partition Tolerance 分区容错性
  
  the system continues to operate despite arbitrary message loss or failure of part of the system
  
### NWR
*Dynamo: Amazons Highly Available Key-value Store*
#+BEGIN_VERSE
W > N / 2
W + R > N
#+END_VERSE

- N 副本数
- W 一次成功的写操作必须完成的写副本数
- R 一次成功的读操作需要读的副本数

### 
![img](./images/object-mapping-simple-consistent-hashing.png)

一致性哈希

### 
![img](./images/object-mapping-modified-consistent-hashing.png)

### 
- [[http://www.cnblogs.com/yuxc/archive/2012/06/22/2558312.html][深入云存储系统Swift核心组件：Ring实现原理剖析)
- [[http://yikun.github.io/2016/06/09/一致性哈希算法的理解与实践][一致性哈希算法的理解与实践)


## 对象存储
![img](./images/bitcask.jpg)

![img](./images/Hash_Tree.svg.png)

Merkle tree


### BeansDB (2009)
Beansdb is a distributed key-value storage system designed for large scale online system, aiming for high avaliablility and easy management. It took the ideas from Amazon's Dynamo, then made some simplify to Keep It Simple Stupid (KISS).

- High availability data storage with multi readable and writable repications
- _Soft state and final consistency, synced with hash tree_
- Easy Scaling out without interrupting online service
- High performance read/write for a key-value based object
- _Configurable availability/consistency by N,W,R_
- _Memcache protocol compatibility_

### Supported memcache commands
- get (key, ?key, @bucket)
- set
- append
- incr
- delete
- stats
- flush _all


```sh
$ telnet 127.0.0.1 7900
get @
VALUE @ 0 170
0/ 43141 9514
1/ 0 0
2/ 42462 2
3/ 39177 2
4/ 37044 1
5/ 44330 3
6/ 45589 3
7/ 28358 3
8/ 17224 3
9/ 5155 3
a/ 53006 1
b/ 27133 4
c/ 26906 3
d/ 0 0
e/ 29129 4
f/ 56710 2

get @42
VALUE @42 0 50
/pay/ucardlogo/1509/2a/1b1fdecfdb5d31.jpg 12806 1

get ?/pay/ucardlogo/1509/2a/1b1fdecfdb5d31.jpg
VALUE ?/pay/ucardlogo/1509/2a/1b1fdecfdb5d31.jpg 0 25
1 12806 1 2853 1442285794
```


![img](./images/beansdb.png)


### 用法
- 面向10M以内文件, 如图片, 音频 (默认最高50M,可以改代码)
- RAID10 数据存3份
- 使用ngx_memc
- qclient-beansdb (xmemcached with NWR)
- rsync同步大文件, sync.py同步小对象
- 数百T数据, 数十亿文件特别棒

### Swift
OpenStack Object Storage（Swift）前身是 Rackspace Cloud Files 项目，于 2010 年贡献给 OpenStack 社区，是 OpenStack 最早的两个项目之一。Swift 可在比较便宜的通用硬件上构筑具有极强可扩展性和数据持久性的存储系统，支持多租户，通过 RESTful API 提供对容器（Container）和对象的 CRUD 操作。


### 
WSGI是为Python语言定义的Web服务器和Web应用程序或框架之间的一种简单而通用的接口。
```sh
$ paster create
Selected and implied templates:
  PasteScript#basic_package  A basic setuptools-enabled package

Enter project name: demo
Variables:
  egg:      demo
  package:  demo
  project:  demo
Enter version (Version (like 0.1)) ['']: 
```

```python
# demo/__init__.py
def app(environ, start_response): # or app = Flask(__name__)
    start_response('200 OK', [('Content-Type', 'text/plain')])
    yield "Hello world!\n"

def factory(global_config, **local_config):
    print global_config
    print local_config
    return app
```

### 
PasteDeploy是一个用来查找和配置 WSGI 应用的系统, 配置支持app, filter, pipeline,factory, type, section等

```ini
# demo.ini
[DEFAULT]
title=my website

[server:main]
use = egg:Paste#http
# use=egg:gunicorn#main
# workers=2
# worker_class=gevent

[app:main]
use=call:demo:app_factory
# set title=demo site
database=mysql://xxxx
```

```sh
$ paster serve demo.ini
{'__file__': '/home/tom/demo/main.ini', 'here': '/home/tom/demo', 'title': 'my website'}
{'database': 'mysql://xxxx'}
Starting server in PID 3756.
serving on http://127.0.0.1:8080
```

### 映射关系
![img](./images/swift.png)

### 数据节点
```sh
$ tree /srv/node/sdb -L 5
├── accounts
│   ├── 4422
│   │   └── d84
│   │       └── 8a32648058cf2185fab166279963fd84
│   │           ├── 8a32648058cf2185fab166279963fd84.db
│   │           └── 8a32648058cf2185fab166279963fd84.db.pending
│   └── 632
│       └── 84a
│           └── 13c312c524cb52e59041e34e9c4f384a
├── containers (同上)
├── objects
│  ├── 1000
│  │   ├── 6a5
│  │   │   └── 1f462ba176864aa5f5c35e28b533b6a5
│  │   │       └── 1473149348.59743.data
│  │   ├── aba
│  │   │   └── 1f42bcc1bcf1ec440d5d5aec24ddaaba
│  │   │       └── 1464159081.76922.ts
└── tmp
```

### Swift Proxy
![img](./images/swift_proxy.jpg)

### Swift Ring
![img](./images/swift_ring.jpg)

### Swift Get
![img](./images/swift_get.jpg)

### Swift Post
![img](./images/swift_post.jpg)

### Swift All
![img](./images/swift_all.png)

### Swift组件
- Proxy Server 对外提供对象服务API, 查找服务地址并转发用户请求
- Account Server 提供账户元数据和统计信息，并维护所含容器列表的服务
- Container Server 提供容器元数据和统计信息，并维护所含对象列表的服务
- Object Server 提供对象元数据和内容服务，对象以文件的形式存储在文件系统
- Replicator 检测本地分区副本和远程副本是否一致, Push更新副本，删除文件
- Updater 高负载时，将更新请求插入队列，之后进行异步更新
- Auditor 检查对象，容器和账户的完整性，发现比特级进行隔离, 并同步副本
- Account Reaper 账户清理服务，删除容器和对象数据
- Authentication Server 验证用户身份
- Cache Server 缓存认证Token, 账户容器的存在信息, 采用Memcached.
   
### 
```sh
$ swift-ring-builder object.builder
object.builder, build version 6
8192 partitions, 2.000000 replicas, 1 regions, 3 zones, 6 devices, 0.02 balance, 0.00 dispersion
The minimum number of hours before a partition can be reassigned is 1
The overload factor is 0.00% (0.000000)
Devices:    id  region  zone      ip address  port  replication ip  replication port      name weight partitions balance meta
             0       1     1     10.86.41.96  6000     10.86.41.96              6000        d1   1.00       2731    0.01 
             1       1     1     10.86.41.96  6000     10.86.41.96              6000        d2   1.00       2731    0.01 
             2       1     2    10.86.35.155  6000    10.86.35.155              6000        d1   1.00       2730   -0.02 
             3       1     2    10.86.35.155  6000    10.86.35.155              6000        d2   1.00       2730   -0.02 
             4       1     3    10.86.35.169  6000    10.86.35.169              6000        d1   1.00       2731    0.01 
             5       1     3    10.86.35.169  6000    10.86.35.169              6000        d2   1.00       2731    0.01 
```

### 
```sh
$ swift-get-nodes /etc/swift/object.ring.gz AUTH_test testdt /no/joss-logo.png
Account  AUTH_test
Containertestdt
Object   /no/joss-logo.png

Partition1778
Hash     3793f1105a148c4a1f180a83a7289f67

Server:Port Device10.86.35.155:6000 d1
Server:Port Device10.86.41.96:6000 d2
Server:Port Device10.86.35.169:6000 d1 [Handoff]
Server:Port Device10.86.41.96:6000 d1 [Handoff]


curl -I -XHEAD "http://10.86.35.155:6000/d1/1778/AUTH_test/testdt//no/joss-logo.png"
curl -I -XHEAD "http://10.86.41.96:6000/d2/1778/AUTH_test/testdt//no/joss-logo.png"
curl -I -XHEAD "http://10.86.35.169:6000/d1/1778/AUTH_test/testdt//no/joss-logo.png" # [Handoff]
curl -I -XHEAD "http://10.86.41.96:6000/d1/1778/AUTH_test/testdt//no/joss-logo.png" # [Handoff]


Use your own device location of servers:
such as "export DEVICE=/srv/node"
ssh 10.86.35.155 "ls -lah ${DEVICE:-/srv/node*}/d1/objects/1778/f67/3793f1105a148c4a1f180a83a7289f67"
ssh 10.86.41.96 "ls -lah ${DEVICE:-/srv/node*}/d2/objects/1778/f67/3793f1105a148c4a1f180a83a7289f67"
ssh 10.86.35.169 "ls -lah ${DEVICE:-/srv/node*}/d1/objects/1778/f67/3793f1105a148c4a1f180a83a7289f67" # [Handoff]
ssh 10.86.41.96 "ls -lah ${DEVICE:-/srv/node*}/d1/objects/1778/f67/3793f1105a148c4a1f180a83a7289f67" # [Handoff]

$ ls /srv/node/d2/objects/1778/f67/3793f1105a148c4a1f180a83a7289f67/
1470970382.20003.data
```
### 
```sh

$ swift-recon object -d  -l -r
===============================================================================
--> Starting reconnaissance on 3 hosts
===============================================================================
[2016-10-19 15:54:25] Checking on replication
[replication_failure] low: 0, high: 0, avg: 0.0, total: 0, Failed: 0.0%, no_result: 0, reported: 1
[replication_success] low: 5408, high: 5408, avg: 5408.0, total: 5408, Failed: 0.0%, no_result: 0, reported: 1
[replication_time] low: 0, high: 0, avg: 0.3, total: 0, Failed: 66.7%, no_result: 2, reported: 1
[replication_attempted] low: 5408, high: 5408, avg: 5408.0, total: 5408, Failed: 0.0%, no_result: 0, reported: 1
Oldest completion was NEVER by 10.86.35.169:6000.
Most recent completion was 2016-10-19 07:43:02 (11 minutes ago) by 10.86.41.96:6000.
===============================================================================
[2016-10-19 15:51:09] Checking load averages
[5m_load_avg] low: 0, high: 0, avg: 0.0, total: 0, Failed: 0.0%, no_result: 0, reported: 3
[15m_load_avg] low: 0, high: 0, avg: 0.0, total: 0, Failed: 0.0%, no_result: 0, reported: 3
[1m_load_avg] low: 0, high: 0, avg: 0.0, total: 0, Failed: 0.0%, no_result: 0, reported: 3
===============================================================================
[2016-10-19 15:51:09] Checking disk usage now
Distribution Graph:
 15%    1 **********************************
 17%    1 **********************************
 18%    1 **********************************
 19%    2 *********************************************************************
 20%    1 **********************************
Disk usage: space used: 22106923008 of 119937073152
Disk usage: space free: 97830150144 of 119937073152
Disk usage: lowest: 15.46%, highest: 20.05%, avg: 18.4321014571%
===============================================================================
```

### 用法
- 通用存储方案, No RAID
- 多存储策略
- 大文件切分并发上传
- 对象超时机制
- 单容器200w记录无压力
- rsync带宽限制
- recon/informant监控
- 调整 /etc/rsyslog.d/10-swift.conf

### Haystack
Haystack是Facebook的海量图片存储系统

- Finding a needle in Haystack: Facebooks photo storage
- 总量260B, 20P, 增量1M 60T/W, 峰值550K/S
- 每张图4种尺寸, 分别存3份
- 三种服务 Cache, Directory, Store
- 两种元信息, Application 生成URL用 / Filesystem 读取文件用

### 
#+BEGIN_CENTER
![img](./images/fb_nfs.jpg)
#+END_CENTER

### 优化尽头
- POSIX读文件效率低
- 缓存文件句柄, 修改内核增加open by filehandle方法减少IO
- 社交网络长尾效应
- 冗余元数据(文件权限, 所属用户)浪费空间

### Needle结构
:PROPERTIES:
:ARTICLE:  smaller
:END:

#+BEGIN_CENTER
![img](./images/needle.jpg)
#+END_CENTER
- cookie 创建时生成, 后续的访问签名
- key 图片编号
- alternate key 图片规格
- padding 8字节对齐

### Needle索引文件
:PROPERTIES:
:ARTICLE:  smaller
:END:

#+BEGIN_CENTER
![img](./images/needle_index.png)

两级映射 needle.key => (needle.alternate key => meta)
#+END_CENTER

### Store服务

- 挂载物理卷(/hay/haystack/<logical valume id>) 作为存储基本单元, 如: 100个100G的物理卷提供10T存储量,系统级用xfs
- 小文件合并成文件块，通过offset和size 直接定位读取
- 异步追加 元信息 到索引文件，加速启动, 运行时全量加载到内存 (由于是异步操作，可能和数据块不一致，启动时需要做对比

### Directory服务  
本质上是一个映射表, 用数据库保存信息，提供一个PHP接口

- 维护逻辑卷到物理卷的映射
- 判断请求转发到Store还是CDN
- 负载均衡
- 维护Store状态, 是否空闲/只读
- 为了便于维护, Store可写与容量有关; 扩容时无需文件迁移

### Cache服务
- 缓存文件, 相当于CDN
- 新上传文件直接缓存, 减少Store读写并发

### HayStack
![img](./images/haystack.jpg)


### 文件访问
`http ://<cdn>/<Cache>/<machine id>/<logical volume, photo>`

- CDN 用 logical volume, photo 查缓存, 命中则返回，否则走Cache机
- Cache  同上, 否则走Store机
- Machine 同上, 否则返回错误

### 开源实现
- SeaweedFS 作者为Facebook员工
- bfs - Bilibili File System

```sh
$ go get github.com/chrislusf/seaweedfs/weed

$ weed -master

$ weed volume -dir="./data1" -mserver="localhost:9333" -port=8080
$ weed volume -dir="./data2" -mserver="localhost:9333" -port=8081

$ curl -X POST http://127.0.0.1:9333/dir/assign
{"fid":"8,01995db7d7","url":"127.0.0.1:8080","publicUrl":"127.0.0.1:8080","count":1}
{"fid":"9,02af1b78ce","url":"127.0.0.1:8080","publicUrl":"127.0.0.1:8080","count":1}
$ curl -X PUT -F file=@v6.pdf http://127.0.0.1:8080/8,01995db7d7
{"name":"v6.pdf","size":189495}

$ curl http://localhost:9333/dir/lookup?volumeId=8
{"volumeId":"8","locations":[{"url":"127.0.0.1:8080","publicUrl":"127.0.0.1:8080"}]}
$ curl -I http://127.0.0.1:8080/8,01995db7d7
HTTP/1.1 200 OK
Accept-Ranges: bytes
Content-Disposition: inline; filename="v6.pdf"
Content-Length: 235616
Content-Type: application/pdf
```

## Nginx
通过lua路由请求、通过插件实时图片处理

### 
```sh
$ cat nginx/conf/vhost/vh_beansdb.conf

    upstream beansdb_0_b {
        ...
    }

    location / {
        set $memc_key $uri;
        set_by_lua_file $server 'conf/vhost/vh_beansdb_fnv1a.lua';
        memc_pass $server;
    }

$ cat nginx/conf/vhost/vh_beansdb_fnv1a.lua
require "libhash"

buckets = {
  "beansdb_0_b",  "beansdb_0_b",  "beansdb_0_b",  "beansdb_0_b",
  "beansdb_0_b",  "beansdb_0_b",  "beansdb_0_b",  "beansdb_0_b",
  "beansdb_0_b",  "beansdb_0_b",  "beansdb_0_b",  "beansdb_0_b",
  "beansdb_c_f",  "beansdb_c_f",  "beansdb_c_f",  "beansdb_c_f"
}
local bucket_size = 268435456; -- 1 << 32 / bucket_count;
local hash = libhash.fnv1a(ngx.var.memc_key, string.len(ngx.var.memc_key));
local bucket = math.floor(hash / bucket_size);
return buckets[bucket+1];
```

### 图片处理配置
```sh
server {
    gm_buffer 10M;

    location /gm {
         alias imgs;

         set $resize 400x300>;
         set $rotate 180;

         gm strip;
         gm auto-orient;

         gm resize $resize;
         gm rotate $rotate;
         gm quality 85;

         gm gravity SouthEast;
         gm composite -geometry +10+10 -image '/your/watermark/file' -min-width 100;
    }
}
```

### 
```c
typedef ngx_int_t (*ngx_http_gm_command_pt)(ngx_http_request_t *r, void *option, Image **image,
        ImageInfo *image_info);
typedef ngx_int_t (*ngx_http_gm_parse_pt)(ngx_conf_t *cf, ngx_array_t *args, void **option);
typedef ngx_buf_t *(*ngx_http_gm_out_pt)(ngx_http_request_t *r, Image *image);

typedef struct {
    ngx_str_t                   name;
    ngx_http_gm_command_pt      handler;
    ngx_http_gm_parse_pt        option_parse_handler;
    ngx_http_gm_out_pt          out_handler;
} ngx_http_gm_command_t;

typedef struct {
    ngx_str_t                   option;
    ngx_http_complex_value_t   *option_cv;
} ngx_http_gm_option_t;

typedef struct {
    ngx_http_gm_command_t       *command;
    void                        *option;
} ngx_http_gm_command_info_t;

typedef struct {
    ngx_array_t                 *cmds;          /## ngx_http_gm_command_info_t */
    size_t                       buffer_size;
    ngx_flag_t                   enable;
} ngx_http_gm_conf_t;
```

### 
```c
ngx_int_t gm_quality_image(ngx_http_request_t *r, void *option, Image **image, ImageInfo *image_info)
{
    u_char                     *quality;
    int                         value;

    dd("start");

    quality = gm_get_option_value(r, (ngx_http_gm_option_t *)option);
    if (quality == NULL) {
        ngx_log_error(NGX_LOG_ERR, r->connection->log, 0, "gm filter: get quality failed");
        return NGX_ERROR;
    }

    if (ngx_strlen(quality) == 0){
        return NGX_OK;
    }

    ngx_log_debug1(NGX_LOG_DEBUG_HTTP, r->connection->log, 0, "gm filter: quality \"%s\"", quality);

    value = atoi((char *) quality);
    if (value <= 0 || value > 100) {
        return NGX_ERROR;
    }

    image_info->quality = value;

    return NGX_OK;
}
```

## 总结
- 对症下药
- 工具链很重要
- 抄是一门艺术
- RTFSC

Thank You

