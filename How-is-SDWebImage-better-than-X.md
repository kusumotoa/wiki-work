## How-is-SDWebImage-better-than-X

### Since iOS 5.0, NSURLCache handles disk caching, what is the advantage of SDWebImage over plain NSURLRequest?

iOS NSURLCache does memory and disk caching (since iOS 5) of raw HTTP responses. Each time the cache is hit, your app will have to transform the raw cached data into an UIImage. This involves extensive operations like data parsing (HTTP data are encoded), memory copy etc.

On the other side, SDWebImage caches the UIImage representation in memory and store the original compressed (but decoded) image file on disk. UIImage are stored as-is in memory using NSCache, so no copy is involved, and memory is freed as soon as your app or the system needs it.

Additionally, image decompression that normally happens in the main thread the first time you use UIImage in an UIImageView is forced in a background thread by SDWebImageDecoder.

Last but not least, SDWebImage will completely bypass the complex and often misconfigured HTTP cache control negotiation. This greatly accelerates cache lookup.

### Since [AFNetworking](https://github.com/AFNetworking/AFNetworking) provides similar functionality for UIImageView, is SDWebImage still useful?

Arguably not. AFNetworking takes advantage of Foundation URL Loading System caching using `NSURLCache`, as well as a configurable in-memory cache for `UIImageView` and `UIButton`, which uses `NSCache` by default. Caching behavior can be further specified in the caching policy of a corresponding `NSURLRequest`. Other SDWebImage features, like background decompression of image data is also provided by AFNetworking.

If you're already using AFNetworking and just want an easy async image loading category, the built-in UIKit extensions will probably fit your needs.

## Competitors

Library | Language | Stargazers | Comments
------- | -------- | ---------- | --------
[**SDWebImage**](https://github.com/rs/SDWebImage) | ObjC | [14.000+](http://ghbtns.com/github-btn.html?user=rs&repo=SDWebImage&type=watch&count=true)
 | | |
[AlamofireImage](https://github.com/Alamofire/AlamofireImage) | Swift | [1500+](http://ghbtns.com/github-btn.html?user=Alamofire&repo=AlamofireImage&type=watch&count=true)
[AFNetworking](https://github.com/AFNetworking/AFNetworking) | ObjC | [25.000+](http://ghbtns.com/github-btn.html?user=AFNetworking&repo=AFNetworking&type=watch&count=true)
[Nuke](https://github.com/kean/Nuke) | Swift | [1500+](http://ghbtns.com/github-btn.html?user=kean&repo=Nuke&type=watch&count=true)
[DFImageManager](https://github.com/kean/DFImageManager) | ObjC | [1100+](http://ghbtns.com/github-btn.html?user=kean&repo=DFImageManager&type=watch&count=true) | discontinued in favour of Nuke
[KingFisher](https://github.com/onevcat/Kingfisher) | Swift | [4700+](http://ghbtns.com/github-btn.html?user=onevcat&repo=Kingfisher&type=watch&count=true)
[HanekeSwift](https://github.com/Haneke/HanekeSwift) | Swift | [3500+](http://ghbtns.com/github-btn.html?user=Haneke&repo=HanekeSwift&type=watch&count=true)
[Haneke](https://github.com/Haneke/Haneke) | ObjC | [1600+](http://ghbtns.com/github-btn.html?user=Haneke&repo=Haneke&type=watch&count=true)
[TMCache](https://github.com/tumblr/TMCache) | ObjC | [2800+](http://ghbtns.com/github-btn.html?user=tumblr&repo=TMCache&type=watch&count=true) | discontinued
[PINCache](https://github.com/pinterest/PINCache) | ObjC | [1000+](http://ghbtns.com/github-btn.html?user=pinterest&repo=PINCache&type=watch&count=true)
[EGOCache](https://github.com/enormego/EGOCache) | ObjC | [1100+](http://ghbtns.com/github-btn.html?user=enormego&repo=EGOCache&type=watch&count=true)