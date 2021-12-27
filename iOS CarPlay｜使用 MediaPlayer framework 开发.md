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



### MPPlayableContentManager

MPPlayableContentManager 是一个管理一个媒 App 和一个外部媒体播放器界面之间交互的类。该 App 向 ContentManager 提供：

* dataSource，该数据源允许媒体播放器浏览 App 提供的媒体内容
* delegate，该代理允许媒体播放器将非媒体远程播放命令转发给该 App

```swift
@available(iOS, introduced: 7.1, deprecated: 14.0, message: "Use CarPlay framework")
open class MPPlayableContentManager : NSObject {

    weak open var dataSource: MPPlayableContentDataSource?
    weak open var delegate: MPPlayableContentDelegate?

    @available(iOS, introduced: 8.4, deprecated: 14.0, message: "Use CarPlay framework")
    open var context: MPPlayableContentManagerContext { get }

    /// 告诉 ContentManager 当前正在播放的 MPContentItems 的 identifiers
    @available(iOS, introduced: 10.0, deprecated: 14.0, message: "Use CarPlay framework")
    open var nowPlayingIdentifiers: [String]

    open class func shared() -> Self

    /// 告诉 ContentManager 数据源已经更改，需要从数据源重新加载数据
    open func reloadData()
    
    /// 用于一个对多个 MPContentItems 开始同步更新
    open func beginUpdates()
    /// 结束同步更新
    open func endUpdates()
}
```

### MPPlayableContentManagerContext

MPPlayableContentManagerContext 表示 playable 内容端点的当前状态，一个 context 是可以从 MPPlayableContentManager 的实例中获取的。

```swift
@available(iOS, introduced: 8.4, deprecated: 14.0, message: "Use CarPlay framework")
open class MPPlayableContentManagerContext : NSObject {

    /// content server 在限制内容时，显示的 items 数量
    /// 如果 contentLimitsEnforced 为 false，返回 NSIntegerMax
    open var enforcedContentItemsCount: Int { get }
    /// content server 允许的导航层次结构的深度，超过这个限制将导致 Crash
    open var enforcedContentTreeDepth: Int { get }
    /// content server 是否限制内容
    open var contentLimitsEnforced: Bool { get }
    /// content server 是否可用
    open var endpointAvailable: Bool { get }
}
```

### MPPlayableContentDataSource

MPPlayableContentDataSource is a protocol that application objects conform to if they want to support external media players, such as vehicle head units. Data sources are responsible for providing metadata about your media to these systems in a meaningful way, so that features like user interfaces and play queues can be setup automatically.

```swift
@available(iOS, introduced: 7.1, deprecated: 14.0, message: "Use CarPlay framework")
public protocol MPPlayableContentDataSource : NSObjectProtocol {

    /// Tells the data source to begin loading content items that are children of the
    /// item specified by indexPath. This is provided so that applications can begin
    /// asynchronous batched loading of content before MediaPlayer begins asking for
    /// content items to display.
    /// Client applications should always call the completion handler after loading
    /// has finished, if this method is implemented.
    optional func beginLoadingChildItems(at indexPath: IndexPath, completionHandler: @escaping (Error?) -> Void)

    /// Tells the data source to begin loading content items that are children of the
    /// item specified by indexPath. This is provided so that applications can begin
    /// asynchronous batched loading of content before MediaPlayer begins asking for
    /// content items to display.
    /// Client applications should always call the completion handler after loading
    /// has finished, if this method is implemented.
    optional func beginLoadingChildItems(at indexPath: IndexPath) async throws

    
    /// Tells MediaPlayer whether the content provided by the data source supports
    /// playback progress as a property of its metadata.
    /// If this method is not implemented, MediaPlayer will assume that progress is
    /// not supported for any content items.
    optional func childItemsDisplayPlaybackProgress(at indexPath: IndexPath) -> Bool

    
    /// Provides a content item for the provided identifier.
    /// Provide nil if there is no content item corresponding to the identifier.
    /// Provide an error if there is an error that will not allow content items
    /// to be retrieved.
    /// Client applications should always call the completion handler after loading
    /// has finished, if this method is implemented.
    @available(iOS, introduced: 10.0, deprecated: 14.0, message: "Use CarPlay framework")
    optional func contentItem(forIdentifier identifier: String, completionHandler: @escaping (MPContentItem?, Error?) -> Void)

    /// Provides a content item for the provided identifier.
    /// Provide nil if there is no content item corresponding to the identifier.
    /// Provide an error if there is an error that will not allow content items
    /// to be retrieved.
    /// Client applications should always call the completion handler after loading
    /// has finished, if this method is implemented.
    @available(iOS, introduced: 10.0, deprecated: 14.0, message: "Use CarPlay framework")
    optional func contentItem(forIdentifier identifier: String) async throws -> MPContentItem

    
    /// Returns the number of child nodes at the specified index path. In a virtual
    /// filesystem, this would be the number of files in a specific folder. An empty
    /// index path represents the root node.
    func numberOfChildItems(at indexPath: IndexPath) -> Int

    
    /// Returns the content item at the specified index path. If the content item is
    /// mutated after returning, its updated contents will be sent to MediaPlayer.
    func contentItem(at indexPath: IndexPath) -> MPContentItem?
}
```

### MPPlayableContentDelegate

The MPPlayableContentDelegate is a protocol that allows for external media players to send playback commands to an application. For instance, the user could browse the application's media content (provided by the MPPlayableContentDataSource) and selects a content item to play. If the media player decides that it wants to play the item, it will ask the application's content delegate to initiate playback.

```swift
@available(iOS, introduced: 7.1, deprecated: 14.0, message: "Use CarPlay framework")
public protocol MPPlayableContentDelegate : NSObjectProtocol {
    
    /// This method is called when a media player interface wants to play a requested
    /// content item. The application should call the completion handler with an
    /// appropriate error if there was an error beginning playback for the item.
    @available(iOS, introduced: 7.1, deprecated: 14.0, message: "Use CarPlay framework")
    optional func playableContentManager(_ contentManager: MPPlayableContentManager, initiatePlaybackOfContentItemAt indexPath: IndexPath, completionHandler: @escaping (Error?) -> Void)

    /// This method is called when a media player interface wants to play a requested
    /// content item. The application should call the completion handler with an
    /// appropriate error if there was an error beginning playback for the item.
    @available(iOS, introduced: 7.1, deprecated: 14.0, message: "Use CarPlay framework")
    optional func playableContentManager(_ contentManager: MPPlayableContentManager, initiatePlaybackOfContentItemAt indexPath: IndexPath) async throws

    
    /// This method is called when a media player interface wants the now playing
    /// app to setup a playback queue for later playback. The application should
    /// load content into its play queue but not start playback until a Play command is
    /// received or if the playable content manager requests to play something else.
    /// The app should call the provided completion handler once it is ready to play
    /// something.
    @available(iOS, introduced: 9.0, deprecated: 9.3, message: "Use Intents framework for initiating playback queues.")
    optional func playableContentManager(_ contentManager: MPPlayableContentManager, initializePlaybackQueueWithCompletionHandler completionHandler: @escaping (Error?) -> Void)

    
    /// This method is called when a media player interface wants the now playing
    /// app to setup a playback queue for later playback. The application should
    /// load content into its play queue based on the provided content items and
    /// prepare it for playback, but not start playback until a Play command is
    /// received or if the playable content manager requests to play something else.
    /// A nil contentItems array means that the app should prepare its queue with
    /// anything it deems appropriate.
    /// The app should call the provided completion handler once it is ready to play
    /// something.
    @available(iOS, introduced: 9.3, deprecated: 12.0, message: "Use Intents framework for initiating playback queues.")
    optional func playableContentManager(_ contentManager: MPPlayableContentManager, initializePlaybackQueueWithContentItems contentItems: [Any]?, completionHandler: @escaping (Error?) -> Void)

    
    /// This method is called when the content server notifies the manager that the current context has changed.
    @available(iOS, introduced: 8.4, deprecated: 14.0, message: "Use CarPlay framework")
    optional func playableContentManager(_ contentManager: MPPlayableContentManager, didUpdate context: MPPlayableContentManagerContext)
}
```



### MPContentItem

MPContentItem 表示用于在客户端应用程序外表示的特定媒体项的高级元数据。开发人员可能想要表示的媒体项的例子包括歌曲文件、流媒体音频 url 或电台。

```swift
@available(iOS 7.1, *)
open class MPContentItem : NSObject {
    /// 指定构造器。需要一个唯一标识符来标识 item 以供以后使用。
    public init(identifier: String)

    /// 唯一标识符. (Required)
    open var identifier: String { get }
    /// 标题
    open var title: String?
    /// 副标题
    open var subtitle: String?
    /// 封面
    open var artwork: MPMediaItemArtwork?
    /// 播放进度，0.0 代表未播放，1.0 代表已播完，默认是 -1.0（不显示进度指示器）
    open var playbackProgress: Float
    /// 是否为流内容，也就是存在 cloud 而不是本地
    @available(iOS 10.0, *)
    open var isStreamingContent: Bool 
    /// 是否为显式内容
    @available(iOS 10.0, *)
    open var isExplicitContent: Bool
    /// 是否为容器，包含其他 content item，如 album 或 playlist
    open var isContainer: Bool
    /// 是否可播放
    open var isPlayable: Bool
}
```





如何在CarPlay for Audio App中添加标签？https://www.xknote.com/ask/60d536b9ec7f1.html

