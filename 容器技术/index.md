# 容器技术


<!--more-->

# Docker

## Docker架构

![截屏2022-09-06 下午6.52.37](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-06%20%E4%B8%8B%E5%8D%886.52.37.png)

1. K8S:CRI(Container Runtime Interface): 用于对接容器
2. Client: 客户端;操作docker服务器的客户端(命令行或者界面)
3. Docker_Host:Docker主机;安装Docker服务的主机
4. Docker_Daemon:后台进程;运行在Docker服务器的后台进程
5. Containers:容器;在Docker服务器中的容器(一个容器一般是一个应用实例，容器间互相隔离)
6. Images:镜像、映像、程序包;Image是只读模板，其中包含创建Docker容器的说明。容器是由Image运 行而来，Image固定不变。
7. Registries:仓库;存储Docker Image的地方。



### Docker和虚拟机

![截屏2022-09-06 下午8.51.30](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-06%20%E4%B8%8B%E5%8D%888.51.30.png)

> **Docker守护进程**可以直接与**主操作系统**进行通信，为各个**Docker容器**分配资源；它还可以将容器与**主操作系统**隔离，并将各个容器互相隔离。**虚拟机**启动需要数分钟，而**Docker容器**可以在数毫秒内启动。由于没有臃肿的**从操作系统**，Docker可以节省大量的磁盘空间以及其他系统资源。
>
> 但是也没有必要完全否定**虚拟机**技术，因为两者有不同的使用场景。**虚拟机**更擅长于彻底隔离整个运行环境。例如，云服务提供商通常采用虚拟机技术隔离不同的用户。而**Docker**通常用于隔离不同的应用，例如**前端**，**后端**以及**数据库**。

## 隔离原理

​		Docker用Go编程语言编写，并利用Linux内核的多种功能来交付其功能。 Docker使用一种称为名称空间(namespace)的技术来提供容器的隔离工作区。 运行容器时，Docker会为该容器创建一组名称空间。 这些 名称空间提供了一层隔离。 容器的每个方面都在单独的名称空间中运行，并且对其的访问仅限于 该名称空间。

- namespace 6项离隔 (资源隔离)

| namespace | 系统调用参数  | 隔离内容                   |
| :-------- | :-----------: | -------------------------- |
| UTS       | CLONE_NEWUTS  | 主机和域名                 |
| IPC       | CLONE_NEWIPC  | 信号量、消息队列和共享内存 |
| PID       | CLONE_NEWPID  | 进程编号                   |
| Network   | CLONE_NEWNET  | 网络设备、网络栈、端口等   |
| Mount     |  CLONE_NEWNS  | 挂载点(文件系统)           |
| User      | CLONE_NEWUSER | 用户和用户组               |

- cgroups 资源限制

cgroup提供的主要功能如下:

1. 资源限制:限制任务使用的资源总额，并在超过这个配额 时发出提示
2. 优先级分配:分配CPU时间片数量及磁盘IO带宽大小、控制任务运行的优先级
3. 资源统计:统计系统资源使用量，如CPU使用时长、内存用量等
4. 任务控制:对任务执行挂起、恢复等操作

> cgroup资源控制系统，每种子系统独立地控制一种资源。功能如下
>
> | 子系统                          |                             功能                             |
> | :------------------------------ | :----------------------------------------------------------: |
> | cpu                             |               使用调度程序控制任务对CPU的使用                |
> | cpuacct(CPU Accounting)         |        自动生成cgroup中任务对CPU资源使用情况的报告。         |
> | cpuset                          |     为cgroup中的任务分配独立的CPU(多处理器系统时)和内存      |
> | devices                         |              开启或关闭cgroup中任务对设备的访问              |
> | freezer                         |                   挂起或恢复cgroup中的任务                   |
> | memory                          | 设定cgroup中任务对内存使用量的限定，并生成这些任务对内存资源使用 情况的报告 |
> | perf_event(Linux CPU性能探测器) |            使cgroup中的任务可以进行统一的性能测试            |
> | net_cls(Docker未使用)           | 通过等级识别符标记网络数据包，从而允许Linux流量监控程序(Traic  Controller)识别从具体cgroup中生成的数据包 |



## Docker存储

### 镜像存储

#### Overlay2

​		overlay2文件系统:  overlay2 的目录是镜像和容器分层的基础，而把这些层统一展现到同一的目录下的过程称为联合挂载（union mount）;**overlay2 把目录的下一层叫作lowerdir，上一层叫作upperdir，联合挂载后的结果叫作merged; workdir为临时目录**

​		**总体来说，overlay2 是这样储存文件的：overlay2将镜像层和容器层都放在单独的目录，并且有唯一 ID，每一层仅存储发生变化的文件，最终使用联合挂载技术将容器层和镜像层的所有文件统一挂载到容器中，使得容器中看到完整的系统文件。**

> 镜像中:  **MergedDir 代表当前镜像层在 overlay2 存储下的目录，LowerDir 代表当前镜像的父层关系，使用冒号分隔，冒号最后代表该镜像的最底层。**
>
> 容器中:**lower 文件为该层的所有父层镜像的短 ID 。diff 目录为容器的读写层，容器内修改的文件都会在 diff 中出现，merged 目录为分层文件联合挂载后的结果，也是容器内的工作目录。**	

​		

#### 镜像存储

​		Docker镜像由一系列层组成。 每层代表图像的Dockerfile中的一条指令。 容器除最后一层外的每一层都是只读的。

> - 每一层只是与上一层不同的一组。 这些层彼此堆叠。
> - 创建新容器时，可以在基础层之上添加一个新的可写层。 该层通常称为“容器层”。 对运行中 的容器所做的所有更改(例如写入新文件，修改现有文件和删除文件)都将写入此薄可写容 器层。
> - 每层之间如同git一样,只记录不同的部分, 所有层文件合并起来即为所有镜像文件

![截屏2022-09-06 下午10.19.59](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-06%20%E4%B8%8B%E5%8D%8810.19.59.png)



#### 容器存储

> - 容器和镜像之间的主要区别是可写顶层。 
> - 在容器中添加新数据或修改现有数据的所有写操作都存储在此可写层中。 
> - 删除容器后，可写层也会被删除。 基础图像保持不变。 因为每个容器都有其自己的可写容器层，并且所有更改都存储在该容器层中，所以多个容器可以共享对同一基础映像的访问， 但具有自己的数据状态。

![截屏2022-09-06 下午10.39.04](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-06%20%E4%B8%8B%E5%8D%8810.39.04.png)



#### 容量查看

```sh
[root@cx-tenc home]# docker ps -s
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS     NAMES             SIZE
cf53f0ea23dd   nginx     "/docker-entrypoint.…"   15 seconds ago   Up 14 seconds   80/tcp    dazzling_carson   1.09kB (virtual 142MB)
```

1. size: 用于每个容器的可写层的数据量(在磁盘上)。
2. virtual size:容器使用的用于只读镜像数据的数据量加上容器的可写图层大小。 多个容器可以共享部分或全部只读图像数据。 从同一镜像开始的两个容器共享100%的只读数据，而具有不同图像的两个容器(具有相同的层)共享这些公共层。 因此，不能只对虚拟大小进行总计。这高估了总磁盘使用量，可能是一笔不小的数目。



#### Copy On Write

- 写时复制是一种共享和复制文件的策略，可最大程度地提高效率。
-  如果文件或目录位于映像的较低层中，而另一层(包括可写层)需要对其进行读取访问，则它仅使用现有文件。
- 另一层第一次需要修改文件时(在构建镜像或运行容器时)，将文件复制到该写层并进行修改。 这样可以将I / O和每个后续层的大小最小化。

> **镜像如何挑选**:
>
> - busybox:是一个集成了一百多个最常用Linux命令和工具的软件。linux工具里的瑞士军刀 
> - alpine:Alpine操作系统是一个面向安全的轻型Linux发行版经典最小镜像，基于busybox，功能比 Busybox完善。
> -  slim:docker hub中有些镜像有slim标识，都是瘦身了的镜像。也要优先选择 无论是制作镜像还是下载镜像，优先选择alpine类型.

### 卷挂载

- 匿名卷使用

```sh
docker run -dP -v :/etc/nginx nginx
#docker将创建出匿名卷，并保存容器/etc/nginx下面的内容
```

​		匿名卷在/var/lib/docker/volumes下创建 sha256代码的文件夹, 其中的内容为容器中对应文件夹下的内容; 

- 具名卷使用

```sh
docker run -dP -v nginx:/etc/nginx nginx
#docker将创建出名为nginx的卷，并保存容器/etc/nginx下面的内容
```

​		如果将空卷装入存在文件或目录的容器中的目录中，则容器中的内容(挂载)到该卷中。如果启动一个容器并指定一个尚不存在的卷，则会创建一个空卷。

- bind mount

```sh
docker run -dP -v /my/nginx:/etc/nginx nginx
# bind mount和 volumes 的方式写法区别在于
# 所有以/开始的都认为是 bind mount ，不以/开始的都认为是 volumes.
```

​		外部目录覆盖内部容器目录内容，但不是修改。所以谨慎，外部空文件夹挂载方式也会导致容器内部是空文件夹







## 网络原理

​		Docker使用Linux桥接，在宿主机虚拟一个Docker容器网桥(docker0)，Docker启动一个容器时会根据Docker网桥的网段分配给容器一个IP地址，称为Container-IP，同时Docker网桥是每个容器的默认网关。 因为在同一宿主机内的容器都接入同一个网桥，这样容器之间就能够通过容器的Container-IP直接通信。![截屏2022-09-07 上午9.44.11](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-07%20%E4%B8%8A%E5%8D%889.44.11.png)

​		Docker容器网络就很好的利用了Linux虚拟网络技术，在本地主机和容器内分别创建一个虚拟接口，并让他们彼此联通(这样一对接口叫veth pair);

​		Docker中的网络接口默认都是虚拟的接口。虚拟接口的优势就是转发效率极高(因为Linux是在内核中进 行数据的复制来实现虚拟接口之间的数据转发，无需通过外部的网络设备交换)，对于本地系统和容器 系统来说，虚拟接口跟一个正常的以太网卡相比并没有区别，只是他的速度快很多。

### 网络模式

| 网络模式       |           配置           | 说明                                                         |
| -------------- | :----------------------: | :----------------------------------------------------------- |
| bridge模式     |       --net=bridge       | 默认值，在Docker网桥docker0上为容器创建新的网络 栈           |
| none模式       |        --net=none        | 不配置网络，用户可以稍后进入容器，自行配置                   |
| container模 式 | -- net=container:name/id | 容器和另外一个容器共享Network namespace。 kubernetes中的pod就是多个容器共享一个Network namespace。 |
| host模式       |        --net=host        | 容器和宿主机共享Network namespace                            |
| 用户自定义     |      net=自定义网络      | 用户自己使用network相关命令定义网络，创建容器的时候可以指定为自己定义的网络 |

> 自定义网络也使用bridge模式,但是只有**用户自定义的网卡可以在容器之间提供自动的 DNS 解析**;  
>
> 缺省的桥接网络上的容器只能通过 IP 地址互相访问，除非使用 --link 参数。官方文档中已经**不推荐**使用 [--link](https://link.zhihu.com/?target=https%3A//docs.docker.com/network/links/) 参数，并且最终可能会被删除，所以最好不要使用 --link 参数来连接两个容器，并且它有多个缺点。
>
> 如果使用 --link 参数，需要在容器之间手动创建链接，这些链接需要双向创建，如果容器多于两个的话，将会很困难。或者也可以通过编辑 hosts 文件的方式来指定解析结果，但是这样将会非常难以调试。





# K8S

## Pod

- 工作负载(Workloads)控制一组Pod;Pod控制一组容器(Containers)
- Pod 天生地为其成员容器提供了两种共享资源: **网络** 和 **存储** 。
- 一个Pod由一个 Pause**容器** 设置好整个Pod里面所有容器的网络、名称空间等信息

![截屏2022-09-07 上午10.03.25](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-07%20%E4%B8%8A%E5%8D%8810.03.25.png)

### 生命周期

![截屏2022-09-07 上午10.04.56](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-07%20%E4%B8%8A%E5%8D%8810.04.56.png)

- Pod启动，会先 **依次** 执行所有初始化容器，有一个失败，则Pod不能启动
- 接下来 **启动所有的应用容器** (每一个应用容器都必须能一直运行起来)，Pod开始正式工作，一个 启动失败就会 **尝试重启**Pod**内的这个容器** ，Pod只要是NotReady，Pod就不对外提供服务了
- 临时容器:线上排错。 有些容器基础镜像。线上没法排错。使用临时容器进入这个Pod。临时容器共享了Pod的所有。临时容器有Debug的一些命令，拍错完成以后，只要exit退出容器，临时容器自动删除



### Probe 探针机制

每个容器三种探针(Probe):

1. **启动探针**(后来才加的)一次性成功探针。只要启动成功了(initProbe)
   - kubelet 使用启动探针，来检测应用是否已经启动。如果启动完成就可以进行后续的探测检查。慢容器一定指定启动探针。一直在等待启动
   -  **启动探针成功以后就不用了，剩下存活探针和就绪探针持续运行**
2. 存活探针(livenessProbe)
   - kubelet 使用存活探针，来检测容器是否正常存活。(有些容器可能产生死锁【应用程序 在运行，但是无法继续执行后面的步骤】)， 如果检测失败就会**重新启动这个容器 **
   - initialDelaySeconds: 3600(长了导致可能应用一段时间不可用) 5(短了陷入无限启动循环)
3. 就绪探针(readinessProbe)
   - kubelet 使用就绪探针，来检测容器是否准备**好了可以接收流量**。当一个 Pod 内的所有 容器都准备好了，才能把这个 Pod 看作就绪了。用途就是:Service后端负载均衡多个 Pod，如果某个Pod还没就绪，就会从service负载均衡里面剔除



## 工作负载

​		ReplicationController (RC)用来确保容器应用的副本数始终保持在用户定义的副本数，即如果有容器异常退出，会自动创建新的pod来替代；而异常多出来的容器也会自动回收。

​		在新版的Kubernetes中建议使用ReplicaSet (RS)来取代ReplicationController。ReplicaSet跟ReplicationController没有本质的不同，只是名字不一样，但ReplicaSet支持集合式selector。

### Deployment

​		一个 Deployment 为 Pods 和 ReplicaSets 提供声明式的更新能力。yaml文件负责描述 Deployment 中的 ，而 Deployment **控制器(**Controller**)** 以受控速率更改**实 际状态**， 使其变为期望状态


> **更新机制**
>
> - 仅当 Deployment Pod 模板(即 .spec.template )发生改变时，例如**模板的标签或容器镜像被更 新， 才会触发** Deployment **上线**。 **其他更新(如对** Deployment **执行扩缩容的操作)不会触发上线 动作。
>
> -  上线动作 原理: 创建新的**rs**，准备就绪后，替换旧的**rs**(此时不会删除，因为**revisionHistoryLimit **指定了保留几个版本)**

### DaemonSet

DaemonSet 控制器确保所有(或一部分)的节点都运行了一个指定的 Pod 副本。

- 每当向集群中添加一个节点时，指定的 Pod 副本也将添加到该节点上 
- 当节点从集群中移除时，Pod 也就被垃圾回收了
-  删除一个 DaemonSet 可以清理所有由其创建的 Pod

> DaemonSet 的典型使用场景有:
>
> 1. 在每个节点上运行集群的存储守护进程，例如 glusterd、ceph 
> 2. 在每个节点上运行日志收集守护进程，例如 fluentd、logstash 
> 3. 在每个节点上运行监控守护进程，例如 Prometheus Node Exporter 、 Sysdig Agent 、collectd等

### StatefulSet

​		有状态副本集;Deployment等属于无状态的应用部署(stateless)
 StatefulSet 使用场景;对于有如下要求的应用程序，StatefulSet 非常适用:

- **稳定、唯一的网络标识(**dnsname**)**

​		StatefulSet **通过与其相关的无头服务为每个**pod**提供**DNS**解析条目** 。假如无头服务的DNS条目为:"$(service name).$(namespace).svc.cluster.local"，那么pod的解析条目就是"$(pod name).$(service name).$(namespace).svc.cluster.local"，每个pod name也是唯一的。

- **稳定的、持久的存储;【每个**Pod**始终对应各自的存储路径****(**PersistantVolumeClaimTemplate**)】**

-  有序的、优雅的部署和缩放。【按顺序地增加副本、减少副本，并在减少副本时执行清理】
-   有序的、自动的滚动更新。【按顺序自动地执行滚动更新】



### Job**、**CronJob

​		Kubernetes中的 Job 对象将创建一个或多个 Pod，并确保指定数量的 Pod 可以成功执行到进程正常结 束:

- 当 Job 创建的 Pod 执行成功并正常结束时，Job 将记录成功结束的 Pod 数量 
- 当成功结束的 Pod 达到指定的数量时，Job 将完成执行
-  删除 Job 对象时，将清理掉由 Job 创建的 Pod

CronJob 按照预定的时间计划(schedule)创建 Job(注意:启动的是Job不是Deploy，rs)。一个 CronJob 对象类似于 crontab (cron table) 文件中的一行记录。该对象根据 Cron 格式定义的时间计划， 周期性地创建 Job 对象。



## 垃圾回收(GC)

​		Kubernetes garbage collector(垃圾回收器)的作用是删除那些曾经有 owner，后来又不再有 owner 的 对象和描述

### 垃圾收集器如何删除从属对象

​		当删除某个对象时，可以指定该对象的从属对象是否同时被自动删除，这种操作叫做级联删除 (cascading deletion)。级联删除有两种模式:后台(background)和前台(foreground)

​		如果删除对象时不删除自动删除其从属对象，此时，从属对象被认为是孤儿(或孤立的 orphaned)

