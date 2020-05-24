## Channel

### 坑：

- 已关闭的 channel，再次关闭会 panic

- 被关闭的 channel 不能再向其中发送内容，否则会 panic。

  注意，如果 close channel 时，有 sender goroutine 挂在 channel 的阻塞发送队列中，会导致 panic：

  ```go
  func main() {
      ch := make(chan int)
      go func() { ch <- 1 }() // panic: send on closed channel,改成 ch := make(chan int,1)就没有问题
      time.Sleep(time.Second)
      go func() { close(ch) }()
      time.Sleep(time.Second)
      x, ok := <-ch
      fmt.Println(x, ok)
  }
  ```

​       close 一个 channel 会唤醒所有等待在该 channel 上的 g，并使其进入      Grunnable 状态，这时这些 writer goroutine 会发现该 channel       已经是 closed 状态，就 panic了。

​       在不确定是否还有 goroutine 需要向 channel 发送数据时，请勿贸然关闭 channel。

- 关闭一个 nil channel 会直接 panic

- nil channel 发送数据会永远阻塞下去



### 数据结构：

```go
// channel 在 runtime 中的结构体
type hchan struct {
    // 队列中目前的元素计数
    qcount uint // total data in the queue
    // 环形队列的总大小，ch := make(chan int, 10) => 就是这里这个 10
    dataqsiz uint // size of the circular queue
    // void * 的内存 buffer 区域
    buf unsafe.Pointer // points to an array of dataqsiz elements
    // sizeof chan 中的数据
    elemsize uint16
    // 是否已被关闭
    closed uint32
    // runtime._type，代表 channel 中的元素类型的 runtime 结构体
    elemtype *_type // element type
    // 发送索引
    sendx uint // send index
    // 接收索引
    recvx uint // receive index
    // 接收 goroutine 对应的 sudog 队列
    recvq waitq // list of recv waiters
    // 发送 goroutine 对应的 sudog 队列
    sendq waitq // list of send waiters

    // lock protects all fields in hchan, as well as several
    // fields in sudogs blocked on this channel.
    //
    // Do not change another G's status while holding this lock
    // (in particular, do not ready a G), as this can deadlock
    // with stack shrinking.
    lock mutex
}
```

关于字段的含义都写在注释里了，再来重点说几个字段：

`buf` 指向底层循环数组，只有缓冲型的 channel 才有。

`sendx`，`recvx` 均指向底层循环数组，表示当前可以发送和接收的元素位置索引值（相对于底层数组）。

`sendq`，`recvq` 分别表示被阻塞的 goroutine，这些 goroutine 由于尝试读取 channel 或向 channel 发送数据而被阻塞。

`waitq` 是 `sudog` 的一个双向链表，而 `sudog` 实际上是对 goroutine 的一个封装：

```go
type waitq struct {
	first *sudog
	last  *sudog
}
```

`lock` 用来保证每个读 channel 或写 channel 的操作都是原子的。

例如，创建一个容量为 6 的，元素为 int 型的 channel 数据结构如下 ：

[![chan data structure](https://user-images.githubusercontent.com/7698088/61179068-806ee080-a62d-11e9-818c-16af42025b1b.png)](https://user-images.githubusercontent.com/7698088/61179068-806ee080-a62d-11e9-818c-16af42025b1b.png)

channel的4个特性的实现：

- channel的goroutine安全，是通过mutex实现的。
- channel的FIFO，是通过循环队列实现的。
- channel的通信：在goroutine间传递数据，是通过仅共享hchan+数据拷贝实现的。
- channel的阻塞是通过goroutine自己挂起，唤醒goroutine是通过对方goroutine唤醒实现的。

channel的其他实现：

- 发送goroutine是可以访问接收goroutine的内存空间的，接收goroutine也是可以直接访问发送goroutine的内存空间的，看`sendDirect`、`recvDirect`函数。
- 无缓冲的channel始终都是直接访问对方goroutine内存的方式，把手伸到别人的内存，把数据放到接收变量的内存，或者从发送goroutine的内存拷贝到自己内存。省掉了对方再加锁获取数据的过程。
- 接收goroutine读不到数据和发送goroutine无法写入数据时，是把自己挂起的，这就是channel的阻塞操作。阻塞的接收goroutine是由发送goroutine唤醒的，阻塞的发送goroutine是由接收goroutine唤醒的，看`gopark`、`goready`函数在`chan.go`中的调用。
- 接收goroutine当channel关闭时，读channel会得到0值，并不是channel保存了0值，而是它发现channel关闭了，把接收数据的变量的值设置为0值。
- channel的操作/调用，是通过reflect实现的，可以看reflect包的`makechan`, `chansend`, `chanrecv`函数。

参考资料：

- [码农桃花源](https://qcrao.com/2019/07/22/dive-into-go-channel/)

