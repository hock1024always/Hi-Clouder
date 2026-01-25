# Redis入门与应用

## Redis的技术全景

Redis一个开源的基于键值对（Key-Value）NoSQL数据库。使用ANSI C语言编写、支持网络、基于内存但支持持久化。性能优秀，并提供多种语言的API。

我们要首先理解一点，我们把Redis称为KV数据库，键值对数据库，那就可以把Redis内部的存储视为存在着一个巨大的Map，对Map的操作无非就是get和put，然后通过key操作这个key所对应的value，而这个value的类型可以多种多样，也就是Redis为我们提供的那些数据结构，比如字符串（String）、哈希(Hash)等等。

Redis就这么简单吗？从某种程度上说，在解决问题时，拥有了系统观，就意味着你能有依据、有章法地定位和解决问题。

那么，如何高效地形成系统观呢？本质上就是，Redis 的知识都包括什么呢？简单来说，就是“两大维度，三大特性”

![image.png](./图片/ba2269e45c1b4a77bf097ff40de3aa8e.png)

### 两大维度

两大维度：应用维度、底层原理维度

我们知道，缓存和集群是Redis 的两大广泛的应用场景。同时Redis 丰富的数据模型，就导致它有很多零碎的应用场景，很多很杂。而且，还有一些问题隐藏得比较深，只有特定的业务场景下（比如亿级访问压力场景）才会出现，所以同时还必须精通Redis的数据结构。

**Redis应用场景**

1.缓存

缓存机制几乎在所有的大型网站都有使用，合理地使用缓存不仅可以加快数据的访问速度，而且能够有效地降低后端数据源的压力。Redis提供了键值过期时间设置,并且也提供了灵活控制最大内存和内存溢出后的淘汰策略。可以这么说,一个合理的缓存设计能够为一个网站的稳定保驾护航。

一般MySQL数据库写的并发是600/s，读的2000/s,对于大型互联网项目的百万并发，根本扛不住，Redis的官方显示Redis能够单台达到10W+/s的并发。

2.排行榜系统

排行榜系统几乎存在于所有的网站，例如按照热度排名的排行榜，按照发布时间的排行榜，按照各种复杂维度计算出的排行榜，Redis提供了列表和有序集合数据结构，合理地使用这些数据结构可以很方便地构建各种排行榜系统。

3.计数器应用

计数器在网站中的作用至关重要，例如视频网站有播放数、电商网站有浏览数，为了保证数据的实时性，每一次播放和浏览都要做加1的操作，如果并发量很大对于传统关系型数据的性能是一种挑战。Redis天然支持计数功能而且计数的性能也非常好,可以说是计数器系统的重要选择。

4.社交网络

赞/踩、粉丝、共同好友/喜好、推送、下拉刷新等是社交网站的必备功能，由于社交网站访问量通常比较大,而且传统的关系型数据不太适合保存这种类型的数据，Redis提供的数据结构可以相对比较容易地实现这些功能。

5.消息队列系统

消息队列系统可以说是一个大型网站的必备基础组件，因为其具有业务解耦、非实时业务削峰等特性。Redis提供了发布订阅功能和阻塞队列的功能，虽然和专业的消息队列比还不够足够强大,但是对于一般的消息队列功能基本可以满足。这个是Redis的作者参考了Kafka做的拓展。

### 三大特性

三大特性：高性能、高可靠和高可扩展

高性能：包括线程模型、数据结构、持久化、网络框架；
高可靠：包括主从复制、哨兵机制；
高可扩：包括数据分片、负载均衡。

因为Redis的应用场景非常多，不同的公司有不同的玩法，但如何不掌握三高这个特性的话，你会遇到以下问题：

1、数据结构的复杂度、跨 CPU 核的访问会导致CPU飙升的问题

2、主从同步和 AOF 的内存竞争，这些会导致内存问题

3、在 SSD 上做快照的性能抖动，这些会导致存储持久化的问题

4、多实例时的异常网络丢包的问题



## Redis的版本选择与安装

在Redis的版本计划中，版本号第二位为奇数，为非稳定版本，如2.7、2.9、3.1；版本号第二为偶数，为稳定版本如2.6、2.8、3.0；一般来说当前奇数版本是下一个稳定版本的开发版本，如2.9是3.0的开发版本。

同时Redis的安装也非常简单，到Redis的官网（[Download | Redis](https://redis.io/download/)），下载对应的版本，简单几个命令安装即可。

![image.png](./图片/f7ff9554827540b8807bb273a30313a8.png)

### **Redis的linux安装**

```
wget https://download.redis.io/releases/redis-6.2.7.tar.gz
tar xzf redis-6.2.7.tar.gz
cd redis-6.2.7/
make
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/7cbc709d5bc4498fa11dc2d8aaa04ddb.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/d56090bdd1fd4e90a1449c6c836d0d05.png)

安装后源码和执行目录会混在一起，为了方便，我做了一次install

```
make install PREFIX=/home/tianming/redis/redis
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/e697ede889734f18a1a9623e6b15f0dd.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/7bb695c563e74321b6055e4eac3bee03.png)

因为Redis的安装一般来说对于系统依赖很少，只依赖了Linux系统基本的类库，所以安装很少出问题

**安装常见问题**

如果执行make命令报错：cc 未找到命令，原因是虚拟机系统中缺少gcc，执行下面命令安装gcc：

```
yum -y install gcc automake autoconf libtool make
```

如果执行make命令报错：致命错误:jemalloc/jemalloc.h: 没有那个文件或目录，则需要在make指定分配器为libc。执行下面命令即可正常编译：

```
make MALLOC=libc
```

### Redis的启动

Redis编译完成后，会生成几个可执行文件，这些文件各有各的作用，我们现在先简单了解下，后面的课程会陆续说到和使用这些可执行文件。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/55847140e0b744c382acf8186fe4ffb9.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/210bdc3df1d941cea0d7f6449105310b.png)

一般来说redis-server和redis-cli这些平时用得最多。

Redis有三种方法启动Redis:默认配置、带参数启动、配置文件启动。

#### 默认配置

使用Redis的默认配置来启动，在bin目录下直接输入 ./redis-server

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/471e7f9ca54f4ace8f020d505014d602.png)

可以看到直接使用redis-server启动Redis后，会打印出一些日志，通过日志可以看到一些信息：

当前的Redis版本的是64位的6.2.7，默认端口是6379。Redis建议要使用配置文件来启动。

**因为直接启动无法自定义配置，所以这种方式是不会在生产环境中使用的。**

#### 带参数启动

redis-server加上要修改配置名和值(可以是多对)，没有设置的配置将使用默认配置，例如：如果要用6380作为端口启动Redis，那么可以执行:

./redis-server --port 6380

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/d1096d2f70a444b1a16693a68c9daf5b.png)

这种方式一般我们也用得比较少。

#### 配置文件启动

配置文件是我们启动的最多的模式，配置文件安装目录中有

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/cb5bcbe8f3144cfda3720664fd58b13f.png)

复制过来

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/93c4ba2542e84a028ee31cbc2367dd48.png)

改一下权限

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/95e8ca2a9c284b0aaa33d224cc027bad.png)

通过配置文件来启动

```
./redis-server ../conf/redis.conf
```

注意：这里对配置文件使用了相对路径，绝对路径也是可以的。

同时配置文件的方式可以方便我们改端口，改配置，增加密码等。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/da1a238ae1844d5dac7e386896577d48.png)

打开注释，设置为自己的密码，重启即可

### 操作

Redis服务启动完成后，就可以使用redis-cli连接和操作Redis服务。redis-cli可以使用两种方式连接Redis服务器。

1、单次操作

用redis-cli -hip {host} -p{port} {command}就可以直接得到命令的返回结果，例如:

那么下一次要操作redis，还需要再通过redis-cli。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/0648f2e321764c82a13bc4b1b29c94cb.png)

2、命令行操作

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/d38322174ebb48b89bbb8af2d80b8463.png)

通过redis-cli -h (host}-p {port}的方式连接到Redis服务，之后所有的操作都是通过控制台进行，例如:

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/816e9938ff2d49d9904ff79e413084bf.png)

我们没有写-h参数，那么默认连接127.0.0.1;如果不写-p，那么默认6379端口，也就是说如果-h和-p都没写就是连接127.0.0.1:6379这个 Redis实例。

### 停止

Redis提供了shutdown命令来停止Redis服务，例如我们目前已经启动的Redis服务，可以执行:

```
./redis-cli -p 6379 shutdown
```

redis服务端将会显示：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/a066e0616af6479cb29e109feae91482.png)

除了可以通过shutdown命令关闭Redis服务以外，还可以通过kill进程号的方式关闭掉Redis，但是强烈不建议使用kill -9强制杀死Redis服务，不但不会做持久化操作，还会造成缓冲区等资源不能被优雅关闭，极端情况会造成AOF和复制丢失数据的情况。如果是集群，还容易丢失数据。

同样还可以在命令行中执行shutdown指令

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/caea54b05bd7468b93e5615fc506dafe.png)

shutdown还有一个参数,代表是否在关闭Redis前，生成持久化文件，缺省是save，生成持久化文件，如果是nosave则不生成持久化文件

## Redis全局命令

对于键值数据库而言，基本的数据模型是 key-value 模型，Redis 支持的 value 类型包括了 String、哈希表、列表、集合等，而Memcached支持的 value 类型仅为 String 类型，所以Redis 能够在实际业务场景中得到广泛的应用，就是得益于支持多样化类型的 value。

Redis里面有16个库，但是Redis的分库功能没啥意义（默认就是0号库，尤其是集群操作的时候），我们一般都是默认使用0号库进行操作。

在了解Rediskey-value 模型之前，Redis的有一些全局命令，需要我们提前了解。

**keys命令**

```
keys *
keys L*
```

查看所有键(支持通配符)：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/a38baeb051bd46dab430d3037bedc48b.png)

但是这个命令请慎用，因为keys命令要把所有的key-value对全部拉出去，如果生产环境的键值对特别多的话，会对Redis的性能有很大的影响，推荐使用dbsize。

keys命令会遍历所有键，所以它的时间复杂度是o(n)，当Redis保存了大量键时线上环境禁止使用keys命令。

**dbsize命令**

dbsize命令会返回当前数据库中键的总数。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/21ed59cad4e84610869308640d57718c.png)

dbsize命令在计算键总数时不会遍历所有键,而是直接获取 Redis内置的键总数变量,所以dbsize命令的时间复杂度是O(1)。

**exists**

检查键是否存在，存在返回1，不存在返回0。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/706705c0c9d7418b8b72eb0be7228f85.png)

**del**

删除键，无论值是什么数据结构类型,del命令都可以将其删除。返回删除键个数，删除不存在键返回0。同时del命令可以支持删除多个键。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/2865421882494708bb0f78fd458a3a59.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/5d304a87b9d44633893581107f0d310a.png)

**键过期**

**expire**

Redis支持对键添加过期时间,当超过过期时间后,会自动删除键，时间单位秒。

ttl命令会返回键的剩余过期时间,它有3种返回值:

大于等于0的整数:键剩余的过期时间。

-1:键没设置过期时间。

-2:键不存在

除了expire、ttl命令以外，Redis还提供了expireat、pexpire,pexpireat、pttl、persist等一系列命令。

**expireat key**
timestamp: 键在秒级时间截timestamp后过期。

ttl命令和pttl都可以查询键的剩余过期时间，但是pttl精度更高可以达到毫秒级别，有3种返回值:

大于等于0的整数:键剩余的过期时间(ttl是秒，pttl是毫秒)。

-1:键没有设置过期时间。

-2:键不存在。

**pexpire key**
milliseconds:键在milliseconds毫秒后过期。

**pexpireat key**
milliseconds-timestamp键在毫秒级时间戳timestamp后过期。

**在使用Redis相关过期命令时,需要注意以下几点。**

1)如果expire key 的键不存在,返回结果为0:

2）如果过期时间为负值,键会立即被删除，犹如使用del命令一样:

3 ) persist命令可以将键的过期时间清除:

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/e288cca51dd446a19384a29adcc1faf7.png)

4）对于字符串类型键，执行set命令会去掉过期时间，这个问题很容易在开发中被忽视。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/5b749a4fd43248ce9af261080058ebaf.png)

5 ) Redis不支持二级数据结构(例如哈希、列表)内部元素的过期功能，不能对二级数据结构做过期时间设置。

**type**

返回键的数据结构类型，例如键tianming是字符串类型，返回结果为string。键mylist是列表类型，返回结果为list，键不存在返回none

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/a5cf8aefd7b2414b9e1fb6ad96f24b47.png)

**randomkey**

随机返回一个键，这个很简单，请自行实验。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/41c50a66594e42f98755f4ca70814050.png)

**rename**

键重命名

但是要注意，如果在rename之前,新键已经存在，那么它的值也将被覆盖。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/ec04e5381693497b9d85040317c25d74.png)

为了防止被强行rename，Redis提供了renamenx命令，确保只有newKey不存在时候才被覆盖。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/41644a5ba52048e9a4b2d8b130d211a0.png)

从上面我们可以看出，由于重命名键期间会执行del命令删除旧的键，如果键对应的值比较大，会存在阻塞Redis的可能性。

### 键名的生产实践

Redis没有命令空间，而且也没有对键名有强制要求。但设计合理的键名，有利于防止键冲突和项目的可维护性，比较推荐的方式是使用“业务名:对象名: id : [属性]”作为键名(也可以不是分号)。、

例如MySQL 的数据库名为mall，用户表名为order，那么对应的键可以用"mall:order:1",
"mall:order:1:name"来表示，如果当前Redis 只被一个业务使用，甚至可以去掉“order:”。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/46d96dd72a2943ceba09f109f563c243.png)

在能描述键含义的前提下适当减少键的长度，从而减少由于键过长的内存浪费。

## Redis常用数据结构

Redis提供了一些数据结构供我们往Redis中存取数据，最常用的的有5种，字符串（String）、哈希(Hash)、列表（list）、集合（set）、有序集合（ZSET）。

### 字符串（String）

字符串类型是Redis最基础的数据结构。首先键都是字符串类型，而且其他几种数据结构都是在字符串类型基础上构建的，所以字符串类型能为其他四种数据结构的学习奠定基础。字符串类型的值实际可以是字符串(简单的字符串、复杂的字符串(例如JSON、XML))、数字(整数、浮点数)，甚至是二进制(图片、音频、视频)，但是值最大不能超过512MB。

（虽然Redis是C写的，C里面有字符串&#x3c;本质使用char数组来实现>，但是处于种种考虑，Redis还是自己实现了字符串类型）

#### 操作命令

##### set 设置值

set key value![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/1dab86326fa249cd9d1ab118b48e9c6a.png)

set命令有几个选项:

ex seconds: 为键设置秒级过期时间。

px milliseconds: 为键设置毫秒级过期时间。

nx: 键必须不存在,才可以设置成功，用于添加（分布式锁常用）。

xx: 与nx相反,键必须存在，才可以设置成功,用于更新。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/a37e8717b892401c8aff82af2d280ac9.png)

从执行效果上看，ex参数和expire命令基本一样。还有一个需要特别注意的地方是如果一个字符串已经设置了过期时间，然后你调用了set 方法修改了它，它的过期时间会消失。

而nx和xx执行效果如下

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/03c37295005c4bd19c33f7f7509778bc.png)

除了set选项，Redis 还提供了setex和 setnx两个命令:

setex key
seconds value

setnx key value

setex和 setnx的作用和ex和nx选项是一样的。也就是，setex为键设置秒级过期时间，setnx设置时键必须不存在,才可以设置成功。

setex示例：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/af158144826048b1ac8d2b43551d39a9.png)

setnx示例：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/864f3f2d1a5a43b5a16d3e71cb808352.png)

因为键foo-ex已存在,所以setnx失败,返回结果为0，键foo-ex2不存在，所以setnx成功,返回结果为1。

有什么应用场景吗?以setnx命令为例子，由于Redis的单线程命令处理机制，如果有多个客户端同时执行setnx key value，根据setnx的特性只有一个客户端能设置成功，setnx可以作为分布式锁的一种实现方案。当然分布式锁没有不是只有一个命令就OK了，其中还有很多的东西要注意，我们后面会用单独的章节来讲述基于Redis的分布式锁。

##### get 获取值

如果要获取的键不存在,则返回nil(空):

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/87d74c2cb95349f2a5191929c8eb7735.png)

##### mset 批量设置值

通过mset命令一次性设置4个键值对

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/21b4a825ec3e43898363ea98c322a512.png)

##### mget 批量获取值

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/36e86de36de446f19392c2e0bf8882a7.png)

批量获取了键a、b、c、d的值:

如果有些键不存在,那么它的值为nil(空)，结果是按照传入键的顺序返回。

批量操作命令可以有效提高效率，假如没有mget这样的命令，要执行n次get命令具体耗时如下:

n次 get时间=n次网络时间+n次命令时间

使用mget命令后，要执行n次get命令操作具体耗时如下:

n次get时间=1次网络时间+n次命令时间

Redis可以支撑每秒数万的读写操作，但是这指的是Redis服务端的处理能力，对于客户端来说，一次命令除了命令时间还是有网络时间，假设网络时间为1毫秒，命令时间为0.1毫秒(按照每秒处理1万条命令算)，那么执行1000次 get命令需要1.1秒(1000*1+1000*0.1=1100ms)，1次mget命令的需要0.101秒(1*1+1000*0.1=101ms)。

##### Incr 数字运算

incr命令用于对值做自增操作,返回结果分为三种情况：

值不是整数,返回错误。

值是整数，返回自增后的结果。

键不存在，按照值为0自增,返回结果为1。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/1073c166f7364cf99ebeacd03ba3ca16.png)

除了incr命令，Redis提供了decr(自减)、 incrby(自增指定数字)、decrby(自减指定数字)、incrbyfloat（自增浮点数)，具体效果请同学们自行尝试。

##### append追加指令

append可以向字符串尾部追加值

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/1c0468546f1e499d9f850e3eabffc351.png)

##### strlen 字符串长度

返回字符串长度

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/7772d93625eb4680befbfb328129bbb4.png)

注意：每个中文占3个字节

##### getset 设置并返回原值

getset和set一样会设置值,但是不同的是，它同时会返回键原来的值

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/d568dc6e0a6c4adeae13d8f63a8fd0d8.png)

##### setrange 设置指定位置的字符

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/bdd7efebb340433e807d61f970649218.png)

下标从0开始计算。

##### getrange 截取字符串

getrange 截取字符串中的一部分，形成一个子串，需要指明开始和结束的偏移量，截取的范围是个闭区间。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/7802d85222fc450fbc4497d94acbd1fe.png)

#### 命令的时间复杂度

字符串这些命令中，除了del 、mset、 mget支持多个键的批量操作，时间复杂度和键的个数相关，为O(n)，getrange和字符串长度相关，也是O(n)，其余的命令基本上都是O(1)的时间复杂度，在速度上还是非常快的。

#### 存储结构：

![image-20231130172320386](E:\图灵课堂\Redis专题\笔记\1、Redis入门与应用.assets\image-20231130172320386.png)

#### 使用场景

字符串类型的使用场景很广泛：

**缓存功能**

Redis 作为缓存层，MySQL作为存储层，绝大部分请求的数据都是从Redis中获取。由于Redis具有支撑高并发的特性,所以缓存通常能起到加速读写和降低后端压力的作用。

**计数**

使用Redis 作为计数的基础工具，它可以实现快速计数、查询缓存的功能,同时数据可以异步落地到其他数据源。

**共享Session**

一个分布式Web服务将用户的Session信息（例如用户登录信息)保存在各自服务器中，这样会造成一个问题，出于负载均衡的考虑，分布式服务会将用户的访问均衡到不同服务器上，用户刷新一次访问可能会发现需要重新登录，这个问题是用户无法容忍的。

为了解决这个问题,可以使用Redis将用户的Session进行集中管理,在这种模式下只要保证Redis是高可用和扩展性的,每次用户更新或者查询登录信息都直接从Redis中集中获取。

**限速**

比如，很多应用出于安全的考虑,会在每次进行登录时,让用户输入手机验证码,从而确定是否是用户本人。但是为了短信接口不被频繁访问,会限制用户每分钟获取验证码的频率，例如一分钟不能超过5次。一些网站限制一个IP地址不能在一秒钟之内方问超过n次也可以采用类似的思路。

### 哈希(Hash)

Java里提供了HashMap，Redis中也有类似的数据结构，就是哈希类型。但是要注意，哈希类型中的映射关系叫作field-value，注意这里的value是指field对应的值，不是键对应的值。

#### 操作命令

基本上，哈希的操作命令和字符串的操作命令很类似，很多命令在字符串类型的命令前面加上了h字母，代表是操作哈希类型，同时还要指明要操作的field的值。

##### hset设值

hset user:1 name tianming

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/27d0a5b94e5e4a97a22b18f6ffbd370b.png)

如果设置成功会返回1，反之会返回0。此外Redis提供了hsetnx命令，它们的关系就像set和setnx命令一样,只不过作用域由键变为field。

##### hget取值

hget user:1 name

如果键或field不存在，会返回nil。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/fb8b44d85a8048e1a650c3e4c691036a.png)

##### hdel删除field

hdel会删除一个或多个field，返回结果为成功删除field的个数。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/45ed460d37c04b31bbd5122cd964a0e3.png)

##### hlen计算field个数

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/348060ead6c44a7a82817187e00c2f18.png)

##### hmset批量设值

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/dbad743d5eed4f7eb98c39e345ba2687.png)

##### hmget批量取值

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/04c34b6e61004b32835ff377a05a5586.png)

##### hexists判断field是否存在

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/7a182a0b69dc43c7a76f2119eca3910e.png)

若存在返回1，不存在返回0

##### hkeys获取所有field

它返回指定哈希键所有的field

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/7f84f9381fcf48a0ac43af16dbea6b6c.png)

##### hvals获取所有value

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/d4e3825c1bd446da9973b971834c37be.png)

##### hgetall获取所有field与value

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/338a05689b304021aa9b1cd6b601ffd9.png)

在使用hgetall时，如果哈希元素个数比较多，会存在阻塞Redis的可能。如果只需要获取部分field，可以使用hmget，如果一定要获取全部field-value，可以使用hscan命令，该命令会渐进式遍历哈希类型，hscan将在后面的章节介绍。

##### hincrby增加

hincrby和 hincrbyfloat，就像incrby和incrbyfloat命令一样，但是它们的作用域是filed。

##### hstrlen 计算value的字符串长度

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/4c52aa4d0f0c4841afa87011982c2151.png)

#### 命令的时间复杂度

哈希类型的操作命令中，hdel,hmget,hmset的时间复杂度和命令所带的field的个数相关O(k)，hkeys,hgetall,hvals和存储的field的总数相关，O(N)。其余的命令时间复杂度都是O(1)。

#### 存储结构

![image-20231130172145514](E:\图灵课堂\Redis专题\笔记\1、Redis入门与应用.assets\image-20231130172145514.png)

#### 使用场景

从前面的操作可以看出，String和Hash的操作非常类似，那为什么要弄一个hash出来存储。

哈希类型比较适宜存放**对象类型**的数据，我们可以比较下，如果数据库中表记录user为：

| id   | name     | age  |
| ---- | -------- | ---- |
| 1    | tianming | 18   |
| 2    | tuling   | 20   |

**1、使用String类型**

需要一条条去插入获取。

set user:1:name tianming;

set user:1:age  18;

set user:2:name tuling;

set user:2:age  20;

**优点：简单直观，每个键对应一个值**

**缺点：键数过多，占用内存多，用户信息过于分散，不用于生产环境**

**2、将对象序列化存入redis**

set user:1 serialize(userInfo);

**优点：编程简单，若使用序列化合理内存使用率高**

**缺点：序列化与反序列化有一定开销，更新属性时需要把userInfo全取出来进行反序列化，更新后再序列化到redis**

**3、使用hash类型**

hmset user:1 name tianming age 18

hmset user:2 name tuling age 20

**优点：简单直观，使用合理可减少内存空间消耗**

**缺点：要控制内部编码格式，不恰当的格式会消耗更多内存**

### 列表（list）

列表( list)类型是用来存储多个有序的字符串，a、b、c、c、b四个元素从左到右组成了一个有序的列表,列表中的每个字符串称为元素(element)，一个列表最多可以存储(2^32-1)个元素(*4294967295*)。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/b2c5e0d8bd6243e59b0c32aa5caa49f1.png)

在Redis 中，可以对列表两端插入( push)和弹出(pop)，还可以获取指定范围的元素列表、获取指定索引下标的元素等。列表是一种比较灵活的数据结构，它可以充当栈和队列的角色，在实际开发上有很多应用场景。

**列表类型有两个特点:**

第一、列表中的元素是有序的，这就意味着可以通过索引下标获取某个元素或者某个范围内的元素列表。

第二、列表中的元素可以是重复的。

#### 操作命令

##### lrange 获取指定范围内的元素列表（不会删除元素）

key start end

索引下标特点：从左到右为0到N-1

lrange 0 -1命令可以从左到右获取列表的所有元素

##### rpush 向右插入

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/f495b4575e994d1f8623beba74dd6fdf.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/33e03ce96b3248ceaac60ccb4fc0b1d8.png)

##### lpush 向左插入

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/302233f9f0284e46ae26930ac8be1cd2.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/abba1d7c8aea4e42a83ca17476c4b9cd.png)

##### linsert 在某个元素前或后插入新元素

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/08d4476bb8e449b7b9efb82f271d69a4.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/c77d0ff5efe44cb8b19dd1d8bac12848.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/ebc063246d934ef19f6611f855273f45.png)

这三个返回结果为命令完成后当前列表的长度，也就是列表中包含的元素个数，同时rpush和lpush都支持同时插入多个元素。

##### lpop 从列表左侧弹出（会删除元素）

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/cf3b7a2f1d85434689734dea9d467c0f.png)r

请注意，弹出来元素就没了。

##### rpop 从列表右侧弹出

rpop将会把列表最右侧的元素d弹出。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/6f55895f739b4a32a1e1c8fde0649031.png)

##### lrem 对指定元素进行删除

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/e533d9a779074c3286d81031ca408450.png)

lrem命令会从列表中找到等于value的元素进行删除，根据count的不同分为三种情况：

count>0，从左到右,删除最多count个元素。

count&#x3c;0，从右到左,删除最多count绝对值个元素。

count=0，删除所有。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/ad2833b4c011453f91928891e8a0e036.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/584b9986d80c46668f3fda91c466e340.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/ba19aa483a884537a687051f80967ebb.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/0480e571bc1e44f79b2e80b376ec288e.png)

返回值是实际删除元素的个数。

##### ltirm 按照索引范围修剪列表

例如想保留列表中第0个到第1个元素

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/8e52841e88f34a05888da5df0a8a4121.png)ls

##### lset修改指定索引下标的元素

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/663c682ce56444e382fc3f1b4da4253c.png)

##### lindex 获取列表指定索引下标的元素

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/f0b6ab8eca224305bcd97a4916c33947.png)l

##### llen 获取列表长度

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/31f34b4b3ec24f2e89d42b958dc3f343.png)

##### blpop和brpop阻塞式弹出元素

blpop和brpop是lpop和rpop的阻塞版本，除此之外还支持多个列表类型，也支持设定阻塞时间，单位秒，如果阻塞时间为0，表示一直阻塞下去。我们以brpop为例说明。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/a83e89040af7495da20b56ef826e6520.png)

A客户端阻塞了（因为没有元素就会阻塞）

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/c11fe0ec650c401cbd69500d99805f85.png)

A客户端一直处于阻塞状态。此时我们从另一个客户端B执行

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/c4c83e7fe2ac4f6385ac04b1ecbcf231.png)

A客户端则输出

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/4b2c7c7b95da4190ab6c772075faec7c.png)

注意：brpop后面如果是多个键，那么brpop会从左至右遍历键，一旦有一个键能弹出元素，客户端立即返回。

#### 存储结构

!![image-20231130172233987](E:\图灵课堂\Redis专题\笔记\1、Redis入门与应用.assets\image-20231130172233987.png)

#### 使用场景

列表类型可以用于比如：

消息队列，Redis 的 lpush+brpop命令组合即可实现阻塞队列，生产者客户端使用lrpush从列表左侧插入元素，多个消费者客户端使用brpop命令阻塞式的“抢”列表尾部的元素,多个客户端保证了消费的负载均衡和高可用性。

**文章列表**

每个用户有属于自己的文章列表，现需要分页展示文章列表。此时可以考虑使用列表,因为列表不但是有序的,同时支持按照索引范围获取元素。

实现其他数据结构

lpush+lpop =Stack（栈)

lpush +rpop =Queue(队列)

lpsh+ ltrim =Capped Collection（有限集合)

lpush+brpop=Message Queue(消息队列)

### 集合（set）

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/4b28f3a2aa6b4d4f9c5dd09317508102.png)

集合( set）类型也是用来保存多个的字符串元素,但和列表类型不一样的是，集合中不允许有重复元素,并且集合中的元素是无序的,不能通过索引下标获取元素。

一个集合最多可以存储2的32次方-1个元素。Redis除了支持集合内的增删改查，同时还支持多个集合取交集、并集、差集，合理地使用好集合类型,能在实际开发中解决很多实际问题。

#### 集合内操作命令

##### sadd 添加元素

允许添加多个，返回结果为添加成功的元素个数

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/d94f1e8871054f6dad9dad1d245d3dfb.png)

##### srem 删除元素

允许删除多个，返回结果为成功删除元素个数

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/fdcc39ccdf4c4ca1bb26003e65d40ec7.png)

##### scard 计算元素个数

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/292819f68b294180ad037dc590e8c562.png)

##### sismember 判断元素是否在集合中

如果给定元素element在集合内返回1，反之返回0

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/94148d18517a44d99cd6c7716d07f9b1.png)

##### srandmember 随机从集合返回指定个数元素

指定个数如果不写默认为1

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/2b386e0de6df44c685dc69e95bf4e7a2.png)

##### spop 从集合随机弹出元素

同样可以指定个数，如果不写默认为1，注意，既然是弹出，spop命令执行后,元素会从集合中删除,而srandmember不会。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/172580272c8b4413908328fad51e0537.png)

##### smembers 获取所有元素(不会弹出元素)

返回结果是无序的

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/f16f1615505b49a2aa5198a75168c4b2.png)

#### 集合间操作命令

现在有两个集合,它们分别是set1和set2

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/8546a5e8eff348bd8c4879fbc37db67b.png)

##### sinter 求多个集合的交集

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/80dd47f94b06433daa0b1f19be6a684f.png)

##### suinon 求多个集合的并集

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/614e67ffe97646389a781099b6e074fd.png)

##### sdiff 求多个集合的差集

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/4c0e783c37e64c15866bb08ccc0abbf8.png)

##### 将交集、并集、差集的结果保存

```
sinterstore destination key [key ...]
suionstore destination key [key ...]
sdiffstore destination key [key ...]

```

集合间的运算在元素较多的情况下会比较耗时，所以 Redis提供了上面三个命令(原命令+store)将集合间交集、并集、差集的结果保存在destination key中，例如：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/450fcb6c2e2342b39925c1779a4ba369.png)



#### 存储结构

![image-20231130172104956](E:\图灵课堂\Redis专题\笔记\1、Redis入门与应用.assets\image-20231130172104956.png)

#### 使用场景

集合类型比较典型的使用场景是标签(tag)。例如一个用户可能对娱乐、体育比较感兴趣，另一个用户可能对历史、新闻比较感兴趣，这些兴趣点就是标签。有了这些数据就可以得到喜欢同一个标签的人，以及用户的共同喜好的标签，这些数据对于用户体验以及增强用户黏度比较重要。

例如一个电子商务的网站会对不同标签的用户做不同类型的推荐，比如对数码产品比较感兴趣的人，在各个页面或者通过邮件的形式给他们推荐最新的数码产品，通常会为网站带来更多的利益。

除此之外，集合还可以通过生成随机数进行比如抽奖活动，以及社交图谱等等。

### 有序集合（ZSET）

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/fe24d4258f4b4dd99ad52a6752a840ae.png)

有序集合相对于哈希、列表、集合来说会有一点点陌生,但既然叫有序集合,那么它和集合必然有着联系,它保留了集合不能有重复成员的特性,但不同的是,有序集合中的元素可以排序。但是它和列表使用索引下标作为排序依据不同的是,它给每个元素设置一个分数( score)作为排序的依据。

有序集合中的元素不能重复，但是score可以重复，就和一个班里的同学学号不能重复,但是考试成绩可以相同。

有序集合提供了获取指定分数和元素范围查询、计算成员排名等功能，合理的利用有序集合，能帮助我们在实际开发中解决很多问题。

#### 集合内操作命令

##### zadd添加成员

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/353cb7b71ca04daab3cf7fb99ceb36f6.png)

返回结果代表成功添加成员的个数

要注意:

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/a8a349c237304f6c94f0a16f5ae3ed24.png)

zadd命令还有四个选项nx、xx、ch、incr 四个选项

nx: member必须不存在，才可以设置成功，用于添加。

xx: member必须存在，才可以设置成功,用于更新。

ch: 返回此次操作后,有序集合元素和分数发生变化的个数

incr: 对score做增加，相当于后面介绍的zincrby

##### zcard 计算成员个数

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/9eb0d86663fd45e6a3032c05c1870339.png)

##### zscore 计算某个成员的分数

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/7da7b741030b4b2b9863d569198f001d.png)

如果成员不存在则返回nil

##### zrank计算成员的排名

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/7395eb62a4fa4e669cf201799aca069e.png)

zrank是从分数从低到高返回排名

zrevrank反之

很明显，排名从0开始计算。

##### zrem 删除成员

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/d4e23637dda8461abba76c303cfbfbad.png)

允许一次删除多个成员。

返回结果为成功删除的个数。

##### zincrby 增加成员的分数

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/b1a0193040334a0f90ddb66bce53fe0c.png)

##### zrange和zrevrange返回指定排名范围的成员

有序集合是按照分值排名的，zrange是从低到高返回,zrevrange反之。如果加上
withscores选项，同时会返回成员的分数

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/d2a8d64fce484b64bf95f0a886fa1f45.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/a17fc90c48dd498f9539f5b834ada3bb.png)

##### zrangebyscore返回指定分数范围的成员

```
zrangebyscore key min max [withscores] [limit offset count]
zrevrangebyscore key max min [withscores][limit offset count]

```

其中zrangebyscore按照分数从低到高返回，zrevrangebyscore反之。例如下面操作从低到高返回200到221分的成员，withscores选项会同时返回每个成员的分数。

同时min和max还支持开区间(小括号）和闭区间(中括号)，-inf和+inf分别代表无限小和无限大:

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/de92b2a82134468ab69bbb8718ccfd1f.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/92edbaef3b2c4c4c9a7ee0daac41ad0a.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/e5dc53cdbff941c9814590973ac96499.png)

##### zcount 返回指定分数范围成员个数

zcount key min max

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/82a1a39c69bf4c8395411312fd276515.png)

##### zremrangebyrank 按升序删除指定排名内的元素

zremrangebyrank key start end

##### zremrangebyscore 删除指定分数范围的成员

zremrangebyscore key min max

#### 集合间操作命令

##### zinterstore 交集

zinterstore![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/0e0688f60b404126895ffbbf4c6ae290.png)

这个命令参数较多，下面分别进行说明

destination:交集计算结果保存到这个键。

numkeys:需要做交集计算键的个数。

key [key ...]:需要做交集计算的键。

weights weight
[weight ...]:每个键的权重，在做交集计算时，每个键中的每个member 会将自己分数乘以这个权重,每个键的权重默认是1。

aggregate sum/
min |max:计算成员交集后，分值可以按照sum(和)、min(最小值)、max(最大值)做汇总,默认值是sum。

不太好理解，我们用一个例子来说明。（算平均分）

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/cfc3967b70cb4dcea41057c3708d8616.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/65f2bee807e1446989277c0fadff86cf.png)

##### zunionstore 并集

该命令的所有参数和zinterstore是一致的，只不过是做并集计算，大家可以自行实验。

#### 存储结构

![image-20231130171955319](E:\图灵课堂\Redis专题\笔记\1、Redis入门与应用.assets\image-20231130171955319.png)



#### 使用场景

有序集合比较典型的使用场景就是排行榜系统。例如视频网站需要对用户上传的视频做排行榜，榜单的维度可能是多个方面的:按照时间、按照播放数量、按照获得的赞数。

## Redis高级数据结构

### Bitmaps

现代计算机用二进制(位)作为信息的基础单位，1个字节等于8位，例如“big”字符串是由3个字节组成，但实际在计算机存储时将其用二进制表示,“big”分别对应的ASCII码分别是98、105、103，对应的二进制分别是01100010、01101001和 01100111。

许多开发语言都提供了操作位的功能，合理地使用位能够有效地提高内存使用率和开发效率。Redis提供了Bitmaps这个“数据结构”可以实现对位的操作。把数据结构加上引号主要因为:

Bitmaps本身不是一种数据结构，实际上它就是字符串，但是它可以对字符串的位进行操作。

Bitmaps单独提供了一套命令，所以在Redis中使用Bitmaps和使用字符串的方法不太相同。可以把 Bitmaps想象成一个以位为单位的数组，数组的每个单元只能存储0和1，数组的下标在 Bitmaps 中叫做偏移量。

#### 操作命令

##### setbit 设置值

setbit key offset value

设置键的第 offset 个位的值(从0算起)。

假设现在有20个用户，userid=0,2,4,6,8的用户对网站进行了访问，存储键名为日期。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/faf895f88aa940ea854dba440b8030e7.png)

##### getbit 获取值

getbit key offset

获取键的第 offset位的值(从0开始算)，比如获取userid=8的用户是否在2022（年/这天）访问过,返回0说明没有访问过:

当然offset是不存在的，也会返回0。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/a32b45af412f4ab6a0345447cd7c4ea7.png)

##### bitcount 获取Bitmaps指定范围值为1的个数

bitcount [start] [end]

下面操作计算26号和27号这天的独立访问用户数量

[start]和[end]代表起始和结束字节数

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/6f10ab793dfa4e4bb9cc297344544f17.png)

##### bitop Bitmaps 间的运算

bitop op destkey key [key . ...]

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/085f985a7e5a4e19b4d97cf247e8385b.png)

bitop是一个复合操作，它可以做多个Bitmaps 的 and(交集)or(并集)not(非)xor(异或）操作并将结果保存在destkey中。

##### bitpos 计算Bitmaps中第一个值为targetBit 的偏移量

bitpos key targetBit [start] [end]

计算0815当前访问网站的最小用户id

除此之外，bitops有两个选项[start]和[end]，分别代表起始字节和结束字节。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/6cc67c513009442a9ecf29beed3188af.png)

#### Bitmaps优势

假设网站有1亿用户，每天独立访问的用户有5千万，如果每天用集合类型和 Bitmaps分别存储活跃用户，很明显，假如用户id是Long型，64位，则集合类型占据的空间为64位x50 000 000= 400MB，而Bitmaps则需要1位×100 000 000=12.5MB，可见Bitmaps能节省很多的内存空间。

##### 面试题和场景

1、目前有10亿数量的自然数，乱序排列，需要对其排序。限制条件-在32位机器上面完成，内存限制为 2G。如何完成？

2、如何快速在亿级黑名单中快速定位URL地址是否在黑名单中？(每条URL平均64字节)

3、需要进行用户登陆行为分析，来确定用户的活跃情况？

4、网络爬虫-如何判断URL是否被爬过？

5、快速定位用户属性（黑名单、白名单等）

6、数据存储在磁盘中，如何避免大量的无效IO？

##### 传统数据结构的不足

当然有人会想，我直接将网页URL存入数据库进行查找不就好了，或者建立一个哈希表进行查找不就OK了。

当数据量小的时候，这么思考是对的，

确实可以将值映射到 HashMap 的 Key，然后可以在 O(1) 的时间复杂度内返回结果，效率奇高。但是 HashMap 的实现也有缺点，例如存储容量占比高，考虑到负载因子的存在，通常空间是不能被用满的，举个例子如果一个1000万HashMap，Key=String（长度不超过16字符，且重复性极小），Value=Integer，会占据多少空间呢？1.2个G。实际上，1000万个int型，只需要40M左右空间，占比3%，1000万个Integer，需要161M左右空间，占比13.3%。可见一旦你的值很多例如上亿的时候，那HashMap 占据的内存大小就变得很可观了。

但如果整个网页黑名单系统包含100亿个网页URL，在数据库查找是很费时的，并且如果每个URL空间为64B，那么需要内存为640GB，一般的服务器很难达到这个需求。

#### 布隆过滤器

##### 布隆过滤器简介

**1970 年布隆提出了一种布隆过滤器的算法，用来判断一个元素是否在一个集合中。
这种算法由一个二进制数组和一个 Hash 算法组成。**

本质上布隆过滤器是一种数据结构，比较巧妙的概率型数据结构（probabilistic data structure），特点是高效地插入和查询，可以用来告诉你 “某样东西一定不存在或者可能存在”。

相比于传统的 List、Set、Map 等数据结构，它更高效、占用空间更少，但是缺点是其返回的结果是概率性的，而不是确切的。

实际上，布隆过滤器广泛应用于网页黑名单系统、垃圾邮件过滤系统、爬虫网址判重系统等，Google 著名的分布式数据库 Bigtable 使用了布隆过滤器来查找不存在的行或列，以减少磁盘查找的IO次数，Google Chrome浏览器使用了布隆过滤器加速安全浏览服务。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/801f60ff2d28436faaaab3007dd7d893.png)

##### 布隆过滤器的误判问题

Ø通过hash计算在数组上不一定在集合

Ø本质是hash冲突

Ø通过hash计算不在数组的一定不在集合（误判）

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/d31bbeaf5a8742d2b15ce65bbc5a4537.png)

**优化方案**

增大数组(预估适合值)

增加hash函数

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/84b1457186f44856b6ad56561ba64229.png)

#### [Redis]()中的布隆过滤器

##### Redisson

Maven引入Redisson

```
   <dependency>
            <groupId>org.redisson</groupId>
            <artifactId>redisson</artifactId>
            <version>3.12.3</version>
        </dependency>
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/419fbea26266439f9f1d7b0d333920f3.png)

##### 自行实现

就是利用Redis的bitmaps来实现。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/261c8c065fdc4fdebbd7f343d83d3c6e.png)

##### 单机下无Redis的布隆过滤器

使用Google的Guava的BloomFilter。

Maven引入Guava

```
   <dependency>
            <groupId>com.google.guava</groupId>
            <artifactId>guava</artifactId>
            <version>30.1.1-jre</version>
        </dependency>
```

### HyperLogLog

#### 介绍

HyperLogLog(Hyper[ˈhaɪpə(r)])并不是一种新的数据结构(实际类型为字符串类型)，而是一种基数算法,通过HyperLogLog可以利用极小的内存空间完成独立总数的统计，数据集可以是IP、Email、ID等。

如果你负责开发维护一个大型的网站，有一天产品经理要网站每个网页每天的 UV 数据，然后让你来开发这个统计模块，你会如何实现？

如果统计 PV 那非常好办，给每个网页一个独立的 Redis 计数器就可以了，这个计数器的 key 后缀加上当天的日期。这样来一个请求，incrby 一次，最终就可以统计出所有的 PV 数据。

但是 UV 不一样，它要去重，同一个用户一天之内的多次访问请求只能计数一次。这就要求每一个网页请求都需要带上用户的 ID，无论是登陆用户还是未登陆用户都需要一个唯一 ID 来标识。

一个简单的方案，那就是为每一个页面一个独立的 set 集合来存储所有当天访问过此页面的用户 ID。当一个请求过来时，我们使用 sadd 将用户 ID 塞进去就可以了。通过 scard 可以取出这个集合的大小，这个数字就是这个页面的 UV 数据。

但是，如果你的页面访问量非常大，比如一个爆款页面几千万的 UV，你需要一个很大的 set集合来统计，这就非常浪费空间。如果这样的页面很多，那所需要的存储空间是惊人的。为这样一个去重功能就耗费这样多的存储空间，值得么？其实需要的数据又不需要太精确，105w 和 106w 这两个数字对于老板们来说并没有多大区别，So，有没有更好的解决方案呢？

这就是HyperLogLog的用武之地，Redis 提供了 HyperLogLog 数据结构就是用来解决这种统计问题的。HyperLogLog 提供不精确的去重计数方案，虽然不精确但是也不是非常不精确，Redis官方给出标准误差是0.81%，这样的精确度已经可以满足上面的UV 统计需求了。

百万级用户访问网站

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/494d8e7c3cbc464db90935208fa20d44.png)

#### 操作命令

HyperLogLog提供了3个命令: pfadd、pfcount、pfmerge。

##### pfadd

pfadd key element [element …]

pfadd用于向HyperLogLog 添加元素,如果添加成功返回1:

pfadd u-9-30 u1 u2 u3 u4 u5 u6 u7 u8

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/c6de0b04ae7b43e69528a24301d7920b.png)

##### pfcount

pfcount key [key …]

pfcount用于计算一个或多个HyperLogLog的独立总数，例如u-9-30 的独立总数为8:

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/a86c5c198cef4717864fd2794eaf3624.png)

如果此时向插入一些用户，用户并且有重复

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/689ddb0df4164a98bb7d078e45af9fda.png)

如果我们继续往里面插入数据，比如插入100万条用户记录。内存增加非常少，但是pfcount 的统计结果会出现误差。

##### pfmerge

pfmerge destkey sourcekey [sourcekey ... ]

pfmerge可以求出多个HyperLogLog的并集并赋值给destkey，请自行测试。

可以看到，HyperLogLog内存占用量小得惊人，但是用如此小空间来估算如此巨大的数据，必然不是100%的正确，其中一定存在误差率。前面说过，Redis官方给出的数字是0.81%的失误率。

#### 原理概述

##### 基本原理

HyperLogLog基于概率论中伯努利试验并结合了极大似然估算方法，并做了分桶优化。

实际上目前还没有发现更好的在大数据场景中准确计算基数的高效算法，因此在不追求绝对准确的情况下，使用概率算法算是一个不错的解决方案。概率算法不直接存储数据集合本身，通过一定的概率统计方法预估值，这种方法可以大大节省内存，同时保证误差控制在一定范围内。目前用于基数计数的概率算法包括:

举个例子来理解HyperLogLog
算法，有一天李瑾老师和马老师玩打赌的游戏。

规则如下: 抛硬币的游戏，每次抛的硬币可能正面，可能反面，没回合一直抛，直到每当抛到正面回合结束。

然后我跟马老师说，抛到正面最长的回合用到了7次，你来猜一猜，我用到了多少个回合做到的？

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/b908ff160cdd4eed858e76b248fd880e.png)

进行了n次实验，比如上图：

第一次试验: 抛了3次才出现正面，此时 k=3，n=1

第二次试验: 抛了2次才出现正面，此时 k=2，n=2

第三次试验: 抛了4次才出现正面，此时 k=4，n=3

…………

第n 次试验：抛了7次才出现正面，此时我们估算，k=7

马老师说大概你抛了128个回合。这个是怎么算的。

k是每回合抛到1所用的次数，我们已知的是最大的k值，可以用kmax表示。由于每次抛硬币的结果只有0和1两种情况，因此，能够推测出kmax在任意回合出现的概率 ，并由kmax结合极大似然估算的方法推测出n的次数n =
2^(k_max) 。概率学把这种问题叫做伯努利实验。

但是问题是，这种本身就是概率的问题，我跟马老师说，我只用到12次，并且有视频为证。

所以这种预估方法存在较大误差，为了改善误差情况，HLL中引入分桶平均的概念。

同样举抛硬币的例子，如果只有一组抛硬币实验，显然根据公式推导得到的实验次数的估计误差较大；如果100个组同时进行抛硬币实验，受运气影响的概率就很低了，每组分别进行多次抛硬币实验，并上报各自实验过程中抛到正面的抛掷次数的最大值，就能根据100组的平均值预估整体的实验次数了。

分桶平均的基本原理是将统计数据划分为m个桶，每个桶分别统计各自的kmax,并能得到各自的基数预估值，最终对这些基数预估值求平均得到整体的基数估计值。LLC中使用几何平均数预估整体的基数值，但是当统计数据量较小时误差较大；HLL在LLC基础上做了改进，**采用调和平均数过滤掉不健康的统计值**。

什么叫调和平均数呢？举个例子

求平均工资：A的是1000/月，B的30000/月。采用平均数的方式就是：
(1000 + 30000) / 2 = 15500

采用调和平均数的方式就是：
2/(1/1000 + 1/30000) ≈ 1935.484

可见调和平均数比平均数的好处就是不容易受到大的数值的影响，比平均数的效果是要更好的。

##### 结合Redis的实现理解原理

现在我们和前面的业务场景进行挂钩：统计网页每天的 UV 数据。

**1.转为比特串**

通过hash函数，将数据转为二进制的比特串，例如输入5，便转为：101。为什么要这样转化呢？

是因为要和抛硬币对应上，比特串中，0 代表了反面，1 代表了正面，如果一个数据最终被转化了 10010000，那么从右往左，从低位往高位看，我们可以认为，首次出现 1 的时候，就是正面。

那么基于上面的估算结论，我们可以通过多次抛硬币实验的最大抛到正面的次数来预估总共进行了多少次实验，同样也就可以根据存入数据中，转化后的出现了 1 的最大的位置 k_max 来估算存入了多少数据。

**2.分桶**

分桶就是分多少轮。抽象到计算机存储中去，就是存储的是一个以单位是比特(bit)，长度为 L 的大数组 S ，将 S 平均分为 m 组，注意这个 m 组，就是对应多少轮，然后每组所占有的比特个数是平均的，设为 P。容易得出下面的关系：

比如有4个桶的话，那么可以截取低2位作为分桶的依据。

比如

10010000   进入0号桶

10010001   进入1号桶

10010010   进入2号桶

10010011   进入3号桶

##### Redis 中的 HyperLogLog 实现

**pfadd**

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/116ef8fb48584cc5910184aaf50092a1.png)

当我们执行这个操作时，tianming这个字符串就会被转化成64个bit的二进制比特串。

0010....0001  64位

然后在Redis中要分到16384个桶中（为什么是这么多桶：第一降低误判，第二，用到了14位二进制：2的14次方=16384）

怎么分？根据得到的比特串的后14位来做判断即可。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/bd71287e85294b14b95e3fcb82243fab.png)

根据上述的规则，我们知道这个数据要分到 1号桶，同时从左往右（低位到高位）计算第1个出现的1的位置，这里是第4位，那么就往这个1号桶插入4的数据（转成二进制）

如果有第二个数据来了，按照上述的规则进行计算。

那么问题来了，如果分到桶的数据有重复了（这里比大小，大的替换小的）：

规则如下，比大小（比出现位置的大小），比如有个数据是最高位才出现1，那么这个位置算出来就是50，50比4大，则进行替换。1号桶的数据就变成了50（二进制是110010）

所以这里可以看到，每个桶的数据一般情况下6位存储即可。

所以我们这里可以推算一下一个key的HyperLogLog只占据多少的存储。

16384*6 /8/1024=12k。并且这里最多可以存储多少数据，因为是64位吗，所以就是2的64次方的数据，这个存储的数据非常非常大的，一般用户用long来定义，最大值也只有这么多。

**pfcount**

进行统计的时候，就是把16384桶，把每个桶的值拿出来，比如取出是 n,那么访问次数就是2的n次方。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/37ea459470614e8fad06c5eab8a009ca.png)

然后把每个桶的值做调和平均数，就可以算出一个算法值。

同时，在具体的算法实现上，HLL还有一个分阶段偏差修正算法。我们就不做更深入的了解了。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/50bed8f5a0394a93aa8033ee9f847672.png)

const和m都是Redis里面根据数据做的调和平均数。

### GEO

Redis 3.2版本提供了GEO(地理信息定位)功能，支持存储地理位置信息用来实现诸如附近位置、摇一摇这类依赖于地理位置信息的功能。

地图元素的位置数据使用二维的经纬度表示，经度范围(-180, 180]，纬度范围(-90,
90]，纬度正负以赤道为界，北正南负，经度正负以本初子午线(英国格林尼治天文台) 为界，东正西负。

业界比较通用的地理位置距离排序算法是GeoHash 算法，Redis 也使用GeoHash
算法。GeoHash
算法将二维的经纬度数据映射到一维的整数，这样所有的元素都将在挂载到一条线上，距离靠近的二维坐标映射到一维后的点之间距离也会很接近。当我们想要计算「附近的人时」，首先将目标位置映射到这条线上，然后在这个一维的线上获取附近的点就行了。

在 Redis 里面，经纬度使用 52 位的整数进行编码，放进了 zset 里面，zset 的 value 是元素的 key，score 是 GeoHash 的 52 位整数值。

#### 操作命令

##### 增加地理位置信息

geoadd key longitude latitude member [longitude latitude member ...J

longitude、latitude、member分别是该地理位置的经度、纬度、成员，例如下面有5个城市的经纬度。

城市            经度             纬度             成员

北京            116.28          39.55            beijing

天津            117.12          39.08            tianjin

石家庄        114.29          38.02            shijiazhuang

唐山            118.01          39.38            tangshan

保定            115.29          38.51            baoding

cities:locations是上面5个城市地理位置信息的集合，现向其添加北京的地理位置信息:

geoadd cities :locations 116.28 39.55 beijing

返回结果代表添加成功的个数，如果cities:locations没有包含beijing,那么返回结果为1，如果已经存在则返回0。

如果需要更新地理位置信息，仍然可以使用geoadd命令，虽然返回结果为0。geoadd命令可以同时添加多个地理位置信息:

geoadd cities:locations 117.12 39.08 tianjin 114.29 38.02
shijiazhuang 118.01 39.38 tangshan 115.29 38.51 baoding

##### 获取地理位置信息

geopos key member [member ...]下面操作会获取天津的经维度:

geopos cities:locations tianjin1)1)"117.12000042200088501"

##### 获取两个地理位置的距离。

geodist key member1 member2 [unit]

**其中unit代表返回结果的单位，包含以下四种:**

m (meters)代表米。

km (kilometers)代表公里。

mi (miles)代表英里。

ft(feet)代表尺。

下面操作用于计算天津到北京的距离，并以公里为单位:

geodist cities : locations tianjin beijing km

##### 获取指定位置范围内的地理信息位置集合

```
georadius key longitude latitude radius m|km|ft|mi [withcoord][withdist]
[withhash][COUNT count] [ascldesc] [store key] [storedist key]
georadiusbymember key member radius m|km|ft|mi  [withcoord][withdist]
[withhash] [COUNT count][ascldesc] [store key] [storedist key]

```

georadius和georadiusbymember两个命令的作用是一样的，都是以一个地理位置为中心算出指定半径内的其他地理信息位置，不同的是georadius命令的中心位置给出了具体的经纬度，georadiusbymember只需给出成员即可。其中radius  m | km |ft |mi是必需参数，指定了半径(带单位)。

这两个命令有很多可选参数，如下所示:

withcoord:返回结果中包含经纬度。

withdist:返回结果中包含离中心节点位置的距离。

withhash:返回结果中包含geohash，有关geohash后面介绍。

COUNT count:指定返回结果的数量。

asc l desc:返回结果按照离中心节点的距离做升序或者降序。

store key:将返回结果的地理位置信息保存到指定键。

storedist key:将返回结果离中心节点的距离保存到指定键。

下面操作计算五座城市中,距离北京150公里以内的城市:

georadiusbymember cities:locations beijing 150 km

##### 获取geohash

```
geohash key member [member ...]
```

Redis使用geohash将二维经纬度转换为一维字符串，下面操作会返回beijing的geohash值。

geohash cities: locations beijing

字符串越长,表示的位置更精确，geohash长度为9时,精度在2米左右，geohash长度为8时,精度在20米左右。

两个字符串越相似,它们之间的距离越近,Redis 利用字符串前缀匹配算法实现相关的命令。

geohash编码和经纬度是可以相互转换的。

##### 删除地理位置信息

zrem key member

GEO没有提供删除成员的命令，但是因为GEO的底层实现是zset，所以可以借用zrem命令实现对地理位置信息的删除。

# Redis高级特性和应用(发布 订阅、Stream)

## 发布和订阅

Redis提供了基于“发布/订阅”模式的消息机制，此种模式下，消息发布者和订阅者不进行直接通信,发布者客户端向指定的频道( channel)发布消息，订阅该频道的每个客户端都可以收到该消息。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/9f6b447fb8024a3595352326d792ba95.png)

### 操作命令

Redis主要提供了发布消息、订阅频道、取消订阅以及按照模式订阅和取消订阅等命令。

#### 发布消息

```
publish channel message
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/a04198c93ac0416eb677631af2adf868.png)

返回值是接收到信息的订阅者数量，如果是0说明没有订阅者，这条消息就丢了（再启动订阅者也不会收到）。

#### 订阅消息

```
subscribe channel [channel ...]
```

订阅者可以订阅一个或多个频道，如果此时另一个客户端发布一条消息，当前订阅者客户端会收到消息。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/1177c507c9d1491bb9a654a600604fab.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/ede17eb77f294947b942524de78c651d.png)

如果有多个客户端同时订阅了同一个频道，都会收到消息。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/2f812f0fb14b45008410122fb7000a28.png)

客户端在执行订阅命令之后进入了订阅状态（类似于监听），只能接收subscribe、psubscribe,unsubscribe、 punsubscribe的四个命令。

#### 查询订阅情况

##### 查看活跃的频道

```
pubsub channels [pattern]
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/6a08c97b389e4807a69152504a2f5c80.png)

Pubsub 命令用于查看订阅与发布系统状态，包括活跃的频道（是指当前频道至少有一个订阅者），其中[pattern]是可以指定具体的模式，类似于通配符。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/87a515a74bb444e3b2a7a71d23caa0dc.png)

##### 查看频道订阅数

```
pubsub numsub channel
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/c42cb62a659e456991363af93a8dac0a.png)

最后也可以通过 help看具体的参数运用

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/1a4ab497c7f0489b93bd26fff1b3420d.png)

#### 使用场景和缺点

需要消息解耦又并不关注消息可靠性的地方都可以使用发布订阅模式。

PubSub 的生产者传递过来一个消息，Redis会直接找到相应的消费者传递过去。如果一个消费者都没有，那么消息直接丢弃。如果开始有三个消费者，一个消费者突然挂掉了，生产者会继续发送消息，另外两个消费者可以持续收到消息。但是挂掉的消费者重新连上的时候，这断连期间生产者发送的消息，对于这个消费者来说就是彻底丢失了。

所以和很多专业的消息队列系统（例如Kafka、RocketMQ)相比，Redis 的发布订阅很粗糙，例如无法实现消息堆积和回溯。但胜在足够简单，如果当前场景可以容忍的这些缺点,也不失为一个不错的选择。

正是因为 PubSub 有这些缺点，它的应用场景其实是非常狭窄的。从Redis5.0 新增了 Stream 数据结构，这个功能给 Redis 带来了持久化消息队列，我们马上将要学习到。

## Redis Stream

Redis5.0 最大的新特性就是多出了一个数据结构 Stream，它是一个新的强大的支持多播的可持久化的消息队列，Redis的作者声明Redis Stream地借鉴了 Kafka 的设计。

### Stream总述

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/78e6d284fc6c4f8ab774287aad02f501.png)

Redis Stream 的结构如上图所示,每一个Stream都有一个消息链表，将所有加入的消息都串起来，每个消息都有一个唯一的 ID 和对应的内容。消息是持久化的，Redis 重启后，内容还在。

**具体的玩法如下：**

1、每个 Stream 都有唯一的名称，它就是 Redis 的 key，在我们首次使用xadd指令追加消息时自动创建。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/f6287b28a1604bf29df9eb2b41388cfc.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/baa99ab58cd4432bbf4848788a6dee74.png)

消息 ID 的形式是timestampInMillis-sequence，例如1527846880572-5，它表示当前的消息在毫米时间戳1527846880572时产生，并且是该毫秒内产生的第 5 条消息。消息 ID 可以由服务器自动生成（*代表默认自动），也可以由客户端自己指定，但是形式必须是整数-整数，而且必须是后面加入的消息的 ID 要大于前面的消息 ID。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/d1b29c3c392c4c329226699acb889c37.png)

消息内容就是键值对，形如 hash 结构的键值对，这没什么特别之处。

2、每个 Stream 都可以挂多个消费组，每个消费组会有个游标last_delivered_id在 Stream 数组之上往前移动，表示当前消费组已经消费到哪条消息了。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/48efbf4656844581bad5dd7470eb4955.png)

每个消费组都有一个Stream 内唯一的名称，消费组不会自动创建，它需要单独的指令xgroup create进行创建，需要指定从 Stream 的某个消息 ID 开始消费，这个 ID 用来初始化last_delivered_id变量。

3、每个消费组 (Consumer Group) 的状态都是独立的，相互不受影响。也就是说同一份 Stream 内部的消息会被每个消费组都消费到。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/cb27e53f2451443f96925f58689f7cbb.png)

4、同一个消费组 (Consumer Group) 可以挂接多个消费者 (Consumer)，这些消费者之间是竞争关系，任意一个消费者读取了消息都会使游标last_delivered_id往前移动。每个消费者有一个组内唯一名称。

5、消费者 (Consumer) 内部会有个状态变量pending_ids，它记录了当前已经被客户端读取,但是还没有 ack的消息。如果客户端没有 ack，这个变量里面的消息 ID 会越来越多，一旦某个消息被 ack，它就开始减少。这个 pending_ids 变量在 Redis 官方被称之为PEL，也就是Pending Entries List，这是一个很核心的数据结构，它用来确保客户端至少消费了消息一次，而不会在网络传输的中途丢失了没处理。

### 常用操作命令

#### 生产端

**xadd 追加消息**

xadd第一次对于一个stream使用可以生成一个stream的结构

```
xadd streamtest * name lijin age 18
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/9ed2e1ca3e144bb784eae61ca4b53552.png)

*号表示服务器自动生成 ID，后面顺序跟着一堆 key/value

1626705954593-0 则是生成的消息 ID，由两部分组成：时间戳-序号。时间戳时毫秒级单位，是生成消息的Redis服务器时间，它是个64位整型。序号是在这个毫秒时间点内的消息序号。它也是个64位整型。

为了保证消息是有序的，因此Redis生成的ID是单调递增有序的。由于ID中包含时间戳部分，为了避免服务器时间错误而带来的问题（例如服务器时间延后了），Redis的每个Stream类型数据都维护一个latest_generated_id属性，用于记录最后一个消息的ID。若发现当前时间戳退后（小于latest_generated_id所记录的），则采用时间戳不变而序号递增的方案来作为新消息ID（这也是序号为什么使用int64的原因，保证有足够多的的序号），从而保证ID的单调递增性质。

强烈建议使用Redis的方案生成消息ID，因为这种时间戳+序号的单调递增的ID方案，几乎可以满足你全部的需求。但ID是支持自定义的。

**xrange 获取消息列表，会自动过滤已经删除的消息**

```
xrange streamtest - +
```

其中-表示最小值 , + 表示最大值

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/af90e1953f8b45f6bb2a41bf5776916c.png)

或者我们可以指定消息 ID 的列表：

```
xrange streamtest - 1665646270814-0
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/867166efa9ef440ea7e4dd8c8dd79960.png)

**xlen 消息长度**

```
xlen streamtest
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/9f115fc376dd4fd18ecf98d177e22d70.png)

**del 删除 Stream**

del streamtest  删除整个 Stream

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/20af39a3b407484e8c7d3608453f56f9.png)

xdel可以删除指定的消息(指定ID)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/30f40f46b0b047568f9b1c8d8098f592.png)

#### 消费端

##### 单消费者

虽然Stream中有消费者组的概念，但是可以在不定义消费组的情况下进行 Stream 消息的独立消费，当 Stream 没有新消息时，甚至可以阻塞等待。Redis 设计了一个单独的消费指令xread，可以将 Stream 当成普通的消息队列 (list) 来使用。使用 xread 时，我们可以完全忽略消费组 (Consumer Group) 的存在，就好比 Stream 就是一个普通的列表 (list)。

```
xread count 1 streams stream2 0-0
```

表示从 Stream 头部读取1条消息，0-0指从头开始

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/aff55f71cbbb490aad07a1ab9c631f3c.png)

```
xread count 2 streams stream1 1665644057564-0
```

也可以指定从streams的消息Id开始(不包括命令中的消息id)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/428bdd838ad44d279756f4946adf9596.png)

```
xread count 1 streams stream1 $
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/04dd6f0da5a647c6b5b4f85767e975ff.png)

$代表从尾部读取，上面的意思就是从尾部读取最新的一条消息,此时默认不返回任何消息

应该以阻塞的方式读取尾部最新的一条消息，直到新的消息的到来

```
xread block 0 count 1 streams stream1 $
```

block后面的数字代表阻塞时间，单位毫秒，0代表一直阻塞

此时我们新开一个客户端，往stream1中写入一条消息

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/c59a0638fe51419f84bfd4a4f8f2a8d0.png)

可以看到看到阻塞解除了，返回了新的消息内容，而且还显示了一个等待时间，这里我们等待了10.82s

一般来说客户端如果想要使用 xread 进行顺序消费，一定要记住当前消费到哪里了，也就是返回的消息 ID。下次继续调用 xread 时，将上次返回的最后一个消息 ID 作为参数传递进去，就可以继续消费后续的消息。不然很容易重复消息，基于这点单消费者基本上没啥运用场景，本课也不深入去讲。

##### 消费组

###### 创建消费组

Stream 通过xgroup create指令创建消费组 (Consumer Group)，需要传递起始消息 ID 参数用来初始化last_delivered_id变量。

0-表示从头开始消费

```
xgroup create stream1 c1 0-0
```

$ 表示从尾部开始消费，只接受新消息，当前 Stream 消息会全部忽略

```
xgroup create stream1 c2 $
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/ccd3dd2307a043c1a127621f2d923848.png)

现在我们可以用xinfo命令来看看stream1的情况：

```
xinfo stream stream1
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/2a4ff34210e84e86ba857a91eb2ce7e9.png)

查看stream1的消费组的情况：

```
xinfo groups stream1
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/78bda19da7044cfba93a37ce65292146.png)

###### 消息消费

有了消费组，自然还需要消费者，Stream提供了 xreadgroup 指令可以进行消费组的组内消费，需要提供消费组名称、消费者名称和起始消息 ID。

它同 xread 一样，也可以阻塞等待新消息。读到新消息后，对应的消息 ID 就会进入消费者的PEL(正在处理的消息) 结构里，客户端处理完毕后使用 xack 指令通知服务器，本条消息已经处理完毕，该消息 ID 就会从 PEL 中移除。

```
xreadgroup GROUP c1 consumer1 count 1 streams stream1 >
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/75a5be4cc4c84ccc959407fe6744e0ad.png)

consumer1代表消费者的名字。

">"表示从当前消费组的 last_delivered_id 后面开始读，每当消费者读取一条消息，last_delivered_id 变量就会前进。前面我们定义cg1的时候是从头开始消费的，自然就获得stream1中第一条消息再执行一次上面的命令

自然就读取到了下条消息。我们将Stream1中的消息读取完，很自然就没有消息可读了。

然后设置阻塞等待![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/80a52389edda4d25acf8d099756515b6.png)

我们新开一个客户端，发送消息到stream1回到原来的客户端，发现阻塞解除，收到新消息

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/5899b8f323634b7482c0ce9a2975ed7f.png)

我们来观察一下观察消费组状态

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/3cf7520617144b309fa02c6ef6b70ca1.png)

如果同一个消费组有多个消费者，我们还可以通过 xinfo consumers 指令观察每个消费者的状态

```
xinfo consumers stream2 c1
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/75b6f8e647764a8598370fdbef4bf631.png)

可以看到目前c1这个消费者有 7 条待ACK的消息，空闲了2086176ms 没有读取消息。

如果我们确认一条消息

```
xack stream1 c1 1665647371850-0
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/21bcc8499e4a487c848afde464afc79a.png)

就可以看到待确认消息变成了6条

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/16985be5de824675aa72f21af08baa9d.png)

xack允许带多个消息id，比如 同时Stream还提供了命令XPENDING 用来获消费组或消费内消费者的未处理完毕的消息。

```
xpending stream1 c1
```

具体操作细节可以参考：[xpending 命令 -- Redis中国用户组（CRUG）](http://www.redis.cn/commands/xpending.html)

命令XCLAIM[kleɪm]用以进行消息转移的操作，将某个消息转移到自己的Pending[ˈpendɪŋ]列表中。需要设置组、转移的目标消费者和消息ID，同时需要提供IDLE（已被读取时长），只有超过这个时长，才能被转移。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/a584a8bb1f274d61a6dc1b3976316fc0.png)[]

具体操作细节可参考：[xclaim 命令 -- Redis中国用户组（CRUG）](http://www.redis.cn/commands/xclaim.html)

### 在Redis中实现消息队列

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/03c4e0d96c40423ca834515d73c9dabb.png)

#### 基于pub/sub

注意必须继承JedisPubSub这个抽象类

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/81da1646a67b4486b26021ff672a53eb.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/b3b97617ed984b3dae34b83b582e00bb.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/d5f7698eed88443682061d7b137451b7.png)

#### 基于Stream

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/77b699b89e7246449fd2e86e2a8d5acb.png)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/98c6d17fd9e74c8c92190126da1b1fda.png)

java封装了两个类用于处理消息及消息的元数据。

StreamEntry和StreamEntryID

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/523f8ac3bb1f4c198f40c1496aa22fe5.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/1d017710e9724326867d898a8935d5a0.png)

#### Redis中几种消息队列实现的总结

##### 基于List的 LPUSH+BRPOP 的实现

足够简单，消费消息延迟几乎为零，但是需要处理空闲连接的问题。

如果线程一直阻塞在那里，Redis客户端的连接就成了闲置连接，闲置过久，服务器一般会主动断开连接，减少闲置资源占用，这个时候blpop和brpop或抛出异常，所以在编写客户端消费者的时候要小心，如果捕获到异常，还有重试。

其他缺点包括：

做消费者确认ACK麻烦，不能保证消费者消费消息后是否成功处理的问题（宕机或处理异常等），通常需要维护一个Pending列表，保证消息处理确认；不能做广播模式，如pub/sub，消息发布/订阅模型；不能重复消费，一旦消费就会被删除；不支持分组消费。

##### 基于Sorted-Set的实现

多用来实现延迟队列，当然也可以实现有序的普通的消息队列，但是消费者无法阻塞的获取消息，只能轮询，不允许重复消息。

##### PUB/SUB，订阅/发布模式

优点：

典型的广播模式，一个消息可以发布到多个消费者；多信道订阅，消费者可以同时订阅多个信道，从而接收多类消息；消息即时发送，消息不用等待消费者读取，消费者会自动接收到信道发布的消息。

缺点：

消息一旦发布，不能接收。换句话就是发布时若客户端不在线，则消息丢失，不能寻回；不能保证每个消费者接收的时间是一致的；若消费者客户端出现消息积压，到一定程度，会被强制断开，导致消息意外丢失。通常发生在消息的生产远大于消费速度时；可见，Pub/Sub 模式不适合做消息存储，消息积压类的业务，而是擅长处理广播，即时通讯，即时反馈的业务。

##### 基于Stream类型的实现

基本上已经有了一个消息中间件的雏形，可以考虑在生产过程中使用。

Streams是Redis5.0新增的数据结构，它提供了消息的持久化以及主备复制功能，可以让任何客户端访问任何时刻的数据，它有一个消息链表可以将所有加入的消息串起来，每个消息都存在一个唯一ID，并且这个ID是递增的，是一种redis专门为消息队列定义的一种数据结构，

添加数据的命令是XADD，语法格式是XADD key ID field value [field value ...]，参数说明如下：

key:redis的key
ID：消息的唯一标识，可以指定，也可以设置为*，设置为*时id会自动生成，id是递增
field value：消息的字段和值

如下生产（创建）若干条消息：

redis> XADD mystream * name Sara surname OConnor
"1601372323627-0"
redis> XADD mystream * field1 value1 field2 value2 field3 value3
"1601372323627-1"
redis> XLEN mystream
(integer) 2
redis> XRANGE mystream - +
1) 1) "1601372323627-0"
   2) 1) "name"
      2) "Sara"
      3) "surname"
      4) "OConnor"
2) 1) "1601372323627-1"
   2) 1) "field1"
      2) "value1"
      3) "field2"
      4) "value2"
      5) "field3"
      6) "value3"
      redis> 

其中XLEN用来查看消息的个数，XRANGE用来通过范围查询基于递增ID获取消息，-相当于是负无穷，+相当于是正无穷，即获取所有消息。我们接着再来看下其它一些命令

XDEL :根据ID删除消息

> XADD mystream * a 1
> 1538561698944-0
> XADD mystream * b 2
> 1538561700640-0
> XADD mystream * c 3
> 1538561701744-0
> XDEL mystream 1538561700640-0
> (integer) 1
> 127.0.0.1:6379> XRANGE mystream - +
> 1) 1) 1538561698944-0
> 2) 1) "a"
> 2) "1"
> 2) 1) 1538561701744-0
> 2) 1) "c"
> 2) "3"

XLEN:获取消息的数量 
redis> XADD mystream * item 1
"1601372563177-0"
redis> XADD mystream * item 2
"1601372563178-0"
redis> XADD mystream * item 3
"1601372563178-1"
redis> XLEN mystream
(integer) 3
redis>

XRANGE:查询指定范围的消息
key ：队列名
start ：开始值， - 表示最小值
end ：结束值， + 表示最大值
count ：数量

redis> XADD writers * name Virginia surname Woolf
"1601372577811-0"
redis> XADD writers * name Jane surname Austen
"1601372577811-1"
redis> XADD writers * name Toni surname Morrison
"1601372577811-2"
redis> XADD writers * name Agatha surname Christie
"1601372577812-0"
redis> XADD writers * name Ngozi surname Adichie
"1601372577812-1"
redis> XLEN writers
(integer) 5
redis> XRANGE writers - + COUNT 2
1) 1) "1601372577811-0"
   2) 1) "name"
      2) "Virginia"
      3) "surname"
      4) "Woolf"
2) 1) "1601372577811-1"
   2) 1) "name"
      2) "Jane"
      3) "surname"
      4) "Austen"
      redis>

 XREVRANGE:从后往前获取消息
语法格式XREVRANGE key end start [COUNT count]

key ：队列名
end ：结束值， + 表示最大值
start ：开始值， - 表示最小值
count ：数量

 redis> XADD writers * name Virginia surname Woolf
"1601372731458-0"
redis> XADD writers * name Jane surname Austen
"1601372731459-0"
redis> XADD writers * name Toni surname Morrison
"1601372731459-1"
redis> XADD writers * name Agatha surname Christie
"1601372731459-2"
redis> XADD writers * name Ngozi surname Adichie
"1601372731459-3"
redis> XLEN writers
(integer) 5
redis> XREVRANGE writers + - COUNT 1
1) 1) "1601372731459-3"
   2) 1) "name"
      2) "Ngozi"
      3) "surname"
      4) "Adichie"

XREAD 
以阻塞或者是非阻塞的方式获取消息，即消费消息的命令，语法格式XREAD [COUNT count] [BLOCK milliseconds] STREAMS key [key ...] id [id ...]，解释如下：

count ：数量
milliseconds ：可选，阻塞毫秒数，没有设置就是非阻塞模式
key ：队列名
id ：消息 ID

# 从 Stream 头部读取两条消息

> XREAD COUNT 2 STREAMS mystream writers 0-0 0-0
> 1) 1) "mystream"
> 2) 1) 1) 1526984818136-0
>  2) 1) "duration"
>     2) "1532"
>     3) "event-id"
>     4) "5"
>     5) "user-id"
>     6) "7782813"
> 2) 1) 1526999352406-0
>  2) 1) "duration"
>     2) "812"
>     3) "event-id"
>     4) "9"
>     5) "user-id"
>     6) "388234"
> 2) 1) "writers"
> 2) 1) 1) 1526985676425-0
>  2) 1) "name"
>     2) "Virginia"
>     3) "surname"
>     4) "Woolf"
> 2) 1) 1526985685298-0
>  2) 1) "name"
>     2) "Jane"
>     3) "surname"
>     4) "Austen" 

XGROUP CREATE 
创建消费者组，使用消费者可以对消息进行并发的消费，解决消费者消费能力不足的问题，语法格式为XGROUP [CREATE key groupname id-or-$] [SETID key groupname id-or-$] [DESTROY key groupname] [DELCONSUMER key groupname consumername]，解释如下：


key ：队列名称，如果不存在就创建
groupname ：组名。
$ ： 表示从尾部开始消费，只接受新消息，当前 Stream 消息会全部忽略。

如下从头开始消费： 

XGROUP CREATE mystream consumer-group-name 0-0  

如下从尾部开始消费： 

XGROUP CREATE mystream consumer-group-name $ 

在实际的场景中我们可以通过设置多个消费者组的不同开始消费的位置来实现并发消费的效果 

#### 总结 

Redis消息队列的实现真正能运用到生产环境的是List和Streams，两者区别如下所示



### 消息队列问题

从我们上面对Stream的使用表明，Stream已经具备了一个消息队列的基本要素，生产者API、消费者API，消息Broker，消息的确认机制等等，所以在使用消息中间件中产生的问题，这里一样也会遇到。

#### Stream 消息太多怎么办?

要是消息积累太多，Stream 的链表岂不是很长，内容会不会爆掉?xdel 指令又不会删除消息，它只是给消息做了个标志位。

Redis 自然考虑到了这一点，所以它提供了一个定长 Stream 功能。在 xadd 的指令提供一个定长长度 maxlen，就可以将老的消息干掉，确保最多不超过指定长度。

#### 消息如果忘记 ACK 会怎样?

Stream 在每个消费者结构中保存了正在处理中的消息 ID 列表 PEL，如果消费者收到了消息处理完了但是没有回复 ack，就会导致 PEL 列表不断增长，如果有很多消费组的话，那么这个 PEL 占用的内存就会放大。所以消息要尽可能的快速消费并确认。

#### PEL 如何避免消息丢失?

在客户端消费者读取 Stream 消息时，Redis 服务器将消息回复给客户端的过程中，客户端突然断开了连接，消息就丢失了。但是 PEL 里已经保存了发出去的消息 ID。待客户端重新连上之后，可以再次收到 PEL 中的消息 ID 列表。不过此时 xreadgroup 的起始消息 ID 不能为参数>，而必须是任意有效的消息 ID，一般将参数设为 0-0，表示读取所有的 PEL 消息以及自last_delivered_id之后的新消息。

#### 死信问题

如果某个消息，不能被消费者处理，也就是不能被XACK，这是要长时间处于Pending列表中，即使被反复的转移给各个消费者也是如此。此时该消息的delivery counter（通过XPENDING可以查询到）就会累加，当累加到某个我们预设的临界值时，我们就认为是坏消息（也叫死信，DeadLetter，无法投递的消息），由于有了判定条件，我们将坏消息处理掉即可，删除即可。删除一个消息，使用XDEL语法，注意，这个命令并没有删除Pending中的消息，因此查看Pending，消息还会在，可以在执行执行XDEL之后，XACK这个消息标识其处理完毕。

#### Stream 的高可用

Stream 的高可用是建立主从复制基础上的，它和其它数据结构的复制机制没有区别，也就是说在 Sentinel 和 Cluster 集群环境下 Stream 是可以支持高可用的。不过鉴于 Redis 的指令复制是异步的，在 failover 发生时，Redis 可能会丢失极小部分数据，这点 Redis 的其它数据结构也是一样的。

#### 分区 Partition

Redis 的服务器没有原生支持分区能力，如果想要使用分区，那就需要分配多个 Stream，然后在客户端使用一定的策略来生产消息到不同的 Stream。

### Stream小结

Stream 的消费模型借鉴了Kafka 的消费分组的概念，它弥补了 Redis Pub/Sub 不能持久化消息的缺陷。但是它又不同于 kafka，Kafka 的消息可以分 partition，而 Stream 不行。如果非要分 parition 的话，得在客户端做，提供不同的 Stream 名称，对消息进行 hash 取模来选择往哪个 Stream 里塞。

关于 Redis 是否适合做消息队列，业界一直是有争论的。很多人认为，要使用消息队列，就应该采用 Kafka、RabbitMQ 这些专门面向消息队列场景的软件，而 Redis 更加适合做缓存。
根据这些年做 Redis 研发工作的经验，我的看法是：Redis 是一个非常轻量级的键值数据库，部署一个 Redis 实例就是启动一个进程，部署 Redis 集群，也就是部署多个 Redis 实例。而 Kafka、RabbitMQ 部署时，涉及额外的组件，例如 Kafka 的运行就需要再部署ZooKeeper。相比 Redis 来说，Kafka 和 RabbitMQ 一般被认为是重量级的消息队列。
所以，关于是否用 Redis 做消息队列的问题，不能一概而论，我们需要考虑业务层面的数据体量，以及对性能、可靠性、可扩展性的需求。如果分布式系统中的组件消息通信量不大，那么，Redis 只需要使用有限的内存空间就能满足消息存储的需求，而且，Redis 的高性能特性能支持快速的消息读写，不失为消息队列的一个好的解决方案。

## Redis的Key和Value的数据结构组织

### 全局哈希表

为了实现从键到值的快速访问，Redis 使用了一个哈希表来保存所有键值对。一个哈希表，其实就是一个数组，数组的每个元素称为一个哈希桶。所以，我们常说，一个哈希表是由多个哈希桶组成的，每个哈希桶中保存了键值对数据。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/1eb5ca015a3b4389ad4ede842d98df1f.png)

哈希桶中的 entry 元素中保存了*key和*value指针，分别指向了实际的键和值，这样一来，即使值是一个集合，也可以通过*value指针被查找到。因为这个哈希表保存了所有的键值对，所以，我也把它称为全局哈希表。

哈希表的最大好处很明显，就是让我们可以用 O(1) 的时间复杂度来快速查找到键值对：我们只需要计算键的哈希值，就可以知道它所对应的哈希桶位置，然后就可以访问相应的 entry 元素。

但当你往 Redis 中写入大量数据后，就可能发现操作有时候会突然变慢了。这其实是因为你忽略了一个潜在
的风险点，那就是哈希表的冲突问题和 rehash 可能带来的操作阻塞。

当你往哈希表中写入更多数据时，哈希冲突是不可避免的问题。这里的哈希冲突，两个 key 的哈希值和哈希桶计算对应关系时，正好落在了同一个哈希桶中。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/f2a1f73d63f1428cad1324a7b67283b4.png)

Redis 解决哈希冲突的方式，就是链式哈希。链式哈希也很容易理解，就是指同一个哈希桶中的多个元素用一个链表来保存，它们之间依次用指针连接。

当然如果这个数组一直不变，那么hash冲突会变很多，这个时候检索效率会大打折扣，所以Redis就需要把数组进行扩容（一般是扩大到原来的两倍），但是问题来了，扩容后每个hash桶的数据会分散到不同的位置，这里设计到元素的移动，必定会阻塞IO，所以这个ReHash过程会导致很多请求阻塞。

### 渐进式rehash

为了避免这个问题，Redis 采用了渐进式 rehash。

首先、Redis 默认使用了两个全局哈希表：哈希表 1 和哈希表 2。一开始，当你刚插入数据时，默认使用哈希表 1，此时的哈希表 2 并没有被分配空间。随着数据逐步增多，Redis 开始执行 rehash。

1、给哈希表 2 分配更大的空间，例如是当前哈希表 1 大小的两倍

2、把哈希表 1 中的数据重新映射并拷贝到哈希表 2 中

3、释放哈希表 1 的空间

在上面的第二步涉及大量的数据拷贝，如果一次性把哈希表 1 中的数据都迁移完，会造成 Redis 线程阻塞，无法服务其他请求。此时，Redis 就无法快速访问数据了。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665642245080/2406a8206e944d449b03f1f390bedf0d.png)

在Redis 开始执行 rehash，Redis仍然正常处理客户端请求，但是要加入一个额外的处理：

处理第1个请求时，把哈希表 1中的第1个索引位置上的所有 entries 拷贝到哈希表 2 中

处理第2个请求时，把哈希表 1中的第2个索引位置上的所有 entries 拷贝到哈希表 2 中

如此循环，直到把所有的索引位置的数据都拷贝到哈希表 2 中。

这样就巧妙地把一次性大量拷贝的开销，分摊到了多次处理请求的过程中，避免了耗时操作，保证了数据的快速访问。

所以这里基本上也可以确保根据key找value的操作在O（1）左右。

不过这里要注意，如果Redis中有海量的key值的话，这个Rehash过程会很长很长，虽然采用渐进式Rehash，但在Rehash的过程中还是会导致请求有不小的卡顿。并且像一些统计命令也会非常卡顿：比如keys

按照Redis的配置每个实例能存储的最大*的key的数量*为2的32次方,即2.5亿，但是尽量把key的数量控制在千万以下，这样就可以避免Rehash导致的卡顿问题，如果数量确实比较多，建议采用分区hash存储。

# Redis高级特性和应用(慢查询、Pipeline、事务、Lua)

## Redis的慢查询

许多存储系统（例如 MySQL)提供慢查询日志帮助开发和运维人员定位系统存在的慢操作。所谓慢查询日志就是系统在命令执行前后计算每条命令的执行时间，当超过预设阀值,就将这条命令的相关信息（例如:发生时间，耗时，命令的详细信息）记录下来，Redis也提供了类似的功能。

Redis客户端执行一条命令分为如下4个部分:

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/06f5790387cc4fd792bcc06469c1c88e.png)

1、发送命令

2、命令排队

3、命令执行

4、返回结果

需要注意，慢查询只统计步骤3的时间，所以没有慢查询并不代表客户端没有超时问题。因为有可能是命令的网络问题或者是命令在Redis在排队，所以不是说命令执行很慢就说是慢查询，而有可能是网络的问题或者是Redis服务非常繁忙（队列等待长）。

### 慢查询配置

对于任何慢查询功能,需要明确两件事：多慢算慢，也就是预设阀值怎么设置？慢查询记录存放在哪？

Redis提供了两种方式进行慢查询的配置

**1、动态设置**

慢查询的阈值默认值是10毫秒

参数：slowlog-log-slower-than就是时间预设阀值，它的单位是微秒(1秒=1000毫秒=1 000 000微秒)，默认值是10 000，假如执行了一条“很慢”的命令（例如keys *)，如果它的执行时间超过了10 000微秒，也就是10毫秒，那么它将被记录在慢查询日志中。

我们通过动态命令修改

```
config set slowlog-log-slower-than 20000  
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/f39ed7fee0d0445395ccad845c49f9ec.png)

使用config set完后,若想将配置持久化保存到Redis.conf，要执行config rewrite

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/5920646bf3644603b6b39634abc85599.png)

```
config rewrite
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/265eaf4d898445ac954eeb773a34741f.png)

**注意：**

如果配置slowlog-log-slower-than=0表示会记录所有的命令，slowlog-log-slower-than&#x3c;0对于任何命令都不会进行记录。

**2、配置文件设置（修改后需重启服务才生效）**

打开Redis的配置文件redis.conf，就可以看到以下配置：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/313284bef6564695ab3471fb60e52bd6.png)

slowlog-max-len用来设置慢查询日志最多存储多少条

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/64425e34a56d4c908787fa4d7c7fbdb5.png)

另外Redis还提供了slowlog-max-len配置来解决存储空间的问题。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/7f1d45305a3740cbae2f1a13a1b2170c.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/04697937eb7645db9c61c5c97f1d1c91.png)

实际上Redis服务器将所有的慢查询日志保存在服务器状态的slowlog链表中（内存列表），slowlog-max-len就是列表的最大长度（默认128条）。当慢查询日志列表被填满后，新的慢查询命令则会继续入队，队列中的第一条数据机会出列。

虽然慢查询日志是存放在Redis内存列表中的，但是Redis并没有告诉我们这里列表是什么,而是通过一组命令来实现对慢查询日志的访问和管理。并没有说明存放在哪。这个怎么办呢？Redis提供了一些列的慢查询操作命令让我们可以方便的操作。

### 慢查询操作命令

**获取慢查询日志**

```
slowlog get [n] 
```

可以看到每个慢查询日志有属性组成，分别是慢查询日志的标识id、发生时间戳、命令耗时（单位微秒）、执行命令和参数，客户端IP+端口和客户端名称。

获取慢查询日志列表当前的长度

```
slowlog len
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/0047c133cccf46c88f30a17190956050.png)

慢查询日志重置

```
slowlog reset
```

实际是对列表做清理操作

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/71b4851637d64eac9e0954f5df3192f9.png)

#### 慢查询建议

慢查询功能可以有效地帮助我们找到Redis可能存在的瓶颈,但在实际使用过程中要注意以下几点:

**slowlog-max-len配置建议:**

建议调大慢查询列表，记录慢查询时Redis会对长命令做截断操作，并不会占用大量内存。增大慢查询列表可以减缓慢查询被剔除的可能，线上生产建议设置为1000以上。

**slowlog-log-slower-than配置建议:**
配置建议：默认值超过10毫秒判定为慢查询，需要根据Redis并发量调整该值。

由于Redis采用单线程响应命令，对于高流量的场景,如果命令执行时间在1毫秒以上，那么Redis最多可支撑OPS不到1000。因此对于高OPS场景的Redis建议设置为1毫秒或者更低比如100微秒。

慢查询只记录命令执行时间，并不包括命令排队和网络传输时间。因此客户端执行命令的时间会大于命令实际执行时间。因为命令执行排队机制,慢查询会导致其他命令级联阻塞，因此当客户端出现请求超时,需要检查该时间点是否有对应的慢查询，从而分析出是否为慢查询导致的命令级联阻塞。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/ac54a157c47649d7a775098157bf6090.png)

由于慢查询日志是一个先进先出的队列，也就是说如果慢查询比较多的情况下，可能会丢失部分慢查询命令，为了防止这种情况发生，可以定期执行slow get命令将慢查询日志持久化到其他存储中。

## Pipeline

前面我们已经说过，Redis客户端执行一条命令分为如下4个部分:1）发送命令2）命令排队3）命令执行4）返回结果。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/ed845d8ca34447bda4e007515cb023f9.png)

其中1和4花费的时间称为Round Trip Time (RTT,往返时间)，也就是数据在网络上传输的时间。

Redis提供了批量操作命令(例如mget、mset等)，有效地节约RTT。

但大部分命令是不支持批量操作的，例如要执行n次 hgetall命令，并没有mhgetall命令存在，需要消耗n次RTT。

举例：Redis的客户端和服务端可能部署在不同的机器上。例如客户端在本地，Redis服务器在阿里云的广州，两地直线距离约为800公里，那么1次RTT时间=800 x2/ ( 300000×2/3 ) =8毫秒，(光在真空中传输速度为每秒30万公里,这里假设光纤为光速的2/3 )。而Redis命令真正执行的时间通常在微秒(1000微妙=1毫秒)级别，所以才会有Redis 性能瓶颈是网络这样的说法。

Pipeline（流水线)机制能改善上面这类问题,它能将一组 Redis命令进行组装,通过一次RTT传输给Redis,再将这组Redis命令的执行结果按顺序返回给客户端,没有使用Pipeline执行了n条命令,整个过程需要n次RTT。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/bad98ee5e9824f669c21b5fc8f4bc67b.png)

使用Pipeline 执行了n次命令，整个过程需要1次RTT。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/87e1613317f64af9a6cfd4cbbd42e0a5.png)

Pipeline并不是什么新的技术或机制，很多技术上都使用过。而且RTT在不同网络环境下会有不同，例如同机房和同机器会比较快，跨机房跨地区会比较慢。

redis-cli的--pipe选项实际上就是使用Pipeline机制，但绝对部分情况下，我们使用Java语言的Redis客户端中的Pipeline会更多一点。

代码参见：

```
com.msb.redis.adv.RedisPipeline
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/ff23cbb3f5b946f196ce7fd03feb1f5b.png)

总的来说，在不同网络环境下非Pipeline和Pipeline执行10000次set操作的效果，在执行时间上的比对如下：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/7537b379814845ad80484b1eaf4346e2.png)

差距有100多倍，可以得到如下两个结论:

1、Pipeline执行速度一般比逐条执行要快。

2、客户端和服务端的网络延时越大，Pipeline的效果越明显。

Pipeline虽然好用,但是每次Pipeline组装的命令个数不能没有节制，否则一次组装Pipeline数据量过大，一方面会增加客户端的等待时间，另一方面会造成一定的网络阻塞,可以将一次包含大量命令的Pipeline拆分成多次较小的Pipeline来完成，比如可以将Pipeline的总发送大小控制在内核输入输出缓冲区大小之内或者控制在单个TCP 报文最大值1460字节之内。

内核的输入输出缓冲区大小一般是4K-8K，不同操作系统会不同（当然也可以配置修改）

最大传输单元（Maximum Transmission Unit，MTU）,这个在以太网中最大值是1500字节。那为什么单个TCP 报文最大值是1460，因为因为还要扣减20个字节的IP头和20个字节的TCP头，所以是1460。

同时Pipeline只能操作一个Redis实例，但是即使在分布式Redis场景中，也可以作为批量操作的重要优化手段。

## 事务

大家应该对事务比较了解，简单地说，事务表示一组动作，要么全部执行，要么全部不执行。

例如在社交网站上用户A关注了用户B，那么需要在用户A的关注表中加入用户B，并且在用户B的粉丝表中添加用户A，这两个行为要么全部执行，要么全部不执行,否则会出现数据不一致的情况。

Redis提供了简单的事务功能，将一组需要一起执行的命令放到multi和exec两个命令之间。multi 命令代表事务开始，exec命令代表事务结束。另外discard命令是回滚。

一个客户端

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/97fd31a4f6174421a5d84029172adeb0.png)

另外一个客户端

在事务没有提交的时查询（查不到数据）

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/c10554292d0f484f9844542243507dda.png)

在事务提交后查询（可以查到数据）

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/96d61976166148d5a82501d41ab4003e.png)

可以看到sadd命令此时的返回结果是QUEUED，代表命令并没有真正执行，而是暂时保存在Redis中的一个缓存队列（所以discard也只是丢弃这个缓存队列中的未执行命令，并不会回滚已经操作过的数据，这一点要和关系型数据库的Rollback操作区分开）。

只有当exec执行后，用户A关注用户B的行为才算完成，如下所示exec返回的两个结果对应sadd命令。

**但是要注意Redis的事务功能很弱。在事务回滚机制上，Redis只能对基本的语法错误进行判断。**

如果事务中的命令出现错误,Redis 的处理机制也不尽相同。

1、语法命令错误

例如操作错将set写成了sett，属于语法错误，会造成整个事务无法执行，事务内的操作都没有执行:

2、运行时错误

例如：事务内第一个命令简单的设置一个string类型，第二个对这个key进行sadd命令，这种就是运行时命令错误，因为语法是正确的:

可以看到Redis并不支持回滚功能，第一个set命令已经执行成功,开发人员需要自己修复这类问题。

### **Redis的事务原理**

事务是Redis实现在服务器端的行为，用户执行MULTI命令时，服务器会将对应这个用户的客户端对象设置为一个特殊的状态，在这个状态下后续用户执行的查询命令不会被真的执行，而是被服务器缓存起来，直到用户执行EXEC命令为止，服务器会将这个用户对应的客户端对象中缓存的命令按照提交的顺序依次执行。

###### Redis事务 - 基本使用

每个事务的操作都有 begin、commit 和 rollback：

begin 指示事务的开始
commit 指示事务的提交
rollback 指示事务的回滚
它大致的形式如下：

begin();
try {
	// 执行业务相关代码
	command1();
	command2();
	....
	commit();
} catch(Exception e) {
	rollback();
}

Redis 在形式上看起来也差不多，MULTI、EXEC、DISCARD这三个指令构成了 redis 事务处理的基础：

MULTI：用来组装一个事务，从输入Multi命令开始，输入的命令都会依次进入命令队列中，但不会执行，直到输入Exec后，redis会将之前的命令依次执行。
EXEC：用来执行一个事务
DISCARD：用来取消一个事务

###### redis事务分2个阶段：组队阶段、执行阶段

组队阶段：只是将所有命令加入命令队列
执行阶段：依次执行队列中的命令，在执行这些命令的过程中，不会被其他客户端发送的请求命令插队或者打断。

![image-20231203161411282](E:\图灵课堂\Redis专题\笔记\1、Redis专题.assets\image-20231203161411282.png)



### Redis的watch命令

有些应用场景需要在事务之前，确保事务中的key没有被其他客户端修改过，才执行事务，否则不执行(类似乐观锁)。Redis 提供了watch命令来解决这类问题。

客户端1：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/347a34b954f04f0296cd36dd6deb138b.png)

客户端2：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/5faebac992e6406999bb0ad61a242bfd.png)

客户端1继续：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/bc340c7fd6a74973a6f8d78e71d2e0db.png)

可以看到“客户端-1”在执行multi之前执行了watch命令，“客户端-2”在“客户端-1”执行exec之前修改了key值，造成客户端-1事务没有执行(exec结果为nil)。

##### Redis 事务特性

**单独的隔离操作**： 事务中的所有命令都会序列化、按顺序地执行，事务在执行过程中，不会被其他客户端发送来的命令请求所打断。
**没有隔离级别的概念**： 队列中的命令没有提交（exec）之前，都不会实际被执行，因为事务提交前任何指令都不会被实际执行。
**不能保证原子性**： 事务中如果有一条命令执行失败，后续的命令仍然会被执行，没有回滚。如果在组队阶段，有1个失败了，后面都不会成功；如果在组队阶段成功了，在执行阶段有那个命令失败就这条失败，其他的命令则正常执行，不保证都成功或都失败。




### Pipeline和事务的区别

**1.1 为什么会出现Pipeline**
　　Redis本身是基于Request/Response协议的，正常情况下，客户端发送一个命令，等待Redis应答，Redis在接收到命令，处理后应答。在这种情况下，如果同时需要执行大量的命令，那就是等待上一条命令应答后再执行，这中间不仅仅多了RTT（Round Time Trip），而且还频繁的调用系统IO，发送网络请求。如下图。
为了提升效率，这时候Pipeline出现了，它允许客户端可以一次发送多条命令，而不等待上一条命令执行的结果，这和网络的Nagel算法有点像（TCP_NODELAY选项）。不仅减少了RTT，同时也减少了IO调用次数（IO调用涉及到用户态到内核态之间的切换）。如下图：
客户端这边首先将执行的命令写入到缓冲中，最后再一次性发送Redis。但是有一种情况就是，缓冲区的大小是有限制的，比如Jedis，限制为8192，超过了，则刷缓存，发送到Redis，但是不去处理Redis的应答，如上图所示那样。

**1.2 实现原理**
　　要支持Pipeline，其实既要服务端的支持，也要客户端支持。对于服务端来说，所需要的是能够处理一个客户端通过同一个TCP连接发来的多个命令，可以理解为，这里将多个命令切分，和处理单个命令一样（之前老生常谈的黏包现象），
Redis就是这样处理的。而客户端，则是要将多个命令缓存起来，缓冲区满了就发送，然后再写缓冲，最后才处理Redis的应答，如Jedis。

**1.3 从哪个方面提升性能**
正如上面所说的，一个是RTT，节省往返时间，但是另一个原因也很重要，就是IO系统调用。一个read系统调用，需要从用户态，切换到内核态。

**1.4 注意点**
　　Redis的Pipeline和Transaction不同，Transaction会存储客户端的命令，最后一次性执行，而Pipeline则是处理一条，响应一条，但是这里却有一点，就是客户端会并不会调用read去读取socket里面的缓冲数据，这也就造就了，
如果Redis应答的数据填满了该接收缓冲（SO_RECVBUF），那么客户端会通过ACK，WIN=0（接收窗口）来控制服务端不能再发送数据，那样子，数据就会缓冲在Redis的客户端应答列表里面。所以需要注意控制Pipeline的大小。如下图：

**2. Codis Pipeline**
　　在一般情况下，都会在Redis前面使用一个代理，来作负载以及高可用。这里在公司里面使用的是Codis，以Codis 3.2版本为例（3.2版本是支持Pipeline的）。
Codis在接收到客户端请求后，首先根据Key来计算出一个hash，映射到对应slots，然后转发请求到slots对应的Redis。在这过程中，一个客户端的多个请求，有可能会对应多个Redis，这个时候就需要保证请求的有序性（不能乱序），
Codis采用了一个Tasks队列，将请求依次放入队列，然后loopWriter从里面取，如果Task请求没有应答，则等待（这里和Java的Future是类似的）。内部BackenRedis是通过channel来进行通信的，dispatcher将Request通过channel发送到BackenRedis，然后BackenRedis处理完该请求，则将值填充到该Request里面。最后loopWriter等待到了值，则返回给客户端。如下图所示：



PipeLine看起来和事务很类似，感觉都是一批批处理，但两者还是有很大的区别。简单来说。

1、pipeline是客户端的行为，对于服务器来说是透明的，可以认为服务器无法区分客户端发送来的查询命令是以普通命令的形式还是以pipeline的形式发送到服务器的；

2、而事务则是实现在服务器端的行为，用户执行MULTI命令时，服务器会将对应这个用户的客户端对象设置为一个特殊的状态，在这个状态下后续用户执行的查询命令不会被真的执行，而是被服务器缓存起来，直到用户执行EXEC命令为止，服务器会将这个用户对应的客户端对象中缓存的命令按照提交的顺序依次执行。

3、应用pipeline可以提服务器的吞吐能力，并提高Redis处理查询请求的能力。

但是这里存在一个问题，当通过pipeline提交的查询命令数据较少，可以被内核缓冲区所容纳时，Redis可以保证这些命令执行的原子性。然而一旦数据量过大，超过了内核缓冲区的接收大小，那么命令的执行将会被打断，原子性也就无法得到保证。因此pipeline只是一种提升服务器吞吐能力的机制，如果想要命令以事务的方式原子性的被执行，还是需要事务机制，或者使用更高级的脚本功能以及模块功能。

4、可以将事务和pipeline结合起来使用，减少事务的命令在网络上的传输时间，将多次网络IO缩减为一次网络IO。

Redis提供了简单的事务，之所以说它简单，主要是因为它不支持事务中的回滚特性,同时无法实现命令之间的逻辑关系计算，当然也体现了Redis 的“keep it simple”的特性，下一小节介绍的Lua脚本同样可以实现事务的相关功能,但是功能要强大很多。



**3. 总结**
　　1、Pipeline减少了RTT，也减少了IO调用次数（IO调用涉及到用户态到内核态之间的切换）
　　2、需要控制Pipeline的大小，否则会消耗Redis的内存
　　3、Codis 3.2 Pipeline默认10K，3.1则是1024Jedis客户端缓存是8192，超过该大小则刷新缓存，或者直接发送

## Lua

Lua语言是在1993年由巴西一个大学研究小组发明，其设计目标是作为嵌入式程序移植到其他应用程序,它是由C语言实现的，虽然简单小巧但是功能强大，所以许多应用都选用它作为脚本语言，尤其是在游戏领域，暴雪公司的“魔兽世界”，“愤怒的小鸟”，Nginx将Lua语言作为扩展。Redis将Lua作为脚本语言可帮助开发者定制自己的Redis命令。

Redis 2.6 版本通过内嵌支持 Lua 环境。也就是说一般的运用，是不需要单独安装Lua的。

**通过使用LUA脚本：**

1、减少网络开销，在Lua脚本中可以把多个命令放在同一个脚本中运行；

2、原子操作，redis会将整个脚本作为一个整体执行，中间不会被其他命令插入（Redis执行命令是单线程）。

3、复用性，客户端发送的脚本会永远存储在redis中，这意味着其他客户端可以复用这一脚本来完成同样的逻辑。

不过为了我们方便学习Lua语言，我们还是单独安装一个Lua。

在Redis使用LUA脚本的好处包括：

1、减少网络开销，在Lua脚本中可以把多个命令放在同一个脚本中运行；

2、原子操作，Redis会将整个脚本作为一个整体执行，中间不会被其他命令插入。换句话说，编写脚本的过程中无需担心会出现竞态条件；

3、复用性，客户端发送的脚本会存储在Redis中，这意味着其他客户端可以复用这一脚本来完成同样的逻辑

### Lua入门

#### 安装Lua

Lua在linux中的安装

到官网下载lua的tar.gz的源码包

```
1、wget
http://www.lua.org/ftp/lua-5.3.6.tar.gz

2、tar -zxvf
lua-5.3.6.tar.gz

进入解压的目录：

3、cd lua-5.3.6

4、make linux

5、make install（需要在root用户下）

如果报错，说找不到readline/readline.h,
可以root用户下通过yum命令安装
yum -y install
libtermcap-devel ncurses-devel libevent-devel readline-devel

安装完以后再make linux  / make install

最后，直接输入 lua命令即可进入lua的控制台：
```

#### Lua基本语法

Lua 学习起来非常简单，当然再简单，它也是个独立的语言，自成体系，不可能完全在本课中全部讲述完毕，如果工作中有深研Lua的需要，可以参考《Lua程序设计》，作者罗伯拖·鲁萨利姆斯奇 (Roberto Ierusalimschy)。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/d21f4ee35b4446849e66b0cad788487b.png)

现在我们需要：print("Hello World！")

可以在命令行中输入程序并立即查看效果。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/d13c2c1b8079476f8027b408278dedd3.png)

或者编写一个Lua脚本

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/9076f1c0a2644eb0aa736ac4abca417f.png)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/6095e0b0147a400abb585c8f448df1eb.png)

然后执行

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/c2693e8536b7463fae6fb646bc596a19.png)

##### 注释

单行注释

```
两个减号是单行注释:  --
```

多行注释

```
--[[

注释内容

注释内容

--]]
```

##### 标示符

Lua 标示符用于定义一个变量，函数获取其他用户定义的项。标示符以一个字母 A 到 Z 或 a 到 z 或下划线 _ 开头后加上 0 个或多个字母，下划线，数字（0 到 9）。

最好不要使用下划线加大写字母的标示符，因为Lua的语言内部的一些保留字也是这样的。

Lua 不允许使用特殊字符如 @, $, 和 % 来定义标示符。 Lua 是一个区分大小写的编程语言。因此在 Lua 中
LIJIN与lijin 是两个不同的标示符。以下列出了一些正确的标示符：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/72723f45d40b4fa4a8ab91c5d589e298.png)

##### 关键词

以下列出了 Lua 的保留关键词。保留关键字不能作为常量或变量或其他用户自定义标示符：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/7b3f208442d5472cb0324fa4464c77d8.png)

同时一般约定，以下划线开头连接一串大写字母的名字（比如 _VERSION）被保留用于 Lua 内部全局变量。

##### 全局变量

在默认情况下，变量总是认为是全局的。

全局变量不需要声明，给一个变量赋值后即创建了这个全局变量，访问一个没有初始化的全局变量也不会出错，只不过得到的结果是：nil。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/24e4a0a6944a41a3a68174973986f0d1.png)

如果你想删除一个全局变量，只需要将变量赋值为nil。这样变量b就好像从没被使用过一样。换句话说, 当且仅当一个变量不等于nil时，这个变量即存在。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/efc037393c584ee99ce69781ed1c3450.png)

#### Lua中的数据类型

Lua 是动态类型语言，变量不要类型定义,只需要为变量赋值。
值可以存储在变量中，作为参数传递或结果返回。

Lua 中有 8 个基本类型分别为：nil、boolean、number、string、userdata、function、thread 和 table。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/d9a82f1846cf4fcfa40b46d6feb3c0f2.png)

我们可以使用 type 函数测试给定变量或者值的类型。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/81e52c43471d4669956b98995a316eba.png)

我们只选择几个要点做说明：

1、nil 类型表示一种没有任何有效值，它只有一个值 – nil，对于全局变量和 table，nil 还有一个"删除"作用，给全局变量或者 table 表里的变量赋一个 nil 值，等同于把它们删掉，nil 作类型比较时应该加上双引号 "。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/cb9db4ff81ca466ca9968a58da90f0c8.png)                  ![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/cbcd220e6e6042c8bb92a305f8c2b7de.png)

2、boolean 类型只有两个可选值：true（真） 和 false（假），Lua 把 false 和 nil 看作是 false，其他的都为 true，数字 0 也是 true。

3、Lua 默认只有一种 number类型 -- double（双精度）类型。

```
print(type(2))

print(type(2.2))

print(type(0.2))

print(type(2e+1))

print(type(0.2e-1))
```

都被看作是 number 类型

4、字符串由一对双引号或单引号来表示，也可以用[[ 与 ]] 表示，一般来说，单行文本用双引号或单引号，多行文本用[[ 与 ]] 。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/df4e1ab03ac847b4af790f24469a7593.png)

5、在对一个数字字符串上进行算术操作时，Lua 会尝试将这个数字字符串转成一个数字。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/840c3d97ca2346958862685e1cba9dcd.png)

6、字符串连接使用的是 ..

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/e220f0236ac045538b9001aa646958e6.png)

7、使用 # 来计算字符串的长度，放在字符串前面

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/ec4475d219bf4fefacc556bbf6835334.png)

8、table可以做为数组，也可以作为为Hash，table 不会固定长度大小，有新数据添加时 table 长度会自动增长，没初始的 table 都是 nil

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/0bc58d556bb44fa8a9e9163aef2c373c.png)

不同于其他语言的数组把 0 作为数组的初始索引，可以看到在Lua里表的默认初始索引一般以 1 开始。

把table做hash表用：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/2191fda0e1f64aaeb8d6be01a76d7810.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/13f693fa8390481291efa2788e541300.png)

#### Lua 中的函数

在 Lua中，函数以function开头，以end结尾，funcName是函数名，中间部分是函数体:

```
function
funcName ()

--[[

函数内容

--]]

end
```

比如定义一个字符串连接函数：

```
function
contact(str1,str2)

return
str1..str2

end

print(contact("hello","Lijin"))
```

#### Lua 变量

变量在使用前，需要在代码中进行声明，即创建该变量。

编译程序执行代码之前编译器需要知道如何给语句变量开辟存储区，用于存储变量的值。

Lua 变量有：全局变量、局部变量。

Lua 中的变量全是全局变量，那怕是语句块或是函数里，除非用 local 显式声明为局部变量。局部变量的作用域为从声明位置开始到所在语句块结束。

变量的默认值均为 nil。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/0fe72d0953ba4b60b1133dcae0c58f8e.png)

#### Lua中的控制语句

Lua中的控制语句和Java语言的差不多。

##### 循环控制

Lua支持while 循环、for 循环、repeat...until循环和循环嵌套，同时，Lua提供了break 语句和goto 语句。

我们重点来看看while 循环、for 循环。

**for 循环**

Lua 编程语言中 for语句有两大类：数值for循环、泛型for循环。

**数值for循环**

Lua 编程语言中数值 for 循环语法格式:

```
for var=exp1,exp2,exp3 do  
    <执行体>  
end  
```

var 从 exp1 变化到 exp2，每次变化以 exp3 为步长递增 var，并执行一次 "执行体"。exp3 是可选的，如果不指定，默认为1。

**泛型for循环**

泛型 for 循环通过一个迭代器函数来遍历所有值，类似 java 中的 foreach 语句。Lua 编程语言中泛型 for 循环语法格式:

--打印数组a的所有值

```
a = {"one", "two", "three"}
for i, v in ipairs(a) do
    print(i, v)
end 
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/2561648bc2f14352a02afca22a8ed0be.png)

i是数组索引值，v是对应索引的数组元素值。ipairs是Lua提供的一个迭代器函数，用来迭代数组。

```
tbl3={age=18,name='lijin'}
	for i, v in pairs(tbl3) do
	print(i,v)
end
```

while循环

```
while(condition)
do
   statements
end
a=10 while(a<20)  do  print("a= ",a)  a=a+1 end
```

##### if条件控制

Lua支持if 语句、if...else 语句和if 嵌套语句。

if **语句语法格式如下：**

```
if(布尔表达式)
then
   --[ 在布尔表达式为 true 时执行的语句 --]
end
if...else 语句语法格式如下：
if(布尔表达式)
then
   --[ 布尔表达式为 true 时执行该语句块 --]
else
   --[ 布尔表达式为 false 时执行该语句块 --]
end
```

#### Lua 运算符

Lua提供了以下几种运算符类型：

算术运算符

```
+	加法
-	减法
*	乘法
/	除法
%	取余
^	乘幂
-	负号
关系运算符
==	等于
~=	不等于
>	大于
<	小于
>=	大于等于
<=	小于等于
逻辑运算符
and	逻辑与操作符
or	逻辑或操作符
not	逻辑非操作符
```

#### Lua其他特性

Lua支持模块与包，也就是封装库，支持元表(Metatable)，支持协程(coroutine)，支持文件IO操作，支持错误处理，支持代码调试，支持Lua垃圾回收，支持面向对象和数据库访问，更多详情请参考对应书籍。

#### Java对Lua的支持

目前Java生态中，对Lua的支持是LuaJ，是一个 Java 的 Lua 解释器，基于 Lua 5.2.x 版本。

##### Maven

```
<dependency>
    <groupId>org.luaj</groupId>
    <artifactId>luaj-jse</artifactId>
    <version>3.0.1</version>
</dependency>
```

##### 参考代码

参见luaj模块，请注意，本代码仅供参考，在工作中需要使用Lua语言或者Java中执行Lua脚本的，请自行仔细学习Lua语言本身和luaj-jse使用，不提供任何技术支持。一般这种形式用得非常少。

### Redis中的Lua

#### eval 命令

##### 命令格式

```
EVAL script numkeys key [key ...] arg [arg ...]
```

##### 命令说明

**1、script 参数：**

是一段 Lua 脚本程序，它会被运行在Redis 服务器上下文中，这段脚本不必（也不应该）定义为一个 Lua 函数。

**2、numkeys 参数：**

用于指定键名参数的个数。

**3、key [key...] 参数：**
从EVAL 的第三个参数开始算起，使用了 numkeys 个键（key），表示在脚本中所用到的那些 Redis 键（key），这些键名参数可以在 Lua 中通过全局变量 KEYS 数组，用1为基址的形式访问（KEYS[1],KEYS[2]···）。

**4、arg [arg...]参数：**

可以在 Lua 中通过全局变量 ARGV 数组访问，访问的形式和 KEYS 变量类似（ARGV[1],ARGV[2]···）。

##### 示例

```
eval "return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}" 2 key1 key2 first second
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/f881ce52030b4dbf9fbd377f14557775.png)

在这个范例中key [key ...] 参数的作用不明显，其实它最大的作用是方便我们在Lua 脚本中调用 Redis 命令

##### Lua 脚本中调用 Redis 命令

这里我们主要记住 call() 命令即可：

```
eval "return redis.call('mset',KEYS[1],ARGV[1],KEYS[2],ARGV[2])" 2 key1 key2 first second
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/2e3bbe28404e46e4b0dc73b369ee73fc.png)

#### evalsha 命令

但是eval命令要求你在每次执行脚本的时候都发送一次脚本，所以Redis 有一个内部的缓存机制，因此它不会每次都重新编译脚本，不过在很多场合，付出无谓的带宽来传送脚本主体并不是最佳选择。

为了减少带宽的消耗， Redis 提供了evalsha 命令，它的作用和 EVAL一样，都用于对脚本求值，但它接受的第一个参数不是脚本，而是脚本的 SHA1 摘要。

这里就需要借助script命令。

script flush ：清除所有脚本缓存。

script exists ：根据给定的脚本校验，检查指定的脚本是否存在于脚本缓存。

script load ：将一个脚本装入脚本缓存，返回SHA1摘要，但并不立即运行它。

script kill ：杀死当前正在运行的脚本。

这里的 SCRIPT LOAD 命令就可以用来生成脚本的 SHA1 摘要

```
script load "return redis.call('set',KEYS[1],ARGV[1])"
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/ae4a0ad74f0842ec8a188c875de9f1f9.png)

然后就可以执行这个脚本

```
evalsha "c686f316aaf1eb01d5a4de1b0b63cd233010e63d" 1 key1 testscript
```

#### redis-cli 执行脚本

可以使用 redis-cli 命令直接执行脚本，这里我们直接新建一个 lua 脚本文件，用来获取刚刚存入 Redis 的
key1的值，vim redis.lua，然后编写 Lua 命令：

```
local value = redis.call('get','key1')
return value
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/ef5fe46077234a54a73c53eff6c03038.png)

然后执行

```
./redis-cli -p 6379 --eval ../scripts/test.lua
```

也可以

```
./redis-cli -p 6379 script load "$(cat ../scripts/test.lua)"
```

​        

## Redis与限流

### 使用Redis+Lua语言实现限流

项目代码

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/68829990c81046f7913800d86dcada24.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/713feae34c8e461fb3fe9b0d7f7b306d.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/481bd751a71b45359430fe956b147aa9.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/a72443074b544e69bfa168dd94210fcb.png)

**方案好处：**

支持分布式

**使用lua脚本的好处：**

减少网络开销

原子操作

复用

### 限流算法

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/3f0faeede01f47dab8fcc33273dfcaf3.png)

#### 固定窗口算法

简单粗暴，但是有临界问题

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/73ebecc6765348858eeb8fe2b92bee01.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/d6a248cc7ea94b3ca987f976e1f325b0.png)

#### 滑动窗口算法

在线演示滑动窗口:

```
https://media.pearsoncmg.com/aw/ecs_kurose_compnetwork_7/cw/content/interactiveanimations/selective-repeat-protocol/index.html
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/f22d3fac77c34d748a666ac66a218099.png)

滑动窗口通俗来讲就是一种流量控制技术。

它本质上是描述接收方的TCP数据报缓冲区大小的数据，发送方根据这个数据来计算自己最多能发送多长的数据，如果发送方收到接收方的窗口大小为0的TCP数据报，那么发送方将停止发送数据，等到接收方发送窗口大小不为0的数据报的到来。

首先是第一次发送数据这个时候的窗口大小是根据链路带宽的大小来决定的。我们假设这个时候窗口的大小是3。这个时候接受方收到数据以后会对数据进行确认告诉发送方我下次希望手到的是数据是多少。这里我们看到接收方发送的ACK=3(这是发送方发送序列2的回答确认，下一次接收方期望接收到的是3序列信号)。这个时候发送方收到这个数据以后就知道我第一次发送的3个数据对方只收到了2个。就知道第3个数据对方没有收到。下次在发送的时候就从第3个数据开始发。

此时窗口大小变成了2 。

于是发送方发送2个数据。看到接收方发送的ACK是5就表示他下一次希望收到的数据是5，发送方就知道我刚才发送的2个数据对方收了这个时候开始发送第5个数据。

这就是滑动窗口的工作机制，当链路变好了或者变差了这个窗口还会发生变话，并不是第一次协商好了以后就永远不变了。

所以滑动窗口协议，是TCP使用的一种流量控制方法。该协议允许发送方在停止并等待确认前可以连续发送多个分组。由于发送方不必每发一个分组就停下来等待确认，因此该协议可以加速数据的传输。

只有在接收窗口向前滑动时（与此同时也发送了确认），发送窗口才有可能向前滑动。

收发两端的窗口按照以上规律不断地向前滑动，因此这种协议又称为滑动窗口协议。

##### TCP中的滑动窗口

发送方和接收方都会维护一个数据帧的序列，这个序列被称作窗口。发送方的窗口大小由接收方确认，目的是控制发送速度，以免接收方的缓存不够大导致溢出，同时控制流量也可以避免网络拥塞。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/157a7223e2314486a4714c7b1761f2a9.png)

在TCP 的可靠性的图中，我们可以看到，发送方每发送一个数据接收方就要给发送方一个ACK对这个数据进行确认。只有接收了这个确认数据以后发送方才能传输下个数据。

存在的问题：如果窗口过小，当传输比较大的数据的时候需要不停的对数据进行确认，这个时候就会造成很大的延迟。

如果窗口过大，我们假设发送方一次发送100个数据，但接收方只能处理50个数据，这样每次都只对这50个数据进行确认。发送方下一次还是发送100个数据，但接受方还是只能处理50个数据。这样就避免了不必要的数据来拥塞我们的链路。

因此，我们引入了滑动窗口。

#### 漏洞算法

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/b16ce9a81f064704b0a78fc496e77d7a.png)

**定义**

先有一个桶，桶的容量是固定的。

以任意速率向桶流入水滴，如果桶满了则溢出(被丢弃)。

桶底下有个洞，按照固定的速率从桶中流出水滴。

**特点**

漏桶核心是：请求来了以后，直接进桶，然后桶根据自己的漏洞大小慢慢往外面漏。

具体实现的时候要考虑性能（比如Redis实现的时候数据结构的操作是不是会导致性能问题）

#### 令牌算法

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/5bf2ba9cd7654e7b926fdac7b05392e5.png)

**定义**

先有一个桶，容量是固定的，是用来放令牌的。

以固定速率向桶放令牌，如果桶满了就不放令牌了。

Ø处理请求是先从桶拿令牌，先拿到令牌再处理请求，拿不到令牌同样也被限流了。

**特点**

突发情况下可以一次拿多个令牌进行处理。

具体实现的时候要考虑性能（比如Redis实现的时候数据结构的操作是不是会导致性能问题）

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/9fe82d044bfd4b3d847bafbd63594c0a.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663569981087/c00f351489d64a36885e9a45a13a9bda.png)****

### Redis底层原理

## 持久化

Redis虽然是个内存数据库，但是Redis支持RDB和AOF两种持久化机制，将数据写往磁盘，可以有效地避免因进程退出造成的数据丢失问题，当下次重启时利用之前持久化的文件即可实现数据恢复。

### RDB

RDB持久化是把当前进程数据生成快照保存到硬盘的过程。所谓内存快照，就是指内存中的数据在某一个时刻的状态记录。这就类似于照片，当你给朋友拍照时，一张照片就能把朋友一瞬间的形象完全记下来。RDB 就是Redis DataBase 的缩写。

#### 给哪些内存数据做快照?

Redis 的数据都在内存中，为了提供所有数据的可靠性保证，它执行的是全量快照，也就是说，把内存中的所有数据都记录到磁盘中。但是，RDB 文件就越大，往磁盘上写数据的时间开销就越大。

分为自动触发与手动触发两种

1.redis.conf中配置：

save 900 1  //900s内有1个key被修改 

save 300 10  //300s内有10个key被修改 

save 60 10000 //60s内有10000个key被修改

2.执行命令

shutdown  正常关闭服务

flushall	备份空文件

save	会阻塞

bgsave	后台线程

#### RDB文件的生成是否会阻塞主线程

Redis 提供了两个手动命令来生成 RDB 文件，分别是 save 和 bgsave。

save：在主线程中执行，会导致阻塞；对于内存比较大的实例会造成长时间阻塞，线上环境不建议使用。
bgsave：创建一个子进程，专门用于写入 RDB 文件，避免了主线程的阻塞，这也是Redis RDB 文件生成的默认配置。

命令实战演示

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663570141014/ab097682d50b4b5d8b6e5046b990000d.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663570141014/7f4d4eaa5daa4bfa9f73dac0feafa985.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663570141014/cc284f936af741808963a29386e505ca.png)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663570141014/81c4dfc0ca4e411e8a95dfa4eaf53f93.png)

除了执行命令手动触发之外，Redis内部还存在自动触发RDB 的持久化机制，例如以下场景:

1)使用save相关配置,如“save m n”。表示m秒内数据集存在n次修改时，自动触发bgsave。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663570141014/f05c9dc4808f4d5197058962036a6680.png)

2）如果从节点执行全量复制操作，主节点自动执行bgsave生成RDB文件并发送给从节点。

3)执行debug reload命令重新加载Redis 时，也会自动触发save操作。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663570141014/ab4880174f914059843d220ca91a89bf.png)

4）默认情况下执行shutdown命令时，如果没有开启AOF持久化功能则自动执行bgsave。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663252342001/a066e0616af6479cb29e109feae91482.png)

关闭RDB持久化，在课程讲述的Redis版本（6.2.4）上，是将配置文件中的save配置改为 save “”

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663570141014/c0e0254d5db84e60b838984d5482b3a4.png)

#### bgsave执的行流程

为了快照而暂停写操作，肯定是不能接受的。所以这个时候，Redis 就会借助操作系统提供的写时复制技术（Copy-On-Write, COW），在执行快照的同时，正常处理写操作。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663570141014/b43882d7ee83429eb8afe0b92fab7ea0.png)

bgsave 子进程是由主线程 fork 生成的，可以共享主线程的所有内存数据。bgsave 子进程运行后，开始读取主线程的内存数据，并把它们写入 RDB 文件。

如果主线程对这些数据也都是读操作（例如图中的键值对 A），那么，主线程和bgsave 子进程相互不影响。但是，如果主线程要修改一块数据（例如图中的键值对 B），那么，这块数据就会被复制一份，生成该数据的副本。然后，bgsave 子进程会把这个副本数据写入 RDB 文件，而在这个过程中，主线程仍然可以直接修改原来的数据。

这既保证了快照的完整性，也允许主线程同时对数据进行修改，避免了对正常业务的影响。

#### RDB文件

RDB文件保存在dir配置指定的目录下，文件名通过dbfilename配置指定。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663570141014/bd096f38a702402db4452162cc2495e7.png)

可以通过执行config set dir {newDir}和config set dbfilename (newFileName}运行期动态执行,当下次运行时RDB文件会保存到新目录。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663570141014/e850ee94caeb4fc290dd24a432eb5598.png)

Redis默认采用LZF算法对生成的RDB文件做压缩处理，压缩后的文件远远小于内存大小，默认开启，可以通过参数config set rdbcompression { yes |no}动态修改。
虽然压缩RDB会消耗CPU，但可大幅降低文件的体积，方便保存到硬盘或通过网维示络发送给从节点,因此线上建议开启。
如果 Redis加载损坏的RDB文件时拒绝启动,并打印如下日志:

```
Short read or OOM loading DB. Unrecoverable error，aborting now.
```

这时可以使用Redis提供的redis-check-rdb工具(老版本是redis-check-dump)检测RDB文件并获取对应的错误报告。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663570141014/86e130bb315f42569c8fd24049aa98da.png)

#### RDB的优缺点

##### RDB的优点

RDB是一个紧凑压缩的二进制文件，代表Redis在某个时间点上的数据快照。非常适用于备份,全量复制等场景。

比如每隔几小时执行bgsave备份，并把 RDB文件拷贝到远程机器或者文件系统中(如hdfs),，用于灾难恢复。

Redis加载RDB恢复数据远远快于AOF的方式。

##### RDB的缺点

RDB方式数据没办法做到实时持久化/秒级持久化。因为bgsave每次运行都要执行fork操作创建子进程,属于重量级操作,频繁执行成本过高。

RDB文件使用特定二进制格式保存，Redis版本演进过程中有多个格式的RDB版本，存在老版本Redis服务无法兼容新版RDB格式的问题。

### Redis中RDB导致的数据丢失问题

针对RDB不适合实时持久化的问题,Redis提供了AOF持久化方式来解决。

如下图所示，我们先在 T0 时刻做了一次快照（下一次快照是T4时刻），然后在T1时刻，数据块 5 和 8 被修改了。如果在T2时刻，机器宕机了，那么，只能按照 T0 时刻的快照进行恢复。此时，数据块 5 和 8 的修改值因为没有快照记录，就无法恢复了。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663570141014/c9113306297743158cf82baad4fe4329.png)

所以这里可以看出，如果想丢失较少的数据，那么T4-T0就要尽可能的小，但是如果频繁地执行全量
快照，也会带来两方面的开销：

1、频繁将全量数据写入磁盘，会给磁盘带来很大压力，多个快照竞争有限的磁盘带宽，前一个快照还没有做完，后一个又开始做了，容易造成恶性循环。

2、另一方面，bgsave 子进程需要通过 fork 操作从主线程创建出来。虽然子进程在创建后不会再阻塞主线程，但是，fork 这个创建过程本身会阻塞主线程，而且主线程的内存越大，阻塞时间越长。如果频繁fork出bgsave 子进程，这就会频繁阻塞主线程了。

所以基于这种情况，我们就需要AOF的持久化机制。

### AOF

AOF(append only file)持久化:以独立日志的方式记录每次写命令，重启时再重新执行AOF文件中的命令达到恢复数据的目的。AOF的主要作用是解决了数据持久化的实时性,目前已经是Redis持久化的主流方式。理解掌握好AOF持久化机制对我们兼顾数据安全性和性能非常有帮助。

#### 使用AOF

开启AOF功能需要设置配置:appendonly yes，默认不开启。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663570141014/dd28cece52794f988be6675f817582ba.png)

AOF文件名通过appendfilename配置设置，默认文件名是appendonly.aof。保存路径同RDB持久化方式一致，通过dir配置指定。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663570141014/f9e1a9fd854b4631876b9156187b54d6.png)

### AOF的工作流程

AOF的工作流程主要是4个部分:命令写入( append)、文件同步( sync)、文件重写(rewrite)、重启加载( load)。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663570141014/1ed964de0c46425d8ee02099c098c775.png)

#### 命令写入

AOF命令写入的内容直接是RESP文本协议格式。例如lpush tianming A B这条命令，在AOF缓冲区会追加如下文本:

```
*3\r\n$6\r\nlupush\r\n$5\r\tianming\r\n$3\r\nA B
```

看看 AOF 日志的内容。其中，“*3”表示当前命令有三个部分，每部分都是由“$+数字”开头，后面紧跟着
具体的命令、键或值。这里，“数字”表示这部分中的命令、键或值一共有多少字节。例如，“$3 set”表示这部分有 3 个字节，也就是“set”命令。

1 )AOF为什么直接采用文本协议格式?

文本协议具有很好的兼容性。开启AOF后，所有写入命令都包含追加操作，直接采用协议格式，避免了二次处理开销。文本协议具有可读性,方便直接修改和处理。

2）AOF为什么把命令追加到aof_buf中?

Redis使用单线程响应命令，如果每次写AOF文件命令都直接追加到硬盘，那么性能完全取决于当前硬盘负载。先写入缓冲区aof_buf中，还有另一个好处，Redis可以提供多种缓冲区同步硬盘的策略，在性能和安全性方面做出平衡。

Redis提供了多种AOF缓冲区同步文件策略，由参数appendfsync控制。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663570141014/251fa2902c7549f09ff37a2a1802a3d5.png)

**always**

同步写回：每个写命令执行完，立马同步地将日志写回磁盘；

**everysec**

每秒写回：每个写命令执行完，只是先把日志写到 AOF 文件的内存缓冲区，每隔一秒把缓冲区中的内容写入磁盘；

**no**

操作系统控制的写回：每个写命令执行完，只是先把日志写到 AOF 文件的内存缓冲区，由操作系统决定何时将缓冲区内容写回磁盘，通常同步周期最长30秒。

很明显，配置为always时，每次写入都要同步AOF文件，在一般的SATA 硬盘上，Redis只能支持大约几百TPS写入,显然跟Redis高性能特性背道而驰,不建议配置。

配置为no，由于操作系统每次同步AOF文件的周期不可控,而且会加大每次同步硬盘的数据量,虽然提升了性能,但数据安全性无法保证。

配置为everysec，是建议的同步策略，也是默认配置，做到兼顾性能和数据安全性。理论上只有在系统突然宕机的情况下丢失1秒的数据。(严格来说最多丢失1秒数据是不准确的)

想要获得高性能，就选择 no 策略；如果想要得到高可靠性保证，就选择always 策略；如果允许数据有一点丢失，又希望性能别受太大影响的话，那么就选择everysec 策略。

#### 重写机制

随着命令不断写入AOF，文件会越来越大，为了解决这个问题，Redis引入AOF重写机制压缩文件体积。AOF文件重写是把Redis进程内的数据转化为写命令同步到新AOF文件的过程。

**重写后的AOF 文件为什么可以变小?有如下原因:**

1)进程内已经超时的数据不再写入文件。

2)旧的AOF文件含有无效命令，如set a 111、set a 222等。重写使用进程内数据直接生成，这样新的AOF文件只保留最终数据的写入命令。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663570141014/1e5236bdea3c4f7d943848ab145b95c8.png)

3）多条写命令可以合并为一个，如:lpush list a、lpush list b、lpush list c可以转化为: lpush list a b c。为了防止单条命令过大造成客户端缓冲区溢出，对于list、set、hash、zset等类型操作，以64个元素为界拆分为多条。

AOF重写降低了文件占用空间，除此之外，另一个目的是:更小的AOF文件可以更快地被Redis加载。

AOF重写过程可以手动触发和自动触发:

手动触发:直接调用bgrewriteaof命令。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663570141014/fe8fa92c17da42ccb7185b022285798e.png)

自动触发:根据auto-aof-rewrite-min-size和 auto-aof-rewrite-percentage参数确定自动触发时机。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663570141014/66e1ef551e764240b8057b98f1ae5e80.png)

auto-aof-rewrite-min-size:表示运行AOF重写时文件最小体积，默认为64MB。

auto-aof-rewrite-percentage  :代表当前AOF 文件空间(aof_currentsize）和上一次重写后AOF 文件空间(aof_base_size)的比值。

**另外，如果在Redis在进行AOF重写时，有写入操作，这个操作也会被写到重写日志的缓冲区。这样，重写日志也不会丢失最新的操作。**

#### 重启加载

AOF和 RDB 文件都可以用于服务器重启时的数据恢复。redis重启时加载AOF与RDB的顺序是怎么样的呢？

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663570141014/edfe68fdaaee4c70a29981f06935eaa5.png)

1，当AOF和RDB文件同时存在时，优先加载AOF

2，若关闭了AOF，加载RDB文件

3，加载AOF/RDB成功，redis重启成功

4，AOF/RDB存在错误，启动失败打印错误信息

#### 文件校验

加载损坏的AOF 文件时会拒绝启动，对于错误格式的AOF文件，先进行备份，然后采用redis-check-aof --fix命令进行修复，对比数据的差异，找出丢失的数据，有些可以人工修改补全。

AOF文件可能存在结尾不完整的情况，比如机器突然掉电导致AOF尾部文件命令写入不全。Redis为我们提供了aof-load-truncated  配置来兼容这种情况，默认开启。加载AOF时当遇到此问题时会忽略并继续启动,同时如下警告日志。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663570141014/9d97eac029914c62b096a8ba16beee3d.png)

### RDB-AOF混合持久化

通过 `aof-use-rdb-preamble`  配置项可以打开混合开关，yes则表示开启，no表示禁用，默认是禁用的，可通过config set修改

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663570141014/2b16c9021dc24f55814b31871c08142a.png)

该状态开启后，如果执行bgrewriteaof命令，则会把当前内存中已有的数据弄成二进程存放在aof文件中，这个过程模拟了rdb生成的过程，然后Redis后面有其他命令，在触发下次重写之前，依然采用AOF追加的方式

**总流程图：**

![image-20231201174433627](E:\图灵课堂\Redis专题\笔记\1、Redis专题.assets\image-20231201174433627.png)



### Redis持久化相关的问题

#### 主线程、子进程和后台线程的联系与区别？

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663570141014/82a52fd3e3d047ba975af4966d194bc4.png)

**进程和线程的区别**

从操作系统的角度来看，进程一般是指资源分配单元，例如一个进程拥有自己的堆、栈、虚存空间（页表）、文件描述符等；

而线程一般是指 CPU 进行调度和执行的实体。

一个进程启动后，没有再创建额外的线程，那么，这样的进程一般称为主进程或主线程。

Redis 启动以后，本身就是一个进程，它会接收客户端发送的请求，并处理读写操作请求。而且，接收请求和处理请求操作是 Redis 的主要工作，Redis 没有再依赖于其他线程，所以，我一般把完成这个主要工作的 Redis 进程，称为主进程或主线程。

**主线程与子进程**

通过fork创建的子进程，一般和主线程会共用同一片内存区域，所以上面就需要使用到写时复制技术确保安全。

**后台线程**

从 4.0 版本开始，Redis 也开始使用pthread_create 创建线程，这些线程在创建后，一般会自行执行一些任务，例如执行异步删除任务

#### Redis持久化过程中有没有其他潜在的阻塞风险？

当Redis做RDB或AOF重写时，一个必不可少的操作就是执行**fork操作创建子进程**,对于大多数操作系统来说fork是个重量级错误。虽然fork创建的子进程不需要拷贝父进程的物理内存空间，但是会复制父进程的空间内存页表。例如对于10GB的Redis进程，需要复制大约20MB的内存页表，因此fork操作耗时跟进程总内存量息息相关，如果使用虚拟化技术，特别是Xen虚拟机,fork操作会更耗时。

**fork耗时问题定位:**

对于高流量的Redis实例OPS可达5万以上，如果fork操作耗时在秒级别将拖慢Redis几万条命令执行，对线上应用延迟影响非常明显。正常情况下fork耗时应该是每GB消耗20毫秒左右。可以在info stats统计中查latest_fork_usec指标获取最近一次fork操作耗时,单位微秒。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663570141014/8dfdf71e2458452593affdea53138644.png)

如何改善fork操作的耗时:

1）优先使用物理机或者高效支持fork操作的虚拟化技术

2）控制Redis实例最大可用内存，fork耗时跟内存量成正比,线上建议每个Redis实例内存控制在10GB 以内。

3）降低fork操作的频率，如适度放宽AOF自动触发时机，避免不必要的全量复制等。

#### 为什么主从库间的复制不使用 AOF？

1、RDB 文件是二进制文件，无论是要把 RDB 写入磁盘，还是要通过网络传输 RDB，IO效率都比记录和传输 AOF 的高。

2、在从库端进行恢复时，用 RDB 的恢复效率要高于用 AOF。

## 分布式锁

### Redis分布式锁最简单的实现

想要实现分布式锁，必须要求 Redis 有「互斥」的能力，我们可以使用 SETNX 命令，这个命令表示SET if Not Exists，即如果 key 不存在，才会设置它的值，否则什么也不做。

两个客户端进程可以执行这个命令，达到互斥，就可以实现一个分布式锁。

客户端 1 申请加锁，加锁成功：

客户端 2 申请加锁，因为它后到达，加锁失败：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663570141014/c5666e4aba6642fcb4c14c43588acf2c.png)

此时，加锁成功的客户端，就可以去操作「共享资源」，例如，修改 MySQL 的某一行数据，或者调用一个 API 请求。

操作完成后，还要及时释放锁，给后来者让出操作共享资源的机会。如何释放锁呢？

也很简单，直接使用 DEL 命令删除这个 key 即可，这个逻辑非常简单。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663570141014/6a35bef0bf0c48d1bbad8e3057ed2024.png)

但是，它存在一个很大的问题，当客户端 1 拿到锁后，如果发生下面的场景，就会造成「死锁」：

1、程序处理业务逻辑异常，没及时释放锁

2、进程挂了，没机会释放锁

这时，这个客户端就会一直占用这个锁，而其它客户端就「永远」拿不到这把锁了。怎么解决这个问题呢？

### 如何避免死锁？

我们很容易想到的方案是，在申请锁时，给这把锁设置一个「租期」。

在 Redis 中实现时，就是给这个 key 设置一个「过期时间」。这里我们假设，操作共享资源的时间不会超过 10s，那么在加锁时，给这个 key 设置 10s 过期即可：

```
SETNX lock 1    // 加锁
EXPIRE lock 10  // 10s后自动过期
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663570141014/d630c8b424ce4f5c82a544232e429521.png)

这样一来，无论客户端是否异常，这个锁都可以在 10s 后被「自动释放」，其它客户端依旧可以拿到锁。

但现在还是有问题：

现在的操作，加锁、设置过期是 2 条命令，有没有可能只执行了第一条，第二条却「来不及」执行的情况发生呢？例如：

* SETNX 执行成功，执行EXPIRE  时由于网络问题，执行失败
* SETNX 执行成功，Redis 异常宕机，EXPIRE 没有机会执行
* SETNX 执行成功，客户端异常崩溃，EXPIRE也没有机会执行

总之，这两条命令不能保证是原子操作（一起成功），就有潜在的风险导致过期时间设置失败，依旧发生「死锁」问题。

在 Redis 2.6.12 之后，Redis 扩展了 SET 命令的参数，用这一条命令就可以了：

```
SET lock 1 EX 10 NX
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663570141014/5c2cd7869c694c42ba0ee03709bb11d5.png)

### 锁被别人释放怎么办？

上面的命令执行时，每个客户端在释放锁时，都是「无脑」操作，并没有检查这把锁是否还「归自己持有」，所以就会发生释放别人锁的风险，这样的解锁流程，很不「严谨」！如何解决这个问题呢？

解决办法是：客户端在加锁时，设置一个只有自己知道的「唯一标识」进去。

例如，可以是自己的线程 ID，也可以是一个 UUID（随机且唯一），这里我们以UUID 举例：

```
SET lock $uuid EX 20 NX
```

之后，在释放锁时，要先判断这把锁是否还归自己持有，伪代码可以这么写：

```
if redis.get("lock") == $uuid:
    redis.del("lock")
```

这里释放锁使用的是 GET + DEL 两条命令，这时，又会遇到我们前面讲的原子性问题了。这里可以使用lua脚本来解决。

安全释放锁的 Lua 脚本如下：

```
if redis.call("GET",KEYS[1]) == ARGV[1]
then
    return redis.call("DEL",KEYS[1])
else
    return 0
end
```

好了，这样一路优化，整个的加锁、解锁的流程就更「严谨」了。

这里我们先小结一下，基于 Redis 实现的分布式锁，一个严谨的的流程如下：

1、加锁

```
SET lock_key $unique_id EX $expire_time NX
```

2、操作共享资源

3、释放锁：Lua 脚本，先 GET 判断锁是否归属自己，再DEL 释放锁

### Java代码实现分布式锁

```
package com.tuling.redis.lock;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.params.SetParams;

import java.util.Arrays;
import java.util.UUID;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;

/**
 * 分布式锁的实现
 */
@Component
public class RedisDistLock implements Lock {

    private final static int LOCK_TIME = 5*1000;
    private final static String RS_DISTLOCK_NS = "tdln:";
    /*
     if redis.call('get',KEYS[1])==ARGV[1] then
        return redis.call('del', KEYS[1])
    else return 0 end
     */
    private final static String RELEASE_LOCK_LUA =
            "if redis.call('get',KEYS[1])==ARGV[1] then\n" +
                    "        return redis.call('del', KEYS[1])\n" +
                    "    else return 0 end";
    /*保存每个线程的独有的ID值*/
    private ThreadLocal<String> lockerId = new ThreadLocal<>();

    /*解决锁的重入*/
    private Thread ownerThread;
    private String lockName = "lock";

    @Autowired
    private JedisPool jedisPool;

    public String getLockName() {
        return lockName;
    }

    public void setLockName(String lockName) {
        this.lockName = lockName;
    }

    public Thread getOwnerThread() {
        return ownerThread;
    }

    public void setOwnerThread(Thread ownerThread) {
        this.ownerThread = ownerThread;
    }

    @Override
    public void lock() {
        while(!tryLock()){
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    @Override
    public void lockInterruptibly() throws InterruptedException {
        throw new UnsupportedOperationException("不支持可中断获取锁！");
    }

    @Override
    public boolean tryLock() {
        Thread t = Thread.currentThread();
        if(ownerThread==t){/*说明本线程持有锁*/
            return true;
        }else if(ownerThread!=null){/*本进程里有其他线程持有分布式锁*/
            return false;
        }
        Jedis jedis = null;
        try {
            String id = UUID.randomUUID().toString();
            SetParams params = new SetParams();
            params.px(LOCK_TIME);
            params.nx();
            synchronized (this){/*线程们，本地抢锁*/
                if((ownerThread==null)&&
                "OK".equals(jedis.set(RS_DISTLOCK_NS+lockName,id,params))){
                    lockerId.set(id);
                    setOwnerThread(t);
                    return true;
                }else{
                    return false;
                }
            }
        } catch (Exception e) {
            throw new RuntimeException("分布式锁尝试加锁失败！");
        } finally {
            jedis.close();
        }
    }

    @Override
    public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {
        throw new UnsupportedOperationException("不支持等待尝试获取锁！");
    }

    @Override
    public void unlock() {
        if(ownerThread!=Thread.currentThread()) {
            throw new RuntimeException("试图释放无所有权的锁！");
        }
        Jedis jedis = null;
        try {
            jedis = jedisPool.getResource();
            Long result = (Long)jedis.eval(RELEASE_LOCK_LUA,
                    Arrays.asList(RS_DISTLOCK_NS+lockName),
                    Arrays.asList(lockerId.get()));
            if(result.longValue()!=0L){
                System.out.println("Redis上的锁已释放！");
            }else{
                System.out.println("Redis上的锁释放失败！");
            }
        } catch (Exception e) {
            throw new RuntimeException("释放锁失败！",e);
        } finally {
            if(jedis!=null) jedis.close();
            lockerId.remove();
            setOwnerThread(null);
            System.out.println("本地锁所有权已释放！");
        }
    }

    @Override
    public Condition newCondition() {
        throw new UnsupportedOperationException("不支持等待通知操作！");
    }

}

```

### 锁过期时间不好评估怎么办？

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663570141014/59852ac3e208452fa1ef0f9957f9231a.png)

看上面这张图，加入key的失效时间是10s，但是客户端C在拿到分布式锁之后，然后业务逻辑执行超过10s，那么问题来了，在客户端C释放锁之前，其实这把锁已经失效了，那么客户端A和客户端B都可以去拿锁，这样就已经失去了分布式锁的功能了！！！

比较简单的妥协方案是，尽量「冗余」过期时间，降低锁提前过期的概率，但是这个并不能完美解决问题，那怎么办呢？

#### 分布式锁加入看门狗

加锁时，先设置一个过期时间，然后我们开启一个「守护线程」，定时去检测这个锁的失效时间，如果锁快要过期了，操作共享资源还未完成，那么就自动对锁进行「续期」，重新设置过期时间。

这个守护线程我们一般也把它叫做「看门狗」线程。

为什么要使用守护线程：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663570141014/d427a96d4a1046c1968e69e44640980f.png)

#### 分布式锁加入看门狗代码实现

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663570141014/a2bb783aad3041ea9c285936bd3b366d.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663570141014/581aa5fa8b24431c8a1c41bcc4491783.png)

运行效果：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663570141014/357109a4cdb44fd2a7da48bb3eb528e8.png)

### Redisson中的分布式锁

Redisson把这些工作都封装好了

```
     <dependency>
            <groupId>org.redisson</groupId>
            <artifactId>redisson</artifactId>
            <version>3.12.3</version>
        </dependency>
```

```
package com.tuling.redis.config;

import org.redisson.Redisson;
import org.redisson.api.RedissonClient;
import org.redisson.config.Config;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MyRedissonConfig {
    /**
     * 所有对Redisson的使用都是通过RedissonClient
     */
    @Bean(destroyMethod="shutdown")
    public RedissonClient redisson(){
        //1、创建配置
        Config config = new Config();
        config.useSingleServer().setAddress("redis://127.0.0.1:6379");

        //2、根据Config创建出RedissonClient实例
        RedissonClient redisson = Redisson.create(config);
        return redisson;
    }
}

```

```
package com.tuling.redis.redisbase.adv;


import com.tuling.redis.lock.rdl.RedisDistLockWithDog;
import org.junit.jupiter.api.Test;
import org.redisson.Redisson;
import org.redisson.api.RLock;
import org.redisson.api.RedissonClient;
import org.redisson.config.Config;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

@SpringBootTest
public class TestRedissionLock {

    private int count = 0;
    @Autowired
    private RedissonClient redisson;

    @Test
    public void testLockWithDog() throws InterruptedException {
        int clientCount =3;
        RLock lock = redisson.getLock("RD-lock");
        CountDownLatch countDownLatch = new CountDownLatch(clientCount);
        ExecutorService executorService = Executors.newFixedThreadPool(clientCount);
        for (int i = 0;i<clientCount;i++){
            executorService.execute(() -> {
                try {
                    lock.lock(10, TimeUnit.SECONDS);
                    System.out.println(Thread.currentThread().getName()+"准备进行累加。");
                    Thread.sleep(2000);
                    count++;
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    lock.unlock();
                }
                countDownLatch.countDown();
            });
        }
        countDownLatch.await();
        System.out.println(count);
    }
}

```

#### 源码跟进

```
//根据这个key 的互斥条件 拿到锁
RLock rLock = redissonClient.getLock("keyA")  
    // 最多等待100秒、上锁10s以后解锁
if(rLock.tryLock(100,10, TimeUnit.SECONDS)){
    System.out.println("获取锁成功");
}    
//进入源码 tryLock  参数等大等待时间  锁多久 也就是锁的过期时间
public boolean tryLock(long waitTime, long leaseTime, TimeUnit unit) 

//进入看看过期时间是怎么 做的
    Long ttl = this.tryAcquire(leaseTime, unit, threadId);
        return (Long)this.get(this.tryAcquireAsync(leaseTime, unit, threadId));

        private <T> RFuture<Long> tryAcquireAsync(long leaseTime, TimeUnit unit, final long threadId) {
// 如果过期时间不等于 -1的话
if (leaseTime != -1L) {
    // 时间是多久就加锁多久
    return this.tryLockInnerAsync(leaseTime, unit, threadId, RedisCommands.EVAL_LONG);
    
    //进入tryLockInnerAsync  方法
        <T> RFuture<T> tryLockInnerAsync(long leaseTime, TimeUnit unit, long threadId, RedisStrictCommand<T> command) {
            this.internalLockLeaseTime = unit.toMillis(leaseTime);
            
            //通过eval 调用Lua脚本        
            return this.commandExecutor.evalWriteAsync(this.getName(), LongCodec.INSTANCE, command, 
            //判断这个大key是否存在   then 调用hset的存储结构设置参数2的值
            "if (redis.call('exists', KEYS[1]) == 0) then redis.call('hset', KEYS[1], ARGV[2], 1);
            //设置过期时间 并返回
             redis.call('pexpire', KEYS[1], ARGV[1]); return nil; end; 
             //判断是不是存在 大key的小key（也就是线程信息） 是否存在 ，存在 加1
             if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then redis.call('hincrby', KEYS[1], ARGV[2], 1);
             //设置过期时间 并返回
             redis.call('pexpire', KEYS[1], ARGV[1]); return nil; end; 
             //最后得到ttl 过期时间
             return redis.call('pttl', KEYS[1]);", 
             //1大key  2要释放的时间  3线程id
             Collections.singletonList(this.getName()), new Object[]{this.internalLockLeaseTime, this.getLockName(threadId)});
        }
        
        //那么这个 上面的this.internalLockLeaseTime释放时间默认是多久呢？           
        this.internalLockLeaseTime  点进去可见下面一个方法 RedissonLock来赋值的
        this.internalLockLeaseTime = commandExecutor.getConnectionManager().getCfg().getLockWatchdogTimeout();
        //点进       .getLockWatchdogTimeout(); 方法可见  这个参数是30000毫秒 也就是30秒
        private long lockWatchdogTimeout = 30000L;
        
       // 当然如果你设置了过期时间  加锁的方法跟刚才的是一模一样的。
         //但是 后面有多了一个 Listener  这个有是干嘛的呢？ 进入    scheduleExpirationRenewal(threadId);方法
         RedissonLock.this.scheduleExpirationRenewal(threadId);
         //从名字可知是一个定时任务 ，那多久执行一次呢？
        //这个参数我们刚看过 30秒 除3也就是10秒（10000毫秒）一次
        }, this.internalLockLeaseTime / 3L, TimeUnit.MILLISECONDS);
        
        //当然它去做什么事情呢？ 调用lua脚本
        RFuture<Boolean> future = RedissonLock.this.commandExecutor.evalWriteAsync(RedissonLock.this.getName(), LongCodec.INSTANCE, RedisCommands.EVAL_BOOLEAN, 
       //如果大key 存在    设置过期时间30s
         "if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then redis.call('pexpire', KEYS[1], ARGV[1]); return 1; end; return 0;",
         Collections.singletonList(RedissonLock.this.getName()), 
         new Object[]{RedissonLock.this.internalLockLeaseTime, RedissonLock.this.getLockName(threadId)});

```

 **总结**：        

 1.如果设置了过期时间   ,通过eval 调用Lua脚本.

​	判断大key是否存在 设置过期时间 ；判断线程信息是否存在  存在 加1 .只有-1的时候才会有俗话说的看门狗机制        

 2.每10s去执行lua脚本判断大key是否存在（是否还持有锁），如果存在重新设置时间为30s   





https://github.com/redisson/redisson/

[https://redisson.org/](https://redisson.org/)



## 集群下的锁还安全么？

基于 Redis 的实现分布式锁，前面遇到的问题，以及对应的解决方案：

1、死锁：设置过期时间

2、过期时间评估不好，锁提前过期：守护线程，自动续期

3、锁被别人释放：锁写入唯一标识，释放锁先检查标识，再释放

之前分析的场景都是，锁在「单个」Redis实例中可能产生的问题，并没有涉及到 Redis 的部署架构细节。

而我们在使用 Redis 时，一般会采用主从集群 +哨兵的模式部署，这样做的好处在于，当主库异常宕机时，哨兵可以实现「故障自动切换」，把从库提升为主库，继续提供服务，以此保证可用性。

但是因为主从复制是异步的，那么就不可避免会发生的锁数据丢失问题（**加了锁却没来得及同步过来**）。从库被哨兵提升为新主库，这个锁在新的主库上，丢失了！

## Redlock真的安全吗？

Redis 作者提出的 Redlock方案，是如何解决主从切换后，锁失效问题的。

**Redlock 的方案基于一个前提：**

不再需要部署从库和哨兵实例，只部署主库；但主库要部署多个，官方推荐至少 5 个实例。

**注意：不是部署 Redis Cluster，就是部署 5 个简单的 Redis 实例。它们之间没有任何关系，都是一个个孤立的实例。**

做完之后，我们看官网代码怎么去用的：

[8. 分布式锁和同步器 · redisson/redisson Wiki · GitHub](https://github.com/redisson/redisson/wiki/8.-%E5%88%86%E5%B8%83%E5%BC%8F%E9%94%81%E5%92%8C%E5%90%8C%E6%AD%A5%E5%99%A8#84-%E7%BA%A2%E9%94%81redlock)

**8.4. 红锁（RedLock）**

基于Redis的Redisson红锁 `RedissonRedLock`对象实现了[Redlock](http://redis.cn/topics/distlock.html)介绍的加锁算法。该对象也可以用来将多个 `RLock`对象关联为一个红锁，每个 `RLock`对象实例可以来自于不同的Redisson实例。

```java
RLock lock1 = redissonInstance1.getLock("lock1");
RLock lock2 = redissonInstance2.getLock("lock2");
RLock lock3 = redissonInstance3.getLock("lock3");

RedissonRedLock lock = new RedissonRedLock(lock1, lock2, lock3);
// 同时加锁：lock1 lock2 lock3
// 红锁在大部分节点上加锁成功就算成功。
lock.lock();
...
lock.unlock();
```

大家都知道，如果负责储存某些分布式锁的某些Redis节点宕机以后，而且这些锁正好处于锁住的状态时，这些锁会出现锁死的状态。为了避免这种情况的发生，Redisson内部提供了一个监控锁的看门狗，它的作用是在Redisson实例被关闭前，不断的延长锁的有效期。默认情况下，看门狗的检查锁的超时时间是30秒钟，也可以通过修改[Config.lockWatchdogTimeout](https://github.com/redisson/redisson/wiki/2.-%E9%85%8D%E7%BD%AE%E6%96%B9%E6%B3%95#lockwatchdogtimeout%E7%9B%91%E6%8E%A7%E9%94%81%E7%9A%84%E7%9C%8B%E9%97%A8%E7%8B%97%E8%B6%85%E6%97%B6%E5%8D%95%E4%BD%8D%E6%AF%AB%E7%A7%92)来另行指定。

另外Redisson还通过加锁的方法提供了 `leaseTime`的参数来指定加锁的时间。超过这个时间后锁便自动解开了。

```java
RedissonRedLock lock = new RedissonRedLock(lock1, lock2, lock3);
// 给lock1，lock2，lock3加锁，如果没有手动解开的话，10秒钟后将会自动解开
lock.lock(10, TimeUnit.SECONDS);

// 为加锁等待100秒时间，并在加锁成功10秒钟后自动解开
boolean res = lock.tryLock(100, 10, TimeUnit.SECONDS);
...
lock.unlock();
```

### Redlock实现整体流程

1、客户端先获取「当前时间戳T1」

2、客户端依次向这 5 个 Redis 实例发起加锁请求

3、如果客户端从 >=3 个（大多数）以上Redis 实例加锁成功，则再次获取「当前时间戳T2」，如果 T2 - T1 &#x3c; 锁的过期时间，此时，认为客户端加锁成功，否则认为加锁失败。

4、加锁成功，去操作共享资源

5、加锁失败/释放锁，向「全部节点」发起释放锁请求。

所以总的来说：客户端在多个 Redis 实例上申请加锁；必须保证大多数节点加锁成功；大多数节点加锁的总耗时，要小于锁设置的过期时间；释放锁，要向全部节点发起释放锁请求。

**我们来看 Redlock 为什么要这么做？**

1) **为什么要在多个实例上加锁？**

本质上是为了「容错」，部分实例异常宕机，剩余的实例加锁成功，整个锁服务依旧可用。

2) **为什么大多数加锁成功，才算成功？**

多个 Redis 实例一起来用，其实就组成了一个「分布式系统」。在分布式系统中，总会出现「异常节点」，所以，在谈论分布式系统问题时，需要考虑异常节点达到多少个，也依旧不会影响整个系统的「正确性」。

这是一个分布式系统「容错」问题，这个问题的结论是：如果只存在「故障」节点，只要大多数节点正常，那么整个系统依旧是可以提供正确服务的。

3) **为什么步骤 3 加锁成功后，还要计算加锁的累计耗时？**

因为操作的是多个节点，所以耗时肯定会比操作单个实例耗时更久，而且，因为是网络请求，网络情况是复杂的，有可能存在延迟、丢包、超时等情况发生，网络请求越多，异常发生的概率就越大。

所以，即使大多数节点加锁成功，但如果加锁的累计耗时已经「超过」了锁的过期时间，那此时有些实例上的锁可能已经失效了，这个锁就没有意义了。

4) **为什么释放锁，要操作所有节点？**

在某一个 Redis 节点加锁时，可能因为「网络原因」导致加锁失败。

例如，客户端在一个 Redis 实例上加锁成功，但在读取响应结果时，网络问题导致读取失败，那这把锁其实已经在 Redis 上加锁成功了。

所以，释放锁时，不管之前有没有加锁成功，需要释放「所有节点」的锁，以保证清理节点上「残留」的锁。

好了，明白了 Redlock 的流程和相关问题，看似Redlock 确实解决了 Redis 节点异常宕机锁失效的问题，保证了锁的「安全性」。

但事实真的如此吗？

### RedLock的是是非非

一个分布式系统，更像一个复杂的「野兽」，存在着你想不到的各种异常情况。

这些异常场景主要包括三大块，这也是分布式系统会遇到的三座大山：NPC。

N：Network Delay，网络延迟

P：Process Pause，进程暂停（GC）

C：Clock Drift，时钟漂移

比如一个进程暂停（GC）的例子

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1663570141014/d655198142af46659e861a38345613b1.png)

1）客户端 1 请求锁定节点 A、B、C、D、E

2）客户端 1 的拿到锁后，进入 GC（时间比较久）

3）所有 Redis 节点上的锁都过期了

4）客户端 2 获取到了 A、B、C、D、E 上的锁

5）客户端 1 GC 结束，认为成功获取锁

6）客户端 2 也认为获取到了锁，发生「冲突」

GC 和网络延迟问题：这两点可以在红锁实现流程的第3步来解决这个问题。

但是最核心的还是时钟漂移，因为时钟漂移，就有可能导致第3步的判断本身就是一个BUG，所以当多个 Redis 节点「时钟」发生问题时，也会导致 Redlock 锁失效。

## RedLock总结

Redlock 只有建立在「时钟正确」的前提下，才能正常工作，如果你可以保证这个前提，那么可以拿来使用。

但是时钟偏移在现实中是存在的：

第一，从硬件角度来说，时钟发生偏移是时有发生，无法避免。例如，CPU 温度、机器负载、芯片材料都是有可能导致时钟发生偏移的。

第二，人为错误也是很难完全避免的。

所以，Redlock尽量不用它，而且它的性能不如单机版 Redis，部署成本也高，优先考虑使用主从+ 哨兵的模式
实现分布式锁（只会有很小的记录发生主从切换时的锁丢失问题）。

# Redis高并发高可用

## 复制

在分布式系统中为了解决单点问题，通常会把数据复制多个副本部署到其他机器，满足故障恢复和负载均衡等需求。Redis也是如此，它为我们提供了复制功能，实现了相同数据的多个Redis 副本。复制功能是高可用Redis的基础，后面章节的哨兵和集群都是在复制的基础上实现高可用的。

默认情况下，Redis都是主节点。每个从节点只能有一个主节点，而主节点可以同时具有多个从节点。复制的数据流是单向的，只能由主节点复制到从节点。

### 复制的拓扑结构

Redis 的复制拓扑结构可以支持单层或多层复制关系，根据拓扑复杂性可以分为以下三种:一主一从、一主多从、树状主从结构,下面分别介绍。

#### 一主一从结构

一主一从结构是最简单的复制拓扑结构，用于主节点出现宕机时从节点提供故障转移支持。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666336676066/d634f64c7b6342c9b82d306ad4845e23.png)

当应用写命令并发量较高且需要持久化时,可以只在从节点上开启AOF ,这样既保证数据安全性同时也避免了持久化对主节点的性能干扰。但需要注意的是，当主节点关闭持久化功能时，如果主节点脱机要避免自动重启操作。

因为主节点之前没有开启持久化功能自动重启后数据集为空，这时从节点如果继续复制主节点会导致从节点数据也被清空的情况,丧失了持久化的意义。安全的做法是在从节点上执行slaveof no one断开与主节点的复制关系，再重启主节点从而避免这一问题。

#### 一主多从结构

一主多从结构(又称为星形拓扑结构）使得应用端可以利用多个从节点实现读写分离。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666336676066/b7dedbfd19d24db48e82c91719f59114.png)

对于读占比较大的场景，可以把读命令发送到从节点来分担主节点压力。同时在日常开发中如果需要执行一些比较耗时的读命令，如:keys、sort等，可以在其中一台从节点上执行，防止慢查询对主节点造成阻塞从而影响线上服务的稳定性。对于写并发量较高的场景,多个从节点会导致主节点写命令的多次发送从而过度消耗网络带宽，同时也加重了主节点的负载影响服务稳定性。

#### 树状主从结构

树状主从结构(又称为树状拓扑结构）使得从节点不但可以复制主节点数据，同时可以作为其他从节点的主节点继续向下层复制。通过引入复制中间层，可以有效降低主节点负载和需要传送给从节点的数据量。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666336676066/8f8edd042ff34531932e699be31bf9c5.png)

数据写入节点A后会同步到B和C节点,B节点再把数据同步到D和E节点,数据实现了一层一层的向下复制。当主节点需要挂载多个从节点时为了避免对主节点的性能干扰,可以采用树状主从结构降低主节点压力。

主从数据同步：

**![image-20231201181741360](E:\图灵课堂\Redis专题\笔记\1、Redis专题.assets\image-20231201181741360.png)**

### 复制的配置

#### 建立复制

参与复制的Redis实例划分为主节点(master)和从节点(slave)。默认情况下，Redis都是主节点。每个从节点只能有一个主节点，而主节点可以同时具有多个从节点。复制的数据流是单向的，只能由主节点复制到从节点。

**配置复制的方式有以下三种**

1)在配置文件中加入slaveof{masterHost } {masterPort}随 Redis启动生效。

2)在redis-server启动命令后加入--slaveof{masterHost} {masterPort }生效。

3）直接使用命令:slaveof {masterHost} { masterPort}生效。

综上所述，slaveof命令在使用时，可以运行期动态配置,也可以提前写到配置文件中。

比如：我在机器上启动2台Redis, 分别是6379 和6380 两个端口。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666336676066/ce991b66b69947159848f8cc047abed2.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666336676066/58e7f3a1d88243bda30e9206226f70f6.png)

slaveof本身是异步命令，执行slaveof命令时，节点只保存主节点信息后返回，后续复制流程在节点内部异步执行,具体细节见之后。主从节点复制成功建立后,可以使用info replication命令查看复制相关状态。

#### 断开复制

slaveof命令不但可以建立复制，还可以在从节点执行slaveof no one来断开与主节点复制关系。例如在6881节点上执行slaveof no one来断开复制。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666336676066/bf21031a0cf243e68ec0d9c722c9855c.png)

slaveof本身是异步命令，执行slaveof命令时，节点只保存主节点信息后返回，后续复制流程在节点内部异步执行,具体细节见之后。主从节点复制成功建立后,可以使用info replication命令查看复制相关状态。

**断开复制主要流程:**

1）断开与主节点复制关系。2)从节点晋升为主节点。

从节点断开复制后并不会抛弃原有数据，只是无法再获取主节点上的数据变化。

通过slaveof命令还可以实现切主操作，所谓切主是指把当前从节点对主节点的复制切换到另一个主节点。

执行slaveof{ newMasterIp} { newMasterPort}命令即可，例如把6881节点从原来的复制6880节点变为复制6879节点。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666336676066/e904361290e54bb98ee01b66dd246dcb.png)

切主内部流程如下:

1)断开与旧主节点复制关系。

2）与新主节点建立复制关系。

3）删除从节点当前所有数据。

4）对新主节点进行复制操作。

#### 只读

默认情况下，从节点使用slave-read-only=yes配置为只读模式。由于复制只能从主节点到从节点，对于从节点的任何修改主节点都无法感知，修改从节点会造成主从数据不一致。因此建议线上不要修改从节点的只读模式。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666336676066/aec38b7f9d8545cbbdc85bea9183fae0.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666336676066/92dbcb842b794158b611f085bd61211b.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666336676066/2b0c0be29d5c44dcb1d2c3ffd1c05866.png)

#### 传输延迟

主从节点一般部署在不同机器上，复制时的网络延迟就成为需要考虑的问题,Redis为我们提供了repl-disable-tcp-nodelay参数用于控制是否关闭TCP_NODELAY，默认关闭，说明如下:

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666336676066/fea28b954f964c35ad77dde58ea34020.png)

当关闭时，主节点产生的命令数据无论大小都会及时地发送给从节点，这样主从之间延迟会变小，但增加了网络带宽的消耗。适用于主从之间的网络环境良好的场景，如同机架或同机房部署。

当开启时，主节点会合并较小的TCP数据包从而节省带宽。默认发送时间间隔取决于Linux的内核，一般默认为40毫秒。这种配置节省了带宽但增大主从之间的延迟。适用于主从网络环境复杂或带宽紧张的场景,如跨机房部署。

### Redis主从复制原理

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666336676066/9c23120f28e74d54bed1bc455d78e471.png)

在从节点执行slaveof命令后，复制过程便开始运作。

#### 1）保存主节点信息

执行slaveof后从节点只保存主节点的地址信息便直接返回，这时建立复制流程还没有开始。

#### 2）建立主从socket连接

从节点(slave)内部通过每秒运行的定时任务维护复制相关逻辑，当定时任务发现存在新的主节点后，会尝试与该节点建立网络连接。

从节点会建立一个socket套接字，专门用于接受主节点发送的复制命令。从节点连接成功后打印日志。

如果从节点无法建立连接，定时任务会无限重试直到连接成功或者执行slaveof no one取消复制。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666336676066/9a4e0f3db914446399ed2be204217411.png)

#### 3）发送ping命令

连接建立成功后从节点发送ping请求进行首次通信，ping请求主要目的：检测主从之间网络套接字是否可用、检测主节点当前是否可接受处理命令。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666336676066/d15620425e52488eb3f6143565229dcf.png)

从节点发送的ping命令成功返回，Redis打印日志，并继续后续复制流程:

#### 4）权限验证

如果主节点设置了requirepass参数，则需要密码验证，从节点必须配置masterauth参数保证与主节点相同的密码才能通过验证;如果验证失败复制将终止，从节点重新发起复制流程。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666336676066/46f6e8b768b14506b81d424d61f956d2.png)

#### 5) 同步数据集

主从复制连接正常通信后，对于首次建立复制的场景,主节点会把持有的数据全部发送给从节点，这部分操作是耗时最长的步骤。Redis在2.8版本以后采用新复制命令 psync进行数据同步，原来的sync命令依然支持，保证新旧版本的兼容性。新版同步划分两种情况:全量同步和部分同步。

#### 6) 命令持续复制

当主节点把当前的数据同步给从节点后，便完成了复制的建立流程。接下来主节点会持续地把写命令发送给从节点,保证主从数据一致性。

### Redis数据同步

Redis早期支持的复制功能只有全量复制（sync命令），它会把主节点全部数据一次性发送给从节点，当数据量较大时，会对主从节点和网络造成很大的开销。

Redis在2.8版本以后采用新复制命令psync进行数据同步，原来的sync命令依然支持，保证新旧版本的兼容性。新版同步划分两种情况:全量复制和部分复制。

#### 全量同步

全量复制:一般用于初次复制场景，Redis早期支持的复制功能只有全量复制，它会把主节点全部数据一次性发送给从节点，当数据量较大时，会对主从节点和网络造成很大的开销。

全量复制是Redis最早支持的复制方式,也是主从第一次建立复制时必须经历的阶段。触发全量复制的命令是sync和psync。

psync全量复制流程,它与2.8以前的sync全量复制机制基本一致。

##### **流程说明**

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666336676066/23ebbf1f77714febb3a5fbcaf9b1ca11.png)

1)发送psync命令进行数据同步，由于是第一次进行复制，从节点没有复制偏移量和主节点的运行ID,所以发送psync ? -1。

2）主节点根据psync ? -1解析出当前为全量复制，回复 +FULLRESYNC响应，从节点接收主节点的响应数据保存运行ID和偏移量offset，并打印日志。

3）主节点执行bgsave保存RDB 文件到本地。

4）主节点发送RDB文件给从节点，从节点把接收的RDB文件保存在本地并直接作为从节点的数据文件,接收完RDB后从节点打印相关日志，可以在日志中查看主节点发送的数据量。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666336676066/17d37667b1854aeb92371a1ee5a4e600.png)

5）对于从节点开始接收RDB快照到接收完成期间，主节点仍然响应读写命令，因此主节点会把这期间写命令数据保存在复制客户端缓冲区内，当从节点加载完RDB文件后，主节点再把缓冲区内的数据发送给从节点,保证主从之间数据一致性。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666336676066/a8315cedc2d54bf2a624c0cbb5296bc4.png)

需要注意,对于数据量较大的主节点,比如生成的RDB文件超过6GB 以上时要格外小心。传输文件这一步操作非常耗时，速度取决于主从节点之间网络带宽

##### 问题

通过分析全量复制的所有流程,会发现全量复制是一个非常耗时费力的操作。它的时间开销主要包括:

1、主节点bgsave时间。

2、RDB文件网络传输时间。

3、从节点清空数据时间。

4、从节点加载RDB的时间。

5、可能的AOF重写时间。

因此当数据量达到一定规模之后，由于全量复制过程中将进行多次持久化相关操作和网络数据传输，这期间会大量消耗主从节点所在服务器的CPU、内存和网络资源。

**另外最大的问题，复制还会失败！！！**

例如我们线上数据量在6G左右的主节点，从节点发起全量复制的总耗时在2分钟左右。

1、如果总时间超过repl-timeout所配置的值（默认60秒)，从节点将放弃接受RDB文件并清理已经下载的临时文件，导致全量复制失败。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665215626018/08db81e074244dd6b7a7e27c54625f90.png)

2、如果主节点创建和传输RDB的时间过长，对于高流量写入场景非常容易造成主节点复制客户端缓冲区溢出。默认配置为

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665215626018/99b4f1f238014df0ad4c29e3fac5a97a.png)

意思是如果60秒内缓冲区消耗持续大于64MB或者直接超过256MB时，主节点将直接关闭复制客户端连接，造成全量同步失败。

所以除了第一次复制时采用全量复制在所难免之外，对于其他场景应该规避全量复制的发生。正因为全量复制的成本问题。

#### 部分同步

部分复制主要是Redis针对全量复制的过高开销做出的一种优化措施。

使用psync  {runId}   {offset}  命令实现

当从节点(slave)正在复制主节点（master)时，如果出现网络闪断或者命令丢失等异常情况时，从节点会向主节点要求补发丢失的命令数据，如果主节点的复制积压缓冲区内存在这部分数据则直接发送给从节点，这样就可以保持主从节点复制的一致性。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666336676066/d284a4e590e5419cb092771ea302b10f.png)

##### 流程说明

1)当主从节点之间网络出现中断时，如果超过repl-timeout时间，主节点会认为从节点故障并中断复制连接,打印日志。如果此时从节点没有宕机，也会打印与主节点连接丢失日志。

2）主从连接中断期间主节点依然响应命令，但因复制连接中断命令无法发送给从节点,不过主节点内部存在的复制积压缓冲区，依然可以保存最近一段时间的写命令数据，默认最大缓存1MB。

3)当主从节点网络恢复后,从节点会再次连上主节点,打印日志。

4）当主从连接恢复后，由于从节点之前保存了自身已复制的偏移量和主节点的运行ID。因此会把它们当作psync参数发送给主节点，要求进行部分复制操作。

5)主节点接到psync命令后首先核对参数runId是否与自身一致,如果一致，说明之前复制的是当前主节点;之后根据参数offset在自身复制积压缓冲区查找，如果偏移量之后的数据存在缓冲区中，则对从节点发送+CONTINUE响应，表示可以进行部分复制。如果不再，则退化为全量复制。

6）主节点根据偏移量把复制积压缓冲区里的数据发送给从节点，保证主从复制进入正常状态。发送的数据量可以在主节点的日志，传递的数据远远小于全量数据。

#### 心跳

主从节点在建立复制后,它们之间维护着长连接并彼此发送心跳命令。

主从心跳判断机制:

1)主从节点彼此都有心跳检测机制，各自模拟成对方的客户端进行通信,通过client list命令查看复制相关客户端信息，主节点的连接状态为flags=M，从节点连接状态为flags=S。

2）主节点默认每隔10秒对从节点发送ping命令，判断从节点的存活性和连接状态。

可通过参数repl-ping-slave-period控制发送频率。

3)从节点在主线程中每隔1秒发送replconf ack {offset}命令，给主节点上报自身当前的复制偏移量。replconf命令主要作用如下:

实时监测主从节点网络状态；

上报自身复制偏移量,检查复制数据是否丢失,如果从节点数据丢失，再从主节点的复制缓冲区中拉取丢失数据

实现保证从节点的数量和延迟性功能，通过min-slaves-to-write、min-slaves-max-lag参数配置定义；

主节点根据replconf命令判断从节点超时时间，体现在info replication统计中的lag信息中，lag表示与从节点最后一次通信延迟的秒数，正常延迟应该在0和1之间。如果超过repl-timeout配置的值(（默认60秒)，则判定从节点下线并断开复制客户端连接。即使主节点判定从节点下线后,如果从节点重新恢复，心跳检测会继续进行。

#### 异步复制机制

主节点不但负责数据读写，还负责把写命令同步给从节点。写命令的发送过程是异步完成,也就是说主节点自身处理完写命令后直接返回给客户端,并不等待从节点复制完成。

由于主从复制过程是异步的，就会造成从节点的数据相对主节点存在延迟。具体延迟多少字节,我们可以在主节点执行info replication命令查看相关指标获得。

在统计信息中可以看到从节点slave信息，分别记录了从节点的ip和 port，从节点的状态，offset表示当前从节点的复制偏移量，master_repl_offset表示当前主节点的复制偏移量，两者的差值就是当前从节点复制延迟量。Redis 的复制速度取决于主从之间网络环境，repl-disable-tcp-nodelay，命令处理速度等。正常情况下，延迟在1秒以内。

## 哨兵Redis Sentinel

Redis 的主从复制模式下，一旦主节点由于故障不能提供服务，需要人工将从节点晋升为主节点，同时还要通知应用方更新主节点地址，对于很多应用场景这种故障处理的方式是无法接受的。

Redis 从 2.8开始正式提供了Redis Sentinel(哨兵）架构来解决这个问题。

### 主从复制的问题

Redis 的主从复制模式可以将主节点的数据改变同步给从节点，这样从节点就可以起到两个作用

第一，作为主节点的一个备份，一旦主节点出了故障不可达的情况，从节点可以作为后备“顶”上来，并且保证数据尽量不丢失(主从复制是最终一致性)。

第二，从节点可以扩展主节点的读能力，一旦主节点不能支撑住大并发量的读操作，从节点可以在一定程度上帮助主节点分担读压力。

但是主从复制也带来了以下问题:

1、一旦主节点出现故障，需要手动将一个从节点晋升为主节点，同时需要修改应用方的主节点地址，还需要命令其他从节点去复制新的主节点,整个过程都需要人工干预。

2、主节点的写能力受到单机的限制。

3、主节点的存储能力受到单机的限制。

### Redis Sentinel

Redis Sentinel是一个分布式架构，其中包含若干个Sentinel节点和Redis数据节点，每个Sentinel节点会对数据节点和其余Sentinel节点进行监控，当它发现节点不可达时，会对节点做下线标识。如果被标识的是主节点，它还会和其他Sentinel节点进行“协商”，当大多数Sentinel节点都认为主节点不可达时，它们会选举出一个Sentinel节点来完成自动故障转移的工作，同时会将这个变化实时通知给Redis应用方。整个过程完全是自动的，不需要人工来介入，所以这套方案很有效地解决了Redis的高可用问题。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666336676066/0bdb4b558f974743ba1172af64f81dd9.png)

### Redis Sentinel的搭建

我们以以3个 Sentinel节点、1个主节点、2个从节点组成一个Redis Sentinel进行说明。

启动主从的方式和普通的主从没有不同。

#### 启动Sentinel节点

Sentinel节点的启动方法有两种:

方法一,使用redis-sentinel命令:

```
./redis-sentinel   ../conf/reids.conf
```

方法二，使用redis-server命令加--sentinel参数:

```
./redis-server ../conf/reids.conf  --sentinel
```

两种方法本质上是—样的。

##### 确认

Sentinel节点本质上是一个特殊的Redis节点，所以也可以通过info命令来查询它的相关信息

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666336676066/a0ee43231155484a9ccf54dc5edf3ceb.png)

### 实现原理

Redis Sentinel的基本实现中包含以下：
Redis Sentinel 的定时任务、主观下线和客观下线、Sentinel领导者选举、故障转移等等知识点，学习这些可以让我们对Redis Sentinel的高可用特性有更加深入的理解和认识。

示意图：





#### 三个定时监控任务

一套合理的监控机制是Sentinel节点判定节点不可达的重要保证，Redis Sentinel通过三个定时监控任务完成对各个节点发现和监控：

##### 1、每隔10秒的定时监控

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666336676066/8a1dfef71fe74479a2b4a4998255652c.png)

每隔10秒，每个Sentinel节点会向主节点和从节点发送info命令获取最新的拓扑结构，Sentinel节点通过对上述结果进行解析就可以找到相应的从节点。

这个定时任务的作用具体可以表现在三个方面:

1、通过向主节点执行info命令，获取从节点的信息,这也是为什么Sentinel节点不需要显式配置监控从节点。

2、当有新的从节点加入时都可以立刻感知出来。

3、节点不可达或者故障转移后，可以通过info命令实时更新节点拓扑信息。

##### 2、每隔2秒的定时监控

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666336676066/fddd1423ec6e4c22978546cd2e428a8c.png)

每隔2秒,每个Sentinel节点会向Redis数据节点的_sentinel_:hello频道上发送该Sentinel节点对于主节点的判断以及当前Sentinel节点的信息，同时每个Sentinel节点也会订阅该频道，来了解其他Sentinel节点以及它们对主节点的判断,所以这个定时任务可以完成以下两个工作:

发现新的Sentinel节点:通过订阅主节点的__sentinel__:hello了解其他的Sentinel节点信息，如果是新加入的Sentinel节点，将该Sentinel节点信息保存起来,并与该 Sentinel节点创建连接。

Sentinel节点之间交换主节点的状态，作为后面客观下线以及领导者选举的依据。

##### 3、每隔1秒的定时监控

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666336676066/905d356c71714e7084583296eeac1350.png)

每隔1秒，每个Sentinel节点会向主节点、从节点、其余Sentinel节点发送一条ping命令做一次心跳检测，来确认这些节点当前是否可达。

通过上面的定时任务，Sentinel节点对主节点、从节点、其余Sentinel节点都建立起连接，实现了对每个节点的监控,这个定时任务是节点失败判定的重要依据。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666336676066/9bb97de253d24fa69d4035942175c872.png)

#### 主观下线和客观下线

##### 主观下线

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666336676066/a17f946612e24d7f8e95c42f9a16e855.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666336676066/ac050de6f3874fbb9b1e79503d69b713.png)

上一小节介绍的第三个定时任务，每个Sentinel节点会每隔1秒对主节点、从节点、其他Sentinel节点发送ping命令做心跳检测,当这些节点超过down-after-milliseconds没有进行有效回复，Sentinel节点就会对该节点做失败判定，这个行为叫做主观下线。从字面意思也可以很容易看出主观下线是当前Sentinel节点的一家之言,存在误判的可能。

##### 客观下线

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666336676066/238a751dc2f84431b84b78584a2fcdcd.png)

当Sentinel主观下线的节点是主节点时，该Sentinel节点会通过sentinel is-master-down-by-addr命令向其他Sentinel节点询问对主节点的判断，当超过&#x3c;quorum>个数,Sentinel节点认为主节点确实有问题，这时该Sentinel节点会做出客观下线的决定，这样客观下线的含义是比较明显了，也就是大部分Sentinel节点都对主节点的下线做了同意的判定，那么这个判定就是客观的。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665215626018/78caadf29ad3402fae0d7d52dc49cb7e.png)

##### 领导者Sentinel节点选举

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666336676066/4708aea0fae3469dae8bec534554749a.png)

假如Sentinel节点对于主节点已经做了客观下线，那么是不是就可以立即进行故障转移了？当然不是，实际上故障转移的工作只需要一个Sentinel节点来完成即可，所以 Sentinel节点之间会做一个领导者选举的工作，选出一个Sentinel节点作为领导者进行故障转移的工作。Redis使用了Raft算法实现领导者选举，Redis Sentinel进行领导者选举的大致思路如下:

1 )每个在线的Sentinel节点都有资格成为领导者，当它确认主节点主观下线时候，会向其他Sentinel节点发送sentinel is-master-down-by-addr命令，要求将自己设置为领导者。

2)收到命令的Sentinel节点，如果没有同意过其他Sentinel节点的sentinel is-master-down-by-addr命令,将同意该请求,否则拒绝。

3）如果该Sentinel节点发现自己的票数已经大于等于max (quorum，num(sentinels)/2+1）,那么它将成为领导者。

4）如果此过程没有选举出领导者,将进入下一次选举。

选举的过程非常快,基本上谁先完成客观下线,谁就是领导者。

Raft协议的详细版本：

[raft-zh_cn/raft-zh_cn.md at master · maemual/raft-zh_cn · GitHub](https://github.com/maemual/raft-zh_cn/blob/master/raft-zh_cn.md)

如果你想手写一个Raft协议，可以看下蚂蚁金服的开发生产的raft算法组件

[GitHub - sofastack/sofa-jraft: A production-grade java implementation of RAFT consensus algorithm.](https://github.com/sofastack/sofa-jraft)

选举很快的！！

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1665215626018/5191985ef85f49c9a8efd144f5272b1a.png)

#### 故障转移

领导者选举出的Sentinel节点负责故障转移，具体步骤如下:

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666336676066/a8b8e35e659b4521b7571b5395b0808d.png)

1)在从节点列表中选出一个节点作为新的主节点,选择方法如下:

```
a)过滤:“不健康”(主观下线、断线)、5秒内没有回复过Sentinel节点 ping响应、与主节点失联超过down-after-milliseconds*10秒。
```

```
b)选择slave-priority(从节点优先级)最高的从节点列表，如果存在则返回,不存在则继续。
```

```
c）选择复制偏移量最大的从节点(复制的最完整)，如果存在则返回,不存在则继续。
```

```
d）选择runid最小的从节点。
```

2 ) Sentinel领导者节点会对第一步选出来的从节点执行slaveof no one命令让其成为主节点。

3 ) Sentinel领导者节点会向剩余的从节点发送命令，让它们成为新主节点的从节点,复制规则和parallel-syncs参数有关。

4 ) Sentinel节点集合会将原来的主节点更新为从节点，并保持着对其关注，当其恢复后命令它去复制新的主节点。

##### 脑裂的问题

单独的服务  去监听所有的redis 服务，如果发现主挂了，会选择一个从节点升级为主节点。

监控主节点 判断主节点状态， 网络问题的时候，以为主挂了，会去选举一个从升级为主。

当和原本主恢复了通信需要 回归原本的主，还得同步替补主的数据，先清除自己的数据，然后从替补拿rdb文件加载。  

此时有些数据 可能并没有写入RDB，所以

这个称之为脑裂   也可能会导致数据丢失！

此哨兵认为它挂了，主观让其下线（此时并不会真正下线）。 

 还需要去询问其他哨兵大多数认为  被观下线 才会真正下线。

（所以 配置哨兵个数用奇数还是偶数？  奇）

动态演示站点：http://thesecretlivesofdata.com/raft/

​    raft算法 节点状态  选举过程  分布式一致性   

详细的投票过程    谁快谁为王！

##### 脑裂应对方案：

问题出在原主假故障后，仍能接收请求，因此，可在主从集群机制的配置项中查找是否有限制主库接收请求的设置。Redis提供如下配置项限制主库的请求处理：

min-replicas-to-write 主库能进行数据同步的最少从库数量

min-replicas-max-lag 主从库间进行数据复制时,从库给主库发送**ACK**消息的最大延迟(单位s）

分别设置阈值N和T，俩配置项组合后的要求是：

主库连接的从库中至少有N个从库

和主库进行数据复制时的ACK消息延迟不能超过T秒

否则，主库就不会再接收客户端请求。

即使原主假故障，假故障期间也无法响应哨兵心跳，也不能和从库进行同步，自然就无法和从库进行ACK确认。这俩配置项组合要求就无法得到满足，原主库就会被限制接收客户端请求，客户端也就不能在原主库中写新数据。

等新主上线，就只有新主能接收和处理客户端请求，此时，新写的数据会被直接写到新主。而原主会被哨兵降为从库，即使它的数据被清空，也不会有新数据的丢失。





### Redis Sentinel的客户端

如果主节点挂掉了，虽然Redis Sentinel可以完成故障转移，但是客户端无法获取这个变化，那么使用Redis Sentinel的意义就不大了，所以各个语言的客户端需要对Redis Sentinel进行显式的支持。

Sentinel节点集合具备了监控、通知、自动故障转移、配置提供者若干功能，也就是说实际上最了解主节点信息的就是Sentinel节点集合，而各个主节点可以通过&#x3c;host-name>进行标识的，所以，无论是哪种编程语言的客户端，如果需要正确地连接Redis Sentinel，必须有Sentinel节点集合和masterName两个参数。

我们依然使用Jedis 作为Redis 的 Java客户端，Jedis能够很好地支持Redis
Sentinel，并且使用Jedis连接Redis Sentinel也很简单，按照Redis Sentinel的原理，需要有masterName和Sentinel节点集合两个参数。Jedis针对Redis Sentinel给出了一个 JedisSentinelPool。

具体代码可以参见redis-sentinel：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666336676066/37b4bfd4f0cc4ba8b2484f5be610ff5e.png)

实现一个Redis Sentinel客户端一般来说需要:

1）遍历Sentinel节点集合获取一个可用的Sentinel节点，Sentinel节点之间可以共享数据，所以从任意一个Sentinel节点获取主节点信息都是可以的。

2)通过sentinel get-master-addr-by-name host-name这个API来获取对应主节点的相关信息。

3）验证当前获取的“主节点”是真正的主节点，这样做的目的是为了防止故障转移期间主节点的变化。

4）保持和 Sentinel节点集合的“联系”，时刻获取关于主节点的相关“信息”。

但是注意，JedisSentinel的实现是不支持读写分离的，所有的连接都是连接到Master上面，Slave就完全当成Master的备份，存在着性能浪费。因此如果想支持读写分离，需要自行实现，这里给一个参考

[基于Spring 的 Redis Sentinel 读写分离 Slave 连接池 (jack-yin.com)](https://www.jack-yin.com/coding/spring-boot/2683.html)

### 高可用读写分离

#### 从节点的作用

第一，当主节点出现故障时，作为主节点的后备“顶”上来实现故障转移，Redis Sentinel已经实现了该功能的自动化,实现了真正的高可用。

第二，扩展主节点的读能力，尤其是在读多写少的场景非常适用。

但上述模型中，从节点不是高可用的:

如果slave-1节点出现故障，首先客户端client-1将与其失联，其次Sentinel节点只会对该节点做主观下线，因为Redis Sentinel的故障转移是针对主节点的。所以很多时候，Redis Sentinel中的从节点仅仅是作为主节点一个热备，不让它参与客户端的读操作，就是为了保证整体高可用性，但实际上这种使用方法还是有一些浪费，尤其是在有很多从节点或者确实需要读写分离的场景，所以如何实现从节点的高可用是非常有必要的。

#### Redis Sentinel读写分离设计思路参考

Redis Sentinel在对各个节点的监控中，如果有对应事件的发生，都会发出相应的事件消息，其中和从节点变动的事件有以下几个:

**+switch-master**

切换主节点(原来的从节点晋升为主节点)，说明减少了某个从节点。

**+convert-to-slave**
切换从节点(原来的主节点降级为从节点)，说明添加了某个从节点。

**+sdown**

主观下线，说明可能某个从节点可能不可用(因为对从节点不会做客观下线)，所以在实现客户端时可以采用自身策略来实现类似主观下线的功能。

**+reboot**

重新启动了某个节点,如果它的角色是slave，那么说明添加了某个从节点。

所以在设计Redis Sentinel的从节点高可用时，只要能够实时掌握所有从节点的状态,把所有从节点看做一个资源池，无论是上线还是下线从节点，客户端都能及时感知到(将其从资源池中添加或者删除)，这样从节点的高可用目标就达到了。

## Redis集群

Redis Cluster是Redis的分布式解决方案，在3.0版本正式推出，有效地解决了Redis分布式方面的需求。当遇到单机内存、并发、流量等瓶颈时，可以采用Cluster架构方案达到负载均衡的目的。之前,Redis分布式方案一般有两种:

1、客户端分区方案，优点是分区逻辑可控，缺点是需要自己处理数据路由、高可用、故障转移等问题。

2、代理方案，优点是简化客户端分布式逻辑和升级维护便利,缺点是加重架构部署复杂度和性能损耗。

现在官方为我们提供了专有的集群方案:Redis Cluster，它非常优雅地解决了Redis集群方面的问题，因此理解应用好 Redis Cluster将极大地解放我们使用分布式Redis 的工作量。

### 集群前置知识

#### 数据分布理论

分布式数据库首先要解决把整个数据集按照分区规则映射到多个节点的问题，即把数据集划分到多个节点上，每个节点负责整体数据的一个子集。。

需要重点关注的是数据分区规则。

常见的分区规则有哈希分区和顺序分区两种，哈希分区离散度好、数据分布业务无关、无法顺序访问，顺序分区离散度易倾斜、数据分布业务相关、可顺序访问。

##### 节点取余分区

使用特定的数据，如Redis的键或用户ID，再根据节点数量N使用公式:
hash(key)%N计算出哈希值，用来决定数据映射到哪一个节点上。这种方案存在一个问题:当节点数量变化时，如扩容或收缩节点，数据节点映射关系需要重新计算，会导致数据的重新迁移。

这种方式的突出优点是简单性，常用于数据库的分库分表规则，一般采用预分区的方式，提前根据数据量规划好分区数，比如划分为512或1024张表，保证可支撑未来一段时间的数据量,再根据负载情况将表迁移到其他数据库中。扩容时通常采用翻倍扩容，避免数据映射全部被打乱导致全量迁移的情况,如图10-2所示。

##### 一致性哈希分区

一致性哈希分区（ Distributed Hash Table)实现思路是为系统中每个节点分配一个 token,范围一般在0~23，这些token构成一个哈希环。数据读写执行节点查找操作时，先根据key计算hash值，然后顺时针找到第一个大于等于该哈希值的token节点。例如：

集群中有三个节点（Node1、Node2、Node3），五个键（key1、key2、key3、key4、key5），其路由规则为：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666925611079/bab557dc5d9c4932bc2682481a50e8b7.png)

当集群中增加节点时，比如当在Node2和Node3之间增加了一个节点Node4，此时再访问节点key4时，不能在Node4中命中，更一般的，介于Node2和Node4之间的key均失效，这样的失效方式太过于“集中”和“暴力”，更好的方式应该是“平滑”和“分散”地失效。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666925611079/9de42d7a40a94ca5ba61daff1d0cd894.png)

这种方式相比节点取余最大的好处在于加入和删除节点只影响哈希环中相邻的节点，对其他节点无影响。但一致性哈希分区存在几个问题:

1、当使用少量节点时，节点变化将大范围影响哈希环中数据映射，因此这种方式不适合少量数据节点的分布式方案。

2、增加节点只能对下一个相邻节点有比较好的负载分担效果，例如上图中增加了节点Node4只能够对Node3分担部分负载，对集群中其他的节点基本没有起到负载分担的效果；类似地，删除节点会导致下一个相邻节点负载增加，而其他节点却不能有效分担负载压力。

正因为一致性哈希分区的这些缺点，一些分布式系统采用虚拟槽对一致性哈希进行改进，比如虚拟一致性哈希分区。

##### 虚拟一致性哈希分区

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666925611079/70c7b32ede304da4bf8498c39e836bc1.png)

为了在增删节点的时候，各节点能够保持动态的均衡，将每个真实节点虚拟出若干个虚拟节点，再将这些虚拟节点随机映射到环上。此时每个真实节点不再映射到环上，真实节点只是用来存储键值对，它负责接应各自的一组环上虚拟节点。当对键值对进行存取路由时，首先路由到虚拟节点上，再由虚拟节点找到真实的节点。

如下图所示，三个节点真实节点：Node1、Node2和Node3，每个真实节点虚拟出三个虚拟节点：X#V1、X#V2和X#V3，这样每个真实节点所负责的hash空间不再是连续的一段，而是分散在环上的各处，这样就可以将局部的压力均衡到不同的节点，虚拟节点越多，分散性越好，理论上负载就越倾向均匀。

##### 虚拟槽分区

Redis则是利用了虚拟槽分区，可以算上面虚拟一致性哈希分区的变种，它使用分散度良好的哈希函数把所有数据映射到一个固定范围的整数集合中，整数定义为槽( slot)。这个范围一般远远大于节点数，比如RedisCluster槽范围是0 ～16383。槽是集群内数据管理和迁移的基本单位。采用大范围槽的主要目的是为了方便数据拆分和集群扩展。每个节点会负责一定数量的槽。

比如集群有3个节点，则每个节点平均大约负责5460个槽。由于采用高质量的哈希算法，每个槽所映射的数据通常比较均匀，将数据平均划分到5个节点进行数据分区。Redis Cluster就是采用虚拟槽分区,下面就介绍Redis 数据分区方法。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666925611079/cbd9c96c04f84d9795c24c5f76cf5234.png)

##### 为什么槽的范围是0 ～16383？

为什么槽的范围是0 ～16383，也就是说槽的个数在16384个？redis的作者在github上有个回答：[https://github.com/redis/redis/issues/2576](https://github.com/redis/redis/issues/2576)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666925611079/1d9939c506b34a9aa5cd004fb83c34c7.png)

这个意思是：

Redis集群中，在握手成功后，连个节点之间会定期发送ping/pong消息，交换数据信息，集群中节点数量越多，消息体内容越大，比如说10个节点的状态信息约1kb，同时redis集群内节点，每秒都在发ping消息。例如，一个总节点数为200的Redis集群，默认情况下，这时ping/pong消息占用带宽达到25M。

那么如果槽位为65536，发送心跳信息的消息头达8k，发送的心跳包过于庞大，非常浪费带宽。

其次redis的集群主节点数量基本不可能超过1000个。集群节点越多，心跳包的消息体内携带的数据越多。如果节点过1000个，也会导致网络拥堵。因此redis作者，不建议redis cluster节点数量超过1000个。

那么，对于节点数在1000以内的redis cluster集群，16384个槽位够用了，可以以确保每个 master 有足够的插槽，没有必要拓展到65536个。

再者Redis主节点的配置信息中，它所负责的哈希槽是通过一张bitmap的形式来保存的，在传输过程中，会对bitmap进行压缩，但是如果bitmap的填充率slots / N很高的话(N表示节点数)，也就是节点数很少，而哈希槽数量很多的话，bitmap的压缩率就很低，也会浪费资源。

所以Redis作者决定取16384个槽，作为一个比较好的设计权衡。

#### Redis数据分区

Redis Cluser采用虚拟槽分区，所有的键根据哈希函数映射到0 ~16383整数槽内，计算公式:slot=CRC16(key) &16383。每一个节点负责维护―部分槽以及槽所映射的键值数据。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666925611079/d8360eb94ffc458294f0f003a7f94380.png)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666925611079/a3a1240b7e74478abba33cd4a6e5821d.png)

##### Redis 虚拟槽分区的特点

1、解耦数据和节点之间的关系,简化了节点扩容和收缩难度。

2、节点自身维护槽的映射关系，不需要客户端或者代理服务维护槽分区元数据。口支持节点、槽、键之间的映射查询,用于数据路由、在线伸缩等场景。

3、数据分区是分布式存储的核心，理解和灵活运用数据分区规则对于掌握Redis Cluster非常有帮助。

hash槽示意图：

![image-20231201174917582](E:\图灵课堂\Redis专题\笔记\1、Redis专题.assets\image-20231201174917582.png)





##### 集群功能限制

Redis集群相对单机在功能上存在一些限制，需要开发人员提前了解，在使用时做好规避。限制如下:

1、 key批量操作支持有限。如mset、mget，目前只支持具有相同slot值的key执行批量操作。对于映射为不同slot值的key由于执行mget、mget等操作可能存在于多个节点上因此不被支持。

2、key事务操作支持有限。同理只支持多key在同一节点上的事务操作，当多个key分布在不同的节点上时无法使用事务功能。

3、key作为数据分区的最小粒度，因此不能将一个大的键值对象如hash、list等映射到不同的节点。

4、不支持多数据库空间。单机下的Redis可以支持16个数据库，集群模式下只能使用一个数据库空间,即 db 0。

5、复制结构只支持一层，从节点只能复制主节点，不支持嵌套树状复制结构。

### 搭建集群

介绍完Redis集群分区规则之后，下面我们开始搭建Redis集群。搭建集群有几种方式：

1）依照Redis 协议手工搭建，使用cluster meet、cluster addslots、cluster replicate命令。

2）5.0之前使用由ruby语言编写的redis-trib.rb，在使用前需要安装ruby语言环境。

3）5.0及其之后redis摒弃了redis-trib.rb，将搭建集群的功能合并到了redis-cli。

我们简单点，采用第三种方式搭建。集群中至少应该有奇数个节点，所以至少有三个节点，官方推荐三主三从的配置方式，我们就来搭建一个三主三从的集群。

#### 节点配置

我们现在规定，主节点的端口为6900、6901、6902，从节点的端口为6930、6931、6932。

首先需要配置节点的conf文件，这个比较统一，所有的节点的配置文件都是类似的，我们以端口为6900的节点举例：

```
port 6900

# 这个部分是为了在一台服务上启动多台Redis服务，相关的资源要改
pidfile /var/run/redis_6900.pid
logfile "/home/tianming/redis/redis/log/6900.log"
dir "/home/tianming/redis/redis/data/"
dbfilename dump-6900.rdb

# Cluster Config
daemonize yes
cluster-enabled yes
cluster-config-file nodes-6900.conf
cluster-node-timeout 15000
appendonly yes
appendfilename "appendonly-6900.aof"

```

在上述配置中，以下配置是集群相关的：

```
cluster-enabled yes # 是否启动集群模式(集群需要修改为yes)

cluster-node-timeout 15000 指定集群节点超时时间(打开注释即可)

cluster-config-file nodes-6900.conf  指定集群节点的配置文件(打开注释即可)，这个文件不需要手工编辑,它由Redis节点创建和更新.每个Redis群集节点都需要不同的群集配置文件.确保在同一系统中运行的实例没有重叠群集配置文件名

appendonly yes  指定redis集群持久化方式(默认rdb,建议使用aof方式,此处是否修改不影响集群的搭建)
```

#### 集群创建

##### 创建集群随机主从节点

```
./redis-cli --cluster create 127.0.0.1:6900 127.0.0.1:6901 127.0.0.1:6902 127.0.0.1:6930 127.0.0.1:6931
127.0.0.1:6932 --cluster-replicas 1
```

说明：--cluster-replicas 参数为数字，1表示每个主节点需要1个从节点。

通过该方式创建的带有从节点的机器不能够自己手动指定主节点，不符合我们的要求。所以如果需要指定的话，需要自己手动指定，先创建好主节点后，再添加从节点。

##### 指定主从节点

###### 创建集群主节点

```
./redis-cli --cluster create  127.0.0.1:6900 127.0.0.1:6901 127.0.0.1:6902
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666925611079/4bff1e9d3ab84d4ab09ad629946dff27.png)

注意：

1、请记录下每个M后形如“dcd818ab48166ccea9563544839187ffa5d79f62”的字符串，在后面添加从节点时有用；

2、如果服务器存在着防火墙，那么在进行安全设置的时候，除了redis服务器本身的端口，比如6900 要加入允许列表之外，Redis服务在集群中还有一个叫集群总线端口，其端口为客户端连接端口加上10000，即 6900 + 10000 = 16900 。所以开放每个集群节点的客户端端口和集群总线端口才能成功创建集群！

###### 添加集群从节点

命令类似：

```
./redis-cli --cluster add-node 127.0.0.1:6930 127.0.0.1:6900 --cluster-slave --cluster-master-id dcd818ab48166ccea9563544839187ffa5d79f62
```

说明：上述命令把6382节点加入到6379节点的集群中，并且当做node_id为dcd818ab48166ccea9563544839187ffa5d79f62 的从节点。如果不指定 --cluster-master-id 会随机分配到任意一个主节点

效果如下：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666925611079/da6debed19fa42ce956ad2fe918b19ab.png)

第二个从，第三个从类似。

#### 集群管理

##### 检查集群

```
./redis-cli --cluster check 127.0.0.1:6900 --cluster-search-multiple-owners
```

说明：任意连接一个集群节点，进行集群状态检查

##### 集群信息查看

```
./redis-cli --cluster info 127.0.0.1:6900
```

说明：检查key、slots、从节点个数的分配情况

##### 修复集群

```
redis-cli --cluster fix 127.0.0.1:6900 --cluster-search-multiple-owners
```

说明：修复集群和槽的重复分配问题

##### 设置集群的超时时间

```
redis-cli --cluster set-timeout 127.0.0.1:6900 10000
```

说明：连接到集群的任意一节点来设置集群的超时时间参数cluster-node-timeout

##### 集群配置

```
redis-cli --cluster call 127.0.0.1:6900 config set requirepass cc

redis-cli --cluster call 127.0.0.1:6900 config set masterauth cc

redis-cli --cluster call 127.0.0.1:6900 config rewrite
```

说明：连接到集群的任意一节点来对整个集群的所有节点进行设置。

#### redis-cli –cluster 参数参考

```
redis-cli --cluster help
Cluster Manager Commands:
  create         host1:port1 ... hostN:portN   #创建集群
                 --cluster-replicas <arg>      #从节点个数
  check          host:port                     #检查集群
                 --cluster-search-multiple-owners #检查是否有槽同时被分配给了多个节点
  info           host:port                     #查看集群状态
  fix            host:port                     #修复集群
                 --cluster-search-multiple-owners #修复槽的重复分配问题
  reshard        host:port                     #指定集群的任意一节点进行迁移slot，重新分slots
                 --cluster-from <arg>          #需要从哪些源节点上迁移slot，可从多个源节点完成迁移，以逗号隔开，传递的是节点的node id，还可以直接传递--from all，这样源节点就是集群的所有节点，不传递该参数的话，则会在迁移过程中提示用户输入
                 --cluster-to <arg>            #slot需要迁移的目的节点的node id，目的节点只能填写一个，不传递该参数的话，则会在迁移过程中提示用户输入
                 --cluster-slots <arg>         #需要迁移的slot数量，不传递该参数的话，则会在迁移过程中提示用户输入。
                 --cluster-yes                 #指定迁移时的确认输入
                 --cluster-timeout <arg>       #设置migrate命令的超时时间
                 --cluster-pipeline <arg>      #定义cluster getkeysinslot命令一次取出的key数量，不传的话使用默认值为10
                 --cluster-replace             #是否直接replace到目标节点
  rebalance      host:port                                      #指定集群的任意一节点进行平衡集群节点slot数量 
                 --cluster-weight <node1=w1...nodeN=wN>         #指定集群节点的权重
                 --cluster-use-empty-masters                    #设置可以让没有分配slot的主节点参与，默认不允许
                 --cluster-timeout <arg>                        #设置migrate命令的超时时间
                 --cluster-simulate                             #模拟rebalance操作，不会真正执行迁移操作
                 --cluster-pipeline <arg>                       #定义cluster getkeysinslot命令一次取出的key数量，默认值为10
                 --cluster-threshold <arg>                      #迁移的slot阈值超过threshold，执行rebalance操作
                 --cluster-replace                              #是否直接replace到目标节点
  add-node       new_host:new_port existing_host:existing_port  #添加节点，把新节点加入到指定的集群，默认添加主节点
                 --cluster-slave                                #新节点作为从节点，默认随机一个主节点
                 --cluster-master-id <arg>                      #给新节点指定主节点
  del-node       host:port node_id                              #删除给定的一个节点，成功后关闭该节点服务
  call           host:port command arg arg .. arg               #在集群的所有节点执行相关命令
  set-timeout    host:port milliseconds                         #设置cluster-node-timeout
  import         host:port                                      #将外部redis数据导入集群
                 --cluster-from <arg>                           #将指定实例的数据导入到集群
                 --cluster-copy                                 #migrate时指定copy
                 --cluster-replace                              #migrate时指定replace

```

### 集群伸缩

Redis集群提供了灵活的节点扩容和收缩方案。在不影响集群对外服务的情况下，可以为集群添加节点进行扩容也可以下线部分节点进行缩容。

Redis集群可以实现对节点的灵活上下线控制。其中原理可抽象为槽和对应数据在不同节点之间灵活移动。首先来看我们之前搭建的集群槽和数据与节点的对应关系。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666925611079/020a4fcfb89345f480bd047a77cd372d.png)

三个主节点分别维护自己负责的槽和对应的数据，如果希望加入1个节点实现集群扩容时，需要通过相关命令把一部分槽和数据迁移给新节点。



#### 集群扩容

##### 节点配置和启动节点

我们加入两个节点，主节点的端口为6903，从节点的端口为6933。配置与前面的6900类似，不再赘述。

启动这两个节点。

./redis-server ../conf/cluster_m_6903.conf

./redis-server ../conf/cluster_s_6933.conf

##### 加入集群

执行命令

```
./redis-cli --cluster info 127.0.0.1:6900
```

执行命令

```
./redis-cli -p 6900  cluster nodes
```

可以看到，6903和6933还属于孤立节点，需要将这两个实例节点加入到集群中。

###### 将主节点6903加入集群

执行命令

```
./redis-cli --cluster add-node 127.0.0.1:6903 127.0.0.1:6900
```

执行命令

```
./redis-cli --cluster info 127.0.0.1:6900
```

执行命令

```
./redis-cli -p 6900  cluster nodes
```

###### 将从节点6933加入集群

执行命令

```
./redis-cli --cluster add-node 127.0.0.1:6933 127.0.0.1:6900 --cluster-slave --cluster-master-id 67dd0e8160a5bf8cd0ca02c2c6268bb9cc17884c
```

同时将刚刚加入的节点6903作为从节点6933的主节点

##### 迁移槽和数据

上面的图中可以看到，6903和6933已正确添加到集群中，接下来就开始分配槽位。我们将6900、6901、6902三个节点中的槽位分别迁出一些槽位给6903，假设分配后的每个节点槽位平均，那么应该分出（16384/4）=4096个槽位。

执行命令

```
./redis-cli --cluster reshard 127.0.01:6900
```

Redis会提问要迁移的槽位数和接受槽位的节点id，我们这里输入4096 67dd0e8160a5bf8cd0ca02c2c6268bb9cc17884c。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666925611079/3df8fdb055e0420a8f6048c4f0fd3c36.png)

接下来，Redis会提问从哪些源节点进行迁移，我们输入“all”

Redis会显示一个分配计划：

填入“yes”。

Redis会开始进行迁移

**这个时间会比较长.........................**

稍等一会，等待Redis迁移完成。

迁移完成后，执行命令

```
./redis-cli -p 6900  cluster nodes
```

```
./redis-cli --cluster info 127.0.0.1:6900
```

可以看到槽位确实被迁移到了节点6903之上。这样就实现了集群的扩容。

#### 集群缩容

##### 迁移槽和数据

命令语法：

```
redis-cli --cluster reshard --cluster-from 要迁出节点ID --cluster-to 接收槽节点ID --cluster-slots 迁出槽数量已存在节点ip 端口
```

例如：

迁出1365个槽位到6900节点

```
./redis-cli --cluster reshard --cluster-from 67dd0e8160a5bf8cd0ca02c2c6268bb9cc17884c
--cluster-to 7353cda9e84f6d85c0b6e41bb03d9c4bd2545c07 --cluster-slots 1365
127.0.0.1:6900
```

迁出1365个槽位到6901节点

```
./redis-cli --cluster reshard --cluster-from 67dd0e8160a5bf8cd0ca02c2c6268bb9cc17884c
--cluster-to 41ca2d569068043a5f2544c598edd1e45a0c1f91 --cluster-slots 1365
127.0.0.1:6900
```

迁出1366个槽位到6902节点

```
./redis-cli --cluster reshard --cluster-from 67dd0e8160a5bf8cd0ca02c2c6268bb9cc17884c
--cluster-to d53bb67e4c82b89a8d04d572364c07b3285e271f --cluster-slots 1366
127.0.0.1:6900
```

稍等片刻，等全部槽迁移完成后，执行命令

```
./redis-cli -p 6900  cluster nodes
```

```
./redis-cli --cluster info 127.0.0.1:6900
```

可以看到6903上不再存在着槽了。

##### 下线节点

执行命令格式redis-cli --cluster del-node 已存在节点:端口 要删除的节点ID

例如：

```
./redis-cli --cluster del-node 127.0.0.1:6900 67dd0e8160a5bf8cd0ca02c2c6268bb9cc17884c
```

```
./redis-cli --cluster del-node 127.0.0.1:6900 23c0ca7519a181f6ff61580eca014dde209f7a67
```

可以看到这两个节点确实脱离集群了，这样就完成了集群的缩容

再关闭节点即可。

#### 迁移相关

##### 在线迁移slot

在线把集群的一些slot从集群原来slot节点迁移到新的节点。其实在前面扩容集群的时候我们已经看到了相关的用法

直接连接到集群的任意一节点

redis-cli --cluster reshard XXXXXXXXXXX:XXXX

按提示操作即可。

##### 平衡（rebalance）slot

1）平衡集群中各个节点的slot数量

redis-cli --cluster rebalance XXXXXXXXXXX:XXXX

2）还可以根据集群中各个节点设置的权重来平衡slot数量

```
./redis-cli --cluster rebalance --cluster-weight 117457eab5071954faab5e81c3170600d5192270=5
815da8448f5d5a304df0353ca10d8f9b77016b28=4
56005b9413cbf225783906307a2631109e753f8f=3 --cluster-simulate
127.0.0.1:6900
```

### 请求路由

目前我们已经搭建好Redis集群并且理解了通信和伸缩细节，但还没有使用客户端去操作集群。Redis集群对客户端通信协议做了比较大的修改,为了追求性能最大化，并没有采用代理的方式而是采用客户端直连节点的方式。因此对于希望从单机切换到集群环境的应用需要修改客户端代码。

#### 请求重定向

在集群模式下，Redis接收任何键相关命令时首先计算键对应的槽,再根据槽找出所对应的节点，如果节点是自身，则处理键命令;否则回复MOVED重定向错误，通知客户端请求正确的节点。这个过程称为MOVED重定向。

例如,在之前搭建的集群上执行如下命令:

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666925611079/639f236673424cc4a13e454fafe70ca3.png)

执行set命令成功，因为键hello对应槽正好位于6900节点负责的槽范围内，可以借助cluster keyslot { key}命令返回key所对应的槽，如下所示:

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666925611079/767247687d254534b8b37d5ebee9ffbe.png)

再执行以下命令:

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666925611079/290fcc2910b14457b32ee0d2c72dd38f.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666925611079/87b18bc6d57345cba8c721f89637a791.png)

由于键对应槽是5798，不属于6900节点，则回复 MOVED (slot}{ip} {port]格式重定向信息，重定向信息包含了键所对应的槽以及负责该槽的节点地址,根据这些信息客户端就可以向正确的节点发起请求。

需要我们在6901节点上成功执行之前的命令:

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666925611079/c897cd9ff947402fbefed12d339c84b7.png)

使用redis-cli命令时，可以加入-c参数支持自动重定向，简化手动发起重定向操作,如下所示:

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666925611079/ead78dd9382c4ff78920855f22b02380.png)

redis-cli自动帮我们连接到正确的节点执行命令，这个过程是在redis-cli内部维护，实质上是client端接到MOVED信息之后再次发起请求,并不在Redis节点中完成请求转发。

同节点对于不属于它的键命令只回复重定向响应，并不负责转发。。正因为集群模式下把解析发起重定向的过程放到客户端完成,所以集群客户端协议相对于单机有了很大的变化。

键命令执行步骤主要分两步:计算槽,查找槽所对应的节点。。

##### 计算槽

Redis首先需要计算键所对应的槽。根据键的有效部分使用CRC16函数计算出散列值,再取对16383的余数,使每个键都可以映射到0 ~16383槽范围内。

##### 槽节点查找

Redis计算得到键对应的槽后，需要查找槽所对应的节点。集群内通过消息交换每个节点都会知道所有节点的槽信息。

根据MOVED重定向机制，客户端可以随机连接集群内任一Redis获取键所在节点，这种客户端又叫 Dummy（傀儡）客户端，它优点是代码实现简单,对客户端协议影响较小，只需要根据重定向信息再次发送请求即可。但是它的弊端很明显,每次执行键命令前都要到Redis上进行重定向才能找到要执行命令的节点,额外增加了IO开销，这不是Redis集群高效的使用方式。

正因为如此通常集群客户端都采用另一种实现:Smart(智能)客户端，我们后面再说。

#### call命令

call命令可以用来在集群的全部节点执行相同的命令。call命令也是需要通过集群的一个节点地址，连上整个集群，然后在集群的每个节点执行该命令。

./redis-cli --cluster call 47.112.44.148:6900 get name

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666925611079/d3f333557644483dad5abe098afbc018.png)

### Smart客户端

#### smart客户端原理

大多数开发语言的Redis客户端都采用Smart 客户端支持集群协议。Smart客户端通过在内部维护 slot →node的映射关系，本地就可实现键到节点的查找，从而保证IO效率的最大化，而MOVED重定向负责协助Smart客户端更新slot →node映射。Java的Jedis就默认实现了这个功能

#### ASK 重定向

1.客户端ASK 重定向流程

Redis集群支持在线迁移槽（slot)和数据来完成水平伸缩，当slot对应的数据从源节点到目标节点迁移过程中，客户端需要做到智能识别，保证键命令可正常执行。例如当一个slot数据从源节点迁移到目标节点时，期间可能出现一部分数据在源节点，而另一部分在目标节点。

当出现上述情况时,客户端键命令执行流程将发生变化:

1)客户端根据本地slots缓存发送命令到源节点，如果存在键对象则直接执行并返回结果给客户端。

2）如果键对象不存在，则可能存在于目标节点，这时源节点会回复ASK重定向异常。格式如下:(error) ASK (slot} {targetIP}:{targetPort}。

3)客户端从ASK重定向异常提取出目标节点信息，发送asking命令到目标节点打开客户端连接标识，再执行键命令。如果存在则执行,不存在则返回不存在信息。

ASK与MOVED虽然都是对客户端的重定向控制，但是有着本质区别。ASK重定向说明集群正在进行slot数据迁移，客户端无法知道什么时候迁移完成，因此只能是临时性的重定向，客户端不会更新slots缓存。但是MOVED重定向说明键对应的槽已经明确指定到新的节点,因此需要更新slots缓存。

#### 集群下的Jedis客户端

参见模块redis-cluster。

同时集群下的Jedis客户端只能支持有限的有限的批量操作，必须要求所有key的slot值相等。这时可以考虑使用hash tags。

##### Hash tags

集群支持hash tags功能，即可以把一类key定位到同一个slot，tag的标识目前不支持配置，只能使用{}，redis处理hash tag的逻辑也很简单，redis只计算从第一次出现{，到第一次出现}的substring的hash值，substring为空，则仍然计算整个key的值。

比如这两个键{user1000}.following 和 {user1000}.followers 会被哈希到同一个哈希槽里，因为只有 user1000 这个子串会被用来计算哈希值。

对于 foo{}{bar} 这个键，整个键都会被用来计算哈希值，因为第一个出现的 { 和它右边第一个出现的 } 之间没有任何字符。

对于 foo{bar}{zap} 这个键，用来计算哈希值的是 bar 这个子串。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666925611079/5c9a4d9d9cb14d2d8a1e7dd3d15419a2.png)

我们在使用hashtag特性时，一定要注意，不能把key的离散性变得非常差。

比如，没有利用hashtag特性之前，key是这样的：mall:sale:freq:ctrl:860000000000001，很明显这种key由于与用户相关，所以离散性非常好。

而使用hashtag以后，key是这样的：mall:sale:freq:ctrl:{860000000000001}，这种key还是与用户相关，所以离散性依然非常好。

我们千万不要这样来使用hashtag特性，例如将key设置为：mall:{sale:freq:ctrl}:860000000000001。

这样的话，无论有多少个用户多少个key，其{}中的内容完全一样都是sale:freq:ctrl，也就是说，所有的key都会落在同一个slot上，导致整个Redis集群出现严重的倾斜问题。

### 集群原理

#### 节点通信

##### 通信流程

在分布式存储中需要提供维护节点元数据信息的机制,所谓元数据是指:节点负责哪些数据,是否出现故障等状态信息。常见的元数据维护方式分为:集中式和P2P方式。Redis集群采用P2P的Gossip（流言)协议，Gossip协议工作原理就是节点彼此不断通信交换信息,一段时间后所有的节点都会知道集群完整的信息,这种方式类似流言传播。

通信过程说明:

1)集群中的每个节点都会单独开辟一个TCP通道,用于节点之间彼此通信,通信端口号在基础端口上加10000。

2)每个节点在固定周期内通过特定规则选择几个节点发送ping消息。

3）接收到ping消息的节点用pong消息作为响应。

集群中每个节点通过一定规则挑选要通信的节点，每个节点可能知道全部节点,也可能仅知道部分节点，只要这些节点彼此可以正常通信，最终它们会达到一致的状态。当节点出故障、新节点加入、主从角色变化、槽信息变更等事件发生时，通过不断的ping/pong消息通信，经过一段时间后所有的节点都会知道整个集群全部节点的最新状态，从而达到集群状态同步的目的。

##### Gossip 消息

Gossip协议的主要职责就是信息交换。信息交换的载体就是节点彼此发送的Gossip消息，了解这些消息有助于我们理解集群如何完成信息交换。

常用的Gossip消息可分为:ping消息、pong消息、meet消息、fail消息等，

**meet消息:**

用于通知新节点加入。消息发送者通知接收者加入到当前集群，meet消息通信正常完成后，接收节点会加入到集群中并进行周期性的ping、pong消息交换。

**ping消息:**

集群内交换最频繁的消息，集群内每个节点每秒向多个其他节点发送ping消息,用于检测节点是否在线和交换彼此状态信息。ping消息发送封装了自身节点和部分其他节点的状态数据。

**pong消息:**

当接收到ping、meet消息时，作为响应消息回复给发送方确认消息正常通信。pong消息内部封装了自身状态数据。节点也可以向集群内广播自身的pong消息来通知整个集群对自身状态进行更新。

**fail消息:**

当节点判定集群内另一个节点下线时，会向集群内广播一个fail消息,其他节点接收到fail消息之后把对应节点更新为下线状态。

所有的消息格式划分为:消息头和消息体。消息头包含发送节点自身状态数据，接收节点根据消息头就可以获取到发送节点的相关数据。

集群内所有的消息都采用相同的消息头结构clusterMsg，它包含了发送节点关键信息，如节点id、槽映射、节点标识(主从角色，是否下线）等。消息体在Redis内部采用clusterMsg Data 结构声明。

消息体clusterMsgData定义发送消息的数据,其中ping,meet、pong都采用clusterMsgDataGossip数组作为消息体数据，实际消息类型使用消息头的type属性区分。每个消息体包含该节点的多个clusterMsgDataGossip结构数据，用于信息交换。

当接收到ping、meet消息时,接收节点会解析消息内容并根据自身的识别情况做出相应处理。

##### 节点选择

虽然Gossip协议的信息交换机制具有天然的分布式特性，但它是有成本的。由于内部需要频繁地进行节点信息交换，而ping/pong消息会携带当前节点和部分其他节点的状态数据，势必会加重带宽和计算的负担。Redis集群内节点通信采用固定频率(定时任务每秒执行10次)。

因此节点每次选择需要通信的节点列表变得非常重要。通信节点选择过多虽然可以做到信息及时交换但成本过高。节点选择过少会降低集群内所有节点彼此信息交换频率，从而影响故障判定、新节点发现等需求的速度。因此Redis集群的Gossip协议需要兼顾信息交换实时性和成本开销。

消息交换的成本主要体现在单位时间选择发送消息的节点数量和每个消息携带的数据量。

1.选择发送消息的节点数量

集群内每个节点维护定时任务默认间隔1秒，每秒执行10次，定时任务里每秒随机选取5个节点，找出最久没有通信的节点发送ping消息，用于保证 Gossip信息交换的随机性。同时每100毫秒都会扫描本地节点列表，如果发现节点最近一次接受pong消息的时间大于cluster_node_timeout/2，则立刻发送ping消息，防止该节点信息太长时间未更新。

根据以上规则得出每个节点每秒需要发送ping消息的数量= 1 +10

* num(node.pong_received >cluster_node_timeout/2)，因此cluster_node_timeout参数对消息发送的节点数量影响非常大。当我们的带宽资源紧张时，可以适当调大这个参数，如从默认15秒改为30秒来降低带宽占用率。过度调大cluster_node_timeout 会影响消息交换的频率从而影响故障转移、槽信息更新、新节点发现的速度。因此需要根据业务容忍度和资源消耗进行平衡。同时整个集群消息总交换量也跟节点数成正比。

⒉消息数据量

每个ping消息的数据量体现在消息头和消息体中,其中消息头主要占用空间的字段是myslots [CLUSTER_SLOTS/8]，占用2KB，这块空间占用相对固定。消息体会携带一定数量的其他节点信息用于信息交换。

根消息体携带数据量跟集群的节点数息息相关，更大的集群每次消息通信的成本也就更高，因此对于Redis集群来说并不是大而全的集群更好。

#### 故障转移

Redis集群自身实现了高可用。高可用首先需要解决集群部分失败的场景：当集群内少量节点出现故障时通过自动故障转移保证集群可以正常对外提供服务。

##### 故障发现

当集群内某个节点出现问题时，需要通过一种健壮的方式保证识别出节点是否发生了故障。Redis集群内节点通过ping/pong消息实现节点通信，消息不但可以传播节点槽信息，还可以传播其他状态如:主从状态、节点故障等。因此故障发现也是通过消息传播机制实现的,主要环节包括:主观下线(pfail)和客观下线(fail)。

**主观下线:**

指某个节点认为另一个节点不可用，即下线状态，这个状态并不是最终的故障判定,只能代表一个节点的意见,可能存在误判情况。

**客观下线:**

指标记一个节点真正的下线，集群内多个节点都认为该节点不可用,从而达成共识的结果。如果是持有槽的主节点故障，需要为该节点进行故障转移。

###### 主观下线

集群中每个节点都会定期向其他节点发送ping消息，接收节点回复pong消息作为响应。如果在cluster-node-timeout时间内通信一直失败,则发送节点会认为接收节点存在故障，把接收节点标记为主观下线(pfail)状态。

流程说明:

1）节点a发送ping消息给节点b，如果通信正常将接收到pong消息，节点 a更新最近一次与节点b的通信时间。

2）如果节点 a与节点b通信出现问题则断开连接，下次会进行重连。如果一直通信失败,则节点a记录的与节点b最后通信时间将无法更新。

3）节点a内的定时任务检测到与节点b最后通信时间超高cluster-node-timeout时，更新本地对节点b的状态为主观下线(pfail)。

主观下线简单来讲就是，当cluster-note-timeout时间内某节点无法与另一个节点顺利完成ping消息通信时，则将该节点标记为主观下线状态。每个节点内的clusterstate结构都需要保存其他节点信息,用于从自身视角判断其他节点的状态。

Redis集群对于节点最终是否故障判断非常严谨，只有一个节点认为主观下线并不能准确判断是否故障。

比如节点6379与6385通信中断，导致6379判断6385为主观下线状态，但是6380与6385节点之间通信正常,这种情况不能判定节点6385发生故障。因此对于一个健壮的故障发现机制,需要集群内大多数节点都判断6385故障时,才能认为6385确实发生故障,然后为6385节点进行故障转移。而这种多个节点协作完成故障发现的过程叫做客观下线。

###### 客观下线

当某个节点判断另一个节点主观下线后，相应的节点状态会跟随消息在集群内传播。

ping/pong消息的消息体会携带集群1/10的其他节点状态数据，当接受节点发现消息体中含有主观下线的节点状态时，会在本地找到故障节点的ClusterNode结构，保存到下线报告链表中。

通过Gossip消息传播，集群内节点不断收集到故障节点的下线报告。当半数以上持有槽的主节点都标记某个节点是主观下线时。触发客观下线流程。这里有两个问题:

1)为什么必须是负责槽的主节点参与故障发现决策?因为集群模式下只有处理槽的主节点才负责读写请求和集群槽等关键信息维护，而从节点只进行主节点数据和状态信息的复制。

2）为什么半数以上处理槽的主节点？必须半数以上是为了应对网络分区等原因造成的集群分割情况，被分割的小集群因为无法完成从主观下线到客观下线这一关键过程，从而防止小集群完成故障转移之后继续对外提供服务。

尝试客观下线

集群中的节点每次接收到其他节点的pfail状态，都会尝试触发客观下线，

流程说明:

1)首先统计有效的下线报告数量,如果小于集群内持有槽的主节点总数的一半则退出。

2）当下线报告大于槽主节点数量一半时，标记对应故障节点为客观下线状态。

3)向集群广播一条fail消息，通知所有的节点将故障节点标记为客观下线,fail消息的消息体只包含故障节点的ID。

广播fail消息是客观下线的最后一步,它承担着非常重要的职责:

通知集群内所有的节点标记故障节点为客观下线状态并立刻生效。

通知故障节点的从节点触发故障转移流程。

##### 故障恢复

故障节点变为客观下线后,如果下线节点是持有槽的主节点则需要在它的从节点中选出一个替换它,从而保证集群的高可用。下线主节点的所有从节点承担故障恢复的义务，当从节点通过内部定时任务发现自身复制的主节点进入客观下线时,将会触发故障恢复流程。

###### 资格检查

每个从节点都要检查最后与主节点断线时间，判断是否有资格替换故障的主节点。如果从节点与主节点断线时间超过cluster-node-time * cluster-slave-validity-factor，则当前从节点不具备故障转移资格。参数cluster-slave-validity-factor用于从节点的有效因子，默认为10。

###### 准备选举时间

当从节点符合故障转移资格后，更新触发故障选举的时间，只有到达该时间后才能执行后续流程。

这里之所以采用延迟触发机制，主要是通过对多个从节点使用不同的延迟选举时间来支持优先级问题。复制偏移量越大说明从节点延迟越低，那么它应该具有更高的优先级来替换故障主节点。

所有的从节点中复制偏移量最大的将提前触发故障选举流程。

主节点b进入客观下线后，它的三个从节点根据自身复制偏移量设置延迟选举时间，如复制偏移量最大的节点slave b-1延迟1秒执行，保证复制延迟低的从节点优先发起选举。

###### 发起选举

当从节点定时任务检测到达故障选举时间(failover_auth_time）到达后，发起选举流程如下:

(1）更新配置纪元

配置纪元是一个只增不减的整数，每个主节点自身维护一个配置纪元(clusterNode .configEpoch)标示当前主节点的版本，所有主节点的配置纪元都不相等，从节点会复制主节点的配置纪元。整个集群又维护一个全局的配置纪元(clusterstate.currentEpoch)，用于记录集群内所有主节点配置纪元的最大版本。执行cluster info命令可以查看配置纪元信息:

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1666925611079/7d2573420ab14414912068c2c0469879.png)

配置纪元的主要作用:

标示集群内每个主节点的不同版本和当前集群最大的版本。

每次集群发生重要事件时，这里的重要事件指出现新的主节点(新加入的或者由从节点转换而来)，从节点竞争选举。都会递增集群全局的配置纪元并赋值给相关主节点,用于记录这一关键事件。

主节点具有更大的配置纪元代表了更新的集群状态，因此当节点间进行ping/pong消息交换时，如出现slots等关键信息不一致时，以配置纪元更大的一方为准，防止过时的消息状态污染集群。

配置纪元的应用场景有:

新节点加入。槽节点映射冲突检测。从节点投票选举冲突检测。

###### 选举投票

只有持有槽的主节点才会处理故障选举消息(FAILOVER_AUTH_REQUEST)，因为每个持有槽的节点在一个配置纪元内都有唯一的一张选票，当接到第一个请求投票的从节点消息时回复FAILOVER_AUTH_ACK消息作为投票，之后相同配置纪元内其他从节点的选举消息将忽略。

投票过程其实是一个领导者选举的过程，如集群内有N个持有槽的主节点代表有N张选票。由于在每个配置纪元内持有槽的主节点只能投票给一个从节点，因此只能有一个从节点获得 N/2+1的选票,保证能够找出唯一的从节点。

Redis集群没有直接使用从节点进行领导者选举，主要因为从节点数必须大于等于3个才能保证凑够N/2+1个节点，将导致从节点资源浪费。使用集群内所有持有槽的主节点进行领导者选举,即使只有一个从节点也可以完成选举过程。

当从节点收集到N/2+1个持有槽的主节点投票时，从节点可以执行替换主节点操作，例如集群内有5个持有槽的主节点，主节点b故障后还有4个，当其中一个从节点收集到3张投票时代表获得了足够的选票可以进行替换主节点操作,。

投票作废:每个配置纪元代表了一次选举周期,如果在开始投票之后的cluster-node-timeout*2时间内从节点没有获取足够数量的投票，则本次选举作废。从节点对配置纪元自增并发起下一轮投票,直到选举成功为止。

依次判断依据：

1.断开时间。（记录断开时间，如果断开太久之后起来的取消你的资格，过滤故障节点）

2.配置 slave_priority   （redis.conf  priority）越小优先级越高

3.如果相同还有数据偏移量  （给每个从 同步的进度，当主从偏移量一致则完全同步）  

4.(进程ID)/runid （redis每次启动随机生成作为redis标识）最小的从节点作为主节点

###### 替换主节点

当从节点收集到足够的选票之后,触发替换主节点操作:

1)当前从节点取消复制变为主节点。

2)执行clusterDelslot 操作撤销故障主节点负责的槽，并执行clusterAddSlot把这些槽委派给自己。

3)向集群广播自己的pong消息，通知集群内所有的节点当前从节点变为主节点并接管了故障主节点的槽信息。

##### 故障转移时间

在介绍完故障发现和恢复的流程后,这时我们可以估算出故障转移时间:

1）主观下线(pfail）识别时间=cluster-node-timeout。

2）主观下线状态消息传播时间<=cluster-node-timeout/2。消息通信机制对超过cluster-node-timeout/2未通信节点会发起ping消息，消息体在选择包含哪些节点时会优先选取下线状态节点，所以通常这段时间内能够收集到半数以上主节点的pfail 报告从而完成故障发现。

3)从节点转移时间<=1000毫秒。由于存在延迟发起选举机制,偏移量最大的从节点会最多延迟1秒发起选举。通常第一次选举就会成功，所以从节点执行转移时间在1秒以内。

根据以上分析可以预估出故障转移时间，如下:

failover-time(毫秒)≤cluster-node-timeout

+ cluster-node-timeout/2 + 1000

因此，故障转移时间跟cluster-node-timeout参数息息相关，默认15秒。配置时可以根据业务容忍度做出适当调整，但不是越小越好。

#### 集群不可用判定

为了保证集群完整性，默认情况下当集群16384个槽任何一个没有指派到节点时整个集群不可用。

比如ABC三个主节点的集群，当B节点发生故障且没有替代方案，会导致整个集群不可用。

执行任何键命令都会返回( error)CLUSTERDOWN Hash slot not served错误。这是对集群完整性的一种保护措施，保证所有的槽都指派给在线的节点。但是当持有槽的主节点下线时，从故障发现到自动完成转移期间整个集群是不可用状态，对于大多数业务无法容忍这种情况，因此可以将参数**cluster-require-full-coverage配置为no**，当主节点故障时只影响它负责槽的相关命令执行，不会影响其他主节点的可用性。

但是从集群的故障转移的原理来说，集群会出现不可用？

1、当访问一个 Master 和 Slave 节点都挂了的时候，cluster-require-full-coverage=yes，会报槽无法获取。

2、集群主库半数宕机(根据 failover 原理，fail 掉一个主需要一半以上主都投票通过才可以)。

另外当集群 Master 节点个数小于 3 个的时候，或者集群可用节点个数为偶数的时候，基于 fail 的这种选举机制的自动主从切换过程可能会不能正常工作，一个是标记 fail 的过程，一个是选举新的 master 的过程，都有可能异常。

其他原因：比如大量的异地访问请求

当一个 Redis 集群面对非常大量的分散在多个地方的并发访问请求时，可能会因过度资源分配或某些节点处理不均衡而导致性能下降。这种情况在高峰期尤为明显。

比如Redis 集群中的 Hash 分区功能会将数据散布在不同的节点上，如果分配不够均衡，则可能会出现某些节点负载过多，而其他节点闲置的问题。

这意味着在集群中存在一些负载高得无法承受更多负载的节点，同时还有其他节点因完全没有负载而浪费资源。当一个集群中只有几个节点运行良好时，如果一个或两个节点宕机，这极有可能导致整个 Redis 集群的不可用状态。





#### 集群读写分离

1.只读连接

集群模式下从节点不接受任何读写请求，发送过来的键命令会重定向到负责槽的主节点上(其中包括它的主节点)。当需要使用从节点分担主节点读压力时，可以使用readonly命令打开客户端连接只读状态。之前的复制配置slave-read-only在集群模式下无效。当开启只读状态时，从节点接收读命令处理流程变为:如果对应的槽属于自己正在复制的主节点则直接执行读命令，否则返回重定向信息。

readonly命令是连接级别生效，因此每次新建连接时都需要执行readonly开启只读状态。执行readwrite命令可以关闭连接只读状态。

2.读写分离

集群模式下的读写分离，同样会遇到:复制延迟，读取过期数据,从节点故障等问题。针对从节点故障问题,客户端需要维护可用节点列表，集群提供了cluster slaves {nodeld}命令，返回nodeId对应主节点下所有从节点信息，命令如下:

cluster slave
41ca2d569068043a5f2544c598edd1e45a0c1f91

解析以上从节点列表信息,排除fail状态节点，这样客户端对从节点的故障判定可以委托给集群处理,简化维护可用从节点列表难度。

同时集群模式下读写分离涉及对客户端修改如下:

1）维护每个主节点可用从节点列表。

2）针对读命令维护请求节点路由。

3）从节点新建连接开启readonly状态。

集群模式下读写分离成本比较高，可以直接扩展主节点数量提高集群性能，一般不建议集群模式下做读写分离。

## Redis缓存使用问题



### 数据一致性

### 一致性分为

###### 1、强一致性 

如果你的项目对缓存的要求是强一致性的，那么请不要使用缓存。这种一致性级别是最符合用户直觉的，它要求系统写入什么，读出来的也会是什么，用户体验好，但实现起来往往对系统的性能影响大。

###### 2、弱一致性 

这种一致性级别约束了系统在写入成功后，不承诺立即可以读到写入的值，也不承诺多久之后数据能够达到一致，但会尽可能地保证到某个时间级别（比如秒级别）后，数据能够达到一致状态。

###### 3、最终一致性 

最终一致性是弱一致性的一个特例，系统会保证在一定时间内，能够达到一个数据一致的状态。这里之所以将最终一致性单独提出来，是因为它是弱一致性中非常推崇的一种一致性模型，也是业界在大型分布式系统的数据一致性上比较推崇的模型。一般情况下，高可用只确保最终一致性，不确保强一致性。

强一致性，读请求和写请求会串行化，串到一个内存队列里去，这样会大大增加系统的处理效率，吞吐量也会大大降低。

#### 为什么会有一致性的问题呢？

对于热点数据（经常查询，但不经常修改的数据），我们可以放入redis缓存中，因为如果我们使用Mysql的话，DB是扛不住的。因此采用缓存中间件来增加查询效率，但需要保证Redis中读取数据与数据库存储数据是一致的。

客户端对于数据库主要是读写两个操作。针对放入redis中缓存的热点数据，当客户端想读取数据的时候就在缓存中直接返回数据，即缓存命中，当读数据不在缓存内，就需要从数据库中将数据读入缓存，即缓存未命中。我们可以看到读操作不会导致缓存与数据库的数据不一致。


只要使用到缓存，无论是本地内存做缓存还是使用 redis 做缓存，那么就会存在数据同步的问题。

我以 Tomcat 向 MySQL 中写入和删改数据为例，来给你解释一下，数据的增删改操作具体是如何进行的。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1667197328004/5a0bb5a11d0d41648dd390c8fecdc6ec.png)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1667197328004/c1723be2ec76469eac667f9210beeede.png)



![img](https://www.yingsoo.com/static/upload/1758_0129210303_kz0srkaik0i.png)

这张图，大多数人的很多业务操作都是根据这个图来做缓存的。但是一旦设计到双写或者数据库和缓存更新等操作，就很容易出现数据一致性的问题。无论是先写数据库，在删除缓存，还是先删除缓存，在写入数据库，都会出现数据一致性的问题。列举两个小例子。

1、 先删除了redis缓存，但是因为其他什么原因还没来得及写入数据库，另外一个线程就来读取，发现缓存为空，则去数据库读取到之前的数据并写入缓存，此时缓存中为脏数据。

2、 如果先写入了数据库，但是在缓存被删除前，写入数据库的线程因为其他原因被中断了，没有删除掉缓存，就也会出现数据不一致的情况。

总的来说，写和读在多数情况下都是并发的，**不能绝对保证先后顺序**，就会很容易出现缓存和数据库数据不一致的情况，还怎么解决呢？

我们分析一下几种解决方案，

1、先更新缓存，再更新数据库

2、先更新数据库，再更新缓存

3、先删除缓存，后更新数据库

4、先更新数据库，后删除缓存

#### 新增数据类

如果是新增数据，数据会直接写到数据库中，不用对缓存做任何操作，此时，缓存中本身就没有新增数据，而数据库中是最新值，此时，缓存和数据库的数据是一致的。

#### 更新缓存类

##### 1、先更新缓存，再更新DB

这个方案我们**一般不考虑**。原因是更新缓存成功，更新数据库出现异常了，导致缓存数据与数据库数据完全不一致，而且很难察觉，因为缓存中的数据一直都存在。

##### 2、先更新DB，再更新缓存

这个方案也我们**一般不考虑**，原因跟第一个一样，数据库更新成功了，缓存更新失败，同样会出现数据不一致问题。同时还有以下问题

*1* *）并发问题：*

*同时有请求A**和请求B**进行更新操作，那么会出现*

*（1* *）线程A*更新了数据库

*（2* *）线程B*更新了数据库

*（3* *）线程B*更新了缓存

*（4* *）线程A*更新了缓存

*这就出现请求A**更新缓存应该比请求B**更新缓存早才对，但是因为网络等原因，B**却比A**更早更新了缓存。这就导致了脏数据，因此不考虑。*

*2* *）业务场景问题*

*如果你是一个写数据库场景比较多，而读数据场景比较少的业务需求，采用这种方案就会导致，数据压根还没读到，缓存就被频繁的更新，浪费性能。*

**除了更新缓存之外，我们还有一种就是删除缓存。**

到底是选择更新缓存还是淘汰缓存呢？

主要取决于“更新缓存的复杂度”，更新缓存的代价很小，此时我们应该更倾向于更新缓存，以保证更高的缓存命中率，更新缓存的代价很大，此时我们应该更倾向于淘汰缓存。

#### 删除缓存类

##### 3、先删除缓存，后更新DB

该方案也**会出问题**，具体出现的原因如下。

1、此时来了两个请求，请求 A（更新操作） 和请求 B（查询操作）

2、请求 A 会先删除 Redis 中的数据，然后去数据库进行更新操作；

3、此时请求 B 看到 Redis 中的数据时空的，会去数据库中查询该值，补录到 Redis 中；

4、但是此时请求 A 并没有更新成功，或者事务还未提交，请求B去数据库查询得到旧值；

5、那么这时候就会产生数据库和 Redis 数据不一致的问题。

如何解决呢？其实最简单的解决办法就是**延时双删的策略**。就是

（1）先淘汰缓存

（2）再写数据库

（3）休眠1秒，再次淘汰缓存

**这段伪代码就是“延迟双删”**

```java
redis.delKey(X)
db.update(X)
Thread.sleep(N)
redis.delKey(X)
```

这么做，可以将1秒内所造成的缓存脏数据，再次删除。

那么，这个1秒怎么确定的，具体该休眠多久呢？

针对上面的情形，读该自行评估自己的项目的读数据业务逻辑的耗时。然后写数据的休眠时间则在读数据业务逻辑的耗时基础上，加几百ms即可。这么做的目的，就是确保读请求结束，写请求可以删除读请求造成的缓存脏数据。



### 解决方案


通常情况下，我们使用缓存的主要目的是为了提升查询的性能。大多数情况下，我们是这样使用缓存的：

![在这里插入图片描述](https://img-blog.csdnimg.cn/fc4e924359ec4b0086139a70117ae076.png)

这是缓存最常用的用法，看上去没问题，但你忽视了一个非常重要的细节：**如果数据库的某条数据，放入缓存后，又立马更新了，那么如何更新缓存呢？**因为不更新缓存，下一次读取的时候命中缓存读取的就是旧数据。
目前更新缓存主要有以下四种方案：
先写缓存，在更新数据库
先更新数据库，在写缓存（双写）

##### 建议缓存一致的处理：

3. 先删除缓存，在更新数据库
4. 先写数据库，在删除缓存

##### 先写缓存，再 写数据库

![在这里插入图片描述](https://img-blog.csdnimg.cn/56ec843cd77d48939f43dd1b8a1963ee.png)

我们想一下，如果我们每次写操作后，刚写完缓存，突然网络不好导致数据库写入失败。

![在这里插入图片描述](https://img-blog.csdnimg.cn/11e0e0ac5c6249108b0f41548088c119.png)

**缓存更新成了最新数据，但数据库没有，这样缓存中的数据不就变成脏数据了？**如果此时该用户的查询请求，正好读取到该数据，就会出现问题，因为该数据在数据库中根本不存在，这个问题非常严重。
我们都知道，缓存的主要目的是把数据库的数据临时保存在内存，便于后续的查询，提升查询速度。
但如果某条数据，在数据库中都不存在，你缓存这种“假数据”又有啥意义呢？
因此，先写缓存，再写数据库的方案是不可取的，在实际工作中用得不多

先更新数据库在更新缓存

![在这里插入图片描述](https://img-blog.csdnimg.cn/7612360f372548f283702ed7ac1b31af.png)

用户的写操作，先写数据库，再写缓存，可以避免之前“假数据”的问题。但它却带来了新的问题。
什么问题呢？

在高并发业务场景下，写数据库和写缓存都属于远程操作，为了防止出现大事物造成死锁问题，通常==建议写数据库和写缓存不要放在一个事务中。==也就是说该方案中，如果数据库成功了但是写缓存失败了，数据库中已写入的数据不会进行回滚。
会出现数据库新数据，缓存是旧数据的问题

![在这里插入图片描述](https://img-blog.csdnimg.cn/960bb3fd8d63407faffff2e95f88c898.png)

请求a先过来，刚写完了数据库。但由于网络原因，卡顿了一下，还没来得及写缓存。
这时候请求b过来了，先写了数据库。
接下来，请求b顺利写了缓存。
此时，请求a卡顿结束，也写了缓存。
很显然，在这个过程当中，请求b在缓存中的新数据，被请求a的旧数据覆盖了。
也就是说：在高并发场景中，如果多个线程同时执行先写数据库，再写缓存的操作，可能会出现数据库是新值，而缓存中是旧值，两边数据不一致的情况。
**从上我们可以看到先写数据库在写缓存是比较浪费系统资源的，不建议使用**

##### 更新数据库更新缓存使用的场景

如果我们的业务对缓存命中率有很高的要求，我们可以采用「更新数据库 + 更新缓存」的方案，因为更新缓存并不会出现缓存未命中的情况。

**解决方案**
在更新缓存前先加个分布式锁，保证同一时间只运行一个请求更新缓存，就会不会产生并发问题了，当然引入了锁后，对于写入的性能就会带来影响。
在更新完缓存时，给缓存加上较短的过期时间，这样即时出现缓存不一致的情况，缓存的数据也会很快过期，对业务还是能接受的
通过上述两个双写我们可以知道，如果直接更新缓存的问题是很多的，因此我们换一种思路，从更新缓存->删除缓存。
先删除缓存，在更新数据库

![在这里插入图片描述](https://img-blog.csdnimg.cn/d2707daeef3a48e384efb3b196d0268c.png)

高并发下

![在这里插入图片描述](https://img-blog.csdnimg.cn/ef846d79c3a4454887b95422d4228bae.png)


A线程删除缓存，但是此时更新数据库的操作还未完成，此时B线程来读取缓存发现缓存没有数据，就去读取数据库的旧值，更新到缓存中，此时A线程更新完了，将新值写入数据库。这种场景下的数据不一致性问题怎么解决呢？。

解决方案，**延迟双删**
A线程删除缓存在更新数据库，此时A的更新操作还未完成，而B线程来读取缓存发现缓存没有，去读取数据库，读取的是旧值，然后把旧值写入缓存。A线程Sleep到B线程写入缓存后，在执行删除缓存操作。当其他线程来读取时，数据库就是最新值。

![在这里插入图片描述](https://img-blog.csdnimg.cn/187b32d6009d4eea8739742f2059ce30.png)


如果第二次删除删除缓存失败，那么可以采用消息队列的重试机制。

如果第二次删除失败，采用重试机制
重试机制原理图

![在这里插入图片描述](https://img-blog.csdnimg.cn/b9e64de3a9974f01bc13b9e9ab420a49.png)

先更新数据库，在删除缓存

![在这里插入图片描述](https://img-blog.csdnimg.cn/a826d5a578ef4fbc9c6aadee189208d0.png)

这个更明显，如果A线程数据库更新成功，而缓存失败的话，或者还未来得及删除，那么此时B线程来读取就是旧值，还是会不一致。

![在这里插入图片描述](https://img-blog.csdnimg.cn/a04542481cb84c778f21415c22128236.png)

解决措施（重试和binlog）：
消息队列
我们可以引入消息队列加粗样式，将第二个操作（删除缓存）要操作的数据加入到消息队列，由消费者来操作数据。
如果应用删除缓存失败，可以从消息队列中重新读取数据，然后再次删除缓存，这个就是重试机制。当然，如果重试超过的一定次数，还是没有成功，我们就需要向业务层发送报错信息了。
如果删除缓存成功，就要把数据从消息队列中移除，避免重复操作，否则就继续重试。

订阅Mysql binLog，在操作缓存
「先更新数据库，再删缓存」的策略的第一步是更新数据库，那么更新数据库成功，就会产生一条变更日志，记录在 binlog 里。

于是我们就可以通过订阅 binlog 日志，拿到具体要操作的数据，然后再执行缓存删除，阿里巴巴开源的 Canal 中间件就是基于这个实现的。

**Canal 模拟 MySQL 主从复制的交互协议，把自己伪装成一个 MySQL 的从节点，**向 MySQL 主节点发送 dump 请求，MySQL 收到请求后，就会开始推送 Binlog 给 Canal，Canal 解析 Binlog 字节流之后，转换为便于读取的结构化数据，供下游程序订阅使用。

![在这里插入图片描述](https://img-blog.csdnimg.cn/f1df6d6557af430eb356ce02ad25ebdf.png)

所以，如果要想保证「先更新数据库，再删缓存」策略第二个操作能执行成功，我们可以使用「消息队列来重试缓存的删除」，或者「订阅 MySQL binlog 再操作缓存」，这两种方法有一个共同的特点，都是采用异步操作缓存

### 归纳总结：

**1.手动清除Redis缓存，在重新查询最新的数据同步到Redis中。** 
**2.更新Mysql数据库，在采用MQ异步的形式同步数据到Redis中，优点是解耦，缺点是延迟的概率大。**
**3.更新数据库，在基于订阅数据库中的binlog日志采用mq异步的形式同步到Redis中。**
**4.订阅mysql中的Binlog文件，异步的形式同步到Redis中（canal框架）**
 如何选择问题

一般在线上，更多的偏向与使用删除缓存类操作，因为这种方式的话，会更容易避免一些问题。

因为删除缓存更新缓存的速度比在DB中要快一些，所以一般情况下我们可能会先用先更新DB，后删除缓存的操作。因为这种情况下缓存不一致性的情况只有可能是查询比删除慢的情况，而这种情况相对来说会少很多。同时结合延时双删的处理，可以有效的避免缓存不一致的情况。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1667197328004/7cc8705c0aee489f9b8b2ecccadf023c.png)

## 







## 缓存穿透、击穿、雪崩

### 缓存穿透

是指查询一个根本不存在的数据，缓存层和存储层都不会命中，于是这个请求就可以随意访问数据库，这个就是缓存穿透，缓存穿透将导致不存在的数据每次请求都要到存储层去查询，失去了缓存保护后端存储的意义。

缓存穿透问题可能会使后端存储负载加大，由于很多后端存储不具备高并发性，甚至可能造成后端存储宕掉。通常可以在程序中分别统计总调用数、缓存层命中数、存储层命中数，如果发现大量存储层空命中，可能就是出现了缓存穿透问题。

造成缓存穿透的基本原因有两个。

第一，自身业务代码或者数据出现问题，比如，我们数据库的 id 都是1开始自增上去的，如发起为id值为 -1 的数据或 id 为特别大不存在的数据。如果不对参数做校验，数据库id都是大于0的，我一直用小于0的参数去请求你，每次都能绕开Redis直接打到数据库，数据库也查不到，每次都这样，并发高点就容易崩掉了。

第二,一些恶意攻击、爬虫等造成大量空命中。下面我们来看一下如何解决缓存穿透问题。

1.缓存空对象

当存储层不命中，到数据库查发现也没有命中，那么仍然将空对象保留到缓存层中，之后再访问这个数据将会从缓存中获取,这样就保护了后端数据源。

缓存空对象会有两个问题:

第一，空值做了缓存，意味着缓存层中存了更多的键，需要更多的内存空间(如果是攻击，问题更严重),比较有效的方法是针对这类数据设置一个较短的过期时间，让其自动剔除。

第二，缓存层和存储层的数据会有一段时间窗口的不一致，可能会对业务有一定影响。例如过期时间设置为5分钟，如果此时存储层添加了这个数据，那此段时间就会出现缓存层和存储层数据的不一致，此时可以利用消前面所说的数据一致性方案处理。

2.布隆过滤器拦截

在访问缓存层和存储层之前,将存在的key用布隆过滤器提前保存起来,做第一层拦截。例如:一个推荐系统有4亿个用户id，每个小时算法工程师会根据每个用户之前历史行为计算出推荐数据放到存储层中,但是最新的用户由于没有历史行为,就会发生缓存穿透的行为,为此可以将所有推荐数据的用户做成布隆过滤器。如果布隆过滤器认为该用户id不存在,那么就不会访问存储层,在一定程度保护了存储层。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1667197328004/91ac8664908a42a1bf9a1ce23aeec6c8.png)

这种方法适用于数据命中不高、数据相对固定、实时性低(通常是数据集较大)的应用场景,代码维护较为复杂,但是缓存空间占用少。

##### 布隆过滤器的应用：

![image-20231203173207760](E:\图灵课堂\Redis专题\笔记\1、Redis专题.assets\image-20231203173207760.png)



### 缓存击穿

缓存击穿是指一个Key非常热点，在不停的扛着大并发，大并发集中对这一个点进行访问，当这个Key在失效的瞬间，持续的大并发就穿破缓存，直接请求数据库，就像在一个完好无损的桶上凿开了一个洞。

缓存击穿的话，设置热点数据永远不过期。或者加上互斥锁就能搞定了。

#### 使用互斥锁(mutex key)

业界比较常用的做法，是使用mutex。简单地来说，就是在缓存失效的时候（判断拿出来的值为空），不是立即去load db，而是先使用缓存工具的某些带成功操作返回值的操作（比如Redis的SETNX或者Memcache的ADD）去set一个mutex key，当操作返回成功时，再进行load db的操作并回设缓存；否则，就重试整个get缓存的方法。

伪代码如下图：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1667197328004/d360eff05cd945739aa9c81ef633c300.png)

#### 永远不过期

这里的“永远不过期”包含两层意思：

(1) 从redis上看，确实没有设置过期时间，这就保证了，不会出现热点key过期问题，也就是“物理”不过期。

(2) 从功能上看，如果不过期，那不就成静态的了吗？所以我们把过期时间存在key对应的value里，如果发现要过期了，通过一个后台的异步线程进行缓存的构建，也就是“逻辑”过期

从实战看，这种方法对于性能非常友好，唯一不足的就是构建缓存时候，其余线程(非构建缓存的线程)可能访问的是老数据，但是对于一般的互联网功能来说这个还是可以忍受。

### 缓存雪崩

缓存雪崩:由于缓存层承载着大量请求,有效地保护了存储层,但是如果缓存层由于某些原因不能提供服务，比如同一时间缓存数据大面积失效，那一瞬间Redis跟没有一样，于是所有的请求都会达到存储层，存储层的调用量会暴增，造成存储层也会级联宕机的情况。

缓存雪崩的英文原意是stampeding herd(奔逃的野牛)，指的是缓存层宕掉后，流量会像奔逃的野牛一样,打向后端存储。

预防和解决缓存雪崩问题,可以从以下三个方面进行着手。

1）保证缓存层服务高可用性。和飞机都有多个引擎一样，如果缓存层设计成高可用的,即使个别节点、个别机器、甚至是机房宕掉，依然可以提供服务，例如前面介绍过的Redis

Sentinel和 Redis Cluster都实现了高可用。

2）依赖隔离组件为后端限流并降级。无论是缓存层还是存储层都会有出错的概率，可以将它们视同为资源。作为并发量较大的系统，假如有一个资源不可用，可能会造成线程全部阻塞(hang)在这个资源上，造成整个系统不可用。

3）提前演练。在项目上线前，演练缓存层宕掉后，应用以及后端的负载情况以及可能出现的问题,在此基础上做一些预案设定。

4）将缓存失效时间分散开，比如我们可以在原有的失效时间基础上增加一个随机值，比如1-5分钟随机，这样每一个缓存的过期时间的重复率就会降低，就很难引发集体失效的事件。

### 热点Key

在Redis中，访问频率高的key称为热点key。

#### 产生原因和危害

#### 原因

热点问题产生的原因大致有以下两种：

用户消费的数据远大于生产的数据（热卖商品、热点新闻、热点评论、明星直播）。

在日常工作生活中一些突发的事件，例如：双十一期间某些热门商品的降价促销，当这其中的某一件商品被数万次点击浏览或者购买时，会形成一个较大的需求量，这种情况下就会造成热点问题。同理，被大量刊发、浏览的热点新闻、热点评论、明星直播等，这些典型的读多写少的场景也会产生热点问题。

请求分片集中，超过单Server的性能极限。在服务端读数据进行访问时，往往会对数据进行分片切分，此过程中会在某一主机Server上对相应的Key进行访问，当访问超过Server极限时，就会导致热点Key问题的产生。

缓存雪崩的场景通常有两个：

- 大量热点key同时过期
- 缓存服务故障或宕机

#### 危害

1、流量集中，达到物理网卡上限。

2、请求过多，缓存分片服务被打垮。

3、DB击穿，引起业务雪崩。

### 发现热点key

#### 预估发现

针对业务提前预估出访问频繁的热点key，例如秒杀商品业务中，秒杀的商品都是热点key。

当然并非所有的业务都容易预估出热点key，可能出现漏掉或者预估错误的情况。

#### 客户端发现

客户端其实是距离key"最近"的地方，因为Redis命令就是从客户端发出的，以Jedis为例，可以在核心命令入口，使用这个Google Guava中的AtomicLongMap进行记录，如下所示。

使用客户端进行热点key的统计非常容易实现，但是同时问题也非常多：

(1) 无法预知key的个数，存在内存泄露的危险。

(2) 对于客户端代码有侵入，各个语言的客户端都需要维护此逻辑，维护成本较高。

(3) 规模化汇总实现比较复杂。

#### Redis发现

##### monitor命令

monitor命令可以监控到Redis执行的所有命令，利用monitor的结果就可以统计出一段时间内的热点key排行榜，命令排行榜，客户端分布等数据。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1667197328004/8aceffd9136f47d5a20466ae69ed46ba.png)

Facebook开源的redis-faina正是利用上述原理使用Python语言实现的，例如下面获取最近10万条命令的热点key、热点命令、耗时分布等数据。为了减少网络开销以及加快输出缓冲区的消费速度，monitor尽可能在本机执行。

**此种方法会有两个问题：**

1、monitor命令在高并发条件下，内存暴增同时会影响Redis的性能，所以此种方法适合在短时间内使用。

2、只能统计一个Redis节点的热点key，对于Redis集群需要进行汇总统计。

**可以参考的框架：Facebook开源的redis-faina正是利用上述原理使用Python语言实现的**

##### hotkeys

Redis在4.0.3中为redis-cli提供了--hotkeys，用于找到热点key。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1667197328004/9b76d5e60f9241688cad544007026736.png)

如果有错误，需要先把内存逐出策略设置为allkeys-lfu或者volatile-lfu，否则会返回错误。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1667197328004/bad34aec274f46aa88c00f3adc35f259.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1667197328004/e1ee50492b024688ab2aaa45ee49859b.png)

但是如果键值较多，执行较慢，和热点的概念的有点背道而驰，同时热度定义的不够准确。

#### 抓取TCP包发现

Redis客户端使用TCP协议与服务端进行交互，通信协议采用的是RESP。如果站在机器的角度，可以通过对机器上所有Redis端口的TCP数据包进行抓取完成热点key的统计

此种方法对于Redis客户端和服务端来说毫无侵入，是比较完美的方案，但是依然存在3个问题：

(1) 需要一定的开发成本

(2) 对于高流量的机器抓包，对机器网络可能会有干扰，同时抓包时候会有丢包的可能性。

(3) 维护成本过高。

对于成本问题，有一些开源方案实现了该功能，例如ELK(ElasticSearch Logstash Kibana)体系下的packetbeat[2] 插件，可以实现对Redis、MySQL等众多主流服务的数据包抓取、分析、报表展示

### 解决热点key

发现热点key之后，需要对热点key进行处理。

#### 使用二级缓存

可以使用 guava-cache或hcache，发现热点key之后，将这些热点key加载到JVM中作为本地缓存。访问这些key时直接从本地缓存获取即可，不会直接访问到redis层了，有效的保护了缓存服务器。

#### key分散

将热点key分散为多个子key，然后存储到缓存集群的不同机器上，这些子key对应的value都和热点key是一样的。当通过热点key去查询数据时，通过某种hash算法随机选择一个子key，然后再去访问缓存机器，将热点分散到了多个子key上。

## key冲突

### 什么是key冲突

所谓 Redis 的并发竞争 Key 的问题也就是多个系统同时对一个 key 进行操作，但是最后执行的顺序和我们期望的顺序不同，这样也就导致了结果的不同！(多个客户端操作同一个Key)

比如，有ABC三个系统

 A系统要把变量a赋值为1；

 B系统要把变量a赋值为2；

 C系统要把变量a赋值为3；

 本来我们期望顺序执行A > B > C后，a的值为3，但是如果并发太大，导致A晚了一步，让BC先执行了，最后a的值就成1了；

#### 解决方案：

首先设计规范：业务隔离及key设计

不同业务key区分开来，业务模块+系统名称+关键字 （比如core-pay-orderid）

##### 方案一：分布式锁+时间戳  

（zookeeper 和 redis 都可以实现分布式锁）。

（如果不存在 Redis 的并发竞争 Key问题，不要使用分布式锁，这样会影响性能）

基于zookeeper临时有序节点可以实现的分布式锁。大致思想为：每个客户端对某个方法加锁时，在zookeeper上的与该方法对应的指定节点的目录下，生成一个唯一的瞬时有序节点。

判断是否获取锁的方式很简单，只需要判断有序节点中序号最小的一个。

当释放锁的时候，只需将这个瞬时节点删除即可。同时，其可以避免服务宕机导致的锁无法释放，而产生的死锁问题。完成业务流程后，删除对应的子节点释放锁。在实践中，当然是从以可靠性为主。所以首推Zookeeper。

在操作a变量时候，额外维护一个时间戳，打个比方

 A在执行的时候时间是19:13:30

 B在执行的时候时间是19:13:33

 C在执行的时候时间是19:13:35

假如B先执行，B执行完后a变量对应的时间戳值应为19:13:33

 这时候A再来，发现当前时间是19:13:30，而a对应的时间戳为19:13:33，早于当前时间，说明在自己执行之前已经有其他系统操作过了，这时候就根据实际业务来决定怎么继续，废弃A操作或者轮询等；

##### 方案二：基于消息队列

 这种实现方式比较简单，是目前主流的解决方案

把所有操作写入同一个队列，利用消息队列把所有操作串行化

详细思路请移步MQ相关课程内容 如何保证消息顺序消费和避免消息重复消费





## 如何进一步提升缓存命中率

### 命中过程：

命中：可以直接通过缓存获取到需要的数据。

不命中：无法直接通过缓存获取到想要的数据，需要再次查询数据库或者执行其它的操作。原因可能是由于缓存中根本不存在，或者缓存已经过期。

通常来讲，缓存的命中率越高则表示使用缓存的收益越高，应用的性能越好（响应时间越短、吞吐量越高），抗并发的能力越强。

由此可见，在高并发的互联网系统中，缓存的命中率是至关重要的指标。

设置好缓存后，第一次访问的页面，会被缓存起来，是不会被命中的。

第二次被访问的时候，会先检查内存（为一级快速缓存）中有无这个页面，如果内存中无，会再检查硬盘上有无此页面，如果有此页面，还会检查此页面是否已经过期，如果在缓存周期内，会直接发送给用户；如果缓存周期已经到了，则检查源站是否有更新，如果无更新，则直接发送已经缓存的页面给用户；如果源站页面有更新，则会发送最新的页面给用户，并且将旧缓存给替换掉。

### 如何监控缓存的命中率

在memcached中，运行state命令可以查看memcached服务的状态信息，其中cmd_get表示总的get次数，get_hits表示get的总命中次数，命中率 = get_hits/cmd_get。

当然，我们也可以通过一些开源的第三方工具对整个memcached集群进行监控，显示会更直观。比较典型的包括：**zabbix\****、**MemAdmin等。

如图：MemAdmin对memcached服务的命中率情况的监控统计

同理，在redis中可以运行info命令查看redis服务的状态信息，其中keyspace_hits为总的命中中次数，keyspace_misses为总的miss次数，命中率=keyspace_hits/（keyspace_hits+keyspace_misses）。

开源工具Redis-star能以图表方式直观redis服务相关信息，同时，zabbix也提供了相关的插件对redis服务进行监控。

### 影响缓存命中率的常见因素

一般情况下，静态资源多的网站，例如图片站，资源站，下载站等一般命中率都较高，因为这些页面资源基本上都不太经常被修改（Modified），缓存后能不断被重复用（Re-used），命中率普遍在90%以上。相反的，动态资源较多的站点，命中率会低一些，因为这些动态页面经常被源站重新生成（Re-generated），而且设置的缓存周期普遍也很短，并且很多页面可能需要拒绝缓存才可以，会造成命中率会低一些。

如果缓存的单个文件大于1MB，则必须设置缓存，将空白分区当作缓存目录，原则上你的热门文件一共有多大，那么空白分区也需要有多大。

a. 缓存比例：就是用户频繁访问的页面，越频繁的页面越是如此，除去必须拒绝缓存的动态页面外，例如后台管理页面等等，其它的公共访问页面尽可能的都缓存起来。这样子第二次再访问相同页面的时候，如果缓存还在有效期内（即缓存还没有过期），就会被命中。

b. 缓存周期：一些不容易更新的页面资源等，缓存周期可以设置的长一点(例如视频缓存30天)，这样子用户第二次访问的时候，发现这些缓存，还在缓存周期之内，就会被命中

c. 用户访问量：用户访问量越大，一般命中率也会越高，举例，一个页面被缓存后，在缓存有效期内，被更多人访问到，也就是单位时间内被命中的次数更多，当然命中率会更高一些。相反的，如果一个页面被缓存后，很久都没有用户再一次的访问到这个页面，即使有访问到时，也已经是过期的页面了，那么就会降低命中率。

### 经验之谈：

从**架构设计**角度，需要应用尽可能的通过缓存直接获取数据，并避免缓存失效。这也是比较考验架构师能力的，需要在业务需求，缓存粒度，缓存策略，技术选型等各个方面去通盘考虑并做权衡。

尽可能的聚焦在高频访问且时效性要求不高的热点业务上，通过缓存预加载（预热）、增加存储容量、调整缓存粒度、更新缓存频率等手段来提高命中率。

对于时效性很高（或缓存空间有限），内容跨度很大（或访问很随机），并且访问量不高的应用来说缓存命中率可能长期很低，可能**预热**后的缓存还没来得被访问就已经过期了。



## Redis 常见性能问题和解决方案

### 1内存溢出问题

1、内存溢出问题 我们知道 Redis 的数据是存储在内存中的，如果数据量过大或者 Redis 存储的 key 较多，就容易引发内存溢出问题。当 Redis 内存占用率接近主机可用内存时，就可能会导致 Redis 运行变得缓慢或不可用。

#### 解决方案：

- 选择合适的 Redis 数据结构: 根据业务的需求，选择更加节约内存的数据结构来存储数据，例如使用哈希表或列表而避免用字符串或集合。
- 数据持久化：将 Redis 的数据定期或实时保存到磁盘上，从而释放一部分内存。采用 RDB 或 AOF 持久化机制，可以将 Redis 内存中的数据备份到磁盘中进行大规模数据恢复。
- 优化 Redis 配置参数：修改 Redis 的配置文件，调整 maxmemory 等内存相关的参数；取消最大使用空间限制（maxmemory），通过主机资源监控的方式，控制 Redis 的内存使用。

### 2、IO 瓶颈 

Redis 是 CPU 密集型应用，瓶颈常常在 I/O 上。较大的数据处理操作可能会阻塞 Redis 主线程，导致整个实例变慢或不可用。

#### 解决方案：

- 合理利用异步操作：对于耗时操作，如批量读取/写入 KV 数据、聚合等可以采用异步方式进行，以降低主线程的压力。
- 使用多线程架构：将 Redis 拆分为多个子进程，每个子进程负责一部分数据处理任务，避免某一个子进程阻塞造成整个 Redis 实例不可用。
- 优化 I/O 操作：调整内核参数最大 FD 数量，增加文件描述符数量，提高硬盘设备等级或使用更好的磁盘设备等，从而优化 I/O 操作。
- 主从尽量同一网段，采用线性结构不要网状

### 3、单线程性能限制

 Redis 是单线程应用，所有请求只能经过同一条路线进入主线程。所以，即使配置合理并使用了现代计算机的多核，Redis 在某些情况下仍会受到单线程的性能限制。

#### 解决方案：

- 集群分片：通过将 Redis 的 Key 分散到多个节点上，拆分负载，实现性能的横向扩展。
- 采用 Redis Proxy：Redis Proxy 即 Redis 代理，可以将请求流量水平拆分到多个 Redis 实例中的不同实例。在这种情况下，Redis 可以按照实例数量并行处理请求。
- 使用 Redis Cluster：Redis Cluster 为 Redis 提供了原生的分区和 sharding 功能，使其能够水平扩展。

### 4、频繁刷新 AOF 文件

 对于使用 AOF 持久化方式的 Redis 应用，在频繁写入数据时会导致 AOF 日志文件的写入次数增加，从而降低 Redis 的性能。

#### 解决方案：

- 选择合适的同步策略：针对 AOF 持久化方式提供的三种同步方式，按业务需求设定合理的同步策略，防止过度同步增加额外 I/O 操作
- 启用 No-appendfsync-on-rewrite 选项: 该选项可以避免在 BGREWRITEAOF 进程期间重写 AOF 日志文件时同时进行 fsync 磁盘操作，减轻 IO 冲击，并且可以避免出现许多正在执行 fsync 而导致延迟问题。
- 设置相关参数: 可以设置 AOF 缓存、内存控制参数等调整策略





- 一、一致性

- - 1、强一致性
  - 2、弱一致性
  - 3、最终一致性

- 二、redis缓存和mysql数据库数据一致性解决

- - 1、方案一：采用延时双删策略
  - 2、方案二：一步更新缓存（基于订阅Binlog的同步机制）

首先，我们先来看看有哪几种一致性的情况呢？



























## BigKey

### 什么是bigkey

bigkey是指key对应的value所占的内存空间比较大。

例如一个字符串类型的value可以最大存到512MB，一个列表类型的value最多可以存储23-1个元素。

如果按照数据结构来细分的话，一般分为字符串类型bigkey和非字符串类型bigkey。

字符串类型：体现在单个value值很大，一般认为超过10KB就是bigkey，但这个值和具体的OPS相关。

非字符串类型：哈希、列表、集合、有序集合,体现在元素个数过多。

bigkey无论是空间复杂度和时间复杂度都不太友好，下面我们将介绍它的危害。

### bigkey的危害

bigkey的危害体现在三个方面:

1、内存空间不均匀.(平衡):例如在Redis Cluster中，bigkey 会造成节点的内存空间使用不均匀。

2、超时阻塞:由于Redis单线程的特性，对元素较多的hash，list，zset做运算会耗时较久，也就意味着阻塞Redis可能性增大。

3、网络拥塞:每次获取bigkey产生的网络流量较大

4、CPU飙升: 对bigkey序列化反序列化导致CPU飙升

假设一个bigkey为1MB，每秒访问量为1000，那么每秒产生1000MB 的流量,对于普通的千兆网卡(按照字节算是128MB/s)的服务器来说简直是灭顶之灾，而且一般服务器会采用单机多实例的方式来部署,也就是说一个bigkey可能会对其他实例造成影响,其后果不堪设想。

bigkey的存在并不是完全致命的：

如果这个bigkey存在但是几乎不被访问,那么只有内存空间不均匀的问题存在,相对于另外两个问题没有那么重要紧急,但是如果bigkey是一个热点key(频繁访问)，那么其带来的危害不可想象,所以在实际开发和运维时一定要密切关注bigkey的存在。

#### 发现bigkey

redis-cli --bigkeys可以命令统计bigkey的分布。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1667197328004/4a07e97ad6f044e2a3454da671e1e832.png)

但是在生产环境中，开发和运维人员更希望自己可以定义bigkey的大小，而且更希望找到真正的bigkey都有哪些,这样才可以去定位、解决、优化问题。

判断一个key是否为bigkey，只需要执行debug object key查看serializedlength属性即可，它表示 key对应的value序列化之后的字节数。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1667197328004/31f5dc8498c942b1a7dc61920feee621.png)

如果是要遍历多个，则尽量不要使用keys的命令，可以使用scan的命令来减少压力。

##### scan

Redis 从2.8版本后，提供了一个新的命令scan，它能有效的解决keys命令存在的问题。和keys命令执行时会遍历所有键不同,scan采用渐进式遍历的方式来解决 keys命令可能带来的阻塞问题，但是要真正实现keys的功能,需要执行多次scan。可以想象成只扫描一个字典中的一部分键，直到将字典中的所有键遍历完毕。scan的使用方法如下:

```
scan cursor [match pattern] [count number]
```

cursor ：是必需参数，实际上cursor是一个游标，第一次遍历从0开始，每次scan遍历完都会返回当前游标的值,直到游标值为0,表示遍历结束。

Match pattern ：是可选参数,它的作用的是做模式的匹配,这点和keys的模式匹配很像。

Count number ：是可选参数,它的作用是表明每次要遍历的键个数,默认值是10,此参数可以适当增大。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1667197328004/10856b780ab34549a4e32b9a67c339de.png)

可以看到，第一次执行scan 0，返回结果分为两个部分:

第一个部分9就是下次scan需要的cursor

第二个部分是10个键。接下来继续

直到得到结果cursor变为0，说明所有的键已经被遍历过了。

除了scan 以外，Redis提供了面向哈希类型、集合类型、有序集合的扫描遍历命令，解决诸如hgetall、smembers、zrange可能产生的阻塞问题，对应的命令分别是hscan、sscan、zscan，它们的用法和scan基本类似，请自行参考Redis官网。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1667197328004/71181d9ff6164fc4ae671e901c19c44e.png)

渐进式遍历可以有效的解决keys命令可能产生的阻塞问题，但是scan并非完美无瑕，如果在scan 的过程中如果有键的变化(增加、删除、修改)，那么遍历效果可能会碰到如下问题:新增的键可能没有遍历到，遍历出了重复的键等情况，也就是说scan并不能保证完整的遍历出来所有的键，这些是我们在开发时需要考虑的。

如果键值个数比较多，scan + debug object会比较慢，可以利用Pipeline机制完成。对于元素个数较多的数据结构，debug object执行速度比较慢，存在阻塞Redis的可能，所以如果有从节点,可以考虑在从节点上执行。

#### 解决bigkey

主要思路

1:优雅的key结构设计

Redis的Key虽然可以自定义，但最好遵循下面的几个最佳实践约定：

- 遵循基本格式：[业务名称]:[数据名]:[id]

- 长度不超过44字节

- 不包含特殊字符

  例如：我们的登录业务，保存用户信息，其key可以设计成如下格式： login:user:138

  这样设计的好处：

  可读性强
  避免key冲突
  方便管理
  更节省内存： key是string类型，底层编码包含int、embstr和raw三种。embstr在小于44字节使用，采用连续内存空间，内存占用更小。当字节数大于44字节时，会转为raw模式存储，在raw模式下，内存空间不是连续的，而是采用一个指针指向了另外一段内存空间，在这段空间里存储SDS内容，这样空间不连续，访问的时候性能也就会收到影响，还有可能产生内存碎片


2:拆分

对 big key 存储的数据 （big value）进行拆分，变成value1，value2… valueN等等。

例如big value 是个大json 通过 mset 的方式，将这个 key 的内容打散到各个实例中，或者一个hash，每个field代表一个具体属性，通过hget、hmget获取部分value，hset、hmset来更新部分属性。

例如big value 是个大list，可以拆成将list拆成。= list_1， list_2, list3, ...listN

其他数据类型同理。



#### Redis阻塞分析：

##### 命令阻塞

使用不当的命令造成客户端阻塞：

- keys * ：获取所有的 key 操作；
- Hgetall：返回哈希表中所有的字段和；
- smembers：返回集合中的所有成员；

这些命令时间复杂度是O(n)，有时候也会全表扫描，随着n的增大耗时也会越大从而导致客户端阻塞。

##### SAVE 阻塞

大家都知道 Redis 在进行 RDB 快照的时候，会调用系统函数 fork() ，创建一个子线程来完成临时文件的写入，而触发条件正是配置文件中的 save 配置。

当达到我们的配置时，就会触发 bgsave 命令创建快照，这种方式是不会阻塞主线程的，而手动执行 save 命令会在主线程中执行，**阻塞**主线程。

##### 同步持久化

当 Redis 直接记录 AOF 日志时，如果有大量的写操作，并且配置为同步持久化

```
appendfsync always
```

即每次发生数据变更会被立即记录到磁盘，因为写磁盘比较耗时，性能较差，所以有时会阻塞主线程。

##### AOF 重写

- fork 出一条子线程来将文件重写，在执行 `BGREWRITEAOF` 命令时，Redis 服务器会维护一个 AOF 重写缓冲区，该缓冲区会在子线程创建新 AOF 文件期间，记录服务器执行的所有写命令。
- 当子线程完成创建新 AOF 文件的工作之后，服务器会将重写缓冲区中的所有内容追加到新 AOF 文件的末尾，使得新的 AOF 文件保存的数据库状态与现有的数据库状态一致。
- 最后，服务器用新的 AOF 文件替换旧的 AOF 文件，以此来完成 AOF 文件重写操作。

阻塞就是出现在第2步的过程中，将缓冲区中新数据写到新文件的过程中会产生**阻塞**。

##### AOF 日志

AOF 的日志记录不像关系型数据库那样在执行命令之前记录日志（方便故障恢复），而是采用先执行命令后记录日志的方式。

原因就是 AOF 记录日志是不会对命令进行语法检查的，这样就能减少额外的检查开销，不会对当前命令的执行产生阻塞，但可能会给下一个操作带来阻塞风险。

**这是因为 AOF 日志也是在主线程中执行的**，如果在把日志文件写入磁盘时，磁盘写压力大，就会导致写盘很慢，进而导致后续的操作也无法执行了。

##### 大 Key 问题

大 key 并不是指 key 的值很大，而是 key 对应的 value 很大。

大 key 造成的阻塞问题如下：

- 客户端超时阻塞：由于 Redis 执行命令是单线程处理，然后在操作大 key 时会比较耗时，那么就会阻塞 Redis，从客户端这一视角看，就是很久很久都没有响应。
- 引发网络阻塞：每次获取大 key 产生的网络流量较大，如果一个 key 的大小是 1 MB，每秒访问量为 1000，那么每秒会产生 1000MB 的流量，这对于普通千兆网卡的服务器来说是灾难性的。
- 阻塞工作线程：如果使用 del 删除大 key 时，会阻塞工作线程，这样就没办法处理后续的命令。

##### 查找大 key

当我们在使用 Redis 自带的 `--bigkeys` 参数查找大 key 时，最好选择在从节点上执行该命令，因为主节点上执行时，会**阻塞**主节点。

- 我们还可以使用 SCAN 命令来查找大 key；

- 为了识别大键，需要先确保 Redis 使用了 RDB 持久化，并分析对应的 RDB 文件。网上有现成的工具：

- - redis-rdb-tools：Python 语言写的用来分析 Redis 的 RDB 快照文件用的工具
  - 这个工具叫做 rdb_bigkeys，是由 Go 语言编写的，可用于分析 Redis 的 RDB 快照文件，具有更高的性能。

##### 删除大 key

删除操作的本质是要释放键值对占用的内存空间。

释放内存只是第一步，为了更加高效地管理内存空间，在应用程序释放内存时，**操作系统需要把释放掉的内存块插入一个空闲内存块的链表**，以便后续进行管理和再分配。这个过程本身需要一定时间，而且会**阻塞**当前释放内存的应用程序。

所以，如果一下子释放了大量内存，空闲内存块链表操作时间就会增加，相应地就会造成 Redis **主线程的阻塞**，如果主线程发生了阻塞，其他所有请求可能都会超时，超时越来越多，会造成 Redis 连接耗尽，产生各种异常。

> 删除大 key 时建议采用分批次删除和异步删除的方式进行。

##### 清空数据库

清空数据库和上面 bigkey 删除也是同样道理，flushdb、flushall 也涉及到删除和释放所有的键值对，也是 Redis 的阻塞点。 比如 主从同步，主从切换，数据迁移。

##### 集群扩容

Redis 集群可以进行节点的动态扩容缩容，这一过程目前还处于**半自动**状态，需要人工介入。

在扩缩容的时候，需要进行数据迁移。而 Redis 为了保证迁移的一致性，迁移所有操作都是**同步**操作。

执行迁移时，两端的 Redis 均会进入时长不等的**阻塞**状态，对于小Key，该时间可以忽略不计，但如果一旦 Key 的内存使用过大，严重的时候会接触发集群内的故障转移，造成不必要的切换





## 数据倾斜

数据倾斜其实分为访问量倾斜或者数据量倾斜:

1、hotkey出现造成集群访问量倾斜

2、bigkey 造成集群数据量倾斜

解决方案前面已经说过了，这里不再赘述。

## Redis脑裂

所谓的脑裂，就是指在有主从集群中，同时有两个主节点，它们都能接收写请求。而脑裂最直接的影响，就是客户端不知道应该往哪个主节点写入数据，结果就是不同的客户端会往不同的主节点上写入数据。而且，严重的话，脑裂会进一步导致数据丢失。

### 哨兵主从集群脑裂

现在假设：有三台服务器一台主服务器，两台从服务器，还有一个哨兵。

基于上边的环境，这时候网络环境发生了波动导致了sentinel没有能够心跳感知到master，但是哨兵与slave之间通讯正常。所以通过选举的方式提升了一个salve为新master。如果恰好此时server1仍然连接的是旧的master，而server2连接到了新的master上。数据就不一致了，哨兵恢复对老master节点的感知后，会将其降级为slave节点，然后从新maste同步数据（full resynchronization），导致脑裂期间老master写入的数据丢失。

而且基于setNX指令的分布式锁，可能会拿到相同的锁；基于incr生成的全局唯一id，也可能出现重复。通过配置参数

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1667197328004/0c9b32eac02a454db35ffdbf7c859330.png)

```
min-replicas-to-write 2
min-replicas-max-lag 10
```

第一个参数表示最少的salve节点为2个

第二个参数表示数据复制和同步的延迟不能超过10秒

配置了这两个参数：如果发生脑裂：原master会在客户端写入操作的时候拒绝请求。这样可以避免大量数据丢失。

### 集群脑裂

Redis集群的脑裂一般是不存在的，因为Redis集群中存在着过半选举机制，而且当集群16384个槽任何一个没有指派到节点时整个集群不可用。所以我们在构建Redis集群时，应该让集群 Master 节点个数最少为 3 个，且集群可用节点个数为奇数。

不过脑裂问题不是是可以完全避免，只要是分布式系统，必然就会一定的几率出现这个问题，CAP的理论就决定了。

## 多级缓存实例

一个使用了Redis集群和其他多种缓存技术的应用系统架构如图

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1667197328004/2067af27ef2a423e9034ca5130382166.png)

首先，用户的请求被负载均衡服务分发到Nginx上，此处常用的负载均衡算法是轮询或者一致性哈希，轮询可以使服务器的请求更加均衡，而一致性哈希可以提升Nginx应用的缓存命中率。

接着，Nginx应用服务器读取本地缓存，实现本地缓存的方式可以是Lua Shared Dict,或者面向磁盘或内存的Nginx Proxy Cache，以及本地的Redis实现等，如果本地缓存命中则直接返回。Nginx应用服务器使用本地缓存可以提升整体的吞吐量，降低后端的压力，尤其应对热点数据的反复读取问题非常有效。

如果Nginx应用服务器的本地缓存没有命中，就会进一步读取相应的分布式缓存——Redis分布式缓存的集群，可以考虑使用主从架构来提升性能和吞吐量，如果分布式缓存命中则直接返回相应数据，并回写到Nginx应用服务器的本地缓存中。

如果Redis分布式缓存也没有命中，则会回源到Tomcat集群，在回源到Tomcat集群时也可以使用轮询和一致性哈希作为负载均衡算法。当然，如果Redis分布式缓存没有命中的话，Nginx应用服务器还可以再尝试一次读主Redis集群操作，目的是防止当从 Redis集群有问题时可能发生的流量冲击。

在Tomcat集群应用中，首先读取本地平台级缓存，如果平台级缓存命中则直接返回数据，并会同步写到主Redis集群，然后再同步到从Redis集群。此处可能存在多个Tomcat实例同时写主Redis集群的情况，可能会造成数据错乱，需要注意缓存的更新机制和原子化操作。

如果所有缓存都没有命中，系统就只能查询数据库或其他相关服务获取相关数据并返回，当然,我们已经知道数据库也是有缓存的。

整体来看，这是一个使用了多级缓存的系统。Nginx应用服务器的本地缓存解决了热点数据的缓存问题，Redis分布式缓存集群减少了访问回源率，Tomcat应用集群使用的平台级缓存防止了相关缓存失效崩溃之后的冲击，数据库缓存提升数据库查询时的效率。正是多级缓存的使用，才能保障系统具备优良的性能。

### 热点数据的处理

#### 1、概念

热点数据就是访问量特别大的数据。

- 1.在产品分析时进行预测，在代码开发中进行缓存处理

- 2.收集用户行为日志，及访问数据日志信息。进行分析热点数据

- 3.对于存储在redis中的数据，进行访问排序定位缓存中的热点数据

  要跟你实际的实际的公司项目集合起来说，比如电商的热点商品，论坛的热点文章，微博的热点新闻，打车的热点区域等等

#### 2、热点数据引起的问题

流量集中，达到物理网卡上限。

请求过多，缓存分片服务被打垮。redis作为一个单线程的结构，所有的请求到来后都会去排队，当请求量远大于自身处理能力时，后面的请求会陷入等待、超时。根本原因在于读，不在写。

redis崩溃或热点数据过期，会有大量数据访问DB，造成DB崩溃，引起业务雪崩。

![img](https://img-blog.csdnimg.cn/img_convert/65b44ceb0a499cc788f37ff1dec8dde6.png)


如上图，hot key即为热点数据,hot key的访问频次远大于其他key的使用。

#### 3、如何排查热点key

3.1 排查标准
以redis访问key为例，我们可以很容易的计算出性能指标，譬如有1000台服务器，某key所在的redis集群能支撑20万/s的访问，那么平均每台机器每秒大概能访问该key200次，超过的部分就会进入等待。由于redis的瓶颈，将极大地限制server的性能。

而如果该key是在本地内存中，读取一个内存中的值，每秒多少个万次都是很正常的，不存在任何数据层的瓶颈。当然，如果通过增加redis集群规模的形式，也能提升数据的访问上限，但问题是事先不知道热key在哪里，而全量增加redis的规模，带来的成本提升又不可接受。

3.2 排查方法
3.2.1 凭借业务经验，进行预估哪些是热key
其实这个方法还是挺有可行性的。比如某商品在做秒杀，那这个商品的key就可以判断出是热key。缺点很明显，并非所有业务都能预估出哪些key是热key。

3.2.2 在客户端进行收集
这个方式就是在操作redis之前，加入一行代码进行数据统计。那么这个数据统计的方式有很多种，也可以是给外部的通讯系统发送一个通知信息。缺点就是对客户端代码造成入侵。

3.2.3 在Proxy层做收集
有些集群架构是下面这样的，Proxy可以是Twemproxy，是统一的入口。可以在Proxy层做收集上报，但是缺点很明显，并非所有的redis集群架构都有proxy。

3.2.4 用redis自带命令
(1)monitor命令，该命令可以实时抓取出redis服务器接收到的命令，然后写代码统计出热key是啥。当然，也有现成的分析工具可以给你使用，比如redis-faina。但是该命令在高并发的条件下，有内存增暴增的隐患，还会降低redis的性能。

(2)hotkeys参数，redis 4.0.3提供了redis-cli的热点key发现功能，执行redis-cli时加上–hotkeys选项即可。但是该参数在执行的时候，如果key比较多，执行起来比较慢。

3.2.5自己抓包评估
Redis客户端使用TCP协议与服务端进行交互，通信协议采用的是RESP。自己写程序监听端口，按照RESP协议规则解析数据，进行分析。缺点就是开发成本高，维护困难，有丢包可能性。

3.3 排查工具
可以使用京东的开源工具hotkey：https://gitee.com/jd-platform-opensource/hotkey

#### 4、解决方案

4.1 二级缓存
利用ehcache或者HashMap或者guava cache都可以。在你发现热key以后，把热key加载到系统的JVM中。

针对这种热key请求，会直接从jvm中取，而不会走到redis层。假设此时有十万个针对同一个key的请求过来,如果没有本地缓存，这十万个请求就直接怼到同一台redis上了。

现在假设，你的应用层有50台机器，OK，你也有jvm缓存了。这十万个请求平均分散开来，每个机器有2000个请求，会从JVM中取到value值，然后返回数据。避免了十万个请求怼到同一台redis上的情形。

优点
读取速度快。

只需要改读取逻辑，不需要改写逻辑。

缺点
需要提前获知热点

缓存容量有限

不一致性时间增长

热点 Key 遗漏

4.2 增加数据副本
既然热点问题是因为某个key被大量访问导致的，那我们将这个Key的请求做下拆分不就行了。

假如hotkey的缓存是一个高频访问的数据，那么大量请求访问这个key时，就会出现压力都有redis server2这个节点来承担，这样redis server2节点就有可能会扛不住压力而罢工了。那么应该怎么解决这个问题呢？

不妨在缓存数据的时候，将这个数据在每个redis节点都缓存一份。而在缓存的时候，将key在程序层面进行加工，如变成hotkey#redis server1、hotkey#redis server2...hotkey#redis server6这样的6个key。此处我们假如这样的6个key会根据crc16算法，将这个6个key分别落在这6个节点之上。那么这样在访问的时候，我们就可以依然遵循这个规则获得一个key，这样一来，获取数据的时候，压力就被分散到不同的redis节点上了。

优点
可扩展，最高访问量和副本成正比。

缺点
每增加一个redis节点都会增加成本。

写入增加逻辑，读也要增加逻辑。

# 互联网大厂中的Redis

经过几年演进，携程金融形成了自顶向下的多层次系统架构，如业务层、平台层、基础服务层等，其中用户信息、产品信息、订单信息等基础数据由基础平台等底层系统产生，服务于所有的金融系统，对这部分基础数据我们引入了统一的缓存服务（系统名utag）。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1667197328004/10fbd6aa05c54c078f10268e30f790e7.png)

缓存数据有三大特点：全量、准实时、永久有效，在数据实时性要求不高的场景下，业务系统可直接调用统一的缓存查询接口。

在构建此统一缓存服务时候，有三个关键目标：

数据准确性：DB中单条数据的更新一定要准确同步到缓存服务。

数据完整性：将对应DB表的全量数据进行缓存且永久有效，从而可以替代对应的DB查询。

系统可用性：我们多个产品线的多个核心服务都已经接入，utag的高可用性显得尤为关键。

## 整体方案

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1667197328004/b4a9d749cc2e41fdaab3ed911a91a207.png)

系统在多地都有部署，故缓存服务也做了相应的异地多机房部署，一来可以让不同地区的服务调用本地区服务，无需跨越网络专线，二来也可以作为一种灾备方案，增加可用性。

对于缓存的写入，由于缓存服务是独立部署的，因此需要感知业务DB数据变更然后触发缓存的更新，本着“可以多次更新，但不能漏更新”的原则，设计了多种数据更新触发源：定时任务扫描，业务系统MQ、binlog变更MQ，相互之间作为互补来保证数据不会漏更新。

对于MQ使用携程开源消息中间件QMQ 和 Kafka，在公司内部QMQ和Kafka也做了异地机房的互通。

使用MQ来驱动多地多机房的缓存更新，在不同的触发源触发后，会查询最新的DB数据，然后发出一个缓存更新的MQ消息，不同地区机房的缓存系统同时监听该主题并各自进行缓存的更新。

对于缓存的读取，utag系统提供dubbo协议的缓存查询接口，业务系统可调用本地区的接口，省去了网络专线的耗时（50ms延迟）。在utag内部查询redis数据，并反序列化为对应的业务model，再通过接口返回给业务方。

## 数据准确性

不同的触发源，对缓存更新过程是一样的，整个更新步骤可抽象为4步：

step1：触发更新，查询DB中的新数据，并发送统一的MQ

step2：接收MQ，查询缓存中的老数据

step3：新老数据对比，判断是否需要更新

step4：若需要，则更新缓存

### 并发控制

若一条DB数据出现了多次更新，且刚好被不同的触发源触发，更新缓存时候若未加控制，可能出现数据更新错乱，如下图所示：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1667197328004/c832d47be75d4a099e317527646fe5da.png)

故需要将第2、3、4步加锁，使得缓存刷新操作全部串行化。由于utag本身就依赖了redis，此处我们的分布式锁就基于redis实现。

### 基于updateTime的更新顺序控制

即使加了锁，也需要进一步判断当前db数据与缓存数据的新老，因为到达缓存更新流程的顺序并不代表数据的真正更新顺序。我们通过对比新老数据的更新时间来实现数据更新顺序的控制。若新数据的更新时间大于老数据的更新时间，则认为当前数据可以直接写入缓存。

我们系统从建立之初就有自己的MySQL规范，每张表都必须有update_time字段，且设置为ON
UPDATE CURRENT_TIMESTAMP，但是并没有约束时间字段的精度，大部分都是秒级别的，因此在同一秒内的多次更新操作就无法识别出数据的新老。

针对同一秒数据的更新策略我们采用的方案是：先进行数据对比，若当前数据与缓存数据不相等，则直接更新，并且发送一条延迟消息，延迟1秒后再次触发更新流程。

举个例子：假设同一秒内同一条数据出现了两次更新，value=1和value=2，期望最终缓存中的数据是value=2。若这两次更新后的数据被先后触发，分两种情况：

case1：若value=1先更新，value=2后更新，（两者都可更新到缓存中，因为虽然是同一秒，但是值不相等）则缓存中最终数据为value=2。

case2：若value=2先更新，value=1后更新，则第一轮更新后缓存数据为value=1，不是期望数据，之后对比发现是同一秒数据后会通过消息触发二次更新，重新查询DB数据为value=2，可以更新到缓存中。如下图所示：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1667197328004/b15ba38e63db48e2b18e721961404e18.png)

## 数据完整性设计

上述数据准确性是从单条数据更新角度的设计，而我们构建缓存服务的目的是替代对应DB表的查询，因此需要缓存对应DB表的全量数据，而数据的完整性从以下三个方面得到保证：

（1）“把鸡蛋放到多个篮子里”，使用多种触发源（定时任务，业务MQ，binglog MQ）来最大限度降低单条数据更新缺失的可能性。

单一触发源有可能出现问题，比如消息类的触发依赖业务系统、中间件canel、中间件QMQ和Kafka，扫表任务依赖分布式调度平台、MySQL等。中间任何一环都可能出现问题，而这些中间服务同时出概率的可能相对来说就极小了，相互之间可以作为互补。

（2）全量数据刷新任务：全表扫描定时任务，每周执行一次来进行兜底，确保缓存数据的全量准确同步。

（3）数据校验任务：监控Redis和DB数据是否同步并进行补偿。





# 缓存策略分析

看到很多小伙伴简历上写了“熟练使用缓存”，但是被我问到“**缓存常用的读写策略**”的时候却一脸懵逼。

造成这个问题的原因是我们在学习 Redis 的时候，可能只是简单了写一些 Demo，并没有去关注缓存的读写策略，或者说压根不知道这回事。

但是，搞懂 这几种常见的缓存读写策略对于实际工作中使用缓存以及面试中被问到缓存都是非常有帮助的！



缓存更新是指在数据发生变化时，保持缓存和数据库的数据一致性的问题。如果缓存和数据库的数据不一致，会导致用户看到过期或者错误的数据，影响业务逻辑和用户体验。

为了实现缓存更新，我们可以采用以下四种方式：

`Cache Aside`、`Read/Write Through` 、`Write Behind Caching` 、`Refresh-Ahead`。

另外，**这四 种缓存读写策略各有优劣，不存在最佳，需要我们根据具体的业务场景选择更适合的。**



### 一：Cache Aside Pattern（旁路缓存模式）

**Cache Aside Pattern 是我们平时使用比较多的一个缓存读写模式，我们可以通过异步的方式在旁路处理缓存。比较适合读请求比较多的场景。**

Cache Aside Pattern 中服务端需要同时维系 DB 和 cache，并且是以 DB 的结果为准。

下面我们来看一下这个策略模式下的缓存读写步骤。

**写** ：

- 先更新 DB
- 然后直接删除 cache 。

简单画了一张图帮助大家理解写的步骤。

![640](E:\图灵课堂\Redis专题\笔记\1、Redis专题.assets\640.png)

**读** :

- 从 cache 中读取数据，读取到就直接返回
- cache 中读取不到的话，就从 DB 中读取数据返回
- 再把数据放到 cache 中。

简单画了一张图帮助大家理解读的步骤。

![图片](E:\图灵课堂\Redis专题\笔记\1、Redis专题.assets\640 (2).png)

你仅仅了解了上面这些内容的话是远远不够的，我们还要搞懂其中的原理。

比如说面试官很可能会追问：“**在写数据的过程中，可以先删除 cache ，后更新 DB 么？**”

**答案：** 那肯定是不行的！因为这样可能会造成**数据库（DB）和缓存（Cache）数据不一致**的问题。为什么呢？比如说请求 1 先写数据 A，请求 2 随后读数据 A 的话就很有可能产生数据不一致性的问题。这个过程可以简单描述为：

> 请求 1 先把 cache 中的 A 数据删除 -> 请求 2 从 DB 中读取数据->请求 1 再把 DB 中的 A 数据更新。

当你这样回答之后，面试官可能会紧接着就追问：“**在写数据的过程中，先更新 DB，后删除 cache 就没有问题了么？**”

**答案**：理论上来说还是可能会出现数据不一致性的问题，不过概率非常小，因为缓存的写入速度是比数据库的写入速度快很多！

比如请求 1 先读数据 A，请求 2 随后写数据 A，并且数据 A 不在缓存中的话也有可能产生数据不一致性的问题。这个过程可以简单描述为：

> 请求 1 从 DB 读数据 A->请求 2 写更新数据 A 到数据库并把删除 cache 中的 A 数据->请求 1 将数据 A 写入 cache。

现在我们再来分析一下 **Cache Aside Pattern 的缺陷**。

**缺陷 1：首次请求数据一定不存在 cache 的问题**

解决办法：可以将热点数据可以提前放入 cache 中。

**缺陷 2：写操作比较频繁的话导致 cache 中的数据会被频繁被删除，这样会影响缓存命中率 。**

解决办法：

- 数据库和缓存数据强一致场景 ：更新 DB 的时候同样更新 cache，不过我们需要加一个锁/分布式锁来保证更新 cache 的时候不存在线程安全问题。

- 可以短暂地允许数据库和缓存数据不一致的场景 ：更新 DB 的时候同样更新 cache，但是给缓存加一个比较短的过期时间，这样的话就可以保证即使数据不一致的话影响也比较小。

  

这种策略简单易用，但是需要维护缓存和数据库的一致性，可能出现[缓存穿透](https://so.csdn.net/so/search?q=缓存穿透&spm=1001.2101.3001.7020)或缓存雪崩的问题，一般采用**延迟双删**来保证最终一致性

**延迟双删**

延迟双删是一种保证数据一致性的常用策略，它的基本思想是在更新数据库后，先删除缓存，然后等待一段时间，再次删除缓存。这样做的目的是为了防止在数据库和缓存主从同步的过程中，有其他请求查询到旧的缓存数据，并写回到缓存中，具体的流程如下：

1. 更新数据库数据
2. 删除缓存数据
3. 休眠一段时间，时间依据数据的读取耗费的时间而定。
4. 再次删除缓存数据

**延迟双删的休眠时间**是根据业务读取数据平均耗时来设置的，目的是确保读请求可以结束，写请求可以删除读请求造成的脏数据的问题。一般来说，休眠时间可以设置为**500毫秒**左右，但具体还要**根据实际情况调整**。休眠时间设置过长会影响性能和实时性，设置过短会导致数据不一致的风险。

延迟双删的优点是简单易实现，能够提高数据的**最终一致性**。但是延迟双删的**缺点**也非常明显：

- 延迟双删**不是强一致性**，有**等待环节**，如果系统要求低延时，这种场景就不合适了
- 延迟双删不适合“秒杀”这种**频繁修改数据和要求数据强一致**的场景
- 延迟双删的延时时间是一个预估值，不能确保数据库和redis在这个时间段内都实时同步或持久化成功了
- 延迟双删不能完全避免redis存在脏数据的问题，只能减轻这个问题，要想彻底解决，还需要用到**同步锁**解决

### 二：Read/Write Through Pattern（读写穿透）

由于上面的旁路缓存模式有一定概率出现数据不一致的情况，即便采用了延迟双删也有不少缺点。倒不如进行写入操作时，直接将结果写入到缓存。

Read/Write Through Pattern 中服务端把 cache 视为主要数据存储（调用方只和缓存打交道），从中读取数据并将数据写入其中。cache 服务负责将此数据读取和写入 DB，从而减轻了应用程序的职责。

这种缓存读写策略小伙伴们应该也发现了在平时在开发过程中非常少见。抛去性能方面的影响，大概率是因为我们经常使用的分布式缓存 Redis 并没有提供 cache 将数据写入 DB 的功能。

核心策略:以缓存为操作为主,数据存先存在于缓存,缓存的数据是不会过期的.

**写（Write Through）：**

- 先查 cache，cache 中不存在，直接更新 DB。
- cache 中存在，则先更新 cache，然后 cache 服务自己更新 DB（**同步更新 cache 和 DB**）。

简单画了一张图帮助大家理解写的步骤。

![图片](E:\图灵课堂\Redis专题\笔记\1、Redis专题.assets\640 (3).png)

**读(Read Through)：**

- 从 cache 中读取数据，读取到就直接返回 。
- 读取不到的话，先从 DB 加载，写入到 cache 后返回响应。

简单画了一张图帮助大家理解读的步骤。

![图片](E:\图灵课堂\Redis专题\笔记\1、Redis专题.assets\640 (4).png)

#### Read-Through Pattern 和 Cache-Aside Pattern 比较

Read-Through Pattern 实际只是在 Cache-Aside Pattern 之上进行了封装。

在 Cache-Aside Pattern 下，发生**读请求**的时候，如果 cache 中不存在对应的数据，是由客户端自己负责把数据写入 cache，

​		而 Read Through Pattern 则是 cache 服务自己来写入缓存的，这对客户端是透明的。

和 Cache Aside Pattern 一样， Read-Through Pattern 也有首次请求数据一定不在 cache 的问题，对于热点数据可以提前放入缓存中。

读写穿透模式以缓存为主，旁路缓存模式以数据库为主。

通常read Through Pattern 和 write Through Pattern 两种模式会配合使用。





### 三：Write Behind Pattern（异步缓存写入）

Write Behind Pattern 和 Read/Write Through Pattern 很相似，两者都是由 cache 服务来负责 cache 和 DB 的读写。

但是，两个又有很大的不同：**Read/Write Through 是同步更新 cache 和 DB，而 Write Behind Caching 则是只更新缓存，不直接更新 DB，而是改为异步批量的方式来更新 DB。**

很明显，这种方式对数据一致性带来了更大的挑战，比如 cache 数据可能还没异步更新 DB 的话，cache 服务可能就挂掉了。

这种策略在我们平时开发过程中也**非常少见**，但是不代表它的应用场景少，比如消息队列中消息的异步写入磁盘、MySQL 的 InnoDB Buffer Pool 机制都用到了这种策略。

Write Behind Pattern 下 DB 的写性能非常高，非常适合一些数据经常变化又对数据一致性要求没那么高的场景，比如浏览量、点赞量。

Write back是相较于Write Through而言的一种异步回写策略.
异步写可以减少与物理磁盘存储的交互,也可以进行合并写等优化.

### 四：Refresh-Ahead（提前刷新）

是在过期之前刷新缓存数据，该方法适用于热数据，即预计在不久的将来会被请求的数据。

是指在读取数据时，**如果缓存中的数据即将过期**，则由**后台线程或服务**自动从数据库中查询最新的数据，并将数据写入缓存中，然后返回给应用程序。**不同于以上三种**，应用程序**无需等待**数据的刷新，也无需自己去触发数据的刷新，而是交由**后台线程或服务**来完成这些操作。其中`后台线程或服务`的实现通常是使用**CDC**模式去实现的

Refresh-Ahead 模式的工作原理如下：

- 当客户端访问缓存中的某个数据时，首先检查该数据**是否即将过期**，如果是，则启动一个**后台线程或服务**去从数据库中获取最新的数据，并**替换**掉缓存中的旧数据；同时返回给客户端
- 如果该数据还**没有即将过期**，则直接返回给客户端
- 如果该数据项**已经过期**，则从数据库中**获取最新的数据**，并**替换**掉缓存中的旧数据，并返回给客户端新数据



#### 总结：

| 策略                 | 性能 | 一致性 | 冗余数据 | 代码复杂度 | 业务逻辑 | 可靠性 |
| :------------------- | :--- | :----- | :------- | :--------- | :------- | :----- |
| Cache Aside          | 较高 | 较低   | 较少     | 较高       | 较复杂   | 较低   |
| Read/Write Through   | 较低 | 最高   | 较多     | 最高       | 最简单   | 最高   |
| Write Behind Caching | 最高 | 最低   | 较少     | 较低       | 较简单   | 较高   |
| Refresh-Ahead        | 次高 | 次高   | 较多     | 最高       | 较复杂   | 最高   |

#### 性能

- `Cache Aside` 的性能较高，它**只在缓存未命中**时才访问数据库
- `Read/Write Through` 的性能较低，它在**每次读写时都需要访问数据库**
- `Write Behind Caching` 的性能最高，它**只在缓存未命中**时才访问数据库，而**写入操作是异步的**
- `Refresh-Ahead` 的性能介于 `Cache Aside` 和 `Write Behind Caching` 之间，它**只在即将过期时**才访问数据库，并且**写入操作也是异步的**

#### 数据一致性

- `Cache Aside` 的数据一致性较低，它**只在缓存未命中时**才更新缓存，而写入操作则是**直接更新数据库**，**并将缓存中的数据删除或更新**
- `Read/Write Through` 的数据一致性最高，它在**每次读写时都**更新数据库和缓存
- `Write Behind Caching` 的数据一致性最低，它**只在缓存未命中**时才更新缓存，而写入操作则是先更新缓存，并在**异步更新数据库**，有较大的延迟。
- `Refresh-Ahead` 的数据一致性介于 `Read/Write Through` 和 `Cache Aside` 之间，它保证了缓存中的数据**总是最新**的，但是有一定的延迟

#### 冗余数据

- `Cache Aside` 的冗余数据较少，它**只将经常访问的数据**保存到缓存中
- `Read/Write Through` 的冗余数据较多，它需要将数据库的**所有数据**都保存到缓存中
- `Write Behind Caching` 的冗余数据与 `Cache Aside` 相同，因为它也**只将经常访问的数据**保存到缓存中
- `Refresh-Ahead` 的冗余数据与 `Read/Write Through` 相同，它也需要将数据库的**所有数据**都保存到缓存中

#### 代码复杂度

- `Cache Aside` 的代码复杂度较高，它需要**同时与缓存和数据库**交互，并处理可能出现的异常情况
- `Read/Write Through` 的代码复杂度最高，它需要实现数据库的读写接口
- `Write Behind Caching` 的代码复杂度较低，它只需要实现**简单的缓存操作**，并在异步执行数据库写入操作
- `Refresh-Ahead` 的代码复杂度与 `Read/Write Through` 相同，他它需要实现数据库的读写接口（关于这点可以使用Debezium）

#### 业务逻辑

- `Cache Aside` 的业务逻辑较复杂，它需要**同时与缓存和数据库**交互，且返回的数据是最新的
- `Read/Write Through `的业务逻辑最简单，它**只与缓存**交互，且返回的数据是最新的
- `Write Behind Caching` 的业务逻辑较简单，它也**只与缓存**交互，且返回的数据是最新的，由于是异步更新，所以比`Read/Write Through`要复杂一些
- `Refresh-Ahead `的业务逻辑较复杂，它会**同时与缓存和数据库**交互，需要处理可能出现的**异常情况**，且返回的数据有可能是**旧的**，也有可能是**新的**（关于这点也可以使用Debezium）

#### 可靠性

- `Cache Aside` 的可靠性较低，因为它将缓存作为数据库的**辅助层**

- `Read/Write Through` 的可靠性最高，因为它将缓存作为数据库的**代理层**

- `Write Behind Caching` 的可靠性较高，因为它将缓存作为数据库**前置层**

- `Refresh-Ahead` 的可靠性与 `Read/Write Through` 相同，因为它也将缓存作为数据库的**代理层**

  

从一个数据存储按需加载数据到一个缓存中。该模式能够提高性能并且也有助于维护缓存中的数据和底层数据存储中的数据之间的一致性。

#### 场景

应用程序使用某个缓存来改进重复访问数据存储的信息的过程。然而，期望缓存的数据总是会和数据存储中的数据完全一致是不切合实际的。应用程序应该采取某种策略来帮助确保缓存中的数据尽可能是最新的，但是也能够发现和处理当缓存中的数据变的过时（失效）的情况。

#### 解决方案

很多商业缓存系统提供穿透读（read-through）和穿透写（write-through）/延迟写（write-behind）等操作。在这些系统中，应用程序通过引用缓存来检索数据。如果数据不在缓存中，它从数据存储中检索并且将数据加入到缓存中去。同样地，任何对于缓存中的数据的修改都会自动地回写到数据存储中。

应用程序能够，通过采取预留缓存（cache-aside）策略，模仿穿透读缓存（read-through caching）的功能。该策略按需加载数据到缓存中。下图展示了使用预留缓存模式在缓存中存储数据：

![img](https:////upload-images.jianshu.io/upload_images/6688932-e4aec19dfa68540c.png?imageMogr2/auto-orient/strip|imageView2/2/w/475/format/webp)

如果应用程序更新信息，可以通过修改数据存储以及使缓存中相关数据失效来遵循写穿透（write-through）策略。

当该数据项紧接着被需要时，使用预留缓存（cache-aside）策略会导致从数据存储中检索已更新的数据并且添加回缓存中。

![20200806194359875](E:\图灵课堂\Redis专题\笔记\1、Redis专题.assets\20200806194359875.png)

#### 考虑点

**缓存数据的生命周期。**很多缓存实现了某种过期策略，如果数据在特定时间段没有被访问，那么缓存会使这些数据失效并且从缓存中移除这些数据。为了高效化预留缓存，确保过期策略匹配应用程序访问数据的模式。不要让过期时间太短，因为这样会导致应用程序持续不断地从数据存储中检索数据并且添加数据到缓存中。同样地，不能让过期时间太长以至于缓存的数据可能失效了。需要注意的是缓存对于相对静态的数据或频繁读取的数据最有效率。

**缓存数据的清除。**大多数缓存相对于数据存储都有固定的大小，并且如果必要，它们会清除数据。大多数缓存采用一种最近最少使用策略来选取清除的数据项，但是该策略也是能够配置的。配置全局过期属性和其它的缓存属性，以及每个缓存项的过期属性，来确保缓存是成本有效的。在缓存中，针对每个项应用某种全局清除策略不总是合适的。例如，如果某个缓存项从数据存储中检索出来代价很大，以牺牲那些频繁访问但检索成本低的数据项来保持该缓存项在缓存中是有好处的。

**缓存数据的准备。**很多解决方案在程序启动时，用应用程序可能需要的数据预填充缓存。如果其中某些数据过期了或被移除了，预留缓存模式也能够很有用，

**一致性。**采用预留缓存模式不能保证数据存储和缓存之间的一致性。数据存储中某个数据项可以在任何时候被外部进程改变，并且该改变可能不会反映在缓存中，直到下一次加载该数据项。在多个数据存储之间复制数据的系统中，如果同步很频繁，该问题会变得很严重。

**本地（内存中）缓存。**对于应用程序实例来说，缓存可以是本地的并且存储在内存中。如果应用程序重复不断地访问相同的数据，在这种环境下预留缓存模式会很有用。然而，本地缓存是私有的，以至于不同的应用程序实例都会有相同缓存数据的副本。这些数据会很快的在缓存之间变得不一致，因此在私有缓存中频繁地过期和刷新数据是必要的。在这些场景下，考虑使用共享或分布式缓存机制。

#### 什么时候使用？

适合：

> 缓存不提供原生的读穿透和写穿透操作。
>
> 资源的需求是不可预测的。该模式允许应用程序按需加载数据。不需要去假设应用程序会优先需要哪些数据。

不适合：

> 当缓存的数据集是静态的。如果数据可以刚好放入可用的缓存空间中，在启动的时候用这些数据来准备缓存，并且采用某个策略来防止这些数据失效。
>
> 对于在网络群中的web应用程序中缓存session状态信息。在该环境中，应该避免引入基于客户端-服务端连接关系的依赖。



# Redis的设计、实现

## 数据结构和内部编码

type命令实际返回的就是当前键的数据结构类型，它们分别是：string(字符串)hash(哈希)、list(列表)、set(集合)、zset (有序集合)，但这些只是Redis对外的数据结构。

实际上每种数据结构都有自己底层的内部编码实现，而且是多种实现，这样Redis会在合适的场景选择合适的内部编码。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1668158598097/d65322ec553941ba802899e1e0071d91.png)

每种数据结构都有两种以上的内部编码实现，例如list数据结构包含了linkedlist和ziplist两种内部编码。同时有些内部编码，例如ziplist,可以作为多种外部数据结构的内部实现，可以通过object encoding命令查询内部编码。

Redis这样设计有两个好处:

第一，可以改进内部编码，而对外的数据结构和命令没有影响，这样一旦开发出更优秀的内部编码，无需改动外部数据结构和命令，例如Redis3.2提供了quicklist，结合了ziplist和linkedlist两者的优势，为列表类型提供了一种更为优秀的内部编码实现，而对外部用户来说基本感知不到。

第二，多种内部编码实现可以在不同场景下发挥各自的优势，例如ziplist比较节省内存，但是在列表元素比较多的情况下，性能会有所下降，这时候Redis会根据配置选项将列表类型的内部实现转换为linkedlist。

### redisobject对象

Redis存储的所有值对象在内部定义为redisobject结构体，内部结构如图所示。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1668158598097/455898bf4c9b421c9bbcaa09ff7df7f8.png)

Redis存储的数据都使用redis0bject来封装，包括string、hash、list、set,zset在内的所有数据类型。理解redis0bject对内存优化非常有帮助，下面针对每个字段做详细说明:

#### type字段

type字段:表示当前对象使用的数据类型，Redis主要支持5种数据类型:string, hash、 list,set,zset。可以使用type { key}命令查看对象所属类型,type命令返回的是值对象类型,键都是string类型。

#### encoding字段

**encoding** **字段** :表示Redis内部编码类型,encoding在 Redis内部使用，代表当前对象内部采用哪种数据结构实现。理解Redis内部编码方式对于优化内存非常重要,同一个对象采用不同的编码实现内存占用存在明显差异。

#### lru字段

lru字段:记录对象最后次被访问的时间,当配置了maxmemory和maxmemory-policy=volatile-lru或者allkeys-lru时，用于辅助LRU算法删除键数据。可以使用object idletime {key}命令在不更新lru字段情况下查看当前键的空闲时间。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1668158598097/06878d251efb42fb95c55ea847c28cfe.png)

*可以使用scan +object idletime* *命令批量查询哪些键长时间未被访问，找出长时间不访问的键进行清理,* *可降低内存占用。*

#### refcount字段

refcount字段:记录当前对象被引用的次数，用于通过引用次数回收内存，当refcount=0时，可以安全回收当前对象空间。使用object refcount(key}获取当前对象引用。当对象为整数且范围在[0-9999]时，Redis可以使用共享对象的方式来节省内存。

PS面试题，Redis的对象垃圾回收算法-----引用计数法。

#### *ptr字段

*ptr字段:与对象的数据内容相关，如果是整数，直接存储数据;否则表示指向数据的指针。

Redis新版本字符串且长度<=44字节的数据，字符串sds和redisobject一起分配，从而只要一次内存操作即可。

*PS* *：高并发写入场景中，在条件允许的情况下，建议字符串长度控制在44**字节以内，减少创建redisobject**内存分配次数，从而提高性能。*

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1668158598097/de771c3770b242be8ca619252dd10c94.png)

## Redis中的线程和IO模型

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1668158598097/f7d5d5855fa2494e96154534adcb6acc.png)

Redis 基于 Reactor 模式开发了自己的网络事件处理器 - 文件事件处理器（file event handler，后文简称为 FEH），而该处理器又是单线程的，所以redis设计为单线程模型。

采用I/O多路复用同时监听多个socket，根据socket当前执行的事件来为socket 选择对应的事件处理器。

当被监听的socket准备好执行accept、read、write、close等操作时，和操作对应的文件事件就会产生，这时FEH就会调用socket之前关联好的事件处理器来处理对应事件。

所以虽然FEH是单线程运行，但通过I/O多路复用监听多个socket，不仅实现高性能的网络通信模型，又能和 Redis 服务器中其它同样单线程运行的模块交互，保证了Redis内部单线程模型的简洁设计。

下面来看文件事件处理器的几个组成部分。

#### socket

文件事件就是对socket操作的抽象， 每当一个 socket 准备好执行连接accept、read、write、close等操作时， 就会产生一个文件事件。一个服务器通常会连接多个socket，多个socket可能并发产生不同操作，每个操作对应不同文件事件。

#### I/O多路复用程序

I/O 多路复用程序会负责监听多个socket。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1668158598097/cdb769bb4c7649e0acd1bad68b4c0662.png)

#### 文件事件分派器

文件事件分派器接收 I/O 多路复用程序传来的socket， 并根据socket产生的事件类型， 调用相应的事件处理器。

#### 文件事件处理器

服务器会为执行不同任务的套接字关联不同的事件处理器， 这些处理器是一个个函数， 它们定义了某个事件发生时， 服务器应该执行的动作。

Redis 为各种文件事件需求编写了多个处理器，若客户端连接Redis，对连接服务器的各个客户端进行应答，就需要将socket映射到连接应答处理器写数据到Redis，接收客户端传来的命令请求，就需要映射到命令请求处理器从Redis读数据，向客户端返回命令的执行结果，就需要映射到命令回复处理器当主服务器和从服务器进行复制操作时，
主从服务器都需要映射到特别为复制功能编写的复制处理器。

#### Epoll网络模型

为了避免用户应用与内核发生冲突，用户应用与内核是分离的:

- 进程的寻址空间会划分为两部分:`内核空间、用户空间`

- `用户空间`只能执行受限的命令(Ring3)，而且不能直接调用系统资源，必须通过内核提供的接口来访问

- `内核空间`可以执行特权命令(Ring0),调用一切系统资源

  文件描述符（File Descriptor）：简称FD，是一个从0 开始的无符号整数，用来关联Linux中的一个文件。在Linux中，一切皆文件，例如常规文件、视频、硬件设备等，当然也包括网络套接字（Socket）。
  IO多路复用：是利用单个线程来同时监听多个FD，并在某个FD可读、可写时得到通知，从而避免无效的等待，充分利用CPU资源。

阶段一：

1.用户进程调用select，指定要监听的FD集合
2.内核监听FD对应的多个socket
-3. 任意一个或多个socket数据就绪则返回readable
4.此过程中用户进程阻塞
阶段二：

1.用户进程找到就绪的socket
2.依次调用recvfrom读取数据
3.内核将数据拷贝到用户空间
用户进程处理数据
IO多路复用： 是利用单个线程来同时监听多个FD，并在某个FD可读、可写时得到通知，从而避免无效的等待，充分利用CPU资源。不过监听FD的方式、通知的方式又有多种实现，常见的有：

select
poll
epoll
差异：

select和poll只会通知用户进程有FD就绪，但不确定具体是哪个FD，需要用户进程逐个遍历FD来确认
epoll则会在通知用户进程FD就绪的同时，把已就绪的FD写入用户空间
IO多路复用之select
select是Linux最早是由的I/O多路复用技术：

![image-20231130192305655](E:\图灵课堂\Redis专题\笔记\1、Redis专题.assets\image-20231130192305655.png)

![image-20231130191955857](E:\图灵课堂\Redis专题\笔记\1、Redis专题.assets\image-20231130191955857.png)

总结：
select模式存在的三个问题：

能监听的FD最大不超过1024
每次select都需要把所有要监听的FD都拷贝到内核空间
每次都要遍历所有FD来判断就绪状态
poll模式的问题：

poll利用链表解决了select中监听FD上限的问题，但依然要遍历所有FD，如果监听较多，性能会下降
**epoll模式中如何解决这些问题的**？

基于epoll实例中的红黑树保存要监听的FD，理论上无上限，而且增删改查效率都非常高
每个FD只需要执行一次epoll_ctl添加到红黑树，以后每次epol_wait无需传递任何参数，无需重复拷贝FD到内核空间
利用ep_poll_callback机制来监听FD状态，无需遍历所有FD，因此性能不会随监听的FD数量增多而下降
IO多路复用-事件通知机制
当FD有数据可读时，我们调用epoll_wait就可以得到通知，但是事件通知的模式有两种:

LevelTriggered：简称LT，也叫做水平触发。当FD有数据可读时，会重复通知多次，直至数据处理完成，是epoll模式的默认模式
EdgeTriggered：简称ET，也叫做边沿触发。当fd有数据可读时，只会被通知一次，不管数据是否处理完成

- 这个就绪list链表是怎么维护的？

  当执行epoll_ctl时，除了把socket放到

epoll文件系统中的file对象对应的红黑树上之外，还会给内核中断处理程序注册一个回调函数，告诉内核，如果这个句柄的中断到了，就把它放到准备就绪的

list链表中。所以，当一个

socket上有数据到了，内核就把网卡上的数据复制到内核中然后就把socket插入到

list链表中。

​	epoll的基础就是回调。

### Redis6中的多线程

#### **Redis6.0之前的版本真的是单线程吗？**

Redis在处理客户端的请求时，包括获取 (socket 读)、解析、执行、内容返回 (socket 写) 等都由一个顺序串行的主线程处理，这就是所谓的“单线程”。但如果严格来讲从Redis4.0之后并不是单线程，除了主线程外，它也有后台线程在处理一些较为缓慢的操作，例如清理脏数据、无用连接的释放、大 key 的删除等等。

![在这里插入图片描述](https://img-blog.csdnimg.cn/bb695ff8f4ef4e9fadad37d97e8af841.png)

#### **Redis6.0之前为什么一直不使用多线程？**

官方曾做过类似问题的回复：使用Redis时，几乎不存在CPU成为瓶颈的情况，
Redis主要受限于内存和网络。例如在一个普通的Linux系统上，Redis通过使用pipelining每秒可以处理100万个请求，所以如果应用程序主要使用O(N)或O(log(N))的命令，它几乎不会占用太多CPU。

使用了单线程后，可维护性高。多线程模型虽然在某些方面表现优异，但是它却引入了程序执行顺序的不确定性，带来了并发读写的一系列问题，增加了系统复杂度、同时可能存在线程切换、甚至加锁解锁、死锁造成的性能损耗。Redis通过AE事件模型以及IO多路复用等技术，处理性能非常高，因此没有必要使用多线程。单线程机制使得 Redis 内部实现的复杂度大大降低，Hash 的惰性 Rehash、Lpush 等等,“线程不安全” 的命令都可以无锁进行。

注意：这里的单线程指的是Redis网络IO以及kv的读写由一个主线程完成。

![image-20231130224157699](E:\图灵课堂\Redis专题\笔记\1、Redis专题.assets\image-20231130224157699.png)

总结如下几点：

​	1，使用Redis，CPU不是瓶颈，受制内存与网络

​	2，提高Redis，Pipeline（批量命令）每秒100W个请求

​	3，单线程，IO多路复用处理都连接减少网络IO消耗

​	4，多线程可能引发的问题（线程切换，加/解/死锁）

​	5，惰性Rehash（渐进式的ReHash）



#### **Redis6.0为什么要引入多线程呢？**

2020年5月Redis6.0 版本发布，引入多线程

主要是为了**提高网络 IO** 读写性能，因为这个算是 Redis 中的一个性能瓶颈（Redis 的瓶颈主要受限于内存和网络）。

虽然，Redis6.0 引入了多线程，但是 Redis 的多线程只是在网络数据的读写这类耗时操作上使用了， 执行命令仍然是单线程顺序执行。因此，你也不需要担心线程安全问题。

而且6.0之前也并非没有多线程操作。实际上主线程单独完成网络请求模块和数据操作模块，其他比如持久化RDB，AOF，脏数据清理，释放无效连接，大key的删除，集群数据的同步等都是多线程执行的。

Redis6.0 的多线程默认是禁用的，只使用主线程。如需开启需要修改 redis 配置文件 redis.conf ：

```
io-threads-do-reads yes
```

开启多线程后，还需要设置线程数，否则是不生效的。同样需要修改 redis 配置文件 redis.conf :

```
io-threads 4 #官网建议4核的机器建议设置为2或3个线程，8核的建议设置为6个线程
```

Redis将所有数据放在内存中，内存的响应时长大约为120纳秒，对于小数据包，Redis服务器可以处理80,000到100,000 QPS，这也是Redis处理的极限了，对于80%的公司来说，单线程的Redis已经足够使用了。

但随着越来越复杂的业务场景，有些公司动不动就上亿的交易量，因此需要更大的QPS。

常见的解决方案是在分布式架构中对数据进行分区并采用多个服务器，但该方案有非常大的缺点，例如要管理的Redis服务器太多，维护代价大；某些适用于单个Redis服务器的命令不适用于数据分区；数据分区无法解决热点读/写问题；数据偏斜，重新分配和放大/缩小变得更加复杂等等。

![image-20231130224228526](E:\图灵课堂\Redis专题\笔记\1、Redis专题.assets\image-20231130224228526.png)

所以总结起来，redis支持多线程主要就是两个原因：

• 大公司海量并发应用场景

• 分布式架构缺点（服务维护，分区，分配等问题）

• 服务器CPU资源充分利用



#### Redis6.0默认是否开启了多线程？

Redis6.0的多线程默认是禁用的，只使用主线程。如需开启需要修改redis.conf配置文件：io-threads-do-reads yes

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1668158598097/fe03fc71ee284640b3de6f6f54fb521b.png)

开启多线程后，还需要设置线程数，否则是不生效的。同样修改redis.conf配置文件

关于线程数的设置，官方有一个建议：4核的机器建议设置为2或3个线程，8核的建议设置为6个线程，线程数一定要小于机器核数。还需要注意的是，线程数并不是越大越好，官方认为超过了8个基本就没什么意义了。

#### **Redis6.0采用多线程后，性能的提升效果如何？**

Redis 作者 antirez 在 RedisConf 2019分享时曾提到：Redis 6 引入的多线程 IO 特性对性能提升至少是一倍以上。国内也有大牛曾使用unstable版本在阿里云esc进行过测试，GET/SET 命令在4线程 IO时性能相比单线程是几乎是翻倍了。如果开启多线程，至少要4核的机器，且Redis实例已经占用相当大的CPU耗时的时候才建议采用，否则使用多线程没有意义。



## 缓存淘汰算法

当 Redis 内存超出物理内存限制时，内存的数据会开始和磁盘产生频繁的交换 (swap)。交换会让 Redis 的性能急剧下降，对于访问量比较频繁的 Redis 来说，这样龟速的存取效率基本上等于不可用。

官网 LFU https://redis.io/topics/lru-cache

The exact behavior Redis follows when the 

maxmemory limit is reached is configured using the 

maxmemory-policy configuration directive.

The following policies are available:

- 1: noeviction: return errors when the memory limit was reached and the client is trying to execute commands that could result in more memory to be used (most write commands, but DEL and a few more exceptions).
- 2:allkeys-**lru**: evict keys by trying to remove the less recently used (LRU) keys first, in order to make space for the new data added.  通过先删除最近最少使用的键来清除
- 3:volatile-**lru**: evict keys by trying to remove the less recently used (LRU) keys first, but only among keys that have an expire set, in order to make space for the new data added.删除最近使用最少的键但只针对设置过期时间的
- 4:allkeys-random: evict keys randomly in order to make space for the new data added. 随机所有
- 5:volatile-random: evict keys randomly in order to make space for the new data added, but only evict keys with an expire set. 随机设置过期的
- 6:volatile-**ttl****:** evict keys with an expire set, and try to evict keys with a shorter time to live (TTL) first, in order to make space for the new data added.首先逐出有过期设置的，然后尝试逐出有较短生存时间的



**maxmemory参数配置**

在生产环境中我们是不允许 Redis 出现交换行为的，为了限制最大使用内存，Redis 提供了配置参数 maxmemory 来限制内存超出期望大小。

当实际内存超出 maxmemory 时，Redis 提供了几种可选策略\(maxmemory-policy) 来让用户自己决定该如何腾出新的空间以继续提供读写服务。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1668158598097/f2ae13d3e120402fa571cf23230820b0.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1668158598097/c8b26778022d4f2487207d79048b6934.png)

### Noeviction

 noeviction 不会继续服务写请求
(DEL 请求可以继续服务)，读请求可以继续进行。这样可以保证不会丢失数据，但是会让线上的业务不能持续进行。这是默认的淘汰策略。

### volatile-lru

 volatile-lru 尝试淘汰设置了过期时间的
key，最少使用的 key 优先被淘汰。没有设置过期时间的 key 不会被淘汰，这样可以保证需要持久化的数据不会突然丢失。

### volatile-ttl

volatile-ttl 跟上面一样，除了淘汰的策略不是 LRU，而是 key 的剩余寿命 ttl 的值，ttl 越小越优先被淘汰。

### volatile-random

volatile-random 跟上面一样，不过淘汰的 key 是过期 key 集合中随机的 key。

### allkeys-lru

allkeys-lru 区别于volatile-lru，这个策略要淘汰的 key 对象是全体的 key 集合，而不只是过期的 key 集合。这意味着没有设置过期时间的 key 也会被淘汰。

### allkeys-random

allkeys-random跟上面一样，不过淘汰的策略是随机的 key。

volatile-xxx 策略只会针对带过期时间的key 进行淘汰，allkeys-xxx 策略会对所有的 key 进行淘汰。如果你只是拿 Redis 做缓存，那应该使用 allkeys-xxx，客户端写缓存时不必携带过期时间。如果你还想同时使用 Redis 的持久化功能，那就使用 volatile-xxx 策略，这样可以保留没有设置过期时间的 key，它们是永久的 key 不会被 LRU 算法淘汰。



### 近似 LRU 算法

###### LRU 算法

实现 LRU 算法除了需要key/value 字典外，还需要附加一个链表，链表中的元素按照一定的顺序进行排列。当空间满的时候，会踢掉链表尾部的元素。当字典的某个元素被访问时，它在链表中的位置会被移动到表头。所以链表的元素排列顺序就是元素最近被访问的时间顺序。

位于链表尾部的元素就是不被重用的元素，所以会被踢掉。位于表头的元素就是最近刚刚被人用过的元素，所以暂时不会被踢。

Redis 使用的是一种近似 LRU 算法，它跟 LRU 算法还不太一样。之所以不使用 LRU 算法，是因为需要消耗大量的额外的内存，需要对现有的数据结构进行较大的改造。近似

LRU 算法则很简单，在现有数据结构的基础上使用随机采样法来淘汰元素，能达到和 LRU 算法非常近似的效果。Redis 为实现近似 LRU 算法，它给每个 key 增加了一个额外的小字段，这个字段的长度是 24 个 bit，也就是最后一次被访问的时间戳。

当 Redis 执行写操作时，发现内存超出maxmemory，就会执行一次 LRU 淘汰算法。这个算法也很简单，就是随机采样出 5(可以配置maxmemory-samples) 个 key，然后淘汰掉最旧的 key，如果淘汰后内存还是超出 maxmemory，那就继续随机采样淘汰，直到内存低于 maxmemory 为止。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1668158598097/8376f180d4aa43b5b7319a6a4d4f721e.png)

如何采样就是看maxmemory-policy 的配置，如果是 allkeys 就是从所有的 key 字典中随机，如果是 volatile 就从带过期时间的 key 字典中随机。每次采样多少个 key 看的是 maxmemory_samples 的配置，默认为 5。

采样数量越大，近似 LRU 算法的效果越接近严格LRU 算法。

同时 Redis3.0 在算法中增加了淘汰池，新算法会维护一个候选池（大小为16），池中的数据根据访问时间进行排序，第一次随机选取的key都会放入池中，随后每次随机选取的key只有在访问时间小于池中最小的时间才会放入池中，直到候选池被放满。当放满后，如果有新的key需要放入，则将池中最后访问时间最大（最近被访问）的移除。进一步提升了近似 LRU 算法的效果。

Redis维护了一个24位时钟，可以简单理解为当前系统的时间戳，每隔一定时间会更新这个时钟。每个key对象内部同样维护了一个24位的时钟，当新增key对象的时候会把系统的时钟赋值到这个内部对象时钟。比如我现在要进行LRU，那么首先拿到当前的全局时钟，然后再找到内部时钟与全局时钟距离时间最久的（差最大）进行淘汰，这里值得注意的是全局时钟只有24位，按秒为单位来表示才能存储194天，所以可能会出现key的时钟大于全局时钟的情况，如果这种情况出现那么就两个相加而不是相减来求最久的key。

### LFU算法

LFU算法是**Redis4.0**里面新加的一种淘汰策略。它的全称是Least Frequently Used，它的核心思想是根据key的最近被访问的频率进行淘汰，很少被访问的优先被淘汰，被访问的多的则被留下来。

LFU算法能更好的表示一个key被访问的热度。假如你使用的是LRU算法，一个key很久没有被访问到，只刚刚是偶尔被访问了一次，那么它就被认为是热点数据，不会被淘汰，而有些key将来是很有可能被访问到的则被淘汰了。如果使用LFU算法则不会出现这种情况，因为使用一次并不会使一个key成为热点数据。LFU原理使用计数器来对key进行排序，每次key被访问的时候，计数器增大。计数器越大，可以约等于访问越频繁。具有相同引用计数的数据块则按照时间排序。

**LFU一共有两种策略：**

7：volatile-lfu：在设置了过期时间的key中使用LFU算法淘汰key

8：allkeys-lfu：在所有的key中使用LFU算法淘汰数据

LFU把原来的key对象的内部时钟的24位分成两部分，前16位ldt还代表时钟，后8位logc代表一个计数器。

logc是8个 bit，用来存储访问频次，因为8个 bit能表示的最大整数值为255，存储频次肯定远远不够，所以这8个 bit存储的是频次的对数值，并且这个值还会随时间衰减，如果它的值比较小，那么就很容易被回收。为了确保新创建的对象不被回收，新对象的这8个bit会被初始化为一个大于零的值LFU INIT_VAL（默认是=5）。

ldt是16个bit，用来存储上一次 logc的更新时间。因为只有16个 bit，所精度不可能很高。它取的是分钟时间戳对2的16次方进行取模。

ldt的值和LRU模式的lru字段不一样的地方是,
ldt不是在对象被访问时更新的,而是在Redis 的淘汰逻辑进行时进行更新，淘汰逻辑只会在内存达到 maxmemory 的设置时才会触发，在每一个指令的执行之前都会触发。每次淘汰都是采用随机策略，随机挑选若干个 key，更新这个 key 的“热度”，淘汰掉“热度”最低的key。因为Redis采用的是随机算法，如果
key比较多的话，那么ldt更新得可能会比较慢。不过既然它是分钟级别的精度，也没有必要更新得过于频繁。

ldt更新的同时也会一同衰减logc的值。



**为什么 Redis 要缓存系统时间戳**

我们平时使用系统时间戳时，常常是不假思索地使用System.currentTimeInMillis或者time.time()来获取系统的毫秒时间戳。Redis不能这样，因为每一次获取系统时间戳都是一次系统调用，系统调用相对来说是比较费时间的，作为单线程的Redis承受不起，所以它需要对时间进行缓存，由一个定时任务，每毫秒更新一次时间缓存，获取时间都是从缓存中直接拿。

##### 附源码解读：

 server.h  redisObject {

里面lru:LRU_BITS } 24bit的  记录对象访问，操作时间的秒单位的最后24位

```
//java里面获取秒
long timeMils = System.currentTimeMilis();
sout( timeMils/1000);//毫秒
sout(1<<24-1) //24位的最大值
sout(timeMils/1000 & (1<<24-1))//最后24位  位操作更快

//假如 12bit（位）
最大值       111111 111111  12bit的最大值
左移12位得到 1 000000 000000   1<<12
再减1       0 111111 111111 

与运算：   假如这样的一串值   
        11111 0000 11110    要去获取最后五位的话
      &            11111    （必须都为1 才是1）
--------------------------
        00000 0000 11110   最快得到最后五位
      

```

server.lru 当前时间的秒单位的最后24bit

server.lru - redisObject.lru

可能出现轮循的问题。 即 server.lru < redisObject.lru

**比如**：

 一个数据 5月访问  现在是6月  多少个月没访问过？ 6-5 =1

 一个数据 5月访问  现在是3月  多少个月没访问过？ 3+12 - 5 =10

解决方案：

```
if(server.lru >= redisObject.lru){
     server.lru - redisObject.lru   
}else{
     server.lru +24bit的最大值 - redisObject.lru   
}
//伪LRU  不用管年    最大的场景 是缓存 
```

源码实现： evict.c   evictionPoolPopulate方法

判断LRU 进入 estimateObjectIdleTime 方法 

跟上面讲的对应上

```
//比较过期
unsigned long long estimateObjectIdleTime(robj *o) {
   //获取秒单位时间的最后24位
    unsigned long long lruclock = LRU_CLOCK();
   //因为只有24位，所有最大的值为2的24次方-1
   //超过最大值从0开始，所以需要判断lruclock（当前系统时间）跟缓存对象的lru字段的大小
   //如果当前秒单位大于等于 对象的LRU的话
    if (lruclock >= o->lru) {
      //如果lruclock>=robj.lru，返回lruclock-o->lru，再转换单位
        return (lruclock - o->lru) * LRU_CLOCK_RESOLUTION;
    } else {
      //否则采用lruclock + (LRU_CLOCK_MAX - o->lru)，得到对象的值越小，返回的值越大，越大越容易被淘汰
        return (lruclock + (LRU_CLOCK_MAX - o->lru)) *
                    LRU_CLOCK_RESOLUTION;
    }
}
```

LRU：根据次数来淘汰，越少使用的越容易被淘汰

存在时效性问题 ： 比如去年某网红桃色新闻 1000W次访问，今年最近王姓艺人出事出来没多久10W次访问   ，按次数淘汰会把今年的给淘汰，但过时的老数据没有淘汰。

reids 怎么解决的呢？ 

server.h 中 redisObject.lru 在LFU的场景  注释说明了 

LFU 将这个24bit的字段拆成两份 

16bit的时间 + 8bit的次数/频率 counter（不是简单的访问+1，越多越慢） 

前面记录对象操作访问 分单位的最后16位

```
sout( timeMils/1000/60);//
sout(timeMils/1000/60 & (1<<16-1))//最后16位 源码看过没 位操作更快
```

根据系统时间分单位的最后16bit，得到对象多少分钟没有访问了。

```
怎么得到的呢？
if(server.lru >= redisObject.lru){
     server.lru - redisObject.lru   
}else{
     server.lru +16bit的最大值 - redisObject.lru   
}
```

玩游戏 会员 v8 但是如果2个月没玩，半年没充的话 掉到v2

根据对象多少分钟没访问，去减少访问次数。

1.16bit的时间就是用来判断多少分钟没访问就去减少多少次， 

可配置

 lfu-log-factor 因子等级默认10  对应前面的8bit

 lfu-decay-time 衰减时间默认 1 对应前面的16bit

8bit的次数 counter  0~255次

2 最多255次 超过怎么办？源码 evict.c  evictionPoolPopulate方法中

LFUDecrAndReturn 

```
unsigned long LFUDecrAndReturn(robj *o) {
   //object.lru字段右移8位，得到前面16位的时间
    unsigned long ldt = o->lru >> 8;
   //lru字段与255进行&运算（255代表8位的最大值），得到8位counter值
    unsigned long counter = o->lru & 255;
   //如果配置了 lfu_decay_time，用 LFUTimeElapsed(ldt) 除以配置的值
    //总的没访问的分钟时间/配置值，得到每分钟没访问衰减多少
   // 默认得到的就是 一分钟减少一次
    unsigned long num_periods = server.lfu_decay_time ? LFUTimeElapsed(ldt) / server.lfu_decay_time : 0;
    if (num_periods)
      //不能减少为负数，非负数用couter值减去衰减值
        counter = (num_periods > counter) ? 0 : counter - num_periods;
    return counter;
}
```

假如：现在次数 50次，有10分钟没访问了， 现在还剩多少次？ 40

够不够继续看 往上一个 LFULogIncr 方法 增加次数

1，最多不能超过255

2，新生key策略。由于新创建的对象如果counter 0 很容易被淘汰 所以初始值 为5

robj *createObject( 方法中 LFU_INIT_VAL  默认5。（在server.h配置））

3，不是简单的访问+1，次数小于5必+1，越大越慢 跟baseval与lfu_log_factor相关

最后再看官网 两参的对照表  点击数与 factor分布  是不是很好理解了。

现在面试官问你：redis怎么解决时效性问题？或者LFU算法怎么实现的？你应该能讲的出来了吧？

因为源码我都带你看，是不是有点深度，课后可以再去复盘下表达。

```
uint8_t LFULogIncr(uint8_t counter) {
   //如果已经到最大值255，返回255 ，8位的最大值
    if (counter == 255) return 255;
   //得到随机数（0-1）
    double r = (double)rand()/RAND_MAX;
   //LFU_INIT_VAL表示基数值,默认为5（在server.h配置）
    double baseval = counter - LFU_INIT_VAL;
   //如果当前counter小于基数，那么p=1,100%加
    if (baseval < 0) baseval = 0;
   //不然，按照几率是否加counter，同时跟baseval与lfu_log_factor相关
    //都是在分母，所以2个值越大，加counter几率越小，越大加的几率越小（所以刚才的255够不够的问题解决）
    double p = 1.0/(baseval*server.lfu_log_factor+1);
    if (r < p) counter++;  //p越小，几率也就越小
    return counter;
}
```



## 过期策略

### 定期过期

Redis 所有的数据结构都可以设置过期时间，时间一到，就需要删除。但是会不会因为同一时间太多的key 过期，以至于忙不过来。同时因为Redis 是单线程的，删除的时间也会占用线程的处理时间，如果删除的太过于繁忙，会不会导致线上读写指令出现卡顿。

过期的 key 集合

redis 会将每个设置了过期时间的
key 放入到一个独立的字典中，以后会定时遍历这个字典来删除到期的 key。除了定时遍历之外，它还会使用惰性策略来删除过期的 key，所谓惰性策略就是在客户端访问这个 key 的时候，redis 对 key 的过期时间进行检查，如果过期了就立即删除。定时删除是集中处理，惰性删除是零散处理。

定时扫描策略

 怎么去过期呢？   

##### 定期过期总结 ：

隔一段时间去监测一下  (redis.conf  hz 10    1秒10次) 

​						多久去看一次 ？

​					    看哪些？

​							流程：  1，每秒10次去检查 设置了过期时间的 key

​											2，根据Hash桶的维度去拿 （拿到20个元素 就不会再拿后面的桶，前面不足20个就拿到400个hash桶）  

​										3，删除过期的数据

​										4，如果没有拿到过期数据   或者过期数据超过 10% 则继续上面的过程 

​										5，循环4这个步骤 到16次 就判断是否超时 25 ms

​	

比如： 第一个hash桶 有 40   第二个桶有 10 个       拿 40 就去删除已经过期的数据，如果过期的有30个怎么办？

​						就会继续拿

​		    第一个hash桶 有 10   第二个桶有 5个  第三个桶 100     拿 115 就去删除

​			前400桶都是空的 ，   

```
//多久执行一次 找config.c   redis.conf  默认10  默认100ms一次 。范围1-500建议小于100.
   return 1000/server.hz;
```

高并发时候  或者 很闲的时候  

```
dynamic-hz yes   自适应加减频率
```

##### 源码流程：

![image-20231205213014848](E:\图灵课堂\Redis专题\笔记\1、Redis专题.assets\image-20231205213014848.png)

### 被动过期

所谓惰性策略就是在客户端访问这个key的时候，redis对key的过期时间进行检查，如果过期了就立即删除，不会给你返回任何东西。

定期删除可能会导致很多过期key到了时间并没有被删除掉。所以就有了惰性删除。假如你的过期 key，靠定期删除没有被删除掉，还停留在内存里，除非你的系统去查一下那个 key，才会被redis给删除掉。这就是所谓的惰性删除，即当你主动去查过期的key时,如果发现key过期了,就立即进行删除,不返回任何东西.

##### 被动过期总结：		

惰性过期  ：访问数据的时候看是否过期 

​							流程 ：1，拿到到设置过期时间的key，如果没有设置过期时间 返回0 

​										2，判断是否主节点 ，如果是 则继续

​										3，同步的和异步的方法去删除 key 

##### 源码流程：

![image-20231205212958031](E:\图灵课堂\Redis专题\笔记\1、Redis专题.assets\image-20231205212958031.png)



#### 附源码解读：

 在 server.h 

过期淘汰策略 从此开始   统一维护的key redisDb 开始

当然我们先补充完 zset 实际在server.h 中可见 zset结构体  ，以及内部的zskiplist  挑表 关键逻辑在t_zset.c中

过期策略的被动过期比较简单  123

源码：db.c  

int  expireIfNeeded(redisDb *db, robj *key) {  方法

```
被动过期
 */
int expireIfNeeded(redisDb *db, robj *key) {
   //如果没过期 返回0
    if (!keyIsExpired(db,key)) return 0;
    //如果配置有masterhost主节点，说明这里是从节点，那么不操作增删改 不然就主从不一致了
  if (server.masterhost != NULL) return 1;
     //判断 是否有在进行定期过期   两个方法 一个异步，一个非异步
int retval = server.lazyfree_lazy_expire ? dbAsyncDelete(db,key) :
                                           dbSyncDelete(db,key);
```

接下来看看定期过期（学员面阿里/京东被问过）

**那到底如何定期过期呢？**  两个方面 需要注意。

一： 查出哪些过期了。  

二： 多久看一下过期。 

一： 查出哪些过期了。 

1，只看设置了过期时间的kwey

首先看看 Redis统一维护了哪些key 。我们这里只查询设置了过期时间的

1.server.h 里面 的

```
typedef struct redisDb { 
dict *dict 所有的 key 。 后面分类key
dict *expires   所有过期时间的key
dict *blocking_keys 这里有个细节的好问题， 比如我们的blpop 这也是为何它不会导致阻塞的原因，因为分开放的
keys ，哨兵key明天会详细讲
}
```



2，不是一次性把所有设置了过期时间的key全部拿出来判断

a 根据hash桶( dictEntry* )的维度 ，拿到20个为止，最多只会检查400个桶

b 删除拿到的数据里面已经过期的数据，

c 如果未拿到设置过期数据，或者删除的数据超过超过拿到的 10%，就继续前面两步

d 最多拿 16次，检查时间

假如 第一个桶  10条数据

 第二个桶 40条数据

会拿多少？50   如果第一个桶 26条，拿？ 26   如果前两桶都不够20还会继续

加入拿到 50条 有10条过期了

源码：expire.c    搜 

activeExpireCycle(

从这个位置开始看

//循环DB，可配，默认16  不是单独的库是所有的都会去过期

for (j = 0; j < dbs_per_call && timelimit_exit == 0; j++) {	

第一步 和 第二步的 源码到//检查是否过期  此结束

if (activeExpireCycleTryExpire(db,e,now)) expired++;

```
// 活跃过期周期
void activeExpireCycle(int type) {
    //循环DB，可配，默认16  不是单独的库是所有的都会去过期
   for (j = 0; j < dbs_per_call && timelimit_exit == 0; j++) {
    //如果没有过期key,循环下一个DB
         if ((num = dictSize(db->expires)) == 0) {
             //最多拿20个 config_keys_per_loop 这个往上可找到来源
         if (num > config_keys_per_loop)
             num = config_keys_per_loop;
     long max_buckets = num*20;//20*20= 400个桶
         long checked_buckets = 0; //检查的hash桶数量

//如果拿到的key大于20 或者  循环的checked_buckets大于400，跳出
         while (sampled < num && checked_buckets < max_buckets) {
             //检查2个table的原因 ，扩容的时候两个hashtable里面都会有数据
            for (int table = 0; table < 2; table++) {
   //判断是否 table=1（第二个桶） 并且 没有在rehashing扩容中 说明第二个桶里面没有
   //扩容中两个桶都会有数据，扩容之后会第二个桶置空
                if (table == 1 && !dictIsRehashing(db->expires)) break;
   //  db->expires 设置了过期时间的
                unsigned long idx = db->expires_cursor;
                idx &= db->expires->ht[table].sizemask;
   //根据index拿到hash桶
                dictEntry *de = db->expires->ht[table].table[idx];
                //循环hash桶里的key 
               while(de) {
                   /* Get the next entry now since this entry may get
                    * deleted. */
                   dictEntry *e = de;
                   de = de->next;

                   ttl = dictGetSignedIntegerVal(e)-now;
   //检查是否过期  第一步 和 第二步的 到此结束
                   if (activeExpireCycleTryExpire(db,e,now)) expired++;
                   
                   //检查16次
        if ((iteration & 0xf) == 0) { /* check once every 16 iterations. */
        // 自循 条件 第三步： sampled==0 表示没有拿到设置过期数据 ；config_cycle_acceptable_stale值往上
   //过期的*100/拿到过期取样的数据 > 10 是不是超过了百分之10 就继续ddd
     } while (sampled == 0 ||
              (expired*100/sampled) > config_cycle_acceptable_stale);
    }
 }
```

二： 多久看一下过期。 

一个 serverCron( 定时执行的方法   

源码 server.c



```
一个 serverCron( 定时执行的方法     最后return的时间 
//多久执行一次 找config.c   redis.conf  默认10 范围1-500建议小于100
   return 1000/server.hz;
```

多久执行一次呢 return 1000/server.hz;   默认hz 10

间隔根据配置文件hz来决定，每秒执行多少次 ，100ms执行一次

默认为10，即每秒执行10次

为何不建议改大？

还有个参数dynamic-hz yes （此参数后面一个） 默认开启状态，当大量客户端连接进行时 hz 会动态临时调高 hz的整数倍。如此空闲的时候redis只会占用少了cpu时间，繁忙时能做出更快的相应



### lazyfree

使用 DEL 命令删除体积较大的键， 又或者在使用
FLUSHDB 和 FLUSHALL 删除包含大量键的数据库时，造成redis阻塞的情况；另外redis在清理过期数据和淘汰内存超限的数据时，如果碰巧撞到了大体积的键也会造成服务器阻塞。

为了解决以上问题， redis 4.0 引入了lazyfree的机制，它可以将删除键或数据库的操作放在后台线程里执行， 从而尽可能地避免服务器阻塞。

lazyfree的原理不难想象，就是在删除对象时只是进行逻辑删除，然后把对象丢给后台，让后台线程去执行真正的destruct，避免由于对象体积过大而造成阻塞。redis的lazyfree实现即是如此，下面我们由几个命令来介绍下lazyfree的实现。

4.0 版本引入了 unlink 指令，它能对删除操作进行懒处理，丢给后台线程来异步回收内存。

UNLINK的实现中，首先会清除过期时间，然后调用dictUnlink把要删除的对象从数据库字典摘除，再判断下对象的大小（太小就没必要后台删除），如果足够大就丢给后台线程，最后清理下数据库字典的条目信息。

主线程将对象的引用从「大树」中摘除后，会将这个 key 的内存回收操作包装成一个任务，塞进异步任务队列，后台线程会从这个异步队列中取任务。任务队列被主线程和异步线程同时操作，所以必须是一个线程安全的队列。

Redis 提供了 flushdb 和 flushall 指令，用来清空数据库，这也是极其缓慢的操作。Redis 4.0 同样给这两个指令也带来了异步化，在指令后面增加 async 参数就会进入后台删除逻辑。

Redis4.0 为这些删除点也带来了异步删除机制，打开这些点需要额外的配置选项。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1668158598097/7a5ee80319354c2192e5147e49ed8838.png)

1、slave-lazy-flush
从库接受完 rdb 文件后的 flush 操作

2、lazyfree-lazy-eviction
内存达到 maxmemory 时进行淘汰

3、lazyfree-lazy-expire
key 过期删除

4、lazyfree-lazy-server-del
rename 指令删除 destKey













