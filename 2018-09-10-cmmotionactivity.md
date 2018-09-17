---
title: CMMotionActivity
author: Mattt
translator: Bei Li
category: Cocoa
excerpt: >
  如今的 iPhone 都有着一整套的传感器，包括相机、气压计、陀螺仪、磁强计和加速规。和人类一样，它们使用不同感觉信息的组合来确定其位置和朝向，通常和我们自身的生物力学过程非常相似。
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

人类通过组合视觉、本体感觉和前庭系统得到的感觉信息来察觉自身的运动。其中前庭系统起到主要作用，前庭系统由感觉旋转的<dfn>半规管</dfn>和对水平与垂直受力敏感的<dfn>耳石</dfn>构成。

Today's iPhones are packed with a full complement of sensors that includes
cameras, barometers, gyroscopes, magnetometers, and accelerometers.
Like humans, they use permutations of different sensory information
to make determinations about their position and orientation,
often by means quite similar to our own biomechanical processes.

如今的 iPhone 都有着一整套的传感器，包括相机、气压计、陀螺仪、磁强计和加速规。和人类一样，它们使用不同感觉信息的组合来确定其位置和朝向，通常和我们自身的生物力学过程非常相似。

Making sense of sensory inputs ---
no matter their origin ---
is challenging.
There's just so much information to consider.
(Heck, it took our species a few million years to get that right,
and we're still confused by newfangled inventions like
elevators, planes, and roller coasters.)

让感觉输入有意义——不管它们是从哪里来的——是非常有挑战性的。需要考虑的信息实在太多了。（见鬼，我们这个种族花了小几百万年才搞定这件事，然而我们还是会被像电梯、飞机和过山车这些新奇的发明弄晕。）

After several major OS releases and hardware versions,
Apple devices have become adroit at differentiating between different
means of locomotion.
But before you run off and try to write your own implementation,
stop and consider using the built-in APIs discussed in this week's article.

经过几个版本的系统与硬件更新，苹果的设备变得擅长于区分运动的不同意义。在你跑去尝试自己实现前，停下来考虑一下使用这周文章中讨论的内置 API。

---

On iOS and watchOS,
`CMMotionActivityManager` takes raw sensor data from the device
and tells you (to what degree of certainty)
whether the user is currently moving,
and if they're walking, running, biking, or driving in an automobile.

在 iOS 和 watchOS 上，`CMMotionActivityManager` 处理设备中传感器的原始数据并告诉你（有多确定）用户是否正在移动，和用户是在行走、跑步、骑行或者开车。

To use this API,
you create an activity manager
and start listening for activity updates
using the `startActivityUpdates` method.
Each time the device updates the motion activity,
it executes the specified closure,
passing a `CMMotionActivity` object.

要使用这个 API，首先创建一个活动管理器，然后使用 `startActivityUpdates` 方法来开始监听活动更新。每当设备更新了运动相关活动，它就会执行指定的闭包，并传入一个 `CMMotionActivity` 对象。

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

`CMMotionActivityManager` 由 Core Motion 框架提供。支持 Core Motion 的设备都装备了一个运动辅助处理器。通过使用专用硬件，系统可以将所有传感器处理工作从 CPU 上移除掉，并减少电量的使用。

The first of the _M-series_ coprocessors was the M7,
which arrived in September 2013 with the iPhone 5S.
This coincided with the release of iOS 7 and the Core Motion APIs.

第一款 **M 系列**辅助处理器是 M7，在 2013 年九月与 iPhone 5S 一同面世。iOS 7 和 Core Motion API 也与此同时发布。

## Feature Drivers
## 司机功能

Possibly the most well-known use of motion activities is
["Do Not Disturb While Driving"](https://support.apple.com/en-us/HT208090),
added in iOS 11.
Automotive detection got much better with the introduction of this feature.

可能运动活动最广为人知的使用是 iOS 11 中新增的[「驾驶勿扰」](https://support.apple.com/zh-cn/HT208090)功能。这项功能推出后，对于开车的检测变好了很多。

> At low speeds, it's hard to distinguish automobile travel from other means
> using accelerometer data alone.
> Although we can only speculate as to how this works,
> it's possible that the iPhone uses magnetometer data from the device compass.
> Because cars and other vehicles are often enclosed in metal,
> electromagnetic flux is reduced.

> 在速度比较低的情况下，只使用加速规的数据是很难区分驾车行驶和其他活动的。我们只能猜测，有可能 iPhone 是使用磁强计的数据来实现这项功能的。因为汽车和其他机动车辆通常被金属围绕着，电磁通量会被减少。

Beyond safety concerns,
some apps might change their behavior
according to the current mode of transportation.
For example,
a delivery service app might relay changes in motion activity to a server
to recalculate estimated ETA or change the UI to communicate
that the courier has parked their vehicle and are now approaching by foot.

除了安全相关的考虑，一些应用可能会根据当前的交通模式改变行为。比如，一个快递应用可能会发送运动活动改变到服务器来重新计算预计到达时间，或者更新 UI 来表明快递员已经停下了车子，正在走路接近。

## Traveling Without Moving

`CMMotionActivity` has Boolean properties
for each of the different types of motion
as well as one for whether the device is stationary.
This seems counter-intuitive,
as logic dictates that
you can either be walking or driving a car at a given moment,
but not both.

This point is clarified by the [`CMMotionActivity` documentation](https://developer.apple.com/documentation/coremotion/cmmotionactivity):

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

**Scenario 1**:
You're in a car stopped at a red light

| 🚶‍ `walking` | 🏃‍ `running` | 🚴‍`cycling` | 🚗 `automotive` | 🛑 `stationary` |
| ------------- | ------------- | ------------ | --------------- | --------------- |
| `false`       | `false`       | `false`      | `true`          | `true`          |

**Scenario 2**:
You're in a moving vehicle

| 🚶‍ `walking` | 🏃‍ `running` | 🚴‍`cycling` | 🚗 `automotive` | 🛑 `stationary` |
| ------------- | ------------- | ------------ | --------------- | --------------- |
| `false`       | `false`       | `false`      | `true`          | `false`         |

**Scenario 3**:
The device is in motion, but you're neither walking nor in a moving vehicle

| 🚶‍ `walking` | 🏃‍ `running` | 🚴‍`cycling` | 🚗 `automotive` | 🛑 `stationary` |
| ------------- | ------------- | ------------ | --------------- | --------------- |
| `false`       | `false`       | `false`      | `false`         | `false`         |

**Scenario 4**:
You're a world-famous detective, who,
in the process of chasing a suspect down the corridors of a moving train,
has reached the last car and has stopped to look around
to surmise where they're hiding
_(perhaps that conspicuous, person-sized box in the corner?)_

| 🚶‍ `walking` | 🏃‍ `running` | 🚴‍`cycling` | 🚗 `automotive` | 🛑 `stationary` | 🕵️‍🇧🇪 `poirot` |
| ------------- | ------------- | ------------ | --------------- | --------------- | --------------- |
| `false`       | `true`        | `false`      | `true`          | `true`          | `true`          |

(We're actually not sure what would happen in that last scenario...)

The overall guidance from the documentation is that
you should treat `stationary` to be orthogonal from
all of the other `CMMotionActivity` properties
and that you should be prepared to handle all other combinations.

Each `CMMotionActivity` object also includes a `confidence` property
with possible values of `.low`, `.medium`, and `.high`.
Unfortunately, not much information is provided by the documentation
about what any of these values mean, or how they should be used.
As is often the case,
an empirical approach is recommended;
test your app in the field to
see when different confidence values are produced
and use that information to determine the correct behavior for your app.

## Combining with Location Queries

Depending on your use case,
it might make sense to coordinate Core Motion readings with
[Core Location](https://nshipster.com/core-location-in-ios-8/) data.

You can combine changes in location over time
with low-confidence motion activity readings
to increase accuracy.
Here are some general guidelines for typical ranges of speeds
for each of the modes of transportation:

- Walking speeds typically peak at 2.5 meters per second (5.6 mph, 9 km/h)
- Running speeds range from 2.5 to 7.5 meters per second (5.6 – 16.8 mph, 9 – 27 km/h)
- Cycling speeds range from 3 to 12 meters per second (6.7 – 26.8 mph, 10.8 – 43.2 km/h)
- Automobile speeds can exceed 100 meters per second (220 mph, 360 km/h)

Alternatively, you might use location data to change the UI
depending on whether the current location is in a body of water.

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

---

`CMMotionActivityManager` is one of many great APIs in Core Motion
that you can use to build immersive, responsive apps.

If you haven't considered the potential of
incorporating device motion into your app
(or maybe haven't looked at Core Motion for a while),
you might be surprised at what's possible.
