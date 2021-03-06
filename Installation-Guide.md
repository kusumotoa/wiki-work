There are four ways to use SDWebImage in your project:
- using CocoaPods
- using Carthage
- using Swift Package Manager
- manual install (xcframework, build-only or embed Xcode Project)

### Installation with CocoaPods

[CocoaPods](http://cocoapods.org/) is a dependency manager for Objective-C, which automates and simplifies the process of using 3rd-party libraries in your projects. See the [Get Started](http://cocoapods.org/#get_started) section for more details.

#### Podfile
```
platform :ios, '8.0'
pod 'SDWebImage', '~> 5.0'
```

##### Swift and static framework

Swift project previously have to use `use_frameworks!` to make all Pods into dynamic framework to let CocoaPods works.

However, start with `CocoaPods 1.5.0+` (with `Xcode 9+`), which supports to build both Objective-C && Swift code into static framework. You can use modular headers to use SDWebImage as static framework, without the need of `use_frameworks!`:

```
platform :ios, '8.0'
# Uncomment the next line when you want all Pods as static framework
# use_modular_headers!
pod 'SDWebImage', :modular_headers => true
```

See more on [CocoaPods 1.5.0 — Swift Static Libraries](http://blog.cocoapods.org/CocoaPods-1.5.0/)

If not, you still need to add `use_frameworks!` to use SDWebImage as dynamic framework:

```
platform :ios, '8.0'
use_frameworks!
pod 'SDWebImage'
```

#### Subspecs

There are 2 subspecs available now: `Core` and `MapKit` (this means you can install only some of the SDWebImage modules. By default, you get just `Core`, so if you need `MapKit`, you need to specify it). 

Podfile example:
```
pod 'SDWebImage/MapKit'
```


### Installation with Carthage (iOS 8+)

[Carthage](https://github.com/Carthage/Carthage) is a lightweight dependency manager for Swift and Objective-C. It leverages CocoaTouch modules and is less invasive than CocoaPods.

To install with carthage, follow the instruction on [Carthage](https://github.com/Carthage/Carthage)

#### Cartfile

```ogdl
github "SDWebImage/SDWebImage"
```

### Installation with Swift Package Manager (Xcode 11+)

[Swift Package Manager](https://swift.org/package-manager/) (SwiftPM) is a tool for managing the distribution of Swift code as well as C-family dependency. From Xcode 11, SwiftPM got natively integrated with Xcode.

SDWebImage support SwiftPM from version 5.1.0. To use SwiftPM, you should use Xcode 11 to open your project. Click `File` -> `Swift Packages` -> `Add Package Dependency`, enter [SDWebImage repo's URL](https://github.com/SDWebImage/SDWebImage.git). Or you can login Xcode with your GitHub account and just type `SDWebImage` to search.

After select the package, you can choose the dependency type (tagged version, branch or commit). Then Xcode will setup all the stuff for you.

If you're a framework author and use SDWebImage as a dependency, update your `Package.swift` file:

```swift
let package = Package(
    // 5.1.0 ..< 6.0.0
    dependencies: [
        .package(url: "https://github.com/SDWebImage/SDWebImage.git", from: "5.1.0")
    ],
    // ...
)
```

### Manual Installation Guide

### Build SDWebImage as XCFramework

SDWebImage supports to build SDWebImage as [xcframework on Xcode 11](https://developer.apple.com/videos/play/wwdc2019/416/) from version 5.1.0. This is the simplest way to build SDWebImage.

To build xcframework, you can clone the repo and open `SDWebImage.xcodeproj`, choose the `SDWebImage XCFramework` target. Then just click `Build` (Command + R). You'll start to build all individual platform's framework and produce a `xcframework` product on your `build` directory. After all the builds success, Finder will pop-up and select this file.

On Xcode 10, this step only build frameworks and placing them in `build` directory, without producing a `xcframework` product.

### Build SDWebImage as Framework or Static Library

For most user, to use SDWebImage, just need to build SDWebImage source code into a framework or static library.

For framework, you can choose to use Dynamic Framework (Link during runtime) or Static Framework (Link to the main executable file like Static Library).

It's strongly recommended to use Framework instead of Static Library. Framework can bundle resource and header files together, and have module map for clang module. Which make it easy to use.

And more importantly, Swift can only import the Objective-C code by using modular framework, or using the Bridging Header. (Bridging header contains its own disadvantage but this is beyond the topic).

#### Clone the repository:

```
git clone https://github.com/SDWebImage/SDWebImage.git
```

Then open the `SDWebImage.xcodeproj`.

#### Select the Build Scheme you need

- `SDWebImage` for dynamic framework. You can also change the `Mach-O Type` to `Static Library` to build static framework in `Build Settings`.
- `SDWebImage Static` for static library.
- `SDWebImageMapKit` for MapKit sub component only.

#### Select Build Platform you need

- `My Mac` for macOS platform.
- `Generic iOS Device` for iOS platform.
- `Generic tvOS Device` for tvOS platform.
- `Generic watchOS Device` for watchOS platform.
- Simulator Device for Simulator platform.

#### Prepare for archive

If you want to build framework for Real Device, don't try to click `Build` (Command + R). Because by default it will use the `DEBUG` configuration, which is not suitable for production. It's mostly used for Simulator.

Instead, you can use `Archive`. But before we click the button, you need some prepare in the `Build Settings`.

Change the `Skip Install` to `NO`. Or the archived product will not contains any framework.

You can do this by modify the xcconfig file `Module-Shared.xcconfig`. Or you can change it using Xcode GUI.

```
SKIP_INSTALL = NO
```

#### Build the Framework or Static Library

Now, you can click the `Archive` button (`Product -> Archive`). After the build success. Xcode will pop-up the Organizer window.

Click `Distribute Content`. Then ensure the `Built Products` is selected. Click `Next` and select a build folder your want to export. Click `Export`.

You can find a `SDWebImage.framework`, or `libSDWebImage.a` and the Headers Files inside the build folder.

![](https://user-images.githubusercontent.com/6919743/55800822-2bd83880-5b07-11e9-8d72-0d57a848aaf4.png)

##### Note for Universal (Fat) Framework

If you need to build Universal Framework (for Simulator and Real Device). You need some command line to combine the framework.

For example, if you already built two frameworks, `iOS/SDWebImage.framework` for iOS Real Device, `Simulator/SDWebImage.framework` for Simulator.

```
mkdir Universal/
cp -R iOS/SDWebImage.framework Universal/SDWebImage.framework
lipo -create Simulator/SDWebImage.framework/SDWebImage iOS/SDWebImage.framework/SDWebImage -output Universal/SDWebImage.framework/SDWebImage
```

For Static Library, just do the same thing.

```
mkdir Universal/
lipo -create Simulator/libSDWebImage.a iOS/libSDWebImage.a -output Universal/libSDWebImage.a
```

#### Link the Framework or Static Library to your project

Under your Project folder. You can create a `Vendor` folder to place the Framework or Static Library.

##### For Framework (Dynamic or Static)

For Framework (Dynamic or Static), the Headers are inside the framework. Just copy the `SDWebImage.framework` into the `Vendor` folder.

If your project is App project and using Dynamic Framework. You need to click `Embedded Binaries`. Select `Add Other...` and select the `SDWebImage.framework`. Xcode automatically add it into the `Linked Frameworks and Libraries` as well.

If not (Framework project or using Static Framework). Click
click `Linked Frameworks and Libraries`. Select `Add Other...` and select the `SDWebImage.framework`.

Then all things done if you use Framework.

![](https://user-images.githubusercontent.com/6919743/55804348-af495800-5b0e-11e9-828c-70711ea5fdca.png)

##### For Static Library

For Static Library, you need copy both the `libSDWebImage.a` as well as the Headers into the `Vendor` folder.

![](https://user-images.githubusercontent.com/6919743/55804133-4e218480-5b0e-11e9-86ac-f17aabf6e0c5.png)

Open your application Xcode Project, click `Linked Frameworks and Libraries`. Select `Add Other...` and select the `libSDWebImage.a`.

After link, you need to specify the Header Search Path for headers. Check Build Settings's `Header Search Path`, add the Header Search Path, where there must be a `SDWebImage` parent directory of `SDWebImage.h` this umbrella header file.

The example above can using the following path.

```
$(SRCROOT)/Vendor
```

Then all things done if you use Static Library.


#### Reference

[Technical Note TN2435 - Embedding Frameworks In An App](https://developer.apple.com/library/archive/technotes/tn2435/_index.html)

### Using SDWebImage as Sub Xcode Project

You can also embed SDWebImage as a Sub Xcode Project using in your Xcode Project/Workspace. This can be used for some specify environment which does not support external dependency manager.

#### Clone the repository as submodule

To embed the Sub Xcode Project, you can simply add SDWebImage entire project using Git Submodule. 

```
cd Vendor/
git submodule add https://github.com/SDWebImage/SDWebImage.git
```

Note: If your project don't using Git Submodule, just copy the entire repo of SDWebImage to that Vendor folder, and you can add to your own Version Control tools.

However, using Git Submodule can make it easy to upgrade framework version and reduce Git repo size.

#### Add `SDWebImage.xcodeproj` into your Workspace/Project

Just drag the `SDWebImage.xcodeproj` you cloned, into your Xcode Workspace/Project 's Project Navigator.

For Xcode Workspace, you can put it the same level of your App Project.

For Xcode Project, you can put it inside your App Project.

![](https://user-images.githubusercontent.com/6919743/55799669-802de900-5b04-11e9-84c0-08d4d9452549.png)

#### Link to your App/Framework Target

To use SDWebImage, you should link the `SDWebImage` target.

Go to your App/Framework target's `General` page. Then click `Lined Frameworks and Libraries`, and add the `SDWebImage.framework` or `libSDWebImage.a` (Depends on your use case).

Then all things done.

![](https://user-images.githubusercontent.com/6919743/55799628-68eefb80-5b04-11e9-8f0b-4b7818c5d1fd.png)

