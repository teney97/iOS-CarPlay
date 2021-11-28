## 系列文章

* [iOS CarPlay｜准备](https://github.com/teney97/iOS-CarPlay/blob/main/iOS%20CarPlay%EF%BD%9C%E5%87%86%E5%A4%87.md)
* [iOS CarPlay｜兼容 UIScene](https://github.com/teney97/iOS-CarPlay/blob/main/iOS%20CarPlay｜兼容%20UIScene)
* [iOS CarPlay｜让你的音频 App 支持 CarPlay](https://github.com/teney97/iOS-CarPlay/blob/main/iOS%20CarPlay%EF%BD%9C%E8%AE%A9%E4%BD%A0%E7%9A%84%E9%9F%B3%E9%A2%91%20App%20%E6%94%AF%E6%8C%81%20CarPlay.md)
* [iOS CarPlay｜常见问题解答](https://github.com/teney97/iOS-CarPlay/blob/main/iOS%20CarPlay%EF%BD%9C%E8%AE%A9%E4%BD%A0%E7%9A%84%E9%9F%B3%E9%A2%91%20App%20%E6%94%AF%E6%8C%81%20CarPlay.md)
* [iOS CarPlay｜WWDC 总结](https://github.com/teney97/iOS-CarPlay/blob/main/iOS%20CarPlay%EF%BD%9CWWDC%20%E6%80%BB%E7%BB%93.md)

## 前言

笔者负责了宝宝巴士 CarPlay App 的研发，即将上线了，欢迎宝爸宝妈们体验。写下这篇文章算是做个收尾吧，并且想与你分享下开发经验。通过本篇文章，你可以学习到：

* CarPlay 是什么，适用于哪些类型的 App
* 让你的工程兼容 UIScene，也就是从传统的 UIWindow 和 AppDelegate 向 SceneDelegate 过渡
* 开发一个音频类 CarPlay App 的详细过程

## CarPlay 是什么

CarPlay 是 Apple 发布的一个车载系统，可以配合 iPhone 使用（iPad 不支持）。其前身是叫 iOS in the Car，2014 年更名为 CarPlay。

简单地说，如果你的汽车支持 CarPlay，那么当你的汽车（通过数据线、蓝牙、Wi-Fi）连接 iPhone 时，汽车显示屏会自动切换到 CarPlay，iPhone 上所有支持 CarPlay 的 App 会自动显示在 CarPlay 中，你可以在 iPhone 设置（通用 > CarPlay 车载）中屏蔽指定 App 或调整顺序。Apple 对 CarPlay App 用户界面采用一致的设计，内容由 App 自己提供。

控制 CarPlay 主要有 3 种方式：Siri、触屏显示屏、物理按键。

## CarPlay 支持哪些 App 及功能

* 音频 App 可以提供音乐、新闻、播客等。
* 通信 App（Messaging、VoIP calling）可以发送和接收消息，同时可以配合 Siri 使用。
* 导航 App 可以提供详细的地图、目的地搜索、路线指导和用户通知。
* 汽车制造商的 App 可以提供特定于车辆的控制和显示，让司机在不离开 CarPlay 的情况下保持联系。
* iOS 14 新功能，支持 EV 充电、停车和快餐订购 App。此外，所有 CarPlay App 都可以利用 CarPlay framework 提供一致的设计，并针对在汽车中的使用进行了优化。

## CarPlay App 开发流程概览

1. 首先需要确定你的 App 是否适用于 CarPlay，然后去开发者网站申请对应 App 类型的 CarPlay 权限，并对工程进行配置。只有这样你的工程才能使用 CarPlay Simulator。因此如果你计划要开发 CarPlay App 的话，最好提前去申请权限，因为 Apple 审核还要时间。在此期间可以看看相关开发文档，等权限申请下来就可以使用 CarPlay Simulator 调试开发啦。参考文档：[Apple｜申请 CarPlay 权限](https://developer.Apple.com/documentation/carplay/requesting_the_carplay_entitlements?language=objc)。

2. 从 iOS 14 开始，你可以使用 CarPlay framework 来开发音频 CarPlay App（如果是导航类 App，在 iOS 12 就可以使用），它提供了一些 UI 模版来支持开发者自定义界面；如果你要兼容 iOS 13 及更早版本的话需要使用 MediaPlayer framework 开发，它向前兼容。因此，如果你的 App 需要在 iOS 14 及更高版本上使用 CarPlay framework，并且兼容 iOS 13 及更早版本的话，就要维护两套代码，开发工作量可能接近 double。笔者仅支持了 iOS 14 及更高版本，在本篇文章中会详细讲解使用 CarPlay framework 的开发细节。如果你想支持低版本的话也可以看看笔者对「WWDC17 - 让您的 App 支持 CarPlay 车载」和「WWDC18 - CarPlay 车载音频和导航 App」做的笔记。

3. 在 iOS 14 及更高版本中使用 CarPlay framework 来开发 CarPlay App 必须使用 UIScene（UIScene 是 Apple 于 iOS 13 引入的，用于构建多窗口应用），因此你的工程必须从传统的 UIWindow 和 AppDelegate 向 SceneDelegate 过渡。如果你的工程已经兼容了 UIScene，那么就可以省去这步骤的工作；如果还未兼容的话，也可以看看我在本篇文章中提到的一些注意点。

4. CarPlay App 是附属于 iPhone App 的，它们是同一进程。

   * 如果是先启动了 CarPlay App，那么系统会在后台启动你的 iPhone App；
   * 如果杀死了 iPhone App 进程，那么 CarPlay App 就会关闭（在 UIScene 中是断开场景的连接）；
   * 如果关闭 CarPlay App（在 UIScene 中是断开场景的连接），iPhone App 进程不会被杀死。（好像只有退出整个 CarPlay，才会关闭所有 CarPlay App，没办法关闭指定 CarPlay App）；

   对于使用 CarPlay framework 开发的 CarPlay App，它和 iPhone App 就是两个 UIScene，因此它们可以一个处于后台一个处于前台。而对于 MediaPlayer framework 开发的 CarPlay App，它和 iPhone App 是高度绑定的，只能共处前台或后台，用户体验不好。例如你在使用 CarPlay 导航时，手机将无法进行别的操作，否则会打断导航进程。

5. 笔者开发的是音频类 CarPlay App，对其它类型的 App 没做了解，不过大致的开发流程应该差不多。开发一个音频类 CarPlay App 就是从 CPTemplateApplicationSceneDelegate 入口开始来构建 UI，填充数据。CarPlay App 的用户界面相对来说比较固定，但使用 CarPlay framework，Apple 支持更多可定制化的 UI 了。当车机连接后，音频将通过汽车扬声器播放。无论你是使用 CarPlay framework 还是 MediaPlayer framework 来构建的 CarPlay App，都是通过 MPNowPlayingInfoCenter 和 MPRemoteCommandCenter 来提供播放界面的音频信息以及响应远程播放控制事件。只不过在 CarPlay framework 中，一些远程控制事件通过 CPNowPlayingButton 的 handler 来处理了，比如播放模式、播放速率等等。当然如果你的 App 是音频类的话，应该已经支持了这些功能，因为 iPhone 锁屏界面以及控制中心的音频播放信息和播放控制也是通过它们提供。因此，我们只需要针对 CarPlay 做下优化或者功能增强就行。

   * 设置和更新 MPNowPlayingInfoCenter 的 nowPlayingInfo，它包含当前播放音频的信息，如标题、作者、时长等等；
   * 响应 MPRemoteCommandCenter 事件，对远程播放控制事件做出响应，如播放、暂停、切换歌曲等等；

   除了上面两点，你可能还需要：

   * 设置和更新 MPNowPlayingInfoCenter 的 playbackState，以更新 CarPlay App 上显示的音频播放状态：播放/暂停；
   * 设置和更新 MPRemoteCommandCenter 的 changeRepeatModeCommand.currentRepeatType，以更新 CarPlay App 上显示的播放模式状态：顺序循环/单曲循环

6. 了解 CarPlay framework 都支持哪些功能、 UI，然后帮助你的 PM 和 UI 完成需求、原型和设计。学习 UI 的使用，界面基本就是由 Template 和 Item 组成，常用的有 CPTabBarTemplate、CPListTemplate、CPListItem、CPListImageRowItem 等等。

6. 然后，就是开发啦！此处省略 ... 字。

8. 最后，在真实环境（汽车中）测试。[Apple｜使用 CarPlay Simulator 运行和调试 CarPlay App](https://developer.Apple.com/documentation/carplay/using_the_carplay_simulator?language=objc) 中列举了一些在 CarPlay Simulator 上无法测试的功能。另外，Apple 建议我们多测试弱网以及无网环境下的用户体验，因为驾驶过程中可能会经过网络不好的路段或区域。

9. 你可以玩玩 Apple 提供的音频类 CarPlay App 示例：[Apple｜CarPlay Music App](https://developer.Apple.com/documentation/carplay/integrating_carplay_with_your_music_App?language=objc)。

## 相关资料

### WWDC

* [WWDC16｜开发 CarPlay 车载系统 - 第 1 部分](https://developer.Apple.com/wwdc16/722)
* [WWDC16｜开发 CarPlay 车载系统 - 第 2 部分](https://developer.Apple.com/wwdc16/723)
* [WWDC17｜开发无线 CarPlay 车载系统](https://developer.Apple.com/wwdc17/717)
* [WWDC17｜让您的 App 支持 CarPlay 车载](https://developer.Apple.com/wwdc17/719)
* [WWDC18｜CarPlay 车载音频和导航 App](https://developer.Apple.com/wwdc18/213)
* [WWDC19｜CarPlay 车载系统改进](https://developer.Apple.com/wwdc19/252)
* [WWDC20｜使用 CarPlay 车载系统为你的 App 提速](https://developer.Apple.com/wwdc20/10635)

### 文档

* [CarPlay｜介绍](https://www.Apple.com.cn/ios/carplay/)
* [CarPlay｜文档首页](https://developer.Apple.com/carplay/)
* [CarPlay｜设计指南](https://developer.Apple.com/design/human-interface-guidelines/carplay/overview/introduction/)
* [CarPlay｜开发者文档](https://developer.Apple.com/documentation/carplay?language=objc)
  * [申请 CarPlay 权限](https://developer.Apple.com/documentation/carplay/requesting_the_carplay_entitlements?language=objc)
  * [使用 CarPlay Simulator 运行和调试 CarPlay App](https://developer.Apple.com/documentation/carplay/using_the_carplay_simulator?language=objc) （需要先完成上一步，才能启用 CarPlay Simulator 以及在 CarPlay 主屏幕中看到你的 App。如果你使用 M1 Mac，那可能用不了 CarPlay Simulator）
  * [在你的 CarPlay App 中显示内容](https://developer.Apple.com/documentation/carplay/displaying_content_in_carplay?language=objc)
  * [兼容 iOS13 及更早 iOS 系统](https://developer.Apple.com/documentation/carplay/supporting_previous_versions_of_ios?language=objc)
* [CarPlay｜App 编程指南](https://developer.Apple.com/carplay/documentation/CarPlay-App-Programming-Guide.pdf)
  * 搜狗机翻版：https://github.com/teney97/iOS-CarPlay/tree/main/Content
* [CarPlay｜适用车型](https://www.Apple.com.cn/ios/carplay/available-models/)
* [CarPlay｜Apple 的 CarPlay Music App 示例](https://developer.Apple.com/documentation/carplay/integrating_carplay_with_your_music_App?language=objc)
  * CarPlay Music 是 Apple 提供的一个音频类 CarPlay App 示例，它演示了如何在 CarPlay 中显示自定义 UI。CarPlay Music 使用 CarPlay framework 并通过实现 CPNowPlayingTemplate 和 CPListTemplate 来集成。这个示例 App 提供了一个日志界面，可以帮助你了解 CarPlay App 的生命周期和音乐控制器。


### 其它

* [苹果 iOS 13 CarPlay 详解：全新 UI 设计 + 独立 App 视图 ](https://www.sohu.com/a/336034138_120178230)
* [高德地图 CarPlay 适配指南](https://lbs.amap.com/api/ios-navi-sdk/guide/tools/carplay_navi)
