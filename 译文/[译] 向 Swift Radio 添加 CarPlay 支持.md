## [译] 向 Swift Radio 添加 CarPlay 支持

原文：[Fethi El Hassasna｜Add CarPlay support to Swift Radio](https://blog.fethica.com/add-carplay-support-to-swiftradio/#)

在本教程中，我们将看到如何向开源的广播 App [SwiftRadio](https://github.com/analogcode/Swift-Radio-Pro) 添加 **CarPlay** 支持，并使用模拟器对其进行测试。

### 设置项目

首先让我们从克隆项目开始，或者直接从 [GitHub](https://github.com/analogcode/Swift-Radio-Pro) 下载：

```
git clone https://github.com/analogcode/Swift-Radio-Pro
```

使用模拟器运行项目后，我们先检查下 **CarPlay** 菜单是不是位于以下位置：**Hardware (I/O) > External Displays > CarPlay**：

![](https://gitee.com/junteng/images/raw/master/img/20220107135013.png)

如果找不到 CarPlay 菜单，只需打开终端并运行以下命令：

```
defaults write com.apple.iphonesimulator CarPlay -bool YES
```

> [注] 现在应该是不需要这一步了。

现在我们需要向工程添加一个权利文件（.entitlement），向其添加条目 `com.apple.developer.playable-content` 并设置值为 `Boolean/YES` 以支持 **CarPlay**。

要自动生成文件，我们只需在工程 **target > capabilities** 中开启 **PushNotification** 然后再关闭即可。

![](https://gitee.com/junteng/images/raw/master/img/20220107140228.png)

> [注]：现在我们可以在 **target > Signing & Capabilities** 中随便添加一个 Capability，就自动生成 .entitlements 文件，然后再将添加的 Capability 删除即可。
>
> ![](https://gitee.com/junteng/images/raw/master/img/20220107140626.png)
>
> ![](https://gitee.com/junteng/images/raw/master/img/20220107141132.png)

该 `SwiftRadio.entitlements` 文件应如下所示：

![](https://gitee.com/junteng/images/raw/master/img/20220107142032.png)

当我们再次运行该 App 时，我们应该会看到我们的 **CarPlay** App：

![](https://gitee.com/junteng/images/raw/master/img/20220107142138.png)

接下来，让我们回到项目并开始添加一些代码。

首先，我们需要在 **AppDelegate** 类中导入 **MediaPlayer** framework，并添加一个新属性 `playableContentManager`。

```swift
// AppDelegate.swift

import UIKit
import MediaPlayer

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {

    var window: UIWindow?
    weak var stationsViewController: StationsViewController?
    var playableContentManager: MPPlayableContentManager?
    
    // ...
```

我们为 `AppDelegate` 创建一个 CarPlay 分类来分离 CarPlay 逻辑，将其命名为 `AppDelegate+CarPlay.swift`。

首先，我们添加一个 `setupCarPlay` 方法来初始化 `playableContentManager` 属性，同时设置其 `delagate` 和 `dataSource` 为 `self (AppDelagate)`:

```swift
// AppDelegate+CarPlay.swift

import Foundation
import MediaPlayer

extension AppDelegate {
    
    func setupCarPlay() {
        playableContentManager = MPPlayableContentManager.shared()
        
        playableContentManager?.delegate = self
        playableContentManager?.dataSource = self
    }
}
```

接下来，我们使用 extension 实现 `delagate` 和  `dataSource` 协议并添加所需的实现：

```swift
// AppDelegate+CarPlay.swift

import Foundation
import MediaPlayer

extension AppDelegate {
    
    func setupCarPlay() {
        playableContentManager = MPPlayableContentManager.shared()
        
        playableContentManager?.delegate = self
        playableContentManager?.dataSource = self
    }
}

extension AppDelegate: MPPlayableContentDelegate {
    
    func playableContentManager(_ contentManager: MPPlayableContentManager, initiatePlaybackOfContentItemAt indexPath: IndexPath, completionHandler: @escaping (Error?) -> Void) {
        completionHandler(nil)
    }
    
    func beginLoadingChildItems(at indexPath: IndexPath, completionHandler: @escaping (Error?) -> Void) {
    }
}

extension AppDelegate: MPPlayableContentDataSource {
    
    func numberOfChildItems(at indexPath: IndexPath) -> Int {
        return 0
    }
    
    func contentItem(at indexPath: IndexPath) -> MPContentItem? {
        return nil
    }
}
```

为了获取 `datasource` 需要的数据（我们例子中广播电台的列表），我们添加一个类，并将其命名为 `CarPlayPlaylist`。在这个类中，我们增加一个属性来保存电台数组，以及一个从我们 `DataManager` 类中加载数据的方法。

```swift
// CarPlayPlaylist.swift

import Foundation

class CarPlayPlaylist {
    
    var stations = [RadioStation]()
    
    func load(_ completion: @escaping (Error?) -> Void) {
        
        DataManager.getStationDataWithSuccess() { (data) in
            
            guard let data = data else {
                completion(nil)
                return
            }
            
            do {
                let jsonDictionary = try JSONDecoder().decode([String: [RadioStation]].self, from: data)
                if let stationsArray = jsonDictionary["station"] {
                    self.stations = stationsArray
                }
            } catch (let error) {
                completion(error)
                return
            }
            
            completion(nil)
        }
    }    
}
```

现在，在我们的 `AppDelegate` 中添加一个 `carPlayPlaylist` 属性：

```swift
// AppDelegate.swift

import UIKit
import MediaPlayer

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {

    var window: UIWindow?
    weak var stationsViewController: StationsViewController?
    var playableContentManager: MPPlayableContentManager?
    let carplayPlaylist = CarPlayPlaylist()
  
// ...
```

在 `datasource` 中我们需要创建一个 tab 来展示电台列表，为此我们需要将 `UIBrowsableContentSupportsSectionedBrowsing` `Boolean/YES` 值添加到 `info.plist` 中。

![](https://gitee.com/junteng/images/raw/master/img/20220107150450.png)

现在，让我们在 `datasource` 中添加所需的数据，对于 `numberOfChildItems`，tabs 的数量是 1，items 的数量是 `CarPlayPlayList stations` `count`：

```swift
// AppDelegate+CarPlay.swift

func numberOfChildItems(at indexPath: IndexPath) -> Int {
    if indexPath.indices.count == 0 {
        return 1
    }
    return carplayPlaylist.stations.count
}
```

在 `contentItem(at indexPath: IndexPath) -> MPContentItem?` 方法中，我们为每个 section（tab 和 list）创建一个 `MPContentItem` 类型的 item。为 tab 添加：

```swift
if indexPath.count == 1 {
    // Tab section
    let item = MPContentItem(identifier: "Stations")
    item.title = "Stations"
    item.isContainer = true
    item.isPlayable = false
    if let tabImage = UIImage(named: "carPlayTab") {
        item.artwork = MPMediaItemArtwork(boundsSize: tabImage.size, requestHandler: { _ -> UIImage in
            return tabImage
        })
    }
    return item
}
```

> 你可以从[这里](https://blog.fethica.com/assets/zips/carPlayTabIcon.zip)下载 carPlayTab 图标，并将它们添加到项目的 images .xcassets 中。

对于电台列表：

```swift
if indexPath.count == 1 {
    // Tab section
    // ...
} else if indexPath.count == 2, indexPath.item < carplayPlaylist.stations.count {
            
	  // Stations section
    let station = carplayPlaylist.stations[indexPath.item]
    let item = MPContentItem(identifier: "\(station.name)")
    item.title = station.name
    item.subtitle = station.desc
    item.isPlayable = true
    item.isStreamingContent = true
    
    // Get the station image from http or local
    if station.imageURL.contains("http") {
        ImageLoader.sharedLoader.imageForUrl(urlString: station.imageURL) { image, _ in
            DispatchQueue.main.async {
                guard let image = image else { return }
                item.artwork = MPMediaItemArtwork(boundsSize: image.size, requestHandler: { _ -> UIImage in
                    return image
                })
            }
        }
    } else {
        if let image = UIImage(named: station.imageURL) ?? UIImage(named: "stationImage") {
            item.artwork = MPMediaItemArtwork(boundsSize: image.size, requestHandler: { _ -> UIImage in
                return image
            })
        }
    }
    return item
} else {
    return nil
}
```

在 `beginLoadingChildItems` delegate 方法中，我们调用了 carPlayPlaylist 加载方法。

```swift
func beginLoadingChildItems(at indexPath: IndexPath, completionHandler: @escaping (Error?) -> Void) {
    carplayPlaylist.load { error in
        completionHandler(error)
    }
}
```

最后，不要忘记调用 AppDelegate 中的 `setupCarPlay` 方法。

```swift
// AppDelegate.swift

func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        
    // ...
        
    setupCarPlay()
        
    return true
}
```

让我们再次运行我们的 App，我们应该会在 CarPlay 显示屏中看到这个列表：

![](https://gitee.com/junteng/images/raw/master/img/20220107160820.png)

### 处理播放

让我们先创建一个新文件，并给 StationsViewController 类扩展一个用于播放电台的方法，命名为 `selectFromCarPlay:`：

```swift
// StationsViewController+CarPlay.swift

import UIKit

extension StationsViewController {
    func selectFromCarPlay(_ station: RadioStation) {
        radioPlayer.station = station
        handleRemoteStationChange()
    }
}
```

在 delegate 方法 `playableContentManager` 中，我们使用 indexPath 获取选中的 station，调用 stationViewController 的 `selectFromCarPlay:` 方法并将 station 传入：

```swift
// AppDelegate+CarPlay.swift

func playableContentManager(_ contentManager: MPPlayableContentManager, initiatePlaybackOfContentItemAt indexPath: IndexPath, completionHandler: @escaping (Error?) -> Void) {
        
    DispatchQueue.main.async {
        if indexPath.count == 2 {
            let station = self.carplayPlaylist.stations[indexPath[1]]
            self.stationsViewController?.selectFromCarPlay(station)
        }
        
        completionHandler(nil)
    }
}
```

如果我们再次运行我们的 App，我们将能够直接在 CarPlay 中播放所选电台。

> 如果你在选择电台时遇到 **out of range** 错误，只需重新启动 **CarPlay** simulator app，这将重新加载播放列表并解决问题。

### 在模拟器中显示 “Now Playing” 界面的解决方案

这是一个能够在模拟器上看到 `NowPlaying` 界面的解决方案（这在实际的设备上是不需要的），[source](https://stackoverflow.com/questions/52818170/handling-playback-events-in-carplay-with-mpnowplayinginfocenter)。

让我们更新 delegate 方法，如下所示:

```swift
func playableContentManager(_ contentManager: MPPlayableContentManager, initiatePlaybackOfContentItemAt indexPath: IndexPath, completionHandler: @escaping (Error?) -> Void) {
        
    DispatchQueue.main.async {
            
        if indexPath.count == 2 {
            let station = self.carplayPlaylist.stations[indexPath[1]]
            self.stationsViewController?.selectFromCarPlay(station)
        }
        completionHandler(nil)
        
        // Workaround to make the Now Playing working on the simulator:
        #if targetEnvironment(simulator)
            UIApplication.shared.endReceivingRemoteControlEvents()
        	  UIApplication.shared.beginReceivingRemoteControlEvents()
        #endif
    }
}
```

如果我们再次运行我们的 App，我们将在 **CarPlay** 上得到 **NowPlaying** 界面:

![](https://gitee.com/junteng/images/raw/master/img/20220107163558.png)

> 你会注意到播放按钮与播放状态不同步，它会在第一次启动时显示为暂停状态，因为我们已经使用此代码禁用了初始远程事件。

### 在实际设备上测试

为了在实际设备上运行该 App 或将其发布到 App Store，你需要使用 [此表单](https://developer.apple.com/contact/carplay/) 向 Apple 请求 **CarPlay** entitlement。

感谢 [@urayoanm](https://github.com/urayoanm) 在这个 Github [issue](https://github.com/analogcode/Swift-Radio-Pro/issues/104#issuecomment-433407949) 上分享他的的经验。

一旦你的请求获得批准，你将收到一封电子邮件，说明 **CarPlay** entitlement 已添加到你的 account，并且你将能够为你的 App 生成带有 **CarPlay** entitlement 的显式配置文件。

### 结尾

所有这些代码都已推送到 **SwiftRadio** repo 上的 **CarPlay** [branch](https://github.com/analogcode/Swift-Radio-Pro/tree/carplay)。

关于 **CarPlay** 的更多信息：

- [CarPlay Audio and Navigation Apps - WWDC 2018](https://developer.apple.com/videos/play/wwdc2018/213/)
- [Enabling Your App for CarPlay - WWDC 2017](https://developer.apple.com/videos/play/wwdc2017/719/)
- [CarPlay - Human Interface Guidelines: Audio Apps](https://developer.apple.com/design/human-interface-guidelines/carplay/overview/audio-apps/)