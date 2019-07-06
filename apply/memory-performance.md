memory-performance页面分析
=========
> 背景： 打开chrome任务管理器发现页面的内存增长速度很快， 当打开多个tab时， 页面很快崩溃

## 打开chrome任务管理， 监测页面的内存变化
- chrome任务管理截图
    ![](/asset/chrome-task.jpg)
- 内存占用内存
- Javascript使用内存

## performance分析一个周期240s的内存、事件、dom/nodes使用变化
  - 如图：
    ![](/asset/240s-performance.jpg)
  - 通过这个模型去发现nodes、JsHeap、Listener、在一个周期的变化情况。有无明显上升、与回落。

## 使用memory分析内存模型
  - 发现分离的dom, 拍快照对比是否有增长
    - 如图：
      ![](/asset/detached-dom.jpg)
    - 快照对比功能发现新增的dom节点

  - 重点关注(closure), 发现过多的内存占用 
    - 通过Comparision对比前后数量变化，以及是否增加占用的内存量
      - #New列代表新增、
      - #Deleted列代表消费掉（比如setTimeout执行后），或清除(removeEventListener)
      - #Delta:  剩余的 
      - #New = #Deleted + #Delta
      - Alloc.Size、Free.Size、Size Delta代表大小单位bit, 与上述三列对应
    - 通过(closure) 发现Vue大量的数据劫持
      - 在vue中不需要做数据响应的对象不要在data声明， 否则vue会对此做过多数据劫持绑定， 影响性能。比如：常用的d3对象，map对象,echart实例对象等.....
    - 结论： 这里可以发现过多的函数引用：比如setTimeout/Vue中的数据劫持reactiveGetter\reactiveSetter\包括listener事件注册函数。
  
  - 通过Comparision揪出地图Bc对象在源源不断的增加
    - 遇到的问题
      - Bc对象明明在代码中清掉了为什么数量还在不断增加
    - 为了解决问题通过memory->Summary.Object ...Snapshot 1 and Snapshot2 揪出增长Bc
    - 查找其引用链发现：
      - window.$BAIDU$._instances.xxxxx中
      - Ka(map对象).Gi.onmousemove
    - 结论： 
      - 代码中虽然清掉了地图Bc对象， 但是每隔段时间还会创建， 致使Bc数量不断增加（因库把引用放在了window引用树上， 与事件函数中）
      - 第三方库使用理解出现差异， 所以Bc对象创建后可以复用即可， 有新的数据设置不同数据即可。

## 总结
  - closure、对象是造成内存不断增加主要载体， 因为JsHeap就是一个内存图， 每一个对象与closure都会是一个节点， 其节点数量不断增加，jsHeap堆内存图就会不断扩张， 也会使浏览器会为维护张图开辟更大内存空间去维护这张图， 这就是所说的任务管理器中：（内存占用内存、Javascript使用内存)。 
  - 垃圾回机制， 从windows根节点开始，逐级扫描，发现不在其因引用中的节点， 进行垃圾回收处理。