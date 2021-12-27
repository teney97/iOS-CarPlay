## iOS CarPlay｜使用 MediaPlayer framework 开发

要让你的 CarPlay App 兼容 iOS 13 及更早版本，你必须看看 [兼容 iOS13 及更早 iOS 系统](https://developer.Apple.com/documentation/carplay/supporting_previous_versions_of_ios?language=objc)。对于音频类 App，主要需要做两件事：

1. 将 Key `com.apple.developer.playable-content` 添加到 Entitlements.plist 中并设置 Value 为 1；

```xml
<key>com.apple.developer.playable-content</key>
<true/>
<key>com.apple.developer.carplay-audio</key>
<true/>
```

2. 使用 MediaPlayer framework 开发。





如何在CarPlay for Audio App中添加标签？https://www.xknote.com/ask/60d536b9ec7f1.html