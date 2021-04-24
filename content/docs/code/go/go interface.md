+++
title = "go interface"
description = "go 接口"

date = "2021-04-21"
categories = [
"Go"
]

menu = "main"

+++

接口类型是对其他类型行为的概括与抽象。通过使用接口，可以写出更加灵活和通用的函数，这些函数不用绑定在一个特定的类型实现上。

go 中的接口独特之处在于它是**隐式实现**。即对于具体类型，无须声明它实现了哪些接口，只要提供接口所需要的方法即可。这种设计让你无须改变已有类型的实现，就可以为这些类型创建新的接口，对于那些不能修改包的类型，这一点特别有用。

**具体类型与接口类型**

*具体类型*：包含内部数据以及其内部操作，通过其方法来提供额外的能力。是什么与能干什么

*接口类型*：一种**抽象类型**，仅包含方法。即能做什么。

{{< expand "标准库中的例子" >}}

`fmt.Pintf`和`fmt.Sprintf`前者把结果发送到标准输出（其实是一个文件），后者把结构以string类型返回。

格式化是两个函数中最复杂的部分，将格式化以接口的形式抽出。（其实两个函数都封装了第三个函数`fmt.Fprintf`）

```go
package fmt

func Fprintf(w io.Writer, format string, args ...interface{}) (int, error)

func Printf(format string,args ...interface{}) (int, error) {
    return Fprintf(os.Stdout, format, args...)
}

func Sprintf(format string, args ...interface{}) string {
    var buf bytes.Buffer
    Fprintf(&buf, format, args...)
    return buf.String()
}
```



{{< /expand >}}



#### 类型断言

**类型断言**作用于接口上，写法类似于`x.T()`，x 是一个接口类型的表达式，T 是一个类型（称为断言类型）。

类型断言检查作为操作数的动态类型是否满足指定的断言类型

两种可能：断言类型 T 是一个具体类型，类型断言则会检查 x 的动态类型是否就是 T，检查成功，类型断言的结果就是 x 的动态值，即 T 类型。  

类型断言就是用来从它的操作数中把具体类型的值提取出来的操作。检查失败，则操作崩溃

```go
var w io.Writer
w := os.Stdout
f := w.(*os.File)	// success， f == os.Stdout
c := w.(*bytes.Buffer)	// fail, 接口类型持有的是 *os.File，不是 *bytes.Buffer
```

其次，断言类型 T 是一个接口类型，类型断言则会检查 x 的动态类型是否满足 T。成功，动态值并没有提取出来，结构仍是一个接口值，接口值的类型和值部分也没有变更，只是结果的类型为接口类型 T 

即类型断言是一个接口值的表达式，从一个接口类型变为拥有另外一套方法的接口类型（通常方法数量是增多）但保留了接口值中的动态类型和动态值部分。

如下类型断代码中，w 和 rw 都持有 os.Stdout，于是所有对应的动态类型都是 *os.File，但 w 作为 io.Writer 仅暴露了文件的 Write 方法，而 rw 还暴露了它的 Read 方法

```go
var w io.Writer
w = os.Stdout
rw := w.(io.ReadWriter)	// success：*os.File 有 Read 和 Write 方法

w = new(ByteCounter)
rw = w.(io.ReadWriter)	// fail：ByteCounter 没有 Read 方法
```

无论哪种类型作为断言类型，如果操作数是一个空接口值，类型断言都失败。很少需要从一个接口类型向一个要求更宽松的类型做类型断言，该宽松类型的接口方法比原类型的少，而且是子集。因为除了在操作 nill 之外的情况下，在其他情况下这种操作与赋值一致

```go
w = rw	// io.ReaderWriter 可以赋值给 io.Writer
w = rw.(io.Writer)	// 仅当 rw == nil 时失败
```

我们经常无法确定一个接口值得动态类型，这时需要检测它是否是某一个特定类型。如果类型断言出现在需要两个结果得赋值表达式中，那么断言不会在失败时崩溃，而是会多返回一个布尔值来指示断言是否成功

```go
var w io.Writer = os.Stdout
f, ok := w.(*os.File)	// success ok, f == os.Stdout
b, ok := w.(*bytes.Buffer)	// fail：!ok, b == nil
```

惯例，第二个返回值赋给名为 ok 的变量，如果操作失败，ok 为 false，第一个返回值为断言类型的零值

ok 返回值通常马上用来决定下一步做什么

```go
if f, ok := w.(*os.File); ok {
    // ...use f...
}
```

当类型断言得操作数是一个变量时，有时会看见返回值得名字与操作数变量名一致，原有的值就被新的返回值掩盖了

```go
if w, ok := w.(*os.File); ok {
    // ...use w...
}
```
