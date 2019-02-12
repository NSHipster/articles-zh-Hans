---
title: NSDataAsset
author: Mattt
translator: Xiang Wang
category: Cocoa
excerpt: >
    有很多加速网络请求的技术：压缩和流技术、缓存和预加载、连接池和多路复用、延迟和后台运行。然而，还有一种比它们优先级更高，效果更好的优化策略：_压根就不发请求_。
status:
    swift: 4.2
---

On the web,
speed isn't a luxury;
it's a matter of survival.

在 Web 的世界里，速度不是一种奢求；它事关生死。

User studies in recent years
suggest that _any_ perceivable latency in page load time ---
that is, greater than 400 milliseconds
(literally in the blink of an eye) ---
can negatively impact conversion and engagement rates.
For every additional second that a webpage takes to load,
one should expect 10% of users to swipe back or close the tab.

近年来的用户研究表明，页面加载中 _任何_ 可以察觉到的延迟 —— 即大于 400 毫秒（字面意义上的“一眨眼的功夫”） —— 都会对转化率和参与率产生负面影响。网页加载时每多花一秒，就会多 10% 的用户返回或者关闭这个页面。

For large internet companies like Google, Amazon, and Netflix,
an extra second here and there
can mean _billions_ of dollars in annual revenue.
So it's no surprise those very same companies
have committed so much engineering effort into making the web fast.

对于谷歌、亚马逊和 Netflix 这样的大型的互联网公司而言，加载时多花一秒钟就意味着损失 _数十亿_ 美元的年收入。所以那些公司投入如此多的工程努力来让网页更快，也没有什么奇怪的了。

There are many techniques for speeding up a network request:
compressing and streaming,
caching and prefetching,
connection pooling and multiplexing,
deferring and backgrounding.
And yet there's one optimization strategy
that both predates and outperforms them all:
_not making the request in the first place_.

有很多加速网络请求的技术：压缩和流技术、缓存和预加载、连接池和多路复用、延迟和后台运行。然而，还有一种比它们优先级更高，效果更好的优化策略：_压根就不发请求_。

Apps, by virtue of being downloaded ahead of time,
have a unique advantage over conventional web pages in this respect.
This week on NSHipster,
we'll show how to leverage the Asset Catalog in an unconventional way
to improve the first launch experience for your app.

在这个方面，App 凭借先下载后使用的特点，拥有传统网页所不具备的独特优势。在这一周的 NSHipster 里，我们将展示如何以一种非传统的方式使用 Asset Catalog 来改善你的 App 的首次启动体验。

---

Asset Catalogs allow you to organize resources
according to the characteristics of the current device.
For a given image,
you can provide different files depending on the
device (iPhone, iPad, Apple Watch, Apple TV, Mac),
screen resolution (`@2x` / `@3x`), or
color gamut (sRGB / P3).
For other kinds of assets,
you might offer variations depending on the available memory
or version of Metal.
Just request an asset by name,
and the most appropriate one is provided automatically.

Asset Catalog 允许你根据当前设备的特性来组织资源文件。对于一个给定的图片，你可以根据设备（iPhone、iPad、Apple Watch、Apple TV、Mac）、屏幕分辨率（`@2x` / `@3x`）或者色域（sRGB / P3），提供不同的文件。对于其他类型的 asset，你可能根据可用内存或者 Metal 版本的不同而提供不同的文件。请求 asset 时仅需提供名字，最合适的那个资源就会自动返回。

Beyond offering a more convenient API,
Asset Catalogs let apps take advantage of
[app thinning](https://help.apple.com/xcode/mac/current/#/devbbdc5ce4f),
resulting in smaller installations that are optimized for each user's device.

除了提供更简便的 API，Asset Catalog 还允许 App 使用 [app thinning](https://help.apple.com/xcode/mac/current/#/devbbdc5ce4f) 为每个用户设备提供一个经过优化的更小的安装包。

Images are far and away the most common types of assets,
but as of iOS 9 and macOS El Capitan,
resources like JSON, XML and other data file can join in the fun by way of
[`NSDataAsset`](https://developer.apple.com/documentation/uikit/nsdataasset).

图片是最常见的 Asset 类型，但是从 iOS 9 和 macOS El Capitan 开始，JSON、XML 和其他数据文件之类的资源也可以通过 [`NSDataAsset`](https://developer.apple.com/documentation/uikit/nsdataasset) 这种有趣的方式参与进来。

## How to Store and Retrieve Data with Asset Catalog

## 如何使用 Asset Catalog 存储和获取数据

As an example,
let's imagine an iOS app for creating digital color palettes.

举个例子，让我们想象一个用于创建数字调色板的 iOS App。

To distinguish between different shades of gray,
we might load a list of colors and their corresponding names.
Normally, we might download this from a server on first launch,
but that could cause a bad user experience if
[adverse networking conditions](https://nshipster.com/network-link-conditioner/)
block app functionality.
Since this is a relatively static data set,
why not include the data in the app bundle itself
by way of an Asset Catalog?

为了区分不同深浅的灰色，我们可能会加载一个颜色和对应名字的列表。通常情况下，我们可能会在第一次启动时从服务器下载这个列表，但是如果恶劣的网络环境限制了 App 的功能，就会导致很差的用户体验。既然它是一个相对静态的数据集，为什么不以一种 Asset Catalog 形式将它添加到 app bundle 中？

### Step 1. Add New Data Set to Asset Catalog

### 步骤 1：向 Asset Catalog 中添加 New Data Set

When you create a new app project in Xcode,
it automatically generates an Asset Catalog.
Select `Assets.xcassets` from the project navigator
to open the Asset Catalog editor.
Click the <kbd>+</kbd> icon at the bottom left
and select "New Data Set"

当你在 Xcode 中新建一个 app 项目时，它会自动生成一个 Asset Catalog。在项目导航（Project navigator）中选中 `Assets.xcassets`，打开 Asset Catalog 编辑器。点击左下方的 <kbd>+</kbd> 图标，然后选择 "New Data Set"。

{% asset add-new-data-set.png %}

![asset add-new-data-set.png](https://nshipster.com/assets/add-new-data-set-b6d8b1604dd12f49f1e034c0a36a42aa9fc6efc3f42d7320d9b489b6cec5fde0.png)

Doing this creates a new subdirectory of `Assets.xcassets`
with the `.dataset` extension.

这样会在 `Assets.xcassets` 下新建一个后缀名为 `.dataset` 的子目录。

{% info do %}

By default,
the Finder treats both of these bundles as directories,
which makes it easy to inspect and modify their contents as needed.

默认情况下，Finder （访达）把这两种 bundle 都当做目录，以便在需要时查看和修改它们的内容。

{% endinfo %}

### Step 2. Add a Data File

### 步骤2：添加数据文件

Open the Finder,
navigate to the data file
and drag-and-drop it
into the empty field for your data set asset in Xcode.

打开 Finder，找到数据文件，把它拖拽到 Xcode 中 data set asset 的空白处。

{% asset asset-catalog-any-any-universal.png %}

![asset asset-catalog-any-any-universal.png](https://nshipster.com/assets/asset-catalog-any-any-universal-f634190ce57540a9fa1406ded75e13936c390fb0552b374d584510896db186bc.png)

When you do this,
Xcode copies the file to the the `.dataset` subdirectory
and updates the `contents.json` metadata file
with the filename and
[Universal Type Identifier](https://en.wikipedia.org/wiki/Uniform_Type_Identifier).
of the file.

当你这么做时，Xcode 会把那个文件复制到 `.dataset` 子目录，并将它的文件名和 [通用类型标识符（Universal Type Identifier）](https://zh.wikipedia.org/wiki/统一类型标识) 更新到 `contents.json` 元数据文件。

```json
{
  "info": {
    "version": 1,
    "author": "xcode"
  },
  "data": [
    {
      "idiom": "universal",
      "filename": "colors.json",
      "universal-type-identifier": "public.json"
    }
  ]
}
```

### Step 3. Access Data Using NSDataAsset

### 步骤3：使用 NSDataAsset 访问数据

Now you can access the file's data
with the following code:

现在你可以使用如下代码访问文件的数据：

```swift
guard let asset = NSDataAsset(name: "NamedColors") else {
    fatalError("Missing data asset: NamedColors")
}

let data = asset.data
```

In the case of our color app,
we might call this from the `viewDidLoad()` method in a view controller
and use the resulting data to decode an array of model objects
to be displayed in a table view:

对于我们颜色 App，我们可能在一个 view controller 的 `viewDidLoad()` 方法中调用上面的代码，然后解码返回的数据，获取 model 对象的数组，并展示在一个 table view 上。

```swift
let decoder = JSONDecoder()
self.colors = try! decoder.decode([NamedColor].self, from: asset.data)
```

## Mixing It Up

## 混合一下

Data sets don't typically benefit from app thinning features of Asset Catalogs
(most JSON files, for example,
couldn't care less about what version of Metal is supported by the device).

Data set 通常无法从 Asset Catalog 的 app thinning 特性中获益（例如，大部分的 JSON 文件都不太关心设备所支持的 Metal 版本）。

But for our color palette app example,
we might provide different color lists on devices with a wide-gamut display.

但是对于我们的调色板 App，我们可能为支持广色域显示的设备提供不同的颜色列表。

To do this,
select the asset in the sidebar of the Asset Catalog editor
and click on the drop-down control labeled Gamut in the Attributes Inspector.

为了做到这一点，在 Asset Catalog 编辑器的侧边栏选中刚才的 asset，然后点击 Attributes Inspector 下名为 Gamut 的下拉控件。

{% asset select-color-gamut.png %}

![asset select-color-gamut.png](https://nshipster.com/assets/select-color-gamut-02114afe2b744c228c2b29b7277abb9ec7e2bcb9afa683cc80115792849988c4.png)

After providing bespoke data files for each gamut,
the `contents.json` metadata file should look something like this:

为每个色域提供定制的数据文件后，`contents.json` 元数据文件应该看起来像这样：

```json
{
  "info": {
    "version": 1,
    "author": "xcode"
  },
  "data": [
    {
      "idiom": "universal",
      "filename": "colors-srgb.json",
      "universal-type-identifier": "public.json",
      "display-gamut": "sRGB"
    },
    {
      "idiom": "universal",
      "filename": "colors-p3.json",
      "universal-type-identifier": "public.json",
      "display-gamut": "display-P3"
    }
  ]
}
```

## Keeping It Fresh

## 保鲜一下

Storing and retrieving data from the Asset Catalog is trivial.
What's actually difficult --- and ultimately more important ---
is keeping that data up-to-date.

使用 Asset Catalog 存储和获取数据是非常简单的。真正困难 —— 并最终更重要 —— 的是保持数据的更新。

Refresh data using `curl`, `rsync`, `sftp`,
Dropbox, BitTorrent, or Filecoin.
Kick things off from a shell script
(and call it in an Xcode Build Phase, if you like).
Add it to your `Makefile`, `Rakefile`, `Fastfile`,
or whatever is required of your build system of choice.
Delegate the task to Jenkins or Travis or that bored-looking intern.
Trigger it from a bespoke Slack integration
or create a Siri Shortcut so you can wow your colleagues with a casual
_"Hey Siri, update that data asset before it gets too stale"_.

使用 `curl`、`rsync`、`sftp`、Dropbox、BitTorrent 或 Filecoin 刷新数据。从一个 shell 脚本开始（如果你喜欢，可以在 Xcode Build Phase 中调用它）。将它添加到你的 `Makefile`、`Rakefile`、`Fastfile`，或者你的编译系统所要求的任何地方。将这个任务分配给 Jenkins、Travis 或者某个烦人的实习生。使用定制的 Slack integration 或者 Siri Shortcuts 触发它，这样你就可以用随意的一句 _"Hey Siri，在数据变得太旧之前更新一下"_，让你的同事大吃一惊。

**However you decide to synchronize your data,
just make sure it's automated and part of your release process.**

**注意，当你决定同步你的数据时，一定要确保它是自动化的，而且是你发布过程的一部分。**

Here's an example of a shell script you might run
to download the latest data file using `curl`:

下面是一个 shell 脚本示例，你可以运行它来使用 `curl` 下载最新的数据文件：

```shell
#!/bin/sh
CURL='/usr/bin/curl'
URL='https://example.com/path/to/data.json'
OUTPUT='./Assets.xcassets/Colors.dataset/data.json'

$CURL -fsSL -o $OUTPUT $URL
```

## Wrapping It Up

## 封装一下

Although the Assets Catalog performs lossless compression of image assets,
nothing from the documentation, Xcode Help, or WWDC sessions  
indicate that any such optimization is done for data assets (at least not yet).

虽然 Assets Catalog 会对 image asset 执行无损压缩，但没有任何文档、Xcode 帮助或 WWDC session 指出 data asset 上也存在这种优化（至少目前没有）。

For data assets larger than, say, a few hundred kilobytes,
you should consider using compression.
This is especially true for text files like JSON, CSV, and XML,
which typically compress down to 60% — 80% of their original size.

当 data asset 的文件大小大于，比如说几百 KB 时，你就要考虑使用压缩了。JSON、CSV 和 XML 之类的文本文件尤其如此，它们通常可以被压缩到原始大小的 60% - 80%。

We can add compression to our previous shell script
by piping the output of `curl` to `gzip` before writing to our file:

我们可以将 `curl` 的输出发送给 `gzip`，然后再写到我们的文件，从而为我们之前的 shell 脚本添加压缩功能。

```shell
#!/bin/sh
CURL='/usr/bin/curl'
GZIP='/usr/bin/gzip'
URL='https://example.com/path/to/data.json'
OUTPUT='./Assets.xcassets/Colors.dataset/data.json.gz'

$CURL -fsSL $URL | $GZIP -c > $OUTPUT
```

If you do adopt compression,
make sure that the `"universal-type-identifier"` field
reflects this:

如果你使用了压缩，请确保 `"universal-type-identifier"` 字段体现了这一点：

```json
{
  "info": {
    "version": 1,
    "author": "xcode"
  },
  "data": [
    {
      "idiom": "universal",
      "filename": "colors.json.gz",
      "universal-type-identifier": "org.gnu.gnu-zip-archive"
    }
  ]
}
```

On the client-side,
it's up to you to decompress data from the asset catalog before use.
If you had a `Gzip` module, you might do the following:

在客户端上，你使用 asset catalog 之前需要先解压数据。如果有 `Gzip` 模块，你可能会做以下事情：

```swift
do {
    let data = try Gzip.decompress(data: asset.data)
} catch {
    fatalError(error.localizedDescription)
}
```

Or, if you do this multiple times in your app,
you could create a convenience method in an extension to `NSDataAsset`:

或者，如果你会在 App 中反复地这么做，那么可以在 `NSDataAsset` 的扩展中创建一个便利方法：

```swift
extension NSDataAsset {
    func decompressedData() throws -> Data {
        return try Gzip.decompress(data: self.data)
    }
}
```

{% info do %}

你还可以考虑使用 [Git Large File Storage (LFS)](https://git-lfs.github.com) 实现大型 data asset 文件的版本控制。

{% endinfo %}

---

Although it's tempting to assume that all of your users
enjoy fast, ubiquitous network access over WiFi and LTE,
this isn't true for everyone,
and certainly not all the time.

尽管人们容易认为所有用户都享受着快速的、无处不在的 WiFi 和 LTE 网络，但这并不适用于所有人，也不适用于所有时段。

Take a moment to see what networking calls your app makes at launch,
and consider if any of these might benefit from being pre-loaded.
Making a good first impression
could mean the difference between a long-term active use
and deletion after a few seconds.

花点时间看看你的 App 在启动时发出的网络请求，然后考虑哪些可能从预加载中受益。给人留下好的第一印象可能意味着你的 App 是被长期地积极地使用着，而不是几秒钟之后就被删除。
