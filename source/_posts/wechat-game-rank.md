---
title: 基于Laya游戏引擎实现微信小游戏排行榜
date: 2018-09-20 19:21:04
tags: 
- 微信小游戏
- Layabox
- 游戏排行榜
categories: 微信小游戏
toc: true
---
 我们都知道，微信小游戏和小程序目前风头十足，很多公司都逐渐增加了相关业务线来迅速推广自己的产品和抢占用户群。说到微信小游戏，就不得不提到排行榜这个功能，就目前游戏行业，似乎都离不开排行榜这个重要功能，用户很大一部分留存都是依仗这个看似不起眼的模块。那么，微信小游戏中具体该如何借助laya引擎实现排行榜这个功能呢？我们先来看一下最终的效果图：


![image](http://upload-images.jianshu.io/upload_images/5256969-06769486c6134cf3.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


按照微信官方的说法，如果我们要使用微信官方提供的好友关系链的数据，我们就不能直接在项目中绘制排行榜，我们需要借助于开放域来绘制排行榜：



![image](http://upload-images.jianshu.io/upload_images/5256969-84c2aea9984ef228.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



​	如果想要展示通过关系链 API 获取到的用户数据，如绘制排行榜等业务场景，需要将排行榜绘制到 `sharedCanvas` 上，再在主域将 sharedCanvas 渲染上屏。简单来说，sharedCanvas 是主域和开放数据域都可以访问的一个离屏画布。在开放数据域调用 wx.getSharedCanvas() 将返回 sharedCanvas。更多相关详情可以去看看官网的介绍：https://mp.weixin.qq.com/debug/wxagame/dev/tutorial/open-ability/open-data.html

那么我们来实际动手操作一下吧。<!--more-->

#### 主域绘制

通过效果可以看出来，我们排行榜是一个弹窗形式展示的，由于开放域只负责排行榜UI绘制，所以，除此以外的UI以及交互我们需要在主域绘制和处理。因此，这里的弹窗 `dialog` 需要在主域绘制，然后将对应的排行榜需要显示的位置信息和长宽映射到开放域，具体代码如下：

```

    /**
	 * 显示排行榜数据
	 */
	function onRankInfoLoad(){
		console.log("查看排行榜～");
		var dialog = new RankDialogUI();
		showShareCanvas();
		// 解决显示对象和鼠标错位而导致的排行榜滑动无效问题
		var globalPosition = dialog.ranking_list.localToGlobal(new Laya.Point());
		var originMatrix = Laya.stage._canvasTransform;
		var mat = new Laya.Matrix(originMatrix.a, 0, 0, originMatrix.d, globalPosition.x * originMatrix.a, globalPosition.y * originMatrix.d);
		wxPostMessage({
            command: 0,
            text: "设置开放域canvas大小",
            canvasData: {
                width: rankViewWidth * mat.a, height: rankViewHeight * mat.d, matrix: mat
            },
            isLoad: false
        }, null, function (message) {
            console.log("再次往开放域发请求");
            window['wx'].postMessage({
                command: 1,
                text: '开放域加载资源',
            });
        });
	
		Laya.stage.addChild(dialog);
		dialog.ranking_list.visible = false;
		dialog.popup();
		 Laya.timer.once(400,this,function(){
			wxPostMessage({
				command: 3,
				text: "获取排行榜数据～"
			}, null, function (message) {
				console.log("获取排行榜的回调～");
			});
		 });
		
	
		dialog.btn_rank_dialog_share.on(Laya.Event.CLICK, this, onGameRankShare);
		dialog.btn_rank_dialog_back.on(Laya.Event.CLICK, this, onDialogClose);
		function onDialogClose(){
			wxPostMessage({
				command: 4,
				text: "关闭排行榜～"
			}, null, function (message) {
				console.log("关闭排行榜的回调～");
			});
		    dialog.close();
	    }

		function onGameRankShare(){
			console.log("分享排行榜～");
			window['wx'].showShareMenu({
				withShareTicket:false,
				success:function(res){
					console.log("开启转发成功～");
				},
				fail:function(res){
					console.log("开启转发失败～");
				},
				complete:function(res){

				}
        	});
			window['wx'].onShareAppMessage(function () {
				return {
					title: '我在飞机大战游戏中排名又上升了,快来挑战我吧～'
				}
			})
			window['wx'].shareAppMessage({

				title: '我在飞机大战游戏中排名又上升了,快来挑战我吧～',
				imageUrl: canvas.toTempFilePathSync({
					x: (screenWidth - rankViewWidth)/2 - 10,
					y: (screenHeight - rankViewHeight)/2+80,
					width: (rankViewWidth - 30)*4,
					height: (rankViewHeight - 40)*4,
					destWidth: 500,
					destHeight: 600
				})
			});
		}
	}
	
	/**
     * 设置共享Canvas
     */
    function showShareCanvas(){
            window['sharedCanvas'].width = rankViewWidth;
            window['sharedCanvas'].height = rankViewHeight;
            //主域显示开放域内容???
            //window['sharedCanvas'].sharedCanvas = window['wx'].getOpenDataContext().canvas;
            Laya.timer.once(1000, this, function () {
                var sprite = new Laya.Sprite();
                sprite.zOrder = 1008;
                sprite.pos(0, 0);
                var texture = new Laya.Texture(window['sharedCanvas']);
                texture.bitmap.alwaysChange = true;//小程序使用，非常费
                sprite.graphics.drawTexture(texture, (screenWidth - rankViewWidth)/2, (screenHeight - rankViewHeight)/2, texture.width, texture.height);
                Laya.stage.addChild(sprite);
            });
    }

/**
     * 向开放域发送消息，并接收开放域返回过来的数据，
     * 可根据发送参数和接收数据在主域这边进行下步处理
     * @param message
     * @param caller
     * @param callback
     */
    function wxPostMessage(message, caller, callback){
        window['wx'].postMessage(message);
        Laya.timer.once(400, this, function (){
            //回调处理
            if (caller == null || caller == undefined) {
                callback(message);
            } else {
                caller.callback(message);
            }
        });
    }

```

这边主要发送的消息体中携带了command字段，用于在开放域执行不同的功能代码，这边大体分为：资源加载命令、初始化排行榜大小命令、获取关系链(排行榜)数据命令以及关闭排行榜的命令，大家可以根据业务具体需要适当增减。



#### 开放域绘制

根据官方的说明，`开放数据域` 是一个封闭、独立的 JavaScript 作用域。要让代码运行在开放数据域，需要在 game.json 中添加配置项 `openDataContext` 指定开放数据域的代码目录。添加该配置项表示**小游戏启用了开放数据域**，这将会导致一些 [限制](https://mp.weixin.qq.com/debug/wxagame/dev/tutorial/open-ability/open-data.html#%E9%99%90%E5%88%B6)。这些限制主要包含：

- 无法设置sharedCanvas的宽高
- 只能使用有限的接口(如逐帧动画、Timer、触摸事件以及获取和设置关系链数据等接口)

下面我们一步步来在开放域绘制排行榜。

1. 首先，需要新建一个项目作为开放域，这个项目目录是与主域项目平行的(主域就是未做排行榜之前的项目目录)，比如我的项目目录是这样的：

   ![image](http://upload-images.jianshu.io/upload_images/5256969-3ba27d9fdc75cdd1.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2. 接着，我们需要在开放域中接收主域发送的消息并处理不同的功能命令，UI就不放了，看看具体代码吧：

   ```
   /**
        * 监听从主域发过来的消息
        */
       function wxOnMessage(){
           if(window['wx'] != undefined){
               window['wx'].onMessage(function (message){
                   dispatchMessage(message);
               });
           }else{
               console.log("微信接口无法使用～");
           }
       }
       /**
        * 处理消息
        * @param {*} message 
        */
       function dispatchMessage(message){
           switch(message.command){
               //设置开放域画布大小
               case 0:
                   sample.setCanvasSize(message.canvasData);
                   break;
               //加载资源
               case 1:
                   sample.loadResource();
                   break;
               //写入排行榜数据
               case 2:
                   sample.writeRankingData(message.rankingData);
                   break;
               //获取微信排行榜数据
               case 3:
                   sample.getRankingData();
                   break;
               //关闭排行榜
               case 4:
                   sample.closeRankingDialog();
                   break;
               default:

                   console.log(JSON.stringify(message));
           }
       }
       /**
        * 设置开放域画布大小
        * @param {*} size 
        */
        _proto.setCanvasSize = function(size){
           console.log("设置开放域canvas大小～");
           window['sharedCanvas'].width = size.width;
           window['sharedCanvas'].height = size.height;
           //Laya.stage.width = size.width;
           //Laya.stage.width = size.height;
           /**
            * 将主域的canvasTransform映射到开放域
            */
           if(size.matrix!=null){
               console.log("收到主域的同步canvasTransform了～");
           }
           var mainMatrix = size.matrix;
           var openMatrix = new Laya.Matrix();
           openMatrix.a = mainMatrix.a;
           openMatrix.b = mainMatrix.b;
           openMatrix.c = mainMatrix.c;
           openMatrix.d = mainMatrix.d;
           openMatrix.tx = mainMatrix.tx;
           openMatrix.ty = mainMatrix.ty;
           //重置矩阵
           Laya.stage._canvasTransform = openMatrix;
           //监听舞台的鼠标移动事件
           Laya.stage.mouseEnabled = true;
       }
      
       /**
        * 用户自己的排名
        */
       var myRanking = -1;
       /**
        * 获取微信排行榜数据
        */
       _proto.getRankingData = function(){
           window['wx'].getUserInfo({
               openIdList: ['selfOpenId'],
               success: (userRes) => {
                   console.log('success', userRes.data);
                   //索引代表各个好友0为自己
                   let userData = userRes.data[0];
                   console.log("取信息索引0" + userData.nickName);
                   //取出所有好友数据
                   window['wx'].getFriendCloudStorage({
                       keyList: [
                           //'击杀排行',
                           '第1关',
                           '第2关',
                           '第3关'
                       ],
                       success: res => {
                           console.log("wx.getFriendCloudStorage success", res);
                           let data = res.data;
                           /*data.sort((a, b) => {
                               if (a.KVDataList.length == 0 && b.KVDataList.length == 0) {
                                   return 0;
                               }
                               if (a.KVDataList.length == 0) {
                                   return 1;
                               }
                               if (b.KVDataList.length == 0) {
                                   return -1;
                               }
                               return b.KVDataList[0].value - a.KVDataList[0].value;
                           });*/
                           for (let i = 0; i < data.length; i++) {
                               var playerInfo = data[i];
                               var currentPlayer = res.data[i].nickname;
                               console.log("当前排行玩家昵称为=>"+res.data[i].nickname);
                               var kvList = playerInfo.KVDataList;
                               var scoreSum = 0;
                               if(kvList.length>0){
                                   for(var j = 0;j<kvList.length;j++){
                                       if(kvList[j].key != null){
                                           //将value转化为int再累加
                                           scoreSum+=Number(kvList[j].value);
                                       }
                                   }
                               }
                               
                               if (data[i].avatarUrl == userData.avatarUrl) {
                                   //获取群好友的时候,没有自己的名字??
                                   data[i].nickName = userData.nickName;
                                   myRanking = i+1;
                                   console.log("此ID为自己,当前排名第"+myRanking);
                                   
                               }
                               //填充总分信息
                               sortData.push({
                                   nickName: currentPlayer, 
                                   avatarUrl: data[i].avatarUrl, 
                                   totalScore: scoreSum 
                               });
                           }
                           sortData.sort((a, b) => {
                               var score1 = Number(a.totalScore);
                               var score2 = Number(b.totalScore);
                               if (score1 > score2) {
                                   return -1;
                               }else if(score1 < score2){
                                   return 1;
                               }else{
                                   return 0;
                               }
                           });
                           showRankingDialog();

                           
                       },
                       fail: res => {
                           console.log("拉取好友信息失败", res);
                       },
                   });
               },
               fail: (res) => {
                   console.log("拉取个人信息失败")
               }
           });
       }
       /**
        * 加载资源
        */
       _proto.loadResource = function(){
           Laya.loader.load(["comp/bg_line.png","comp/ranking1.png","comp/ranking2.png",
           "comp/ranking3.png","comp/userholder_img.png"], Laya.Handler.create(null,function(){
               console.log("开放域资源加载完毕～");
               sample.rankView = new RankingViewUI();
               Laya.stage.addChild(sample.rankView);
           }));
       
       }

       /**
        * 写入排行榜数据
        */
       _proto.writeRankingData = function(rankingData){
           console.log("写入排行榜数据～");
           //KVDataList代表排行数据,可以为多个,多个代表多个排行
           //key-排行类型,value-排行分数
           window['wx'].setUserCloudStorage({
               KVDataList: [
                   //{ key: '击杀排行', value: "" + 1 },
                   { key: '第'+rankingData.fightLevel+'关', value: rankingData.fightScore+"" },//需要改成动态的值
               ],
               success: function (res) {
                   console.log('setUserCloudStorage', 'success', res)
               },
               fail: function (res) {
                   console.log('setUserCloudStorage', 'fail')
               }
           });
       }
       var sortData = [];
       /**
        * 渲染排行榜列表
        */
       function showRankingDialog(){
           console.log("拿到好友排行榜信息", sortData);
           sample.rankView.ranking_list.vScrollBarSkin = "";
           sample.rankView.ranking_list.array = sortData;
           sample.rankView.ranking_list.renderHandler = new Laya.Handler(this, onRender);
           sample.rankView.ranking_list.selectHandler = new Laya.Handler(this, onSelect);
           
       }

       var lastRenderIndex = -1;
       function onRender(cell, index){
           if (index == lastRenderIndex) {
               return;
           }
           lastRenderIndex = index;
           //根据子节点的name获取子节点对象
           var name = cell.getChildByName("item_rank_name");
           var ranking = cell.getChildByName("item_rank_text");
           var userlogo = cell.getChildByName("item_rank_logo");
           var score = cell.getChildByName("item_rank_score");
   		var rank_icon = cell.getChildByName("item_rank_icon");
   		
           name.text = sortData[index].nickName;
           console.log("渲染排行榜当前的用户名为="+sortData[index].nickName+"，渲染索引："+lastRenderIndex);
           ranking.text = (index+1)+"";
           userlogo.skin = sortData[index].avatarUrl;
           score.text = sortData[index].totalScore+"分";
           if(lastRenderIndex === 0 || lastRenderIndex === 1 || lastRenderIndex === 2){
   			ranking.visible = false;
   			rank_icon.visible = true;
   			rank_icon.skin = "comp/ranking"+(lastRenderIndex+1)+".png";
   		}else{
   			rank_icon.visible = false;
               ranking.visible = true;
           }

       }

       function onSelect(index){
           console.log("当前选择的索引是："+index);

       }

       /**
        * 关闭排行榜
        */
       _proto.closeRankingDialog = function(){
           //dialog.close();
   		console.log("关闭排行榜～");
           sortData = [];
           lastRenderIndex = -1;
           sample.rankView.removeChildren();
           sample.rankView = null;
           
       }

       _proto.start = function(){
           console.log("开始接收主域的消息～");
           wxOnMessage();
       }
   }

   ```

   可以看到，我们这里通过 `wx.onMessage` 方法来获取主域发送的数据，然后借助 `dispatchMessage` 方法作消息的分发处理。值得注意的是，我们在设置开放域canvas大小的时候，需要重置坐标矩阵，将主域的排行榜显示位置映射到开放域中来，否则会发生滑动无效的问题。我这里设置的排行榜数据有三条，大家可以根据具体需求来传入，获取到排行榜数据后，需要对它进行排序处理并展示，这里直接借助于laya中的 `List` 控件展示就可以了，对于它用法不熟悉的可以去laya官网了解一下：https://ldc.layabox.com/doc/?nav=zh-js-6-0-0

3. 开放域图片加载问题：
用过laya游戏引擎的都知道，我们一般用官方推荐的打包图集的方式来加载游戏中的图片资源，这种方式在主域中是可行的，然而，在开放域中却不能成功加载图片资源。因此，开放域中，我们不需要将图片资源打包成图集，只要像下面这样直接加载图片即可：
```
 /**
     * 加载资源
     */
    _proto.loadResource = function(){
        Laya.loader.load(["comp/bg_line.png","comp/ranking1.png","comp/ranking2.png",
        "comp/ranking3.png","comp/userholder_img.png"], Laya.Handler.create(null,function(){
            console.log("开放域资源加载完毕～");
            sample.rankView = new RankingViewUI();
            Laya.stage.addChild(sample.rankView);
        }));
    
    }

```
然后，我们还需要将对应的图片文件夹拷贝到wx_publish目录下，否则会提示找不到图片资源。


#### 合并主域和开放域

主域和开放域功能代码实现了之后，我们就需要打包成微信小游戏项目了。首先，我们需要先将主域项目发布成微信小游戏，发布目录直接为项目的根目录，如上图的 `wx_publish` 目录。然后，在该目录下创建src/myOpenDataContext目录。接着，我们需要将开放域项目也发布成微信小游戏，目录可以选择桌面，名称为 `wx_open`，发布成功后，进入该目录，将code.js、weapp-adapter.js以及index.js文件复制到 `wx_publish/src/myOpenDataContext` 目录下，并在 `game.json` 文件中增加开放域映射目录：

```

{
  "deviceOrientation": "portrait",
  "showStatusBar": "false",
  "networkTimeout": {
    "request": 10000,
    "connectSocket": 10000,
    "uploadFile": 10000,
    "downloadFile": 10000
  },
  "openDataContext": "src/myOpenDataContext"
}
```



到这里，微信小游戏排行榜功能就算实现了，到头来发现，其实实现起来并不难，难的是缺乏资料，此文仅用来抛砖引玉，如有问题欢迎提出。

