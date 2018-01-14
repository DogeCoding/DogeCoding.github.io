title: 视频开发
tags: Swift
categories: iOS
date: 2017/7/23 23:47
---

下面會介紹視頻的一些基本知識，和在iOS上實現視頻播放和緩存的幾種方案。

> - **软解码和硬解码**
> GPU解码就是所谓的硬解码，CPU解码就是软解码。iOS提供的播放器类使用的是硬解码，所以视频播放对CPU不会有很大的压力，但是支持的播放格式比较单一，一般就是MP4、MOV、M4V这几个。
> - **HTTP Live Streaming**
> HTTP Live Streaming（缩写是 HLS）是一个由苹果公司提出的基于HTTP的流媒体网络传输协议。它的工作原理是把整个流分成一个个小的基于HTTP的文件来下载，每次只下载一些。当媒体流正在播放时，客户端可以选择从许多不同的备用源中以不同的速率下载同样的资源，允许流媒体会话适应不同的数据速率。支持的视频流编码为H.264。我们在视频网站上看到的M3U8后缀的播放链接就是使用HLS协议的视频。HLS优点，1、看完一段缓存一段，防止只看一段视频但是把整个视频文件都缓存下来的用户，减少服务器压力和节省流量。2、根据用户网速切换不同的码率，兼顾流程性和清晰度。

# 播放
实现视频播放的两个方案。
***

## 一、自己实现对数据编码解码
可以在一些开源播放器上进行二次开发，如Bilibili的[ijkplayer](https://github.com/Bilibili/ijkplayer)，或者直接对[FFmpeg](https://github.com/FFmpeg/FFmpeg)开发，优点在整个播放过程可控，为后续进行缓存、流量控制、码率切换等开发提供了基础，缺点是复杂，要求高，工程量大。
***

## 二、AVFoundation
Media Assets, Playback and Editing. 使用Apple自有框架。

### AVAsset
> AVAsset is an abstract, immutable class used to model timed audiovisual media such as videos and sounds. An asset may contain one or more tracks that are intended to be presented or processed together, each of a uniform media type, including but not limited to audio, video, text, closed captions, and subtitles.

Audiovisual media的资源类，通常通过`AVURLAsset`用URL来实例化，可以用[Atom Inspector](https://developer.apple.com/download/more/)(一个Apple提供的用来查看视频信息的工具)来观察一个视频的属性，再去`AVAsset`中对应其属性。

| AVAsset属性 | 视频文件属性 |
| --- | --- |
| duration: CMTime | duration, timescale时长和时间尺度 |
| preferredRate: Float | 默认速度 |
| preferredVolume: Float | 默认音量 |
| creationDate: AVMetadataItem? | 视频创建时间 |
| tracks: [AVAssetTrack] | 轨道 |
| trackGroups: [AVAssetTrackGroup] | 轨道组 |
| lyrics: String? | 当前语言环境合适的歌词 |
| metadata: [AVMetadataItem] | 元数据 |


### AVPlayer
> An AVPlayer is a controller object used to manage the playback and timing of a media asset. It provides the interface to control the player’s transport behavior such as its ability to play, pause, change the playback rate, and seek to various points in time within the media’s timeline. You can use an AVPlayer to play local and remote file-based media, such as QuickTime movies and MP3 audio files, as well as audiovisual media served using HTTP Live Streaming.

AVPlayer是一个控制对象用于管理媒体asset的播放，它提供了相关的接口控制播放器的行为，比如：播放、暂停、改变播放的速率、跳转到媒体时间轴上的某一个点（简单理解就是实现拖动功能显示对应的视频位置内容）。我们能够使用AVPlayer播放本地和远程的媒体文件（使用 HTTP Live Streaming），比如： QuickTime movies 和 MP3 audio files，所以AVPlayer可以满足音视频播放的基本需求。
![AVFoundation的层次](http://oo8snaf4x.bkt.clouddn.com/15009677046266.png?imageView2/0/q/100)

### AVPlayerItem
> AVPlayerItem models the timing and presentation state of an asset played by an AVPlayer object. It provides the interface to seek to various times in the media, determine its presentation size, identify its current time, and much more. 

`AVPlayerItem`是一个负责处理`AVAsset`的资源并通过`AVPlayer`来播放的载体，提供了`seek`、确定显示大小、ID、时间等的接口。

### AVPlayerLayer
> AVPlayerLayer is a subclass of CALayer to which an AVPlayer object can direct its visual output. It can be used as the backing layer for a UIView or NSView or can be manually added to the layer hierarchy to present your video content on screen.

负责`AVPlayer`的视频输出展示。

![依赖关系图](http://oo8snaf4x.bkt.clouddn.com/15012312808745.png?imageView2/0/q/100)


### 简单使用

``` //
class AVPlayerTestView: UIView {
    let view: UIView? = nil
    func initPlayerView() {
        guard let url = URL.init(string: "http://meipu1.video.meipai.com/5e81c08e-2850-4fbd-bfc4-4ded297f9f1c.mp") else { return }
        let asset = AVAsset.init(url: url)
        let item = AVPlayerItem.init(asset: asset)
        let player = AVPlayer.init(playerItem: item)
        let playerLayer = AVPlayerLayer.init(layer: player)
        view?.layer.addSublayer(playerLayer)
    }
}
```

设置好一个`AVPlayer`的依赖关系和输出图层后，`AVPlayerItem`会根据你的URL去请求数据，自己内部做缓冲然后播放。我们需要做的是用KVO监听`AVPlayerItem`内部几个关键属性的状态，然后做出我们的处理。

| AVPlayerItem属性 | 状态 |
| --- | --- |
| status: AVPlayerItemStatus |  |
| .unknown: AVPlayerItemStatus | 未知状态 |
| .readyToPlay: AVPlayerItemStatus | 准备好去播放 |
| .failed: AVPlayerItemStatus | 资源无法被播放 |
| loadedTimeRanges: [NSValue] | 加载了的资源的时间范围(一般用来更新缓冲UI) |
| playbackBufferEmpty: Bool | 没有缓冲数据 |
| playbackLikelyToKeepUp: Bool | 有足够的缓冲大致能播放无卡顿 |

> General State Observations: You can use Key-value observing (KVO) to observe state changes to many of the player’s dynamic properties, such as its currentItem or its playback rate. You should register and unregister for KVO change notifications on the main thread. This avoids the possibility of receiving a partial notification if a change is being made on another thread. AVFoundation invokes observeValue(forKeyPath:of:change:context:) on the main thread, even if the change operation is made on another thread.
>  ***
> 基本状态观察者：你能够使用KVO来观察player动态属性的状态改变，比如像： currentItem 或者它的播放速度。我们应该在主线程注册和去除KVO，这能够避免如果在其它线程发送改变而导致接收局部通知，当发生通知，AVFoundation将在主线程调用observeValue(forKeyPath:of:change:context:) 方法，即使是在其他线程发生。

KVO能够很好的观察生成的状态，但是并不能够观察播放时间的改变，所以AVPlayer提供了两个方法来观察时间的改变:

``` //
/*
    @param interval
    调用block的时间间隔
    @param queue
    推荐使用串行队列，放在主线程就行了，并行队列会产生不明确的结果
*/
func addPeriodicTimeObserver(forInterval interval: CMTime, queue: DispatchQueue?, using block: @escaping (CMTime) -> Void) -> Any {
    // 可以在里面去设置控制状态，刷新进度UI
}

func addBoundaryTimeObserver(forTimes times: [NSValue], queue: DispatchQueue?, using block: @escaping () -> Void) -> Any
```


> Tips:
> 
> - 创建多个`AVPlayerLayer`只有最近的layer才会显示视频帧
> - 可以创建多个`AVPlayerItem`来替换`AVPlayer`的当前item，`func replaceCurrentItem(with item: AVPlayerItem?)`
> - 监听后要注意控制监听的生命周期

# 缓存
Apple自有的框架是没有提供缓存功能的，`AVPlayer`也没有提供直接获取其下载数据的接口，所以想做缓存只能自己来完整的实现。下面有几个方案。

## 一、自己实现的播放器
这种情况大多是根据下载来的数据解码播放，下载的时候做下缓存就好了

## 二、自带播放器+LocalServer
在iOS本地开启`Local Server`服务，然后`MPMoviePlayerController`请求本地`Local Server`服务。本地`Local Server`服务再不停的去对应的视频地址获取视频流。本地Local Server请求的时候，就可以把视频流缓存在本地。[Demo来源:Code4App](http://code4app.com/ios/视频边下载边播放/5292c381cb7e8445678b5ac2#)

## 三、AVPlayer+AVMutableComposition+AVAssetExportSession
原理是直接给`AVPlayer`传URL，让其内部自己去处理数据下载，然后通过`AVMutableComposition`和`AVAssetExportSession`从`AVAsset`提取视频的数据进行缓存。

### AVMutableComposition
> AVMutableComposition is a mutable subclass of AVComposition you use when you want to create a new composition from existing assets. You can add and remove tracks, and you can add, remove, and scale time ranges.

作用是从现有的`AVAsset`中创建出一个新的`AVComposition`，使用者能够从别的asset中提取他们的音频轨道或视频轨道，并且把它们添加到新建的composition中。

### AVAssetExportSession
> An AVAssetExportSession object transcodes the contents of an AVAsset source object to create an output of the form described by a specified export preset.

作用是把`AVAsset`解码输出到本地文件中。

关键需要把原先的`AVAsset(AVURLAsset)`实现的数据提取出来后拼接成另一个`AVAsset(AVComposition)`的数据然后解码输出，由于通过网络url下载下来的视频没有保存视频的原始数据（苹果没有暴露接口给我们获取），下载后播放的avasset不能使用`AVAssetExportSession`输出到本地文件，要曲线地把下载下来的视频通过重构成另外一个`AVAsset`实例才能输出。

``` //
NSString *documentDirectory = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES)[0];
NSString *myPathDocument = [documentDirectory stringByAppendingPathComponent:[NSString stringWithFormat:@"%@.mp4",[_source.videoUrl MD5]]];


NSURL *fileUrl = [NSURL fileURLWithPath:myPathDocument];

if (asset != nil) {
AVMutableComposition *mixComposition = [[AVMutableComposition alloc]init];
AVMutableCompositionTrack *firstTrack = [mixComposition addMutableTrackWithMediaType:AVMediaTypeVideo preferredTrackID:kCMPersistentTrackID_Invalid];
[firstTrack insertTimeRange:CMTimeRangeMake(kCMTimeZero, asset.duration) ofTrack:[[asset tracksWithMediaType:AVMediaTypeVideo]objectAtIndex:0] atTime:kCMTimeZero error:nil];

AVMutableCompositionTrack *audioTrack = [mixComposition addMutableTrackWithMediaType:AVMediaTypeAudio preferredTrackID:kCMPersistentTrackID_Invalid];
[audioTrack insertTimeRange:CMTimeRangeMake(kCMTimeZero, asset.duration) ofTrack:[[asset tracksWithMediaType:AVMediaTypeAudio]objectAtIndex:0] atTime:kCMTimeZero error:nil];

AVAssetExportSession *exporter = [[AVAssetExportSession alloc]initWithAsset:mixComposition presetName:AVAssetExportPresetHighestQuality];
exporter.outputURL = fileUrl;
if (exporter.supportedFileTypes) {
exporter.outputFileType = [exporter.supportedFileTypes objectAtIndex:0] ;
exporter.shouldOptimizeForNetworkUse = YES;
[exporter exportAsynchronouslyWithCompletionHandler:^{

}];

}
}
```


## 四、AVPlayer+AVAssetResourceLoader
### AVAssetResourceLoadingRequest
> An AVAssetResourceLoader object mediates resource requests from an AVURLAsset object with a delegate object that you provide. When a request arrives, the resource loader asks your delegate if it is able to handle the request and reports the results back to the asset.
`AVAssetResourceLoader`协调来自`AVURLAsset`的资源请求，你需要实现它的`delegate`。当收到一个请求时，`ResourceLoader`询问你的`delegate`是否能处理并将结果返回给`asset`。

![AVPlayer和AVAssetResourceLoader的层次结构](http://oo8snaf4x.bkt.clouddn.com/15003581605281.jpg?imageView2/0/q/100)
`AVAssetResourceLoader`通过你提供的委托对象去调节`AVURLAsset`所需要的加载资源。而很重要的一点是，`AVAssetResourceLoader`仅在`AVURLAsset`不知道如何去加载这个URL资源时才会被调用，就是说你提供的委托对象在`AVURLAsset`不知道如何加载资源时才会得到调用。一般我们可以更改URL的scheme用来隐藏真实的URL。如：




> 参考
> [iOS开发系列--音频播放、录音、视频播放、拍照、视频录制](http://www.cnblogs.com/kenshincui/p/4186022.html#video)
> [AVplayer实现播放本地和网络视频（Swift3.0）](http://blog.csdn.net/longshihua/article/details/53909733)
> [iOS视频流开发（2）—视频播放](http://www.cnblogs.com/zy1987/p/5028624.html)
> [iOS音频播放 (九)：边播边缓存](http://msching.github.io/blog/2016/05/24/audio-in-ios-9/)
> [iOS音视频实现边下载边播放](http://sky-weihao.github.io/2015/10/06/Video-streaming-and-caching-in-iOS/)
> [AVFoundation(二)：核心AVAsset](http://www.jianshu.com/p/9805be76ee68)
> [AVFoundation编程指南2-用AVPlayer播放视频](http://1199game.com/2016/10/avfoundation-2/)
> [AV Foundation系列（五）媒体组合](http://www.jianshu.com/p/c6be05ffe418)


