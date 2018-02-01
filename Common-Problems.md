### Using dynamic image size with UITableViewCell

UITableView determines the size of the image by the first image set for a cell. If your remote images
don't have the same size as your placeholder image, you may experience strange anamorphic scaling issue.
The following article gives a way to workaround this issue:

[http://www.wrichards.com/blog/2011/11/sdwebimage-fixed-width-cell-images/](http://www.wrichards.com/blog/2011/11/sdwebimage-fixed-width-cell-images/)


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
