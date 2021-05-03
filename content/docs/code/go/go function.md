+++
title = "go function"
description = "go 方法"

date = "2021-05-02"
categories = [
"Go"
]

menu = "main"

+++

### 方法声明

与声明普通函数类似，只是在函数名字前面多一个参数，参数把这个方法绑定到这个参数对应的类型上

附加的参数称为方法的*接收者*

```go
package geometry

import "math"

type Point struct{ X, Y float64 }

// 普通函数
func Distance(p, q Point) float64 {
    return math.Hypot(q.X-p.X, q.Y-p,Y)
}

// Point 类型方法
func (p Point) Distance(q point) float64 {
    return math.Hypot(q.X-p.X, q.Y-p,Y)
}
```

go 中接收者不使用特殊名（this 或 self）；而是我们自己选择接收者名字。通常取类型名称的首字符

```go
p := Point{1, 2}    
q := Point{4, 6}
fmt.Println(Distance(p, q))
fmt.Println(p.Distance(q))
```
上述两个 Distance 函数声明没有冲突，第一个是包级别的函数（称为 geometry.Distance）。第二个声明一个类型 Point 的方法（Point.Distance）

表达式 p.Distance 称作选择子（selector）因为它为接收者 p 选择合适的 Distance 方法。选择子也作用于选择结构类型中的字段值，由于方法与字段来自同一个命名空间，同名的字段与方法，编译器会报错。

go 中的方法可以绑定到任何类型上，同一个包下的任何类型都可以声明方法，只要它的类型既不是指针类型也不是接口类型

类型拥有的方法名必须是唯一的，但不同类型可以使用相同的方法名

方法可以比函数更简单，在包的外部进行调用的时候，方法能狗使用更加简短的名字且省略包的名字

### 指针接收者的方法

主调函数会复制每一个实参变量，如果函数需要更新一个变量，或者实参太大，为了避免复制整个实参，可以使用指针来传递变量的地址。这同样适用于更新接收者

```go
func (p *Point) ScaleBy(factor float64) {
    p.X *= factor
    p.Y *= factor
}
```

方法的名字为 (*Point).ScaleBy。

如果 Point 的任何一个方法都使用指针接收者，那么所有的 Point 方法都应该使用指针接收者

命名类型（Point）与指向它的指针（*Point）是唯一可以出现在接收者处声明的类型。而且。为了防止混淆，不允许本身是指针的类型进行方法声明

```go
type P *int

func (P) f() { /*...*/ }    // 编译错误，非法的接收者类型
```

通过提供 *Point 能够调用 (*Point).ScaleBy 方法。以下均输出 "{2, 4}"

{{< columns >}}
```go
r := &Point{1, 2}
r.ScaleBy(2)
fmt.Println(*r)
```
<--->

```go
p := Point{1, 2}
pptr := &p
pptr.ScaleBy(2)
fmt.Println(p)
```
<--->

```go
p := Point{1, 2}
(&p).ScaleBy(2)
fmt.Println(p)
```
{{< /columns >}}

如果接收者 p 是 Point 类型的变量，但方法要求一个 *Point 接收者，可以使用简写 `p.ScaleBy(2)`

实际上编译器会对变量进行 &p 的隐式转换。只有变量才允许这么做，包括结构体字段。不能对一个不能取地址的 Point 接收者参数调用 *Point 方法，因为无法获取临时变量的地址。

```go
Point{1, 2}.ScaleBy(2)  // 编译错误，不能获取 Point 类型字面量的地址
```

但是如果实参接收者是 *Point 类型，以 Point.Distance 的方式调用 Point 类型的方法是合法的，因为有办法从地址中获取 Point 的值；只要解析引用指向接收值的指针值即可。编译器会自动插入一个隐式的 * 操作符。下面两个函数的调用效果是一样的

```go
pptr.Distance(q)
(*pptr).Distance(q)
```

