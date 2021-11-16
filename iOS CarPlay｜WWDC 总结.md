## 概览

**WWDC16 - 开发 CarPlay 车载系统 - 第 1 部分**

* **地址**：https://developer.apple.com/wwdc16/722
* **时长**：30 分钟
* **概览**：CarPlay 车载让你能够更智能、更安全地在车内使用 iPhone。了解 CarPlay 车载的工作方式，以及如何设计你的车载信息娱乐系统来与 iPhone 密切协作。了解通过将 CarPlay 车载与车辆原生系统整合来打造出色用户体验的最佳做法。

**WWDC16 - 开发 CarPlay 车载系统 - 第 2 部分**

* **地址**：https://developer.apple.com/wwdc16/723
* **时长**：26 分钟
* **概览**：了解 CarPlay 车载如何与你的车辆信息娱乐系统整合。了解 CarPlay 车载如何与你的车载资源协作，如显示屏、扬声器、麦克风、用户输入、方向盘控制键、仪表盘和传感器等。

**WWDC17 - 开发无线 CarPlay 车载系统**

* **地址**：https://developer.apple.com/wwdc17/717
* **时长**：35 分钟
* **概览**：无论去向哪里，无线 CarPlay 车载都是旅程的绝佳搭档。无需将 iPhone 从包里或口袋中取出，直接开门上车，轻松开始享受 CarPlay 车载体验。学习如何设计你的 CarPlay 车载系统来以无线方式连接至 iPhone。了解相关的硬件要求、提供出色用户体验的最佳做法，以及如何优化配对和重新连接过程。

**WWDC17 - 让您的 App 支持 CarPlay 车载**

* **地址**：https://developer.apple.com/wwdc17/719
* **时长**：28 分钟
* **概览**：了解如何让你的音频、信息、VolP 通话或汽车制造商 App 支持 CarPlay 车载。音频、信息、VolP 通话 App 采用一致的设计，并且为在车内使用进行过优化。汽车制造商 App 提供车辆相关的控制和显示功能，让驾驶员无需离开 CarPlay 车载就能保持互联。探索最佳做法，并了解适用于 CarPlay 车载 App 的工具和框架。

**WWDC18 - CarPlay 车载音频和导航 App**

* **地址**：https://developer.apple.com/wwdc18/213
* **时长**：38 分钟
* **概览**：了解如何更新音频或导航 App 来支持 CarPlay 车载。CarPlay 车载中的 App 针对车用进行了优化，能够自动适应可用的汽车屏幕和输入控制。音频 App 能够输出音乐、新闻、播客等。通过新的 CarPlay 车载框架，导航 App 可以提供详细地图、目的地搜索、逐向导航和用户通知。

**WWDC19 - CarPlay 车载系统改进**

* **地址**：https://developer.apple.com/wwdc19/252
* **时长**：16 分钟
* **概览**：CarPlay 车载让你能够在车内更加智能、安全地使用 iPhone。了解如何更新你的车载系统，以利用 iOS 13 中的新功能。添加对动态变换屏幕尺寸、仪表盘等辅助屏幕，甚至是不规则显示屏的支持。了解如何支持 “嘿 Siri” 来进行免提语音激活。

**WWDC20 - 使用 CarPlay 车载系统为你的 App 提速**

* **地址**：https://developer.apple.com/wwdc20/10635
* **时长**：26 分钟
* **概览**：CarPlay 车载可为用户提供更智能、更安全的 iPhone 车内使用方式。我们将向你展示如何为车辆屏幕打造优质 App，并教你开发电动机车充电、泊车、快速订购食物外卖等类型的 CarPlay 车载 App。此外，我们还会使用现存的音频与通讯 App 作为范例，详细解释如何利用 CarPlay 车载框架的种种改进，制作出更加灵活多变的用户界面。
* **内参**：https://xiaozhuanlan.com/topic/7620814593

## WWDC16 - 开发 CarPlay 车载系统 - 第 1 部分

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

## WWDC16 - 开发 CarPlay 车载系统 - 第 2 部分

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

## WWDC17 - 开发无线 CarPlay 车载系统

该 Session 还是面向汽车制造商，主要内容是如何开发一个无线 CarPlay 车载系统。

* 硬件要求
  * Bluetooth
  * Wi-Fi
  * Location：在无线连接中是必需的，因为 iPhone 可能放包里或口袋里，完全不在视线范围内，为了提高可靠精确的位置信息，车载系统需要由 GNSS（全球卫星导航系统）接收器、车辆速度传感器的接入权限、提供 Dead Reckoning（航位推算）信息的功能。
* ......

## WWDC17 - 让您的 App 支持 CarPlay 车载

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

## WWDC18 - CarPlay 车载音频和导航 App

该 Session 主要内容为以下两点，由于笔者要开发的是 CarPlay 音频 App，所以导航 App 的相关内容就不做研究了。

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

## WWDC19 - CarPlay 车载系统改进

该 Session 主要面向汽车制造商。

* 支持不规则显示屏
* 支持在仪表盘等辅助屏幕上显示一个额外的 CarPlay 用户界面
* 支持动态改变 CarPlay 用户界面的大小
* 支持 “嘿 Siri”
  * 只需用声音即可调用 Siri
  * 只需按一下按钮，Siri 就会立即启动
  * 在与 Siri 互动的同时继续听音乐

**Vehicle System Requirements**

* Always on microphone input stream processed within the vehicle
* Continuous echo cancellation and noise reduction
* Voice activity detector
* Keyword detector
* Audio mixer for Siri audio, media playback, and navigation prompts

## WWDC20 - 使用 CarPlay 车载系统为你的 App 提速

内参：https://xiaozhuanlan.com/topic/7620814593
