---
title: Flutter 之 AppBar 这样的骚操作你知道吗？
date: 2019-03-01 17:06:14
tags: 
- Flutter
- AppBar
categories: Flutter
toc: true
---
好久不见了，这阵子在忙公司的项目，加班比较严重，这周终于抽了点时间来学习一下新技术。由于去年九月份在上海参加过 Google 举办的 Google develop days， 受益颇多，特别在其目前正在大力热推的 Flutter 框架。相比于目前热门的跨平台框架 React Native，Flutter在 UI 绘制以及性能方便不遑多让。因此，这款 app 打算基于 Dart 语言，并采用 Flutter 框架来完成。

花了大概几天时间熟悉了下 Dart 语法和 Flutter 基本组成控件，就开始摸索着做一个练手项目。AppBar 想必大家都用过，当其处于启动页面时是隐藏 `leading` 的，而处于其他页面时左边默认携带返回按钮。目前，我们的需求是：首页的  AppBar 最左边为用户头像，点击用户头像可以自动打开左侧抽屉栏。<!--more-->

最终效果图如下所示：

![效果图](https://upload-images.jianshu.io/upload_images/5256969-7b5d97364d1815d3.gif?imageMogr2/auto-orient/strip)




初学者一开始总是痛苦的，还好，解决问题的途径“万变不离其宗”。我们先来查看一下 Flutter [官方文档 ](https://docs.flutter.io/flutter/material/AppBar-class.html)发现，要使用 AppBar来操作页面，决定其左边点击事件的属性就是 `leading` ：

```
AppBar({
Key key,
Widget leading,
bool automaticallyImplyLeading: true,
Widget title,
List<Widget> actions,
Widget flexibleSpace,
PreferredSizeWidget bottom,
double elevation,
Color backgroundColor,
Brightness brightness,
IconThemeData iconTheme,
TextTheme textTheme,
bool primary: true,
bool centerTitle,
double titleSpacing: NavigationToolbar.kMiddleSpacing,
double toolbarOpacity: 1.0,
double bottomOpacity: 1.0
})
```

可以看到，我们 AppBar 中的 `leading` 属性是一个 `Widget` 类型，那么它接收的参数范围就很广了，换句话说，我们可以直接将一个按钮传递给它，并在 `onPressed` 方法中处理左边侧滑栏的开启和关闭状态。👌，有思路了咱们久开始试验一下：

```
 final GlobalKey<ScaffoldState> _scaffoldKey = new 													GlobalKey();
 
 @override
  Widget build(BuildContext context) {
    return new DefaultTabController(
        length: _pages.length,
        child: new Scaffold(
          key: _scaffoldKey,
          appBar: new AppBar(
            /// 第一种方式
            /// 通过可监听点击的IconButton传入widget，
            /// 并在onPressed中处理drawer开启，借助于GlobalKey
            leading: new IconButton(
                icon: new Container(
                  padding: EdgeInsets.all(3.0),
                  child: new CircleAvatar(
                      radius: 30.0,
                      backgroundImage: AssetImage("assets/images/moosphon_logo.jpeg")
                  ),
                ),
              onPressed: (){
                _scaffoldKey.currentState.openDrawer();
              },
            ),
            centerTitle: true,
            title: new TabBar(
                isScrollable: true,
                labelPadding: EdgeInsets.all(14),
                indicatorSize: TabBarIndicatorSize.label,
                indicatorColor: Colors.white,
                tabs: _titles.map((String title) =>
                new Tab(text: title)).toList()
            ),
            actions: <Widget>[
              new IconButton(
                  icon: new Icon(Icons.search),
                  tooltip: '搜索',
                  onPressed: (){
                    NavigatorUtil.intentToPage(context, new SearchPage(), pageName: "SearchPage");
                  }
              )
            ],
          ),
          body: new TabBindingView(),
          drawer: new Drawer(
            child: new HomeLeftDrawerPage(),
          ),
        )
    );
  }

```

为了不影响篇幅，这里我只贴了关键代码，至于 `Drawer` 用法相信大家没什么问题，这里我们给 AppBar 的`leading` 参数传了一个 `IconButton`  控件，里面我还实现了圆形头像。这样一来，我们的用户头像就可以被点击了响应了，接下来我们需要处理头像点击事件与 `Drawer` 的联动性了。这里我们通过设置 `GlobalKey` 来让 `Scaffold` 容器获取到其内部的 `Drawer` 组件，进而控制它的开闭，这样，我们就已经可以通过点击自定义的用户头像来开启左边侧滑栏啦😄。

细心的小伙伴一定会发现，上面的代码中我注释这只是第一种实现方式，难道还有第二种实现方式？哈哈，没错，的确还有第二种思路。其实，通过查看 `AppBar` 的源码，我们可以发现：AppBar内部已经自动实现了 AppBar 的 `leading` 与 `Drawer` 抽屉栏的关联：

```
Widget leading = widget.leading;
    if (leading == null && widget.automaticallyImplyLeading) {
      if (hasDrawer) {
        leading = IconButton(
          icon: const Icon(Icons.menu),
          onPressed: _handleDrawerButton,
          tooltip: MaterialLocalizations.of(context).openAppDrawerTooltip,
        );
      } else {
        if (canPop)
          leading = useCloseButton ? const CloseButton() : const BackButton();
      }
    }
    if (leading != null) {
      leading = ConstrainedBox(
        constraints: const BoxConstraints.tightFor(width: _kLeadingWidth),
        child: leading,
      );
    }
```

由于代码较多，这里我只放上了 `build` 方法中关于 `leading` 部分的关键代码，通过分析我们可以发现：

- `leading` 如果 为空并且 `automaticallyImplyLeading` 属性为true，那么就会自动推断出 `leading` 的 `Widget` 的类型和用途，另外如果当前 `Scaffold` 中存在 `Drawer` ，则会自动创建一个 ` IconButton` 作为`leading`  使用，同时它的点击事件中处理了与 `Drawer` 抽屉栏的关联事件，无需开发者再处理。也就是说，在这种情况下，如果我们自定义了 `leading` （例如当前我们给它传的是用户的头像 `Widget` ），那么 `leading` 就无法自动关联 `Drawer` ，也就是说关联 `Drawer`  的这部分代码需要开发者自行实现。
- 如果  `leading` 不为空呢？并且 `automaticallyImplyLeading` 开关关闭，那么 `leading` 的空间就会被 `title` 给占据。

说了这么多，最终的结论是：如果我们想要自定义 `leading` ,那么目前官方源码中 AppBar 中 `leading`  自动关联 `Drawer` 的处理我们没办法使用。如果我们非要用呢？那就只能修改源码啦：

```
/// 注意，前方高能来袭，以下为修改的部分code

    Widget leading = widget.leading;
    if (/*leading == null && */widget.automaticallyImplyLeading) {
      if (hasDrawer) {
        leading = IconButton(
          icon: /*const Icon(Icons.menu)*/ leading ?? Icon(Icons.home), // 如果leading指定了widget那么
          onPressed: _handleDrawerButton,
          tooltip: MaterialLocalizations.of(context).openAppDrawerTooltip,
        );
      } else {
        if (canPop)
          leading = useCloseButton ? const CloseButton() : const BackButton();
      }
    }
    if (leading != null) {
      leading = ConstrainedBox(
        constraints: const BoxConstraints.tightFor(width: _kLeadingWidth),
        child: leading,
      );
    }
```

铛铛铛铛～这就改好啦，改动的地方是不是很少？哈哈😄，其实只要把 `leading` 为空的限制条件去掉，并且可以传入我们自己定义的 `Widget` 就好啦！

这样一来，我们就不用手动去处理 `leading` 与 `Drawer` 的关联事件，只需要交给系统帮我们完成就可以啦：

```
@override
  Widget build(BuildContext context) {
    return new DefaultTabController(
        length: _pages.length,
        child: new Scaffold(
          key: _scaffoldKey,
          appBar: new DrawerAutoBindingAppBar(
            /// 第二种方式
            /// 通过修改[AppBar]源码来将普通widget作为icon传给IconButton
            /// 源码中已处理onPressed关联drawer事件，无需额外处理，详情见[DrawerAutoBindingAppBar]
            leading: new Container(
                padding: EdgeInsets.all(3.0),
                child: new CircleAvatar(
                    radius: 30.0,
                    backgroundImage: AssetImage("assets/images/moosphon_logo.jpeg")
                )
            ),
            centerTitle: true,
            title: new TabBar(
                isScrollable: true,
                labelPadding: EdgeInsets.all(14),
                indicatorSize: TabBarIndicatorSize.label,
                indicatorColor: Colors.white,
                tabs: _titles.map((String title) =>
                new Tab(text: title)).toList()
            ),
            actions: <Widget>[
              new IconButton(
                  icon: new Icon(Icons.search),
                  tooltip: '搜索',
                  onPressed: (){
                    NavigatorUtil.intentToPage(context, new SearchPage(), pageName: "SearchPage");
                  }
              )
            ],
          ),
          body: new TabBindingView(),
          drawer: new Drawer(
            child: new HomeLeftDrawerPage(),
          ),
        )
    );
  }

```

代码就不多做解释了，效果是和之前一样的，好啦，这样我们就更加了解 AppBar 的用法以及 `leading` 的背后“小彩蛋”啦😁。

感谢大家的阅读，以后我将给大家带来更多好玩实用的应用型开发技巧，如果大家有任何建议，欢迎留言👏。



> QQ群： 601924443  一起玩技术～