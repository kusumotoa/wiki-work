### Since iOS 5.0, NSURLCache handles disk caching, what is the advantage of SDWebImage over plain NSURLRequest?

iOS NSURLCache does memory and disk caching (since iOS 5) of raw HTTP responses. Each time the cache is hit, you're app will have to transform the raw cached data into an UIImage. This involves extensive operations like data parsing (HTTP data are encoded), memory copy etc.

On the other side, SDWebImage caches the UIImage representation in memory and store the original compressed (but decoded) image file on disk. UIImage are stored as-is in memory using NSCache, so no copy is involved, and memory is freed as soon as your app or the system needs it.

Additionally, image decompression that normally happen in the main thread the first time your use UIImage in an UIImageView is forced in a background thread by SDWebImageDecoder.

Last but not least, SDWebImage will completely bypass the complex and often miss-configured HTTP cache control negotiation. This greatly accelerate cache lookup.

### Since [AFNetworking](https://github.com/AFNetworking/AFNetworking) added similar category on UIImageView, is SDWebImage still useful?

AFNetworking doesn't handle disk caching but relies on OS's implementation. See previous question to see why it's not an ideal solution. It doesn't handle background image decompression either.

If you're already using AFNetworking and just want an easy async image loading category but you don't care about performance and memory usage, AFNetworking UIImageView category may fits your needs.