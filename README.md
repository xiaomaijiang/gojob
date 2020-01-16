
[![License](https://img.shields.io/badge/license-Apache%202-4EB1BA.svg)](https://www.apache.org/licenses/LICENSE-2.0.html)

# gojob

gojob是轻量级的任务调度解决方案，使用Go语言编写，协程并发机制可支持大规模任务并行调度。gojob为调度节点，你的业务系统就是执行节点，凡是暴露出HTTP服务的业务方法都可以被调度，对业务系统完全无侵入。HTTP协议方便夸平台，契合当前主流的微服务架构。

gojob通过raft强数据一致性协议、数据库异步多写、执行节点路由等措施，消除调度节点和执行节点的单点隐患，使调度系统具有极高的可用性。无人值守也能高枕无忧。

通过WEB UI进行作业元数据管理，你可以将将散落在各个业务系统的定时任务进行收拢，统一管理和调度。gojob使用方便，解压安装包，配置好MySQL连接(表结构会自动创建)，即可运行。

# 特性

1、易部署：原生Native程序、无需要运行时环境，如JDK、.net framework等；支持单机和集群部署两种部署模式。

2、极少依赖：只依赖MySQL 数据库，分布式环境下使用内建的分布式协调机制，不需要安装第三方分布式协调服务，如zookeeper、etcd等。

3、任务重试：支持自定义任务重试次数，和重试间隔时间，当任务执行失败时，会按照间隔时间进行重试。

4、任务超时：支持自定义任务超时时间，当任务超时，会强制结束执行。

5、失败转移：当任务在一个执行节点上执行失败，会转移到下一个可用执行节点上继续执行。如任务在执行节点A上失败，会转移到执行节点B上继续执行，如果失败会转移到执行节点C上继续执行，直到执行成功。

6、misfire补偿：由于调度节点服务器故障、无资源、重启致使任务错过激活时间，称之为misfire(哑火)。比如每天23点统计当日数据生成报表，恰巧在23点服务器宕机、此任务就错失了一次调度。如果我们设置了misfireThreshold为10分钟，服务器在23点10分前恢复，调度器会进行一次补充执行。

7、负载均衡：支持执行节点集群部署，调度节点使用轮询、随机、加权轮询、加权随机等路由策略，为任务选择合适的执行节点，保证执行节点高可用。

8、任务分片：将大任务拆解为多个小任务均匀的散落在多个节点上并行执行，以协作的方式完成任务。比如订单核对业务，我们有天津、上海、重庆、河北、山西、辽宁、吉林、江苏、浙江、安徽十个省市的账单。单机的处理能力毕竟有限，假如我们将任务分为3片：node1-->天津、上海、重庆、河北；node2-->山西、辽宁、吉林; node3-->江苏、浙江、安徽，这样可以用3台机器来合力完成这个任务。如果你的机器足够，可以分成更多片用跟多的机器来处理。

9、弹性扩缩容：执行节点的每一次上线下线、增加删除都会被调度器感知，并应用在下一次的负载均衡或者节点分片中。调度系统可以通过计算节点的横向扩展，来弹性伸缩整个系统的任务处理能力。

10、调度唯一性：调度节点集群使用Raft算法进行主节点选举，一个集群中只存在一个主节点，因此任务在一个行周期内只被主节点调度一次，保证调度的一致性。

11、调度节点高可用：调度节点内建BoltDB存储引擎来保存作业元数据，集群节点通过Raft强数据库一致性协议和数据快照进行数据实时同步。数据多副本存储，任务可在任意一个调度节点被调度，节点间无缝衔接。保证调度节点无单点隐患。

12、数据库异步多写：作业元数据保存在节点内部的存储引擎，MySQL数据库只用来保存调度日志。日志数据可容忍短时间内不一致甚至丢失(虽然极少发生但理论上可容忍)，因此用异步方式写入多库，无需对数据库做任何设置，可保证数据库节点无单点隐患。即便数据库节点全部宕机都不会影响调度业务的正常运行。

13、任务依赖：可为任务设置多个子任务，并选择适当时机触发，如：任务结束触发子任务、任务成功触发子任务、任务失败触发子任。

14、告警：使用邮箱告警，分为运维类告警和任务告警。当调度节点出现故障、数据库节点出现故障会发送运维类告警邮件；当任务调度失败会发送调度失败告警邮件，每个任务可配置多个告警邮箱。


