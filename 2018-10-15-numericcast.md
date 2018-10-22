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

Everyone has their favorite analogy to describe programming.

每个人都曾将编程比喻成其他事物。

It's woodworking or it's knitting or it's gardening.
Or maybe it's problem solving and storytelling and making art.
That programming is like writing there is no doubt;
the question is whether it's poetry or prose.
And if programming is like music,
it's always jazz for whatever reason.

类比成木工、编织或者园艺。又或者可能类比成解决问题、讲故事或者制作艺术品。毫无疑问，编程与写作也很像；问题是更像诗歌还是散文。如果编程像音乐的话，不管怎么样它都应该是爵士乐。

But perhaps the closest point of comparison for what we do all day
comes from Middle Eastern folk tales:
Open any edition of
_The Thousand and One Nights_ (أَلْف لَيْلَة وَلَيْلَة‎)
and you'll find descriptions of supernatural beings known as
<dfn>jinn</dfn>, <dfn>djinn</dfn>, <dfn>genies</dfn>, or 🧞‍.
No matter what you call them,
you're certainly familiar with their habit of granting wishes,
and the misfortune that inevitably causes.

或许对我们每天所做工作最近似的类比来自中东民间故事：打开任何版本的《一千零一夜（أَلْف لَيْلَة وَلَيْلَة）》，你会找到对一种被称作<dfn>镇尼</dfn>、<dfn>杰尼</dfn>、<dfn>精灵</dfn>或者 🧞‍ 的神奇生物的描述。不管你怎么称呼它们，你一定熟悉它们实现愿望的习惯，和必然会引起的不幸。

In many ways,
computers are the physical embodiment of metaphysical wish fulfillment.
Like a genie, a computer will happily go along with whatever you tell it to do,
with no regard for what your actual intent may have been.
And by the time you've realized your error,
it may be too late to do anything about it.

从许多方面来看，电脑是抽象的愿望满足机的物理体现。像精灵一样，电脑会开心的执行任何你告诉它要做的事，而不会考虑你真正的意图是什么。之后当你意识到自己的错误时，就已经太晚了。

As a Swift developer,
there's a good chance that you've been hit by integer type conversion errors
and thought
"I wish these warnings would go away and my code would finally compile."

作为一个 Swift 开发者，很有可能你遇到过整数类型转换错误并想着「我希望这些警告赶紧消失，代码能编译通过」。

If that sounds familiar,
you'll happy to learn about `numericCast(_:)`,
a small utility function in the Swift Standard Library
that may be exactly what you were hoping for.
But be careful what you wish for,
it might just come true.

如果这听起来很熟悉，那你会对学习 `numbericCast(_:)` 感到高兴，它是 Swift Standard Libray 中一个小小的实用函数，有可能正是你所希望的。但是请小心提出你的愿望，它有可能马上会成真。

---

Let's start by dispelling any magical thinking
about what `numericCast(_:)` does by
[looking at its implementation](https://github.com/apple/swift/blob/7f7b4f12d3138c5c259547c49c3b41415cd4206e/stdlib/public/core/Integers.swift#L3508-L3510):

让我们从消除觉得 `numericCast(_:)` 有什么魔法开始，通过[查看它的实现](https://github.com/apple/swift/blob/7f7b4f12d3138c5c259547c49c3b41415cd4206e/stdlib/public/core/Integers.swift#L3508-L3510)：

```swift
public func numericCast<T : BinaryInteger, U : BinaryInteger>(_ x: T) -> U {
  return U(x)
}
```

(As we learned in [our article about `Never`](/never),
even the smallest amount of Swift code can have a big impact.)

（像从[我们有关 `Never` 的文章](/never)里学到的一样，极小量的 Swift 代码也能有巨大的作用。） 

The [`BinaryInteger`](https://developer.apple.com/documentation/swift/binaryinteger) protocol
was introduced in Swift 4
as part of an overhaul to how numbers work in the language.
It provides a unified interface for working with integers,
both signed and unsigned, and of all shapes and sizes.

Swift 4 推出的 [`BinaryInteger`](https://developer.apple.com/documentation/swift/binaryinteger) 协议，作为语言中整个数字实现的一部分。它提供了与整数工作的统一接口，包括有符号和无符号，还有所有的结构和大小。

When you convert an integer value to another type,
it's possible that the value can't be represented by that type.
This happens when you try to convert a signed integer
to an unsigned integer (for example, `-42` as a `UInt`),
or when a value exceeds the representable range of the destination type
(for example, `UInt8` can only represent numbers between `0` and `255`).

当你将一个整数值转换为另一个类型时，另一个类型有可能无法表示这个值。这会发生在你尝试将一个有符号整数转换成一个无符号整数时（比如将 `-42` 转换为 `UInt`）或者数值超过了目标类型所能表示的范围时（比如 `UInt8` 只能表示 `0` 到 `255` 之间的数字）。

`BinaryInteger` defines four strategies of conversion between integer types,
each with different behaviors for handling out-of-range values:

`BinaryInteger` 为整数类型转换定义了四种策略，每一种在处理超出范围的值时都有不同行为：

- **Range-Checked Conversion**
  ([`init(_:)`](https://developer.apple.com/documentation/swift/binaryinteger/2885704-init)):
  Trigger a runtime error for out-of-range values
- **范围检查转换**（[`init(_:)`](https://developer.apple.com/documentation/swift/binaryinteger/2885704-init)）：
  遇到超出范围的值时触发运行时错误
- **Exact Conversion**
  ([`init?(exactly:)`](https://developer.apple.com/documentation/swift/binaryinteger/2925955-init)):
  Return `nil` for out-of-range values
- **准确转换**（[`init?(exactly:)`](https://developer.apple.com/documentation/swift/binaryinteger/2925955-init)）：
  遇到超出范围的值时返回 `nil`
- **Clamping Conversion**
  ([`init(clamping:)`](https://developer.apple.com/documentation/swift/binaryinteger/2886143-init)):
  Use the closest representable value for out-of-range values
- **钳制转换**（[`init(clamping:)`](https://developer.apple.com/documentation/swift/binaryinteger/2886143-init)）：
  遇到超出范围的值时使用最近可表示的值
- **Bit Pattern Conversion**
  ([`init(truncatingIfNeeded:)`](https://developer.apple.com/documentation/swift/binaryinteger/2925529-init)):
  Truncate to the width of the target integer type
- **位模式转换**（[`init(truncatingIfNeeded:)`](https://developer.apple.com/documentation/swift/binaryinteger/2925529-init)）：
  截断至目标整数类型宽度

The correct conversion strategy
depends on the situation in which it's being used.
Sometimes it's desireable to clamp values to a representable range;
other times, it's better to get no value at all.
In the case of `numericCast(_:)`,
range-checked conversion is used for convenience.
The downside is that
calling this function with out-of-range values
causes a runtime error
(specifically, it traps on overflow in `-O` and `-Onone`).

正确的转换策略取决于使用时的情况。有些时候，希望能钳制数值到可表示的范围；有些时候，最好不要获取到任何值。对于 `numbericCast(_:)` 来说，它为了方便使用了范围检查转换。缺点就是使用超过范围的数值调用这个函数会导致运行时错误（具体来说，在 `-O` 和 `-Onone` 时陷入溢出错误）。

{% info %}

For more information about the changes to how numbers work in Swift 4,
see [SE-0104: "Protocol-oriented integers"](https://github.com/apple/swift-evolution/blob/master/proposals/0104-improved-integers.md).

更多有关 Swift 4 中数字实现改变的信息，请查阅 [SE-0104: "Protocol-oriented integers"](https://github.com/apple/swift-evolution/blob/master/proposals/0104-improved-integers.md)。

This subject is also discussed at length in the
[Flight School Guide to Numbers](https://gumroad.com/l/swift-numbers).

这个主题也在[《Swift 数字详解》](https://juejin.im/book/5b260350e51d4558c2322fbe)中有更详细的讨论。

{% endinfo %}

## Thinking Literally, Thinking Critically
## 字面地思考，批判地思考

Before we go any further,
let's take a moment to talk about integer literals.

在更进一步之前，让我们先来谈论一下整数字面量。

[As we've discussed in previous articles](https://nshipster.com/swift-literals/),
Swift provides a convenient and extensible way to represent values in source code.
When used in combination with the language's use of type inference,
things often "just work"
...which is nice and all, but can be confusing when things "just don't".

[我们在之前的文章讨论过](https://nshipster.com/swift-literals/)，Swift 提供了一个方便且可扩展的方式来在源代码中表示值。当和语言中的类型推断一起使用时，它们通常「可以工作」……这样一切都很好，但是当它们「无法工作」时就非常令人困惑了。

Consider the following example
in which arrays of signed and unsigned integers
are initialized from identical literal values:

考虑下面的例子，有符号整型数组和无符号整型数组使用同样的字面量初始化：

```swift
let arrayOfInt: [Int] = [1, 2, 3]
let arrayOfUInt: [UInt] = [1, 2, 3]
```

Despite their seeming equivalence,
we can't, for example, do this:

尽管它们好像是相等的，但我们不能做下面例子中的事情：

```swift
arrayOfInt as [UInt] // Error: Cannot convert value of type '[Int]' to type '[UInt]' in coercion
```

One way to reconcile this issue
would be to pass the `numericCast` function as an argument to `map(_:)`:

解决这个问题的一种方式是，将 `numericCast` 函数作为参数传入 `map(_:)`：

```swift
arrayOfInt.map(numericCast) as [UInt]
```

This is equivalent to passing the `UInt` range-checked initializer directly:

这样等同于直接传入 `UInt` 范围检查构造器：

```swift
arrayOfInt.map(UInt.init)
```

But let's take another look at that example,
this time using slightly different values:

让我们再看一次这个例子，这次使用稍微不同的数值：

```swift
let arrayOfNegativeInt: [Int] = [-1, -2, -3]
arrayOfNegativeInt.map(numericCast) as [UInt] // 🧞‍ Fatal error: Negative value is not representable
```

As a run-time approximation of compile-time type functionality
`numericCast(_:)` is closer to `as!` than `as` or `as?`.

作为一个编译时类型功能的运行时近似物，`numericCast(_:)` 更像是 `as!` 而不是 `as` 或 `as?`。

Compare this to what happens if you instead pass
the exact conversion initializer, `init?(exactly:)`:

将这个和传入精确转换构造器 `init?(exactly:)` 的结果相比：

```swift
let arrayOfNegativeInt: [Int] = [-1, -2, -3]
arrayOfNegativeInt.map(UInt.init(exactly:)) // [nil, nil, nil]
```

`numericCast(_:)`, like its underlying range-checked conversion,
is a blunt instrument,
and it's important to understand what tradeoffs you're making
when you decide to use it.

`numericCast(_:)`，像它内在的范围检查转换一样，是一个钝器，当你决定使用它时，明白你在权衡什么是非常重要的。

## The Cost of Being Right
## 正确的代价

In Swift,
the general guidance is to use `Int` for integer values
(and `Double` for floating-point values)
unless there's a _really_ good reason to use a more specific type.
Even though the `count` of a `Collection` is nonnegative by definition,
we use `Int` instead of `UInt`
because the cost of going back and forth between types
when interacting with other APIs
outweighs the potential benefit of a more precise type.
For the same reason,
it's almost always better to represent even small numbers,
like [weekday numbers](/datecomponents),
with an `Int`,
despite the fact that any possible value would fit into an 8-bit integer
with plenty of room to spare.

在 Swift 中，通常指导是为整数值使用 `Int`（且为浮点值使用 `Double`），除非有**非常**好的理由来使用更具体的类型。尽管 `Collection` 的 `count` 在定义上是非负的，但我们使用 `Int` 而不是 `UInt`。因为在与其他 API 交互时转换来转换去类型的代价要比更精确类型带来的好处要大。同样的原因，用 `Int` 来表示小数字几乎总是会更好，比如[工作日数字](https://nshipster.com/datecomponents)，尽管它所有的可能值用一个 8 位整型存储都绰绰有余。

The best argument for this practice
is a 5-minute conversation with a C API from Swift.

理解这个实践最好的方式就是在 Swift 里和 C API 对话几分钟。

Older and lower-level C APIs are rife with
architecture-dependent type definitions
and finely-tuned value storage.
On their own, they're manageable.
But on top of all the other inter-operability woes
like headers to pointers,
they can be a breaking point for some
(and I don't mean the debugging kind).

古老且低级的 C API 里充斥着体系结构相关的类型定义和细微调整过的值存储空间。独立的来看，它们是可管理的。但从像头文件到指针这些互操作性麻烦上看，它们对某些问题可能会是一个断点（我不是在说调试中那种）。

`numericCast(_:)` is there for when you're tired of seeing red
and just want to get things to compile.

当你看红色看到烦，只想要编译通过时，`numericCast(_:)` 就在那等着你。

## Random Acts of Compiling
## 编译的随机性

The [example in the official docs](https://developer.apple.com/documentation/swift/2884564-numericcast)
should be familiar to many of us:

很多人应该会熟悉[官方文档中的例子](https://developer.apple.com/documentation/swift/2884564-numericcast)：

Prior to [SE-0202](https://github.com/apple/swift-evolution/blob/master/proposals/0202-random-unification.md),
the standard practice for generating numbers in Swift (on Apple platforms)
involved importing the `Darwin` framework
and calling the `arc4random_uniform(3)` function:

在 [SE-0202](https://github.com/apple/swift-evolution/blob/master/proposals/0202-random-unification.md) 之前，（在苹果的平台上）Swift 中生成随机数的标准实践需要引入 `Darwin` 框架然后调用 `arc4random_uniform(3)` 函数：

```c
uint32_t arc4random_uniform(uint32_t __upper_bound)
```

`arc4random` requires not one but two separate type conversions in Swift:
first for the upper bound parameter (`Int` → `UInt32`)
and second for the return value (`UInt32` → `Int`):

在 Swift 中使用 `arc4random` 需要进行不止一次而是两次类型转换：一是上限参数（`Int` → `UInt32`），二是返回值（`UInt32` → `Int`）：

```swift
import Darwin

func random(in range: Range<Int>) -> Int {
    return Int(arc4random_uniform(UInt32(range.count))) + range.lowerBound
}
```

_Gross._

**真恶心。**

By using `numericCast(_:)`, we can make things a little more readable,
albeit longer:

通过使用 `numericCast(_:)`，我们可以让代码更可读一些，尽管也会变长一点：

```swift
import Darwin

func random(in range: Range<Int>) -> Int {
    return numericCast(arc4random_uniform(numericCast(range.count))) + range.lowerBound
}
```

`numericCast(_:)` isn't doing anything here
that couldn't otherwise be accomplished with type-appropriate initializers.
Instead, it serves as an indicator
that the conversion is perfunctory ---
the minimum of what's necessary to get the code to compile.

在这里 `numericCast(_:)` 没有做任何类型合适的构造器做不到的事情。它的作用是指明这个转换是敷衍的——为了让代码编译需要做的最少的事情。

But as we've learned from our run-ins with genies,
we should be careful what we wish for.

不过从前言有关精灵的事情中学到，我们应该谨慎的对待我们的愿望。

Upon closer inspection,
it's apparent that the example usage of `numericCast(_:)` has a critical flaw:
_it traps on values that exceed `UInt32.max`!_

经过仔细检查，上面对例子中对 `numericCast(_:)` 的使用有一个明显的缺陷：**当值超过 `UInt32.max` 时会造成崩溃！**

```swift
random(in: 0..<0x1_0000_0000) // 🧞‍ Fatal error: Not enough bits to represent the passed value
```

If we [look at the Standard Library implementation](https://github.com/apple/swift/blob/7f7b4f12d3138c5c259547c49c3b41415cd4206e/stdlib/public/core/Integers.swift#L2537-L2560)
that now lets us do `Int.random(in: 0...10)`,
we'll see that it uses clamping, rather than range-checked, conversion.
And instead of delegating to a convenience function like `arc4random_uniform`,
it [populates values from a buffer of random bytes](https://github.com/apple/swift/blob/7f7b4f12d3138c5c259547c49c3b41415cd4206e/stdlib/public/core/Random.swift#L156-L177).

如果我们查看现在 `Int.random(in: 0...10)` [在 Swift Standard Library 中的实现](https://github.com/apple/swift/blob/7f7b4f12d3138c5c259547c49c3b41415cd4206e/stdlib/public/core/Integers.swift#L2537-L2560)，可以看到其使用了钳制转换而不是类型检查转换。并且[从一个随机字节缓冲区中取值]（https://github.com/apple/swift/blob/7f7b4f12d3138c5c259547c49c3b41415cd4206e/stdlib/public/core/Random.swift#L156-L177）而不是委托给像 `arc4random_uniform` 这样的简便函数。

---

Getting code to compile is different than doing things correctly.
But sometimes it takes the former to ultimately get to the latter.
When used judiciously,
`numericCast(_:)` is a convenient tool to resolve issues quickly.
It also has the added benefit of
signaling potential misbehavior more clearly than
a conventional type initializer.

编译通过的代码和正确的代码是不一样的。但有时候需要通过前者来最终获得后者。审慎的使用，`numericCast(_:)` 会是一个方便且能快速解决问题的工具。和类型转换构造器相比它还有表明潜在异常行为的好处。

Ultimately, programming is about describing _exactly_ what we want ---
often with painstaking detail.
There's no genie-equivalent CPU instruction for "Do the Right Thing"
(and even if there was,
[would we really trust it](https://github.com/FixIssue/FixCode)?)
Fortunately for us,
Swift allows us to do this in a way that's
safer and more concise than many other languages.
And honestly, who could wish for anything more?

根本上来说，编程就是**准确**描述我们想要怎么样——通常伴随艰苦的细节。并没有一个和精灵似的「做正确的事情」 CPU 指令（就算有的话，[我们能信赖它吗](ttps://github.com/FixIssue/FixCode)？）。幸好，Swift 可以让我们比其他很多语言更安全和简洁的做这些事情。老实说，谁还能要求更多呢？
