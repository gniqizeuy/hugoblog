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