---
layout: post
title: "Core Data Libraries & Utilities"
category: Open Source
author: Mattt
translator: JJ Mao
retired: true
excerpt: "上周，我们感觉Core Data有些难，所以为了NSHipster的这个问题，我们将奉上关于使用Core Data的最好开源库的引导。仔细阅读，看看你如何充分利用这次Core Data体验。"
---

因此我们姑且认为你已经确定自己的特定需求并且对比过所有的备选方案，选择在你的下一个应用中使用[Core Data](https://developer.apple.com/library/mac/#documentation/cocoa/Conceptual/CoreData/cdProgrammingGuide.html)。

没错！Core Data 是应用模型、持久化和大量对象图查找的最佳选择。

当然，Core Data 很复杂、繁琐，有时还是个[讨厌鬼](https://nshipster.com/nscoding#figure-2)— 但是，天杀的，史上最好和最受欢迎的一些应用都用 Core Data。既然 Core Data 对他们来说已经足够好了，那对你来说也应该够好了。

...但这并不意味着 Core Data 无法改善了。

虽然很多库试着替换 Core Data，但是有更多库在尝试改善它。这些库涵盖了从必要的语法糖到广泛的全栈框架。

本周的 NSHipster：将奉上关于使用 Core Data 的最好开源库的引导。仔细阅读，看看你如何充分利用这次 Core Data 体验。

---

> 为方便起见，提供下表。该表包含了使用 Core Data 的最重要的开源库和实用工具。该清单并不全面，如果你觉得有什么需要补充或是不合适，请在推特上联系[@NSHipster](https://twitter.com/NSHipster)—或是[提交一个 pull request](https://github.com/NSHipster/articles)，那就更好了。

<table>
  <thead>
    <tr>
      <th colspan="2">Wrappers</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><a href="https://github.com/magicalpanda/MagicalRecord">Magical Record</a></td>
      <td><a href="https://github.com/casademora">Saul Mora</a></td>
    </tr>
    <tr>
      <td><a href="https://github.com/mneorr/Objective-Record">Objective-Record</a></td>
      <td><a href="https://github.com/mneorr">Marin Usalj</a></td>
    </tr>
    <tr>
      <td><a href="https://github.com/soffes/SSDataKit">SSDataKit</a></td>
      <td><a href="https://github.com/soffes">Sam Soffes</a></td>
    </tr>
    <tr>
      <td><a href="https://github.com/martydill/ios-queryable">ios-queryable</a></td>
      <td><a href="https://github.com/martydill">Marty Dill</a></td>
    </tr>
    <tr>
      <td><a href="https://github.com/apparentsoft/ReactiveCoreData">ReactiveCoreData</a></td>
      <td><a href="https://github.com/apparentsoft">Jacob Gorban</a></td>
    </tr>
  </tbody>

  <thead>
    <tr>
      <th colspan="2">Adapters</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><a href="https://github.com/RestKit/RestKit">RestKit</a></td>
      <td><a href="https://github.com/blakewatters">Blake Watters</a></td>
    </tr>
    <tr>
      <td><a href="https://github.com/AFNetworking/AFIncrementalStore">AFIncrementalStore</a></td>
      <td><a href="https://github.com/mattt">Mattt Thompson</a></td>
    </tr>
    <tr>
      <td><a href="https://github.com/mutualmobile/MMRecord">MMRecord</a></td>
      <td><a href="https://github.com/cnstoll">Conrad Stoll</a></td>
    </tr>
    <tr>
      <td><a href="https://github.com/OliverLetterer/SLRESTfulCoreData">SLRESTfulCoreData</a></td>
      <td><a href="https://github.com/OliverLetterer">Oliver Letterer</a></td>
    </tr>
    <tr>
      <td><a href="https://github.com/gonzalezreal/Overcoat">Overcoat</a></td>
      <td><a href="https://github.com/gonzalezreal">Guillermo Gonzalez</a></td>
    </tr>
    <tr>
      <td><a href="https://github.com/github/Mantle">Mantle</a></td>
      <td><a href="https://github.com/GitHub">GitHub</a></td>
    </tr>
  </tbody>

  <thead>
    <tr>
      <th colspan="2">Synchronizers</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><a href="https://github.com/nothirst/TICoreDataSync">TICoreDataSync</a></td>
      <td><a href="https://github.com/timisted">Tim Isted</a>, <a href="https://github.com/MrRooni">Michael Fey</a>, <a href="https://github.com/kevinhoctor">Kevin Hoctor</a>, <a href="https://github.com/chbeer">Christian Beer</a>, <a href="https://github.com/tonyarnold">Tony Arnold</a>, and <a href="https://github.com/dannygreg">Danny Greg</a></td>
    </tr>
    <tr>
      <td><a href="https://github.com/lhunath/UbiquityStoreManager">UbiquityStoreManager</a></td>
      <td><a href="https://github.com/lhunath">Maarten Billemont</a></td>
    </tr>
  </tbody>

  <thead>
    <tr>
      <th colspan="3">Utilities</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><a href="https://github.com/rentzsch/mogenerator">mogenerator</a></td>
      <td><a href="https://github.com/rentzsch">Jonathan 'Wolf' Rentzsch</a></td>
    </tr>
  </tbody>
</table>

## Wrappers

Wrapper 库为 Core Data 繁琐复杂的接口提供一些必要的语法糖和便利方法。

例如，为了在一个 managed object context 中插入新的 managed object，这是`NSEntityDescription`中（而不是如人们可能预期的`NSManagedObject`或`NSManagedObjectContext`）的一个类方法。`NSEntityDescription +insertNewObjectForEntityForName:inManagedObjectContext:`。神马？

有许多开源库为 Core Data 接口最艰难的部分一起识别和纠正。在`AppDelegate`外部管理主要和私有的 context，便于操作和查询 managed objects 的方法。

#### [SSDataKit](https://github.com/soffes/SSDataKit)

> 有许多引用代码需要编写一个 Core Data 应用。这很令人不爽。自从 iOS 有了 Core Data，几乎所有我编写的应用都用到了下面的类。

### 来自[Active Record](http://api.rubyonrails.org/classes/ActiveRecord/Base.html)的启发

学习了一种做事方式后将这些想法和习惯用于其他技术这件事对于程序员们来说应该是不足为奇的。一大波熟悉范式[Active Record](http://api.rubyonrails.org/classes/ActiveRecord/Base.html)的 Ruby 开发者涌入 iOS 行列。

与主流看法相反，Core Data _不是_ 一个[Object-Relational Mappers](https://en.wikipedia.org/wiki/Object-relational_mapping)，而是一个对象图和持久性框架，它的能力远超于单独的[Active Record 模式](https://en.wikipedia.org/wiki/Active_record_pattern)。使用 Core Data 作为 ORM（对象关系映射）必然限制 Core Data 功能并混淆其概念的纯粹。但对于许多渴望熟悉 ORM 的开发者，这个交易的代价是双倍的！

#### [Magical Record](https://github.com/magicalpanda/MagicalRecord)

> MagicalRecord 的灵感来自于 Ruby on Rails 的 Active Record 轻松抓取。该代码是为了清理 Core Data 的相关代码，允许简单明了的一行抓取，当需要请求优化时还允许修改`NSFetchRequest`。

#### [Objective-Record](https://github.com/mneorr/Objective-Record)

> 这是管理 Core Data 对象的轻量级 ActiveRecord 方式。
> 该语法借助于 Ruby on Rails。
> 没有 AppDelegate 代码。
> 经过[Kiwi](https://github.com/allending/Kiwi)的完整测试。

### 来自[LINQ](https://en.wikipedia.org/wiki/Language_Integrated_Query)的启发

这儿有个有趣的游戏：下次你遇到来自.NET 界的开发者，设置一个定时器看看他们夸赞[LINQ](https://en.wikipedia.org/wiki/Language_Integrated_Query)用了多长时间。说真的，LINQ 是 _真爱_ 。

对外行来说，LINQ 就像是[SQL](https://en.wikipedia.org/wiki/SQL)，但整合成了一种语言特性。想想`NSPredicate`，[`NSSortDescriptor`](https://nshipster.com/nssortdescriptor/)，和[`Key-Value Coding`](https://nshipster.com/kvc-collection-operators/)都有更棒的语法：

```
from c in SomeCollection
  where c.SomeProperty < 10
  select new {c.SomeProperty, c.OtherProperty};
```

#### [ios-queryable](https://github.com/martydill/ios-queryable)

> ios-queryable 支持 LINQ-风格组合查询和延迟执行，并实现了 IEnumerable 方法的一个子集，包括`where`，`take`，`skip`，`orderBy`，`first/firstOrDefault`，`single/singleOrDefault`，`count`，`any`和`all`。

### 来自[ReactiveCocoa](https://github.com/ReactiveCocoa)的启发

ReactiveCocoa [为 Objective-C 带来了函数式反应型范例](https://nshipster.com/reactivecocoa/), 它现在正为 Core Data 带来一些函数式的稳健性和条理性。这仍是个未知的领域，但最初的结果是的确是有希望的。

#### [ReactiveCoreData](https://github.com/apparentsoft/ReactiveCoreData)

> ReactiveCoreData (RCD)试图将 Core Data 带入 ReactiveCocoa (RAC)的世界。

## Adapters

大多数 iOS 应用与网络服务通信。对于使用 Core Data 的应用，从 REST 或 RPC-风格的网络服务获取、更新、删除记录是很常见的。保持本地缓存和服务器之间的一致性是一件看似棘手的事情。

保持对象更新，删除重复的记录，将实体映射到接口端点，协调冲突，管理网络可通性...这些只是一个开发者创建稳健的客户端-服务器应用时面对的部分挑战。

幸运的是，有大量的开源库可以减轻部分烦恼。

#### [RestKit](https://github.com/RestKit/RestKit)

> RestKit 是一个现代化的 Objective-C 框架用于在 iOS 和 OS X 上实现 RESTful 网络服务客户端。它提供一种强大的[对象映射](https://github.com/RestKit/RestKit/wiki/Object-mapping)引擎与[Core Data](https://developer.apple.com/library/mac/#documentation/cocoa/Conceptual/CoreData/cdProgrammingGuide.html)无缝集成，并为建立在[AFNetworking](https://github.com/AFNetworking/AFNetworking)之上的映射 HTTP 请求和响应提供了一组简单的网络层原语。它有一套精心设计的接口，使得访问和建模 RESTful 资源几乎有种不可思议的感觉。

#### [AFIncrementalStore](https://github.com/AFNetworking/AFIncrementalStore)

> AFIncrementalStore 是 NSIncrementalStore 的子类，它使用 AFNetworking 自动请求资源作为必要的属性和联系。

#### [MMRecord](https://github.com/mutualmobile/MMRecord)

> MMRecord 是用于 iOS 和 OS X 的基于 block 的无缝网络服务集成库。它利用 Core Data 模型配置从接口响应自动创建和填充一个完整的对象图。MMRecord 适用于任何网络库，设置简单，包含了许多流行特性使得网络服务的使用更加容易。

#### [SLRESTfulCoreData](https://github.com/OliverLetterer/SLRESTfulCoreData)

> `SLRESTfulCoreData`建于[AFNetworking](https://github.com/AFNetworking/AFNetworking)和[SLCoreDataStack](https://github.com/OliverLetterer/SLCoreDataStack) 之上，使你在几分钟内就能映射 JSON REST API 到你的 CoreData 模型。

#### [Overcoat](https://github.com/gonzalezreal/Overcoat)

> Overcoat 是一个[AFNetworking](https://github.com/AFNetworking/AFNetworking)扩展，它让开发者使用 REST 客户端的 Mantle 模型对象变得超级简单。

#### [Mantle](https://github.com/github/Mantle)

> Mantle 简化了编写简单的 Cocoa 或 Cocoa Touch 应用模型层。

## Synchronizers

鉴于适配器通过现有的、通用接口（如，REST）同步信息，同步装置以可移植性和通用性为代价，使用更加直接的协议提供更好的集成和性能。

### [TICoreDataSync](https://github.com/nothirst/TICoreDataSync)

> 自动同步 Core Data 应用，在 OS X 和 iOS 之间任意组合：从 Mac 到 iPhone 或 iPad 或 iPod touch，以及从 iPhone 或 iPad 或 iPod touch 到 Mac。

#### [UbiquityStoreManager](https://github.com/lhunath/UbiquityStoreManager)

> UbiquityStoreManager 控制器实现了 iCloud 与 Core Data 的集成。

## Utilities

在未提及 Mogenerator 时我们会疏忽对开源 Core Data 生态系统的反馈。在前 iPhone 时代幸存下来的项目中，Mogenerator 已成为这些年来开发者不可或缺的工具。这些年来，虽然 Core Data 改变了很多，但是这一常量已经成为 Apple 综合工具的典型不足。幸运的是，有 Mr. Wolf Rentzsch 罩着我们。

#### [Mogenerator](https://github.com/rentzsch/mogenerator)

> `mogenerator`是命令行工具，创建一个`.xcdatamodel` 文件，并创建*两个类（每个实体中）*。第一个类`_MyEntity`是单独为机器准备的，它将不断重写保持与数据模型的同步。第二个类`MyEntity`，是`_MyEntity`的子类，它将不会被重写，是个存放自定义逻辑的好地方。

---

记住：这儿没有包治百病的良药，也没有放之四海皆准的解答。就像 Core Data 可能只是在特定情况下建议使用，前面提到的 Core Data 库也是如此。

即使只为了帮助每个库确定相对优势和对比权衡，把生态系统分成大类也是有益的。只有你自己知道（当然，有时候需要通过试错）哪种方法是最适合自己的。
