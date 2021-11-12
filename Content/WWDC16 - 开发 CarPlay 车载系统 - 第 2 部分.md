## WWDC16 - 开发 CarPlay 车载系统 - 第 2 部分

**地址**：https://developer.apple.com/wwdc16/723

**时长**：26 分钟

**概览**：了解 CarPlay 车载如何与你的车辆信息娱乐系统整合。了解 CarPlay 车载如何与你的车载资源协作，如显示屏、扬声器、麦克风、用户输入、方向盘控制键、仪表盘和传感器等。

**总结**：

该 Session 讲了第 1 部分的内容的更多细节：

1、系统概览

* CarPlay sub-system
* Native sub-system
* Hardware and system resources

![image.png](https://cdn.nlark.com/yuque/0/2021/png/12376889/1630054163254-df96d547-e501-43d9-a75e-522ccbe7a663.png?x-oss-process=image%2Fresize%2Cw_750%2Climit_0)

2、音量管理

* 在 CarPlay 中，每一个功能的音量控制都是独立管理的，比如音乐播放、导航、通话、Siri 等等

3、资源管理

* 只访问汽车的两个资源

  * mainScreen：访问汽车显示器
  * mainAudio：访问扬声器和麦克风
  * Alternate Audio：没有被管理，基本上都是可用的，所以不需要占用或借用它

![image.png](https://cdn.nlark.com/yuque/0/2021/png/12376889/1630054215465-84a0ecff-3d7e-4582-9cdd-0faf1687e436.png?x-oss-process=image%2Fresize%2Cw_750%2Climit_0)

  * 两种访问方式：

    * 占有：长时间占用，比如切换到 CarPlay 界面，这时候 CarPlay 就占有了整个显示器
    * 借用：短时间借用，比如当前显示器切换到原生界面，这时候用户在 CarPlay 里接到电话，这时候显示器就会暂时切换到 CarPlay 界面，通话结束后自动回到原生界面
  * 需要一个资源管理器

    * 知道系统当前状态
    * 根据规则决定哪个系统获得资源
    * 根据当前状态和一系列规则，把资源分配给其中一方

4、App 状态管理

* 如果 CarPlay 和原生都想访问资源，这时候就需要定义 App 状态管理规则

总之，CarPlay 和汽车的原生系统依赖相同的资源，被设计成和原生用户体验共存。为了优秀的 CarPlay 体验，需要遵守 CarPlay 设计建议，考虑每一种使用情况的资源处理。可以通过 MFi Program 获取 CarPlay 详述。
