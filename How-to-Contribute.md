# SDWebImage Contributing Guide

Hi! I'm really excited that you are interested in contributing to SDWebImage. Before submitting your contribution, please make sure to take a moment and read through the following guidelines:

- [Issue Reporting Guidelines](#issue-reporting-guidelines)
- [Pull Request Guidelines](#pull-request-guidelines)
- [Development Setup](#development-setup)
- [Project Structure](#project-structure)
- [Adding Files](#adding-files)
- [Unit Test](#unit-test)
- [Coding Style](#coding-style)
- [API and semversion](#api-and-semversion)

## Issue Reporting Guidelines

- If you face issue, use GitHub's [New Issue](https://github.com/SDWebImage/SDWebImage/issues/new) to submit issues.
- There is an issue template built in. You should always fill them as necessary, to make it easy for us to reproduce issue and fix it.

## Pull Request Guidelines

- If you face issue, use GitHub's [New PR](https://github.com/SDWebImage/SDWebImage/compare) to submit pull requests.
- The `master` branch is used for releasing the next mainstream version of SDWebImage.
- There are also some other branches used for maintain the legacy version. Because we follows the [Semantic Versioning](https://semver.org/). Like [5.1.x](https://github.com/SDWebImage/SDWebImage/tree/5.1.x) and [4.x](https://github.com/SDWebImage/SDWebImage/tree/4.x)

- If adding a new feature:
  - Add accompanying test case (See more about Testing later)
  - Provide a convincing reason to add this feature. You can add this in your Pull Request description.
  - Ideally, you can open a feature request issue first before working on it. We will tag it with [feature request](https://github.com/SDWebImage/SDWebImage/labels/feature%20request). Good example: #2743 and #2808

- If fixing bug:
  - If you are resolving a special issue, fill the issue `#xxxx` you want to fix in the pull request template.
  - Provide a detailed description of the bug in the PR. A demo or screenshot is better.
  - Add appropriate test coverage if applicable.

## Development Setup

You need the following toolchain setup to development SDWebImage all features.

+ Xcode 11.0 above
+ macOS 10.14 above (10.15 is required to debug Mac Catalyst features)
+ CocoaPods 1.7 and above (or latest)
+ Carthage 0.32 and above (or latest)

After cloning the repo, run:

``` bash
$ pod install
```

This can make the example in `SDWebImage.xcworkspace` ready to run. Make it easy to debug and develop new features.

There are also one `SDWebImage.xcodeproj`. This contains the framework target to build, used in non-CocoaPods users. 

## Project Structure

- **`SDWebImage/Core`**: Contains SDWebImage's core feature's code, mostly you should only change. Any of these files should expose to public API
- **`SDWebImage/Private`**:
  Contains SDWebImage private API used internally in unit test. Any of these files should not expose to public API
- **`SDWebImage/MapKit`**:
  Contains SDWebImage MapKit integration code. For CocoaPods it produce the `SDWebImage/MapKit` subspec. For Carthage and SwiftPM, it produce `SDWebImageMapKit` standalone framework.
- **`WebImage`**:
  Contains the umbrella headers and Info.plist
- **`Examples`**:
  Contains the example code and Xcode Project. Now we have both iOS/tvOS/watchOS/macOS example.
- **`Tests`**:
  Contains the unit test code and Xcode Project. Now we have both iOS/macOS test target. 
- **`Configs`**:
  Contains the Example and Test build setting configuration file. We don't use Xcode GUI build settings and prefers xcconfig to maintain it, to reduce the complexity across massive targets.
- **`Docs`**:
  Contains the documentation for SDWebImage. Some of them can also be found at [SDWebImage Wiki](https://github.com/SDWebImage/SDWebImage/wiki)
  
## Adding Files
In most cases, if you just want to add a small patch with only modification. You don't need to read this at all.

But, if you decide to adding new source files for your feature. You should have a detailed look at this.

Since SDWebImage support 4 different install ways. This may be a challenge for most new contributor. Here are the steps to do:

1. Open `SDWebImage.xcodeproj` (not workspace)
2. Xcode -> New File... -> Cocoa Touch Class, add your class
3. Click the target `SDWebImage static` and `SDWebImage`. (If you add feature to MapKit integration, only click `SDWebImageMapKit`)
4. Save the file into folder `SDWebImage/Core`. (If you add private APIs, choose `SDWebImage/Private`)
5. Check the right side `Target Membership` for new source file. Change to `Public` (If you add private APIs, change to `Private`)
6. Click on the top `SDWebImage.xcodeproj`. Goto the `Build Phase` tab
7. Choose `SDWebImage static` target's `Copy Headers` build phase, add your new header into the file list
8. Open `SDWebImage.h`, add one new line `#import <SDWebImage/XXX.h>` with your class header name

<img src="https://user-images.githubusercontent.com/6919743/65586966-a282c880-dfb7-11e9-9ae0-364393bedef0.jpg" width=960 />

Then all things done for adding new source file. These source file is available in all 4 CocoaPods/Carthage/SwiftPM/Manual install ways.

Since our demo use CocoaPods's development Pod feature. After adding new source file, if you need to see it on our demo.  remember to re-run `pod install` again, and open `SDWebImage.xcworkspace`. For source file modification you don't need to re-run this.

## Unit Test
SDWebImage provide unit test to ensure function. After you change all your code. You should run unit test to see if the test passed.

1. Open `SDWebImage.xcworkspace`
2. choose `Test` (for iOS) or `Test Mac` (for macOS) target.
3. Click Test (Command + U)
4. You can using Xcode to run single test case
  
## Coding Style
SDWebImage is written in Objective-C. However, we provide both Objective-C and Swift support.

For Objective-C, we prefers to use this [Objective-C Style Guide](https://github.com/raywenderlich/objective-c-style-guide). You should use the modern Objective-C 2.0 syntax and ARC for new features.

At the same time, SDWebImage provide a better Swift API adaptation, which means you should use modern syntax to make clang generate a better Swift API. Including the followings:

+ Use property instead of `-(void)setXXX` , `-(id)XXX` to make the property available in Swift
+ Use class property with the correct name instead of `+(instanceType)sharedInstance` in singleton to make it more easy to use in Swift. The generated interface should be simple `open class var shared { get }`
+ Add all nullability annotation to avoid any `AnyObject!` implicitly unwrapped optionals(Except that `null_resettable`)
+ Add all Core Foundation Ownership using `CF_RETURNS_RETAINED` for 
 Get Rule and `CF_RETURNS_NOT_RETAINED` for Create Rule to avoid any `Unmanaged` CF value
+ Change all key for Dictionary with `NS_STRING_ENUM` to make it easy to use in Swift with dot syntax
+ Change all global value type which represent enum with `NS_TYPED_ENUM` to make it easy to use in Swift with dot syntax

## API and semversion
SDWebImage follows [Semantic Versioning](https://semver.org/). Your pull request should take care of API stability as well.

API-break changes may be delayed to next major release. And our maintainers can also review the pull request and help you to found the accidental changes which break the API. 

If possible, don't expose too much unnecessary public APIs for internal process logic of SDWebImage. Use Private API for this case. Which can make it easy to maintain and changes in the future development.

## Credits

Thank you to all the people who have already contributed to SDWebimage.

You can also help for this open source project by becoming sponsor. See [opencollective/SDWebImage](https://opencollective.com/sdwebimage)

[![Contributors](https://img.shields.io/github/contributors/SDWebImage/SDWebImage.svg)](https://github.com/SDWebImage/SDWebImage/graphs/contributors)


