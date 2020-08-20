---
title: Android将library代码发布到jcenter
date: 2018-03-25 14:25:21
tags: 
- Android
- Jcenter
- 开源
categories: Jcenter
toc: true
---
作为一名Android开发者，日常开发工作中肯定会用到一些强大的第三方开源库，有了这些开源的帮助，开发工作可谓是如鱼得水。很多时候，我们自己写了一套代码，为了让别人体验到自己代码的便捷之处，通常我们需要将其部署到github，发布到jcenter，然后生成gradle代码供他人使用。网上对于这部分的教程很多，不管是通过gradle-bintray-plugin插件或者是bintray-release插件，实现流程几乎一致，但有些细节存在差异，最终都可以通过compile或者implemention引用。具体步骤可以参考以下两篇文章：

- 通过bintray-release插件发布：<https://blog.csdn.net/lmj623565791/article/details/51148825>
- 通过gradle-bintray-plugin插件发布：<https://www.cnblogs.com/AbrahamCaiJin/p/7058147.html>

当然，发布过程并不是百分百成功的，难免会遇到以下常见问题：<!--more-->

1. 终端输入`gradlew install`命令后出现`bash: gradlew: command not found`问题：

   该问题的原因一般是没有配置gradle的环境变量，我们可以将gradle所在位置复制到环境变量中去，windows较为简单，Mac比较特殊，这里特别说明一下，可以参考这篇文章：<https://blog.csdn.net/u013424496/article/details/52684213>

2. 权限问题：

   可能会遇到`-bash: /Applications/Android Studio3.0.app/Contents/gradle/gradle-4.1/bin/gradle: Permission denied`问题，一般是gradle权限引起的，我们可以在as终端输入：

   ```
   $ chmod +x gradle
   $ chmod +x gradle.bat
   ```

   一般情况下，重新执行gradlew 命令就没问题了，如果还是不行，可以在`gradlew xxx`命令前加上`./`,如：

   ```
   ./gradlew clean build bintrayUpload 
    -PbintrayUser=xxx 
    -PbintrayKey=xxxxxxxxxxxxxxxxxxxxxx 
    -PdryRun=false
   ```

3. 执行jcenter发布命令时出现`Execution failed for task ':library:bintrayUpload'. Bintray user cannot be empty!`问题：

   这种问题可能是user为空，没有设置，也可能是发布命令中间多了空格符合或者换行符号，不要从网上复制命令直接执行，最好先复制到文本编辑器查看格式是否正确。

4. library发布成功后，一定要记得先在jcenter个人仓库的项目页点击`add to jcenter`审核通过后才能使用compile和maven。

5. 如果jcenter审核通过，可以通过`compile....`命令复制到gradle中编译一下，如果出现：

   ```
   Error:(32, 13) Failed to resolve: com.android.support:support-annotations:27.1.0 Install Repository and sync project
   Show in File.
   ```

   可以尝试在project到build.gradle文件中添加如下代码：

   ```
   allprojects {
       repositories {
           maven { url "https://maven.google.com" }
           jcenter()
       }
   }
   ```

   重新编一下看能否通过。

6. 如果使用`classpath 'com.jfrog.bintray.gradle:gradle-bintray-plugin:xx'`插件，发布到jcenter版本version必须通过library的gradle文件中定义：

   ```

   version = "1.0.0"

   ```

   方可在jcenter正常显示版本号，否则会出现版本号未知的问题。

7. 如果使用`classpath 'com.jfrog.bintray.gradle:gradle-bintray-plugin:xx`插件，最终jcenter中的compile依赖项目名会自动使用你library的moudle名作为最终的项目名，如，我的library的moudle名为library，那么最终生成的compile代码会是：

   ```
   com.moos:library:1.0.0
   ```

   因此，在发布之前，请务必确认好最终的名字。这里，强烈推荐使用`classpath 'com.novoda:bintray-release:0.x.x'`插件来进行jcenter发布操作，真的很方便！

8. 最后，如果你的library中有中文的注释，可能会有编码问题，可以在project的build.gradle文件中添加如下代码：

   ```
   allprojects {
       tasks.withType(Javadoc) {
           options{
               encoding "UTF-8"
               charSet 'UTF-8'
               links "http://docs.oracle.com/javase/8/docs/api"
           }
       }
   }
   ```

   ​

