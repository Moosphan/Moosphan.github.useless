---
title: 微信小游戏开发笔记（上）
date: 2018-08-31 15:19:46
tags: 
- 微信小游戏
- Layabox
categories: 微信小游戏
toc: true
---
好久不见～最近两个月一直在忙公司的微信小游戏项目，没时间打理博客，很多人留言可能没有及时回复，接下来我将继续和大家探索Android方面的好玩的东西。之前没有接触过游戏开发和前端开发，一开始上手可能较为困难。微信小游戏一般要借助于主流的游戏引擎进行开发，常见的有laya、cocos、egret等，这里我选用了laya游戏引擎，laya相对于其他游戏引擎来说，文档记录可能不够全面，而且社区问题反馈效率较低，但好在功能较为齐全，需要自己花时间研究，如有想开发小游戏的可以自行选择。

本来想要按照开发流程写一个系列教程，但是发现这样写下去篇幅较长，而且开发过程中依靠的大多还是官方的文档和之前累积的开发经验，这里还是先放一下日常的开发文档吧，后续将写文章记录一下微信排行榜绘制等难点。
<!--more-->
#### 2018/07/04

1. 修改Layabox的初始化页面：
>以actionScript为例，需要修改actionScriptProperties、asconfig.json、xxx.as3proj、xxx.max.js中的target关键字，target指代目标的替换初始化页面。

2. 将laya项目打包出微信小游戏项目时出错：
>错误信息：[ERR]|FileNotFound: /Users/moos/Documents/Emerson/EmersonGame/src/LayaUISample.js。

>解决方案：全局搜索关键词并删除该相关类信息

#### 2018/07/05
1. layabox简单适配：
- 边距位置适配
>left、right、top、bottom四个属性分别基于父容器的左边缘、右边缘、上边缘、下边缘。这里特别要注意的是，left、right、top、bottom的属性效果是基于父容器（页面）的各个边缘，而不是屏幕的各个边缘。父容器（页面）的分辨率一定要与项目中Laya.init()设置的分辨率相同，如果没有设置成相同，那么并不能实现效果。
- 边距的拉伸适配
>除了居于某一个边缘的适配作用外，同时设置left、right、top、bottom的属性值，还可以根据不同屏幕对组件进行拉伸适配。Tips：拉伸适配的边距设置方式通常需要结合九宫格来实现。
- 中心位置适配
>中心适配常用于基于屏幕中间的游戏启动LOGO，弹出提示框等。我们可以通过centerX、centerY进行位置居中设置.

2. bin目录下的index文件中的js文件声明顺序决定了UI渲染的顺序。
```
<script src="../src/GameInfo.js"></script>
	<script src="../src/Role.js"></script>
	<script src="../src/BackGround.js"></script>
	<script src="../src/Game.js"></script>
```

3. layaUI.max.all.js文件中加载UI的图片资源是bin目录下的图集。注意：如果没有特别设置，创建的UI组件都会被打包成图集存储在bin/res/atlas目录下。


#### 2018/07/09
1. UI组件的图片资源加载失败的问题：
>日志信息：
```
    lose skin template/StartDialog/btn_dialog_close.png
    lose skin template/StartDialog/btn_start_game.png
```
>问题原因：没有待资源加载完毕就将组件添加到舞台了。
>解决方法：在添加UI到舞台之前需要预加载UI图集资源
```
function beginLoad(){
	Laya.loader.load("res/atlas/template/HomePage.atlas", Handler.create(null, onLoaded));
	// 预加载弹窗的UI资源
	Laya.loader.load("res/atlas/template/StartDialog.atlas", Handler.create(null, onLoaded));
}
```
2. 加载dialog时出现崩溃异常：
>异常信息：dialog is not defined

>解决方案：dialog并不属于HomePageUI
```
function onSecondLevelLoad()
	{
		console.log("第二关准备中～");
		var dialog = new StartDialogUI();
		Laya.stage.addChild(dialog);
		dialog.popup();
		dialog.title_start_dialog.font
		dialog.title_start_dialog.text = "第二关";
		dialog.btn_start_game.on(Laya.Event.CLICK, this, onGameStart);
		dialog.btn_start_dialog_close.on(Laya.Event.CLICK, this, onDialogClose);
		function onDialogClose(){
		    console.log("关闭对话框～");
		    dialog.close();
	    }

	}
```

3. 新建GameScene并添加到舞台后，出现如下异常：
>异常信息：
```
"TypeError: Cannot read property 'call' of undefined    at Function.<anonymous>

(file:///Users/moos/Company/Emerson-AirWar/EmersonGame/bin/libs/laya.core.js:130:41)    at new GameScene

(file:///Users/moos/Company/Emerson-AirWar/EmersonGame/src/GameScene.js:4:19)    at HomePageUI.onGameStart

(file:///Users/moos/Company/Emerson-AirWar/EmersonGame/src/HomePage.js:78:27)    at EventHandler.__proto.runWith

(file:///Users/moos/Company/Emerson-AirWar/EmersonGame/bin/libs/laya.core.js:724:59)    at Button.__proto.event

(file:///Users/moos/Company/Emerson-AirWar/EmersonGame/bin/libs/laya.core.js:499:30)    at TouchManager.__proto.sendEvents

(file:///Users/moos/Company/Emerson-AirWar/EmersonGame/bin/libs/laya.core.js:3340:7)    at TouchManager.__proto.onMouseUp

(file:///Users/moos/Company/Emerson-AirWar/EmersonGame/bin/libs/laya.core.js:3499:10)    at MouseManager.__proto.onMouseUp

(file:///Users/moos/Company/Emerson-AirWar/EmersonGame/bin/libs/laya.core.js:3043:18)    at MouseManager.__proto.check

(file:///Users/moos/Company/Emerson-AirWar/EmersonGame/bin/libs/laya.core.js:3070:13)    at MouseManager.__proto.check

(file:///Users/moos/Company/Emerson-AirWar/EmersonGame/bin/libs/laya.core.js:3063:15)"
```
>问题原因：
cmd+点击GameSceneUI跳转不到定义的地方，应该是找不到GameSceneUI定义，将ui.去掉即可，修改所在目录。


#### 2018/07/10
1. 图集资源加载完毕的回调函数没有回调问题：
```
        //加载图集资源
        Laya.loader.load("res/atlas/war.json",Laya.Handler.create(this,this.onLoaded),null,Laya.Loader.ATLAS);
        function onLoaded(){
        console.log("准备加载背景了～");
        }
```
>解决方案：需要将onloaded方法移到Game()方法的内部，否则无法找到onloaded方法。

2. 游戏运行成功后，游戏里的角色图片素材都发生了破碎和模糊问题：
>问题原因：游戏中的角色素材是以打包图集的方式被游戏加载出来的，在未知原因下，图集中资源个数以及位置发生了变化，导致之前图集json文件中位置参数等无效，所以，需要将图集资源图片替换成之前的内容。


#### 2018/07/12
1. 打包assets下的war文件夹中的图集后并没有在对应的文件夹下生成图集资源问题：
>问题原因：资源路径需要选择图片所在的上上级文件夹才能生成。

>解决方案：选择assets目录作为要打包的图片资源的目标路径。

2. 加载progressbar失败问题：
>日志信息：
```
[warn]Retry to load: GameScene/progress.png
[warn]Retry to load: GameScene/progress$bar.png
[error]Failed to load: GameScene/progress.png
[error]Failed to load: GameScene/progress$bar.png
```

>解决方案：问题出在ui编辑页面的progressBar的skin资源路径不全，需要包含其根路径。


### 2018/07/16
1. 使用定时器来实现警告效果，无法关闭定时器问题：
>注册定时器的时候和移除定时器时绑定的回调函数需要是同一个：
```
proto.startWarning = function(){
        Laya.timer.frameLoop(10,this,onWarn);
        
    }
    
.......

function onWarn(){
        this.warningTime++;
        if(this.warningState){
            hideWarning();
        }else{
            showWarning();
        }
        if(this.warningTime === 5){
            
            console.log("该结束警告了～");
            Laya.timer.clear(this, onWarn);
            this.warningState = false;
            this.warning_title.visible = false;
            this.warning_content.visible = false;
        }
        
    }
```

2. UI编辑页面的右侧控件属性栏无法显示的问题：
>点击窗口栏选择重置面板即可

3. 实现点击弹窗dialog外部，不关闭dialog：
>需要在laya全局初始化之前设置以下属性：
```
UIConfig.closeDialogOnSide = false;
```
4. 自定义了RadioGroup后，发现radio的状态图片资源并没有出现：
>问题原因：没有预加载dialog的UI资源，导致图片加载失败，在首页预加载以下就可以了：
```
    Laya.loader.load("res/atlas/template/StartDialog.atlas", Handler.create(null));
	Laya.loader.load("res/atlas/template/LeaveDialog.atlas", Handler.create(null));
	Laya.loader.load("res/atlas/template/RankDialog.atlas", Handler.create(null));
	Laya.loader.load("res/atlas/template/QuestionDialog.atlas", Handler.create(null));
```

#### 2018/07/19
1. laya自定义RadioGroup后，每个radio的icon的skin资源无法加载问题：
>问题原因：laya中自定义radioGroup如果要设置每个选项的文字信息，不能通过指定radioGroup的labels属性来实现，否则icon不会显示。
>解决方案：可以通过获取radioGroup的每一个radio来动态为它们设置label：
```
function completeHandler(e){
                dialog.title_question_dialog.visible = false;
                dialog.radioGroup_question.visible = true;
                console.log(e);
                console.log(e.result.question);
                
                answer = e.result.answerIndex;
                console.log(answer);
                dialog.subtitle_question_dialog.text = e.result.question;
                dialog.radioGroup_question.getChildAt(0).label = e.result.choices[0].content;
                dialog.radioGroup_question.getChildAt(1).label = e.result.choices[1].content;
                dialog.radioGroup_question.getChildAt(2).label = e.result.choices[2].content;
                dialog.radioGroup_question.getChildAt(3).label = e.result.choices[3].content;
            }
```

2. laya中网络请求的正确姿势：
>查看官方文档，并按照文档中教程来，发现网络请求虽然进行了，但无法回调获取到数据，可以通过以下方式：
```
//网络请求答题数据
    var xhr = new Laya.HttpRequest();
    xhr.http.timeout = 10000;//设置超时时间；
            xhr.on('complete',this,completeHandler);
            xhr.on('error',this,errorHandler);
            xhr.send("http://www.wanandroid.com/tools/mockapi/4080/question","","get","json");
            function completeHandler(e){
                dialog.title_question_dialog.visible = false;
                dialog.radioGroup_question.visible = true;
                console.log(e);
                console.log(e.result.question);
                
                answer = e.result.answerIndex;
                console.log(answer);
                dialog.subtitle_question_dialog.text = e.result.question;
                dialog.radioGroup_question.getChildAt(0).label = e.result.choices[0].content;
                dialog.radioGroup_question.getChildAt(1).label = e.result.choices[1].content;
                dialog.radioGroup_question.getChildAt(2).label = e.result.choices[2].content;
                dialog.radioGroup_question.getChildAt(3).label = e.result.choices[3].content;
            }
            function errorHandler(data){
                console.log("网络出错了哟，先检查一下网络吧～");
            }


```


#### 2018/07/20
1. laya游戏打包成微信小游戏后，运行失败：
>错误信息：gameThirdScriptError
window.focus is not a function
TypeError: window.focus is not a function。

>问题原因：使用laya自带的httpRequest请求网络数据后出现该问题

2. 子弹的处理逻辑：

英雄战机子弹 | Boss子弹| 碰撞结果
---|---|---
1  | 0 | 1
0  | 1 | 1
1  | 1 | 0
0  | 0 | 1

#### 2018/07/26
1. 游戏结束重新开始游戏后，boss生命值没有被重置的问题：
>问题原因：boss角色对象被回收，hp属性值变为0，导致boss瞬间死亡

>解决方案：
- 回收方案：将boss回收后，可以通过判断当前是否存在boss，并且hp值小于1时才提示；
- 非回收方案：不回收的话需要及时重置boss的血量值，restart过程中设置，并隐藏boss，并利用isCanAttack来判断当前是否可以被攻击，需要增加一些额外逻辑，因为，boss即使设置了visible属性，攻击boss位置，依旧会掉血。

2. laya中无法渲染list的问题：
>解决方案：需要现将目标的那些组件打包成box类型，然后为其指定renderType为render。
```
    dialog.equipment_list.array = data;
    dialog.equipment_list.renderHandler = new Handler(this, onRender);
    
    function onRender(cell, index){
                console.log("装备列表的渲染回调");
                if(index > data.length)return;
                //获取当前渲染条目item的数据
                var equipmentData = data[index];
                //根据子节点的名字weapon，获取子节点对象
                //var weaponData = cell.getChildByName("weapon");
                var weaponData = equipmentData.weapon;
                //获取子条目各项数据
                var weaponImg = weaponData.res;
                var weaponName = weaponData.name;
                console.log("装备列表中武器名称="+weaponName+",武器资源="+weaponImg);

        }
```

#### 2018/07/27
1. 根据答题逻辑和打boss等逻辑需要，需要后台返回的随机问题数据格式如下：
```
{
		"code":200,
		"message":"success",
		"result":{
			"question":"调整型孔板流量计中间不开孔的主要原因是？",
			"answerIndex":1,
			"choices":[
				{
					"choiceId":0,
					"choiceHead":"A",
					"content":"中间开孔不好看"
				},
				{
					"choiceId":1,
					"choiceHead":"B",
					"content":"中间开孔抗干扰能力差"
				},
				{
					"choiceId":2,
					"choiceHead":"C",
					"content":"中间开孔不利于安装"
				},
				{
					"choiceId":3,
					"choiceHead":"D",
					"content":"以上都不正确"
				}
			],
			"weaponType":"11",
			"weaponName":"内藏孔板(变送器)",
			"weaponInfo":"流量量程比大于10：1，四螺栓设计，孔板沟槽密封和对中，带内抛光直管段，耐压900#（过程连接1500#），计算标准兼容ASME MFC-14M和SO/TR15377"
		}

	}
```

2. 筛选出重复问题及其对应的重复武器数据，防止list中重复展示：
```
/**
         * 获取装备数据
         */
        function getListData(){
            data = [];
            
            //根据当前关卡来判断存在哪些武器
            levelWeaponData = [];
            if(currentLevel === 1){
                console.log("加载第一关武器数据");
                levelWeaponData.push(11);
                levelWeaponData.push(12);
            }else if(currentLevel === 2){
                console.log("加载第二关武器数据");
                levelWeaponData.push(21);
                levelWeaponData.push(22);
                levelWeaponData.push(23);
            }else if(currentLevel === 3){
                console.log("加载第三关武器数据");
                levelWeaponData.push(31);
                levelWeaponData.push(32);
                levelWeaponData.push(33);
            }
            console.log("levelWeaponData数量为="+levelWeaponData.length);
            console.log("answerData数量为="+answerData.length);
            if(answerData.length > 0){
                for(var i = 0;i<levelWeaponData.length;i++){
                    //根据关卡筛选数据，需要防止重复性数据
                    console.log("当前关卡中武器代号为="+levelWeaponData[i]);
                    for(var j = 0;j<answerData.length;j++){
                        //如果已获取的武器里存在当前关卡武器中，那么存入data
                        var currentAnswer = answerData[j].weapon.code;
                        
                        if(currentAnswer === levelWeaponData[i]){
                            console.log("当前装备代号为="+currentAnswer);
                            data.push(answerData[j]);
                            //一旦查询到数据，关闭二级循环，防止数据重叠
                            break;
                        }
                    }
                }
            }
            console.log("列表数据数目为="+data.length);
            dialog.equipment_list.array = data;
            dialog.equipment_list.renderHandler = new Handler(this, onRender);

        }
```

#### 2018/07/30
1. 问题选项无法重复选中的问题：
>问题原因：由于radiogroup采用了复用原则，所以，第二次展示答题卡选项时依旧会记录上次的选择。

>解决方案：只需要在每次点击下一题按钮时重置选中状态即可：
```
dialog.radioGroup_question.selectedIndex = -1;

```

2. 技能栏的实现：
>可以通过listview实现，监听每个item的选中状态：
```
//显示待使用的武器技能列表
    if(this.weaponCount > 0){
        this.weapon_ready_list.visible = true;
        this.weapon_ready_list.array = data;
        this.weapon_ready_list.selectEnable = true;
        this.weapon_ready_list.renderHandler = new Handler(this, onWeaponRender);
        this.weapon_ready_list.selectHandler = new Handler(this, onSelect);
    }
    
    /**
      * item 的渲染处理
      * @param {*} cell 
      * @param {*} index 
      */
    function onWeaponRender(cell,index){
            console.log("渲染待使用武器列表的回调～");
            if(index > data.length)return;
            //获取当前渲染条目item的数据
            var equipmentData = data[index];
            //根据子节点的名字weapon，获取子节点对象
            var weaponData = equipmentData.weapon;
            //获取子条目各项数据
            var weaponImg = weaponData.res;
            var weaponName = weaponData.name;
            console.log("装备列表中武器名称="+weaponName+",武器资源="+weaponImg);
            var icon = cell.getChildByName("icon_weapon");
            icon.skin = weaponImg;
    }
    /**
      * item的点击选中处理
      * @param {*} index 
      */
    function onSelect(index){
        console.log("当前选中了第"+index+"个武器");
        for(var i=0; i<data.length; i++){
            var cell = this.weapon_ready_list.getCell(index);
            var bg_weapon = cell.getChildByName("bg_weapon");
                        
            if(i === index){
                //设置选中的状态
                bg_weapon.skin = "template/GameScene/bg_weapon_selected.png";
            }else{
                //设置未选中状态
                bg_weapon.skin = "template/GameScene/bg_weapon_unselected.png";
            }
        }
                    
    }
                    
                
```

3. 重置list的item的选中状态时游戏崩溃问题：
>日志信息："Cannot read property 'getChildByName' of null"。

>解决方案：将获取index位置的item容器改为获取第i位置的：
```
/**
                 * item的点击选中处理
                 * @param {*} index 
                 */
                function onSelect(index){
                    console.log("当前选中了第"+index+"个武器");
                    for(var i=0; i<data.length; i++){
                        var cell = this.weapon_ready_list.getCell(i);
                        var bg_weapon_img = cell.getChildByName("bg_weapon");
                        
                        if(i === index){
                            //设置选中的状态
                            bg_weapon_img.skin = "template/GameScene/bg_weapon_selected.png";
                        }else{
                            //设置未选中状态
                            bg_weapon_img.skin = "template/GameScene/bg_weapon_unselected.png";
                        }
                    }
                    
                }
```

#### 2018/07/31
1. laya请求后台的问题接口后无法获取数据：
>需要后台处理一下跨域的问题

2. 排行榜UI实现：
>通过laya提供的UI组件list来实现，判断item位置进而进行差异化渲染，从而实现前三名的独特效果。此外，圆形头像需要通过mask蒙版来进行处理。

3. 装备列表只渲染一次的问题：
>问题原因：没有设置repeatx和repeaty，而且list的box设置的高度太大，导致渲染一次。

>解决方案：设置repeatx和repeaty属性，代表横向和纵向列表中item的数量






