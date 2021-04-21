+++
title = "go summary"
description = "go 的自我总结"

date = "2021-04-19"
categories = [
"Go"
]

menu = "main"

+++

有关 go 的一些自我总结

<!--more-->

### 1 函数

#### 1.1 内置函数 make

{{< columns >}}

创建 map

```go
// 创建一个从 string 到 int 的 map
ages := make(map[string]int)	
```

<--->

创建通道

```go
// ch 的类型是 'chan int'
ch := make(chan int)
```

{{< /columns >}}

创建 slice（指定元素类型，长度和容量）

```go
make([]T, len)	// 省略容量参数，slice 的长度和容量相等
make([]T, len, cap)	// 等同于 make([]T, cap)[:len]
```

{{< details "代码解释" >}}
其实，make 创建了一个无名数组并返回了它的一个 slice；这个数组仅可以通过这个 slice 来访问。  
第一行代码返回的 slice 引用了整个数组。  
第二行中，slice 只引用了数组的前 len 个元素，但是它的容量是数组的长度，这为未来的 slice 元素留出空间。
{{< /details >}}

### 2. 比较
