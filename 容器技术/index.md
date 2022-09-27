# 容器技术


<!--more-->

# Docker

## Docker架构

![截屏2022-09-18 下午7.32.48](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-18%20%E4%B8%8B%E5%8D%887.32.48.png)

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

### namespace

1. 6项离隔 (资源隔离)

| namespace类型 | 系统调用参数  | 隔离内容                   |
| :------------ | :-----------: | -------------------------- |
| UTS           | CLONE_NEWUTS  | 主机和域名                 |
| IPC           | CLONE_NEWIPC  | 信号量、消息队列和共享内存 |
| PID           | CLONE_NEWPID  | 进程编号                   |
| Network       | CLONE_NEWNET  | 网络设备、网络栈、端口等   |
| Mount         |  CLONE_NEWNS  | 挂载点(文件系统)           |
| User          | CLONE_NEWUSER | 用户和用户组               |

2. linux Namespace 是一种Linux Kernel提供的资源隔离方案:
   1. 系统为进城分配不同的Namespace
   2. 保证不同namespace资源独立分配、进程彼此隔离

```c
// 进程数据结构
struct tash_struct{
	...
  /* namespace */
  struct nsproxy *nsproxy;
  ...
}
// namespace 数据结构
struct nsproxy {
  atomic_t count;
  struct uts_namespace *uts_ns
  struct ipc_namespace *ipc_ns
  struct mnt_namespace *mnt_ns
  struct pid_namespace *pid_ns_for_children
  struct net *net_ns
}
```

3. Linux 对namespace操作方法

- clone: 在创建新进程的系统调用,可以通过flags参数指定需要新建的namespace类型: 

```c
// flags 可以为CLONE_NEWCGROUPS / CLONE_NEWIPC ...
int clone(int (*fn)(void*), void *child_stack, int flags, void *arg)
```

- sets: 该系统调用可以让调用进程加入某个已经存在的namespace中

```c
int setns(int fd, int nstype)
```

- unshare : 该系统调用可以将调用进程移动到新的namespace下

```c
int unshare(int flags)
```

### 隔离机制的问题

​		容器的隔离不彻底问题:

1. 多个容器之间使用的还是同一宿主机的操作系统内核

> 尽管可以在容器里通过 `Mount Namespace` 单独挂载其他不同版本的操作系统文件, 但这并不能改变共享宿主机内核的事实! 这代表如果要在Windows宿主机上运行Linux容器，或者在低版本的Linux宿主机上运行高版本的Linux容器，都是impossible!

2. Linux内核中很多资源和对象是不能被Namespace化的

> 最典型的例子：时间; 如果你的容器中的程序使用settimeofday(2)系统调用修改了时间，整个宿主机的时间都会被随之修改，这显然不符合用户的预期; kernel 5.6中已经有time namespace解决此问题

3. 容器给应用暴露出来的攻击面是相当大的

> 应用“越狱”的难度自然也比虚拟机低得多。尽管可以通过Seccomp等技术过滤和甄别容器内部发起的所有系统调用来进行安全加固, 但一定会拖累容器的性能; 何况，默认情况下，谁也不知道到底该开启哪些系统调用，禁止哪些系统调用。

### cgroups 资源限制

​		Cgroups(control Groups)是linux下用于对一个或一组进程进行资源控制和监控的机制; 可以对诸如CPU使用时间、内存、磁盘IP等进程所需的资源进行限制;不同资源的具体管理工作由相应的Cgroup子系统(Subsystem)来实现; 针对不同类型的资源限制,只需要将限制策略在不同的子系统上进行关联即可;

​		Cgroups在不同的系统资源管理子系统中以层级树(Hierarchy)的方式来组织管理;每个Cgroup都可以包含其他的子Cgroup,因此子Cgroup能使用的资源除了受到本Cgroup配置的资源参数限制,还受到父Cgroups的资源限制;即把一个cgroup目录中的资源划分给它的子目录，子目录可以把资源继续划分给它的子目录，为子目录分配的资源之和不能超过父目录，进程或者线程可以使用的资源受到它们委身的目录的限制。

```c
// 进程数据结构
struct tash_struct{
	#ifdef CONFIG_CGROUPS
  struct css_set__rcu *cgroups;
  struct list_head cg_list;
  #endif
}
// css_set 是cgroup_subsys_state对象的集合数据结构
struct css_set{
  struct cgroup_subsys_state *subsys[CGROUP_SUBSYS_COUNT];
}
```

![截屏2022-09-16 下午3.35.37](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-16%20%E4%B8%8B%E5%8D%883.35.37.png)

cgroup提供的主要功能如下:

1. 限制进程组可以使用的资源数量（Resource limiting ）。比如：memory子系统可以为进程组设定一个memory使用上限，一旦进程组使用的内存达到限额再申请内存，就会出发OOM（out of memory）。
2. 进程组的优先级控制（Prioritization ）。比如：可以使用cpu子系统为某个进程组分配特定cpu share。
3. 记录进程组使用的资源数量（Accounting ）。比如：可以使用cpuacct子系统记录某个进程组使用的cpu时间
4. 进程组隔离（Isolation）。比如：使用ns子系统可以使不同的进程组使用不同的namespace，以达到隔离的目的，不同的进程组有各自的进程、网络、文件系统挂载空间。
5. 进程组控制（Control）。比如：使用freezer子系统可以将进程组挂起和恢复。
   

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

Cgroups主要由task,cgroup,subsystem及hierarchy构成：

- task：在Cgroups中，task就是系统的一个进程。
- cgroup：Cgroups中的资源控制都以cgroup为单位实现的。cgroup表示按照某种资源控制标准划分而成的任务组，包含一个或多个子系统。一个任务可以加入某个cgroup，也可以从某个cgroup迁移到另外一个cgroup。
- subsystem：Cgroups中的subsystem就是一个资源调度控制器(Resource Controller)。比如CPU子系统可以控制CPU时间分配，内存子系统可以限制cgroup内存使用量。
- hierarchy：hierarchy由一系列cgroup以一个树状结构排列而成，每个hierarchy通过绑定对应的subsystem进行资源调度。hierarchy中的cgroup节点可以包含零或多个子节点，子节点继承父节点的属性。整个系统可以有多个hierarchy。

### cgroups 层级结构

​		内核使用 cgroup 结构体来表示一个 control group 对某一个或者某几个 cgroups 子系统的资源限制。cgroup 结构体可以组织成一颗树的形式，每一棵cgroup 结构体组成的树称之为一个 cgroups 层级结构。

1. cgroups层级结构可以 attach 一个或者几个 cgroups 子系统，当前层级结构可以对其 attach 的 cgroups 子系统进行资源的限制。每一个 cgroups 子系统只能被 attach 到一个层级结构中。

![image20210614160004934](https://raw.githubusercontent.com/NoobMidC/pics/main/2a6ee33a486594ba43ecc4bb9e2593c4-20220919185533084.png)

2. 创建了 cgroups 层级结构中的节点（cgroup 结构体）之后，可以把进程加入到某一个节点的控制任务列表中，一个节点的控制列表中的所有进程都会受到当前节点的资源限制。同时某一个进程也可以被加入到不同的 cgroups 层级结构的节点中，因为不同的 cgroups 层级结构可以负责不同的系统资源。所以说进程和 cgroup 结构体是一个多对多的关系。

![image-20210614160108105](https://raw.githubusercontent.com/NoobMidC/pics/main/bcf34c062946d8ab7b6aff598653b8d5.png)

> 上面这个图从整体结构上描述了进程与 cgroups 之间的关系。最下面的P代表一个进程。每一个进程的描述符中有一个指针指向了一个辅助数据结构css_set（cgroups subsystem set）。 指向某一个css_set的进程会被加入到当前css_set的进程链表中。一个进程只能隶属于一个css_set，一个css_set可以包含多个进程，隶属于同一css_set的进程受到同一个css_set所关联的资源限制。
> 一个css_set关联多个 cgroups 层级结构的节点时，表明需要对当前css_set下的进程进行多种资源的控制。而一个 cgroups 节点关联多个css_set时，表明多个css_set下的进程列表受到同一份资源的相同限制。

3. 一个task不能存在于同一个hierarchy的不同cgroup，但可以存在在不同hierarchy中的多个cgroup

> 系统每次新建一个hierarchy时，该系统上的所有task默认构成了这个新建的hierarchy的初始化cgroup，这个cgroup也称为root cgroup。
>
> 对于你创建的每个hierarchy，task只能存在于其中一个cgroup中，即一个task不能存在于同一个hierarchy的不同cgroup中，但是一个task可以存在在不同hierarchy中的多个cgroup中。
>
> 如果操作时把一个task添加到同一个hierarchy中的另一个cgroup中，则会从第一个cgroup中移除。

如下图，cpu和memory被附加到cpu_mem_cg的hierarchy。而net_cls被附加到net hierarchy。并且httpd进程被同时加到了cpu_mem_cg hierarchy的cg1 cgroup中和net hierarchy的cg3 cgroup中。并通过两个hierarchy的subsystem分别对httpd进程进行cpu,memory及网络带宽的限制。

![image-20210614161534314](https://raw.githubusercontent.com/NoobMidC/pics/main/3df363a36e0f1db76dee865a73c1d6e2.png)

4. 子task继承父task cgroup的关系

> 系统中的任何一个task(Linux中的进程)fork自己创建一个子task(子进程)时，子task会自动的继承父task cgroup的关系，在同一个cgroup中，但是子task可以根据需要移到其它不同的cgroup中。父子task之间是相互独立不依赖的。

如下图，httpd进程在cpu_and_mem hierarchy的/cg1 cgroup中并把PID 4537写到该cgroup的tasks中。之后httpd(PID=4537)进程fork一个子进程httpd(PID=4840)与其父进程在同一个hierarchy的统一个cgroup中，但是由于父task和子task之间的关系独立不依赖的，所以子task可以移到其它的cgroup中。

![image-20210614161733536](https://raw.githubusercontent.com/NoobMidC/pics/main/d8934c87f88938ef77e9f73cf9931040.png)

### Cgroup版本

​		与v1不同，cgroup v2仅具有单个进程层次结构，并且在进程之间进行区分，而不对线程进行区分。在cgroup v2中，所有已安装的控制器都位于一个统一的层次结构中。尽管（不同的）控制器可以同时安装在v1和v2层次结构下，但是不可能同时在v1和v2层次结构下同时安装相同的控制器。

- Cgroups v2提供了安装所有控制器所依据的统一层次结构。

- 不允许“内部”过程。除根cgroup以外，进程只能驻留在叶节点（本身不包含子cgroup的cgroup）中。
- 必须通过文件cgroup.controllers 和cgroup.subtree_control指定活动的cgroup 。
- 该任务的文件已被删除。此外，cpuset 控制器使用的 cgroup.clone_children文件已被删除。
- cgroup.events文件提供了一种用于通知空cgroup的改进机制 。



### cpu绑核操作

1. 使用sched_setaffinity系统调用,可以将某个进程绑定到一个特定的CPU。

```c
int sched_getaffinity(pid_t pid, size_t cpusetsize, cpu_set_t *mask);/* 获得pid所指示的进程的CPU位掩码,并将该掩码返回到mask所指向的结构中 */
```

2. 绑定线程到cpu核上使用pthread_setaffinity_np函数系统调用

```c
int pthread_setaffinity_np(pthread_t thread, size_t cpusetsize, const cpu_set_t *cpuset);
int pthread_getaffinity_np(pthread_t thread, size_t cpusetsize, cpu_set_t *cpuset);
```

## Docker存储

### 镜像存储

#### Overlay2

​		overlay2文件系统:  overlay2 的目录是镜像和容器分层的基础，而把这些层统一展现到同一的目录下的过程称为联合挂载（union mount）;**overlay2 把目录的下一层叫作lowerdir，上一层叫作upperdir，联合挂载后的结果叫作merged; workdir为临时目录**

​		**总体来说，overlay2 是这样储存文件的：overlay2将镜像层和容器层都放在单独的目录，并且有唯一 ID，每一层仅存储发生变化的文件，最终使用联合挂载技术将容器层和镜像层的所有文件统一挂载到容器中，使得容器中看到完整的系统文件。**

> 镜像中:  **MergedDir 代表当前镜像层在 overlay2 存储下的目录，LowerDir 代表当前镜像的父层关系，使用冒号分隔，冒号最后代表该镜像的最底层。**
>
> 容器中:**lower 文件为该层的所有父层镜像的短 ID 。diff 目录为容器的读写层，容器内修改的文件都会在 diff 中出现，merged 目录为分层文件联合挂载后的结果，也是容器内的工作目录。**	

![img](https://raw.githubusercontent.com/NoobMidC/pics/main/v2-70619bfa96ffb2a0bc80d422a3926061_1440w.jpg)

- 容器run起来时对应的3个层：

- - image layer (只读)，镜像的层
  - init layer 容器在启动时写入的一些配置文件，发生在 container layer之前
  - container layer 新增的可写层

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

- 好处是减少镜像体积，提升启动速度，
- 缺点就是写入的速度慢，所以在 container layer 中不适合进行大量的文件读写，应该使用Volume

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



## 容器级别

### Docker 组件

![img](https://raw.githubusercontent.com/NoobMidC/pics/main/watermark%252Ctype_ZmFuZ3poZW5naGVpdGk%252Cshadow_10%252Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2RpbGxhbnpob3U%253D%252Csize_16%252Ccolor_FFFFFF%252Ct_70.png)

- docker组件只是一个最外围的入口，为使用者提供一种命令行形式的客户端(CLI)来执行容器的各种操作，使用golang实现。docker客户端将用户输入的命令和参数转换为后端服务的调用参数，通过调用后端服务来实现各类容器操作。
- dockerd：dockerd是运行于服务器上的后台守护进程（daemon），负责实现容器镜像的拉取和管理以及容器创建、运行等各类操作。dockerd向外提供RESTful API，其他程序（例如docker客户端）可以通过API来调用dockerd的各种功能，实现对容器的操作。但时至今日，在dockerd中实现的容器管理功能也已经不多，**主要是镜像下载和管理相关的功能**，其他的容器操作能力已经分离到containerd组件中，通过grpc接口来调用。又被称为docker engine、docker daemon。
- containerd：containerd是另一个后台守护进程，是真正**实现容器创建、运行、销毁等各类操作**的组件，它也包含了独立于dockerd的镜像下载、上传和管理功能。containerd向外暴露grpc形式的接口来提供容器操作能力。dockerd在启动时会自动启动containerd作为其容器管理工具，当然containerd也可以独立运行。containerd是从docker中分离出来的容器管理相关的核心能力组件，是为了支持容器功能实现的灵活性和开放性，更底层的容器操作实现（例如cgroup的创建和管理、namespace的创建和使用等）并不是由containerd提供的，而是通过调用另一个组件runc来实现。
- runC: runc实现了容器的底层功能，例如创建、运行等。runc通过调用内核接口为容器创建和管理cgroup、namespace等Linux内核功能，来实现容器的核心特性。
- containerd-shim：除了这些主要组件外，图中还有containerd-shim这个组件。containerd-shim位于containerd和runc之间，当containerd需要创建运行容器时，它没有直接运行runc，而是运行了shim，再由shim间接的运行runc; shim主要有3个用途：
  1. 让runc进程可以退出，不需要一直运行。这样设计的原因可能还是想让runc的功能集中在容器核心功能本身，同时也便于runc的后续升级。shim作为一个简单的中间进程，不太需要升级，其他组件升级时它可以保持运行，从而不影响已运行的容器。
  2. 作为容器中进程的父进程，为容器进程维护stdin等管道fd。如果containerd直接作为容器进程的父进程，那么一旦containerd需要升级重启，就会导致管道和tty master fd被关闭，容器进程也会执行异常而退出。

对于容器运行时来说, 主要有两个级别Low Level(接近内核层)和High Level(接近用户层)

![截屏2022-09-19 下午8.38.55](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-19%20%E4%B8%8B%E5%8D%888.38.55.png)

- lxc: lxc是最早的linux容器技术，早期版本的docker直接使用lxc(后面被runc代替)来实现容器的底层功能。虽然使用者相对较少，但lxc项目仍在持续开发演进中。

- **libcontainer：**docker从0.9版本开始自行开发了libcontainer模块来作为lxc的替代品实现容器底层特性，并在1.10版本彻底去除了lxc。在1.11版本拆分出runc后，libcontainer也随之成为了runc的核心功能模块。
- CRI：CRI是Container Runtime Interface（容器运行时接口）的缩写。如上文所述，它是k8s团队提出的容器操作接口标准，符合CRI标准的容器模块才能集成到k8s体系中与kubelet交互。符合CRI的容器技术模块包括dockershim（用于兼容dockerd）、rktlet（用于兼容rkt）、containerd(with CRI plugin)、CRI-O等。
- rkt与rktlet：rkt是CoreOS公司主导的容器技术，在早期得到了k8s的支持成为k8s集成的两种容器技术之一。随着CRI接口的提出，k8s团队也为rkt提供了rktlet模块用于与rkt交互，rktlet和dockersim的意义基本相同。随着CoreOS被Redhat收购，rkt已经停止了研发，rktlet已停止维护了。
- CRI-O：CRI-O是Redhat公司推出的容器技术。从名字就能看出CRI-O的出发点就是一种原生支持CRI接口规范的容器技术。CRI-O同时兼容OCI接口和docker镜像格式。CRI-O的设计目标和特点在于它是一项轻量级的技术，k8s可以通过使用CRI-O来调用不同的底层容器运行时模块，例如runc。
- OCI：OCI是Open Container Initiative（开放容器倡议）的缩写。OCI是以docker为首的容器技术公司创建的组织，也是这个组织制定的容器相关标准的统称。OCI标准主要包括两部分：镜像标准和运行时标准。符合OCI运行时标准的容器底层实现模块能够被containerd、CRI-O等容器操作模块集成调用。runc就是从docker中拆分出来捐献给OCI组织的底层实现模块，也是第一个支持OCI标准的模块。除了runc外，还有gVisor（runsc）、kata等其他符合OCI标准的实现。
- dockershim：kubelet并没有直接和dockerd交互，而是通过了一个dockershim的组件间接操作dockerd。dockershim提供了一个标准的接口，让kubelet能够专注于容器调度逻辑本身，而不用去适配dockerd的接口变动。而其他实现了相同标准接口的容器技术也可以被kubelet集成使用，这个接口称作CRI。

![img](https://raw.githubusercontent.com/NoobMidC/pics/main/v2-10a11376ea876b10f79464fd66e44dcf_1440w.jpg)

# K8S

## 架构图

![截屏2022-09-19 下午9.28.20](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-19%20%E4%B8%8B%E5%8D%889.28.20.png)

![截屏2022-09-19 下午9.29.01](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-19%20%E4%B8%8B%E5%8D%889.29.01.png)



## OCI标准

OCI即Open Container Initiative(轻量级开放式管理项目),OCI（Open Container Initiative）规范是事实上的容器标准，已经被大部分容器实现以及容器编排系统所采用。任何实现了OCI规范的工具都可以打镜像

- OCI主要定义两个规范
  1. Runtime Specification(运行时标准): 文件系统包如何解压至硬盘,  运行时运行方式(以来namespace和cgroups)
  2. Image Specification(镜像标准): 如何通过构建系统打包, 生成镜像清单(Manifest)、文件系统序列化文件、镜像配置;(overlays)

- 规范要求镜像内容包括以下几个部分:

  三个必须的:

  1. [Image Manifest](https://link.zhihu.com/?target=https%3A//github.com/opencontainers/image-spec/blob/main/manifest.md) ：提供了镜像的配置和文件系统层定位信息，可以看作是镜像的目录，文件格式为 json 
  2. [Image Layer Filesystem Changeset](https://link.zhihu.com/?target=https%3A//github.com/opencontainers/image-spec/blob/main/layer.md) ：序列化之后的文件系统和文件系统变更，它们可按顺序一层层应用为一个容器的 rootfs，因此通常也被称为一个 layer（与下文提到的镜像层同义），文件格式可以是 tar ，gzip 等存档或压缩格式。
  3. [Image Configuration](https://link.zhihu.com/?target=https%3A//github.com/opencontainers/image-spec/blob/main/config.md) ：包含了镜像在运行时所使用的执行参数以及有序的 rootfs 变更信息，文件类型为 json。

  一个可选的:

  1. [image-index](https://link.zhihu.com/?target=https%3A//github.com/opencontainers/image-spec/blob/main/image-index.md) : 图像索引是一种更高级别的清单，它指向特定的图像清单，非常适合一个或多个平台

## CRI

​		Container Runtime Interface：容器运行时接口，提供计算资源;

​		Container Runtime实现了CRI gRPC Server，包括`RuntimeService`和`ImageService`。该gRPC Server需要监听本地的Unix socket，而kubelet则作为gRPC Client运行。

![img](https://raw.githubusercontent.com/NoobMidC/pics/main/v2-e6f73288fd955cf0e642962fbe301fac_1440w.jpg)

![img](https://raw.githubusercontent.com/NoobMidC/pics/main/v2-ab209f7c32ceb17ed43dcf6b66056cea_1440w.jpg)

- ImageService : 主要是拉取镜像、查看和删除镜像等操作
- RuntimeService : 用来管理 Pod 和容器的生命周期，以及与容器交互的调用（exec/attach/port-forward）等操作

```go
service RuntimeService {
    // Version returns the runtime name, runtime version, and runtime API version.
    rpc Version(VersionRequest) returns (VersionResponse) {}
    // RunPodSandbox creates and starts a pod-level sandbox. Runtimes must ensure
    // the sandbox is in the ready state on success.
    rpc RunPodSandbox(RunPodSandboxRequest) returns (RunPodSandboxResponse) {}
    // StopPodSandbox stops any running process that is part of the sandbox and
    // reclaims network resources (e.g., IP addresses) allocated to the sandbox.
    // If there are any running containers in the sandbox, they must be forcibly
    // terminated.
    // This call is idempotent, and must not return an error if all relevant
    // resources have already been reclaimed. kubelet will call StopPodSandbox
    // at least once before calling RemovePodSandbox. It will also attempt to
    // reclaim resources eagerly, as soon as a sandbox is not needed. Hence,
    // multiple StopPodSandbox calls are expected.
    rpc StopPodSandbox(StopPodSandboxRequest) returns (StopPodSandboxResponse) {}
    // RemovePodSandbox removes the sandbox. If there are any running containers
    // in the sandbox, they must be forcibly terminated and removed.
    // This call is idempotent, and must not return an error if the sandbox has
    // already been removed.
    rpc RemovePodSandbox(RemovePodSandboxRequest) returns (RemovePodSandboxResponse) {}
    // PodSandboxStatus returns the status of the PodSandbox. If the PodSandbox is not
    // present, returns an error.
    rpc PodSandboxStatus(PodSandboxStatusRequest) returns (PodSandboxStatusResponse) {}
    // ListPodSandbox returns a list of PodSandboxes.
    rpc ListPodSandbox(ListPodSandboxRequest) returns (ListPodSandboxResponse) {}

    // CreateContainer creates a new container in specified PodSandbox
    rpc CreateContainer(CreateContainerRequest) returns (CreateContainerResponse) {}
    // StartContainer starts the container.
    rpc StartContainer(StartContainerRequest) returns (StartContainerResponse) {}
    // StopContainer stops a running container with a grace period (i.e., timeout).
    // This call is idempotent, and must not return an error if the container has
    // already been stopped.
    // TODO: what must the runtime do after the grace period is reached?
    rpc StopContainer(StopContainerRequest) returns (StopContainerResponse) {}
    // RemoveContainer removes the container. If the container is running, the
    // container must be forcibly removed.
    // This call is idempotent, and must not return an error if the container has
    // already been removed.
    rpc RemoveContainer(RemoveContainerRequest) returns (RemoveContainerResponse) {}
    // ListContainers lists all containers by filters.
    rpc ListContainers(ListContainersRequest) returns (ListContainersResponse) {}
    // ContainerStatus returns status of the container. If the container is not
    // present, returns an error.
    rpc ContainerStatus(ContainerStatusRequest) returns (ContainerStatusResponse) {}
    // UpdateContainerResources updates ContainerConfig of the container.
    rpc UpdateContainerResources(UpdateContainerResourcesRequest) returns (UpdateContainerResourcesResponse) {}

    // ExecSync runs a command in a container synchronously.
    rpc ExecSync(ExecSyncRequest) returns (ExecSyncResponse) {}
    // Exec prepares a streaming endpoint to execute a command in the container.
    rpc Exec(ExecRequest) returns (ExecResponse) {}
    // Attach prepares a streaming endpoint to attach to a running container.
    rpc Attach(AttachRequest) returns (AttachResponse) {}
    // PortForward prepares a streaming endpoint to forward ports from a PodSandbox.
    rpc PortForward(PortForwardRequest) returns (PortForwardResponse) {}

    // ContainerStats returns stats of the container. If the container does not
    // exist, the call returns an error.
    rpc ContainerStats(ContainerStatsRequest) returns (ContainerStatsResponse) {}
    // ListContainerStats returns stats of all running containers.
    rpc ListContainerStats(ListContainerStatsRequest) returns (ListContainerStatsResponse) {}

    // UpdateRuntimeConfig updates the runtime configuration based on the given request.
    rpc UpdateRuntimeConfig(UpdateRuntimeConfigRequest) returns (UpdateRuntimeConfigResponse) {}

    // Status returns the status of the runtime.
    rpc Status(StatusRequest) returns (StatusResponse) {}
}
```

```go
// ImageService defines the public APIs for managing images.
service ImageService {
    // ListImages lists existing images.
    rpc ListImages(ListImagesRequest) returns (ListImagesResponse) {}
    // ImageStatus returns the status of the image. If the image is not
    // present, returns a response with ImageStatusResponse.Image set to
    // nil.
    rpc ImageStatus(ImageStatusRequest) returns (ImageStatusResponse) {}
    // PullImage pulls an image with authentication config.
    rpc PullImage(PullImageRequest) returns (PullImageResponse) {}
    // RemoveImage removes the image.
    // This call is idempotent, and must not return an error if the image has
    // already been removed.
    rpc RemoveImage(RemoveImageRequest) returns (RemoveImageResponse) {}
    // ImageFSInfo returns information of the filesystem that is used to store images.
    rpc ImageFsInfo(ImageFsInfoRequest) returns (ImageFsInfoResponse) {}
} 
```

#### 支持CRI的后段

- [cri-o](https://link.zhihu.com/?target=https%3A//github.com/kubernetes-incubator/cri-o)：同时兼容OCI和CRI的容器运行时
- [cri-containerd](https://link.zhihu.com/?target=https%3A//github.com/containerd/cri-containerd)：基于[Containerd](https://link.zhihu.com/?target=https%3A//github.com/containerd/containerd)的Kubernetes CNI实现
- [rkt](https://link.zhihu.com/?target=https%3A//coreos.com/rkt/)：由于CoreOS主推的用来跟docker抗衡的容器运行时
- [frakti](https://link.zhihu.com/?target=https%3A//github.com/kubernetes/frakti)：基于hypervisor的CRI
- [docker](https://link.zhihu.com/?target=https%3A//www.docker.com/)：kuberentes最初就开始支持的容器运行时，目前还没完全从kubelet中解耦，docker公司同时推广了[OCI](https://link.zhihu.com/?target=https%3A//www.opencontainers.org/)标准
- [clear-containers](https://link.zhihu.com/?target=https%3A//github.com/clearcontainers)：由Intel推出的同时兼容OCI和CRI的容器运行时
- [kata-containers](https://link.zhihu.com/?target=https%3A//katacontainers.io/)：符合OCI规范同时兼容CRI



## CNI

​		Container Network Interface: 容器网络接口; 是CNCF旗下的一个项目，由一组用于配置Linux容器的网络接口的规范和库组成，同时还包含了一些插件。CNI仅关心容器创建时的网络分配，和当容器被删除时释放网络资源。

CNI的接口中包括以下几个方法：

```go
type CNI interface {
    AddNetworkList(net *NetworkConfigList, rt *RuntimeConf) (types.Result, error) // 添加网络
    DelNetworkList(net *NetworkConfigList, rt *RuntimeConf) error // 删除网络

    AddNetwork(net *NetworkConfig, rt *RuntimeConf) (types.Result, error) // 添加网络列表
    DelNetwork(net *NetworkConfig, rt *RuntimeConf) error // 删除网络列表
} 
```



## CSI

​		 Container Storage Interface: 容器存储接口; CSI 代表[容器存储接口](https://link.zhihu.com/?target=https%3A//github.com/container-storage-interface/spec/blob/master/spec.md)，CSI 试图建立一个行业标准接口的规范，借助 CSI 容器编排系统（CO）可以将任意存储系统暴露给自己的容器工作负载。





## 资源对象

![截屏2022-09-20 下午6.36.49](https://raw.githubusercontent.com/NoobMidC/pics/main/截屏2022-09-20 下午6.36.49.png)

> 对于k8s集群应用来说,  所有的程序应用都是
>
> - 运行在Pod资源对象里
> - 借助于service资源对象向外提供服务访问
> - 借助于各种存储资源实现数据的可持久化
> - 借助于各种配置资源对象实现配置属性、敏感信息饿管理操作

> 资源对象的分类:
>
> - 工作负载型资源: Pod、Deployment、Daemonset、Replica、StatefulSet、Job、Cronjob、Operator
> - 服务发现和负载均衡资源: Service、Ingress
> - 配置和存储: configMap、Secret、PersistentVolume、PersistentVolumeChain、 DownwardAPI
> - 动态调整资源: HPA、VPA
> - 资源隔离权限控制资源: namespace、nodes、clusterroles、Roles

### Pod

- 工作负载(Workloads)控制一组Pod;Pod控制一组容器(Containers)
- Pod 天生地为其成员容器提供了两种共享资源: **网络** 和 **存储** 。
- 一个Pod由一个 Pause**容器** 设置好整个Pod里面所有容器的网络、名称空间等信息

![截屏2022-09-07 上午10.03.25](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-07%20%E4%B8%8A%E5%8D%8810.03.25.png)



#### 设计模式

- sidecar(边车模式):  pod 中的一个额外容器，用于**增强**或**扩展**主容器的功能。
- Adapter(适配器模式): **转换主容器的输出**的容器。
- Ambassador(大使模式): 将网络连接**代理**到主容器的容器。

#### 完整结构

![截屏2022-09-20 下午7.09.35](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-20%20%E4%B8%8B%E5%8D%887.09.35.png)

> 1. 一个pod可以有多个容器, 彼此间共享网络和存储资源; 每个pod中有一个pause容器保存所有容器的状态, 通过管理pause容器达到管理pod中所有容器的效果;
> 2. 同个pod中的容器总会被调度到相同的Node节点,不同节点间的pod通信是基于虚拟二层网络技术实现的;
> 3. 每个pod都是应用的一个实例,有专门的ip,与容器暴露的端口组合为一个service的endpoint地址(每个service由kube-proxy转换为本地的ipvs或iptables规则)
> 4. 每个service的endpoint地址由coredns组件解析为对应的服务名称,其他的service的pod通过访问该名称达到应用间通信的效果
> 5. 外部流量通过节点网卡进入k8s集群,基于ipvs或iptables规则找到对应的service进而找到对应的pod地址

#### 资源管理

- **通信机制**: pod内又个Pause容器,其有独立的ip地址,其他容器基于container网络模式实现统一的对外服务, 大大简化了关联业务容器之间的通信问题;
- **存储机制**: pod内有专用的数据存储资源对象, 其他容器共享该数据卷, 实现多容器数据信息的统一存储;
- 所有子应用容器信息都保存在pause容器中, 通过对pause容器的管理, 达到管理所有容器的效果



#### 创建流程

![截屏2022-09-21 下午6.06.16](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-21%20%E4%B8%8B%E5%8D%886.06.16.png)

1. 用户向master节点上的API-Server发起创建一个Pod的请求
2. apiserver 将信息写入etcd
3. scheduluer检测api-Server上有建立Pod请求, 开始调度该Pod到指定的Node, 同时更新信息到etcd
4. kubelet检测到有新的Pod调度过来, 通过Docker引擎运行该pod对象
5. kubelet通过container runtime 取到Pod状态, 并同步信息到apiserver, 由它更新信息到etcd

#### 生命周期

![截屏2022-09-21 下午6.21.52](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-21%20%E4%B8%8B%E5%8D%886.21.52.png)

![截屏2022-09-21 下午6.22.34](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-21%20%E4%B8%8B%E5%8D%886.22.34.png)

- 启动流程: 初始化容器独立于主容器之外, pod可以拥有任意数量init容器、 init容器顺序执行;最后一个init容器执行完成后才启动主容器;(init容器主要是为了为主容器准备应用的功能, 比如向主容器的存储卷写入数据, 然后将存储卷挂在到主容器)
- 关闭pod流程: 当API服务器接收到删除pod对象的命令后, 按照下面的流程进行:
  1. 执行pre stop钩子,等待它执行完毕
  2. 向容器的主进程发送SIGTERM信号
  3. 等待容器优雅关闭或者等待终止宽限期超时
  4. 如果容器没有优雅关闭, 使用SIGKILL信号强制终止进程
- 临时容器:线上排错。 有些容器基础镜像。线上没法排错。使用临时容器进入这个Pod。临时容器共享了Pod的所有。临时容器有Debug的一些命令，拍错完成以后，只要exit退出容器，临时容器自动删除

#### 生命周期钩子

- 启动后钩子(post start): 初始化容器启动完成后执行, 与主进程并行运行
- 运行中钩子: Liveiness(判断当前容器是否处于存活状态); Readiness(判断当前容器是否可以正常的对外提供服务);
- 停止前钩子(pre stop): 先执行钩子, 并在钩子执行完成后, 想容器发送SIGTERM; 如果没有优雅终止, 则会被强制杀死容器, 不管执行成功与否容器都会终止, 如果未成功则告警

> 关于钩子函数的执行主要两种方式:
>
> Exec: 用于执行一段特定的命令,不过该命令消耗的资源会计入容器
>
> HTTP: 对容器的特定的端点执行HTTP请求

#### Probe 探针机制

每个容器三种探针(Probe):

1. 启动探针(startupProbe)一次性成功探针。只要启动成功了(initProbe)
   - kubelet 使用启动探针，来检测应用是否已经启动。如果启动完成就可以进行后续的探测检查。慢容器一定指定启动探针。一直在等待启动
   - **启动探针成功以后就不用了，剩下存活探针和就绪探针持续运行**
2. 存活探针(livenessProbe), 周期性检测
   - kubelet 使用存活探针，来检测容器是否正常存活。(有些容器可能产生死锁【应用程序 在运行，但是无法继续执行后面的步骤】)， 如果检测失败就会**重新启动这个容器 **
   - initialDelaySeconds: 3600(长了导致可能应用一段时间不可用) 5(短了陷入无限启动循环)
3. 就绪探针(readinessProbe)
   - kubelet 使用就绪探针，来检测容器是否准备**好了可以接收流量**。当一个 Pod 内的所有 容器都准备好了，才能把这个 Pod 看作就绪了。用途就是:Service后端负载均衡多个 Pod，如果某个Pod还没就绪，就会从service负载均衡里面剔除

探针的实现方式:

1. ExecAction: 直接执行命令, 命令成功返回表示探测成功;
2. TCPSocketAction: 端口能正常打开,即成功
3. HTTPGetAction: 向指定的path发送http请求, 2xx,3xx的响应码表示成功

#### pod状态

| pod状态          | 描述                                                         |
| ---------------- | ------------------------------------------------------------ |
| Pending          | APIserver已经创建该server, 但pod内有一个或多个容器的镜像还未创建, 可能还在下载中 |
| Running          | Pod所有容器已经创建, 且至少一个容器处于运行状态, 正在启动或重启状态 |
| Succeeded        | 所有容器均成功执行退出, 且不再重启                           |
| Failed           | Pod内所有容器都已经退出, 其中至少一个容器退出失败.           |
| Unknown          | 由于某种原因无法获取到pod状态, 比如网络不通                  |
| CrashLookBackOff | K8s曾经启动容器成功, 但是后来异常的情况下, 重启次数过多导致异常终止(容器退避算法: 第一次0秒重启, 第二次10秒重启, 第三次20秒后重启.... 如果重启失败,则置为CrashLookBackOff状态; 其他的比如镜像获取就无效了) |
| Error            | 因为集群配置、安全限制、资源等原因导致pod启动过程中发生了错误 |
| Evicted          | 集群节点系统内存或硬盘资源不足导致pod出现异常                |
|                  |                                                              |

```sh
kubectl explain pods.status.phase # 查看状态
# 业务运行过程中不可避免会出现意外,这个时候有三种策略对pod进行管理: OnFailure、Never、Always(默认)
# Always: 容器失效时,即重启
# OnFailure: 容器终止运行, 且退出码不为0时重启
# Never: pod不重启
```

| 容器状态   | 描述                                      |
| ---------- | ----------------------------------------- |
| Waiting    | 容器处于Running和Terminated状态之前的状态 |
| Running    | 容器能够正常的运行状态                    |
| Terminated | 容器已经被成功的关闭了                    |

```sh
kubectl explain pods.status.containerStatuses.state
```

| 流程状态       | 描述                             |
| -------------- | -------------------------------- |
| PodScheduled   | pod被调度到某个节点              |
| Ready          | 准备就绪,pod可以处理请求         |
| Initialized    | pod中所有初始化容器启动完成      |
| Unschedulable  | 由于资源等限制,导致pod无法被调度 |
| ContainerReady | pod中所有容器都启动完毕          |

```sh
kubectl explain pods.status.conditions.status
```

> 镜像拉取策略:
>
> - always: 总是拉取,默认值
> - IfNotPresent: 如果本地仓库不存在, 再拉取新镜像
> - Never: 不获取新镜像



#### Qos

​		QOS是K8S中的一种资源保护机制，其主要是针对不可压缩资源比如内存的一种控制技术。比如在内存中，其通过为不同的Pod和容器构造OOM评分，并且通过内核策略的辅助，从而实现当节点内存资源不足的时候，内核可以按照策略的优先级，优先kill掉那些优先级比较低（分值越高，优先级越低）的Pod。

​		 QoS（Quality of Service），可译为 "服务质量等级"，或者译作 "服务质量保证"，是作用在 Pod 上的一个配置，当 Kubernetes 创建一个 Pod 时，它就会给这个 Pod 分配一个 QoS 等级。

![img](https://raw.githubusercontent.com/NoobMidC/pics/main/1620.png)

1. **Guaranteed**策略: **该策略下，设置的requests 等于 limits**, 会存在cpu或memory的request和limit。顾名思义是该容器对资源的最低要求和最高使用量限制。如果我们配置了limit，没有配置request，默认会以limit的值来定义request。
2. **BestEffort**策略: **该策略下，没有设置requests 、 limits**, 意味着这个容器想跑多少资源就跑多少资源，其资源使用上限实际上即所在node的capacity。
3. **Burstable**策略: **该策略下，设置的requests 小于 limits**; 当resource.limit和resource.request以上述两种方式以外的形式配置的时候，就会采用本模式。 QoS目前只用cpu和memory来描述，其中cpu可压缩资源，当一个容器的cpu使用率超过limit时会被进行流控，而当内存超过limit时则会被oom_kill。这里kubelet是通过自己计算容器的oom_score，确认相应的linux进程的oom_adj，oom_adj最高的进程最先被oom_kill。 Guaranteed模式的容器oom_score最小：-998，对应的oom_adj为0或1，BestEffort模式则是1000，Burstable模式的oom_score随着其内存使用状况浮动，但会处在2-1000之间。

> 当某个node内存被严重消耗时，BestEffort策略的pod会最先被kubelet杀死，其次Burstable（该策略的pods如有多个，也是按照内存使用率来由高到低地终止），再其次Guaranteed。



### 拓扑控制器

拓扑管理器是一个 Kubelet 组件，扮演信息源的角色，以便其他 Kubelet 组件可以做出与拓扑结构相对应的资源分配决定。对所有 QoS 类的 Pod 执行对齐操作。

- 多种资源管理器在给pod分配设备时，都是独立工作的，不会有一个全局观念，这可能会造成资源分配不合理的问题
- Topology Manager就是提供全局的视角，为了尽量将资源分配在同一个numa节点下，提升性能

拓扑管理器有两个主要的配置

1. 作用域(定义了你希望的资源对齐粒度): container （默认）、pod

2. 拓扑策略(对齐时实际使用的策略): 拓扑管理器支持四种分配策略。
   - none (默认): 不执行任何拓扑对齐。
   - best-effort: 拓扑管理器存储该容器的首选 NUMA 节点亲和性。 如果亲和性不是首选，则拓扑管理器将存储该亲和性，并且无论如何都将 pod 接纳到该节点。
   - restricted: 拓扑管理器存储该容器的首选 NUMA 节点亲和性。 如果亲和性不是首选，则拓扑管理器将从节点中拒绝此 Pod 。 这将导致 Pod 处于 Terminated 状态，且 Pod 无法被节点接纳。
   - single-numa-node: 拓扑管理器确定单 NUMA 节点亲和性是否可能。 如果是这样，则拓扑管理器将存储此信息，然后 *建议提供者* 可以在做出资源分配决定时使用此信息。 如果不可能，则拓扑管理器将拒绝 Pod 运行于该节点。 这将导致 Pod 处于 Terminated 状态，且 Pod 无法被节点接受。

numa的物理结构：

![截屏2022-09-22 下午2.10.46](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-22%20%E4%B8%8B%E5%8D%882.10.46.png)



#### CPU的管理策略

kubelet 使用 [CFS 配额](https://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/Completely_Fair_Scheduler) 来执行 Pod 的 CPU 约束;

CPU 管理策略:

1. None: 默认策略，表示现有的调度行为。显式地启用现有的默认 CPU 亲和方案，不提供操作系统调度器默认行为之外的亲和性策略。 通过 CFS 配额来实现 [Guaranteed pods](https://link.zhihu.com/?target=https%3A//kubernetes.io/zh/docs/tasks/configure-pod-container/quality-service-pod/) 的 CPU 使用限制。
2. static: 允许为节点上具有某些资源特征的 pod 赋予增强的 CPU 亲和性和独占性。针对具有整数型 CPU requests 的 Guaranteed Pod ，它允许该类 Pod 中的容器访问节点上的独占 CPU 资源。这种独占性是使用 [cpuset cgroup 控制器](https://link.zhihu.com/?target=https%3A//www.kernel.org/doc/Documentation/cgroup-v1/cpusets.txt) 来实现的。

> CPU 管理器定期通过 CRI 写入资源更新，以保证内存中 CPU 分配与 cgroupfs 一致。 

## 工作负载

### 部署流程

1. 用户向APIserver中插入一个应用资源的数据形态: 这个数据形态中定义了该资源对象的期望状态, 数据经由APIserver保存到etcd中
2. kube-controller-manager 中的各种控制器监控APIserver 上与自己相关的资源对象的变动; 比如service controller只负责service资源额度控制等;
3. 一旦APIserver中的资源对象发生变动, 对应controller执行相关的配置代码, 到对应的node上运行; 这些资源对象会在当前节点上, 按照用户期望运行; 
4. controller将这些实际的资源对象状态通过APIserver存储到etcd的同一个数据条目的status字段;
5. 资源对象运行过程中,controller会循环的方式向APIserver监控spec和status是否相同; 不相同则只会node节点的资源进行修改,保证两者一致; 状态一直后通过APIserver同步更新到当前资源对象在etcd上的数据;



### RC&RS

#### RC

​		ReplicationController (RC)用来确保容器应用的副本数始终保持在用户定义的副本数，即如果有容器异常退出，会自动创建新的pod来替代；而异常多出来的容器也会自动回收。

​		RC定义了一个用户期望场景, 其由Pod的期望状态包括数量(replicas),筛选标签(Label Selector)和模板(template)



#### RS

​		在新版的Kubernetes中建议使用ReplicaSet (RS)来取代ReplicationController。ReplicaSet跟ReplicationController没有本质的不同，只是名字不一样，但ReplicaSet支持集合式selector。



#### 扩缩容机制

利用 k8s 的 `Deployment/RS` 的 `Scale` 机制来实现服务的扩缩容工作

1. 手动扩缩容: 手动设置rs的replicas数量,则实现扩缩容
2. 自动扩缩容: `HPA（HorizontalPodAutoscaler）` 的控制器，即 Pod 水平扩缩容

> 周期性的监测目标 Pod 的资源性能指标，获取监视指标后将与 HPA 资源对象中的扩缩容条件进行对比，当满足条件时对 Pod 副本数量进行调整。`HPA` 控制器依据 `Metrics Server` 获取资源性能监控指标调整工作负载副本和资源。

HPA工作原理

- `metrics-server` 实现了 `Metrics API`。该 API 允许你访问集群中 Node 节点和 Pod 的 `CPU` 和 `Memory/内存` 使用情况。
- `metrics-server` 的主要作用是 **将资源使用指标提供给 K8s 自动缩放器（Scale）组件**。

### Deployment

​		一个 Deployment 为 Pods 和 ReplicaSets 提供声明式的更新能力。yaml文件负责描述 Deployment 中的 ，而 Deployment **控制器(**Controller**)** 以受控速率更改**实 际状态**， 使其变为期望状态


> **更新机制**
>
> - 仅当 Deployment Pod 模板(即 .spec.template )发生改变时，例如**模板的标签或容器镜像被更 新， 才会触发** Deployment **上线**。 **其他更新(如对** Deployment **执行扩缩容的操作)不会触发上线 动作。
>
> -  上线动作 原理: 创建新的**rs**，准备就绪后，替换旧的**rs**(此时不会删除，因为**revisionHistoryLimit **指定了保留几个版本)



#### 滚动更新

```yaml
strategy:
  type: RollingUpdate # 一种是RollingUpdate，即滚动升级。另一种方式为Recreate。即先将所有旧的Pod停止，然后再启动新的pod。默认策略即为RollingUpdate	
  rollingUpdate:
    maxSurge: 10%
    maxUnavailable: 0    
```

- maxSurge： 指定在升级时，最大可以创建多少个pod。这个值可以是一个绝对值数字，也可以是个百分比。例如，当这个值指定为30%时，也就是说，新旧pod的总量不能超过130%。简单来讲，就是在滚动升级时，会先启动30%的新的pod。然后开始杀掉旧的pod，每当一个旧的pod被杀掉，一个新的pod的会被启动，始终保持总量不超过130%，直至更新完成。需要说明的是，当maxUnavailable为0时，maxSurge的值不能为0。
- maxUnavailable： 指定在升级时，最大不可用的pods的值。可以是一个绝对值数字，也可以是个百分比。例如，当这个值指定为30%时，最少可用的Pod为70%,也就是说，在滚动升级的时候，会先杀掉30%旧的pod，然后开始启动新pod。当一个新的Pod被创建，一个旧的Pod就会被销毁。始终保持可用的pod在总量的70%，直至升级完成。需要说明的是，当maxSurge为0时，maxUnavailable的值不能为0
  

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

- **稳定的、持久的存储;【每个**Pod**始终对应各自的存储路径**(**PersistantVolumeClaimTemplate**)】

-  有序的、优雅的部署和缩放。【按顺序地增加副本、减少副本，并在减少副本时执行清理】
-   有序的、自动的滚动更新。【按顺序自动地执行滚动更新】



### Job**、**CronJob

​		Kubernetes中的 Job 对象将创建一个或多个 Pod，并确保指定数量的 Pod 可以成功执行到进程正常结 束:

- 当 Job 创建的 Pod 执行成功并正常结束时，Job 将记录成功结束的 Pod 数量 
- 当成功结束的 Pod 达到指定的数量时，Job 将完成执行
-  删除 Job 对象时，将清理掉由 Job 创建的 Pod

CronJob 按照预定的时间计划(schedule)创建 Job(注意:启动的是Job不是Deploy，rs)。一个 CronJob 对象类似于 crontab (cron table) 文件中的一行记录。该对象根据 Cron 格式定义的时间计划， 周期性地创建 Job 对象。





## 存储

Kubernetes Volume（数据卷）主要解决了如下两方面问题：

- 数据持久性：通常情况下，容器运行起来之后，写入到其文件系统的文件暂时性的。当容器崩溃后，kubelet 将会重启该容器，此时原容器运行后写入的文件将丢失，因为容器将重新从镜像创建。
- 数据共享：同一个 Pod（容器组）中运行的容器之间，经常会存在共享文件/文件夹的需求

### 数据卷类型

| 类型         | 举例                      |
| ------------ | ------------------------- |
| 本地数据卷   | emptyDir、hostPath、local |
| 云存储数据卷 | awsElasticBlockStore      |
| 网络存储卷   | NFS、gitRepo、NAS、SAN    |
| 分布式存储卷 | CephFS、rdb、             |
| 信息数据卷   | flocker、secret等         |



### 存储机制

1. 在专用的存储设备上,创建各种类型级别的PV(Persistent Volume)或者通过存储模板文件SC(storageclasses)来自动创建大量不同类型的pv对象
2. 开发人员定制需要的PVC(Persistent Volume Claim),然后关联到pod上
3. pod通过pvc和pv上请求一块独立大小的网络存储空间,然后直接用

![截屏2022-09-22 下午8.19.25](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-22%20%E4%B8%8B%E5%8D%888.19.25.png)

- PV和PVC之间的相互作用遵循这个生命周期 ：供应-->绑定-->使用--> 释放--> 循环
- 随着PV数量的增加，管理员需要不停的定义PV的数量，衍生了通过StorageClass动态生成PV
- StorageClass通过PVC中声明存储的容量，会调用底层的提供商生成PV。



#### configMap和secret的热加载原理

​		这两种挂载都是将数据以k/v的方式存在etcd中;  在Kubernetes中使用ConfigMap的卷挂载方式时，一旦ConfigMap有更新，由于此ConfigMap和Pod进行了关联，Kubelet在进行Pod同步时会将所关联的卷标记为RequireRemount（需要重新挂载）的卷，而热更新最大的时间延迟则来源于这个同步的间隔，
​		sync-frequency为kubelet同步时间间隔参数, 缺省值为1m



## 网络

![截屏2022-09-19 下午9.39.31](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-19%20%E4%B8%8B%E5%8D%889.39.31.png)





### 企业内DNS解决方案

![截屏2022-09-20 下午6.10.48](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-20%20%E4%B8%8B%E5%8D%886.10.48.png)

> CoreDNS: K8S集群内部的pod资源的主机名解析
>
> 内网DNS: 项目网站架构内部的主机名解析
>
> 公网DNS: 项目以来的第三方地址的域名解析

## 垃圾回收(GC)

​		Kubernetes garbage collector(垃圾回收器)的作用是删除那些曾经有 owner，后来又不再有 owner 的 对象和描述

### 垃圾收集器如何删除从属对象

​		当删除某个对象时，可以指定该对象的从属对象是否同时被自动删除，这种操作叫做级联删除 (cascading deletion)。级联删除有两种模式:后台(background)和前台(foreground)

​		如果删除对象时不删除自动删除其从属对象，此时，从属对象被认为是孤儿(或孤立的 orphaned)	

