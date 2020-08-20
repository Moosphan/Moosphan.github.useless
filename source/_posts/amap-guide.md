---
title: Android项目中调起手机地图导航
date: 2017-09-19 16:39:15
tags: 
- 地图导航
- 高德地图
categories: 
- Android
- 地图开发
toc: true
---
现在,移动应用中集成地图已经成为一种趋势。导航 - 作为地图中不可或缺的一项功能,被很多移动应用所青睐,然而,导航方式选择上,为了减少不必要的资源和apk容量,一般应用都选择通过调用第三方的地图应用来实现导航功能。在介绍之前,先看一下最终效果:
![未安装状态截图](http://upload-images.jianshu.io/upload_images/5256969-e4b6bf746c66c7b1.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![未安装地图应用](http://upload-images.jianshu.io/upload_images/5054046-f2388931682c6d1c.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
本文主要提供几种常用的调起三方地图应用的导航平台,以高德,百度,腾讯,谷歌地图为例。


地图平台  |  Uri文档 
---|---
高德地图 | [http://lbs.amap.com/api/uri-api/guide/travel/route](http://lbs.amap.com/api/uri-api/guide/travel/route)
百度地图 | [http://lbsyun.baidu.com/index.php?title=uri/api/android](http://lbsyun.baidu.com/index.php?title=uri/api/android)
谷歌地图 | [https://developers.google.com/maps/documentation/android-api/intents](https://developers.google.com/maps/documentation/android-api/intents)
腾讯地图 |[ http://lbs.qq.com/uri_v1/guide-route.html](http://lbs.qq.com/uri_v1/guide-route.html)

<!--more-->
各大地图服务商基本都提供了Uri,方便其他应用调用(除了腾讯),uri网址如上所示.

基本看了以上的文档就会使用了,下面就提供几个平台的基本写法吧:

- 高德地图导航:

高德地图较为特殊,其他地图平台都可以选择传入地址或者经纬度作为参数,而高德要求必须有经纬度。没办法,那就先撸个地理编码的轮子吧,方便我们将地址信息转化为准确的经纬度坐标,具体看下面代码:
```
/**
     * by moos on 2017/09/18
     * func:调用高德地图的地理编码接口,返回经纬度坐标
     * @param address
     */
    private void translateAddressToLocation(String address){


        OkHttpUtils
                .get()
                .url(GEOCODE_HTTP_URL + "address="+address+"&key="+SELF_AMAP_KEY)
                .build()
                .execute(new StringCallback() {
                    @Override
                    public void onError(Call call, Exception e, int id) {
                        Log.e("地理编码失败=", e.getMessage());
                        //DialogUtils.dismissProgressDialog();
                        Toast.makeText(UnityScanActivity.this, getString(R.string.act_qr_code_fail), Toast.LENGTH_LONG).show();
                    }
                    @Override
                    public void onResponse(String response, int id) {

                        Logger.e("地理编码结果 =" + response);
                        resultBean = JSONObject.parseObject(response, GeocodeResultBean.class);
                        if (resultBean.getStatus().equals("1")) {

                            locationList = resultBean.getGeocodes();
                            if(locationList!=null && locationList.size()>0){
                                //默认获取第一条
                                locationString = locationList.get(0).getLocation();
                                lon = locationString.substring(0,locationString.indexOf(","));
                                lat = locationString.substring(locationString.indexOf(",")+1,locationString.length());
                                Log.e("经纬度信息为===",lon+"======="+lat);
                            }else {
                                Toast.makeText(UnityScanActivity.this, "无相关信息", Toast.LENGTH_LONG).show();
                            }


                        } else {
                            Toast.makeText(UnityScanActivity.this, resultBean.getInfo(), Toast.LENGTH_LONG).show();
                        }

                    }
                });

    }

```
&emsp;&emsp;这里我并没有采用高德包中的地理编码接口来实现,而是调用的其web接口。请原谅我的任性,最近接口调上瘾了😆,值得注意的是,url中需要传入key,这个要在高德官网自己创建web应用申请,不过很方便,不需要包名和sha1等。

下面看看如何调起手机中的高德地图吧:
```
//调起导航的uri
    private final String BAIDU_MAP_NAVI_URI = "baidumap://map/navi?query=";
    private final String GAODE_MAP_NAVI_URI = "androidamap://navi?sourceApplication=";
    private final String GOOGLE_MAP_NAVI_URI = "google.navigation:q=";
    //map app包名
    private final String BAIDU_MAP_APP = "com.baidu.BaiduMap";
    private final String GAODE_MAP_APP = "com.autonavi.minimap";
    private final String GOOGLE_MAP_APP = "com.google.android.apps.maps";

    private final String QQ_MAP_URL = "http://apis.map.qq.com/uri/v1/routeplan?type=drive&";
    //高德web服务的临时key(用于地理编码)
    private final String SELF_AMAP_KEY = "your key";
    private final String GEOCODE_HTTP_URL = "http://restapi.amap.com/v3/geocode/geo?";
    private GeocodeResultBean resultBean;
    private List<GeocodeResultBean.GeocodeBean> locationList;
    private String locationString;
    private String lon;
    private String lat;
/**
     * by moos on 2017/09/18
     * func:调起高德导航
     * @param lat 纬度
     * @param lon 经度
     * @param dev 是否偏移(0:lat 和 lon 是已经加密后的,不需要国测加密; 1:需要国测加密)
     * @param style 导航方式(0 速度快; 1 费用少; 2 路程短; 3 不走高速；4 躲避拥堵；5 不走高速且避免收费；6 不走高速且躲避拥堵；7 躲避收费和拥堵；8 不走高速躲避收费和拥堵)
     */
    private void goNaviByGaoDeMap(String lat,String lon,String dev,String style){

        Intent intent = new Intent();
        intent.setAction(Intent.ACTION_VIEW);
        intent.addCategory(Intent.CATEGORY_DEFAULT);
        //将功能Scheme以URI的方式传入data
        Uri uri = Uri.parse(GAODE_MAP_NAVI_URI +
                 "Test"
               + "&lat=" + lat
               + "&lon=" + lon
               + "&dev=" + dev
               + "&style=" +style);
        intent.setData(uri);
        intent.setPackage("com.autonavi.minimap");
        startActivity(intent);
    }

```
注释比较详细了,就不做具体讲解了,可以参考官方提供的文档。

- 百度地图导航:
```
/**
     * by moos on 2017/09/18
     * func:调起百度导航
     */
    private void goNaviByBaiDuMap(String address){

        Intent intent = new Intent();
        intent.setData(Uri.parse(BAIDU_MAP_NAVI_URI + address));
        startActivity(intent);

    }

```

- 谷歌地图导航:
```
/**
     * by moos on 2017/09/18
     * func:调起谷歌导航
     * @param lat
     * @param lon
     */
    private void goNaviByGoogleMap(String lat,String lon,String address){

        Uri uri = Uri.parse(GOOGLE_MAP_NAVI_URI+lat+","+lon +"," + address);
        Intent mapIntent = new Intent(Intent.ACTION_VIEW, uri);
        mapIntent.setPackage("com.google.android.apps.maps");
        context.startActivity(mapIntent);
    }
```

- 腾讯地图导航:
&emsp;&emsp;刚刚说过,腾讯地图目前还未支持其他应用以uri形式调起腾讯自家的地图功能,不过我们可以通过web 接口方式调用,具体代码如下,比较简单:
```
/**
     * by moos on 2017/09/18
     * func:通过网页形式调起腾讯地图
     * @param startPoint
     * @param endPoint
     * @param policy 规划策略(0:快捷,1:无高速,2:距离短)
     * @param appName 当前应用的名称
     */
    private void goNaviByTencentMap(String startPoint,String endPoint,String policy,String appName){

        Uri uri = Uri.parse(QQ_MAP_URL + "from="+startPoint + "&to=" +endPoint + "&policy="+policy + "&referer="+appName );
        Intent intent = new Intent(Intent.ACTION_VIEW,uri);
        startActivity(intent);
    }
```
我们只需要在url中传入参数,然后访问该网址就可以,很简单吧😆。

- 其他:
&emsp;&emsp;为了更好的用户体验,我们会加上一些提醒用户的交互,如:用户手机没有安装地图应用,可以弹窗让用户前往商店下载,来吧,一大波代码来袭:
```
/**
     * by moos on 2017/09/18
     * func:接受导航请求,并发起导航
     */
    private void navigateForDestination(){
        if(isApplicationInstall(GAODE_MAP_APP)){
            //安装了高德map
            Log.e("Tminstore","已安装高德地图");
            translateAddressToLocation("东方明珠");//地理编码
            goNaviByGaoDeMap(lat,lon,"1","2");
            //goNaviByTencentMap(MyApplication.mapLocation.getAddress(),"上海东方明珠","0","com.ucardstore.Activity");

        }else if(isApplicationInstall(BAIDU_MAP_APP)){
            //安装了百度map
            Log.e("Tminstore","已安装百度地图");
            goNaviByBaiDuMap("上海东方明珠");

        }else if(isApplicationInstall(GOOGLE_MAP_APP)){
            //安装了google map
            Log.e("Tminstore","已安装谷歌地图");
            goNaviByGoogleMap("","","上海东方明珠");

        }else {

            //默认安装高德app
            showInstallAppTip();

            //使用腾讯网页地图(可选)
            //goNaviByTencentMap(MyApplication.mapLocation.getAddress(),"上海东方明珠","0","com.ucardstore.Activity");
        }
    }
    
    .....
    
    /**
     * by moos on 2017/09/18
     * func:判断手机是否安装了该应用
     * @param packageName
     * @return
     */
    private boolean isApplicationInstall(String packageName){
        return new File("/data/data/" + packageName).exists();
    }
    
    ......
    
    /**
     * by moos on 2017/09/18
     * func:安装地图app的提示
     */
    private void showInstallAppTip(){
        TipDialog dialog = new TipDialog(this, new View.OnClickListener() {

            @Override
            public void onClick(View arg0) {
                switch (arg0.getId()) {
                    case R.id.dialog_btn_sure:

                        Intent intent ;
                        Uri uri = Uri.parse("market://details?id=com.autonavi.minimap");
                        intent = new Intent(Intent.ACTION_VIEW, uri);
                        startActivity(intent);

                        break;
                    case R.id.dialog_btn_cancel:

                        break;
                }
            }
        });


        dialog.setMessage(getString(R.string.install_app_tip));
        dialog.setBtnSure(getString(R.string.go_to_install));
        dialog.setBtnCancel(getString(R.string.cancel));
        dialog.show();
    }
```

好了,这差不多就是全部代码了,至于弹窗TipDialog是我自定义的dialog,没什么难度,就不贴出来😝。
