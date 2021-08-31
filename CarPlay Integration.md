## 开发 CarPlay 音频 App

### 使用 UIScene

在 iOS14 中使用 CarPlay framework 提供的全新模版开发 CarPlay App 必须采用 UIScene。因此必须从传统的 UIWindow 和 UIApplicationDelegate API 向 UIScene 过渡。

### 声明一个 CarPlay scene

在 Info.plist 中声明一个 scene。

```swift
// CarPlay Scene Manifest

<key>UIApplicationSceneManifest</key>
<dict>
    <key>UISceneConfigurations</key>
    <dict>
        <key>CPTemplateApplicationSceneSessionRoleApplication</key>
        <array>
            <dict>
                <key>UISceneClassName</key>
                <string>CPTemplateApplicationScene</string>
                <key>UISceneConfigurationName</key>
                <string>MyApp—Car</string>
                <key>UISceneDelegateClassName</key>
                <string>MyApp.CarPlaySceneDelegate</string>
            </dict>
        </array>
    </dict>
</dict>
```

### 实现 CarPlaySceneDelegate

```swift
// CarPlay App Lifecycle

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

* 实现 didConnect 方法

  当应用程序在车机屏幕上启动时，CarPlay framework 将调用此方法，在该方法中对模板进行初始化。可以看到，系统自动创建了 CPInterfaceController 实例，作为我们 CarPlay App 的入口 Controller，我们只需在回调中持有这个实例即可。然后我们创建了一个列表模板 CPListTemplate，其接收多组 CPListSection，section 中包含多个 CPListItem，这是一个最基本的列表元素，有点类似 tableViewCell。在这里，我们将 CPListTemplate 设置为 App 的根模板。

* 实现 didDisconnect 方法

  而车机断开连接时，该方法将被调用，在里面可以做一些清理工作。





## API

### CarPlay Integration

#### CPTemplateApplicationSceneDelegate

响应应用场景生命周期事件的方法。

```swift
@protocol CPTemplateApplicationSceneDelegate
```

该协议定义了 CarPlay 在场景连接、断开连接、以及响应某些用户操作的方法。你需要在 CarPlay 启动你的 App 并连接其场景时创建和设置根模板。

你可以在 Info.plist 中声明一个 scene，而不是直接创建一个 CPTemplateApplicationSceneDelegate 示例。

* 响应场景声明周期

  [`- templateApplicationScene:didConnectInterfaceController:`](https://developer.apple.com/documentation/carplay/cptemplateapplicationscenedelegate/3578119-templateapplicationscene?language=objc)

  告诉代理将 CarPlay 场景添加到应用程序。

  [`- templateApplicationScene:didConnectInterfaceController:toWindow:`](https://developer.apple.com/documentation/carplay/cptemplateapplicationscenedelegate/3340482-templateapplicationscene?language=objc)

  告诉代理将 CarPlay 场景添加到您的导航应用程序。

  [`- templateApplicationScene:didDisconnectInterfaceController:`](https://developer.apple.com/documentation/carplay/cptemplateapplicationscenedelegate/3578120-templateapplicationscene?language=objc)

  当 CarPlay 从应用程序中删除场景时通知代理。

  [`- templateApplicationScene:didDisconnectInterfaceController:fromWindow:`](https://developer.apple.com/documentation/carplay/cptemplateapplicationscenedelegate/3340483-templateapplicationscene?language=objc)

  当 CarPlay 从您的导航应用程序中删除场景时通知代理。

* 响应用户操作

  [`- templateApplicationScene:didSelectManeuver:`](https://developer.apple.com/documentation/carplay/cptemplateapplicationscenedelegate/3340484-templateapplicationscene?language=objc)

  当用户在应用程序处于后台时选择操作时通知代理。

  [`- templateApplicationScene:didSelectNavigationAlert:`](https://developer.apple.com/documentation/carplay/cptemplateapplicationscenedelegate/3340485-templateapplicationscene?language=objc)

  当用户在应用程序处于后台时选择导航警报时通知代理。

### General Purpose Templates

使用提供一致的 CarPlay 布局和外观的各种模板来显示你的 App 的内容。

| Templates                                                    | Description                  |
| ------------------------------------------------------------ | ---------------------------- |
| [`CPListTemplate`](https://developer.apple.com/documentation/carplay/cplisttemplate?language=objc) | 列表模版，有点类似 tableView |
| [`CPGridTemplate`](https://developer.apple.com/documentation/carplay/cpgridtemplate?language=objc) |                              |
| [`CPTabBarTemplate`](https://developer.apple.com/documentation/carplay/cptabbartemplate?language=objc) |                              |
| [`CPTemplate`](https://developer.apple.com/documentation/carplay/cptemplate?language=objc) |                              |
| [`CPBarButtonProviding`](https://developer.apple.com/documentation/carplay/cpbarbuttonproviding?language=objc) |                              |

显示一个新的 template，类似 UINavigationController 的 push 方法，调用 CPInterfaceController 的 [`pushTemplate:animated:completion:`](https://developer.apple.com/documentation/carplay/cpinterfacecontroller/3650358-pushtemplate?language=objc)。

#### CPListTemplate

三种 listItem：CPListItem、CPListImageRowItem、CPMessageListItem

```swift
// CPListItem: CPSelectableListItem: CPListTemplateItem
let item1 = CPListItem(text: "Rubber Soul", detailText: "The Beatles")
// CPListImageRowItem: CPSelectableListItem: CPListTemplateItem
let item2 = CPListImageRowItem(text: "", images: [UIImage(named: "")!])
// CPMessageListItem: CPListTemplateItem
let item3 = CPMessageListItem(fullName: "", phoneOrEmailAddress: "", leadingConfiguration: CPMessageListItemLeadingConfiguration(), trailingConfiguration: nil, detailText: nil, trailingText: nil)
// CPListSection, public init(items: [CPListTemplateItem], header: String?, sectionIndexTitle: String?)
let section = CPListSection(items: [item1, item2, item3])
let listTemplate = CPListTemplate(title: "Albums", sections: [section])
```

使用 CPListTemplate，对于点餐 App，section 数量不超过 2 个，而其它类型 App 的 session 数量不超过 5 个。此外，某些车辆会限制 session 数量。有关更多信息，请参阅 [`CPSessionConfiguration`](https://developer.apple.com/documentation/carplay/cpsessionconfiguration?language=objc)。

集成 Siri，参阅 https://developer.apple.com/documentation/carplay/cplisttemplate?language=objc

![显示列表模板顶部的助理单元格的屏幕截图。](https://docs-assets.developer.apple.com/published/49959584ba/rendered2x-1619630673.png)

