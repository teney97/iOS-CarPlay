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
        // create window
        // do something after window created
        // 需要注意原先在 window created 之后才执行的代码，也要兼容 iOS 13
    }
    ...
    return YES;
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
        // create window
        let window = UIWindow(windowScene: windowScene)
        // ...
        self.window = window
        window.makeKeyAndVisible()
        // do something after window created
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

#### TTScenes: 用于获取场景

```swift
import UIKit
import CarPlay

@available(iOS 13.0, *)
class TTScenes: NSObject {

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
                    [(UIWindow *)self setWindowScene:TTScenes.mainScene];
                }
                return self;
            }), method_getTypeEncoding(method));
        });
    }
}

@end
```

#### 首次启动隐私弹窗适配

如果你的首次启动隐私弹窗是通过在 AppDelegate 的 init 方法中 hook `application:didFinishLaunchingWithOptions:` 方法进行拦截的话，也需要 hook `scene:willConnectToSession:options:`，然后可以将隐私弹窗弹出的时机放在这里。`scene:willConnectToSession:options:` 调用时机将在 `application:didFinishLaunchingWithOptions:` return 之后。