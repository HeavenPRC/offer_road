GO的GC是基于并发的标记-清扫

// 源码位置：runtime.gcstart

分三种模式:

gcBackgroundMode: 并发的执行标记清扫

* 调度器驱使的GC
  * 在有大的内存增量时才会执行，默认值时翻倍（可以修改SetGCPercent或者修改环境变量GOGC）
* 系统监测任务中的强制GC(在循环中判断间隔时间)

gcForceMode: 串行的执行标记（执行时停止调度），并发的执行清扫（DEBUG）

gcForceBlockMode:串行地执行标记清扫（DEBUG,关闭GC）





