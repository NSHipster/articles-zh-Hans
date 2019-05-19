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

在 Web 的世界里，速度不是一种奢求；它事关生死。

近年来的用户研究表明，页面加载中 _任何_ 可以察觉到的延迟 —— 即大于 400 毫秒（字面意义上的“一眨眼的功夫”） —— 都会对转化率和参与率产生负面影响。网页加载时每多花一秒，就会多 10% 的用户返回或者关闭这个页面。

对于谷歌、亚马逊和 Netflix 这样的大型的互联网公司而言，加载时多花一秒钟就意味着损失 _数十亿_ 美元的年收入。所以那些公司投入如此多的工程努力来让网页更快，也没有什么奇怪的了。

有很多加速网络请求的技术：压缩和流技术、缓存和预加载、连接池和多路复用、延迟和后台运行。然而，还有一种比它们优先级更高，效果更好的优化策略：_压根就不发请求_。

在这个方面，App 凭借先下载后使用的特点，拥有传统网页所不具备的独特优势。在这一周的 NSHipster 里，我们将展示如何以一种非传统的方式使用 Asset Catalog 来改善你的 App 的首次启动体验。

---

Asset Catalog 允许你根据当前设备的特性来组织资源文件。对于一个给定的图片，你可以根据设备（iPhone、iPad、Apple Watch、Apple TV、Mac）、屏幕分辨率（`@2x` / `@3x`）或者色域（sRGB / P3），提供不同的文件。对于其他类型的 asset，你可能根据可用内存或者 Metal 版本的不同而提供不同的文件。请求 asset 时仅需提供名字，最合适的那个资源就会自动返回。

除了提供更简便的 API，Asset Catalog 还允许 App 使用 [app thinning](https://help.apple.com/xcode/mac/current/#/devbbdc5ce4f) 为每个用户设备提供一个经过优化的更小的安装包。

图片是最常见的 Asset 类型，但是从 iOS 9 和 macOS El Capitan 开始，JSON、XML 和其他数据文件之类的资源也可以通过 [`NSDataAsset`](https://developer.apple.com/documentation/uikit/nsdataasset) 这种有趣的方式参与进来。

## 如何使用 Asset Catalog 存储和获取数据

举个例子，让我们想象一个用于创建数字调色板的 iOS App。

为了区分不同深浅的灰色，我们可能会加载一个颜色和对应名字的列表。通常情况下，我们可能会在第一次启动时从服务器下载这个列表，但是如果恶劣的网络环境限制了 App 的功能，就会导致很差的用户体验。既然它是一个相对静态的数据集，为什么不以一种 Asset Catalog 形式将它添加到 app bundle 中？

### 步骤 1：向 Asset Catalog 中添加 New Data Set

当你在 Xcode 中新建一个 app 项目时，它会自动生成一个 Asset Catalog。在项目导航（Project navigator）中选中 `Assets.xcassets`，打开 Asset Catalog 编辑器。点击左下方的 <kbd>+</kbd> 图标，然后选择 "New Data Set"。

![asset add-new-data-set.png](https://nshipster.com/assets/add-new-data-set-b6d8b1604dd12f49f1e034c0a36a42aa9fc6efc3f42d7320d9b489b6cec5fde0.png)

这样会在 `Assets.xcassets` 下新建一个后缀名为 `.dataset` 的子目录。

{% info do %}

默认情况下，Finder （访达）把这两种 bundle 都当做目录，以便在需要时查看和修改它们的内容。

{% endinfo %}

### 步骤2：添加数据文件

打开 Finder，找到数据文件，把它拖拽到 Xcode 中 data set asset 的空白处。

![asset asset-catalog-any-any-universal.png](https://nshipster.com/assets/asset-catalog-any-any-universal-f634190ce57540a9fa1406ded75e13936c390fb0552b374d584510896db186bc.png)

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

### 步骤3：使用 NSDataAsset 访问数据

现在你可以使用如下代码访问文件的数据：

```swift
guard let asset = NSDataAsset(name: "NamedColors") else {
    fatalError("Missing data asset: NamedColors")
}

let data = asset.data
```

对于我们颜色 App，我们可能在一个 view controller 的 `viewDidLoad()` 方法中调用上面的代码，然后解码返回的数据，获取 model 对象的数组，并展示在一个 table view 上。

```swift
let decoder = JSONDecoder()
self.colors = try! decoder.decode([NamedColor].self, from: asset.data)
```

## 混合一下

Data set 通常无法从 Asset Catalog 的 app thinning 特性中获益（例如，大部分的 JSON 文件都不太关心设备所支持的 Metal 版本）。

但是对于我们的调色板 App，我们可能为支持广色域显示的设备提供不同的颜色列表。

为了做到这一点，在 Asset Catalog 编辑器的侧边栏选中刚才的 asset，然后点击 Attributes Inspector 下名为 Gamut 的下拉控件。

![asset select-color-gamut.png](https://nshipster.com/assets/select-color-gamut-02114afe2b744c228c2b29b7277abb9ec7e2bcb9afa683cc80115792849988c4.png)

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

## 保鲜一下

使用 Asset Catalog 存储和获取数据是非常简单的。真正困难 —— 并最终更重要 —— 的是保持数据的更新。

使用 `curl`、`rsync`、`sftp`、Dropbox、BitTorrent 或 Filecoin 刷新数据。从一个 shell 脚本开始（如果你喜欢，可以在 Xcode Build Phase 中调用它）。将它添加到你的 `Makefile`、`Rakefile`、`Fastfile`，或者你的编译系统所要求的任何地方。将这个任务分配给 Jenkins、Travis 或者某个烦人的实习生。使用定制的 Slack integration 或者 Siri Shortcuts 触发它，这样你就可以用随意的一句 _"Hey Siri，在数据变得太旧之前更新一下"_，让你的同事大吃一惊。

**注意，当你决定同步你的数据时，一定要确保它是自动化的，而且是你发布过程的一部分。**

下面是一个 shell 脚本示例，你可以运行它来使用 `curl` 下载最新的数据文件：

```shell
#!/bin/sh
CURL='/usr/bin/curl'
URL='https://example.com/path/to/data.json'
OUTPUT='./Assets.xcassets/Colors.dataset/data.json'

$CURL -fsSL -o $OUTPUT $URL
```

## 封装一下

虽然 Assets Catalog 会对 image asset 执行无损压缩，但没有任何文档、Xcode 帮助或 WWDC session 指出 data asset 上也存在这种优化（至少目前没有）。

当 data asset 的文件大小大于，比如说几百 KB 时，你就要考虑使用压缩了。JSON、CSV 和 XML 之类的文本文件尤其如此，它们通常可以被压缩到原始大小的 60% - 80%。

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

在客户端上，你使用 asset catalog 之前需要先解压数据。如果有 `Gzip` 模块，你可能会做以下事情：

```swift
do {
    let data = try Gzip.decompress(data: asset.data)
} catch {
    fatalError(error.localizedDescription)
}
```

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

尽管人们容易认为所有用户都享受着快速的、无处不在的 WiFi 和 LTE 网络，但这并不适用于所有人，也不适用于所有时段。

花点时间看看你的 App 在启动时发出的网络请求，然后考虑哪些可能从预加载中受益。给人留下好的第一印象可能意味着你的 App 是被长期地积极地使用着，而不是几秒钟之后就被删除。
