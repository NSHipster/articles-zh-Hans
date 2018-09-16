---
title: guard & defer
author: Mattt & Nate Cook
authors:
    - Nate Cook
    - Mattt
translator: 
    - Croath Liu
    - Bei Li
category: Swift
excerpt: >
    Swift 2.0 带来了两个新的能够简化程序和提高效率的控制流表达形式。前者可以让代码编写更流畅，后者则相反的能够让执行推迟。
revisions:
    "2015-10-05": 首次发布
    "2018-08-01": 为 Swift 4.2 更新
status:
    swift: 4.2
    reviewed: August 1, 2018
---

> 「我们应该（聪明的程序员明白自己的局限性）尽力……让文本里的程序（program）和时间轴上的进程（process）的对应尽量简单。」
>
> —[Edsger W. Dijkstra](https://en.wikipedia.org/wiki/Edsger_W._Dijkstra),
> [《Go To 有害论》](https://homepages.cwi.nl/~storm/teaching/reader/Dijkstra68.pdf)

很遗憾，他的文章通常只因为使《\_\_\_\_有害论》这种文章标题在程序员中流行起来，还有网上对这些论文不妥当的抨击出现时，才会被想起。因为 Dijkstra（照常）提出了一个很好的观点：**代码结构应该反映其行为。**

Swift 2.0 带来了两个新的能够简化程序和提高效率的控制流表达形式：`guard` 和 `defer`。前者可以让代码编写更流畅，后者能够让执行推迟。

我们应该如何使用这两个新的声明方式呢？`guard` 和 `defer` 将如何帮我们厘清程序和进程间的对应关系呢？

我们 defer（推迟）一下 `defer` 先看 `guard`。

---

## guard

`guard` 是一个要求表达式的值为 `true` 从而继续执行的条件语句。如果表达式为 `false`，则会执行必须提供的 `else` 分支。

```swift
func sayHello(numberOfTimes: Int) {
    guard numberOfTimes > 0 else {
        return
    }

    for _ in 1...numberOfTimes {
        print("Hello!")
    }
}
```

`guard` 语句中的 `else` 分支必须退出当前的区域，通过使用 `return` 来退出函数，`continue` 或者 `break` 来退出循环，或者使用像 `fatalError(_:file:line:)` 这种返回 [`Never`](https://nshipster.com/never) 的函数。

`guard` 语句和 optional 绑定组合在一起非常好用。在 `guard` 语句的条件里进行的 optional 绑定可以在函数或闭包其后的部分使用。

对比一下 `guard-let` 语句和 `if-let` 语句中的 optional 绑定：

```swift
var name: String?

if let name = name {
    // name 在这里面不是 optional（类型是 String）
}
// name 在外面是 optional（类型是 String?）


guard let name = name else {
    return
}

// name 从这里开始都不是 optional 了（类型是 String）
```

如果说在 [Swift 1.2](/swift-1.2/) 中介绍的并行 optional 绑定领导了对 [厄运金字塔](http://www.scottlogic.com/blog/2014/12/08/swift-optional-pyramids-of-doom.html) 的革命，那么 `guard` 声明则与之一并将金字塔摧毁。

```swift
for imageName in imageNamesList {
    guard let image = UIImage(named: imageName)
        else { continue }

    // do something with image
}
```

### 使用 guard 来避免过多的缩进和错误

我们来对比一下使用 `guard` 关键字之后能如何改善代码且帮助我们避免错误。

比如，我们要实现一个 `readBedtimeStory()` 函数：

```swift
enum StoryError: Error {
    case missing
    case illegible
    case tooScary
}

func readBedtimeStory() throws {
    if let url = Bundle.main.url(forResource: "book",
                               withExtension: "txt")
    {
        if let data = try? Data(contentsOf: url),
            let story = String(data: data, encoding: .utf8)
        {
            if story.contains("👹") {
                throw StoryError.tooScary
            } else {
                print("Once upon a time... \(story)")
            }
        } else {
            throw StoryError.illegible
        }
    } else {
        throw StoryError.missing
    }
}
```

要读一个睡前故事，我们需要能找到一本书，这本故事书必须要是可读的，并且故事不能太吓人（**请不要让怪物出现在书的结尾，谢谢你！**）。

请注意 `throw` 语句离检查本身有多远。你需要读完整个方法来找到如果没有 `book.txt` 会发生什么。

像一本好书一样，代码应该讲述一个故事：有着易懂的情节，清晰的开端、发展和结尾。（请尝试不要写太多「后现代」风格的代码。）

使用 `guard` 语句组织代码可以让代码读起来更加的线性：

```swift
func readBedtimeStory() throws {
    guard let url = Bundle.main.url(forResource: "book",
                                  withExtension: "txt")
    else {
        throw StoryError.missing
    }

    guard let data = try? Data(contentsOf: url),
        let story = String(data: data, encoding: .utf8)
    else {
        throw StoryError.illegible
    }

    if story.contains("👹") {
        throw StoryError.tooScary
    }

    print("Once upon a time ...\(story)")
}
```

**这样就好多了！** 每一个错误都在相应的检查之后立刻被抛出，所以我们可以按照左手边的代码顺序来梳理工作流的顺序。

### 不要在 guard 中双重否定

不要滥用这个新的流程控制机制——特别是在条件表达式已经表示否定的情况下。

举个例子，如果你想要在一个字符串为空是提早退出，不要这样写：

```swift
// 啊？
guard !string.isEmpty else {
    return
}
```

保持简单。自然的走下去。避免双重否定。

```swift
// 噢！
if string.isEmtpy {
    return
}
```

## defer

在错误处理方面，`guard` 和新的 `throw` 语法之间，Swift 鼓励用尽早返回错误（这也是 NSHipster 最喜欢的方式）来代替嵌套 if 的处理方式。尽早返回让处理更清晰了，但是已经被初始化（可能也正在被使用）的资源必须在返回前被处理干净。

`defer` 关键字为此提供了安全又简单的处理方式：声明一个 block，当前代码执行的闭包退出时会执行该 block。

看看下面这个包装了系统调用 `gethostname(2)` 的函数，用来返回当前系统的[主机名称](https://zh.wikipedia.org/zh-cn/主機名稱)：

```swift
import Darwin

func currentHostName() -> String {
    let capacity = Int(NI_MAXHOST)
    let buffer = UnsafeMutablePointer<Int8>.allocate(capacity: capacity)

    guard gethostname(buffer, capacity) == 0 else {
        buffer.deallocate()
        return "localhost"
    }

    let hostname = String(cString: buffer)
    buffer.deallocate()

    return hostname
}
```

这里有一个在最开始就创建的 `UnsafeMutablePointer<UInt8>` 用于存储目标数据，但是我**既要**在错误发生后销毁它，**又要**在正常流程下不再使用它时对其进行销毁。

这种设计很容易导致错误，而且不停地在做重复工作。

通过使用 `defer` 语句，我们可以排除潜在的错误并且简化代码：

```swift
func currentHostName() -> String {
    let capacity = Int(NI_MAXHOST)
    let buffer = UnsafeMutablePointer<Int8>.allocate(capacity: capacity)
    defer { buffer.deallocate() }

    guard gethostname(buffer, capacity) == 0 else {
        return "localhost"
    }

    return String(cString: buffer)
}
```

尽管 `defer` 紧接着出现在 `allocate(capacity:)` 调用之后，但它要等到当前区域结束时才会被执行。多亏了 `defer`，`buffer` 才能无论在哪个点退出函数都可以被释放。

考虑在任何需要配对调用的 API 上都使用 `defer`，比如 `allocate(capacity:)` / `deallocate()`、`wait()` / `signal()` 和 `open()` / `close()`。这样的话，你不仅可以消除一种程序员易犯的错误，还能让 Dijkstra 自豪地用它的母语德语说：「Goed gedaan!」。

### 经常 defer

如果在同一个作用域内使用多个 `defer` 语句，它们会根据出现顺序反过来执行——像栈一样。这个反序是非常重要的细节，保证了被延迟的代码块创建时作用域内存在的东西，在代码块执行同样存在。

举个例子，执行这段代码会得到下面的输出：

```swift
func procrastinate() {
    defer { print("wash the dishes") }
    defer { print("take out the recycling") }
    defer { print("clean the refrigerator") }

    print("play videogames")
}
```

<samp>
play videogames<br/>
clean the refrigerator<br/>
take out the recycling<br/>
wash the dishes<br/>
</samp>

> 如果你像这样嵌套 `defer` 语句，会怎么样？

```swift
defer { defer { print("clean the gutter") } }
```

> 你的第一想法可能是语句会被压入栈的最底部。但并不是这样的。仔细想一想，然后在 Playground 里验证你的猜想。

### 正确 defer

如果在 `defer` 语句中引用了一个变量，执行时会用到变量最终的值。换句话说：`defer` 代码块不会捕获变量当前的值。

如果你运行这段代码，你会得到下面的输出：

```swift
func flipFlop() {
    var position = "It's pronounced /ɡɪf/"
    defer { print(position) }

    position = "It's pronounced /dʒɪf/"
    defer { print(position) }
}
```

<samp>
It's pronounced /dʒɪf/ <br/>
It's pronounced /dʒɪf/
</samp>

### 仔细 defer

另一件需要注意的事情，那就是 `defer` 代码块无法跳出它所在的作用域。因此如你尝试调用一个会 throw 的方法，抛出的错误就无法传递到其周围的上下文。

```swift
func burnAfterReading(file url: URL) throws {
    defer { try FileManager.default.removeItem(at: url) }
    // 🛑 Errors not handled

    let string = try String(contentsOf: url)
}
```

作为替代，你可以使用 `try?` 来无视掉错误，或者直接将语句移出 `defer` 代码块，将其放到函数的最后，正常的执行。

### （其他情况下）Defer 会带来坏处

虽然 `defer` 像一个语法糖一样，但也要小心使用避免形成容易误解、难以阅读的代码。在某些情况下你可能会尝试用 `defer` 来对某些值返回之前做最后一步的处理，例如说在后置运算符 `++` 的实现中：

```swift
postfix func ++(inout x: Int) -> Int {
    let current = x
    x += 1
    return current
}
```

在这种情况下，可以用 `defer` 来进行一个很另类的操作。如果能在 defer 中处理的话为什么要创建临时变量呢？ 

```swift
postfix func ++(inout x: Int) -> Int {
    defer { x += 1 }
    return x
}
```

这种写法确实聪明，但这样却颠倒了函数的逻辑顺序，极大降低了代码的可读性。应该严格遵循 `defer` 在整个程序最后运行以释放已申请资源的原则，其他任何使用方法都可能让代码乱成一团。

---

「聪明的程序员明白自己的局限性」，我们必须权衡每种语言特性的好处和其成本。

类似于 `guard` 的新特性能让代码流程上更线性，可读性更高，就应该尽可能使用。

同样 `defer` 也解决了重要的问题，但是会强迫我们一定要找到它声明的地方才能追踪到其销毁的方法，因为声明方法很容易被滚动出了视野之外，所以应该尽可能遵循它出现的初衷尽可能少地使用，避免造成混淆和晦涩。
