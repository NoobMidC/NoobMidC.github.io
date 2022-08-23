# Golang


<!--more-->

# Golang 八股文

------

## 基础篇

### 1. **go的指针和c的指针**

​		相同点:			

- ​	运算符相同, &为取地址, *为解引用

​		不同点:

- ​	 数组名和数组首地址. **c语言**中arr、&arr[0]为数组的首个**元素**地址,单位偏移量为元素大小; &arr为**数组**的首地址,单位偏移量为数组大小. **golang**中&arr[0]和&arr与c语言中相同, 但是arr表示为整个数组的值.

  ```go
  // C
  int arr[5] = {1, 2, 3, 4, 5};
  // Go
  // 需要指定长度，否则类型为切片
  arr := [5]int{1, 2, 3, 4, 5}
  ```

- ​    指针运算.   **c语言**中指针本质为无符号整数,代表内存地址, 可以进行加减运算.    **Golang**中指针为 `*uint32`类型非数字,不可以加减运算.

  > Go 标准库中提供了一个 `unsafe` 包用于编译阶段绕过 Go 语言的类型系统，直接操作内存
  >
  > `uintptr` : Go 的内置类型。是一个无符号整数，用来存储地址，支持数学运算。常与 `unsafe.Pointer` 配合做指针运算
  >
  > `unsafe.Pointer` : 表示指向任意类型的指针，可以和任何类型的指针互相转换（类似 C 语言中的 `void*` 类型的指针），也可以和 `uintptr` 互相转换
  >
  > `unsafe.Sizeof` : 返回操作数在内存中的字节大小，参数可以是任意类型的表达式，例如 fmt.Println(unsafe.Sizeof(uint32(0)))的结果为 4
  >
  > `unsafe.Offsetof` : 函数的参数必须是一个字段 x.f，然后返回 f 字段相对于 x 起始地址的偏移量，用于计算结构体成员的偏移量



### 2. String的底层结构

```go
type StringHeader struct {  // 16 字节
	Data uintptr
	Len  int
}
```

![string底层结构](https://raw.githubusercontent.com/noobmid/pics/main/string%E5%BA%95%E5%B1%82%E7%BB%93%E6%9E%84.png)

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



### 4. Map底层结构

用于存储一系列无序的键值对

Go语言采用的是哈希查找表，并且使用链表解决哈希冲突

```go
// hashmap的简称
type hmap struct {
 count     int       //元素个数
 flags     uint8     //标记位
 B         uint8     //buckets的对数 log_2
 noverflow uint16    //overflow的bucket的近似数
 hash0     uint32    //hash种子
 buckets    unsafe.Pointer //指向buckets数组的指针，数组个数为2^B
 oldbuckets unsafe.Pointer //扩容时使用，buckets长度是oldbuckets的两倍
 nevacuate  uintptr  //扩容进度，小于此地址的buckets已经迁移完成
 extra *mapextra     //扩展信息
}

//当map的key和value都不是指针，并且size都小于128字节的情况下，会把 bmap 标记为不含指针，这样可以避免gc时扫描整个hmap。但是，我们看bmap其实有一个overflow的字段，是指针类型的，破坏了bmap不含指针的设想，这时会把overflow移动到extra字段来。
type mapextra struct {
 overflow    *[]*bmap
 oldoverflow *[]*bmap
 nextOverflow *bmapx
}

// bucket
type bmap struct {
 tophash [bucketCnt]uint8 //bucketCnt = 8
 // keys     [8]keytype
 // values   [8]valuetype
 // pad      uintptr
 // overflow uintptr
}
```



### 4. new 和make的区别

- new(T) 返回的是 T 的指针：new(T) 为一个 T 类型新值分配空间并将此空间初始化为 T 的零值，返回的是新值的地址，也就是 T 类型的指针 *T，该指针指向 T 的新分配的零值。

- make 只能用于 slice,map,channel，返回值是经过初始化之后的 T 的引用
- make 分配空间后，会进行初始化

### 5.struct能不能比较

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

### 6. defer

-  defer关键字定义的函数是在调用函数返回之后执行，而不是在代码块退出之后执行。defer的执行顺序是先创建的后执行。看做是一个 `FILO`(First In Last Out) 栈.
- 所有传入defer函数的参数都是在创建的时候立即计算处理的，而不是调用函数退出的时候计算的
- defer等到包含它的程序返回时(包含它的函数执行了return语句、运行到函数结尾自动返回、对应的goroutine panic）defer函数才会被执行。通常用于资源释放、打印日志、异常捕获等



### 7. init函数

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

![在这里插入图片描述](https://raw.githubusercontent.com/noobmid/pics/main/watermark%252Ctype_d3F5LXplbmhlaQ%252Cshadow_50%252Ctext_Q1NETiBA5bCP6I-c6bih5pys6I-c%252Csize_20%252Ccolor_FFFFFF%252Ct_70%252Cg_se%252Cx_16.png)



### 8.包的循环引用

- 为什么不允许循环引用

  ​	加快编译速度、规范框架设计，使项目结构更加清晰明了

- 解决办法:

					1. mvc 结构,将包规划好
					1. 新建公共接口包(父包), 将需要循环调用的函数或方法抽象为接口
					1. 新建公共组合包(子包), 在组合包中组合调用
					1. 全局存储需要相互依赖的函数, 通过关键字进行调用



### 9. SELECT

1. select可以用来等待多个channel的可读可写，其中select中的case表达式必须都是channel的读写操作。

2. 当存在default时，select执行的就是非阻塞收发，当不存在时，必须等待某一个channel可读或可写。

3. 当多个channel都可读可写时，会随机选择一个分支。

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





------



## 进阶






