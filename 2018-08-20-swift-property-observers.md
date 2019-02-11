---
layout: post
title: Swift Property Observers
author: Mattt
translator: Hale
category: Swift
excerpt: >
  现代软件开发已经被视为可能成为鲁布·戈德堡机械装置的典范。然而在进行一些远程操作时，它将显得更加清晰。
status:
  swift: 4.2
---

到了 20 世纪 30 年代，Rube Goldberg 已成为家喻户晓的名字，与[“自营餐巾”](https://upload.wikimedia.org/wikipedia/commons/a/a9/Rube_Goldberg%27s_%22Self-Operating_Napkin%22_%28cropped%29.gif) 等漫画中描绘的奇异复杂和异想天开的发明同义。大约在同一时期，阿尔伯特·爱因斯坦对尼尔斯·玻尔量子力学的普遍解释进行了[批判](https://zh.wikipedia.org/wiki/%E7%88%B1%E5%9B%A0%E6%96%AF%E5%9D%A6-%E6%B3%A2%E5%A4%9A%E5%B0%94%E6%96%AF%E5%9F%BA-%E7%BD%97%E6%A3%AE%E4%BD%AF%E8%B0%AC)，并从中提出了“鬼魅似的远距作用”这一词汇。

近一个世纪之后，现代软件开发已经被视为可能成为 Goldbergian 装置的典范——通过量子计算机相信我们会越来越接近这个鬼魅的领域。

作为软件开发人员，我们提倡尽可能减少代码中的远程操作。这是根据一些众所周知的规范法则得出的，如[单一职责原则](https://zh.wikipedia.org/wiki/%E5%8D%95%E4%B8%80%E5%8A%9F%E8%83%BD%E5%8E%9F%E5%88%99)、[最少意外原则](https://en.wikipedia.org/wiki/Principle_of_least_astonishment) 和[得墨忒耳定律](https://zh.wikipedia.org/wiki/%E5%BE%97%E5%A2%A8%E5%BF%92%E8%80%B3%E5%AE%9A%E5%BE%8B)。尽管它们可能会对代码产生一定的副作用，但更多的时候这些原则能使代码逻辑变得清晰。

这是本周文章的焦点 Swift 属性观察器，它是系统内置的，比模型 - 视图 - 视图模型（MVVM）、函数响应式编程（FRP）这些更正式的解决方案更轻量。

---

Swift 中有两种属性：<dfn>存储属性</dfn>，它们将状态和对象相关联；<dfn>计算属性</dfn>，则根据该状态执行计算。例如，

```swift
struct S {
    // 存储属性
    var stored: String = "stored"

    // 计算属性
    var computed: String {
        return "computed"
    }
}
```

当你声明一个存储属性，你可以使用闭包定义一个 <dfn>属性观察器</dfn>，该闭包中的代码会在属性被设值的时候执行。`willSet` 观察器会在属性被赋新值之前被运行，`didSet` 观察器则会在属性被赋新值之后运行。无论新值是否等于属性的旧值它们都会被执行。

```swift
struct S {
    var stored: String {
        willSet {
            print("willSet was called")
            print("stored is now equal to \(self.stored)")
            print("stored will be set to \(newValue)")
        }

        didSet {
            print("didSet was called")
            print("stored is now equal to \(self.stored)")
            print("stored was previously set to \(oldValue)")
        }
    }
}
```

例如，运行下面的代码在控制台的输出如下：

```swift
var s = S(stored: "first")
s.stored = "second"
```

- <samp>willSet was called</samp>
- <samp>stored is now equal to first</samp>
- <samp>stored will be set to second</samp>
- <samp>didSet was called</samp>
- <samp>stored is now equal to second</samp>
- <samp>stored was previously set to first</samp>

{% warning do %}

需要注意的是当属性在初始化方法中进行赋值时，不会触发观察器的代码。从 Swift4.2 开始，你可以将赋值逻辑包装在 `defer` 代码块来解决这个问题，但这是[一个很快就会被修复的问题](https://twitter.com/jckarter/status/926459181661536256)，因此你不应该依赖于这种行为。

{% endwarning %}

---

Swift 的属性观察器从一开始就是语言的一部分。为了更好地理解其原理，让我们快速了解一下它在 Objective-C 中的工作原理。

## Properties in Objective-C

## Objective-C 中的属性

从某种意义上说，Objective-C 中的所有属性都是被计算出来的。每次通过点语法访问属性时，都会转换为等效的 getter 或 setter 方法调用。这些调用最终被编译成消息发送，随后再执行读取或写入实例变量的方法。

```objc
// 点语法访问
person.name = @"Johnny";

// ...等价于
[person setName:@"Johnny"];

// ...它被编译成
objc_msgSend(person, @selector(setName:), @"Johnny");

// ...最终实现
person->_name = @"Johnny";
```

编程过程中我们通常想要避免引入副作用，因为它会导致程序的行为难以推断。但很多 Objective-C 开发者已经依赖于这种特性，他们会根据需要在 getter 或 setter 中注入各种额外的行为。

Swift 的属性设计使这些模式更加标准化，并对装饰状态访问（存储属性）的副作用和重定向状态访问（计算属性）的副作用进行了区分。对于存储属性，`willSet` 和 `didSet` 观察器将替换你在 ivar 访问时的代码。对于计算属性，`get` 和 `set` 访问器可能会替换在 Objective-C 中实现的一些 `@dynamic` 属性。

正因为如此，我们才可以获取更一致的语义，并更好地保证键值观察（KVO）和键值编码（KVC）等属性交互机制。

---

那么你可以使用 Swift 属性观察器做些什么呢？以下是一些供你参考的想法：

---

## Validating / Normalizing Values

## 标准化或验证值

有时，你希望对类型接受的值增加额外的约束。

例如，你正在开发一个和政府机构对接的应用程序，你需要保证用户填写了所有的必填项并且不包含非法的值才能提交表单。

如果一个表单要求名称字段使用大写字母且不使用重音符号，你可以使用 `didSet` 属性观察器自动去除重音符号并转化为大写。

```swift
var name: String? {
    didSet {
        self.name = self.name?
                        .applyingTransform(.stripDiacritics,
                                            reverse: false)?
                        .uppercased()
    }
}
```

幸运的是在观察器内部设置属性不会触发额外的回调，所以上面的代码中不会产生无限循环。我们之所以不使用 `willSet` 观察器是因为即使我们在其回调中进行任何赋值，都会在属性被赋予 `newValue` 时覆盖。

虽然这种方法可以解决一次性问题，但像这样需要重复使用的业务逻辑可以封装到一个类型中。

更好的设计是创建一个 `NormalizedText` 类型，它封装了要以这种形式输入的文本的规则：

```swift
struct NormalizedText {
    enum Error: Swift.Error {
        case empty
        case excessiveLength
        case unsupportedCharacters
    }

    static let maximumLength = 32

    private(set) var value: String

    init(_ string: String) throws {
        if string.isEmpty {
            throw Error.empty
        }

        guard let value = string.applyingTransform(.stripDiacritics,
                                                   reverse: false)?
                                .uppercased(),
              value.canBeConverted(to: .ascii)
        else {
             throw Error.unsupportedCharacters
        }

        guard value.count < NormalizedText.maximumLength else {
            throw Error.excessiveLength
        }

        self.value = value
    }
}
```

一个可抛出异常的初始化方法可以向调用者发送错误信息，这是 `didSet` 观察器无法做到的。现在面对[兰韦尔普尔古因吉尔戈格里惠尔恩德罗布尔兰蒂西利奥戈戈戈赫](https://zh.wikipedia.org/wiki/%E5%85%B0%E9%9F%A6%E5%B0%94%E6%99%AE%E5%B0%94%E5%8F%A4%E5%9B%A0%E5%90%89%E5%B0%94%E6%88%88%E6%A0%BC%E9%87%8C%E6%83%A0%E5%B0%94%E6%81%A9%E5%BE%B7%E7%BD%97%E5%B8%83%E5%B0%94%E5%85%B0%E8%92%82%E8%A5%BF%E5%88%A9%E5%A5%A5%E6%88%88%E6%88%88%E6%88%88%E8%B5%AB) 的 *约翰尼* 这样的麻烦制造者，我们能为他做些什么！（换言之，以合理的方式传达错误比提供无效的数据更好）


## Propagating Dependent State

## 传播依赖状态

属性观察器的另一个潜在用例是将状态传播到依赖于视图控制器的组件。

考虑下面的 `Track` 模型示例和一个呈现它的 `TrackViewController`：

```swift
struct Track {
    var title: String
    var audioURL: URL
}

class TrackViewController: UIViewController {
    var player: AVPlayer?

    var track: Track? {
        willSet {
            self.player?.pause()
        }

        didSet {
            guard let track = self.track else {
                return
            }

            self.title = track.title

            let item = AVPlayerItem(url: track.audioURL)
            self.player = AVPlayer(playerItem: item)
            self.player?.play()
        }
    }
}
```

当视图控制器的 `track` 属性被赋值，以下事情会自动发生：

1. 之前轨道的音频都会暂停
2. 视图控制器的 `title` 会被设置为新轨道对象的标题
3. 新轨道对象的音频信息会被加载并播放

_很酷, 对吗?_

你甚至可以像[*捕鼠记* 中描绘的场景](https://www.youtube.com/watch?v=TVAhhVrpkwM) 一样，将行为与多个观察属性级联起来。

---

当然，观察器也存在一定的副作用，它使得有些复杂的行为难以被推断，这是我们在编程中需要避免的。今后在使用这一特性的同时也需要注意这一点。

然而，在这摇摇欲坠的抽象塔的顶端，一定限度的系统混乱是诱人的，有时是值得的。一直遵循规则的是波尔理论而非爱因斯坦。



