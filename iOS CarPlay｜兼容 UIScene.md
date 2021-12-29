## 兼容 UIScene

在 iOS 14 及更高版本中使用 CarPlay framework 来开发 CarPlay app 必须使用 UIScene（UIScene 是 Apple 于 iOS 13 引入的，用于构建多窗口应用），因此你的工程必须从传统的 UIWindow 和 AppDelegate 向 SceneDelegate 过渡。如果你的工程已经兼容了 UIScene，那么就可以省去这步骤的工作；如果还未兼容的话，可以参考本章节中的步骤。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/201ca825e81f427e88ed74560cb1f8ae~tplv-k3u1fbpfcp-watermark.image?)

### UIScene 是什么

在 iOS 13 之前，在功能职责上，UIApplication 负责 App 状态，UIApplicationDelegate（AppDelegate）负责 App 事件和生命周期，包括进程和 UI 的。对于单窗口的 App 来说这没有问题，但是要想开发多窗口的 iPad App 或者 Mac Catalyst App 的话，这种功能职责的划分已经不支持了。

因此， Apple 于 iOS 13 引入用于构建多窗口应用的 UIScene，并对功能职责进行了拆分，将 UI 相关的状态、事件和生命周期交与 [UIWindowScene](https://developer.apple.com/documentation/uikit/uiwindowscene/) 和 [UIWindowSceneDelegate](https://developer.apple.com/documentation/uikit/uiwindowscenedelegate/)（SceneDelegate）负责，[UISceneSession](https://developer.apple.com/documentation/uikit/uiscenesession/) 负责持久化的 UI 状态。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b6ceb358854349a49eee654d4796940f~tplv-k3u1fbpfcp-watermark.image?)

### 兼容 UIScene 

因为 UIScene 只能在 iOS 13 及更高版本中使用，因此如果你的 App 最低版本支持小于 iOS 13 的话，你就不能完全使用 SceneDelegate，在 iOS 13 及更高版本中使用 AppDelegate + SceneDelegate，而在低于 iOS 13 的版本中继续只使用 AppDelegate。

#### 在 Info.plist 声明一个 UIWindowScene

Info.plist 中添加以下 key-value。一些参数说明：

* Enable Multiple Windows，需要设置为 NO，否则你的 iPad App 将支持多窗口（如果你的 iPhone 和 iPad 工程放在同一工程下的话）。
* Application Session Role，一个数组，配置你的 App 场景，每一项有 4 个参数：
  * Class Name：Scene 类型
  * Configuration Name：当前配置的名字
  * Delegate Class Name：与哪个 Scene 代理类关联
  * StoryBoard name：这个 Scene 使用的哪个 storyBoard 如果有的话

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/50a9bf67131049c297b58ef719e18f36~tplv-k3u1fbpfcp-watermark.image?)

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

**Targets > General > Deployment Info > Supports multiple windows** 取消勾选。该选项选中状态会影响 Info.plist 中 Enable Multiple Windows 的值。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/98192ea75aa84ed6b0dda12f6dc95bd7~tplv-k3u1fbpfcp-watermark.image?)

#### AppDelegate 改动

由于类功能职责的变化，一些原本在 AppDelegate API 中的实现需要迁移到 SceneDelegate API 中。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/840edc05115e4d0a948c7d713437ea40~tplv-k3u1fbpfcp-watermark.image?)

```objectivec
@implementation AppDelegate

- (UIWindow *)window {
    if (@available(iOS 13, *)) {
        return [(SceneDelegate *)TTScene.main.delegate window];
    } else {
        return _window;
    }
}

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    ...
    if (@available(iOS 13.0, *)) {} else {
        // 1. create window
        // 2. do something after window created。需要注意原先在 window created 之后才执行的代码，也要兼容 iOS 13
    }
    ...
    return YES;
}

@end
```

以下方法选择性实现。

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
        // 1. create window
        let window = UIWindow(windowScene: windowScene)
        // ...
        self.window = window
        window.makeKeyAndVisible()
        // 2. do something after window created
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

#### TTScene: 用于获取场景

```swift
import UIKit
import CarPlay

@available(iOS 13.0, *)
class TTScene: NSObject {

    static var connectedScenes: Set<UIScene> {
        UIApplication.shared.connectedScenes
    }
    
    @objc static var main: UIWindowScene? {
        connectedScenes.first(where: { $0 is UIWindowScene }) as! UIWindowScene?
    }
    
    static var carPlay: CPTemplateApplicationScene? {
        connectedScenes.first(where: { $0 is CPTemplateApplicationScene }) as! CPTemplateApplicationScene?
    }
}
```

#### UIWindow+SceneHook

使用 UIScene 后，UI 层级结构发生了一些变化，原本的 UIScreen 和 UIWindow 层中加入了一层 UIWindowScene。

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/af3a8af87c2a4c3db1d36a68e5a80626~tplv-k3u1fbpfcp-watermark.image?)

而 UIWindow 也新增了一个 windowScene 属性，以及 windowScene 构造器。一个 UIWindow 必须使用 windowScene 进行初始化或者设置 windowScene 属性才能显示在屏幕上。

```objectivec
// instantiate a UIWindow already associated with a given UIWindowScene instance, with matching frame & interface orientations.
- (instancetype)initWithWindowScene:(UIWindowScene *)windowScene API_AVAILABLE(ios(13.0));

// If nil, window will not appear on any screen.
// changing the UIWindowScene may be an expensive operation and should not be done in performance-sensitive code
@property (nullable, nonatomic, weak) UIWindowScene *windowScene API_AVAILABLE(ios(13.0));
```

为了兼容旧代码，我们可以 hook UIView 的 initWithFrame: 方法，为 UIWindow 设置 windowScene。

```objectivec
@implementation UIView (SceneHook)

+ (void)load {

    if (@available(iOS 13.0, *)) {
      
        static dispatch_once_t onceToken;
        dispatch_once(&onceToken, ^{
            SEL selector = @selector(initWithFrame:);
            Method method = class_getInstanceMethod(self, selector);
            if (!method) {
                NSAssert(NO, @"Method not found for [UIView initWithFrame:]");
            }
            IMP imp = method_getImplementation(method);
            class_replaceMethod(self, selector, imp_implementationWithBlock(^(UIView *self, CGRect frame) {
                ((UIView * (*)(UIView *, SEL, CGRect))imp)(self, selector, frame);
                if ([self isKindOfClass:UIWindow.class]) {
                    [(UIWindow *)self setWindowScene:TTScene.main];
                }
                return self;
            }), method_getTypeEncoding(method));
        });
    }
}

@end
```

#### 首次启动隐私弹窗适配

如果你的首次启动隐私弹窗是通过在 AppDelegate 的 init 方法中 hook `- application:didFinishLaunchingWithOptions:` 方法进行拦截的话，也需要 hook `- scene:willConnectToSession:options:`，然后可以将隐私弹窗弹出的时机放在这里。`- scene:willConnectToSession:options:` 调用时机将在 `- application:didFinishLaunchingWithOptions:` return 之后。

### 相关资料

* [WWDC19｜Introducing Multiple Windows on iPad](https://developer.apple.com/videos/play/wwdc2019/212)
    * [WWDC19 内参｜iPad 上的多窗口](https://xiaozhuanlan.com/topic/0342159876)
* [Apple｜Specifying the Scenes Your App Supports](https://developer.apple.com/documentation/uikit/app_and_environment/scenes/specifying_the_scenes_your_app_supports?language=objc)
