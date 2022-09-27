# 数据结构


<!--more-->

# 数据结构

## B树 和 B+树

1. 多路平衡查找树; 即B树

![截屏2022-09-13 下午5.37.58](https://raw.githubusercontent.com/NoobMidC/pics/main/截屏2022-09-13 下午5.37.58.png)

2. B+树

![截屏2022-09-13 下午5.38.39](https://raw.githubusercontent.com/NoobMidC/pics/main/%E6%88%AA%E5%B1%8F2022-09-13%20%E4%B8%8B%E5%8D%885.38.39.png)

- B+跟B树不同B+树的非叶子节点不保存关键字记录的指针，只进行数据索引，这样使得B+树每个非叶子节点所能保存的关键字大大增加；
- B+树叶子节点保存了父节点的所有关键字记录的指针，所有数据地址必须要到叶子节点才能获取到。所以每次数据查询的次数都一样；

3. B 和 B+树的区别

   - B+树的层级更少：相较于B树B+每个非叶子节点存储的关键字数更多，树的层级更少所以查询数据更快；
   - B+树查询速度更稳定：B+所有关键字数据地址都存在叶子节点上，所以每次查找的次数都相同所以查询速度要比B树更稳定;
   - B+树天然具备排序功能：B+树所有的叶子节点数据构成了一个有序链表，在查询大小区间的数据时候更方便，数据紧密性很高，缓存的命中率也会比B树高。
   - B+树全节点遍历更快：B+树遍历整棵树只需要遍历所有的叶子节点即可，而不需要像B树一样需要对每一层进行遍历，这有利于数据库做全表扫描
   - B树相对于B+树的优点是，如果经常访问的数据离根节点很近，而B树的非叶子节点本身存有关键字其数据的地址，所以这种数据检索的时候会要比B+树快。

4. B+的应用场景

   B/B+树是为了磁盘或其它存储设备而设计的一种平衡多路查找树；与红黑树相比,在相同的的节点的情况下,一颗B/B+树的高度远远小于红黑树的高度



## 平衡二叉树

​		**平衡二叉查找树**：简称平衡二叉树。也称为 AVL 树。windows对进程地址空间的管理用到了AVL; 它具有如下几个性质:

1. 可以是空树。
2. 假如不是空树，任何一个结点的左子树与右子树都是平衡二叉树，并且高度之差的绝对值不超过 1。
3. 左节点比根节点小, 右节点比根节点小

![img](https://raw.githubusercontent.com/NoobMidC/pics/main/format%252Cpng.png)



## 红黑树

​		学习二叉搜索树、平衡二叉树时，我们不止一次提到，二叉搜索树容易退化成一条链; 这时，查找的时间复杂度从O ( l o g 2 N ) 也将退化成O ( N ) ;  引入对左右子树高度差有限制的平衡二叉树，保证查找操作的最坏时间复杂度也为O ( l o g 2 N ) 

> - AVL的左右子树高度差不能超过1，每次进行插入/删除操作时，几乎都需要通过旋转操作保持平衡
> - 在频繁进行插入/删除的场景中，频繁的旋转操作使得AVL的性能大打折扣
> - 红黑树通过牺牲严格的平衡，换取插入/删除时少量的旋转操作，整体性能优于AVL
> - 红黑树插入时的不平衡，不超过两次旋转就可以解决；删除时的不平衡，不超过三次旋转就能解决
> - 红黑树的红黑规则，保证最坏的情况下，也能在O ( l o g 2 N ) 时间内完成查找操作。

红黑树规则:

1. 节点不是红色就是黑色，根节点是黑色
2. 叶节点为黑色（叶节点是指末梢的空节点 `Nil`或`Null`）
3. 一个节点为红色，则其两个子节点必须是黑色的（根到叶子的所有路径，不可能存在两个连续的红色节点）
4. 每个节点到叶子节点的所有路径，都包含相同数目的黑色节点（相同的黑色高度）

![在这里插入图片描述](https://raw.githubusercontent.com/NoobMidC/pics/main/watermark%252Ctype_ZmFuZ3poZW5naGVpdGk%252Cshadow_10%252Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQ0NTQ1Mzg%253D%252Csize_16%252Ccolor_FFFFFF%252Ct_70.png)



### AVL和红黑树的区别

- 和红黑树相比，AVL树是严格的平衡二叉树，平衡条件必须满足。
- 平衡二叉树相当于全局平衡，而红黑树相当于局部平衡。维护全局平衡耗费的代价较高



### 红黑树和哈希表对比

- 红黑树是有序的，Hash是无序的，根据需求来选择。
- 红黑树占用的内存更小（仅需要为其存在的节点分配内存），而Hash事先应该分配足够的内存存储散列表,即使有些槽可能弃用
- 红黑树查找和删除的时间复杂度都是O(logn)，Hash查找和删除的时间复杂度都是O(1)。

### 红黑树应用场景

- 多路复用技术的Epoll,其核心结构是红黑树 + 双向链表。IO多路复用epoll的实现采用红黑树组织管理sockfd，以支持快速的增删改查.
- ngnix中,用红黑树管理timer,因为红黑树是有序的,可以很快的得到距离当前最小的定时器.
- 广泛应用在C++STL中，比如map和set，Java的TreeMap

## Hash表

### **Hash函数构造方法**

1. 直接定制法——仅适用于地址大小=关键字的情况

   该方法构造的Hash函数为线性函数， H(key)=a*key+b。这种哈希函数构造简单，且不会产生哈希冲突，但限制较大。

2. **除留余数法——最常用**

   H（key）=key MOD p

3. **数字分析法——数字位数大、且有规律可循（事先知道数据分布）**

4. 平方取中位数法——数据位数小（事先不知道数据分布）

5. **折叠法——数字位数大（事先不知道数据分布）**



### **Hash冲突解决方法**

1. 开放地址方法

   ​		关键字key的哈希地址p=H（key）出现冲突时，以p为基础，产生另一个哈希地址p1，如果p1仍然冲突，再以p为基础，产生另一个哈希地址p2，…，直到找出一个不冲突的哈希地址pi ，将相应元素存入其中;

   

2. 链式地址法

   ​        将所有哈希地址为i的元素构成一个称为同义词链的单链表，并将单链表的头指针存在哈希表的第i个单元中，因而查找、插入和删除主要在同义词链中进行。链地址法适用于经常进行插入和删除的情况。

   

3. 建立公共溢出区

   ​       将Hash表分为基本表和溢出表两部分，在基本表中发生冲突的元素都放入溢出表中。

4. 再哈希法

   ​          当哈希地址Hi=RH1（key）发生冲突时，再计算Hi=RH2（key）……，直到冲突不再产生。这种方法不易产生聚集，但增加了计算时间。



## 排序算法

#### 基础排序

- 冒泡排序(稳定)

```go
func sortArray(nums []int) []int {
  // 冒泡排序，比较交换，稳定算法，时间O(n^2), 空间O(1)
	// 每一轮遍历，将该轮最大值放到后面，同时小的往前冒
	// 从而形成后部是有序区
	// compare and swap
	for i:=0;i<len(nums);i++ {
		// 适当剪枝，len()-i到最后的部分都是有序区，避免再排
		for j:=1;j<len(nums)-i;j++ {
			if nums[j-1] > nums[j] {
				nums[j-1], nums[j] = nums[j], nums[j-1]
			}
		}
	}
	return nums
}
```

- 选择排序(不稳定)

```go
func sortArray(nums []int) []int {
	// 选择排序，比较交换，不稳定算法，时间O(n^2)，空间O(1)
	// 每一轮遍历，该轮的最小值前挪，从而形成前面部分是有序区
	// compare and swap
	for i:=0;i<len(nums);i++ {
		// 剪枝前面部分，比较后面部分
		for j:=i+1;j<len(nums);j++ {
			if nums[i] > nums[j] {
				nums[i], nums[j] = nums[j], nums[i]
			}
		}
	}
	return nums
}
```

- 插入排序(稳定)

```go
func sortArray(nums []int) []int {
	// 插入排序，比较交换，稳定算法，时间O(n^2)，空间O(1)
	// 0->len方向，每轮从后往前比较，相当于找到合适位置，插入进去
	// 数据规模小的时候，或基本有序，效率高
	n := len(nums)
	for i := 1; i < n; i++ {
		tmp := nums[i]
		j := i - 1
		for j >= 0 && nums[j] > tmp { //左边比右边大
			nums[j+1] = nums[j] //右移1位
			j--                 //扫描前一个数
		}
		nums[j+1] = tmp //添加到小于它的数的右边
	}
	return nums
}
```

#### 改良排序

- 希尔排序(不稳定)

```go
func sortArray(nums []int) []int {
	// 希尔排序，比较交换，不稳定算法，时间O(nlog2n)最坏O(n^2), 空间O(1)
	// 改进插入算法
	// 每一轮按照间隔插入排序，间隔依次减小，最后一次一定是1
	/*
	主要思想：
	设增量序列个数为k，则进行k轮排序。每一轮中，
	按照某个增量将数据分割成较小的若干组，
	每一组内部进行插入排序；各组排序完毕后，
	减小增量，进行下一轮的内部排序。
	*/
	gap := len(nums)/2
	for gap > 0 {
		for i:=gap;i<len(nums);i++ {
			j := i
			for j-gap >= 0 && nums[j-gap] > nums[j] {
				nums[j-gap], nums[j] = nums[j], nums[j-gap]
				j -= gap
			}
		}
		gap /= 2
	}
	return nums
}
```

- 归并排序(稳定)

```go
// 递归实现归并算法
func sortArray(nums []int) []int {
	// 归并排序，基于比较，稳定算法，时间O(nlogn)，空间O(logn) | O(n)
	// 基于递归的归并-自上而下的合并，
	// 都需要开辟一个大小为n的数组中转
	// 将数组分为左右两部分，递归左右两块，最后合并，即归并
	merge := func(left, right []int) []int {
		res := make([]int, len(left)+len(right))
		var l,r,i int
		// 通过遍历完成比较填入res中
		for l < len(left) && r < len(right) {
			if left[l] <= right[r] {
				res[i] = left[l]
				l++
			} else {
				res[i] = right[r]
				r++
			}
			i++
		}
		// 如果left或者right还有剩余元素，添加到结果集的尾部
		copy(res[i:], left[l:])
		copy(res[i+len(left)-l:], right[r:])
		return res
	}
	var sort func(nums []int) []int
	sort = func(nums []int) []int {
		if len(nums) <= 1 {
			return nums
		}
		// 拆分递归与合并
		// 分割点
		mid := len(nums)/2
		left := sort(nums[:mid])
		right := sort(nums[mid:])
		return merge(left, right)
	}
	return sort(nums)
}


// 非递归实现归并算法
func sortArray(nums []int) []int {
	if len(nums) <= 1 {return nums}
	merge := func(left, right []int) []int {
		res := make([]int, len(left)+len(right))
		var l,r,i int
		// 通过遍历完成比较填入res中
		for l < len(left) && r < len(right) {
			if left[l] <= right[r] {
				res[i] = left[l]
				l++
			} else {
				res[i] = right[r]
				r++
			}
			i++
		}
		// 如果left或者right还有剩余元素，添加到结果集的尾部
		copy(res[i:], left[l:])
		copy(res[i+len(left)-l:], right[r:])
		return res
	}
	i := 1 //子序列大小初始1
	res := make([]int, 0)
	// i控制每次划分的序列长度
	for i < len(nums) {
		// j根据i值执行具体的合并
		j := 0
		// 按顺序两两合并，j用来定位起始点
		// 随着序列翻倍，每次两两合并的数组大小也翻倍
		for j < len(nums) {
			if j+2*i > len(nums) {
				res = merge(nums[j:j+i], nums[j+i:])
			} else {
				res = merge(nums[j:j+i], nums[j+i:j+2*i])
			}
			// 通过index控制每次将合并的数据填入nums中
			// 重填入的次数和合并及二叉树的高度相关
			index := j
			for _, v := range res {
				nums[index] = v
				index++
			}
			j = j + 2*i
		}
		i *= 2
	}
	return nums
}
```

- 快速排序(不稳定)

```go
func quickSort(nums []int, head, tail int) {
    if head >= tail {
        return 
    }
    l, r := head, tail
    mid := (r + l) / 2 
    pivot := nums[mid]
    for l <= r {
        for l <= r && nums[l] < pivot {
            l ++
        }
        for l <= r && nums[r] > pivot {
            r --
        }
        if l <= r {
            nums[l], nums[r] = nums[r], nums[l]
            l++
            r--
        }
    }
    quickSort(nums, head, r)
    quickSort(nums, l, tail)
}
```

- 堆排序(不稳定)

```go
func sortArray(nums []int) []int {
    // 堆排序-大根堆，升序排序，基于比较交换的不稳定算法，时间O(nlogn)，空间O(1)-迭代建堆
	// 遍历元素时间O(n)，堆化时间O(logn)，开始建堆次数多些，后面次数少 
	// 主要思路：
	// 1.建堆，从非叶子节点开始依次堆化，注意逆序，从下往上堆化
	// 建堆流程：父节点与子节点比较，子节点大则交换父子节点，父节点索引更新为子节点，循环操作
	// 2.尾部遍历操作，弹出元素，再次堆化
	// 弹出元素排序流程：从最后节点开始，交换头尾元素，由于弹出，end--，再次对剩余数组元素建堆，循环操作
	// 建堆函数，堆化
	var heapify func(nums []int, root, end int)
	heapify = func(nums []int, root, end int) {
		// 大顶堆堆化，堆顶值小一直下沉
		for {	
			// 左孩子节点索引
			child := root*2 + 1
			// 越界跳出
			if child > end {
				return
			}
			// 比较左右孩子，取大值，否则child不用++
			if child < end && nums[child] <= nums[child+1] {
				child++
			}
			// 如果父节点已经大于左右孩子大值，已堆化
			if nums[root] > nums[child] {
				return
			}
			// 孩子节点大值上冒
			nums[root], nums[child] = nums[child], nums[root]
			// 更新父节点到子节点，继续往下比较，不断下沉
			root = child
		}
	}
	end := len(nums)-1
	// 从最后一个非叶子节点开始堆化
	for i:=end/2;i>=0;i-- {
		heapify(nums, i, end)
	}
	// 依次弹出元素，然后再堆化，相当于依次把最大值放入尾部
	for i:=end;i>=0;i-- {
		nums[0], nums[i] = nums[i], nums[0]
		end--
		heapify(nums, 0, end)
	}
	return nums
}
```

- 桶排序

```go
func bin_sort(li []int, bin_num int) {
    min_num, max_num := li[0], li[0]
    for i := 0; i < len(li); i++ {
        if min_num > li[i] {
            min_num = li[i]
        }
        if max_num < li[i] {
            max_num = li[i]
        }
    }
    bin := make([][]int, bin_num)
    for j := 0; j < len(li); j++ {
        n := (li[j] - min_num) / ((max_num - min_num + 1) / bin_num)
        bin[n] = append(bin[n], li[j])
        k := len(bin[n]) - 2
        for k >= 0 && li[j] < bin[n][k] {
            bin[n][k+1] = bin[n][k]
            k--
        }
        bin[n][k+1] = li[j]
    }
    o := 0
    for p, q := range bin {
        for t := 0; t < len(q); t++ {
            li[o] = bin[p][t]
            o++
        }
    }
}
```

#### 基本排序算法（稳定）

- 算法稳定指的是排序后相同的值位置是否改变
- 堆排序、快速排序、希尔排序、选择排序不是稳定的排序算法；
- 基数排序、冒泡排序、插入排序、归并排序是稳定的排序算法。



#### 堆排序和快排对比

1. 10w 数据量两种排序速度基本相当，但是堆排序交换次数明显多于快速排序；10w+数据，随着数据量的增加快速排序效率要高的多，数据交换次数快速排序相比堆排序少的多。
2. 实际应用中，堆排序的时间复杂度要比快速排序稳定，快速排序的最差的时间复杂度是O（n*n）,平均时间复杂度是O(nlogn)。堆排序的时间复杂度稳定在O(nlogn)。但是从综合性能来看，快速排序性能更好。
3. 堆排序数据访问的方式比快速排序友好。对于快速排序来说，数据是跳着访问的。比如：堆排序中，最重要的一个操作就是数据的堆化。比如对堆顶节点进行堆化，会依次访问数组下标是1.2.4.8的元素，而不是像快速排序那样，局部顺讯访问，所以，这样快排对cpu缓存是不友好的。



# 设计模式

## 设计模式的分类

- 创建型模式：单例模式、抽象工厂模式、建造者模式、工厂模式、原型模式。
- 结构型模式：适配器模式、桥接模式、装饰模式、组合模式、外观模式、享元模式、代理模式。
- 行为型模式：模版方法模式、命令模式、迭代器模式、观察者模式、中介者模式、备忘录模式、解释器模式、状态模式、策略模式、职责链模式、访问者模式。



## 单例模式的使用场景

- 资源共享的情况下，避免由于资源操作时导致的性能或损耗等。如日志文件，应用配置。
- 控制资源的情况下，方便资源之间的互相通信。如线程池等。



## 设计原则七大原则

- 开放封闭原则:对扩展开放，对修改关闭。在程序需要进行拓展的时候，不能去修改原有的代码，实现
  一个热插拔的效果。
- 单一职责原则:一个类、接口或方法只负责一个职责，降低代码复杂度以及变更引起的风险。
- 依赖倒置原则:针对接口编程，依赖于抽象类或接口而不依赖于具体实现类。
- 接口隔离原则:将不同功能定义在不同接口中实现接口隔离。
- 里氏替换原则:任何基类可以出现的地方，子类一定可以出现。
- 迪米特原则:每个模块对其他模块都要尽可能少地了解和依赖，降低代码耦合度。
- 合成复用原则:尽量使用组合(has-a)/聚合(contains-a)而不是继承(is-a)达到软件复用的目的。



## **工厂模式**

### 简单工厂

​		简单工厂模式指由一个工厂对象来创建实例,适用于工厂类负责创建对象较少的情况。工厂方法模式指定义一个创建对象的接口，让接口的实现类决定创建哪种对象，让类的实例化推迟到子 类中进行。



### 抽象工厂模式

​		抽象工厂模式指提供一个创建一系列相关或相互依赖对象的接口，无需指定它们的具体类。



## **单例模式**

```go
// 饿汉式实现: 如果singleton创建初始化比较复杂耗时时，加载时间会延长。
type singleton struct{}
var ins *singleton = &singleton{}
func GetIns() *singleton{
    return ins
}

// 懒汉式实现: 非线程安全。当正在创建时，有线程来访问此时ins = nil就会再创建，单例类就会有多个实例了。
type singleton struct{}
var ins *singleton
func GetIns() *singleton{
    if ins == nil {
        ins = &singleton{}
    }
    return ins
}

// 懒汉加锁
type singleton struct{}
var ins *singleton
var mu sync.Mutex
func GetIns() *singleton{
    mu.Lock()
    defer mu.Unlock()

    if ins == nil {
        ins = &singleton{}
    }
    return ins
}
```



## **代理模式**

​		代理模式为其他对象提供一种代理以控制对这个对象的访问。优点是可以增强目标对象的功能，降低代码耦合度，扩展性好。缺点是在客户端和目标对象之间增加代理对象会导致请求处理速度变慢，增加系统复杂度。

- 静态代理:在程序运行前就已经存在代理类的字节码文件，代理类和委托类的关系在运行前就确定了。
- 动态代理:程序运行期间动态的生成，所以不存在代理类的字节码文件。代理类和委托类的关系是在程
  序运行时确定。



## **适配器模式**

​		适配器模式将一个接口转换成客户希望的另一个接口，使接口不兼容的那些类可以一起工作。(多态)



## **模板模式**

​	模板模式定义了一个操作中的算法的骨架，并将一些步骤延迟到子类，适用于抽取子类重复代码到公共父类。
​	可以封装固定不变的部分，扩展可变的部分。但每一个不同实现都需要一个子类维护，会增加类的数量。



## **装饰器模式**

​		装饰者模式可以动态地给对象添加一些额外的属性或行为，即需要修改原有的功能，但又不愿直接去修 改原有的代码时，设计一个Decorator套在原有代码外面; 如struct 直接引用其他的struct



### **观察者模式**

​		观察者模式表示的是一种对象与对象之间具有依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新。


