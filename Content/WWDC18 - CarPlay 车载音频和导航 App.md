## WWDC18 - CarPlay 车载音频和导航 App

**地址**：https://developer.apple.com/wwdc18/213

**时长**：38 分钟

**概览**：了解如何更新音频或导航 App 来支持 CarPlay 车载。CarPlay 车载中的 App 针对车用进行了优化，能够自动适应可用的汽车屏幕和输入控制。音频 App 能够输出音乐、新闻、播客等。通过新的 CarPlay 车载框架，导航 App 可以提供详细地图、目的地搜索、逐向导航和用户通知。

**总结**：该 Session 主要内容为以下两点，由于笔者要开发的是 CarPlay 音频 App，所以导航 App 的相关内容就不做研究了。

* CarPlay 音频 App 的性能改善和优化
* 针对 CarPlay 在导航 App 的全新 CarPlay framework

**如何让音频 App 支持 CarPlay 车载系统**

* Template based
* Works with all CarPlay systems
* Uses existing MediaPlayer APIs

CarPlay 所使用的模板会抽象剥离 CarPlay 中所具备的一些不同的复杂性，比如输入方式和屏幕尺寸等等，因此你的音频 App 只需提供数据内容并在 CarPlay 上显示即可，通常是通过 `tableview` 或 `tabs` 来实现，取决于你希望如何呈现你的数据。

你需要关注的是如何向 CarPlay 提供数据内容，如果你已经在开发一款音频 App 并且它使用的是现有的 API，那么你可能已经非常熟悉了。以下是你要在 CarPlay 车载系统中发布 App 所需要了解的 3 个 API，去年的 WWDC 视频已经详细解释了每一个 API。

要在 CarPlay 上浏览内容，需要使用 `MPPlayableContent` API，它有一个 `dataSource` 和一个 `delegate`，因此你的音频 App 可以提供数据内容给 CarPlay，并且委托会接收回调。`MPNowPlayingInfoCenter` 和 `MPRemoteCommandCenter` API 用于开发 “正在播放” 功能。

![](https://cdn.nlark.com/yuque/0/2021/png/12376889/1630374246223-1cad5cf8-1c87-4479-882d-aa337d98532b.png?x-oss-process=image%2Fresize%2Cw_750%2Climit_0)

以下代码是支持 CarPlay 音频 App 所需要的最少操作：

```swift
func application(
_ application: UIApplication, didFinishLaunchingWithOptions launchOptions:
[UIApplicationLaunchOptionsKey: Any]?) -> Bool {
    
    // Set up data source and delegate
    MPPlayableContentManager.shared().dataSource = SrirockaContentManager.shared
    MPPlayableContentManager.shared().delegate = SrirockaContentManager.shared
    
    // Set Now Playing metadata in MPNowPlayingInfoCenter
    let nowPlayingInfo: [String: Any] = [:]
    MPNowPlayingInfoCenter.default().nowPlayingInfo = nowPlayingInfo
    
    // Respond to remote command events
    let commandCenter = MPRemoteCommandCenter.shared()
    commandCenter.playCommand.isEnabled = true
    commandCenter.playCommand.addTarget { _ in .success }
}
```

**在 iOS12 中所做的性能改善和优化**

* Improved performance in `MPPlayableContent`
* Faster startup sequence 
* Smoother animations 
* Better communication to your app

优化 `MPPlayableContent` API，改善了它在如何调取数据源和委托方面的性能，你不需要修改任何代码。加速了启动过程，并提供了更流畅的动画。无论何时当前音频 App 在 CarPlay 上的内容发生变更时，还会向你的 App 提供更好的通讯，以预测用户可能希望播放什么内容，或在 CarPlay 上想要浏览什么内容。

**Best Practices**

下面我们来具体看看如何优化音频 App：

* Call `reloadData()` only when needed
  * 这是 `MPPlayableContent` 的一个 API， 它会了解如何为你的音频 App 进行更好优化。你应该只在必要的时候调用 `reloadData()`，它是一个非常昂贵的操作，会降低 App 的响应性。
* Use `beginUpdates()` and `endUpdates()`
  * 相反，如果你只需要更新一些内容，你应该把它们放到这两个 API 调用中，从而可以适当地更新内容。
* Keep an internal representation of the data source to optimize performance
  * 现在 `MPPlayableContent` 所拥有的调用是异步操作，当我们向你的 App 请求数据时。因此，需要在你 App 的某个位置保存内部陈述或信息缓存，那么当我们请求你的内容信息时，你就能迅速给我们提供信息，并使你的 App 保持响应。

**Don’t Miss a Beat!**

Account for these common scenarios

* Screen locked with passcode
* Unreliable network connectivity

接下来让我们讨论一些关于 “进一步优化音频 App 在 CarPlay 中的性能” 的方法：

播放列表加载不流畅，如果 App 不及时提供内容的话还会导致 CarPlay 加载超时。可能是由于什么原因导致的呢？

* CarPlay 用户通常会在不能快速连接的区域或手机屏幕锁定情况下驾驶汽车，而且解锁手机屏幕需要密码。如果你的 App 的数据保护政策依赖于手机处于解锁状态，你将不能获取你 App 的信息，并且最终 CarPlay 将会超时。因此如果你的数据，需要在手机处于解锁状态时获取，你就需要审查你的 App 的数据过渡政策。
* CarPlay 用户可能会在弱网或无网的区域驾驶，不同的 CarPlay 会提供不同的数据服务，并且你需要测试没有恒定 WIFI 网络连接的情况。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/12376889/1630376442054-bafd9ea7-678c-48b2-86b8-5d2b260313f4.png?x-oss-process=image%2Fresize%2Cw_750%2Climit_0)

如何优化？

使用 `beginLoadingChildItems()` 来初始化获取内容。

有一个API 叫做 `beginLoadingChildItems` 用于发起获取内容，无论何时当你的任意一个索引路径在 CarPlay 车上显示该 API 都会被调用（可以类比 `tableview` 的 `cellForRow` 方法）。当用户在 `tableview` 间滚动，或选择不同 `tabs`时，会针对显示屏上的每个单一索引路径调用 `beginLoadingChildItems`。这样在用户实际选择内容之前，你的 App 就有机会开始加载。这样当用户在选择内容时，要么正在网络请求中，要么数据已经准备完毕。

```swift
func beginLoadingChildItems(at indexPath: IndexPath,
                            completionHandler: @escaping (Error?) -> Void) {
    if indexPath.indexAtPosition(2) == 0 {
        // Start fetching content that requires a network connection or needs some setup
        startProcessingHeatingHabaneros()
        
    }
    ...
    completionHandler(nil)
}
```

下面 Apple 工程时举了一个场景，可能会在你开发 CarPlay App 时遇到。

如果你的音频 App 需要登录才能提供数据内容，那么在 CarPlay 上的体验将非常不好，用户将不能与你的 App 进行交互。你应该确保你提供某种类型的体验，从而用户起码在未登录情况下在 CarPlay 中使用你的 App，以提高用户体验。 

![](https://cdn.nlark.com/yuque/0/2021/png/12376889/1630378644753-6e95dfd9-dd93-4dfd-bbb3-a77ae77e92c4.png?x-oss-process=image%2Fresize%2Cw_750%2Climit_0)

**Greatest Hits**

* Use `MPPlayableContent` to populate the CarPlay screen 
* Anticipate user scenarios while driving 
* Run your audio apps in CarPlay!

总结一下，CarPlay 音频 App 拥有自己最棒的特色，你可以直接使用 `MPPlayableContent` API 来为你的 CarPlay App 提高数据内容。你需要注意一些情况，比如 iPhone 锁屏、用户未登录时，你的 App 仍然需要在 CarPlay 中展示完美的功能性。

在 iOS12 中，我们做了一些很棒的优化和性能改善，以使你的 App 在 CarPlay 中运行良好。你需要再次运行你的 App，并了解是否还有性能改善的空间，从而让你的 App 变得越来越好。
