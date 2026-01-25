# MQ专题

## 为什么要用消息中间件？

### 异步处理

场景说明：用户注册后，需要发注册邮件和注册短信。传统的做法有两种1.串行的方式；2.并行方式。

串行方式：将注册信息写入数据库成功后，发送注册邮件，再发送注册短信。以上三个任务全部完成后，返回给客户端。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670340547050/1404290bd1bd4e17a0bbbc0390607166.png)

（2）并行方式：将注册信息写入数据库成功后，发送注册邮件的同时，发送注册短信。以上三个任务完成后，返回给客户端。与串行的差别是，并行的方式可以提高处理的时间。

假设三个业务节点每个使用50毫秒钟，不考虑网络等其他开销，则串行方式的时间是150毫秒，并行的时间可能是100毫秒。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670340547050/b877bea2082948ef9bff5443052744cd.png)

小结：如以上案例描述，传统的方式系统的性能（并发量，吞吐量，响应时间）会有瓶颈。如何解决这个问题呢？

引入消息队列，将不是必须的业务逻辑，异步处理。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670340547050/4defe2d5d1104531a992e0993a1f8895.png)

按照以上约定，用户的响应时间相当于是注册信息写入数据库的时间，也就是50毫秒。注册邮件，发送短信写入消息队列后，直接返回，因此写入消息队列的速度很快，基本可以忽略，因此用户的响应时间可能是50毫秒。因此架构改变后，系统的吞吐量提高到每秒20 QPS。比串行提高了3倍，比并行提高了两倍。

### 应用解耦

场景说明：用户下单后，订单系统需要通知库存系统。传统的做法是，订单系统调用库存系统的接口。

传统模式的缺点：

1）  假如库存系统无法访问，则订单减库存将失败，从而导致订单失败；

2）  订单系统与库存系统耦合；



如何解决以上问题呢？引入应用消息队列后的方案

订单系统：用户下单后，订单系统完成持久化处理，将消息写入消息队列，返回用户订单下单成功。

库存系统：订阅下单的消息，采用拉/推的方式，获取下单信息，库存系统根据下单信息，进行库存操作。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670340547050/61c38c04414847378f5dfc8e4ce8d3af.png)

假如：在下单时库存系统不能正常使用。也不影响正常下单，因为下单后，订单系统写入消息队列就不再关心其他的后续操作了。实现订单系统与库存系统的应用解耦。

### 流量削峰

流量削峰也是消息队列中的常用场景，一般在秒杀或团抢活动中使用广泛。

应用场景：秒杀活动，一般会因为流量过大，导致流量暴增，应用挂掉。为解决这个问题，一般需要在应用前端加入消息队列：可以控制活动的人数；可以缓解短时间内高流量压垮应用。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670340547050/cfe93431f821480198e9c6039a442235.png)



用户的请求，服务器接收后，首先写入消息队列。假如消息队列长度超过最大数量，则直接抛弃用户请求或跳转到错误页面；秒杀业务根据消息队列中的请求信息，再做后续处理。

### 日志处理

日志处理是指将消息队列用在日志处理中，比如Kafka的应用，解决大量日志传输的问题。架构简化如下：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670340547050/a61a9b97e3ed40aba390c7555e58f535.png)

日志采集客户端，负责日志数据采集，定时写入Kafka队列：Kafka消息队列，负责日志数据的接收，存储和转发；日志处理应用：订阅并消费kafka队列中的日志数据；

# 基础入门

## **消息中间件(MQ)的定义**

一般认为，消息中间件属于分布式系统中一个子系统，关注于数据的发送和接收，利用高效可靠的异步消息传递机制对分布式系统中的其余各个子系统进行集成。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1683530057014/3af8d04386094d5f96013720762aae1f.png)

### **应用异步与解耦**

系统的耦合性越高，容错性就越低。以电商应用为例，用户创建订单后，如果耦合调用库存系统、物流系统、支付系统，任何一个子系统出了故障或者因为升级等原因暂时不可用，都会造成下单操作异常，影响用户使用体验

使用消息中间件，系统的耦合性就会提高了。比如物流系统发生故障，需要几分钟才能来修复，在这段时间内，物流系统要处理的数据被缓存到消息队列中，用户的下单操作正常完成。当物流系统恢复后，继续处理存放在消息队列中的订单消息即可，终端系统感知不到物流系统发生过几分钟故障。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1683530057014/515f06e1af544da28f995dc74f14f27f.png)

### **流量削峰**

应用系统如果遇到系统请求流量的瞬间猛增，有可能会将系统压垮。有了消息队列可以将大量请求缓存起来，分散到很长一段时间处理，这样可以大大提到系统的稳定性和用户体验。

**互联网公司的大促场景（双十一、店庆活动、秒杀活动）都会使用到** **MQ** **。**

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1683530057014/3464914df4f84cf6a175bfb3ce23a84f.png)

### 常见的消息队列

 IBM MQ、RabbitMQ、ActiveMQ 和 Apache Kafka 等。 这些系统提供了丰富的特性和功能，能够满足各种不同的应用场景需求。

1983，Teknekron 提出了总线思想，设计了The Infomation Bus
1990-1993，IBM公司实现了WebSphere MQ
1997，Teknekron 实现了TIBCO
1997, 微软实现了MSMQ
2001，java提出了JMS模型
2003，apache提出了activemq
2004，出现了AMQP规范
2007，RabbitMQ诞生
2011，Kafka诞生，消息队列处理能力显著提升。
2014，Amazon SQS 是一种云消息队列服务，由亚马逊 Web Services（AWS）提供。 SQS 可以轻松地在云中构建可伸缩的应用程序，并提供高可用性和容错能力。
2015，Google Cloud Pub/Sub 是一种云消息队列服务，由谷歌云提供。 Pub/Sub 可以轻松地在云中构建可伸缩的应用程序，并提供高可用性和容错能力。

# 各种消息队列优缺点

## 1、RabbitMQ

优点：rabbitMq 几万级数据量，基于erlang语言开发，因此响应速度快些，并且社区活跃度比较活跃，可视化界面。
缺点：数据吞吐量相对与小一些，并且是基于erlang语言开发，比较重的问题难以维护

## 2、RocketMQ

rocketMq几十万级别数据量，基于Java开发，应对了淘宝双十一考验，并且文档十分的完善，拥有一些其他消息队列不具备的高级特性。
如定时推送，其他消息队列是延迟推送，如 rabbitMq 通过设置 expire 字段设置延迟推送时间。又比如rocketmq实现分布式事务，比较可靠的。

## 3、kafka

kafka真正的大规模分布式消息队列，提供的核心功能比较少。基于zookeeper实现的分布式消息订阅。

# ![img](https://img2023.cnblogs.com/blog/1252513/202307/1252513-20230714024339784-1318227002.png) 

## 

# RocketMQ

## 为什么懂得了一个个技术点，却依然用不好 RocketMQ？

我知道，很多同学都是带着一个个具体的问题来学这门课的，比如说

RocketMQ数据怎么做持久化？

消息重复方案应该怎么做？这些问题当然很重要，但是，如果你只是急于解决这些细微的问题，你的RocketMQ使用能力就很难得到质的提升。

只关注零散的技术点，没有建立起一套完整的知识框架，缺乏系统观，但是，系统观其实是至关重要的。从某种程度上说，在解决问题时，拥有了系统观，就意味着你能有依据、有章法地定位和解决问题。

如何高效地形成系统观呢？我们做事情一般都希望“多快好省”，说白了，就是希望花很少的时间掌握更丰富的知识和经验，解决更多的问题。听起来好像很难，但实际上，只要你能抓住主线，在自己的脑海中绘制一幅 RocketMQ的 全景知识图，这完全是可以实现的。而这也是我在设计这门课时所遵循的思路。

本课程基于RocketMQ5版本来进行讲解，下图是RocketMQ5的“两大维度、三大主线”

“两大维度”就是指实战应用维度和底层与源码维度。

“三大主线”也就是指高性能、高可用和高可扩展（可以简称为“三高”）。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1683530057014/d573a3c336c542db95b95fd6f2c3deab.png)

## 如何多快好省的学习RocketMQ

RocketMQ 作为庞大的消息中间件，可以说遍地都是知识，一抓一大把，我们怎么能快速地知道该学哪些呢？

**你完全可以按照这三大主线**

高性能：包括存储模型、数据结构、多线程运用；

高可用：包括重试机制、流控机制、HA机制；

高可扩展：包括消息过滤、批量消息、proxy机制；

**在应用维度上**，我建议大家按照两种方式学习: “应用场景驱动”和“典型案例驱动”，一个是“面”的梳理，一个是“点”的掌握。

异步和解耦是 RocketMQ的两大广泛的应用场景。在这些场景中，本身就具有一条显式的技术链。比如说，提到消息中间件的场景，你肯定会想到消息丢失、消息重复、消息顺序等一连串的问题。

## RockeMQ的安装

### RocketMQ的windows下的安装

#### 官方下载地址

[https://rocketmq.apache.org/zh/download/]()

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1682342232048/120961307015418080c839752881c84a.png)

控制台下载

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1682342232048/6524c731667e43cb9c34ebf9977d5bcf.png)

#### 环境要求

JDK1.8(64位)

#### 配置环境变量

变量名：ROCKETMQ_HOME

变量值：MQ解压路径\MQ文件夹名

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1682342232048/a594af429cd64d549863344d6ac9c6f0.png)

### 启动

RocketMQ的物理架构中，都是需要先启动NameServer再启动Broker的。所以启动顺序一定不要搞反了。

#### 启动NAMESERVER

Cmd命令框执行进入至‘MQ文件夹\bin’下，然后执行‘start mqnamesrv.cmd’，启动NAMESERVER。成功后会弹出提示框，此框勿关闭。

![](file:///C:\Users\Administrator\AppData\Local\Temp\msohtmlclip1\01\clip_image008.jpg)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1682342232048/aba943b4be324474b811df034cef63ae.png)

弹出boot success后，说明启动成功，那么NameServer的监听端口是本机是9876端口。

这种启动方式后，日志的一般放在用户目录下的 logs目录下。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1682342232048/4be26137ebca4bc099a3c897b917aa24.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1682342232048/c162b4ce538a45d3bf8f5aa45d78a38a.png)

#### 启动BROKER

Cmd命令框执行进入至‘MQ文件夹\bin’下，然后执行

```
 start mqbroker.cmd -n localhost:9876 
```

启动BROKER。成功后会弹出提示框，此框勿关闭。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1682342232048/8adb02c3f43b47db95c2375069e9e3f3.png)

日志的一般放在用户目录下的 logs目录下。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1682342232048/ba0ff006dae346d4b9826bc21aa12b06.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1682342232048/3950455488024692ac3659c8b0c06865.png)

### 控制台启动

下载地址：[https://github.com/apache/rocketmq-dashboard](https://github.com/apache/rocketmq-dashboard)

**控制台端口及服务地址配置：**

下载完成之后，进入‘\rocketmq-console\src\main\resources’文件夹，打开‘application.properties’进行配置。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1682342232048/420d5c00d2d947f3913090445507d488.png)

因为本身控制台也是单独的Java应用，默认的是8080，为了防止与Tomcat冲突，我改成了8089

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1682342232048/2f749e4cdab34d13938008e5b8282f20.png)

然后进入dashboard根目录执行‘mvn clean package -Dmaven.test.skip=true’，编译生成。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1683530057014/0da25241600640848cbd9f6a131d6261.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1683530057014/a070c76ecb064533aea9c7cf25eab654.png)

执行‘java -jar rocketmq-dashboard-1.0.0.jar’，启动dashboard。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1683530057014/83ed790632fb4fe9b9b5c1b4d827b720.png)

### 启动脚本整合

每次启动RocketMQ的命令还是挺麻烦的，所以这里做了一个脚本整合。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1683530057014/71921b2e8c6e41fdba29ebcbc8ec5a95.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1683530057014/39bdd125c6e04a86acd64a6a041b4356.png)

因为都是采用默认配置，数据文件的地址默认都是在C盘（C盘最好确保有足够的空间，数据文件默认启动就是1G。）

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1683530057014/49a920ee9f32456d9edcf25b45a202d8.png)

## RockeMQ的基本概念

### 整体架构：

![image-20240122135749441](E:\图灵课堂\MQ专题\Kafka.assets\image-20240122135749441.png)



### 消息的发送与消费模型

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1683530057014/3846461a51dd4b4eb8e69c3f9904f74c.png)

### nameServer ：

（早于版本2.x没有此组件，需要zookeeper完成）

用于存储topic，Broker关系信息 提供动态拓展和负载均衡.平时开销主要维持Broker所发的Topic信息 心跳 

### Broker 模块  :

 broker启动进程 主要负责消息的读写。重点关注顺序写和随机写； 负载均衡与动态伸缩

注意：broker和namesev心跳请求间隔30s，请求包含broker和其所有的Topic信息。 namesev会反查broker的心跳topic情况。 如果2m没有心跳则认为下线。

### **主题(Topic)**

标识RocketMQ中一类消息的逻辑名字，消息的逻辑管理单位。无论消息生产还是消费，都需要指定Topic。主题主要用于区分消息的种类：一个生产者可以发送消息给一个或者多个Topic，消息的消费者也可以订阅一个或者多个Topic消息。

### **消息队列**  **(**  **Message Queue** **)**

简称Queue或Q。消息物理管理单位。一个Topic将有若干个Q。

无论生产者还是消费者，实际的生产和消费都是针对Q级别。例如Producer发送消息的时候，会预先选择（默认轮询）好该Topic下面的某一条Q发送；Consumer消费的时候也会负载均衡地分配若干个Q，只拉取对应Q的消息。

### **生产者(Producer)**

**生产者** ：也称为消息发布者，负责发送消息至RocketMQ。

### **消费者(Consumer)**

**消费者** ：也称为消息订阅者，负责从RocketMQ接收并消费消息。

### **消费者分组(ConsumerGroup)**

标识一类Consumer的集合名称，这类Consumer通常消费一类消息（也称为Consumer Group），且消费逻辑一致。同一个Consumer Group下的各个实例将共同消费topic的消息，起到负载均衡的作用。

### 订阅关系（Subscription）

RocketMQ中的消费者订阅关系（Subscription）是指消费者与主题（Topic）之间的订阅关系。

消费者可以通过指定订阅规则来订阅某个主题的消息。



## RocketMQ生产消费流程

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1683530057014/1dc10bb4affd41338e68d4007a2b3a1c.png)

## 普通消息的发送与消费

### **三种消息发送方式**

#### **同步发送消息**

同步发送是指消息发送方发出数据后，同步等待，直到收到接收方发回响应之后才发下一个请求。这种可靠性同步地发送方式使用的比较广泛，比如：重要的消息通知，短信通知。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1683530057014/eb79152ffc864f28834f14bc975cc937.png)

#### **异步发送消息**

异步消息通常用在对响应时间敏感的业务场景，即发送端不能容忍长时间地等待Broker的响应。消息发送方在发送了一条消息后，不等接收方发回响应，接着进行第二条消息发送。发送方通过回调接口的方式接收服务器响应，并对响应结果进行处理。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1683530057014/d41c240f1bfa489a9b399d627b42d1e7.png)

#### **单向发送消息**

这种方式主要用在不特别关心发送结果的场景，例如日志发送。单向（Oneway）发送特点为发送方只负责发送消息，不等待服务器回应且没有回调函数触发，即只发送请求不等待应答。此方式发送消息的过程耗时非常短，一般在微秒级别。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1683530057014/ec2538a7d75b400ab8a4a6edac90866b.png)

### **两种消息消费方式**

- 推：默认，主动推送给消费者
- 拉：消费者主动从broker拉取Q  （用户通过代码拿到MessageQueue集合并遍历，然后获取Q并记录offset）

那么两种方式哪种更好呢？

- 推：推相较于拉不需要轮询，但是推具有不可靠性
- 拉：可以保证消息拉取动作由消费者自己去发起，只要消费者能够正常工作就可以持续去 broker 拉取新消息，但是弊端是，需要消费者自己去轮询拉取，如果没有新消息时，会导致空轮询浪费 cpu 资源。

#### **集群消费（负载均衡模式）**

消费者采用集群消费方式消费消息，一个分组(Group)下的多个消费者共同消费队列消息，每个消费者处理的消息不同。一个Consumer Group中的各个Consumer实例分摊去(主动写Pull拉取代码或Push被推送)消费消息，即一条消息只会投递到一个Consumer Group下面的一个实例。例如某个Topic有3个队列，其中一个Consumer Group 有 3 个实例，那么每个实例只消费其中的1个队列。集群消费模式是消费者默认的消费方式。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1683530057014/cf9c879d3897424bbf8b332c9f80b071.png)

![image-20240122143617265](E:\图灵课堂\MQ专题\Kafka.assets\image-20240122143617265.png)

这里稍微深入一点，就是集群消费提交的偏移量，持久化是存在broker上的。



![image-20240122144700118](E:\图灵课堂\MQ专题\Kafka.assets\image-20240122144700118.png)

#### **广播消费**

广播消费模式中消息将对一个Consumer Group下的各个Consumer实例都投递（**Push推送**）一遍。即使这些 Consumer属于同一个Consumer Group，消息也会被Consumer Group 中的每个Consumer都消费一次。实际上，是一个消费组下的每个消费者实例都获取到了topic下面的每个Message Queue去拉取消费。所以消息会投递到每个消费者实例。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1683530057014/8a2775e53e514aa898db202f0750effa.png)



这里稍微深入一点，就是广播消费提交的偏移量，持久化是存在客户端（消费者）的。

![image-20240122144725618](E:\图灵课堂\MQ专题\Kafka.assets\image-20240122144725618.png)

![image-20240122144741927](E:\图灵课堂\MQ专题\Kafka.assets\image-20240122144741927.png)

拉取消费者，**拉取的过程比较简单**，发送了一次网络请求去获取数据，可以类比为发起了一次 http 请求去向服务器获取数据，不做过多介绍。

### Push消息流程图

![f1b1f9873f7e3785c40978736b673d1b](E:\图灵课堂\MQ专题\Kafka.assets\f1b1f9873f7e3785c40978736b673d1b.jpeg)

通过 PullMessageService 去轮询，查看阻塞队列 LinkedBlockingQueue<PullRequest> pullRequestQueue 中是否有拉取请求，有的话从队列中弹出拉取请求进行拉取新消息，否则阻塞队列阻塞等待。拉取请求什么时候被放到阻塞队列中？

RebalanceService 会定时 20s 轮询一次查看消费者组中的消费者是否有变化（增/减），例如新建消费者节点时就属于增行为，会重平衡为当前消费者节点分配 queueId ，包装 PullRequest 拉取请求放入到阻塞队列中。

在上一次拉取请求完成后并且消费者消费完成后将拉取请求再次放入到阻塞队列中。

请求到达 broker 的 PullMessageProcessor 后，如果有新消息立即返回，如果没有新消息，会将请求放入到 PullRequestHoldService 中进行等待，等待期间 PullRequestHoldService 会每 1s 轮训一次检查有没有新消息到达。等待有新消息到达或者等待 30s 后进行返回。

## Push方式的启动流程时序图

这一节主要先讲下RocketMQ消费者的启动流程，看下在启动的时候究竟完成了什么样的操作。由于RocketMQ的DefaultMQPushConsumer和DefaultMQPullConsumer启动流程大部分类似，而DefaultMQPushConsumer更为复杂一些，因此我们先只分析DefaultMQPushConsumer启动流程。Push方式的Consumer启动流程的时序图如下图所示：（后续源码部分详细讲解）

![1705906325847](E:\图灵课堂\MQ专题\Kafka.assets\1705906325847.jpg)

时序图上可以看出，Push方式的Consumer启动流程完成的任务比较多，主要任务如下：
（1）设置consumerGroup、NameServer服务地址、消费起始偏移地址并根据参数Topic构建Consumer端的SubscriptionData（订阅关系值）；
（2）在Consumer端注册消费者监听器，当消息到来时完成消费消息；
（3）启动defaultMQPushConsumerImpl实例，主要完成前置校验、复制订阅关系（将defaultMQPushConsumer的订阅关系复制至rebalanceImpl中，包括retryTopic（重试主题）对应的订阅关系）、创建MQClientInstance实例、设置rebalanceImpl的各个属性值、pullAPIWrapper包装类对象的初始化、初始化offsetStore实例并加载消费进度、启动消息消费服务线程以及在MQClientInstance中注册consumer等任务；
（4）启动MQClientInstance实例，其中包括完成客户端网络通信线程、拉取消息服务线程、负载均衡服务线程和若干个定时任务的启动；
（5）向所有的Broker端发送心跳（采用加锁方式）；
（6）最后，唤醒负载均衡服务线程在Consumer端开始负载均衡；



### 两种消费监听方式

#### 消息并发监听



MessageListenerConcurrently：同时开启多个线程并发消费消息。所以这里有可能胡

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1683530057014/4f138c0400c74e779391baa7c8579760.png)

对于消费的结果，有两种方式：1、成功，2、重试。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1683530057014/fd3ec50a4ddc43eebe726d9e4ace8ab9.png)

消费端如果发生消息失败，没有提交成功（或者直接抛出异常、或者返回null值），消息默认情况下会进入重试队列中。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1683530057014/01e6eb2343b74b19a8a6a7fe3c6baf90.png)

**注意重试队列的名字其实是跟消费群组有关，不是主题，因为一个主题可以有多个群组消费，所以要注意**

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1683530057014/3887f09d358246eeb9103d210abd55d4.png)

重试的间隔默认如下：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1683530057014/84cc1a8a8f0d4d1a94528cc9ad6fb57d.png)

这里重试机制的大体逻辑是这样，消息一旦重试，比如消息A，那么这个消息就会进入%RETRY%....这个队列，然后看是第几次重试，达到了重试间隔，这个消息才会进行消费者重新投递。所以这种情况很容易导致消息的无序。

#### 消息顺序监听



MessageListenerOrderly：在同一时刻只允许一个线程消费一个队列的消息，并且保证在消费这个队列消息的顺序性。

玩顺序消息时。consume消费消息失败时，不能返回reconsume——later，这样会导致乱序，应该返回suspend_current_queue_a_moment,意思是先等一会（默认1s），一会儿再处理这批消息，而不是放到重试队列里。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1683530057014/4f536bf82de14f8c81af4df5c23de710.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1683530057014/0fd91d77063b4843862d47a9cf2f541f.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1683530057014/364c8139663b481294c92448813b043e.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1683530057014/5ad80f9c77764ebbabd7de53acde10cb.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1683530057014/71828a3f4c024984bf56787e29dccfe5.png)

该消费线程在消费消息时，会使用锁来保证消息的顺序性。当某个消息消费失败时，该消息所属的队列会进入暂停状态，直到该消息处理成功后才会继续消费下一条消息。

##### 严格使用顺序消息注意事项

但是这里有一个误区，不要认为这么做就可以确保顺序消费，因为这个顺序保障只是确保队列级的。消息在不同的队列中依然是无序的。



所以要做到顺序消费，就必须要创建只能有一个队列的主题。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1683530057014/c98f8e8702a844a891f04d98458664d1.png)

同时就算一个主题只有一个队列，你也要使用顺序消费监听，如果使用并发消费监听一样会有问题。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1683530057014/4457e7388d2943769061f992210f38c4.png)

##### 分区顺序消息

## 批量消息的发送与消费

![image-20240122143932509](E:\图灵课堂\MQ专题\Kafka.assets\image-20240122143932509.png)



如果发送的消息超过了4M，就需要进行拆分。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1683530057014/2b0b35d838534fef89de1e82d21b7d88.png)

## **过滤消息**

有**两种方案**：

​		一种是在 Broker 端按照 Consumer 的去重逻辑进行过滤，这样做的好处是避免了无用的消息传输到 Consumer 端，缺点是加重了 Broker 的负担，实现起来相对复杂。
​		另一种是在 Consumer 端过滤，比如按照消息设置的 tag 去重，这样的好处是实现起来简单，缺点是有大量无用的消息到达了 Consumer 端只能丢弃不处理。
一般采用Cosumer端过滤，如果希望提高吞吐量，可以采用Broker过滤。

对消息的过滤有**三种方式**：

### **T**ag过滤

在大多数情况下，TAG是一个简单而有用的设计，其可以来选择您想要的消息。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1683530057014/cd9e3d7b0eb84ac9b3bb44cb9086ce15.png)

消费者将接收包含TAGA或TAGB或TAGC的消息。但是限制是一个消息只能有一个标签，这对于复杂的场景可能不起作用。在这种情况下，可以使用SQL表达式筛选消息。SQL特性可以通过发送消息时的属性来进行计算。

### **Sql过滤**

SQL表达式过滤更加灵活。需要修改Broker.conf配置文件。加入enablePropertyFilter=true 然后重启Broker服务

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1683530057014/cde4bc1f9ebf4070a3e012c63cc74d92.png)

如果抛出错误：说明Sql92功能没有开启

#### ***SQL基本语法***

RocketMQ定义了一些基本语法来支持这个特性。你也可以很容易地扩展它。

只有使用push模式的消费者才能用使用SQL92标准的sql语句，常用的语句如下：

**数值比较：** 比如：>，>=，<，<=，BETWEEN，=；

**字符比较：** 比如：=，<>，IN；

IS NULL 或者IS NOT NULL；

**逻辑符号：** AND，OR，NOT；

**常量支持类型为：**

数值，比如：123，3.1415；

字符，比如：'abc'，必须用单引号包裹起来；

**NULL** ，特殊的常量

布尔值，TRUE 或FALSE

#### 消息生产者

发送消息时，你能通过putUserProperty来设置消息的属性(Filter)

![image-20240122153407479](E:\图灵课堂\MQ专题\Kafka.assets\image-20240122153407479.png)

#### ***消息消费者*** 

用MessageSelector.bySql来使用sql筛选消息

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1683530057014/468be740138b44b98a5e79b7d76a35fd.png)



```java
DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("please_rename_unique_group_name_4");
// 只有订阅的消息有这个属性a, a >=0 and a <= 3
consumer.subscribe("TopicTest", MessageSelector.bySql("a between 0 and 3");
consumer.registerMessageListener(new MessageListenerConcurrently() {
   @Override
   public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {
       return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
   }
});
consumer.start();
```

- Filter Server 方式：最灵活，也是最复杂的一种方式，允许用户自定义函数进行过滤

## **延时消息**

### **概念介绍**

**延时消息：** Producer 将消息发送到消息队列RocketMQ 服务端，但并不期望这条消息立马投递，而是延迟一定时间后才投递到Consumer 进行消费，该消息即延时消息。

### **适用场景**

消息生产和消费有时间窗口要求：比如在电商交易中超时未支付关闭订单的场景，在订单创建时会发送一条延时消息。这条消息将会在 30 分钟以后投递给消费者，消费者收到此消息后需要判断对应的订单是否已完成支付。如支付未完成，则关闭订单。如已完成支付则忽略。

### **使用方式**

Apache RocketMQ目前只支持固定精度的定时消息，因为如果要支持任意的时间精度，在Broker 层面，必须要做消息排序，如果再涉及到持久化，那么消息排序要不可避免的产生巨大性能开销。（阿里云RocketMQ提供了任意时刻的定时消息功能，Apache的RocketMQ并没有,阿里并没有开源）

RocketMQ是支持延时消息的，只需要在生产消息的时候设置消息的延时级别：

```java
// 实例化一个生产者来产生延时消息
DefaultMQProducer producer = new DefaultMQProducer("ExampleProducerGroup");
// 启动生产者
producer.start();
int totalMessagesToSend = 100;
for (int i = 0; i < totalMessagesToSend; i++) {
    Message message = new Message("TestTopic", ("Hello scheduled message " + i).getBytes());
    // 设置延时等级3,这个消息将在10s之后发送(现在只支持固定的几个时间,详看delayTimeLevel)
    message.setDelayTimeLevel(3);
    // 发送消息
    producer.send(message);
}
```

发送延时消息时需要设定一个延时时间长度，消息将从当前发送时间点开始延迟固定时间之后才开始投递。

延迟消息是根据延迟队列的level来的，延迟队列默认是

msg.setDelayTimeLevel(3)代表延迟10秒

"1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h"

是这18个等级（秒（s）、分（m）、小时（h）），level为1，表示延迟1秒后消费，level为5表示延迟1分钟后消费，level为18表示延迟2个小时消费。生产消息跟普通的生产消息类似，只需要在消息上设置延迟队列的level即可。消费消息跟普通的消费消息一致。

消息进入主题对应的lev队列后，等到延迟时到再转发到目标队列，推送给消费者进行消费  

同时RocketMQ5还支持任意时间的演示

### 具体流程图：

![image-20240122141419573](E:\图灵课堂\MQ专题\Kafka.assets\image-20240122141419573.png)

**源码重点：**

 Broker收到延时消息了，会先发送到主题（SCHEDULE_TOPIC_XXXX）的相应时间段的Message Queue中，然后通过一个定时任务轮询这些队列，到期后，把消息投递到目标Topic的队列中，然后消费者就可以正常消费这些消息。

在写 CommitLog 的时候   

BrokerController 启动之后会 加载 ScheduleMessageService的handler，DefaultMessageStore的start 方法中会调用handleScheduleMessageService.   里面会启动start 

完成转存消息的操作 （不管是主从还是延迟都归它）

CommitLog 的  asyncPutMessage handleScheduleMessageService 消息转存到目标Topic的源码  .     每隔10s执行一个 executeOnTimeup任务 将消息从延迟队列中写入正常Topic中 scheduleMessageService 的DeliverDelayedMessageTimerTask.run.executeOnTimeup方法

 

优点在于  没有了排序。 

​	先发一条lev 5s  ，再发一条3s 因为属于不同的scheduleQueueu所以投递顺序能保证正确

​	如果lev相同 由于处于同一个consumeQueue 也能保持顺序

不足：不同的lev 独立的定时器开销不小。 且lev虽多但还是不够灵活个性化需求无法满足



# RocketMQ流处理

流（Stream）是指一系列连续的数据元素按照特定的顺序组成的数据序列。

流可以是输入流（Input Stream）或输出流（Output Stream），用于表示数据的输入和输出。

在下图中，通过RocketMQ的Streams的处理，可以从source topic（源主题）进行处理，最后把数据加工处理到sink topic（目标主题）

![总体架构](https://rocketmq.apache.org/zh/assets/images/%E6%80%BB%E4%BD%93-1-83fd1dda4e3d43b6852f1805845b9a22.png)

```
 <dependencies>
    <dependency>
        <groupId>org.apache.rocketmq</groupId>
        <artifactId>rocketmq-streams</artifactId>
            <!-- 根据需要修改 -->
        <version>1.1.0</version>
    </dependency>
</dependencies>
```

API文档地址：[https://rocketmq.apache.org/zh/docs/streams/02RocketMQ%20Streams%20Concept#streambuilder-1]()

# RocketMQ性能调优

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1689757354075/03baa95c22cc47f9bd8825247af9368c.png)

## **JVM层面**

### ***STW***

#### ***监控暂停***

 rocketmq-console/rocketmq-dashboard这个是官方提供了一个WEB项目，可以查看rocketmq数据和执行一些操作。但是这个监控界面又没有权限控制，并且还有一些消耗性能的查询操作，如果要提高性能，建议这个可以暂停。

一般的公司在运维方面会有专门的监控组件，如zabbix会做统一处理。或者是简单的shell命令

监控的方式有很多，比如简单点的，我们可以写一个shell脚本，监控执行rocketmqJava进程的存活状态，如果rocketmq crash了，发送告警。

#### ***消除偏向锁***

大家了解，在JDK1.8 sync有偏向锁，但是在RocketMQ都是多线程的执行，所以竞争比较激烈，建议把偏向锁取消，以免没有必要的开销。（默认是禁止了）

-XX:-UseBiasedLocking: 禁用偏向锁

### **垃圾回收**

RocketMQ推荐使用G1垃圾回收器（默认是G1）。

-Xms8g -Xmx8g -Xmn4g:这个就是很关键的一块参数了，也是重点需要调整的，就是默认的堆大小是8g内存，新生代是4g内存。

如果是内存比较大，比如有48g的内存，所以这里完全可以给他们翻几倍，比如给堆内存20g，其中新生代给10g，甚至可以更多些，当然要留一些内存给操作系统来用

-XX:+UseG1GC -XX:G1HeapRegionSize=16m:这几个参数也是至关重要的，这是选用了G1垃圾回收器来做分代回收，对新生代和老年代都是用G1来回收。这里把G1的region大小设置为了16m,这个因为机器内存比较多，所以region大小可以调大一些给到16m，不然用2m的region, 会导致region数量过多。

-XX:G1ReservePercent=25:这个参数是说，在G1管理的老年代里预留25%的空闲内存，保证新生代对象晋升到老年代的时候有足够空间，避免老年代内存都满了，新生代有对象要进入老年代没有充足内存了。默认值是10%，略微偏少，这里RocketMQ给调大了一些

-XX:initiatingHeapOccupancyPercent= :30:这个参数是说，当堆内存的使用率达到30%之后就会自动启动G1的并发垃圾回收，开始尝试回收一些垃圾对象。默认值是45%，这里调低了一些，也就是提高了GC的频率，但是避免了垃圾对象过多，一次垃圾回收耗时过长的问题

-XX:-OmitStackTraceInFastThrow:这个参数是说，有时候JVM会抛弃-些异常堆栈信息，因此这个参数设置之后，就是禁用这个特性，要把完整的异常堆栈信息打印出来。

-XX:+AIwaysPreTouch:这个参数的意思是我们刚开始指定JVM用多少内存，不会真正分配给他，会在实际需要使用的时候再分配给他

所以使用这个参数之后，就是强制让JVM启动的时候直接分配我们指定的内存，不要等到使用内存的时候再分配

-XX:MaxDirectMemorySize=15g:这是说RocketMQ里大量用了NIO中的direct buffer，这里限定了direct buffer最多申请多少，如果你机器内存比较大，可以适当调大这个值，不了解direct buffer是什么，可以自己查看JVM三期。

-XX:-UseLargePages:这个参数的意思是禁用大内存页，某些情况下会导致内存浪费或实例无法启动。默认启动。

## **操作系统层面**

#### ***基本参数***

**vm.overcommit_memory=1**

是否允许内存的过量分配

当为0的时候，当用户申请内存的时候，内核会去检查是否有这么大的内存空间

当为1的时候，内核始终认为，有足够大的内存空间，直到它用完了为止

当为2的时候，内核禁止任何形式的过量分配内存

**vm.** **swappiness=10**

swappiness=0 仅在内存不足的情况下，当剩余空闲内存低于vm.min_free_kbytes limit时，使用交换空间

swappiness=1 内核版本3.5及以上、Red Hat内核版本2.6.32-303及以上，进行最少量的交换，而不禁用交换

swappiness=10 当系统存在足够内存时，推荐设置为该值以提高性能

swappiness=60 默认值

swappiness=100 内核将积极的使用交换空间

**vm.max_max_count=655360**

定义了一个进程能拥有的最多的内存区域，默认为65536

**ulimit=1000000**

limits.conf 设置用户能打开的最大文件数.

1、查看当前大小

ulimit -a

2、临时修改

ulimit -n 1000000

3、永久修改

vim /etc/security/limits.conf

#### ***NIC***

一个请求到RocketMQ的应用，一般会经过网卡、内核空间、用户空间。

**网卡**

网络接口控制器（英语：network interface controller，NIC）

因 Ring Buffer 写满导致丢包的情况很多。当业务流量过大且出现网卡丢包的时候，建议调整Ring Buffer的大小，这个大小的设置一定程度上是可以缓解丢包的状况。

在Linux操作系统中，可以通过修改网络接口的Ring Buffer大小来调整其大小。

1. 使用 `ifconfig` 或 `ip` 命令查看当前网络接口的配置信息，找到要调整Ring Buffer大小的接口名称。
2. 使用 `ethtool` 命令来查看和修改网络接口的参数。

例如，使用以下命令查看当前Ring Buffer大小：

```
ethtool -g 
```

其中， `<interface_name>` 是要调整Ring Buffer大小的网络接口名称。

3. 使用 `ethtool -G` 命令来修改Ring Buffer大小。例如，使用以下命令将Ring Buffer大小设置为新值：

```
ethtool -G <interface_name> rx <new_size> tx <new_size>
```

其中， `<new_size>` 是要设置的新的Ring Buffer大小。

4. 使用 `ethtool` 命令再次验证Ring Buffer大小是否已经成功修改。

#### ***Kernel***

##### 中断聚合

在中断（IRQ），

在操作系统级别，是可以做软中断聚合的优化。

什么是中断？

举例，假如你是一位开发同学，和你对口的产品经理一天有10个小需求需要让你帮忙来处理。她对你有两种中断方式：

· 第一种：产品经理想到一个需求，就过来找你，和你描述需求细节，然后让你帮你来改

· 第二种：产品经理想到需求后，不来打扰你，等攒够5个来找你一次，你集中处理

我们现在不考虑及时性，只考虑你的工作整体效率，你觉得那种方案下你的工作效率会高呢？或者换句话说，你更喜欢哪一种工作状态呢？

很明显，只要你是一个正常的开发，都会觉得第二种方案更好。对人脑来讲，频繁的中断会打乱你的计划，你脑子里刚才刚想到一半技术方案可能也就废了。当产品经理走了以后，你再想捡起来刚被中断之的工作的时候，很可能得花点时间回忆一会儿才能继续工作。

对于CPU来讲也是一样，CPU要做一件新的事情之前，要加载该进程的地址空间，load进程代码，读取进程数据，各级别cache要慢慢热身。因此如果能适当降低中断的频率，多攒几个包一起发出中断，对提升CPU的工作效率是有帮助的。所以，网卡允许我们对硬中断进行合并。

##### 网卡队列CPU绑定

现在的主流网卡基本上都是支持多队列的，我们可以通过将不同的队列分给不同的CPU核心来处理，从而加快Linux内核处理网络包的速度。这是最为有用的一个优化手段。

每一个队列都有一个中断号，可以独立向某个CPU核心发起硬中断请求，让CPU来poll包。通过将接收进来的包被放到不同的内存队列里，多个CPU就可以同时分别向不同的队列发起消费了。这个特性叫做RSS（Receive Side Scaling，接收端扩展）。通过ethtool工具可以查看网卡的队列情况。

##### 关闭IRQBalance

IRQBalance主要功能是可以合理的调配使用各个CPU核心，特别是对于目前主流多核心的CPU，简单的说就是能够把压力均匀的分配到各个CPU核心上，对提升性能有很大的帮助。

但实际中往往影响cpu的使用均衡，建议服务器环境中关闭

##### net.core.dev_weight

每个CPU一次NAPI中断能够处理网络包数量的最大值，可以根据实际情况调整。

##### TCP NODEALY

Nagle算法用于对缓冲区内的一定数量的消息进行自动连接。该处理过程(称为Nagling)，通过减少必须发送的封包的数量，提高了网络应用 程序系统的效率。（Nagle虽然解决了小封包问题，但也导致了较高的不可预测的延迟，同时降低了吞吐量。）

RocketMQ通讯层已经禁止了

##### 缓冲区调整

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1689757354075/8ba18280427a45c7b9bafeaef0843f37.png)



### RocketMQ的源码整体分析

解读的源码版本基于RocketMQ5.1.0

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1684485576063/618d3360184241a596e0b5e661d8f9c5.png)

RocketMQ的源码是非常的多，我们没有必要把RocketMQ所有的源码都读完，所以我们把核心、重点的源码进行解读，RocketMQ核心流程如下：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1684485576063/96b0e98b71b144a7b0bb39b6e0311eda.png)

## 1、NameServer的模块

命名服务，更新和路由发现 broker服务。
NameServer 要作用是为消息生产者、消息消费者提供关于主题 Topic 的路由信息，NameServer除了要存储路由的基础信息，还要能够管理 Broker节点，包括路由注册、路由删除等功能

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1684485576063/2b1453f87b1842c586c797d45234ba75.png)

## 2、Produce/Consumer模块

在源码中属于client子模块，java版本的mq客户端实现

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1684485576063/c9a8a37fcbf945fe9626734b8888968f.png)

## 3、broker模块

mq的核心。它能接收producer和consumer的请求，并调用store层服务对消息进行处理。HA服务的基本单元，支持同步双写，异步双写等模式。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1684485576063/c84fa0b38ec54fdda05a659ed16f6e06.png)

## 4、Remote模块

基于netty的底层通信实现，所有服务间的交互都基于此模块。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1684485576063/f625a6142b5a409abac858363d0a5363.png)

# NameServer源码分析

从业务流程上，我们先不看源码，我们来分析一下NameSerever要干哪些活！然后基于业务点进行源码分析。

1、启动作为注册中心

启动监听，等待Broker、Producer、Comsumer连接来注册。

2、路由注册与发现

Broker在启动时向所有NameServer注册（包括主题队列信息、broker信息、集群信息、broker存活信息、过滤服务器信息、broker和主题队列映射信息）

3、路由剔除

移除心跳超时的Broker相关路由信息。NameServer与每台Broker服务保持长连接，并间隔10S检查Broker是否存活，如果检测到Broker宕机，则从路由注册表中将其移除。这样就可以实现RocketMQ的高可用。

## NameServer的启动流程分析

启动入口，NameServer是以main方法启动的。

这里要注意一个点，就是NameServer所有的存储的信息都是基于内存，而这些信息的来源都是broker启动的时候发过来的，所以可以这么认为，NameServer就是一个内存版的zookeeper，一个精简版的zookeeper。

main0 方法中核心流程如下：

1. 创建 NamesrvController
2. 启动 NamesrvController
3. 启动成功后打印 The Name Server boot success. serializeType=JSON，打印序列化类型，RocketMQ 提供的序列化类型有两种：JSON 和 ROCKETMQ

启动分成了两块：

1. NameServer启动
2. Controller启动(5.0为自动自主切换新增的一个模块，内嵌NameServer的时候会启动)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1684485576063/4296402e0b3f4e0e83f9f7a1cb76f6c0.png)

![image-20240122173610412](E:\图灵课堂\MQ专题\Kafka.assets\image-20240122173610412.png)

NameServer存储的核心类:基于ConcurrentHashMap存储。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1684485576063/b58281ef25864a32870e8c0107b99ec9.png)

![image-20240122180356632](E:\图灵课堂\MQ专题\Kafka.assets\image-20240122180356632.png)



#### 初始化NamesrvController

首先调用NamesrvController#initialize进行初始化

```java
public boolean initialize() {
        loadConfig();
        initiateNetworkComponents();  //创建NameServer的网络服务，以及NameServer的客户端。
    /**
    这里初始化了两个线程池：
	clientRequestExecutor线程池处理客户端(生产者和消费者)获取Topic的路由信息(RequestCode.GET_ROUTEINFO_BY_TOPIC)
	defaultExecutor线程池处理除了RequestCode.GET_ROUTEINFO_BY_TOPIC以外的请求。
    **/
        initiateThreadExecutors();
    //将线程池和处理器绑定。 Rocketmq5.0版本对处理器进行了线程池隔离，将获取路由相关的处理和其他的处理例如Broker的注册进行线程池的隔离。
        registerProcessor();
    //启动三个定时任务，两个是打印的的定时任务没有业务逻辑，只有scanNotActiveBroker定时任务的作用：默认每5秒扫描一次Broker是否过期
        startScheduleService();
    //初始化SsL
        initiateSslContext();
    //目前只注册了一个ZoneRouteRPCHook，主要用于区域路由。
        initiateRpcHooks();
        return true;
    }
```

#### 然后ControllerManager启动start方法

```java
//在文件监控服务不为空的情况下启动服务。 路由管理服务启动： 主要是启动了批量注销服务。到这里整个服务就已经启动完成。
public void start() throws Exception {
    this.remotingServer.start();

    // In test scenarios where it is up to OS to pick up an available port, set the listening port back to config
    if (0 == nettyServerConfig.getListenPort()) {
        nettyServerConfig.setListenPort(this.remotingServer.localListenPort());
    }

    this.remotingClient.updateNameServerAddressList(Collections.singletonList(NetworkUtil.getLocalAddress()
        + ":" + nettyServerConfig.getListenPort()));
    this.remotingClient.start();

    if (this.fileWatchService != null) {
        this.fileWatchService.start();
    }

    this.routeInfoManager.start();
}
```

#### 总结

1. 启动参数解析：NameServer 启动时需要指定一些参数，例如监听端口、RocketMQ 集群的名称等等。NameServer 会先解析这些参数，并根据这些参数进行初始化。
2. 加载配置文件：NameServer 还会加载配置文件，包括 broker 配置、路由配置、Topic 配置等等，这些配置文件可以指定在启动参数中，也可以在启动后进行修改。
3. 创建 MBeanServer：NameServer 还会创建一个 MBeanServer，用于对 NameServer 进行监控和管理。
4. 启动 Netty 服务端：NameServer 的主要功能是接收 Broker 节点的注册请求和心跳信息，并维护 Broker 节点的状态。为此，NameServer 会启动一个 Netty 服务端，用于接收和处理这些请求。
5. 注册 ShutdownHook：NameServer 还会注册一个 ShutdownHook，用于在 NameServer 关闭时执行一些清理工作，例如关闭 Netty 服务端、保存路由信息等等。
6. 初始化定时任务：NameServer 还会初始化一些定时任务，例如定时刷新路由信息、定时清理过期的 Broker 节点等等。这些定时任务是通过 Java 自带的 ScheduledExecutorService 实现的。
7. 启动完成：最后，NameServer 启动完成，并等待 Broker 节点的注册和心跳信息。
    以上就是 RocketMQ NameServer 的启动流程。需要注意的是，RocketMQ 集群中至少需要一个 NameServer 节点，多个 NameServer 节点可以提高系统的可用性和容错性。

## NameServer的对外提供的功能

NameServer主要对外提供两类功能：

1、注册/查询Broker(维护Broker的服务地址并进行即时的更新)

2、注册/查询Topic路由信息(给Producer和Consumer提供服务获取Broker列表)

源码中的核心就是RouteInfoManager类提供的对外方法。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1684485576063/e2ff6248fc744634a7690dd7fa715d5b.png)

### 注册/查询Broker

注册

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1684485576063/677e6cd38b674a5093898e1a1820f6bd.png)

查询

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1684485576063/edd45585367a42338d78c1cb82934baf.png)

### 注册/查询Topic路由信息

注册

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1684485576063/212f258b98124889904897d917b32553.png)

查询

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1684485576063/d5b04c9074c6438c8f221f19231006ff.png)

### 其他组件调用NameServer的功能

#### 注册/查询Broker

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1684485576063/0539fff522f748c78c4cff1077a3d515.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1684485576063/3bf83d14ef3f43469bfb18483e2c88d5.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1684485576063/cc807e1c5dd848f7a86c8b3c49db2fa1.png)

#### 注册/查询Topic路由信息

注册Topic路由信息

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1684485576063/ec0058353130407f8e25bf228391fd94.png)

查询Topic路由信息

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1684485576063/308910eba6554a43989eb1a8dd51fe10.png)

### 读写锁设计

RouteInfoManager类中有一个读写锁的设计

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1684485576063/80e62d9bfd42403e867185df3a5ab133.png)

注册拿写锁

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1684485576063/2ce6f0f24616464a801f1f7220b8c624.png)

查询拿读锁

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1684485576063/97d30ac1d9ca4731b0ad40e4fc90434b.png)

同时包括Broker的注册与查询。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1684485576063/1a7520ccbd2843e7b6290387bc73ab88.png)

RocketMQ的RouteInfoManager是用来管理NameServer中的路由信息的，包括Topic和Broker之间的映射关系、Broker和集群之间的映射关系等。由于大量的生产者和消费者都需要访问这些路由信息，因此RouteInfoManager需要支持高并发读取操作。

对于读取路由信息的操作，RouteInfoManager使用读锁进行保护，多个线程可以同时获取读锁进行并发读取。而对于更新路由信息的操作，RouteInfoManager使用写锁进行保护，只允许单个线程获取写锁进行更新，避免了并发更新操作导致的数据一致性问题。

### 定时任务剔除超时Broker

核心控制器会启动定时任务： 每隔1s扫描一次Broker,移除不活跃的Broker。

Broker每隔1s向NameServer发送一个心跳包，心跳包包含BrokerId，Broker地址，Broker名称，Broker所属集群名称、Broker关联的FilterServer列表。但是如果Broker宕机，NameServer无法收到心跳包，

此时NameServer如何来剔除这些失效的Broker呢？

NameServer会每隔5s扫描brokerLiveTable状态表，如果BrokerLive的**lastUpdateTimestamp**的时间戳距当前时间超过120s，则认为Broker失效，移除该Broker，关闭与Broker连接，同时更新topicQueueTable、brokerAddrTable、brokerLiveTable、filterServerTable。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1684485576063/81fccb231cfd4718afcb198c2ebd5d7d.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1684485576063/f210e6c2ffdc49419ae1e14302644cc9.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1684485576063/05d669aa4a7940509fdd0b0e4d607901.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1684485576063/3dac33b9f8754052b579ab6a47f2756a.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1684485576063/faa90354bda543c899fba96a66d71282.png)


路由剔除机制中，Borker每隔30S向NameServer发送一次心跳，而NameServer是每隔10S扫描确定有没有不可用的主机（120S没心跳），那么问题就来了！这种设计是存在问题的，就是NameServer中认为可用的Broker，实际上已经宕机了，那么，某一时间段，从NameServer中读到的路由中包含了不可用的主机，会导致消息的生产/消费异常，不过不用担心，在生产和消费端有故障规避策略及重试机制可以解决以上问题（原理后续源码解读）。这个设计符合RocketMQ的设计理念：整体设计追求简单与性能，同时这样设计NameServer是可以做到无状态化的，可以随意的部署多台，其代码也非常简单，非常轻量。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1648432544069/8197b7cf417440158c2c745a165fcf71.png)

**RocketMQ有两个触发点来删除路由信息：**

* NameServer定期扫描brokerLiveTable检测上次心跳包与当前系统的时间差，如果时间超过10s，则需要移除broker。

* Broker在正常关闭的情况下，会执行unregisterBroker指令这两种方式路由删除的方法都是一样的，都是从相关路由表中删除与该broker相关的信息。

  在消费者启动之后，第一步都要从NameServer中获取Topic相关信息





#### 总结流程图：

![image-20240122140422052](E:\图灵课堂\MQ专题\Kafka.assets\image-20240122140422052.png)



## **Broker 的注册流程**

RouteInfoManager#registerBroker() 方法进行 Broker 的注册

1. 加写锁，RouteInfoManager 管理元数据内存结构读的并发一般是大于写的并发的，所以通过读写锁来保证并发安全性的前提下，还可以提升读的性能。
2. 根据 Broker 所属的集群名称 clusterName，从 clusterAddrTable 这个 Map 中获取 BrokerName 的集合，如果没有的话就创建一个，然后把 BrokerName 添加进去。
3. 维护 brokerAddrTable，根据 BrokerName 获取 Broker 的信息，如果是第一次维护的话，registerFirst 设置为 true，然后创建 BrokerData，并且维护到 brokerAddrTable 中。
4. 从 BrokerData 中获取当前所有 Broker 的地址信息，然后进行遍历，判断一下这个 BrokerId 是否正常，因为可能出现主从切换，原先是主节点，brokerId 是 0 ，现在变为从节点 brokerId 变为 2，那需要将这个数据移除掉之后重新维护进来
5. 维护 BrokerData 的地址数据，并且获取 oldAddr，然后校验一下是否是第一次注册。
6. 维护 **topicQueueTable** 数据结构，如果是主节点，Topic 数据有变更或者是Broker第一次注册，需要重新维护一下 TopicQueueTable 的数据。
7. 维护 Broker 的心跳信息到 BrokerLiveTable 中
8. 如果注册的 Broker 是 Slave 节点，查找对应的 Master 节点信息并返回 Master 的地址。

![v2-f8b0260b54a5a68acd40f8af0a2cb5b8_r](E:\图灵课堂\MQ专题\Kafka.assets\v2-f8b0260b54a5a68acd40f8af0a2cb5b8_r.jpg)

## **Broker 下线流程**

Broker 下线调用的是 RouteInfoManager#unregisterBroker() 方法，主要就是 Broker 下线从数据结构中把响应的内容给移除掉。

摘除 **BrokerLiveTable** 心跳信息

移除 **brokerAddrTable** 下面的地址，如果地址已经空了，就将 broker 从 **brokerAddrTable** 进行移除。

摘除掉 Broker 之后，从集群中摘除该 Broker，如果集群中没有 Broker 了，从 **clusterAddrTable** 把集群给移除掉。

根据 BrokerName 移除 **topicQueueTable** 下面包含该 Broker 的信息，如果 Topic 下面的 Broker 已经空了，就将 Topic 从 **topicQueueTable** 进行摘除。

![image-20240122174637132](E:\图灵课堂\MQ专题\Kafka.assets\image-20240122174637132.png)

## **Broker 心跳检测**

启动 NamesrvController 的流程时候有讲到过 NameServer 有启动一个每 10s 执行一次的心跳检测任务，调用的是 scanNotActiveBroker() 方法，我们分析一下这块的源码，如下图所示：

![image-20240122174501845](E:\图灵课堂\MQ专题\Kafka.assets\image-20240122174501845.png)

1. 遍历 **brokerLiveTable** 获取 Broker 所有的心跳信息
2. 获取 Broker 最后一次更新的心跳时间，加上 BROKER_CHANNEL_EXPIRED_TIME（默认是 120s），如果小于当前时间，就认为 Broker 已经断开，关闭 NameServer 与 Broker 的 Channel 连接，然后将心跳从 **brokerLiveTable** 中移除掉。

### 总结：几个关键：

#### NameServer 启动流程是什么样的？会创建哪些核心数据结构？

NameServer 通过 NamesrvStartup 进行启动，主要是对 NamesrvController 的创建、初始化、启动的流程。NamesrvController 中最重要的变量 RouteInfoManager 和 RemotingServer。

RouteInfoManager 中包含了下面的几种数据结构：

- **topicQueueTable**：包含着 Topic 与 Broker、Queue 的数量之间的关系。
- **brokerAddrTable**：存储 BrokerName 与 BrokerAddr 之间的关系，也就是一个 Broker 分组的各个地址信息。
- **clusterAddrTable**：存储集群与 Broker 之间的关系。
- **brokerLiveTable**：存储每个 Broker 地址的心跳信息。

#### NameServer 以什么样的数据结构存储着 Broker 与路由信息的？

最核心的两个数据结构：**topicQueueTable** 和 **brokerAddrTable**，一个根据 Topic 获取 Broker 信息，一个是根据 BrokerName 获取 Broker 地址信息。

#### **Broker 上线、下线、发送心跳这些操作在 NameServer 中是如何进行的**？

上线、下线、发送心跳都需要加写锁，然后维护 RouteInfoManager 这里面的数据结构

#### **NameServer 是如何进行 Broker 心跳检测的**？

NameServer 启动的时候会开启一个定时调度线程，每 10s 执行一次，对 **brokerLiveTable** 的心跳时间进行检测，如果上一次心跳时间距离当前时间超过 120s，就认为 Broker 的连接断开，需要从 **brokerLiveTable** 中移除该 Broker，并且移除掉 RouteInfoManager 中与该 Broker 相关的数据信息。

关于 NameServer 的网络知识，后续会结合 remoting 模块的源码进行深入剖析。

# Broker源码分析

**功能回顾：**

核心组件，所有消息的存储 转发 都在此进行。

内部架构，类似于JavaWeb的MVC，有Controller负责响应请求，各种Service组件负责处理具体业务，Dao就相当于 这里负责消息存盘的功能。

Broker是如何像他注册信息，发送心跳维护的。

### 总体流程图：

![image-20240122140728404](E:\图灵课堂\MQ专题\Kafka.assets\image-20240122140728404.png)

源码重点：

BrokerStartup createBrokerController

**源码重点：**

前面nameserver其实已简单讲解过，需要深入可跟进 boolean initResult = controller.initialize();

 加载磁盘配置文件 加载之后的处理 消息持久化管理组件 DLeger技术，会初始化其相关组件 Broker状态统计组件 load plugin 加载插件 （消息分发相关后期进入） Broker的Netty组件  注意即server又client  各种组件的线程池 发送 恢复 查询 管理 心跳 事务 等 各种定时任务  记录状态 offset持久化 filter过滤器（上推） 保护水位 DLeger相关  TLS安全相关 事务权限RPC的SPI拓展

# **Producer**源码分析

**功能回顾：**

普通消息发送者 DefaultMQProducer（优先） 只需要构建一个Netty客户端，往Broker发送消息就行。注意异步回调是在Producer接受到Broker的响应后自动调整流程不需要提供Netty服务

事务消息发送者：TransactionMQProducer 需要构建Netty客户端，往Broker发送消息的同时 还需要构建Netty服务端，往Broker回查本地事务的状态。

![image-20240122163520178](E:\图灵课堂\MQ专题\Kafka.assets\image-20240122163520178.png)

可简单回顾下消息设计：

![image-20240122163738703](E:\图灵课堂\MQ专题\Kafka.assets\image-20240122163738703.png)



### 总体流程图：

![image-20240122140838226](E:\图灵课堂\MQ专题\Kafka.assets\image-20240122140838226.png)

还有Producer的负载均衡

默认会把消息平均发送到所有的MssageQueue里，获取路由信息后会选出一个MessageQueue发送消息，按 索引自增然后取模的方法 （selectOneMessageQueue()）

核心源码流程图：

![image-20240122163506280](E:\图灵课堂\MQ专题\Kafka.assets\image-20240122163506280.png)



**源码重点：**

example项目的Producer   producer.start(); 以及 send(); 负载均衡转入 Producer

```java
@Override
 public SendResult send(
        Message msg) throws MQClientException, RemotingException, MQBrokerException, InterruptedException {
        return this.defaultMQProducerImpl.send(msg);
    }
...
@Override
public SendResult send(Message msg,
        long timeout) throws MQClientException, RemotingException, MQBrokerException, InterruptedException {
        return this.defaultMQProducerImpl.send(msg, timeout);
    }
...
@Override
public void send(Message msg,
        SendCallback sendCallback) throws MQClientException, RemotingException, InterruptedException {
        this.defaultMQProducerImpl.send(msg, sendCallback);
    }
```

先会进行消息校验

```java
public static void checkMessage(Message msg, DefaultMQProducer defaultMQProducer)
        throws MQClientException {
        //消息不能为空
        if (null == msg) {
            throw new MQClientException(ResponseCode.MESSAGE_ILLEGAL, "the message is null");
        }
        // 主题不能为空
        Validators.checkTopic(msg.getTopic());

        // 消息的body不能为空
        if (null == msg.getBody()) {
            throw new MQClientException(ResponseCode.MESSAGE_ILLEGAL, "the message body is null");
        }
    // 消息的body长度不能为0
        if (0 == msg.getBody().length) {
            throw new MQClientException(ResponseCode.MESSAGE_ILLEGAL, "the message body length is zero");
        }

    // 消息的body长度不能唱过最大长度4M
        if (msg.getBody().length > defaultMQProducer.getMaxMessageSize()) {
            throw new MQClientException(ResponseCode.MESSAGE_ILLEGAL,
                "the message body size over max value, MAX: " + defaultMQProducer.getMaxMessageSize());
        }
    }
```

消息的校验没有问题则会调用`tryToFindTopicPublishInfo(msg.getTopic())`方法。我们需要获取主题的路由信息，通过路由信息我们才知道消息需要被投递到哪个具体的`Broker`节点之上。

我们来一起看下查找主题的路由信息方法，如下所示：

```java
private TopicPublishInfo tryToFindTopicPublishInfo(final String topic) {
        //先从本地缓存变量topicPublishInfoTable中先get一次
        TopicPublishInfo topicPublishInfo = this.topicPublishInfoTable.get(topic);
        if (null == topicPublishInfo || !topicPublishInfo.ok()) {
            this.topicPublishInfoTable.putIfAbsent(topic, new TopicPublishInfo());
         //然后从nameServer上更新topic路由信息   
         this.mQClientFactory.updateTopicRouteInfoFromNameServer(topic);
         //然后再从本地缓存变量topicPublishInfoTable中再get一次
            topicPublishInfo = this.topicPublishInfoTable.get(topic);
        }

        if (topicPublishInfo.isHaveTopicRouterInfo() || topicPublishInfo.ok()) {
            return topicPublishInfo;
        } else {      
//第一次的时候isDefault为false，第二次的时候default为true，即为用默认的topic的参数进行更新
            this.mQClientFactory.updateTopicRouteInfoFromNameServer(topic, true, this.defaultMQProducer);
            topicPublishInfo = this.topicPublishInfoTable.get(topic);
            return topicPublishInfo;
        }
    }
```

在生产者中如果缓存了`topic`路由信息，路由信息中如果包含了消息队列，那么会进行路由信息的返回。如果没有缓存或者没有队列信息，那么就会向`NameServer`查询`topic`的路由信息。`TopicPublishInfo`属性如下所示：

```java
public class TopicPublishInfo {
    //是否是顺序消息
    private boolean orderTopic = false;
    //是否包含主题路由信息
    private boolean haveTopicRouterInfo = false;
    //主题队列的消息队列
    private List<MessageQueue> messageQueueList = new ArrayList<MessageQueue>();
    // 没选择一次消息队列，值自增1
    private volatile ThreadLocalIndex sendWhichQueue = new ThreadLocalIndex();
    
    private TopicRouteData topicRouteData;

    public boolean isOrderTopic() {
        return orderTopic;
    }

    public void setOrderTopic(boolean orderTopic) {
        this.orderTopic = orderTopic;
    }

    public boolean ok() {
        return null != this.messageQueueList && !this.messageQueueList.isEmpty();
    }

    public List<MessageQueue> getMessageQueueList() {
        return messageQueueList;
    }

    public void setMessageQueueList(List<MessageQueue> messageQueueList) {
        this.messageQueueList = messageQueueList;
    }

    public ThreadLocalIndex getSendWhichQueue() {
        return sendWhichQueue;
    }

    public void setSendWhichQueue(ThreadLocalIndex sendWhichQueue) {
        this.sendWhichQueue = sendWhichQueue;
    }

    public boolean isHaveTopicRouterInfo() {
        return haveTopicRouterInfo;
    }

    public void setHaveTopicRouterInfo(boolean haveTopicRouterInfo) {
        this.haveTopicRouterInfo = haveTopicRouterInfo;
    }

    public MessageQueue selectOneMessageQueue(final String lastBrokerName) {
        if (lastBrokerName == null) {
            return selectOneMessageQueue();
        } else {
            int index = this.sendWhichQueue.getAndIncrement();
            for (int i = 0; i < this.messageQueueList.size(); i++) {
                int pos = Math.abs(index++) % this.messageQueueList.size();
                if (pos < 0)
                    pos = 0;
                MessageQueue mq = this.messageQueueList.get(pos);
                if (!mq.getBrokerName().equals(lastBrokerName)) {
                    return mq;
                }
            }
            return selectOneMessageQueue();
        }
    }

    public MessageQueue selectOneMessageQueue() {
        int index = this.sendWhichQueue.getAndIncrement();
        int pos = Math.abs(index) % this.messageQueueList.size();
        if (pos < 0)
            pos = 0;
        return this.messageQueueList.get(pos);
    }

    public int getQueueIdByBroker(final String brokerName) {
        for (int i = 0; i < topicRouteData.getQueueDatas().size(); i++) {
            final QueueData queueData = this.topicRouteData.getQueueDatas().get(i);
            if (queueData.getBrokerName().equals(brokerName)) {
                return queueData.getWriteQueueNums();
            }
        }

        return -1;
    }

    @Override
    public String toString() {
        return "TopicPublishInfo [orderTopic=" + orderTopic + ", messageQueueList=" + messageQueueList
            + ", sendWhichQueue=" + sendWhichQueue + ", haveTopicRouterInfo=" + haveTopicRouterInfo + "]";
    }

    public TopicRouteData getTopicRouteData() {
        return topicRouteData;
    }

    public void setTopicRouteData(final TopicRouteData topicRouteData) {
        this.topicRouteData = topicRouteData;
    }
}
```

## 

# **消息存储**源码分析



**功能回顾：**

上面的Producer发送到Broker之后，Broker接受到消息如何存储的呢？最终文件有哪些？

**commitLot** 消息存储目录（消息生成写入）

config 运行期间的一些配置信息

**consumerqueue** 消息消费队列存储目录（消息消费从此找）

index 消息索引文件存储目录

abort 关于文件寿命Broker非正常关闭

checkpoint 文件检查点 存储CommitLog文件最后一次刷盘的时间戳

Broker核心组件中MessageStore就是负责消息存储的核心组件。

### 存储结构图：

![image-20240122171836776](E:\图灵课堂\MQ专题\Kafka.assets\image-20240122171836776.png)

消息生成先写CommitLog

异步线程生成msg的offset size hash 放入Comsumequeue。完成主题队列相应关系

接下来再看下详细的

### 流程图：

![1705915758460](E:\图灵课堂\MQ专题\Kafka.assets\1705915758460-1705915898169.jpg)

**消息发送与消费流程：**

![image-20240122141015607](E:\图灵课堂\MQ专题\Kafka.assets\image-20240122141015607.png)



**源码重点：**

1.commitLog写入  doAppend入口 写入消息，最终会将追加到MappedFile隐射的一块内存中，并没有写入磁盘。写入消息的过程是串行的，一次只会允许一个线程写入。

package org.apache.rocketmq.store; public Class DefaultMessageStore //生产者普通消息的 同步步存储方法 public PutMessageResult putMessage(MessageExtBrokerInner msg)   以及异步 和异步批次              

 2.分发ConsumerQueue和IndexFile

​     底层逻辑：

​     当CommitLog写入一条消息后，入口在DefaultMessageStore的start方法中，(Broker启动的时候调用)

​     会启动一个后台线程reputMessageService每隔1s拉取CommitLoh最新更新的

​    一批消息，然后分别转发到ComsumerQueue和IndexFile里。 

​    如果服务宕机，会造成文件不一致，有消息写入CommitLog，没有分发到

​    索引文件IndexFile，存在消息丢失的情况。DefaultMappedStore的load方法

​    提供了恢复索引文件的逻辑，在load方法

3.文件刷盘机制 

入口 CommitLog.putMessage >  handleDiskFlush

其中主要设计到是否开启了对外内存。TransientStorePoolEnable，如果开启了对外内存，会启动时申请一个跟CommitLog文件大小一致的对外内存，这部分内存可以确保不会被交换到虚拟机内存中

4.过期文件处理/删除

入口：DefaultMessageStore.addScheduleTask > cleanFilesPeriodically()

默认情况下 Broker会启动后台线程  每60s检查 CommitLog，ConsumeQueue文件，然后对超过72小时（3天）的数据进行删除。可以配置fileReservedTime保留时间，注意删除时不会检查是否被消费过

5.整体 文件存储部分

包含 消息文件commitlog 消息消费队列文件 consumerqueue ，hash索引文件IndexFile，监测点文件checkPoint，abort关闭异常文件。



### 刷盘策略

分为同步刷盘结合主从异步复制 和 异步刷盘结合主从同步复制

入口 CommitLog.putMessage >  handleDiskFlush

其中主要设计到是否开启了对外内存。TransientStorePoolEnable，如果开启了对外内存，会启动时申请一个跟CommitLog文件大小一致的对外内存，这部分内存可以确保不会被交换到虚拟机内存中

Broker通过CommitLog类来完成数据的落盘工作，对于前面的流程我们直接略过，直接从关键方法putMessage(….)开始。

```java
public class CommitLog { 
    ......
    /** * 添加消息，返回消息结果 * * @param msg 消息 * @return 结果 */
    public PutMessageResult putMessage(final MessageExtBrokerInner msg) {
        ......
        // 获取写入映射文件
        MappedFile unlockMappedFile = null;
        MappedFile mappedFile = this.mappedFileQueue.getLastMappedFile();
        // 获取追加锁,限制同一时间只能有一个线程进行数据的Put工作
        lockForPutMessage();                                           //##1
        try {
            long beginLockTimestamp = this.defaultMessageStore.getSystemClock().now();
            this.beginTimeInLock = beginLockTimestamp;
            // Here settings are stored timestamp, in order to ensure an orderly
            // global
            msg.setStoreTimestamp(beginLockTimestamp);
            // 当不存在映射文件或者文件已经空间已满，进行创建
            if (null == mappedFile || mappedFile.isFull()) {
                mappedFile = this.mappedFileQueue.getLastMappedFile(0); // Mark: NewFile may be cause noise
            }
            if (null == mappedFile) {
                log.error("create maped file1 error, topic: " + msg.getTopic() + " clientAddr: " + msg.getBornHostString());
                beginTimeInLock = 0;
                return new PutMessageResult(PutMessageStatus.CREATE_MAPEDFILE_FAILED, null);
            }
            // 将消息追加到MappedFile的MappedByteBuffer/writeBuffer中,更新其写入位置wrotePosition,但还没Commit及Flush
            result = mappedFile.appendMessage(msg, this.appendMessageCallback);           //##2
            switch (result.getStatus()) {
                case PUT_OK:
                    break;
                case END_OF_FILE: // 当文件剩余空间不足以插入当前消息时,创建新的MapperFile,进行插入
                    unlockMappedFile = mappedFile;
                    // Create a new file, re-write the message
                    mappedFile = this.mappedFileQueue.getLastMappedFile(0);
                    if (null == mappedFile) {
                        // XXX: warn and notify me
                        log.error("create maped file2 error, topic: " + msg.getTopic() + " clientAddr: " + msg.getBornHostString());
                        beginTimeInLock = 0;
                        return new PutMessageResult(PutMessageStatus.CREATE_MAPEDFILE_FAILED, result);
                    }
                    result = mappedFile.appendMessage(msg, this.appendMessageCallback);
                    break;
                case MESSAGE_SIZE_EXCEEDED:
                case PROPERTIES_SIZE_EXCEEDED:
                    beginTimeInLock = 0;
                    return new PutMessageResult(PutMessageStatus.MESSAGE_ILLEGAL, result);
                case UNKNOWN_ERROR:
                    beginTimeInLock = 0;
                    return new PutMessageResult(PutMessageStatus.UNKNOWN_ERROR, result);
                default:
                    beginTimeInLock = 0;
                    return new PutMessageResult(PutMessageStatus.UNKNOWN_ERROR, result);
            }
            eclipseTimeInLock = this.defaultMessageStore.getSystemClock().now() - beginLockTimestamp;
            beginTimeInLock = 0;
        } finally {
            // 释放锁
            releasePutMessageLock();
        }
        if (eclipseTimeInLock > 500) {
            log.warn("[NOTIFYME]putMessage in lock cost time(ms)={}, bodyLength={} AppendMessageResult={}", eclipseTimeInLock, msg.getBody().length, result);
        }
        if (null != unlockMappedFile && this.defaultMessageStore.getMessageStoreConfig().isWarmMapedFileEnable()) {
            this.defaultMessageStore.unlockMappedFile(unlockMappedFile);
        }
        PutMessageResult putMessageResult = new PutMessageResult(PutMessageStatus.PUT_OK, result);
        // Statistics
        storeStatsService.getSinglePutMessageTopicTimesTotal(msg.getTopic()).incrementAndGet();
        storeStatsService.getSinglePutMessageTopicSizeTotal(topic).addAndGet(result.getWroteBytes());
        // 进行同步||异步 flush||commit
        GroupCommitRequest request = null;
        // Synchronization flush
        if (FlushDiskType.SYNC_FLUSH == this.defaultMessageStore.getMessageStoreConfig().getFlushDiskType()) {
            final GroupCommitService service = (GroupCommitService)this.flushCommitLogService;
            if (msg.isWaitStoreMsgOK()) {
                request = new GroupCommitRequest(result.getWroteOffset() + result.getWroteBytes());
                service.putRequest(request);
                boolean flushOK = request.waitForFlush(this.defaultMessageStore.getMessageStoreConfig().getSyncFlushTimeout());
                if (!flushOK) {
                    log.error("do groupcommit, wait for flush failed, topic: " + msg.getTopic() + " tags: " + msg.getTags()
                        + " client address: " + msg.getBornHostString());
                    putMessageResult.setPutMessageStatus(PutMessageStatus.FLUSH_DISK_TIMEOUT);
                }
            } else {
                service.wakeup();
            }
        }
        // Asynchronous flush
        else {
            if (!this.defaultMessageStore.getMessageStoreConfig().isTransientStorePoolEnable()) {
                flushCommitLogService.wakeup(); //异步刷盘,使用MappedByteBuffer,默认策略 //##4
            } else {
                commitLogService.wakeup();  //异步刷盘,使用写入缓冲区+FileChannel //##3
            }
        }
        ......
    }
}
```

1 .

> lockForPutMessage()

，借助AtomicBoolean类型变量实现自旋锁，线程阻塞和重启涉及到上下文间的切换，在线程停顿时间很短的情况下，自旋锁消耗的CPU资源比阻塞要低得多，即使在自旋期间CPU会一直空转

```java
/** * Spin util acquired the lock. * 获取 putMessage 锁 */
    private void lockForPutMessage() {
        if (this.defaultMessageStore.getMessageStoreConfig().isUseReentrantLockWhenPutMessage()) {
            putMessageNormalLock.lock();
        } else {
            boolean flag;
            do {  //通过AtomicBoolean实现自旋锁,自旋直到当前线程获得锁
                flag = this.putMessageSpinLock.compareAndSet(true, false);
            }
            while (!flag);
        }
    }
```

2 . mappedFile.appendMessage(msg, this.appendMessageCallback) 在这个步骤中，会根据是否开启写入缓冲池来决定将消息写入到缓冲池writeBuffer中还是mappedByteBuffer，默认策略是写入mappedByteBuffer，借鉴其映射虚拟内存的特性实现极高速的写入性能。这个步骤仅仅是写入到流中，还没有实际同步到数据文件中。

```java
/** * 附加消息到文件。 * 实际是插入映射文件buffer * * @param msg 消息 * @param cb 逻辑 * @return 附加消息结果 */
    public AppendMessageResult appendMessage(final MessageExtBrokerInner msg, final AppendMessageCallback cb) {
        assert msg != null;
        assert cb != null;
        int currentPos = this.wrotePosition.get();
        if (currentPos < this.fileSize) {    
            //判断是否开启写入缓冲池
            ByteBuffer byteBuffer = writeBuffer != null ? writeBuffer.slice() : this.mappedByteBuffer.slice();
            byteBuffer.position(currentPos);
            AppendMessageResult result =
                cb.doAppend(this.getFileFromOffset(), byteBuffer, this.fileSize - currentPos, msg);
            this.wrotePosition.addAndGet(result.getWroteBytes());
            this.storeTimestamp = result.getStoreTimestamp();
            return result;
        }
        log.error("MappedFile.appendMessage return null, wrotePosition: " + currentPos + " fileSize: "
            + this.fileSize);
        return new AppendMessageResult(AppendMessageStatus.UNKNOWN_ERROR);
    }
```

3 . 当配置为异步刷盘且开启了写入缓冲池时，commitLogService.wakeup()，commitLogService在BrokerStartup启动时，会将其实例化为CommitRealTimeService类型的对象，这个对象间接继承自Thread，且在BrokerStartup启动时就执行其start()方法。 在commitLogService启动后一直在循环，且每休眠500ms执行一次Commit操作，但可能因为前提条件不满足而没有Commit成功。通过commitLogService.wakeup()能够立即唤醒Commit线程，让其在接受到消息的第一时间尝试Commit操作。

```java
/** * 实时 commit commitLog 线程服务 */
    class CommitRealTimeService extends FlushCommitLogService { 
        ......
        @Override
        public void run() {
            CommitLog.log.info(this.getServiceName() + " service started");
            while (!this.isStopped()) {
                int interval = CommitLog.this.defaultMessageStore.getMessageStoreConfig().getCommitIntervalCommitLog();
                int commitDataLeastPages = CommitLog.this.defaultMessageStore.getMessageStoreConfig().getCommitCommitLogLeastPages();
                int commitDataThoroughInterval = CommitLog.this.defaultMessageStore.getMessageStoreConfig().getCommitCommitLogThoroughInterval();
                long begin = System.currentTimeMillis();
                //当最近200毫秒内没有消息Commit时,此次消息触发Commit
                if (begin >= (this.lastCommitTimestamp + commitDataThoroughInterval)) {
                    this.lastCommitTimestamp = begin;
                    commitDataLeastPages = 0;
                }
                try {
                    //Commit需要缓冲区内至少含有4页数据，也就是16KB,或者是最近200毫秒内没有消息Commit
                    boolean result = CommitLog.this.mappedFileQueue.commit(commitDataLeastPages);
                    long end = System.currentTimeMillis();
                    //代表着writeBuffer里的数据commit到了fileChannel中，
                    //可能是writeBuffer里数据超过16KB或者最近200毫秒内没有消息Commit
                    if (!result) {   
                        this.lastCommitTimestamp = end;
                        //now wake up flush thread.
                        flushCommitLogService.wakeup();
                    }
                    if (end - begin > 500) {
                        log.info("Commit data to file costs {} ms", end - begin);
                    }
                    // 等待执行
                    this.waitForRunning(interval);
                } catch (Throwable e) {
                    CommitLog.log.error(this.getServiceName() + " service has exception. ", e);
                }
            }
            //在循环退出也就是CommitLog Stop停止时，强制刷盘
            boolean result = false;
            for (int i = 0; i < RETRY_TIMES_OVER && !result; i++) {
                result = CommitLog.this.mappedFileQueue.commit(0);
                CommitLog.log.info(this.getServiceName() + " service shutdown, retry " + (i + 1) + " times " + (result ? "OK" : "Not OK"));
            }
            CommitLog.log.info(this.getServiceName() + " service end");
        }
    }
```

Commit是将写入缓冲池writeBuffer中的数据转移到fileChannel中，触发Commit有两种前提：

1. 写入缓冲池内的数据页数超过了最小提交值，默认是4，也就是writeBuffer的缓冲了超过4*4KB=16KB的数据
2. 最近200ms内未接收到消息，也就是没有消息写入到writeBuffer中，写入缓冲池的最大作用就是能够将多条消息合并后写入到fileChannel中，在一定程度上提高IO性能。但如果一段时间内没有消息，那么这个时间也正好可以当做写入时间，因为此时IO压力不大。

Commit仅仅是数据从缓冲池转移到fileChannel文件通道中，此时也还没有实际的同步到数据文件。
.

4 . 当配置为异步刷盘，未开启写入缓冲池(默认策略)时，flushCommitLogService.wakeup()。flushCommitLogService在BrokerStartup启动时，将其实例化为FlushRealTimeService类型的对象，它和commitLogService一样，间接继承自Thread，BrokerStartup启动时就开启循环，每500ms尝试执行Flush工作，但Flush需要有些前提条件，wakeup()能立即唤醒此线程，使其在接收到消息的第一时间尝试Flush。

```java
/** * 实时 flush commitLog 线程服务 */
    class FlushRealTimeService extends FlushCommitLogService {
        ......
        @Override
        public void run() {
            CommitLog.log.info(this.getServiceName() + " service started");
            while (!this.isStopped()) {
                boolean flushCommitLogTimed = CommitLog.this.defaultMessageStore.getMessageStoreConfig().isFlushCommitLogTimed();
                int interval = CommitLog.this.defaultMessageStore.getMessageStoreConfig().getFlushIntervalCommitLog();
                int flushPhysicQueueLeastPages = CommitLog.this.defaultMessageStore.getMessageStoreConfig().getFlushCommitLogLeastPages();
                int flushPhysicQueueThoroughInterval = CommitLog.this.defaultMessageStore.getMessageStoreConfig().getFlushCommitLogThoroughInterval();
                // Print flush progress
                boolean printFlushProgress = false;
                long currentTimeMillis = System.currentTimeMillis();
                // 当时间满足距离上次flush时间超过10秒时，即使写入的数量不足flushPhysicQueueLeastPages(4页，16KB)，也进行flush
                if (currentTimeMillis >= (this.lastFlushTimestamp + flushPhysicQueueThoroughInterval)) {
                    this.lastFlushTimestamp = currentTimeMillis;
                    flushPhysicQueueLeastPages = 0;
                    printFlushProgress = (printTimes++ % 10) == 0;
                }
                try {
                    // 刷盘是否设置了定时,默认否
                    if (flushCommitLogTimed) {
                        Thread.sleep(interval);
                    } else {  //当没有消息触发wakeup()时,每休眠500毫秒执行一次flush; 当commit消息后触发wakeup(),若正在休眠则直接终止休眠，若不在休眠则跳过下次休眠
                        this.waitForRunning(interval);
                    }
                    if (printFlushProgress) {
                        this.printFlushProgress();
                    }
                    // flush commitLog
                    long begin = System.currentTimeMillis();
                    //刷盘至少需要4页数据，也就是16KB
                    CommitLog.this.mappedFileQueue.flush(flushPhysicQueueLeastPages);
                    long storeTimestamp = CommitLog.this.mappedFileQueue.getStoreTimestamp();
                    if (storeTimestamp > 0) {
                        CommitLog.this.defaultMessageStore.getStoreCheckpoint().setPhysicMsgTimestamp(storeTimestamp);
                    }
                    long past = System.currentTimeMillis() - begin;
                    if (past > 500) {
                        log.info("Flush data to disk costs {} ms", past);
                    }
                } catch (Throwable e) {
                    CommitLog.log.warn(this.getServiceName() + " service has exception. ", e);
                    this.printFlushProgress();
                }
            }
            // Normal shutdown, to ensure that all the flush before exit
            boolean result = false;
            for (int i = 0; i < RETRY_TIMES_OVER && !result; i++) {
                result = CommitLog.this.mappedFileQueue.flush(0);
                CommitLog.log.info(this.getServiceName() + " service shutdown, retry " + (i + 1) + " times " + (result ? "OK" : "Not OK"));
            }
            this.printFlushProgress();
            CommitLog.log.info(this.getServiceName() + " service end");
        }
    }
```

**无论是否开启写入缓冲池，刷盘最终都由FlushRealTimeService来执行，CommitRealTimeService在Commit成功后，会执行flushCommitLogService.wakeup();也就是让FlushRealTimeService尝试将内存中的数据同步至磁盘。**

是否实际将内存中的数据同步至磁盘，也就是刷盘有一些前提条件。

1. 若当前时间距离上次实际刷盘时间已经超过10S，则会忽略其他所有前提，确定刷盘，这样即使服务器宕机了最多也仅丢失10S的数据，提高了消息队列的可靠性。
2. 正常情况下刷盘需要满足持久化数据大于配置的最小页数，默认4，也就是新写入内存中的数据 >=(4*4KB=16KB)，当开启写入缓冲，也就是追加到fileChannel的数据>=16KB，未开启写入缓冲则是追加到mappedByteBuffer的数据>=16KB

总结:

![这里写图片描述](https://image.dandelioncloud.cn/images/20220530/dc710a42313a4219bab3be5b1abe7fa4.png)

1. 异步刷盘有两种策略，一种是**writeBuffer+fileChannel**，另一种是**mappedByteByffer**
2. 在最开始写入数据时，writeBuffer+fileChannel的形式是写入到缓冲池writeBuffer中，而另一种则是写入mappedByteByffer中。
3. writeBuffer+fileChannel形式相比mappedByteByffer多了一个写入缓冲池，当200ms内没有消息Commit成功或者缓冲了超过最小提交页数时，将writeBuffer内的数据Commit到fileChannel，比mappedByteByffer多了个Commit的过程。
4. 两种形式的刷盘策略相同，都是距离上次刷盘后新写入的数据量大于最小页数或者是时间超过10S。
5. 建议使用默认的mappedByteByffer，其映射虚拟内存的特性使得写入性能已经非常高了，不需要再额外开启写入缓冲。

# 分布式事务与实战运用

## 什么是分布式事务？

业务场景：用户A转账100元给用户B，这个业务比较简单，具体的步骤：
1、用户A的账户先扣除100元
2、再把用户B的账户加100元

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1684156852025/7a79c3d1742645a7952bb344bf695ff8.png)

如果在同一个数据库中进行，事务可以保证这两步操作，要么同时成功，要么同时不成功。这样就保证了转账的数据一致性。
但是在微服务架构中，因为各个服务都是独立的模块，都是远程调用，都没法在同一个事务中，都会遇到分布式事务问题。

## RocketMQ的解决方案

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1684156852025/64fe7d7a7f7340d0991ba9f0846710be.png)

RocketMQ采用两阶段提交，把扣款业务和加钱业务异步化，在A系统扣款成功后，发送“扣款成功消息”到消息中间件；B系统中加钱业务订阅“扣款成功消息”，再对用户进行加钱。

#### 具体的处理方案

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1684156852025/c76bc1cd2fc7432fbeac2f240c90d588.png)

1. 生产者发送半消息（half message）到RocketMQ服务器

2. RocketMQ服务器向生产者返回半消息的提交结果

   ![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1684156852025/92d3d9a0fcf744db8c3e45822f95d087.png)

3. 生产者执行本地的事务

   ![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1684156852025/14566cce97c8465fad1b50caec991911.png)

   ![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1684156852025/a31824993e234f23a0ab77a63e4c93fe.png)1）这里如果是标记为可提交状态（commit），消费者监听主题即可立马消费（TransactionTopic主题），消费者进行事务处理，提交。

   2）如果这里标记为回滚，那么消费者就看不到这条消息，整个事务都是回滚的

   ![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1684156852025/8a43a2905f3049fba9066c30195f0f09.png)

   3）当然本地事务中还有一种情况，那就是没执行完，这个时候，可以提交UNKNOW,交给事务回查机制。 如果是事务回查中，生产者本地事务执行成功了，则提交commit，消费者监听主题即可立马消费，消费者进行事务处理，提交。                                                                                                                            如果这里标记为回滚，那么消费者就看不到这条消息，整个事务都是回滚的。                                           
   
     当然本地事务中还有一种情况，那就是还没执行完，这个时候还是可以继续提交UNKNOW,交给事务回查机制（过段时间继续进入事务回查）。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1684156852025/5544838d4cb140de9a7c4e9bc90963e5.png)

## RocketMQ分布式事务方案中的异常处理

### 事务回查失败的处理机制

在生产者有可能是要进行定时的事务回查的，所以在这个过程中有可能生产者宕机导致这条分布式事务消息不能正常进行。那么在RocketMQ中的生产者分组就会发挥作用

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1684156852025/842ff8440dc64610a5b47d1ea1bbb0db.png)

也就是如果在进行分布式事务回查中（RocketMQ去调用生产者客户端）某一台生产者宕机了，这个时候只要还有一台分组名相同的生产者在运行，那么就可以帮助之前宕机的生产者完成事务回查。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1684156852025/df8b30b3dbc147f0932c63a7d71ff94d.png)

### 消费者失败补偿机制

虽然在消费者采用最大可能性的方案（重试的机制）确保这条消息能够执行成功，从而确保消费者事务的确保执行。但是还是有可能会发生消费者无法执行事务的情况，这个时候就必须要使用事务补偿方案。

业务场景：用户A转账100元给用户B，这个业务比较简单，具体的步骤：
1、用户A的账户先扣除100元----生产者成功执行了
2、再把用户B的账户加100元----消费者一直加100元失败。

那么就需要去通知生产者把之前扣除100元的操作进行补偿回滚操作。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1684156852025/97e3371b42344321bff70feded453610.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1684156852025/86f2b386aad449faa61beb69b84a34ef.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1684156852025/dc929a82a0ef4feb974ea2fea7ae5e0c.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1684156852025/9c2f2045fff8485b86a04acc59e376ea.png)

### 你知道半消息吗？RocketMQ是怎么实现分布式事务消息的？

半消息：是指暂时还不能被Consumer消费的消息，Producer成功发送到broker端的消息，但是此消息被标记为“暂不可投递”状态，只有等Producer端执行完本地事务后经过二次确认了之后，Consumer才能消费此条消息。

![img](https://pic2.zhimg.com/80/v2-c351af170fb71313629830434feb33c9_720w.webp)

上图就是分布式事务消息的实现过程，依赖半消息，二次确认以及消息回查机制。

1、Producer向broker发送半消息
2、Producer端收到响应，消息发送成功，此时消息是半消息，标记为“不可投递”状态，Consumer消费不了。
3、Producer端执行本地事务。
4、正常情况本地事务执行完成，Producer向Broker发送Commit/Rollback，如果是Commit，Broker端将半消息标记为正常消息，Consumer可以消费，如果是Rollback，Broker丢弃此消息。
5、异常情况，Broker端迟迟等不到二次确认。在一定时间后，会查询所有的半消息，然后到Producer端查询半消息的执行情况。
6、Producer端查询本地事务的状态
7、根据事务的状态提交commit/rollback到broker端。（5，6，7是消息回查）

## RocketMQ的实战运用

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1684156852025/d3d4f19b3b004b5f9777add0cf6cb2e6.png)

### **相关问题分析**

#### **分布式系统宕机问题**

整个系统是分布式部署，有订单系统、商品系统、会员系统。三个系统通过RPC调用完成整个下单流程。RPC调用会导致下单中各系统耦合在一起，假如会员系统宕机，会导致下单流程的不可用。

**如何异步解耦：**

利用RocketMQ，订单系统在下单后，作为生产者把“下单消息”写入MQ，商品系统与会员系统作为消费者消费MQ中的“下单消息”。这样可以达到异步解耦的目的，只要订单系统正常，对于用户来说下单业务都可以正常进行。

#### **数据完整性问题**

用户提交订单后，扣减库存成功、扣减优惠券成功，但是在确认订单操作失败（比如：支付失败），那么就需要对库存、优惠券进行回退。

如何保证数据的完整性？前面讲到的利用分布式事物消息确保要么都成功，要么都失败。





### **消息的可用性**

**RocketMQ如何能保证消息的可用性/可靠性？（这个问题的另一种问法：如何保证消息不丢失）**

消息可能在哪些阶段丢失呢？可能会在这三个阶段发生丢失：生产阶段、存储阶段、消费阶段。

所以要从这三个阶段考虑：从Producer，Consumer和Broker三个方面来回答。



![image-20230918202031047](https://img-blog.csdnimg.cn/img_convert/01d53bdae7e8d1b4123e2047411da308.png)

#### **从Producer角度主要**通过请求确认机制，来保证消息的可靠传递**。**

1、可以**采用同步发送**，即发送一条数据等到接受者返回响应之后再发送下一个数据包。如果返回响应OK，表示消息成功发送到了broker，状态超时或者失败都会触发二次重试。
2、异步发送的时候，应该在回调方法里检查，如果发送失败或者异常，都应该进行重试。。
3、如果一条消息发送之后超时，也可以通过查询日志的API，来检查是否在Broker存储成功。

​		总的来说，Producer还是采用同步发送来保证的。

### **从Broker角度通过**配置可靠性优先的 Broker 参数来避免因为宕机丢消息

1、消息只要持久化到CommitLog（日志文件）中，即使Broker宕机，未消费的消息也能重新恢复再消费。

2、Broker的刷盘机制：**同步刷盘**和**异步刷盘**，不管哪种刷盘都可以保证消息一定存储在pagecache中（内存中），但是同步刷盘更可靠，它是Producer发送消息后等数据持久化到磁盘之后再返回响应给Producer。

3、Broker支持多**Master多Slave同步双写**和**多Master多Slave异步复制**模式，消息都是发送给Master主机，但是消费既可以从Master消费，也可以从Slave消费。同步双写模式可以保证即使Master宕机，消息肯定在Slave中有备份，保证了消息不会丢失。

![image-20230918202927817](https://img-blog.csdnimg.cn/img_convert/f905f109d04a6c613650e85b47cfe537.png)

### **从Consumer角度分析，如何保证消息被成功消费？**

Consumer自身维护了个持久化的offset（对应Message Queue里的min offset），用来标记已经成功消费且已经成功发回Broker的消息下标。

如果Consumer消费失败，它会向Broker发回消费失败的状态，发回成功才会更新自己的offset。

如果发回给broker时broker挂掉了，Consumer会定时重试，

如果Consumer和Broker一起挂掉了，消息还在Broker端存储着，Consumer端的offset也是持久化的，重启之后继续拉取offset之前的消息进行消费。

### 怎么处理消息积压？

发生了消息积压，这时候就得想办法赶紧把积压的消息消费完，就得考虑提高消费能力，一般有两种办法：

![image-20230918203127950](https://img-blog.csdnimg.cn/img_convert/7c992329c5e6c84c3795de0242a82366.png)

消费者扩容：如果当前Topic的Message Queue的数量大于消费者数量，就可以对消费者进行扩容，增加消费者，来提高消费能力，尽快把积压的消息消费玩。
消息迁移Queue扩容：如果当前Topic的Message Queue的数量小于或者等于消费者数量，这种情况，再扩容消费者就没什么用，就得考虑扩容Message Queue。可以新建一个临时的Topic，临时的Topic多设置一些Message Queue，然后先用一些消费者把消费的数据丢到临时的Topic，因为不用业务处理，只是转发一下消息，还是很快的。接下来用扩容的消费者去消费新的Topic里的数据，消费完了之后，恢复原状。

![image-20230918203319817](https://img-blog.csdnimg.cn/img_convert/a2acd49913bd609ca760f4f5f04f2bf7.png)

### **顺序消息,如何保证顺序消息？**

顺序消息是指消息的消费顺序和产生顺序相同，在有些业务逻辑下，必须保证顺序，比如订单的生成、付款、发货，这个消息必须按顺序处理才行。

顺序消息分为全局顺序消息和部分顺序消息：

顺序由producer发送到broker的消息队列是满足FIFO的，所以发送是顺序的，单个queue里的消息是顺序的。多个Queue同时消费是无法绝对保证消息的有序性的。所以，**同一个topic，同一个queue，发消息的时候一个线程发送消息，消费的时候一个线程去消费一个queue里的消息。**

RocketMQ给我们提供了**MessageQueueSelector**接口，可以重写里面的接口，实现自己的算法，比如判断i%2==0，那就发送消息到queue1否则发送到queue2。

全局顺序消息指某个 Topic 下的所有消息都要保证顺序；

部分顺序消息只要保证每一组消息被顺序消费即可，比如订单消息，只要保证同一个订单 ID 个消息能按顺序消费即可。

部分顺序消息
部分顺序消息相对比较好实现，生产端需要做到把同 ID 的消息发送到同一个 Message Queue ；在消费过程中，要做到从同一个Message Queue读取的消息顺序处理——消费端不能并发处理顺序消息，这样才能达到部分有序。 

![image-20230918203535409](https://img-blog.csdnimg.cn/img_convert/0d8760b7caf3af1d80edb4bb88b4bbd2.png)

发送端使用 MessageQueueSelector 类来控制 把消息发往哪个 Message Queue 。

消费端通过使用 MessageListenerOrderly 来解决单 Message Queue 的消息被并发处理的问题。

#### 全局顺序消息

RocketMQ 默认情况下不保证顺序，比如创建一个 Topic ，默认八个写队列，八个读队列，这时候一条消息可能被写入任意一个队列里；在数据的读取过程中，可能有多个 Consumer ，每个 Consumer 也可能启动多个线程并行处理，所以消息被哪个 Consumer 消费，被消费的顺序和写人的顺序是否一致是不确定的。

要保证全局顺序消息， 需要先把 Topic 的读写队列数设置为 一，然后Producer Consumer 的并发设置，也要是一。简单来说，为了保证整个 Topic全局消息有序，只能消除所有的并发处理，各部分都设置成单线程处理 ，这时候就完全牺牲RocketMQ的高并发、高吞吐的特性了。
 ![image-20230918203719183](https://img-blog.csdnimg.cn/img_convert/dafc079a5ade67d8468fa2c7bf596750.png)



### **消息过滤,**如何实现消息过滤？

有两种方案：

​	一种是在 Broker 端按照 Consumer 的去重逻辑进行过滤，这样做的好处是避免了无用的消息传输到 Consumer 端，缺点是加重了 Broker 的负担，实现起来相对复杂。
​	另一种是在 Consumer 端过滤，比如按照消息设置的 tag 去重，这样的好处是实现起来简单，缺点是有大量无用的消息到达了 Consumer 端只能丢弃不处理。
一般采用Cosumer端过滤，如果希望提高吞吐量，可以采用Broker过滤。

对消息的过滤有三种方式： 

![image-20230918203833864](https://img-blog.csdnimg.cn/img_convert/fec0ba5a7f5b42e38a4adcd5b6e838fb.png)

### **消息去重**

**如果由于网络等原因，多条重复消息投递到了Consumer端，你怎么进行消息去重？**

业务端自己保证，主要的方式有两种：**业务幂等**和**消息去重**

这个得先说下消息的**幂等性原则**：就是用户对于同一种操作发起的多次请求的结果是一样的，不会因为操作了多次就产生不一样的结果。只要保持幂等性，不管来多少条消息，最后处理结果都一样，需要Consumer端自行实现。

去重的方案：因为每个消息都有一个**MessageId**, 保证每个消息都有一个唯一键，可以是**数据库的主键或者唯一约束**，也可以是**Redis缓存中的键**，当消费一条消息前，先检查数据库或缓存中是否存在这个唯一键，如果存在就不再处理这条消息，如果消费成功，要保证这个**唯一键插入到去重表中**。

根据Tag过滤：这是最常见的一种，用起来高效简单

DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("CID_EXAMPLE");
consumer.subscribe("TOPIC", "TAGA || TAGB || TAGC");

SQL 表达式过滤：SQL表达式过滤更加灵活



```java
DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("please_rename_unique_group_name_4");
// 只有订阅的消息有这个属性a, a >=0 and a <= 3
consumer.subscribe("TopicTest", MessageSelector.bySql("a between 0 and 3");
consumer.registerMessageListener(new MessageListenerConcurrently() {
   @Override
   public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {
       return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
   }
});
consumer.start();
```

Filter Server 方式：最灵活，也是最复杂的一种方式，允许用户自定义函数进行过滤


### RocketMQ如何实现负载均衡？

RocketMQ是分布式消息服务，负载均衡时再生产和消费的客户端完成的。

![img](https://pic1.zhimg.com/80/v2-3023ecfd062efce6e73d8a663d868e60_720w.webp)


具体来说的话，主要可以分为Producer端发送消息时候的负载均衡和Consumer端订阅消息的负载均衡。

下面我从生产者负载均衡和消费者负载均衡两个角度来说明：

### **生产者的负载均衡**

Producer端在发送消息的时候，会先根据Topic找到指定的TopicPublishInfo，在获取了TopicPublishInfo路由信息后，RocketMQ的客户端在默认方式下selectOneMessageQueue()方法会从TopicPublishInfo中的messageQueueList中选择一个队列（MessageQueue）进行发送消息。具这里有一个sendLatencyFaultEnable开关变量，如果开启，在随机递增取模的基础上，再过滤掉not available的Broker代理。
 ![image-20230918215220144](https://img-blog.csdnimg.cn/img_convert/ca310b4cf57da60d4e7be140b567f1a2.png)

源码中可见采用索引递增 再利用索引值取余的方法

所谓的"latencyFaultTolerance"，是指对之前失败的，按一定的时间做退避。例如，如果上次请求的latency超过550Lms，就退避3000Lms；超过1000L，就退避60000L；如果关闭，采用随机递增取模的方式选择一个队列（MessageQueue）来发送消息，latencyFaultTolerance机制是实现消息发送高可用的核心关键所在。 

### **消费者的负载均衡**

在RocketMQ中，Consumer端的两种消费模式（Push/Pull）都是基于拉模式来获取消息的，而在Push模式只是对pull模式的一种封装，其本质实现为消息拉取线程在从服务器拉取到一批消息后，然后提交到消息消费线程池后，又“马不停蹄”的继续向服务器再次尝试拉取消息。如果未拉取到消息，则延迟一下又继续拉取。在两种基于拉模式的消费方式（Push/Pull）中，均需要Consumer端知道从Broker端的哪一个消息队列中去获取消息。因此，有必要在Consumer端来做负载均衡，即Broker端中多个MessageQueue分配给同一个ConsumerGroup中的哪些Consumer消费。
 多种负载均衡算法解析：

1、平均分配算法

![img](https://pic1.zhimg.com/80/v2-b77b33253518b31d243b0edd74d182fc_720w.webp)

2、环形算法

![img](https://pic2.zhimg.com/80/v2-231664ff79f3acd2c25eb96a057717e1_720w.webp)

3、指定机房算法
4、就近机房算法
5、统一哈希算法

使用一致性哈希算法进行负载，每次负载都会重建一致性hash路由表，获取本地客户端负责的所有队列信息。默认的hash算法为MD5，假设有4个消费者客户端和2个消息队列mq1和mq2，通过hash后分布在hash环的不同位置，按照一致性hash的顺时针查找原则，mq1被client2消费，mq2被client3消费。

![img](https://pic2.zhimg.com/80/v2-872613ad74796d828135cfcdb87931b9_720w.webp)



6、手动配置算法

Consumer端实现负载均衡的核心接口RebalanceImpl
在Consumer实例的启动流程中的启动MQClientInstance实例部分，会完成负载均衡服务线程—RebalanceService的启动（每隔20s执行一次）。

通过查看源码可以发现，RebalanceService线程的run()方法最终调用的是RebalanceImpl类的rebalanceByTopic()方法，这个方法是实现Consumer端负载均衡的核心。

rebalanceByTopic()方法会根据消费者通信类型为“广播模式”还是“集群模式”做不同的逻辑处理。这里主要来看下集群模式下的主要处理流程：

(1) 从rebalanceImpl实例的本地缓存变量—topicSubscribeInfoTable中，获取该Topic主题下的消息消费队列集合（mqSet）；

(2) 根据topic和consumerGroup为参数调用mQClientFactory.findConsumerIdList()方法向Broker端发送通信请求，获取该消费组下消费者Id列表；

(3) 先对Topic下的消息消费队列、消费者Id排序，然后用消息队列分配策略算法（默认为：消息队列的平均分配算法），计算出待拉取的消息队列。这里的平均分配算法，类似于分页的算法，将所有MessageQueue排好序类似于记录，将所有消费端Consumer排好序类似页数，并求出每一页需要包含的平均size和每个页面记录的范围range，最后遍历整个range而计算出当前Consumer端应该分配到的的MessageQueue。

(4) 然后，调用updateProcessQueueTableInRebalance()方法，具体的做法是，先将分配到的消息队列集合（mqSet）与processQueueTable做一个过滤比对。

![image-20230918215322343](https://img-blog.csdnimg.cn/img_convert/f8184f5d88176a3f18e6d169e990bf93.png)

上图中processQueueTable标注的红色部分，表示与分配到的消息队列集合mqSet互不包含。将这些队列设置Dropped属性为true，然后查看这些队列是否可以移除出processQueueTable缓存变量，这里具体执行removeUnnecessaryMessageQueue()方法，即每隔1s 查看是否可以获取当前消费处理队列的锁，拿到的话返回true。如果等待1s后，仍然拿不到当前消费处理队列的锁则返回false。如果返回true，则从processQueueTable缓存变量中移除对应的Entry；

上图中processQueueTable的绿色部分，表示与分配到的消息队列集合mqSet的交集。判断该ProcessQueue是否已经过期了，在Pull模式的不用管，如果是Push模式的，设置Dropped属性为true，并且调用removeUnnecessaryMessageQueue()方法，像上面一样尝试移除Entry；

最后，为过滤后的消息队列集合（mqSet）中的每个MessageQueue创建一个ProcessQueue对象并存入RebalanceImpl的processQueueTable队列中（其中调用RebalanceImpl实例的computePullFromWhere(MessageQueue mq)方法获取该MessageQueue对象的下一个进度消费值offset，随后填充至接下来要创建的pullRequest对象属性中），并创建拉取请求对象—pullRequest添加到拉取列表—pullRequestList中，最后执行dispatchPullRequest()方法，将Pull消息的请求对象PullRequest依次放入PullMessageService服务线程的阻塞队列pullRequestQueue中，待该服务线程取出后向Broker端发起Pull消息的请求。其中，可以重点对比下，RebalancePushImpl和RebalancePullImpl两个实现类的dispatchPullRequest()方法不同，RebalancePullImpl类里面的该方法为空。
消息消费队列在同一消费组不同消费者之间的负载均衡，其核心设计理念是在一个消息消费队列在同一时间只允许被同一消费组内的一个消费者消费，一个消息消费者能同时消费多个消息队列



### **RocketMq的特点**

1. **亿级消息的堆积能力**，单个队列中的百万级消息的累积容量。
2. **高可用性**：Broker服务器支持多Master多Slave的同步双写以及Master多Slave的异步复制模式，其中同步双写可保证消息不丢失。
3. **高可靠性**：生产者将消息发送到Broker端有三种方式，同步、异步和单向，其中同步和异步都可以保证消息成功的成功发送。Broker在对于消息刷盘有两种策略：同步刷盘和异步刷盘，其中同步刷盘可以保证消息成功的存储到磁盘中。消费者的消费模式也有集群消费和广播消费两种，默认集群消费，如果集群模式中消费者挂了，一个组里的其他消费者会接替其消费。综上所述，是高可靠的。
4. 支持**分布式事务消息**：这里是采用半消息确认和消息回查机制来保证分布式事务消息的，下面会详细描述。
5. 支持**消息过滤**：建议采用消费者业务端的tag过滤
6. 支持**顺序消息**：消息在Broker中是采用队列的FIFO模式存储的，也就是发送是顺序的，只要保证消费的顺序性即可
7. 支持**定时消息和延迟消息**：Broker中由定时消息的机制，消息发送到Broker中，不会立即被Consumer消费，会等到一定的时间才被消费。延迟消息也是一样，延迟一定时间之后才会被Consumer消费。

# RabbitMQ

### 一、RabbitMQ介绍

#### 1.1 现存问题

- 服务调用：两个服务调用时，我们可以通过传统的HTTP方式，让服务A直接去调用服务B的接口，但是这种方式是同步的方式，虽然可以采用SpringBoot提供的@Async注解实现异步调用，但是这种方式无法确保请求一定回访问到服务B的接口。[那如何保证服务A的请求信息一定能送达到服务B去完成一些业务操作呢？]()|                                                           如何实现异步调用![1642517531404.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/2746/1642502600000/c7cf971cd2234baba817bc1c68a5fab8.png)
- 海量请求：在我们在做一些秒杀业务时，可能会在某个时间点突然出现大量的并发请求，这可能已经远远超过服务器的并发瓶颈，这时我们需要做一些削峰的操作，也就是将大量的请求缓冲到一个队列中，然后慢慢的消费掉。[如何提供一个可以存储千万级别请求的队列呢？]()![1642517747632.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/2746/1642502600000/b7f3de45852141ec805a61676eda89f5.png)
- 在微服务架构下，可能一个业务会出现同时调用多个其他服务的场景，而且这些服务之间一般会用到Feign的方式进行轻量级的通讯，如果存在一个业务，用户创建订单成功后，还需要去给用户添加积分、通知商家、通知物流系统、扣减商品库存，而在执行这个操作时，如果任意一个服务出现了问题，都会导致整体的下单业务失败，并且会导致给用户反馈的时间延长。这时就造成了服务之间存在一个较高的耦合性的问题。[如何可以降低服务之间的耦合性呢？]()![1642517948196.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/2746/1642502600000/dd1af5ae8d024515a22da1ce05064b62.png)

#### 1.2 处理问题

RabbitMQ就可以解决上述的全部问题

- 服务之间如何想实现可靠的异步调用，可以通过RabbitMQ的方式实现，服务A只需要保证可以把消息发送到RabbitMQ的队列中，服务B就一定会消费到队列中的消息只不过会存在一定的延时。|                                                               异步访问![1642518013295.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/2746/1642502600000/6444fc077b294d37a013cd26f7c9f1de.png)
- 忽然的海量请求可以存储在RabbitMQ的队列中，然后由消费者慢慢消费掉，RabbitMQ的队列本身就可以存储上千万条消息                                                                                  ![1642518109219.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/2746/1642502600000/265d7b4b242643b4a8fb381f3df1cce3.png)
- 在调用其他服务时，如果允许延迟效果的出现，可以将消息发送到RabbitMQ中，再由消费者慢慢消费|                                                               服务解耦
  ![1642518233825.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/2746/1642502600000/b596d2d4b3ca43d5bba4f13c9d9c31ea.png)

#### 1.3 RabbitMQ介绍

百度百科：

&#x3e; **RabbitMQ**是实现了高级消息队列协议（AMQP）的开源消息代理软件（亦称面向消息的中间件）。RabbitMQ服务器是用[Erlang](https://baike.baidu.com/item/Erlang)语言编写的，而集群和故障转移是构建在[开放电信平台](https://baike.baidu.com/item/开放电信平台)框架上的。所有主要的[编程语言](https://baike.baidu.com/item/编程语言/9845131)均有与代理接口通讯的[客户端](https://baike.baidu.com/item/客户端/101081)库。

首先RabbitMQ基于AMQP协议开发，所以很多基于AMQP协议的功能RabbitMQ都是支持的，比如SpringCloud中的消息总线bus

其次RabbitMQ是基于Erlang编写，这是也是RabbitMQ天生的优势，Erlang被称为面向并发编程的语言，并发能力极强，在众多的MQ中，RabbitMQ的延迟特别低，在微秒级别，所以一般的业务处理RabbitMQ比Kafka和RocketMQ更有优势。

最后RabbitMQ提供自带了图形化界面，操作方便，还自带了多种集群模式，可以保证RabbitMQ的高可用，并且SpringBoot默认就整合RabbitMQ，使用简单方便。

### AMQP协议

RabbitMQ是Advanced Message Queuing Protocol （AMQP，**高级消息队列协议**）开放标准的实现，由美国微处理器厂商Rabbit公司开源。RabbitMQ服务器是用Erlang语言编写的。

AMQP（高级消息队列协议）是一个网络协议。它支持符合要求的客户端应用（application）和消息中间件代理（messaging middleware broker）之间进行通信。主要特征是面向消息、队列、路由（包括点对点和发布/订阅）、可靠性、安全。

说简单点就是在异步通讯中，消息不会立刻到达接收方，而是被存放到一个容器中，当满足一定的条件之后，消息会被容器发送给接收方，这个容器即消息队列（MQ），而完成这个功能需要双方和容器以及其中的各个组件遵守统一的约定和规则，AMQP就是这样的一种协议，消息发送与接受的双方遵守这个协议可以实现异步通讯。这个协议约同时规定了消息的格式和工作方式。

消息代理（message brokers）从发布者（publishers）亦称生产者（producers）那儿接收消息，并根据既定的路由规则把接收到的消息发送给处理消息的消费者（consumers）。

- 高级消息交换协议模型：一套确定的消息交换功能，必须有三个部分：
  - 交换器(exchange)：生产者把消息发到交换器上
  - 队列(queue)：消息到达队列，等待消费者接收
  - 绑定(binding)：决定了消息如何从交换器到指定的队列

由于AMQP是一个网络协议，所以这个过程中的**发布者**，**消费者**，**消息代理** 可以存在于不同的设备上。

消息（message）被发布者（publisher）发送给交换机（exchange），交换机常常被比喻成邮局或者邮箱。然后交换机将收到的消息根据路由规则分发给绑定的队列（queue）。最后AMQP代理会将消息投递给订阅了此队列的消费者，或者消费者按照需求自行获取。

AMQP的机制如下图所示

![2020060921065549](E:\图灵课堂\MQ专题\MQ专题.assets\2020060921065549.png)

这是一个简单的“Hello，world”示例，从发布者到生成者消息的大致流向，其中还省略了一些AMQP实际的组件细节。这里主要包括以下一些组件：
**Publisher**，数据的发送方。
**Exchange**，消息交换机，它指定消息按什么规则，路由到哪个队列，这里的规则后面会有介绍。
**Queue**，消息队列载体，每个消息都会被投入到一个或多个队列。
**Consumer**，数据的接收方。
发布者（Publisher）发布消息时可以给消息指定各种消息属性（message meta-data）。有些属性有可能会被消息代理（Brokers）使用，然而其他的属性则是完全不透明的，它们只能被接收消息的应用所使用。

从安全角度考虑，网络是不可靠的，接收消息的应用也有可能在处理消息的时候失败。基于此原因，AMQP模块包含了一个消息确认（message acknowledgements）的概念：当一个消息从队列中投递给消费者后（Consumer），消费者会通知一下消息代理（Broker），这个可以是自动的，也可以由处理消息的应用的开发者执行。当“消息确认”被启用的时候，消息代理不会完全将消息从队列中删除，直到它收到来自消费者的确认回执（acknowledgement）。

在某些情况下，例如当一个消息无法被成功路由时，消息或许会被返回给发布者并被丢弃。或者，如果消息代理执行了延期操作，消息会被放入一个所谓的死信队列中。此时，消息发布者可以选择某些参数来处理这些特殊情况。

队列，交换机和绑定统称为AMQP实体（AMQP entities）。

以上这些是在上图中显示出来的一些AMQP组件元件，除了这些外，还有一些额外的概念，主要为：
Binding，绑定，消息队列与交换器直接关联的，它的作用就是把Exchange和Queue按照路由规则绑定起来。

**Routing Key**，路由关键字，Exchange根据这个关键字进行消息投递。

**Channel**，信道，多路复用连接中的一条独立的双向数据流通道，为会话提供物理传输介质。Channel是在connection内部建立的逻辑连接，如果应用程序支持多线程，通常每个thread创建单独的channel进行通讯，AMQP method包含了channel id帮助客户端和message broker识别channel，所以channel之间是完全隔离的。Channel作为轻量级的Connection极大减少了操作系统建立TCP connection的开销。在客户端的每个连接里，可建立多个Channel，每个Channel代表一个会话任务。

**Broker** ，AMQP的服务端称为Broker。其实Broker就是接收和分发消息的应用，也就是说RabbitMQ Server就是Message Broker。

**Virtual Host**，虚拟主机，一批交换器（Exchange），消息队列（Queue）和相关对象。虚拟主机是共享相同身份认证和加密环境的独立服务器域。同时一个Broker里可以开设多个vhost，用作不同用户的权限分离。

**Connection** ，连接，一个网络连接，比如TCP/IP套接字连接。Channel是建立在Connection之上的，一个Connection可以建立多个Channel。

**Message**，消息体，是AMQP所操纵的基本单位，它由Producer产生，经过Broker被Consumer所消费。它的基本结构有两部分: Header和Body。Header是由Producer添加上的各种属性的集合，这些属性有控制Message是否可被缓存，接收的Queue是哪个，优先级是多少等。Body是真正需要传送的数据，它是对Broker不可见的二进制数据流，在传输过程中不应该受到影响。

上面的简单实例图只是简单说明了一些消息传送的基本流程，可能其中的频道、虚拟主机、连接、信道等都没有体现出来。到后面RabbitMQ的知识点都会慢慢介绍到。

AMQP的结构图如下所示：

![20200609210907441](E:\图灵课堂\MQ专题\MQ专题.assets\20200609210907441.png)





### 二、RabbitMQ安装

---

#### 2.1 安装RabbitMQ

这里推荐搭建采用Docker的方式在Linux中安装RabbitMQ，如果对Docker不了解，推荐去学习一下Docker的应用，不然学习其他的知识时，安装的成本都特别高，这里我们就采用Docker的方式安装RabbitMQ。

直接使用docker-compose.yml文件即可安装RabbitMQ服务

```yml
version: '3.1'
services:
  rabbitmq:
    restart: always
    image: daocloud.io/library/rabbitmq:3.8.8
    restart: always
    volumes:
      - ./data/:/var/lib/rabbitmq/
      - ./log/:/var/log/rabbitmq/log/
    ports:
      - 15672:15672
      - 5672:5672
```

执行 `docker-compose up -d`运行

测试效果：`curl localhost:5672`

|                           查看效果                           |
| :----------------------------------------------------------: |
| ![image20220121005749217.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/2746/1642502600000/70db9f72a7454af58b0e20179ced1835.png) |

#### 2.2 开启图形化界面

默认情况下，当前镜像的图形化界面默认没有开启，需要进入到容器内部开启图形化管理界面

|                      启动图形化界面插件                      |
| :----------------------------------------------------------: |
| ![image20220121005619975.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/2746/1642502600000/5ac8efade76648a288ccb7e463d93230.png) |
| ![image20220121005624253.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/2746/1642502600000/539962d77c0b4ce0baa0cf48b44bc59b.png) |

通过浏览器访问15672，查看图形化界面

|                         查看登录页面                         |
| :----------------------------------------------------------: |
| ![image20220121005852818.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/2746/1642502600000/42aa9718eda944008abf91328b210851.png) |

默认用户和密码均为：guest，查看首页

|                           查看首页                           |
| :----------------------------------------------------------: |
| ![image20220121005930123.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/2746/1642502600000/5224bc537e3c4cae916e356289e12bd2.png) |

### 三、RabbitMQ构架

RabbitMQ的架构可以查看官方地址：https://rabbitmq.com/tutorials/amqp-concepts.html

|                         官方简单架构                         |
| :----------------------------------------------------------: |
| ![image20220121010054992.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/2746/1642502600000/6ec4892681bd42e1b28f7b27b7b94ee1.png) |

可以看出RabbitMQ中主要分为三个角色：

- Publisher：消息的发布者，将消息发布到RabbitMQ中的Exchange
- RabbitMQ服务：Exchange接收Publisher的消息，并且根据Routes策略将消息转发到Queue中
- Consumer：消息的消费者，监听Queue中的消息并进行消费

官方提供的架构图相对简洁，我们可以自己画一份相对完整一些的架构图：

|                        RabbitMQ架构图                        |
| :----------------------------------------------------------: |
| ![image20220121011000157.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/2746/1642502600000/0479ebd0dd8e4445ba91a99b54d2d37e.png) |

可以看出Publisher和Consumer都是单独和RabbitMQ服务中某一个Virtual Host建立Connection的客户端

后续通过Connection可以构建Channel通道，用来发布、接收消息

一个Virtual Host中可以有多个Exchange和Queue，Exchange可以同时绑定多个Queue

在基于架构图查看图形化界面，会更加清晰

|                        图形化界面信息                        |
| :----------------------------------------------------------: |
| ![image20220121011418076.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/2746/1642502600000/375a8000de2649989ebae2f1e3f0ddd1.png) |

### 交换机的四种类型

#### 一、Direct类型交换机

Direct类型交换机
1.在发送消息的时候，通过Direct类型的路由转发；
要求Direct类型交换机和队列绑定；绑定需要一个标识，生产者在发送消息的时候，也需要指定一个标识，消息发送给交换机以后，交换机进行标识的匹配；知道和交换机绑定队列完全吻合的标识，只要是匹配到了，就把消息通过交换机转发给当前绑定标识吻合的这个队列中去；

Direct类型的交换机可以做点啥？

//如果说有一个生产者发送了很多消息，需要把消息分类处理；
//消息需要分几类，就可以定义几个队列；分别把队列和交换机在绑定的是偶，分别给出不同的表示，发送消息的时候，就给出不同标识，就可以把消息发送到不同的经过分类的队列中去了；

//记录日志：记录日志，分类的记录，如果是异常，就需要另外的处理；
//还需要来一个所有日志的记录；
//定义一个记录所有日志的队列，定义一个专门为异常日志存在的队列；
//定义一个Dirct类型的交换机，分别绑定不同的标识，日志生产出来以后，就可以根据日志的类型不同，发给路由，把类型作为标识，路由匹配后，就可以转发到不同的队列中去中，就可以把日志分类处理；

![image-20240125173801933](E:\图灵课堂\MQ专题\MQ专题.assets\image-20240125173801933.png)

生产者




```java
  public class DirectExchangeProducer{
    public static void Send()
    {
        var factory = new ConnectionFactory();
        factory.HostName = "localhost";//RabbitMQ服务在本地运行
        factory.UserName = "guest";//用户名
        factory.Password = "guest";//密码 
        using (var connection = factory.CreateConnection())
        {
            using (IModel channel = connection.CreateModel())
            {
                #region 删除队列和交换机
                channel.ExchangeDelete("DirectExChange");
                channel.QueueDelete("DirectExchangeLogAllQueue");
                channel.QueueDelete("DirectExchangeErrorQueue");
                #endregion        
 
 channel.QueueDeclare(queue: "DirectExchangeLogAllQueue", durable: true, exclusive: false, autoDelete: false, arguments: null); 
            channel.QueueDeclare(queue: "DirectExchangeErrorQueue", durable: true, exclusive: false, autoDelete: false, arguments: null); 
            //交换机的类型：type：ExchangeType
            channel.ExchangeDeclare(exchange: "DirectExChange", type: ExchangeType.Direct, durable: true, autoDelete: false, arguments: null);

            //定义四种类型的日志
            string[] logtypes = new string[] { "debug", "info", "warn", "error" };

            //把交换机和队列绑定，把所有的日志类型作为标识绑定起来；
            //DirectExchangeLogAllQueue：用来接受所有的日志消息

            //交换机和队列可以绑定多个标识
            foreach (string logtype in logtypes)
            {
                channel.QueueBind(queue: "DirectExchangeLogAllQueue",
                        exchange: "DirectExChange",
                        routingKey: logtype);
            }

            //针对异常处理的：这里DirectExChange 绑定DirectExchangeErrorQueue，只指定一个标识就是error
            channel.QueueBind(queue: "DirectExchangeErrorQueue",
                      exchange: "DirectExChange",
                      routingKey: "error");
         
         //通过取模，得到四种类型的日志各自25个日志信息；
            List<LogMsgModel> logList = new List<LogMsgModel>();
            for (int i = 1; i <= 100; i++)
            {
                if (i % 4 == 0)
                {
                    logList.Add(new LogMsgModel() { LogType = "info", Msg = Encoding.UTF8.GetBytes($"info第{i}条信息") });
                }
                if (i % 4 == 1)
                {
                    logList.Add(new LogMsgModel() { LogType = "debug", Msg = Encoding.UTF8.GetBytes($"debug第{i}条信息") });
                }
                if (i % 4 == 2)
                {
                    logList.Add(new LogMsgModel() { LogType = "warn", Msg = Encoding.UTF8.GetBytes($"warn第{i}条信息") });
                }
                if (i % 4 == 3)
                {
                    logList.Add(new LogMsgModel() { LogType = "error", Msg = Encoding.UTF8.GetBytes($"error第{i}条信息") });
                }
            }

            Console.WriteLine("生产者发送100条日志信息");

            logList = logList.OrderBy(l => l.LogType).ToList();

            //发送日志信息
            foreach (var log in logList)
            {
                channel.BasicPublish(exchange: "DirectExChange",
                                    routingKey: log.LogType,
                                    basicProperties: null,
                                    body: log.Msg);
                Console.WriteLine($"{Encoding.UTF8.GetString(log.Msg)}  已发送~~");
            }

            }
        }
    }
    public class LogMsgModel
    {
        public string LogType { get; set; }

        public byte[] Msg { get; set; }
    }
}
```


消费者     

```java
    public class DirectExchangeConsumerLogAll{
    public static void Consumption()
    {
        var factory = new ConnectionFactory();
        factory.HostName = "localhost";//RabbitMQ服务在本地运行
        factory.UserName = "guest";//用户名
        factory.Password = "guest";//密码 
        using (var connection = factory.CreateConnection())
        {
            using (IModel channel = connection.CreateModel())
            {
                channel.QueueDeclare(queue: "DirectExchangeLogAllQueue", durable: true, exclusive: false, autoDelete: false, arguments: null);
                channel.ExchangeDeclare(exchange: "DirectExChange", type: ExchangeType.Direct, durable: true, autoDelete: false, arguments: null);  
		string[] logtypes = new string[] { "debug", "info", "warn", "error" };
            foreach (string logtype in logtypes)
            {
                channel.QueueBind(queue: "DirectExchangeLogAllQueue",
                        exchange: "DirectExChange",
                        routingKey: logtype);
            }

            //消费队列中的所有消息；                                   
            var consumer = new EventingBasicConsumer(channel);
            consumer.Received += (model, ea) =>
            {
                var body = ea.Body;
                var message = Encoding.UTF8.GetString(body.ToArray());
                Console.WriteLine($"【{message}】，写入文本~~");
            };
            //处理消息
            channel.BasicConsume(queue: "DirectExchangeLogAllQueue",
                                 autoAck: true,
                                 consumer: consumer);
            Console.ReadLine();
        }
    }
}
}
```


#### 二、Fanout 类型交换机

fanout类型的Exchange路由规则非常简单，它会把所有发送到该Exchange的消息路由到所有与它绑定的Queue中。
交换机和队列绑定不需要指定标识；对于生产者发过来的消息，发给交换机以后，只要是整个交换机和队列有绑定，交换机就转发给队列；

生产者发送的消息都可以转发给和他绑定额队列；
广播式；发布订阅模式；一个生产者把消息发送过去，多个消费者都可以接受到了；

![image-20240125173750173](E:\图灵课堂\MQ专题\MQ专题.assets\image-20240125173750173.png)

生产者


```java
    public class FanoutExchange
{
    public static void Send()
    {
        var factory = new ConnectionFactory();
        factory.HostName = "localhost";//RabbitMQ服务在本地运行
        factory.UserName = "guest";//用户名
        factory.Password = "guest";//密码 
        using (var connection = factory.CreateConnection())
        {
            using (IModel channel = connection.CreateModel())
            {
                channel.QueueDeclare(queue: "FanoutExchangeZhaoxi001", durable: true, exclusive: false, autoDelete: false, arguments: null);
                channel.QueueDeclare(queue: "FanoutExchangeZhaoxi002", durable: true, exclusive: false, autoDelete: false, arguments: null);
                //在这里声明一个Fanout 类型的交换机
                channel.ExchangeDeclare(exchange: "FanoutExchange", type: ExchangeType.Fanout, durable: true, autoDelete: false, arguments: null);
                //交换机绑定队列，不需要标识
                channel.QueueBind(queue: "FanoutExchangeZhaoxi001", exchange: "FanoutExchange", routingKey: string.Empty, arguments: null);
                channel.QueueBind(queue: "FanoutExchangeZhaoxi002", exchange: "FanoutExchange", routingKey: string.Empty, arguments: null);
                    
//在控制台输入消息，按enter键发送消息
            int i = 1;
            while (true)
            {
                var message = $"通知{i}";
                if (i>10)
                {
                    Console.WriteLine("请输入通知~~");
                   message = Console.ReadLine();
                } 
                var body = Encoding.UTF8.GetBytes(message);
                //基本发布
                channel.BasicPublish(exchange: "FanoutExchange",
                                     routingKey: string.Empty,
                                     basicProperties: null,
                                     body: body);
                Console.WriteLine($"通知【{message}】已发送到队列");
                Thread.Sleep(2000);
                i++;
            }
        }
    }
}
}
```



消费者

 


```java
      public class FanoutExchange{
            public static void Consumption()
            {
                var factory = new ConnectionFactory();
                factory.HostName = "localhost";//RabbitMQ服务在本地运行
                factory.UserName = "guest";//用户名
                factory.Password = "guest";//密码 
                using (var connection = factory.CreateConnection())
                {
                    //创建通道channel
                    using (var channel = connection.CreateModel())
                    {
                        channel.QueueDeclare(queue: "FanoutExchangeZhaoxi001", durable: true, exclusive: false, autoDelete: false, arguments: null);
                        channel.QueueDeclare(queue: "FanoutExchangeZhaoxi002", durable: true, exclusive: false, autoDelete: false, arguments: null);
                        //在这里声明一个Fanout 类型的交换机
                        channel.ExchangeDeclare(exchange: "FanoutExchange", type: ExchangeType.Fanout, durable: true, autoDelete: false, arguments: null);
                        //交换机绑定队列，不需要标识
                        channel.QueueBind(queue: "FanoutExchangeZhaoxi001", exchange: "FanoutExchange", routingKey: string.Empty, arguments: null);
                        channel.QueueBind(queue: "FanoutExchangeZhaoxi002", exchange: "FanoutExchange", routingKey: string.Empty, arguments: null);

        //定义消费者                                      
                    var consumer = new EventingBasicConsumer(channel);
                    consumer.Received += (model, ea) =>
                    {
                        var body = ea.Body;
                        var message = Encoding.UTF8.GetString(body.ToArray());
                        //只是为了演示，并没有存入文本文件
                        Console.WriteLine($"消费者0：接收成功！【{message}】，邮件通知");
                    };
                    Console.WriteLine("消费者0:通知服务准备就绪...");
                    //处理消息
                    channel.BasicConsume(queue: "FanoutExchangeZhaoxi002",
                                         autoAck: true,
                                         consumer: consumer);
                    Console.ReadLine();
                }

            }
        }

        public static void Consumption1()
        {
            var factory = new ConnectionFactory();
            factory.HostName = "localhost";//RabbitMQ服务在本地运行
            factory.UserName = "guest";//用户名
            factory.Password = "guest";//密码 
            using (var connection = factory.CreateConnection())
            {
                //创建通道channel
                using (var channel = connection.CreateModel())
                {
                    channel.QueueDeclare(queue: "FanoutExchangeZhaoxi001", durable: true, exclusive: false, autoDelete: false, arguments: null);
                    channel.QueueDeclare(queue: "FanoutExchangeZhaoxi002", durable: true, exclusive: false, autoDelete: false, arguments: null);
                    //在这里声明一个Fanout 类型的交换机
                    channel.ExchangeDeclare(exchange: "FanoutExchange", type: ExchangeType.Fanout, durable: true, autoDelete: false, arguments: null);
                    //交换机绑定队列，不需要标识
                    channel.QueueBind(queue: "FanoutExchangeZhaoxi001", exchange: "FanoutExchange", routingKey: string.Empty, arguments: null);
                    channel.QueueBind(queue: "FanoutExchangeZhaoxi002", exchange: "FanoutExchange", routingKey: string.Empty, arguments: null);
                    //定义消费者                                      
                    var consumer = new EventingBasicConsumer(channel);
                    consumer.Received += (model, ea) =>
                    {
                        var body = ea.Body;
                        var message = Encoding.UTF8.GetString(body.ToArray());
                        //只是为了演示，并没有存入文本文件
                        Console.WriteLine($"消费者1：接收成功！【{message}】，邮件通知");
                    };
                    Console.WriteLine("消费者1：通知服务准备就绪...");
                    //处理消息
                    channel.BasicConsume(queue: "FanoutExchangeZhaoxi001",
                                         autoAck: true,
                                         consumer: consumer);
                    Console.ReadLine();
                }
            }
        }
}
```


#### 三、Topic 类型交换机

Topic交换机：可以做到模糊匹配；
Exchange绑定队列需要制定标识 标识 可以有自己的规则；标识可以有占位符、通配符；*/#*匹配一个单词、#匹配多个单词，在Direct基础上加上模糊匹配；

![image-20240125173739950](E:\图灵课堂\MQ专题\MQ专题.assets\image-20240125173739950.png)

生产者



```java
public class TopicExchange
{
    public static void Send()
    {
        var factory = new ConnectionFactory();
        factory.HostName = "localhost";//RabbitMQ服务在本地运行
        factory.UserName = "guest";//用户名
        factory.Password = "guest";//密码 
        using (var connection = factory.CreateConnection())
        {
            using (IModel channel = connection.CreateModel())
            {
                //声明一个Topic类型的交换机
                channel.ExchangeDeclare(exchange: "TopicExchange", type: ExchangeType.Topic, durable: true, autoDelete: false, arguments: null);
channel.QueueDeclare(queue: "ChinaQueue", durable: true, exclusive: false, autoDelete: false, arguments: null);

            channel.QueueDeclare(queue: "newsQueue", durable: true, exclusive: false, autoDelete: false, arguments: null);

            channel.QueueBind(queue: "ChinaQueue", exchange: "TopicExchange", routingKey: "China.#", arguments: null);

            channel.QueueBind(queue: "newsQueue", exchange: "TopicExchange", routingKey: "#.news", arguments: null);
            {
                string message = "来自中国的新闻消息。。。。";
                var body = Encoding.UTF8.GetBytes(message);
                channel.BasicPublish(exchange: "TopicExchange", routingKey: "China.news", basicProperties: null, body: body);
                Console.WriteLine($"消息【{message}】已发送到队列");
            }

            {
                string message = "来自中国的天气消息。。。。";
                var body = Encoding.UTF8.GetBytes(message);
                channel.BasicPublish(exchange: "TopicExchange", routingKey: "China.weather", basicProperties: null, body: body);
                Console.WriteLine($"消息【{message}】已发送到队列");
            }
            {
                string message = "来自美国的新闻消息。。。。";
                var body = Encoding.UTF8.GetBytes(message);
                channel.BasicPublish(exchange: "TopicExchange", routingKey: "usa.news", basicProperties: null, body: body);
                Console.WriteLine($"消息【{message}】已发送到队列");
            }
            {
                string message = "来自美国的天气消息。。。。";
                var body = Encoding.UTF8.GetBytes(message);
                channel.BasicPublish(exchange: "TopicExchange", routingKey: "usa.weather", basicProperties: null, body: body);
                Console.WriteLine($"消息【{message}】已发送到队列");
            }
        }
    }
}
}
```



消费者



```java
  public class TopicExchange
{
    public static void Consumption()
    {
        var factory = new ConnectionFactory();
        factory.HostName = "localhost";//RabbitMQ服务在本地运行
        factory.UserName = "guest";//用户名
        factory.Password = "guest";//密码 
        using (var connection = factory.CreateConnection())
        {
            using (IModel channel = connection.CreateModel())
            {
                channel.ExchangeDeclare(exchange: "TopicExchange", type: ExchangeType.Topic, durable: true, autoDelete: false, arguments: null);
                channel.QueueDeclare(queue: "ChinaQueue", durable: true, exclusive: false, autoDelete: false, arguments: null);
                channel.QueueBind(queue: "ChinaQueue", exchange: "TopicExchange", routingKey: "China.#", arguments: null);
                //定义消费者                                      
                var consumer = new EventingBasicConsumer(channel);
                consumer.Received += (model, ea) =>
                {
                    var body = ea.Body;
                    var message = Encoding.UTF8.GetString(body.ToArray());
                    Console.WriteLine($"接收成功！【{message}】");
                };
			//处理消息
            channel.BasicConsume(queue: "ChinaQueue",
                                 autoAck: true,
                                 consumer: consumer);

            Console.WriteLine("对来自于中国的消息比较感兴趣的 消费者");
        }
    }
}
}
```


#### 四、Headers 类型交换机

规则：headers类型的Exchange不依赖于routing key与binding key的匹配规则来路由消息，而是根据发送的消息内容中的headers属性进行匹配。在绑定Queue与Exchange时指定一组键值对以及x-match参数，x-match参数是字符串类型，可以设置为any或者all。如果设置为any，意思就是只要匹配到了headers表中的任何一对键值即可，all则代表需要全部匹配。

生产者      


```java
public class HeaderExchange
{
    public static void Send()
    {
        var factory = new ConnectionFactory();
        factory.HostName = "localhost";//RabbitMQ服务在本地运行
        factory.UserName = "guest";//用户名
        factory.Password = "guest";//密码 
        using (var connection = factory.CreateConnection())
        {
            using (var channel = connection.CreateModel())
            {

//声明Headers类型的交换机：HeaderExchange
            channel.ExchangeDeclare(exchange: "HeaderExchange", type: ExchangeType.Headers, durable: false, autoDelete: false, arguments: null);

            channel.QueueDeclare(queue: "HeaderExchangeAllqueue", durable: false, exclusive: false, autoDelete: false, arguments: null);
            channel.QueueDeclare(queue: "HeaderExchangeAnyqueue", durable: false, exclusive: false, autoDelete: false, arguments: null);
		Console.WriteLine("生产者准备就绪....");           
//绑定的时候，需要给arguments 指定一个字典的实例；根据字典中的 { "x-match","all/any"},
            //如果：{ "x-match","all"}, 发送消息的时候，带的参数列表必须和arguments参数中除了x-match以外，其他的必须都具备才能转发到对应的队列中去；
            //如果：{ "x-match","any"},发送消息的时候，带的参数列表必须和arguments参数中除了x-match以外，任何一个能够匹配就转发到该队列中去；

            channel.QueueBind(queue: "HeaderExchangeAllqueue", exchange: "HeaderExchange", routingKey: string.Empty,
                arguments: new Dictionary<string, object> {
                                                            { "x-match","all"},
                                                            { "teacher","Richard"},
                                                            { "pass","123"}});
            {
                string message = "teacher和pass都相同时发送的消息";
                IBasicProperties props = channel.CreateBasicProperties();
                props.Headers = new Dictionary<string, object>() {
                                                                   { "teacher","Richard"},
                                                                   { "pass","123"}
                                                                  };
                var body = Encoding.UTF8.GetBytes(message);
                //基本发布
                channel.BasicPublish(exchange: "HeaderExchange",
                                     routingKey: string.Empty,
                                     basicProperties: props,
                                     body: body);
                Console.WriteLine($"消息【{message}】已发送");
            }
            {
                string message = "teacher和pass有一个不相同时发送的消息";
                var props = channel.CreateBasicProperties();
                props.Headers = new Dictionary<string, object>() {
                                                                   { "teacher","Richard"},
                                                                   { "pass","234"}
                                                                  };
                var body = Encoding.UTF8.GetBytes(message);
                channel.BasicPublish(exchange: "HeaderExchange",
                                     routingKey: string.Empty,
                                     basicProperties: props,
                                     body: body);
                Console.WriteLine($"消息【{message}】已发送");
            }
            Console.WriteLine("**************************************************************");
            {
                channel.QueueBind(queue: "HeaderExchangeAnyqueue", exchange: "HeaderExchange", routingKey: string.Empty,
                arguments: new Dictionary<string, object> {
                                    { "x-match","any"},
                                    { "teacher","Richard"},
                                    { "pass","123"},});

                string msg = "teacher和pass完全相同时发送的消息";
                var props = channel.CreateBasicProperties();
                props.Headers = new Dictionary<string, object>() {
                                         { "teacher","Richard"},
                                         { "pass","123"}
                                    };
                var body = Encoding.UTF8.GetBytes(msg);
                channel.BasicPublish(exchange: "HeaderExchange",
                                     routingKey: string.Empty,
                                     basicProperties: props,
                                     body: body);
                Console.WriteLine($"消息【{msg}】已发送");
            }

            {
                string msg = "teacher和pass有一个不相同时发送的消息";
                var props = channel.CreateBasicProperties();
                props.Headers = new Dictionary<string, object>() {
                                         { "teacher","Richard"},
                                         { "pass","234"}
                                    };
                var body = Encoding.UTF8.GetBytes(msg);
                channel.BasicPublish(exchange: "HeaderExchange",
                                     routingKey: string.Empty,
                                     basicProperties: props,
                                     body: body);
                Console.WriteLine($"消息【{msg}】已发送");
            }
        }
    }
    Console.ReadKey();
	}
}
```



如果：{ “x-match”,“all”}, 发送消息的时候，带的参数列表必须和arguments参数中除了x-match以外，其他的必须都具备才能转发到对应的队列中去；

如果：{ “x-match”,“any”},发送消息的时候，带的参数列表必须和arguments参数中除了x-match以外，任何一个能够匹配就转发到该队列中去；

消费者

消费者同前面，因为这种交换机是对发布者的限制

### 四、RabbitMQ应用场景

---

RabbitMQ提供了很多中通讯方式，依然可以去官方查看：https://rabbitmq.com/getstarted.html

|                         七种通讯方式                         |
| :----------------------------------------------------------: |
| ![image20220121011637076.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/2746/1642502600000/51b8deace91547c6afa4e42de8a34f4c.png) |

#### 4.1 RabbitMQ提供的应用场景

- [Hello World!](https://rabbitmq.com/tutorials/tutorial-one-python.html)：为了入门操作！
- [Work queues](https://rabbitmq.com/tutorials/tutorial-two-python.html)：一个队列被多个消费者消费
- [Publish/Subscribe](https://rabbitmq.com/tutorials/tutorial-three-python.html)：手动创建Exchange（FANOUT）
- [Routing](https://rabbitmq.com/tutorials/tutorial-four-python.html)：手动创建Exchange（DIRECT）
- [Topics](https://rabbitmq.com/tutorials/tutorial-five-python.html)：手动创建Exchange（TOPIC）
- [RPC](https://rabbitmq.com/tutorials/tutorial-six-python.html)：RPC方式
- [Publisher Confirms](https://rabbitmq.com/tutorials/tutorial-seven-java.html)：保证消息可靠性

#### 4.2 构建Connection工具类

- 导入依赖：amqp-client，junit

  ```xml
  <dependencies>
      <dependency>
          <groupId>com.rabbitmq</groupId>
          <artifactId>amqp-client</artifactId>
          <version>5.9.0</version>
      </dependency>
      <dependency>
          <groupId>junit</groupId>
          <artifactId>junit</artifactId>
          <version>4.12</version>
      </dependency>
  </dependencies>
  ```

- 构建工具类：

  ```java
  package com.mashibing.util;
  
  import com.rabbitmq.client.Connection;
  import com.rabbitmq.client.ConnectionFactory;
  
  import java.io.IOException;
  import java.util.concurrent.TimeoutException;
  
  /**
   * @author tianming
   * @description
   */
  public class RabbitMQConnectionUtil {
  
      public static final String RABBITMQ_HOST = "192.168.11.32";
  
      public static final int RABBITMQ_PORT = 5672;
  
      public static final String RABBITMQ_USERNAME = "guest";
  
      public static final String RABBITMQ_PASSWORD = "guest";
  
      public static final String RABBITMQ_VIRTUAL_HOST = "/";
  
      /**
       * 构建RabbitMQ的连接对象
       * @return
       */
      public static Connection getConnection() throws Exception {
          //1. 创建Connection工厂
          ConnectionFactory factory = new ConnectionFactory();
  
          //2. 设置RabbitMQ的连接信息
          factory.setHost(RABBITMQ_HOST);
          factory.setPort(RABBITMQ_PORT);
          factory.setUsername(RABBITMQ_USERNAME);
          factory.setPassword(RABBITMQ_PASSWORD);
          factory.setVirtualHost(RABBITMQ_VIRTUAL_HOST);
  
          //3. 返回连接对象
          Connection connection = factory.newConnection();
          return connection;
      }
  
  }
  ```

#### 4.3 Hello World

|                                                              |
| :----------------------------------------------------------: |
| ![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/2746/1642502600000/d95491e1c6ef48ecbe988e6c89517f73.png) |

一个P向queue发送一个message，一个C从该queue接收message并打印。

producer，连接至RabbitMQ Server，声明队列，发送message，关闭连接，退出

生产者：

```java
package com.mashibing.helloworld;

import com.mashibing.util.RabbitMQConnectionUtil;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import org.junit.Test;

/**
 * @author tianming
 * @description
 * @date 2022/1/24 22:54
 */
public class Publisher {

    public static final String QUEUE_NAME = "hello";

    @Test
    public void publish() throws Exception {
        //1. 获取连接对象
        Connection connection = RabbitMQConnectionUtil.getConnection();

        //2. 构建Channel
        Channel channel = connection.createChannel();

        //3. 构建队列
        channel.queueDeclare(QUEUE_NAME,false,false,false,null);

        //4. 发布消息
        String message = "Hello World!";
        channel.basicPublish("",QUEUE_NAME,null,message.getBytes());
        System.out.println("消息发送成功！");
    }
}
```

消费者：

```java
package com.mashibing.helloworld;

import com.mashibing.util.RabbitMQConnectionUtil;
import com.rabbitmq.client.*;
import org.junit.Test;

import java.io.IOException;

/**
 * @author tianming
 * @description
 * @date 2022/1/24 23:02
 */
public class Consumer {

    @Test
    public void consume() throws Exception {
        //1. 获取连接对象
        Connection connection = RabbitMQConnectionUtil.getConnection();

        //2. 构建Channel
        Channel channel = connection.createChannel();

        //3. 构建队列
        channel.queueDeclare(Publisher.QUEUE_NAME,false,false,false,null);

        //4. 监听消息
        DefaultConsumer callback = new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("消费者获取到消息：" + new String(body,"UTF-8"));
            }
        };
        channel.basicConsume(Publisher.QUEUE_NAME,true,callback);
        System.out.println("开始监听队列");

        System.in.read();
    }
}
```

#### 4.4 Work Queues

|                   WorkQueues需要学习的内容                   |
| :----------------------------------------------------------: |
| ![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/2746/1642502600000/6e48f282dfa44770816df4ffa4f3b2fc.png) |

- 生产者：生产者和Hello World的形式是一样的，都是将消息推送到默认交换机。

- 消费者：让消费者关闭自动ack，并且设置消息的流控，最终实现消费者可以尽可能去多消费消息

  将耗时的消息处理通过队列分配给多个consumer来处理，我们称此处的consumer为worker，我们将此处的queue称为Task Queue，其目的是为了避免资源密集型的task的同步处理，也即立即处理task并等待完成。相反，调度task使其稍后被处理。也即把task封装进message并发送到task queue，worker进程在后台运行，从task queue取出task并执行job，若运行了多个worker，则task可在多个worker间分配。
  
  
  
  producer
  建立连接，声明队列，发送可以模拟耗时任务的message，断开连接、退出。
  
  consumer
  建立连接，声明队列，不断的接收message，处理任务，进行确认。
  
  ```java
  package com.mashibing.workqueues;
  
  import com.mashibing.util.RabbitMQConnectionUtil;
  import com.rabbitmq.client.*;
  import org.junit.Test;
  
  import java.io.IOException;
  
  /**
   * @author tianming
   * @description
   * @date 2022/1/25 19:52
   */
  public class Consumer {
  
      @Test
      public void consume1() throws Exception {
          //1. 获取连接对象
          Connection connection = RabbitMQConnectionUtil.getConnection();
  
          //2. 构建Channel
          Channel channel = connection.createChannel();
  
          //3. 构建队列
          channel.queueDeclare(Publisher.QUEUE_NAME,false,false,false,null);
  
          //3.5 设置消息的流控
          channel.basicQos(3);
  
          //4. 监听消息
          DefaultConsumer callback = new DefaultConsumer(channel){
              @Override
              public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                  try {
                      Thread.sleep(100);
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
                  System.out.println("消费者1号-获取到消息：" + new String(body,"UTF-8"));
                  channel.basicAck(envelope.getDeliveryTag(),false);
              }
          };
          channel.basicConsume(Publisher.QUEUE_NAME,false,callback);
          System.out.println("开始监听队列");
  
          System.in.read();
      }
  
      @Test
      public void consume2() throws Exception {
          //1. 获取连接对象
          Connection connection = RabbitMQConnectionUtil.getConnection();
  
          //2. 构建Channel
          Channel channel = connection.createChannel();
  
          //3. 构建队列
          channel.queueDeclare(Publisher.QUEUE_NAME,false,false,false,null);
  
          channel.basicQos(3);
  
          //4. 监听消息
          DefaultConsumer callback = new DefaultConsumer(channel){
              @Override
              public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                  try {
                      Thread.sleep(1000);
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
                  System.out.println("消费者2号-获取到消息：" + new String(body,"UTF-8"));
                  channel.basicAck(envelope.getDeliveryTag(),false);
              }
          };
          channel.basicConsume(Publisher.QUEUE_NAME,false,callback);
          System.out.println("开始监听队列");
  
          System.in.read();
      }
  }
  ```

#### 4.5 Publish/Subscribe

|                       自定义一个交换机                       |
| :----------------------------------------------------------: |
| ![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/2746/1642502600000/82e4111651154c3c82ea003ecc7c2ad2.png) |

在应用场景2中一个message(task)仅被传递给了一个comsumer(worker)。现在我们设法将一个message传递给多个consumer。

这种模式被称为publish/subscribe**发布订阅**。此处以一个简单的日志系统为例进行说明。该系统包含一个log发送程序和一个log接收并打印的程序。由log发送者发送到queue的消息可以被所有运行的log接收者接收。因此，我们可以运行一个log接收者直接在屏幕上显示log，同时运行另一个log接收者将log写入磁盘文件。

生产者：自行构建Exchange并绑定指定队列[（FANOUT类型）]()

```java
package com.mashibing.pubsub;

import com.mashibing.util.RabbitMQConnectionUtil;
import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import org.junit.Test;

/**
 * @author tianming
 * @description
 * @date 2022/1/25 20:08
 */
public class Publisher {

    public static final String EXCHANGE_NAME = "pubsub";
    public static final String QUEUE_NAME1 = "pubsub-one";
    public static final String QUEUE_NAME2 = "pubsub-two";
    @Test
    public void publish() throws Exception {
        //1. 获取连接对象
        Connection connection = RabbitMQConnectionUtil.getConnection();

        //2. 构建Channel
        Channel channel = connection.createChannel();

        //3. 构建交换机
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.FANOUT);

        //4. 构建队列
        channel.queueDeclare(QUEUE_NAME1,false,false,false,null);
        channel.queueDeclare(QUEUE_NAME2,false,false,false,null);

        //5. 绑定交换机和队列，使用的是FANOUT类型的交换机，绑定方式是直接绑定
        channel.queueBind(QUEUE_NAME1,EXCHANGE_NAME,"");
        channel.queueBind(QUEUE_NAME2,EXCHANGE_NAME,"");

        //6. 发消息到交换机
        channel.basicPublish(EXCHANGE_NAME,"45jk6h645jk",null,"publish/subscribe!".getBytes());
        System.out.println("消息成功发送！");
    }
}
```

#### 4.6 Routing

|                      DIRECT类型Exchange                      |
| :----------------------------------------------------------: |
| ![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/2746/1642502600000/4d314cc2631040bba6ae01e1ac0e6dc3.png) |

生产者：在绑定Exchange和Queue时，需要指定好routingKey，同时在发送消息时，也指定routingKey，只有routingKey一致时，才会把指定的消息路由到指定的Queue

只把指定的message类型发送给其subscriber，比如，只把error message写到log file而将所有log message显示在控制台。

```java
package com.mashibing.routing;

import com.mashibing.util.RabbitMQConnectionUtil;
import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import org.junit.Test;

/**
 * @author tianming
 * @description
 * @date 2022/1/25 20:20
 */
public class Publisher {

    public static final String EXCHANGE_NAME = "routing";
    public static final String QUEUE_NAME1 = "routing-one";
    public static final String QUEUE_NAME2 = "routing-two";
    @Test
    public void publish() throws Exception {
        //1. 获取连接对象
        Connection connection = RabbitMQConnectionUtil.getConnection();

        //2. 构建Channel
        Channel channel = connection.createChannel();

        //3. 构建交换机
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);

        //4. 构建队列
        channel.queueDeclare(QUEUE_NAME1,false,false,false,null);
        channel.queueDeclare(QUEUE_NAME2,false,false,false,null);

        //5. 绑定交换机和队列
        channel.queueBind(QUEUE_NAME1,EXCHANGE_NAME,"ORANGE");
        channel.queueBind(QUEUE_NAME2,EXCHANGE_NAME,"BLACK");
        channel.queueBind(QUEUE_NAME2,EXCHANGE_NAME,"GREEN");

        //6. 发消息到交换机
        channel.basicPublish(EXCHANGE_NAME,"ORANGE",null,"大橙子！".getBytes());
        channel.basicPublish(EXCHANGE_NAME,"BLACK",null,"黑布林大狸子".getBytes());
        channel.basicPublish(EXCHANGE_NAME,"WHITE",null,"小白兔！".getBytes());
        System.out.println("消息成功发送！");


    }

}

```

#### 4.7 Topic

|                          Topic模式                           |
| :----------------------------------------------------------: |
| ![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/2746/1642502600000/d5d670ab4cd943219a85b6973354b005.png) |

生产者：TOPIC类型可以编写带有特殊意义的routingKey的绑定方式

```java
package com.mashibing.topics;

import com.mashibing.util.RabbitMQConnectionUtil;
import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import org.junit.Test;

/**
 * @author tianming
 * @description
 * @date 2022/1/25 20:28
 */
public class Publisher {

    public static final String EXCHANGE_NAME = "topic";
    public static final String QUEUE_NAME1 = "topic-one";
    public static final String QUEUE_NAME2 = "topic-two";
    @Test
    public void publish() throws Exception {
        //1. 获取连接对象
        Connection connection = RabbitMQConnectionUtil.getConnection();

        //2. 构建Channel
        Channel channel = connection.createChannel();

        //3. 构建交换机
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.TOPIC);

        //4. 构建队列
        channel.queueDeclare(QUEUE_NAME1,false,false,false,null);
        channel.queueDeclare(QUEUE_NAME2,false,false,false,null);

        //5. 绑定交换机和队列，
        // TOPIC类型的交换机在和队列绑定时，需要以aaa.bbb.ccc..方式编写routingkey
        // 其中有两个特殊字符：*（相当于占位符），#（相当通配符）
        channel.queueBind(QUEUE_NAME1,EXCHANGE_NAME,"*.orange.*");
        channel.queueBind(QUEUE_NAME2,EXCHANGE_NAME,"*.*.rabbit");
        channel.queueBind(QUEUE_NAME2,EXCHANGE_NAME,"lazy.#");

        //6. 发消息到交换机
        channel.basicPublish(EXCHANGE_NAME,"big.orange.rabbit",null,"大橙兔子！".getBytes());
        channel.basicPublish(EXCHANGE_NAME,"small.white.rabbit",null,"小白兔".getBytes());
        channel.basicPublish(EXCHANGE_NAME,"lazy.dog.dog.dog.dog.dog.dog",null,"懒狗狗狗狗狗狗".getBytes());
        System.out.println("消息成功发送！");

    }
}
```

#### 4.8 RPC（了解）

> 因为两个服务在交互时，可以尽量做到Client和Server的解耦，通过RabbitMQ进行解耦操作
>
> 需要让Client发送消息时，携带两个属性：
>
> - replyTo告知Server将相应信息放到哪个队列
> - correlationId告知Server发送相应消息时，需要携带位置标示来告知Client响应的信息

|                           RPC方式                            |
| :----------------------------------------------------------: |
| ![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/2746/1642502600000/3fc0407cedf6428dacbcc0c69ffa3682.png) |

客户端：

```java
package com.mashibing.rpc;

import com.mashibing.util.RabbitMQConnectionUtil;
import com.rabbitmq.client.*;
import org.junit.Test;

import java.io.IOException;
import java.util.UUID;

/**
 * @author tianming
 * @description
 * @date 2022/2/8 20:03
 */
public class Publisher {

    public static final String QUEUE_PUBLISHER = "rpc_publisher";
    public static final String QUEUE_CONSUMER = "rpc_consumer";

    @Test
    public void publish() throws Exception {
        //1. 获取连接对象
        Connection connection = RabbitMQConnectionUtil.getConnection();

        //2. 构建Channel
        Channel channel = connection.createChannel();

        //3. 构建队列
        channel.queueDeclare(QUEUE_PUBLISHER,false,false,false,null);
        channel.queueDeclare(QUEUE_CONSUMER,false,false,false,null);

        //4. 发布消息
        String message = "Hello RPC!";
        String uuid = UUID.randomUUID().toString();
        AMQP.BasicProperties props = new AMQP.BasicProperties()
                .builder()
                .replyTo(QUEUE_CONSUMER)
                .correlationId(uuid)
                .build();
        channel.basicPublish("",QUEUE_PUBLISHER,props,message.getBytes());

        channel.basicConsume(QUEUE_CONSUMER,false,new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String id = properties.getCorrelationId();
                if(id != null && id.equalsIgnoreCase(uuid)){
                    System.out.println("接收到服务端的响应：" + new String(body,"UTF-8"));
                }
                channel.basicAck(envelope.getDeliveryTag(),false);
            }
        });
        System.out.println("消息发送成功！");

        System.in.read();
    }


}
```

服务端：

```java
package com.mashibing.rpc;

import com.mashibing.helloworld.Publisher;
import com.mashibing.util.RabbitMQConnectionUtil;
import com.rabbitmq.client.*;
import org.junit.Test;

import java.io.IOException;

/**
 * @author tianming
 * @description
 * @date 2022/1/24 23:02
 */
public class Consumer {

    public static final String QUEUE_PUBLISHER = "rpc_publisher";
    public static final String QUEUE_CONSUMER = "rpc_consumer";

    @Test
    public void consume() throws Exception {
        //1. 获取连接对象
        Connection connection = RabbitMQConnectionUtil.getConnection();

        //2. 构建Channel
        Channel channel = connection.createChannel();

        //3. 构建队列
        channel.queueDeclare(QUEUE_PUBLISHER,false,false,false,null);
        channel.queueDeclare(QUEUE_CONSUMER,false,false,false,null);


        //4. 监听消息
        DefaultConsumer callback = new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("消费者获取到消息：" + new String(body,"UTF-8"));
                String resp = "获取到了client发出的请求，这里是响应的信息";
                String respQueueName = properties.getReplyTo();
                String uuid = properties.getCorrelationId();
                AMQP.BasicProperties props = new AMQP.BasicProperties()
                        .builder()
                        .correlationId(uuid)
                        .build();
                channel.basicPublish("",respQueueName,props,resp.getBytes());
                channel.basicAck(envelope.getDeliveryTag(),false);
            }
        };
        channel.basicConsume(QUEUE_PUBLISHER,false,callback);
        System.out.println("开始监听队列");

        System.in.read();
    }
}
```

#### 4.9 publisher Confirms

发布确认模式。生产者发送消息给RabbitMQ时候，如果RabbitMQ正确接收到消息后，需要发给一个ACK标识给生产者，生产者接收到ACK标记后，就可以确认这一条消息发送成功啦。如果生产者没有接收到ACK标识，则可以重复发送这一条消息给RabbitMQ，这就可以确保消息不丢失。



![258b0e866b1e403095f7924c1d1a044e](E:\图灵课堂\MQ专题\MQ专题.assets\258b0e866b1e403095f7924c1d1a044e.png)

生产者：【消息确认--单条确认】

```java
package com.rabbitmq.demo.confirm;
 
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
 
/**
 * @version 1.0.0
 * @Date: 2024/2/25 16:23
 * @Copyright (C) tianming
 * @Description: 消息生产者
 */
public class Producer {
    public static void main(String[] args) {
        // 1、创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        // 2、设置连接的 RabbitMQ 服务地址
        factory.setHost("127.0.0.1"); // 默认就是本机
        factory.setPort(5672); // 默认就是 5672 端口
        // 3、获取连接
        Connection connection = null; // 连接
        Channel channel = null; // 通道
        try {
            connection = factory.newConnection();
            // 4、获取通道
            channel = connection.createChannel();
            // TODO 开启消息确认机制
            channel.confirmSelect();
            // 5、声明 Exchange，如果不存在，则会创建
            String exchangeName = "exchange_direct_2023";
            channel.exchangeDeclare(exchangeName, "direct");
            // 6、发送消息
            for (int i = 0; i < 10; i++) {
                // 路由键唯一标识
                String routingKey = "error";
                if (i % 3 == 0) {
                    routingKey = "info";
                } else if (i % 3 == 1) {
                    routingKey = "warn";
                }
                String message = "这是发布确认模式，发送的第【" + (i+1) + "】条【" + routingKey + "】消息数据";
                channel.basicPublish(exchangeName, routingKey, null, message.getBytes());
                // 等待RabbitMQ返回ACK标识
                boolean wait = channel.waitForConfirms();
                System.out.println("RabbitMQ是否接收成功: " + wait);
                if (!wait) {
                    // 消息发送失败，则可以重新发送
                    channel.basicPublish(exchangeName, routingKey, null, message.getBytes());
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (null != channel) {
                try {
                    channel.close();
                } catch (Exception e) {}
            }
            if (null != connection) {
                try {
                    connection.close();
                } catch (Exception e) {}
            }
        }
    }
}
```

生产者【消息确认--批量确认】

```java
package com.rabbitmq.demo.confirm;
 
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
 
import java.io.IOException;
 
/**
 * @version 1.0.0
 * @Date: 2024/2/25 16:23
 * @Copyright (C) tianming
 * @Description: 消息生产者
 */
public class ProducerBatch {
    public static void main(String[] args) {
        // 1、创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        // 2、设置连接的 RabbitMQ 服务地址
        factory.setHost("127.0.0.1"); // 默认就是本机
        factory.setPort(5672); // 默认就是 5672 端口
        // 3、获取连接
        Connection connection = null; // 连接
        Channel channel = null; // 通道
        try {
            connection = factory.newConnection();
            // 4、获取通道
            channel = connection.createChannel();
            // TODO 开启消息确认机制
            channel.confirmSelect();
            // 5、声明 Exchange，如果不存在，则会创建
            String exchangeName = "exchange_direct_2023";
            channel.exchangeDeclare(exchangeName, "direct");
            // 6、发送消息
            int batchSize = 3;
            int count = 0;
            for (int i = 0; i < 10; i++) {
                // 路由键唯一标识
                String routingKey = "error";
                if (i % 3 == 0) {
                    routingKey = "info";
                } else if (i % 3 == 1) {
                    routingKey = "warn";
                }
                String message = "这是发布确认模式，发送的第【" + (i+1) + "】条【" + routingKey + "】消息数据";
                channel.basicPublish(exchangeName, routingKey, null, message.getBytes());
                // 批量确认
                if (count == batchSize) {
                    // 等待RabbitMQ返回ACK标识
                    channel.waitForConfirmsOrDie();
                    count = 0;
                }
                count++;
            }
        } catch (IOException e) {
            System.out.println("消息发送失败啦");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (null != channel) {
                try {
                    channel.close();
                } catch (Exception e) {}
            }
            if (null != connection) {
                try {
                    connection.close();
                } catch (Exception e) {}
            }
        }
    }
}
```

生产者【消息确认--异步确认】

```java
package com.rabbitmq.demo.confirm;
 
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.ConfirmCallback;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
 
import java.io.IOException;
 
/**
 * @version 1.0.0
 * @Date: 2024/2/25 16:23
 * @Copyright (C) tianming
 * @Description: 消息生产者
 */
public class ProducerAsync {
    public static void main(String[] args) {
        // 1、创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        // 2、设置连接的 RabbitMQ 服务地址
        factory.setHost("127.0.0.1"); // 默认就是本机
        factory.setPort(5672); // 默认就是 5672 端口
        // 3、获取连接
        Connection connection = null; // 连接
        Channel channel = null; // 通道
        try {
            connection = factory.newConnection();
            // 4、获取通道
            channel = connection.createChannel();
            // TODO 开启消息确认机制
            channel.confirmSelect();
            // 5、声明 Exchange，如果不存在，则会创建
            String exchangeName = "exchange_confirm_2023";
            channel.exchangeDeclare(exchangeName, "direct");
            // TODO 一定要先调用监听接口，在发送消息
            channel.addConfirmListener(new ConfirmCallback() {
                @Override
                public void handle(long deliveryTag, boolean multiple) throws IOException {
                    System.out.println("RabbitMQ接收成功啦.....消息的标识deliveryTag=" + deliveryTag 
                            + ",批量发送多条消息multiple=" + multiple);
                }
            }, new ConfirmCallback() {
                @Override
                public void handle(long deliveryTag, boolean multiple) throws IOException {
                    System.out.println("RabbitMQ接收失败啦.....");
                }
            });
            for (int i = 0; i < 10; i++) {
                // 6、发送消息
                String message = "这是发布确认模式，发送的消息数据";
                channel.basicPublish(exchangeName, "queue_confirm_2023", null, message.getBytes());
            }
        } catch (IOException e) {
            System.out.println("消息发送失败啦");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (null != channel) {
                try {
                    channel.close();
                } catch (Exception e) {}
            }
            if (null != connection) {
                try {
                    connection.close();
                } catch (Exception e) {}
            }
        }
    }
}
```



### 五、SpringBoot操作RabbitMQ

---

#### 5.1 SpringBoot声明信息

- 创建项目

- 导入依赖

  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-amqp</artifactId>
  </dependency>
  ```

- 配置RabbitMQ信息

  ```yml
  spring:
    rabbitmq:
      host: 192.168.11.32
      port: 5672
      username: guest
      password: guest
      virtual-host: /
  
  ```

- 声明交换机&队列

  ```java
  package com.mashibing.rabbitmqboot.config;
  
  import org.springframework.amqp.core.*;
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;
  
  /**
   * @author tianming
   * @description
   * @date 2022/2/8 20:25
   */
  @Configuration
  public class RabbitMQConfig {
  
      public static final String EXCHANGE = "boot-exchange";
      public static final String QUEUE = "boot-queue";
      public static final String ROUTING_KEY = "*.black.*";
  ```


      @Bean
      public Exchange bootExchange(){
          // channel.DeclareExchange
          return ExchangeBuilder.topicExchange(EXCHANGE).build();
      }
    
      @Bean
      public Queue bootQueue(){
          return QueueBuilder.durable(QUEUE).build();
      }
    
      @Bean
      public Binding bootBinding(Exchange bootExchange,Queue bootQueue){
          return BindingBuilder.bind(bootQueue).to(bootExchange).with(ROUTING_KEY).noargs();
      }

  }

  ```
#### 5.2 生产者操作

​```java
package com.mashibing.rabbitmqboot;

import com.mashibing.rabbitmqboot.config.RabbitMQConfig;
import org.junit.jupiter.api.Test;
import org.springframework.amqp.AmqpException;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.core.MessagePostProcessor;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

/**
 * @author tianming
 * @description
 * @date 2022/2/8 21:05
 */
@SpringBootTest
public class PublisherTest {

    @Autowired
    public RabbitTemplate rabbitTemplate;

    @Test
    public void publish(){
        rabbitTemplate.convertAndSend(RabbitMQConfig.EXCHANGE,"big.black.dog","message");
        System.out.println("消息发送成功");
    }


    @Test
    public void publishWithProps(){
        rabbitTemplate.convertAndSend(RabbitMQConfig.EXCHANGE, "big.black.dog", "messageWithProps", new MessagePostProcessor() {
            @Override
            public Message postProcessMessage(Message message) throws AmqpException {
                message.getMessageProperties().setCorrelationId("123");
                return message;
            }
        });
        System.out.println("消息发送成功");
    }
}
  ```

#### 5.3 消费者操作

```java
package com.mashibing.rabbitmqboot;

import com.mashibing.rabbitmqboot.config.RabbitMQConfig;
import com.rabbitmq.client.Channel;
import org.junit.jupiter.api.Test;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.context.annotation.Configuration;
import org.springframework.stereotype.Component;

import java.io.IOException;

/**
 * @author tianming
 * @description
 * @date 2022/2/8 21:11
 */
@Component
public class ConsumeListener {

    @RabbitListener(queues = RabbitMQConfig.QUEUE)
    public void consume(String msg, Channel channel, Message message) throws IOException {
        System.out.println("队列的消息为：" + msg);
        String correlationId = message.getMessageProperties().getCorrelationId();
        System.out.println("唯一标识为：" + correlationId);
        channel.basicAck(message.getMessageProperties().getDeliveryTag(),false);
    }
}

```

### 六、RabbitMQ保证消息可靠性



---

#### 6.1 保证消息一定送达到Exchange

Confirm机制

可以通过Confirm效果保证消息一定送达到Exchange，官方提供了三种方式，选择了对于效率影响最低的异步回调的效果

```java
//4. 开启confirms
channel.confirmSelect();

//5. 设置confirms的异步回调
channel.addConfirmListener(new ConfirmListener() {
    @Override
    public void handleAck(long deliveryTag, boolean multiple) throws IOException {
        System.out.println("消息成功的发送到Exchange！");
    }

    @Override
    public void handleNack(long deliveryTag, boolean multiple) throws IOException {
        System.out.println("消息没有发送到Exchange，尝试重试，或者保存到数据库做其他补偿操作！");
    }
});
```

#### 6.2 保证消息可以路由到Queue

Return机制

为了保证Exchange上的消息一定可以送达到Queue

```java
//6. 设置Return回调，确认消息是否路由到了Queue
channel.addReturnListener(new ReturnListener() {
    @Override
    public void handleReturn(int replyCode, String replyText, String exchange, String routingKey, AMQP.BasicProperties properties, byte[] body) throws IOException {
        System.out.println("消息没有路由到指定队列，做其他的补偿措施！！");
    }
});
//7. 在发送消息时，将basicPublish方法参数中的mandatory设置为true，即可开启Return机制，当消息没有路由到队列中时，就会执行return回调
```

#### 6.3 保证Queue可以持久化消息

DeliveryMode设置消息持久化

DeliveryMode设置为2代表持久化，如果设置为1，就代表不会持久化。

```java
//7. 设置消息持久化
AMQP.BasicProperties props = new AMQP.BasicProperties()
    .builder()
    .deliveryMode(2)
    .build();

//7. 发布消息
channel.basicPublish("","confirms",true,props,message.getBytes());
```

#### 6.4 保证消费者可以正常消费消息

`详情看WorkQueue模式`

#### 6.5 SpringBoot实现上述操作

##### 6.5.1 Confirm

- 编写配置文件开启Confirm机制

  ```yml
  spring:
    rabbitmq:
      publisher-confirm-type: correlated  # 新版本
      publisher-confirms: true  # 老版本 
  ```

- 在发送消息时，配置RabbitTemplate

  ```java
  @Test
  public void publishWithConfirms() throws IOException {
      rabbitTemplate.setConfirmCallback(new RabbitTemplate.ConfirmCallback() {
          @Override
          public void confirm(CorrelationData correlationData, boolean ack, String cause) {
              if(ack){
                  System.out.println("消息已经送达到交换机！！");
              }else{
                  System.out.println("消息没有送达到Exchange，需要做一些补偿操作！！retry！！！");
              }
          }
      });
      rabbitTemplate.convertAndSend(RabbitMQConfig.EXCHANGE,"big.black.dog","message");
      System.out.println("消息发送成功");
  
      System.in.read();
  }
  ```

##### 6.5.2 Return

- 编写配置文件开启Return机制

  ```yml
  spring:
    rabbitmq:
      publisher-returns: true # 开启Return机制
  ```

- 在发送消息时，配置RabbitTemplate

  ```java
  @Test
  public void publishWithReturn() throws IOException {
      // 新版本用 setReturnsCallback ，老版本用setReturnCallback
      rabbitTemplate.setReturnsCallback(new RabbitTemplate.ReturnsCallback() {
          @Override
          public void returnedMessage(ReturnedMessage returned) {
              String msg = new String(returned.getMessage().getBody());
              System.out.println("消息：" + msg + "路由队列失败！！做补救操作！！");
          }
      });
      rabbitTemplate.convertAndSend(RabbitMQConfig.EXCHANGE,"big.black.dog","message");
      System.out.println("消息发送成功");
  
      System.in.read();
  }
  ```

##### 6.5.3 消息持久化

```java
@Test
public void publishWithBasicProperties() throws IOException {
    rabbitTemplate.convertAndSend(RabbitMQConfig.EXCHANGE, "big.black.dog", "message", new MessagePostProcessor() {
        @Override
        public Message postProcessMessage(Message message) throws AmqpException {
            // 设置消息的持久化！
            message.getMessageProperties().setDeliveryMode(MessageDeliveryMode.PERSISTENT);
            return message;
        }
    });
    System.out.println("消息发送成功");
}
```

### 一：开启持久化的注意事项

### 1、RabbitMQ 持久化概述

**持久化**，即将原本存在于内存中的数据写入到磁盘上永久保存数据，防止服务宕机时内存数据的丢失。

上面给大家简单介绍了消息的持久化，实际上：

Rabbitmq 的持久化分为**队列持久化**、**消息持久化**和**交换器持久化**。

对于消息来说，不管是持久化的消息还是非持久化的消息都可以被写入到磁盘。

持久化的消息会同时写入磁盘和内存（加快读取速度），非持久化消息会在内存不够用时，将消息写入磁盘（一般重启之后就没有了）。

鱼和熊掌不能兼得。持久化开启比如会损失性能。

### 2、队列持久化

队列的持久化是在定义队列时的通过 `durable `参数来决定的，当 durable 为 true 时，才代表队列会持久化。例如：

```java
Connection connection = connectionFactory.newConnection();``Channel channel = connection.createChannel();
//第二个餐胡设置为true，代表队列持久化``
    channel.queueDeclare(``"queue.persistent.name"``, ``true``, ``false``, ``false``, ``null``);
```

关键的是第二个参数设置为 true，即 durable = true。Channel 类中 queueDeclare 的完整定义如下：

```java
Queue.DeclareOk queueDeclare(String queue, ``boolean` `durable, ``boolean` `exclusive, ``boolean` `autoDelete,``              ``Map<String, Object> arguments) ``throws` `IOException;
```

参数说明：

- queue：queue 的名称
- exclusive：排他队列，如果一个队列被声明为排他队列，该队列仅对首次申明它的连接可见，并在连接断开时自动删除。这里需要注意三点：
  - 排他队列是基于连接可见的，同一连接的不同信道是可以同时访问同一连接创建的排他队列；
  - “首次”，如果一个连接已经声明了一个排他队列，其他连接是不允许建立同名的排他队列的，这个与普通队列不同；
  - 即使该队列是持久化的，一旦连接关闭或者客户端退出，该排他队列都会被自动删除的，这种队列适用于一个客户端发送读取消息的应用场景。
- autoDelete：自动删除，如果该队列没有任何订阅的消费者的话，该队列会被自动删除。这种队列适用于临时队列。

**总结**：如果**将 queue 的持久化标识 durable 设置为 true，则代表是一个持久的队列**。当服务重启之后，队列仍然会存在，这是因为服务会把持久化的 queue 存放在硬盘上，当服务重启的时候，会重新加载这些被持久化的 queue。



### 3、消息持久化

**队列是可以被持久化，但是里面的消息是否为持久化那还要看消息的持久化设置**。

也就是说，重启之前 queue 里面如果还有未发出去的消息的话，重启之后，消息是否还存在队列里面就要取决于在发送消息时对消息的设置。

消息持久化的实现需要在发送消息时设置消息的持久化标识，例如：

```java
channel.basicPublish(``"exchange01"``, ``"routing_key01"``, MessageProperties.PERSISTENT_TEXT_PLAIN, ``"persistent_message"``.getBytes());
```

方法原型是：

```java
void` `basicPublish(String exchange, String routingKey, BasicProperties props, ``byte``[] body) ``throws` `IOException;
```

这里关键的是 `BasicProperties props `这个参数，它的定义如下：

```java
public` `BasicProperties(String contentType,``
                         //消息类型如：text/plain``   
                         String contentEncoding,
                         //编码``   
                         Map<String,Object> headers,
                         Integer deliveryMode,
                         //1:nonpersistent 2:persistent``   
                         Integer priority,
                         //优先级``   
                         String correlationId,
                         String replyTo,
                         //反馈队列
                         String expiration,
                         //expiration到期时间
                         String messageId,
                         Date timestamp,
                         String type,
                         String userId,
                         String appId,
                         String clusterId
                        )
```

其中 `deliveryMode=1 `代表不持久化， `deliveryMode=2 `代表持久化。而代码实现中的 `MessageProperties.PERSISTENT_PLAIN `值是官方提供的一个将 deliveryMode 设置为 2 的 BasicProperties 的对象：

```java
public static final BasicProperties PERSISTENT_TEXT_PLAIN =
    new BasicProperties(
        "text/plain",
        null,
        null,
        2,
        0, null, null, null,
        null, null, null, null,
        null, null
    );
```

除此之外，我们也可以使用另一种方式来设置消息持久化标志位：

```java
AMQP.BasicProperties.Builder builder = new AMQP.BasicProperties.Builder();
builder.deliveryMode(2); // 将 deliveryMode 值设置为 2 表示消息持久化
AMQP.BasicProperties properties = builder.build();
channel.basicPublish("exchange01", "routing_key01", properties, "persistent_message".getBytes());

```

至此，我们可以知道：**当 broker 服务其重启后，想要消息不丢失，既需要设置队列持久化，也需要设置消息持久化**。

**扩展知识：**

basicPublish 方法还有另外两个**重载方法**：

```java
void basicPublish(String exchange, String routingKey, boolean mandatory, BasicProperties props, byte[] body)
        throws IOException;
void basicPublish(String exchange, String routingKey, boolean mandatory, boolean immediate, BasicProperties props, byte[] body)
        throws IOException;
```

这里有两个关键的参数： `mandatory `和 `immediate `。这两个标识位都有当消息传递过程中不可达目的地时将消息返回给生产者的功能。下面简单讲解下这两个标识位：

**1）mandatory**

- 当 mandatory 标志位设置为 true 时：如果 exchange 根据自身类型和消息 routeKey 无法找到一个符合条件的 queue，那么会调用 `basic.return `方法将消息返回给生产者（Basic.Return + Content-Header + Content-Body）；
- 当 mandatory 设置为 false 时，如果出现上述情形的话，broker 会直接将消息扔掉。

**2）immediate**

- 当 immediate 标志位设置为 true 时：如果 exchange 在将消息路由到 queue(s) 时发现对于的 queue 上么有消费者，那么这条消息不会放入队列中。当与消息 routeKey 关联的所有 queue（一个或者多个）都没有消费者时，该消息会通过 `basic.return `方法返还给生产者。

**概括来说就是：**

- mandatory 标志告诉服务器至少将该消息 route 到一个队列中，否则将消息返还给生产者；
- immediate 标志告诉服务器如果该消息关联的 queue上有消费者，则马上将消息投递给它，如果所有 queue 都没有消费者，直接把消息返还给生产者，不用将消息入队列等待消费者了。



### 4、交换器持久化

对于消息的可靠性来说，只需要设置队列的持久化和消息的持久化即可。exchange 的持久化并没有什么影响，但是，如果 exchange 不设置持久化的话，当 broker 服务重启之后，exchange 将不复存在，这样会导致消息发送者 producer 无法正常发送消息。

所以，建议同样设置 exchange 的持久化。而 exchange 的持久化设置也特别简单，设置方法原型如下：

```
Exchange.DeclareOk exchangeDeclare(String exchange,String type,``boolean` `durable)``throws` `IOException;
```

- **exchange**：交换器的名称；
- **type**：交换器的类型，常见的如 `fanout direct topic `；
- **durable**：持久话标志位， durable 设置为 `true `表示持久化， 反之为非持久 。

所以，只需要在声明的时候将 durable 字段设置为 true 即可：

```
channel.exchangeDeclare(exchangeName, “direct/topic/header/fanout”, ``true``);
```



## 二、RabbitMQ 知识扩展



### 1、内存告警与内存换页



#### 1-1、内存告警

当**内存使用超过配置的阈值**时，RabbitMQ 会暂时**阻塞客户端的连接，并停止接收从客户端发来的消息，以避免服务崩溃**，客户端与服务端的心跳检测也会失败。

当出现内存告警时，可以通过管理命令临时调整内存大小：

```
RabbitMQctl set_vm_memory_high_watermark <fraction>
```

- `fraction `为内存阈值，RabbitMQ 默认是 0.4，表示当 RabbitMQ 使用的内存超过总内存的 40% 时，就会产生告警并阻塞所有生产则连接。

通过此命令修改的阈值在 RabbitMQ 重启之后将会失效，如果想要设置的阈值永久有效需要修改配置文件：

```
# 相对值，也就是前面的fraction，建议设置在``0.4``~``0.66``之间，不要超过``0.7``vm_memory_high_watermark.relative=``0.4``# 绝对值，单位为KB,MB,GB,对应的临时命令是：RabbitMQctl set_vm_memory_high_watermark absolute <value>``#vm_memory_high_watermark.absolute=1GB
```

修改完配置文件后，需要重启服务才会生效。



#### 1-2、内存换页

在某个 broker 节点触及内存阈值并阻塞生产者之前，它会尝试将队列内存中的消息换页存储到磁盘以释放内存空间。持久化和非持久化的消息都会被转储到磁盘中，其中持久化的消息本身就在磁盘中有一个备份，所以这里会将持久化的消息直接从内存中清除掉。

默认情况下，在**内存使用达到设置的阈值的 50% 时会进行换页操作**。也就是说，在默认的内存阈值 40% 的情况下，当内存超过 `40% * 50% = 20% `时会经行换页动作。

内存换页阈值可以通过在配置文件中设置来进行调整：

```
vm_memory_high_watermark_paging_ratio=``0.75
```

上面的配置将会在 RabbitMQ 内存使用率达到 30%（假设内存阈值是 0.4）时进行换页动作，并在 40% 时阻塞生产者（**当 vm_memory_high_watermark_paging_ratio 的值大于 1 时，相当于禁用了换页功能**）。



### 2、磁盘告警与配置



#### 2-1、磁盘告警

当磁盘剩余空间低于设置的阈值时，RabbitMQ 同样会阻塞生产者，这样可以避免因非持久化的消息持续换页而耗尽磁盘空间导致服务崩溃。

默认情况下，磁盘的阈值是50M，表示当磁盘剩余空间低于50M时，会阻塞生产者并停止内存中消息的换页动作。

这个阈值的设置可以减小，但不能完全消除因磁盘耗尽而导致崩溃的可能性。比如在两次磁盘空间检测期间内，磁盘空间从大于50M被耗尽到0M。



#### 2-2、修改磁盘告警阈值

可以通过以下命令临时调整磁盘阈值：

```
#设置具体大小，单位为KB/MB/GB``RabbitMQctl set_disk_free_limit <disk_limit>``#设置相对值，建议取值为``1.0``~``2.0``（相对于内存的倍数，如内存大小是8G,若为``1.0``，则表示磁盘剩余8G时，阻塞）``RabbitMQctl set_disk_free_limit mem_relative <fraction>
```

如果要永久生效需要对应的配置文件，配置如下（需要重启生效）：

```
disk_free_limit.relative=``2.0``#disk_free_limit_absolute=50MB
```

这里有个建议：**一个相对谨慎的做法是将磁盘阈值设置为与操作系统所显示的内存大小一致**。



### 3、数据写入磁盘时机

- 消息的正常写入磁盘流程为：消息数据写入到缓存 Buffer 中（大小为 1 M），Buffer 数据满了之后会写入内存文件中，最后再刷新到磁盘文件中；
- 存在个固定的刷盘时间：25ms，也就是不管 Buffe r满不满，每隔 25ms，Buffer 里的数据及未刷新到磁盘的文件内容必定会刷到磁盘；
- 每次消息写入后，如果没有后续写入请求，则会直接将已写入的消息刷到磁盘：使用 Erlang 的 receive x after 0 来实现。只要进程的信箱里没有消息，则产生一个 timeout 消息，而 timeout 会触发刷盘操作。



### 4、磁盘消息格式

消息保存于 $MNESIA/msg_store_persistent/x.rdq 文件中，其中 x 为数字编号，从 1 开始，每个文件最大为 16M（16777216），超过这个大小会生成新的文件，文件编号加 1。

文件中的消息格式如下：

```
<<Size:``64``, MsgId:``16``/binary, MsgBody>>
```

- **MsgId** 为 RabbitMQ 通过 `rabbit_guid:gen()` 每一个消息生成的 GUID；
- **MsgBody** 会包含消息对应的 exchange，routing_keys，消息的内容，消息对应的协议版本，消息内容格式（二进制还是其它）等等。



### 5、磁盘文件删除机制

- 当所有磁盘文件中的垃圾消息（已经被删除的消息）比例大于阈值（GARBAGE_FRACTION = 0.5）时，会触发文件合并操作（至少有三个文件存在的情况下），以提高磁盘利用率。
- publish 消息时写入内容，ack 消息时删除内容（更新该文件的有用数据大小），当一个文件的有用数据等于 0 时，删除该文件。



### 七、RabbitMQ死信队列&延迟交换机

#### 7.1 什么是死信

|                        死信&死信队列                         |
| :----------------------------------------------------------: |
| ![image-20240125182100056](E:\图灵课堂\MQ专题\MQ专题.assets\image-20240125182100056.png) |

- 消息存活时间超过 TTL 过期
- 队列达到最大长度（队列满了，无法再添加数据到 mq 中）
- 消息被拒绝（basic.reject 或 basic.nack）并且 requeue=false（不再重新入队）

上面消息被视为 死信 消息，如果配置了死信队列，此类型消息会进入死信队列。没有配置则直接丢弃。

死信队列的应用：需要延迟处理的死信消息，根据业务队列路由key匹配去消费 死信队列里面的消息。

（可为任何类型路由Direct，Fanout，Topic）

- 基于死信队列在队列消息已满的情况下，消息也不会丢失
- 实现延迟消费的效果。比如：下订单时，有15分钟的付款时间

#### 7.2 实现死信队列

##### 7.2.1 准备Exchange&Queue

```java
package com.mashibing.rabbitmqboot.config;

import org.springframework.amqp.core.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @author tianming
 * @description
 * @date 2022/2/10 15:04
 */
@Configuration
public class DeadLetterConfig {

    public static final String NORMAL_EXCHANGE = "normal-exchange";
    public static final String NORMAL_QUEUE = "normal-queue";
    public static final String NORMAL_ROUTING_KEY = "normal.#";

    public static final String DEAD_EXCHANGE = "dead-exchange";
    public static final String DEAD_QUEUE = "dead-queue";
    public static final String DEAD_ROUTING_KEY = "dead.#";


    @Bean
    public Exchange normalExchange(){
        return ExchangeBuilder.topicExchange(NORMAL_EXCHANGE).build();
    }

    @Bean
    public Queue normalQueue(){
        return QueueBuilder.durable(NORMAL_QUEUE).deadLetterExchange(DEAD_EXCHANGE).deadLetterRoutingKey("dead.abc").build();
    }

    @Bean
    public Binding normalBinding(Queue normalQueue,Exchange normalExchange){
        return BindingBuilder.bind(normalQueue).to(normalExchange).with(NORMAL_ROUTING_KEY).noargs();
    }


    @Bean
    public Exchange deadExchange(){
        return ExchangeBuilder.topicExchange(DEAD_EXCHANGE).build();
    }

    @Bean
    public Queue deadQueue(){
        return QueueBuilder.durable(DEAD_QUEUE).build();
    }

    @Bean
    public Binding deadBinding(Queue deadQueue,Exchange deadExchange){
        return BindingBuilder.bind(deadQueue).to(deadExchange).with(DEAD_ROUTING_KEY).noargs();
    }

}
```

##### 7.2.2 实现效果

- 基于消费者进行reject或者nack实现死信效果

  ```java
  package com.mashibing.rabbitmqboot;
  
  import com.mashibing.rabbitmqboot.config.DeadLetterConfig;
  import com.rabbitmq.client.Channel;
  import org.springframework.amqp.core.Message;
  import org.springframework.amqp.rabbit.annotation.RabbitListener;
  import org.springframework.stereotype.Component;
  
  import java.io.IOException;
  
  /**
   * @author tianming
   * @description
   * @date 2022/2/10 15:17
   */
  @Component
  public class DeadListener {
  
      @RabbitListener(queues = DeadLetterConfig.NORMAL_QUEUE)
      public void consume(String msg, Channel channel, Message message) throws IOException {
          System.out.println("接收到normal队列的消息：" + msg);
          channel.basicReject(message.getMessageProperties().getDeliveryTag(),false);
          channel.basicNack(message.getMessageProperties().getDeliveryTag(),false,false);
      }
  }
  ```

- 消息的生存时间

  - 给消息设置生存时间

    ```java
    @Test
    public void publishExpire(){
        String msg = "dead letter expire";
        rabbitTemplate.convertAndSend(DeadLetterConfig.NORMAL_EXCHANGE, "normal.abc", msg, new MessagePostProcessor() {
            @Override
            public Message postProcessMessage(Message message) throws AmqpException {
                message.getMessageProperties().setExpiration("5000");
                return message;
            }
        });
    }
    ```

  - 给队列设置消息的生存时间

    ```java
    @Bean
    public Queue normalQueue(){
        return QueueBuilder.durable(NORMAL_QUEUE)
                .deadLetterExchange(DEAD_EXCHANGE)
                .deadLetterRoutingKey("dead.abc")
                .ttl(10000)
                .build();
    }
    ```

- 设置Queue中的消息最大长度

  ```java
  @Bean
  public Queue normalQueue(){
      return QueueBuilder.durable(NORMAL_QUEUE)
              .deadLetterExchange(DEAD_EXCHANGE)
              .deadLetterRoutingKey("dead.abc")
              .maxLength(1)
              .build();
  }
  ```

  只要Queue中已经有一个消息，如果再次发送一个消息，这个消息会变为死信！

#### 7.3 延迟交换机

需要延迟交换机插件

下载地址：https://github.com/rabbitmq/rabbitmq-delayed-message-exchange/releases/tag/3.8.9

死信队列实现延迟消费时，如果延迟时间比较复杂，比较多，直接使用死信队列时，需要创建大量的队列还对应不同的时间，可以采用延迟交换机来解决这个问题。

- 构建延迟交换机

  ```java
  package com.mashibing.rabbitmqboot.config;
  
  import org.springframework.amqp.core.*;
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;
  
  import java.util.HashMap;
  import java.util.Map;
  
  /**
   * @author tianming
   * @description
   */
  @Configuration
  public class DelayedConfig {
  
      public static final String DELAYED_EXCHANGE = "delayed-exchange";
      public static final String DELAYED_QUEUE = "delayed-queue";
      public static final String DELAYED_ROUTING_KEY = "delayed.#";
  
      @Bean
      public Exchange delayedExchange(){
          Map<String, Object> arguments = new HashMap<>();
          arguments.put("x-delayed-type","topic");
          Exchange exchange = new CustomExchange(DELAYED_EXCHANGE,"x-delayed-message",true,false,arguments);
          return exchange;
      }
  
      @Bean
      public Queue delayedQueue(){
          return QueueBuilder.durable(DELAYED_QUEUE).build();
      }
  
      @Bean
      public Binding delayedBinding(Queue delayedQueue,Exchange delayedExchange){
          return BindingBuilder.bind(delayedQueue).to(delayedExchange).with(DELAYED_ROUTING_KEY).noargs();
      }
  }
  ```

- 发送消息

  ```java
  package com.mashibing.rabbitmqboot;
  
  import com.mashibing.rabbitmqboot.config.DelayedConfig;
  import org.junit.jupiter.api.Test;
  import org.springframework.amqp.AmqpException;
  import org.springframework.amqp.core.Message;
  import org.springframework.amqp.core.MessagePostProcessor;
  import org.springframework.amqp.rabbit.core.RabbitTemplate;
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.boot.test.context.SpringBootTest;
  
  /**
   * @author tianming
   * @description
   */
  @SpringBootTest
  public class DelayedPublisherTest {
  
      @Autowired
      private RabbitTemplate rabbitTemplate;
  
      @Test
      public void publish(){
          rabbitTemplate.convertAndSend(DelayedConfig.DELAYED_EXCHANGE, "delayed.abc", "xxxx", new MessagePostProcessor() {
              @Override
              public Message postProcessMessage(Message message) throws AmqpException {
                  message.getMessageProperties().setDelay(30000);
                  return message;
              }
          });
      }
  }
  ```

#### 延迟交换机的缺点

延迟交换机主要是变更了消息存储的维度到交换机，加入现在我们的消息正在交换机中存储，但是还未路由到[延迟队列](https://so.csdn.net/so/search?q=延迟队列&spm=1001.2101.3001.7020)，一旦服务宕机，延迟交换机中存储的消息直接就丢失了。这种适用于对于数据少量丢失容忍性比较强的业务场景。

### 八、RabbitMQ的集群

RabbitMQ的镜像模式

|                        RabbitMQ的集群                        |
| :----------------------------------------------------------: |
| ![1644926959251.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/2746/1642502600000/68e14c9292a84e528801bf78ba8102d7.png) |

高可用

提升RabbitMQ的效率

**搭建RabbitMQ集群**

- 准备两台虚拟机（克隆）

- 准备RabbitMQ的yml文件

  rabbitmq1：

  ```yml
  version: '3.1'
  services:
    rabbitmq1:
      image: rabbitmq:3.8.5-management-alpine
      container_name: rabbitmq1
      hostname: rabbitmq1
      extra_hosts:
        - "rabbitmq1:192.168.11.32"
        - "rabbitmq2:192.168.11.33"
      environment: 
        - RABBITMQ_ERLANG_COOKIE=SDJHFGDFFS
      ports:
        - 5672:5672
        - 15672:15672
        - 4369:4369
        - 25672:25672
  ```

  rabbitmq2：

  ```yml
  version: '3.1'
  services:
    rabbitmq2:
      image: rabbitmq:3.8.5-management-alpine
      container_name: rabbitmq2
      hostname: rabbitmq2
      extra_hosts:
        - "rabbitmq1:192.168.11.32"
        - "rabbitmq2:192.168.11.33"
      environment: 
        - RABBITMQ_ERLANG_COOKIE=SDJHFGDFFS
      ports:
        - 5672:5672
        - 15672:15672
        - 4369:4369
        - 25672:25672
  ```

  准备完毕之后，启动两台RabbitMQ

  |                           启动效果                           |
  | :----------------------------------------------------------: |
  | ![1644924815935.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/2746/1642502600000/f047925205a9446fa6822945dab02eb5.png) |

- 让RabbitMQ服务实现join操作

  需要四个命令完成join操作

  让rabbitmq2   join  rabbitmq1，需要进入到rabbitmq2的容器内部，去执行下述命令

  ```sh
  rabbitmqctl stop_app
  rabbitmqctl reset 
  rabbitmqctl join_cluster rabbit@rabbitmq1
  rabbitmqctl start_app
  ```

  执行成功后：

  |                          执行成功后                          |
  | :----------------------------------------------------------: |
  | ![1644925359203.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/2746/1642502600000/d980f871d229468d925ec556bb72fc43.png) |

- 设置镜像模式

  在指定的RabbitMQ服务中设置好镜像策略即可

  |                           镜像模式                           |
  | :----------------------------------------------------------: |
  | ![1644925812667.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/2746/1642502600000/4157a49c99e6426fa24a618d57f48e67.png) |

### 九、RabbitMQ其他内容

#### 9.1 Headers类型Exchange

headers就是一个基于key-value的方式，让Exchange和Queue绑定的到一起的一种规则

相比Topic形式，可以采用的类型更丰富。

|                       headers绑定方式                        |
| :----------------------------------------------------------: |
| ![1645705080465.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/2746/1642502600000/530466d70c4f44a586158986c8dd713b.png) |

具体实现方式

```java
package com.mashibing.headers;

import com.mashibing.util.RabbitMQConnectionUtil;
import com.rabbitmq.client.AMQP;
import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import org.junit.Test;

import java.util.HashMap;
import java.util.Map;

/**
 * @author tianming
 * @description
 */
public class Publisher {

    public static final String HEADER_EXCHANGE = "header_exchange";
    public static final String HEADER_QUEUE = "header_queue";


    @Test
    public void publish()throws  Exception{
        //1. 获取连接对象
        Connection connection = RabbitMQConnectionUtil.getConnection();

        //2. 构建Channel
        Channel channel = connection.createChannel();


        //3. 构建交换机和队列并基于header的方式绑定
        channel.exchangeDeclare(HEADER_EXCHANGE, BuiltinExchangeType.HEADERS);
        channel.queueDeclare(HEADER_QUEUE,true,false,false,null);
        Map<String,Object> args = new HashMap<>();
        // 多个header的key-value只要可以匹配上一个就可以
        // args.put("x-match","any");
        // 多个header的key-value要求全部匹配上！
        args.put("x-match","all");
        args.put("name","jack");
        args.put("age","23");
        channel.queueBind(HEADER_QUEUE,HEADER_EXCHANGE,"",args);

        //4. 发送消息
        String msg = "header测试消息！";
        Map<String, Object> headers = new HashMap<>();
        headers.put("name","jac");
        headers.put("age","2");
        AMQP.BasicProperties props = new AMQP.BasicProperties()
                .builder()
                .headers(headers)
                .build();

        channel.basicPublish(HEADER_EXCHANGE,"",props,msg.getBytes());

        System.out.println("发送消息成功，header = " + headers);

    }
}
```

### 十、RabbitMQ实战

在掌握了SpringCloudAlibaba的应用后，再来玩！！

为了更好的理解RabbitMQ在项目中的作用，来一套实战操作。

#### 10.1 RabbitMQ实战场景

首先模拟一个场景，电商中对应的处理方案。

模拟一个用户在电商平台下单：

* 需要调用库存服务，扣除商品库存，扣除成功后，才可以继续往下走业务
* 需要调用订单服务，创建订单（待支付）。
* 还需要很多后续的处理
  * 下单时，会使用优惠券，预扣除当前用户使用的优惠券
  * 下单时，会使用用户积分顶金额，预扣除当前用户的积分
  * 创建成功后，需要通知商家，有用户下单。
  * ………………

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/2746/1679467956009/5c753b85377d4d6988da03fcc162db50.png)

#### 10.2 RabbitMQ实战场景搭建

因为场景设计到了服务之间的调用。

这里需要大家提前掌握一些知识：Nacos，OpenFeign的应用层面。

##### 1、构建聚合工程，作为父工程管理所有的模块

准备好pom.xml文件

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.3.12.RELEASE</version>
    <relativePath />
</parent>

<groupId>com.mashibing</groupId>
<artifactId>rabbitmq</artifactId>
<version>1.0-SNAPSHOT</version>
<packaging>pom</packaging>

<properties>
    <spring.cloud-version>Hoxton.SR12</spring.cloud-version>
    <spring.cloud.alibaba-version>2.2.7.RELEASE</spring.cloud.alibaba-version>
</properties>
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring.cloud-version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <version>${spring.cloud.alibaba-version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

##### 2、构建其他六个子服务

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/2746/1679467956009/78528cfe98d84075a8dab81dbcbfff83.png)

##### 3、从下单服务开始一次完成配置以及接口的提供

下单服务：

* 导入依赖

  ```
  <dependencies>
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-web</artifactId>
      </dependency>
      <dependency>
          <groupId>com.alibaba.cloud</groupId>
          <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
      </dependency>
      <dependency>
          <groupId>org.springframework.cloud</groupId>
          <artifactId>spring-cloud-starter-openfeign</artifactId>
      </dependency>
  </dependencies>
  ```

* 构建启动类

  ```
  @SpringBootApplication
  @EnableDiscoveryClient
  public class PlaceOrderStarterApp {
  
      public static void main(String[] args) {
          SpringApplication.run(PlaceOrderStarterApp.class,args);
      }
  }
  ```

* 编写配置文件

  ```
  server:
    port: 80
  
  spring:
    application:
      name: placeorder
    cloud:
      nacos:
        discovery:
          server-addr: 114.116.226.76:8848
  ```

* 处理问题：启动后发现，无法正常的注册到Nacos上，需要将Alibaba的版本降到2.2.6.RELEASE

其他服务的基本配置，我这里直接写好，然后大家可以去Git中找到指定提交点~

##### 4、完成整个下单的流程

* 下单服务接口（前置操作）

  ```
  @RestController
  public class PlaceOrderController {
  
      /**
       * 模拟用户下单操作
       * @return
       */
      @GetMapping("/po")
      public String po(){
          //1、调用库存服务扣除商品库存
  
          //2、调用订单服务，创建订单
  
          //3、调用优惠券服务，预扣除使用的优惠券
  
          //4、调用用户积分服务，预扣除用户使用的积分
  
          //5、调用商家服务，通知商家用户已下单
  
          return "place order is ok!";
      }
  
  }
  ```

* 库存服务接口

  ```
  @RestController
  public class ItemStockController {
  
      private static int stock = 10;
  
      @GetMapping("/decr")
      public void decr() throws InterruptedException {
          Thread.sleep(400);
          stock--;
          if(stock < 0){
              throw new RuntimeException("商品库存不足！");
          }
          System.out.println("扣减库存成功！");
      }
  }
  ```

* 订单服务接口

  ```
  @RestController
  public class OrderManageController {
  
      @GetMapping("create")
      public void create() throws InterruptedException {
          Thread.sleep(400);
          System.out.println("创建订单成功！");
      }
  
  }
  ```

* 优惠券服务接口

  ```
  @RestController
  public class CouponController {
  
      @GetMapping("/coupon")
      public void coupon() throws InterruptedException {
          Thread.sleep(400);
          System.out.println("优惠券预扣除成功！");
      }
  
  }
  ```

* 用户积分服务接口

  ```
  @RestController
  public class UserPointsController {
  
      @GetMapping("/up")
      public void up() throws InterruptedException {
          Thread.sleep(400);
          System.out.println("扣除用户积分成功！！");
      }
  
  }
  ```

* 商家服务接口

  ```
  @RestController
  public class BusinessController {
  
      @GetMapping("/notify")
      public void notifyBusiness() throws InterruptedException {
          Thread.sleep(400);
          System.out.println("通知商家成功！！");
      }
  
  }
  ```

##### 5、完善下单接口服务调用

* 先给启动类添加OpenFeign注解

  ```
  @EnableFeignClients
  ```

* 给5个服务提供对应的OpenFeign接口

  ![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/2746/1679467956009/052ca84c000b4e71b79910c05bdfc736.png)

* 在下单服务的Controller中实现服务的调用

  ```
  @RestController
  public class PlaceOrderController {
  
      @Autowired
      private ItemStockClient itemStockClient;
      @Autowired
      private OrderManageClient orderManageClient;
      @Autowired
      private CouponClient couponClient;
      @Autowired
      private UserPointsClient userPointsClient;
      @Autowired
      private BusinessClient businessClient;
  ```


```java
  /**
   * 模拟用户下单操作
   * @return
   */
  @GetMapping("/po")
  public String po(){
      long start = System.currentTimeMillis();
      //1、调用库存服务扣除商品库存
      itemStockClient.decr();
      //2、调用订单服务，创建订单
      orderManageClient.create();
      //3、调用优惠券服务，预扣除使用的优惠券
      couponClient.coupon();
      //4、调用用户积分服务，预扣除用户使用的积分
      userPointsClient.up();
      //5、调用商家服务，通知商家用户已下单
      businessClient.notifyBusiness();

      long end = System.currentTimeMillis();
      System.out.println(end - start);
      return "place order is ok!";
  }
```

  }

  ```
#### 10.3 完成异步调用

因为下单功能，核心就在于扣除库存成功，以及创建订单成功。只要这两个操作么得问题，直接就可以让后续的优惠券，用户积分，通知商家等等操作实现一个异步的效果。而且基于RabbitMQ做异步之后，还可以让下单服务与其他服务做到解耦。

异步：可以让整个业务的处理速度更快，从而更快的给用户一个响应，下单是成功还是失败。

解耦：优惠券，用户积分，商家服务，无论哪个服务宕机，都不影响正常的下单流程。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/2746/1679467956009/e6ef540ee8c5483da662540e7ae82a79.png)

##### 1、下单服务

* 导入依赖
  ```

  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-amqp</artifactId>
  </dependency>

  ```
* 编写配置文件链接RabbitMQ
  ```

  spring:
    rabbitmq:
      host: 114.116.226.76
      port: 5672
      username: rabbitmq
      password: rabbitmq
      virtual-host: rabbitmq

  ```
* 构建交换机&队列
  ```

  @Configuration
  public class RabbitMQConfig {

      // 下单服务的交换机
      public static final String PLACE_ORDER_EXCHANGE = "place_order_exchange";
      // 三个服务的Queue
      public static final String COUPON_QUEUE = "coupon_queue";
      public static final String USER_POINTS_QUEUE = "user_points_queue";
      public static final String BUSINESS_QUEUE = "business_queue";


      @Bean
      public Exchange placeOrderExchange(){
          return ExchangeBuilder.fanoutExchange(PLACE_ORDER_EXCHANGE).build();
      }
    
      @Bean
      public Queue couponQueue(){
          return QueueBuilder.durable(COUPON_QUEUE).build();
      }
      @Bean
      public Queue userPointsQueue(){
          return QueueBuilder.durable(USER_POINTS_QUEUE).build();
      }
      @Bean
      public Queue businessQueue(){
          return QueueBuilder.durable(BUSINESS_QUEUE).build();
      }
    
      @Bean
      public Binding couponBinding(Exchange placeOrderExchange,Queue couponQueue){
          return BindingBuilder.bind(couponQueue).to(placeOrderExchange).with("").noargs();
      }
      @Bean
      public Binding userPointsBinding(Exchange placeOrderExchange,Queue userPointsQueue){
          return BindingBuilder.bind(userPointsQueue).to(placeOrderExchange).with("").noargs();
      }
      @Bean
      public Binding businessBinding(Exchange placeOrderExchange,Queue businessQueue){
          return BindingBuilder.bind(businessQueue).to(placeOrderExchange).with("").noargs();
      }

  }

  ```
* 修改下单接口Controller
  ```

  @RestController
  public class PlaceOrderController {

      @Autowired
      private ItemStockClient itemStockClient;
      @Autowired
      private OrderManageClient orderManageClient;
      @Autowired
      private RabbitTemplate rabbitTemplate;
      /**
       * 模拟用户下单操作
       * @return
       */
      @GetMapping("/po")
      public String po(){
          long start = System.currentTimeMillis();
          //1、调用库存服务扣除商品库存
          itemStockClient.decr();
          //2、调用订单服务，创建订单
          orderManageClient.create();
    
          String userAndOrderInfo = "用户信息&订单信息&优惠券信息等等…………";
          // 将同步方式修改为基于RabbitMQ的异步方式
          rabbitTemplate.convertAndSend(RabbitMQConfig.PLACE_ORDER_EXCHANGE,"",userAndOrderInfo);
    
          long end = System.currentTimeMillis();
          System.out.println(end - start);
          return "place order is ok!";
      }

  }

  ```
##### 2、优惠券服务

* 导入依赖
  ```

  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-amqp</artifactId>
  </dependency>

  ```
* COPY配置文件
  ```

  spring:
    rabbitmq:
      host: 114.116.226.76
      port: 5672
      username: rabbitmq
      password: rabbitmq
      virtual-host: rabbitmq
      listener:
        simple:
          acknowledge-mode: manual

  ```
* COPY配置类： **复制的下单服务的RabbitMQConfig**
* 编写消费者，实现预扣除优惠券
  ```

  @Component
  public class CouponListener {

      @RabbitListener(queues = {RabbitMQConfig.COUPON_QUEUE})
      public void consume(String msg, Channel channel, Message message) throws Exception {
          // 预扣除优惠券
          Thread.sleep(400);
          System.out.println("优惠券预扣除成功！" + msg);
          // 手动ACK
          channel.basicAck(message.getMessageProperties().getDeliveryTag(),false);
      }

  }

  ```
##### 3、用户积分服务

类似优惠券服务操作！

##### 4、商家服务

类似优惠券服务操作！

#### 10.4 下单服务保证消息的可靠性

下单服务需要保证消息一定可以发送到RabbitMQ服务中，如果发送失败。

如果消息没有发送到Exchange或者是消息没有从Exchange路由到指定队列。

* 可以将消息存储到数据库，基于定时任务的方式重新发送。
* 可以直接在confirm中做重试。
* 或者是记录error日志，通过日志的形式做重新发送。
* …………

开始完成当前操作

##### 1、修改配置文件

​```yml
spring:
  rabbitmq:
    publisher-confirm-type: correlated
    publisher-returns: true
  ```

##### 2、重新配置RabbitTemplate对象，指定confirm和return的回调处理

```java
@Configuration
public class RabbitTemplateConfig {

    @Bean
    public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory){
        //1、new出RabbitTemplate对象
        RabbitTemplate rabbitTemplate = new RabbitTemplate();
        //2、将connectionFactory设置到RabbitTemplate对象中
        rabbitTemplate.setConnectionFactory(connectionFactory);
        //3、设置confirm回调
        rabbitTemplate.setConfirmCallback(confirmCallback());
        //4、设置return回调
        rabbitTemplate.setReturnCallback(returnCallback());
        //5、设置mandatory为true
        rabbitTemplate.setMandatory(true);
        //6、返回RabbitTemplate对象即可
        return rabbitTemplate;
    }

    public RabbitTemplate.ConfirmCallback confirmCallback(){
        return new RabbitTemplate.ConfirmCallback(){
            @Override
            public void confirm(CorrelationData correlationData, boolean ack, String cause) {
                if (correlationData == null) return;
                String msgId = correlationData.getId();
                if(ack){
                    System.out.println("消息发送到Exchange成功!! msgId = " + msgId);
                }else{
                    System.out.println("消息发送到Exchange失败!! msgId = " + msgId);
                }
            }
        };
    }

    public RabbitTemplate.ReturnCallback returnCallback(){
        return new RabbitTemplate.ReturnCallback(){
            @Override
            public void returnedMessage(Message message, int replyCode, String replyText, String exchange, String routingKey) {
                System.out.println("消息未路由到队列");
                System.out.println("return：消息为：" + new String(message.getBody()));
                System.out.println("return：交换机为：" + exchange);
                System.out.println("return：路由为：" + routingKey);
            }
        };
    }

}
```

##### 3、重新完成Controller中消息的发送并且完善confirm和return的回调

###### 3.1、需要在Controller中将correlationData和发送的消息信息绑定

准备全局的Cache

```java
public class GlobalCache {

    private static Map map = new HashMap();

    public static void set(String key,Object value){
        map.put(key,value);
    }

    public static Object get(String key){
        Object value = map.get(key);
        return value;
    }

    public static void remove(String key){
        map.remove(key);
    }

}
```

重新编写Controller，实现标识和消息信息的绑定

```java
@RestController
public class PlaceOrderController {

    @Autowired
    private ItemStockClient itemStockClient;
    @Autowired
    private OrderManageClient orderManageClient;

    @Autowired
    private RabbitTemplate rabbitTemplate;
    /**
     * 模拟用户下单操作
     * @return
     */
    @GetMapping("/po")
    public String po(){
        long start = System.currentTimeMillis();
        //1、调用库存服务扣除商品库存
        itemStockClient.decr();
        //2、调用订单服务，创建订单
        orderManageClient.create();

        // 将之前的同步方式注释
        String userAndOrderInfo = "用户信息&订单信息&优惠券信息等等…………";
        // 声明当前消息的id标识
        String id = UUID.randomUUID().toString();
        // 封装消息的全部信息
        Map map = new HashMap<>();
        map.put("message",userAndOrderInfo);
        map.put("exchange",RabbitMQConfig.PLACE_ORDER_EXCHANGE);
        map.put("routingKey","");
        map.put("sendTime",new Date());
        // 将id标识和消息存储到全局缓存中
        GlobalCache.set(id,map);
        // 将同步方式修改为基于RabbitMQ的异步方式
        rabbitTemplate.convertAndSend(RabbitMQConfig.PLACE_ORDER_EXCHANGE,"",userAndOrderInfo,new CorrelationData(id));


        long end = System.currentTimeMillis();
        System.out.println(end - start);
        return "place order is ok!";
    }

}
```

###### 3.2、需要在confirm的回调中完成两个操作

* 消息发送成功，删除之前绑定的消息

  ```
  if(ack){
      log.info("消息发送到Exchange成功!!");
      GlobalCache.remove(msgId);
  }
  ```

* 消息发送失败，将之前绑定的消息存储到数据库

  * 准备库表信息，存储发送失败的信息。

    ```
    CREATE TABLE `resend` (
      `id` varchar(255) NOT NULL,
      `message` varchar(255) NOT NULL,
      `exchange` varchar(255) NOT NULL,
      `routing_key` varchar(255) NOT NULL,
      `send_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
      `send_count` int(11) NOT NULL DEFAULT '0' COMMENT '最多重新发送3次',
      `is_send` int(11) NOT NULL DEFAULT '0' COMMENT '0-发送失败，1-发送成功',
      PRIMARY KEY (`id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
    ```

  * 实现数据源和MyBatis的基本配置：……

  * 改造confirm实现

    ```
    public RabbitTemplate.ConfirmCallback confirmCallback(){
        return new RabbitTemplate.ConfirmCallback(){
            @Override
            public void confirm(CorrelationData correlationData, boolean ack, String cause) {
                if (correlationData == null) return;
                String msgId = correlationData.getId();
                if(ack){
                    log.info("消息发送到Exchange成功!!");
                    GlobalCache.remove(msgId);
                }else{
                    log.error("消息发送失败！");
                    Map value = (Map) GlobalCache.get(msgId);
                    // 推荐自己玩的时候，用service做增删改操作，控制事务~
                    resendMapper.save(value);
                }
            }
        };
    }
    ```

##### 4、测试效果![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/2746/1679467956009/de817b1acb10417089f467fc65fb2d30.png)

#### 10.5 消费者避免重复消费问题

采用数据库的幂等表解决消费者可能存在重复消费的问题。

再真正处理消费执行业务前做一些操作，先去查看数据库中的幂等表信息：

* 如果消息的唯一标识已经存在了，证明当前消息已经被消费，直接告辞。
* 如果消息的唯一标识不存在，先将当前唯一标识存储到幂等表中，然后再执行消费业务。

基于用户积分服务实现幂等性操作。

##### 1、准备幂等表

```sql
CREATE TABLE `user_points_idempotent` (
  `id` varchar(255) NOT NULL,
  `createtime` timestamp NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

##### 2、给用户积分服务追加连接数据库信息：

* 导入依赖

* 编写配置

* 准备Mapper接口

  ```
  public interface UserPointsIdempotentMapper {
  
      @Select("select count(1) from user_points_idempotent where id = #{id}")
      int findById(@Param("id") String id);
  
      @Insert("insert into user_points_idempotent (id) values (#{id})")
      void save(@Param("id") String id);
  
  }
  ```

##### 3、准备消费方法

```java
@Service
@Slf4j
public class UserPointsConsumeImpl implements UserPointsConsume {

    @Resource
    private UserPointsIdempotentMapper userPointsIdempotentMapper;

    private final String ID_NAME = "spring_returned_message_correlation";


    @Override
    @Transactional
    public void consume(Message message) {
        // 获取生产者提供的CorrelationId要基于header去获取。
        String id = message.getMessageProperties().getHeader(ID_NAME);
        //1、查询幂等表是否存在当前消息标识
        int count = userPointsIdempotentMapper.findById(id);
        //2、如果存在，直接return结束
        if(count == 1){
            log.info("消息已经被消费！！！无需重复消费！");
            return;
        }
        //3、如果不存在，插入消息标识到幂等表
        userPointsIdempotentMapper.save(id);
        //4、执行消费逻辑
        // 预扣除用户积分
        try {
            Thread.sleep(400);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("扣除用户积分成功！！");
    }
}
```

##### 4、测试功能效果

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/2746/1679467956009/a0063f41ed334d5fbbe85c3ffed6f7e2.png)

#### 10.6 实现延迟取消订单状态

当客户端下单之后，会基于订单服务在数据库中构建一个订单信息，默认情况下，订单信息是待支付状态。

如果用户正常支付了，会将当前订单从待支付状态改为已支付/待发货状态。

如果超过一定的时间，用户没有支付，此时需要将订单状态从待支付改为已取消的状态。

基于RabbitMQ提供的死信队列来实现当前的延迟修改订单状态的功能，同时也可以采用延迟交换机插件的形式实现，But，因为当前业务中，延迟时间是统一的，不使用延迟交换机也是ok的。

##### 1、准备订单表并修改订单服务的业务

* 准备表结构

  ```
  CREATE TABLE `tb_order` (
    `id` varchar(36) NOT NULL AUTO_INCREMENT,
    `total` decimal(10,2) DEFAULT NULL,
    `order_state` int(11) DEFAULT '0' COMMENT '订单状态  0-待支付， 1-已支付，2-待发货，3-已发货，-1-已取消',
    PRIMARY KEY (`id`)
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
  ```

* 修改订单服务，将之前模拟数据库操作，改为真实的数据库操作

  * 导入依赖

    ```
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.47</version>
    </dependency>
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>2.2.2</version>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>
    ```

  * 编写配置信息

    ```
    spring:
      datasource:
        driver-class-name: org.gjt.mm.mysql.Driver
        url: jdbc:mysql:///rabbitmq
        username: root
        password: root
    ```

  * 启动类添加注解

    ```
    @MapperScan(basePackages = "com.mashibing.mapper")
    ```

  * 实现添加操作

    * 准备Mapper接口

      ```
      public interface TBOrderMapper {
      
          @Insert("insert into tb_order (id) values (#{id})")
          void save(@Param("id") String id);
      
      }
      ```

    * 准备Service层

      ```
      @Service
      public class TBOrderServiceImpl implements TBOrderService {
      
          @Resource
          private TBOrderMapper orderMapper;
      ```


          @Override
          public void save() {
      	String id = UUID.randomUUID().toString();
              orderMapper.save(id);
          }
      }
      ```
    * Controller调用Service层
      ```
      @RestController
      @Slf4j
      public class OrderManageController {
    
          @Autowired
          private TBOrderService orderService;
    
          @GetMapping("create")
          public void create() throws InterruptedException {
              orderService.save();
              log.info("创建订单成功！！");
          }
    
      }


##### 2、在订单服务中准备死信队列配置

* 导入依赖

  ```
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-amqp</artifactId>
  </dependency>
  ```

* 编写配置文件

  ```
  spring:
    rabbitmq:
      host: 114.116.226.76
      port: 5672
      username: rabbitmq
      password: rabbitmq
      virtual-host: rabbitmq
      listener:
        simple:
          acknowledge-mode: manual
  ```

* 编写配置类完成死信队列的构建

  ```
  @Configuration
  public class RabbitMQConfig {
  
      public static final String ORDER_EXCHANGE = "order_exchange";
      public static final String ORDER_QUEUE = "order_queue";
  
      public static final String DEAD_EXCHANGE = "dead_exchange";
      public static final String DEAD_QUEUE = "dead_queue";
  
      @Bean
      public Exchange orderExchange(){
          return ExchangeBuilder.fanoutExchange(ORDER_EXCHANGE).build();
      }
  
      @Bean
      public Queue orderQueue(){
          return QueueBuilder.durable(ORDER_QUEUE).deadLetterExchange(DEAD_EXCHANGE).build();
      }
  
      @Bean
      public Exchange deadExchange(){
          return ExchangeBuilder.fanoutExchange(DEAD_EXCHANGE).build();
      }
  
      @Bean
      public Queue deadQueue(){
          return QueueBuilder.durable(DEAD_QUEUE).build();
      }
  
      @Bean
      public Binding orderBinding(Exchange orderExchange,Queue orderQueue){
          return BindingBuilder.bind(orderQueue).to(orderExchange).with("").noargs();
      }
  
      @Bean
      public Binding deadBinding(Exchange deadExchange,Queue deadQueue){
          return BindingBuilder.bind(deadQueue).to(deadExchange).with("").noargs();
      }
  
  }
  ```

##### 3、完成订单构建成功后，发送消息到死信队列

前面的准备工作，没考虑到订单的主键需要作为消息的问题，将之前的主键自增的形式，更改为UUID作为主键，方便作为消息传递。

处理了两个问题：

* 订单表的主键，为了方便作为消息，将之前主键自增的ID，设置为了自然主键，用UUID。
* 发送消息后，发现队列没有收到消息，定位到是忘记在配置文件追加Binding信息。

完成消息发送

```java
@Service
public class TBOrderServiceImpl implements TBOrderService {

    @Resource
    private TBOrderMapper orderMapper;

    @Autowired
    private RabbitTemplate rabbitTemplate;


    @Override
    @Transactional
    public void save() {
        // 生成主键ID
        String id = UUID.randomUUID().toString();
        // 创建订单
        orderMapper.save(id);
        // 订单构建成功~
        // 发送消息到RabbitMQ的死信队列
        rabbitTemplate.convertAndSend(RabbitMQConfig.ORDER_EXCHANGE, "", id, new MessagePostProcessor() {
            @Override
            public Message postProcessMessage(Message message) throws AmqpException {
                // 设置消息的生存时间为15s，当然，也可以在构建队列时，指定队列的生存时间。
                message.getMessageProperties().setExpiration("15000");
                return message;
            }
        });
    }
}
```

##### 4、声明消费者消费延迟取消订单的消息

* 声明消费者：

  ```
  @Component
  public class DelayMessageListener {
  
      @Autowired
      private TBOrderService orderService;
  
      @RabbitListener(queues = RabbitMQConfig.DEAD_QUEUE)
      public void consume(String id, Channel channel, Message message) throws IOException {
          //1、 调用Service实现订单状态的处理
          orderService.delayCancelOrder(id);
  
          //2、 ack的干活~
          channel.basicAck(message.getMessageProperties().getDeliveryTag(),false);
      }
  
  }
  ```

* 完善Service业务处理

  ```
  @Override
  @Transactional
  public void delayCancelOrder(String id) {
      //1、基于id查询订单信息。 for update
      int orderState = orderMapper.findOrderStateByIdForUpdate(id);
      //2、判断订单状态
      if(orderState != 0){
          log.info("订单已经支付！！");
          return;
      }
      //3、修改订单状态
      log.info("订单未支付，修改订单状态为已取消");
      orderMapper.updateOrderStateById(-1,id);
  }
  ```

* 提供Mapper与数据库交互的业务

  ```
  public interface TBOrderMapper {
  
      @Select("select order_state from tb_order where id = #{id} for update")
      int findOrderStateByIdForUpdate(@Param("id") String id);
  
      @Update("update tb_order set order_state = #{orderState} where id = #{id}")
      void updateOrderStateById(@Param("orderState") int i, @Param("id") String id);
  }
  ```



# Kafka

### 消息中间件的编年史

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670340547050/ff0e0903ad2d4c8da59a57d921bd80d8.png)

### Kafka的外在表现和内在设计

kafka最初是LinkedIn的一个内部基础设施系统。最初开发的起因是，LinkedIn虽然有了数据库和其他系统可以用来存储数据，但是缺乏一个可以帮助处理持续数据流的组件。

所以在设计理念上，开发者不想只是开发一个能够存储数据的系统，如关系数据库、Nosql数据库、搜索引擎等等，更希望把数据看成一个持续变化和不断增长的流，并基于这样的想法构建出一个数据系统，一个数据架构。

Kafka外在表现很像消息系统，允许发布和订阅消息流，但是它和传统的消息系统有很大的差异：

1、Kafka是个现代分布式系统，以集群的方式运行，可以自由伸缩。

2、Kafka可以按照要求存储数据，保存多久都可以，

3、流式处理将数据处理的层次提示到了新高度，消息系统只会传递数据，Kafka的流式处理能力可以让我们用很少的代码就能动态地处理派生流和数据集。所以Kafka不仅仅是个消息中间件。

Kafka不仅仅是一个消息中间件，同时它是一个流平台，这个平台上可以发布和订阅数据流（Kafka的流，有一个单独的包Stream的处理），并把他们保存起来，进行处理，这个是Kafka作者的设计理念。

大数据领域，Kafka还可以看成实时版的Hadoop，但是还是有些区别，Hadoop可以存储和定期处理大量的数据文件，往往以TB计数，而Kafka可以存储和持续处理大型的数据流。Hadoop主要用在数据分析上，而Kafka因为低延迟，更适合于核心的业务应用上。

Kafka名字的由来：卡夫卡与法国作家马塞尔·普鲁斯特，爱尔兰作家詹姆斯·乔伊斯并称为西方现代主义文学的先驱和大师。《变形记》是卡夫卡的短篇代表作，是卡夫卡的艺术成就中的一座高峰，被认为是20世纪最伟大的小说作品之一（达到管理层的高度同学可以多看下人文相关的书籍，增长管理知识和人格魅力）。

本次课程，将会以kafka_2.13-3.3.1版本做主讲（这是讲课时的最新版本）。

### 市场主流消息中间件对比

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670340547050/e73c107aab80440cbbe364c70eb52f9e.png)

## Kafka中的基本概念

### 消息和批次

**消息** ，Kafka里的数据单元，也就是我们一般消息中间件里的消息的概念（可以比作数据库中一条记录）。消息由字节数组组成。消息还可以包含键（可选元数据，也是字节数组），主要用于对消息选取分区。

作为一个高效的消息系统，为了提高效率，消息可以被分批写入Kafka。**批次**就是一组消息，这些消息属于同一个主题和分区。如果只传递单个消息，会导致大量的网络开销，把消息分成批次传输可以减少这开销。但是，这个需要权衡（时间延迟和吞吐量之间），批次里包含的消息越多，单位时间内处理的消息就越多，单个消息的传输时间就越长（吞吐量高延时也高）。如果进行压缩，可以提升数据的传输和存储能力，但需要更多的计算处理。

对于Kafka来说，消息是晦涩难懂的字节数组，一般我们使用序列化和反序列化技术，格式常用的有JSON和XML，还有Avro（Hadoop开发的一款序列化框架），具体怎么使用依据自身的业务来定。

### 主题和分区

Kafka里的消息用**主题**进行分类（主题好比数据库中的表），主题下有可以被分为若干个 **分区（分表技术）** 。分区本质上是个提交日志文件，有新消息，这个消息就会以追加的方式写入分区（写文件的形式），然后用先入先出的顺序读取。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670340547050/6340e6f8eba544a3a75c7656ad5ae3b7.png)

但是因为主题会有多个分区，所以在整个主题的范围内，是无法保证消息的顺序的，单个分区则可以保证。

Kafka通过分区来实现数据冗余和伸缩性，因为分区可以分布在不同的服务器上，那就是说一个主题可以跨越多个服务器（这是Kafka高性能的一个原因，多台服务器的磁盘读写性能比单台更高）。

前面我们说Kafka可以看成一个流平台，很多时候，我们会把一个主题的数据看成一个流，不管有多少个分区。

### 生产者和消费者、偏移量、消费者群组

就是一般消息中间件里生产者和消费者的概念。一些其他的高级客户端API，像数据管道API和流式处理的Kafka Stream，都是使用了最基本的生产者和消费者作为内部组件，然后提供了高级功能。

生产者默认情况下把消息均衡分布到主题的所有分区上，如果需要指定分区，则需要使用消息里的消息键和分区器。

消费者订阅主题，一个或者多个，并且按照消息的生成顺序读取。消费者通过检查所谓的偏移量来区分消息是否读取过。偏移量是一种元数据，一个不断递增的整数值，创建消息的时候，Kafka会把他加入消息。在一个主题中一个分区里，每个消息的偏移量是唯一的。每个分区最后读取的消息偏移量会保存到Zookeeper或者Kafka上，这样分区的消费者关闭或者重启，读取状态都不会丢失。

多个消费者可以构成一个消费者群组。怎么构成？共同读取一个主题的消费者们，就形成了一个群组。群组可以保证每个分区只被一个消费者使用。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670340547050/3a17a68b35024f62a1618b775bfe3065.png)

消费者和分区之间的这种映射关系叫做消费者对分区的所有权关系，很明显，一个分区只有一个消费者，而一个消费者可以有多个分区。

（吃饭的故事：一桌一个分区，多桌多个分区，生产者不断生产消息(消费)，消费者就是买单的人，消费者群组就是一群买单的人），一个分区只能被消费者群组中的一个消费者消费（不能重复消费），如果有一个消费者挂掉了&#x3c;James跑路了>，另外的消费者接上）

### Broker和集群

一个独立的Kafka服务器叫Broker。

broker的主要工作是，接收生产者的消息，设置偏移量，提交消息到磁盘保存；为消费者提供服务，响应请求，返回消息。在合适的硬件上，单个broker可以处理上千个分区和每秒百万级的消息量。（要达到这个目的需要做操作系统调优和JVM调优）

多个broker可以组成一个集群。每个集群中broker会选举出一个集群控制器。控制器会进行管理，包括将分区分配给broker和监控broker。

集群里，一个分区从属于一个broker，这个broker被称为首领。但是分区可以被分配给多个broker，这个时候会发生分区复制。

集群中Kafka内部一般使用管道技术进行高效的复制。

![image-20240125203645012](E:\图灵课堂\MQ专题\MQ专题.assets\image-20240125203645012.png)

分区复制带来的好处是，提供了消息冗余。一旦首领broker失效，其他broker可以接管领导权。当然相关的消费者和生产者都要重新连接到新的首领上。（详细过程可见下面的选举部分）

### 保留消息

在一定期限内保留消息是Kafka的一个重要特性，Kafka  broker默认的保留策略是：要么保留一段时间（7天），要么保留一定大小（比如1个G）。到了限制，旧消息过期并删除。但是每个主题可以根据业务需求配置自己的保留策略（开发时要注意，Kafka不像Mysql之类的永久存储）。

# 为什么选择Kafka

## 优点

多生产者和多消费者

基于磁盘的数据存储，换句话说，Kafka的数据天生就是持久化的。

高伸缩性，Kafka一开始就被设计成一个具有灵活伸缩性的系统，对在线集群的伸缩丝毫不影响整体系统的可用性。

高性能，结合横向扩展生产者、消费者和broker，Kafka可以轻松处理巨大的信息流（LinkedIn公司每天处理万亿级数据），同时保证亚秒级的消息延迟。

## 常见场景

### 活动跟踪

跟踪网站用户和前端应用发生的交互，比如页面访问次数和点击，将这些信息作为消息发布到一个或者多个主题上，这样就可以根据这些数据为机器学习提供数据，更新搜素结果等等（头条、淘宝等总会推送你感兴趣的内容，其实在数据分析之前就已经做了活动跟踪）。

### 传递消息

标准消息中间件的功能

### 收集指标和日志

收集应用程序和系统的度量监控指标，或者收集应用日志信息，通过Kafka路由到专门的日志搜索系统，比如ES。（国内用得较多）

### 提交日志

收集其他系统的变动日志，比如数据库。可以把数据库的更新发布到Kafka上，应用通过监控事件流来接收数据库的实时更新，或者通过事件流将数据库的更新复制到远程系统。

还可以当其他系统发生了崩溃，通过重放日志来恢复系统的状态。（异地灾备）

### 流处理

操作实时数据流，进行统计、转换、复杂计算等等。随着大数据技术的不断发展和成熟，无论是传统企业还是互联网公司都已经不再满足于离线批处理，实时流处理的需求和重要性日益增长
。

近年来业界一直在探索实时流计算引擎和API，比如这几年火爆的Spark
Streaming、Kafka Streaming、Beam和Flink，其中阿里双11会场展示的实时销售金额，就用的是流计算，是基于Flink，然后阿里在其上定制化的Blink。

# Kafka的安装、管理和配置

## 安装

### 预备环境

Kafka是Java生态圈下的一员，用Scala编写，运行在Java虚拟机上，所以安装运行和普通的Java程序并没有什么区别。

安装Kafka官方说法，Java环境推荐Java8。

Kafka需要Zookeeper保存集群的元数据信息和消费者信息。Kafka一般会自带Zookeeper，但是从稳定性考虑，应该使用单独的Zookeeper，而且构建Zookeeper集群。

### 运行

#### Kafka with ZooKeeper

启动Zookeeper

进入Kafka目录下的bin\windows

执行kafka-server-start.bat ../../config/server.properties

Linux下与此类似，进入bin后，执行对应的sh文件即可

#### Kafka with KRaft

1、**生成集群id**

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670340547050/18a05dbdd55947768e9ce56479cb2a0b.png)

2、**格式化存储目录**

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670340547050/d65376a9befa494c92d23436faba58dd.png)

3、**启动服务**

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670340547050/0b449ddaef004c7495265bd1b7303da9.png)

启动正确后的界面如下：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670340547050/0c265d30b90d470e8f203a67940c8cd6.png)

### kafka基本的操作和管理

**##** **列出所有主题**

```
./kafka-topics.sh --bootstrap-server localhost:9092 --list
```

**##** **列出所有主题的详细信息**

```
./kafka-topics.sh --bootstrap-server localhost:9092 --describe
```

**##** ***创建主题主题名 my-topic ，1副本，8***分区

```
./kafka-topics.sh --bootstrap-server localhost:9092  --create --topic my-topic --replication-factor 1 --partitions 8
```

**##** **增加分区，注意：分区无法被删除**

```
./kafka-topics.sh --bootstrap-server localhost:9092 --alter --topic my-topic --partitions 16
```

**##** **创建生产者（控制台）**

```
./kafka-console-producer.sh --broker-list localhost:9092 --topic my-topic
```

**##** **创建消费者（控制台）**

```
./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic my-topic --from-beginning --consumer.config ../config/consumer.properties
```

**##** **kafka终止命令**

```
./kafka-server-stop.sh 
```

总结就是：

## Broker配置

配置文件放在Kafka目录下的config目录中，主要是server.properties文件

### 常规配置

#### broker.id

在单机时无需修改，但在集群下部署时往往需要修改。它是个每一个broker在集群中的唯一表示，要求是正数。当该服务器的IP地址发生改变时，broker.id没有变化，则不会影响consumers的消息情况

#### listeners

监听列表(以逗号分隔 不同的协议(如plaintext,trace,ssl、不同的IP和端口)),hostname如果设置为0.0.0.0则绑定所有的网卡地址；如果hostname为空则绑定默认的网卡。如果没有配置则默认为java.net.InetAddress.getCanonicalHostName()。

如：PLAINTEXT://myhost:9092,TRACE://:9091或 PLAINTEXT://0.0.0.0:9092,

#### zookeeper.connect

zookeeper集群的地址，可以是多个，多个之间用逗号分割。（一组hostname:port/path列表,hostname是zk的机器名或IP、port是zk的端口、/path是可选zk的路径，如果不指定，默认使用根路径）

#### log.dirs

Kafka把所有的消息都保存在磁盘上，存放这些数据的目录通过log.dirs指定。可以使用多路径，使用逗号分隔。如果是多路径，Kafka会根据“最少使用”原则，把同一个分区的日志片段保存到同一路径下。会往拥有最少数据分区的路径新增分区。

#### num.recovery.threads.per.data.dir

每数据目录用于日志恢复启动和关闭时的线程数量。因为这些线程只是服务器启动（正常启动和崩溃后重启）和关闭时会用到。所以完全可以设置大量的线程来达到并行操作的目的。注意，这个参数指的是每个日志目录的线程数，比如本参数设置为8，而log.dirs设置为了三个路径，则总共会启动24个线程。

#### auto.create.topics.enable

是否允许自动创建主题。如果设为true，那么produce（生产者往主题写消息），consume（消费者从主题读消息）或者fetch
metadata（任意客户端向主题发送元数据请求时）一个不存在的主题时，就会自动创建。缺省为true。

#### delete.topic.enable=true

删除主题配置，默认未开启

### 主题配置

新建主题的默认参数

#### num.partitions

每个新建主题的分区个数（分区个数只能增加，不能减少 ）。这个参数一般要评估，比如，每秒钟要写入和读取1000M数据，如果现在每个消费者每秒钟可以处理50MB的数据，那么需要20个分区，这样就可以让20个消费者同时读取这些分区，从而达到设计目标。（一般经验，把分区大小限制在25G之内比较理想）

#### log.retention.hours

日志保存时间，默认为7天（168小时）。超过这个时间会清理数据。bytes和minutes无论哪个先达到都会触发。与此类似还有log.retention.minutes和log.retention.ms，都设置的话，优先使用具有最小值的那个。（提示：时间保留数据是通过检查磁盘上日志片段文件的最后修改时间来实现的。也就是最后修改时间是指日志片段的关闭时间，也就是文件里最后一个消息的时间戳）

#### log.retention.bytes

topic每个分区的最大文件大小，一个topic的大小限制 = 分区数*log.retention.bytes。-1没有大小限制。log.retention.bytes和log.retention.minutes任意一个达到要求，都会执行删除。(注意如果是log.retention.bytes先达到了，则是删除多出来的部分数据)，一般不推荐使用最大文件删除策略，而是推荐使用文件过期删除策略。

#### log.segment.bytes

分区的日志存放在某个目录下诸多文件中，这些文件将分区的日志切分成一段一段的，我们称为日志片段。这个属性就是每个文件的最大尺寸；当尺寸达到这个数值时，就会关闭当前文件，并创建新文件。被关闭的文件就开始等待过期。默认为1G。

如果一个主题每天只接受100MB的消息，那么根据默认设置，需要10天才能填满一个文件。而且因为日志片段在关闭之前，消息是不会过期的，所以如果log.retention.hours保持默认值的话，那么这个日志片段需要17天才过期。因为关闭日志片段需要10天，等待过期又需要7天。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670340547050/7c7d1cc5fdad48ff8494373aabae22f0.png)

#### log.segment.ms

作用和log.segment.bytes类似，只不过判断依据是时间。同样的，两个参数，以先到的为准。这个参数默认是不开启的。

#### message.max.bytes

表示一个服务器能够接收处理的消息的最大字节数，注意这个值producer和consumer必须设置一致，且不要大于fetch.message.max.bytes属性的值(消费者能读取的最大消息,这个值应该大于或等于message.max.bytes)。该值默认是1000000字节，大概900KB~1MB。如果启动压缩，判断压缩后的值。这个值的大小对性能影响很大，值越大，网络和IO的时间越长，还会增加磁盘写入的大小。

Kafka设计的初衷是迅速处理短小的消息，一般10K大小的消息吞吐性能最好（LinkedIn的kafka性能测试）

## 硬件配置对Kafka性能的影响

为Kafka选择合适的硬件更像是一门艺术，就跟它的名字一样，我们分别从磁盘、内存、网络和CPU上来分析，确定了这些关注点，就可以在预算范围之内选择最优的硬件配置。

### 磁盘吞吐量/磁盘容量

磁盘吞吐量（IOPS 每秒的读写次数）会影响生产者的性能。因为生产者的消息必须被提交到服务器保存，大多数的客户端都会一直等待，直到至少有一个服务器确认消息已经成功提交为止。也就是说，磁盘写入速度越快，生成消息的延迟就越低。（SSD固态贵单个速度快，HDD机械偏移可以多买几个，设置多个目录加快速度，具体情况具体分析）

磁盘容量的大小，则主要看需要保存的消息数量。如果每天收到1TB的数据，并保留7天，那么磁盘就需要7TB的数据。

### 内存

Kafka本身并不需要太大内存，内存则主要是影响消费者性能。在大多数业务情况下，消费者消费的数据一般会从内存（页面缓存，从系统内存中分）中获取，这比在磁盘上读取肯定要快的多。一般来说运行Kafka的JVM不需要太多的内存，剩余的系统内存可以作为页面缓存，或者用来缓存正在使用的日志片段，所以我们一般Kafka不会同其他的重要应用系统部署在一台服务器上，因为他们需要共享页面缓存，这个会降低Kafka消费者的性能。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670340547050/5c86e0eeab2845239eea4cf6ba403c45.png)

### 网络

网络吞吐量决定了Kafka能够处理的最大数据流量。它和磁盘是制约Kafka拓展规模的主要因素。对于生产者、消费者写入数据和读取数据都要瓜分网络流量。同时做集群复制也非常消耗网络。

### CPU

Kafka对cpu的要求不高，主要是用在对消息解压和压缩上。所以cpu的性能不是在使用Kafka的首要考虑因素。

### 总结

我们要为Kafka选择合适的硬件时，优先考虑存储，包括存储的大小，然后考虑生产者的性能（也就是磁盘的吞吐量），选好存储以后，再来选择CPU和内存就容易得多。网络的选择要根据业务上的情况来定，也是非常重要的一环。





# Kafka中的 zookeeper

ZooKeeper是一个分布式的，开放源码的分布式应用程序协调服务，是Google的Chubby一个开源的实现，是Hadoop和Hbase的重要组件。它是一个为分布式应用提供一致性服务的软件，提供的功能包括：配置维护、域名服务、分布式同步、组服务等。

## Zookeeper的工作机制

　　Zookeeper从设计模式的角度来理解：是一个基于观察者模式的分布式服务管理框架，它负责存储和管理重要的数据，然后接受观察者的注册，一旦这些被观察的数据状态发生变化，Zookeeper就负责通知已经在Zookeeper上注册的那些观察者让他们做出相应的反应。

![img](https://img2020.cnblogs.com/blog/1034836/202102/1034836-20210228200248214-181193185.png)

## Zookeeper的特点

【1】Zookeeper是存在一个领导者（Leader）和多个跟随者（Follower） 组成的集群。
【2】集群中若存在半数以上的（服务器存活数量必须大于一半，小于等于一半都不行）节点存活，就能正常工作。
【3】数据一致：每个Server保存一份相同的数据副本，Client无论连接到哪一个Server获取的数据都是一样的。
【4】更新请求按发送的顺序依次执行。
【5】数据更新原子性原则，要么一次更新成功，要么失败。
【6】实时性，Client能够读取到最新的数据。



早期版本的kafka用zk做meta信息存储，consumer的消费状态，group的管理以及 offset的值。考虑到zk本身的一些因素以及整个架构较大概率存在单点问题，新版本中逐渐弱化了zookeeper的作用

新的consumer使用了kafka内部的group coordination协议，也减少了对zookeeper的依赖， 但是broker依然依赖于ZK，zookeeper 在kafka中还用来选举controller 和 检测broker是否存活等等。

2021年 4月发布的2.8.0版本已取消 zk的依赖

## Zookeeper的Leader 选举

　　服务器状态：

- looking：寻找leader状态。当服务器处于该状态时，它会认为当前集群中没有Leader，因此需要进入 Leader 选举流程。
- leading：领导者状态。表明当前服务器角色是leader。
- following：跟随者状态。表明当前服务器角色是follower。
- observing：观察者状态。表明当前服务器角色是observer。

　　在zookeeper运行期间，leader与非leader服务器各司其职，即便当有非leader服务器宕机或新加入，此时也不会影响leader，但是一旦leader服务器挂了，那么整个集群将暂停对外服务，进入新一轮leader选举，其过程和启动时期的Leader选举过程基本一致。

 

　　假设正在运行的有server1、server2、server3三台服务器，当前leader是server2，若某一时刻leader挂了，此时便开始Leader选举。选举过程如下:

1. 变更状态。leader挂后，余下的非 Observer 服务器都会将自己的服务器状态变更为looking，然后开始进入leader选举过程。
2. 每个server会发出一个投票。在运行期间，每个服务器上的zxid可能不同，此时假定server1的zxid为123，server3的zxid为122，在第一轮投票中，server1和server3都会投自己，产生投票(1, 123)，(3, 122)，然后各自将投票发送给集群中所有机器。
3. 接收来自各个服务器的投票。
4. 处理投票。对于投票的处理，和上面提到的服务器启动期间的处理规则是一致的。在这个例子里面，由于 Server1 的 zxid 为 123，Server3 的 zxid 为 122，那么显然，Server1 会成为 Leader。
5. 统计投票。
6. 改变服务器状态。



## Kafka的Leader选举

### 　　1.基础概念　

#### 　　【AR&ISR&OSR】

　　分区中的所有副本统称为AR（**Assigned Replicas**）。所有与leader副本保持一定程度同步的副本（包括leader副本在内）组成ISR（**In-Sync Replicas**），ISR集合是AR集合中的一个子集。与leader副本同步滞后过多的副本（不包括leader副本）组成OSR（**Out-of-Sync Replicas**），由此可见，AR=ISR+OSR。消息会先发送到leader副本，然后follower副本才能从leader副本中拉取消息进行同步，同步期间内follower副本相对于leader副本而言会有一定程度的滞后。在正常情况下，所有的 follower 副本都应该与 leader 副本保持一定程度的同步，即 AR=ISR，OSR集合为空。
　　leader副本负责维护和跟踪ISR集合中所有follower副本的滞后状态，当follower副本落后太多或失效时，leader副本会把它从ISR集合中剔除。如果OSR集合中有follower副本“追上”了leader副本，那么leader副本会把它从OSR集合转移至ISR集合。默认情况下，当leader副本发生故障时，只有在ISR集合中的副本才有资格被选举为新的leader，而在OSR集合中的副本则没有任何机会（不过这个原则也可以通过修改相应的参数配置来改变）。

#### 　　【ISR的伸缩】

　　Kafka在启动的时候会开启两个与ISR相关的定时任务，名称分别为“isr-expiration"和”isr-change-propagation".。isr-expiration任务会周期性的检测每个分区是否需要缩减其ISR集合。这个周期和“**replica.lag.time.max.ms**”(延迟时间)参数有关。大小是这个参数一半。默认值为5000ms，当检测到ISR中有是失效的副本的时候，就会缩减ISR集合。如果某个分区的ISR集合发生变更， 则会将变更后的数据记录到ZooKerper对应/brokers/topics//partition//state节点中。

### 　　2.leader

　　在kafka集群中有2个种leader，一种是broker的leader即controller leader，还有一种就是partition的leader。

#### 　　【Controller leader】

　　当broker启动的时候，都会创建KafkaController对象，但是集群中只能有一个leader对外提供服务，这些每个节点上的KafkaController会在指定的zookeeper路径下创建临时节点，只有第一个成功创建的节点的KafkaController才可以成为leader，其余的都是follower。当leader故障后，所有的follower会收到通知，再次竞争在该路径下创建节点从而选举新的leader

#### 　　【Partition leader】

 　由controller leader执行

- 从Zookeeper中读取当前分区的所有ISR(in-sync replicas)集合

- 调用配置的分区选择算法选择分区的leader

  #### 选举示意图：

  LEO 表示  Log End Offset 下一条等待写入的消息的offset（最新的Offset+1。每一个消息都有一个offset ，offset实际也是topic默认50个）

  HW 表示 Hign Watermark ISR中最小的LED（因为数据可能没有完全保持同步，所以Consumer最多只能消费到HW之前的位置，比如这里消费到offset5的消息，也就是说其他副本没有同步过去的消息，是不能被消费的）

  follower故障 

  follower发生故障后会被临时踢出ISR，待该follower恢复后，follower会读取本地磁盘记录的上次的HW，并将log文件高于HW的部分截取掉，从HW开始向leader进行同步。

  等该follower的LEO大于等于该Partition的HW，即follower追上leader之后，就可以重新加入ISR了。

![image-20240125205935981](E:\图灵课堂\MQ专题\MQ专题.assets\image-20240125205935981.png)



# Kafka生产与消费全流程

Kafka是一款消息中间件，消息中间件本质就是收消息与发消息，所以这节课我们会从一条消息开始生产出发，去了解生产端的运行流程，然后简单的了解一下broker的存储流程，最后这条消息是如何被消费者消费掉的。其中最核心的有以下内容。

1、Kafka客户端是如何去设计一个非常优秀的生产级的保证高吞吐的一个缓冲机制

2、消费端的原理：每个消费组的群主如何选择，消费组的群组协调器如何选择，分区分配的方法，分布式消费的实现机制，拉取消息的原理，offset提交的原理。

# Kafka一条消息发送和消费的流程(非集群)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670654953054/35d3bd71bad045dd92d5a117c71e1368.png)

## 简单入门

我们这里使用Kafka内置的客户端API开发kafka应用程序。因为我们是Java程序员，所以这里我们使用Maven，使用较新的版本

```
  <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-clients</artifactId>
            <version>3.3.1</version>
  </dependency>
```

### 生产者

先创建一个主题，推荐在消息发送时创建对应的主题。当然就算没有创建主题，Kafka也能自动创建。

**auto.create.topics.enable**

是否允许自动创建主题。如果设为true，那么produce（生产者往主题写消息），consume（消费者从主题读消息）或者fetch metadata（任意客户端向主题发送元数据请求时）一个不存在的主题时，就会自动创建。缺省为true。

**num.partitions**

每个新建主题的分区个数（分区个数只能增加，不能减少 ）。这个参数默认值是1（最新版本）

#### 必选属性

创建生产者对象时有三个属性必须指定。

##### bootstrap.servers

该属性指定broker的地址清单，地址的格式为host:port。

清单里不需要包含所有的broker地址，生产者会从给定的broker里查询其他broker的信息。不过最少提供2个broker的信息(用逗号分隔，比如:127.0.0.1:9092,192.168.0.13:9092)，一旦其中一个宕机，生产者仍能连接到集群上。

##### key.serializer

生产者接口允许使用参数化类型，可以把Java对象作为键和值传broker，但是broker希望收到的消息的键和值都是字节数组，所以，必须提供将对象序列化成字节数组的序列化器。

key.serializer必须设置为实现org.apache.kafka.common.serialization.Serializer的接口类

Kafka的客户端默认提供了ByteArraySerializer,IntegerSerializer,StringSerializer，也可以实现自定义的序列化器。

##### value.serializer

同 key.serializer。

#### 三种发送方式

我们通过生成者的send方法进行发送。send方法会返回一个包含RecordMetadata的Future对象。RecordMetadata里包含了目标主题，分区信息和消息的偏移量。

##### 发送并忘记

忽略send方法的返回值，不做任何处理。大多数情况下，消息会正常到达，而且生产者会自动重试，但有时会丢失消息。

```java
package com.tuling.producer;

import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.apache.kafka.common.serialization.StringSerializer;

import java.util.Properties;

/**
 * 类说明：kafak生产者
 */
public class HelloKafkaProducer {

    public static void main(String[] args) {
        // 设置属性
        Properties properties = new Properties();
        // 指定连接的kafka服务器的地址
        properties.put("bootstrap.servers","127.0.0.1:9092");
        // 设置String的序列化
        properties.put("key.serializer", StringSerializer.class);
        properties.put("value.serializer", StringSerializer.class);

        // 构建kafka生产者对象
        KafkaProducer<String,String> producer  = new KafkaProducer<String, String>(properties);
        try {
            ProducerRecord<String,String> record;
            try {
                // 构建消息
                record = new ProducerRecord<String,String>("msb", "teacher","lijin");
                // 发送消息
                producer.send(record);
                System.out.println("message is sent.");
            } catch (Exception e) {
                e.printStackTrace();
            }
        } finally {
            // 释放连接
            producer.close();
        }
    }


}

```

##### 同步发送

获得send方法返回的Future对象，在合适的时候调用Future的get方法。参见代码。

```java
package com.tuling.producer;

import org.apache.kafka.clients.producer.Callback;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.clients.producer.RecordMetadata;
import org.apache.kafka.common.serialization.StringSerializer;

import java.util.Properties;
import java.util.concurrent.Future;

/**
 * 类说明：发送消息--同步模式
 */
public class SynProducer {

    public static void main(String[] args) {
        // 设置属性
        Properties properties = new Properties();
        // 指定连接的kafka服务器的地址
        properties.put("bootstrap.servers","127.0.0.1:9092");
        // 设置String的序列化
        properties.put("key.serializer", StringSerializer.class);
        properties.put("value.serializer", StringSerializer.class);

        // 构建kafka生产者对象
        KafkaProducer<String,String> producer  = new KafkaProducer<String, String>(properties);
        try {
            ProducerRecord<String,String> record;
            try {
                // 构建消息
                record = new ProducerRecord<String,String>("msb", "teacher2333","lijin");
                // 发送消息
                Future<RecordMetadata> future =producer.send(record);
                RecordMetadata recordMetadata = future.get();
                if(null!=recordMetadata){
                    System.out.println("offset:"+recordMetadata.offset()+","
                            +"partition:"+recordMetadata.partition());
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        } finally {
            // 释放连接
            producer.close();
        }
    }




}

```

##### 异步发送

实现接口org.apache.kafka.clients.producer.Callback，然后将实现类的实例作为参数传递给send方法。

```java
package com.tuling.producer;

import org.apache.kafka.clients.producer.Callback;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.clients.producer.RecordMetadata;
import org.apache.kafka.common.serialization.StringSerializer;

import java.util.Properties;
import java.util.concurrent.Future;

/**
 * 类说明：发送消息--异步模式
 */
public class AsynProducer {

    public static void main(String[] args) {
        // 设置属性
        Properties properties = new Properties();
        // 指定连接的kafka服务器的地址
        properties.put("bootstrap.servers","127.0.0.1:9092");
        // 设置String的序列化
        properties.put("key.serializer", StringSerializer.class);
        properties.put("value.serializer", StringSerializer.class);

        // 构建kafka生产者对象
        KafkaProducer<String,String> producer  = new KafkaProducer<String, String>(properties);

        try {
            ProducerRecord<String,String> record;
            try {
                // 构建消息
                record = new ProducerRecord<String,String>("msb", "teacher","lijin");
                // 发送消息
                producer.send(record, new Callback() {
                    @Override
                    public void onCompletion(RecordMetadata recordMetadata, Exception e) {
                        if (e == null){
                            // 没有异常，输出信息到控制台
                            System.out.println("offset:"+recordMetadata.offset()+"," +"partition:"+recordMetadata.partition());
                        } else {
                            // 出现异常打印
                            e.printStackTrace();
                        }
                    }
                });
            } catch (Exception e) {
                e.printStackTrace();
            }
        } finally {
            // 释放连接
            producer.close();
        }
    }




}

```

### 消费者

消费者的含义，同一般消息中间件中消费者的概念。在高并发的情况下，生产者产生消息的速度是远大于消费者消费的速度，单个消费者很可能会负担不起，此时有必要对消费者进行横向伸缩，于是我们可以使用多个消费者从同一个主题读取消息，对消息进行分流。

#### 必选属性

创建消费者对象时一般有四个属性必须指定。

bootstrap.servers、value.Deserializer key.Deserializer 含义同生产者

#### 可选属性

group.id  并非完全必需，它指定了消费者属于哪一个群组，但是创建不属于任何一个群组的消费者并没有问题。不过绝大部分情况我们都会使用群组消费。

#### 消费者群组

Kafka里消费者从属于消费者群组，一个群组里的消费者订阅的都是同一个主题，每个消费者接收主题一部分分区的消息。

![](file:///C:\Users\ADMINI~1\AppData\Local\Temp\msohtmlclip1\01\clip_image002.jpg)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670654953054/5693e8540e2b4b31b73c8b6ac6ee7038.png)

如上图，主题T有4个分区，群组中只有一个消费者，则该消费者将收到主题T1全部4个分区的消息。

![](file:///C:\Users\ADMINI~1\AppData\Local\Temp\msohtmlclip1\01\clip_image004.jpg)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670654953054/dc2400aa869a414abe8909c1e59190e5.png)

如上图，在群组中增加一个消费者2，那么每个消费者将分别从两个分区接收消息，上图中就表现为消费者1接收分区1和分区3的消息，消费者2接收分区2和分区4的消息。

![](file:///C:\Users\ADMINI~1\AppData\Local\Temp\msohtmlclip1\01\clip_image006.jpg)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670654953054/1a2816e2b9b94ffa88598d5e7e021841.png)

如上图，在群组中有4个消费者，那么每个消费者将分别从1个分区接收消息。

![](file:///C:\Users\ADMINI~1\AppData\Local\Temp\msohtmlclip1\01\clip_image008.jpg)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670654953054/6992d6eacdbb4e31b1ab2facaf09dd0f.png)

但是，当我们增加更多的消费者，超过了主题的分区数量，就会有一部分的消费者被闲置，不会接收到任何消息。

往消费者群组里增加消费者是进行横向伸缩能力的主要方式。所以我们有必要为主题设定合适规模的分区，在负载均衡的时候可以加入更多的消费者。但是要记住，一个群组里消费者数量超过了主题的分区数量，多出来的消费者是没有用处的。

## 序列化

创建生产者对象必须指定序列化器，默认的序列化器并不能满足我们所有的场景。我们完全可以自定义序列化器。只要实现org.apache.kafka.common.serialization.Serializer接口即可。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670654953054/8e473e701c814e6e9b755f6fe44499b1.png)

### 自定义序列化

代码见：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670654953054/8eb6dfa47624416fb16ed4b02739ee5d.png)

代码中使用到了自定义序列化。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670654953054/0a38704c657a48d38c2f4e55102547d6.png)

id的长度4个字节，字符串的长度描述4个字节， 字符串本身的长度nameSize个字节

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670654953054/a6d7fd2e37924e61bb63ccad4dd0c657.png)

自定义序列化容易导致程序的脆弱性。举例，在我们上面的实现里，我们有多种类型的消费者，每个消费者对实体字段都有各自的需求，比如，有的将字段变更为long型，有的会增加字段，这样会出现新旧消息的兼容性问题。特别是在系统升级的时候，经常会出现一部分系统升级，其余系统被迫跟着升级的情况。

解决这个问题，可以考虑使用自带格式描述以及语言无关的序列化框架。比如Protobuf，Kafka官方推荐的Apache Avro

## 分区

因为在Kafka中一个topic可以有多个partition，所以当一个生产发送消息，这条消息应该发送到哪个partition，这个过程就叫做分区。

当然，我们在新建消息的时候，我们可以指定partition，只要指定partition，那么分区器的策略则失效。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670654953054/65ed2b6059b746e2b43532ca18c0369a.png)

### 系统分区器

在我们的代码中可以看到，生产者参数中是可以选择分区器的。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670654953054/c088dd4584bb4285b282a306abef438b.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670654953054/af5e0e57c3ca412aad7e668cf2142f8e.png)

#### DefaultPartitioner 默认分区策略

全路径类名：org.apache.kafka.clients.producer.internals.DefaultPartitioner

* 如果消息中指定了分区，则使用它
* 如果未指定分区但存在key，则根据序列化key使用murmur2哈希算法对分区数取模。
* 如果不存在分区或key，则会使用**粘性分区策略**

采用默认分区的方式，键的主要用途有两个：

一，用来决定消息被写往主题的哪个分区，拥有相同键的消息将被写往同一个分区。

二，还可以作为消息的附加消息。

#### RoundRobinPartitioner 分区策略

全路径类名：org.apache.kafka.clients.producer.internals.RoundRobinPartitioner

* 如果消息中指定了分区，则使用它
* 将消息平均的分配到每个分区中。

即key为null，那么这个时候一般也会采用RoundRobinPartitioner

#### UniformStickyPartitioner 纯粹的粘性分区策略

全路径类名：org.apache.kafka.clients.producer.internals.UniformStickyPartitioner

他跟**DefaultPartitioner** 分区策略的唯一区别就是。

**DefaultPartitionerd 如果有key的话,那么它是按照key来决定分区的,这个时候并不会使用粘性分区**
**UniformStickyPartitioner 是不管你有没有key, 统一都用粘性分区来分配**

#### 另外关于粘性分区策略

从客户端最新的版本上来看（3.3.1），有两个序列化器已经进入 弃用阶段。

这个客户端在3.1.0都还不是这样。关于粘性分区策略

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670654953054/c9e0c8d6613b4b73b87e320823f76391.png)

如果感兴趣可以看下这篇文章

[https://bbs.huaweicloud.com/blogs/348729?utm_source=oschina&amp;utm_medium=bbs-ex&amp;utm_campaign=other&amp;utm_content=content]()

### 自定义分区器

我们完全可以去实现Partitioner接口，去实现有一个自定义的分区器

```java
package com.tuling.selfpartition;

import org.apache.kafka.clients.producer.Partitioner;
import org.apache.kafka.common.Cluster;
import org.apache.kafka.common.PartitionInfo;
import org.apache.kafka.common.utils.Utils;

import java.util.List;
import java.util.Map;

/**
 * 类说明：自定义分区器，以value值进行分区
 */
public class SelfPartitioner implements Partitioner {
    public int partition(String topic, Object key, byte[] keyBytes,
                         Object value, byte[] valueBytes, Cluster cluster) {
        List<PartitionInfo> partitionInfos = cluster.partitionsForTopic(topic);
        int num = partitionInfos.size();
        int parId = Utils.toPositive(Utils.murmur2(valueBytes)) % num;//来自DefaultPartitioner的处理
        return parId;
    }

    public void close() {
        //do nothing
    }

    public void configure(Map<String, ?> configs) {
        //do nothing
    }

}

```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670654953054/f07c49100ca44560975e7f94f00713c4.png)

## 生产缓冲机制

客户端发送消息给kafka服务器的时候、消息会先写入一个内存缓冲中，然后直到多条消息组成了一个Batch，才会一次网络通信把Batch发送过去。主要有以下参数：

**buffer.memory**

设置生产者内存缓冲区的大小，生产者用它缓冲要发送到服务器的消息。如果数据产生速度大于向broker发送的速度，导致生产者空间不足，producer会阻塞或者抛出异常。缺省33554432 (32M)

buffer.memory: 所有缓存消息的总体大小超过这个数值后，就会触发把消息发往服务器。此时会忽略batch.size和linger.ms的限制。
buffer.memory的默认数值是32 MB，对于单个 Producer 来说，可以保证足够的性能。 需要注意的是，如果您在同一个JVM中启动多个 Producer，那么每个 Producer 都有可能占用 32 MB缓存空间，此时便有可能触发 OOM。

**batch.size**

当多个消息被发送同一个分区时，生产者会把它们放在同一个批次里。该参数指定了一个批次可以使用的内存大小，按照字节数计算。当批次内存被填满后，批次里的所有消息会被发送出去。但是生产者不一定都会等到批次被填满才发送，半满甚至只包含一个消息的批次也有可能被发送（linger.ms控制）。缺省16384(16k) ，如果一条消息超过了批次的大小，会写不进去。

**linger.ms**

指定了生产者在发送批次前等待更多消息加入批次的时间。它和batch.size以先到者为先。也就是说，一旦我们获得消息的数量够batch.size的数量了，他将会立即发送而不顾这项设置，然而如果我们获得消息字节数比batch.size设置要小的多，我们需要“linger”特定的时间以获取更多的消息。这个设置默认为0，即没有延迟。设定linger.ms=5，例如，将会减少请求数目，但是同时会增加5ms的延迟，但也会提升消息的吞吐量。

### 为何要设计缓冲机制

1、减少IO的开销（单个 ->批次）但是这种情况基本上也只是linger.ms配置>0的情况下才会有，因为默认inger.ms=0的，所以基本上有消息进来了就发送了，跟单条发送是差不多！！

2、减少Kafka中Java客户端的GC。

比如缓冲池大小是32MB。然后把32MB划分为N多个内存块，比如说一个内存块是16KB（batch.size），这样的话这个缓冲池里就会有很多的内存块。

你需要创建一个新的Batch，就从缓冲池里取一个16KB的内存块就可以了，然后这个Batch就不断的写入消息

下次别人再要构建一个Batch的时候，再次使用缓冲池里的内存块就好了。这样就可以利用有限的内存，对他不停的反复重复的利用。因为如果你的Batch使用完了以后是把内存块还回到缓冲池中去，那么就不涉及到垃圾回收了。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670654953054/5638282571c44238b3b362e3985468d5.png)

## 消费者偏移量提交

一般情况下，我们调用poll方法的时候，broker返回的是生产者写入Kafka同时kafka的消费者提交偏移量，这样可以确保消费者消息消费不丢失也不重复，所以一般情况下Kafka提供的原生的消费者是安全的，但是事情会这么完美吗？

### 自动提交

最简单的提交方式是让消费者自动提交偏移量。 如果enable.auto.commit被设为 true，消费者会自动把从poll()方法接收到的**最大**偏移量提交上去。提交时间间隔由auto.commit.interval.ms控制，默认值是5s。

自动提交是在轮询里进行的，消费者每次在进行轮询时会检査是否该提交偏移量了，如果是，那么就会提交从上一次轮询返回的偏移量。

不过,在使用这种简便的方式之前,需要知道它将会带来怎样的结果。

假设我们仍然使用默认的5s提交时间间隔, 在最近一次提交之后的3s发生了再均衡，再均衡之后,消费者从最后一次提交的偏移量位置开始读取消息。这个时候偏移量已经落后了3s，所以在这3s内到达的消息会被重复处理。可以通过修改提交时间间隔来更频繁地提交偏移量, 减小可能出现重复消息的时间窗, 不过这种情况是无法完全避免的。

在使用自动提交时,每次调用轮询方法都会把上一次调用返回的最大偏移量提交上去,它并不知道具体哪些消息已经被处理了,所以在再次调用之前最好确保所有当前调用返回的消息都已经处理完毕(enable.auto.comnit被设为 true时，在调用 close()方法之前也会进行自动提交)。一般情况下不会有什么问题,不过在处理异常或提前退出轮询时要格外小心。

#### 消费者的配置参数

**auto.offset.reset**

earliest
当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，从头开始消费
latest
当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，消费新产生的该分区下的数据

只要group.Id不变，不管auto.offset.reset 设置成什么值，都从上一次的消费结束的地方开始消费。



# Kafka的消费全流程

我们接着继续去理解最后这条消息是如何被消费者消费掉的。其中最核心的有以下内容。

1、多线程安全问题

2、群组协调

3、分区再均衡

# 多线程安全问题

当多个线程访问某个类时，这个类始终都能表现出正确的行为，那么就称这个类是线程安全的。

对于线程安全，还可以进一步定义：

当多个线程访问某个类时，不管运行时环境采用何种调度方式或者这些线程将如何交替进行，并且在主调代码中不需要任何额外的同步或协同，这个类都能表现出正确的行为，那么就称这个类是线程安全的。

## 生产者

KafkaProducer的实现是线程安全的。

KafkaProducer就是一个不可变类。线程安全的，可以在多个线程中共享单个KafkaProducer实例

所有字段用private final修饰，且不提供任何修改方法，这种方式可以确保多线程安全。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670940113013/c439a638df354357b3dc857f7cc83e86.png)

如何节约资源的多线程使用KafkaProducer实例

```java
package com.tuling.concurrent;

import com.tuling.selfserial.User;
import org.apache.kafka.clients.producer.Callback;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.clients.producer.RecordMetadata;
import org.apache.kafka.common.serialization.StringSerializer;

import java.util.Properties;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * 类说明：多线程下使用生产者
 */
public class KafkaConProducer {

    //发送消息的个数
    private static final int MSG_SIZE = 1000;
    //负责发送消息的线程池
    private static ExecutorService executorService = Executors.newFixedThreadPool(
            Runtime.getRuntime().availableProcessors());
    private static CountDownLatch countDownLatch  = new CountDownLatch(MSG_SIZE);

    private static User makeUser(int id){
        User user = new User(id);
        String userName = "msb_"+id;
        user.setName(userName);
        return user;
    }

    /*发送消息的任务*/
    private static class ProduceWorker implements Runnable{

        private ProducerRecord<String,String> record;
        private KafkaProducer<String,String> producer;

        public ProduceWorker(ProducerRecord<String, String> record, KafkaProducer<String, String> producer) {
            this.record = record;
            this.producer = producer;
        }

        public void run() {
            final String id = Thread.currentThread().getId() +"-"+System.identityHashCode(producer);
            try {
                producer.send(record, new Callback() {
                    public void onCompletion(RecordMetadata metadata, Exception exception) {
                        if(null!=exception){
                            exception.printStackTrace();
                        }
                        if(null!=metadata){
                            System.out.println(id+"|" +String.format("偏移量：%s,分区：%s", metadata.offset(),
                                    metadata.partition()));
                        }
                    }
                });
                System.out.println(id+":数据["+record+"]已发送。");
                countDownLatch.countDown();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
        // 设置属性
        Properties properties = new Properties();
        // 指定连接的kafka服务器的地址
        properties.put("bootstrap.servers","127.0.0.1:9092");
        // 设置String的序列化
        properties.put("key.serializer", StringSerializer.class);
        properties.put("value.serializer", StringSerializer.class);
        // 构建kafka生产者对象
        KafkaProducer<String,String> producer  = new KafkaProducer<String, String>(properties);
        try {
            for(int i=0;i<MSG_SIZE;i++){
                User user = makeUser(i);
                ProducerRecord<String,String> record = new ProducerRecord<String,String>("concurrent-test",null,
                        System.currentTimeMillis(), user.getId()+"", user.toString());
                executorService.submit(new ProduceWorker(record,producer));
            }
            countDownLatch.await();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            producer.close();
            executorService.shutdown();
        }
    }




}

```

## 消费者

KafkaConsumer的实现**不是**线程安全的

实现消费者多线程最常见的方式： **线程封闭** ——即**为每个线程实例化一个 KafkaConsumer对象**

```java
package com.tuling.concurrent;

import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.common.serialization.StringDeserializer;

import java.time.Duration;
import java.util.Collections;
import java.util.HashMap;
import java.util.Map;
import java.util.Properties;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * 类说明：多线程下正确的使用消费者，需要记住，一个线程一个消费者
 */
public class KafkaConConsumer {

    public static final int CONCURRENT_PARTITIONS_COUNT = 2;

    private static ExecutorService executorService = Executors.newFixedThreadPool(CONCURRENT_PARTITIONS_COUNT);

    private static class ConsumerWorker implements Runnable{

        private KafkaConsumer<String,String> consumer;

        public ConsumerWorker(Map<String, Object> config, String topic) {
            Properties properties = new Properties();
            properties.putAll(config);
            this.consumer = new KafkaConsumer<String, String>(properties);
            consumer.subscribe(Collections.singletonList(topic));
        }

        public void run() {
            final String ThreadName = Thread.currentThread().getName();
            try {
                while(true){
                    ConsumerRecords<String, String> records = consumer.poll(Duration.ofSeconds(1));
                    for(ConsumerRecord<String, String> record:records){
                        System.out.println(ThreadName+"|"+String.format(
                                "主题：%s，分区：%d，偏移量：%d，" +
                                        "key：%s，value：%s",
                                record.topic(),record.partition(),
                                record.offset(),record.key(),record.value()));
                        //do our work
                    }
                }
            } finally {
                consumer.close();
            }
        }
    }

    public static void main(String[] args) {

        /*消费配置的实例*/
        Map<String,Object> properties = new HashMap<String, Object>();
        properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG,"127.0.0.1:9092");
        properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG,StringDeserializer.class);
        properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG,StringDeserializer.class);
        properties.put(ConsumerConfig.GROUP_ID_CONFIG,"c_test");
        properties.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG,"earliest");

        for(int i = 0; i<CONCURRENT_PARTITIONS_COUNT; i++){
            //一个线程一个消费者
            executorService.submit(new ConsumerWorker(properties, "concurrent-test"));
        }
    }




}

```

# 群组协调

消费者要加入群组时，会向群组协调器发送一个JoinGroup请求，第一个加入群主的消费者成为群主，群主会获得群组的成员列表，并负责给每一个消费者分配分区。分配完毕后，群主把分配情况发送给群组协调器，协调器再把这些信息发送给所有的消费者，每个消费者只能看到自己的分配信息，只有群主知道群组里所有消费者的分配信息。群组协调的工作会在消费者发生变化(新加入或者掉线)，主题中分区发生了变化（增加）时发生。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670940113013/b070d19ea7584a088c03fed8acfb0b96.png)

## 组协调器

**组协调器是Kafka服务端自身维护的。**

组协调器( **GroupCoordinator** )可以理解为各个消费者协调器的一个中央处理器, 每个消费者的所有交互都是和组协调器( **GroupCoordinator** )进行的。

1. 选举Leader消费者客户端
2. 处理申请加入组的客户端
3. 再平衡后同步新的分配方案
4. 维护与客户端的心跳检测
5. 管理消费者已消费偏移量,并存储至 `__consumer_offset`中

kafka上的组协调器( **GroupCoordinator** )协调器有很多，有多少个 `__consumer_offset`分区, 那么就有多少个组协调器( **GroupCoordinator** )

默认情况下, `__consumer_offset`有50个分区, 每个消费组都会对应其中的一个分区，对应的逻辑为 hash(`group.id`)%分区数。

## 消费者协调器

**每个客户端（消费者的客户端）都会有一个消费者协调器,** 他的主要作用就是向组协调器发起请求做交互, 以及处理回调逻辑

1. 向组协调器发起入组请求
2. 向组协调器发起同步组请求(如果是Leader客户端,则还会计算分配策略数据放到入参传入)
3. 发起离组请求
4. 保持跟组协调器的心跳线程
5. 向组协调器发送提交已消费偏移量的请求

## 消费者加入分组的流程

1、客户端启动的时候, 或者重连的时候会发起JoinGroup的请求来申请加入的组中。

2、当前客户端都已经完成JoinGroup之后, 客户端会收到JoinGroup的回调, 然后客户端会再次向组协调器发起SyncGroup的请求来获取新的分配方案

3、当消费者客户端关机/异常 时, 会触发离组LeaveGroup请求。

当然有主动的消费者协调器发起离组请求，也有组协调器一直会有针对每个客户端的心跳检测, 如果监测失败,则就会将这个客户端踢出Group。

4、客户端加入组内后, 会一直保持一个心跳线程,来保持跟组协调器的一个感知。

并且组协调器会针对每个加入组的客户端做一个心跳监测，如果监测到过期, 则会将其踢出组内并再平衡。

## 消费者消费的offset的存储

__consumer_offsets topic，并且默认提供了kafka_consumer_groups.sh脚本供用户查看consumer信息。

__consumer_offsets 是 kafka 自行创建的，和普通的 topic 相同。它存在的目的之一就是保存 consumer 提交的位移。

```
kafka-consumer-groups.bat --bootstrap-server :9092 --group c_test --describe
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670940113013/2589b72c9be7424bbb87a28a800d0e80.png)

那么如何使用 kafka 提供的脚本查询某消费者组的元数据信息呢？

```
Math.abs(groupID.hashCode()) % numPartitions，
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670940113013/c191a390dffc46a99dc99842960c35a1.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670940113013/6c84975c90324927ba9733b6ca6ff426.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670940113013/f13949780b95482481e81b07d8f8fde2.png)

__consumer_offsets 的每条消息格式大致如图所示

可以想象成一个 KV 格式的消息，key 就是一个三元组：`group.id+topic+分区号`，而 value 就是 offset 的值

# 分区再均衡

当消费者群组里的消费者发生变化，或者主题里的分区发生了变化，都会导致再均衡现象的发生。从前面的知识中，我们知道，Kafka中，存在着消费者对分区所有权的关系，

这样无论是消费者变化，比如增加了消费者，新消费者会读取原本由其他消费者读取的分区，消费者减少，原本由它负责的分区要由其他消费者来读取，增加了分区，哪个消费者来读取这个新增的分区，这些行为，都会导致分区所有权的变化，这种变化就被称为 **再均衡** 。

再均衡对Kafka很重要，这是消费者群组带来高可用性和伸缩性的关键所在。不过一般情况下，尽量减少再均衡，因为再均衡期间，消费者是无法读取消息的，会造成整个群组一小段时间的不可用。

消费者通过向称为群组协调器的broker（不同的群组有不同的协调器）发送心跳来维持它和群组的从属关系以及对分区的所有权关系。如果消费者长时间不发送心跳，群组协调器认为它已经死亡，就会触发一次再均衡。

心跳由单独的线程负责，相关的控制参数为max.poll.interval.ms。

## 消费者提交偏移量导致的问题

当我们调用poll方法的时候，broker返回的是生产者写入Kafka但是还没有被消费者读取过的记录，消费者可以使用Kafka来追踪消息在分区里的位置，我们称之为 **偏移量** 。消费者更新自己读取到哪个消息的操作，我们称之为 **提交** 。

消费者是如何提交偏移量的呢？消费者会往一个叫做_consumer_offset的特殊主题发送一个消息，里面会包括每个分区的偏移量。发生了再均衡之后，消费者可能会被分配新的分区，为了能够继续工作，消费者者需要读取每个分区最后一次提交的偏移量，然后从指定的地方，继续做处理。

**分区再均衡的例子：**

某软件公司，有一个项目，有两块的工作，有两个码农，一个小王、一个小李，一个负责一块（分区消费），干得好好的。突然一天，小王桌子一拍不干了，老子中了5百万了，不跟你们玩了，立马收拾完电脑就走了。这个时候小李就必须承担两块工作，这个时候就是发生了分区再均衡。

过了几天，你入职，一个萝卜一个坑，你就入坑了，你承担了原来小王的工作。这个时候又会发生了分区再均衡。

1）如果提交的偏移量小于消费者实际处理的最后一个消息的偏移量，处于两个偏移量之间的消息会被重复处理，

2）如果提交的偏移量大于客户端处理的最后一个消息的偏移量,那么处于两个偏移量之间的消息将会丢失

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670940113013/1f787d4a41484ab28e617decfe094b38.png)

### 再均衡监听器实战

我们创建一个分区数是3的主题rebalance

```
kafka-topics.bat --bootstrap-server localhost:9092  --create --topic rebalance --replication-factor 1 --partitions 3
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1670940113013/7af6e805d1d14056bdfd5ae0634d71a7.png)

在为消费者分配新分区或移除旧分区时,可以通过消费者API执行一些应用程序代码，在调用 subscribe()方法时传进去一个 ConsumerRebalancelistener实例就可以了。

ConsumerRebalancelistener有两个需要实现的方法。

1) public void
   onPartitionsRevoked( Collection< TopicPartition> partitions)方法会在

再均衡开始之前和消费者停止读取消息之后被调用。如果在这里提交偏移量，下一个接管分区的消费者就知道该从哪里开始读取了

2) public void
   onPartitionsAssigned( Collection< TopicPartition> partitions)方法会在重新分配分区之后和消费者开始读取消息之前被调用。

具体使用，我们先创建一个3分区的主题，然后实验一下，

在再均衡开始之前会触发onPartitionsRevoked方法

在再均衡开始之后会触发onPartitionsAssigned方法



## Kafka集群的目标

1、高并发

2、高可用（防数据丢失）

3、动态伸缩

## Kafka集群规模如何预估

**吞吐量：**

集群可以提高处理请求的能力。单个Broker的性能不足，可以通过扩展broker来解决。

**磁盘空间：**

比如，如果一个集群有10TB的数据需要保留，而每个broker可以存储2TB，那么至少需要5个broker。如果启用了数据复制，则还需要一倍的空间，那么这个集群需要10个broker。

## Kafka集群搭建实战

使用两台Linux服务器：一台192.68.10.7  一台192.168.10.8  （课程视频中IP地址可能会不同）

192.68.10.7 的配置信息修改

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1672019756062/156b6579a45f426294c034030e653b58.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1672019756062/74a5d5271b7f47d2a99e0988b09e1c80.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1672019756062/9095c87e89034107a40326eb596bbf41.png)

192.168.10.8的配置信息修改

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1672019756062/71b91ac433864925aa4f9d886f12f708.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1672019756062/cd990f2e9b874f01ade5c75c4ee646b2.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1672019756062/d7052fc16a3d4347851f288e37baee16.png)

## Kafka集群原理

### **成员关系与控制器**

控制器其实就是一个broker, 只不过它除了具有一般 broker的功能之外, 还负责分区首领的选举。

当控制器发现一个broker加入集群时, 它会使用 broker ID来检査新加入的 broker是否包含现有分区的副本。 如果有, 控制器就把变更通知发送给新加入的 broker和其他 broker, 新 broker上的副本开始从首领那里复制消息。

简而言之, Kafka使用 Zookeeper的临时节点来选举控制器,并在节点加入集群或退出集群时通知控制器。 控制器负责在节点加入或离开集群时进行分区首领选举。

从下面的两台启动日志中可以明显看出，192.168.10.7 这台服务器是控制器。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1672019756062/19b8a57332e04ee88e9149bfea2b52f4.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1672019756062/7040d717f3db4413ab7a7e94c86658cb.png)

### **集群工作机制**

复制功能是 Kafka 架构的核心。在 Kafka 的文档里, Kafka 把自己描述成“一个分布式的、可分区的、可复制的提交日志服务”。

复制之所以这么关键, 是因为它可以在个别节点失效时仍能保证 Kafka 的可用性和持久性。
Kafka 使用主题来组织数据, 每个主题被分为若干个分区,每个分区有多个副本。那些副本被保存在 broker 上, 每个 broker 可以保存成百上千个属于 不同主题和分区的副本。

#### replication-factor参数

比如我们创建一个lijin的主题，复制因子是2，分区数是2

```
./kafka-topics.sh --bootstrap-server 192.168.10.7:9092  --create --topic lijin --replication-factor 2 --partitions 2
```

replication-factor用来设置主题的副本数。每个主题可以有多个副本，副本位于集群中不同的 broker 上，也就是说副本的数量不能超过 broker 的数量。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1672019756062/e1a1e3b2bfc84f70bbd4ad019443e1cc.png)

从这里可以看出，lijin分区有两个分区，partition0和partition1 ，其中

在partition0 中，broker1（broker.id =0）是Leader，broker2（broker.id =1）是跟随副本。

在partition1 中，broker2（broker.id =1）是Leader，broker1（broker.id =0）是跟随副本。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1672019756062/142a0a3fea0648ef9db60ce163894673.png)

#### ***首领副本***

每个分区都有一个首领副本。为了保证一致性，所有生产者请求和消费者请求都会经过这个副本 。

#### ***跟随者副本***

首领以外的副本都是跟随者副本。跟随者副本不处理来自客户端的请求,它们唯一一的任务就是从首领那里复制消息, 保持与首领一致的状态 。 如果首领发生崩溃, 其中的一个跟随者会被提升为新首领 。

#### auto.leader.rebalance.enable参数

是否允许定期进行 Leader 选举。

设置它的值为true表示允许Kafka定期地对一些Topic 分区进行Leader重选举，当然这个重选举不是无脑进行的，它要满足一定的条件才会发生。

比如Leader A一直表现得很好，但若auto.leader.rebalance.enable=true，那么有可能一段时间后Leader A就要被强行卸任换成Leader B。
你要知道换一次Leader 代价很高的，原本向A发送请求的所有客户端都要切换成向B发送请求，而且这种换Leader本质上没有任何性能收益，因此建议在生产环境中把这个参数设置成false。

### 集群消息生产

**复制系数、不完全的首领选举、最少同步副本**

#### **可靠系统里的生产者**

发送确认机制

3 种不同的确认模式。

acks=0 意味着如果生产者能够通过网络把消息发送出去，那么就认为消息已成功写入Kafka 。

acks=1 意味若首领在收到消息并把它写入到分区数据文件（不一定同步到磁盘上）时会返回确认或错误响应。

acks=all 意味着首领在返回确认或错误响应之前，会等待（min.insync.replicas）同步副本都收到悄息。

##### ISR

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1672019756062/5ded939eac6f4a54ab7cbe2f6c370e05.png)

Kafka的数据复制是以Partition为单位的。而多个备份间的数据复制，通过Follower向Leader拉取数据完成。从一这点来讲，有点像Master-Slave方案。不同的是，Kafka既不是完全的同步复制，也不是完全的异步复制，而是基于ISR的动态复制方案。

ISR，也即In-Sync Replica。每个Partition的Leader都会维护这样一个列表，该列表中，包含了所有与之同步的Replica（包含Leader自己）。每次数据写入时，只有ISR中的所有Replica都复制完，Leader才会将其置为Commit，它才能被Consumer所消费。

这种方案，与同步复制非常接近。但不同的是，这个ISR是由Leader动态维护的。如果Follower不能紧“跟上”Leader，它将被Leader从ISR中移除，待它又重新“跟上”Leader后，会被Leader再次加加ISR中。每次改变ISR后，Leader都会将最新的ISR持久化到Zookeeper中。

至于如何判断某个Follower是否“跟上”Leader，不同版本的Kafka的策略稍微有些区别。

从0.9.0.0版本开始，replica.lag.max.messages被移除，故Leader不再考虑Follower落后的消息条数。另外，Leader不仅会判断Follower是否在replica.lag.time.max.ms时间内向其发送Fetch请求，同时还会考虑Follower是否在该时间内与之保持同步。

##### 示例

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1672019756062/4413573076b44b3fb500bc817bb68eba.png)

在第一步中，Leader A总共收到3条消息，但由于ISR中的Follower只同步了第1条消息（m1），故只有m1被Commit，也即只有m1可被Consumer消费。此时Follower B与Leader A的差距是1，而Follower C与Leader A的差距是2，虽然有消息的差距，但是满足同步副本的要求保留在ISR中。

在第二步中，由于旧的Leader A宕机，新的Leader B在replica.lag.time.max.ms时间内未收到来自A的Fetch请求，故将A从ISR中移除，此时ISR={B，C}。同时，由于此时新的Leader B中只有2条消息，并未包含m3（m3从未被任何Leader所Commit），所以m3无法被Consumer消费。

##### 使用ISR方案的原因

由于Leader可移除不能及时与之同步的Follower，故与同步复制相比可避免最慢的Follower拖慢整体速度，也即ISR提高了系统可用性。

ISR中的所有Follower都包含了所有Commit过的消息，而只有Commit过的消息才会被Consumer消费，故从Consumer的角度而言，ISR中的所有Replica都始终处于同步状态，从而与异步复制方案相比提高了数据一致性。

##### ISR相关配置说明

Broker的min.insync.replicas参数指定了Broker所要求的ISR最小长度，默认值为1。也即极限情况下ISR可以只包含Leader。但此时如果Leader宕机，则该Partition不可用，可用性得不到保证。

只有被ISR中所有Replica同步的消息才被Commit，但Producer发布数据时，Leader并不需要ISR中的所有Replica同步该数据才确认收到数据。Producer可以通过acks参数指定最少需要多少个Replica确认收到该消息才视为该消息发送成功。acks的默认值是1，即Leader收到该消息后立即告诉Producer收到该消息，此时如果在ISR中的消息复制完该消息前Leader宕机，那该条消息会丢失。而如果将该值设置为0，则Producer发送完数据后，立即认为该数据发送成功，不作任何等待，而实际上该数据可能发送失败，并且Producer的Retry机制将不生效。

更推荐的做法是，将acks设置为all或者-1，此时只有ISR中的所有Replica都收到该数据（也即该消息被Commit），Leader才会告诉Producer该消息发送成功，从而保证不会有未知的数据丢失。

### 总结一下

设置acks=all，且副本数为3
极端情况1：
默认min.insync.replicas=1，极端情况下如果ISR中只有leader一个副本时满足min.insync.replicas=1这个条件，此时producer发送的数据只要leader同步成功就会返回响应，如果此时leader所在的broker crash了，就必定会丢失数据！这种情况不就和acks=1一样了！所以我们需要适当的加大min.insync.replicas的值。

极端情况2：
min.insync.replicas=3（等于副本数），这种情况下要一直保证ISR中有所有的副本，且producer发送数据要保证所有副本写入成功才能接收到响应！一旦有任何一个broker crash了，ISR里面最大就是2了，不满足min.insync.replicas=3，就不可能发送数据成功了！

根据这两个极端的情况可以看出min.insync.replicas的取值，是kafka系统可用性和数据可靠性的平衡！

减小 min.insync.replicas 的值，一定程度上增大了系统的可用性，允许kafka出现更多的副本broker crash并且服务正常运行；但是降低了数据可靠性，可能会丢数据（极端情况1）。
增大 min.insync.replicas 的值，一定程度上增大了数据的可靠性，允许一些broker crash掉，且不会丢失数据（只要再次选举的leader是从ISR中选举的就行）；但是降低了系统的可用性，会允许更少的broker crash（极端情况2）。

# **Producer源码解读**

在 Kafka 中, 我们把产生消息的一方称为 Producer 即 生产者, 它是 Kafka 的核心组件之一, 也是消息的来源所在。它的主要功能是将客户端的请求打包封装发送到 kafka 集群的某个 Topic 的某个分区上。那么这些生产者产生的消息是怎么传到 Kafka 服务端的呢？



## **Producer之整体流程**

我们回顾一下之前我们讲过Kafka一条消息发送和消费的流程

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1673448012083/8c79905d44964112bd4b9e42430a2fa7.png)

但是站在源码的核心角度，我们可以把Producer分成以下几个核心部分：

1、Producer之初始化

2、Producer之发送流程

3、Producer之缓冲区

4、Producer之参数与调优

### Kafaka发送&拉取消息基本流程：

![image-20240125213945565](E:\图灵课堂\MQ专题\MQ专题.assets\image-20240125213945565.png)



## **Producer源码解读**

从生产流程可以知道，Producer里面的核心有序列化器，分区器，还有缓冲，所以初始化的流程肯定是围绕这几个核心来处理。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1673448012083/9aa5781ed028438ea859ecc9504872ae.png)

### Kafka**Producer之初始化**

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1673448012083/1e91052f8e6a4284bcb19354950bbe4c.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1673448012083/95af24feedf243a9ace1f1043e3404a8.png)

因为源码中有非常多的一些额外处理，所以我们解读源码没必要每行都读，只需要根据我们之前梳理的主流程找到核心代码进行解读就可以，这也是推荐大家去初次解读源码的最优方式。

#### 1)、设置分区器

设置分区器(partitioner)，分区器是支持自定义的

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1673448012083/c5285c5cec7c45aea8bef0e01c24b188.png)

#### 2)、设置重试时间

设置重试时间(retryBackoffMs)默认100ms

如果发送消息到broker时抛出异常，且是允许重试的异常，那么就会最大重试retries参数指定的次数，同时retryBackoffMs是重试的间隔。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1673448012083/b0fb0a7f01a84f64a05ebe060bf041d8.png)

#### 3)、设置序列化器

设置序列化器(Serializer)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1673448012083/4f228a2bc03c4910b3fb0e86c9b24b50.png)

#### 4)、设置拦截器

设置拦截器(interceptors)，关于拦截器，这个后面会有讲解和介绍。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1673448012083/849e7e8dbb524a18b146c4f3b1639aee.png)

#### 5)、设置缓冲区

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1673448012083/e30cc9e0bfb341188a88ce0571bc492c.png)

在之前，还有一些参数的设置。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1673448012083/c842023a00a3424b9970516f49b4a030.png)

1、设置最大的消息为多大(maxRequestSize), 默认最大1M, 生产环境可以提高到10M

2、设置缓存大小(totalMemorySize) 默认是32M

3、设置压缩格式(compressionType)

4、初始化RecordAccumulator也就是缓冲区指定为32M

#### 6)、设置消息累加器

因为生产者是通过缓冲的方式发送，发送的条件之前的课程讲过，所以这里需要一个消息累加器配合才能完成消息的发送。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1673448012083/1966d8a5b4b8400aa63f69ce5577148e.png)

5、初始化集群元数据(metadata),刚开始空的

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1673448012083/844359ba9b7844bd92d66655d14d1ca0.png)

#### 6)、创建Sender线程

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1673448012083/51397864088a41bc8e23ac5fb6830dfd.png)

**这里还初始化了一个重要的管理网路的组件 NetworkClient**

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1673448012083/a1db6f1c2ea445b2b70173621a6ca6ae.png)

KafkaThread将Sender设置为守护线程并启动

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1673448012083/1ad55fed9c7e47eaafb6cb6ef8d7dffd.png)

#### 拦截器使用及介绍

这里讲一讲拦截器的使用和基本作用，拦截器一般用得不多，所以这里只是讲一讲案例，不推荐生产中使用。

想要实现拦截器，我们需要先实现ProducerInterceptor接口即可，然后在生产者中设置进去即可。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1673448012083/c45d820e0090463bbbc311b6af5dad6b.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1673448012083/a1c928dc2ea14517825f36afdbeeb358.png)

1、想要把发送的数据都带上时间戳![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1673448012083/7963041d455a42c499704ff3b732198f.png)

2、实现统计发送消息的成功次数和失败次数

在 `onAcknowledgement(RecordMetadata, Exception)`里面，根据消息发送后返回的异常信息来判断是否发送成功。一般异常如果为空就说明发送成功了，反之就说明发送失败了。

然后定义两个变量，并根据Exception的值分别累加就可以统计到了

最后在close方法里输出两个变量的值，这样当producer发送数据结束并close后，会自动调用拦截器的close方法来输出咱们想要统计的成功和失败次数

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1673448012083/4045cee3f9e34231a44d4e328bf2ede9.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1673448012083/00b4900d49dc4515a97dcd5ea0ea96a1.png)

不过这里要注意一个点：

**onAcknowledgement运行在producer的IO线程中，因此不要在该方法中放入很复杂的逻辑，否则会拖慢producer的消息发送效率。**

3、拦截链路

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1673448012083/0309be9ecb55419f80c04da04b331194.png)

拦截器链里的拦截器是按照顺序组成的，因此我们要注意前后拦截器对彼此的影响，比如这里拦截器1的onsend方法不能返回null，不然拦截器2的onsend就丢失了信息，会发生异常。

### Producer之发送流程

**Producer之发送流程**

**Kafka Producer 发送消息流程如下:**

#### 1)、执行拦截器逻辑

执行拦截器逻辑，预处理消息, 封装 Producer Record

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1673448012083/5fa3fad8c6ba46a096bae4865a01507c.png)

#### 2)、集群元数据

从 Kafka Broker 集群获取集群元数据metadata

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1673448012083/88cef73ae0ac4a7385f279e01f122af3.png)

#### 3)、序列化

调用Serializer.serialize()方法进行消息的key/value序列化

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1673448012083/19d75fb3d7974a22a0accec19ef38c3e.png)

#### 4)、分区

调用partition()选择合适的分区策略，给消息体 Producer Record 分配要发送的 topic 分区号

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1673448012083/03ce486dfd2741baa2e7a129ed6bfb26.png)

#### 5)、消息累加进缓存

将消息缓存到RecordAccumulator 收集器中, 最后判断是否要发送。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1673448012083/f3e6f52f6eef4386896e742d8dcceb65.png)

#### 7)、消息发送

前面我们也知道真正的消息发送是Sender线程来做，并且这里还要结合缓冲区来处理。后面会对这个进行详细的讲解，这里我们只需要知道发送的条件：

批次发送的条件为:缓冲区数据大小达到 batch.size 或者 linger.ms 达到上限，哪个先达到就算哪个

### Producer之缓冲区

Kafka生产者的缓冲区，也就是内存池，可以将其类比为连接池(DB, Redis),主要是避免不必要的创建连接的开销, 这样内存池可以对 RecordBatch 做到反复利用, 防止引起Full GC问题。那我们看看 Kafka 内存池是怎么设计的。

核心就是这段代码：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1673448012083/eccde941f3df42f18db3bbaa6dc07c9b.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1673448012083/87e44ff9bcae4bd2bed0538375fe9f1f.png)

```
   Kafka 内存设计有两部分，下面的粉色的是可用的内存（未分配的内存，初始的时候是 32M），上面紫色的是已经被分配了的内存，每个小 Batch 是 16K，然后这一个个的 Batch 就可以被反复利用，不需要每次都申请内存,  两部分加起来是 32M。
```

#### **申请内存的过程**

从 Producer 发送流程的第6步中可以看到会把消息放入 accumulator中, 即调用 accumulator.append() 追加, 然后把消息封装成一个个Batch 进行发送, 然后去申请内存(free.allocate())

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1673448012083/7848d12efec0431aa30c55d0b80ba5d5.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1673448012083/991ff49f67894a11a7e057912526d7cf.png)

（1）如果申请的内存大小超过了整个缓存池的大小，则抛异常出来

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1673448012083/fab2be01474c4ed4be493eddc600090f.png)

（2）对整个方法加锁：

```
this.lock.lock();
```

（3）如果申请的大小是每个 recordBatch 的大小（16K），并且已分配内存不为空，则直接取出来一个返回。

```
if (size == poolableSize && !this.free.isEmpty())
    return this.free.pollFirst();
```

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1673448012083/225f22f26e4646a9bf06ed39904e52ba.png)

（4）如果整个内存池大小比要申请的内存大小大 （this.availableMemory + freeListSize >= size），则直接从可用内存（即上图粉色的区域）申请一块内存。并且可用内存要去掉申请的那一块内存。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1673448012083/13493a6034d64c149e343f9e7cc551c0.png)

#### Sender线程

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1673448012083/23791d2314dd4c8188c3afb3c65c216d.png)

**Producer之参数调优**

```
     我们知道在 Kafka 实际使用中，Producer 端既要保证吞吐量，又要确保无消息丢失，一些核心参数的配置就显得至关重要。接下来我们就来看看生产端都有哪些重要的参数,及调优建议。
```

**acks**

参数说明：对于 Kafka Producer 来说是一个非常重要的参数，它表示指定分区中成功写入消息的副本数量，是 Kafka 生产端消息的持久性的保证, 详细可以查看

**max.request.size**

参数说明：这个参数对于 Kafka Producer 也比较重要， **表示生产端能够发送的最大消息大小，默认值为1048576(1M)** 。

```
  调优建议：这个配置对于生产环境来说有点小， **为了避免因消息过大导致发送失败，生产环境建议适当调大，比如可以调到10485760(10M)** 。
```

**retries**

参数说明：表示生产端消息发送失败时的重试次数，默认值为0，即不重试。 **这个参数一般是为了解决因系统瞬时故障导致的消息发送失败，比如网络抖动、Leader 选举及重选举，其中瞬时的 Leader 重选举是比较常见的。因此这个参数的设置对于 Kafka Producer 就显得非常重要** 。

```
 调优建议：这里建议设置为一个大于0的值，比如3次。
```

**retry.backoff.ms**

参数说明：**设定两次重试之间的时间间隔，避免无效的频繁重试，默认值为100, ****主要跟 retries 配合使用， **在配置 retries 和 retry.backoff.ms 之前，最好先估算一下可能的异常恢复时间，需要设定总的重试时间要大于异常恢复时间，避免生产者过早的放弃重试。

**connections.max.idele.ms**

参数说明：主要用来判断多久之后关闭空闲的链接，默认值540000（ms）即9分钟。

**compression.type**

参数说明： **该参数表示生产端是否要对消息进行压缩，默认值为不压缩(none)。** 压缩可以显著减少网络IO传输、磁盘IO以及磁盘空间，从而提升整体吞吐量，但也是以牺牲CPU开销为代价的。

```
 调优建议：出于提升吞吐量的考虑，建议在生产端对消息进行压缩。**对于Kafka来说，综合考虑吞吐量与压缩比，建议选择lz4压缩。如果追求最高的压缩比则推荐zstd压缩。**
```

**buffer.memory**

参数说明： **该参数表示生产端消息缓冲池或缓冲区的大小，默认值为即33554432(32M)** 。这个参数基本可以认为是 Producer 程序所使用的内存大小。

```
调优建议：通常我们应尽量保证生产端整体吞吐量，建议适当调大该参数，也意味着生产客户端会占用更多的内存。
```

**batch.size**

参数说明： **该参数表示发送到缓冲区中的消息会被封装成一个一个的Batch，分批次的发送到 Broker 端，默认值为16KB。** 因此减小 batch 大小有利于降低消息延时，增加 batch 大小有利于提升吞吐量。

```
 调优建议：通常合理调大该参数值，能够显著提升生产端吞吐量，比如可以调整到32KB，调大也意味着消息会有相对较大的延时。
```

**linger.ms**

参数说明： **该参数表示用来控制 Batch 最大的空闲时间，超过该时间的 Batch 也会自动被发送到 Broker 端。** 实际情况中, 这是吞吐量与延时之间的权衡。默认值为0，表示消息需要被立即发送，无需关系 batch 是否被填满。

```
  调优建议：通常为了减少请求次数、提升整体吞吐量，建议设置一个大于0的值，比如设置为100，此时会在负载低的情况下带来100ms的延时。  
```

### 核心方法doSend

```java
return this.doSend(producerRecord);

  rotected ListenableFuture<SendResult<K, V>> doSend(ProducerRecord<K, V> producerRecord) {
    //先判断是否事务性的 默认false
     if (this.transactional) {
        Assert.state(this.inTransaction(), "No transaction is in process; possible solutions: run the template operation within the scope of a template.executeInTransaction() operation, start a transaction with @Transactional before invoking the template method, run in a transaction started by a listener container when consuming a record");
    }
    //拿到Producer 发送者  进入可见是放到一个 ThreadLocal 里面的如果没有肯定是会去创建 Alt+回退到这里
    Producer<K, V> producer = this.getTheProducer();
   

//拿到Producer生产者
private Producer<K, V> getTheProducer() {
    if (this.transactional) {
        Producer<K, V> producer = (Producer)this.producers.get();
        if (producer != null) {
            return producer;
        } else {
            KafkaResourceHolder<K, V> holder = ProducerFactoryUtils.getTransactionalResourceHolder(this.producerFactory, this.transactionIdPrefix, this.closeTimeout);
            return holder.getProducer();
        }
    } else {
        //创建Producer 的方法
        return this.producerFactory.createProducer(this.transactionIdPrefix);
    }
}

//定位到 createProducer的实现 ，可以看到这是一个工厂 DefaultKafkaProducerFactory
public Producer<K, V> createProducer(@Nullable String txIdPrefixArg) {
    else {
        //这个时候它肯定为空
        if (this.producer == null) {
            synchronized(this) {
                if (this.producer == null) {
                    //进入此CloseSafeProducer 可见他会有一个 delegate /ˈdelɪɡət/ 的委派 那么这个委派是去干嘛呢 这里可见是 创建一个KafkaProducer
                    //有了这个kafkaProducer之后我们回到前面的 发送方法
                    this.producer = new DefaultKafkaProducerFactory.CloseSafeProducer(this.createKafkaProducer());
                }
            }
        }
```

Producer的send方法可见有三个实现   (首先是CloseSafeProducer 是 DefaultKafkaProducerFactory的)

而这个工厂刚才看过是通过 委派 创建了KafkaProducer 来发送的  

![image-20240125222532328](E:\图灵课堂\MQ专题\MQ专题.assets\image-20240125222532328.png)

可见第一个 里面实际上用的也是delegate 来发送的 return this.delegate.send(record, callback);

重点关注刚才 的KafkaProducer  的send方法 。 这里的代码跟我们的mybatis的二级缓存 相似

最终到核心发送doSend方法

```java
public Future<RecordMetadata> send(ProducerRecord<K, V> record, Callback callback) {
    //send前先经过拦截器定制化消息的拦截链(
//1.0 引入 发送前的定制化需求。   比如分类统计  修改消息 异常监测 数据加密 字段过滤等 可多个拦截链
//可实现ProducerInterceptor 接口 重写onSend方法定制化消息)
    ProducerRecord<K, V> interceptedRecord = this.interceptors.onSend(record);
    //接下来进入 真正的发送方法
    return this.doSend(interceptedRecord, callback);
}


private Future<RecordMetadata> doSend(ProducerRecord<K, V> record, Callback callback) {
    TopicPartition tp = null;
    try {
        this.throwIfProducerClosed();
        //1.首先确保该主题topic对应的元数据metadata可用
        KafkaProducer.ClusterAndWaitTime clusterAndWaitTime= this.waitOnMetadata(record.topic(), record.partition(), this.maxBlockTimeMs);
        //2.计算剩余等待时间 
        long remainingWaitMs = Math.max(0L, this.maxBlockTimeMs - clusterAndWaitTime.waitedOnMetadataMs);
        Cluster cluster = clusterAndWaitTime.cluster;
        //3.得到序列化 key:serializedKey  。根据record中topic ，key 利用 valueSerializer得到
        byte[] serializedKey=this.keySerializer.serialize(record.topic(), record.headers(), record.key());
        //4.得到序列化 value:serializedValue   。    根据record中topic ，value 利用 valueSerializer得到
        byte[] serializedValue = this.valueSerializer.serialize(record.topic(), record.headers(), record.value());
        //5.调用partition方法 获得分区号 这个分区可传入也可auto
        int partition = this.partition(record, serializedKey, serializedValue, cluster);
        //6.根据record中的topic和partition构造TopicPartition实例tp
        TopicPartition  tp = new TopicPartition(record.topic(), partition);
        this.setReadOnly(record.headers());
        Header[] headers = record.headers().toArray();
        //7.计算序列化后的 key value及offset size所占大小
        int serializedSize = AbstractRecords.estimateSizeInBytesUpperBound(this.apiVersions.maxUsableProduceMagic(), this.compressionType, serializedKey, serializedValue, headers);
        //8.确保记录大小是否有效
        this.ensureValidRecordSize(serializedSize);
        long timestamp = record.timestamp() == null ? this.time.milliseconds() : record.timestamp();
        this.log.trace("Sending record {} with callback {} to topic {} partition {}", new Object[]{record, callback, record.topic(), partition});
        Callback interceptCallback = new KafkaProducer.InterceptorCallback(callback, this.interceptors, tp);
        if (this.transactionManager != null && this.transactionManager.isTransactional()) {
            this.transactionManager.maybeAddPartitionToTransaction(tp);
        }
        //9.调用append方法添加记录，获得记录添加结果
        RecordAppendResult result = this.accumulator.append(tp, timestamp, serializedKey, serializedValue, headers, interceptCallback, remainingWaitMs);
        //10.根据结果状态判断是否需要 wakeup
        if (result.batchIsFull || result.newBatchCreated) {
            this.log.trace("Waking up the sender since topic {} partition {} is either full or getting a new batch", record.topic(), partition);
            this.sender.wakeup();
        }
        //11.返回
        return result.future;
    } catch 
```

```java
KafkaProducer(Map<String, Object> configs, Serializer<K> keySerializer, Serializer<V> valueSerializer, ProducerMetadata metadata, KafkaClient kafkaClient, ProducerInterceptors interceptors, Time time) {
    ProducerConfig config = new ProducerConfig(ProducerConfig.addSerializerToConfig(configs, keySerializer, valueSerializer));
    //可见前面判断参数  最后catch异常中间 开启 Sender线程 发送消息
    this.errors = this.metrics.sensor("errors");
     // 关键点在这个  Sender线程
        this.sender = this.newSender(logContext, kafkaClient, this.metadata);
        String ioThreadName = "kafka-producer-network-thread | " + clientId;
        this.ioThread = new KafkaThread(ioThreadName, this.sender, true);
        this.ioThread.start();
        config.logUnused();
        AppInfoParser.registerAppInfo("kafka.producer", clientId, this.metrics, time.milliseconds());
        this.log.debug("Kafka producer started");
        
        
       //再往里面就是 Sender线程  的run 方法 调用的 runOnce方法  最后
       //将需要发送的消息发送给client的待发数据中  （进入可见 判断哪些topic partition 可发 批次）
        long pollTimeout = this.sendProducerData(currentTimeMs);
        //真正NIO发送 Poll 网络模型发送数据
        this.client.poll(pollTimeout, currentTimeMs);
```

最终  交由 NetworkClient  的 poll方法发送     此处就是网络 IO模型的知识了（详见网络通信专题） 

#### 拓展知识点：

自定义选择分区规则：

```java
/**
 * 自定义kafka分区主要解决用户分区数据倾斜问题 提高并发效率（假设 3 分区）
 * @param topic 消息队列名
 * @param key 用户传入key
 * @param keyBytes key字节数组
 * @param value 用户传入value
 * @param valueBytes value字节数据
 * @param cluster 当前kafka节点数
 * @return 如果3个节点数 返回 0 1 2 如果5个 返回 0 1 2 3 4 5
 当同一个key的消息会被分配到同一个partition中。消息在同一个partition处理的顺序是FIFO，这就保证了消息的顺序性
 */
public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
    // 得到 topic 的 partitions 信息
    List<PartitionInfo> partitions = cluster.partitionsForTopic(topic);
    //获取分区总个数
    int numPartitions = partitions.size();
    //如果指定的key数组不为空
    if (keyBytes == null) {
        int nextValue = this.nextValue(topic);
        List<PartitionInfo> availablePartitions = cluster.availablePartitionsForTopic(topic);
        //可用分区不为空
        if (availablePartitions.size() > 0) {
            //将消息分配给下一个分区 通过murmur /ˈmɜːrmər/ 的hash算法计算分区分配
            int part = Utils.toPositive(nextValue) % availablePartitions.size();
            return ((PartitionInfo)availablePartitions.get(part)).partition();
            //可用分区为空
        } else {
            //如果指定了key ，对key做hash 分配到指定的分区
            return Utils.toPositive(nextValue) % numPartitions;
        }
    } else {
        //设置了key 则直接给key取模计算分区
        return Utils.toPositive(Utils.murmur2(keyBytes)) % numPartitions;
    }

} 
```

然后在properties配置 partitioner.class 即可

注意：自定义分区应用于分布式项目 需要将分区类打包放到lib包 Conusumer通过partition.assignment.strategy=指定

我们回到默认的 DefaultPartitioner.partition方法 看看它是如何发送的。

如果设置了key 会怎么去发送？

我们回顾下下原来的：如果key为空会根据给的key取模也就是指定到某个分区  ，不为空的话会拿到可用分区去轮循到不同的分区

完全可以根据需要加入不同的算法选择不同的分区

### 发送流程图：



# **Consumer源码解读**

**本课程的核心技术点如下：**

1、consumer初始化
2、如何选举Consumer Leader
3、Consumer Leader是如何制定分区方案

4、Consumer如何拉取数据
5、Consumer的自动偏移量提交

### 源码分析：

消费者 comsumer 会在一个 group 里面，然后消费一个topic

怎么指定的呢 @KafkaListener(topics = "tianmingtest",groupId = "tulinggroup")

```java
/**
 * 消费者
 */
@Component
public class ConsumerListener {
    //指定订阅的 topic 以及归属的group
    @KafkaListener(topics = "tianmingtest",groupId = "tulinggroup")
    public void onMessage(String msg){
        System.out.println("----收到消息：" + msg + "----");
    }
}
```

KafkaListenerAnnotationBeanPostProcessor

 实现了 BeanPostProcessor 接口（讲Spring的时候还有印象没  ？ Bean生命周期   初始化Bean前后的回调的两个方法），

主要开始流程在 postProcessAfterInitialization  后置处理器中：对象实例化之后调用

 （初始化回调之前的方法before先不用管）

```java
public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        if (!this.nonAnnotatedClasses.contains(bean.getClass())) {
        Class targetClass = AopUtils.getTargetClass(bean);
        //是否有类添加KafkaListener注解
        Collection  classLevelListeners =  this.findListenerAnnotations(targetClass);
        boolean hasClassLevelListeners = classLevelListeners.size() > 0;
        List<Method> multiMethods = new ArrayList();
        //是否有方法添加 KafkaListener 注解，并与方法建立隐射关系
        Map<Method, Set<KafkaListener>> annotatedMethods = MethodIntrospector.selectMethods(targetClass, (methodx) -> {
            Set<KafkaListener> listenerMethods = this.findListenerAnnotations(methodx);
            return !listenerMethods.isEmpty() ? listenerMethods : null;
        });
        //如果KafkaListener在类级别上
        if (hasClassLevelListeners) {
            //去找 KafkaHandler的方法注解 （如果类上被 KafkaListener修饰，那么类方法必须要使用 KafkaHandle配合实现）
            Set<Method> methodsWithHandler = MethodIntrospector.selectMethods(targetClass, (methodx) -> {
                return AnnotationUtils.findAnnotation(methodx, KafkaHandler.class) != null;
            });
            multiMethods.addAll(methodsWithHandler);
        
        } else {
            Iterator var13 = annotatedMethods.entrySet().iterator();
            
            // 遍历annotatedMethods  根据隐射关系执行 processKaListener
            while(true) {
                if (!var13.hasNext()) {
                    this.logger.debug(() -> {
                        return annotatedMethods.size() + " @KafkaListener methods processed on bean '" + beanName + "': " + annotatedMethods;
                    });
                    break;
                }
                 Entry<Method, Set<KafkaListener>> entry = (Entry)var13.next();
                    Method method = (Method)entry.getKey();
                    Iterator var11 = ((Set)entry.getValue()).iterator();
            
                    while(var11.hasNext()) {
                        KafkaListener listener = (KafkaListener)var11.next();
                        // 注解在方法上的逻辑 
                        this.processKafkaListener(listener, method, bean, beanName);
                    }
                }
            }
                // 处理类级别的 ClassLevelListeners 和 multiMethods隐射关系
                  if (hasClassLevelListeners) {
                    this.processMultiMethodListeners(classLevelListeners, multiMethods, bean, beanName);
                }
            }
            
            return bean;
```

进入 processKafListener 方法 可见 设置 endpoint 信息    和 this.processListener方法   进入可见    registerEndpoint 方法 。 

进入 registerEndpoint 方法可见   startImmediately  默认false 所以   将信息add 到 List< KafakaListenerEndpointDescriptor>endpointDescriptors  List集合中

真正创建MessageListenerContainer的地方是在KafkaListenerEndpointRegistry中，因为 最初的  

KafkaListenerAnnotationBeanPostProcessor类的  单列实例化 之后  调用的方法afterSingletonsInstantiated   最后

  this.registrar.afterPropertiesSet(); 里面 调用

的 registerAllEndpoints()

最终调用到了KafkaListenerEndpointRegistrar的  registerListenerContainer

此方法

```java
public void registerListenerContainer(KafkaListenerEndpoint endpoint, KafkaListenerContainerFactory<?> factory, boolean startImmediately) {
    Assert.notNull(endpoint, "Endpoint must not be null");
    Assert.notNull(factory, "Factory must not be null");
    String id = endpoint.getId();
    Assert.hasText(id, "Endpoint id must not be empty");
    synchronized(this.listenerContainers) {
        Assert.state(!this.listenerContainers.containsKey(id), "Another endpoint is already registered with id '" + id + "'");
        //创建 MessageListenerContainer 实例
        MessageListenerContainer container = this.createListenerContainer(endpoint, factory);
        this.listenerContainers.put(id, container);
        ConfigurableApplicationContext appContext = this.applicationContext;
        String groupName = endpoint.getGroup();
        if (StringUtils.hasText(groupName) && appContext != null) {
            Object containerGroup;
            ContainerGroup group;
            if (appContext.containsBean(groupName)) {
                containerGroup = (List)appContext.getBean(groupName, List.class);
                group = (ContainerGroup)appContext.getBean(groupName + ".group", ContainerGroup.class);
            } else {
                containerGroup = new ArrayList();
                appContext.getBeanFactory().registerSingleton(groupName, containerGroup);
                group = new ContainerGroup(groupName);
                appContext.getBeanFactory().registerSingleton(groupName + ".group", group);
            }
            // 加入分组
            ((List)containerGroup).add(container);
            group.addContainers(new MessageListenerContainer[]{container});
        }
        // 默认false  所以启动时 暂时不会执行start方法，会先交给 SmartLifecycle 接口处理
        // 这里KafkaListenerEndpointRegistry类实现了此接口，会在项目所有bean加载和初始化完毕之后执行start方法
        if (startImmediately) {
            this.startIfNecessary(container);
        }

    }
}
```

进入 this.startIfNecessay 实际用的参数的start方法  再进入 方法参数 MessageListenerContainer  接口的实现类   AbstractMessageListenerContainer 的 start 方法 

发现是一个abstract的  doStart 方法  ，说明实现是交给它的子类去实现

我们来到 它的子类  KafkaMessageListenerContainer  的 doStart 方法

```java
protected void doStart() {
    if (!this.isRunning()) {
        if (this.clientIdSuffix == null) {
            this.checkTopics();
        }

        ContainerProperties containerProperties = this.getContainerProperties();
        this.checkAckMode(containerProperties);
        Object messageListener = containerProperties.getMessageListener();
        AsyncListenableTaskExecutor consumerExecutor = containerProperties.getConsumerTaskExecutor();
        if (consumerExecutor == null) {
            consumerExecutor = new SimpleAsyncTaskExecutor((this.getBeanName() == null ? "" : this.getBeanName()) + "-C-");
            containerProperties.setConsumerTaskExecutor((AsyncListenableTaskExecutor)consumerExecutor);
        }

        GenericMessageListener<?> listener = (GenericMessageListener)messageListener;
        ListenerType listenerType = this.determineListenerType(listener);
        // 实例化一个listenerConsumer.
        // 进入这个属性 listenerConsumer可见实现了 SchedulingAwareRunnable 说明是个线程 肯定 runabler接口 run方法 这个我们一会进去瞧瞧
        this.listenerConsumer = new KafkaMessageListenerContainer.ListenerConsumer(listener, listenerType);
        this.setRunning(true);
        this.startLatch = new CountDownLatch(1);
        //将 listenerConsumer 添加到线程池 Executor
        this.listenerConsumerFuture = ((AsyncListenableTaskExecutor)consumerExecutor).submitListenable(this.listenerConsumer);
        try {
            if (!this.startLatch.await(containerProperties.getConsumerStartTimeout().toMillis(), TimeUnit.MILLISECONDS)) {
                this.logger.error("Consumer thread failed to start - does the configured task executor have enough threads to support all containers and concurrency?");
                 this.publishConsumerFailedToStart();
            }
        } catch (InterruptedException var7) {
            Thread.currentThread().interrupt();
        }

    }
}
```

来到 listenerConsumer 的run方法

这个里面我们能 最终看到 消费端获取消息采用的 poll方式 ，一次性拿多条数据

```java
public void run() {
    ListenerUtils.setLogOnlyMetadata(this.containerProperties.isOnlyLogRecordMetadata());
    KafkaMessageListenerContainer.this.publishConsumerStartingEvent();
    this.consumerThread = Thread.currentThread();
    this.setupSeeks();
    KafkaUtils.setConsumerGroupId(this.consumerGroupId);
    this.count = 0;
    this.last = System.currentTimeMillis();
    this.initAssignedPartitions();
    KafkaMessageListenerContainer.this.publishConsumerStartedEvent();
    Object exitThrowable = null;
    //循环调用
    while(KafkaMessageListenerContainer.this.isRunning()) {
        try {
            // poll 拉取数据  ， 消费端获取消息采用的 poll方式 ，一次性拿多条数据 
            // 进入 可见封装成 ConsumerRecords 消息对象
            this.pollAndInvoke();
        } catch (NoOffsetForPartitionException var12) {
     ....
            this.logger.error(var16, "Stopping container due to an Error");
            this.wrapUp(var16);
            throw var16;
        } catch (Exception var17) {
            this.handleConsumerException(var17);
        } finally {
            this.clearThreadState();
        }
    }

    this.wrapUp((Throwable)exitThrowable);
}
```

进入 pollAdnInvoke 方法   最后  invokeIfHaveRecords > invokeListener(records)  >  判断是否批量消费  invokeBatchListener / invokeRecordListener  

可以配置参数：concurrency 控制listener 的线程数量  ，并发开关可以通过batchListener = true 开启  配合max_poll_records_config=50 多少条一次poll返回  也可以配置间隔时间tnterval.ms 间隔多久poll一次  最多多少条

\# 如果有 两个方法标记 @KafkaListener 会启动 2 * 3 个Consuumer线程 ，6个Listener线程

 #spring.kafka.listener.concurrency=3

可以测试：

![image-20240125230724650](E:\图灵课堂\MQ专题\MQ专题.assets\image-20240125230724650.png)

默认 doInvokeWithRecords  方法

   是否开启事务模板   invokeRecordListenerInTx  和 doInvokeWithRecords

   默认  >   doInvokeRecordListener   > invokeOnMessage   > doInvokeOnMessage  

四种type 类型的listener  . 提供不同的接口处理

 1。 Simple 不考虑提交偏移量和consumer对象  ； 

2。Acknowledgine 需要手动提交时 而不是自动提交或者 spring-kafka自己实现提交方式时，需要如下接口中acknowledment 的 acknowlegge()方法提交偏移量 

3.consumer_aware 类似SpringIOC的 ApplicationContextAware功能，如果消费消息时，需要用consumer对象，则使用这个类型

4.ACKNOWLEDGING_CONSUMER_AWARE，同时支持ACKNOWLEDGING和CONSUMER_AWARE两种类型

这些类型由 接口GenericMessageListener接口具体哪个实现类来决定 

\> onMessage 返回消息

## Consumer初始化

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/439273e13ce94cc49f0d7e29bc356b08.png)

从KafkaConsumer的构造方法出发，我们跟踪到核心实现方法

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/609b80b7970a41be9629af99225b20d7.png)

这个方法的前面代码部分都是一些配置，我们分析源码要抓核心，我把核心代码给摘出来

### **NetworkClient**

**Consumer与Broker的核心通讯组件**

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/245206b57c7f40049332a97358f3b2f5.png)

### **ConsumerCoordinator**

**协调器，在Kafka消费中是组消费，协调器在具体进行消费之前要做很多的组织协调工作。**

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/af362192f2634f11a450acbac20d17bc.png)

### Fetcher

提取器，因为Kafka消费是拉数据的，所以这个Fetcher就是拉取数据的核心类

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/98a3f9fb7bec42ed8e1fea15babb8b34.png)

而在这个核心类中，我们发现有很多很多的参数设置，这些就跟我们平时进行消费的时候配置有关系了，这里我们挑一些核心重点参数来讲一讲

#### fetch.min.bytes

每次fetch请求时，server应该返回的最小字节数。如果没有足够的数据返回，请求会等待，直到足够的数据才会返回。缺省为1个字节。多消费者下，可以设大这个值，以降低broker的工作负载。

#### fetch.max.bytes

每次fetch请求时，server应该返回的最大字节数。这个参数决定了可以成功消费到的最大数据。

比如这个参数设置的是50M，那么consumer能成功消费50M以下的数据，但是最终会卡在消费大于10M的数据上无限重试。fetch.max.bytes一定要设置到大于等于最大单条数据的大小才行。

默认是50M

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/3553f7b44045496fbd072e4e8da6a273.png)

#### fetch.wait.max.ms

如果没有足够的数据能够满足fetch.min.bytes，则此项配置是指在应答fetch请求之前，server会阻塞的最大时间。缺省为500个毫秒。和上面的fetch.min.bytes结合起来，要么满足数据的大小，要么满足时间，就看哪个条件先满足。

这里说一下参数的默认值如何去找：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/735734c77a83480a82c504fd0aaff1a5.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/fdb8f426d5c74a04a98ba8d74a32f768.png)

#### max.partition.fetch.bytes

指定了服务器从每个分区里返回给消费者的最大字节数，默认1MB。

假设一个主题有20个分区和5个消费者，那么每个消费者至少要有4MB的可用内存来接收记录，而且一旦有消费者崩溃，这个内存还需更大。注意，这个参数要比服务器的message.max.bytes更大，否则消费者可能无法读取消息。

*备注：1、Kafka入门笔记*

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/149b38d2fdb744afb7342ab18a919b5c.png)

#### max.poll.records

控制每次poll方法返回的最大记录数量。

默认是500

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/14d8ab011f9e46f7b2ff35e3d0fa0e9a.png)



### **重复消费问题**

**首先要明白由于网络问题，重复消费不可避免**

可能原因：

1.消费线程中断导致消费后的数据offset没有提交

2.消费消息后offset提交时partition中断

3.消费超时或者调整partition重新reblance重平衡时

最简单的处理：

​				1：减少每次拉取的消息记录数 spring.kafka.consumer.max-poll-records=100  和增大poll之间的时间间隔 max.**poll**.interval.ms为（旧的版本）新  spring.kafka.listener.poll-timeout=50

　　　　2：拉取到消息之后异步处理（保证成功消费） spring.kafka.consumer.enable-auto-commit=false   注意这个要在producer设置

​				 3：消费者可以实现消费幂等性。应用程序应设计为幂等的，即使在消息重复消费的情况下，也能保持正确性。

（一次和多次请求某个资源对于资源本身应该有相同的结果，可以插入数据库之前在redis中存一个id操作前查看id是否存在，不存在才插入数据库，如果还不放心可以在数据库记录并判断这个唯一标志id）

这个幂等性的问题确实需要业务方自己判断，阿里的RocketMQ来存在也是这样子的，它为了保证消息的稳定性和高可用性，它会在不确定的情况下，比如网络抖动的情况下，即使有ack，有也可能会重复发送同一条消息。

​				4：可以通过在应用程序中维护一个已处理消息的记录，或者使用外部存储（如数据库）来实现消息的去重。这样，在消费消息前，可以先检查消息是否已被处理过，如果是，则跳过该消息。

如果还不够，也提供了 Low-level API  自己去维护offset和partition 

​				5：Kafka消息消费有两个consumer接口，Low-level API和High-level API：

Low-level API：消费者自己维护offset等值，可以实现对Kafka的完全控制；（名字叫低级别但是 实际不简单需要跟踪offset，指定topic partition 以及处理broker变动）

大致流程：可见后面源码的过程 ：获取topic partition   以及备份broker 情况，监听请求 ，处理消息，记录offset等，处理变更等

High-level API：封装了对parition和offset的管理，使用简单；

如果使用高级接口High-level API，可能存在一个问题就是当消息消费者从集群中把消息取出来、并提交了新的消息offset值后，还没来得及消费就挂掉了，那么下次再消费时之前没消费成功的消息就“诡异”的消失了；

### Low Level Consumer API说明

Low Level Consumer API控制灵活性高，但使用也相对复杂。
Low Level Consumer API，作为底层的Consumer API，提供了消费Kafka Message更大的控制，如：

- Read a message multiple times(重复读取）
- Consume only a subset of the partitions in a topic in a process（跳读）
- Manage transactions to make sure a message is processed once and only once（消息事务处理）

Low Level Consumer API提供更大灵活控制是以复杂性为代价的：

- Offset不再透明
- Broker自动失败转移需要处理
- 增加Consumer、Partition、Broker需要自己做负载均衡

**注意如下：**

- You must keep track of the offsets in your application to know where you left off consuming.（在自己的应用中管理Offset）
- You must figure out which Broker is the lead Broker for a topic and partition(如果一个Partition有多个副本，那么Lead Partition所在的Broker就称为这个Partition的Lead Broker，得自己判断)
- You must handle Broker leader changes（Broker Leader变更也得自己来）

**使用Low Level Consumer API的步骤**

- Find an active Broker and find out which Broker is the leader for your topic and partition（找到一个活跃的broker，并且找出当前topic和partition的leader broker）
- Determine who the replica Brokers are for your topic and partition（要决定哪些broker是当前topic和partition的leader broker的副本备份）
- Build the request defining what data you are interested in（创建请求数据说明）
- Fetch the data（拉取数据）
- Identify and recover from leader changes（在leader broker变更后，重新标识和恢复）。

### 流程总结：

​	1.入口KafkaListenerAnnotationBeanPostProcessor，实现了 BeanPostProcessor等接口，所以Bean加载完后会进入postProcessAfterInitialization方法 

​	2.找到有@KafkaListener注解的类或者方法，并且将注解的信息封装成MethodKafkaListenerEndpoint 进入processListener中registerEndpoint方法，将数据封装到 KafkaListenerEndpointDescriptor，并且添加到List

​	3.KafkaListenerAnnotationBeanPostProcessor 实现了SmartInitializingSingleton，在当所有单例 bean 都初始化完成以后，会调 用afterSingletonsInstantiated方法

 	4.在afterSingletonsInstantiated方法中调用 registerListenerContainer方法，将消费者信息封装到 listenerContainers中 				      5.KafkaListenerEndpointRegistry类实现了SmartLifecycle接口，所以会容器启动调用start方法 

6.进入AbstractMessageListenerContainer的start方法 发现静态方法交给子类实现 

7.进入KafkaMessageListenerContainer的doStart方法 

8.实例化一个listenerConsumer，listenerConsumer实现了SchedulingAwareRunnable。SchedulingAwareRunnable继承了runnable，并将listenerConsumer添加至线程池

9.进入listenerConsumer的run方法，循环调用this.pollAndInvoke();

 10.pollAndInvoke中调用this.doPoll()；获取信息



### 消费端源码流程图：





## 如何选举Consumer Leader

回顾之前的内容

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/d4d5010160d64359835601bd11ac1721.png)

那么如何完成以上的逻辑的，我们跟踪代码：

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/d860e57c56084f7089c68b1b96b7f086.png)

### 1、消费者协调器与组协调器的通讯

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/dd7f9ef84d0b4b40803df2666799ef1c.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/1245275c5c06426587f22ec9b5b173cc.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/9fdccd7c773a432b95b6b4330e84ef81.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/18c5f2bfe54a411b898836c3e0b939bc.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/cc02631ecf8645e1b7166b708ed61687.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/b7d3f1bebe81418aa3f25ec2fbb82919.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/f2b7b514701749bcb100917863231961.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/6454beef06114ecd80bc22fb294eff82.png)

对Broker的响应进行处理

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/399824c22ffd4e2295bd2c338be5b146.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/c48b224ebd6e44908af948c6befb67fb.png)

### 1、消费者协调器发起入组请求

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/d266f57732d94d2288f76e2d16d6cfce.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/42c86951d77d4a86a87726567983088a.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/37153d3e469a4190a7b53538fb6c6b75.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/5b3b8fa273584a81bdb66c66ba3d7e54.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/f4d91abfa5c74cf2ac696a9eb35c7552.png)

## Consumer Leader如何制定分区方案

回顾之前的内容

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/30efcaccb4314670b776e7b7545c0d8f.png)

### 消费者分区策略

可以通过消费者参数

**partition.assignment.strategy**

设置分区分配给消费者的策略。默认为Range。允许自定义策略。

#### **Range（范围）**

把主题的连续分区分配给消费者。（如果分区数量无法被消费者整除、第一个消费者会分到更多分区）

**对每个Topic进行独立的分区分配**，首先对分区按照分区ID进行排序，然后订阅这个Topic的消费组的消费者再进行排序，尽量均衡地将分区分配给消费者。这里的“尽量均衡”是因为分区数可能无法被消费者数量整除，导致某些消费者可能会多分配到一些分区

它的特点是以topic为主进行划分的，通过partition数/consumer数来决定每个消费者消费几个分区。如果有余则交给消费者1

假设消费者数量为N，主题分区数量为M，则有当前主题分配数量 = M%N==0? M/N +1 : M/N ;

简单来说就是将主题中的分区除以group中订阅此主题的消费者，除数有余则一号多分配。

![img](https://img-blog.csdnimg.cn/56eb09efa9a64537acad4444d54ad759.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5b-16Iqx5byAXw==,size_20,color_FFFFFF,t_70,g_se,x_16)

 Range策略的缺点在于如果Topic足够多、且分区数量不能被平均分配时，会出现消费过载的情景，举一个例子

![img](https://img-blog.csdnimg.cn/e2676ccf42be417fa57dce8f3cefe001.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5b-16Iqx5byAXw==,size_20,color_FFFFFF,t_70,g_se,x_16)

可以看到此种情况已经相差3个分区，如果主题进一步扩大差距会愈发明显。

#### **RoundRobin（轮询）**

把主题的分区循环分配给消费者。

一种轮询式的分配策略，即每个人都会得到一个分区，顺序取决于他们注册时的顺序。这有助于确保所有消费者都能访问到所有数据。

简单来说就是把所有partition和所有consumer列出来，然后按照hashcode排序，最后进行轮询算法分配。

如果主题中分区不一样的时候如下：

![img](https://img-blog.csdnimg.cn/83a19428ba554b8aba3aa87d297ce26e.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5b-16Iqx5byAXw==,size_20,color_FFFFFF,t_70,g_se,x_16)

不难看出轮询策略是将partition当做最小分配单位，将所有topic的partition都看作一个整体。然后为消费者轮询分配partition。当然得到此结果的前提是Consumer Group种的消费者订阅信息是一致的，如果订阅信息不一致，得到的结果也不均匀，下面举个例子：

![img](https://img-blog.csdnimg.cn/0df41ae307f241a4882efdda148ad8aa.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5b-16Iqx5byAXw==,size_20,color_FFFFFF,t_70,g_se,x_16)

如图，Consumer0订阅Topic-A、B，Consumer1订阅Topic-B、C

顺序注意图中的Seq，先分配TopicA

第一轮 : Consumer-0: Topic-A-Partition0

由于Consumer-1没有订阅Topic-A，所以只能找到Topic-B给Consumer-1分配

于是 Consumer-1: Topic-B-Partition0

------------------------------------------------------------------------------------------------------

第二轮: Consumer-0: Topic-A-Partition0,**Topic-A-Partition1**

​       Consumer-1: Topic-B-Partition0,**Topic-B-Partition1**

------------------------------------------------------------------------------------------------------

第三轮: Consumer-0: Topic-A-Partition0,Topic-A-Partition1，**Topic-A-Partition2**

​         Consumer-1: Topic-B-Partition0,Topic-B-Partition1，**Topic-B-Partition2**

------------------------------------------------------------------------------------------------------

第四、五、六轮：

Consumer-0: Topic-A-Partition0,Topic-A-Partition1，Topic-A-Partition2

Consumer-1: Topic-B-Partition0,Topic-B-Partition1，Topic-B-Partition2,**Topic-C-Partition-0,Topic-C-Partition-1,Topic-C-Partition-2**

可以看到Consumer-1多消费了3个分区。所以在Consumer Group有订阅消息不一致的情况下，我们尽量不要选用RoundRobin。



注意：上面介绍的两种分区分配方式,多多少少都会有一些分配上的偏差, 而且每次**重新分配**的时候都是把所有的都**重新来计算并分配**一遍, 那么每次分配的结果都会偏差很多, 如果我们在计算的时候能够考虑上一次的分配情况,来尽量的减少分配的变动,这样我们将尽可能地撤销更少的分区，因为撤销过程是昂贵的

#### StickyAssignor(粘性)

粘性分区：每一次分配变更相对上一次分配做最少的变动.

当某个消费者的某个分区出现故障或不可用时，它会尝试保留它已经分配但尚未处理的分区。如果其他消费者也出现了问题，则会尽力维持原先的分区分配。这意味着一旦某个分区被分配给了某个消费者，除非该消费者退出或者分区不可用，否则不会重新分配给其他消费者。

目标：

1、**分区的分配尽量的均衡，分配给消费者者的主题分区数最多相差一个；**

2、**每一次重分配的结果尽量与上一次分配结果保持一致**

当这两个目标发生冲突时，优先保证第一个目标

首先, **StickyAssignor粘性分区**在进行分配的时候,是以**RoundRobin的分配逻辑来计算的,但是它又弥补了RoundRobinAssignor的一些可能造成不均衡的弊端。

比如在讲RoundRobin弊端的那种case, 但是在StickyAssignor中就是下图的分配情况

把RoundRobinAssignor的弊端给优化了

![img](https://img-blog.csdnimg.cn/a0dbfc41392c4e41be8dc8a3294bffaa.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5b-16Iqx5byAXw==,size_20,color_FFFFFF,t_70,g_se,x_16)

 此时的结果明显非常不均衡，如果使用Sticky策略的话结果应该是如此：

![img](https://img-blog.csdnimg.cn/cd6613fa797b474f8d56cc1b1a6142b1.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5b-16Iqx5byAXw==,size_20,color_FFFFFF,t_70,g_se,x_16)

在这里我给出实际测试结果参考

比如有3个消费者（C0、C1、C2）、4个topic(T0、T1、T2、T3)，每个topic有2个分区（P1、P2）

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/0f5688b418cf4100a97af87c46c9b0c5.png)

**C0:**  **T0P0、T1P1、T3P0**

**C1: T0P1、T2P0、T3P1**

**C2: T1P0、T2P1**

如果C1下线 、如果按照RoundRobin

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/4c352e4724cf4326b51b3b4e3bfd8ec7.png)

**C0:**  **T0P0、T1P0、T2P0、T3P0**

**C2:  T0P1、T1P1、T2P1、T3P1**

对比之前

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/488fb784b67645e6b17d54e6a42aa812.png)

如果C1下线 、如果按照StickyAssignor

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/92cd0754ee2d4785aff805e0ccf2ca7e.png)

**C0:**  **T0P0、T1P1、T2P0、T3P0**

**C2:  T0P1、T1P0、T2P1、T3P1**

对比之前

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/488fb784b67645e6b17d54e6a42aa812.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/71a63c396528426f81c85c27e252066b.png)





#### 自定义策略

extends 类AbstractPartitionAssignor，然后在消费者端增加参数：

properties.put(ConsumerConfig.PARTITION_ASSIGNMENT_STRATEGY_CONFIG,类.class.getName());

即可。

### 消费者分区策略源码分析

接着上个章节的代码。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/2df5eb1f747e46718f854ba4a537c24b.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/76f057d2b969462d8fe35120a447eeaf.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/da22757d6471436cb349cbd1f442c79d.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/9c8fab4167394382acfd3096851cf13d.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/88165821148b4dda8eabb85a673769d6.png)

## Consumer拉取数据

这里就是拉取数据，核心Fetch类

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/d547217dfb1e47d4aed48b659eae4003.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/6741976e4860466289e0ca387cc6f283.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/74b908d7a6fd493f9651b3bb51537642.png)

## 自动提交偏移量

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/a1c6353daa0e4184bf50f402386ba38d.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/68a4e535116047b8a4f75d07d8b78579.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/1345cc1f1e724f6583d8ea9a51b28fab.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/d54727b2226945cc89d27c84544a815e.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/50b6a6657e1e42b48a787c1b92526ece.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/d96d48ea405240a2be36bb6ac5f37060.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/ed503ca6d42d4b58880613d1a5893a8d.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/7d12a36e5c4d4d029ce461da4212e12d.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/69b0fb87371e46c08d27e1dab133c902.png)

当然，自动提交auto.commit.interval.ms

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/38e257be210e44158cf8c340443df2b0.png)

默认5s

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1675692369064/f701faa4444a4e9ab97ba3605acae83d.png)

从源码上也可以看出

maybeAutoCommitOffsetsAsync 最后这个就是poll的时候会自动提交，而且没到auto.commit.interval.ms间隔时间也不会提交，如果没到下次自动提交的时间也不会提交。

这个autoCommitIntervalMs就是auto.commit.interval.ms设置的



### 源码流程图：



# 1、生产者网络设计

## 架构设计图

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1676558275075/4b6bc02f69d841c688b0385e2f0fb112.jpg)

# 2、生产者消息缓存机制

### 1、RecordAccumulator

将消息缓存到RecordAccumulator收集器中, 最后判断是否要发送。这个加入消息收集器，首先得从 Deque<RecordBatch> 里找到自己的目标分区，如果没有就新建一个批量消息 Deque 加进入

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1676558275075/e82e4b9b468c48b5892ed605f82ea93b.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1676558275075/dee80d3529d648079e67b86da480eb9a.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1676558275075/d2c48ec28b8c490dbfe29200a7edbd49.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1676558275075/81fc2531358c4a8fa58e2847711cb2be.png)

### 2、消息发送时机

如果达到发送阈值（**批次发送的条件为:缓冲区数据大小达到 batch.size 或者 linger.ms 达到上限，哪个先达到就算哪个**），唤醒Sender线程，

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1676558275075/20cd7ef1988c4082b27a655d03b6a42e.png)

NetWorkClient 将 batch record 转换成 request client 的发送消息体, 并将待发送的数据按 【Broker Id <=> List】的数据进行归类

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1676558275075/295fdbad2f2d49d5841478989b66fc40.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1676558275075/f67309c1f8594d3c960e26eff8dda319.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1676558275075/9cd086d2acec4f0f81b263430a7f9366.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1676558275075/bca8ce31efa044adaf7b7c2b95463679.png)

与服务端不同的 Broker 建立网络连接，将对应 Broker 待发送的消息 List 发送出去。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1676558275075/9363d1add5634550b3a106d38d00fb36.png)

9)、

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1676558275075/904ea02aa0c24cb6a2f752d1b72063c4.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1676558275075/735d2da9d4494f189ce55164bb985ac0.png)

经过几轮跳转

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1676558275075/b989e3bbfbfe46e88c5c13f566e2e25d.png)

# 3、Kafka通讯组件解析

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1676558275075/cb489cf938f7455ba3625d186e507867.png)

![深入理解Kafka网络通讯模型](E:\图灵课堂\MQ专题\Kafka.assets\深入理解Kafka网络通讯模型.png)

# **Kafka与时间轮**

Kafka中存在大量的延时操作。
1、发送消息-超时+重试机制

2、ACKS  用于指定分区中必须要有多少副本收到这条消息，生产者才认为写入成功（延时 等）

Kafka并没有使用JDK自带的Timer或者DelayQueue来实现延迟的功能，而是基于时间轮自定义了一个用于实现延迟功能的定时器（SystemTimer）

JDK的Timer和DelayQueue插入和删除操作的平均时间复杂度为O(log(n))，并不能满足Kafka的高性能要求，而基于时间轮可以将插入和删除操作的时间复杂度都降为O(1)。

时间轮的应用并非Kafka独有，其应用场景还有很多，在Netty、Akka、Quartz、Zookeeper等组件中都存在时间轮的踪影。

# **时间轮**

## Java中任务调度

要回答这个问题，我们先从Java中最原始的任务调度的方法说起。

给你一批任务（假设有1000个任务），都是不同的时间执行的，时间精确到秒，你怎么实现对所有的任务的调度？

第一种思路是启动一个线程，每秒钟对所有的任务进行遍历，找出执行时间跟当前时间匹配的，执行它。如果任务数量太大，遍历和比较所有任务会比较浪费时间。

![](file:///C:/Users/root/AppData/Local/Temp/ksohtml10964/wps14.jpg)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/1463/1663134961030/3dcc1a5fff9b4ad998f23b99af66d62b.png)

第二个思路，把这些任务进行排序，执行时间近（先触发）的放在前面。

![](file:///C:/Users/root/AppData/Local/Temp/ksohtml10964/wps15.jpg)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/1463/1663134961030/2ca7b8bc01eb4bc09360e7ed310ac2ca.png)

如果是数组的时间的话，这里会涉及到大量的元素移动（新加入任务，任务执行--删除任务之类，都需要重新排序）

那么在Java代码怎么实现呢？

JDK包里面自带了一个Timer工具类（java.util包下），可以实现延时任务（例如30分钟以后触发），也可以实现周期性任务（例如每1小时触发一次）。

它的本质是一个优先队列（TaskQueue），和一个执行任务的线程（TimerThread）。

（普通的队列是一种先进先出的数据结构，元素在队列尾追加，而从队列头删除。在优先队列中，元素被赋予优先级。当访问元素时，具有最高优先级的元素最先删除。优先队列具有最高级先出 （first in, largest out）的行为特征。通常采用堆数据结构来实现。）

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1678872315001/9da0c308f3254cf49d3b59790d497f59.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1678872315001/307681c9a98e46558389670fb63c529a.png)

在这个优先队列中，最先需要执行的任务排在优先队列的第一个。然后 TimerThread 不断地拿第一个任务的执行时间和当前时间做对比。如果时间到了先看看这个任务是不是周期性执行的任务，如果是则修改当前任务时间为下次执行的时间，如果不是周期性任务则将任务从优先队列中移除。最后执行任务。

但是Timer是单线程的，在很多场景下不能满足业务需求。

在JDK1.5之后，引入了一个支持多线程的任务调度工具ScheduledThreadPoolExecutor用来替代TImer，它是几种常用的线程池之一。看看构造函数，里面是一个延迟队列DelayedWorkQueue，也是一个优先队列。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1678872315001/7b50860d60d140a1b42593530196b763.png)

## DelayedWorkQueue的最小堆实现

优先队列的使用的是最小堆实现。

最小堆的含义: 一种完全二叉树, 父结点的值小于或等于它的左子节点和右子节点

比如插入以下的数据 [1,2,3,7,17,19,25,36,100]

最小堆就长成这个样子。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1678872315001/e9247ac5fdb74cdcaf85008526e961e1.png)

优先队列的插入和删除的时间复杂度是O(logn)，当数据量大的时候，频繁的入堆出堆性能不是很好。

比如要插入0，过程如下：

1、插入末尾元素

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1678872315001/403cdc29897b40a5a08ba52f749689be.png)

2、0比19小，所以要向上移动且互换。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1678872315001/2e6becddd5084c6685090fc926a281b8.png)

3、0比2小，所以要向上移动且互换。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1678872315001/2574d820cf384bb2adf7c85a0a1a4851.png)

4、0比2小，所以要向上移动且互换。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1678872315001/1a00dbaaad6e4338a7b5af1b13d32fa5.png)

算法复杂度

N个数据的最小堆, 共有logN层, 最坏的情况下, 需要移动logN次

## 时间轮

这里我们先考虑对所有的任务进行分组，把相同执行时刻的任务放在一起。比如这里，数组里面的一个下标就代表1秒钟。它就会变成一个数组加链表的数据结构。分组以后遍历和比较的时间会减少一些。

![](file:///C:/Users/root/AppData/Local/Temp/ksohtml10964/wps16.jpg)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/1463/1663134961030/2bdb3b481fd746c09db185965573d79f.png)

但是还是有问题，如果任务数量非常大，而且时间都不一样，或者有执行时间非常遥远的任务，那这个数组长度是不是要非常地长？比如有个任务2个月之后执行，从现在开始计算，它的下标是5253120。

所以长度肯定不能是无限的，只能是固定长度的。比如固定长度是8，一个格子代表1秒（现在叫做一个bucket槽），一圈可以表示8秒。遍历的线程只要一个格子一个格子的获取任务，并且执行就OK了。

固定长度的数组怎么用来表示超出最大长度的时间呢？可以用循环数组。

比如一个循环数组长度8，可以表示8秒。8秒以后执行的任务怎么放进去？只要除以8，用得到的余数，放到对应的格子就OK了。比如10%8=2，它放在第2个格子。这里就有了轮次的概念，第10秒的任务是第二轮的时候才执行。

![](file:///C:/Users/root/AppData/Local/Temp/ksohtml10964/wps17.jpg)![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1678872315001/d413a99069a04071a50f7903efa3be79.png)

这时候，时间轮的概念已经出来了。

如果任务数量太多，相同时刻执行的任务很多，会导致链表变得非常长。这里我们可以进一步对这个时间轮做一个改造，做一个多层的时间轮。

比如：最内层8个格子，每个格子1秒；外层8个格子，每个格子8*8=64秒；最内层走一圈，外层走一格。这时候时间轮就跟时钟更像了。随着时间流动，任务会降级，外层的任务会慢慢地向内层移动。

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1678872315001/42725ff891ed46a3915fbb33be50bff1.png)

时间轮任务插入和删除时间复杂度都为O(1)，应用范围非常广泛，更适合任务数很大的延时场景。Dubbo、Netty、Kafka中都有实现。

## Kafka中时间轮实现

Kafka里面TimingWheel的数据结构

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1678872315001/06096467ad98448b92dc6ecbc404e551.png)

kafka会启动一个线程，去推动时间轮的指针转动。其实现原理其实就是通过queue.poll()取出放在最前面的槽的TimerTaskList

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1678872315001/2d050bd3d3cf4415848bd9bf093a6311.png)

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1678872315001/79e85454a83246cebaee05e553723057.png)

**添加新的延迟任务**

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1678872315001/1f2b38b0c46d490babdcef8aaf87439f.png)

**往时间轮添加新的任务**

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1678872315001/2b70038bad0747d5af2065d6b26b53fa.png)

**时间轮指针的推进**

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1678872315001/e4732c56cda54101bc398a1d931d6e09.png)

第二层时间轮的创建代码如下

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/5983/1678872315001/268bf7929707430c80fe05c8ef0e2c5f.png)

# Kafka性能问题

## 1、kafka如何确保消息的可靠性传输

这个问题需要从以下3个方面分析和解决

**（1）消费端弄丢了数据**

唯一可能导致消费者弄丢数据的情况，就是说，你那个消费到了这个消息，然后消费者那边自动提交了offset，让kafka以为你已经消费好了这个消息，其实你刚准备处理这个消息，你还没处理，你自己就挂了，此时这条消息就丢咯。

大家都知道kafka会自动提交offset，那么只要关闭自动提交offset，在处理完之后自己手动提交offset，就可以保证数据不会丢。但是此时确实还是会重复消费，比如你刚处理完，还没提交offset，结果自己挂了，此时肯定会重复消费一次，自己保证幂等性就好了。

生产环境碰到的一个问题，就是说我们的kafka消费者消费到了数据之后是写到一个内存的queue里先缓冲一下，结果有的时候，你刚把消息写入内存queue，然后消费者会自动提交offset。

然后此时我们重启了系统，就会导致内存queue里还没来得及处理的数据就丢失了

**（2）kafka弄丢了数据**

这块比较常见的一个场景，就是kafka某个broker宕机，然后重新选举partiton的leader时。大家想想，要是此时其他的follower刚好还有些数据没有同步，结果此时leader挂了，然后选举某个follower成leader之后，他不就少了一些数据？这就丢了一些数据啊。

所以此时一般是要求起码设置如下4个参数：

给这个topic设置replication.factor参数：这个值必须大于1，要求每个partition必须有至少2个副本。

在kafka服务端设置min.insync.replicas参数：这个值必须大于1，这个是要求一个leader至少感知到有至少一个follower还跟自己保持联系，没掉队，这样才能确保leader挂了还有一个follower吧。

在producer端设置acks=all：这个是要求每条数据，必须是写入所有replica之后，才能认为是写成功了。

在producer端设置retries=MAX（很大很大很大的一个值，无限次重试的意思）：这个是要求一旦写入失败，就无限重试，卡在这里了。

**（3）生产者会不会弄丢数据**

如果按照上述的思路设置了ack=all，一定不会丢，要求是，你的leader接收到消息，所有的follower都同步到了消息之后，才认为本次写成功了。如果没满足这个条件，生产者会自动不断的重试，重试无限次。

## 2、如何实现Kafka的高性能？

### 1、宏观架构层面利用Partition实现并行处理

Kafka中每个Topic都包含一个或多个Partition，不同Partition可位于不同节点。同时Partition在物理上对应一个本地文件夹，每个Partition包含一个或多个Segment，每个Segment包含一个数据文件和一个与之对应的索引文件。在逻辑上，可以把一个Partition当作一个非常长的数组，可通过这个“数组”的索引（offset）去访问其数据。

一方面，由于不同Partition可位于不同机器，因此可以充分利用集群优势，实现机器间的并行处理。另一方面，由于Partition在物理上对应一个文件夹，即使多个Partition位于同一个节点，也可通过配置让同一节点上的不同Partition置于不同的disk drive上，从而实现磁盘间的并行处理，充分发挥多磁盘的优势。

利用多磁盘的具体方法是，将不同磁盘mount到不同目录，然后在server.properties中，将log.dirs设置为多目录（用逗号分隔）。Kafka会自动将所有Partition尽可能均匀分配到不同目录也即不同目录（也即不同disk）上。

Partition是最小并发粒度，Partition个数决定了可能的最大并行度。

### 2、充分利用PageCache

Page Cache，又称pcache，其中文名称为页高速缓冲存储器，简称页高缓。page cache的大小为一页，通常为4K。在linux读写文件时，它用于缓存文件的逻辑内容，从而加快对磁盘上映像和数据的访问。 是Linux操作系统的一个特色。

![image.png](E:\图灵课堂\MQ专题\MQ专题.assets\580ae3478cf641bd97bad211cdae8108.png)

##### 1、读Cache

当内核发起一个读请求时(例如进程发起read()请求)，首先会检查请求的数据是否缓存到了Page Cache中。

如果有，那么直接从内存中读取，不需要访问磁盘，这被称为cache命中(cache hit)；

如果cache中没有请求的数据，即cache未命中(cache miss)，就必须从磁盘中读取数据。然后内核将读取的数据缓存到cache中，这样后续的读请求就可以命中cache了。

page可以只缓存一个文件部分的内容，不需要把整个文件都缓存进来。

##### 2、写Cache

当内核发起一个写请求时(例如进程发起write()请求)，同样是直接往cache中写入，后备存储中的内容不会直接更新(当服务器出现断电关机时，存在数据丢失风险)。

内核会将被写入的page标记为dirty，并将其加入dirty list中。内核会周期性地将dirty list中的page写回到磁盘上，从而使磁盘上的数据和内存中缓存的数据一致。

当满足以下两个条件之一将触发脏数据刷新到磁盘操作：

数据存在的时间超过了dirty_expire_centisecs(默认300厘秒，即30秒)时间；

脏数据所占内存 > dirty_background_ratio，也就是说当脏数据所占用的内存占总内存的比例超过dirty_background_ratio(默认10，即系统内存的10%)的时候会触发pdflush刷新脏数据。

如何查看Page Cache参数

执行命令 sysctl -a|grep dirty

##### 如何调整内核参数来优化IO性能？

##### (1)vm.dirty_background_ratio参数优化

这个参数指定了当文件系统缓存脏页数量达到系统内存百分之多少时（如5%）就会触发后台回写进程运行，将一定缓存的脏页异步地刷入磁盘；

当cached中缓存当数据占总内存的比例达到这个参数设定的值时将触发刷磁盘操作。

把这个参数适当调小，这样可以把原来一个大的IO刷盘操作变为多个小的IO刷盘操作，从而把IO写峰值削平。

对于内存很大和磁盘性能比较差的服务器，应该把这个值设置的小一点。

##### (2)vm.dirty_ratio参数优化

这个参数则指定了当文件系统缓存脏页数量达到系统内存百分之多少时（如10%），系统不得不开始处理缓存脏页（因为此时脏页数量已经比较多，为了避免数据丢失需要将一定脏页刷入外存）；在此过程中很多应用进程可能会因为系统转而处理文件IO而阻塞。

对于写压力特别大的，建议把这个参数适当调大；对于写压力小的可以适当调小；如果cached的数据所占比例(这里是占总内存的比例)超过这个设置，

系统会停止所有的应用层的IO写操作，等待刷完数据后恢复IO。所以万一触发了系统的这个操作，对于用户来说影响非常大的。

##### (3)vm.dirty_expire_centisecs参数优化

这个参数会和参数vm.dirty_background_ratio一起来作用，一个表示大小比例，一个表示时间；即满足其中任何一个的条件都达到刷盘的条件。

为什么要这么设计呢？我们来试想一下以下场景：

如果只有参数 vm.dirty_background_ratio ，也就是说cache中的数据需要超过这个阀值才会满足刷磁盘的条件；

如果数据一直没有达到这个阀值，那相当于cache中的数据就永远无法持久化到磁盘，这种情况下，一旦服务器重启，那么cache中的数据必然丢失。

结合以上情况，所以添加了一个数据过期时间参数。当数据量没有达到阀值，但是达到了我们设定的过期时间，同样可以实现数据刷盘。

这样可以有效的解决上述存在的问题，其实这种设计在绝大部分框架中都有。

##### (4)vm.dirty_writeback_centisecs参数优化

理论上调小这个参数，可以提高刷磁盘的频率，从而尽快把脏数据刷新到磁盘上。但一定要保证间隔时间内一定可以让数据刷盘完成。

##### (5)vm.swappiness参数优化

禁用swap空间，设置vm.swappiness=0

### 3、减少网络开销批处理

批处理是一种常用的用于提高I/O性能的方式。对Kafka而言，批处理既减少了网络传输的Overhead，又提高了写磁盘的效率。

Kafka 的send方法并非立即将消息发送出去，而是通过**batch.size**和**linger.ms**控制实际发送频率，从而实现批量发送。

由于每次网络传输，除了传输消息本身以外，还要传输非常多的网络协议本身的一些内容（称为Overhead），所以将多条消息合并到一起传输，可有效减少网络传输的Overhead，进而提高了传输效率。

### 4、数据压缩降低网络负载

Kafka支持将数据压缩后再传输给Broker。除了可以将每条消息单独压缩然后传输外，Kafka还支持在批量发送时，将整个Batch的消息一起压缩后传输。数据压缩的一个基本原理是，重复数据越多压缩效果越好。因此将整个Batch的数据一起压缩能更大幅度减小数据量，从而更大程度提高网络传输效率。

Broker接收消息后，并不直接解压缩，而是直接将消息以压缩后的形式持久化到磁盘。Consumer Fetch到数据后再解压缩。因此Kafka的压缩不仅减少了Producer到Broker的网络传输负载，同时也降低了Broker磁盘操作的负载，也降低了Consumer与Broker间的网络传输量，从而极大得提高了传输效率，提高了吞吐量。

### 5、高效的序列化方式

Kafka消息的Key和Value的类型可自定义，只需同时提供相应的序列化器和反序列化器即可。

因此用户可以通过使用快速且紧凑的序列化-反序列化方式（如Avro，Protocal Buffer）来减少实际网络传输和磁盘存储的数据规模，从而提高吞吐率。这里要注意，如果使用的序列化方法太慢，即使压缩比非常高，最终的效率也不一定高。

### 6、最大化发挥消费端能力

首先采用的拉取消息模式，主动权在消费端自己。完全可以发挥其最大消费能力。而且可以可以配置参数：**concurrency** 控制listener 的线程数量  ，并发开关可以通过batchListener = true 开启  配合**max_poll_records_config**=50 多少条一次poll返回  也可以配置间隔时间**tnterval**.**ms** 间隔多久poll一次  最多多少条



















