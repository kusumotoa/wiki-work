### Using dynamic image size with UITableViewCell

UITableView determines the size of the image by the first image set for a cell. If your remote images
don't have the same size as your placeholder image, you may experience strange anamorphic scaling issue.
The following article gives a way to workaround this issue:

[http://www.wrichards.com/blog/2011/11/sdwebimage-fixed-width-cell-images/](http://www.wrichards.com/blog/2011/11/sdwebimage-fixed-width-cell-images/)

### Handle cell reusing for view category

The View Category including `UIImage+WebCache`, `UIView+WebCache` or some other categories supports cell reusing from the scratch design. Actually, for most of use cases. All you need to do to work with cell reusing in `UITableView`, `UICollectionView` on iOS (`NSTableView`, `NSCollectionView`on macOS) is quite simple, just use:

* Objective-C:

```objective-c
[cell.imageView sd_setImageWithURL:url placeholderImage:[UIImage imageNamed:@"placeholder"]];
```

* Swift:

```swift
cell.imageView?.sd_setImage(with: url, placeholderImage: UIImage(named: “placeholder"))
```

However, there are some options to control the detailed behavior using for View Category such as `SDWebImageDelayPlaceholder`, `SDWebImageAvoidAutoSetImage`. You can try these options if the default behavior cause some issues with your use cases.

##### Detailed behavior when using view category

1. When you call `sd_setImageWithURL:`, whatever you provide a valid url or nil, we will firstly set the placeholder(Or nil, which means will clear the current rendered image) to the current imageView's image before any cache or network request. Then the placeholder argument is totally unused, unless you specify `SDWebImageDelayPlaceholder`.

2. If the image fetched in memory cache, the callback from manager is synchronously and is dispatched to the main queue if need(If you already call `sd_setImageWithURL:` in the main queue, there is no queue disaptch and just like a normal function call). So it will immediately set the result image to override the placeholder. In the set image process, we will call [setNeedsLayout](https://developer.apple.com/documentation/uikit/uiview/1622601-setneedslayout) to mark the view it's ready for rendering. UIKit draw all the views during the same runloop(No asynchronous or queue dispatch) depended on view's first state and last state but not the intermediate state. The state we set the placeholder(or nil) at the beginning will be totally ignored and not even rendering to cause a flashing. If you do not understand this behavior, see [Understanding UIKit Rendering](https://developer.apple.com/videos/play/wwdc2011/121/)

3. If the image fetched in disk cache, the callback is asynchronously and dispatched from the cache queue to the main queue. So the placeholder (or nil) will be rendered with a little short time(depend on the IO speed and decoding time. This may looks like a flashing). And then replaced by the final image. (Note if you really do not want any flashing, try use [Image Transition](https://github.com/rs/SDWebImage/wiki/Advanced-Usage#image-transition-430). Or using `SDWebImageQueryDiskSync`, which may decrease the frame rate)

4. If the image fetched from network, the callback is asynchronously and dispatched from the downloader session queue to the main queue. So the placeholder (or nil) will be rendered with a long time(depend on the network speed and decoding time). And then replcaed by the final image. This behavior is nearlly the same as disk cache except the duration for placeholder rendering.(network speed is far more slower than disk cache)

5. If you use `SDWebImageDelayPlaceholder`, we will use the placeholder as target image after the fetch finished if the image is nil.


### Handle image refresh

SDWebImage does very aggressive caching by default. It ignores all kind of caching control header returned by the HTTP server and cache the returned images with no time restriction. It implies your images URLs are static URLs pointing to images that never change. If the pointed image happen to change, some parts of the URL should change accordingly.

If you don't control the image server you're using, you may not be able to change the URL when its content is updated. This is the case for Facebook avatar URLs for instance. In such case, you may use the `SDWebImageRefreshCached` flag. This will slightly degrade the performance but will respect the HTTP caching control headers:

* Objective-C

``` objective-c
[imageView sd_setImageWithURL:[NSURL URLWithString:@"https://graph.facebook.com/olivier.poitrey/picture"]
             placeholderImage:[UIImage imageNamed:@"avatar-placeholder.png"]
                      options:SDWebImageRefreshCached];
```

* Swift

```swift
imageView.sd_setImage(with: URL(string: "https://graph.facebook.com/olivier.poitrey/picture"), placeholderImage: UIImage(named: "avatar-placeholder.png"), options: .refreshCached)
```

### Add a progress indicator

Add these before you call ```sd_setImageWithURL```

* Objective-C

``` objective-c
[imageView sd_setShowActivityIndicatorView:YES];
[imageView sd_setIndicatorStyle:UIActivityIndicatorViewStyleGray];
```

* Swift

``` swift
imageView.sd_setShowActivityIndicatorView(true)
imageView.sd_setIndicatorStyle(.Gray)
```

### Handle self capture in completion block

When you try to use `sd_setImageWithURL:completed` and write some code in the completion block which contains `self`. You sohuld take care that this may cause [Strong Reference Cycles for Closures
](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/AutomaticReferenceCounting.html#//apple_ref/doc/uid/TP40014097-CH20-ID56). This because the completion block you provided will be retained by `SDWebImageManager`'s running operations. So the `self` in the completionBlock will also be captured until the load progress finished (Cache fetched or network finished).

If you do not want to keep `self` instance alive during load progress, just mark it to weak. For Objective-C, use a weak reference to `self`:

```objective-c
__weak typeof(self) wself = self;
[self.imageView sd_setImageWithURL:url completed:^(UIImage * _Nullable image, NSError * _Nullable error, SDImageCacheType cacheType, NSURL * _Nullable imageURL) {
    wself.imageView.image = image;
}];
```

For Swift, use `weak self` in capture lists:

```swift
self.imageView.sd_setImage(with: url, completed: { [weak self] (image, error, cacheType, imageURL) in
    self?.imageView.image = image
})
```
