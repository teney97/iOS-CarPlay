## iOS CarPlay｜让你的音频 App 支持 CarPlay

从 iOS 14 开始，你可以使用 CarPlay framework 来开发音频 CarPlay App（如果是导航类 App，在 iOS 12 就可以使用），它提供了一些 UI 模版来支持开发者自定义界面；如果你要兼容 iOS 13 及更早版本的话需要使用 MediaPlayer framework 开发，它向前兼容。因此，如果你的 App 需要在 iOS 14 及更高版本上使用 CarPlay framework，并且兼容 iOS 13 及更早版本的话，就要维护两套代码，开发工作量可能接近 double。笔者仅支持了 iOS 14 及更高版本，在本章节中会详细讲解使用 CarPlay framework 的开发细节。如果你想支持低版本的话也可以看看笔者对「WWDC17 - 让您的 App 支持 CarPlay 车载」和「WWDC18 - CarPlay 车载音频和导航 App」做的笔记。

### 申请权限并配置工程

首先，需要确定你的 App 是否适用于 CarPlay，然后去开发者网站申请对应 App 类型的 CarPlay 权限，并对工程进行配置。只有这样你的工程才能使用 CarPlay Simulator，否则你的 CarPlay Simulator 无法打开（灰显禁用）。不过笔者注意到，只要你的 CarPlay Simulator 启用过，即便是不支持 CarPlay 的 App 也是可以打开 CarPlay Simulator 的。因此你可以跑一遍 Apple 的 CarPlay 示例 App [CarPlay Music App](https://developer.apple.com/documentation/carplay/integrating_carplay_with_your_music_app?language=objc) 来启用 CarPlay Simulator。

不过，要使用 CarPlay Simulator 运行和调试你的 CarPlay App，还是得你自己的工程支持才行。

因此，如果你计划要开发 CarPlay App 的话，最好提前去申请权限，因为 Apple 审核还要时间。在此期间可以看看相关开发文档，等权限申请下来并配置好工程就可以使用 CarPlay Simulator 调试开发啦。

参考文档：[申请 CarPlay 权限](https://developer.apple.com/documentation/carplay/requesting_the_carplay_entitlements?language=objc)

### 使用 CarPlay Simulator 运行和调试 CarPlay App

每个 iPhone Simulator 都附带一个 CarPlay Simulator，在 **I/O > External Displays > CarPlay** 打开。默认的标准的 CarPlay Simulator 窗口大小和比例为 `800 x 480, @2x`。

如果是导航类 App，Apple 建议开启 CarPlay Simulator 的附加选项，在终端输入以下命令。这允许你每次启动 CarPlay Simulator 前都可以设置窗口大小和比例，用来测试确保你的地图内容适配了所有推荐的配置。也许只支持导航类 App 吧，音频类 App 改变窗口大小后显示效果不尽如人意，建议关闭。

```
defaults write com.apple.iphonesimulator CarPlayExtraOptions -bool YES
```

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2d693e45a0ca4c1598fa3b71e5706e45~tplv-k3u1fbpfcp-watermark.image?)

如果你打开了 CarPlay Simulator 却没有在主屏幕上看到你的 App，那么你可能是忘了添加对应的权利。你需要将 Key `com.apple.developer.carplay-audio` 添加到 Entitlements.plist 中并设置 Value 为 1。

```xml
<key>com.apple.developer.carplay-audio</key>
<true/>
```

参考文档：[使用 CarPlay Simulator 运行和调试 CarPlay App](https://developer.apple.com/documentation/carplay/using_the_carplay_simulator?language=objc)

如果你是 M1 Mac，那可能无法使用 CarPlay Simulator。如果你的 Xcode 以 Rosetta 模式运行，那么启动 CarPlay App 会直接 crash。将 Simulator 也以 Rosetta 运行并不能解决问题。这个问题暂时没有解决方案。https://issueexplorer.com/issue/mapbox/mapbox-navigation-ios/3355。

### 声明一个 CarPlay scene

在 Info.plist 中声明一个 scene。

```xml
<key>UIApplicationSceneManifest</key>
<dict>
    <key>UIApplicationSupportsMultipleScenes</key>
		<false/>
		<key>UISceneConfigurations</key>
		<dict>
			<key>CPTemplateApplicationSceneSessionRoleApplication</key>
			<array>
			    <dict>
			        <key>UISceneClassName</key>
			        <string>CPTemplateApplicationScene</string>
			        <key>UISceneConfigurationName</key>
			        <string>CarPlaySceneConfiguration</string>
			        <key>UISceneDelegateClassName</key>
			        <string>$(PRODUCT_MODULE_NAME).CarPlaySceneDelegate</string>
			    </dict>
			</array>
    </dict>
</dict>
```

### 实现 CarPlaySceneDelegate

```swift
import CarPlay

class CarPlaySceneDelegate: UIResponder, CPTemplateApplicationSceneDelegate {
    var interfaceController: CPInterfaceController?

    func templateApplicationScene(_ templateApplicationScene: CPTemplateApplicationScene,
            didConnect interfaceController: CPInterfaceController) {

        self.interfaceController = interfaceController
        let item = CPListItem(text: "Rubber Soul", detailText: "The Beatles")
        let section = CPListSection(items: [item])
        let listTemplate = CPListTemplate(title: "Albums", sections: [section])
        interfaceController.setRootTemplate(listTemplate, animated: true)
    }

    func templateApplicationScene(_ templateApplicationScene: CPTemplateApplicationScene,
            didDisconnect interfaceController: CPInterfaceController) {
        self.interfaceController = nil
    }
}
```

CPTemplateApplicationSceneDelegate 协议定义了 CarPlay 在场景连接、断开连接、以及响应某些用户操作的方法。你需要在 CarPlay 启动你的 App 并连接其场景时创建和设置根模板。一般我们实现以下两个方法：

* [- templateApplicationScene:didConnectInterfaceController:](https://developer.apple.com/documentation/carplay/cptemplateapplicationscenedelegate/3578119-templateapplicationscene?language=objc)

  通知代理 CarPlay Scene 已连接。当 App 在车机屏幕上启动时，CarPlay framework 将调用此方法，在该方法中对模板进行初始化。可以看到，系统自动创建了 [CPInterfaceController](https://developer.apple.com/documentation/carplay/cpinterfacecontroller/) 实例（类似 UINavigationController），作为我们 CarPlay App 的入口 controller，我们只需在回调中持有这个实例即可。在以上示例代码中，我们创建了一个列表模板 CPListTemplate（类似 UITableView），其接收多组 CPListSection，section 中包含多个 CPListItem（类似 UITableViewCell）。最后，我们将 CPListTemplate 设置为 App 的根模板（类似 rootViewController）。

* [- templateApplicationScene:didDisconnectInterfaceController:](https://developer.apple.com/documentation/carplay/cptemplateapplicationscenedelegate/3578120-templateapplicationscene?language=objc)

  通知代理 CarPlay Scene 已断开连接。当车机断开连接时，该方法将被调用，可以做一些清理工作。

参考文档：[在你的 CarPlay App 中显示内容](https://developer.apple.com/documentation/carplay/displaying_content_in_carplay?language=objc)

### CarPlay 界面搭建

CarPlay App 界面基本就是由 Template 和 Item 组成，而音频类的 CarPlay App 基本上使用 CPTabBarTemplate、CPListTemplate、CPListImageRowItem、CPListItem 等等即可完成界面的搭建。

| Templates                                                    | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [CPListTemplate](https://developer.apple.com/documentation/carplay/cplisttemplate?language=objc)<br />- [CPListItem](https://developer.apple.com/documentation/carplay/cplistitem?language=objc)<br />- [CPListImageRowItem](https://developer.apple.com/documentation/carplay/cplistimagerowitem?language=objc)<br />- [CPMessageListItem](https://developer.apple.com/documentation/carplay/cpmessagelistitem?language=objc) | 列表模版（类似 UITableView）<br />- 一个通用的、可选择的列表项（对应下图第 2 个）<br />- 显示一系列图像的列表项（对应下图第 3 个）<br />- 表示对话或联系人的列表项（用于通信类 App）（对应下图第 1 个） |
| [CPGridTemplate](https://developer.apple.com/documentation/carplay/cpgridtemplate?language=objc) | 显示和管理 items 网格的模版                                  |
| [CPTabBarTemplate](https://developer.apple.com/documentation/carplay/cptabbartemplate?language=objc) | TabBar 模版（类似 UITabBarController）                       |

![](https://docs-assets.developer.apple.com/published/49959584ba/rendered2x-1619630673.png)

#### CPTabBarTemplate

TabBar 模版，类似 UIKit 中的 UITabBarController。使用一组 CPTemplate 进行初始化，可以将其作为 interfaceController 的 rootTemplate。

```swift
let tabBarTemplate = CPTabBarTemplate(templates: templates)
interfaceController.setRootTemplate(tabBarTemplate, animated: true)
```

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/90920a58c3264fb98908516fffb09449~tplv-k3u1fbpfcp-watermark.image?)

需要注意一下，CPTabBarTemplate 的 templates 是有个数限制的，最大数量通过 [maximumTabCount](https://developer.apple.com/documentation/carplay/cptabbartemplate/3589351-maximumtabcount/) 类属性获取，它的值取决于在 Entitlements.plist 中添加的权利，音频 App 最多添加 4 个，超过数量会 crash。

> 在 [WWDC17 - 让你的 App 支持 CarPlay 车载](https://github.com/teney97/iOS-CarPlay/blob/main/Content/WWDC17%20-%20%E8%AE%A9%E6%82%A8%E7%9A%84%20App%20%E6%94%AF%E6%8C%81%20CarPlay%20%E8%BD%A6%E8%BD%BD.md) 中 Apple 提到过使用 MediaPlayer framework 来构建的 CarPlay App 时，推荐使用最多 4 个 tabs 并且使用较短的标题，因为空间有限并且有些汽车的屏幕比较窄，而且有音频正在播放的时候还需在 rootTemplate 右上角显示 “正在播放” 按钮。

```swift
/**
 The maximum number of tabs that your app may display in a @c CPTabBarTemplate,
 depending on the entitlements that your app declares.

 @warning The system will throw an exception if your app attempts to display more
 than this number of tabs in your tab bar template.
 */
open class var maximumTabCount: Int { get }
```

可以设置每个 template 的 tab 的 tabTitle 和 tabImage，也可以设置 tabSystemItem 用系统样式（可用样式较少，且没办法自定义文案）。如果你没有设置 tabSystemItem 并且 tabImage 为 nil 的话，那么该 tabBarItem 将会使用 UITabBarItem.SystemItem.more。

```swift
// 自定义 tab 样式
listTemplate.tabTitle = "推荐"
listTemplate.tabImage = UIImage(named: "tabbar_recommend")
// 使用系统 tab 样式，如果同时设置了 tabTitle 和 tabImage，那么 tabSystemItem 将不生效
listTemplate.tabSystemItem = .favorites
// 显示红点
listTemplate.showsTabBadge = true
```

这时候就要找 UI 出图了，参考文档：[CarPlay UI 设计指南](https://developer.apple.com/design/human-interface-guidelines/carplay/icons-and-images/custom-icons/)。

CPTabBarTemplate 有个遵循 [CPTabBarTemplateDelegate](https://developer.apple.com/documentation/carplay/cptabbartemplatedelegate/) 协议的 delegate 属性，CPTabBarTemplateDelegate 就一个方法：

```swift
public protocol CPTabBarTemplateDelegate : NSObjectProtocol {
    func tabBarTemplate(_ tabBarTemplate: CPTabBarTemplate, didSelect selectedTemplate: CPTemplate)
}
```

你可以在该方法中刷新 selectedTemplate 数据，如果需要的话。

需要注意一点，与 UITabBarController 的 `- tabBarController:didSelectViewController:` 不同的是：

* 在 CPTabBarTemplate 呈现并默认选中第一个 tab 时，就会调用该代理方法一次，因此你需要注意在该代理方法实现中是否需要过滤掉第一次调用
* 点击当前选中的 tab 也会调用代理方法，因此你也需要注意下是否过滤这一情况

#### CPListTemplate

列表模版，类似 UITableview。可以使用一组 CPListTemplate 来初始化 CPTabBarTemplate。CPListTemplate 由遵循 [CPListTemplateItem](https://developer.apple.com/documentation/carplay/cplisttemplateitem/) 协议的 item 组成，item 类似 UITableviewCell。一般音频 App 就使用 CPListItem 和 CPListImageRowItem。

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c284c0486f8947dd82d8334b1b6c10b0~tplv-k3u1fbpfcp-watermark.image?)

CPListTemplate 有个遵循[CPListTemplateDelegate](https://developer.apple.com/documentation/carplay/cplisttemplatedelegate/) 协议的 delegate 属性，CPListTemplateDelegate 就一个方法，在用户点击 item 时触发，我们可以在该方法实现中 push 其它 Template。

```swift
protocol CPListTemplateDelegate : NSObjectProtocol {
    func listTemplate(_ listTemplate: CPListTemplate, didSelect item: CPListItem, completionHandler: @escaping () -> Void)
}
```

注意这里有个 completionHandler 参数。当 `- listTemplate:didSelectListItem:completionHandler:` 方法被调用，在 completionHandler 调用之前，didSelectListItem 上会在右边显示一个 loading 活动指示器。最佳实践是，在要播放的内容已经准备好，或者页面跳转完成时（`- pushTemplate:animated:completion:` 的 completion 中）调用。当然，你也要保证 completionHandler 被调用，比如提前退出时，否则活动指示器会一直存在。

#### CPListImageRowItem

可以用来展示由一组专辑组成的模块。使用文本和一组图片初始化，文本可以展示模块名称，图片可以展示模块里前 n 个专辑的封面。

```swift
// 遵循协议：CPListImageRowItem: CPSelectableListItem: CPListTemplateItem
let imagesCount = CPMaximumNumberOfGridImages // 取决于汽车显示屏的可用宽度
let images = Array(repeating: image, count: imagesCount)
let listImageRowItem = CPListImageRowItem(text: text, images: images)
```

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0c4ef7bb3d374843b3daf25ae649fb4f~tplv-k3u1fbpfcp-watermark.image?)

CPListImageRowItem 的点击区域可以分为每张图片区域、图片以外的所有区域。

每张图片的点击回调通过设置 CPListImageRowItem 实例的 listImageRowHandler 属性来监听。点击可以 push 到该专辑的音频列表页面。

```swift
var listImageRowHandler: ((CPListImageRowItem, Int, @escaping () -> Void) -> Void)? // The image row item that the user selected.
```

图片以外的区域的点击回调走的是 CPListTemplateDelegate 的方法。点击可以 push 到该模块的专辑列表页面。

```swift
protocol CPListTemplateDelegate : NSObjectProtocol {
    func listTemplate(_ listTemplate: CPListTemplate, didSelect item: CPListItem, completionHandler: @escaping () -> Void)
}
```

#### CPListItem

可以用来展示专辑或音频，或者其它的 item 如 “正在加载中”、“还没有播放记录”、”播放全部“ 等等。

对于音频 item，一般由音频封面、音频名称、音频描述组成，可以使用以下构造器初始化 CPListItem。

```swift
let listItem = CPListItem(text: audioName, 
                          detailText: audioDesc, 
                          image: audioCover)
```

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/125d842edab74a5d97cddbe2edbd2357~tplv-k3u1fbpfcp-watermark.image?)

对于专辑 item，还需要右边的导航箭头来和音频 item 做区分，指示用户点击可以打开音频列表页面，可以使用以下构造器初始化 CPListItem。

```swift
let listItem = CPListItem(text: albumName, 
                          detailText: albumDesc, 
                          image: albumCover, 
                          accessoryImage: nil, 
                          accessoryType: .disclosureIndicator)
```

右边的图标有两个系统样式（箭头、云朵），同时也支持自定义。

```swift
enum CPListItemAccessoryType : Int {
    case none = 0
    case disclosureIndicator = 1
    case cloud = 2
}
```

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1d384991a82f4deba5f3fe53d9fc2f56~tplv-k3u1fbpfcp-watermark.image?)

需要注意一点，如果 CPListItem 的 detailText 为 nil 的话，view 高度会减小，并将 text 居中显示，如下图所示，这会影响美观以及整体统一性。因此，你可以考虑当音频没有副标题时，使用音频时长或其它数据填充 detailText。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0aaab77e2bc146649b32fe54b06b4c2b~tplv-k3u1fbpfcp-watermark.image?)

#### CPNowPlayingTemplate

音频播放页是音频类 CarPlay App 最重要的页面了，使用 [CPNowPlayingTemplate](https://developer.apple.com/documentation/carplay/cpnowplayingtemplate/)，它是一个单例。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/95c7eb8bdac4467396a7c9dce25a11a8~tplv-k3u1fbpfcp-watermark.image?)

你可以根据自己的需求配置 CPNowPlayingTemplate，比如添加控制按钮。你可以使用 CarPlay framework 提供的一些系统按钮，也可以自定义按钮。需要注意的是，你要在 `- templateApplicationScene:didConnectInterfaceController:` 的时机就配置好 CPNowPlayingTemplate，而不应该在 push 到 CPNowPlayingTemplate 的时候才去配置，因为 CPNowPlayingTemplate 并不一定是通过主动 push 时触发，还可能是通过 “播放中” App 或者 rootTemplate 右上角的 “正在播放按钮”。

```swift
let nowPlayingTemplate = CPNowPlayingTemplate.shared
let repeatButton = CPNowPlayingRepeatButton() { ... }
let playbackRateButton = CPNowPlayingPlaybackRateButton() { ... }
nowPlayingTemplate.updateNowPlayingButtons([repeatButton, playbackRateButton])
```

无论你是使用 CarPlay framework 还是 MediaPlayer framework 来构建的 CarPlay App，都是通过 [MPNowPlayingInfoCenter](https://developer.apple.com/documentation/mediaplayer/mpnowplayinginfocenter/) 和 [MPRemoteCommandCenter](https://developer.apple.com/documentation/mediaplayer/mpremotecommandcenter/) 来提供播放界面的音频信息以及响应远程播放控制事件。只不过在 CarPlay framework 中，一些远程控制事件通过 [CPNowPlayingButton](https://developer.apple.com/documentation/carplay/cpnowplayingbutton/) 的 handler 来处理了，比如播放模式、播放速率等等。当然如果你的 App 是音频类的话，应该已经支持了这些功能，因为 iPhone 锁屏界面以及控制中心的音频播放信息和播放控制也是通过它们提供。因此，我们只需要针对 CarPlay 做下优化或者功能增强就行。我们具体要做的是：

* 设置和更新 MPNowPlayingInfoCenter 的 [nowPlayingInfo](https://developer.apple.com/documentation/mediaplayer/mpnowplayinginfocenter/1615903-nowplayinginfo)，它包含当前播放音频的元数据，如标题、作者、时长等等。时机：

  * 切换音频（上一首、下一首等等）
  * 暂停、恢复、停止播放
  * seek（跳过片头、拖动进度等等）
  * 更新播放速度（CPNowPlayingTemplate 中播放速度按钮的显示状态）。如果当前音频不在播放状态，需将播放速度设置为 0
  * ...

```swift
import MediaPlayer
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

需要注意一点，对于播放进度也就是当前音频已播放时长的更新，系统会根据先前提供的**已播放时长**和**播放速度**自动推断出来。因此不需要也不推荐频繁更新 nowPlayingInfo，这代价很大。

* 除了 nowPlayingInfo，还有一些状态需要通过其它方式同步到 CarPlay App，比如：

  * 音频播放状态：暂停/播放（CPNowPlayingTemplate 中播放按钮的显示状态）

  ```objectivec
  typedef NS_ENUM(NSUInteger, MPNowPlayingPlaybackState) {
      MPNowPlayingPlaybackStateUnknown = 0,
      MPNowPlayingPlaybackStatePlaying,
      MPNowPlayingPlaybackStatePaused,
      MPNowPlayingPlaybackStateStopped,
      MPNowPlayingPlaybackStateInterrupted
  };
  
  if (@available(iOS 13.0, *)) {
      MPNowPlayingInfoCenter.defaultCenter.playbackState = MPNowPlayingPlaybackStatePlaying; 
  }
  ```
  
  * 播放模式状态：顺序循环/单曲循环（CPNowPlayingTemplate 中播放模式按钮的显示状态）
  
  ```objectivec
  typedef NS_ENUM(NSInteger, MPRepeatType) {
      MPRepeatTypeOff,    /// Nothing is repeated during playback.
      MPRepeatTypeOne,    /// Repeat a single item indefinitely.
      MPRepeatTypeAll,    /// Repeat the current container or playlist indefinitely.
  };
  
  MPRemoteCommandCenter.sharedCommandCenter.changeRepeatModeCommand.currentRepeatType = MPRepeatTypeOne;
  ```

下图中的音频封面、名称、描述、时长、当前播放进度、播放模式、播放速度等音频播放信息，都是通过以上方式同步显示到 CPNowPlayingTemplate 上的。你的音频 App 之前应该已经实现了该功能以将音频播放信息同步到 iPhone 锁屏界面以及控制中心，现在只需要检查下 CarPlay App 的 CPNowPlayingTemplate 中显示的音频播放信息是否准确即可。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e90b21bcb08844f28653fec9258721c3~tplv-k3u1fbpfcp-watermark.image?)

* 响应 MPRemoteCommandCenter 事件，对远程播放控制事件做出响应，如播放、暂停、切换歌曲等等。

  * playCommand
  * pauseCommand
  * previousTrackCommand
  * nextTrackCommand
  * togglePlayPauseCommand
  * changeRepeatModeCommand
  * changePlaybackRateCommand
  * ...
* 使用 CarPlay framework 时，changeRepeatModeCommand、changePlaybackRateCommand 不再由 MPRemoteCommandCenter 触发，而是通过 [CPNowPlayingRepeatButton](https://developer.apple.com/documentation/carplay/cpnowplayingrepeatbutton/)、[CPNowPlayingPlaybackRateButton](https://developer.apple.com/documentation/carplay/cpnowplayingplaybackratebutton) 的 handler 处理，但 command.enabled 还是要开启。例如：
  * 当 command.enabled 为 true 时，用户点击了播放模式按钮，CPNowPlayingRepeatButton 的 handler 就会被触发，然后你可以更新 App 播放模式，并将播放模式状态通过上述方式同步到 CarPlay。如果你的 App 还支持随机播放模式，可以自定义一个 CPNowPlayingButton，配合 CPNowPlayingRepeatButton 完成 3 种模式的切换。
  * 由于只能得知用户点击了按钮，而不知道用户点击按钮的具体意图，因此 CPNowPlayingPlaybackRateButton handler 的最佳实践是，设定一个播放速度范围，当用户点击时，增加 App 播放速度，并通过更新 nowPlayingInfo 同步到 CarPlay。如果当前音频正在以最快的支持速度播放，那么继续增加播放速度就将其调到最小速度，以此循环。

### 最佳实践

#### 使用 userInfo 存储数据

CPListItem、CPListImageRowItem 都有个 [userInfo](https://developer.apple.com/documentation/carplay/cplistitem/2977574-userinfo?language=objc) 属性，它用来存储数据。`CPListItem -> userInfo` 关系就类似于 `UITableViewCell -> model`。

```swift
// Use this property to attach a value that provides additional context to the list item. For example, you can attach a model object and reference it in the list item’s handler when processing the selection.
var userInfo: Any?
```

#### 通过 isEnabled 设置 item 的可交互性（iOS 15）

CPListItem、CPListImageRowItem 都有个 [isEnabled](https://developer.apple.com/documentation/carplay/cplistitem/3751895-enabled?language=objc) 属性，它用来设置 item 的可交互性（默认值为 true）。isEnabled 设置为 false 的 item 将灰显且不可点击，也就是不会触发 CPListTemplateDelegate 的 `- listTemplate:didSelectListItem:completionHandler:` 方法。最佳实践是，将 “还没有播放记录”、“正在加载中” 这些本身就没有交互的 item 的 isEnabled 设置为 false，这样呈现的 UI 效果更好，而且你也不用在 CPListTemplateDelegate 的方法中对这些 item 做 guard 处理了。不过该 API 在 iOS 15 开始才支持 😭 ，但我们还有其它方式可以避免在 CPListTemplateDelegate 的方法中做 guard 处理，那就是用 item 的 handler 属性。

```swift
// A Boolean value that indicates if the item is enabled.
@available(iOS 15.0, *)
var isEnabled: Bool
```

#### 使用 handle 响应 item 的点击事件

CPListItem、CPListImageRowItem 都有个 [handler](https://developer.apple.com/documentation/carplay/cplistitem/3667716-handler?language=objc) 属性，用来响应 item 的点击事件。如果你给 item 设置了 handler，那么点击 item 将触发 handler 而不触发 CPListTemplateDelegate 的方法。

```swift
// An optional action block, fired when the user selects this item in a list template.
var handler: ((CPSelectableListItem, @escaping () -> Void) -> Void)?
```

我们可以利用 handler 把一些特殊 item 的点击事件剥离出来，而不是全部放到 CPListTemplateDelegate 的 `- listTemplate:didSelectListItem:completionHandler:` 中处理，这样可以**提高代码可维护性**。对于上面说的 isEnabled 仅在 iOS 15 以上才支持的问题，我们也可以像下面这样处理，以避免在 CPListTemplateDelegate 的方法中做 guard 处理。

```swift
let item = CPListItem(text: "正在加载中", detailText: nil)
if #available(iOS 15.0, *) {
    item.isEnabled = false
} else {
    item.handler = { (item, completionHandler) in
        completionHandler()
    }
}
```

#### 通过 isPlaying 属性来显示正在播放的指示器

使用 CPListItem 来展示音频，你还可以通过 [isPlaying](https://developer.apple.com/documentation/carplay/cplistitem/3551780-isplaying) 属性来显示正在播放的指示器。

```swift
var isPlaying: Bool
```

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/33d26d619a764d15a0449063b6c531b4~tplv-k3u1fbpfcp-watermark.image?)

指示器的位置默认在左边，隐藏了 image。你还可以通过 [playingIndicatorLocation](https://developer.apple.com/documentation/carplay/cplistitem/3566414-playingindicatorlocation?language=objc) 属性设置指示器位置为右边。

```swift
enum CPListItemPlayingIndicatorLocation : Int {
    case leading = 0
    case trailing = 1
}

var playingIndicatorLocation: CPListItemPlayingIndicatorLocation
```

### 页面跳转

还记得在 CarPlay App 入口 [- templateApplicationScene:didConnectInterfaceController:](https://developer.apple.com/documentation/carplay/cptemplateapplicationscenedelegate/3578119-templateapplicationscene?language=objc) 中的 [CPInterfaceController](https://developer.apple.com/documentation/carplay/cpinterfacecontroller/) 吗，它作为我们 CarPlay App 的入口 controller，我们将一个 template 作为 rootTemplate 赋值给它。当我们要进行页面跳转时也是靠它，有点类似于 UINavigationController，它支持 push、pop、present、dismiss 等等操作（present、dismiss 操作仅 CPActionSheetTemplate、CPVoiceControlTemplate、CPAlertTemplate）。对于音频 App，一般 push 操作就够用，子页面的左上角都自带返回按钮的。

### 代码设计

可参考：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2f1c43bf92714e8e8dde65bd14dc3089~tplv-k3u1fbpfcp-watermark.image?)

### 图片

#### 图标和图片

可以看看 [CarPlay - 设计指南](https://developer.apple.com/design/human-interface-guidelines/carplay/overview/introduction/)，并将它发给你的 PM 和 UI。

#### 深色/浅色模式

适配方案和 iPhone App 一样，如果你的 App 需要两种风格的话就支持一下。

#### 异步图片

* CarPlay 不支持 GIF 图片，配置会导致 crash。笔者尝试取出 GIF 图的第一帧，发现还是不支持，或许是我使用方式不对。
* asyncImage 也需要适配下 scale，否则会模糊。
* 可以对 CPListItem 和 CPListImageRowItem 扩展下 asyncImage 的方法，方便使用。

```swift
@available(iOS 14, *)
protocol CPAsyncImage {
    func loadImage(with url: URL?, complete: @escaping (UIImage?) -> (Void))
    static var placeholderImage: UIImage { get }
}

@available(iOS 14, *)
extension CPAsyncImage {
    
    func loadImage(with url: URL?, complete: @escaping (UIImage?) -> (Void)) {
        
        guard let url = url else {
            complete(nil)
            return
        }
        
        // 这儿也可以根据 urlType 做下过滤
        
        SDWebImageManager.shared.loadImage(with: url, options: .retryFailed, progress: nil) { image, data, error, type, finished, imageURL in
            
            guard var image = image, image.images == nil else {
                complete(nil)
                return
            }
                   
            if let cgImage = image.cgImage {
                let screen = TTScene.carPlay?.value(forKey: "screen") as? UIScreen
                let scale = screen?.scale ?? 1
                image = UIImage(cgImage: cgImage, scale: scale, orientation: .up)
            }
            complete(image)
        }
    }
    
    static var placeholderImage: UIImage {
        UIImage(named: "CP_default") ?? UIImage()
    }
}

@available(iOS 14, *)
extension CPListItem: CPAsyncImage {
    
    func asyncImage(with url: URL?, placeholderImage: UIImage? = CPListItem.placeholderImage, complete: ((UIImage?) -> (Void))? = nil) {
        
        self.setImage(placeholderImage)
        loadImage(with: url) { [weak self] image in
            guard let image = image else { return }
            self?.setImage(image)
            complete?(image)
        }
    }
}

@available(iOS 14, *)
extension CPListImageRowItem: CPAsyncImage {
    
    func asyncImage(with urls: [URL?], placeholderImage: UIImage = CPListImageRowItem.placeholderImage, complete: (((index: Int, image: UIImage?)) -> (Void))? = nil) {
        
        var images = Array(repeating: placeholderImage, count: urls.count)
        self.update(images)
        
        for (index, url) in urls.enumerated() {
            loadImage(with: url) { [weak self] image in
                guard let image = image else { return }
                images[index] = image
                self?.update(images)
                complete?((index, image))
            }
        }
    }
}
```

### 重新加载数据

网络不好的情况下启动 CarPlay App，可能就会存在请求超时，CarPlay App 中无数据。处理方式有以下几种：

1. 不重新加载。如果是首页，则用户需要重新启动 CarPlay App 才能重新加载；如果是子页面，则用户需要退出并重新进入子页面才能重新加载。看了网易云 CarPlay App 就是这样处理的，不过它的首页有固定的几个 item，倒是不影响体验。如果你的首页内容没有本地 item，那不建议以这种方式处理；
2. 如果 rootTemplate 是 CPTabBarTemplate，那么你可以在 `- tabBarTemplate:didSelectTemplate:` 时机对 selectedTemplate 进行重新加载操作；
3. 对于第二种方式，重新加载对用户来说是无感知的，因为你没办法或者不方便自己添加个活动指示器。我的方案是，在请求数据的过程中，添加个 loading item（比如使用 CPListItem 并显示 “正在加载中”）。如果请求失败，则更新 item 为 failure item（比如使用 CPListItem 并显示 “加载失败，点击重试”）。当用户点击 failure item 时，更新 item 为 loading item，并重新请求数据。对于每个需要从服务端拉取数据的页面都可以这样处理。

是否需要刷新？如果想要允许使用 CarPlay App 的过程中去刷新数据，那么可以通过第二种方式处理。不过，用户单次使用 CarPlay 的时间不会很长，没有必要做刷新，下次启动的时间拉取新数据就好。因此，我只针对首次数据加载失败的情况做了重新加载处理。

### 关注弱网以及无网环境下的用户体验

在 WWDC 或相关文档中，Apple 多次提到要关注弱网以及无网环境下的用户体验，因为驾驶过程中可能会经过网络不好的路段或区域。例如上面提到的 ”请求超时，重新加载数据“ 问题、CPNowPlayingTemplate 中数据同步以及播放控制事件是否出现异常等等。

### 埋点

一些埋点可能需要通过投机取巧的方法。比如在哪个页面触发了返回按钮，可以通过 `- templateDidAppear`、`- templateWillDisappear` 等方法配合实现。emmm... 加了埋点后代码一点儿也不简洁了。

### 测试

在真实环境（汽车中）测试。[Apple｜使用 CarPlay Simulator 运行和调试 CarPlay App](https://developer.apple.com/documentation/carplay/using_the_carplay_simulator?language=objc) 中列举了一些在 CarPlay Simulator 上无法测试的功能。

对于音频 App，Simulator 在播放状态下有一些局限，不能反映真实的用户体验。

为了用 LLDB 全面调试你的 App，Xcode 9 开始支持无线调试，这样就可以在 iPhone 连接汽车的同时调试你的 App。可以观看 [WWDC19 - 使用 Xcode 9 进行调试](https://developer.apple.com/wwdc17/404) 了解更多。

## 注意点 & 最佳实践

* 单例问题。CarPlay 用到了单例类，CarPlay App 关闭，但 iPhone App 没关闭，进程是还在的，单例还未释放，可能会造成一些问题。可以在 `- templateApplicationScene:didConnectInterfaceController:` 中初始化单例，在 `- templateApplicationScene:didDisconnectInterfaceController:` 中释放单例。
* CarPlay 的语言是跟随 iPhone 的，Simulator 也是如此。
* CarPlay framework 需要设置为弱引用 optional（Target > Build phases > Link Binary With Libraries），否则在 iOS 12 以下启动 App 会 crash。
* CarPlay 断开连接时，建议暂停音乐。
* CarPlay 断开连接时，可以通过 Memory Graph 检查下有无内存泄漏。
* Template 页面最好至少显示一项内容，特别是你没有使用 CPTabBarTemplate 作为 rootTemplate 的情况，否则页面将一片空白，影响用户体验。例如，在最近播放页面，当没有播放记录时，填充一个 CPListItem 并显示 “当前没有播放记录”。
* 你需要注意一些情况，比如 iPhone 锁屏、用户未登录时，你的 App 仍然需要在 CarPlay 中展示完美的功能性。
