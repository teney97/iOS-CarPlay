## WWDC17 - 让你的 App 支持 CarPlay 车载

**地址**：https://developer.apple.com/wwdc17/719

**时长**：28 分钟

**概览**：了解如何让你的音频、信息、VolP 通话或汽车制造商 App 支持 CarPlay 车载。音频、信息、VolP 通话 App 采用一致的设计，并且为在车内使用进行过优化。汽车制造商 App 提供车辆相关的控制和显示功能，让驾驶员无需离开 CarPlay 车载就能保持互联。探索最佳做法，并了解适用于 CarPlay 车载 App 的工具和框架。

**总结**：

该 Session 是面向开发者的，简要介绍了 App 在 CarPlay 中是如何运行的、如何将 App 集成到 CarPlay 平台上、开发规范以及如何避免常见的错误。

CarPlay 中的所有 App 均可使用的功能：

* 显示正在播放的音频信息以及播放控制
* 音频将通过汽车的扬声器播放
* SiriKit 支持一些适当的意图。如果用户触发了一个 CarPlay 不支持的 SiriKit 意图，Siri 会告诉用户这个功能在车中不可使用

即使你的 App 不支持 CarPlay，用户也可能在开始驾驶之前使用你的 App。为了 CarPlay 良好的用户体验，要确保你的 App 在连接 CarPlay 时不会播放音频，除非有请求，因为你的 App 播放的音频可能会与用户当前在 CarPlay 中播放的音频内容交叠。

在一些情况下，你的 App 可能希望与其它音频进行适当的交互。

* 对于音频类 App，使用以下协议，在其它音频播放的时候，让你的音频暂停。
  * AVAudioSessionModeSpokenAudio
* 对于导航类 App，可能会间歇性发出导航指示音频，使用以下两个协议。第二个协议将会在你的音频对话之前，暂停标记为 “spoken audio” 的音频的播放，因为你的导航音频将会与其它音频混合播放，所以将你的音频对话标为 “可以混合” 以便与其它音频交互。
  * AVAudioSessionModeSpokenAudio
  * AVAudioSessionModeCategoryOptionInterruptSpokenAudioAndMixWithOthers

对于在 CarPlay 上集成的 App，App icon 将会出现在 CarPlay 主屏幕上，在 CarPlay 中运行的 App 将会在 CarPlay 和 iPhone 屏幕上显示（iOS 13 及更早版本）。

* 音频 App，可以提供音乐、新闻、播客等内容，采用一致的设计来适合在车中使用
* 使用 Siri 短信和语音通话 App 可以经过升级以支持 CarPlay
* 汽车制造商的 App，可以提供对车辆特定的控制和信息显示功能，让驾驶员无需离开 CarPlay 即可使用这些功能。

对于没有在 CarPlay 上集成的 App，不会在 CarPlay 上显示，即使你在 iPhone 中将该 App 置于前台。

**集成**

如果你的 App 需要在 CarPlay 上集成：

* 需要在 [CarPlay 开发者网站](https://developer.apple.com/carplay) 上申请权限。
* 需要提供 2x 和 3x 图以适配 CarPlay 在不同车辆上的分辨率和大小，关于 icon 以及界面设计指南，参考以下：
  * iOS Human Interface Guidelines - App Icon
  * CarPlay Human Interface Guidelines

**Xcode and Simulator**

你可以使用 Xcode 的 CarPlay Simulator 运行和调试 CarPlay App，但为了达到最好的真实的测试结果，建议还是连接真实的设备在车上调试。

* 对于音频 App 的局限性：Simulator 在播放状态下有一些局限，不能反映真实的用户体验
* 为了用 LLDB 全面调试你的 App，Xcode 现在支持无线调试，这样 iPhone 就可以连接汽车，同时调试你的 App。可以观看 [WWDC19 - 使用 Xcode 9 进行调试](https://developer.apple.com/wwdc17/404) 了解更多。

**数据保护**

如果你的 App 对用户数据或内容使用数据保护，要注意 CarPlay App 可能会在 iPhone 锁定密码时运行。如果你的 App 有密码保护的数据，对文件、钥匙串或 SQL 使用了以下的类型，那么你的数据在 CarPlay 中将无法使用，从而导致你的 App 使用异常，

| Files    | FileProtectionType.complete<br />FileProtectionType.completeUnlessOpen |
| -------- | ------------------------------------------------------------ |
| Keychain | kSecAttrAccessibleWhenPasscodeSetThisDeviceOnly<br />kSecAttrAccessibleWhenUnlocked<br />kSecAttrAccessibleWhenUnlockedThisDeviceOnly |
| SQLite   | SQLITE_OPEN_FILEPROTECTION_COMPLETE<br />SQLITE_OPEN_FILEPROTECTION_COMPLETEUNLESSOPEN |

**CarPlay App Integration**

讲解了音频 App、信息和语音通话 App、汽车制造商 App 这三种 App 的集成。因为笔者要开发的是音频 App，所有其它两种就不花时间去研究了。

在 CarPlay 中，音频 App 将内容呈现在一个针对平台进行优化的界面中，为用户提供一种方便的导航方式来访问其内容。

音频 App 的开发者需提供 **数据来源** 和 **导航层次结构** 以便在 CarPlay 中显示。另外，“正在播放” 的信息会显示给用户相关的元数据，以及一些可以调节的操作选项，以便用户设置播放速度、音轨和其它音频 App 可能支持的选项。

![](https://cdn.nlark.com/yuque/0/2021/png/12376889/1630306034624-f9bdcca2-98df-4d77-8135-66e6975f28b4.png?x-oss-process=image%2Fresize%2Cw_750%2Climit_0)

**Requirement for Audio Apps**

* App 在获得 CarPlay 的权限之后，必须至少实现 MPPlayableContent API，包括 `MPPlayableContentDataSource` 和 `MPPlayableContentDelegate`，这样 CarPlay 才能获取内容和开始播放。
* 必须响应 MPRemoteCommandCenter 事件，让用户对你的内容执行命令，如播放、暂停、跳过曲目等等。
* 设置和更新 MPNowPlayingInfoCenter 字典，它包含当前播放音频的元数据，如标题、作者、时长等等。

**音频 App 的数据结构**

CarPlay 会使用 NSIndexPath 在特定的索引路径上请求内容项，这和 UITableView 中使用的 NSIndexPath 是不同的。让我们来看一个示例， 这个例子中最左边的内容是根数据项，在用户界面中可以用单个的窗口（individual tabs）或者是 rootTableView 的形式来体现。

![](https://cdn.nlark.com/yuque/0/2021/png/12376889/1630307422458-6ff255b5-0b3a-490c-8dd0-38b8d5986bf5.png?x-oss-process=image%2Fresize%2Cw_750%2Climit_0)

下面是层次结构中表示的每个内容项的索引路径

![](https://cdn.nlark.com/yuque/0/2021/png/12376889/1630308298929-d31bf075-2607-47f8-a748-40e1d07f4ae1.png?x-oss-process=image%2Fresize%2Cw_750%2Climit_0)

当 CarPlay 根据一个特定的索引路径请求一个内容项时，音频 App 将遍历请求的内容项的层次结构，来查找所需内容。

在这个例子中，我们正在请求第一个内容的第一个子内容，它是正在运行的播放列表内容项。

![](https://cdn.nlark.com/yuque/0/2021/png/12376889/1630308534480-f27e0d05-b52e-4301-9e6b-26792a82cf14.png?x-oss-process=image%2Fresize%2Cw_750%2Climit_0)

CarPlay 还会询问特定索引的子项目数量。在本例中，我们询问第二个内容项的第三个子项的子项数量。在本例中，我们将返回 2。

![](https://cdn.nlark.com/yuque/0/2021/png/12376889/1630308579595-7d3c7e4f-7bce-4504-9422-90d85b5cece2.png?x-oss-process=image%2Fresize%2Cw_750%2Climit_0)

**内容限制**

有些车辆可能会根据车辆是否在行驶而强制在屏幕上显示有限的内容。这可能会限制屏幕上可显示的行数，以及点击进某一个目录后可显示的内容。你的 App 可以利用 MPPlayableContentManager 来处理这些变化。

查询所显示的内容是否会受到限制，实现以下代理方法，判断是否执行了内容限制，以及获取受到内容限制时 列表中显示的项目的最大数目 和 层次结构中的最大深度。

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

![](https://cdn.nlark.com/yuque/0/2021/png/12376889/1630309774681-d69089c3-f02a-4904-b467-2e14f61c450c.png?x-oss-process=image%2Fresize%2Cw_750%2Climit_0)

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

![](https://cdn.nlark.com/yuque/0/2021/png/12376889/1630311745737-506d49af-0864-45c6-8b59-71c84978124d.png?x-oss-process=image%2Fresize%2Cw_750%2Climit_0)

![image.png](https://cdn.nlark.com/yuque/0/2021/png/12376889/1630311760127-50f6f43c-d035-43fd-9d8c-5e11b19f0785.png?x-oss-process=image%2Fresize%2Cw_750%2Climit_0)

**Best Practices**

* 在调用 `MPPlayableContentDataSource` 和 `MPPlayableContentDelegate` 中的 completion handlers 之前，确保将要播放的内容实际上已经准备好播放或显示，在此期间显示加活动指示器。
* 对于那些不用 `Tabs` 而只用 `TableView` 的 App，`TableView` 必须返回至少一项内容。
* CarPlay 会显示一个加载活动指示器，如果 App 在一定时限内没有返回内容就会超时。如果你的 App 需要一些初始的设置，比如登录凭据，在页面的第一行应该提示用户 App 当前的状态，通过填充一个 `MPContentItem`，MPContentItem is neither playable nor a container indicating the state of the app to the user。

**其它** 

信息 App 的开发：

* ......

汽车制造商 App 的开发：

* 汽车制造商 App 可以让用户在不离开 CarPlay 系统的情况下，就可以对汽车进行一些操作，比如调节空调温度等等，

开发者还可以开发自己的 CarPlay App 界面：

* ......