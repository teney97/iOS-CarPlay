本文是对 [WWDC22｜10016 - Get more mileage out of your app with CarPlay](https://developer.apple.com/videos/play/wwdc2022/10016/) 的梳理。

## 概要 

时隔 2 年，CarPlay 终于更新了。以下是该 session 的主要内容：

* 介绍 iOS16 中新增的两种支持 CarPlay 的 App 类型：Fueling App 和 Driving Task App
* 探索 Navigation App 如何在受支持车辆中的数字仪表盘上实时绘制地图
* 🌟 CarPlay Simulator：全新的 CarPlay App 开发与测试工具。它可以帮助你在不离开办公桌的情况下连接 iPhone Device 来开发和测试 CarPlay App，模拟真实环境，而无需经常来回跑到你的车上或购买售后主机进行测试
* 简单演示使用 CarPlay Simulator 对 CarPlay App 进行测试

![20220628141134.jpg](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/132d77361244433190a20070c9201ca0~tplv-k3u1fbpfcp-watermark.image?)

**注**｜本文中除 `CarPlay App` 外，单独出现的 `App` 均指 `iPhone App`。

## 新增两种支持 CarPlay 的 App 类型：Fueling App 和 Driving Task App

**题外话**｜CarPlay 是为驾驶员打造的，当你在构建 CarPlay App 时，你应该考虑的主要用户是驾驶员。因此，你应该只在 CarPlay 上启用与驾驶时相关的功能，忽略在驾驶时不应该做的任何功能，忽略复杂的、不常见的功能。诸如一次性配置、登录或阅读条款等事情最好在驾驶之前或之后执行，因此它们不应该出现在你的 CarPlay App 中。请注意，你的 App 需要授权才能显示在 CarPlay 中。你可以在 Apple CarPlay Developer 上根据你的 App 类型申请授权。

在这之前，有 6 种支持 CarPlay 的 App 类型：

![20220628141415.jpg](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2bf5619542804a899d3e8e8513b6248f~tplv-k3u1fbpfcp-watermark.image?)

今年，Apple 新增了 2 种支持 CarPlay 的 App 类型：Fueling App 和 Driving Task App。

![20220628141428.jpg](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/21a0ed99eef34c058f869ac71f231648~tplv-k3u1fbpfcp-watermark.image?)

### 题外话｜Templates

Templates（模版）是在 CarPlay App 呈现 UI 的方式。你的 CarPlay App 负责提供数据，Templates 负责代表你将 UI 绘制到汽车的显示屏上。你的 CarPlay App 可以很方便地接入 Templates，并且有几个好处：

* Templates UI 不像 UIKit UI 那样可高度定制化，不用担心设计给你花里胡哨的界面了。Templates UI 的复杂度低，通过简单的 Templates API 即可让你在 CarPlay App 中呈现 UI
* 不必担心字体大小问题，Templates 会自己适配不同的汽车显示屏
* 使用 Templates，你的 CarPlay App UI 与其他 CarPlay App UI 风格保持一致，可以让你的用户更容易上手使用你的 CarPlay App
* Templates 会负责确保你的 CarPlay App UI 在任何支持 CarPlay 的汽车中都能正常呈现和使用，无论汽车中使用的显示屏类型和大小如何

总之，Templates 会为你处理大部分工作。

在构建 App 时，有多种 Templates 可供选择。例如，显示按钮数组的 Grid Template（下图第 1 行第 4 个），显示表格的 List Template （下图第 2 行第 1 个）等等。

![2022062815492444.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5758885944044755a47e59c0123bfa2b~tplv-k3u1fbpfcp-watermark.image?)

无论作为开发人员还是 iOS 用户，你都应该熟悉这些 Templates。最重要的是，你的 CarPlay 用户会熟悉它们，因为它们出现在整个 CarPlay 车载中。

有些 Templates 是所有 App 类型通用的，而有些 Templates 仅限于特定类型的 App 才能使用。具体可以通过以下图表查看。

![20220628142803.jpg](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5f2df0e10ea340439c181feaa41fb21b~tplv-k3u1fbpfcp-watermark.image?)

### Fueling App

**EV Charging App**｜EV Charging App 是 Apple 在 iOS14 新增的支持 CarPlay 的 App 类型。这类 App 可以用于查找电车充电站的位置，还可以帮助驾驶员将汽车连接到正确的充电站并启动它。

这种类型的功能不仅适用于电车，它还适用于油车。于是 Apple 今年让 Fueling App 也支持了 CarPlay，主要用于帮助驾驶员给汽车加油。请注意，Fueling CarPlay App 除了支持查找加油站位置外，还应该有启动加油机等功能。

### Driving Task App

Driving Task App 是一种新型 CarPlay App，它可以支持各种各样可以完成简单任务的 App。 但你仍然需要注意的是，这些 CarPlay App 的主要目的必须是实现驾驶员在驾驶时需要完成的任务，它是用于真正有助于驾驶的任务，而不仅仅是在你（开发者）驾驶时需要完成的任务。

以下是属于此类型的一些 App 示例：

* 帮助控制汽车配件的 App
* 提供驾驶或道路状态和信息的 App
* 在驾驶开始或结束时帮助完成任务的 App

下面让我们看一些更具体的例子。

#### Road Status App

Road Status App 可以通知用户重要的道路信息。这个 CarPlay App 是使用 CPPointOfInterestTemplate 构建的。注意，使用此 CarPlay App 的用户正在开车，因此这样的 CarPlay App 应该提供一个非常简短的用户当前所在位置附近的重要的道路信息列表。这不适用于帮助用户在开车前进行完整路线规划的 App。

![20220628143633.jpg](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/957fca709c8e4ffdb2c750ae7dd8a7ce~tplv-k3u1fbpfcp-watermark.image?)

在这个 CarPlay App 中，这是用户在选择位置时看到的内容，提示信息建议言简意赅。

![20220628143523.jpg](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a9654090dea64e8293323793d2f9b497~tplv-k3u1fbpfcp-watermark.image?)

####  Trailer Controller App

Trailer Controller App 可以用于控制汽车配件。这个 CarPlay App 使用 CPInformationTemplate 提供有关已连接配件的基本信息，并提供了两个按钮来给用户采取操作。这个 CarPlay App 的 UI 和功能就是如此简单。当然，该 iPhone App 还有许多其他功能，但与驾驶不相关的功能不在 CarPlay App 中提供。用户可以在离开车辆或停车时使用该 iPhone App 中的更多功能。

![20220628143811.jpg](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a60ce7bbb308463ba6d167ba10286caa~tplv-k3u1fbpfcp-watermark.image?)

#### Mileage Logger、Express Lane

最后，让我们看几个使用 CPGridTemplate 的示例。

Mileage Logger 是一个非常简单的 CarPlay App，就只有两个按钮。它支持让驾驶员将他们的里程记录为个人里程或商务里程。此 App 非常适合新的 Driving Task App 类型，因为它可以让用户在驾驶时非常方便地完成简单的任务。

![20220628143915.jpg](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c369fc2cc2294620acfb0aad483a90ea~tplv-k3u1fbpfcp-watermark.image?)

Express Lane App 与 Mileage Logger App 的 UI 近似，它是一个快速车道收费应答器 CarPlay App，使用 CPGridTemplate 让用户选择车内有多少名乘客。这个 CarPlay App 的功能也非常简单方便，也是一个完美的 Driving Task App。

![20220628144018.jpg](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/63ea594ddf684870b70ee866cc2a3cbc~tplv-k3u1fbpfcp-watermark.image?)

#### 小结

回顾一下，在设计你的 Driving Task App 时，你需要注意以下几点：

* 一定要考虑制作一个单屏 CarPlay App，提供在驾驶时所需的最少功能
* 只提供可以在几秒钟内完成的功能
* 避免提供复杂或不常用的功能，例如首次设置或详细配置等
* 不应该提供驾驶时不需要的功能，即使它与汽车相关

## 探索 Navigation App 如何在受支持车辆中的数字仪表盘上实时绘制地图

![20220628135234.jpg](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/16f6e62ad22a4da9b7ee019932ef5796~tplv-k3u1fbpfcp-watermark.image?)

**CarPlay Dashboard**｜在 iOS 13 中，Apple 添加了 API 以支持 Navigation App 在 CarPlay Dashboard 中绘制地图。你可以编辑 Info.plist 来添加声明对 Dashboard 的支持，并添加所需的 Scene session role，然后实现所需的 delegate。当 Navigation CarPlay App 在 Dashboard 中出现或消失时，系统会通知你的 delegate，并将 UIWindow 传递给你，你可以在其中绘制地图内容。

![20220628153406.jpg](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/29b656a687774d40b85af5369a853319~tplv-k3u1fbpfcp-watermark.image?)

如果你之前已经对 Dashboard 做了支持，那么添加对 Instrument Cluster 的支持将是小菜一碟，因为它的实现方式与 Dashboard 类似。

首先在 Info.plist 的 Application scene manifest 中添加声明对 Instrument Cluster 的支持（`CPSupportsInstrumentClusterNavigationScene = true`），并添加所需的 Scene session role。

```xml
<key>UIApplicationSceneManifest</key>
<dict>
    <!-- Indicate support for CarPlay dashboard -->
    <key>CPSupportsDashboardNavigationScene</key>
    <true/>
    <!-- Indicate support for instrument cluster displays -->
    <key>CPSupportsInstrumentClusterNavigationScene</key>
    <true/>
    <!-- Indicate support for multiple scenes -->
    <key>UIApplicationSupportsMultipleScenes</key>
    <true/>
    <key>UISceneConfigurations</key>
    <dict>
        <!-- For device scenes -->
        <key>UIWindowSceneSessionRoleApplication</key>
        <array>
            <dict>
                <key>UISceneClassName</key>
                <string>UIWindowScene</string>
                <key>UISceneConfigurationName</key>
                <string>Phone</string>
                <key>UISceneDelegateClassName</key>
                <string>MyAppWindowSceneDelegate</string>
            </dict>
        </array>
        <!-- For the main CarPlay scene -->
        <key>CPTemplateApplicationSceneSessionRoleApplication</key>
        <array>
            <dict>
                <key>UISceneClassName</key>
                <string>CPTemplateApplicationScene</string>
                <key>UISceneConfigurationName</key>
                <string>CarPlay</string>
                <key>UISceneDelegateClassName</key>
                <string>MyAppCarPlaySceneDelegate</string>
            </dict>
        </array>
        <!-- For the CarPlay Dashboard scene -->
        <key>CPTemplateApplicationDashboardSceneSessionRoleApplication</key>
        <array>
            <dict>
                <key>UISceneClassName</key>
                <string>CPTemplateApplicationDashboardScene</string>
                <key>UISceneConfigurationName</key>
                <string>CarPlay-Dashboard</string>
                <key>UISceneDelegateClassName</key>
                <string>MyAppCarPlayDashboardSceneDelegate</string>
            </dict>
        </array>
        <!-- For the CarPlay instrument cluster scene -->
        <key>CPTemplateApplicationInstrumentClusterSceneSessionRoleApplication</key>
        <array>
            <dict>
                <key>UISceneClassName</key>
                <string>CPTemplateApplicationInstrumentClusterScene</string>
                <key>UISceneConfigurationName</key>
                <string>CarPlay-Instrument-Cluster</string>
                <key>UISceneDelegateClassName</key>
                <string>MyAppCarPlayInstrumentClusterSceneDelegate</string>
            </dict>
        </array>
    </dict>
</dict>
```

然后实现你的 Application instrument cluster scene delegate。API 会提供一个 UIWindow 给你来绘制你的地图内容，并在 Instrument Cluster 启动和关闭时通知你。你可以将你的地图实时绘制到 Instrument Cluster 中。

```swift
extension TemplateApplicationSceneDelegate: CPTemplateApplicationInstrumentClusterSceneDelegate {
    
    func templateApplicationInstrumentClusterScene(
        _ templateApplicationInstrumentClusterScene: CPTemplateApplicationInstrumentClusterScene,
        didConnect instrumentClusterController: CPInstrumentClusterController) {
        // Connected to Instrument Cluster
        TemplateManager.shared.clusterController(instrumentClusterController, didConnectWith: templateApplicationInstrumentClusterScene.contentStyle)
    }
    
…

    func instrumentClusterControllerDidConnect(_ instrumentClusterWindow: UIWindow) {
        // Window in which to draw instrument cluster contents 
       self.instrumentClusterWindow = instrumentClusterWindow
    }
}
```

虽然对 Instrument Cluster 与 Dashboard 添加支持的实现非常相似，但有一些特定于 Instrument Cluster 的注意事项。

* 首先，Instrument Cluster 支持用户放大和缩小地图，因此你需要使用 CPInstrumentClusterControllerDelegate 在你的 CarPlay App 中实现此功能。
* 同样，如果你的 CarPlay App 支持显示指南针或速度限制，系统会在适当的时间通知你的 delegate 来绘制它们。
* 最后，请注意，你的 Instrument Cluster 视图可能会被汽车 Instrument Cluster 中的其他元素部分遮挡。当然，iOS 已经有了一流的机制来处理这种事情，那就是安全区域。你可以在视图控制器上重写 `viewSafeAreaInsetsDidChange` 来监听安全区域的改变，并在 Cluster View 上使用 `safeAreaLayoutGuide` 以保证视图区域中的关键内容可见而不被遮挡。

## CarPlay Simulator

![carplay_simulator.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c42456f739ec426eab9c0d2581e23b55~tplv-k3u1fbpfcp-watermark.image?)

这是该今年 CarPlay 更新的重磅，所有类型的 CarPlay App 都能用上它。

在这之前，你有 2 种方式来测试 CarPlay App：

1. Xcode iPhone Simulator 内置有 CarPlay Window，可以很方便地快速测试你的 CarPlay App
2. 通过将你的 iPhone Device 连接到支持 CarPlay 的汽车或售后主机上来测试 CarPlay App，这是之前在你的 iPhone Device 上测试 CarPlay App 的唯一方式

今年，Apple 给我们带来了 CarPlay Simulator，它是一个 Mac App，可模拟汽车中的 CarPlay 的完整环境。你可以在 Apple 开发者网站上下载 [Xcode 附加工具包 - Additional Tools for Xcode 14 beta](https://developer.apple.com/download/all/)。

![20220628144938.jpg](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4096db6cfcd64c46ba1a4fcaab0fafdf~tplv-k3u1fbpfcp-watermark.image?)

然后打开它，在 Hardware 文件夹下可以找到 CarPlay Simulator。

![20210628093610.jpg](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/31bd5dc71ef14aaf87e977ff67992de1~tplv-k3u1fbpfcp-watermark.image?)

![20210628093642.jpg](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/abd512da8b2d4a309ead345b7901bc96~tplv-k3u1fbpfcp-watermark.image?)

运行该 App，然后将你的 iPhone Device 通过数据线连接到你的 Mac，即可启动 CarPlay。

**注**｜你不需要升级 MacOS 系统及 Xcode 版本也能使用 CarPlay Simulator。

![20220628145204.jpg](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bb219deb0c9641089c9bf3a0a3646147~tplv-k3u1fbpfcp-watermark.image?)

使用 CarPlay Simulator 有几个好处：

* 当你使用 CarPlay Simulator 时，你的 iPhone device 已经连接到你的 Mac，你可以同时使用 Mac 上的其它开发工具，例如在 Xcode 中调试或在 Instruments 中测试性能。
* 在现有的 iPhone Simulator 内置的 CarPlay Window 上无法测试某些场景。例如，你需要测试你的 Navigation App 的语音指令是否与汽车的原生音频源（如 FM radio）正确混合，在这之前，你可能要经常来回跑到你的车上或购买售后主机进行测试。而现在，你可以使用 CarPlay Simulator，CarPlay 实际上是在你的 iPhone Device 上运行的，就像在汽车中一样。这意味着你现在可以很方便的在办公位上进行测试。
* 你还可以使用 CarPlay Simulator 来测试多种不同配置的汽车，例如在不同尺寸的显示屏上进行测试。

下面来介绍下 CarPlay Simulator 如何使用。它的窗口界面由 3 个部分组成：

* 窗口中间是 CarPlay 视图，模拟汽车显示屏
* 窗口底部是模拟汽车中各种不同硬键和旋钮的按钮
* 窗口顶部有一些按钮
  * Configure：配置按钮，点击将弹出一个具有更高级功能的辅助窗口，稍后详细讲解
  * Limit UI Off/On：模拟限制 CarPlay 上的某些内容，例如缩短音频 CarPlay App 中列表的内容
  * Light/Dark UI：模拟在汽车的 CarPlay 中切换 UI 的深浅色外观
  * Light/Dark Map：模拟在汽车的 CarPlay 中切换地图的深浅色外观
  * Connected/Disconnected：模拟 iPhone Device 与汽车 CarPlay 断开连接或重新连接，而不需要拔插数据线。因为当你使用此按钮时你的 iPhone Device 仍将保持与 Mac 的连接，所以你可以使用它在 Xcode 调试 CarPlay Scene 的断开连接或重新连接。

![WX20220628-230108.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/adc44f7d154f44fcba1cebcefa5d96a4~tplv-k3u1fbpfcp-watermark.image?)

### Configure

点击主窗口中的 Configure 按钮即可打开一个具有更高级功能的辅助窗口。

![20220628154833.jpg](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e71bfb0a4f51453b875e12f802d4c4ee~tplv-k3u1fbpfcp-watermark.image?)

在 General tab 中，你可以设置 CarPlay 显示屏的尺寸。如果你的 CarPlay App UI 仅由 Templates 组成，Templates 会负责确保你的 UI 在任何支持 CarPlay 的汽车都能正常呈现和使用，不过你仍然可以尝试配置不同的尺寸来查看你的 UI 在不同汽车中的显示效果。但是，如果你的 App 是 Navigation App，那么尝试配置不同的尺寸大小和宽高比来测试并确保地图绘制的代码正常工作是至关重要的。

![20220628154842.jpg](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7a08626b419c45da92bde0e40efb8594~tplv-k3u1fbpfcp-watermark.image?)

以下是一些推荐的用于测试 Navigation App 的显示尺寸：

* 800 x 480（default）
* 1280 x 720
* 900 x 1200

在 Cluster Display tab 中，你可以勾选 Instrument Cluster Display enabled 并点击 Restart Session 按钮，这时候会打开一个新的窗口，它用于模拟汽车的仪表盘显示屏。

![20220628154918.jpg](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/379fba09b50a4784afa1bdd8574b3bc6~tplv-k3u1fbpfcp-watermark.image?)

![20220628154924.jpg](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/533151b1cc3b4ed191b45f6dc9f4b66a~tplv-k3u1fbpfcp-watermark.image?)

这个功能是与 Navigation App 相关的，仪表盘显示屏用于在汽车仪表盘的视野中为驾驶员显示地图或转向卡。

Restart Session 按钮用于当你修改 Configure 中的配置后，使其立即生效，而不需要重新启动 CarPlay Simulator。

以上就是 CarPlay Simulator 的相关内容，更多功能就交由你自己探索。

## 使用 CarPlay Simulator 对 CarPlay App 进行测试

启动 CarPlay Simulator。

![20220628112419.jpg](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2b3e24f13601442f966871ba85fc6cda~tplv-k3u1fbpfcp-watermark.image?)

连接你的 iPhone Device 到 Mac，CarPlay 会随之启动。

![20220628112601.jpg](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/712038a060bf435b81f1b3b7182e9e5d~tplv-k3u1fbpfcp-watermark.image?)

如果你的 CarPlay App 为深浅色外观提供了两套 icon，那么你可以通过 Light/Dark UI 按钮来切换深浅色外观，以进行测试。不过在此之前，你需要先到 CarPlay 设置中设置外观模式为 `自动`，而不是 `始终深色模式`。

![20220628112724.jpg](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/528c38871cf6483a917d335d9a04de0d~tplv-k3u1fbpfcp-watermark.image?)

![20220628112639.jpg](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8716a200bc8545d68c9579b541c9431e~tplv-k3u1fbpfcp-watermark.image?)

你可以在 Configure - General tab 中设置 CarPlay 显示屏的尺寸，然后点击 Restart Session 按钮就能使其立即生效，而无需重启 Carplay Simulator，这很方便。你可以通过此功能来测试你的 CarPlay App 特别是 Navigation App 是否良好地适配了不同尺寸的汽车显示屏。

![20220628115604.jpg](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/14457fe2ab144ca8ba68b1d0c938659d~tplv-k3u1fbpfcp-watermark.image?)

![20220628115614.jpg](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/db8674e8796e4c3ab9c02d016c3f363f~tplv-k3u1fbpfcp-watermark.image?)

你可以在 Cluster Display tab 中启用 Instrument Cluster Display 来测试你的 Navigation App 对汽车 Instrument Cluster 的支持。

![20220628115640.jpg](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0aa6cb08e50d40928563678ba91056e3~tplv-k3u1fbpfcp-watermark.image?)

在你使用 CarPlay Simulator 对你的 CarPlay App 进行全面测试且情况良好后，你可以完全相信你的 CarPlay App 会在真正的汽车中运行良好。不过建议你还是去车上试一试。

![20220628135107.jpg](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8823c6c6539c45bf9b23a1ecc15885cd~tplv-k3u1fbpfcp-watermark.image?)

Templates 会负责确保你的 CarPlay App UI 在任何支持 CarPlay 的汽车中上都能正常呈现和使用。

![20220628135123.jpg](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4d99c61f763a48e192a9a3ecff3abd12~tplv-k3u1fbpfcp-watermark.image?)

一些用户可能会喜欢使用汽车的旋钮来操作 CarPlay App。如果你已经使用 CarPlay Simulator 中的模拟旋钮进行测试，那么它在汽车中的表现也一定是不错的。Templates 会帮你处理好旋钮事件。

![20220628135132.jpg](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a4ce841a0f6148c6b27a1ad5f7952575~tplv-k3u1fbpfcp-watermark.image?)

![20220628135152.jpg](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4d4c34aaab334c089f950be6f332a4a3~tplv-k3u1fbpfcp-watermark.image?)


你还可以测试下你的 Navigation App 对汽车数字仪表盘的支持效果。对驾驶员来说，将实时地图显示在他的视线范围内是非常棒的，使用你的 Navigation App 的驾驶员一定会喜欢它。

![20220628135234.jpg](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e8246be568e64617b308e7fc524b5bed~tplv-k3u1fbpfcp-watermark.image?)

![20220628135241.jpg](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4a68ab8d6e5d4585b6d1820557a57c8f~tplv-k3u1fbpfcp-watermark.image?)

## 参考 

* [WWDC22｜10016 - Get more mileage out of your app with CarPlay](https://developer.apple.com/videos/play/wwdc2022/10016/) 
* [Apple｜CarPlay App Programming Guide](https://developer.apple.com/carplay/documentation/CarPlay-App-Programming-Guide.pdf)
* [iOS CarPlay｜与你分享 CarPlay 音频 App 的开发过程与细节](https://juejin.cn/post/7035671279218720805)