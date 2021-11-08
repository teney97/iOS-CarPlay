### 前言

### CarPlay 是什么

CarPlay 是 Apple 发布的一个车载系统，可以配合 iPhone 使用（iPad 不支持）。其前身是叫 iOS in the Car，2014 年更名为 CarPlay。

简单地说，如果你的汽车支持 CarPlay，那么当你的汽车（通过数据线、蓝牙、Wi-Fi）连接 iPhone 时，汽车显示屏会自动切换到 CarPlay，iPhone 上所有支持 CarPlay 的 App 会自动显示在 CarPlay 中（你可以在 iPhone 设置（通用 - CarPlay 车载）中屏蔽指定 App）。CarPlay 上的 App 的用户界面是固定的，由 Apple 设计，内容由 App 自己提供。

控制 CarPlay 主要有 3 种方式：Siri、触屏显示屏、物理按键。

### CarPlay 支持哪些 App 及功能

* 音频 App 可以提供音乐、新闻、播客等。
* 通信 App （Messaging、VoIP calling）可以发送和接收消息，同时可以配合 Siri 使用。（2017 年引入）
* 导航 App 可以提供详细的地图、目的地搜索、路线指导和用户通知。
* 汽车制造商的 App 可以提供特定于车辆的控制和显示，让司机在不离开 CarPlay 的情况下保持联系。
* iOS 14 新功能，支持 EV 充电、停车和快餐订购 App。此外，所有 CarPlay App 都可以利用 CarPlay 框架提供一致的设计，并针对在汽车中的使用进行了优化。

### CarPlay App 如何开发

这里先说个大概，站在开发者的角度来理解 CarPlay App。是让你的 App 支持 CarPlay，而不是另外开发一个 CarPlay App，只是为了叫的更顺口些，读者们注意一下。

1. 首先需要确定你的 App 是否适用于 CarPlay，然后去开发者网站申请对应 App 类型的 CarPlay 权限，并对工程进行配置。只有这样你的工程才能使用 CarPlay Simulator。因此如果你计划要开发 CarPlay App 的话，最好提前去申请权限，因为 Apple 审核还要时间。在此期间可以看看相关开发文档，等权限申请下来就可以使用 CarPlay Simulator 调试开发啦。参考文档：[申请 CarPlay 权限](https://developer.apple.com/documentation/carplay/requesting_the_carplay_entitlements?language=objc)。
2. 从 iOS 14 开始，你要使用 CarPlay Framework 来开发 CarPlay App；而 iOS 13 及更早版本使用的是 MediaPlayer Framework，它没有向前兼容。因此，如果你的 App 需要在 iOS 14 及更高版本以及 iOS 13 级更早版本中都支持 CarPlay 的话，就要维护两套代码，开发工作量可能接近 double。笔者仅支持了 iOS 14 及更高版本，因此在下篇文章中会详细讲解开发细节。如果你想支持低版本的话也可以看看笔者对「WWDC17 - 让您的 App 支持 CarPlay 车载」和「WWDC18 - CarPlay 车载音频和导航 App」做的笔记。
3. 在 iOS 14 及更高版本中使用 CarPlay Framework 来开发 CarPlay App 必须使用 UIScene（UIScene 是 Apple 于 iOS 13 引入的，用于构建多窗口应用），因此你的工程必须从传统的 UIWindow 和 UIApplicationDelegate API 向 UIScene 过渡。如果你的工程已经兼容了 UIScene，那么就可以省去这步骤的工作；如果还未兼容的话，也可以看看我在下篇文章中讲到的一些注意点。
4. UI 固定
5. 只是展示 UI 和页面跳转，nowPlayingInfo
6. 学习模版（UI）的使用
7. 测试

## WWDC

### WWDC16 - 开发 CarPlay 车载系统 - 第 1 部分

* **地址**：https://developer.apple.com/wwdc16/722
* **时长**：30 分钟
* **概览**：CarPlay 车载让你能够更智能、更安全地在车内使用 iPhone。了解 CarPlay 车载的工作方式，以及如何设计你的车载信息娱乐系统来与 iPhone 密切协作。了解通过将 CarPlay 车载与车辆原生系统整合来打造出色用户体验的最佳做法。
* **总结**：[WWDC16 - 722](https://github.com/teney97/iOS-CarPlay/blob/main/Content/WWDC16%20-%20%E5%BC%80%E5%8F%91%20CarPlay%20%E8%BD%A6%E8%BD%BD%E7%B3%BB%E7%BB%9F%20-%20%E7%AC%AC%201%20%E9%83%A8%E5%88%86.md)


### WWDC16 - 开发 CarPlay 车载系统 - 第 2 部分

* **地址**：https://developer.apple.com/wwdc16/723
* **时长**：26 分钟
* **概览**：了解 CarPlay 车载如何与您的车辆信息娱乐系统整合。了解 CarPlay 车载如何与您的车载资源协作，如显示屏、扬声器、麦克风、用户输入、方向盘控制键、仪表盘和传感器等。
* **总结**：[WWDC16 - 723](https://github.com/teney97/iOS-CarPlay/blob/main/Content/WWDC16%20-%20%E5%BC%80%E5%8F%91%20CarPlay%20%E8%BD%A6%E8%BD%BD%E7%B3%BB%E7%BB%9F%20-%20%E7%AC%AC%202%20%E9%83%A8%E5%88%86.md)


### WWDC17 - 开发无线 CarPlay 车载系统

* **地址**：https://developer.apple.com/wwdc17/717
* **时长**：35 分钟
* **概览**：无论去向哪里，无线 CarPlay 车载都是旅程的绝佳搭档。无需将 iPhone 从包里或口袋中取出，直接开门上车，轻松开始享受 CarPlay 车载体验。学习如何设计你的 CarPlay 车载系统来以无线方式连接至 iPhone。了解相关的硬件要求、提供出色用户体验的最佳做法，以及如何优化配对和重新连接过程。
* **总结**：[WWDC17 - 717](https://github.com/teney97/iOS-CarPlay/blob/main/Content/WWDC17%20-%20%E5%BC%80%E5%8F%91%E6%97%A0%E7%BA%BF%20CarPlay%20%E8%BD%A6%E8%BD%BD%E7%B3%BB%E7%BB%9F.md)


### WWDC17 - 让您的 App 支持 CarPlay 车载

* **地址**：https://developer.apple.com/wwdc17/719
* **时长**：28 分钟
* **概览**：了解如何让你的音频、信息、VolP 通话或汽车制造商 App 支持 CarPlay 车载。音频、信息、VolP 通话 App 采用一致的设计，并且为在车内使用进行过优化。汽车制造商 App 提供车辆相关的控制和显示功能，让驾驶员无需离开 CarPlay 车载就能保持互联。探索最佳做法，并了解适用于 CarPlay 车载 App 的工具和框架。
* **总结**：[WWDC17 - 719](https://github.com/teney97/iOS-CarPlay/blob/main/Content/WWDC17%20-%20%E8%AE%A9%E6%82%A8%E7%9A%84%20App%20%E6%94%AF%E6%8C%81%20CarPlay%20%E8%BD%A6%E8%BD%BD.md)


### WWDC18 - CarPlay 车载音频和导航 App

* **地址**：https://developer.apple.com/wwdc18/213
* **时长**：38 分钟
* **概览**：了解如何更新音频或导航 App 来支持 CarPlay 车载。CarPlay 车载中的 App 针对车用进行了优化，能够自动适应可用的汽车屏幕和输入控制。音频 App 能够输出音乐、新闻、播客等。通过新的 CarPlay 车载框架，导航 App 可以提供详细地图、目的地搜索、逐向导航和用户通知。
* **总结**：[WWDC18 - 213](https://github.com/teney97/iOS-CarPlay/blob/main/Content/WWDC18%20-%20CarPlay%20%E8%BD%A6%E8%BD%BD%E9%9F%B3%E9%A2%91%E5%92%8C%E5%AF%BC%E8%88%AA%20App.md)


### WWDC19 - CarPlay 车载系统改进

* **地址**：https://developer.apple.com/wwdc19/252
* **时长**：16 分钟
* **概览**：CarPlay 车载让你能够在车内更加智能、安全地使用 iPhone。了解如何更新你的车载系统，以利用 iOS 13 中的新功能。添加对动态变换屏幕尺寸、仪表盘等辅助屏幕，甚至是不规则显示屏的支持。了解如何支持 “嘿 Siri” 来进行免提语音激活。
* **总结**：[WWDC19 - 252](https://github.com/teney97/iOS-CarPlay/blob/main/Content/WWDC19%20-%20CarPlay%20%E8%BD%A6%E8%BD%BD%E7%B3%BB%E7%BB%9F%E6%94%B9%E8%BF%9B.md)


### WWDC20 - 使用 CarPlay 车载系统为你的 App 提速

* **地址**：https://developer.apple.com/wwdc20/10635
* **时长**：26 分钟
* **概览**：CarPlay 车载可为用户提供更智能、更安全的 iPhone 车内使用方式。我们将向你展示如何为车辆屏幕打造优质 App，并教你开发电动机车充电、泊车、快速订购食物外卖等类型的 CarPlay 车载 App。此外，我们还会使用现存的音频与通讯 App 作为范例，详细解释如何利用 CarPlay 车载框架的种种改进，制作出更加灵活多变的用户界面。
* **内参**：https://xiaozhuanlan.com/topic/7620814593
* **总结**：[WWDC20 - 10635](https://github.com/teney97/iOS-CarPlay/blob/main/Content/WWDC20%20-%20使用%20CarPlay%20车载系统为你的%20App%20提速.md)


## 相关资料

### WWDC

* [WWDC16 - 开发 CarPlay 车载系统 - 第 1 部分](https://developer.apple.com/wwdc16/722)
* [WWDC16 - 开发 CarPlay 车载系统 - 第 2 部分](https://developer.apple.com/wwdc16/723)
* [WWDC17 - 开发无线 CarPlay 车载系统](https://developer.apple.com/wwdc17/717)
* [WWDC17 - 让您的 App 支持 CarPlay 车载](https://developer.apple.com/wwdc17/719)
* [WWDC18 - CarPlay 车载音频和导航 App](https://developer.apple.com/wwdc18/213)
* [WWDC19 - CarPlay 车载系统改进](https://developer.apple.com/wwdc19/252)
* [WWDC20 - 使用 CarPlay 车载系统为你的 App 提速](https://developer.apple.com/wwdc20/10635)
  * [内参](https://xiaozhuanlan.com/topic/7620814593)

### 文档

* [CarPlay - 介绍](https://www.apple.com.cn/ios/carplay/)
* [CarPlay - 文档首页](https://developer.apple.com/carplay/)
* [CarPlay - 设计指南](https://developer.apple.com/design/human-interface-guidelines/carplay/overview/introduction/)
* [CarPlay - 开发者文档](https://developer.apple.com/documentation/carplay?language=objc)
  * [申请 CarPlay 权限](https://developer.apple.com/documentation/carplay/requesting_the_carplay_entitlements?language=objc)（如果申请了 CarPlay 权限后，启动 CarPlay Simulator 还是没有显示你的 app，看看是不是漏了 “添加权限文件” 步骤还是漏了哪一步）
  * [使用 CarPlay Simulator 运行和调试 CarPlay App](https://developer.apple.com/documentation/carplay/using_the_carplay_simulator?language=objc) （需要先申请 CarPlay 权限，创建配置文件后才能使用 CarPlay Simulator，而且审核得没那么快。如果你使用 M1 Mac，那可能用不了 CarPlay Simulator）
  * [在 CarPlay 中显示内容](https://developer.apple.com/documentation/carplay/displaying_content_in_carplay?language=objc)
  * [兼容 iOS13 及更早 iOS 系统](https://developer.apple.com/documentation/carplay/supporting_previous_versions_of_ios?language=objc)
* [CarPlay - App 编程指南](https://developer.apple.com/carplay/documentation/CarPlay-App-Programming-Guide.pdf)
  * 搜狗机翻版：https://github.com/teney97/iOS-CarPlay/tree/main/Content
* [CarPlay - 适用车型](https://www.apple.com.cn/ios/carplay/available-models/)
* [CarPlay Music App 示例](