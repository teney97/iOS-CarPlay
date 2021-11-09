## 兼容 SceneDelegate

在 iOS 14 及更高版本中使用 CarPlay Framework 来开发 CarPlay app 必须使用 UIScene（UIScene 是 Apple 于 iOS 13 引入的，用于构建多窗口应用），因此你的工程必须从传统的 UIWindow 和 UIApplicationDelegate API 向 UIScene 过渡。如果你的工程已经兼容了 UIScene，那么就可以省去这步骤的工作；如果还未兼容的话，也可以参考如下步骤。

### SceneDelegate 是什么

在 iOS 13 之前，AppDelegate 的职责是管理 App 生命周期和 UI 生命周期。这种模式完全没问题，因为只有一个进程，只有一个与这个进程对应的用户界面。

![img](https://upload-images.jianshu.io/upload_images/738839-595e8a80104972f8.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

但是要想开发多窗口的 iPad App 或者 Mac Catalyst App 的话，AppDelegate 已经不支持了。于是 Apple 于 iOS 13 引入 UIScene，用于构建多窗口应用。在 iOS 13 之后，如果你在工程中使用 SceneDelegate 的话，UI 生命周期将交由 SceneDelegate 管理，而 AppDelegate 则继续管理 App 生命周期以及新的 Scene Session 生命周期，职责单一化。

![img](https://upload-images.jianshu.io/upload_images/738839-0d9367533f3ced83.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

![img](https://upload-images.jianshu.io/upload_images/738839-495e5d5aedba2b0f.png?imageMogr2/auto-orient/strip|imageView2/2/w/666/format/webp)

### 兼容 SceneDelegate

因为 UIScene 只能在 iOS 13 及更高版本中使用，因此如果你的 App 最低版本支持小于 iOS 13 的话，你就不能完全使用 SceneDelegate，在 iOS 13 及更高版本中使用 AppDelegate + SceneDelegate，而在低于 iOS 13 的版本中继续只使用 AppDelegate。

#### Info.plist

Info.plist 中添加以下 key-value。一些参数说明：

* Enable Multiple Windows，需要设置为 NO，否则你的 iPad App 将支持多窗口。
* Application Session Role，一个数组，配置你的 app 场景，每一项有 4 个参数：
  * Class Name：Scene 类型
  * Configuration Name：当前配置的名字
  * Delegate Class Name：与哪个 Scene 代理类关联
  * StoryBoard name：这个 Scene 使用的哪个 storyBoard 如果有的话

![image-20211109143559164](/Users/chenjunteng/Library/Application Support/typora-user-images/image-20211109143559164.png)

```xml
<key>UIApplicationSceneManifest</key>
<dict>
    <key>UIApplicationSupportsMultipleScenes</key>
        <false/>
    <key>UISceneConfigurations</key>
    <dict>
        <key>UIWindowSceneSessionRoleApplication</key>
        <array>
            <dict>
                <key>UISceneClassName</key>
                <string>UIWindowScene</string>
                <key>UISceneConfigurationName</key>
                <string>DefaultSceneConfiguration</string>
                <key>UISceneDelegateClassName</key>
                <string>$(PRODUCT_MODULE_NAME).SceneDelegate</string>
            </dict>
        </array>
        <key>CPTemplateApplicationSceneSessionRoleApplication</key>
        // ...
        // CPTemplateApplicationScene
        // ...
    </dict>
</dict>
```

#### Project

Targets > General > Deployment Info > Supports multiple windows 取消勾选。该选项选中状态会影响 Info.plist 中 Enable Multiple Windows 的值。

![image-20211109150539574](/Users/chenjunteng/Library/Application Support/typora-user-images/image-20211109150539574.png)

#### AppDelegate 改动

```objectivec
@implementation AppDelegate

- (UIWindow *)window {
    if (@available(iOS 13, *)) {
        return [(SceneDelegate *)HTScenes.mainScene.delegate window];
    } else {
        return _window;
    }
}

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    ...
    if (@available(iOS 13.0, *)) {} else {
        // createWindow
    }
    ...
}

@end
```

以下根据你的需求添加。

```swift
@available(iOS 13, *)
extension AppDelegate {

    /*
     1.如果没有在 app 的 Info.plist 文件中包含 scene 的配置数据，或者要动态更改场景配置数据，需要实现此方法。UIKit 会在创建新 scene 前调用此方法。
     2.方法会返回一个 UISceneConfiguration 对 象，其中包含场景详细信息，包括要创建的场景类型，用于管理场景的委托对象以及包含要显示的初始视图控制器的情节提要。 如果未实现此方法，则必须在应用程序的 Info.plist 文件中提供场景配置数据。

     总结下：默认在 Info.plist 中进行了配置，不用实现该方法也没有关系。如果没有配置就需要实现这个方法并返回一个 UISceneConfiguration 对象。
     配置参数中 Application Session Role 是个数组，每一项有三个参数:
         1) Configuration Name:   当前配置的名字;
         2) Delegate Class Name:  与哪个 Scene 代理对象关联;
         3) StoryBoard name: 这个 Scene 使用的哪个 storyboard。
     注意：代理方法中调用的是配置名为 Default Configuration 的 Scene，则系统就会自动去调用 SceneDelegate 这个类。这样 SceneDelegate 和 AppDelegate 产生了关联。
     */
    func application(_ application: UIApplication, configurationForConnecting connectingSceneSession: UISceneSession, options: UIScene.ConnectionOptions) -> UISceneConfiguration {
        // Called when a new scene session is being created.
        // Use this method to select a configuration to create the new scene with.
        return UISceneConfiguration(name: "Default Configuration", sessionRole: connectingSceneSession.role)
    }

    // 在分屏中关闭其中一个或多个 scene 时候回调用
    func application(_ application: UIApplication, didDiscardSceneSessions sceneSessions: Set<UISceneSession>) {
        // Called when the user discards a scene session.
        // If any sessions were discarded while the application was not running, this will be called shortly after application:didFinishLaunchingWithOptions.
        // Use this method to release any resources that were specific to the discarded scenes, as they will not return.
    }
}
```

#### SceneDelegate

```swift
@available(iOS 13, *)
class SceneDelegate: UIResponder, UIWindowSceneDelegate {

    let configurationName = "DefaultSceneConfiguration"
    @objc var window: UIWindow?

    func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
        guard let windowScene = (scene as? UIWindowScene), session.configuration.name == configurationName else { return }
        // createWindow
        let window = UIWindow(windowScene: windowScene)
        // ...
        self.window = window
        window.makeKeyAndVisible()
    }
  
    func sceneDidDisconnect(_ scene: UIScene) {
        guard scene.session.configuration.name == configurationName else { return }
    }

    func sceneDidBecomeActive(_ scene: UIScene) {
        guard scene.session.configuration.name == configurationName else { return }
        UIApplication.shared.delegate?.applicationDidBecomeActive?(UIApplication.shared)
    }

    func sceneWillResignActive(_ scene: UIScene) {
        guard scene.session.configuration.name == configurationName else { return }
        UIApplication.shared.delegate?.applicationWillResignActive?(UIApplication.shared)
    }

    func sceneWillEnterForeground(_ scene: UIScene) {
        // Called as the scene transitions from the background to the foreground.
        // Use this method to undo the changes made on entering the background.
        guard scene.session.configuration.name == configurationName else { return }
        UIApplication.shared.delegate?.applicationWillEnterForeground?(UIApplication.shared)
    }

    func sceneDidEnterBackground(_ scene: UIScene) {
        guard scene.session.configuration.name == configurationName else { return }
        UIApplication.shared.delegate?.applicationDidEnterBackground?(UIApplication.shared)
    }

    func scene(_ scene: UIScene, openURLContexts URLContexts: Set<UIOpenURLContext>) {
        guard scene.session.configuration.name == configurationName else { return }
        guard let url = URLContexts.first?.url else { return }
        _ = UIApplication.shared.delegate?.application?(UIApplication.shared, open: url, options: [:])
    }
    
    func scene(_ scene: UIScene, continue userActivity: NSUserActivity) {
        guard scene.session.configuration.name == configurationName else { return }
        _ = UIApplication.shared.delegate?.application?(UIApplication.shared, continue: userActivity, restorationHandler: { _ in })
    }
}
```

#### HTScenes 获取场景

```swift
import UIKit
import CarPlay

@available(iOS 13.0, *)
class HTScenes: NSObject {

    static var connectedScenes: Set<UIScene> {
        UIApplication.shared.connectedScenes
    }
    
    @objc static var mainScene: UIWindowScene? {
        connectedScenes.first(where: { $0 is UIWindowScene }) as! UIWindowScene?
    }
    
    static var carPlayScene: CPTemplateApplicationScene? {
        connectedScenes.first(where: { $0 is CPTemplateApplicationScene }) as! CPTemplateApplicationScene?
    }
}
```

#### UIView+SceneHook

使用 UIScene，一个 UIWindow 必须使用 windowScene 进行初始化或者设置 windowScene 属性才能显示。这里我们可以 hook UIView 的 initWithFrame: 方法：

```objectivec
@implementation UIView (SceneHook)

+ (void)load {
    if (@available(iOS 13.0, *)) {
        // hook initWithFrame
    }
}

- (instancetype)htScene_initWithFrame:(CGRect)frame API_AVAILABLE(ios(13)) {
    [self htScene_initWithFrame:frame];
    if ([self isKindOfClass:UIWindow.class]) {
        [(UIWindow *)self setWindowScene:HTScenes.mainScene];
    }
    return self;
}

@end
```

#### 首次启动隐私弹窗适配

如果你的首次启动隐私弹窗是通过在 AppDelegate 的 init 方法中 hook `application:didFinishLaunchingWithOptions:` 方法进行拦截的话，可以 hook `scene:willConnectToSession:options:` 并将隐私弹窗弹出的时机放在这里。`scene:willConnectToSession:options:` 调用时机将在 `application:didFinishLaunchingWithOptions:` return 之后。

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

每个 iPhone Simulator 都附带一个 CarPlay Simulator，在 I/O > External Displays > CarPlay... 打开。

如果你想在每次启动 CarPlay Simulator 前都可以设置屏幕尺寸等参数，可以在终端输入以下命令。不过 Width、Height、Scale 我设置后都不尽如人意，不知道是不是我使用方式不对。

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

#### 正在播放页面

这是音频类 CarPlay app 最重要的页面了，使用 CPNowPlayingTemplate，它是一个单例。

使用 CarPlay Framework 或者 MediaPlayer Framework 来构建的 CarPlay app，都是通过 MPRemoteCommandCenter 和 MPNowPlayingInfoCenter 来提供播放界面的音频信息以及播放控制。只不过在 CarPlay Framework 中，一些 RemoteCommand 事件通过 Button Handle 来处理了，比如播放模式、播放速率等等。

* 设置和更新 MPNowPlayingInfoCenter 的 nowPlayingInfo，它包含当前播放音频的元数据，如标题、作者、时长等等。

下图中的音频名称、音频描述、音频时长、当前播放进度、播放模式、播放速率等音频播放信息，都是通过更新 MPNowPlayingInfoCenter 的 nowPlayingInfo 来显示到 CPNowPlayingTemplate 上的。你的音频 app 之前应该已经实现了该功能以将音频播放信息同步到锁屏界面，现在根据你的 CarPlay app 需求补充下音频播放信息到 nowPlayingInfo 就行。

* 响应 MPRemoteCommandCenter 事件，让用户对你的内容执行命令，如播放、暂停、切换歌曲等等。

  

![image-20211109173720890](/Users/chenjunteng/Library/Application Support/typora-user-images/image-20211109173720890.png)























