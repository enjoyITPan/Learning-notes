## Golang 自定义error避坑实践

### 原生error

​        熟悉go开发的朋友都知道，在go的设计中是推荐显示去处理error的。在使用时，通过建议函数返回一个error值，通过把返回的 error 变量与 nil 的比较，来判定操作是否成功。

go中的error定义如下：

```golang
type error interface {
	Error() string
}
```

原始的error被定义为interface。

可以看出原生的error非常简单，只有msg，但是一般业务开发上，我们还需要code信息。根据go 的接口设计，只要我们实现了Error() string 方法就可以实现自定义error。

### 自定义error

```golang
package errors

import "fmt"

type myErr struct {
   code int
   msg  string
}

func (e myErr) Error() string {
   return fmt.Sprintf("code:%d,msg:%v", e.code, e.msg)
}

func New(code int, msg string) error {
   return myErr{
      code: code,
      msg:  msg,
   }
}

func GetCode(err error) int {
   if e, ok := err.(myErr); ok {
      return e.code
   }
   return -1
}

func GetMsg(err error) string {
   if e, ok := err.(myErr); ok {
      return e.msg
   }
   return ""
}
```

通过自定义一个struct，并且实现Error() string 方法，我们自定义了一个带code的error。

这里需要注意，我们将myErr声明为小写开头，这样可以避免在在包外被直接声明，只允许通过函数New(code int, msg string)得到error。

如果可以被在包外直接声明，会有什么问题呢？这就是本次要介绍的一个坑了。

请看下面这个例子：

```golang
package main

import "fmt"

func main() {
   err := HelloWorld()
   fmt.Println(err == nil) //输出为false
}

func HelloWorld() error {
   var err *MyErr
   //do something
   return err
}

type MyErr struct {
   code int
   msg  string
}

func (e *MyErr) Error() string {
   return fmt.Sprintf("code:%d,msg:%v", e.code, e.msg)
}
```

我们会惊奇的发现err == nil 输出为false。

这一切是由于go的interface设计导致的，go 将interface 设计为了两部分：

- type
- value

其中，**value** 由一个任意的具体值表示，称作 interface 的 dynamic value ；而 **type** 则对应该 value 的类型（即 dynamic type）；例如对于对于 var a int = 3来说，把a 赋值给interface时， interface是使用(int, 3)进行存储的。

当想判断 interface 的值为 `nil`时 ，则必须是其内部 value 和 type 均未设置的情况，即 `(nil, nil)` ；

那么回到上面的例子，当把 var err `*MyErr` 赋值给interface时，interface的存储数据为(`*MyErr`,nil)。这个时候进行nil判断的结果肯定是false。

所以当自定义error时，我们要尽可能的避免error可以被直接声明。通过提供函数的形式去生成error。

一来可以避免上述的问题，二来可以屏蔽实现细节，可以更好的扩展。

