## iOS CarPlay｜使用 MediaPlayer framework 开发

要让你的 CarPlay App 兼容 iOS 13 及更早版本，你必须看看 [兼容 iOS13 及更早 iOS 系统](https://developer.Apple.com/documentation/carplay/supporting_previous_versions_of_ios?language=objc)。对于音频类 App，主要需要做两件事：

1. 将 Key `com.apple.developer.playable-content` 添加到 Entitlements.plist 中并设置 Value 为 1；

```xml
<key>com.apple.developer.playable-content</key>
<true/>
<key>com.apple.developer.carplay-audio</key>
<true/>
```

2. 使用 MediaPlayer framework 开发。

### 开发流程

**CarPlay App Integration**

音频 App 的开发者需提供 **数据源** 和 **导航层次结构** 以便在 CarPlay 中显示。

![](https://gitee.com/junteng/images/raw/master/img/image.png)

要在 CarPlay 上浏览内容，需要使用 `MPPlayableContent` API，它有一个 `dataSource` 和一个 `delegate`，因此你的音频 App 可以提供数据内容给 CarPlay，并且委托会接收回调。`MPNowPlayingInfoCenter` 和 `MPRemoteCommandCenter` API 用于开发 “Now Playing” 功能。

![](https://gitee.com/junteng/images/raw/master/img/image (9).png)

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

**Requirement for Audio Apps**

* App 在获得 CarPlay 的权限之后，必须至少实现 MPPlayableContent API，包括 `MPPlayableContentDataSource` 和 `MPPlayableContentDelegate`，这样 CarPlay 才能获取内容和开始播放。
* 必须响应 MPRemoteCommandCenter 事件，让用户对你的内容执行命令，如播放、暂停、跳过曲目等等。
* 设置和更新 MPNowPlayingInfoCenter 字典，它包含当前播放音频的元数据，如标题、作者、时长等等。

**音频 App 的数据结构**

CarPlay 会使用 NSIndexPath 在特定的索引路径上请求 content item，这和 UITableView 中使用的 NSIndexPath 是不同的。类比在虚拟文件系统中，这将是特定文件夹中的文件数量。空索引路径（0, 0）表示根节点。

让我们来看一个示例， 这个例子中最左边的内容是根数据项，在用户界面中可以用单个的窗口（individual tabs）或者是 rootTableView 的形式来体现。

![](https://gitee.com/junteng/images/raw/master/img/image (1).png)

下面是层次结构中表示的每个内容项的索引路径

![](https://gitee.com/junteng/images/raw/master/img/image (3).png)

当 CarPlay 根据一个特定的索引路径请求一个 content item 时，音频 App 将遍历请求的 content item 的层次结构，来查找所需内容。

在这个例子中，我们正在请求第一个内容的第一个子内容，它是 Running content item。

![](https://gitee.com/junteng/images/raw/master/img/image (4).png)

CarPlay 还会询问特定索引的子项目数量。在本例中，我们询问第二个内容项的第三个子项的子项数量。在本例中，我们将返回 2。

![](https://gitee.com/junteng/images/raw/master/img/image (5).png)

**内容限制**

有些车辆可能会根据车辆是否在行驶而强制在屏幕上显示有限的内容。这可能会限制屏幕上可显示的行数，以及点击进某一个目录后可显示的内容。你的 App 可以利用 MPPlayableContentManager 来处理这些变化。

查询所显示的内容是否会受到限制，实现以下代理方法，判断是否执行了内容限制，以及获取受到内容限制时 `列表中显示的项目的最大数目 enforcedContentItemsCount` 和 `层次结构中的最大深度 enforcedContentTreeDept`。

```swift
extension YourAppContentManager : MPPlayableContentDelegate {
    // called whenever CarPlay state changes
    func playableContentManager(
                _ contentManager: MPPlayableContentManager,
               didUpdate context: MPPlayableManagerContext) {
        // check to see if content limits are enforced
        let contentLimitsEnforced = context.contentLimitsEnforced
        if contentLimitsEnforced {
            // the maximum number of items shown in a list when content limits are enforced
            let contentLimitItemCount = context.enforcedContentItemsCount
            // the maximum depth in the hierarchy when content limits are enforced
            let contentLimitTreeDepth = context.enforcedContentTreeDept
        } else {
            …
        }
    }
}
```

如果你的 App 更适合使用 Tabs 而不是 TableView。

* 通过添加 `UIBrowsableContentSupportsSectionedBrowsing` 到 Info.plist 来添加 tabs
* 推荐使用最多 4 个 tabs 并且使用较短的标题，因为空间有限并且有些汽车的屏幕比较窄，而且有正在播放的内容的时候还需显示 “正在播放” 按钮
* Tab image assets 会在 CarPlay 屏幕上作为 template images 渲染

**Now Playing Screen**

现在让我们来看一下 CarPlay 中 “正在播放” 的界面。一些控件和元数据与 iPhone 的控制中心显示的是一样的。总的来说，你在控制中心所看到的控件和数据也应该出现在 CarPlay “正在播放” 界面上。用户可以通过点击 App 导航界面右上角的 “正在播放” 或通过 CarPlay 主屏幕上的 “正在播放” App 进入 “正在播放” 界面。运行过程中，你的 App 名字会显示在 “正在播放” 界面的右上角。

![](https://gitee.com/junteng/images/raw/master/img/image (6).png)

CarPlay “正在播放” 界面的元数据设置与 `控制中心` 和其它源相同。设置 `MPNowPlayingInfoCenter` 的 nowPlayingInfo 属性 ，并填写尽可能多的信息。

```swift
// Set Metadata to be Displayed in Now Playing Info Center

let infoCenter = MPNowPlayingInfoCenter.default()
infoCenter.nowPlayingInfo = [MPMediaItemPropertyTitle: "Style",
                             MPMediaItemPropertyArtist: "Taylor Swift",
                             MPMediaItemPropertyAlbumTitle: "1989",
                             MPMediaItemPropertyGenre: "Pop",
                             MPMediaItemPropertyReleaseDate: "2014",
                             MPMediaItemPropertyPlaybackDuration: 231,
                             MPMediaItemPropertyArtwork: mediaItemArtwork,
                             MPNowPlayingInfoPropertyElapsedPlayback: 53,
                             MPNowPlayingInfoPropertyDefaultPlaybackRate: 1,
                             MPNowPlayingInfoPropertyPlaybackRate: 1,
                             MPNowPlayingInfoPropertyPlaybackQueueCount: 13,
                             MPNowPlayingInfoPropertyPlaybackQueueIndex: 3,
                            … ]
```

**Playback Controls**

另外，响应播放命令，以便用户可以选择调整内容，如播放、暂停、跳过曲目、随机播放或重复播放歌单。以下是 CarPlay “正在播放” 界面支持的指令

MPRemoteCommandCenter

* Play, pause, stop
* Previous track, next track
* Seek backward, seek forward
* Skip backward, skip forward
* Shuffle, repeat
* Like, dislike, bookmark
* Change playback rate

**Changing Playback Rate**

iOS 11 的新功能是 “正在播放” 界面支持显示和调整播放速度。如果你的 App 需要做此支持的话：

* 添加 `MPNowPlayingInfoPropertyDefaultPlaybackRate` 到 `MPNowPlayingInfoCenter` 中
* 并且利用 `MPNowPlayingInfoCenter` 里存有所支持的一系列的播放速度的数组，来响应调整播放速度的指令
* Implement `changePlaybackRateCommand` with `supportedPlaybackRates` in `MPRemoteCommandCenter`

下面是一个如何实现调整播放速度的代码示例。在这个例子中，当前播放速度是 1.0，支持的播放速度选项有 0.5、1.0、1.5、2.0。当用户想要增加播放速度时，如果音频正在以最快的支持速度播放，那么继续增加播放速度将循环调到最小速度。在这个例子中，如果用户连续三次调快播放速度，其变化为 1.0 -> 1.5 -> 2.0 -> 0.5。

```swift
// Change Playback Rate

let infoCenter = MPNowPlayingInfoCenter.default()
infoCenter.nowPlayingInfo = [MPNowPlayingInfoPropertyDefaultPlaybackRate: 1.0,
                             … ]

let changePlaybackRateCommand = MPRemoteCommandCenter.shared().changePlaybackRateCommand
changePlaybackRateCommand.supportedPlaybackRates = [0.5, 1.0, 1.5, 2.0]
```

**Playback Controls**

一些音频 App 可能会响应 `MPRemoteCommandCenter` 中的多项指令。根据所支持的命令，CarPlay 的 “正在播放” 界面将把某些相关的命令组合成一个按钮，如 `省略号` 或 `菜单` 按钮，来替代之前的按钮。

![](https://gitee.com/junteng/images/raw/master/img/image (7).png)

![](https://gitee.com/junteng/images/raw/master/img/image (8).png)

**Best Practices**

* **在调用 `MPPlayableContentDataSource` 和 `MPPlayableContentDelegate` 中的 completion handlers 之前，确保将要播放的内容实际上已经准备好播放或显示，在此期间显示加活动指示器。**(否则会进入再退出播放页面)
* 对于那些不用 `Tabs` 而只用 `TableView` 的 App，`TableView` 必须返回至少一项内容。
* CarPlay 会显示一个加载活动指示器，如果 App 在一定时限内没有返回内容就会超时。如果你的 App 需要一些初始的设置，比如登录凭据，在页面的第一行应该提示用户 App 当前的状态，通过填充一个 `MPContentItem`，MPContentItem is neither playable nor a container indicating the state of the app to the user。

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

![](https://gitee.com/junteng/images/raw/master/img/image (10).png)

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

![](https://gitee.com/junteng/images/raw/master/img/image (11).png)

**Greatest Hits**

* Use `MPPlayableContent` to populate the CarPlay screen 
* Anticipate user scenarios while driving 
* Run your audio apps in CarPlay!

总结一下，CarPlay 音频 App 拥有自己最棒的特色，你可以直接使用 `MPPlayableContent` API 来为你的 CarPlay App 提高数据内容。你需要注意一些情况，比如 iPhone 锁屏、用户未登录时，你的 App 仍然需要在 CarPlay 中展示完美的功能性。

在 iOS12 中，我们做了一些很棒的优化和性能改善，以使你的 App 在 CarPlay 中运行良好。你需要再次运行你的 App，并了解是否还有性能改善的空间，从而让你的 App 变得越来越好。

### API

#### MPPlayableContentManager

MPPlayableContentManager 是一个管理一个媒体 App 和一个外部媒体播放器界面之间交互的类。该 App 向 ContentManager 提供：

* dataSource，该数据源允许媒体播放器浏览 App 提供的媒体内容
* delegate，该代理允许媒体播放器将非媒体远程播放命令转发给该 App

```swift
@available(iOS, introduced: 7.1, deprecated: 14.0, message: "Use CarPlay framework")
class MPPlayableContentManager : NSObject {

    weak var dataSource: MPPlayableContentDataSource?
    weak var delegate: MPPlayableContentDelegate?

    @available(iOS, introduced: 8.4, deprecated: 14.0, message: "Use CarPlay framework")
    var context: MPPlayableContentManagerContext { get }

    /// 告诉 ContentManager 当前正在播放的 MPContentItems 的 identifiers
    @available(iOS, introduced: 10.0, deprecated: 14.0, message: "Use CarPlay framework")
    var nowPlayingIdentifiers: [String]

    class func shared() -> Self

    /// 告诉 ContentManager 数据源已经更改，需要从数据源重新加载数据
    func reloadData()
    
    /// 用于一个对多个 MPContentItems 开始同步更新
    func beginUpdates()
    /// 结束同步更新
    func endUpdates()
}
```

#### MPPlayableContentManagerContext

MPPlayableContentManagerContext 表示 playable 内容端点的当前状态，一个 context 是可以从 MPPlayableContentManager 的实例中获取的。

```swift
@available(iOS, introduced: 8.4, deprecated: 14.0, message: "Use CarPlay framework")
class MPPlayableContentManagerContext : NSObject {

    /// content server 在限制内容时，显示的 items 数量
    /// 如果 contentLimitsEnforced 为 false，返回 NSIntegerMax
    var enforcedContentItemsCount: Int { get }
    /// content server 允许的导航层次结构的深度，超过这个限制将导致 Crash
    var enforcedContentTreeDepth: Int { get }
    /// content server 是否限制内容
    var contentLimitsEnforced: Bool { get }
    /// content server 是否可用
    var endpointAvailable: Bool { get }
}
```

#### MPPlayableContentDataSource

MPPlayableContentDataSource 是一个协议，如果 application 对象想要支持外部媒体播放器（如 CarPlay），则需要遵循该协议。数据源负责以一种有意义的方式向这些系统提供有关媒体的元数据，这样就可以自动设置用户界面和播放队列等功能。

```swift
@available(iOS, introduced: 7.1, deprecated: 14.0, message: "Use CarPlay framework")
protocol MPPlayableContentDataSource : NSObjectProtocol {

    /// 告诉数据源开始加载由 indexPath 指定 item 的 child content items
    /// 这是为了让 App 可以在 MediaPlayer 开始要求显示 content items 之前开始异步批量加载内容
    /// 如果实现了这个方法，App 应该总是在加载完成后调用 completionHandler
    optional func beginLoadingChildItems(at indexPath: IndexPath, completionHandler: @escaping (Error?) -> Void)

    /// 告诉 MediaPlayer 数据源提供的内容是否支持播放进度，作为其元数据的属性。
    /// 如果未实现此方法，MediaPlayer 将假定不支持任何 content items 的进度。
    optional func childItemsDisplayPlaybackProgress(at indexPath: IndexPath) -> Bool

    /// 为提供的 identifier 提供一个 content item
    /// 如果没有与 identifier 对应的 content item，则提供 nil
    /// 如果存在不允许检索 content item 的 error，则提供一个 error
    /// 如果实现了这个方法，App 应该总是在加载完成后调用 completionHandler
    @available(iOS, introduced: 10.0, deprecated: 14.0, message: "Use CarPlay framework")
    optional func contentItem(forIdentifier identifier: String, completionHandler: @escaping (MPContentItem?, Error?) -> Void)
    /// 返回指定 indexPath 上的子节点数
    /// 在虚拟文件系统中，这将是特定文件夹中的文件数量。空索引路径（0, 0）表示根节点
    func numberOfChildItems(at indexPath: IndexPath) -> Int

    /// 返回位于指定 indexPath 上的 content item
    /// 如果 content item 在返回后发生了变化，则其更新后的内容将被发送到 MediaPlayer
    func contentItem(at indexPath: IndexPath) -> MPContentItem?
}
```

#### MPPlayableContentDelegate

MPPlayableContentDelegate 是一个允许外部媒体播放器向 App 发送播放命令的协议。例如，用户可以浏览 App 的媒体内容（由 MPPlayableContentDataSource 提供）并选择要播放的 content item。如果媒体播放器决定要播放该 item，它将要求 App 的 content delegate 启动播放。

```swift
@available(iOS, introduced: 7.1, deprecated: 14.0, message: "Use CarPlay framework")
protocol MPPlayableContentDelegate : NSObjectProtocol {
    
    /// 当媒体播放器界面希望播放请求的 content item 时，将调用此方法
    /// 如果 item 开始播放时出现错误，App 应该调用 completionHandler 并传入 error
    /// [注] 点击 isPlayable 为 true 的 MPContentItem 会调用此方法
    @available(iOS, introduced: 7.1, deprecated: 14.0, message: "Use CarPlay framework")
    optional func playableContentManager(_ contentManager: MPPlayableContentManager, initiatePlaybackOfContentItemAt indexPath: IndexPath, completionHandler: @escaping (Error?) -> Void)
    
    /// 当媒体播放器界面希望 now playing app 为以后的播放设置一个播放队列时，将调用此方法
    /// app 应该将内容加载到播放队列中，但在接收到播放命令或者 playableContentManager 请求播放其他内容时才开始播放
    /// 一旦 app 准备好播放一些东西，就应该调用 completionHandler
    @available(iOS, introduced: 9.0, deprecated: 9.3, message: "Use Intents framework for initiating playback queues.")
    optional func playableContentManager(_ contentManager: MPPlayableContentManager, initializePlaybackQueueWithCompletionHandler completionHandler: @escaping (Error?) -> Void)
  
    /// 当媒体播放器界面希望 now playing app 为以后的播放设置一个播放队列时，将调用此方法
    /// app 应该根据所提供的 content item 将内容加载到播放队列中，并为播放做好准备，但在接收到播放命令或 playableContentManager 请求播放其他内容之前，不启动播放
    /// 一个 nil contentItems array 意味着 app 应该用它认为合适的任何东西来准备它的队列。
    /// 一旦 app 准备好播放一些东西，就应该调用 completionHandler
    @available(iOS, introduced: 9.3, deprecated: 12.0, message: "Use Intents framework for initiating playback queues.")
    optional func playableContentManager(_ contentManager: MPPlayableContentManager, initializePlaybackQueueWithContentItems contentItems: [Any]?, completionHandler: @escaping (Error?) -> Void)

    /// 当 content server 通知 manager 当前 context 已更改时，调用此方法
    @available(iOS, introduced: 8.4, deprecated: 14.0, message: "Use CarPlay framework")
    optional func playableContentManager(_ contentManager: MPPlayableContentManager, didUpdate context: MPPlayableContentManagerContext)
}
```

#### MPContentItem

MPContentItem 表示用于在客户端应用程序外表示的特定媒体项的高级元数据。开发人员可能想要表示的媒体项的例子包括歌曲文件、流媒体音频 url 或电台。

```swift
@available(iOS 7.1, *)
class MPContentItem : NSObject {
    /// 指定构造器。需要一个唯一标识符来标识 item 以供以后使用。
    init(identifier: String)

    /// 唯一标识符 (Required)
    var identifier: String { get }
    /// 标题
    var title: String?
    /// 副标题
    var subtitle: String?
    /// 封面
    var artwork: MPMediaItemArtwork?
    /// 播放进度，0.0 代表未播放，1.0 代表已播完，默认是 -1.0（不显示进度指示器）
    var playbackProgress: Float
    /// 是否为流内容，也就是存在 cloud 而不是本地
    @available(iOS 10.0, *)
    var isStreamingContent: Bool 
    /// 是否为显式内容
    @available(iOS 10.0, *)
    var isExplicitContent: Bool
    /// 是否为容器，包含其他 content item，如 album 或 playlist
    var isContainer: Bool
    /// 是否可播放
    var isPlayable: Bool
}
```

### 播放页

无论你是使用 CarPlay framework 还是 MediaPlayer framework 来构建的 CarPlay App，都是通过 [MPNowPlayingInfoCenter](https://developer.apple.com/documentation/mediaplayer/mpnowplayinginfocenter/) 和 [MPRemoteCommandCenter](https://developer.apple.com/documentation/mediaplayer/mpremotecommandcenter/) 来提供播放界面的音频信息以及响应远程播放控制事件。不同之处在于，使用 CarPlay framework 时，播放重复模式（changeRepeatModeCommand）、播放速度（changePlaybackRateCommand）等远程控制命令不再通过 target-action 处理，而是通过 CPNowPlayingRepeatButton、CPNowPlayingPlaybackRateButton 的 handler 处理；而使用 MediaPlayer framework，这些 command 通过 target-action 来处理。

#### 调整播放速度的坑

播放页支持显示和调整播放速度是 iOS 11 的新功能。

要支持该功能，使用 CarPlay framework 你需要做几件事情：

1. 添加 MPNowPlayingInfoPropertyPlaybackRate、MPNowPlayingInfoPropertyDefaultPlaybackRate 到 MPNowPlayingInfoCenter 中
2. 设置 changePlaybackRateCommand.enabled 为 true
3. 实例化一个 CPNowPlayingRepeatButton，并通过 updateNowPlayingButtons 更新到 nowPlayingTemplate
4. 在 CPNowPlayingRepeatButton 的 handler 中，更新 App 的音频播放速度，并更新 nowPlayingInfo（更新  MPNowPlayingInfoPropertyPlaybackRate、MPNowPlayingInfoPropertyDefaultPlaybackRate 的值）

```swift
// 1.
let infoCenter = MPNowPlayingInfoCenter.default()
infoCenter.nowPlayingInfo = [MPNowPlayingInfoPropertyDefaultPlaybackRate: 1,
                             MPNowPlayingInfoPropertyPlaybackRate: 1,
                            … ]
// 2.
let remoteCommandCenter = MPRemoteCommandCenter.shared()
remoteCommandCenter.changePlaybackRateCommand.enabled = true
// 3.
let playbackRateButton = CPNowPlayingPlaybackRateButton() { 
    // 4. 更新 App 的音频播放速度，并更新 nowPlayingInfo
    ... 
}
nowPlayingTemplate.updateNowPlayingButtons([playbackRateButton, ...])
```

而使用 MediaPlayer framework 你需要做的事情是不同的：

1. 添加 MPNowPlayingInfoPropertyPlaybackRate、MPNowPlayingInfoPropertyDefaultPlaybackRate 到 MPNowPlayingInfoCenter 中
2. 设置 changePlaybackRateCommand.enabled 为 true
3. 为 changePlaybackRateCommand.enabled 添加 target-action
4. 设置 changePlaybackRateCommand.supportedPlaybackRates，值为所支持的播放速度数组
5. 响应 action，更新 App 的音频播放速度，并更新 nowPlayingInfo（更新  MPNowPlayingInfoPropertyPlaybackRate、MPNowPlayingInfoPropertyDefaultPlaybackRate 的值）

```swift
// 1.
let infoCenter = MPNowPlayingInfoCenter.default()
infoCenter.nowPlayingInfo = [MPNowPlayingInfoPropertyDefaultPlaybackRate: 1,
                             MPNowPlayingInfoPropertyPlaybackRate: 1,
                            … ]
// 2.
MPRemoteCommandCenter *commandCenter = [MPRemoteCommandCenter sharedCommandCenter];
commandCenter.changePlaybackRateCommand.enabled = YES;
// 3.
[commandCenter.changePlaybackRateCommand addTarget:self action:@selector(changePlaybackRate)];
// 4.
commandCenter.changePlaybackRateCommand.supportedPlaybackRates = @[@0.5, @1.0, @1.5, @2.0];
// 5.
- (MPRemoteCommandHandlerStatus)changePlaybackRate {
    // 更新 App 的音频播放速度，并更新 nowPlayingInfo
    ...
    return MPRemoteCommandHandlerStatusSuccess;
}
```

对于在 CarPlay 中调整播放速度的最佳实践是，例如，当前播放速度是 1.0，支持的播放速度选项有 `[0.5, 1.0, 1.5, 2.0]`。当用户点击播放速度按钮时，增加 App 播放速度，并通过更新 nowPlayingInfo 同步到 CarPlay。如果当前音频正在以最快的支持速度播放，那么继续增加播放速度就将其调到最小速度，以此循环。在这个例子中，如果用户连续三次调快播放速度，其变化为 1.0 -> 1.5 -> 2.0 -> 0.5。

在使用 MediaPlayer framework 支持播放速度时，我遇到了一个问题：我们 App 的 音频所支持的播放速度选项有 `[0.5, 0.6, 0.7, 0.8, 0.9, 1.0, 1.1, 1.2, 1.3, 1.4, 1.5]`。在 CarPlay 播放页中点击播放速度按钮，播放速度成功更改了一次（1.0 -> 1.1），而接下来无论怎么点击都没有效果，action 都没有被调用；而在 iPhone App 中更改播放速度，是可以正常同步到 CarPlay App 中的。令我困惑的是，在《WWDC17 - 让你的 App 支持 CarPlay 车载》中 Apple 工程师对于 CarPlay App 支持调整播放速度并没有其他额外的步骤。通过查看 Apple 的文档以及谷歌，我也没有找到该问题的原因和解决方案。

最后，我将 supportedPlaybackRates 的值改为《WWDC17 - 让你的 App 支持 CarPlay 车载》中 Apple 工程师演示的示例值 `[0.5, 1.0, 1.5, 2.0]`，并将我们 App 所支持的播放速度也改为此，发现该功能能正常使用了。我似乎发现了点蛛丝马迹，将所支持的播放速度改为 `[0.5, 0.6, 1.0, 1.5, 2.0]`，结果验证了我的猜想是对的。通过点击 CarPlay 播放页中的播放速度按钮，播放速度成功更改 1.0 -> 1.5 -> 2.0 -> 0.5，目前为止都没有问题，但当我再次点击按钮时，action 就没有被调用了。然后我通过 iPhone App 将播放速度调至为 1.0、1.5、2.0 的任意一值并更新 nowPlayingInfo，CarPlay App 上的播放速度按钮功能又正常了，直到调整到 0.5 时继续失效。

因此，我得出的结论为在 MediaPlayer framework 下，CarPlay App 所支持的播放速度需为 0.5 的倍数，否则会导致功能异常，但目前我并没有在 Apple 的文档中找到依据。为了兼容 iPhone App 所支持的一些列播放速度值 `[0.5, 0.6, 0.7, 0.8, 0.9, 1.0, 1.1, 1.2, 1.3, 1.4, 1.5]`，并让 CarPlay App 支持调整播放速度功能  `[0.5, 1.0, 1.5]`，我的解决方案是：

* 若用户通过 CarPlay App 调整播放速度，则播放速度在 `[0.5, 1.0, 1.5]` 三个值之间进行切换
* 若用户通过 iPhone App 调整播放速度，如果设置的播放速度值包含于 `[0.5, 1.0, 1.5]`，那就在 CarPlay App 中显示播放速度按钮；否则隐藏播放速度按钮





如何在CarPlay for Audio App中添加标签？https://www.xknote.com/ask/60d536b9ec7f1.html

## 问题

* 如果点击 item 进行播放，而这时候实际上还未开始播放的话，会 push 进播放页又紧接着自动 pop。应该是要等到音频开始播放了再调用 completionHandler
