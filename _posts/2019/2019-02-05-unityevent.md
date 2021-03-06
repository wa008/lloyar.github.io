---
layout: article
title: Hello C#（二）
date: '2019-02-05 10:40:00 +08:00'
key: '2019-02-05_10:40'
tags:
  - Unity
  - C#
  - Language
---

本文讨论了 C# delegate 、C# Event 和 UnityEvent 的关系。

<!--more-->

## 前置知识点

请一定要先食用下面这篇博文，十分清晰简洁的讲解了委托、事件和 UnityEvent 的概念。

[Look this.](https://blog.csdn.net/qq_28849871/article/details/78366236)

## 补充

### 委托的存在意义

我并不愿意拿发布者、订阅者来举例子。相比于把委托和事件这种语法机制放到具体的设计模式（观察者模式）中进行理解，我认为通过语言本身来理解会更加好，因为这更加贴近根源，而且也不会对用法（观察者模式）的灵活性产生影响。

委托可理解为方法的容器，它最重要的一个特点是可以容纳多个方法，并进行多播。那么是不是说，委托对单一方法对调用，可以被直接替换为调用那个方法，而不是间接调用呢？其实是可以的，那这么一来，貌似委托并没有为我们提供多少便利，那是不是意味着单一方法的引用没用存在的必要？

其实并不是这样的，多了一层抽象，可以使得用法更加灵活，而且更加安全。

> 创建了委托实例后，有关它的一切就不能改变。这样一来，就可以安全地传递委托实例的引用，并把它们与其他委托实例合并，同时不必担心一致性、线程安全性或者是否有其他人试图更改它。 在这一点上， 委托实例和`string`是一样的。`string` 的实例也是不易变的。之所以提到 `string` ，是因为 `Delegate.Combine` 和 `String.Concat` 很像——都是合并现有的实例来形成一个新实例，同时根本不更改原始对象。对于委托实例，原始调用列表被连接到一起。注意，如果试图将 `null` 和委托实例合并到一起， `null` 将被视为带有空调用列表的一个委托。

### 委托的多播与事件

接着我们谈谈委托的多播，这个就很有意思了，这是实现观察者模式的重要特性。一个委托实例的创建，意味着我们创建了一个空盒子，而且指定了只有特定种类的东西才能放进去（由委托的声明描述）。而其他类，可以往空盒子里面塞方法，等到合适的场合，打开盒子，一次性调用里面所有的方法。

至于事件，可是说是对委托多播的扩展，我们可以把事件看做类似于属性（property）的东西是很有好处的。首先，两者都声明为具有一种特定的类型。对于事件来说，必须是一个委托类型。

使用属性时，感觉就像是直接对它的字段进行取值和赋值，但你实际是在调用方法，也就是取值方法和赋值方法。实现属性时，可以在那些方法中做任何事情。但凑巧的是，大多数属性都只是实现了简单的字段，有时会在赋值方法中添加一些校验机制，有时则会添加一些线程安全性。

同样，在订阅或取消订阅一个事件时，看起来就像是在通过+=和-=运算符使用委托类型的字段。但和属性的情况一样，这个过程实际是在调用方法（add和remove方法）。对于一个纯粹的事件，你所能做的事情就是订阅（添加一个事件处理程序）或者取消订阅（删除一个事件处理程序） 。最终是由事件方法来做真正有用的事情，如找到你试图添加和删除的事件处理程序，并使它们在类中的其他地方可用。

“事件”存在的首要理由和“属性”差不多——它们添加了一个封装层，实现发布/订阅模式（publish/subscribe pattern）。通常，我们不希望其他代码能直接设置字段值；最起码也要先由所有者（owner）对新值进行验证。

同样，我们通常不希望类外部的代码随意更改（或调用）一个事件的处理程序。当然，类能通过添加方法的方式来提供额外的访问。例如，可以重置事件的处理程序列表，或者引发事件（也就是调用它的事件处理程序） 。例如，BackgroundWorker.OnProgressChanged只是调用了ProgressChanged事件的处理程序。然而，如果只对外揭示事件本身，类外部的代码就只能添加和删除事件处理程序。
类内的代码能看见字段；类外的代码只能看见事件。这样一来，表面上似乎能调用一个事件，但为了调用事件处理程序，实际做的事情是调用存储在字段中的委托实例。

### UnityEvent 的两个调用版本（静态和动态）

  [Look this ](https://docs.unity3d.com/Manual/UnityEvents.html)

> * Static. Static calls are preconfigured calls, with preconfigured values that are set in the UI
. This means that when the callback is invoked, the target function is invoked with the argument that has been entered into the UI.
* Dynamic. Dynamic calls are invoked using an argument that is sent from code, and this is bound to the type of UnityEvent that is being invoked. The UI filters the callbacks and only shows the dynamic calls that are valid for the UnityEvent.

  [and this.](https://forum.unity.com/threads/unityaction-unityevent-parameters-and-serialization.469240/)

  > There are Static versions of the call and then there are Dynamic versions of the same call. you are describing the static version. The static version allows any type of UnityEvent (not just UnityEvent<int>) to call a function that takes a int, string, bool, etc. The dynamic version does what I believe you are expecting (from the quoted sentence, not your initial issue). Look closer that the functions listed in the dropdown and you should see two methods listed "target(int)". One of them static, the other dynamic
