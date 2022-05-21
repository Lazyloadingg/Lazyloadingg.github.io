---
title: iOS15推送后台语音播报探索
date: 2021-12-19 14:38:57
tags: iOS
categories: 开发
---
meta <meta name="referrer" content="no-referrer" />


### 前言 
前一段时间公司项目有个推送内容语音播报的需求，当时让做技术调研，简单搜了下相关的文章和资料，调研一半的时候突然来了优先级更高的需求，搁置了，这两天空下来，所以继续看，并且有了可行的方案。

调研后发现这个需求用到**Notification Service Extension**，网上有一些文章讲这个需求的实现，但是绝大多数讲的方案现在已经不适用了或者是只提供一个大致的思路没有具体的实现，所以把我实现这个需求的过程记录分享出来

### Notification Service Extension
`Notification Service Extension`iOS10之后才能使用，如果要想使用`Notification Service Extension`对通知内容进行更改，需要在推送中增加`mutable-content`字段并将值设置为`true`，使用通知扩展后推送的处理流程如图所示
![image](https://upload-images.jianshu.io/upload_images/2334192-0078503af33c455b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
创建步骤如下图所示
![image](https://upload-images.jianshu.io/upload_images/2334192-f1a6486c00c4936f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image](https://upload-images.jianshu.io/upload_images/2334192-2438899df962bf8f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

完成后会在项目中生成一个target和对应的文件夹，我们的代码就要卸载`NotificationService.m`中

`NotificationService.m`文件内部有两个方法，我们可以在这个方法中对通知内容进行修改
```objectivec
// Call contentHandler with the modified notification content to deliver. If the handler is not called before the service's time expires then the unmodified notification will be delivered.
// You are expected to override this method to implement push notification modification.
- (void)didReceiveNotificationRequest:(UNNotificationRequest *)request withContentHandler:(void (^)(UNNotificationContent *contentToDeliver))contentHandler
```
在这个方法中做通知扩展终止前的兜底处理

```objectivec
// Will be called just before this extension is terminated by the system. You may choose whether to override this method.
- (void)serviceExtensionTimeWillExpire;
```

到此`Notification Service Extension`创建完成

### 播报探索
#### 系统语音合成
系统提供的有文字转语音播报的方法，我们在收到推送后可以传入文字直接播报出语音
```objectivec
- (void)didReceiveNotificationRequest:(UNNotificationRequest *)request withContentHandler:(void (^)(UNNotificationContent * _Nonnull))contentHandler {
    self.contentHandler = contentHandler;
    self.bestAttemptContent = [request.content mutableCopy];
    
    // Modify the notification content here...
    self.bestAttemptContent.title = [NSString stringWithFormat:@"%@ [modified]", self.bestAttemptContent.title];
    NSString *content = self.bestAttemptContent.userInfo[@"aps"][@"alert"][@"body"];
    AVSpeechUtterance *utterance = [AVSpeechUtterance speechUtteranceWithString:content];
    AVSpeechSynthesisVoice *voice = [AVSpeechSynthesisVoice voiceWithLanguage:@"zh-CN"];
    utterance.voice = voice;
    AVSpeechSynthesizer *synth = [[AVSpeechSynthesizer alloc] init];
    [synth speakUtterance:utterance];
    self.contentHandler(self.bestAttemptContent);
}
```
网上大部分文章也会这么写，但是现在实际应用中基本不会用这个方案原因有两个：
1. **iOS12.1**之后系统限制了在扩展里进行播报的能力，所以此方案只能用在**iOS12.1**之前
2. 语音生硬，并且多音字和英文字母在汉语语境下经常读错，比如字母`E`会读成`额`的音(这个我同一套代码在不同设备上读音不一致，没找到原因)，还是三方服务效果好点>_<!

#### 内置本地音频
后来就想，不能语音合成，那播放本地语音呢？
```objectivec
- (void)didReceiveNotificationRequest:(UNNotificationRequest *)request withContentHandler:(void (^)(UNNotificationContent * _Nonnull))contentHandler {
    self.contentHandler = contentHandler;
    self.bestAttemptContent = [request.content mutableCopy];
    
    // Modify the notification content here...
    self.bestAttemptContent.title = [NSString stringWithFormat:@"%@ [modified]", self.bestAttemptContent.title];
    NSString * voiceType = self.bestAttemptContent.userInfo[@"voiceType"];
    UNNotificationSound * sound = [UNNotificationSound soundNamed:[NSString stringWithFormat:@"%@.mp3",voiceType]];
    self.bestAttemptContent.sound = sound;
    self.contentHandler(self.bestAttemptContent);
}
```
扩展类中有个`bestAttemptContent`属性，他是`UNMutableNotificationContent`类型，我们在修改推送内容时也是对它进行修改，而播放本地语音就是修改它的`sound`属性，但是这个时候产品跳出来了，说对这样的实现不太满意，太死板，只能播放固定的音频不够灵活😂，没办法只能继续看

然后就想把播报内容拆开，本地内置几段语音，根据推送内容进行拼接，然后修改`sound`进行播报，我先随便找了两段比较短的音频内置进项目进行测试

```objectivec
-(void)audioMergeClick{
    //1.获取本地音频素材
    NSString *audioPath1 = [[NSBundle mainBundle]pathForResource:@"1" ofType:@"mp3"];
    NSString *audioPath2 = [[NSBundle mainBundle]pathForResource:@"2" ofType:@"mp3"];
    AVURLAsset *audioAsset1 = [AVURLAsset assetWithURL:[NSURL fileURLWithPath:audioPath1]];
    AVURLAsset *audioAsset2 = [AVURLAsset assetWithURL:[NSURL fileURLWithPath:audioPath2]];
    //2.创建两个音频轨道,并获取两个音频素材的轨道
    AVMutableComposition *composition = [AVMutableComposition composition];
    //音频轨道
    AVMutableCompositionTrack *audioTrack1 = [composition addMutableTrackWithMediaType:AVMediaTypeAudio preferredTrackID:0];
    AVMutableCompositionTrack *audioTrack2 = [composition addMutableTrackWithMediaType:AVMediaTypeAudio preferredTrackID:0];
    //获取音频素材轨道
    AVAssetTrack *audioAssetTrack1 = [[audioAsset1 tracksWithMediaType:AVMediaTypeAudio] firstObject];
    AVAssetTrack *audioAssetTrack2 = [[audioAsset2 tracksWithMediaType:AVMediaTypeAudio]firstObject];
    //3.将两段音频插入音轨文件,进行合并
    //音频合并- 插入音轨文件
    // `startTime`参数要设置为第一段音频的时长，即`audioAsset1.duration`, 表示将第二段音频插入到第一段音频的尾部。
    
    [audioTrack1 insertTimeRange:CMTimeRangeMake(kCMTimeZero, audioAsset1.duration) ofTrack:audioAssetTrack1 atTime:kCMTimeZero error:nil];
    [audioTrack2 insertTimeRange:CMTimeRangeMake(kCMTimeZero, audioAsset2.duration) ofTrack:audioAssetTrack2 atTime:audioAsset1.duration error:nil];
    //4. 导出合并后的音频文件
    //`presetName`要和之后的`session.outputFileType`相对应
    //音频文件目前只找到支持m4a 类型的
    AVAssetExportSession *session = [[AVAssetExportSession alloc]initWithAsset:composition presetName:AVAssetExportPresetAppleM4A];
    
    NSString *outPutFilePath = [[self.filePath stringByDeletingLastPathComponent] stringByAppendingPathComponent:@"test.m4a"];
    
    if ([[NSFileManager defaultManager] fileExistsAtPath:outPutFilePath]) {
        [[NSFileManager defaultManager] removeItemAtPath:outPutFilePath error:nil];
    }
    // 查看当前session支持的fileType类型
    NSLog(@"---%@",[session supportedFileTypes]);
    session.outputURL = [NSURL fileURLWithPath:self.filePath];
    session.outputFileType = AVFileTypeAppleM4A; //与上述的`present`相对应
    session.shouldOptimizeForNetworkUse = YES;   //优化网络
    [session exportAsynchronouslyWithCompletionHandler:^{
        if (session.status == AVAssetExportSessionStatusCompleted) {
            NSLog(@"合并成功----%@", outPutFilePath);
            UNNotificationSound * sound = [UNNotificationSound soundNamed:@"test.m4a"];
            self.bestAttemptContent.sound = sound;
            self.contentHandler(self.bestAttemptContent);
        } else {
            self.contentHandler(self.bestAttemptContent);
        }
    }];
}
```
然后，推送过来后播报的还是默认声音😳，于是开始找原因，一开始以为是文件格式的问题，于是把`m4a`转化成`mp3`(省略代码)，还是不行，最后找到了一片文章（感谢大佬），文章说`sound`读取本地音频不是所有路径都可以，是有优先级的

1. 主应用中的文件夹
2. AppGroups共享目录中的Library/Sounds文件夹
3. main bundle

根据这个说法我开始测试，首先是第一条，我打印出APP沙盒`Library`下所有目录
* Library/Sounds
![image](https://upload-images.jianshu.io/upload_images/2334192-c3802d151760f3cb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
看到里边的`Sounds`目录了吧，这是我创建的，之前并没有，然后修改合成文件的保存路径，重新走起！

额，依然播报的是系统默认声音，短时间内没找到原因，于是直接看第二优先级

* AppGroups

我们知道因为沙盒机制，iOS系统的App只能访问自己的文件夹，`AppGroups`就是苹果提供的同一开发者账号下多App资源共享的一种方案，最低支持`iOS8`，我们用的是企业签名，因为公司组织架构和权限的原因麻烦了一天才把一个简单`AppGroups`配置完成😭，这里就不贴具体的创建和配置过程了，网上相关的资料也很多了

最终将上边的音频文件导出路径修改为`AppGroups`下的`Library/Sounds`
```objectivec
    [session exportAsynchronouslyWithCompletionHandler:^{
        if (session.status == AVAssetExportSessionStatusCompleted) {
            NSLog(@"合并成功----%@", outPutFilePath);
            UNNotificationSound * sound = [UNNotificationSound soundNamed:@"test.m4a"];
            self.bestAttemptContent.sound = sound;
            self.contentHandler(self.bestAttemptContent);
        } else {
            self.contentHandler(self.bestAttemptContent);
        }
    }];
```
记得判断文件夹是否存在，我这里简单贴一下`AppGroups`的操作吧
```objectivec
    NSURL *groupURL = [[NSFileManager defaultManager] containerURLForSecurityApplicationGroupIdentifier:kGroupDefaultSuiteName];
        NSURL * sounds = [groupURL URLByAppendingPathComponent:@"/Library/Sounds/" isDirectory:YES];
        if (![[NSFileManager defaultManager] contentsOfDirectoryAtPath:sounds.path error:nil]) {
            [[NSFileManager defaultManager] createDirectoryAtPath:sounds.path withIntermediateDirectories:YES attributes:nil error:nil];
        }
```
最终，发起推送，播报成功，但这个方案产品虽然说可以但还是不太满意，于是，继续摸索

### 最终方案
那么总结下之前方案不行的原因：
* 直接语音转文字播报，系统限制iOS12.1后播报能力
* 固定音频，不够灵活产品不满意
* 拆分固定音频，拼接后播报（同上）

既然这样，能不能把上边几种方案的优点结合下？将文字转语音后的音频文件存到本地然后再去播报？这里得到了另一个大佬的指点（感谢大佬），尝试过后确认方案可行☺️

查看了`AVSpeechSynthesizer`文档后没找到转音频文件的相关方法（可能是我眼拙，找到的请告诉我）于是去看了三方的能力，其中百度和科大讯飞的离线合成都提供了获取音频文件的方法，但是最终我用的是科大讯飞的，因为百度的只提供两个设备码供测试，科大讯飞的十个（格局打开）

详细方法看[科大讯飞](https://www.xfyun.cn/doc/tts/offline_tts/iOS-SDK.html#_3、在线合成)文档，注册申请过程我这里就不赘述了，但是如果想用三方服务强烈建议先看文档！！！

##### 1. 初始化离线合成引擎
```objectivec
  //讯飞
    [IFlySetting setLogFile:LVL_ALL];
    [IFlySetting showLogcat:YES];
    NSString *initString = [[NSString alloc] initWithFormat:@"appid=%@", @"你的appid"];
    [IFlySpeechUtility createUtility:initString];
```

##### 2. 设置参数
```objectivec
  _iFlySpeechSynthesizer = [IFlySpeechSynthesizer sharedInstance];
    _iFlySpeechSynthesizer.delegate = self;
    [[IFlySpeechUtility getUtility] setParameter:@"tts" forKey:[IFlyResourceUtil ENGINE_START]];
    //设置本地引擎类型，普通版设置为TYPE_LOCAL，高品质版设置为TYPE_LOCAL_XTTS
    [_iFlySpeechSynthesizer setParameter:[IFlySpeechConstant TYPE_LOCAL] forKey:[IFlySpeechConstant ENGINE_TYPE]];
    //设置发音人为小燕
    [_iFlySpeechSynthesizer setParameter:@"xiaoyan" forKey:[IFlySpeechConstant VOICE_NAME]];
    //获取离线语音合成发音人资源文件路径。以发音人小燕为例，请确保资源文件的存在。
    NSString *resPath = [[NSBundle mainBundle] pathForResource:@"common" ofType:@"jet"];
    NSString *resPath1 = [[NSBundle mainBundle] pathForResource:@"xiaoyan" ofType:@"jet"];
    NSString *vcnResPath = [[NSString alloc] initWithFormat:@"%@;%@",resPath,resPath1];
    //设置离线语音合成发音人资源文件路径
    [_iFlySpeechSynthesizer setParameter:vcnResPath forKey:@"tts_res_path"];
    [_iFlySpeechSynthesizer synthesize:content toUri:[self pcmPath]];
```
其中`-(void)synthesize:(NSString *)text toUri:(NSString*)uri`方法就是离线合成后讲语音文件保存本地的方法，两个参数，第一个是要播报的文字内容，第二个是音频文件要存储的路径

##### 3. 获取本地音频
这边有个问题就是离线合成的语音文件是`pcm`格式的，不仅是讯飞，百度也是一样，`pcm`我们是不能直接给`sound`播放的，所以我们要做一个格式转换，转成`mp3`进行播放，贴一个pcm转mp3方法，需要用的`lame`三方库

```objectivec
/***
 * pcm 文件转mp3文件
 */
- (BOOL)convertPcm:(NSString *)pcmPath toMp3:(NSString *)mp3Path {
    @try {
        FILE *fpcm = fopen([pcmPath cStringUsingEncoding:NSASCIIStringEncoding], "rb");
        if (fpcm == NULL) {
            return false;
        }
//        fseek(fpcm, 1024*4, SEEK_CUR); //跳过源文件的信息头，不然在开头会有爆破音
        FILE *fmp3 = fopen([mp3Path cStringUsingEncoding:NSASCIIStringEncoding], "wb");

        int channelCount = 1;   // 声道数, 跟录音时配置一样的
        lame_t lame = lame_init();
        lame_set_in_samplerate(lame, 16000); //设置采样率, 需要跟录音时的采样率相同
        lame_set_num_channels(lame, channelCount); //声道，不设置默认为双声道
        lame_set_VBR(lame, vbr_default);
//        lame_set_mode(lame, 0);//设置最终mp3编码输出的声道模式，如果不设置则和输入声道数一样。参数是枚举，STEREO代表双声道，MONO代表单声道
        lame_set_quality(lame, 2);//设置压缩品质，quality=0..9. 0=best (very slow). 9=worst. 品质越好转码速度越慢
        lame_init_params(lame);

        const int PCM_SIZE = 8192;//
        const int MP3_SIZE = 8192; //
        short int pcm_buffer[PCM_SIZE*channelCount];
        unsigned char mp3_buffer[MP3_SIZE];

        int read;
        int write;
        do {
            read = fread(pcm_buffer, channelCount*sizeof(short int), PCM_SIZE, fpcm);
            if (read == 0) {
                write = lame_encode_flush(lame, mp3_buffer, MP3_SIZE);
            } else {
                if (channelCount == 1) {
                    write = lame_encode_buffer(lame, pcm_buffer, NULL, read, mp3_buffer, MP3_SIZE); // 单声道音频转码
                } else {
                    write = lame_encode_buffer_interleaved(lame, pcm_buffer, read, mp3_buffer, MP3_SIZE); // 多声道音频转码
                }
            }
            fwrite(mp3_buffer, write, 1, fmp3);
        } while (read != 0);
        lame_mp3_tags_fid(lame, fmp3);
        lame_close(lame);
        fclose(fmp3);
        fclose(fpcm);
    } @catch (NSException *exception) {
        NSLog(@"catch exception, %@", exception);
        return false;
    } @finally {
        return true;
    }
}
```

最后在合成完成方法中做转换和播报处理
```objectivec
- (void) onCompleted:(IFlySpeechError *) error {
    if (error.errorCode == 0) {
        NSURL *groupURL = [[NSFileManager defaultManager] containerURLForSecurityApplicationGroupIdentifier:kGroupDefaultSuiteName];
        NSURL * sounds = [groupURL URLByAppendingPathComponent:@"/Library/Sounds/" isDirectory:YES];
        if (![[NSFileManager defaultManager] contentsOfDirectoryAtPath:sounds.path error:nil]) {
            [[NSFileManager defaultManager] createDirectoryAtPath:sounds.path withIntermediateDirectories:YES attributes:nil error:nil];
        }
        NSURL *mp3Path = [groupURL URLByAppendingPathComponent:@"Library/Sounds/voice.mp3" isDirectory:NO];
        BOOL result = [self convertPcm:[self pcmPath] toMp3:mp3Path.path];
        if (result) {
            if (@available(iOS 12.1,*)) {
                UNNotificationSound * sound = [UNNotificationSound soundNamed:@"voice.mp3"];
                self.bestAttemptContent.sound = sound;
                self.contentHandler(self.bestAttemptContent);
            }else{
                _player = [[AVAudioPlayer alloc] initWithContentsOfURL:mp3Path error:nil];
                [_player play];
                self.contentHandler(self.bestAttemptContent);
            }
        }else{
            self.contentHandler(self.bestAttemptContent);
        }
    }else{
        self.contentHandler(self.bestAttemptContent);
    }
}
```

这里依然做了系统区分，因为实际测试后发现，iOS11的系统设置合成音频给`sound`后还是播放的默认声音，后来发现有人遇到类似的问题，iOS10-iOS12系统无法在推送扩展里读取到`AppGroups`中的音频文件，之前手边只有iOS15系统的测试机，没有发现这个问题，所以最后在依然做了区分处理低版本系统用`AVAudioPlayer`播放合成音频

### 实现总结
1. 创建`Notification Service Extension`以实现对推送消息做最后的修改
2. 添加`AppGroup`
3. 将要播报的文字内容用离线合成转成音频文件并存入`AppGroup`内的`/Library/Sounds`下
4. 修改`bestAttemptContent`的`sound`为存入本地的音频文件
 
虽然实现这个需求废了些时间，但是实现后回头看看，也就几步😂😂😂

### 注意点
* `Extension`是单独的进程，离线合成引擎要在`Extension`中启动
* `Extension`启动后只有约30s时间供你操作，超时会播放默认声音
* 推送内容要添加`mutable-content`字段并将值设置为`true`


### 参考文章
[iOS小技能：消息推送扩展的使用](https://juejin.cn/post/7026905121866924063)

[iOS13微信收款到账语音提醒开发总结](https://juejin.cn/post/6844904042515152903)


















