## MPRemoteCommandCenter

MPRemoteCommandCenter 是一个响应由外部附件和系统控件发送的远程控制事件的对象。使用 sharedCommandCenter 方法来获取它的单例来使用，而不是自己创建实例。它提供了一系列的 MPRemoteCommand 属性，用来响应各种远程控制事件，你可以根据自己需求来配置。

### 使用方式

#### 1. 启用命令

```objectivec
MPRemoteCommandCenter *commandCenter = [MPRemoteCommandCenter sharedCommandCenter];
// 播放, 暂停, 上下曲的命令默认都是启用状态, 即 enabled 默认为 YES
commandCenter.playCommand.enabled = enable;
commandCenter.pauseCommand.enabled = enable;
commandCenter.previousTrackCommand.enabled = enable;
commandCenter.nextTrackCommand.enabled = enable;
// 启用耳机的播放/暂停命令 (耳机上的播放按钮触发的命令)
commandCenter.togglePlayPauseCommand.enabled = enable;
// 播放模式
commandCenter.changeRepeatModeCommand.enabled = enable;
// 播放速率
commandCenter.changePlaybackRateCommand.enabled = enable;
// 拖动进度
if (@available(iOS 9.1, *)) {
    commandCenter.changePlaybackPositionCommand.enabled = enable;
}
...
```

#### 2. 监听远程控制事件

```objectivec
[commandCenter.playCommand addTarget:self action:@selector(play)];
[commandCenter.pauseCommand addTarget:self action:@selector(pause)];
if (@available(iOS 9.1, *)) {
    [commandCenter.changePlaybackPositionCommand addTarget:self action:@selector(changePlaybackPosition:)];
}

// 也可以使用 addTargetWithHandler: 方法
[commandCenter.playCommand addTargetWithHandler:^MPRemoteCommandHandlerStatus(MPRemoteCommandEvent * _Nonnull event) {
    if (self.player.rate == 0.0) {
        [self.player play];
        return MPRemoteCommandHandlerStatusSuccess;
    }
    return MPRemoteCommandHandlerStatusCommandFailed;
}];
```

#### 3. 响应事件

```objectivec
- (MPRemoteCommandHandlerStatus)play {
    // play audio
    return MPRemoteCommandHandlerStatusSuccess;
}

- (MPRemoteCommandHandlerStatus)pauseAction {
    // pause audio
    return MPRemoteCommandHandlerStatusSuccess;
}

- (MPRemoteCommandHandlerStatus)changePlaybackPositionCommand:(MPChangePlaybackPositionCommandEvent *)event {
    // change playback position
    // event.positionTime：获取拖动后的进度位置，单位秒
    return MPRemoteCommandHandlerStatusSuccess;
}
```

#### 4. 移除远程控制事件的监听

```objectivec
[commandCenter.playCommand removeTarget:self];
[commandCenter.pauseCommand removeTarget:self];
if (@available(iOS 9.1, *)) {
    [commandCenter.changePlaybackPositionCommand removeTarget:self];
}
```

### MPRemoteCommand

响应远程命令事件的对象。

如果你明确地不想启用指定命令，则获取命令对象并将其 enable 属性设置为 NO。禁用远程命令可以让系统知道，当你的应用程序是“正在播放”应用程序时，它不应该为该命令显示任何相关的 UI。
MediaPlayer framework 定义了许多 MPRemoteCommand 子类来处理特定类型的命令。有时，这些子类允许您指定与命令相关的其他信息。例如，反馈命令 MPFeedbackCommand 允许你指定一个本地化的字符串来描述反馈的含义。在支持特定命令时，请确保查找用于处理这些事件的特定类。

```objectivec
typedef NS_ENUM(NSInteger, MPRemoteCommandHandlerStatus) {
    /// 执行命令成功
    MPRemoteCommandHandlerStatusSuccess = 0,
    /// 无法执行该命令，因为请求的内容在当前应用程序状态下不存在
    MPRemoteCommandHandlerStatusNoSuchContent = 100,
    /// 无法执行该命令，因为目前没有此命令所需的可用播放项。例如，如果应用程序收到“启用语言选项”命令，但当前没有播放任何内容，则会返回此错误代码
    MPRemoteCommandHandlerStatusNoActionableNowPlayingItem MP_API(ios(9.1), macos(10.12.2)) = 110,
    /// 无法执行该命令，因为所需的设备不可用。例如，如果需要戴耳机，或者一个手表应用程序意识到它需要同伴来完成请求
    MPRemoteCommandHandlerStatusDeviceNotFound MP_API(ios(11.0), macos(10.13)) = 120,
    /// 由于其他原因，该命令无法执行
    MPRemoteCommandHandlerStatusCommandFailed = 200
};

@interface MPRemoteCommand : NSObject

@property (nonatomic, assign, getter = isEnabled) BOOL enabled;

// 通过 Target-action style 添加事件响应，action 接收一个 MPRemoteCommandEvent 实例作为第一个参数。addTarget:action: 不会保留 target，让 target 被释放时应该调用 removeTarget: 移除

// selector 应该返回一个 MPRemoteCommandHandlerStatus 值，使得系统能够适当地响应那些根据应用程序的当前状态可能无法执行的命令。
- (void)addTarget:(id)target action:(SEL)action;
- (void)removeTarget:(id)target action:(nullable SEL)action;
- (void)removeTarget:(nullable id)target;

/// Returns an opaque object to act as the target.
- (id)addTargetWithHandler:(MPRemoteCommandHandlerStatus(^)(MPRemoteCommandEvent *event))handler;

@end
```

使用示例：

```objectivec
// Get the shared command center.
MPRemoteCommandCenter *commandCenter = [MPRemoteCommandCenter sharedCommandCenter];

// Add a handler for the play command.
[commandCenter.playCommand addTargetWithHandler:^MPRemoteCommandHandlerStatus(MPRemoteCommandEvent * _Nonnull event) {
    if (self.player.rate == 0.0) {
        [self.player play];
        return MPRemoteCommandHandlerStatusSuccess;
    }
    return MPRemoteCommandHandlerStatusCommandFailed;
}];
```

### 相关命令

```objectivec
// Playback Commands
@property (nonatomic, readonly) MPRemoteCommand *pauseCommand; // 暂停播放
@property (nonatomic, readonly) MPRemoteCommand *playCommand;  // 开始播放
@property (nonatomic, readonly) MPRemoteCommand *stopCommand;  // 停止播放
@property (nonatomic, readonly) MPRemoteCommand *togglePlayPauseCommand; // 用于在播放和暂停当前项目之间切换的命令对象，用于耳机等
@property (nonatomic, readonly) MPRemoteCommand *enableLanguageOptionCommand MP_API(ios(9.0), macos(10.12.2)); // 启用语言选项
@property (nonatomic, readonly) MPRemoteCommand *disableLanguageOptionCommand MP_API(ios(9.0), macos(10.12.2)); // 禁用语言选项
@property (nonatomic, readonly) MPChangePlaybackRateCommand *changePlaybackRateCommand; // 播放速度
@property (nonatomic, readonly) MPChangeRepeatModeCommand *changeRepeatModeCommand;     // 播放重复模式（单曲循环/顺序循环）
@property (nonatomic, readonly) MPChangeShuffleModeCommand *changeShuffleModeCommand;   // 随机播放模式

// Previous/Next Track Commands
@property (nonatomic, readonly) MPRemoteCommand *nextTrackCommand; 		 // 下一曲
@property (nonatomic, readonly) MPRemoteCommand *previousTrackCommand; // 上一曲

// Skip Interval Commands
@property (nonatomic, readonly) MPSkipIntervalCommand *skipForwardCommand;  // 快进
@property (nonatomic, readonly) MPSkipIntervalCommand *skipBackwardCommand; // 倒退

// Seek Commands
@property (nonatomic, readonly) MPRemoteCommand *seekForwardCommand;
@property (nonatomic, readonly) MPRemoteCommand *seekBackwardCommand;
@property (nonatomic, readonly) MPChangePlaybackPositionCommand *changePlaybackPositionCommand MP_API(ios(9.1), macos(10.12.2)); // 拖动进度

// Rating Command
@property (nonatomic, readonly) MPRatingCommand *ratingCommand; // 评分

// Feedback Commands
// These are generalized to three distinct actions. Your application can provide
// additional context about these actions with the localizedTitle property in
// MPFeedbackCommand.
@property (nonatomic, readonly) MPFeedbackCommand *likeCommand;     // 喜欢
@property (nonatomic, readonly) MPFeedbackCommand *dislikeCommand;  // 不喜欢
@property (nonatomic, readonly) MPFeedbackCommand *bookmarkCommand; // 添加书签
```

### 参考资料

* [Apple Developer Documentation｜MPRemoteCommandCenter](https://developer.apple.com/documentation/mediaplayer/mpremotecommandcenter?language=objc)