---
title: Android Gradle3.0开始Implementation 与 Api的爱恨情仇
date: 2020-08-19 13:48:35
tags: Gradle
categories: Android Gradle
toc: true
---
本文翻译于 Bedanta Bikash Borah 的文章，原文链接如下：

> ***[Implementation Vs Api in Android Gradle plugin 3.0](https://medium.com/mindorks/implementation-vs-api-in-gradle-3-0-494c817a6fa)***

![1*-3BcPdB6bl6Q6TcFWimcTA](https://miro.medium.com/max/700/1*-3BcPdB6bl6Q6TcFWimcTA.png)

当我们在Android项目中使用 Gradle 3.0 及以上版本的插件，你一定会注意到 `compile` 关键字已经被弃用来支持 `implementation` 和 `api`。让我们借助一个例子来了解它们。

示例应用 (Kotlin) 可以在[这里](https://github.com/iamBedant/ApiVsImplementation)找到。<!-- more -->

让我们假设有一个 Android 项目包含以下四个 library module：

- LibraryA
- LibraryB
- LibraryC
- LibraryD

它们之间的依赖关系如下所示：

![1*mC48Yf8QvH62155Q9d4aDw](https://miro.medium.com/max/361/1*mC48Yf8QvH62155Q9d4aDw.png)



每个 Library module 都包含一个简单的类。

**LibraryD:**

```kotlin
class ClassD {

    fun tellMeAJoke():String{
        return "You are funny :D"
    }
}
```

**LibraryC:**

```kotlin
class ClassC {
    fun tellMeAJoke(): String {
        return "You are funny :C"
    }
}
```

**LibraryB:**

```kotlin
class ClassB {

    val b = ClassD()

    fun whereIsMyJoke(): String {
        return b.tellMeAJoke()
    }
}
```

**LibraryA:**

```kotlin
class ClassA {

    val c = ClassC()

    fun whereIsMyJoke(): String {
        return c.tellMeAJoke()
    }
}
```

从上面这些类文件中可以看到 LibraryA 和 LibraryB 分别依赖于 LibraryC 和 LibraryD。因此，需要将它们之间的依赖关系添加到 `build.gradle` 文件当中。

### Compile (2.0) or Api (3.0):

3.0 中新的 `api` 关键字与之前版本的 `compile` 关键字意义完全相同。因此，如果项目中所有 `compile` 关键字被 `api` 所取代，这完全可以正常工作。现在，让我们在 **LibraryB** 中通过 `api` 关键字来依赖 **LibraryD**：

```groovy
dependencies {
          . . . . 
          api project(path: ':libraryD')
}
```

同样地，将 **LibraryB** 添加到 **app** 模块中：

```groovy
dependencies {
          . . . . 
          api project(path: ':libraryB')
}
```

现在，我们可以在 app 模块中访问到 **LibraryB** 和 **LibraryD**。在示例应用中，两个库的访问形式如下：

![1*NPw6NXM8l6eMq8jUD86RNw](https://miro.medium.com/max/288/1*NPw6NXM8l6eMq8jUD86RNw.png)

### Implementation (3.0):

现在是时候来找出 `implementation` 和 `api` 之间的不同之处了。再次回到上面这个例子，现在让我们在 **LibraryA** 中通过 `implementation` 关键字引入 **LibraryC**：

```groovy
dependencies {
          . . . . 
          implementation project(path: ':libraryC')
}
```

App 模块同理：

```groovy
dependencies {
          . . . . 
          implementation project(path: ':libraryA')
}
```

现在，如果我们在 app 模块访问 **LibraryC**，Android studio 将会抛出一个错误：

![1*cy_TZPrbcpngrjnGC1pTSQ](https://miro.medium.com/max/428/1*cy_TZPrbcpngrjnGC1pTSQ.png)



这意味着如果我们使用 `implementation` 关键字代替 `api`，**LibraryC** 将无法在 App 模块中被访问。那么 `implementation` 这样做有什么好处呢？

### Implementation vs api:

在第一个场景中，**LibraryD** 是通过 `api` 关键字来编译的。一旦 **LibraryD** 中的实现做了任何改动，那么 gradle 都需要重新编译 **LibraryD**，**LibraryB** 和所有其他引入 **LibraryB** 的模块一样可能使用的是 **LibraryD** 中的实现。

![1*WGFbkIccHZRJs3rDpQsFRA](https://miro.medium.com/max/499/1*WGFbkIccHZRJs3rDpQsFRA.png)



但在第二个场景中，如果更改了 **LibraryC** 中的代码实现，那么 Gradle 只会重新编译 **LibraryC** 和 **LibraryA**，因为其他任何没有**直接依赖** **LibraryC** 的类都无法使用其中任何实现。



![1*oFJWUsL7FFMETVFBNkkL2Q](https://miro.medium.com/max/464/1*oFJWUsL7FFMETVFBNkkL2Q.png)



如果你正在开发具有多个模块的项目，那么这种策略可以显著加快构建过程。我在示例程序中进行了尝试，几秒钟后就有了一些改进。以下是所有方案的构建报告。

**全量构建:**

![1*ihaFEa8AF1xzuRoeOEXg3Q](https://miro.medium.com/max/700/1*ihaFEa8AF1xzuRoeOEXg3Q.png)

**变更 LibraryD:**

![1*gs2jl_HAT7lPl0-WYprfpA](https://miro.medium.com/max/700/1*gs2jl_HAT7lPl0-WYprfpA.png)

**变更 libraryC:**

![1*2CaNBOd7yi_akrg6PAO6Ag](https://miro.medium.com/max/700/1*2CaNBOd7yi_akrg6PAO6Ag.png)

### 总结

Android Gradle 插件版本升级至3.0之后，我们只需要将所有 `compile` 替换为 `implementation` 关键字并尝试构建项目。如果可以构建成功，那么很好。否则，请查找任何你可能正在使用的遗漏的依赖库，并找到宿主 library 换用 `api` 关键字引入这些 library。



