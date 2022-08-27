# Golang八股文


<!--more-->

# Golang 八股文

------

## 基础数据结构

### 1. **go的指针和c的指针**

​		相同点:			

- ​	运算符相同, &为取地址, *为解引用

​		不同点:

- 数组名和数组首地址. **c语言**中arr、&arr[0]为数组的首个**元素**地址,单位偏移量为元素大小; &arr为**数组**的首地址,单位偏移量为数组大小. **golang**中&arr[0]和&arr与c语言中相同, 但是arr表示为整个数组的值.

  ```go
  // C
  int arr[5] = {1, 2, 3, 4, 5};
  // Go
  // 需要指定长度，否则类型为切片
  arr := [5]int{1, 2, 3, 4, 5}
  ```

-  指针运算.   **c语言**中指针本质为无符号整数,代表内存地址, 可以进行加减运算.    **Golang**中指针为 `*uint32`类型非数字,不可以加减运算.

  > Go 标准库中提供了一个 `unsafe` 包用于编译阶段绕过 Go 语言的类型系统，直接操作内存
  >
  > `uintptr` : Go 的内置类型。是一个无符号整数，用来存储地址，支持数学运算。常与 `unsafe.Pointer` 配合做指针运算
  >
  > `unsafe.Pointer` : 表示指向任意类型的指针，可以和任何类型的指针互相转换（类似 C 语言中的 `void*` 类型的指针），也可以和 `uintptr` 互相转换
  >
  > `unsafe.Sizeof` : 返回操作数在内存中的字节大小，参数可以是任意类型的表达式，例如 fmt.Println(unsafe.Sizeof(uint32(0)))的结果为 4
  >
  > `unsafe.Offsetof` : 函数的参数必须是一个字段 x.f，然后返回 f 字段相对于 x 起始地址的偏移量，用于计算结构体成员的偏移量

> golang中**不**能被寻址的类型:(不可变的，临时结果和不安全的。)  常量、字符串、函数或方法、map中的元素...
>

### 2. String的底层结构

```go
type StringHeader struct {  // 16 字节
	Data uintptr
	Len  int
}
```

![string底层结构](https://raw.githubusercontent.com/noobmid/pics/main/string%E5%BA%95%E5%B1%82%E7%BB%93%E6%9E%84.png)

本质为byte类型的数组

### 3.slice和array的区别和底层结构

​	array 为定长数组, slice为不定长数组

- 底层结构

```go
type slice struct { 
  array unsafe.Pointer   // array指针指向底层array的某一个元素，其决定slice所能控制的内存片段的起始位置，这里需要注意的是，array不一定指向底层array的首元素，这与slice的创建有关。
  len int //len 代表当前切片的长度,限定slice可直接通过索引(下标)存取元素的范围
  cap int // cap 是当前切片的容量,表示slice所引用的array片段的真实大小
}
// slice 扩张: slice扩容规则是：在1024字节以内，扩容一倍，大于1024时，增加cap的1/4
```

- 初始化方式:	

```go
// 数组																																						// 切片
a := [3]int{1,2,3} //指定长度																											s := make([]int, 3) //指定长度
a := [...]int{1,2,3} //不指定长度																									s := []int{1,2,3} //不指定长度
```

- 函数传递

  当切片和数组作为参数在函数（func）中传递时，数组传递的是值，而切片传递的是指针。因此当传入的切片在函数中被改变时，函数外的切片也会同时改变。相同的情况，函数外的数组则不会发生任何变化。

- nil切片和空切片最大的区别在于**指向的数组引用地址是不一样的**

  ![图片](https://raw.githubusercontent.com/noobmid/pics/main/640.png)

  - **所有的空切片指向的数组引用地址都是一样的**

    ![图片](https://raw.githubusercontent.com/noobmid/pics/main/640-20220824105030840.png)

### 4. Map底层结构

用于存储一系列无序的键值对,**hashmap**作为底层实现

##### Map基本数据结构:

```go
// hashmap的简称
type hmap struct {
 count     int       //元素个数
 flags     uint8     //标记位
 B         uint8     //buckets的对数, 说明包含2^B个bucket
 noverflow uint16    //溢出的bucket的个数
 hash0     uint32    //hash种子
 buckets    unsafe.Pointer //指向buckets数组的指针，数组个数为2^B
 oldbuckets unsafe.Pointer //扩容时使用，buckets长度是oldbuckets的两倍
 nevacuate  uintptr  //扩容进度，小于此地址的buckets已经迁移完成
 extra *mapextra     //扩展信息
}

//当map的key和value都不是指针，并且size都小于128字节的情况下，会把 bmap 标记为不含指针，这样可以避免gc时扫描整个hmap。但是，我们看bmap其实有一个overflow的字段，是指针类型的，破坏了bmap不含指针的设想，这时会把overflow移动到extra字段来。
type mapextra struct {
 overflow    *[]*bmap
 oldoverflow *[]*bmap  //用于扩容
 nextOverflow *bmapx  //prealloc的地址
}

// bucket
type bmap struct {
 tophash [bucketCnt]uint8 //bucketCnt = 8,用于记录8个key哈希值的高8位，这样在寻找对应key的时候可以更快，不必每次都对key做全等判断
 // keys     [8]keytype
 // values   [8]valuetype
 // pad      uintptr
 // overflow uintptr
}
```

![img](https://raw.githubusercontent.com/noobmid/pics/main/v2-5e4be7641d03d56c2dc68db1563cb6c9_1440w.jpg)

![img](https://raw.githubusercontent.com/noobmid/pics/main/v2-fe7664a42e47d3faeadf4f2663718cb2_1440w.jpg)

​																				hmap结构图

- 如何扩容

​		触发条件: 1. 装填因子大于6.5(装填因子为2^B); 2. overflow bucket 太多

​		解决办法:

   1. 双倍扩容:扩容采取了一种称为“渐进式”地方式，原有的 key 并不会一次性搬迁完毕，每次最多只会搬迁2 个 bucket.(条件1)

   2. 等量扩容:重新排列，极端情况下，重新排列也解决不了，map成了链表，性能大大降低，此时哈希种子 hash0 的设置，可以降低此类极端场景的发生。(条件2)

      

- 赋值操作

   1. 在查找key之前，会做异常检测，校验map是否未初始化，或正在并发写操作，如果存在，则抛出异常：（这就是为什么map 并发写回panic的原因）

   2. 需要计算key 对应的hash 值，如果buckets 为空（初始化的时候小于一定长度的map 不会初始化数据）还需要初始化一个bucket

   3. 通过hash 值，获取对应的bucket。如果map 还在迁移数据，还需要在oldbuckets中找对应的bucket，并搬迁到新的bucket。

   4. 拿到bucket之后，还需要按照链表方式一个一个查，找到对应的key， 可能是已经存在的key，也可能需要新增。

   5. 插入数据前，会先检查数据太多了，需要扩容，如果需要扩容，那就从第③开始拿到新的bucket，并查找对应的位置。

   6. 如果没有空的位置，那就需要在链表后追加一个bucket，拿到kv

   7. 最后更新tophash 和 key 的字面值, 并解除hashWriting 约束

      

- 数据迁移

	1. 先要判断当前bucket是不是已经转移。 (oldbucket 标识需要搬迁的bucket 对应的位置)
	1. 如果没有被转移，那就要迁移数据了。数据迁移时，可能是迁移到大小相同的buckets上，也可能迁移到2倍大的buckets上。这里xy 都是标记目标迁移位置的标记：x 标识的是迁移到相同的位置，y 标识的是迁移到2倍大的位置上。
	1. 确定bucket位置后，需要按照kv 一条一条做迁移。（目的就是清除空闲的kv）



- 数据查找

​	Go语言中 map采用的是**哈希查找表**，由一个 key 通过哈希函数得到哈希值，64 位系统中就生成一个64bit 的哈希值，由这个哈希值将 key 对应到不同的桶bucket中，当有多个哈希映射到相同的的桶中时，使用链表解决哈希冲突。key 经过 hash 后共64 位，根 据 hmap中 B的值，计算它到底要落在哪个桶时，桶的数量为2^B，如 B=5，那么用64 位最后5 位表示第几号桶，在用 hash 值的高8 位确定在 bucket 中的存储位置，当前 bmap中的 bucket 未找到，则查询对应的 overflow bucket，对应位置有数据则对比完整的哈希值，确定是否是要查找的数据。如果两个不同的 key 落在的同一个桶上，hash 冲突使用链表法接近，遍历 bucket 中的 key ;如果当前处于 map进 行了扩容，处于数据搬移状态，则优先从 oldbuckets 查找。	

1. 根据key计算出hash值。
2. 如果存在old table, 首先在old table中查找，如果找到的bucket已经evacuated，转到步骤3。 反之，返回其对应的value。
3. 在new table中查找对应的value。

##### map 顺序读取方法

​		给key排序后读取

##### set实现

```go
map[string]bool //会有bool的空间占用，可以替换成空结构体

type void struct{}
var member void
set := make(map[string]void)
set["test"] = member
```



### 5. Channel 底层结构

```go
type hchan struct {
  qcount   uint           // 队列中的数据个数
  dataqsiz uint           // 环形队列的大小，channel本身是一个环形队列
  buf      unsafe.Pointer // 存放实际数据的指针，用unsafe.Pointer存放地址，为了避免gc
  elemsize uint16 
  closed   uint32 // 标识channel是否关闭
  elemtype *_type // 数据 元素类型
  sendx    uint   // send的 index
  recvx    uint   // recv 的 index
  recvq    waitq  // 阻塞在 recv 的队列
  sendq    waitq  // 阻塞在 send 的队列
  lock mutex  // 锁 
}
```

> channel本身是一个**环形缓冲区**，数据存放到堆上面，channel的同步是通过锁实现的，并不是想象中的lock-free的方式，channel中有两个队列，一个是发送阻塞队列，一个是接收阻塞队列。当向一个已满的channel发送数据会被阻塞，此时发送协程会被添加到sendq中，同理，当向一个空的channel接收数据时，接收协程也会被阻塞，被置入recvq中。

​					 		![ringbuf 实现](https://raw.githubusercontent.com/noobmid/pics/main/%E6%88%AA%E5%B1%8F2022-08-24%20%E4%B8%8B%E5%8D%885.24.22.png)



1. 创建channel
   1. 缓冲区大小为0: 只需要分配hchansize大小的内存就ok;
   
   2. 缓冲区大小不为0，且channel的类型不包含指针:   buf为hchanSize+元素大小*元素个数的连续内存

   3. 缓冲区大小不为0，且channel的类型包含指针，则不能简单的根据元素的大小去申请内存，需要通过mallocgc去分配内存(即内存逃逸)

   2. channel特性
   
      		> 1. 给一个 nil channel 发送数据，造成永远阻塞
      > 2. 从一个 nil channel 接收数据，造成永远阻塞
      > 3. 关闭一个 nil channel 将会发生 panic
      > 4. 给一个已经关闭的 channel 发送数据，引起 panic

      ![图片](https://raw.githubusercontent.com/noobmid/pics/main/640-20220824180223858.png)

      - 当 `c.closed != 0` 则为通道关闭，此时执行写，源码提示直接 panic，输出的内容就是上面提到的 `"send on closed channel"`。

      
   > 5. 从一个已经关闭的 channel 接收数据，如果缓冲区中为空，则返回一个零值
   
   ![图片](https://raw.githubusercontent.com/noobmid/pics/main/640-20220824180427191.png)
   
   - `c.closed != 0 && c.qcount == 0` 指通道已经关闭，且缓存为空的情况下（已经读完了之前写到通道里的值）
      - 如果接收值的地址 `ep` 不为空
      - 那接收值将获得是一个**该类型的零值**
      - `typedmemclr` 会**根据类型清理**相应地址的内存
      - 这就解释了上面代码为什么关闭的 `chan` 会返回对应类型的零值
   > 6. 无缓冲的 channel 是同步的，而有缓冲的 channel 是非同步的

### 6. interface

```go
// interface 分为空接口和非空接口，分别用eface 和 iface实现。
// eface
type eface struct {
    _type *_type
    data  unsafe.Pointer //指向数据的数据指针
}
//_type 定义
type _type struct {
    size       uintptr  // 类型的大小
    ptrdata    uintptr  // size of memory prefix holding all pointers
    hash       uint32   // 类型的Hash值
    tflag      tflag    // 类型的Tags 
    align      uint8    // 结构体内对齐
    fieldalign uint8    // 结构体作为field时的对齐
    kind       uint8    // 类型编号 定义于runtime/typekind.go
    alg        *typeAlg // 类型元方法 存储hash和equal两个操作。map key便使用key的_type.alg.hash(k)获取hash值
    gcdata     *byte    // GC相关信息
    str        nameOff  // 类型名字的偏移    
    ptrToThis  typeOff
}

// iface
type iface struct {
    tab  *itab
    data unsafe.Pointer
}
// 非空接口的类型信息
type itab struct {
    inter  *interfacetype    // 接口定义的类型信息
    _type  *_type            // 接口实际指向值的类型信息
    link   *itab  
    bad    int32
    inhash int32
    fun    [1]uintptr       // 接口方法实现列表，即函数地址列表，按字典序排序
}
// 非空接口类型，接口定义，包路径等。
type interfacetype struct {
   typ     _type
   pkgpath name
   mhdr    []imethod      // 接口方法声明列表，按字典序排序
}
// 接口的方法声明 
type imethod struct {
   name nameOff          // 方法名
   ityp typeOff         // 描述方法参数返回值等细节
}
```

> 非空interface与eface不同的，所有空interface的结构是一样的，而非空interface每个都不一样，因为彼此定义的方法可以不一样的，所以相对eface，iface的定义复杂多。



### 7. reflect

```go
t := reflect.TypeOf(stru).Elem()
for i := 0; i < t.NumField(); i++ {
  			// 获取Tag
        t.Field(i).Name, t.Field(i).Tag.Get("json"), t.Field(i).Tag.Get("otherTag")
}
//reflect.TypeOf(stru).Elem()获取指针指向的值对应的结构体内容。
//NumField()可以获得该结构体的含有几个字段。
//遍历结构体内的字段，通过t.Field(i).Tag.Get("json")可以获取到tag为json的字段。
```

> `json`包里不能导出私有变量的`tag`是因为`json`包里认为私有变量为不可导出的`Unexported`，所以**跳过获取**名为`json`的`tag`的内容

#### 动态类型判断: 类型开关是在运行时检查变量类型的最佳方式

```go
//1. 类型识别
var data interface{} = "hello"
strValue, ok := data.(string)
if ok {
	fmt.Printf("%s is string type\n", strValue)
}

//2. 类型获取
var str string = "hello"
fmt.Println(reflect.TypeOf(str))

//3. 类型判断
func typeJudge(x interface{})  {
	switch x.(type){
	case int,int8,int64,int16,int32,uint,uint8,uint16,uint32,uint64:
		fmt.Println("整型变量")
	case float32,float64:
		fmt.Println("浮点型变量")
	case []byte,[]rune,string:
		fmt.Println("字符串变量")
	default:
		fmt.Println("不清楚...")
	}
}
```

### 8. new 和make的区别

- new(T) 返回的是 T 的指针：new(T) 为一个 T 类型新值分配空间并将此空间初始化为 T 的零值，返回的是新值的地址，也就是 T 类型的指针 *T，该指针指向 T 的新分配的零值。

- make 只能用于 slice,map,channel，返回值是经过初始化之后的 T 的引用
- make 分配空间后，会进行初始化

### 9.struct能不能比较

​	go中的struct能不能比较**取决于struct内部存储的数据**，如果struct中的字段都是可比较的，那么该struct就是可比较的，如果其中的字段是不可比较的，那么该struct不可比较。slice,map就无法比较。

- struct字段顺序要一致才能比较

- 数组的长度是类型的一部分，如果数组长度不同，无法比较。

- interface{}类型的比较包含该接口变量存储的值和值的类型两部分组成，分别称为接口的动态类型和动态值。只有动态类型和动态值都相同时，两个接口变量才相同。

- slice在go设计之初为了和数组区分，不让其可以比较（浅层指针比较没有意义）

  > 1、引用类型，比较地址没有意义。
  >
  > 2、切片有len，cap，比较的维度不好衡量，因此go设计的时候就不允许切片可比较。
  >
  > 3、map的value, 函数均可以包含slice,因而均不可比较

### 10. defer

-  defer关键字定义的函数是在调用函数返回之后执行，而不是在代码块退出之后执行。defer的执行顺序是先创建的后执行。看做是一个 `FILO`(First In Last Out) 栈.
- 所有传入defer函数的参数都是在创建的时候预先计算处理的，而不是调用函数退出的时候计算的
- defer等到包含它的程序返回时(包含它的函数执行了return语句、运行到函数结尾自动返回、对应的goroutine panic）defer函数才会被执行。通常用于资源释放、打印日志、异常捕获等
- `defer` 关键字对应的 `runtime.deferproc` 会将延迟调用函数与调用方所在 Goroutine 进行关联。所以当程序发生崩溃时只会调用当前 Goroutine 的延迟调用函数



### 11. init函数

- 整个golang程序初始化顺序

  先调用osinit，再调用schedinit，创建就绪队列并新建一个G，接着就是mstart

  即设置好本地线程存储，设置好main函数参数，根据环境变量GOMAXPROCS设置好使用的procs，初始化调度器和内存管理等等。

-  main.main之前的准备

  1. **sysmon** :Go语言的runtime库会初始化一些后台任务，其中一个任务就是sysmon. 它由物理线程运行.主要处理两个事件：对于网络的epoll以及抢占式调度的检测.

  - 释放闲置超过5 分钟的 span 物理内存;
  - 如果超过2 分钟没有垃圾回收，强制执行;
  - 将长时间未处理的 netpoll 添加到全局队列;
  - 向长时间运行的 G 任务发出抢占调度(超过10ms的 g，会进行 retake); 
  - 收回因 syscall 长时间阻塞的 P;

  1. **scavenger**: 只是由goroutine运行.用于执行heap的内存回收给os

​	先于main函数执行，实现包级别的一些初始化操作

​	主要作用:

- 初始化不能采用初始化表达式初始化的变量;
- 程序运行前的注册
- 实现sync.Once功能

​	主要特点:

- init函数先于main函数自动执行，不能被其他函数调用；
- init函数没有输入参数、返回值；
- 每个包可以有多个init函数；
- **包的每个源文件也可以有多个init函数**，这点比较特殊；
- 同一个包的init执行顺序，golang没有明确定义，编程时要注意程序不要依赖这个执行顺序
- 不同包的init函数按照包导入的依赖关系决定执行顺序

![image-20220824111732112](https://raw.githubusercontent.com/noobmid/pics/main/image-20220824111732112.png)



### 12.包的循环引用

- 为什么不允许循环引用

  ​	加快编译速度、规范框架设计，使项目结构更加清晰明了

- 解决办法:

					1. mvc 结构,将包规划好
					1. 新建公共接口包(父包), 将需要循环调用的函数或方法抽象为接口
					1. 新建公共组合包(子包), 在组合包中组合调用
					1. 全局存储需要相互依赖的函数, 通过关键字进行调用



### 13. SELECT

1. select可以用来等待多个channel的可读可写，其中select中的case表达式必须都是channel的读写操作。
2. 当存在default时，select执行的就是非阻塞收发，当不存在时，必须等待某一个channel可读或可写。
3. 当多个channel都可读可写时，会随机选择一个分支。
4. 如果有一个channel已关闭,则每次都执行到这个case; 如果只有一个case,则出现死循环.

```go
var c1, c2, c3 chan int
var i1, i2 int
select {
  case i1 = <-c1:
     fmt.Printf("received ", i1, " from c1\n")
  case c2 <- i2:
     fmt.Printf("sent ", i2, " to c2\n")
  case i3, ok := (<-c3):  // same as: i3, ok := <-c3
     if ok {
        fmt.Printf("received ", i3, " from c3\n")
     } else {
        fmt.Printf("c3 is closed\n")
     }
  default:
     fmt.Printf("no communication\n")
}
```

5. x, ok := <-ch,如果ch已经关闭,则ok为false,置ch为nil,则可以继续阻塞.

#### 保证case的优先级

```go
for {
		select {
		case <-stopCh:
			return
		case job1 := <-ch1:
			fmt.Println(job1)
		case job2 := <-ch2:
		priority:  // label
			for {
				select {
				case job1 := <-ch1:
					fmt.Println(job1)
				default:
					break priority
				}
			}
			fmt.Println(job2)
		}
}
```



### 14. for range 循环

```go
{
    for_temp := slice1
    len_temp := len(for_temp)
    for index_temp := 0; index_temp < len_temp; index_temp++ { 
        value_temp := for_temp[index_temp] 
        // index = index_temp 
         // value = value_temp 
        // origin body
     }
} // 新的语句块结束
```



### 15. Panic和Recover

	#### panic

​	panic的作用是制造一次宕机，宕机就代表程序运行终止，但是已经“生效”的(当前 Goroutine )延迟函数仍会执行（即已经压入栈的defer延迟函数，panic之前的）。

```go
// 嵌套崩溃
func main() {
	defer fmt.Println("in main")
	defer func() {
		defer func() {
			panic("panic again and again")
		}()
		panic("panic again")
	}()

	panic("panic once")
}

$ go run main.go
in main
panic: panic once
	panic: panic again
	panic: panic again and again

goroutine 1 [running]:
...
exit status 2
```

#### recover

​		`recover` 只有在发生 `panic` 之后调用才会生效。然而在上面的控制流中，`recover` 是在 `panic` 之前调用的，并不满足生效的条件，所以需要在 `defer` 中使用 `recover` 关键字。



### 16. golang函数调用规则

- 通过堆栈传递参数，入栈的顺序是从右到左，而参数的计算是从左到右
- 函数返回值通过堆栈传递并由调用者预先分配内存空间；
- 调用函数时都是传值，接收方会对入参进行复制再计算；



### 17. go相比于c++线程的优势

-  开销更小

- 切换更方便 

  1. C线程的上下文切换涉及到模式转换-从用户态到内核态
  2. go的协程中的上下文切换只是在用户态的操作

- 用户可控制（用户态）

- 高级调度策略:   任务窃取和减少阻塞

  





------

## 进阶原理

### 1. GMP模型和golang调度原理

​		go中调度采用GMP算法，G表示一个goroutine，M表示machine一个真实的线程，P表示processor表示一个调度器

- P由启动时环境变量 $GOMAXPROCS 或者是由 runtime 的方法 GOMAXPROCS() 决定。这意味着在程序执行的任意时刻都只有 $GOMAXPROCS 个 goroutine 在同时运行。最大256
- M的最大限制是10000个，但是内核很难支持这么多的线程数，所以这个限制可以忽略; 一般为CPU数
- 在P没有足够的M绑定运行时,则会创建一个M;每次创建一个M都会同步创建一个G0，它负责调度其它的G，每个M都有一个G0

![截屏2022-08-24 下午6.18.32](https://raw.githubusercontent.com/noobmid/pics/main/%E6%88%AA%E5%B1%8F2022-08-24%20%E4%B8%8B%E5%8D%886.18.32.png)

![截屏2022-08-24 下午6.18.54](https://raw.githubusercontent.com/noobmid/pics/main/%E6%88%AA%E5%B1%8F2022-08-24%20%E4%B8%8B%E5%8D%886.18.54.png)

![截屏2022-08-24 下午6.24.44](https://raw.githubusercontent.com/noobmid/pics/main/%E6%88%AA%E5%B1%8F2022-08-24%20%E4%B8%8B%E5%8D%886.24.44.png)

- 每个 P有个局部队列，局部队列保存待执行的 goroutine(流程2)，当 M绑定的 P的的局部队列已经满了之后就 会把 goroutine 放到全局队列(流程2-1)
-  每个 P和一个 M绑定，M是真正的执行 P中 goroutine 的实体(流程3)，M 从绑定的 P中的局部队列获取 G来 执行

- 当 M绑定的 P的局部队列为空时，M会从全局队列获取到本地队列来执行 G(流程3.1)，当从全局队列中没有获取到可执行的 G时候，M会从其他 P 的局部队列中偷取 G来执行(流程3.2)，这种从其他 P偷的方式称为 **work stealing**

-  当 G因系统调用(syscall)阻塞时会阻塞 M，此时 P会和 M解绑即 **hand off**，并寻找新的 idle 的 M，若没有 idle 的 M就会新建一个 M(流程5.1)。

-  当 G因 channel 或者 network I/O 阻塞时，不会阻塞 M，M会寻找其他 runnable 的 G;当阻塞的 G恢复后会重新进入 runnable 进入 P队列等待执行(流程5.3)

1. Work stealing

​				当本线程无可运行的 G 时，尝试从其他线程绑定的 P 偷取 G，而不是销毁线程。

2. Hand off 

   本线程因为 G 进行系统调用阻塞时，线程释放绑定的 P，把 P 转移给其他空闲的线程执行。

   利用并行：GOMAXPROCS 设置 P 的数量，最多有 GOMAXPROCS 个线程分布在多个 CPU 上同时运行。GOMAXPROCS 也限制了并发的程度，比如 GOMAXPROCS = 核数/2，则最多利用了一半的 CPU 核进行并行。

   抢占：在 coroutine 中要等待一个协程主动让出 CPU 才执行下一个…在 Go 中，一个 goroutine 最多占用 CPU 10ms，防止其他 goroutine 被饿死，这就是 goroutine 不同于 coroutine 的一个地方。

3. 如果没有P或怎样?

   调度器把 G都分配到 M上，不同的 G在不同的 M并发运行时，都需要向系统申请资源，比如堆栈内存等，因为资源 是全局的，就会因为资源竞争照成很多性能损耗。

4. **GMP** 调度过程中存在哪些阻塞

- I/O, select
- block on **syscall** channel
- 等待锁 
- runtime.Gosched()

5. 抢占式调度

   sysmon。这个函数会周期性地做epoll操作，同时它还会检测每个P是否运行了较长时间。如果检测到某个P状态处于syscall超过了一个sysmon的时间周期(20us)，并且还有其它可运行的任务，则切换P。

   如果检测到某个P的状态为running，并且它已经运行了超过10ms，则会将P的当前的G的stackguard设置为StackPreempt。这个操作其实是相当于加上一个标记，通知这个G在合适时机进行调度。





### 2. 垃圾回收(GC)

#### 垃圾回收常用方法

	1. 引用计数(reference counting) 

​			如C++中的智能指针:shard_ptr; 

- 优点：简单直接，回收速度快

- 缺点：需要额外的空间存放计数，无法处理循环引用的情况；

2. 标记-清除(mark and sweep)

​			标记出所有不需要回收的对象，在标记完成后统一回收掉所有未被标记的对象。

-  优点：简单直接，速度快，适合可回收对象不多的场景

- 缺点：会造成不连续的内存空间（内存碎片），导致有大的对象创建的时候，明明内存中总内存是够的，但是空间不是连续的造成对象无法分配；

3. 分代搜集(generation)

​		java的jvm 就使用的分代回收的思路。在面向对象编程语言中，绝大多数对象的生命周期都非常短。分代收集的基 本思想是，将堆划分为两个或多个称为代(generation)的空间。新创建的对象存放在称为新生代(young generation)中(一般来说，新生代的大小会比 老年代小很多)，随着垃圾回收的重复执行，生命周期较⻓的对 象会被提升(promotion)到老年代中(这里用到了一个分类的思路，这个是也是科学思考的一个基本思路)。

​		因此，新生代垃圾回收和老年代垃圾回收两种不同的垃圾回收方式应运而生，分别用于对各自空间中的对象执行垃
圾回收。新生代垃圾回收的速度非常快，比老年代快几个数量级，即使新生代垃圾回收的频率更高，执行效率也仍
然比老年代垃圾回收强，这是因为大多数对象的生命周期都很短，根本无需提升到老年代。

#### GOLANG的GC策略

​	golang采用无分代（对象没有代际之分）、不整理（回收过程中不对对象进行移动与整理）、并发（与用户代码并发执行）的三色标记清扫算法。

​	原因:

1. golang的内存分配算法tcmalloc，基本上不会造成内存碎片，因此不需要使用对象整理。

2. golang对于存活时间短的对象直接分配在栈上面，go程死亡后栈会被回收，不需要gc的参与。

 Go 以 STW 为界限，可以将 GC 划分为五个阶段：：**栈扫描**（开始时STW）;**第一次标记**（并发）;**第二次标记**（STW）;**清除**（并发）,归还

![截屏2022-08-24 下午7.11.32](https://raw.githubusercontent.com/noobmid/pics/main/%E6%88%AA%E5%B1%8F2022-08-24%20%E4%B8%8B%E5%8D%887.11.32.png)

##### 三色标记清扫法

white，grep，black;白色为需要清理的数据，黑色则不要清理。从根对象（全局变量、执行栈、寄存器(主要是指针)）开始循环，能访问到的标记为灰色，然后从灰色队列开始遍历，自身变成黑色。后续没有访问到的直接清理掉。

![test](https://raw.githubusercontent.com/noobmid/pics/main/test.gif)

##### 没有STW的三色标记法

![img](https://raw.githubusercontent.com/noobmid/pics/main/afb1d9bb3a9d4cf785e65f163a73d933%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A3024%3A0%3A0%3A0.awebp)

![img](https://raw.githubusercontent.com/noobmid/pics/main/a24467e8a92745939cba73b40656930c%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A3024%3A0%3A0%3A0.awebp)

![img](https://raw.githubusercontent.com/noobmid/pics/main/554354937b6a49f18a422db41817c13f%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A3024%3A0%3A0%3A0.awebp)

![img](https://raw.githubusercontent.com/noobmid/pics/main/7057f2f4ffb345a4926c10221ea68d62%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A3024%3A0%3A0%3A0.awebp)

![img](https://raw.githubusercontent.com/noobmid/pics/main/29df20ca665340b68c6e63a51bc6223d%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A3024%3A0%3A0%3A0.awebp)

- 条件1: 一个白色对象被黑色对象引用 **(白色被挂在黑色下)**
- 条件2: 灰色对象与它之间的可达关系的白色对象遭到破坏 **(灰色同时丢了该白色)**

当以上两个条件同时满足时, 就会出现对象丢失现象!



##### 屏障保护

为了减少STW的影响,又防止三色标记法出现对象丢失现象,出现屏障保护技术;

1. 插入写屏障

![img](https://raw.githubusercontent.com/noobmid/pics/main/5ca502f0c5b1467585f9c40c0d29ab82%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A3024%3A0%3A0%3A0.awebp)

​		垃圾收集器将根对象指向 A 对象标记成黑色并将 A 对象指向的对象 B 标记成灰色；用户程序修改 A 对象的指针，将原本指向 B 对象的指针指向 C 对象，这时触发写屏障将 C 对象标记成灰色；一种相对保守的屏障技术，它会将有存活可能的对象都标记成灰色以满足强三色不变性.

​		写屏障只会针对堆进行限制.

2. 删除写屏障

![img](https://raw.githubusercontent.com/noobmid/pics/main/b8d26a88219d40d897cf0e7f86b43f42%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A3024%3A0%3A0%3A0.awebp)

​		垃圾收集器将根对象指向 A 对象标记成黑色并将 A 对象指向的对象 B 标记成灰色；用户程序将 A 对象原本指向 B 的指针指向 C，触发删除写屏障，但是因为 B 对象已经是灰色的，所以不做改变；用户程序将 B 对象原本指向 C 的指针删除，触发删除写屏障，白色的 C 对象被涂成灰色；

​		缺点是开始收集器时需要STW快照全局对象

3. 混合写屏障

操作流程:

​	1、GC开始将栈上的对象全部扫描并标记为黑色(之后不再进行第二次重复扫描，无需STW)，

​	2、GC期间，任何在栈上创建的新对象，均为黑色。

​	3、被删除的对象标记为灰色。(删除写屏障)

​	4、被添加的对象标记为灰色。(插入写屏障)

**该写屏障会将被覆盖的对象标记成灰色并在当前栈没有扫描时将新对象也标记成灰色;**  只需要针对堆内存扫描即可.



#### GC触发机制

1. 主动: 通过调用 runtime.GC 来触发 GC，此调用阻塞式地等待当前 GC 运行完毕。
2. 被动: 
   - 使用系统监控(sysmon)，当超过两分钟没有产生任何 GC 时，强制触发 GC。
   - 使用步调（Pacing）算法，其核心思想是控制内存增长的比例。(步调算法可以通过gogc传参设置量控制gc的时间。也是go中唯一对外开放的配置gc的参数。默认值为100，也就是达到百分百后触发gc机制)



#### GC细节

	##### 增量垃圾收集

​		增量地标记和清除垃圾，降低应用程序暂停的最长时间；

​		传统的垃圾收集算法会在垃圾收集的执行期间暂停应用程序，一旦触发垃圾收集，垃圾收集器会抢占 CPU 的使用权占据大量的计算资源以完成标记和清除工作，然而很多追求实时的应用程序无法接受长时间的 STW。增量式（Incremental）的垃圾收集是减少程序最长暂停时间的一种方案，它可以将原本时间较长的暂停时间切分成多个更小的 GC 时间片，虽然从垃圾收集开始到结束的时间更长了，但是这也减少了应用程序暂停的最大时间。

##### 并发垃圾收集 

​		利用多核的计算资源，在用户程序执行时并发标记和清除垃圾；

#### **GC** 如何调优

 通过 go tool pprof 和 go tool trace 等工具

- 控制内存分配的速度，限制 goroutine 的数量，从而提高赋值器对 CPU 的利用率。 
- 减少并复用内存，例如使用 sync.Pool 来复用需要频繁创建临时对象，例如提前分配足够的内存来降低多余的 拷贝。
-  需要时，增大 GOGC 的值，降低 GC 的运行频率。



### 3. 内存模型和内存管理

#### 内存逃逸

​		`golang程序变量`会携带有一组校验数据，用来证明它的整个生命周期是否在运行时完全可知。如果变量通过了这些校验，它就可以在`栈上`分配。否则就说它 `逃逸` 了，必须在`堆上分配`

- 内存逃逸的情况:

  1. **在方法内把局部变量指针返回**
  2. **发送指针或带有指针的值到 channel 中**
  3. **在一个切片上存储指针或带指针的值**
  4. **slice 的背后数组被重新分配了，因为 append 时可能会超出其容量( cap )**;slice 初始化的地方在编译时是可以知道的，它最开始会在栈上分配。如果切片背后的存储要基于运行时的数据进行扩充，就会在堆上分配。
  5. **在 interface 类型上调用方法。**在 interface 类型上调用方法都是动态调度的 —— 方法的真正实现只能在运行时知道。

- 查看内存逃逸的情况:

  ```go
  go build -gcflags=-m main.go
  ```

- 避免内存逃逸

  Noescape函数, 可以在逃逸分析中**隐藏一个指针**。让这个指针在逃逸分析中**不会被检测为逃逸**。 

  ```go
  func noescape(p unsafe.Pointer) unsafe.Pointer {
       x := uintptr(p)
       return unsafe.Pointer(x ^ 0)
  }
  ```

  - `noescape()` 函数的作用是遮蔽输入和输出的依赖关系。使编译器不认为 `p` 会通过 `x` 逃逸， 因为 `uintptr()` 产生的引用是编译器无法理解的。
  - 内置的 `uintptr` 类型是一个真正的指针类型，但是在编译器层面，它只是一个存储一个 `指针地址` 的 `int` 类型。代码的最后一行返回 `unsafe.Pointer` 也是一个 `int`。

- 闭包

  闭包是由函数及其相关引用环境组合而成的实体(即：闭包=函数+引用环境)。

  ```go
  func f(i int) func() int {
      return func() int {
          i++
          return i
      }
  }
  ```

  不可在栈上分配:  变量i是函数f中的局部变量，假设这个变量是在函数f的栈中分配的，是不可以的。因为函数f返回以后，对应的栈就失效了，f返回的那个函数中变量i就引用一个失效的位置了。所以闭包的环境中引用的变量不能够在栈上分配。

  因此以上代码在汇编中就类似于:

  ```go
  type Closure struct {
      F func()() 
      i *int
  }
  ```

  在堆中创建结构体, 将函数地址赋值给F, 闭包环境的局部变量在堆上开辟空间,写值后再将地址赋值给i;

  返回闭包时并不是单纯返回一个函数，而是返回了一个结构体，记录下函数返回地址和引用的环境中的变量地址

#### 栈空间管理

​		Go语言的运行环境(runtime)会在goroutine需要的时候动态地分配栈空间，而不是给每个goroutine分配固定大小的内存空间。这样就避免了需要程序员来决定栈的大小。当创建一个goroutine的时候，它会分配一个8KB的内存空间来给goroutine的栈使用。

##### 分段栈

​		当检测到函数需要更多栈时，分配一块新栈，旧栈和新栈使用指针连接起来，函数返回就释放。

1. 每个Go函数的开头都有一小段检测代码。这段代码会检查我们是否已经用完了分配的栈空 间。如果是的话，它会调用 morestack 函数。 morestack 函数分配一块新的内存作为栈空间，并且在这块栈空间 的底部填入各种信息(包括之前的那块栈地址)。在分配了这块新的栈空间之后，它会重试刚才造成栈空间不足的函数。这个过程叫做栈分裂(stack split).

		2.	在新分配的栈底有个lessstack的函数指针; 当我们从那个函数返回时，它会跳转到 lessstack 。 lessstack 函 数会查看在栈底部存放的数据结构里的信息，然后调整栈指针(stack pointer)。这样就完成了从新的栈块到老的 栈块的跳转。接下来，新分配的这个块栈空间就可以被释放掉了。

问题:

- 多次循环调用同一个函数会出现“hot split”问题, 即如果函数产生的返回在一个循环或者递归中, 会频繁的alloc/free,导致严重性能问题
- 每次分配和释放都要额外消耗

##### 连续栈

连续栈的实现方式：当检测到需要更多栈时，分配一块比原来大一倍的栈，把旧栈数据copy到新栈，释放旧栈

![img](https://raw.githubusercontent.com/noobmid/pics/main/dGeih5.png)

1. 栈的扩缩容何时触发?

   goroutine运行并用完栈空间的时候，与之前的方法一样，栈溢出检查会被 触发

   

2. 栈的扩缩容大小

   扩容为原来的两倍,缩容为原来的1/2

   

3. 栈的扩缩容过程中做了哪些事?

   重新申请一块新栈，然后把旧栈的数据复制到新栈。协程占用的物理内存完全被替换了，而Go在运行时会把指针保存到内存里面，例如：`gp.sched.ctxt` ，`gp._defer` ，`gp._panic`，包括函数里的指针。这部分指针值会被转换成整数型`uintptr`，然后 `+ delta`进行调整

    如果栈空间发现不够用，会调用`stackalloc`分配一块新的栈，大小比原来大一倍进行扩容 ;栈的缩容主要是发生在GC期间.



#### 内存泄漏

1. **字符串截取**:   

   解决办法: string和[]byte 转化

   

2. **切片截取**

   解决办法:  append

   

3. **没有重置丢失的子切片元素中的指针:** 原切片元素为指针类型，原切片被截取后，丢失的子切片元素中的指针元素未被置空，导致内存泄漏

   解决办法:元素置空 

   

4. **函数数组传参**: 由于数组为值类型,赋值和函数传参会复制整个数组; 如果数组较大,短时间内传递多次,会消耗大量内存又来不及gc,就会产生临时性的内存泄漏

   解决办法:采用指针传递、使用切片

   

5. **gorouting**  :有些编码不当的情况下，goroutine被长期挂住，导致该协程中的内存也无法被释放，就会造成永久性的内存泄漏。例如协程结束时协程中的channel没有关闭，导致一直阻塞；例如协程中有死循环

   可见并发和sync包的使用

   

6. 定时器: 定时器未到触发时间，该定时器不会被gc回收，从而导致临时性的内存泄漏，而如果定时器一直在创建，那么就造成了永久性的内存泄漏了

   解决办法:创建timer定时器，每次需要启动定时器的时候，使用Reset方法重置定时器

   
   
7. 外部资源没有办法GC的:  如打开的文件句柄

#### 内存泄漏实例

```go
func main() {
   num := 6
   for index := 0; index < num; index++ {
      resp, _ := http.Get("https://www.baidu.com")
      _, _ = ioutil.ReadAll(resp.Body)
   }
   fmt.Printf("此时goroutine个数= %d\n", runtime.NumGoroutine())
}// 没有执行resp.Body.Close(), 一共泄漏了3个goroutine
```

> - 虽然执行了 `6` 次循环，而且每次都没有执行 `Body.Close()` ,就是因为执行了`ioutil.ReadAll()`把内容都读出来了，连接得以复用，因此只泄漏了一个`读goroutine`和一个`写goroutine`，最后加上`main goroutine`，所以答案就是`3个goroutine`
> - 正常情况下我们的代码都会执行 `ioutil.ReadAll()`，但如果此时忘了 `resp.Body.Close()`，确实会导致泄漏。但如果你**调用的域名一直是同一个**的话，那么只会泄漏一个 `读goroutine` 和一个`写goroutine`，**这就是为什么代码明明不规范但却看不到明显内存泄漏的原因**。





#### 内存对齐

> **一个非空结构体包含有尾部size为0的变量(字段)，如果不给它分配内存，那么该变量(字段)的指针地址将指向一个超出该结构体内存范围的内存空间。这可能会导致内存泄漏，或者在内存垃圾回收过程中，程序crash掉。**

![img](https://raw.githubusercontent.com/noobmid/pics/main/v2-97a135f5b4ab1a0a7db2998f2e518918_1440w.jpg)

- 为什么对齐?

  操作系统并非一个字节一个字节访问内存，而是按`2, 4, 8`这样的字长来访问。当被访问的数据长度为 `n` 字节且该数据地址为`n`字节对齐，那么操作系统就可以高效地一次定位到数据，无需多次读取、处理对齐运算等额外操作.

- struct 的对齐是：如果类型 t 的对齐保证是 n，那么类型 t 的每个值的**地址**在运行时必须是 n 的倍数。
- struct 内字段如果填充过多，可以尝试重排，使字段排列更紧密，减少内存浪费
- 零大小字段要避免作为 struct 最后一个字段，会有内存浪费(零大小也会补一个对齐保证的长度,防止指针错误,内存泄漏)

对齐规则:

- 对于任意类型的变量 x ，unsafe.Alignof(x) 至少为 1。
- 对于 struct 结构体类型的变量 x，计算 x 每一个字段 f 的 unsafe.Alignof(x.f)，unsafe.Alignof(x) 等于其中的最大值。
- 对于 array 数组类型的变量 x，unsafe.Alignof(x) 等于构成数组的元素类型的对齐倍数。
- 没有任何字段的空 struct{} 和没有任何元素的 array 占据的内存空间大小为 0，不同的大小为 0 的变量可能指向同一块地址。



### 4. 并发和sync包

#### Data Race

​		同步访问共享数据是处理数据竞争的一种有效的方法. 可以使用 go run - race 或者 go build -race来进行静态检测。其在内部的实现是,开启多个协程执行同一个命令， 并且记录下每个变 量的状态.

```go
go test -race mypkg // 测试包
go run -race mysrc.go // 编译和运行程序 
go build -race mycmd // 构建程序
go install -race mypkg // 安装程序
```

解决数据竞争的方法: 

- 互斥锁: sync.Mutex、sync.WaitGroup
- 通道: channel ,channel的效率是高于互斥锁的



#### 并发模型

- 通过channel通知实现并发控制

- 通过sync包中的WaitGroup实现并发控制

  a. 	Add(), 可以添加或减少 goroutine的数量.

  b. 	Done(), 相当于Add(-1).
  c. 	Wait(), 执行后会堵塞主线程，直到WaitGroup 里的值减至0.

  > 注意,在 WaitGroup 第一次使用后，不能被拷贝
  >
  > ```go
  >  func main(){
  >  wg := sync.WaitGroup{}
  >     for i := 0; i < 5; i++ {
  >         wg.Add(1)
  >         go func(wg sync.WaitGroup, i int) {
  >             fmt.Printf("i:%d", i)
  >             wg.Done()
  > }(wg, i) }
  > wg.Wait()
  >     fmt.Println("exit")
  > } // error: all goroutines are asleep - deadlock!
  > ```
  >
  > 因为 wg 给拷⻉传递到了 goroutine 中，导致只有 Add 操作，其实 Done操作是在 wg 的副本执行的。
  >
  > 可以将 传入类型改为 *sync.WaitGroup, 或者使用闭包

- Context 上下文

  context 包主要是用来处理多个 goroutine 之间共享数据，及多个 goroutine 的管理。Context 对象是线程安全的，你可以把一个 Context 对象传递给任意个数的 gorotuine，对它执行取消 操作时， 所有 goroutine 都会接收到取消信号。
  
  1. web编程中，一个请求对应多个goroutine之间的数据交互、同步数据，主要是共享数据，如token
  
  2. 超时控制：请求被取消或是处理时间太长，这有可能是使用者关闭了浏览器或是已经超过了请求方规定的超时时间，请求方直接放弃了这次请求结果。这时，所有正在为这个请求工作的 goroutine 需要快速退出，因为它们的“工作成果”不再被需要了。在相关联的 goroutine 都退出后，系统就可以回收相关的资源。
  
  3. 上下文控制

#### CAS

 Compare And Swap，直译就是比较交换;是一种实现并发算法时常用到的技术.

作用是让 CPU先进行比较两 个值是否相等，然后原子地更新某个位置的值，其实现方式是给予硬件平台的汇编指令，

```go
func CompareAndSwapUint32(addr *uint32, old, new uint32) (swapped bool)
```

![img](https://raw.githubusercontent.com/noobmid/pics/main/v2-99995e2de042f2cf723c731f65c369db_1440w.jpg)

缺陷: 

1. CAS在共享资源竞争比较激烈的时候，每个goroutine会容易处于自旋状态，影响效率，在竞争激烈的时候推荐使用锁。

2. 无法解决ABA问题
   ABA问题是无锁结构实现中常见的一种问题，可基本表述为：

   > 进程P1读取了一个数值A
   > P1被挂起(时间片耗尽、中断等)，进程P2开始执行
   > P2修改数值A为数值B，然后又修改回A
   > P1被唤醒，比较后发现数值A没有变化，程序继续执行。



#### Sync包

##### 互斥锁: sync.Mutex

```go
 //Mutex 是互斥锁， 零值是解锁的互斥锁， 首次使用后不得复制互斥锁。 
type Mutex struct {
	state int32
	sema  uint32 
}

//Locker表示可以锁定和解锁的对象。 type Locker interface {
Lock()
Unlock() }
//锁定当前的互斥量 //如果锁已被使用，则调用goroutine //阻塞直到互斥锁可用。
func (m *Mutex) Lock()
//对当前互斥量进行解锁 
//如果在进入解锁时未锁定m，则为运行时错误。 
//锁定的互斥锁与特定的goroutine无关。 
//允许一个goroutine锁定Mutex然后安排另一个goroutine来解锁它。 
func (m *Mutex) Unlock()
```

- Mutex的几种状态:	mutexLocked —表示互斥锁的锁定状态;	mutexWoken —表示从正常模式被从唤醒;	mutexStarving —当前的互斥锁进入饥饿状态;	

- Mutex的正常模式和饥饿模式:

  1. 正常模式**(**非公平锁): 	正常模式下，所有等待锁的 goroutine 按照 FIFO(先进先出)顺序等待。唤醒的 goroutine 不会直接拥有锁，而是会 和新请求锁的 goroutine 竞争锁的拥有。但是和正在使用cpu的goroutine相比很大可能失败.

  2. 饥饿模式**(**公平锁):      一个等待的 goroutine 超过1ms没有获取锁,或者当前队列只剩下一个 g 的时候那么它将会把锁转 变为饥饿模式。

     ​		饥饿模式下，直接由 unlock 把锁交给等待队列中排在第一位的 G(队头)，同时，饥饿模式下，新进来的 G不会参与 抢锁也不会进入自旋状态，会直接进入等待队列的尾部,这样很好的解决了老的 g 一直抢不到锁的场景

  3. 自旋锁: 循环等待锁的释放,一直处于内核态,不进行内核态和用户态切换.

     	1. 锁已被占用，并且锁不处于饥饿模式
     	1. 积累的自旋次数小于最大自旋次数(active_spin=4)。
     	1.  cpu 核数大于1。
     	1. 有空闲的 P
     	1. 当前 goroutine 所挂载的 P下，本地待运行队列为空。

     ​		



##### 读写锁: sync.RWMutex

1 多个写操作之间是互斥的 

2 写操作与读操作之间也是互斥的

 3 多个读操作之间不是互斥的

```go
// RWMutex是一个读/写互斥锁，可以由任意数量的读操作或单个写操作持有。
// RWMutex的零值是未锁定的互斥锁。
//首次使用后，不得复制RWMutex。 
//如果goroutine持有RWMutex进行读取而另一个goroutine可能会调用Lock，那么在释放初始读锁之前， goroutine不应该期望能够获取读锁定。
//特别是，这种禁止递归读锁定。 这是为了确保锁最终变得可用; 阻止的锁定会阻止新读操作获取锁定。
type RWMutex struct {
  		w 					Mutex	//如果有待处理的写操作就持有 uint32 
      writerSem		int32	// 写操作等待读操作完成的信号量 
      readerSem		uint32 //读操作等待写操作完成的信号量
      readerCount int32		// 待处理的读操作数量
      readerWait  int32  // number of departing readers
}
 
//对读操作的锁定
func (rw *RWMutex) RLock() //对读操作的解锁
func (rw *RWMutex) RUnlock() //对写操作的锁定
func (rw *RWMutex) Lock() //对写操作的解锁
func (rw *RWMutex) Unlock()
//返回一个实现了sync.Locker接口类型的值，实际上是回调rw.RLock and rw.RUnlock. func (rw *RWMutex) RLocker() Locker
```

​		通过记录 readerCount 读锁的数量来进行控制，当有一个写锁的时候，会将读锁数量设置为负数1<<30。目的是让 新进入的读锁等待写锁之后释放通知读锁。同样的写锁也会等等待之前的读锁都释放完毕，才会开始进行后续的操 作。而等写锁释放完之后，会将值重新加上1<<30,并通知刚才新进入的读锁(rw.readerSem)，两者互相限制。

> RWMutex 的读锁不要用于递归调用，比较容易产生死锁。
>
> 写锁被解锁后，所有因操作锁定读锁而被阻塞的 goroutine 会被唤醒，并都可以成功锁定读锁。
>
> 读锁被解锁后，在没有被其他读锁锁定的前提下，所有因操作锁定写锁而被阻塞的 goroutine，其中等待时间 最⻓的一个 goroutine 会被唤醒。	



##### 安全锁: Sync.Map

golang中的sync.Map是并发安全的，其实也就是sync包中golang自定义的一个名叫Map的结构体。它通过空间换时间的方式，使用 read 和 dirty 两个 map 来进行读写分离，降低锁时间来提高效率。

![img](https://raw.githubusercontent.com/noobmid/pics/main/v2-e96c8332e9451c5fc701fc914e2bf238_1440w.jpg)

```go
type Map struct {
    mu Mutex	// 该锁用来保护dirty
    read atomic.Value // readOnly// 存读的数据，因为是atomic.value类型，只读类型，所以它的读是并发安全的
    dirty map[interface{}]*entry	//包含最新的写入的数据，并且在写的时候，会把read中未被删除的数据拷⻉到该dirty中，因为是普通的map存在并发安全问题，需要用到上面的mu字段。
    misses int	// 从read读数据失败的时候，会将该字段+1，当等于len(misses)的时候，会将dirty拷⻉到read中(从而提升读的性能)。
}

func (m *Map) Delete(key interface{}) // 删除元素
func (m *Map) Store(key, value interface{}) //添加元素
func (m *Map) Load(key interface{})// 查找key的值
func (m *Map) LoadOrStore(key, value interface{})
func (m *Map) Range(f func(key, value interface{}) bool)便利
```

   1. store() 流程

      ​	a. 如果在 read 里能够找到待存储的 key，并且对应的 entry 的 p 值不为 expunged，也就是没被删除时，直接更新对应的 entry 即可

      ​	b. 第一步没有成功：要么 read 中没有这个 key，要么 key 被标记为删除。则先加锁，再进行后续的操作

      ​	c. 再次在 read 中查找是否存在这个 key，也就是 double check 一下;如果 read 中存在该 key，但 `p == expunged`,说明 m.dirty != nil 并且 m.dirty 中不存在该 key 值 此时: a. 将 p 的状态由 expunged 更改为 nil；b. dirty map 插入 key。然后，直接更新对应的 value。(删除老key,新值写入dirty中)

      ​	d. 如果 read 中没有此 key，那就查看 dirty 中是否有此 key，如果有，则直接更新对应的 value，这时 read 中还是没有此 key

      ​	e. 如果 read 和 dirty 中都不存在该 key，则：a. 如果 dirty 为空，则需要创建 dirty，并从 read 中拷贝未被删除的元素；b. 更新 amended 字段，标识 dirty map 中存在 read map 中没有的 key；c. 将 k-v 写入 dirty map 中，read.m 不变。最后，更新此 key 对应的 value。

2. Load()

   ​	a. 直接在 read 中找，如果找到了直接调用 entry 的 load 方法，取出其中的值。

   ​	b. 如果 read 中没有这个 key，且 amended 为 fase，说明 dirty 为空，那直接返回 空和 false。

   ​	c. 如果 read 中没有这个 key，且 amended 为 true，说明 dirty 中可能存在我们要找的 key。当然要先上锁，再尝试去 dirty 中查找。在这之前，仍然有一个 double check 的操作。若还是没有在 read 中找到，那么就从 dirty 中找。不管 dirty 中有没有找到，都要"记一笔"，因为在 dirty 被提升为 read 之前，都会进入这条路径

3. Delete()

​					a. 现在read里找, 找到了则将p 设置为nil

​					b.  如果 read 中没有这个 key，且 dirty map 不为空,则在dirty中找.在 dirty中直接删除这个key



##### sync.WaitGroup

```go
type WaitGroup struct {
    noCopy noCopy
    // 64-bit value: high 32 bits are counter, low 32 bits are waiter count.
    // 64-bit atomic operations require 64-bit alignment, but 32-bit
    // compilers do not ensure it. So we allocate 12 bytes and then use
    // the aligned 8 bytes in them as state, and the other 4 as storage
    // for the sema.
    state1 [3]uint32
}
```

- WaitGroup 主要维护了2 个计数器，一个是请求计数器 v，一个是等待计数器 w，二者组成一个64bit 的值， 请求计数器占高32bit，等待计数器占低32bit。
-  每次 Add执行，请求计数器 v 加1，Done方法执行，请求计数器减1，v 为0 时通过信号量唤醒 Wait()。



##### sync.Once

```go
type Once struct {
	// done indicates whether the action has been performed.
	// It is first in the struct because it is used in the hot path.
	// The hot path is inlined at every call site.
	// Placing done first allows more compact instructions on some architectures (amd64/x86),
	// and fewer instructions (to calculate offset) on other architectures.
	done uint32
	m    Mutex
}

func (o *Once) Do(f func()) {
	// Note: Here is an incorrect implementation of Do:
	//
	//	if atomic.CompareAndSwapUint32(&o.done, 0, 1) {
	//		f()
	//	}
	//  Do 保证当它返回时，f 已完成。此实现不会实现该保证：给定两个同时调用，cas的获胜者将调用f，第二个将立即返回，而无需等待第一个对 f 的调用完成。这就是为什么慢速路径回落到mutex，以及为什么atomic.StoreUint32必须延迟到 f 返回之后。

	if atomic.LoadUint32(&o.done) == 0 {
		// Outlined slow-path to allow inlining of the fast-path.
		o.doSlow(f)
	}
}

func (o *Once) doSlow(f func()) {
	o.m.Lock()
	defer o.m.Unlock()
	if o.done == 0 {
		defer atomic.StoreUint32(&o.done, 1)
		f()
	}
}
```

`done`是标识位，用来判断方法`f`是否被执行完，其初始值为0，当`f`执行结束时，`done`被设为1

- Once 可以用来执行且仅仅执行一次动作，常常用于**单例对象**的初始化场景。
-  Once 常常用来初始化单例资源，或者并发访问只需初始化一次的共享资源，或者在测试的时候初始化一次测 试资源。
-  sync.Once 只暴露了一个方法 Do，你可以多次调用 Do 方法，但是只有第一次调用 Do 方法时 f 参数才会执 行，这里的 f 是一个无参数无返回值的函数。
- 不可嵌套,否则死锁 



##### sync.Pool

```go
// Local per-P Pool appendix.
type poolLocalInternal struct {
	private interface{}   // Can be used only by the respective P.
	shared  []interface{} // Can be used by any P.
	Mutex                 // Protects shared.
}

type poolLocal struct {
	poolLocalInternal

	// Prevents false sharing on widespread platforms with
	// 128 mod (cache line size) = 0 .
	pad [128 - unsafe.Sizeof(poolLocalInternal{})%128]byte
}
```

作用:	对于很多需要重复分配、回收内存的地方，sync.Pool 是一个很好的选择。频繁地分配、回收内存会给 GC 带来一 定的负担，严重的时候会引起 CPU 的毛刺，而 sync.Pool 可以将暂时不用的对象缓存起来，待下次需要的时候直 接使用，不用再次经过内存分配，复用对象的内存，减轻 GC 的压力，提升系统的性能。

##### sync.cond















#### 协程同步问题: 主协程如何等其余协程完再操作?

1. channel同步: 利用引用计数

```go
package main
import (
	"fmt"
)
func printString(str string) {
	for _, data := range str {
		fmt.Printf("%c", data)
	}
	fmt.Printf("\n")
}
var ch = make(chan int)
var tongBu = make(chan int)
func person1() {
	printString("Gerald")
	tongBu <- 1
	ch <- 1
}
func person2() {
	<- tongBu
	printString("Seligman")
	ch <- 2
}
func main() {
	// 目的：使用 channel 来实现 person1 先于 person2 执行
	go person1()
	go person2()
	count := 2
	// 判断所有协程是否退出
	for range ch {
		count--
		if 0 == count {
			close(ch)
		}
	}
}
```

2. Sync.WaitGroup





### 5. golang编译过程

 go build -gcflags -S main.go 可以生成中间的汇编代码

1. 词法分析：源码翻译成token（分词）
2. 语法分析：将token序列输出成AST结构
3. 词义分析： 类型检查（类型推断等）
4. 中间码生产：对于不同的操作系统和硬件进行处理；提高后端编译的重用
5. 代码优化： 
   1. 并行性，充分利用现在多核计算机的特性
   2. 流水线，cpu 有时候在处理 a 指令的时候，还能同时处理 b 指令
   3.  指令的选择，为了让 cpu 完成某些操作，需要使用指令，但是不同的指令效率有非常大的差别，这里会进行指令优化
   4. 利用寄存器与高速缓存，我们都知道 cpu 从寄存器取是最快的，从高速缓存取次之。这里会进行充分的利用
6. 机器码生产： 先生成汇编代码，其汇编器使用GOARCH参数进行初始化，然后调用对应架构便携的特定方法来生成机器码，从而跨平台。

![截屏2022-08-27 下午4.21.07](https://raw.githubusercontent.com/noobmid/pics/main/%E6%88%AA%E5%B1%8F2022-08-27%20%E4%B8%8B%E5%8D%884.21.07.png)

