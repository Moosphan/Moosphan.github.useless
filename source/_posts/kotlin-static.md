---
title: Kotlin中的静态变量和静态方法
date: 2018-12-11 19:10:01
tags: Kotlin
categories: Kotlin
toc: true
---
在日常开发过程中，`静态变量 `和  `静态方法` 是我们常见的用法，Java中相信大家并不陌生了，那么在 Kotlin 中该如何使用呢？其实很简单，只需要一个将变量和方法包含在 `companion object` 域中即可，比如这样：

```
class Constant {
    companion object {
        // 接口根地址
        const val BASE_URL = "http://xxxx.xxx.xxx/"
        // 友盟
        const val UMENG_APP_KEY = "xxxxxxxxxx"
        const val UMENG_CHANNEL = "umeng"
        // 微博
        const val WEIBO_APP_KEY = "xxxxxxxx"
        const val WEIBO_SECRET  = "xxxxxxxxxx"
        
        
        fun getVideoFactor(){
            // do some work
        }
    }

}
```
<!--more-->
看后是不是很简单？在纯kotlin代码中可以直接这样使用：

```
//初始化各平台的APIKey
        PlatformConfig.setWeixin(Constant.WECHAT_APP_ID, Constant.WECHAT_APP_SECRET)
        PlatformConfig.setSinaWeibo(Constant.WEIBO_APP_KEY, Constant.WEIBO_SECRET, Constant.WEIBO_AUTH_RETURN_URL)
```

然而，如果我们使用的是Java和kotlin混合开发，在Java代码中就无法通过 `Constant.静态变量` 的方式来使用静态变量或者方法来，而是通过如下方式：

```
//初始化各平台的APIKey
        PlatformConfig.setWeixin(Constant.Companion.WECHAT_APP_ID, Constant.WECHAT_APP_SECRET)
        PlatformConfig.setSinaWeibo(Constant.Companion.WEIBO_APP_KEY, Constant.WEIBO_SECRET, Constant.WEIBO_AUTH_RETURN_URL)
```



如果我们想像kotlin那样直接通过 `类名.静态变量` 方式使用呢？我们可以借助于注解 `@JvmField` 和 `@JvmStatic` 来分别标注静态变量和静态方法，之后我就能在Java代码中像以前方式那样直接使用静态的成员啦！例如这样：

```
/**
 * @author moosphon on 2018/12/12
 * desc: 异常的统一处理者
 */
class ExceptionHandler {


    companion object {
        @JvmField
        var errorCode = NetRequestStatus.UNKNOWN_ERROR

        @JvmField
        var errorMessage = "请求失败，请稍后重试"

        @JvmStatic
        fun handleException(e : Throwable): String{
            e.printStackTrace()
            when(e){
                is SocketException  -> {
                    Logger.e("ExceptionHandler", "网络连接异常： " + e.message)
                    errorCode = NetRequestStatus.NETWORK_ERROR
                    errorMessage = "网络连接异常"
                }

                is JsonParseException -> {
                    Logger.e("ExceptionHandler", "数据解析异常： " + e.message)
                    errorCode = NetRequestStatus.PARSE_ERROR
                    errorMessage = "数据解析异常"
                }

                else -> {
                    try {
                        Logger.e("ExceptionHandler", "其他错误： " + e.message)
                    } catch (e1: Exception) {
                        Logger.e("ExceptionHandler", "未知错误： " + e.message)
                    }

                    errorCode = NetRequestStatus.UNKNOWN_ERROR
                    errorMessage = "未知错误，一起祷告快点好起来吧～"
                }
            }

            return errorMessage
        }

    }


}
```

前段时间比较忙，之后会继续为大家带来kotlin方面的文章，大家拭目以待。Android-kotlin交流群：601924443
