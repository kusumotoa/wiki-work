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
    view.transform = .init(rotationAngle: .pi)
}
transition.animations = { (view, image) in
    view.transform = .identity
}
imageView.sd_imageTransition = transition
```	

Image transition is only applied for image from network by default. If you want it been applied for image from cache as well, using `SDWebImageForceTransition` option.

### Animated Image (5.0)
SDWebImage 5.0 provide a full stack solution for animated image loading, rendering and decoding.

Since animated image format like GIF, WebP, APNG appears everywhere in today's Internet and Apps. It's important to provide a better solution to keep both simple and powerful. Since `FLAnimatedImage` we used in 4.x, is tied to GIF format only, use a `NSObject` subclass image representation which is not integrated with our framework, and we can not do further development since it's a third-party lib. We decide to build our own solution for this task.

#### Loading
`UIImage/NSImage` is really familiar for Cocoa/Cocoa Touch developer. Which hold a reference for bitmap image storage and let it works for `UIImageView/NSImageView`. However, these class for animated image support not works so well for many aspects. You can check `-[SDImageCoderHelper animatedImageWithFrames:]` for detailed reason.

So we create a new class called `SDAnimatedImage`, which is subclass of `UIImage/NSImage`. This class will use our animated image coder and support animated image rendering for `SDAnimatedImageView`. By subclassing `UIImage/NSImage`, this allow it to be integrated with most framework APIs. Including our `SDImageCache`'s memory cache, as well as `SDWebImageManager`, `SDWebImageDownloader`'s completion block.

All the methods not listed in the header files will just fallback to the super implementation. Which allow you to just set this `SDAnimatedImage` to normal `UIImageView/NSImageView` class and show a static poster image. At most of the time, you don't need complicated check for this class. It just works.

This animated image works together with `SDAnimatedImageView` (see below), and we have a `SDAnimatedImageView+WebCache` category to use all the familiar view category methods.

* Objective-C:

```objective-c
SDAnimatedImageView *imageView = [SDAnimatedImageView new];
[imageView sd_setImageWithURL:url];
```

* Swift:

```swift
let imageView = SDAnimatedImageView()
imageView.sd_setImage(with: url)
```

#### Rendering

SDWebImage provide an animated image view called `SDAnimatedImageView` to do animated image rendering. It's a subclass of `UIImageView/NSImageView`, which means you can integrate it easier for most of framework APIs.

When the `SDAnimatedImageView` was trigged by `setImage:`, it will decide how to rendering the image base on image instance. If the image is a `SDAnimatedImage`, it will try to do animated image rendering, and start animation immediately. When the image is a `UIImage/NSImage`, it will call super method instead to behave like a normal `UIImageView/NSImageView` (Note `UIImage/NSImage` also can represent an animated one)

The benefit from this custom image view is that you can control the rendering behavior base on your use case. We all know that animated image will consume much more CPU & memory because it needs to decode all image frames into memory. `UIImageView` with animated `UIImage` on iOS/tvOS will always store all frames into memory, which may consume huge memory for big animated images (like 100 frames GIF).
 and cause OOM. However, `NSImageView` on macOS will always decode animated image just in time without cache, consume huge CPU usage for big animated images (like Animated WebP because of low decoding speed).

So now we have `SDAnimatedImageView`, by default it will decode and cache the image frames with a buffer (where the buffer size is calculated based on current memory status), when the buffer is out of limit size, the older image frame will be purged to free up memory. This allows you to keep a balance in both CPU & memory. However, you can also set up your own buffer size, check `maxBufferSize` property for more detailed information.

* Objective-C

```objective-c
SDAnimatedImageView *imageView = [SDAnimatedImageView new];
// this only support bundle file but not asset catalog
// you can also use other initializer like `imageWithData:scale:`
SDAnimatedImage *animatedImage = [SDAnimatedImage imageNamed:@"image.gif"];
// auto start animation
imageView.image = animatedImage;
// when you want to stop
[imageView stopAnimating];
```

* Swift

```swift
let imageView = SDAnimatedImageView()
// this only support bundle file but not asset catalog
// you can also use other initializer like `init?(data:scale:)`
let animatedImage = SDAnimatedImage(named: "image.gif")
// auto start animation
imageView.image = animatedImage
// when you want to stop
imageView.stopAnimating
```

Another feature, `SDAnimatedImageView` supports progressive animated image rendering. Which behave just looks like when Chrome showing a large GIF image. (The animation pause at last downloaded frame, and continue when new frames available). To enable this, at first you must specify `SDWebImageProgressiveLoad` in the options for view category method. This behavior can also be controlled by `shouldIncrementalLoad` property. You can disable it and just showing the first poster image during progressive image loading.

#### Decoding

To support animated image view rendering. All the image coder plugin should support to decode individual frame instead of just return a simple image using the `SDImageCoder` method `decodedImageWithData:options:`. So this is why we introduce a new protocol `SDAnimatedImageCoder`.

In `SDAnimatedImageCoder`, you need to provide all information used for animated image rendering. Such as frame image, frame duration, total frame count and total loop count. All the built-in animated coder (including `GIF`, `WebP`, `APNG`) support this protocol, which means you can use it to show these animated image format right in `SDAnimatedImageView`.

* Objective-C

```objective-c
id<SDAnimatedImageCoder> coder = SDImageGIFCoder.sharedCoder;
UIImage *firstFrame = [coder animatedImageFrameAtIndex:0];
```

* Swift

```swift
let coder = SDImageGIFCoder.shared
let firstFrame = coder.animatedImageFrame(at: 0)
```

However, if you have your own desired image format, you can try to provide your own by implementing this protocol, add your coder plugin, then all things done.

Note if you also want to support progressive animated image rendering. Your custom coder must conform `SDAnimatedImageCoder` and `SDProgressiveImageCoder` as well. The coder will receive `updateIncrementalData:finished:` call when new data is available. And you should update your decoder and let it produce the correct `animatedImageFrameCount` and other method list in `SDAnimatedImageCoder`, which can be decoded with current data.

#### Customization

For advanced user, since `SDAnimatedImage` also contains a protocol, which allow you to customize your own animated image class. You can specify `SDWebImageContextAnimatedImageClass` for your custom implementation class to load. For example, here is a plugin for YYImage, which you can directly load `YYImage` with `YYAnimatedImageView` using SDWebImage's loading & caching system, see [SDWebImageYYPlugin](https://github.com/SDWebImage/SDWebImageYYPlugin) for detailed information.

### Image Transformer (5.0)

Image transformer is the feature, that allows you to do some [Image Process](https://en.wikipedia.org/wiki/Digital_image_processing) on the image which is just loaded and decoded from the image data. For example, you can resize or crop the current loaded image, then produce the output image to be used for the final result and store to cache.

The different with this and [Image Coder](https://github.com/rs/SDWebImage/wiki/Advanced-Usage#custom-coder-420) is that image coder is focused on the decoding from compressed image format data (JPEG/PNG/WebP) to the image memory representation (typically `UIImage/NSImage`). But image transformer is focused on the **input** and **output** of image memory representation.

#### Usage

For basic image transformer. It should conforms to `SDImageTransformer` protocol. We provide many common built-in transformers for list below, which supports iOS/tvOS/macOS/watchOS all Apple platforms. You can also create your own with the custom protocol.

+ Rounded Corner (Clip a rounded corner): `SDImageRoundCornerTransformer`
+ Resize (Scale down or up): `SDImageResizingTransformer`
+ Crop (Crop a rectangle): `SDImageCroppingTransformer`
+ Flip (Flip the orientation): `SDImageFlippingTransformer`
+ Rotate (Rotate a angle): `SDImageRotationTransformer`
+ Tint Color (Overlay a color mask): `SDImageTintTransformer`
+ Blur Effect (Gaussian blur): `SDImageBlurTransformer`
+ Core Image Filter (CIFilter based, exclude watchOS): `SDImageFilterTransformer`

And also, you can chain multiple transformers together, to let the input image been processed from the first one to the last one. By using `SDImagePipelineTransformer` with a list of transformers.

+ Objective-C

```objectivec
id<SDImageTransformer> transformer1 = [SDImageFlippingTransformer transformerWithHorizontal:YES vertical:NO];
id<SDImageTransformer> transformer2 = [SDImageRotationTransformer transformerWithAngle:M_PI_4 fitSize:YES];
id<SDImageTransformer> pipelineTransformer = [SDImagePipelineTransformer transformerWithTransformers:@[transformer1, transformer2]];
```

+ Swift

```swift
let transformer1 = SDImageFlippingTransformer(horizontal: true, vertical: false)
let transformer2 = SDImageRotationTransformer(angle: .pi / 4, fitSize: true)
let pipelineTransformer = SDImagePipelineTransformer(transformers: [transformer1, transformer2])
```

To apply a transformer for image requests, provide the context arg with you transformer, or you can specify a default transformer for image manager level to process all the image requests from that manager.

+ Objective-C:

```objectivec
id<SDImageTransformer> transformer = [SDImageResizingTransformer transformerWithSize:CGSizeMake(300, 300) scaleMode:SDImageScaleModeFill];
UIImageView *imageView;
[imageView sd_setImageWithURL:url placeholderImage:nil options:0 context:@{SDWebImageContextImageTransformer: transformer}];
```

+ Swift: 

```swift
let transformer = SDImageResizingTransformer(size: CGSize(300, 300), scaleMode: .fill)
let imageView: UIImageView
imageView.sd_setImage(withURL: url, placeholderImage: nil, context: [.imageTransformer: transformer])
```


#### Image Extensions

Our built-in image transformers are only the wrapper which call the `UIImage/NSImage` extension for image transform process. You can also use that for common image transform usage. 

The list of the common image extension method is almost equivalent to the built-in image transformer above. See `UIImage+Transform.h` for more detail information.

+ Objective-C:

```objectivec
UIImage *image;
UIImage *croppedImage = [image sd_croppedImageWithRect:CGRectMake(0, 0, 100, 100)];
```

+ Swift:

```swift
let image: UIImage
let croppedImage = image.sd_croppedImage(with: CGRect(0, 0, 100, 100))
```


