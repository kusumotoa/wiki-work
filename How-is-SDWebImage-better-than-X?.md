### Since iOS 5.0, NSURLCache handles disk caching, what is the advantage of SDWebImage over plain NSURLRequest?

iOS NSURLCache does memory and disk caching (since iOS 5) of raw HTTP responses. Each time the cache is hit, your app will have to transform the raw cached data into an UIImage. This involves extensive operations like data parsing (HTTP data are encoded), memory copy etc.

On the other side, SDWebImage caches the UIImage representation in memory and store the original compressed (but decoded) image file on disk. UIImage are stored as-is in memory using NSCache, so no copy is involved, and memory is freed as soon as your app or the system needs it.

Additionally, image decompression that normally happens in the main thread the first time you use UIImage in an UIImageView is forced in a background thread by SDWebImageDecoder.

Last but not least, SDWebImage will completely bypass the complex and often misconfigured HTTP cache control negotiation. This greatly accelerates cache lookup.

### Since [AFNetworking](https://github.com/AFNetworking/AFNetworking) provides similar functionality for UIImageView, is SDWebImage still useful?

Arguably not. AFNetworking takes advantage of Foundation URL Loading System caching using `NSURLCache`, as well as a configurable in-memory cache for `UIImageView` and `UIButton`, which uses `NSCache` by default. Caching behavior can be further specified in the caching policy of a corresponding `NSURLRequest`. Other SDWebImage features, like background decompression of image data is also provided by AFNetworking.

If you're already using AFNetworking and just want an easy async image loading category, the built-in UIKit extensions will probably fit your needs.