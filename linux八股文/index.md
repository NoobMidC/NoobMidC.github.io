# Linux八股文


<!--more-->

# Linux八股文

## 基础命令

### 1. 怎么查看当前进程？怎么执行退出？怎么查看当前路径？

​	查看当前进程：ps、执行退出：exit、查看当前路径：pwd

### 2. 查看当前用户id

```shell
[root@cx-ali ~]# id
uid=0(root) gid=0(root) groups=0(root)  # 用户id、组id、所属附加群组的id
```

### 3. 建立软链接和硬链接

```sh
ln -s  src dist
ln src dist
```

### 4. 查看文件命令

```sh
vi filename #编辑方式查看，可修改
cat filename #显示全部文件内容
more filename #分页显示文件内容
less filename #与 more 相似，更好的是可以往前翻页
tail filename #仅查看尾部，还可以指定行数
head filename #仅查看头部,还可以指定行数

# 一页一页查看大文件命令
cat filename | more
```

### 5.统计文件内容命令

```shell
[root@cx-ali ~]# wc -c -l -w .viminfo 
47 148 770 .viminfo		# -c 统计字节数 -l 统计行数 -w 统计字数
```

### 6. grep搜索命令

```sh
grep -i "error"   #忽略大小写区分
grep -v "grep"  #忽略grep命令本身，在文档中过滤掉包含有grep字符的行
grep [^string] filename #正则表达式搜索
```

### 7. 后台运行命令

​	使用 & 在命令结尾来让程序自动运行。

### 8. 查看所有进程

```sh
ps -ef # system v 输出
ps -aux # bsd 格式输出
ps -ef | grep pid
```

### 9. 查看后台任务

```sh
jobs -l
```

### 10. 把后台任务调到前台执行使用什么命令?把停下的后台任务在后台执行起来用什么命令?

```shell
fg # 把后台任务调到前台执行
bg # 把停下的后台任务在后台执行起来
```

### 11. 终止进程用什么命令?

```sh
# kill [-s <信息名称或编号>][程序] 或 kill [-l <信息编号>]
kill -9 pid
kill -l # 查看系统支持的所有信号
```

### 12. 搜索文件的命令

```shell
# find <指定目录> <指定条件> <指定动作>
# find 直接搜索磁盘，较慢。
find / -name "string*"

locate 只加文件名
```

```sh
whereis 
# whereis [-bfmsu][-B <目录>...][-M <目录>...][-S <目录>...][文件...]
#-b 只查找二进制文件。
#-B<目录> 只在设置的目录下查找二进制文件。-f 不显示文件名前的路径名称。
#-m 只查找说明文件。
#-M<目录> 只在设置的目录下查找说明文件。-s 只查找原始代码文件。
#-S<目录> 只在设置的目录下查找原始代码文件。-u 查找不包含指定类型的文件。
#which 指令会在 PATH 变量指定的路径中，搜索某个系统命令的位置，并且返回第一个搜索结果。
#-n 指定文件名长度，指定的长度必须大于或等于所有文件中最长的文件名。
#-p 与-n 参数相同，但此处的包括了文件的路径。-w 指定输出时栏位的宽度。
#-V 显示版本信息

#which 只能查可执行文件
#whereis 只能查二进制文件、说明文档，源文件等
```

### 13. 使用什么命令查看磁盘使用空间？

```sh
[root@cx-ali ~]# df -hl
Filesystem      Size  Used Avail Use% Mounted on    # 文件系统 容量 已用 可用 已用% 挂载点
devtmpfs        1.8G     0  1.8G   0% /dev
```

```sh
# du 和 df 的定义，以及区别？
# du 显示目录或文件的大小
# df 显示每个<文件>所在的文件系统的信息，默认是显示所有文件系统。
# df 命令获得真正的文件系统数据，而 du 命令只查看文件系统的部分情况。
```

### 14. 查看网络

```shell
netstat 
netstat -nlpt # 查看tcp的网络信息
```

### 15. 对命令取别名

```shell
alias la='ls -a'
```

### 16. awk

```sh
cat /etc/passwd |awk -F ':' '{print $1"\t"$7}' # -F 的意思是以':'分隔
```

### 17. 列出所有支持的命令

```sh
compgen -c
```

### 18. 不重启机器的条件下，有什么方法可以把所有正在运行的进程移除呢？

```sh
disown -r
```

### 19. 定时任务

```sh
crontab [-u username]　　　　#省略用户表表示操作当前用户的crontab
    -e      (编辑工作表)
    -l      (列出工作表里的命令)
    -r      (删除工作作)

# 实例
* * * * * myCommand   # 每分钟执行
3,15 8-11 * * * myCommand # 在上午8点到11点的第3和第15分钟执行
```

### 20. 查看路由表

```shell
route -n
nestat -rn
```

### 21. 查看系统资源占用

```sh
top # top 命令
```

![截屏2022-08-26 下午12.08.46](https://raw.githubusercontent.com/noobmid/pics/main/%E6%88%AA%E5%B1%8F2022-08-26%20%E4%B8%8B%E5%8D%8812.08.46.png)

> **第一行 — 任务队列信息**: top - 20:45:10 up 10:08,  1 user,  load average: 0.00, 0.01, 0.05

| 内容                           | 意义                                                         |
| :----------------------------- | :----------------------------------------------------------- |
| 20:45:10                       | 当前时间                                                     |
| up 10:08                       | 系统运行时间（10小时08分钟）                                 |
| 1 user                         | 当前登录用户数                                               |
| load average: 0.00, 0.01, 0.05 | 系统负载（任务队列的平均长度），分别是1分钟、5分钟、15分钟到现在的平均值 |

> **第二行 — 进程信息**: Tasks: 105 total,   1 running, 104 sleeping,   0 stopped,   0 zombie

| 内容         | 意义             |
| :----------- | :--------------- |
| 105 total    | 进程总数         |
| 1 running    | 正在运行的进程数 |
| 104 sleeping | 睡眠进程数       |
| 0 stopped    | 停止进程数       |
| 0 zombie     | 僵尸进程数       |

> **第三行 — CPU信息**: Cpu(s):  0.0%us,  0.1%sy,  0.0%ni, 99.9%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st

| 内容    | 意义                                          |
| :------ | :-------------------------------------------- |
| 0.0%us  | 用户空间占CPU百分比                           |
| 0.1%sy  | 内核空间占CPU百分比                           |
| 0.0%ni  | 用户进程空间内改变过优先级的进程占用CPU百分比 |
| 99.9%id | 空闲CPU百分比                                 |
| 0.0%wa  | 等待输入输出的CPU时间百分比                   |
| 0.0%hi  | 硬件中断占CPU时间百分比                       |
| 0.0%si  | 软件终端占CPU时间百分比                       |
| 0.0%st  | 提供给虚拟化环境执行占CPU时间百分比           |

> **第四行 — 内存信息**: Mem:    288428k total,   257956k used,    30472k free,    40160k buffers

| 内容           | 意义                 |
| :------------- | :------------------- |
| 288428k total  | 物理内存总量         |
| 257956k used   | 使用的物理内存总量   |
| 30472k free    | 空闲内存总量         |
| 40160k buffers | 用作内核缓存的内存量 |

> **第五行 — 内存交换区信息**: Swap:  1046524k total,     3856k used,  1042668k free,    82000k cached

| 内容           | 意义             |
| :------------- | :--------------- |
| 1046524k total | 交换区总容量     |
| 3856k used     | 使用交换区的总量 |
| 1042668k free  | 空闲交换区总量   |
| 82000k cached  | 缓冲交换区总量   |

> 进程信息:

| PID  | 进程ID                                                       | S       | 进程状态               |
| ---- | ------------------------------------------------------------ | ------- | ---------------------- |
| USER | 进程所有者用户名                                             | %CPU    | CPU 时间占用百分比     |
| PR   | 优先级                                                       | %MEM    | 进程使用物理内存百分比 |
| NI   | nice值,负数表示高优先级                                      | TIME+   | 进程使用的CPU时间总计  |
| VIRT | 进程使用虚拟内存总量（以KB为单位） VIRT=SWAP+RES             | COMMAND | 命令名/命令行          |
| RES  | 进程使用的未被换出的物理内存大小（以KB为单位） RES=CODE+DATA | SHR     | 共享内存总大小         |

- lsof

​			 **lsof**表示文件列表，我们可以知道哪个进程打开了哪个文件。

- 查看系统负载

  ```sh
  [root@cx-ali ~]# w
   12:57:02 up 27 days, 22:54,  1 user,  load average: 0.01, 0.02, 0.05
  USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
  root     pts/0    113.246.112.27   11:35    6.00s  0.06s  0.00s w
  [root@cx-ali ~]# uptime 
   12:57:34 up 27 days, 22:55,  1 user,  load average: 0.00, 0.02, 0.05
  ```

  

## 22. 查看物理CPU和CPU核数

```sh
cat /proc/cpuinfo|grep -c 'physical id' # CPU数
cat /proc/cpuinfo|grep -c 'processor'   # 核数
```



## 23. 查看内存信息

```sh
[root@centos6 ~ 10:57 #39]# vmstat
procs -----------memory---------- ---swap-- -----io---- --system-- -----cpu-----
r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
0  0      0 1783964  13172 106056    0    0    29     7   15   11  0  0 99  0  0

[root@cx-ali ~]# free
              total        used        free      shared  buff/cache   available
Mem:        3733516     1493136      196568         616     2043812     1960704
Swap:             0 
```

> r即running，表示正在跑的任务数; b即blocked，表示被阻塞的任务数; si表示有多少数据从交换分区读入内存; so表示有多少数据从内存写入交换分区; bi表示有多少数据从磁盘读入内存; bo表示有多少数据从内存写入磁盘





##  24.网络修改

- 编辑/etc/sysconfig/network-scripts/ifcft-eth0 文件。重启网络服务service network restart

-  给一个网卡配置多个ip, 新建一个ifcfg-eth0:1文件,将DEVICE名称改为eth0:1 ,修改ip,重启网络服务即可

- 在文件 /etc/resolv.conf 中设置DNS



## 系统管理

## 1. 终端是哪个文件夹下的哪个文件？黑洞文件是哪个文件夹下的哪个命令？

终端	/dev/tty				黑洞文件 /dev/null



## 2. Linux 中进程有哪几种状态？在 ps 显示出来的信息中，分别用什么符号表示的？

- 不可中断状态：进程处于睡眠状态，但是此刻进程是不可中断的。不可中断， 指进程不响应异步信号
- 暂停状态/跟踪状态：向进程发送一个 SIGSTOP 信号，它就会因响应该信号 而进入 TASK_STOPPED 状态;当进程正在被跟踪时，它处于 TASK_TRACED 这个特殊的状态。“正在被跟踪”指的是进程暂停下来，等待跟踪它的进程对它进行操作
- 就绪状态：
- 运行状态：在 run_queue 队列里的状态
- 可中断睡眠状态：处于这个状态的进程因为等待某某事件的发生（比如等待 socket 连接、等待信号量），而被挂起
- zombie 状态（僵尸）：父亲没有通过 wait 系列的系统调用会顺便将子进程的尸体（task_struct）也释放掉
- 退出状态

> D 不可中断 Uninterruptible（usually IO）
> R 正在运行，或在队列中的进程
> S 处于休眠状态
> T 停止或被追踪
> Z 僵尸进程
> W 进入内存交换（从内核 2.6 开始无效）
> X 死掉的进程





## 3. 进程管理

- 父子进程通信

  父进程通过使用管道，套接字，消息队列等与子进程进行通信。 

- 僵尸进程

   	这是一个执行已完成但进程表中甚至存在信息的进程。由于父进程需要读取子进程的状态，因此发生在父进程中。一旦使用wait系统调用完成了该任务，则僵尸进程将从进程表中删除。这被称为僵尸进程。

- 异步和非阻塞的区别
  1. 异步:调用在发出之后，这个调用就直接返回，不管有无结果;异步是过程。
  2. 非阻塞:关注的是程序在等待调用结果(消息，返回值)时的状态，指在不能立刻得到结果之前，该调用不会阻塞当前线程。



## 4. 内存管理

- buffer和cache如何区分?

  ​	**Cache是加速“读”，**而buffer是缓冲“写

  ​	buffer和cache都是内存中的一块区域，当需要写数据到磁盘时，由于磁盘速度比较慢，所以CPU先把数据存进buffer，然后CPU去执行其他任务，buffer中的数据会定期写入磁盘；当需要从磁盘读入数据时，由于磁盘速度比较慢，可以把即将用到的数据提前存入cache，CPU直接从Cache中拿数据要快的多。

- Swap空间

  交换空间的主要功能是当全部的 RAM 被占用并且需要更多内存时，用磁盘空间代替 RAM 内存。Linux 计算机中的内存总量是 RAM + 交换分区，交换分区被称为虚拟内存.



## 5. Epoll原理

IO多路复用技术,可以非常高效的处理数以百万计的Socket句柄.

```
int epoll_create(int size);
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
int epoll_wait(int epfd, struct epoll_event *events,int maxevents, int timeout);
```

​	使用起来很清晰，首先要调用 epoll_create 建立一个epoll对象。参数size是内核保证能够正确处理的最大句柄 数，多于这个最大数时内核可不保证效果。 epoll_ctl可以操作上面建立的epoll，例如，将刚建立的 socket 加入到 epoll中让其监控，或者把 epoll正在监控的某个socket句柄移出epoll，不再监控它等等。

epoll_wait 在调用时，在给定的timeout时间内，当在监控的所有句柄中有事件发生时，就返回用户态的进程。

调用 epoll_wait 时就相当于以往调用select/poll，但是这时却不 用传递socket句柄给内核，因为内核已经在epoll_ctl中拿到了要监控的句柄列表。

