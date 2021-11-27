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





