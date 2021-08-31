## WWDC19 - CarPlay 车载系统改进

**地址**：https://developer.apple.com/wwdc19/252

**时长**：16 分钟

**概览**：CarPlay 车载让你能够在车内更加智能、安全地使用 iPhone。了解如何更新你的车载系统，以利用 iOS 13 中的新功能。添加对动态改变屏幕尺寸、仪表盘等辅助屏幕（第二屏幕）、甚至对不规则显示屏的支持。了解如何支持 “嘿 Siri” 来进行免提语音激活。

**总结**：该 Session 主要面向汽车制造商。

* 支持不规则显示屏
* 支持在仪表盘等辅助屏幕上显示一个额外的 CarPlay 用户界面
* 支持动态改变 CarPlay 用户界面的大小
* 支持 “嘿 Siri”
  * 只需用声音即可调用 Siri
  * 只需按一下按钮，Siri 就会立即启动
  * 在与 Siri 互动的同时继续听音乐

**Vehicle System Requirements**

* Always on microphone input stream processed within the vehicle
* Continuous echo cancellation and noise reduction
* Voice activity detector
* Keyword detector
* Audio mixer for Siri audio, media playback, and navigation prompts