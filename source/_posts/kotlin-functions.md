---
title: 当 Kotlin 中的监听器包含多个方法时，如何让它 “巧夺天工”？
date: 2018-12-22 18:06:44
tags: Kotlin
---
> * 原文地址：[Listeners with several functions in Kotlin. How to make them shine?](https://antonioleiva.com/listeners-several-functions-kotlin/)
> * 原文作者：[Antonio Leiva](https://antonioleiva.com)
> * 译文出自：[掘金翻译计划](https://github.com/xitu/gold-miner)
> * 本文永久链接：[https://github.com/xitu/gold-miner/blob/master/TODO1/listeners-several-functions-kotlin.md](https://github.com/xitu/gold-miner/blob/master/TODO1/listeners-several-functions-kotlin.md)
> * 译者：[Moosphon](https://github.com/Moosphan)
> * 校对者：[Qiuk17](https://github.com/Qiuk17), [zx-Zhu](https://github.com/zx-Zhu)

# 当 Kotlin 中的监听器包含多个方法时，如何让它 “巧夺天工”？

![](https://user-gold-cdn.xitu.io/2018/12/22/167d6372f9869079?w=880&h=440&f=jpeg&s=71887)

我经常遇到的一个问题是在使用 Kotlin 时如何简化具有多个方法的监听器的交互。对于具有只具有一个方法的监听器（或任何接口）很简单：Kotlin 会自动让您用 lambda 替换它。但对于具有多个方法的监听器来说，情况并非如此。

因此，在本文中，我想向您展示处理问题的不同方法，您甚至可以在途中学习一些[新的 Kotlin 技巧](https://antonioleiva.com/kotlin-awesome-tricks-for-android/)！<!--more-->

## 问题所在

当我们处理监听器时，我们知道 `OnclickListener` 作用于视图，归功于 Kotlin 对 Java 库的优化，我们可以将以下代码：

```
view.setOnClickListener(object : View.OnClickListener {
    override fun onClick(v: View?) {
        toast("View clicked!")
    }
})
```

转化为这样：

```
view.setOnClickListener { toast("View clicked!") }
```

问题在于，当我们习惯它时，我们希望它能够无处不在。然而当接口存在多个方法时，这种做法将不再适用。

例如，如果我们想为视图动画设置一个监听器，我们最终得到以下“漂亮”的代码：

```
view.animate()
        .alpha(0f)
        .setListener(object : Animator.AnimatorListener {
            override fun onAnimationStart(animation: Animator?) {
                toast("Animation Start")
            }

            override fun onAnimationRepeat(animation: Animator?) {
                toast("Animation Repeat")
            }

            override fun onAnimationEnd(animation: Animator?) {
                toast("Animation End")
            }

            override fun onAnimationCancel(animation: Animator?) {
                toast("Animation Cancel")
            }
        })
```

你可能会反驳说 Android framework 已经为它提供了一个解决方案：适配器。对于几乎任何具有多个方法的接口，它们都提供了一个抽象类，将所有方法实现为空。在上述例子中，您可以这样：

```
view.animate()
        .alpha(0f)
        .setListener(object : AnimatorListenerAdapter() {
            override fun onAnimationEnd(animation: Animator?) {
                toast("Animation End")
            }
        })
```

好的，是改善了一些，但这存在几个问题：

*   适配器是类，这意味着如果我们想要一个类作为此适配器的实现，它不能扩展其他任何东西。
*   我们把一个本可以用 lambda 清晰表达的事物，变成了一个具有一个方法的匿名对象。

我们有什么选择？

## Kotlin 中的接口：它们可以包含代码

还记得我们谈到 Kotlin 中的接口吗？ 它们内部可以包含代码，因此，您能够声明可以实现而不是继承适配器（以防您现在将其用于 Android 开发中，您可以使用 Java 8 和接口中的默认方法执行相同的操作）：

```
interface MyAnimatorListenerAdapter : Animator.AnimatorListener {
    override fun onAnimationStart(animation: Animator) = Unit
    override fun onAnimationRepeat(animation: Animator) = Unit
    override fun onAnimationCancel(animation: Animator) = Unit
    override fun onAnimationEnd(animation: Animator) = Unit
}
```

有了这个，默认情况下所有方法都不会执行任何操作，这意味着一个类可以实现此接口并仅声明它所需的方法：

```
class MainActivity : AppCompatActivity(), MyAnimatorListenerAdapter {
    ...
    override fun onAnimationEnd(animation: Animator) {
        toast("Animation End")
    }
}
```

之后，您可以将它作为监听器的参数：

```
view.animate()
        .alpha(0f)
        .setListener(this)
```

这个方案解决了开始时提出的一个问题，但是我们仍然要显式地声明它。如果我想使用 lambda 表达式呢？

此外，虽然这可能会不时地使用继承，但在大多数情况下，您仍将使用匿名对象，这与使用 framework 适配器并无不同。

但是，这是一个有趣的想法：如果你需要为具有多个方法的监听器定义一种适配器，**那么最好使用接口而不是抽象类**。[继承 FTW 的构成](https://en.wikipedia.org/wiki/Composition_over_inheritance)。

## 一般情况下的扩展功能

让我们转向更加简洁的解决方案。可能会碰到这种情况（如上所述）：大多数时候你只需要相同的功能，而对另一个功能则不太感兴趣。对于 `AnimatorListener`，最常用的一个方法通常是 `onAnimationEnd`。那么为什么不创建一个涵盖这种情况的[扩展方法](https://antonioleiva.com/extension-functions-kotlin/)呢？

```
view.animate()
        .alpha(0f)
        .onAnimationEnd { toast("Animation End") }
```

真棒！扩展函数应用于 `ViewPropertyAnimator`，这是 `animate()`、`alpha` 和所有其他动画方法返回的内容。

```
inline fun ViewPropertyAnimator.onAnimationEnd(crossinline continuation: (Animator) -> Unit) {
    setListener(object : AnimatorListenerAdapter() {
        override fun onAnimationEnd(animation: Animator) {
            continuation(animation)
        }
    })
}
```

> 我[之前已经谈过 `内联`](https://antonioleiva.com/lambdas-kotlin/)，但如果你还有一些疑问，我建议你看一下[官方的文档](https://kotlinlang.org/docs/reference/inline-functions.html)。

如您所见，该函数只接收在动画结束时调用的 lambda。这个扩展函数为我们完成了创建适配器并调用 setListener 这种不友好的工作。

这样就好多了！我们可以在监听器中为每个方法创建一个扩展方法。但在这种特殊情况下，我们遇到了动画只接受一个监听器的问题。因此我们一次只能使用一个。

在任何情况下，对于大多数重复的情况（像上面那样），它并不会损害到像如上提到的 `Animator` 本身的方法。这是更简单的解决方案，非常易于阅读和理解。

## 使用命名参数和默认值

但是你和我喜欢 Kotlin 的原因之一是它有很多令人惊奇的功能来简化我们的代码！所以你可以想象我们还有一些选择的余地。接下来我们将使用命名参数：这允许我们定义 lambda 表达式并明确说明它们的用途，这将极大地提高代码的可读性。

我们会有类似于上面的功能，但涵盖所有方法的情况：

```
inline fun ViewPropertyAnimator.setListener(
        crossinline animationStart: (Animator) -> Unit,
        crossinline animationRepeat: (Animator) -> Unit,
        crossinline animationCancel: (Animator) -> Unit,
        crossinline animationEnd: (Animator) -> Unit) {

    setListener(object : AnimatorListenerAdapter() {
        override fun onAnimationStart(animation: Animator) {
            animationStart(animation)
        }

        override fun onAnimationRepeat(animation: Animator) {
            animationRepeat(animation)
        }

        override fun onAnimationCancel(animation: Animator) {
            animationCancel(animation)
        }

        override fun onAnimationEnd(animation: Animator) {
            animationEnd(animation)
        }
    })
}
```

方法本身不是很好，但通常是伴随扩展方法的情况。他们隐藏了 framework 不好的部分，所以有人必须做艰苦的工作。现在您可以像这样使用它：

```
view.animate()
        .alpha(0f)
        .setListener(
                animationStart = { toast("Animation start") },
                animationRepeat = { toast("Animation repeat") },
                animationCancel = { toast("Animation cancel") },
                animationEnd = { toast("Animation end") }
        )
```

感谢命名参数，让我们可以很清楚这里发生了什么。

你需要确保没有命名参数的时候就不要使用它，否则它会变得有点乱：

```
view.animate()
        .alpha(0f)
        .setListener(
                { toast("Animation start") },
                { toast("Animation repeat") },
                { toast("Animation cancel") },
                { toast("Animation end") }
        )
```

无论如何，这个解决方案仍然迫使我们实现所有方法。但它很容易解决：只需使用[参数的默认值](https://antonioleiva.com/kotlin-android-extension-functions/)。空的 lambda 表达式将上面的代码演变成：

```
inline fun ViewPropertyAnimator.setListener(
        crossinline animationStart: (Animator) -> Unit = {},
        crossinline animationRepeat: (Animator) -> Unit = {},
        crossinline animationCancel: (Animator) -> Unit = {},
        crossinline animationEnd: (Animator) -> Unit = {}) {

    ...
}
```

现在你可以这样做：

```
view.animate()
        .alpha(0f)
        .setListener(
                animationEnd = { toast("Animation end") }
        )
```

还不错，对吧？虽然比之前的做法要稍微复杂一点，但却更加灵活了。

## 杀手锏操作：DSL

到目前为止，我一直在解释简单的解决方案，诚实地说可能涵盖大多数情况。但如果你想发疯，你甚至可以创建一个让事情变得更加明确的小型 DSL。

这个想法 [来自 Anko 如何实现一些侦听器](https://github.com/Kotlin/anko/blob/master/anko/library/generated/sdk23-listeners/src/Listeners.kt)，它是创建一个实现了一组接收 lambda 表达式的方法帮助器。这个 lambda 将在接口的相应实现中被调用。我想首先向您展示结果，然后解释使其实现的代码：

```
view.animate()
        .alpha(0f)
        .setListener {
            onAnimationStart {
                toast("Animation start")
            }
            onAnimationEnd {
                toast("Animation End")
            }
        }
```

看到了吗？ 这里使用了一个小型的 DSL 来定义动画监听器，我们只需调用我们需要的功能即可。对于简单的行为，这些方法可以是单行的：

```
view.animate()
        .alpha(0f)
        .setListener {
            onAnimationStart { toast("Start") }
            onAnimationEnd { toast("End") }
        }
```

这相比于之前的解决方案有两个优点：

*   **它更加简洁**：您在这里保存了一些特性，但老实说，仅仅因为这个还不值得努力。
*   **它更加明确**：它迫使开发人员说出他们所重写的功能。在前一个选择中，由开发人员设置命名参数。这里没有选择，只能调用该方法。

所以它本质上是一个不太容易出错的解决方案。

现在来实现它。首先，您仍需要一个扩展方法：

```
fun ViewPropertyAnimator.setListener(init: AnimListenerHelper.() -> Unit) {
    val listener = AnimListenerHelper()
    listener.init()
    this.setListener(listener)
}
```

这个方法只获取一个[带有接收器的 lambda 表达式](https://tech.io/playgrounds/6973/kotlin-function-literal-with-receiver)，它应用于一个名为 `AnimListenerHelper` 的新类。它创建了这个类的一个实例，使它调用 lambda 表达式，并将实例设置为监听器，因为它正在实现相应的接口。让我们看看如何实现 `AnimeListenerHelper`：

```
class AnimListenerHelper : Animator.AnimatorListener {
    ...
}
```

然后对于每个方法，它需要：

*   保存 lambda 表达式的属性
*   DSL 方法，它接收在调用原始接口的方法时执行的 lambda 表达式
*   在原有接口基础上重写方法

```
private var animationStart: AnimListener? = null

fun onAnimationStart(onAnimationStart: AnimListener) {
    animationStart = onAnimationStart
}

override fun onAnimationStart(animation: Animator) {
    animationStart?.invoke(animation)
}
```

这里我使用的是 `AnimListener` 的一个 [类型别名](https://kotlinlang.org/docs/reference/type-aliases.html)：

```
private typealias AnimListener = (Animator) -> Unit
```

这里是完整的代码：

```
fun ViewPropertyAnimator.setListener(init: AnimListenerHelper.() -> Unit) {
    val listener = AnimListenerHelper()
    listener.init()
    this.setListener(listener)
}

private typealias AnimListener = (Animator) -> Unit

class AnimListenerHelper : Animator.AnimatorListener {

    private var animationStart: AnimListener? = null

    fun onAnimationStart(onAnimationStart: AnimListener) {
        animationStart = onAnimationStart
    }

    override fun onAnimationStart(animation: Animator) {
        animationStart?.invoke(animation)
    }

    private var animationRepeat: AnimListener? = null

    fun onAnimationRepeat(onAnimationRepeat: AnimListener) {
        animationRepeat = onAnimationRepeat
    }

    override fun onAnimationRepeat(animation: Animator) {
        animationRepeat?.invoke(animation)
    }

    private var animationCancel: AnimListener? = null

    fun onAnimationCancel(onAnimationCancel: AnimListener) {
        animationCancel = onAnimationCancel
    }

    override fun onAnimationCancel(animation: Animator) {
        animationCancel?.invoke(animation)
    }

    private var animationEnd: AnimListener? = null

    fun onAnimationEnd(onAnimationEnd: AnimListener) {
        animationEnd = onAnimationEnd
    }

    override fun onAnimationEnd(animation: Animator) {
        animationEnd?.invoke(animation)
    }
}
```

最终的代码看起来很棒，但代价是做了很多工作。

## 我该使用哪种方案？

像往常一样，这要看情况。**如果您不在代码中经常使用它，我会说哪种方案都不要使用**。在这些情况下要根据实际情况而定，如果你要编写一次监听器，只需使用一个实现接口的匿名对象，并继续编写重要的代码。

如果您发现需要使用更多次监听器，请使用其中一种解决方案进行重构。我通常会选择只使用我们感兴趣的功能进行简单的扩展。如果您需要多个监听器，请评估两种最新替代方案中的哪一种更适合您。像往常一样，这取决于你将要如何广泛地使用它。

希望这篇文章能够在您下一次处于这种情况下时帮助到您。**如果您以不同方式解决此问题，请在评论中告诉我们！**

感谢您的阅读 🙂

> 如果发现译文存在错误或其他需要改进的地方，欢迎到 [掘金翻译计划](https://github.com/xitu/gold-miner) 对译文进行修改并 PR，也可获得相应奖励积分。文章开头的 **本文永久链接** 即为本文在 GitHub 上的 MarkDown 链接。
