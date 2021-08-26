# iOS-CarPlay
## CarPlay 是什么

CarPlay 是 Apple 发布的一个车载系统，可以配合 iPhone 使用。其前身是叫 iOS in the Car，2014 年更名为 CarPlay。

CarPlay 让你能够更智能、更安全地在车内使用 iPhone，它可以在驾驶时使用 iPhone 做你想做的事情，并将它们显示在汽车的内置显示屏上。你可以查询路线、拨打电话、发送和接收消息以及听音乐，同时专注于驾驶。

控制 CarPlay 主要有 3 种方式：Siri、触屏显示屏、物理按键。

## CarPlay 支持哪些 App 及功能

* 音频 App 可以提供音乐、新闻、播客等。
* 通信 App 可以发送和接收消息，同时可以配合 Siri 使用。
* 导航 App 可以提供详细的地图、目的地搜索、路线指导和用户通知。
* 汽车制造商的 App 可以提供特定于车辆的控制和显示，让司机在不离开 CarPlay 的情况下保持联系。
* iOS 14 新功能，支持 EV 充电、停车和快餐订购 App。此外，所有 CarPlay App 都可以利用 CarPlay 框架提供一致的设计，并针对在汽车中的使用进行了优化。

## WWDC

### WWDC16 - 开发 CarPlay 车载系统 - 第 1 部分

https://developer.apple.com/wwdc16/722

**概览：**CarPlay 车载让你能够更智能、更安全地在车内使用 iPhone。了解 CarPlay 车载的工作方式，以及如何设计你的车载信息娱乐系统来与 iPhone 密切协作。了解通过将 CarPlay 车载与车辆原生系统整合来打造出色用户体验的最佳做法。

该 Session 是面向汽车制造商的，主要内容是：

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

## 相关资料

### WWDC

* [WWDC16 - 开发 CarPlay 车载系统 - 第 1 部分](https://developer.apple.com/wwdc16/722)
* [WWDC16 - 开发 CarPlay 车载系统 - 第 2 部分](https://developer.apple.com/wwdc16/723)
* [WWDC17 - 开发无线 CarPlay 车载](https://developer.apple.com/wwdc17/717)
* [WWDC17 - 让您的 App 支持 CarPlay 车载](https://developer.apple.com/wwdc17/719)
* [WWDC18 - CarPlay 车载音频和导航 App](https://developer.apple.com/wwdc18/213)
* [WWDC19 - CarPlay 车载系统改进](https://developer.apple.com/wwdc18/213)
* [WWDC20 - 使用 CarPlay 车载系统为你的 App 提速](https://developer.apple.com/wwdc20/10635)
  * [内参](https://xiaozhuanlan.com/topic/7620814593)

### 文档

* [CarPlay - 介绍](https://www.apple.com.cn/ios/carplay/)
* [CarPlay - 文档首页](https://developer.apple.com/carplay/)
* [CarPlay - 设计指南](https://developer.apple.com/design/human-interface-guidelines/carplay/overview/introduction/)
* [CarPlay - 开发者文档](https://developer.apple.com/documentation/carplay?language=objc)
  * [申请 CarPlay 权利](https://developer.apple.com/documentation/carplay/requesting_the_carplay_entitlements?language=objc)
  * [使用 CarPlay Simulator 运行和调试 CarPlay App](https://developer.apple.com/documentation/carplay/using_the_carplay_simulator?language=objc) （需要先申请 CarPlay 权利，创建配置文件）
  * [在 CarPlay 中显示内容](https://developer.apple.com/documentation/carplay/displaying_content_in_carplay?language=objc)
  * [兼容 iOS13 及更早 iOS 系统](https://developer.apple.com/documentation/carplay/supporting_previous_versions_of_ios?language=objc)
* [CarPlay - App 编程指南](https://developer.apple.com/carplay/documentation/CarPlay-App-Programming-Guide.pdf)
* [CarPlay - 适用车型](#Bdvd9)



