## Coder Plugin
SDWebImage supports custom image coder plugin. Which can easily extern image format support when using SDWebImage's core feature.

Our core lib, only use the Apple System build-in codec. Which support the common image format, like JPEG/PNG/GIF/BMP/HEIF. But you can use any of your preferred image formats by using the correct image coder plugins.

You can check some public coder plugin here for image format which is not available from SDWebImage's core repo. See [Custom Coder](https://github.com/rs/SDWebImage/wiki/Advanced-Usage#custom-coder-420) for more detailed usage.

## SDWebImage organization maintained
| Format | Repo | Format Code | Decode | Encode | Animation | Vector |
| ------ | ---- | ----------- | ------ | ------ | -------- | ------- |
| [WebP](https://developers.google.com/speed/webp/) | [SDWebImageWebPCoder](https://github.com/SDWebImage/SDWebImageWebPCoder) | `SDImageFormatWebP` | Y (Progressive) | Y | Y | N |
| [APNG](https://en.wikipedia.org/wiki/APNG) | [SDWebImageAPNGCoder](https://github.com/SDWebImage/SDWebImageAPNGCoder) | `SDImageFormatPNG` | Y (Progressive) | Y | Y | N |
| [HEIF](http://nokiatech.github.io/heif/) | [SDWebImageHEIFCoder](https://github.com/SDWebImage/SDWebImageHEIFCoder) | `SDImageFormatHEIF` | Y | Y | N | N |
| [BPG](https://bellard.org/bpg/) | [SDWebImageBPGCoder](https://github.com/SDWebImage/SDWebImageBPGCoder) | `SDImageFormatBPG` = 11 | Y | Y (from v0.4) | Y | N |
| [SVG](https://en.wikipedia.org/wiki/Scalable_Vector_Graphics) | [SDWebImageSVGCoder](https://github.com/SDWebImage/SDWebImageSVGCoder) | `SDImageFormatSVG` = 12 | Y | N | N | Y |
| [PDF](https://en.wikipedia.org/wiki/PDF) | [SDWebImagePDFCoder](https://github.com/SDWebImage/SDWebImagePDFCoder) | `SDImageFormatPDF` = 13 | Y | N | N | Y |
| [FLIF](https://flif.info/) | [SDWebImageFLIFCoder](https://github.com/SDWebImage/SDWebImageFLIFCoder) | `SDImageFormatFLIF` = 14 | Y (Progressive) | Y | Y | N |

## Community contribution (Welcome !)
| Format | Repo | Format Code | Decode | Encode | Animation | Vector |
| ------ | ---- | ----------- | ------ | ------ | -------- | ------- |


## For Coder Plugin Developers

If your custom coder add new image format support, which is not listed in SDWebImage built-in define of `SDImageFormat`, you can define it and update the format code here. To avoid the format code conflict with others. The custom minimum format code should be larger than **10**, to keep future reserved format used by SDWebImage core repo.

+ Objective-C

```objectivec
static const SDImageFormat SDImageFormatBPG = 11;
```

+ Swift

```swift
extension SDImageFormat {
    public static let BPG: SDImageFormat = SDImageFormat(rawValue: 11)
}
```

