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

## 使用memory分析内存模型
  - 发现分离的dom
    - 如图：
      ![](/asset/detached-dom.jpg)


