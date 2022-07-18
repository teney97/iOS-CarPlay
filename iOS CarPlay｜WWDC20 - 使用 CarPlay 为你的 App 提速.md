![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167921395-assets/web-upload/4770bd80-5f39-4b0a-9d6e-c607daa3f30b.png)

## 前言

本文是对 [WWDC20｜10635 - Accelerate your app with CarPlay](https://developer.apple.com/wwdc20/10635) 的梳理。如果你正要构建一款 CarPlay app，那么该 session 你一定不能错过。Apple 在 iOS 14 中为 CarPlay 带来了大更新，新增了电车充电、停车和快速点餐三种支持 CarPlay 的 app 类型，并首次将 CarPlay framework 提供给除导航外的其它类型的 app 使用。本文将向你介绍 CarPlay framework 新增的 template 以及对旧 template 的种种改进，并教你使用这些 template 来开发受支持的各种类型的 CarPlay app。

本文会尽量还原 session 的内容，并适当扩展一些内容，但不会做太多的扩展。如果你想了解更多有关 CarPlay 的内容，可以阅读笔者的 CarPlay 专栏下的其它文章。

## 概要

Apple 于 iOS 12 首次引入 CarPlay framework 用于也仅限于开发导航类 app。你可能在 CarPlay 中使用过一些优秀的第三方导航 app，如 Waze 和 Google 地图，它们就是基于 CarPlay framework 提供的 template 来构建的。

在 iOS 14 中，Apple 将为 CarPlay 带来如下更新：

* 新增了电车充电、停车和快速点餐三种支持 CarPlay 的 app 类型
* 首次将 CarPlay framework 提供给除导航外的其它类型的 app 使用。除新增的 app 类型外，你还可以使用 CarPlay framework 升级你的音频和通信 CarPlay app
* 为各种类型的 CarPlay app 新增了一些通用或特定的 template
* 对一些旧 template 做了种种改进，使其在不同类型的 CarPlay app 中能够发挥不同的作用

**注**｜为了便于阅读，本文将按照 app 类型划分目录，读者可以仅阅读自己关心的 app 类型在 CarPlay 中的更新。

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167090875-assets/web-upload/ec98a7eb-0e9a-4253-9f93-c4712cdbfa0c.png)

## 设计原则

首先，我们来回顾一下在 CarPlay 中构建 app 时应该牢记的一些设计原则。

* **为驾驶员设计**。CarPlay 是为驾驶员而不是为乘客设计的，你应该只在 CarPlay app 中提供有助于驾驶的功能。
* **简化交互**。CarPlay app 是具有针对性的，你的 CarPlay app 应从最常见的场景启动，并简化每个交互操作到能够由驾驶员在几秒内完成。
* **复用 app 配置**。尽可能复用 iPhone app 的配置，以最小化用户在 CarPlay 中需要进行的任何设置操作。用户对你的 app 的初体验应当在开始驾驶之前就已经在 iPhone 端完成了。
* **去除 app 中依赖于在 iPhone 上启动的任何逻辑**。app 可能会首先并且只在 CarPlay 中启动。当用户在 CarPlay 主屏幕上点击你的 app 时，你的 app 将会启动并连接到一个 car scene，而不会连接 iPhone scene。因此，去除 app 中依赖于在 iPhone 上启动的任何逻辑非常重要。使用 UIScene 很容易做到这一点。实际上，你的 app 必须采用 UIScene 才能使用 CarPlay framework。所以，你必须从传统的 UIWindow 和 UIApplicationDelegate API 向 UIScene 过渡。

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167097487-assets/web-upload/4af1e461-357d-4b9b-ba4d-5b5fedd8ae75.png)

## CarPlay framework 初体验

前面提到过，你的 app 必须采用 UIScene 才能使用 CarPlay framework。采用 UIScene 的重要一环是在 info.plist 中为你的 app 声明一份场景清单（Scene Manifest）。

除了 iPhone scene 之外，你还需要在场景清单中提供一个 CarPlay scene 配置，如以下示例所示。

```xml
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

**注**｜如果你正在构建导航 CarPlay app，你还可以为 CarPlay dashboard 提供一个 scene 配置。

在 CarPlay scene 配置中，我们需要指定一个类名称，该类在你的 app 中充当 CarPlay scene 的 scene delegate。

然后，我们需要实现在 scene 配置中指定的代理类，并做以下几步：

* 实现 CPTemplateApplicationSceneDelegate 协议中的 `didConnect` 方法。当你的 app 在 CarPlay 屏幕上启动时，CarPlay framework 会调用此方法。
  * 首先，你可能需要保留方法参数中 CPInterfaceController 对象，因为稍后你将需要它。
  * 接着，这里将一个 CPListTemplate 实例设置为 app 的 rootTemplate。
* 实现 `didDisConnect` 方法，在该方法中释放刚刚保留的 CPInterfaceController 对象，

示例代码如下：

```swift
// CarPlay App Lifecycle

import CarPlay

class CarPlaySceneDelegate: UIResponder, CPTemplateApplicationSceneDelegate {
    var interfaceController: CPInterfaceController?
   
    func templateApplicationScene(_ templateApplicationScene: CPTemplateApplicationScene,
            didConnect interfaceController: CPInterfaceController) {

        self.interfaceController = interfaceController
        let listTemplate: CPListTemplate = ...
        interfaceController.setRootTemplate(listTemplate, animated: true)
    }

  	func templateApplicationScene(_ templateApplicationScene: CPTemplateApplicationScene,
            didDisconnect interfaceController: CPInterfaceController) {
    		self.interfaceController = nil
		}
}
```

CPListTemplate 是列表样式的 template。CPListItem 是组成 CPListTemplate 的一种基本单元。每个 CPListItem 都代表 CPListTemplate 中的一行，它们的关系就像 UITableViewCell 与 UITableView。

以下是创建一个 CPListTemplate 的示例代码：

```swift
// CPListTemplate

import CarPlay

let item = CPListItem(text: "Rubber Soul", detailText: "The Beatles") 
let section = CPListSection(items: [item]) 
let listTemplate = CPListTemplate(title: "Albums", sections: [section]) 
self.interfaceController.pushTemplate(listTemplate, animated: true)
```

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167088589-assets/web-upload/f885fa2d-3d66-49c6-bb15-d5c9e8426f6f.png)

当用户点击一个 CPListItem 时，CPListItem 的属性 listItemHandler block 会被调用。listItemHandler block 接收两个参数，第一个是被点击的 listItem，第二个是 completion block。当一个 listItem 被点击时，你的 app 可能会执行一些任务，例如：

* 如果是音频 CarPlay app，播放被点击的 listItem 对应的音频
* 跳转到一个新的 template

当一个 listItem 被点击时，CarPlay 屏幕上将会显示一个加载指示器，指示用户你的 app 当前正在加载中。当你调用了 completion block 后，加载指示器就会消失。因此，无论你执行完成了哪种任务，你都必须调用 completion block。

```swift
// CPListTemplate

import CarPlay

let item = CPListItem(text: "Rubber Soul", detailText: "The Beatles") 
item.listItemHandler = { item, completion, [weak self] in
    // Start playback, then...
    self?.interfaceController.pushTemplate(CPNowPlayingTemplate.shared, animated: true)
    completion()
}
```

## Template

使用 CarPlay framework，你的 CarPlay app 就是基于 template（模板）来构建的。Template 是在 CarPlay App 呈现 UI 的方式。你的 CarPlay app 负责提供数据，Template 负责代表你将 UI 绘制到汽车的显示屏上。

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658053819343-assets/web-upload/93eeafa8-d444-4af3-8af7-b874decb0159.png)

建议你前往 [iOS CarPlay｜通过 CarPlay 让你的 App 发挥更大的作用 - Template](https://juejin.cn/post/7114239495360233479#heading-2) 了解更多关于 Template 的介绍。

## All App

### CPListTemplate (improved)

#### CPListItem (improved)

在 iOS 14 中，listItem 支持动态更新了，包括 CPListItem 和 CPListImageRowItem 等等。CPListItem 上许多以前 readonly 的属性现在 readwrite 了。这很有用，举些音频 CarPlay app 的例子：

* 专辑封面图片需要从网络获取，可以先给 listItem 设置一个 placeholderImage，待网络图片加载完毕后再重新设置给 listItem。CarPlay 将仅自动重新加载 CPListTemplate 中需要更新的 listItem。

```swift
// CPListTemplate

import CarPlay

let item = CPListItem(text: "Rubber Soul", detailText: "The Beatles") 
let section = CPListSection(items: [item]) 
let listTemplate = CPListTemplate(title: "Albums", sections: [section]) 
self.interfaceController.pushTemplate(listTemplate, animated: true)

// Later...
item.image = ...
```

* Apple Podcasts（播客 app）使用动态更新来使进度指示器随着音频播放进度的更新而更新。

#### CPListImageRowItem (new)

CPListImageRowItem 是 iOS 14 中新增的 listItem，它是网格样式，支持显示一个标题和一组图片。

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167088163-assets/web-upload/055dcf3d-a4bb-48c5-bae8-c8028339481f.png)

### CPTabbarTemplate (new)

CPTabbarTemplate 是 iOS 14 中的新 template，它是一种容器模板，可以在 tab bar 风格的界面中显示几个其它的 template，让用户一目了然。CPTabbarTemplate 很适合作为 CarPlay interfaceController 的 rootTemplate。

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658067080784-assets/web-upload/1432eb45-fe09-4dd0-b117-093be72aca78.png)

接下来让我们通过一段示例代码，看看如何创建一个 CPTabbarTemplate。

* 首先，我们创建了两个 template，分别是 CPListTemplate 和 CPGridTemplate 类型。在 iOS 14 中，每种 template 都继承了一些新属性，支持你自定义 template 在 tab bar 中的显示样式，你可以设置 tabTitle、tabImage、tabSystemItem、showsTabBadge 等等。
* 接着，我们通过一个 template 数组来创建一个 CPTabbarTemplate，数组中的每个 template 都将成为 tab bar 上的一个 tab。
* 稍后，我们可以动态更新 CPTabbarTemplate，例如添加新的 tab、移除 tab、重新排列 tab 等等。还可以像该示例所示，显示或隐藏一个或多个 tab 的 badge。

```swift
// CPTabBarTemplate

import CarPlay

let item = CPListItem(text: "Rubber Soul", detailText: "The Beatles") 
let section = CPListSection(items: [item]) 
let favorites = CPListTemplate(title: "Albums", sections: [section])
favorites.tabSystemItem = .favorites
favorites.showsTabBadge = true

let albums: CPGridTemplate = ...
albums.tabTitle = "Albums"
albums.tabImage = ...

let tabBarTemplate = CPTabBarTemplate(templates: [favorites, albums])
self.interfaceController.setRootTemplate(tabBarTemplate, animated: false)

// Later...
favorites.showsTabBadge = false
tabBarTemplate.updateTemplates([favorites, albums])
```

## 音频 App

### 将 MPPlayableContent API 和 template 共存于你的 app 中

在 iOS 14 之前，如果你要开发音频 CarPlay app，那么你就会使用到 MediaPlayer framework 的 MPPlayableContent API。MPPlayableContent API 是一种基于元数据的 API。你负责描述你 app 的数据，比如专辑和歌曲，然后系统会代表你组装出一个树形 UI 界面。值得注意的是，如果你想在 iOS 13 及更早版本中让你的音频 app 支持 CarPlay，你可以将 MPPlayableContent API 和 template 共存于你的 app 中。

* 在 iOS 13 及更早版本中，系统将会通过 MPPlayableContent API 来启动你的 CarPlay app；
* 在 iOS 14 及更新版本中，如果你提供了 template，系统将会用它来启动你的 CarPlay app。

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658068581534-assets/web-upload/f09e22bc-3f2e-4502-a803-0c7c6036b2fe.png)

### CPListTemplate (improved)

#### CPListItem (improved)

针对音频 CarPlay app，CPListItem 新增了一些属性，例如播放进度指示器 [playbackProgress](https://developer.apple.com/documentation/carplay/cplistitem/3551779-playbackprogress) 和 Now Playing 的弹跳动画 [isPlaying](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.apple.com%2Fdocumentation%2Fcarplay%2Fcplistitem%2F3551780-isplaying)。

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658069090845-assets/web-upload/7485e27e-f747-4d9e-a4e0-7f9ac5b067f2.png)

#### CPListImageRowItem (new)

CPListImageRowItem 是 iOS 14 中新增的 listItem，它是网格样式。Apple Books（图书 app）就使用了这个 template 来展示用户最近播放的有声书，如下图所示。

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658069090835-assets/web-upload/ff087095-2084-400c-aa20-ba0245961b1c.png)

接下来让我们通过一段示例代码，看看如何创建一个 CPListImageRowItem 并添加到 CPListTemplate 中，它和 CPListItem 一样简单。

* 首先，我们通过一个标题和一组图片创建了一个 CPListImageRowItem。
* CPListImageRowItem 也有 listItemHandler block 来响应点击事件。当用户在 CPListImageRowItem 中点击标题时，listItemHandler block 将被调用。在 Apple Books 中，将打开一个新的 CPListTemplate 来显示更多最近播放的有声书。你需要确保 completion block 被调用。
* 在 CPListImageRowItem 中每张图片都是可以被独立点击的，listItemRowHandler block 就是用来响应每张图片的点击事件。同样的，你要确保 completion block 被调用。在 Apple Books 中，点击图片将打开对应的有声书进行播放。

```swift
// List Items for Audio Apps

import CarPlay

let gridImages: [UIImage] = ...
let imageRowItem = CPListImageRowItem(text: "Recent Audiobooks", images: gridImages) 

imageRowItem.listItemHandler = { item, completion in
    print("Selected image row header!")
    completion()
}

imageRowItem.listImageRowHandler = { item, index, completion in
    print("Selected artwork at index \(index)!")
    completion()
}

let section = CPListSection(items: [imageRowItem]) 
let listTemplate = CPListTemplate(title: "Listen Now", sections: [section]) 
self.interfaceController.pushTemplate(listTemplate, animated: true)
```

### CPNowPlayingTemplate (new)

CPNowPlayingTemplate 是 iOS 14 中供音频 CarPlay app 使用的新 template，它是音频 CarPlay app 的基石。

使用过音频 CarPlay app 的用户一定非常熟悉 Now Playing 界面。借助 CPNowPlayingTemplate，你可以更好地控制其外观和功能。

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658077486096-assets/web-upload/4840b1ad-a0ec-476c-aa10-7dde9186073d.png)

你的 app 可以选择启用“待播清单”和“艺人”按钮。

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658077486073-assets/web-upload/45cc9c97-fc3a-44b0-b066-26632fc0be93.png)

你的 app 也可以选择自定义底部的播放操作按钮。可以使用系统图标（系统提供了许多常见播放操作的图标）或自定义图标。

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658077486060-assets/web-upload/be68c992-51d8-45a5-a1d7-696bf956a470.png)

你需要记住 CPNowPlayingTemplate 的几个特性：

* CPNowPlayingTemplate 使用了单例模式，你应该使用和配置这个单例。
* 如果你启用了可选的“待播清单”和“艺人”按钮，那么你应该为这些按钮的操作添加至少一个观察者。
* 你应该在你的 CarPlay app 启动时（即 CarPlay scene 连接时）立即配置 CPNowPlayingTemplate。因为系统也许会代表你来显示 CPNowPlayingTemplate，甚至只是为了显示 CPNowPlayingTemplate 而启动了你的 app。
  * 例如，当用户点击 CarPlay 主界面上的 Now Playing 按钮时，系统会启动你的 CarPlay app，并且立刻显示 CPNowPlayingTemplate。
  * 又例如，当你的 app 变成了 Now Playing app，系统还会添加 Now Playing bar 按钮在你的 app 的 navigation bar 或 tab bar 上。如果用户点击了这个按钮，系统将会显示 CPNowPlayingTemplate。如果这时候其他 app 变成了 Now Playing app，系统会自动移除该 Now Playing bar 按钮。
* 在 template stack 中，你只能 push 一个 CPListTemplate 到 CPNowPlayingTemplate 之上。如果你的 app 启用了“待播清单”按钮，在 CPNowPlayingTemplate 中 push 一个新的 CPListTemplate 来向用户展示接下来的播放队列就是一个不错的方法。

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167097469-assets/web-upload/d1020ddc-bfc8-440b-b4c8-598d289ad329.png)

在《CarPlay framework 初体验》章节中，我们看到了 CarPlay app 生命周期的代码。在 `didConnect` 方法中就是配置 CPNowPlayingTemplate 单例的最佳时机。

在以下示例代码中，我们给 CPNowPlayingTemplate 配置了一个播放速度按钮，当用户点击它时会调用 completion block。完成初始配置后，你的 app 就已经准备好让系统代表你来呈现 CPNowPlayingTemplate 了，并且在这之后你也能随时更新 CPNowPlayingTemplate。

```swift
// Now Playing Template

import CarPlay

class CarPlaySceneDelegate: UIResponder, CPTemplateApplicationSceneDelegate {

    func templateApplicationScene(_ templateApplicationScene: CPTemplateApplicationScene,
            didConnect interfaceController: CPInterfaceController) {
      
        let nowPlayingTemplate = CPNowPlayingTemplate.shared

        let rateButton = CPNowPlayingPlaybackRateButton() { button in
                                                           
            // Change the playback rate!
                                                           
        }
        nowPlayingTemplate.updateNowPlayingButtons([rateButton])
    }
}
```

## 通信 App

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167096792-assets/web-upload/fb74438a-45e0-4b98-bc62-d93b48c8ef96.png)

在 iOS 14 中，你可以使用 Carplay framework 对你的通信 Carplay app 进行升级，以在汽车显示屏中提供更好的通信体验。

在 iOS 14 之前，通信 app 在 CarPlay 中是作为信息和语音 app 存在。这些 app 通常是基于 SiriKit 和 CallKit 实现功能的，通信 app 也必须继续使用 SiriKit 和 CallKit 来提供语音和电话功能。从 iOS 14 开始，它们也可以使用 CarPlay framework 来显示联系人、信息列表以及信息状态。

### CPListTemplate (improved)

#### CPMessageListItem (new)

通信 app 最常见的一个功能是显示消息列表。在 iOS 14 中，Apple 引入了一个全新的 CPListItem 的子类，叫做 CPMessageListItem，用于构建消息列表。

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167096859-assets/web-upload/eb096201-b357-44e5-a976-1a48f3979f9e.png)

当用户点击 CPMessageListItem 时不会调用 completion block，而是根据你在 CPMessageListItem 中指定的参数来自动激活 Siri。用户也将继续通过 Siri 编辑、阅读或回复信息。

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167096775-assets/web-upload/a33391a0-2be1-49d8-9e95-b745d7b6be16.png)

在 CPMessageListItem 的左侧，你可以选择显示未读标志、图钉（置顶）、星标（收藏）或其它图标。你可以根据你的 app 的功能来启用它们。

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167096546-assets/web-upload/b352b6df-da62-4150-9e16-f1b4e679f008.png)

在 CPMessageListItem 的右侧，你可以选择显示静音标志、文本或者可选图标。和其他 item 一样，CPMessageListItem 也支持动态更新，你只需更新它的属性便能轻松地改变一条信息的元素。

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167096619-assets/web-upload/e73fb43f-fe76-45ea-bd15-17451fc13043.png)

### CPContactTemplate (new)

通信 app 的另一个重要功能是展示联系人信息。CPContactTemplate 是 iOS 14 中的新 template，它就是为了实现这个功能而生。

你可以使用 CPContactTemplate 来显示联系人头像、描述文本、一组操作按钮和导航栏按钮。该 template 最多支持三行文本以及四个按钮，这些文本和图标都可以自定义。

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167096546-assets/web-upload/05a61010-fce1-486f-8319-bba4f271e981.png)

## 电车充电、停车和快速点餐 App

在 iOS14 中，Apple 新增了三种支持 CarPlay 的 app 类型：

* 电车充电
* 停车
* 快速点餐

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167097173-assets/web-upload/590ff989-2e56-4e92-a518-de2b315e3ed9.png)

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167096282-assets/web-upload/d47e48fb-a13b-4a15-8420-f7f095bfcc85.png)

这几种 app 有许多共同点，它们都与驾驶体验密切相关，可以帮助驾驶员在地图上找到目的地。此外，这些 app 可以查询汽车充电桩可用性、预约充电、停车或点餐。iOS 14 中推出了一组新的 template，可以让你的 app 在 CarPlay 中提供这些功能。

接下来我们用一个快速点餐 CarPlay app 的示例来讲解如何使用新的 template 来开发这几种 app。这个示例 app 在显示屏顶部有 4 个 tab，分别是附近商店的位置、购买记录、收藏列表和订单信息。这个 CarPlay app 是经过精简的，不显示详细的食品菜单、账户管理或其他设置，只提供最常用的功能且操作简单。

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167094130-assets/web-upload/d59234ce-db96-41ee-a3dd-07f3b97b8d98.png)

### CPPointOfInterestTemplate (new)

无论用户是在寻找附近的得来速餐厅、电动汽车充电站还是停车场，app 的首要任务都是定位。

CPPointOfInterestTemplate 是 iOS 14 中的新 template，它结合了 MapKit framework 提供的交互式地图和信息面板，可以用来显示附近的地点并支持让用户选择。你的 app 提供要在地图上显示的地点列表和可选图标。这个 template 还提供了平移和缩放功能，并且会在地图区域发生变化时通知你。

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167097159-assets/web-upload/f55da0ca-16b3-45e8-82c3-58456a0605cd.png)

CPPointOfInterest（POI）是组成 CPPointOfInterestTemplate 的信息面板的基本单元。你需要通过一个 `[CPPointOfInterest]` 数组来创建一个 CPPointOfInterestTemplate 实例，且这个数组最多只能包含 12 个 POI 元素。其中，每一个 POI 元素都包含了一个标题、一个可选的副标题和一个将显示在详细信息面板上的信息文本。

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167096950-assets/web-upload/59d02b6f-827d-482e-af24-68ed78c74060.png)

**注**｜请记住，在 CarPlay 中只向驾驶员展示最相关的信息。在 iPhone 上，你的 app 也许会显示所有可能的地点；但在 CarPlay 中，你应该有所限制，只显示最相关或最近的地点。如果有多个地点在地图上显示挨得非常近的话，CarPlay 会自动将它们分组。

CPPointOfInterestTemplate 支持动态更新属性，就像其它 template 一样。

在该示例中，为了表示和区分某个地点是被选中的，我们更新被选中地点的 `pinImage` 属性，使用了一个不同的颜色（这里用了黄色，而其它未被选中的用红色）。

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167096916-assets/web-upload/e853cf5c-0d4a-4cb6-bcc2-0b0af433aca1.png)

当列表中的一个地点被选中时，你可以选择显示一个详细信息面板，上面显示关于该地点的详细信息和最多两个按钮。这些按钮可用于各种任务。比如：

* 可以切换到导航 CarPlay app 来导航到被选中地点。
* 可以 push 到另一个 template 来展示被选中地点的详细信息。

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167096937-assets/web-upload/c51fdc04-0b19-4301-9961-714c2db73722.png)

#### 更新地点列表

用户可以在 CPPointOfInterestTemplate 的上平移和缩放地图，这两个操作都会导致可视地图区域发生变化，你的 app 应该使用与发生变化后的可视区域对应的新的地点列表来更新这个 template。

下面我们来看看示例代码：

* 首先，给 template 设置一个遵循 CPPointOfInterestTemplateDelegate 协议的代理对象，该代理对象会在每次地图区域变化时得到通知。
* 然后，实现 `pointOfInterestTemplate(:didChangeMapRegion)` 方法来监听地图区域发生变化。在方法实现中，我们根据变化后的地图区域 `region` 来获取新的地点列表 `locations`，然后将其传递给 template。此时，CarPlay 将会使用新的 `locations` 更新你的地点列表和地图。

```swift
// CPPointOfInterestTemplateDelegate

func pointOfInterestTemplate(_ template: CPPointOfInterestTemplate, 
                             didChangeMapRegion region: MKCoordinateRegion) {

    self.locationManager.locations(for: region) { locations in
        template.setPointsOfInterest(locations, selectedIndex: 0)
    }
}
```

在以下示例代码中，我们实现了获取地点列表的方法。

* 首先，先构建一个 `[CPPointOfInterest]` 数组。
* 然后，通过查询你自己的地点数据库或者利用 MapKti 来确定附近的地点。
* 接着，遍历查询到的地点信息数据列表，使用这些数据来创建 CPPointOfInterest 实例并添加到数组中。你可以缓存 CPPointOfInterest 实例以重用。
* 最后，调用 handler 将 `[CPPointOfInterest]` 数组回调出去。

```swift
// CPPointOfInterest creation

func locations(for region: MKCoordinateRegion, 
               handler: ([CPPointOfInterest]) -> Void) {
    var tempateLocations: [CPPointOfInterest] = []
        
    for clientModel in self.executeQuery(for: region) {
        let templateModel : CPPointOfInterest = self.locations[clientModel.mapItem] ??
                CPPointOfInterest(location: clientModel.mapItem,
                                  title: clientModel.title,
                                  subtitle: clientModel.subtitle,
                                  informativeText: clientModel.informativeText,
                                  image: clientModel.mapImage)
            
            
        tempateLocations.append(templateModel)
    }
    handler(templateLocations)
}
```

更新地点列表就是这么简单，通过响应地图区域的变化，实时展示最新的 POI 列表，就能让用户找到附近的目的地。

#### 支持选中地点

当用户点击地点列表中的某个地点时，显示该地点的详细信息面板。我们可以通过设置 CPPointOfInterest 的可选按钮属性，给详细信息面板中添加一个 Select 按钮，供用户选择该地点。

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167096950-assets/web-upload/cfea2db9-87a2-441e-a49a-7b718c03e822.png)

示例代码如下：

* 首先，创建一个 CPPointOfInterestButton 实例，并赋值一个 completion block。completion block 在按钮被点击时调用。这里我们将被选中地点对应的 CPPointOfInterest 实例的 image 设置为被选中的样式，同时将 CPPointOfInterestTemplate 的 selectedIndex 设置为被选中地点的 index。CarPlay 将会在汽车显示屏上动态更新。
* 然后，将按钮赋值给 CPPointOfInterest 实例的 primaryButton 属性，该按钮就会显示在详细信息面板上。

```swift
// Point of Interest Template location selection

let primaryButton = CPPointOfInterestButton(title: "Select") { button, [weak self] in
            let selectedIndex = ...
            
            if selectedIndex != NSNotFound {
                // Remove any existing selected state on previous location
                self?.selectedLocation.image = defaultMapImage
                // Change annotation for selected POI
                self?.selectedLocation = templateModel
                templateModel.image = selectedMapImage
                // Update the template with new values
                self?.pointOfInterestTemplate.selectedIndex = selectedIndex
            }
        }

let templateModel: CPPointOfInterest = ...

templateModel.primaryButton = primaryButton
```

### CPInformationTemplate (new)

我们刚刚完成了显示和更新可视地图区域相关的地点列表以及支持选中地点的功能。现在，一旦用户选择了一个地点，或者执行了一个任务，你可能需要显示摘要或以其他方式完成交互。

CPInformationTemplate 也是 iOS 14 中的新 template。CPInformationTemplate 将许多内容集中在一个页面中，它由一列或两列的标签列表和一组页脚按钮组成，用于显示文本和接收用户响应。

在需要全屏交互的情况下，这个 template 可以满足很多需要：

* 就拿我们这个快速点餐 app 示例来说，它可以用作订单摘要，就如下图所示，或者可以用作订单确认页面。

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167096385-assets/web-upload/26c2d96b-6337-4315-9440-8bc3019fbc5c.png)

* 对于电车充电 app，它可用于展示有关充电站的重要信息。

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167095996-assets/web-upload/fce8f432-c5bd-4b45-8151-ca343d71863a.png)

## 构建 CarPlay App 的一些注意点

* 你需要为你的 app 申请一个 CarPlay 权限。CarPlay app 必须是单一类别的，你需要选择你的 app 所支持的类别。你所选择的权限将决定你的 app 可以使用哪些 CarPlay template。
* 如果你正在构建音频 CarPlay app，并且想要支持 iOS 13 及更早版本，那么 MPPlayableContent API 和 template 将共存于你的 app 中。

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167089726-assets/web-upload/a2241a4a-3bbf-464e-ac82-35e77d01f2aa.png)

## 总结

本文对 CarPlay framework 在 iOS 14 中的更新内容做了简要的介绍。如果你的 app 类别是 CarPlay 所支持的，那么就从 import CarPlay framework 开始，让你的 app 在 CarPlay 中大放光彩！

有关 CarPlay 的更多信息，可以查看 [CarPlay for developers](https://developer.apple.com/carplay/) 上更新的[《CarPlay app programming guide》](https://developer.apple.com/carplay/documentation/CarPlay-App-Programming-Guide.pdf)。在这个网站上，你还可以为你的 app 申请 CarPlay 权限，以及查看每个权限具体可以使用哪些 CarPlay template。

如果你需要将你的音频 CarPlay app 部署到 iOS 13 或更早版本中，建议你再次阅读 [MPPlayableContent API 说明文档](https://developer.apple.com/documentation/mediaplayer/mpplayablecontentmanager/)；如果你正在构建一个通信 CarPlay app，你的 app 必须使用 SiriKit，你可以在 CarPlay 开发网站上找到大量的说明文档和资源；如果你正在构建一个基于 template 的导航 CarPlay app，请观看 [WWDC18 - CarPlay 车载音频和导航 App](https://developer.apple.com/wwdc18/213)。

## 相关链接

* [WWDC20｜10635 - Accelerate your app with CarPlay](https://developer.apple.com/wwdc20/10635)
* [WWDC18｜213 - CarPlay Audio and Navigation Apps](https://developer.apple.com/wwdc18/213)
* [Apple Developer｜CarPlay for developers](https://developer.apple.com/carplay/)
* [Apple Developer｜CarPlay app programming guide](https://developer.apple.com/carplay/documentation/CarPlay-App-Programming-Guide.pdf)
* [Apple Developer｜MPPlayableContent API](https://developer.apple.com/documentation/mediaplayer/mpplayablecontentmanager/)

## 推荐阅读

* [师大小海腾的 GitHub｜iOS- CarPlay](https://github.com/teney97/iOS-CarPlay)
* [iOS CarPlay｜与你分享 CarPlay 音频 App 的开发过程与细节](https://juejin.cn/post/7035671279218720805)
* [iOS CarPlay｜WWDC22 - 通过 CarPlay 让你的 App 发挥更大的作用](https://juejin.cn/post/7114239495360233479)
* [iOS CarPlay｜WWDC20 - 使用 CarPlay 车载系统为你的 App 提速]()



