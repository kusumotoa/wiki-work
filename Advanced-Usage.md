This section will introduce some advacend usages for SDWebImage. You can find the code snippet for basic usage and the minimum library version supports the feature. The code snippet is using Objective-C and Swift based on iOS. (Some feature's usage may be a little different crossing the platforms)

### Custom Download Operation (4.0)

Custom download operation is a way to custom some specify logic for advanced usage. Mostly you don't need to do this because we have many configurations to control the bahavior for download. Only if you find no way to achieve your goal, you can create a subclass of `NSOperation` which conforms to `SDWebImageDownloaderOperationInterface`. However, the best practice is to subclass `SDWebImageDownloaderOperation` directly and override some functions because some protocol method is not easy to implement.

* Objective-C

```objective-c
@interface MySDWebImageDownloaderOperation : SDWebImageDownloaderOperation
@end
```

* Swift

```swift
class MySDWebImageDownloaderOperation : SDWebImageDownloaderOperation {}
```

After you create a custom download operation class, you can now tell the downloader use that class instead of built-in one.

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

For more practical usage, check the demo project [SDWebImageProgressiveJPEGDemo](https://github.com/SDWebImage/SDWebImageProgressiveJPEGDemo), which integrate [Concorde](https://github.com/contentful-labs/Concorde) to support better progressive JPEG decoding.

There are the list of custom coders from the contributors.

+ APNG (Decoding & Encoding with Image/IO): [SDWebImageAPNGCoder](https://github.com/SDWebImage/SDWebImageAPNGCoder)
+ BPG (Decoding with [libbpg](https://github.com/mirrorer/libbpg)): [SDWebImageBPGCoder](https://github.com/SDWebImage/SDWebImageBPGCoder)
+ HEIF (Decoding & Encoding with [libheif](https://github.com/strukturag/libheif)): [SDWebImageHEIFCoder](https://github.com/SDWebImage/SDWebImageHEIFCoder)

#### GIF coder

SDWebImage does not enable full GIF decoding/encoding for all `UIImageView` by default in 4.x. Because it may reduce the performance for `FLAnimatedImageView`. However, you can enable this for all `UIImageView` instance by simply adding the built-in GIF coder.

For macOS, `NSImage` supports built-in GIF decoding. However, the frame duration is limit to 50ms and may not correctly rendering some GIF images. However, if you enable GIF coder, we will use a subclass of `NSBitmapImageRep` to fix the frame duration to match the standard way modern browsers rendering GIF images. See #2223 for more detail.

* Objective-C

```objective-c
[[SDWebImageCodersManager sharedInstance] addCoder:[SDWebImageGIFCoder sharedCoder]];
```

* Swift

```swift
SDWebImageCodersManager.sharedInstance().addCoder(SDWebImageGIFCoder.shared())
```


### Image Progress (4.3.0)

Image progress use a [NSProgress](https://developer.apple.com/documentation/foundation/progress) to allow user to observe the download progress. Through you can still use the `progressBlock`. But using `NSProgress` with [KVO](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/KeyValueObserving/KeyValueObserving.html) make your code more easy to maintain. The code snippet use [KVOController](https://github.com/facebook/KVOController) to simplify the usage for KVO.

* Objective-C

```objective-c
UIProgressView *progressView;
[self.KVOController observe:imageView.sd_imageProgress keyPath:NSStringFromSelector(@selector(fractionCompleted)) options:NSKeyValueObservingOptionNew block:^(id  _Nullable observer, id  _Nonnull object, NSDictionary<NSString *,id> * _Nonnull change) {
    float progress = [change[NSKeyValueChangeNewKey] floatValue];
    dispatch_main_async_safe(^{ // progress value updated in background queue
        [progressView setProgress:progress animated:YES]; // update UI
    });
}];
```

* Swift

```swift
let progressView = UIProgressView();
self.kvoController.observe(imageView.sd_imageProgress, keyPath: #keyPath(Progress.fractionCompleted), options: [.new]) { (observer, object, change) in
    if let progress = (change[NSKeyValueChangeKey.newKey.rawValue] as? NSNumber)?.floatValue {
        DispatchQueue.main.async { // progress value updated in background queue
            progressView.setProgress(progress, animated: true) // update UI
        }
    }
}
```

### Image Transition (4.3.0)

Image transition is used to provide custom transition animation after the image load finished. You can use the built-in convenience method to easily specify your desired animation.

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

And you can custom complicated animation using the `prepares` and `animations` property.

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
transition.prepares = { (view, image, imageData, cacheType, imageURL) in
    view.transform = .init(rotationAngle: CGFloat.pi)
}
transition.animations = { (view, image) in
    view.transform = .identity
}
imageView.sd_imageTransition = transition
```	

Image transition is only applied for image from network by default. If you want it been applied for image from cache as well, using `SDWebImageForceTransition` option.

### Animated Image (5.0)
SDWebImage 5.0 provide a full stack solution for animated image loading, rendering and decoding.

Since animated image appear many part of current applications. 

#### Loading
`UIImage/NSImage` is really familiar for Cocoa/Cocoa Touch developer. Which hold a reference for bitmap image storage and let it works for `UIImageView/NSImageView`. However, these class for animated image support not works so well for many aspect. You can check `-[SDImageCoderHelper animatedImageWithFrames:]` for detailed reason.

So we create a new class called `SDAnimatedImage`, which is subclass of `UIImage/NSImage`. This class will use our animated image coder and support animated image rendering for `SDAnimatedImageView`. By subclassing `UIImage/NSImage`, this allow it to be integrated with most framework API. Including our `SDImageCache`'s memory cache, as well as `SDWebImageManager`, `SDWebImageDownloader`'s completion block.

All the method not listed in the header files will just fallback to the super implementation. Which allow you to just set this `SDAnimatedImage` to normal `UIImageView/NSImageView` class and show a static poster image. At most of time, you don't need complicated check for this class. It just work.

This animated image works together with `SDAnimatedImageView` (see below), and we have a `SDAnimatedImageView+WebCache` category to use all the familiar view category methods.

#### Rendering

SDWebImage provide a animated image view called `SDAnimatedImageView` to do animated image rendering. It's a subclass of `UIImageView/NSImageView`, which means you can integrate it easier for most of framework API.

When the `SDAnimatedImageView` was trigged by `setImage:`, it will decided how to rendering the image base on image instance. If the image is a `SDAnimatedImage`, it will try to do animated image rendering, and start animation immediately. ]When the image is a `UIImage/NSImage`, it will call super method instead to behave like a normal `UIImageView/NSImageView` (Note `UIImage/NSImage` also can represent a animated one.)

The benefit from this custom image view is that you can control the rendering config base on your use case. We all know that animated image will consume much more memory because it needs to decode all image frames into memory. `UIImageView` with animated `UIImage` on iOS/tvOS will always store all frames into memory, which may consume huge memory forbig animated images like GIF.
 and cause OOM. However, `NSImageView` on macOS will always decode animated image just in time without cache, consume huge CPU usage for big animated images like GIF.

So as `SDAnimatedImageView`, by default will decode and cache the image frames with a buffer (where the buffer size is calculated based on memory usage), when the buffer size out of limit, the older image frame will be purged to free up memory. This allows you to keep a balance in both CPU & memory. However, you can also set up your own buffer size, check `maxBufferSize` property for more detailed information.

#### Decoding

To support animated image view rendering. All the image coder plugin should support to decode individual frame instead of just return a simple image using the `SDImageCoder` method `decodedImageWithData:options:`. So this is why we introduce a new protocol `SDAnimatedImageCoder`.

In `SDAnimatedImageCoder`, you need to provide all information used for animated image rendering. Such as frame image, frame duration, total frame count and total loop count. All the built-in animated coder (including `GIF`, `WebP`, `APNG`) support this protocol, which means you can use it to show these animated image format right in `SDAnimatedImageView`.

However, if you have your own desired image format, you can try to provide your own by implementing this protocol, add your coder plugin, then all things done.

#### Customization

For advanced user, since `SDAnimatedImage` also contains a protocol, which allow you to customize your own animated image class. You can specify `SDWebImageContextAnimatedImageClass` for your custom implementation class to load. For example, here is a plugin for YYImage, which you can directly load `YYImage` with `YYAnimatedImageView` using SDWebImage's loading & caching system, see [SDWebImageYYPlugin](https://github.com/SDWebImage/SDWebImageYYPlugin) for detailed information.

