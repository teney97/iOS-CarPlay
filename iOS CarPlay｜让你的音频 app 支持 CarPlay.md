## 开发 CarPlay 音频 App

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
        self.interfaceController = ni
    }
}
```

CPTemplateApplicationSceneDelegate 协议定义了 CarPlay 在场景连接、断开连接、以及响应某些用户操作的方法。你需要在 CarPlay 启动你的 App 并连接其场景时创建和设置根模板。一般我们实现以下两个方法

* [- templateApplicationScene:didConnectInterfaceController:](https://developer.apple.com/documentation/carplay/cptemplateapplicationscenedelegate/3578119-templateapplicationscene?language=objc)

  通知代理 CarPlay Scene 已连接。当应用程序在车机屏幕上启动时，CarPlay framework 将调用此方法，在该方法中对模板进行初始化。可以看到，系统自动创建了 CPInterfaceController 实例，作为我们 CarPlay App 的入口 Controller，我们只需在回调中持有这个实例即可。然后我们创建了一个列表模板 CPListTemplate，其接收多组 CPListSection，section 中包含多个 CPListItem，这是一个最基本的列表元素，有点类似 tableViewCell。在这里，我们将 CPListTemplate 设置为 App 的根模板。

* [- templateApplicationScene:didDisconnectInterfaceController:](https://developer.apple.com/documentation/carplay/cptemplateapplicationscenedelegate/3578120-templateapplicationscene?language=objc)

  通知代理 CarPlay Scene 已断开连接。当车机断开连接时，该方法将被调用，在里面可以做一些清理工作。

### CarPlay Simulator

每个 iPhone Simulator 都附带一个 CarPlay Simulator，在 I/O > External Displays > CarPlay 打开。默认的标准的 CarPlay Simulator 窗口大小和比例为 `800 x 480, @2x`。

如果是导航类 app，Apple 建议开启 CarPlay Simulator 的附加选项，在终端输入以下命令。这允许你每次启动 CarPlay Simulator 前都可以设置窗口大小和比例，以测试确保你的地图内容适配了所有推荐的配置。也许只支持导航类 app 吧，音频类 app 改变窗口大小后显示效果不尽如人意，建议关闭。

```
defaults write com.apple.iphonesimulator CarPlayExtraOptions -bool YES
```

![image-20211109154924029](/Users/chenjunteng/Library/Application Support/typora-user-images/image-20211109154924029.png)

如果你的工程无法打开 CarPlay Simulator（灰显禁用），那么可能你还未在开发者网站申请 CarPlay 权限并将配置文件导入到工程中。

如果你打开了 CarPlay Simulator 却没有在主屏幕上看到你的 app，那么你可能是忘了添加权利文件。你需要将 Key `com.apple.developer.carplay-audio` 添加到 Entitlements.plist 中并设置 Value 为 `1`。

```xml
<key>com.apple.developer.carplay-audio</key>
<true/>
```

可以参考 [申请 CarPlay 权限](https://developer.apple.com/documentation/carplay/requesting_the_carplay_entitlements?language=objc)。

### CarPlay 界面搭建

| Templates                                                    | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`CPListTemplate`](https://developer.apple.com/documentation/carplay/cplisttemplate?language=objc)<br />- [`CPListItem`](https://developer.apple.com/documentation/carplay/cplistitem?language=objc)<br />- [`CPListImageRowItem`](https://developer.apple.com/documentation/carplay/cplistimagerowitem?language=objc)<br />- [`CPMessageListItem`](https://developer.apple.com/documentation/carplay/cpmessagelistitem?language=objc) | 列表模版，有点类似 tableView<br />- 一个通用的、可选择的列表项<br />- 显示一系列图像的列表项<br />- 表示对话或联系人的列表项 |
| [`CPGridTemplate`](https://developer.apple.com/documentation/carplay/cpgridtemplate?language=objc) |                                                              |
| [`CPTabBarTemplate`](https://developer.apple.com/documentation/carplay/cptabbartemplate?language=objc) | tabbar 模版                                                  |
| [`CPTemplate`](https://developer.apple.com/documentation/carplay/cptemplate?language=objc) |                                                              |
| [`CPBarButtonProviding`](https://developer.apple.com/documentation/carplay/cpbarbuttonproviding?language=objc) |                                                              |

![](https://docs-assets.developer.apple.com/published/49959584ba/rendered2x-1619630673.png)

CarPlay app 界面基本就是由 Template 和 Item 组成，而音频类的 CarPlay app 基本上使用 CPTabBarTemplate、CPListTemplate、CPListImageRowItem、CPListItem 等等即可完成界面的搭建。

#### CPTabBarTemplate

TabBar 模版，类似 UIKit 中的 UITabBar。使用一组 CPTemplate 进行初始化，可以将其作为 interfaceController 的 rootTemplate。

```swift
let tabBarTemplate = CPTabBarTemplate(templates: templates)
interfaceController.setRootTemplate(tabBarTemplate, animated: true)
```



![image-20211109162742747](/Users/chenjunteng/Library/Application Support/typora-user-images/image-20211109162742747.png)

需要注意一下，CPTabBarTemplate 的 templates 是有个数限制的，它受汽车显示屏尺寸的应用。可以显示多少个标签取决于 maximumTabCount 属性，取决于 app 种类，音频 app 貌似最多显示 4 个，超过数量会 Crash。

> 在 [WWDC17 - 让你的 App 支持 CarPlay 车载](https://github.com/teney97/iOS-CarPlay/blob/main/Content/WWDC17%20-%20%E8%AE%A9%E6%82%A8%E7%9A%84%20App%20%E6%94%AF%E6%8C%81%20CarPlay%20%E8%BD%A6%E8%BD%BD.md) 中 Apple 提到过使用 MediaPlayer Framework 来构建的 CarPlay app 时，推荐使用最多 4 个 tabs 并且使用较短的标题，因为空间有限并且有些汽车的屏幕比较窄，而且有正在播放的内容的时候还需显示 “正在播放” 按钮。

```swift
/**
 The maximum number of tabs that your app may display in a @c CPTabBarTemplate,
 depending on the entitlements that your app declares.

 @warning The system will throw an exception if your app attempts to display more
 than this number of tabs in your tab bar template.
 */
open class var maximumTabCount: Int { get }
```

可以设置每个 template 标签的 tabTitle 和 tabImage，也可以设置 tabSystemItem 用系统样式（可用样式较少，且没办法自定义文案了）。

```swift
// 自定义 tab 样式
listTemplate.tabTitle = "推荐"
listTemplate.tabImage = UIImage(named: "tabbar_home_high")
// 使用系统 tab 样式，如果同时设置了 tabTitle 和 tabImage， tabSystemItem 不生效
listTemplate.tabSystemItem = .favorites
// 显示红点
listTemplate.showsTabBadge = true
```

这时候就要找 UI 出图了，[CarPlay UI 设计指南](https://developer.apple.com/design/human-interface-guidelines/carplay/icons-and-images/custom-icons/)。

CPTabBarTemplateDelegate 代理方法就一个：

```swift
@available(iOS 14.0, *)
public protocol CPTabBarTemplateDelegate : NSObjectProtocol {
    func tabBarTemplate(_ tabBarTemplate: CPTabBarTemplate, didSelect selectedTemplate: CPTemplate)
}
```

#### CPListTemplate

列表模版，有点类似 UITableview。可以一组 CPListTemplate 来初始化 CPTabBarTemplate。CPListTemplate 由遵循 CPListTemplateItem 协议的 item 组成，有点类似 UITableviewCell。一般音频 App 使用 CPListItem、CPListImageRowItem 足以。

![image-20211109164540974](/Users/chenjunteng/Library/Application Support/typora-user-images/image-20211109164540974.png)

#### CPListImageRowItem

可以用来展示由一组专辑组成的模块。使用文本和一组图片初始化，文本可以展示模块名称，图片可以展示模块里前 n 个专辑的封面。

```swift
// 继承链 CPListImageRowItem: CPSelectableListItem: CPListTemplateItem
let imagesCount = CPMaximumNumberOfGridImages
let images = Array(repeating: image, count: imagesCount)
let listImageRowItem = CPListImageRowItem(text: text, images: images)
```



![image-20211109165230578](/Users/chenjunteng/Library/Application Support/typora-user-images/image-20211109165230578.png)

CPListImageRowItem 的可点击区域为上方标题栏（如上面蓝色区域），下方的每张图片区域。

标题栏的点击回调走的是 CPListTemplateDelegate 的方法。点击可以 push 到该模块的专辑列表页面。

```swift
protocol CPListTemplateDelegate : NSObjectProtocol {
    func listTemplate(_ listTemplate: CPListTemplate, didSelect item: CPListItem, completionHandler: @escaping () -> Void)
}
```

下方的每张图片的点击回调通过设置 listImageRowItem 的 listImageRowHandler 属性来监听。点击可以 push 到该专辑的音频列表页面。

```swift
var listImageRowHandler: ((CPListImageRowItem, Int, @escaping () -> Void) -> Void)? // The image row item that the user selected.
```

#### CPListItem

可以用来展示专辑或音频。

对于音频 item，一般由音频封面、音频名称、音频描述组成，可以使用以下构造器初始化 CPListItem。

```swift
let listItem = CPListItem(text: audioName, detailText: audioDesc, image: audioCover)
```

![image-20211109172454397](/Users/chenjunteng/Library/Application Support/typora-user-images/image-20211109172454397.png)

对于专辑 item，还需要右边的导航箭头做区分，指示用户点击可以打开音频列表页面，可以使用以下构造器初始化 CPListItem。

```swift
let listItem = CPListItem(text: albumName, detailText: albumDesc, image: albumCover, accessoryImage: nil, accessoryType: .disclosureIndicator)
```

右边的图标有两个系统样式（箭头、云朵），同时也支持自定义。

```swift
enum CPListItemAccessoryType : Int {
    case none = 0
    case disclosureIndicator = 1
    case cloud = 2
}
```

![image-20211109172500877](/Users/chenjunteng/Library/Application Support/typora-user-images/image-20211109172500877.png)

#### CPNowPlayingTemplate: 正在播放页面（Use MPNowPlayingInfoCenter and MPRemoteCommandCenter API）

这是音频类 CarPlay app 最重要的页面了，使用 CPNowPlayingTemplate，它是一个单例。

你可以根据自己的需求配置 CPNowPlayingTemplate，比如添加控制按钮。你可以使用 CarPlay Framework 提供的一些系统按钮，也可以自定义按钮。需要注意的是，你要在 `- templateApplicationScene:didConnectInterfaceController:` 的时机就配置好 CPNowPlayingTemplate，而不应该在 push 到 CPNowPlayingTemplate 的时候才去配置，因为 CPNowPlayingTemplate 并不一定是通过主动 push 时触发，还可能是通过 “播放中” app 或者 rootTemplate 右上角的 “正在播放按钮”。

```swift
let nowPlayingTemplate = CPNowPlayingTemplate.shared
let repeatButton = CPNowPlayingRepeatButton() { ... }
let playbackRateButton = CPNowPlayingPlaybackRateButton() { ... }
nowPlayingTemplate.updateNowPlayingButtons([repeatButton, playbackRateButton])
```

使用 CarPlay Framework 或者 MediaPlayer Framework 来构建的 CarPlay app，都是通过 MPRemoteCommandCenter 和 MPNowPlayingInfoCenter 来提供播放界面的音频信息以及播放控制。只不过在 CarPlay Framework 中，一些 RemoteCommand 事件通过 Button Handle 来处理了，比如播放模式、播放速率等等。

* 设置和更新 MPNowPlayingInfoCenter 的 nowPlayingInfo，它包含当前播放音频的元数据，如标题、作者、时长等等。时机：

  * 切换音频（上一首、下一首等等）
  * 暂停、恢复、停止播放
  * seek（跳过片头、拖动进度等等）
  * 更新播放速率，nowPlayingInfo.key : `MPNowPlayingInfoPropertyPlaybackRate`（CPNowPlayingTemplate 中播放速率按钮的显示状态）
  * ...
  
* 除了 nowPlayingInfo，还有一些状态需要通过其它方式同步到 CarPlay app

  * 音频播放状态（CPNowPlayingTemplate 中播放按钮的显示状态）

    ```objectivec
    #import <MediaPlayer/MediaPlayer>
    // 更新 CarPlay App 上的音频播放状态
    if (@available(iOS 13.0, *)) {
        MPNowPlayingInfoCenter.defaultCenter.playbackState = MPNowPlayingPlaybackStatePlaying;
        // MPNowPlayingPlaybackStatePaused、MPNowPlayingPlaybackStateStopped
    }
    ```

  * 播放模式状态：顺序循环/单曲循环（CPNowPlayingTemplate 中播放模式按钮的显示状态）

    ```objectivec
    MPRemoteCommandCenter.sharedCommandCenter.changeRepeatModeCommand.currentRepeatType = repeatType;
    ```

下图中的音频名称、音频描述、音频时长、当前播放进度、播放模式、播放速率等音频播放信息，都是通过以上方式同步显示到 CPNowPlayingTemplate 上的。你的音频 app 之前应该已经实现了该功能以将音频播放信息同步到锁屏界面，现在根据你的 CarPlay app 需求补充下音频播放信息到 nowPlayingInfo 就行。

![image-20211116113630475](/Users/chenjunteng/Library/Application Support/typora-user-images/image-20211116113630475.png)

* 响应 MPRemoteCommandCenter 事件，让用户对你的内容执行命令，如播放、暂停、切换歌曲等等。

  * playCommand
  * pauseCommand
  * previousTrackCommand
  * nextTrackCommand
  * togglePlayPauseCommand
  * changeRepeatModeCommand
  * changePlaybackRateCommand
  
* 使用 CarPlay Framework 时，changeRepeatModeCommand、changePlaybackRateCommand 不再由 MPRemoteCommandCenter 触发，而是通过 CPNowPlayingRepeatButton、CPNowPlayingPlaybackRateButton 的 handle 监听。但 command.enabled 还是要开启。

  ```swift
  // Discussion
  CPNowPlayingRepeatButton is a concrete subclass of CPNowPlayingButton. Use the button’s handler to invoke your existing functionality for cycling through repeat modes, using the same MPChangeRepeatModeCommand that you provide to MPRemoteCommandCenter.
  CarPlay uses MPRemoteCommandCenter to observe changes to the repeat mode and updates the button’s appearance accordingly.
  ```

#### Push and Pop

在调用 `MPPlayableContentDataSource` 和 `MPPlayableContentDelegate` 中的 completion handlers 之前，确保将要播放的内容实际上已经准备好播放或显示，在此期间显示加活动指示器。



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

### 通用 Section

开发音频类 CarPlay app，你应该需要以下通用 Section。

```Swift
@available(iOS 14, *)
class CPCommonSection: NSObject {
    
    enum SectionType {
        case loading
        case playAll
        case nonePlayRecord
    }

    static func loadingSection() -> CPListSection {
        let item = CPListItem(text: "正在加载中", detailText: nil)
        item.userInfo = SectionType.loading
        let section = CPListSection(items: [item])
        return section
    }
    
    static func playAllSection(count: Int) -> CPListSection {
        let item = CPListItem(text: "播放全部", detailText: "\(count)首", image: UIImage(named: "CP_play"))
        item.userInfo = SectionType.playAll
        let section = CPListSection(items: [item])
        return section
    }
    
    static func nonePlayRecordSection() -> CPListSection {
        let item = CPListItem(text: "还没有播放记录哦", detailText: nil)
        item.userInfo = SectionType.nonePlayRecord
        let section = CPListSection(items: [item])
        return section
    }
}
```



### 埋点

一些埋点可能需要通过投机取巧的方法，比如在哪个页面触发了返回按钮，可以通过 `templateDidAppear`、`templateWillDisappear` 等方法配合实现。



## 注意点

* 单例问题。CarPlay 用到了单例类，CarPlay app 关闭，但 iPhone app 没关闭，进程是还在的，单例还未释放，可能会造成一些问题。可以在 `- templateApplicationScene:didConnectInterfaceController:` 中初始化单例，在 `- templateApplicationScene:didDisconnectInterfaceController:` 中释放单例。
* TabBar，如果 tabImage 为 nil，那么该 tabBarItem 将会使用 UITabBarItem.SystemItem.more。
* 数据可以存在 CPListItem.userInfo，不需要扩展属性。它是强引用，需要注意循环引用问题。
* CarPlay Simulator 的语言是跟随 iPhone Simulator 的，真机是否也如此我没有测试过。
* CarPlay Framework 需要设置为弱引用 optional（Target > Build phases > Link Binary With Libraries），否则在 iOS 14 以下启动 App 会 crash。
* CarPlay 断开连接，建议暂停音乐



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





