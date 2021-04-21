+++
title = "Go Composite Data"
description = "Mysql 调优"

date = "2021-04-12"
categories = [
"Go"
]

menu = "main"

+++

## 复合数据类型

复合数据类型是由基本数据类型以各种方式组合而构成的

<!--more-->

### 1.1 数组



### 1.2 slice

slice 表示一个拥有相同类型元素的可变长度的序列。slice 通常写成 **[]T**（例如：[]string、[]int）  
其中元素的类型都是 T；它看上去像没有长度的数组类型。

数组和 slice 紧密相关。slice 是一种轻量级的数据结构，可以用来访问数组的部分或全部的元素  
这个数组称为 slice 的**底层数组**。

slice 有三个属性

- 指针：指向数组的第一个可以从 slice 中访问的元素
- 长度：slice 中的元素个数
- 容量：slice 起始元素到**底层数组的最后一个元素间元素的个数**  
(Go 内置函数`len`和 `cap` 用来返回 slice 的长度和容量）

一个底层数组可以对应多个 slice，这些 slice 可以引用数组的任何位置，彼此之间的元素可以重叠

如果 slice 的引用超过了被引用对象的容量，即`cap(s)`，那么会导致程序宕机  
但是如果 slice 的引用超出了被引用对象的长度，即`len(s)`，那么最终 slice 会比原 slice 长

### 1.3 map

Go 语言中，map 是散列表的引用，map 的类型是 **map[k]V**，

键的类型 K，必须是可以通过操作符 == 来进行比较的数据类型，所以 map 可以检测某一个键是否已经存在。虽然浮点型可以进行比较，但比较浮点型的相等性不是一个好主意。尤其是在 NaN 可以是浮点型值的时候

#### 1.3.1 创建

内置函数`make`可以用来创建一个 map：

```go
// 创建一个从 string 到 int 的 map
ages := make(map[string]int)	
```
{{< columns >}}

使用 map 的*字面量* 创建带初始化元素的字典：

```go
ages := map[string]int{
    "alice": 31,
    "charlie": 34,
}
```
<--->

等价于

```go
ages := make(map[strint]int)
map["alice"] = 31
map["charlie"] = 34
```
{{< /columns >}}
因此，新的空 map 的另一种表达式是：`map[string]int{}`

map 类型零值是 `nill`，即没有引用任何散列表

```go
var ages map[string]int
fmt.Println(ages == nill)	// true
fmt.Println(len(ages) == 0)// true
```

向零值 map 设置元素会导致错误（设置元素之前，必须初始化 map）

```go
ages["carol"] = 21	// 宕机，为零值 map 中的项赋值
```
#### 1.3.2 查找

map 使用给定的键来查找元素，如果对应的元素不存在，就返回值类型的零值

```go
ages["alice"] = 32
fmt.Println(ages["alice"])	// 32
fmt.Println(ages["bob"])	// 0
```

如果想查找元素是否在 map 中，可以采取如下操作

```go
age, ok := ages["bob"]
if !ok {/* "bob" 不是字典中的键，age == 0 */}
```

两个可以合并为一个语句

```go
if age, 0k := ages["bob"]; !ok {/* "bob" 不是字典中的键，age == 0 */}
```

通过下标访问 map 中的元素输出两个值，第二个值是一个布尔值，用来报告元素是否存在，布尔变量一般叫做 ok，尤其在 if 判断中

#### 1.3.3 移除

可以使用内置函数`delete`从字典中根据键移除一个元素：

```go
delete(ages, "alice")	// 移除元素 age["alice"]
```

即使键不在 map 中，移除元素也是安全的

快捷赋值方式（x+=y 和 x++）对 map 的元素同样适用

但是 map 元素不是一个变量，不可以获取它的地址

```go
_ = &ages["bob"]	// 编译报错，无法获取 map 元素的地址
```

map 中元素的迭代顺序是不固定的，不同的实现方法会使用不同的散列算法，得到不同的元素顺序，实践中，认为这种顺序是随机的。从一个元素开始到后一个元素，依次执行。可以使得程序在不同散列算法的实现下变得健壮。

如果需要按照某种顺序来遍历 map 中的元素，**必须显式地来给键排序**

```go
import sort

var names []string
for name := range ages {
    names = append(names, name)
}
sort.Strings(names)
for _, name := range names {
    fmt.Printf("%s\t%d\n", name, ages[name])
}
```

因为一开始就知道 slice names 的长度，所以直接指定一个 slice 的长度会更加高效

```go
names := make([]string, 0, len(ages))	// 初始化元素为空但容量足够容纳 ages map 中所有键的 slice
```



