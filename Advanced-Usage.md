##Adcaned Usage

This section will introduce some advacend usages for SDWebImage. You can find the code snippet for basic usage and the minimum library version supports the feature. The code snippet is using Objective-C and Swift based on iOS. (Some feature's usage may be a little different crossing the platforms)

### Custom Download Operation (4.0)

Custom download operation is a way to custom some specify logic for advanced usage. Mostly you do need to do this because we have many configurations to control the bahavior for download. Only if you find no way to achieve your goal, you can create a subclass of `NSOperation` which conforms to `SDWebImageDownloaderOperationInterface`. However, the best practice is to subclass `SDWebImageDownloaderOperation` directly and override some functions because some protocol method is not easy to implement.

* Objective-C

```objective-c
@interface MySDWebImageDownloaderOperation : SDWebImageDownloaderOperation
@end
```

* Swift

```swift
class MySDWebImageDownloaderOperation : SDWebImageDownloaderOperation {}
```

After you create a custom download class, you should now tell the downloader use that class instead of built-in one.

* Objective-C

```objective-c
[[SDWebImageDownloader sharedDownloader] setOperationClass:[MySDWebImageDownloaderOperation class]];
```

* Swift

```swift
SDWebImageDownloader.shared().setOperationClass(MySDWebImageDownloaderOperation.self)
```

### Custom Coder (4.2.0)

Custom coder is a way to provide the function to decode or encode for specify image format. Through we have some built-in coder for common image format like JPEG/PNG/GIF/WebP(Via subspec). You can add custom coder to support more image format without any hacking.

For basic custom coder, it must conform to the protocol `SDWebImageCoder`. For custom coder supports [progressive decoding](https://en.wikipedia.org/wiki/Incremental_encoding), it need to conform `SDWebImageProgressiveCoder`.

* Objective-C

```objective-c
@interface MySDWebImageCoder : NSObject <SDWebImageCoder>
@end
```

* Swift

```swift
class MySDWebImageCoder : NSObject, SDWebImageCoder {}
```

After you create a custom coder class, don't forget to register that into the coder manager.

* Objective-C

```objective-c
MySDWebImageCoder *coder = [MySDWebImageCoder new];
[[SDWebImageCodersManager sharedInstance] addCoder:coder];
```

* Swift

```swift
let coder = MySDWebImageCoder()
SDWebImageCodersManager.sharedInstance().addCoder(coder)
```

For more practical usage, check [SDWebImageBPGCoder](https://github.com/SDWebImage/SDWebImageBPGCoder), which supports the [BPG](https://bellard.org/bpg/) image format for SDWebImage.


### Image Progress (4.3.0)

Image progress use a [NSProgress](https://developer.apple.com/documentation/foundation/progress) to allow user to observe the download progress. Through you can still use the `progressBlock`. But using `NSProgress` with [KVO](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/KeyValueObserving/KeyValueObserving.html) make your code more easy to maintain. The code snippet use [KVOController](https://github.com/facebook/KVOController) to simplify the usage for KVO.

* Objective-C

```objective-c
UIProgressView *progressView;
[self.KVOController observe:imageView.sd_imageProgress keyPath:NSStringFromSelector(@selector(fractionCompleted)) options:NSKeyValueObservingOptionNew block:^(id  _Nullable observer, id  _Nonnull object, NSDictionary<NSString *,id> * _Nonnull change) {
    float progress = [change[NSKeyValueChangeNewKey] doubleValue];
    dispatch_main_async_safe(^{ // progress value updated in background queue
        [progressView setProgress:progress animated:YES]; // update UI
    });
}];
```

* Swift

```swift
let progressView = UIProgressView();
self.kvoController.observe(imageView, keyPath: #keyPath(UIView.sd_imageProgress), options: [.new]) { (observer, object, change) in
    if let progress = (change[NSKeyValueChangeKey.newKey.rawValue] as? NSNumber)?.floatValue {
        DispatchQueue.main.async {
            progressView.setProgress(progress, animated: true)
        }
    }
}
```

### Image Transition (4.3.0)

Image transition is used to provide custom transition animation after the image load finished. You can use the built-in convenience method to easily specify your desired animation or use the custom animation.

* Objective-C

```objective-c
imageView.sd_imageTransition = SDWebImageTransition.fadeTransition;
NSURL *url = [NSURL URLWithString:@"https://foo/bar.jpg"];
[imageView sd_setImageWithURL:url];
```

* Swift

```swift
imageView.sd_imageTransition = .fade
let url = URL(string: "https://foo/bar.jpg")
imageView.sd_setImage(with: url)
```

And you can custom complicated animation using the `animations` and `prepares` property.

* Objective-C

```objective-c
SDWebImageTransition *transition = SDWebImageTransition.fadeTransition;
transition.prepares = ^(__kindof UIView * _Nonnull view, UIImage * _Nullable image, NSData * _Nullable imageData, SDImageCacheType cacheType, NSURL * _Nullable imageURL) {
    view.transform = CGAffineTransformMakeRotation(M_PI);
};
transition.animations = ^(__kindof UIView * _Nonnull view, UIImage * _Nullable image) {
    view.transform = CGAffineTransformIdentity;
};
imageView.sd_imageTransition = transition;
```

* Swift:

```swift
let transition = SDWebImageTransition.fade
transition.prepares = { (view, image, data, cacheType, URL) in
    view.transform = .init(rotationAngle: CGFloat.pi)
}
transition.animations = { (view, image) in
    view.transform = .identity
}
imageView.sd_imageTransition = transition
```