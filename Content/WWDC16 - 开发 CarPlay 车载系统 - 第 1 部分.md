## WWDC16 - 开发 CarPlay 车载系统 - 第 1 部分

**地址**：https://developer.apple.com/wwdc16/722

**概览**：CarPlay 车载让你能够更智能、更安全地在车内使用 iPhone。了解 CarPlay 车载的工作方式，以及如何设计你的车载信息娱乐系统来与 iPhone 密切协作。了解通过将 CarPlay 车载与车辆原生系统整合来打造出色用户体验的最佳做法。

**总结**：

该 Session 的主讲师为一个 Apple 汽车体验工程师和一个 Apple 设计工程师，是面向汽车制造商的，主要内容是：

* 介绍了 CarPlay 以及它的工作方式
* 如何连接：分为有线和无线（Wi-Fi、蓝牙）
* 对汽车的要求：触摸显示屏、物理按键（包括 Siri、选择、后退、旋钮按键等等）、传感器
* 对车载系统的设计指导，如何与 CarPlay、iPhone 进行交互
  * 在汽车连接手机时，汽车的显示屏会自动切换成显示 CarPlay。用户可以在 CarPlay 界面和原生界面中切换。如果重连 iPhone，应该根据用户上一次选择的界面，来决定是否要自动切换显示 CarPlay。
  * 当汽车连接 iPhone 时，显示器原生界面上应该显示或启用 CarPlay 按钮，且在断开时隐藏或禁用
  * 若断开连接时 CarPlay 在播放音频，那么重新连接时 CarPlay 应该自动继续播放音频，即使用户断开连接时显示的是原生界面。
  * ...
* CarPlay 中可以运行哪些应用
  * 当汽车连接 iPhone 并启动 CarPlay 时，iPhone 上所有支持 CarPlay 的应用会自动显示在 CarPlay 中
  * CarPlay App 的用户界面是固定的，由 Apple 设计，内容由 App 自己提供
  * 汽车制造商如何开发专门针对特定车辆的应用
  * CarPlay 在 iOS 10 中支持了消息 App，可以收发消息、与 Siri 交互
  * ...