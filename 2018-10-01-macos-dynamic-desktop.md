---
title: "macOS 动态桌面"
author: Mattt
translator: saitjr
category: ""
excerpt: >
  Dark Mode 可谓是 macOS 最受欢迎的特性之一了 —— 尤其是对于你我这样的开发者来说。过去几年，和这个特性旗鼓相当的要数 Night Shift，而在 Mojave 当中，推出了全新的 Dynamic Desktop。
status:
  swift: 4.2
---

Dark Mode is one of the most popular additions to macOS ---
especially among us developer types,
who tend towards light-on-dark color themes in text editors
and appreciate this new visual consistency across the system.

Dark Mode（深色模式）可谓是 macOS 最受欢迎的特性之一了 —— 尤其是对于你我这样的开发者来说。我们不仅喜欢文本编辑器是暗色的主题，还很看中整个系统色调的一致性。

A couple of years back, there was similar fanfare for Night Shift,
which helped to reduce eye strain
from hacking late into the night (or early in the morning, as it were).

过去几年，和这个特性旗鼓相当的要数 Night Shift（夜览），它主要是在日夜更替的时候减少对眼睛的劳损。

If you triangulate from those two macOS features,
you get Dynamic Desktops, also new in Mojave.
Now when you go to "System Preferences > Desktop & Screen Saver",
you have the option to select a "Dynamic" desktop picture
that changes throughout the day, based on your location.

纵观这两个功能，Dynamic Desktop（动态桌面）也就呼之欲出了，当然这也是 Mojave 的新特性之一。进入“System Preferences（系统偏好设置）> Desktop & Screen Saver（桌面与屏幕保护程序）”并且选择“Dynamic（动态）”，就能得到一个基于地理位置且全天候动态变化的壁纸。

{% asset desktop-and-screen-saver-preference-pane.png %}

The result is subtle and delightful.
Having a background that tracks the passage of time
makes the desktop feel alive;
in tune with the natural world.
(If nothing else,
it makes for a lovely visual effect when switching dark mode on and off)

效果不仅微妙，而且让人愉悦。桌面仿佛被赋予了生命，能随着时间的推移而变化；符合自然规律。（不出意外的话，结合 dark mode 的切换，还会有讨喜的特效）

_But how does it work, exactly?_<br/>
That's the question for this week's NSHipster article.

_这到底是如何实现的呢？_<br/>
这便是本周 NSHipster 讨论的问题。

The answer involves a deep dive into image formats,
a little bit of reverse-engineering
and even some spherical trigonometry.

答案会深入探究图片格式，同时涉及一些逆向工程以及球面三角学相关的内容。

---

The first step to understanding how Dynamic Desktop works
is to get hold of a dynamic image.

理解 Dynamic Desktop 第一步，就是要找到这些动态图片。

If you're running macOS Mojave
open Finder,
select "Go > Go to Folder..." (<kbd>⇧</kbd><kbd>⌘</kbd><kbd>G</kbd>),
and enter "/Library/Desktop Pictures/".

在 macOS Mojave 系统下，打开 Finder（访达），选择“Go（前往）> Go to Folder...（前往文件夹...）” （<kbd>⇧</kbd><kbd>⌘</kbd><kbd>G</kbd>），输入“/Library/Desktop Pictures/”。

{% asset go-to-library-desktop-pictures.png %}

In this directory,
you should find a file named "Mojave.heic".
Double-click it to open it in Preview.

在这个目录下，可以找到名为“Mojave.heic”的文件。双击通过 Preview（预览）打开。

{% asset mojave-heic.png %}

In Preview,
the sidebar shows a list of thumbnails numbered 1 through 16,
each showing a different view of the desert scene.

在预览中，左边栏会显示从 1 ~ 16 的缩略图，每张都是不同状态的沙漠图。

{% asset mojave-dynamic-desktop-images.png %}

If we select "Tools > Show Inspector" (<kbd>⌘</kbd><kbd>I</kbd>),
we get some general information about what we're looking at:

如果选择“Tools（工具）> Show Inspector（显示检查器）”（<kbd>⌘</kbd><kbd>I</kbd>），可以看到更为详细的信息，如下图所示：

{% asset mojave-heic-preview-info.png %}

Unfortunately, that's about all Preview gives us
(at least at the time of writing).
If we click on the next panel over, "More Info Inspector",
we don't learn a whole lot more about our subject:

不幸的是，这些就是预览所展示的全部信息了（截至发稿前）。即使点击旁边的“More Info Inspector（更多信息检查器）”，我们也只是能得到下面这个表格，其余的无从得知：

|              |            |
| ------------ | ---------- |
| Color Model  | RGB        |
| Depth:       | 8          |
| Pixel Height | 2,880      |
| Pixel Width  | 5,120      |
| Profile Name | Display P3 |

{% info %}

The `.heic` file extension corresponds to image containers
encoded using the <abbr title="High-Efficiency Image File Format">HEIF</abbr>,
or High-Efficiency Image File Format
(which is itself based on <abbr title="High-Efficiency Video Compression">HEVC</abbr>,
or High-Efficiency Video Compression ---
also known as H.265 video).
For more information, check out
[WWDC 2017 Session 503 "Introducing HEIF and HEVC"](https://developer.apple.com/videos/play/wwdc2017/503/)

后缀 `.heic` 表示图片容器采用 <abbr title="High-Efficiency Image File Format">HFIF</abbr>（High-Efficiency Image File Format）编码，即高效率图档格式（这种格式基于 <abbr title="High-Efficiency Video Compression">HEVC</abbr>（High-Efficiency Video Compression），即高效率视频压缩，也就是 H.265）。更多信息，可以参考 [WWDC 2017 Session 503 "Introducing HEIF and HEVC"](https://developer.apple.com/videos/play/wwdc2017/503/)

{% endinfo %}

If we want to learn more,
we'll need to roll up our sleeves
and get our hands dirty with some low-level APIs.

想要获得更多的数据，我们还需要脚踏实地，真真切切的深入底层 API。

## Digging Deeper with CoreGraphics

## 利用 CoreGraphics 一探究竟

Let's start our investigation by creating a new Xcode Playground.
For simplicity, we can hard-code a URL to the "Mojave.heic" file on our system.

第一步先创建 Xcode Playground。简单起见，我们将“Mojave.heic”文件路径硬编码到代码中。

```swift
import Foundation
import CoreGraphics

// macOS 10.14 Mojave Required
let url = URL(fileURLWithPath: "/Library/Desktop Pictures/Mojave.heic")
```

Next, create a `CGImageSource`,
copy its metadata,
and enumerate over each of its tags:

然后，创建 `CGImageSource`，拷贝元数据并遍历全部标签：

```swift
let source = CGImageSourceCreateWithURL(url as CFURL, nil)!
let metadata = CGImageSourceCopyMetadataAtIndex(source, 0, nil)!
let tags = CGImageMetadataCopyTags(metadata) as! [CGImageMetadataTag]
for tag in tags {
    guard let name = CGImageMetadataTagCopyName(tag),
        let value = CGImageMetadataTagCopyValue(tag)
    else {
        continue
    }

    print(name, value)
}
```

When we run this code, we get two results:
`hasXMP`, which has a value of `"True"`,
and `solar`, which has a decidedly less understandable value:

运行这段代码，会得到两个值：一个是 `hasXMP`，值为 `"True"`，另一个是 `solar`，它的值是一串看不大懂的数据：

```
YnBsaXN0MDDRAQJSc2mvEBADDBAUGBwgJCgsMDQ4PEFF1AQFBgcICQoLUWlRelFh
UW8QACNAcO7vOubr3yO/1e+pmkOtXBAB1AQFBgcNDg8LEAEjQFRxqCKOFiAjwCR6
waUkDgHUBAUGBxESEwsQAiNAVZV4BI4c+CPAEP2uFrMcrdQEBQYHFRYXCxADI0BW
tALKmrjwIz/2ObLnx6l21AQFBgcZGhsLEAQjQFfTrJlEjnwjQByrLle1Q0rUBAUG
Bx0eHwsQBSNAWPrrmI0ISCNAKiwhpSRpc9QEBQYHISIjCxAGI0BgJff9KDpyI0BE
NTOsilht1AQFBgclJicLEAcjQGbHdYIVQKojQEq3fAg86lXUBAUGBykqKwsQCCNA
bTGmpC2YRiNAQ2WFOZGjntQEBQYHLS4vCxAJI0BwXfII2B+SI0AmLcjfuC7g1AQF
BgcxMjMLEAojQHCnF6YrsxcjQBS9AVBLTq3UBAUGBzU2NwsQCyNAcTcSnimmjCPA
GP5E0ASXJtQEBQYHOTo7CxAMI0BxgSADjxK2I8AoalieOTyE1AQFBgc9Pj9AEA0j
QHNWsnnMcWIjwEO+oq1pXr8QANQEBQYHQkNEQBAOI0ABZpkFpAcAI8BKYGg/VvMf
1AQFBgdGR0hAEA8jQErBKblRzPgjwEMGElBIUO0ACAALAA4AIQAqACwALgAwADIA
NAA9AEYASABRAFMAXABlAG4AcAB5AIIAiwCNAJYAnwCoAKoAswC8AMUAxwDQANkA
4gDkAO0A9gD/AQEBCgETARwBHgEnATABOQE7AUQBTQFWAVgBYQFqAXMBdQF+AYcB
kAGSAZsBpAGtAa8BuAHBAcMBzAHOAdcB4AHpAesB9AAAAAAAAAIBAAAAAAAAAEkA
AAAAAAAAAAAAAAAAAAH9
```

### Shining Light on Solar

### 太阳之光

Most of us would look at that wall of text
and quietly close the lid of our MacBook Pro.
But, as some of you surely noticed,
this text looks an awful lot like it's
[Base64-encoded](https://en.wikipedia.org/wiki/Base64).

大多数人看到这串文字，就会默默合上 MacBook Pro，大呼告辞。但一定有人发现，这串文字非常像 [Base64 编码](https://en.wikipedia.org/wiki/Base64) 的杰作。

Let's test out our hypothesis in code:

让我们来验证一下这个假设：

```swift
if name == "solar" {
    let data = Data(base64Encoded: value)!
    print(String(data: data, encoding: .ascii))
}
```

<samp>
bplist00Ò\u{01}\u{02}\u{03}...
</samp>

What's that?
`bplist`, followed by a bunch of garbled nonsense?

这又是什么？
`bplist` 后面接了一串乱码？

By golly, that's the
[file signature](https://en.wikipedia.org/wiki/File_format#Magic_number)
for a [binary property list](https://en.wikipedia.org/wiki/Property_list).

天哪，原来这是[二进制属性列表](https://en.wikipedia.org/wiki/Property_list)的[文件签名](https://en.wikipedia.org/wiki/File_format#Magic_number)。

Let's see if `PropertyListSerialization` can make any sense of it...

利用 `PropertyListSerialization` 来看看呢...

```swift
if name == "solar" {
    let data = Data(base64Encoded: value)!
    let propertyList = try PropertyListSerialization
                            .propertyList(from: data,
                                          options: [],
                                          format: nil)
    print(propertyList)
}
```

```
(
    ap = {
        d = 15;
        l = 0;
    };
    si = (
        {
            a = "-0.3427528387535028";
            i = 0;
            z = "270.9334057827345";
        },
        ...
        {
            a = "-38.04743388682423";
            i = 15;
            z = "53.50908581251309";
        }
    )
)
```

_Now we're talking!_

_清晰多了！_

We have two top-level keys:

首先有两个一级键：

The `ap` key corresponds to
a dictionary containing integers for the `d` and `l` keys.

`ap` 键对应的值是包含 `d` 和 `l` 两个键的字典，它们的值都是整型。

The `si` key corresponds to
an array of dictionaries with integer and floating-point values.
Of the nested dictionary keys,
`i` is the easiest to understand:
incrementing from 0 to 15,
they're the index of the image in the sequence.
It'd be hard to guess `a` and `z` without any additional information,
but they turn out to represent the altitude (`a`) and azimuth (`z`)
of the sun in the corresponding pictures.

`si` 键对应的值是包含多个字典的数组，字典中有整型，也有浮点型的值。在嵌套的字典中，`i` 最容易理解：它从 0 一直递增到 15，这表示的是图片序列的下标。在没有更多信息的情况下，很难猜测 `a` 与 `z` 的含义，其实它们表示相应图片中太阳的高度（`a`）和方位角（`z`）。

### Calculating Solar Position

### 计算太阳的位置

At the time of writing,
those of us in the northern hemisphere
are settling into the season of autumn
and its shorter, colder days,
whereas those of us in the southern hemisphere
are gearing up for hotter and longer days.
The changing of the seasons reminds us that
the duration of a solar day depends where you are on the planet
and where the planet is in its orbit around the sun.

就在我落笔之时，身处北半球的人正在进入秋季，白昼变短，气温变低，而南半球的人却经历着白昼变长，气温变高。季节的变化告诉我们，日照的时长取决于你在星球上的位置，以及星球绕太阳的轨道。

The good news is that astronomers can tell you ---
with perfect accuracy ---
where the sun is in the sky for any location or time.
The bad news is that
the necessary calculations are
[complicated](https://en.wikipedia.org/wiki/Position_of_the_Sun)
to say the least.

可喜的是，天文学家能告诉你 —— 而且相当准确 —— 太阳在天空中的位置或时间。不可贺的是，这其中的计算十分[复杂](https://en.wikipedia.org/wiki/Position_of_the_Sun)。

Honestly, we don't really understand it ourselves,
and are pretty much just porting whatever code we manage to find online.
After some trial and error,
we were able to arrive at
[something that seems to work](https://github.com/NSHipster/DynamicDesktop/blob/master/SolarPosition.playground)
(PRs welcome!):

但老实讲，我们并不用过分深究它，在网上能找到相关的代码。经过不断的试错，[它们就能为我所用](https://github.com/NSHipster/DynamicDesktop/blob/master/SolarPosition.playground)（欢迎 PR！）：

```swift
import Foundation
import CoreLocation

// Apple Park, Cupertino, CA
let location = CLLocation(latitude: 37.3327, longitude: -122.0053)
let time = Date()

let position = solarPosition(for: location, at: time)
let formattedDate = DateFormatter.localizedString(from: time,
                                                    dateStyle: .medium,
                                                    timeStyle: .short)
print("Solar Position on \(formattedDate)")
print("\(position.azimuth)° Az / \(position.elevation)° El")
```

<samp>
Solar Position on Oct 1, 2018 at 12:00
180.73470025840783° Az / 49.27482549913847° El
</samp>

At noon on October 1, 2018,
the sun shines on Apple Park from the south,
about halfway between the horizon and directly overhead.

2018 年 10 月 1 日中午，太阳从南面照射在 Apple Park，大约处于地平线中间，直射头顶。

If track the position of the sun over an entire day,
we get a sinusoidal shape reminiscent of the Apple Watch "Solar" face.

如果绘制出太阳一天的位置，我们可以得到一个正弦曲线，这不禁让人联想到 Apple Watch 的“太阳表盘”。

{% asset solar-position-watch-faces.jpg %}

### Extending Our Understanding of XMP

### 扩展对 XMP 的理解

Alright, enough astronomy for the moment.
Let's ground ourselves in something much more banal:
_de facto_ XML metadata standards.

好吧，天文学到此结束。接下来是一个乏味的过程：_摆在眼前的_ XML 元数据。

Remember the `hasXMP` metadata key from before?
Yeah, _that_.

还记得之前的元数据键 `hasXMP` 吗？对，_就是它_。

<abbr title="Extensible Metadata Platform">XMP</abbr>,
or Extensible Metadata Platform,
is a standard format for tagging files with metadata.
What does XMP look like?
Brace yourself:

<abbr title="Extensible Metadata Platform">XMP</abbr>（Extensible Metadata Platform），即可扩展元数据平台，是一种使用元数据标记文件的标准格式。XMP 长什么样呢？请打起精神来：

```swift
let xmpData = CGImageMetadataCreateXMPData(metadata, nil)
let xmp = String(data: xmpData as! Data, encoding: .utf8)!
print(xmp)
```

```xml
<x:xmpmeta xmlns:x="adobe:ns:meta/" x:xmptk="XMP Core 5.4.0">
   <rdf:RDF xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#">
      <rdf:Description rdf:about=""
            xmlns:apple_desktop="http://ns.apple.com/namespace/1.0/">
         <apple_desktop:solar>
            <!-- (Base64-Encoded Metadata) -->
        </apple_desktop:solar>
      </rdf:Description>
   </rdf:RDF>
</x:xmpmeta>
```

_Yuck._

_呕。_

But it's a good thing that we checked.
We'll need to honor that `apple_desktop` namespace
to make our own Dynamic Desktop images work correctly.

不过也幸好我们检查了一下。之后想要成功自定义 Dynamic Desktop，还得仰仗 `apple_desktop` 命名空间。

Speaking of, let's get started on that.

既然如此，就开始吧。

## Creating Our Own Dynamic Desktop

## 创建自定义 Dynamic Desktop

Let's create a data model to represent a Dynamic Desktop:

首先，创建一个数据模型来表示 Dynamic Desktop：

```swift
struct DynamicDesktop {
    let images: [Image]

    struct Image {
        let cgImage: CGImage
        let metadata: Metadata

        struct Metadata: Codable {
            let index: Int
            let altitude: Double
            let azimuth: Double

            private enum CodingKeys: String, CodingKey {
                case index = "i"
                case altitude = "a"
                case azimuth = "z"
            }
        }
    }
}
```

Each Dynamic Desktop comprises an ordered sequence of images,
each of which has image data, stored in a `CGImage` object,
and metadata, as discussed before.
We adopt `Codable` in the `Metadata` declaration
in order for the compiler to automatically synthesize conformance.
We'll take advantage of that when it comes time
to generate the Base64-encoded binary property list.

如前文所述，每个 Dynamic Desktop 都由一个有序的图片序列构成，每个图片又包含存储在 `CGImage` 对象中的图片数据和元数据。`Metadata` 采用 `Codable` 类型，是为了编译器自动合成相关函数。我们能在生成 Base64 编码的二进制属性列表时感受到它的优势。

### Writing to an Image Destination

### 写入图片目标

First, create a `CGImageDestination`
with a specified output URL.
The file type is `heic` and the source count
is equal to the number of images to be included.

首先，创建一个指定输出 URL 的 `CGImageDestination`。文件类型为 `heic`，资源数量即需要包含的图片张数。

```swift
guard let imageDestination = CGImageDestinationCreateWithURL(
                                outputURL as CFURL,
                                AVFileType.heic as CFString,
                                dynamicDesktop.images.count,
                                nil
                             )
else {
    fatalError("Error creating image destination")
}
```

Next, enumerate over each image in the dynamic desktop object.
By using the `enumerated()` method,
we also get the current `index` for each loop
so that we can set the image metadata on the first image:

接着，遍历动态桌面对象中的全部图片。通过 `enumerated()` 方法，我们还能获取到当前 `index`，这样就可以在第一张图片上设置图片元数据：

```swift
for (index, image) in dynamicDesktop.images.enumerated() {
    if index == 0 {
        let imageMetadata = CGImageMetadataCreateMutable()
        guard let tag = CGImageMetadataTagCreate(
                            "http://ns.apple.com/namespace/1.0/" as CFString,
                            "apple_desktop" as CFString,
                            "solar" as CFString,
                            .string,
                            try! dynamicDesktop.base64EncodedMetadata() as CFString
                        ),
            CGImageMetadataSetTagWithPath(
                imageMetadata, nil, "xmp:solar" as CFString, tag
            )
        else {
            fatalError("Error creating image metadata")
        }

        CGImageDestinationAddImageAndMetadata(imageDestination,
                                              image.cgImage,
                                              imageMetadata,
                                              nil)
    } else {
        CGImageDestinationAddImage(imageDestination,
                                   image.cgImage,
                                   nil)
    }
}
```

Aside from the unrefined nature of Core Graphics APIs,
the code is pretty straightforward.
The only part that requires further explanation is the call to
`CGImageMetadataTagCreate(_:_:_:_:_:)`.

除了较为繁杂的 Core Graphics API 以外，代码可以说非常直观了。唯一需要进一步解释的只有 `CGImageMetadataTagCreate(_:_:_:_:_:)`。

Because of a mismatch between how image and container metadata are structured
and how they're represented in code,
we have to implement `Encodable` for `DynamicDesktop` ourselves:

由于图片与元数据容器的结构、代码的表现形式均不同，所以我们不得不为 `DynamicDesktop` 实现 `Encodable` 协议：

```swift
extension DynamicDesktop: Encodable {
    private enum CodingKeys: String, CodingKey {
        case ap, si
    }

    private enum NestedCodingKeys: String, CodingKey {
        case d, l
    }

    func encode(to encoder: Encoder) throws {
        var keyedContainer =
            encoder.container(keyedBy: CodingKeys.self)

        var nestedKeyedContainer =
            keyedContainer.nestedContainer(keyedBy: NestedCodingKeys.self,
                                           forKey: .ap)

        // FIXME: Not sure what `l` and `d` keys indicate
        try nestedKeyedContainer.encode(0, forKey: .l)
        try nestedKeyedContainer.encode(self.images.count, forKey: .d)

        var unkeyedContainer =
            keyedContainer.nestedUnkeyedContainer(forKey: .si)
        for image in self.images {
            try unkeyedContainer.encode(image.metadata)
        }
    }
}
```

With that in place,
we can implement the aforementioned `base64EncodedMetadata()` method like so:

有了这个，就可以实现之前代码中提到的 `base64EncodedMetadata()` 方法了：

```swift
extension DynamicDesktop {
    func base64EncodedMetadata() throws -> String {
        let encoder = PropertyListEncoder()
        encoder.outputFormat = .binary

        let binaryPropertyListData = try encoder.encode(self)
        return binaryPropertyListData.base64EncodedString()
    }
}
```

Once the for-in loop is exhausted,
and all images and metadata are written,
we call `CGImageDestinationFinalize(_:)` to finalize the image source
and write the image to disk.

当 for-in 循环执行完，也就表明所有图片和元数据均被写入，我们可以调用 `CGImageDestinationFinalize(_:)` 方法终止图片源，并将图片写入磁盘。

```swift
guard CGImageDestinationFinalize(imageDestination) else {
    fatalError("Error finalizing image")
}
```

If everything worked as expected,
you should now be the proud owner of a brand new Dynamic Desktop.
Nice!

如果一切顺利，就可以为重新定义 Dynamic Desktop 的自己而感到骄傲了。
棒！

---

We love the Dynamic Desktop feature in Mojave,
and are excited to see the same proliferation of them
that we saw when wallpapers hit the mainstream with Windows 95.

我们非常喜欢 Mojave 的 Dynamic Desktop 特性，并且也很欣慰看到它仿佛重现了 Windows 95 壁纸进入主流市场时的辉煌。

If you're so inclined,
here are a few ideas for where to go from here:

如果你也这样想，下面还有些想法可供参考：

### Automatically Generating a Dynamic Desktop from Photos

### 照片自动生成 Dynamic Desktop

It's mind-blowing to think that something as transcendent as
the movement of celestial bodies
can be reduced to a system of equations with two inputs:
time and place.

让人振奋的是，天体运动这样高不可攀的研究，竟然可以简化用二元方程来表达：时间与位置。

In the example before,
this information is hard-coded,
but you could ostensibly extract that information
from images automatically.

在之前的例子中，这部分信息都是硬编码的，但其实它们可以通过读取图片数据来自动获取。

By default,
the camera on most phones captures
[Exif metadata](https://en.wikipedia.org/wiki/Exif)
each time a photo is snapped.
This metadata can include the time which the photo was taken
and the GPS coordinates of the device at the time.

默认情况下，绝大部分手机的相机都会捕获拍摄时的 [Exif 元数据](https://en.wikipedia.org/wiki/Exif)。元数据包含了照片拍摄的时间，以及当时设备的 GPS 坐标。

By reading time and location information directly from image metadata,
you can automatically determine solar position
and simplify the process of generating a Dynamic Desktop
from a series of photos.

通过读取元数据中的时间与位置信息，能自动获取太阳的位置，那么从一系列图片中生成 Dynamic Desktop 也就顺理成章了。

### Shooting a Time Lapse on Your iPhone

### iPhone 上的延时摄影

Want to put your new iPhone XS to good use?
(Or more accurately,
"Want to use your old iPhone for something productive
while you procrastinate selling it?")

想要好好利用手上全新的 iPhone Xs 吗？（更确切的说，“在纠结卖不卖旧 iPhone 的时候，可以先用它来做些有创意的事？”）

Mount your phone against a window,
plug it into a charger,
set the Camera to Timelapse mode,
and hit the "Record" button.
By extracting key frames from the resulting video,
you can make your very own _bespoke_ Dynamic Desktop.

将手机充上电，摆在窗前，打开相机的延时摄影模式，点击“拍摄”按钮。从最后的视频中选出一些关键帧，就可以制作 _专属_ Dynamic Desktop 了。

You might also want to check out
[Skyflow](https://itunes.apple.com/us/app/skyflow-time-lapse-shooting/id937208291?mt=8)
or similar apps that more easily allow you to
take still photos at predefined intervals.

当然，你可以看看 [Skyflow](https://itunes.apple.com/us/app/skyflow-time-lapse-shooting/id937208291?mt=8) 这类应用，它能设置时间间隔来拍摄静态图片。

### Generating Landscapes from GIS Data

### 通过 GIS 数据打造风景

If you can't stand to be away from your phone for an entire day (sad)
or don't have anything remarkable to look (also sad),
you could always create your own reality (sounds sadder than it is).

如果你无法忍受手机一整天不在身边（伤心），又或者没什么标志性景象值得拍摄（依然伤心），你还可以创造一个属于自己的世界（这比现实本身还要令人伤心）。

Using an app like
[Terragen](https://planetside.co.uk),
you can render photo-realistic 3D landscapes,
with fine-tuned control over the earth, sun, and sky.

可以选择用 [Terragen](https://planetside.co.uk/) 这类应用，它打造了一个逼真的 3D 世界，还能对太阳、地球、天空进行微调。

You can make it even easier for yourself by
downloading an elevation map from the U.S. Geological Survey's
[National Map website](https://viewer.nationalmap.gov/basic/)
and using that as a template for your 3D rendering project.

想要更加简化，还可以从美国地质调查局的[国家地图网站](https://viewer.nationalmap.gov/basic/)上下载高程地图，以用于 3D 渲染的模板。

### Downloading Pre-Made Dynamic Desktops

### 下载预制的 Dynamic Desktops

Or if you have actual work to do
and can't be bothered to spend your time making pretty pictures,
you can always just pay someone else to do it for you.

再或者，你每天都非常多的工作要做，抽不出时间捣腾好看的图片，也可以选择付费从别人那里购买。

We're personally fans of the the
[24 Hour Wallpaper](https://www.jetsoncreative.com/24hourwallpaper/) app.
If you have any other recommendations,
[@ us on Twitter!](https://twitter.com/NSHipster/).

我个人是 [24 Hour Wallpaper](https://www.jetsoncreative.com/24hourwallpaper) 这款应用的粉丝。如果你有别的推荐，欢迎[联系我们](https://twitter.com/NSHipster/)。