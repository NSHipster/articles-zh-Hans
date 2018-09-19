---
title: CMMotionActivity
author: Mattt
translator: Bei Li
category: Cocoa
excerpt: >
  如今的 iPhone 都有着一整套传感器，包括相机、气压计、陀螺仪、磁强计和加速规。和人类一样，它们使用不同感觉信息的组合来确定其位置和朝向，通常和我们自身的生物力学过程非常相似。
status:
  swift: 4.2
---

人类通过组合视觉、本体感觉和前庭系统得到的感觉信息来察觉自身的运动。其中前庭系统起到主要作用，前庭系统由感觉旋转的<dfn>半规管</dfn>和对水平与垂直受力敏感的<dfn>耳石</dfn>构成。

如今的 iPhone 都有着一整套传感器，包括相机、气压计、陀螺仪、磁强计和加速规。和人类一样，它们使用不同感觉信息的组合来确定其位置和朝向，通常和我们自身的生物力学过程非常相似。

让感觉输入有意义——不管它们是从哪里来的——是非常有挑战性的。需要考虑的信息实在太多了。（见鬼，我们这个种族花了小几百万年才搞定这件事，然而我们还是会被像电梯、飞机和过山车这些新奇的发明弄晕。）

经过几个版本的系统与硬件更新，苹果的设备变得擅长于区分不同运动方式。在你跑去尝试自己实现前，停下来考虑一下使用这周文章中讨论的内置 API。

---

在 iOS 和 watchOS 上，`CMMotionActivityManager` 处理设备中传感器的原始数据并告诉你（有多确定）用户是否正在移动，和用户是在行走、跑步、骑行或者开车。

要使用这个 API，首先创建一个活动管理器，然后使用 `startActivityUpdates` 方法来开始监听活动更新。每当设备更新了运动相关活动，它就会执行指定的闭包，并传入一个 `CMMotionActivity` 对象。

```swift
let manager = CMMotionActivityManager()
manager.startActivityUpdates(to: .main) { (activity) in
    guard let activity = activity else {
        return
    }

    var modes: Set<String> = []
    if activity.walking {
        modes.insert("🚶‍")
    }

    if activity.running {
        modes.insert("🏃‍")
    }

    if activity.cycling {
        modes.insert("🚴‍")
    }

    if activity.automotive {
        modes.insert("🚗")
    }

    print(modes.joined(separator: ", "))
}
```

`CMMotionActivityManager` 由 Core Motion 框架提供。支持 Core Motion 的设备都装备了一个运动协处理器。通过使用专用硬件，系统可以将所有传感器处理工作从 CPU 上卸载掉，并减少电量的使用。

第一款 **M 系列**协处理器是 M7，在 2013 年九月与 iPhone 5S 一同面世。iOS 7 和 Core Motion API 也与此同时发布。

## 司机功能

可能运动活动最广为人知的使用是 iOS 11 中新增的[「驾驶勿扰」](https://support.apple.com/zh-cn/HT208090)功能。这项功能推出后，对于开车的检测变好了很多。

> 在速度比较低的情况下，只使用加速规的数据是很难区分驾车行驶和其他活动的。我们只能猜测，有可能 iPhone 是使用磁强计的数据来实现这项功能的。因为汽车和其他机动车辆通常被金属围绕着，电磁通量会减少。

除了安全相关的考虑，一些应用可能会根据当前的交通模式改变行为。比如，一个快递应用可能会发送运动活动改变到服务器来重新计算预计到达时间，或者更新 UI 来表明快递员已经停下了车子，正在走路接近。

## 不动地移动

`CMMotionActivity` 对每种运动类型都有一个布尔值属性，并且还有还有一个设备是否处于静止状态的属性。这似乎违反直觉，逻辑上来说你在同一时间只能步行或者开车，但不能一起进行。

关于这一点在 [`CMMotionActivity` 的文档](https://developer.apple.com/documentation/coremotion/cmmotionactivity)中有澄清：

> 本类中关于运动的属性并不是互相排斥的。换句话说，有可能有多个运动相关属性的值为 `true`。举个例子，如果用户在**驾驶一辆车**，然后在红灯前停了下来，对于这个事情中运动相关的更新事件会将 `cycling` 和 `stationary` 属性都设为真。
>
> 文档原文：
>
> The motion-related properties of this class are not mutually exclusive.
> In other words, it is possible for more than one of the
> motion-related properties to contain the value `true`.
> For example, if the user was **driving in a car**
> and the car stopped at a red light,
> the update event associated with that change in motion would have both
> the `cycling` and `stationary` properties set to true.

等等，我刚才说了澄清？我的意思是……反正不是和这个文档这样。（我很确定这个例子应该是 `automotive`）幸好，头文件的文档有比我们想知道的更多信息。下面有一些这个 API 在不同情况下具体的例子：

**场景 1**：你在一辆停在红灯前的车里

| 🚶‍ `walking` | 🏃‍ `running` | 🚴‍`cycling` | 🚗 `automotive` | 🛑 `stationary` |
| ------------- | ------------- | ------------ | --------------- | --------------- |
| `false`       | `false`       | `false`      | `true`          | `true`          |

**场景 2**：你在一辆移动中的机动车里

| 🚶‍ `walking` | 🏃‍ `running` | 🚴‍`cycling` | 🚗 `automotive` | 🛑 `stationary` |
| ------------- | ------------- | ------------ | --------------- | --------------- |
| `false`       | `false`       | `false`      | `true`          | `false`         |

**场景 3**：设备正在运动，但你没有在走路也没有在机动车辆上

| 🚶‍ `walking` | 🏃‍ `running` | 🚴‍`cycling` | 🚗 `automotive` | 🛑 `stationary` |
| ------------- | ------------- | ------------ | --------------- | --------------- |
| `false`       | `false`       | `false`      | `false`         | `false`         |

**场景 4**：你是一位名侦探，在一辆行驶中的火车上的走廊里追逐嫌疑犯，跑到了最后一节车厢并停下来四处查看并猜测他们藏在哪。**（可能在角落里那个可疑、一人大小的箱子里？）**

| 🚶‍ `walking` | 🏃‍ `running` | 🚴‍`cycling` | 🚗 `automotive` | 🛑 `stationary` | 🕵️‍🇧🇪 `poirot` |
| ------------- | ------------- | ------------ | --------------- | --------------- | --------------- |
| `false`       | `true`        | `false`      | `true`          | `true`          | `true`          |

（我们实际上并不确定最后一个场景会发生什么……）

总的来说这个文档的大意是，你应该认为 `stationary` 和 `CMMotionActivity` 中其他属性是正交的，并且你应该准备好处理所有的组合情况。

每个 `CMMotionActivity` 对象还包括了一个其值可能为 `.low`、`.medium` 或 `.high` 的 `confidence` 属性。不幸的是，文档既没有提供多少关于这些值的有用信息，也没有说明它们要如何使用。对于这种情况，推荐使用一种经验性的方法：实际测试你的应用，观察不同情况下出现的 `confidence` 值，使用这些信息来修正应用的行为。

## 与位置查询组合使用

根据你的实际情况，组合使用 Core Motion 和 [Core Location](https://nshipster.cn/core-location-in-ios-8/) 数据可能是有意义的。

你可以组合一段时间的位置变化和把握比较低的运动活动数据来提高精确度。这里有一些不同移动方式典型速度范围的指导方针：

- 步行速度通常最高能达到 2.5 米每秒（5.6 mph, 9 km/h）
- 跑步速度范围从 2.5 到 7.5 米每秒（5.6 – 16.8 mph, 9 – 27 km/h）
- 骑行速度范围从 3 到 12 米每秒（6.7 – 26.8 mph, 10.8 – 43.2 km/h）
- 汽车的速度可以超过 100 米每秒（220 mph, 360 km/h）

或者，你可能会使用位置数据来改变 UI，取决于现在的位置是否在一片水域。

```swift
if currentLocation.intersects(waterRegion) {
    if activity.walking {
        print("🏊‍")
    } else if activity.automotive {
        print("🚢")
    }
}
```

然而，位置数据应该只在绝对必要的时候再查询——当你需要查询时，也应该尽量少的查询，比如只检测显著的位置改变。这样的原因是，获取位置数据要求使用 GPS 且/或移动网络，它们都非常耗电。

---

`CMMotionActivityManager` 是 Core Motion 里那些好用 API 的其中一个，你可以使用它来构造沉浸般的、快速响应的应用。

如果你还没有考虑过将设备运动信息纳入应用的潜力（或者可能你很久没有关注过 Core Motion），你可能会被它能做到的事情惊讶到。
