## 主goroutine

* 设定栈空间
* 检查当前M是runtine.M0

* 创建一个特殊的defer,用于在主goroutine退出时做必要的善后处理

* 启用专用于在后台清扫内存垃圾的goroutine,并设置GC可用的标识

* 执行main的init

```go
runtime.GOMAXPROCS // 设置P的数量 1～257
runtime.Goexit // 执行defer 终止当前G，置于本地P的自由G列表
runtime.Gosched // 暂停
runtime.NumGoroutine // 非Gdead的用户G的数量 也就是可调度运行的G
runtime.LockOSThread｜runtime.LockOSThread // 会使当前G和M绑定在一起
runtime/debug.SetMaxStack
runtime/debug.SetMaxThreads
```

## channle

普通声明是双向的

```go
chan<- T // 单向接收
<-chan   // 单向发送
elem := <-strChan // 如果没有就会阻塞，关闭通道后返回类型0值
elem, ok := <-strChan
close(elem) // 关闭通道
len() cap()

```

注意：

1. 从一个未初始化的通道中接收数据会导致goroutine永久阻塞

2. 发送操作会使通道复制被发送的元素，若因通道的缓冲空间已满而无法立即复制，则阻塞进行发送操作的goroutine.

   复制的目的地址有两种：

   * 当通道已空且有接收方在等待元素值时，会是最早进入等待的接受者的内存地址
   * 否则会是通道持有的缓冲中的内存地址

3. 接收操作会使通道给出一个已发给它的元素值的副本，若因通道的缓冲空间已空无法立即给出，则阻塞接收操作的goroutine

4. 接收操作和发送操作都是原子的

5. 传输值都是值的副本，发送时和接收时会分别复制一次。修改引用类型会导致两方持有的值都发生修改

通道的关闭：

1. 向一个已关闭的chan发送值会导致panic，所以chan一般由发送者关闭（或者利用for和select来安全的退出）
2. 通道的关闭堆接收操作没有影响，通过第二个参数可以获取通道的状态
3. 只能关闭一次，且关闭时通道值不能为nil









