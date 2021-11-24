## 开发 CarPlay 音频 App

从 iOS 14 开始，你可以使用 CarPlay framework 来开发音频 CarPlay app（如果是导航类 app，在 iOS 12 就可以使用 CarPlay Framework），它提供了一些 UI 模版来支持开发者自定义界面；如果你要兼容 iOS 13 及更早版本的话需要使用 MediaPlayer framework 开发，它向前兼容。如果你的 app 需要在 iOS 14 及更高版本上使用 CarPlay framework，并且兼容 iOS 13 级更早版本的话，就要维护两套代码，开发工作量可能接近 double。笔者仅支持了 iOS 14 及更高版本，因此在下篇文章中会详细讲解开发细节。如果你想支持低版本的话也可以看看笔者对「WWDC17 - 让您的 App 支持 CarPlay 车载」和「WWDC18 - CarPlay 车载音频和导航 App」做的笔记。

### 申请权限并配置工程

首先，需要确定你的 app 是否适用于 CarPlay，然后去开发者网站申请对应 app 类型的 CarPlay 权限，并对工程进行配置。只有这样你的工程才能使用 CarPlay Simulator，否则你的 CarPlay Simulator 无法打开（灰显禁用）。不过笔者注意到，只要你的 CarPlay Simulator 启用过，即便是不支持 CarPlay 的 app 也是可以打开 CarPlay Simulator 的。因此你可以跑一遍 Apple 的 CarPlay 示例 app [CarPlay Music App](https://developer.apple.com/documentation/carplay/integrating_carplay_with_your_music_app?language=objc) 来启用 CarPlay Simulator。

不过，要使用 CarPlay Simulator 运行和调试你的 CarPlay App，还是得你自己的工程支持才行。

因此，如果你计划要开发 CarPlay app 的话，最好提前去申请权限，因为 Apple 审核还要时间。在此期间可以看看相关开发文档，等权限申请下来并配置好工程就可以使用 CarPlay Simulator 调试开发啦。

参考文档：[申请 CarPlay 权限](https://developer.apple.com/documentation/carplay/requesting_the_carplay_entitlements?language=objc)

### 使用 CarPlay Simulator 运行和调试 CarPlay App

每个 iPhone Simulator 都附带一个 CarPlay Simulator，在 **I/O > External Displays > CarPlay** 打开。默认的标准的 CarPlay Simulator 窗口大小和比例为 `800 x 480, @2x`。

如果是导航类 app，Apple 建议开启 CarPlay Simulator 的附加选项，在终端输入以下命令。这允许你每次启动 CarPlay Simulator 前都可以设置窗口大小和比例，以测试确保你的地图内容适配了所有推荐的配置。也许只支持导航类 app 吧，音频类 app 改变窗口大小后显示效果不尽如人意，建议关闭。

```
defaults write com.apple.iphonesimulator CarPlayExtraOptions -bool YES
```

![image-20211109154924029](/Users/chenjunteng/Library/Application Support/typora-user-images/image-20211109154924029.png)

如果你打开了 CarPlay Simulator 却没有在主屏幕上看到你的 app，那么你可能是忘了添加对应的权利。你需要将 Key `com.apple.developer.carplay-audio` 添加到 Entitlements.plist 中并设置 Value 为 `1`。

```xml
<key>com.apple.developer.carplay-audio</key>
<true/>
```

参考文档：[使用 CarPlay Simulator 运行和调试 CarPlay App](https://developer.apple.com/documentation/carplay/using_the_carplay_simulator?language=objc)

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

CPTemplateApplicationSceneDelegate 协议定义了 CarPlay 在场景连接、断开连接、以及响应某些用户操作的方法。你需要在 CarPlay 启动你的 app 并连接其场景时创建和设置根模板。一般我们实现以下两个方法：

* [- templateApplicationScene:didConnectInterfaceController:](https://developer.apple.com/documentation/carplay/cptemplateapplicationscenedelegate/3578119-templateapplicationscene?language=objc)

  通知代理 CarPlay Scene 已连接。当 app 在车机屏幕上启动时，CarPlay framework 将调用此方法，在该方法中对模板进行初始化。可以看到，系统自动创建了 CPInterfaceController 实例（类似 UINavigationController），作为我们 CarPlay app 的入口 controller，我们只需在回调中持有这个实例即可。然后我们创建了一个列表模板 CPListTemplate（类似 UITableView），其接收多组 CPListSection，section 中包含多个 CPListItem（类似 UITableViewCell），这是一个最基本的列表元素。在这里，我们将 CPListTemplate 设置为 app 的根模板（类似 rootViewController）。

* [- templateApplicationScene:didDisconnectInterfaceController:](https://developer.apple.com/documentation/carplay/cptemplateapplicationscenedelegate/3578120-templateapplicationscene?language=objc)

  通知代理 CarPlay Scene 已断开连接。当车机断开连接时，该方法将被调用，在里面可以做一些清理工作。

参考文档：[在你的 CarPlay app 中显示内容](https://developer.apple.com/documentation/carplay/displaying_content_in_carplay?language=objc)

### CarPlay 界面搭建

CarPlay app 界面基本就是由 Template 和 Item 组成，而音频类的 CarPlay app 基本上使用 CPTabBarTemplate、CPListTemplate、CPListImageRowItem、CPListItem 等等即可完成界面的搭建。

| Templates                                                    | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [CPListTemplate](https://developer.apple.com/documentation/carplay/cplisttemplate?language=objc)<br />- [CPListItem](https://developer.apple.com/documentation/carplay/cplistitem?language=objc)<br />- [CPListImageRowItem](https://developer.apple.com/documentation/carplay/cplistimagerowitem?language=objc)<br />- [CPMessageListItem](https://developer.apple.com/documentation/carplay/cpmessagelistitem?language=objc) | 列表模版（类似 UITableView）<br />- 一个通用的、可选择的列表项（对应下图第 2 个）<br />- 显示一系列图像的列表项（对应下图第 3 个）<br />- 表示对话或联系人的列表项（用于信息类 App）（对应下图第 1 个） |
| [CPGridTemplate](https://developer.apple.com/documentation/carplay/cpgridtemplate?language=objc) | 显示和管理 items 网格的模版                                  |
| [CPTabBarTemplate](https://developer.apple.com/documentation/carplay/cptabbartemplate?language=objc) | TabBar 模版（类似 UITabBarController）                       |

![](https://docs-assets.developer.apple.com/published/49959584ba/rendered2x-1619630673.png)

#### CPTabBarTemplate

TabBar 模版，类似 UIKit 中的 UITabBarController。使用一组 CPTemplate 进行初始化，可以将其作为 interfaceController 的 rootTemplate。

```swift
let tabBarTemplate = CPTabBarTemplate(templates: templates)
interfaceController.setRootTemplate(tabBarTemplate, animated: true)
```

![image-20211123143752733](/Users/chenjunteng/Library/Application Support/typora-user-images/image-20211123143752733.png)

需要注意一下，CPTabBarTemplate 的 templates 是有个数限制的，最大数量通过 maximumTabCount 类属性获取，它的值取决于在 Entitlements.plist 中添加的权利，音频 app 最多添加 4 个，超过数量会 crash。

> 在 [WWDC17 - 让你的 App 支持 CarPlay 车载](https://github.com/teney97/iOS-CarPlay/blob/main/Content/WWDC17%20-%20%E8%AE%A9%E6%82%A8%E7%9A%84%20App%20%E6%94%AF%E6%8C%81%20CarPlay%20%E8%BD%A6%E8%BD%BD.md) 中 Apple 提到过使用 MediaPlayer framework 来构建的 CarPlay app 时，推荐使用最多 4 个 tabs 并且使用较短的标题，因为空间有限并且有些汽车的屏幕比较窄，而且有音频正在播放的时候还需在 rootTemplate 右上角显示 “正在播放” 按钮。

```swift
/**
 The maximum number of tabs that your app may display in a @c CPTabBarTemplate,
 depending on the entitlements that your app declares.

 @warning The system will throw an exception if your app attempts to display more
 than this number of tabs in your tab bar template.
 */
open class var maximumTabCount: Int { get }
```

可以设置每个 template 的 tab 的 tabTitle 和 tabImage，也可以设置 tabSystemItem 用系统样式（可用样式较少，且没办法自定义文案了）。

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

CPTabBarTemplateDelegate 代理方法就一个：

```swift
public protocol CPTabBarTemplateDelegate : NSObjectProtocol {
    func tabBarTemplate(_ tabBarTemplate: CPTabBarTemplate, didSelect selectedTemplate: CPTemplate)
}
```

需要注意一点，与 UITabBarController 的 `- tabBarController:didSelectViewController:` 不同的是：

* 在 CPTabBarTemplate 呈现并默认选中第一个 tab 时，就会调用该代理方法一次，因此你需要注意在该代理方法实现中是否需要过滤掉第一次调用
* 点击当前选中的 tab 也会调用代理方法，因此你也需要注意下是否过滤这一情况

#### CPListTemplate

列表模版，有点类似 UITableview。可以使用一组 CPListTemplate 来初始化 CPTabBarTemplate。CPListTemplate 由遵循 CPListTemplateItem 协议的 item 组成，item 有点类似 UITableviewCell。一般音频 App 就使用 CPListItem 和 CPListImageRowItem。

![image-20211123153134163](/Users/chenjunteng/Library/Application Support/typora-user-images/image-20211123153134163.png)

CPListTemplate 有个遵循 CPListTemplateDelegate 协议的 delegate 属性，CPListTemplateDelegate 就一个方法，在用户点击 item 时触发。

```swift
protocol CPListTemplateDelegate : NSObjectProtocol {
    func listTemplate(_ listTemplate: CPListTemplate, didSelect item: CPListItem, completionHandler: @escaping () -> Void)
}
```

注意这里有个 completionHandler 参数。当 `- listTemplate:didSelectListItem:completionHandler:` 方法被调用，在 completionHandler 调用之前，didSelectListItem 上会在右边显示一个活动指示器。最佳实践是，

在调用 `MPPlayableContentDataSource` 和 `MPPlayableContentDelegate` 中的 completion handlers 之前，确保将要播放的内容实际上已经准备好播放或显示，在此期间显示加活动指示器。

#### CPListImageRowItem

可以用来展示由一组专辑组成的模块。使用文本和一组图片初始化，文本可以展示模块名称，图片可以展示模块里前 n 个专辑的封面。

```swift
// 继承链 CPListImageRowItem: CPSelectableListItem: CPListTemplateItem
let imagesCount = CPMaximumNumberOfGridImages // 取决于汽车显示屏的可用宽度
let images = Array(repeating: image, count: imagesCount)
let listImageRowItem = CPListImageRowItem(text: text, images: images)
```

![image-20211123153329881](/Users/chenjunteng/Library/Application Support/typora-user-images/image-20211123153329881.png)

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

![image-20211123143752733](/Users/chenjunteng/Library/Application Support/typora-user-images/image-20211123143752733.png)

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

![image-20211123154215694](/Users/chenjunteng/Library/Application Support/typora-user-images/image-20211123154215694.png)

需要注意一点，如果 CPListItem 的 detailText 为 nil 的话，view 高度会减小，并将 text 居中显示，如下图所示，这会影响美观以及整体统一性。因此，你可以考虑当音频没有副标题时，使用音频时长或其它数据填充 detailText。

![image-20211124091144806](/Users/chenjunteng/Library/Application Support/typora-user-images/image-20211124091144806.png)

#### CPNowPlayingTemplate

这是音频类 CarPlay app 最重要的页面了，使用 CPNowPlayingTemplate，它是一个单例。

你可以根据自己的需求配置 CPNowPlayingTemplate，比如添加控制按钮。你可以使用 CarPlay framework 提供的一些系统按钮，也可以自定义按钮。需要注意的是，你要在 `- templateApplicationScene:didConnectInterfaceController:` 的时机就配置好 CPNowPlayingTemplate，而不应该在 push 到 CPNowPlayingTemplate 的时候才去配置，因为 CPNowPlayingTemplate 并不一定是通过主动 push 时触发，还可能是通过 “播放中” app 或者 rootTemplate 右上角的 “正在播放按钮”。

```swift
let nowPlayingTemplate = CPNowPlayingTemplate.shared
let repeatButton = CPNowPlayingRepeatButton() { ... }
let playbackRateButton = CPNowPlayingPlaybackRateButton() { ... }
nowPlayingTemplate.updateNowPlayingButtons([repeatButton, playbackRateButton])
```

无论你使用 CarPlay framework 还是 MediaPlayer framework 来构建的 CarPlay app，都是通过 MPNowPlayingInfoCenter 和 MPRemoteCommandCenter 来提供播放界面的音频信息以及响应播放控制事件。只不过在 CarPlay framework 中，一些 remote command 事件通过 button handle 来处理了，比如播放模式、播放速率等等。当然如果你的 app 是音频类的话，应该已经支持了这些功能，因为 iPhone 锁屏界面以及控制中心的音频播放信息和播放控制也是通过它们提供。因此，我们只需要针对 CarPlay 做下优化或者功能增强就行。我们具体要做的是：

* 设置和更新 MPNowPlayingInfoCenter 的 nowPlayingInfo，它包含当前播放音频的元数据，如标题、作者、时长等等。时机：

  * 切换音频（上一首、下一首等等）
  * 暂停、恢复、停止播放
  * seek（跳过片头、拖动进度等等）
  * 更新播放速率（CPNowPlayingTemplate 中播放速率按钮的显示状态）
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

需要注意一点，对于播放进度也就是当前音频已经播放的时长的更新，系统会根据先前提供的已播放时长和播放速率自动推断出来。因此不需要也不推荐频繁更新 nowPlayingInfo，这代价很大。

* 除了 nowPlayingInfo，还有一些状态需要通过其它方式同步到 CarPlay app，比如：

  * 音频播放状态：暂停/播放（CPNowPlayingTemplate 中播放按钮的显示状态）

  ```objectivec
  if (@available(iOS 13.0, *)) {
      MPNowPlayingInfoCenter.defaultCenter.playbackState = MPNowPlayingPlaybackStatePlaying; 
      // MPNowPlayingPlaybackStatePaused、MPNowPlayingPlaybackStateStopped
  }
  ```
  
  * 播放模式状态：顺序循环/单曲循环（CPNowPlayingTemplate 中播放模式按钮的显示状态）
  
  ```objectivec
  MPRemoteCommandCenter.sharedCommandCenter.changeRepeatModeCommand.currentRepeatType = repeatType;
  ```

下图中的音频名称、音频描述、音频时长、当前播放进度、播放模式、播放速率等音频播放信息，都是通过以上方式同步显示到 CPNowPlayingTemplate 上的。你的音频 app 之前应该已经实现了该功能以将音频播放信息同步到 iPhone 锁屏界面以及控制中心，现在只需要检查下 CarPlay app 的 CPNowPlayingTemplate 中的音频播放信息是否准确即可。

![image-20211116113630475](/Users/chenjunteng/Library/Application Support/typora-user-images/image-20211116113630475.png)

* 响应 MPRemoteCommandCenter 事件，对用户点击播放控制按钮做出响应，如播放、暂停、切换歌曲等等。

  * playCommand
  * pauseCommand
  * previousTrackCommand
  * nextTrackCommand
  * togglePlayPauseCommand
  * changeRepeatModeCommand
  * changePlaybackRateCommand
  * ...
* 使用 CarPlay framework 时，changeRepeatModeCommand、changePlaybackRateCommand 不再由 MPRemoteCommandCenter 触发，而是通过 CPNowPlayingRepeatButton、CPNowPlayingPlaybackRateButton 的 handle 监听。但 command.enabled 还是要开启。

```
// Discussion
CPNowPlayingRepeatButton is a concrete subclass of CPNowPlayingButton. Use the button’s handler to invoke your existing functionality for cycling through repeat modes, using the same MPChangeRepeatModeCommand that you provide to MPRemoteCommandCenter.
CarPlay uses MPRemoteCommandCenter to observe changes to the repeat mode and updates the button’s appearance accordingly.
```

### 页面跳转

还记得在 CarPlay app 入口 [- templateApplicationScene:didConnectInterfaceController:](https://developer.apple.com/documentation/carplay/cptemplateapplicationscenedelegate/3578119-templateapplicationscene?language=objc) 中的 CPInterfaceController 吗，它作为我们 CarPlay app 的入口 controller，我们将一个 template 作为 rootTemplate 赋值给它。当我们要进行页面跳转时也是靠它，有点类似于 UINavigationController，它支持 push、pop、present、dismiss 等等操作（present、dismiss 操作仅 CPActionSheetTemplate、CPVoiceControlTemplate、CPAlertTemplate）。 



**Best Practices**

* 在调用 `MPPlayableContentDataSource` 和 `MPPlayableContentDelegate` 中的 completion handlers 之前，确保将要播放的内容实际上已经准备好播放或显示，在此期间显示加活动指示器。
* 对于那些不用 `Tabs` 而只用 `TableView` 的 App，`TableView` 必须返回至少一项内容。
* CarPlay 会显示一个加载活动指示器，如果 App 在一定时限内没有返回内容就会超时。如果你的 App 需要一些初始的设置，比如登录凭据，在页面的第一行应该提示用户 App 当前的状态，通过填充一个 `MPContentItem`，MPContentItem is neither playable nor a container indicating the state of the app to the user。

### 图片

#### 图标和图片

可以看看 [CarPlay - 设计指南](https://developer.apple.com/design/human-interface-guidelines/carplay/overview/introduction/)，并将它发给你的 PM 和 UI。

#### 异步图片

* CarPlay 不支持 gif 图片，配置会导致 crash。
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
                let screen = TTScenes.carPlayScene?.value(forKey: "screen") as? UIScreen
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

### 重新加载



### 埋点

一些埋点可能需要通过投机取巧的方法，比如在哪个页面触发了返回按钮，可以通过 `templateDidAppear`、`templateWillDisappear` 等方法配合实现。



## 注意点

* 单例问题。CarPlay 用到了单例类，CarPlay app 关闭，但 iPhone app 没关闭，进程是还在的，单例还未释放，可能会造成一些问题。可以在 `- templateApplicationScene:didConnectInterfaceController:` 中初始化单例，在 `- templateApplicationScene:didDisconnectInterfaceController:` 中释放单例。
* TabBar，如果 tabImage 为 nil，那么该 tabBarItem 将会使用 UITabBarItem.SystemItem.more。
* 数据可以存在 CPListItem.userInfo，不需要扩展属性。它是强引用，需要注意循环引用问题。
* CarPlay Simulator 的语言是跟随 iPhone Simulator 的，真机是否也如此我没有测试过。
* CarPlay Framework 需要设置为弱引用 optional（Target > Build phases > Link Binary With Libraries），否则在 iOS 14 以下启动 App 会 crash。
* CarPlay 断开连接时，建议暂停音乐



## 常见问题解答

### CarPlay 连接

**1. 为什么车机连接了，CarPlay 车载却没有出来？**

* 汽车不支持 CarPlay；
* iPhone 没有开启 Siri。在 `设置 > Siri 与搜索` 中开启 `用“嘿 Siri”唤醒`。如果没有开启 Siri 的话，iPhone 的 `设置 > 通用 ` 中也不会显示 `CarPlay 车载` 这一项。

### Simulator

**1. 为什么 Simulator 菜单栏 I/O > External Displays 中没有 CarPlay 选项？**

你可能还未在开发者网站申请 CarPlay 权限并将配置文件导入到工程中。[申请 CarPlay 权限](https://developer.apple.com/documentation/carplay/requesting_the_carplay_entitlements?language=objc)

**2. 为什么打开 CarPlay Simulator 没有显示我的 app？**

你可能是忘了添加权利文件。你需要将 Key `com.apple.developer.carplay-audio` 添加到 Entitlements.plist 中并设置 Value 为 `1`。

```xml
<key>com.apple.developer.carplay-audio</key>
<true/>
```

可以参考 [申请 CarPlay 权限](https://developer.apple.com/documentation/carplay/requesting_the_carplay_entitlements?language=objc)。

**3. 为什么 M1 Mac 打开 CarPlay app 直接崩溃？** 

如果你的 Xcode 以 Rosetta 模式运行，那将无法使用 CarPlay Simulator。将 Simulator 也以 Rosetta 运行，结果无效。这个问题暂时没有解决方案。https://issueexplorer.com/issue/mapbox/mapbox-navigation-ios/3355。

**4. 从 “播放中” app 返回到音频源 app，页面显示异常**

感觉是 Apple 的 bug，模拟器和真机都可能出现。bug 出现的步骤是：

1. 先不要启动你的 CarPlay app
2. 在你的 iPhone app 中播放音频
3. 打开 CarPlay 的 “播放中” app
4. 通过 “播放中” app 返回到你的 CarPlay app

可能会出现的问题：

* tabBarItem 重叠
* CPListImageRowItem 本应该只显示 4 张图片，却显示了 5 张

切换 template、退后台再进入，页面恢复正常。

### 图片

**1. 为什么 CarPlay 上的图片模糊？**

无论是本地图片还是异步图片都需要适配下 scale。

### 正在播放

**1. rootTemplate 右上角的 “正在播放按钮” 什么时候出现？**

当前 app 正在播放音频时出现，点击它 push 到 CPNowPlayingTemplate。

**2. CarPlay 主界面的 “播放中（Now Playing）” app 是什么？**

* CPNowPlayingTemplate 是个单例类，所有 CarPlay app 的 “正在播放” 界面都是使用这个单例。
* “播放中” app 将从 nowPlayingCenter 中取数据，也就是说该 app 将显示 iPhone 上正在播放的音频信息，并将音频源 app 的 appName 显示在右上角，即使该音频源 app 不支持 CarPlay。因此，即使你的 app 暂时不支持 CarPlay，你也可以通过适配好 MPNowPlayingInfoCenter 和 MPRemoteCommandCenter 来使你的 app 支持 CarPlay ”播放中“ app。

![](/Users/chenjunteng/Library/Application Support/typora-user-images/image-20211115092835324.png)

* 如果 “播放中” app 的音频源 app 支持 CarPlay，那么启动 “播放中” app 就会触发音频源 app 的 CarPlay 场景连接，也就是启动音频源 CarPlay app。这就是为什么 Apple 让我们在 `- templateApplicationScene:didConnectInterfaceController:` 的时机就配置好 CPNowPlayingTemplate 的原因，而不应该在 push 到 CPNowPlayingTemplate 的时候去配置，因为 CPNowPlayingTemplate 并不一定通过主动 push 时触发，可能是通过 “播放中” app 或者 rootTemplate 右上角的 “正在播放按钮”。点击 “播放中” app 左上角的返回按钮，将返回到该音频源的 CarPlay app 的 rootTemplate。

**3. 正在播放页面中音频插图（封面）不显示**

* 如果是显示了占位图，那可能是因为你没将有效图片设置到 nowPlayingInfo 的 MPMediaItemPropertyArtwork key 中。
* 如果是连占位图都没有：
  * 如果是真实环境汽车中不显示，需要在 CarPlay 设置中将 `显示专辑插图` 打开。如果设置中没有显示该选项，那可能是当前 iPhone 设置的语言和地区不支持，在 iPhone 的  `设置 > 语言和地区` 中设置。参考帖子 https://tieba.baidu.com/p/6276976841。
  * 如果是 CarPlay Simulator，那应该没办法显示，即使按照上面的步骤操作了，这点我暂时没有依据。





