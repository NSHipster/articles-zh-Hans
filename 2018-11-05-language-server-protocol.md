---
title: Language Server Protocol
author: Mattt
translator: Xiang Wang
category: Open Source
excerpt: >-
  决定支持 LSP 可能是苹果自 2014 年将 Swift 作为开源软件发布以来，为 Swift 做出的最重要的决定。这对于 APP 开发者来说是一件大事，对于其他平台上的 Swift 开发者来说更是一件大事。
status:
  swift: n/a
---

Last month, Apple
[announced on the Swift.org forums](https://forums.swift.org/t/new-lsp-language-service-supporting-swift-and-c-family-languages-for-any-editor-and-platform/17024)
that it was starting work to adopt
the [Language Server Protocol](https://microsoft.github.io/language-server-protocol/)
(<abbr title="Language Server Protocol">LSP</abbr>)
for Swift and C languages.

上个月，苹果公司 [在 Swift.org 论坛上宣布](https://forums.swift.org/t/new-lsp-language-service-supporting-swift-and-c-family-languages-for-any-editor-and-platform/17024)，正在着手为 Swift 和 C 语言支持 [Language Server Protocol](https://microsoft.github.io/language-server-protocol/)（语言服务器协议，LSP）。

> At Apple we are making it a priority
> to support high-quality tooling for all Swift developers,
> including those working on non-Apple platforms.
> We want to collaborate with the open-source community
> and focus our efforts on building common infrastructure
> that can be shared by Xcode and other editors and platforms.
> To that end, [ ... ] we've chosen to adopt LSP.
>
> <cite>Argyrios Kyrtzidis, October 15<sup>th</sup>, 2018</cite>

> 对于苹果公司而言，为所有 Swift 开发者 —— 包括非苹果平台上的 —— 提供高质量的工具支持非常重要。我们希望与开源社区合作，将精力集中在构建 Xcode 和其他编辑器、其他平台可以共享的公共基础设施上。为实现这一目标，[……]，我们决定支持 LSP。
>
> _Argyrios Kyrtzidis，2018 年 10 月 15 日_

**This is arguably the most important decision Apple has made for Swift
since releasing the language as open source in 2014.**
It's a big deal for app developers,
and it's an even bigger deal for Swift developers on other platforms.

**这可能是苹果自 2014 年将 Swift 作为开源软件发布以来，为 Swift 做出的最重要的决定。** 这对于 APP 开发者来说是一件大事，对于其他平台上的 Swift 开发者来说更是一件大事。

To understand why,
this week's article will take a look at
what problem the Language Server Protocol solves,
how it works,
and what its long-term impacts may be.

为了理解其中的原因，本周的文章将研究 Language Server Protocol 解决了什么问题，它是如何工作的，以及它的长期影响可能是什么。

{% info %}
**Update**:
The sourcekit-lsp project is now
[live on GitHub](https://github.com/apple/sourcekit-lsp).
{% endinfo %}

{% info %}
**更新**：sourcekit-lsp 项目现在已经可以 [在 GitHub 上访问](https://github.com/apple/sourcekit-lsp) 了。
{% endinfo %}

---

Imagine a grid
with each row representing a different programming language
(Swift, JavaScript, Ruby, Python, etc.)
and each column representing a different code editor
(Xcode, Visual Studio, Vim, Atom, etc.),
such that each cell represents
the level of support that a particular editor has for a language.

想象这样一个矩阵，每一行表示不同的编程语言（Swift、JavaScript、Ruby、Python 等），每一列表示不同的代码编辑器（Xcode、Visual Studio、Vim、Atom 等），这样每个单元格表示特定编辑器对一种语言的支持级别。

{% asset lsp-languages-times-editors.svg  %}

Up until recently,
what you'd find was a patchwork of compatibility
across the various combinations.
Some editors offered deep integration with a few languages
and little to no support for anything else,
whereas other editors aimed to be general-purpose
with at least a modicum of support for many languages.
(The term <abbr title="Integrated Development Environment">IDE</abbr>
is often used to describe the former.)

然后，你就发现各种组合形成了一种支离破碎的兼容。有些编辑器和部分语言深度集成，但除此之外几乎什么都干不了；其他编辑器则比较通用，对很多语言都提供了基本的支持。（IDE 这个术语通常用来描述前者。)

Case in point:
_You'd be stubborn not to use Xcode for app development
and foolish to use it for anything else._

举个奇葩的例子：_你不用 Xcode 来开发 APP，却偏用来干其他事情。_

For an editor to have better support for a particular language,
it needs to write integration code ---
either directly in the code base or via a plugin system.
Due to implementation differences across languages and editors,
improvements to, say,
Ruby support in Vim
wouldn't translate into better support for Python,
nor could they be applied to make Ruby work better in Atom.
The end result: inconsistent support across technologies
and a lot of wasted effort.

为了更好地支持某一特定的语言，编辑器必须编写一些集成代码（integration code）—— 要么直接写在项目里，要么通过插件。由于不同语言和编辑器的实现机制不一样，因此比方说 Vim 改进了对 Ruby 支持，但这并不能让它更好地支持 Python，也不能让 Ruby 在 Atom 上运行地更好。最终的结果是：大量精力浪费在了不同技术的兼容上。

The situation we described
is often referred to as an <dfn>M × N problem</dfn>,
where the number of integrations is the _product_ of
`M` editors and `N` languages.
What the Language Server Protocol does is change this M × N problem
into a <dfn>M + N problem</dfn>.

我们上面描述的情况通常被称为 _M × N 问题_，即最终的集成方案数量为编译器数量 M 与语言数量 N 的乘积。Language Server Protocol 所做的事情就是将 M × N 问题变成 _M + N 问题_。

Rather than an editor having to implement support for each language,
it only needs to support the LSP.
And in doing so, it gets the same level of functionality
for all languages that support the LSP.

编辑器不必实现对每种语言的支持，只需支持 LSP 即可。之后，它就能同等程度地支持所有支持 LSP 的语言。

{% asset lsp-languages-plus-editors.svg  %}

{% info %}
An essay from 2010 by Tomohiro Matsuyama
entitled ["Emacs は死んだ" (_"Emacs is Dead"_)](https://tkf.github.io/2013/06/04/Emacs-is-dead.html)
offers a nice encapsulation of this problem.
Describing the limitations of Emacs' scripting language
(no multi-threading, few low-level APIs, and a small user base)
Matsuyama argues that the preferred approach to writing a plugin
should be to interface with an external program
rather than implement something natively.
{% endinfo %}

{% info %}
Tomohiro Matsuyama 在 2010 年写的 ["Emacs は死んだ" (_"Emacs 已死"_)](https://tkf.github.io/2013/06/04/Emacs-is-dead.html) 这篇文章就对这种问题做出了一个很好的论述。Matsuyama 描述了 Emacs 脚本语言的局限性（不支持多线程、底层 API 过少、用户基数太小），他认为编写插件的首选方法应该是与外部程序进行交互，而不是原生实现。
{% endinfo %}

Language Server Protocol provides a common set of functionality
for supported languages, including:

Language Server Protocol 为支持的语言提供了一套通用的功能集，包括：

- Syntax Highlighting
- Automatic Formatting
- Autocomplete
- Syntax
- Tooltips
- Inline Diagnostics
- Jump to Definition
- Find References in Project
- Advanced Text and Symbol Search

- 语法高亮（Syntax Highlighting）
- 自动格式化（Automatic Formatting）
- 自动补全（Autocomplete）
- 语法（Syntax）
- 工具提示（Tooltips）
- 内联诊断（Inline Diagnostics）
- 跳转到定义（Jump to Definition）
- 项目内查找引用（Find References in Project）
- 高级文本和符号搜索（Advanced Text and Symbol Search）

Rather than reinventing the wheel for each new technology,
tools and editors can invest in better usability
and more advanced functionality.

各种工具和编辑器可以将精力用于提升可用性和提供更高级的功能，而不是为每种新技术再造个轮子。

## How Language Server Protocol Works

## Language Server Protocol 的工作原理

If you're an iOS developer,
you may be most familiar with
the terms <dfn>server</dfn> and <dfn>protocol</dfn>
in the sense of communicating with web applications
in JSON format via HTTP.
This actually isn't too far off from how the Language Server Protocol works.

如果你是一个 iOS 程序员，那么一定很熟悉 _server_ 和 _protocol_ 这两个术语在 Web 应用程序的 HTTP + JSON 通信场景下的含义。实际上 Language Server Protocol 差不多也是这么工作的。

In the case of LSP,
the _client_ refers to the editor ---
or more generally, the tool ---
and the _server_ refers to
an external program run locally in a separate process.

对于 LSP，_client_ 是指编辑器 —— 或者更宽泛一点，是指工具，_server_ 是指本地独立进程里运行的一个外部程序。

As for the _protocol_ itself,
LSP resembles a simplified version of HTTP:

至于名字中包含 _protocol_，是因为 LSP 类似于一个精简版的 HTTP：

- Each message consists of
  a header part and a content part.
- The header part has a required `Content-Length` field
  containing the size of the content part in bytes,
  and an optional `Content-Type` field
  (`application/vscode-jsonrpc; charset=utf-8` by default)
- The content part uses [JSON-RPC](https://www.jsonrpc.org/specification)
  to describe the structure of requests, responses, and notifications.

- 每个消息都由报头部分和内容部分组成。
- 报头部分包含一个必填的 `Content-Length` 字段，用于说明内容部分的大小（以字节为单位），以及一个可选的 `Content-Type` 字段（默认值为 `application/vscode-jsonrpc; charset=utf-8`）。
- 内容部分使用 [JSON-RPC](https://www.jsonrpc.org/specification) 描述请求、响应和通知的结构。

Whenever something happens in the tool,
such as the user jumping to the definition of a symbol,
the tool sends a request to the server.
The server receives that request
and then returns an appropriate response.

每当工具中发生了什么事情，比如用户需要跳转到符号的定义，工具就会向 server 发送一个请求。server 接收到该请求，然后返回适当的响应。

For example,
imagine that a user opens the following Swift code
in an Xcode-like editor that supported the Language Server Protocol:

例如，假设用户在支持 Language Server Protocol 的类 Xcode 编辑器中打开以下 Swift 代码：

```swift
class Parent {}
class Child: Parent {}
```

When the user <kbd>⌘</kbd>-clicks the symbol `Parent`
in the inheritance clause on line 2,
the editor jumps to the definition of the `Parent` class on line 1.

当用户按住 <kbd>⌘</kbd> 后点击第二行继承位置的 `Parent`，编辑器跳转到第一行定义 `Parent` 的位置。

{% asset lsp-jump-to-definition.gif  %}

Here's how LSP enables this interaction behind the scenes:

以下是 LSP 如何在幕后实现这种交互：

First,
when the user opens the Swift code,
the editor launches its Swift language server in a separate process,
if it isn't running already,
and performs any additional setup.

首先，当用户打开 Swift 代码时，若 Swift language server 并未运行，编辑器将在一个独立进程中启动它，并执行一些额外的配置。

When the user executes the "jump to definition" command,
the editor sends the following request to its Swift language server:

当用户执行 "跳转到定义（jump to definition）" 指令时，编辑器向 Swift language server 发送以下请求：

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "textDocument/definition",
  "params": {
    "textDocument": {
      "uri": "file:///Users/NSHipster/Example.swift"
    },
    "position": {
      "line": 1,
      "character": 13
    }
  }
}
```

Upon receiving this request,
the Swift language server uses a compiler tool like
[SourceKit](https://github.com/apple/swift/tree/master/tools/SourceKit)
to identify the corresponding code entity
and find the location of its declaration on the preceding line.
The language server then responds with the following message:

收到这个请求后，Swift language server 使用 [SourceKit](https://github.com/apple/swift/tree/master/tools/SourceKit) 等编译器工具来标识相应的代码实体，并在代码的上一行找到其声明的位置。然后 language server 用以下消息进行响应:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "uri": "file:///Users/NSHipster/Example.swift",
    "range": {
      "start": {
        "line": 0,
        "character": 6
      },
      "end": {
        "line": 0,
        "character": 12
      }
    }
  }
}
```

Finally,
the editor navigates to the file
(which, in this case, is already open),
moves the cursor to that range,
and highlights the token.

最后，编辑器导航到文件(在本例中，该文件已经打开)，将光标移动到该范围，并高亮显示出来。

The beauty of this approach is that the editor did all of this
without knowing anything about the Swift programming language
other than that `.swift` files are associated with Swift code.
All the editor needs to do
is talk to the language server and update the UI.
And knowing how to do that,
the editor can follow the same procedure
to facilitate this interaction
for code written in any language with a language server implementation.

这种方法的美妙之处在于，编辑器完成所有这些操作时，除了 .swift 文件与 Swift 代码相关以外，对 Swift 编程语言一无所知。编辑器需要做的就是与 language server 对话并更新 UI。而且编辑器知道如何做到这一点后，就可以遵循相同的过程，与任何带有 language server 的语言所编写的代码进行交互。

## Language Server Protocol Support in Clang / LLVM

## Clang / LLVM 里的 Language Server Protocol

If the _M + N_ diagram from before looks familiar,
it might be because it's the same approach taken by LLVM.

如果你觉得之前的 _M + N_ 图有点眼熟，那可能是因为 LLVM 也采用了同样的方法。

At the core of LLVM is an intermediate representation (IR).
Supported languages generate IR using a <dfn>compiler frontend</dfn>,
and that IR can generate machine code
for any platform supported by a <dfn>compiler backend</dfn>.

LLVM 的核心是中间表示（intermediate representation，IR）。LLVM 所支持的语言使用 _编译器前端（compiler frontend）_ 生成 IR，再使用 _编译器后端（compiler backend）_ 将 IR 生成所支持平台的机器码。

{% asset lsp-llvm-ir.svg %}

{% info %}
If you're curious to learn more about how Swift code is compiled,
check out [our article about SwiftSyntax](https://nshipster.com/swiftsyntax/).
{% endinfo %}

{% info %}
如果你想了解 Swift 代码编译的更多细节，请查看 [我们关于 SwiftSyntax 的文章](https://nshipster.com/swiftsyntax/)。
{% endinfo %}

The LLVM compiler frontend for C languages
is called [Clang](https://clang.llvm.org).
It's also used by Swift for inter-operability with Objective-C.
In its recent 5.0.0 release,
Clang added a new tool called
[Clangd](https://clang.llvm.org/extra/clangd.html),
LLVM's implementation for the Language Server Protocol.

[Clang](https://clang.llvm.org) 是 C 语言的 LLVM 编译器前端。Swift 与 Objective-C 的互操作性（inter-operability）就是靠它实现的。在最近的 5.0.0 版本中，Clang 添加了一个名为 [Clangd](https://clang.llvm.org/extra/clangd.html) 的新工具，它是 LLVM 对 Language Server Protocol 的实现。

In April 2018,
[Apple announced to the LLVM mailing list](http://lists.llvm.org/pipermail/cfe-dev/2018-April/057668.html)
that it was switching the focus of its development efforts from
[libclang](https://clang.llvm.org/doxygen/group__CINDEX.html)
to Clangd as the primary way to create interactive tooling.

2018 年 4 月，[苹果公司向 LLVM 邮件组宣布](http://lists.llvm.org/pipermail/cfe-dev/2018-April/057668.html)，将把开发的重心从 [libclang](https://clang.llvm.org/doxygen/group__CINDEX.html) 转向 Clangd，以其作为创建交互工具的主要方式。

Now you might think, _"So what?"_
Apple is among the most prominent supporters of the LLVM project ---
having, among other things,
employed the project's founder, Chris Lattner, for over a decade.
Apple's decision to switch from one obscure Clang tool to another
would seem to be an implementation detail (so to speak).

现在你可能会想，_“那又怎样？”_ 苹果公司是 LLVM 项目最重要的支持者之一，该项目创始人 Chris Lattner 已经在苹果公司工作了十多年。苹果公司决定从不透明的 Clang 工具切换到另一个，似乎是一个实现细节了(可以这么说)。

What makes this announcement quite interesting
is that Clangd appears to have been created entirely outside of Apple,
with significant contributions from Google and other companies.
This announcement signals a significant shift
in the direction of tooling development going forward ---
something that would be confirmed 6 months later on the Swift.org forums.

这个官宣很有趣的一点是，Clangd 似乎完全是在苹果以外开发的，谷歌和其他公司也做出了重大贡献。这个官宣标志着未来工具开发方向的重大转变 —— 六个月后 Swift.org 论坛将证实这一点。

## Potential Consequences of Apple's Support of Language Server Protocol

## 苹果支持 Language Server Protocol 的潜在影响

According to Apple's LSP announcement from October,
we should expect to see the first bits of code for the project
within the coming weeks (as of the time of writing; early to mid-November).

根据苹果公司 10 月份发布的 LSP 公告，我们预计在未来几周内（撰写本文时，最早 11 月中旬）将看到该项目的首批代码。

It'll take a bit longer to feel the full impact of these developments,
but believe me: your patience will be rewarded.
Here are just a few of what I believe will happen as a result of LSP
in the coming months and years.

要感受这些发展的全部影响还需要一些时间，但请相信我：你的耐心是值得的。我相信以下是 LSP 在未来几个月和几年将会发生的一些事情。

### Swift Becomes More Appealing as a General-Purpose Programming Language

### Swift 变成一种更加通用的编程语言

Although Swift is used primarily for app development,
it was designed from the start to be
a capable general-purpose programming language.
Between
[Swift for TensorFlow](https://www.tensorflow.org/swift/),
[SwiftNIO](https://github.com/apple/swift-nio),
and other projects,
we're just starting to see the promise of
what Swift can do beyond the App Store.

虽然 Swift 主要用于 APP 开发，但它从一开始就被设计成一种功能强大的通用编程语言。在 [Swift for TensorFlow](https://www.tensorflow.org/swift/)、
[SwiftNIO](https://github.com/apple/swift-nio) 和其他项目中，我们正开始看到 Swift 承诺的在 App Store 之外的使用。

Among the biggest factors holding Swift back from
mainstream adoption up to this point
has been its reliance on Xcode.

到目前为止，阻碍 Swift 被主流采用的最大因素之一是它对 Xcode 的依赖。

It's a lot to ask, say, a web developer or machine learning engineer
to download Xcode just to try Swift
when there are so many great alternatives
with a much lower barrier to entry.
Support for the Language Server Protocol
should make it significantly easier for folks outside the Apple ecosystem
to evaluate Swift with the same, familiar tools they use for everything else.

人们会有很多质疑，当有那么多优秀的、门槛低很多的替代方案可选的情况下，为什么还要让 Web 开发者或机器学习工程师仅仅为了尝试 Swift 而去下载 Xcode？支持 Language Server Protocol 可以让苹果生态圈以外的人更容易地使用他们熟悉工具去感受 Swift。

### Xcode Gets Better

### Xcode 变得更好

Adopting LSP isn't just about making Swift work better in other editors;
Xcode stands to benefit immensely, as well.

支持 LSP 不仅仅是让 Swift 在其他编辑器中运行地更好，Xcode 也将受益匪浅。

Consider [this forum post](https://forums.swift.org/t/new-lsp-language-service-supporting-swift-and-c-family-languages-for-any-editor-and-platform/17024/29)
from Project Lead for Swift at Apple, Ted Kremenek:

看看苹果 Swift 项目负责人 Ted Kremenek 的 [这篇论坛帖子](https://forums.swift.org/t/new-lsp-language-service-supporting-swift-and-c-family-languages-for-any-editor-and-platform/17024/29):

> The LSP service \[Argyrios\] is describing
> will be functionally more powerful than SourceKit is today.

> @akyrtzi 所描述的 LSP 服务将比今天的 SourceKit 更强大。

LSP is an opportunity for the Xcode team to take a fresh approach to
Swift integration,
and to capitalize on all of the improvements to the language and tooling
in the four years since its 1.0 release.

LSP 对 Xcode 团队来说是一个机遇，让他们使用一种新的方法实现 Swift 集成，并可以将其应用在语言和工具自 1.0 版本发布以来四年中的所有改进上。

### Xcode (Eventually) Becomes More Capable

### Xcode（最终）变得更强大

The benefit of LSP isn't limited to Swift and Objective-C;
[from another post by Argyrios in that thread](https://forums.swift.org/t/new-lsp-language-service-supporting-swift-and-c-family-languages-for-any-editor-and-platform/17024/33):

LSP 的好处并不限于 Swift 和 Objective-C，[Argyrios 在那个帖子的另一个留言中指出](https://forums.swift.org/t/new-lsp-language-service-supporting-swift-and-c-family-languages-for-any-editor-and-platform/17024/33)：

> Getting Xcode to use our new LSP service
> should make it viable to use other LSP services as well,
> and it’s something that we are interested in,
> but we don’t have specific plans to announce at this moment.

> Xcode 使用我们新的 LSP 服务，这意味着它也可以使用其他 LSP 服务，我们对此很感兴趣，不过目前暂无具体计划。

The main focus for the current efforts are to improve the story for Swift.
But once that's done, it should be relatively straightforward
to have those benefits cascade down to other languages with LSP support.

目前的工作重点是改进 Swift。但是，一旦实现了这一点，就应该能相对简单地将这些优化转移到支持 LSP 的其他语言中。

---

The architecture of software
reflects the structure and values of the organizations that create it.
The converse is true as well, to some extent.

软件的架构反映了创建它的组织的结构和价值。在某种程度上，反之亦然。

By adopting the open Language Server Protocol standard for Xcode,
Apple is making good on its commitment to the success of Swift
on platforms outside the Apple ecosystem.
And I think it'll work:
tooling (or lack thereof) is often the key decider
in which technologies gain mindshare.
But perhaps more importantly,
I believe this decision demonstrates an increased willingness within
(at least some small part of) the company
for collaboration and transparency.

通过让 Xcode 支持开放的 Language Server Protocol 标准，苹果正在履行其在苹果生态系统之外的平台上实现 Swift 成功的承诺。我认为这是可行的：工具（或缺少工具）通常是技术获得人心的关键决定因素。但或许更重要的是，我认为这一决定表明，公司内部（至少是一小部分）对合作和透明度的意愿有所增强。
