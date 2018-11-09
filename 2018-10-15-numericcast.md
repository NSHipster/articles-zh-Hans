---
title: numericCast(_:)
author: Mattt
translator: Bei Li
category: Swift
excerpt: >
  编译通过的代码和正确的代码是不一样的。但有时候需要通过前者来最终获得后者。
status:
  swift: 4.2
---

每个人都曾将编程比喻成其他事物。

类比成木工、编织或者园艺。又或者可能类比成解决问题、讲故事或者制作艺术品。毫无疑问，编程与写作也很像；问题是更像诗歌还是散文。如果编程像音乐的话，不管怎么样它都应该是爵士乐。

或许对我们每天所做工作最近似的类比来自中东民间故事：打开任何版本的《一千零一夜 (<span lang="ar">أَلْف لَيْلَة وَلَيْلَة</span>) 》，你会找到对一种被称作<dfn>镇尼</dfn>、<dfn>杰尼</dfn>、<dfn>精灵</dfn>或者 🧞 的神奇生物的描述。不管你怎么称呼它们，你一定熟悉它们实现愿望的习惯，和必然会引起的不幸。

从许多方面来看，电脑是抽象的愿望满足机的物理体现。像精灵一样，电脑会开心的执行任何你告诉它要做的事，而不会考虑你真正的意图是什么。之后当你意识到自己的错误时，就已经太晚了。

作为一个 Swift 开发者，很有可能你遇到过整数类型转换错误并想着「我希望这些警告赶紧消失，代码能编译通过」。

如果这听起来很熟悉，那你会对学习 `numbericCast(_:)` 感到高兴，它是 Swift Standard Libray 中一个小小的实用函数，有可能正是你所希望的。但是请小心提出你的愿望，它有可能马上会成真。

---

让我们从消除觉得 `numericCast(_:)` 有什么魔法开始，通过[查看它的实现](https://github.com/apple/swift/blob/7f7b4f12d3138c5c259547c49c3b41415cd4206e/stdlib/public/core/Integers.swift#L3508-L3510)：

```swift
public func numericCast<T : BinaryInteger, U : BinaryInteger>(_ x: T) -> U {
  return U(x)
}
```

（像从[我们有关 `Never` 的文章](/never)里学到的一样，极小量的 Swift 代码也能有巨大的作用。） 

Swift 4 推出的 [`BinaryInteger`](https://developer.apple.com/documentation/swift/binaryinteger) 协议，作为语言中整个数字实现的一部分。它提供了与整数工作的统一接口，包括有符号和无符号，还有所有的结构和大小。

当你将一个整数值转换为另一个类型时，另一个类型有可能无法表示这个值。这会发生在你尝试将一个有符号整数转换成一个无符号整数时（比如将 `-42` 转换为 `UInt`）或者数值超过了目标类型所能表示的范围时（比如 `UInt8` 只能表示 `0` 到 `255` 之间的数字）。

`BinaryInteger` 为整数类型转换定义了四种策略，每一种在处理超出范围的值时都有不同行为：

- **范围检查转换**（[`init(_:)`](https://developer.apple.com/documentation/swift/binaryinteger/2885704-init)）：
  遇到超出范围的值时触发运行时错误
- **准确转换**（[`init?(exactly:)`](https://developer.apple.com/documentation/swift/binaryinteger/2925955-init)）：
  遇到超出范围的值时返回 `nil`
- **钳制转换**（[`init(clamping:)`](https://developer.apple.com/documentation/swift/binaryinteger/2886143-init)）：
  遇到超出范围的值时使用最近可表示的值
- **位模式转换**（[`init(truncatingIfNeeded:)`](https://developer.apple.com/documentation/swift/binaryinteger/2925529-init)）：
  截断至目标整数类型宽度

正确的转换策略取决于使用时的情况。有些时候，希望能钳制数值到可表示的范围；有些时候，最好不要获取到任何值。对于 `numbericCast(_:)` 来说，它为了方便使用了范围检查转换。缺点就是使用超过范围的数值调用这个函数会导致运行时错误（具体来说，在 `-O` 和 `-Onone` 时陷入溢出错误）。

{% info %}

更多有关 Swift 4 中数字实现改变的信息，请查阅 [SE-0104: "Protocol-oriented integers"](https://github.com/apple/swift-evolution/blob/master/proposals/0104-improved-integers.md)。

这个主题也在[《Swift 数字详解》](https://juejin.im/book/5b260350e51d4558c2322fbe)中有更详细的讨论。

{% endinfo %}

## 字面地思考，批判地思考

在更进一步之前，让我们先来谈论一下整数字面量。

[我们在之前的文章讨论过](https://nshipster.com/swift-literals/)，Swift 提供了一个方便且可扩展的方式来在源代码中表示值。当和语言中的类型推断一起使用时，它们通常「可以工作」……这样一切都很好，但是当它们「无法工作」时就非常令人困惑了。

考虑下面的例子，有符号整型数组和无符号整型数组使用同样的字面量初始化：

```swift
let arrayOfInt: [Int] = [1, 2, 3]
let arrayOfUInt: [UInt] = [1, 2, 3]
```

尽管它们好像是相等的，但我们不能做下面例子中的事情：

```swift
arrayOfInt as [UInt] // Error: Cannot convert value of type '[Int]' to type '[UInt]' in coercion
```

解决这个问题的一种方式是，将 `numericCast` 函数作为参数传入 `map(_:)`：

```swift
arrayOfInt.map(numericCast) as [UInt]
```

这样等同于直接传入 `UInt` 范围检查构造器：

```swift
arrayOfInt.map(UInt.init)
```

让我们再看一次这个例子，这次使用稍微不同的数值：

```swift
let arrayOfNegativeInt: [Int] = [-1, -2, -3]
arrayOfNegativeInt.map(numericCast) as [UInt] // 🧞‍ Fatal error: Negative value is not representable
```

作为一个编译时类型功能的运行时近似物，`numericCast(_:)` 更像是 `as!` 而不是 `as` 或 `as?`。

将这个和传入精确转换构造器 `init?(exactly:)` 的结果相比：

```swift
let arrayOfNegativeInt: [Int] = [-1, -2, -3]
arrayOfNegativeInt.map(UInt.init(exactly:)) // [nil, nil, nil]
```

`numericCast(_:)`，像它内在的范围检查转换一样，是一个钝器，当你决定使用它时，明白你在权衡什么是非常重要的。

## 正确的代价

在 Swift 中，通常指导是为整数值使用 `Int`（且为浮点值使用 `Double`），除非有**非常**好的理由来使用更具体的类型。尽管 `Collection` 的 `count` 在定义上是非负的，但我们使用 `Int` 而不是 `UInt`。因为在与其他 API 交互时转换来转换去类型的代价要比更精确类型带来的好处要大。同样的原因，用 `Int` 来表示小数字几乎总是会更好，比如[工作日数字](https://nshipster.com/datecomponents)，尽管它所有的可能值用一个 8 位整型存储都绰绰有余。

理解这个实践最好的方式就是在 Swift 里和 C API 对话几分钟。

古老且低级的 C API 里充斥着体系结构相关的类型定义和细微调整过的值存储空间。独立的来看，它们是可管理的。但从像头文件到指针这些互操作性麻烦上看，它们对某些问题可能会是一个断点（我不是在说调试中那种）。

当你看红色看到烦，只想要编译通过时，`numericCast(_:)` 就在那等着你。

## 编译的随机性

很多人应该会熟悉[官方文档中的例子](https://developer.apple.com/documentation/swift/2884564-numericcast)：

在 [SE-0202](https://github.com/apple/swift-evolution/blob/master/proposals/0202-random-unification.md) 之前，（在苹果的平台上）Swift 中生成随机数的标准实践需要引入 `Darwin` 框架然后调用 `arc4random_uniform(3)` 函数：

```c
uint32_t arc4random_uniform(uint32_t __upper_bound)
```

在 Swift 中使用 `arc4random` 需要进行不止一次而是两次类型转换：一是上限参数（`Int` → `UInt32`），二是返回值（`UInt32` → `Int`）：

```swift
import Darwin

func random(in range: Range<Int>) -> Int {
    return Int(arc4random_uniform(UInt32(range.count))) + range.lowerBound
}
```

**真恶心。**

通过使用 `numericCast(_:)`，我们可以让代码更可读一些，尽管也会变长一点：

```swift
import Darwin

func random(in range: Range<Int>) -> Int {
    return numericCast(arc4random_uniform(numericCast(range.count))) + range.lowerBound
}
```

在这里 `numericCast(_:)` 没有做任何类型合适的构造器做不到的事情。它的作用是指明这个转换是敷衍的——为了让代码编译需要做的最少的事情。

不过从前言有关精灵的事情中学到，我们应该谨慎的对待我们的愿望。

经过仔细检查，上面对例子中对 `numericCast(_:)` 的使用有一个明显的缺陷：**当值超过 `UInt32.max` 时会造成崩溃！**

```swift
random(in: 0..<0x1_0000_0000) // 🧞‍ Fatal error: Not enough bits to represent the passed value
```

如果我们查看现在 `Int.random(in: 0...10)` [在 Swift Standard Library 中的实现](https://github.com/apple/swift/blob/7f7b4f12d3138c5c259547c49c3b41415cd4206e/stdlib/public/core/Integers.swift#L2537-L2560)，可以看到其使用了钳制转换而不是类型检查转换。并且[从一个随机字节缓冲区中取值](https://github.com/apple/swift/blob/7f7b4f12d3138c5c259547c49c3b41415cd4206e/stdlib/public/core/Random.swift#L156-L177)而不是委托给像 `arc4random_uniform` 这样的简便函数。

---

编译通过的代码和正确的代码是不一样的。但有时候需要通过前者来最终获得后者。审慎的使用，`numericCast(_:)` 会是一个方便且能快速解决问题的工具。和类型转换构造器相比它还有表明潜在异常行为的好处。

根本上来说，编程就是**准确**描述我们想要怎么样——通常伴随艰苦的细节。并没有一个和精灵似的「做正确的事情」 CPU 指令（就算有的话，[我们能信赖它吗](ttps://github.com/FixIssue/FixCode)？）。幸好，Swift 可以让我们比其他很多语言更安全和简洁的做这些事情。老实说，谁还能要求更多呢？
