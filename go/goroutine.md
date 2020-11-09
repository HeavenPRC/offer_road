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

