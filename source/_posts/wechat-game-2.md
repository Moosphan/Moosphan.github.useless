---
title: 微信小游戏开发笔记（下）
date: 2018-09-11 14:18:06
tags: 
- 微信小游戏
- Layabox
categories: 微信小游戏
toc: true
---
上次我们介绍了基于laya实现微信小游戏的排行榜功能，过程中免不了遇到一些坑，主要包含一些图片加载、逻辑处理、排行榜绘制、微信头像跨域等问题，具体可参考下面的记录文档。
#### 2018/08/01
1. 设置laya舞台全屏显示出现游戏崩溃问题：
- 错误信息：Stage is not defined
- 解决方案：增加前缀包名Laya
```
Laya.stage.scaleMode = Laya.Stage.SCALE_SHOWALL;
```

2. 第二个boss出现后，boss子弹发生了重叠：
- 解决方案：在生成每个boss前移除之前的boss
```
    /**
     * 根据关卡生成boss
     */
    function generateBoss(level,grade,hp,shootType){
        if(this.boss!=null){
            this.boss.removeSelf();
            //回收到对象池
            Laya.Pool.recover("role",this.boss);
        }
        isLastTime = true;
        //在循环中创建敌人
        Laya.timer.frameLoop(1,this,onLoop);
        this.boss = new Role();
        this.boss.init("boss"+level+"_"+grade,1,hp,0,60 );
        this.boss.shootType = shootType;
        this.boss.shootInterval = 800;
        //boss出现位置
        this.boss.pos(187, 100);
        this.boss.visible = true;
        //添加到舞台上
        this.roleBox.addChild(this.boss);
    }
```
<!--more-->

3. 第一个boss被击败后，需要根据第二个boss刷新攻击属性和伤害值：
```
   function appearBoss(){
        Laya.timer.clear(this, onNextBossWarning);
            removeEnemys();
            generateBoss(currentLevel,this.bossGrade,20,1);
            console.log("当前选中的技能是"+levelWeaponData[this.weapon_ready_list.selectedIndex]);
            if(levelWeaponData[this.weapon_ready_list.selectedIndex] != (currentLevel*10 + this.bossGrade)){
                //如果派出的boss与选中的武器属性不吻合，那么降低伤害值
                console.log("70%火力～");
                upgradeBullet(false);
            }else{
                console.log("100%火力～");
                upgradeBullet(true);
            }
   }
```

#### 2018/08/02
1. laya图集打包后，部分图片加载不出来的问题：
- 错误日志：
```

[warn]Retry to load: war/war/bg_second_level.png
[warn]Retry to load: war/hero_down1.png
[warn]Retry to load: war/hero_down2.png
[warn]Retry to load: war/hero_down3.png
[warn]Retry to load: war/hero_down4.png
[error]Failed to load: war/war/bg_second_level.png
```
- 问题原因：图集打包最大只允许2048x512，而最终打包后的图片大小超出了范围，导致无法显示出来。
- 解决方案：可以分开打包图集

#### 2018/08/03

1. 将boss的出现改在答题之前后，战机无法发射子弹或者一直warning：

- 解决方案：需要修改generateBoss方法逻辑，将继续游戏与boss出现功能代码分离，先在warning结束时生成boss，然后在答题结束后调用resume方法继续游戏。

2. 退出当前关卡后，进入下一关，然后再退出，游戏场景卡住不返回首页的问题：
- 问题原因：没有合适的移除UI资源
- 解决方案：关闭游戏场景前需要移除里面的背景资源、场景资源以及角色资源，并判断首页UI是否存在，如果存在需要回收，然后新建。
```
function onGameLeave(){
            dialog.close();
            console.log("结束游戏～");
             removeGame();
			 this.removeChildren();
			 this.removeSelf();
			if(!GameScene.homePage){
				GameScene.homePage = null;
			}
            GameScene.homePage = new HomePageUI();
			
			Laya.stage.addChild(GameScene.homePage);
        }
```


#### 2018/08/06
1. 给点击事件的回调方法添加参数回调：
```
btn_use.on(Laya.Event.CLICK, this,onUseEquipment, [index]);

......

/**
 * list中对应位置的使用按钮的点击事件处理
 * @param {*} index 
 */
 function onUseEquipment(index){
        console.log("当前使用了第"+index+"个装备");
                    
        for(var i=0; i<data.length; i++){
                        
            if(i === index){
                //设置选中的状态 
                console.log("当前boss级别->"+(currentLevel*10 + this.bossGrade)+"，当前选中的武器->"+levelWeaponData[index]);
                if((currentLevel*10 + this.bossGrade) === levelWeaponData[index]){
                    //使用的武器与boss是对应的
                    console.log("100%火力～");
                    upgradeBullet(true);
                }else{
                    console.log("70%火力～");
                    upgradeBullet(false);
                }
            }
        }
}
```

#### 2018/08/07
1. 开放域打包集成到主域后，出现以下问题：
- 日志信息：
```
gameSubContextThirdScriptError
wx.getFileSystemManager is not a function;at onMessage callback function
TypeError: wx.getFileSystemManager is not a function
```
- 问题原因：开放域中的图集资源并没有成功加载
- 解决方案：开放域中并不能加载图集资源，可以直接加载图片集
```
 _proto.loadResource = function(){
       
        Laya.loader.load(["comp/bg_line.png","comp/ranking1.png","comp/ranking2.png",
        "comp/ranking3.png","comp/userholder_img.png"], Laya.Handler.create(null,function(){
            console.log("开放域资源加载完毕～");
            //sample.dialog = new RankDialogUI();
            sample.rankView = new RankingViewUI();
            Laya.stage.addChild(sample.rankView);
        }));
    
    }

```

2. 开放域代码打包到微信小游戏项目中时无法运行的问题：
- 错误信息：
```
dispatchMessage is not defined;at onMessage callback function ReferenceError: dispatchMessage is not defined
```
- 解决方案：通过_proto来创建方法：
```
var _proto = LayaUISample.prototype;

 /**
     * 写入排行榜数据
     */
    _proto.writeRankingData = function(rankingData){
        //KVDataList代表排行数据,可以为多个,多个代表多个排行
        //key-排行类型,value-排行分数
        window['wx'].setUserCloudStorage({
            KVDataList: [
                //{ key: '击杀排行', value: "" + 1 },
                { key: '分数排行', value: "" + 100 },//需要改成动态的值
            ],
            success: function (res) {
                console.log('setUserCloudStorage', 'success', res)
            },
            fail: function (res) {
                console.log('setUserCloudStorage', 'fail')
            }
        });
    }
```

#### 2018/08/08
1. 开放域绘制的排行榜被主域的dialog覆盖的问题：
- 问题原因：开放域与主域的视图层级不一样，dialog的层级远远大于普通视图容器。
- 解决方案：dialog的默认层级zOrder为1000，最简单的方式是将shareCanvas的容器sprite的层级zOrder大于1000即可。
```
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
```

2. 提交排行榜数据时出现以下问题：
- 错误信息：setUserCloudStorage:fail invalid KVData item。
- 问题原因：提交数据的格式错误，key与value都应该是string类型
- 参考：https://developers.weixin.qq.com/minigame/dev/document/open-api/data/KVData.html

#### 2018/08/09
1. 小游戏分享失败的问题：
- 问题原因：在开放域分享用户排名后，提示无该方法，开放域官方只提供了几个有限的api接口，应该在主域去请求分享
```
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
				// 用户点击了“转发”按钮
				return {
					title: '我在飞机大战游戏中排名又上升了,快来挑战我吧～'
				}
			})
			window['wx'].shareAppMessage({

				title: '我在飞机大战游戏中排名又上升了,快来挑战我吧～',
				imageUrl: canvas.toTempFilePathSync({
					destWidth: 500,
					destHeight: 400
				})
			});
```

2. 提交排行榜分数：
```
    /**
     * 向开放域发送消息，并接收开放域返回过来的数据，
     * 可根据发送参数和接收数据在主域这边进行下步处理
     * @param message
     * @param caller
     * @param callback
     */
    function wxPostMessage(message, caller, callback){
        window['wx'].postMessage(message);
        Laya.timer.once(300, this, function (){
            //回调处理
            if (caller == null || caller == undefined) {
                callback(message);
            } else {
                caller.callback(message);
            }
        });
    }
    /**
     * 提交分数到微信服务器
     */
    function postScoreToWXServer(){
        wxPostMessage({
            command: 2,
            text: "提交玩家分数",
            rankingData: {
                fightScore:this.score*scoreFactor,
                fightTime:new Date().getTime()
            }
        }, null, function (message) {
            console.log("提交分数的回调");
            
        });
    }
```

#### 2018/08/10
1. 调取用户好友排行数据并排序整合后，得到的nickName为undefine：
- 问题原因：获取好友排行榜的数据中当前登录用户的昵称是空值
- 解决方案：先通过getUserInfo获取当前用户的数据，然后通过avatarUrl去查找好友排行榜中数据，如果相同，说明为同一个用户，将当前用户昵称复制给好友排行榜中当前用户的昵称。

2. 统计三关分数，并对它们的总分进行排序的解决思路：
>可以通过微信提供的排行榜分类统计功能，这样就无需理会玩家是一次性通关还是多次选择关卡通关的问题了。只要将玩家玩每一关通过时的分数通过key(关卡)、value(分值)键值对形式存储，然后每次重复选择都会刷新关卡的分值。而获取好友排行榜数据时，需要将分数先转化为number，然后累加进行排序，并展示即可。


3. 关闭排行榜后，再次打开，数据没了或者出现混乱的问题：
- 解决方案：需要重新清空重置数据，并渲染list：
```
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
```

4. 关于如何在laya游戏引擎中显示外部网页的问题：
- 方法一：通过设置href来添加外部网页资源。但这种方法无法关闭和设置外部网页的位置和大小。
```
Laya.Browser.window.location.href = "http://www.baidu.com";
```
- 方法二：通过iframe来添加外部网页并控制其位置、大小以及移除：
```
        var iframe = Laya.Browser.document.createElement("iframe");
        iframe.style.position ="absolute";//设置布局定位。这个不能少。
        iframe.style.zIndex = 1006;//设置层级
        iframe.style.left = "36px";
        iframe.style.top = "136px";
		iframe.width = 180;
		iframe.height = 220;
        iframe.src = "http://ask.layabox.com/";
        Laya.Browser.document.body.appendChild(iframe);
		function onHelpClose(){
			Laya.Browser.document.body.removeChild(iframe);;
		    dialog.close();
	    }
		
```

#### 2018/08/13
1. 无法加载微信用户头像的问题：
- 错误信息：
```
gameThirdScriptError
module "code.js" is not defined
Error: module "code.js" is not defined
```
- 解决方案：应该是变量的作用域不同造成的，需要定义一个外部的全局变量，并将this.userLogoImage赋值给它，然后在获取完毕微信头像信息后加载该全局变量即可。

2. 关于如何让laya的label内容超出空间大小后自动滚动的问题：
- 解决方案：可以通过Panel组件来包裹label控件，让其达到滚动效果。

3. 使用panel组件后，里面的label内容超过本身大小后无法滚动的问题：
- 解决方案：panel组件只是让其内部的控件在大小超过其容器(即panel)的时候，尽量达到滑动显示全部信息，所以，需要将里面的label控件高度设置尽量大，以保证文字内容全部显示出来。

4. 打包成微信小游戏后包太大无法预览上传的问题：
- 问题原因：微信限制上传的代码包不得超过4m。
- 解决方案：
  - 压缩图片资源
  - 去除index中没有使用到的library
  - 删除图集打包生成的多余的文件

5. 头像设置的遮罩无效的问题：
- 问题原因：为了降低包大小，不小心将webgl代码库注释了，导致mask遮罩无法渲染成功

#### 2018/08/14
1. ios手机中无法播放游戏声音的问题：
- 问题原因：laya引擎内部的方法播放音频只对Android手机有效，对ios的机型没有效果
- 解决方案：需要借助于微信自带的接口来播放音频
```
        var audio = wx.createInnerAudioContext()
        audio.src = "res/sound/game_over.mp3" // src 可以设置 http(s) 
        audio.obeyMuteSwitch = false
        audio.play();
```
2. 调试模式下可以正常显示用户的头像，但在非调试模式下无法显示用户头像问题：
- 问题原因：微信用户授权方法失效引起的
- 解决方案：
- 

3. 排行榜玩家昵称不显示问题以及排序错乱问题：
- 问题原因：没有正确复制好友排行榜的数据，头像链接、分数等都可以获取到，但是偏偏昵称获取为undefined，排行榜排序方式也有问题。
- 解决方案：由于直接拿到微信返回的排行榜数据中用户昵称是存在的，而通过变量赋值获取后便为undefined了，可以拿到原始数据res.data[i].nickname，然后复制给变量currentPlayer，再便遍历添加到数组集合。排序可以通过sort方法，通过比较目标集合中的两个任意对象的分值，从大到小排列：
```
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
```

4. 三张背景图打包图集后，有一张无法显示出来的问题：
- 问题原因：图集打包的图片超过了最大限制
- 解决方案：可以不打包图片，直接在bin中使用


#### 2018/08/15
1. laya绘制的好友排行榜在微信开发工具无法滑动的问题：
- 问题原因：主域和开放域的排行榜的位置坐标矩阵不同步造成的
- 解决方案：需要将主域和开放域的排行榜对应位置坐标矩阵同步
```
主域代码：
// 解决显示对象和鼠标错位而导致的排行榜滑动无效问题
		var globalPosition = dialog.ranking_list.localToGlobal(new Laya.Point());
		var originMatrix = Laya.stage._canvasTransform;
		var mat = new Laya.Matrix(originMatrix.a, 0, 0, originMatrix.d, globalPosition.x * originMatrix.a, globalPosition.y * originMatrix.d);
		//var mat = new Laya.Matrix(Laya.Browser.clientWidth/Laya.stage.width, 0, 0, Laya.Browser.clientHeight/Laya.stage.height);
		mat.translate(globalPosition.x * mat.a, globalPosition.y * mat.d);
		wxPostMessage({
            command: 0,
            text: "设置开放域canvas大小",
            canvasData: {
                width: rankViewWidth * mat.a, height: rankViewHeight * mat.d, matrix: mat
				//width:rankViewWidth, height:rankViewHeight, matrix: Laya.stage._canvasTransform
            },
            isLoad: false
        }, null, function (message) {
            console.log("再次往开放域发请求");
            window['wx'].postMessage({
                command: 1,
                text: '开放域加载资源',
            });
        });
        
开放域代码：
 var openMatrix = new Laya.Matrix();
 openMatrix.a = mainMatrix.a;
 openMatrix.b = mainMatrix.b;
 openMatrix.c = mainMatrix.c;
 openMatrix.d = mainMatrix.d;
 openMatrix.tx = mainMatrix.tx;
 openMatrix.ty = mainMatrix.ty;
 //重置矩阵
 Laya.stage._canvasTransform = openMatrix;
```

2. 主域修改了坐标矩阵的同步代码后，小游戏排行榜无法显示：
- 错误信息：
```
gameSubContextThirdScriptError
Cannot read property 'ranking_list' of undefined;at api getFriendCloudStorage success callback function
TypeError: Cannot read property 'ranking_list' of undefined
```
- 问题原因：laya平台自身问题，只是存在以下代码，开放域的排行榜便无法绘制：
```
// 解决显示对象和鼠标错位而导致的排行榜滑动无效问题
		......
		
		var mat = new Laya.Matrix(Laya.Browser.clientWidth/Laya.stage.width, 0, 0, Laya.Browser.clientHeight/Laya.stage.height);
		mat.translate(globalPosition.x * mat.a, globalPosition.y * mat.d);
		
		......
```

#### 2018/08/21
1. 将游戏发布到微信平台后，微信头像无法显示的问题：
- 问题原因：微信头像url跨域了
- 解决方案：获取到微信头像地址后，需要将其传到服务器，让服务器复制一份生成自己的服务器图片url
