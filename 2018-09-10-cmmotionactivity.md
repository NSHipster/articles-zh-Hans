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

Humans perceive self-motion using a combination of
sensory information from their visual, proprioceptive, and vestibular systems.
Of these,
the primary determination is made by the vestibular system,
comprising the
<dfn>semicircular canals</dfn>,
which sense changes in rotation,
and the <dfn>otoliths</dfn>,
which are sensitive to horizontal and vertical forces.

人类通过组合视觉、本体感觉和前庭系统得到的感觉信息来察觉自身的运动。其中前庭系统起到主要作用，前庭系统由感觉旋转的<dfn>半规管</dfn>和对水平与垂直受力敏感的<dfn>耳石</dfn>构成。

Today's iPhones are packed with a full complement of sensors that includes
cameras, barometers, gyroscopes, magnetometers, and accelerometers.
Like humans, they use permutations of different sensory information
to make determinations about their position and orientation,
often by means quite similar to our own biomechanical processes.

如今的 iPhone 都有着一整套传感器，包括相机、气压计、陀螺仪、磁强计和加速规。和人类一样，它们使用不同感觉信息的组合来确定其位置和朝向，通常和我们自身的生物力学过程非常相似。

Making sense of sensory inputs ---
no matter their origin ---
is challenging.
There's just so much information to consider.
(Heck, it took our species a few million years to get that right,
and we're still confused by newfangled inventions like
elevators, planes, and roller coasters.)

让感觉输入有意义——不管它们是从哪里来的——是非常有挑战性的。需要考虑的信息实在太多了。（见鬼，我们这个种族花了小几百万年才搞定这件事，然而我们还是会被像电梯、飞机和过山车这些新奇的发明弄晕。）

After several major OS releases and hardware versions,
Apple devices have become adroit at differentiating between different
means of locomotion.
But before you run off and try to write your own implementation,
stop and consider using the built-in APIs discussed in this week's article.

经过几个版本的系统与硬件更新，苹果的设备变得擅长于区分运动的不同意义。在你跑去尝试自己实现前，停下来考虑一下使用这周文章中讨论的内置 API。

---

On iOS and watchOS,
`CMMotionActivityManager` takes raw sensor data from the device
and tells you (to what degree of certainty)
whether the user is currently moving,
and if they're walking, running, biking, or driving in an automobile.

在 iOS 和 watchOS 上，`CMMotionActivityManager` 处理设备中传感器的原始数据并告诉你（有多确定）用户是否正在移动，和用户是在行走、跑步、骑行或者开车。

To use this API,
you create an activity manager
and start listening for activity updates
using the `startActivityUpdates` method.
Each time the device updates the motion activity,
it executes the specified closure,
passing a `CMMotionActivity` object.

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

`CMMotionActivityManager` is provided by the Core Motion framework.
Devices that support Core Motion are equipped with a motion coprocessor.
By using dedicated hardware,
the system can offload all sensor processing from the CPU
and minimize energy usage.

`CMMotionActivityManager` 由 Core Motion 框架提供。支持 Core Motion 的设备都装备了一个运动辅助处理器。通过使用专用硬件，系统可以将所有传感器处理工作从 CPU 上移除掉，并减少电量的使用。

The first of the _M-series_ coprocessors was the M7,
which arrived in September 2013 with the iPhone 5S.
This coincided with the release of iOS 7 and the Core Motion APIs.

第一款 **M 系列**辅助处理器是 M7，在 2013 年九月与 iPhone 5S 一同面世。iOS 7 和 Core Motion API 也与此同时发布。

## Feature Drivers
## 司机功能

Possibly the most well-known use of motion activities is
["Do Not Disturb While Driving"](https://support.apple.com/en-us/HT208090),
added in iOS 11.
Automotive detection got much better with the introduction of this feature.

可能运动活动最广为人知的使用是 iOS 11 中新增的[「驾驶勿扰」](https://support.apple.com/zh-cn/HT208090)功能。这项功能推出后，对于开车的检测变好了很多。

> At low speeds, it's hard to distinguish automobile travel from other means
> using accelerometer data alone.
> Although we can only speculate as to how this works,
> it's possible that the iPhone uses magnetometer data from the device compass.
> Because cars and other vehicles are often enclosed in metal,
> electromagnetic flux is reduced.

> 在速度比较低的情况下，只使用加速规的数据是很难区分驾车行驶和其他活动的。我们只能猜测，有可能 iPhone 是使用磁强计的数据来实现这项功能的。因为汽车和其他机动车辆通常被金属围绕着，电磁通量会减少。

Beyond safety concerns,
some apps might change their behavior
according to the current mode of transportation.
For example,
a delivery service app might relay changes in motion activity to a server
to recalculate estimated ETA or change the UI to communicate
that the courier has parked their vehicle and are now approaching by foot.

除了安全相关的考虑，一些应用可能会根据当前的交通模式改变行为。比如，一个快递应用可能会发送运动活动改变到服务器来重新计算预计到达时间，或者更新 UI 来表明快递员已经停下了车子，正在走路接近。

## Traveling Without Moving
## 不动地移动

`CMMotionActivity` has Boolean properties
for each of the different types of motion
as well as one for whether the device is stationary.
This seems counter-intuitive,
as logic dictates that
you can either be walking or driving a car at a given moment,
but not both.

`CMMotionActivity` 对每种运动类型都有一个布尔值属性，并且还有还有一个设备是否停滞的属性。这似乎违反直觉，逻辑上来说你在同一时间只能步行或者开车，但不能一起进行。

This point is clarified by the [`CMMotionActivity` documentation](https://developer.apple.com/documentation/coremotion/cmmotionactivity):

关于这一点在 [`CMMotionActivity` 的文档](https://developer.apple.com/documentation/coremotion/cmmotionactivity)中有澄清：

> The motion-related properties of this class are not mutually exclusive.
> In other words, it is possible for more than one of the
> motion-related properties to contain the value `true`.
> For example, if the user was **driving in a car**
> and the car stopped at a red light,
> the update event associated with that change in motion would have both
> the `cycling` and `stationary` properties set to true.

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

Wait, did I say clarified?
I meant... whatever the opposite is.
(I'm pretty sure this should be `automotive`)
Fortunately, the header docs tell us more about what we might expect.
Here are some concrete examples of how this API behaves in different situations:

等等，我刚才说了澄清？我的意思是……反正不是和这个文档这样。（我很确定这个例子应该是 `automotive`）幸好，头文件的文档有比我们想知道的更多信息。下面有一些这个 API 在不同情况下具体的例子：

**Scenario 1**:
You're in a car stopped at a red light

**场景 1**：你在一辆停在红灯前的车里

| 🚶‍ `walking` | 🏃‍ `running` | 🚴‍`cycling` | 🚗 `automotive` | 🛑 `stationary` |
| ------------- | ------------- | ------------ | --------------- | --------------- |
| `false`       | `false`       | `false`      | `true`          | `true`          |

**Scenario 2**:
You're in a moving vehicle

**场景 2**：你在一辆移动中的机动车里

| 🚶‍ `walking` | 🏃‍ `running` | 🚴‍`cycling` | 🚗 `automotive` | 🛑 `stationary` |
| ------------- | ------------- | ------------ | --------------- | --------------- |
| `false`       | `false`       | `false`      | `true`          | `false`         |

**Scenario 3**:
The device is in motion, but you're neither walking nor in a moving vehicle

**场景 3**：设备正在运动，但你没有在走路也没有在机动车辆上

| 🚶‍ `walking` | 🏃‍ `running` | 🚴‍`cycling` | 🚗 `automotive` | 🛑 `stationary` |
| ------------- | ------------- | ------------ | --------------- | --------------- |
| `false`       | `false`       | `false`      | `false`         | `false`         |

**Scenario 4**:
You're a world-famous detective, who,
in the process of chasing a suspect down the corridors of a moving train,
has reached the last car and has stopped to look around
to surmise where they're hiding
_(perhaps that conspicuous, person-sized box in the corner?)_

**场景 4**：你是一个名侦探，在一辆行驶中的火车上的走廊里追逐嫌疑犯，跑到了最后一节车厢并停下来四处查看猜测他们藏在哪。**（可能在角落里那个可疑、一人大小的箱子里？）**

| 🚶‍ `walking` | 🏃‍ `running` | 🚴‍`cycling` | 🚗 `automotive` | 🛑 `stationary` | 🕵️‍🇧🇪 `poirot` |
| ------------- | ------------- | ------------ | --------------- | --------------- | --------------- |
| `false`       | `true`        | `false`      | `true`          | `true`          | `true`          |

(We're actually not sure what would happen in that last scenario...)

（我们实际上并不确定最后一个场景会发生什么……）

The overall guidance from the documentation is that
you should treat `stationary` to be orthogonal from
all of the other `CMMotionActivity` properties
and that you should be prepared to handle all other combinations.

总的来说这个文档的大意是，你应该认为 `stationary` 和 `CMMotionActivity` 中其他属性是正交的，并且你应该准备好处理所有的组合情况。

Each `CMMotionActivity` object also includes a `confidence` property
with possible values of `.low`, `.medium`, and `.high`.
Unfortunately, not much information is provided by the documentation
about what any of these values mean, or how they should be used.
As is often the case,
an empirical approach is recommended;
test your app in the field to
see when different confidence values are produced
and use that information to determine the correct behavior for your app.

每个 `CMMotionActivity` 对象还包括了一个其值可能为 `.low`、`.medium` 或 `.high` 的 `confidence` 属性。不幸的是，文档没有提供多少关于这些值的有用信息，或者如何使用。对于这种情况，推荐使用一种经验性的方法：实际测试你的应用，观察不同情况下出现的 `confidence` 值，使用这些信息来修正应用的行为。

## Combining with Location Queries
## 与位置查询组合使用

Depending on your use case,
it might make sense to coordinate Core Motion readings with
[Core Location](https://nshipster.com/core-location-in-ios-8/) data.

根据你的实际情况，组合使用 Core Motion 和 [Core Location](https://nshipster.cn/core-location-in-ios-8/) 数据可能是有意义的。

You can combine changes in location over time
with low-confidence motion activity readings
to increase accuracy.
Here are some general guidelines for typical ranges of speeds
for each of the modes of transportation:

你可以组合一段时间的位置变化和把握比较低的运动活动数据来提高精确度。这里有一些不同移动方式典型速度范围的指导方针：

- Walking speeds typically peak at 2.5 meters per second (5.6 mph, 9 km/h)
- 步行速度通常最高能达到 2.5 米每秒（5.6 mph, 9 km/h）
- Running speeds range from 2.5 to 7.5 meters per second (5.6 – 16.8 mph, 9 – 27 km/h)
- 跑步速度范围从 2.5 到 7.5 米每秒（5.6 – 16.8 mph, 9 – 27 km/h）
- Cycling speeds range from 3 to 12 meters per second (6.7 – 26.8 mph, 10.8 – 43.2 km/h)
- 骑行速度范围从 3 到 12 米每秒（6.7 – 26.8 mph, 10.8 – 43.2 km/h）
- Automobile speeds can exceed 100 meters per second (220 mph, 360 km/h)
- 汽车的速度可以超过 100 米每秒（220 mph, 360 km/h）

Alternatively, you might use location data to change the UI
depending on whether the current location is in a body of water.

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

However, location data should only be consulted if absolutely necessary ---
and when you do, it should be done sparingly,
such as by monitoring for only significant location changes.
Reason being,
location data requires turning on the GPS and/or cellular radio,
which are both energy intensive.

然而，位置数据应该只在绝对必要的时候再查询——当你需要查询时，也应该尽量少的查询，比如只检测显著的位置改变。这样的原因是，获取位置数据要求使用 GPS 且/或移动网络，它们都非常耗电。

---

`CMMotionActivityManager` is one of many great APIs in Core Motion
that you can use to build immersive, responsive apps.

`CMMotionActivityManager` 是 Core Motion 里那些好用 API 的其中一个，你可以使用它来构造沉浸、反应灵敏的应用。

If you haven't considered the potential of
incorporating device motion into your app
(or maybe haven't looked at Core Motion for a while),
you might be surprised at what's possible.

如果你还没有考虑过将设备运动信息纳入应用的可能性（或者可能你很久没有关注过 Core Motion），你可能会被它能做到的事情惊讶到。
