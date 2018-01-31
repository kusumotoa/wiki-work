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
