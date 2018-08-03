---
title: "密码规则 / UITextInputPasswordRules"
author: Mattt
translator: Bei Li
category: "Cocoa"
excerpt:
    Unless it's the title of a hacker movie from the 90's
    or the solution to an escape room puzzle,
    a password should be utterly devoid of meaning.

    密码应该完全没有任何意义，除非它是一个 90 年代骇客电影的标题或者一个密室逃脱游戏的答案。
hiddenlang: ""
status:
    swift: "4.2"
---

It's no wonder why hipsters obsess over artisanal _this_ and handcrafted _that_.
Whether it's a thick slice of avocado toast,
a bottle of small batch (nondairy) golden milk,
or a perfect cup of pour-over coffee ---
there's no replacement for a human touch.

也难怪 hipster 们着迷于工艺品和手工制品。不管是一片厚切鳄梨吐司、一瓶限量（非乳制）姜黄奶或一杯完美的手冲咖啡——其中的人情味是无法替代的。

In stark contrast,
good passwords are the opposite of artisanal.
Unless it's the title of a hacker movie from the 90's
or the solution to an escape room puzzle,
a password should be utterly devoid of meaning.

相反，好密码和工艺品截然不同。密码应该完全没有任何意义，除非它是一个 90 年代骇客电影的标题或者一个密室逃脱游戏的答案。

With Safari in iOS 12 and macOS Mojave,
it'll be easier than ever to generate
the strongest,
most meaningless,
most impossible-to-guess passwords imaginable ---
all thanks to a few new features.

有了 iOS 12 和 macOS Mojave 中的 Safari，生成可以想象到的最强、最没有意义、最难猜到的密码从未如此简单——这都要感谢一些新功能。

---

An ideal password policy is simple:
Enforce a minimum number of characters (at least 8)
and allow for longer passwords (64 or more).

理想的密码策略非常简单：强制要求最少字符数（至少 8 位）并且允许长密码（64 位或者更多）。

Anything more elaborate, be it
pre-selected security questions,
periodic password expiration,
or arcane character requirements
do little more than annoy the people these policies try to protect.

其他更复杂的策略，像预置的安全问题、周期性的失效密码或者强制要求一些奇怪的符号，只不过让这些策略想要保护人感到厌烦。

> But don't take my word for it ---
> I'm not a security expert.
>
> Instead,
> check out the latest
> [Digital Identity Guidelines](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-63b.pdf)
> from
> <abbr title="National Institute of Standards and Technology">NIST</abbr>
> (published June 2017).

> 但是不要太相信我说的话——我不是安全专家
>
> 相对的，请查看美国国家标准技术研究所最新发布（2017 年 6 月）的 [Digital Identity Guidelines](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-63b.pdf)

The good news is that more companies and organizations
are starting to pay attention to security best practices.
The bad news is that it took
a series of massive data breaches affecting millions of people
in order for things to change.
And the ugly truth is that
because corporations and governments take forever to do anything,
many of the aforementioned security anti-patterns aren't going away anytime soon.

好消息是越来越多的公司和组织开始注意安全性最佳实践了。坏消息则是改变这些事情需要进行一系列影响数百万人的大范围数据改动。事实上前面说到的安全性反面模式并不会很快消失，因为公司和政府做任何事情都需要花很久的时间。

## Automatic Strong Passwords
## 自动强密码（生成）

Safari AutoFill has been able to generate passwords since iOS 8,
but one of its shortcomings was that it couldn't guarantee
that a generated password satisfied the requirements of a particular service.

Safari 的自动填充从 iOS 8 起就可以生成密码了，但是它有一个缺点就是不能保证生成的密码符合某些服务的要求。

Apple aims to solve this problem with a new Automatic Strong Passwords feature
in Safari, iOS 12, and macOS Mojave.

Apple 通过 iOS 12 和 macOS Mojave 里 Safari 中的自动强密码（生成）功能来解决这个问题。

WebKit engineer Daniel Bates submitted
[this proposal](https://github.com/whatwg/html/issues/3518)
for consideration to the
<abbr title="Web Hypertext Application Technology Working Group">WHATWG</abbr>
on March 1st.
On June 6th,
the WebKit team
[announced Safari Technology Preview Release 58](https://webkit.org/blog/8327/safari-technology-preview-58-with-safari-12-features-is-now-available/),
with support for strong password generation
using the new `passwordrules` attribute.
This announcement coincided with the release iOS 12 beta SDKs at WWDC,
which included a new `UITextInputPasswordRules` API,
along with a number of other password management features,
including Security Code AutoFill and federated authentication.

WebKit 工程师 Daniel Bates 在 3 月 1 日给 <acronym title="Web Hypertext Application Technology Working Group">WHATWG</acronym> 提交了[这个提案](https://github.com/whatwg/html/issues/3518)。6 月 6 日，WebKit 团队[发布了 Safari Technology Preview 58](https://webkit.org/blog/8327/safari-technology-preview-58-with-safari-12-features-is-now-available/)，使用新属性 `passwordrules` 来支持强密码生成。同时，WWDC 发布了 iOS 12 beta SDK，包括新的 `UITextInputPasswordRules` API，还有验证码自动输入和联合身份验证等其他一些密码管理功能。

## Password Rules
## 密码规则

Password rules are like a recipe for password generators.
By following a few simple rules,
the password generator can randomly generate new, secure passwords
that comply with the particular requirements of the service provider.

密码规则就像是密码生成器的配方。根据一些简单的规则，密码生成器就可以随机生成满足服务提供方需求的新密码。

Password rules consist of one or more key-value pairs
in the following form:

密码规则由一个或多个键值对组成：

`required: lower; required: upper; required: digit; allowed: ascii-printable; max-consecutive: 3;`

### Keys
### 键

Each rule may specify one of the following keys:

每个规则可以指定下列键：

- `required`: The kinds of characters that are required
- `required`: 需要的字符类型
- `allowed`: The kinds of characters that are allowed
- `allowed`: 允许使用的字符类型
- `max-consecutive`: The maximum number of consecutive characters allowed
- `max-consecutive`: 允许字符连续出现次数的最大值
- `minlength`: The minimum password length
- `minlength`: 密码最小长度
- `maxlength`: The maximum password length
- `maxlength`: 密码最大长度

The `required` and `allowed` keys
have one of the character classes listed below as their value.
The `max-consecutive`, `minlength`, and `maxlength` keys
have a nonnegative integer as their value.

`required` 和 `allowed` 键使用下面列出的字符类别作为值。`max-consecutive`、`minlength` 和 `maxlength` 使用非负整数作为值。

### Character Classes
### 字符类别

The `required` and `allowed` keys
may have any one of the following named character classes
as their value.

`required` 和 `allowed` 键可以使用下面的字符类别作为值：

- `upper` (`A-Z`)
- `lower` (`a-z`)
- `digits` (`0-9`)
- `special` (`` -~!@#$%^&\*\_+=`|(){}[:;"'<>,.? ] `` and space)
- `special` (`` -~!@#$%^&\*\_+=`|(){}[:;"'<>,.? ] `` 和空格)
- `ascii-printable` (U+0020 — 007f)
- `unicode` (U+0 — 10FFFF)

In addition to these presets,
you may specify a custom character class
with ASCII characters surrounded by square brackets
(for example, `[abc]`).

除了这些预置字符类别，还可以用方括号包住 ASCII 字符来指定自定义字符类别（比如 `[abc]`）。

---

Apple's
[Password Rules Validation Tool](https://developer.apple.com/password-rules/)
allows you to experiment with different rules
and get real-time feedback of their results.
You can even generate and download passwords by the thousands
to use during development and testing!

Apple 的 [Password Rules Validation Tool](https://developer.apple.com/password-rules/) 让你可以对不同的规则进行实验，并得到实时的结果反馈。甚至可以生成并下载上千个密码用来开发和测试。

{% asset password-rules-validation-tool.png alt="Password Rules Validation Tool" %}

For more information about Password Rules syntax,
check out Apple's
["Customizing Password AutoFill Rules"](https://developer.apple.com/documentation/security/password_autofill/customizing_password_autofill_rules).

更多有关于密码规则的语法，请查看 Apple 的文档[「Customizing Password AutoFill Rules」](https://developer.apple.com/documentation/security/password_autofill/customizing_password_autofill_rules)。

---

## Specifying Password Rules
## 指定密码规则

On iOS,
you set the `passwordRules` property of a `UITextField`
with a `UITextInputPasswordRules` object
(you should also set the `textContentType` to `.newPassword` while you're at it):

在 iOS 上，给 `UITextField` 的 `passwordRules` 属性设置一个 `UITextInputPasswordRules` 对象（同时也应该将 `textContentType` 属性设置为 `.newPassword`）：

```swift
let newPasswordTextField = UITextField()
newPasswordTextField.textContentType = .newPassword
newPasswordTextField.passwordRules = UITextInputPasswordRules(descriptor: "required: upper; required: lower; required: digit; max-consecutive: 2; minlength: 8;")
```

On the web,
you set the `passwordrules` attribute
to an `<input>` element with `type="password"`:

在网页上，设置 `<input>` 元素（且 `type="password"`）的 `passwordrules` 属性：

```html
<input type="password" passwordrules="required: upper; required: lower; required: special; max-consecutive: 3;"/>
```

> If unspecified,
> the default password rule is
> `allowed: ascii-printable`.
> Though if your form has a password confirmation field,
> it will automatically follow the rules from the preceding field.

> 如果没有指定，默认的密码规则是 `allowed: ascii-printable`。如果表单中有密码验证区域，它的密码规则会从上一个区域继承下来。

## Generating Password Rules in Swift
## 在 Swift 中生成密码规则

If the thought of working with a string-based format
without a proper abstraction gives you the heebie-jeebies,
you're not alone.

不光是只有你会觉得直接使用没有良好抽象的字符串格式令人感到不安。

Here's one way to encapsulate Password Rules in a Swift API
([also available as a Swift package](https://github.com/NSHipster/PasswordRules)):

下面是一种将密码规则封装成 Swift API 的方式（[可以作为 Swift package 获取](https://github.com/NSHipster/PasswordRules)）：

```swift
enum PasswordRule {
    enum CharacterClass {
        case upper, lower, digits, special, asciiPrintable, unicode
        case custom(Set<Character>)
    }

    case required(CharacterClass)
    case allowed(CharacterClass)
    case maxConsecutive(UInt)
    case minLength(UInt)
    case maxLength(UInt)
}

extension PasswordRule: CustomStringConvertible {
    var description: String {
        switch self {
        case .required(let characterClass):
            return "required: \(characterClass)"
        case .allowed(let characterClass):
            return "allowed: \(characterClass)"
        case .maxConsecutive(let length):
            return "max-consecutive: \(length)"
        case .minLength(let length):
            return "minlength: \(length)"
        case .maxLength(let length):
            return "maxlength: \(length)"
        }
    }
}

extension PasswordRule.CharacterClass: CustomStringConvertible {
    var description: String {
        switch self {
        case .upper: return "upper"
        case .lower: return "lower"
        case .digits: return "digits"
        case .special: return "special"
        case .asciiPrintable: return "ascii-printable"
        case .unicode: return "unicode"
        case .custom(let characters):
            return "[" + String(characters) + "]"
        }
    }
}
```

With this in place,
we can now specify a series of rules in code
and use them to generate a string with valid password rules syntax:

有了这个，我们就可以在代码里指定一些规则，然后用它们生成有效的密码规则语法字符串：

```swift
let rules: [PasswordRule] = [ .required(.upper),
                              .required(.lower),
                              .required(.special),
                              .minLength(20) ]

let descriptor = rules.map{ "\($0.description);" }
                      .joined(separator: " ")

// "required: upper; required: lower; required: special; max-consecutive: 3;"
```

If you feel so inclined,
you could even extend `UITextInputPasswordRules`
to provide a convenience initializer
that takes an array of `PasswordRule` values:

只要你愿意，你甚至可以扩展 `UITextInputPasswordRules` 给它添加一个接收 `PasswordRule` 数组的 convenience initializer。

```swift
extension UITextInputPasswordRules {
    convenience init(rules: [PasswordRule]) {
        let descriptor = rules.map{ $0.description }
                              .joined(separator: "; ")

        self.init(descriptor: descriptor)
    }
}
```

---

If you're the sentimental type when it comes to personal credentials,
and enjoy name dropping your college or dog or favorite sports team
behind the anonymous bullets of password input fields,
please consider reforming your ways.

如果你是一个在个人认证信息上非常有感情的人，喜欢在密码输入区域中的小圆点后面输入你的大学、小狗或者最喜欢的运动团队，请考虑不要再这样做了。

Speaking personally,
I can't imagine going about my day-to-day without my password manager.
It's hard to overstate the peace of mind you get
by knowing that any information you ever needed
is accessible to you --- and only you --- whenever you need it.

就我个人来说，我无法想象没有密码管理器的日子。当你知道任何时候你都能访问到你需要的信息，并且只有你能访问到时，你的心灵将会获得极大的平静。

By taking this step now,
you'll be able to take full advantage of these improvements
coming to Safari when iOS 12 and macOS Mojave arrive later this year.

从现在开始改变，你就能完全利用上在之后今年发布的 iOS 12 和 macOS Mojave 的 Safari 中的这些改进。
