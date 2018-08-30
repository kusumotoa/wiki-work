## Coder Plugin
SDWebImage supports custom image coder plugin. Which can easily extern image format support when using SDWebImage's core feature.

You can check some public coder plugin here for image format which is not available from SDWebImage's core repo. See [Custom Coder](https://github.com/rs/SDWebImage/wiki/Advanced-Usage#custom-coder-420) for more detailed usage.


## For Coder Plugin Developers

If your custom coder add new image format support, which is not listed in SDWebImage built-in define of `SDImageFormat`, you can define it and place and update the format code here. To avoid the format code conflict with others. The custom minimum format code should be larger than **10**, to keep future reserved format used by SDWebImage core repo.

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

## List
| Format | Repo | Format Code | Decode | Encode | Animation |
| ------ | ---- | ----------- | ------ | ------ | -------- |
| APNG | [SDWebImageAPNGCoder](https://github.com/SDWebImage/SDWebImageAPNGCoder) | `SDImageFormatPNG` | Y | Y | Y |
| HEIF | [SDWebImageHEIFCoder](https://github.com/SDWebImage/SDWebImageHEIFCoder) | `SDImageFormatHEIF` | Y | Y | N |
| BPG | [SDWebImageBPGCoder](https://github.com/SDWebImage/SDWebImageBPGCoder) | `SDImageFormatBPG` = 11 | Y | N | Y |


