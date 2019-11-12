#### Using `UIImageView+WebCache` category with `UITableView`

Just import the `UIImageView+WebCache.h` header, and call the `sd_setImageWithURL:placeholderImage:`
method from the `tableView:cellForRowAtIndexPath:` `UITableViewDataSource` method. Everything will be
handled for you, from async downloads to caching management.

```objective-c
#import <SDWebImage/UIImageView+WebCache.h>

...

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
    static NSString *MyIdentifier = @"MyIdentifier";

    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:MyIdentifier];
    if (cell == nil) {
        cell = [[[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault
                                       reuseIdentifier:MyIdentifier] autorelease];
    }

    // Here we use the new provided sd_setImageWithURL: method to load the web image
    [cell.imageView sd_setImageWithURL:[NSURL URLWithString:@"http://www.domain.com/path/to/image.jpg"]
                      placeholderImage:[UIImage imageNamed:@"placeholder.png"]];

    cell.textLabel.text = @"My Text";
    return cell;
}
```

```swift
import SDWebImage
...
func tableView(tableView: UITableView!, cellForRowAtIndexPath indexPath: NSIndexPath!) -> UITableViewCell! {
    static let myIdentifier = "MyIdentifier"
    let cell = tableView.dequeueReusableCellWithIdentifier(myIdentifier, forIndexPath: indexPath) as UITableViewCell

    cell.imageView.sd_setImageWithURL(imageUrl, placeholderImage:placeholderImage)
    return cell
```

### Using blocks

With blocks, you can be notified about the image download progress and whenever the image retrieval has completed with success or not:

```objective-c
// Here we use the new provided sd_setImageWithURL: method to load the web image
[cell.imageView sd_setImageWithURL:[NSURL URLWithString:@"http://www.domain.com/path/to/image.jpg"]
                  placeholderImage:[UIImage imageNamed:@"placeholder.png"]
                         completed:^(UIImage *image, NSError *error, SDImageCacheType cacheType, NSURL *imageURL) {
                                ... completion code here ...
                             }];
```

Note: neither your success nor failure block will be call if your image request is canceled before completion.

### Using SDWebImageManager

The `SDWebImageManager` is the class behind the `UIImageView(WebCache)` category. It ties the asynchronous downloader with the image cache store. You can use this class directly to benefit from web image downloading with caching in another context than a `UIView` (ie: with Cocoa).

Note: When the image is from memory cache, it will not contains any `NSData` by default. However, if you need image data, you can pass `SDWebImageQueryDataWhenInMemory` in options arg.

Here is a simple example of how to use `SDWebImageManager`:

```objective-c
SDWebImageManager *manager = [SDWebImageManager sharedManager];
[manager loadImageWithURL:imageURL
                  options:0
                 progress:^(NSInteger receivedSize, NSInteger expectedSize) {
                        // progression tracking code
                 }
                 completed:^(UIImage *image, NSError *error, SDImageCacheType cacheType, BOOL finished, NSURL *imageURL) {
                    if (image) {
                        // do something with image
                    }
                 }];
```

### Using SDWebImagePrefetcher

`SDWebImageManager` is used for image query in cache and network. However, sometimes, you don't want to consume the queried image right now, but want to do a prefetching in advance. Here we have `SDWebImagePrefetcher`.

`SDWebImagePrefetcher` can prefetch multiple url list with array. Each url list will bind to single token (which allows cancel on current url list). Don't worry about duplicated urls, the prefetcher is backed by `SDWebImageManager`, it will check cache firstly, only image not in cache will go through network download.

+ Objective-C

```objectivec
NSArray<NSURL> *imageURLs;
[SDWebImagePrefetcher.sharedImagePrefetcher] prefetchURLs:imageURLs progress:^(NSUInteger noOfFinishedUrls, NSUInteger noOfTotalUrls) {
    // prefetch progress changed
} completed:^(NSUInteger noOfFinishedUrls, NSUInteger noOfSkippedUrls) {
    // all provided urls prefetched
}];
```

+ Swift

```swift
let imageURLs: [URL]
SDWebImagePrefetcher.shared.prefetchURLs(urls, progress: { (noOfFinishedUrls, noOfTotalUrls) in
    // prefetch progress changed
}) { (noOfFinishedUrls, noOfSkippedUrls) in
    // all provided urls prefetched
}
```

`SDWebImagePrefetcher` has a shared instance for convenience, but remember it's light-weight. If you need extra configuration for image urls which share the same options and context, create a new object instead.

`SDWebImagePrefetcher` supports delegate methods to observe the fetch count changes as well. You can use that for detailed fetching status check.

Notes:
+ When you want to prefetch animated image format to used with `SDAnimatedImageView`, make sure you pass the `.customImageClass` context option, or we will still use the UIImage animation, which is less performant.
+ Before 5.0, the cancel method on prefetcher will cancel all previous requests. Which means each prefetcher can prefetch one url lists at the same time. But 5.0 change that to use the `SDWebImagePrefetchToken` instead. You can just current url list, without effect all other fetching url list.

### Using Asynchronous Image Downloader Independently

It's also possible to use the async image downloader independently:

```objective-c
SDWebImageDownloader *downloader = [SDWebImageDownloader sharedDownloader];
[downloader downloadImageWithURL:imageURL
                         options:0
                        progress:^(NSInteger receivedSize, NSInteger expectedSize) {
                            // progression tracking code
                        }
                       completed:^(UIImage *image, NSData *data, NSError *error, BOOL finished) {
                            if (image && finished) {
                                // do something with image
                            }
                        }];
```

### Using Asynchronous Image Caching Independently

It is also possible to use the async based image cache store independently. SDImageCache
maintains a memory cache and an optional disk cache. Disk cache write operations are performed
asynchronous so it doesn't add unnecessary latency to the UI.

The SDImageCache class provides a singleton instance for convenience but you can create your own
instance if you want to create separated cache namespace.

To lookup the cache, you use the `queryDiskCacheForKey:done:` method. If the method returns nil, it means the cache
doesn't currently own the image. You are thus responsible for generating and caching it. The cache
key is an application unique identifier for the image to cache. It is generally the absolute URL of
the image.

```objective-c
SDImageCache *imageCache = [[SDImageCache alloc] initWithNamespace:@"myNamespace"];
[imageCache queryDiskCacheForKey:myCacheKey done:^(UIImage *image) {
    // image is not nil if image was found
}];
```

By default SDImageCache will lookup the disk cache if an image can't be found in the memory cache.
You can prevent this from happening by calling the alternative method `imageFromMemoryCacheForKey:`.

To store an image into the cache, you use the storeImage:forKey:completion: method:

```objective-c
[[SDImageCache sharedImageCache] storeImage:myImage forKey:myCacheKey completion:^{
    // image stored
}];
```

By default, the image will be stored in memory cache as well as on disk cache (asynchronously). If
you want only the memory cache, use the alternative method storeImage:forKey:toDisk:completion: with a negative
third argument.

### Using Cache Key Filter

Sometime, you may not want to use the image URL as cache key because part of the URL is dynamic
(i.e.: for access control purpose). SDWebImageManager provides a way to set a cache key filter that
takes the NSURL as input, and output a cache key NSString.

The following example sets a filter in the application delegate that will remove any query-string from
the URL before to use it as a cache key:

+ Objective-C

```objectivec
SDWebImageCacheKeyFilter *cacheKeyFilter = [SDWebImageCacheKeyFilter cacheKeyFilterWithBlock:^NSString * _Nullable(NSURL * _Nonnull url) {
    NSURLComponents *urlComponents = [[NSURLComponents alloc] initWithURL:url resolvingAgainstBaseURL:NO];
    urlComponents.query = nil;
    return urlComponents.URL.absoluteString;
}];
SDWebImageManager.sharedManager.cacheKeyFilter = cacheKeyFilter;
```

+ Swift

```swift
let cacheKeyFilter = SDWebImageCacheKeyFilter { (url) -> String? in
    var urlComponents = URLComponents(url: url, resolvingAgainstBaseURL: false)
    urlComponents?.query = nil
    return urlComponents?.url?.absoluteString
}
SDWebImageManager.shared.cacheKeyFilter = cacheKeyFilter
```

Note: In 4.x version, the cache key filter is actual a block. However, in 5.x we wrap the block into a object `SDWebImageCacheKeyFilter`. This is because we support the cache key filter as context option. However, Swift is hard to use a `NSDictionary<NSString *, id>` (bridging as `[String: Any]`) which contains a Objective-C block object.

### Use Request Modifier (5.0)
Sometime, you want to specify some custom HTTP Header, or modify the download request just beyond the default configuration from SDWebImage. You can use the request modifier, to modify and return a new `NSHTTPRequest` object for the actual network request.

The following example sets a custom HTTP Header for specify URL on the shared downloader level. You can also use the request modifier as context option to make it work for individual image request level.

+ Objective-C

```objectivec
SDWebImageDownloaderRequestModifier *requestModifier = [SDWebImageDownloaderRequestModifier requestModifierWithBlock:^NSURLRequest * _Nullable(NSURLRequest * _Nonnull request) {
    if ([request.URL.host isEqualToString:@"foo"]) {
        NSMutableURLRequest *mutableRequest = [request mutableCopy];
        [mutableRequest setValue:@"foo=bar" forHTTPHeaderField:@"Cookie"];
        return [mutableRequest copy];
    }
    return request;
}];
SDWebImageDownloader.sharedDownloader.requestModifier = requestModifier;
```

+ Swift

```swift
let requestModifier = SDWebImageDownloaderRequestModifier { (request) -> URLRequest? in
    if (request.url?.host == "foo") {
        var mutableRequest = request
        mutableRequest.setValue("foo=bar", forHTTPHeaderField: "Cookie")
        return mutableRequest
    }
    return request
};
SDWebImageDownloader.shared.requestModifier = requestModifier
```

Note: For some special image format like WebP/APNG, image servers may prefer client to send HTTP Accept header. Check wiki here: [Coder Accept MIME Type](https://github.com/SDWebImage/SDWebImage/wiki/Advanced-Usage#coder-accept-mime-type). You can also use the request modifier feature for the same configuration.

Note: Though this feature is introduced in 5.x, in early 4.x version, you can use the Headers Filter to specify custom HTTP Headers. However, you can not custom any other properties inside `NSURLRequest` like `HTTPBody`, `cachePolicy`, etc.

### Use Response Modifier (5.3.0)
Like request modifier, SDWebImage provide the response modifier for HTTP response. You can modify the HTTP header,  status code if you want to mock the data, or provide custom process logic.

+ Objective-C

```objective-c
SDWebImageDownloaderResponseModifier *responseModifier = [SDWebImageDownloaderResponseModifier responseModifierWithBlock:^NSURLResponse * _Nullable(NSURLResponse * _Nonnull response) {
    if ([response.URL.host isEqualToString:@"foo"]) {
        NSMutableDictionary *mutableHeaderFields = [((NSHTTPURLResponse *)response).allHeaderFields mutableCopy];
        mutableHeaderFields[@"Foo"] = @"Bar";
        NSHTTPURLResponse *modifiedResponse = [[NSHTTPURLResponse alloc] initWithURL:response.URL statusCode:404 HTTPVersion:nil headerFields:[mutableHeaderFields copy]];
        return [modifiedResponse copy];
    }
    return response;
}];
SDWebImageDownloader.sharedDownloader.responseModifier = responseModifier;
```

+ Swift

```swift
let responseModifier = SDWebImageDownloaderResponseModifier { (response) -> URLResponse? in
    if (response.url?.host == "foo") {
        var mutableHeaderFields = (response as! HTTPURLResponse).allHeaderFields as? [String: String]
        mutableHeaderFields?["Foo"] = "Bar"
        let modifiedResponse = HTTPURLResponse(url: response.url!, statusCode: 404, httpVersion: nil, headerFields: mutableHeaderFields)
        return modifiedResponse
    }
    return response
};
SDWebImageDownloader.shared.responseModifier = responseModifier
```

### Use View Indicator (5.0)
SDWebImage provide an easy and extensible API for image loading indicator. Which will show or animate during image loading from network. All you need to do is to setup the indicator before your image loading start.

Objective-C:

```objective-c
imageView1.sd_imageIndicator = SDWebImageActivityIndicator.grayIndicator;
imageView2.sd_imageIndicator = SDWebImageProgressIndicator.defaultIndicator;
```

Swift:

```swift
imageView1.sd_imageIndicator = SDWebImageActivityIndicator.gray
imageView2.sd_imageIndicator = SDWebImageProgressIndicator.`default`
```

Then all thing done. Just call `sd_setImageWithURL:` to start load image, and you'll see the indicator showing during network request and stop automatically after loading finished.

We provide two built-in indicator type, the Activity Indicator and Progress Indicator, for both iOS/tvOS/macOS support. However, all indicator use a protocol `SDWebImageIndicator` to build, so it allows you to customize and provide your own loading view and start animate. Check `SDWebImageIndicator` protocol for all the methods you need to implement.

