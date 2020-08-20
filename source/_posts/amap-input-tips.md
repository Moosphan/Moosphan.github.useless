---
title: Android基于高德地图实现搜索框的自动输入提示功能
date: 2017-09-24 16:37:26
tags: 
- 高德地图
- 输入提示
categories: 
- Android
- 地图开发
toc: true
---
  最近公司项目中一直在搞地图开发,今天产品经理就给我布置了一些(无法想象)任务,其中一个就是实现地点搜索输入框的自动输入提示功能。拿到任务肯定想讨价还价一番,但是想到以前也写过,就不再负隅顽抗了。
  以前在学校的时候实现过类似功能,是使用高德自带的InputtipsListener来实现的,想了解可以看看:[文章传送点](http://blog.csdn.net/s1674521/article/details/61914859),这里就不详细介绍了。作为一名头脑发热的开发者,肯定不能安于现状,这里主要介绍其他两种方式 - poi实现和http请求接口实现,不管能不能成功,试了再说,撸起袖子就是干。先看看最终的效果:

![搜索历史效果](http://upload-images.jianshu.io/upload_images/5256969-927cb389ce13c524.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![关键词搜索](http://upload-images.jianshu.io/upload_images/5256969-f51bda6ea78060c4.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<!--more-->

  做之前先分析一下功能需求,首先输入框中要添加内容清除的icon,当输入框有文字时,需要显示,为空时隐藏;接着,需要实现地址搜索功能并通过listview展示结果;最后需要实现展示搜索历史的功能。好的,那么下面我们来一步步实现。

  其实,实现效果中的输入框并不难,只需要三个东西就够了:LinearLayout,EditText,ImageView。直接上代码吧,上了代码你就知道它到底有多简单了:
```
<LinearLayout
            android:layout_width="wrap_content"
            android:layout_height="36dp"
            android:layout_weight="1"
            android:layout_marginLeft="20dp"
            android:background="@drawable/search_view_bg"
            android:orientation="horizontal"
            android:gravity="center_vertical">
            <EditText
                android:id="@+id/search_edit_text"
                android:layout_width="wrap_content"
                android:layout_height="36dp"
                android:hint="@string/input_cross_location"
                android:textColorHint="#9B9B9B"
                android:textSize="12sp"
                android:maxLines="1"
                android:layout_weight="1"
                android:paddingBottom="10dp"
                android:paddingTop="10dp"
                android:paddingLeft="10dp"
                android:background="@drawable/search_edit_bg"
                android:drawableLeft="@mipmap/icon_edit_search"
                android:drawablePadding="16dp"/>
            <ImageView
                android:id="@+id/search_edit_delete"
                android:layout_width="12dp"
                android:layout_height="12dp"
                android:layout_marginLeft="5dp"
                android:layout_marginRight="8dp"
                android:visibility="gone"
                android:src="@mipmap/iocn_search_cancel"/>

        </LinearLayout>
```
  没错,这里为EditText父容器LinearLayout设置背景,然后EditText设置同样的背景,只不过需要将右边的圆角效果去掉,达到预期效果。也即是说,我们的输入框相当于是LinearLayout,里面包含了edittext和删除图标imageview,来看看drawable的代码吧:
>search_view_bg:
```
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:state_window_focused="false">
        <shape xmlns:android="http://schemas.android.com/apk/res/android">
            <!--<solid android:color="#F4F4F4" />-->
            <corners android:radius="3dp"/>
            <solid android:color="#F3F3F3"/>
            <!--<stroke android:color="#ffececec" android:width="1dp"/>-->
        </shape>
    </item>
    <item android:state_window_focused="true">
        <shape>
            <corners android:radius="3dp"/>
            <!--<stroke android:color="#ececec" android:width="1dp" />-->
            <solid android:color="#F3F3F3"/>
            <!--<solid android:color="#F4F4F4" />-->
        </shape>
    </item>
</selector>
```

> search_edit_bg:
```
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:state_window_focused="false">
        <shape xmlns:android="http://schemas.android.com/apk/res/android">
            <!--<solid android:color="#F4F4F4" />-->
            <corners
                android:topLeftRadius="3dp"
                android:bottomLeftRadius="3dp"/>
            <solid android:color="#F3F3F3"/>
            <!--<stroke android:color="#ffececec" android:width="1dp"/>-->
        </shape>
    </item>
    <item android:state_window_focused="true">
        <shape>
            <corners
                android:topLeftRadius="3dp"
                android:bottomLeftRadius="3dp"/>
            <!--<stroke android:color="#ececec" android:width="1dp" />-->
            <solid android:color="#F3F3F3"/>
            <!--<solid android:color="#F4F4F4" />-->
        </shape>
    </item>
</selector>
```
ok,这就实现了最终的输入框UI,当然,你可以使用其他方式实现,比如自定义view,第三方开源等等,但我觉得这完全满足我们的需求,而且简单,不是吗?接下来,我们需要通过监听EditText的变化来实现搜索框中删除的变化,代码如下:
```
    @Bind(R.id.search_edit_text)
    EditText inputText;
    @Bind(R.id.search_edit_delete)
    ImageView buttonDelete;
    
    ......
    
    buttonDelete.setOnClickListener(this);
    inputText.addTextChangedListener(this);
        
    ......
    
    @Override
    public void beforeTextChanged(CharSequence charSequence, int i, int i1, int i2) {

    }

    @Override
    public void onTextChanged(CharSequence charSequence, int i, int i1, int i2) {
        if(charSequence!=null){
             buttonDelete.setVisibility(View.VISIBLE);
            
        }else {
             buttonDelete.setVisibility(View.GONE);
            
        }

    }

    @Override
    public void afterTextChanged(Editable editable) {


    }
    
```
代码比较简单,就不解释了,不理解这个方法的可以谷歌一下,我们接着往下看。

  用过高德地图api的开发者应该都知道里面有个常用的功能:POI搜索.高德提供了千万级别的 POI（Point of Interest，兴趣点）。在地图表达中，一个 POI 可代表一栋大厦、一家商铺、一处景点等等。通过POI搜索，完成找餐馆、找景点、找厕所等等的功能。如果我们的需求是获取周围兴趣点,那么搜索输入提示只要显示兴趣点就可以了。
  下面我们来依次通过两种方法来实现快捷输入提示功能:

- POI搜索实现:

话不多说,直接上代码:
```
//方法一:使用poi搜索接口方法
    private PoiResult poiResult; // poi返回的结果
    private int currentPage = 0;// 当前页面，从0开始计数
    private PoiSearch.Query query;// Poi查询条件类
    private LatLonPoint latLonPoint;
    private PoiSearch poiSearch;
    private List<PoiItem> poiItems;// poi数据
    private String keyWord;
    private CommonAdapter adapter;
    private final int ADDRESS_LOCATION_GET = 3242;
    private String POI_SEARCH_TYPE = "汽车服务|汽车销售|" +
            "//汽车维修|摩托车服务|餐饮服务|购物服务|生活服务|体育休闲服务|医疗保健服务|" +
            "//住宿服务|风景名胜|商务住宅|政府机构及社会团体|科教文化服务|交通设施服务|" +
            "//金融保险服务|公司企业|道路附属设施|地名地址信息|公共设施";
            
    ......
    
    /**
     * 开始进行poi搜索
     */
    protected void doSearchQuery() {
        latLonPoint = new LatLonPoint(MyApplication.mapLocation.getLatitude(), MyApplication.mapLocation.getLongitude());// 116.472995,39.993743

        keyWord = inputText.getText().toString().trim();
        currentPage = 0;
        //keyWord表示搜索字符串，
        //第二个参数表示POI搜索类型，二者选填其一，选用POI搜索类型时建议填写类型代码，码表可以参考下方（而非文字）
        //cityCode表示POI搜索区域，可以是城市编码也可以是城市名称，也可以传空字符串，空字符串代表全国在全国范围内进行搜索
        query = new PoiSearch.Query(keyWord, POI_SEARCH_TYPE, "");
        query.setPageSize(30);// 设置每页最多返回多少条poiItem
        query.setPageNum(currentPage);// 设置查第一页
        if (latLonPoint != null) {
            poiSearch = new PoiSearch(this, query);
            poiSearch.setOnPoiSearchListener(this);
            poiSearch.setBound(new PoiSearch.SearchBound(latLonPoint, 3000, true));//设置搜索范围
            poiSearch.searchPOIAsyn();// 异步搜索
        }
        
    }
    
    ......
    
    @Override
    public void onPoiSearched(PoiResult result, int code) {

        //DialogUtils.dismissProgressDialog();
        if (code == AMapException.CODE_AMAP_SUCCESS) {
            if (result != null && result.getQuery() != null) {// 搜索poi的结果
                loge("搜索的code为===="+code+", result数量=="+result.getPois().size());
                if (result.getQuery().equals(query)) {// 是否是同一次搜索
                    poiResult = result;
                    loge("搜索的code为===="+code+", result数量=="+poiResult.getPois().size());
                    List<SuggestionCity> suggestionCities = poiResult.getSearchSuggestionCitys();// 当搜索不到poiitem数据时，会返回含有搜索关键字的城市信息
                    if (poiItems != null && poiItems.size() > 0) {
                        poiItems.clear();
                        if (adapter != null) {
                            adapter.notifyDataSetChanged();
                        }
                    }
                    poiItems = poiResult.getPois();// 取得第一页的poiitem数据，页数从数字0开始
                
                //通过listview显示搜索结果的操作省略
                ......
                }
            } else {
                loge("没有搜索结果");
                toast(getString(R.string.search_no_result));
                empty_view.setText(getString(R.string.search_no_result));
            }
        } else {
            loge("搜索出现错误");
            toast(getString(R.string.search_error));
            empty_view.setText(getString(R.string.search_error));
        }

    }

    @Override
    public void onPoiItemSearched(PoiItem poiItem, int i) {

    }

    
```
注释都比较清楚,大家理解起来应该也不难,具体用法可以参考[高德官方文档](http://lbs.amap.com/api/android-sdk/guide/map-data/poi),可以直接在onTextChangeed()方法中判断是否有内容来调用doSearchQuery()方法即可。

- 通过实时访问http接口实现:
除了以上方法实现,还可以用高德提供的web端API接口实现功能,详情见[高德web服务开发文档](http://lbs.amap.com/api/webservice/guide/api/inputtips)。我们可以直接通过请求高德为我们提供的搜索url接口来访问并获取数据,输入提示API服务地址为：
http://restapi.amap.com/v3/assistant/inputtips?
需要我们填充相应的字段,如key,keyword等,具体介绍看官方文档就可以了,大波代码来袭:

```
    //方法二:使用http请求返回搜索结果
    private List<POISearchResultBean.Tips> tipsList;
    private POISearchResultBean resultBean;
    private String locationString;
    private String lon;
    private String lat;
    private final int SEARCH_OK = 3266;
    
    @Bind(R.id.search_result_listview)
    ListView resultListView;
    
    .......
    
    private MapSerchActivity.MyWeakReferenceHandler handler = new MapSerchActivity.MyWeakReferenceHandler(this) {
        @Override
        public void handleMessage(Message msg, Activity weakReferenceActivity) {
            if (msg.what == ADDRESS_LOCATION_GET) {
                if (tipsList != null && tipsList.size() > 0) {

                    if (adapter == null && resultListView != null) {
                        //wrong
                        resultListView.setAdapter(adapter = new CommonAdapter<POISearchResultBean.Tips>(SearchAddressActivity.this, tipsList, R.layout.search_result_item) {
                            @Override
                            public void convert(ViewHolder helper, final POISearchResultBean.Tips item) {
                                helper.setText(R.id.search_result_item_address_name, item.getName());
                                helper.setText(R.id.search_result_item_address_detail, item.getDistrict()+item.getAddress());
                                helper.getView(R.id.search_result_item_address_layout).setOnClickListener(new View.OnClickListener() {
                                    @Override
                                    public void onClick(View v) {
                                        loge("点击了item");
                                        toast(item.getName());
                                        boolean hasData = hasData(item.getName());
                                        if (!hasData) {
                                            insertData(item.getName());
                                            //queryData("");
                                        }
                                        locationString = item.getLocation();
                                        lon = locationString.substring(0,locationString.indexOf(","));
                                        lat = locationString.substring(locationString.indexOf(",")+1,locationString.length());
                                        loge("经纬度信息为==="+lon+","+lat);

                                        Intent intent = new Intent();
                                        intent.putExtra("location_lon",lon);
                                        intent.putExtra("location_lat",lat);
                                        setResult(SEARCH_OK, intent);
                                        finish();
                                    }
                                });
                            }
                        });
                    } else {
                        adapter = null;
                        resultListView.setAdapter(adapter = new CommonAdapter<POISearchResultBean.Tips>(SearchAddressActivity.this, tipsList, R.layout.search_result_item) {
                            @Override
                            public void convert(ViewHolder helper, final POISearchResultBean.Tips item) {
                                helper.setText(R.id.search_result_item_address_name, item.getName());
                                helper.setText(R.id.search_result_item_address_detail, item.getDistrict()+item.getAddress());
                                helper.getView(R.id.search_result_item_address_layout).setOnClickListener(new View.OnClickListener() {
                                    @Override
                                    public void onClick(View v) {
//                                        loge("点击了item");
                                        loge("点击了item");
                                        toast(item.getName());
                                        boolean hasData = hasData(item.getName());
                                        if (!hasData) {
                                            insertData(item.getName());
                                            //queryData("");
                                        }
                                        locationString = item.getLocation();
                                        lon = locationString.substring(0,locationString.indexOf(","));
                                        lat = locationString.substring(locationString.indexOf(",")+1,locationString.length());
                                        loge("经纬度信息为==="+lon+"====="+lat);
                                        Intent intent = new Intent();
                                        intent.putExtra("location_lon",lon);
                                        intent.putExtra("location_lat",lat);
                                        setResult(SEARCH_OK, intent);
                                        finish();
                                    }
                                });
                            }
                        });
                    }

                }
            }
        }
    };
    
    .......
    
    /**
     * by moos on 2017/09/11
     * func:http请求返回关键词搜索结果
     * 请求路径范例:http://restapi.amap.com/v3/assistant/inputtips?key=您的key&keywords=肯德基&types=050301&location=116.481488,39.990464&city=北京&datatype=all
     */
    private void searchAddressByHttp(String keyWord){
        //DialogUtils.createProgressDialog(SearchAddressActivity.this,"Searching...");
        OkHttpUtils
                .get()
                .url(HttpAPI.AMAP_POI_SEARCH_URL + "key="+Const.amap_poi_search_key+"&keywords="+keyWord)
                .build()
                .execute(new StringCallback() {
                    @Override
                    public void onError(Call call, Exception e, int id) {
                        loge("获取http poi搜索结果失败=" + e.getMessage());
                        //DialogUtils.dismissProgressDialog();
                        Toast.makeText(SearchAddressActivity.this, getString(R.string.act_qr_code_fail), Toast.LENGTH_LONG).show();
                    }
                    @Override
                    public void onResponse(String response, int id) {

                        Logger.e("获取http poi搜索结果 =" + response);
                        resultBean = JSONObject.parseObject(response, POISearchResultBean.class);
                        if (resultBean.getStatus()==1) {
                            //处理和显示搜索数据列表
                            if (tipsList != null && tipsList.size() > 0) {
                                tipsList.clear();
                                if (adapter != null) {
                                    adapter.notifyDataSetChanged();
                                }
                            }
                            tipsList = resultBean.getTips();
                            Message message = Message.obtain(handler);
                            message.what = ADDRESS_LOCATION_GET;
                            handler.sendMessage(message);




                        } else {
                            toast("搜索失败,请重新尝试");
                        }

                    }
                });
    }

    
```
  通过okhttp请求网络接口有很多大神封装好的工具库,这里我使用的鸿神的okHttpUtils,大家可以根据自己的需要来选择。同时,这里使用了CommonAdapter来作为listview的适配器,同样是鸿神的杰作,如果你对它不熟悉,建议去看一下这篇文章:[打造listview万能适配器](http://blog.csdn.net/lmj623565791/article/details/38902805/)。其他的就没什么难点了,关键还是靠自己研究和练习一下了。

  最后,让我们来看看如何实现展示搜索历史的功能吧。先分析一下需求:首先进入到搜索界面要展示搜索历史列表,然后可以点击列表下方的清空历史来清除数据,接着,当我们搜索地名并选中时,自动存入搜索历史。其实,说到底,就是两个小功能,数据存储和数据展示,下面依次来探讨如何实现。

- 搜索历史数据的存储:

  一般地,我们会将搜索的历史数据保存在本地。常用的两种方式分别为数据库存储和sp(SharedPreference)存储,两种方式都可以实现我们的需求,这里我才用的是数据库,有时间的话大家可以试试sp存储方式。这里不研究数据库的基本用法比较简单,就一笔带过了,直接上代码:
>首先是创建数据库:
```

/**
 * Created by moos on 17/9/11.
 */

public class SearchHistorySQLiteHelper extends SQLiteOpenHelper {

    private static String name = "search.db";
    private static Integer version = 1;
    public SearchHistorySQLiteHelper(Context context) {
        super(context, name, null, version);
    }

    @Override
    public void onCreate(SQLiteDatabase sqLiteDatabase) {

        sqLiteDatabase.execSQL("create table history(id integer primary key autoincrement,name varchar(200))");
    }

    @Override
    public void onUpgrade(SQLiteDatabase sqLiteDatabase, int i, int i1) {

    }
}
```
> 具体操作:
```
//历史搜索功能
    private SearchHistorySQLiteHelper helper = new SearchHistorySQLiteHelper(this);
    private SQLiteDatabase db;
    private BaseAdapter baseAdapter;
    
    @Bind(R.id.search_history_listview)
    ListView search_history_listView;
    @Bind(R.id.search_history_view)
    LinearLayout search_history_view;
    
    ......
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_search_address);
        ButterKnife.bind(this);
        initView();
        

    }

    private void initView(){
        buttonCancel.setOnClickListener(this);
        buttonDelete.setOnClickListener(this);
        inputText.addTextChangedListener(this);
    
        search_history_listView.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> parent, View view, int position, long id) {

                loge("点击了第"+position+"个搜索历史item");

                TextView textView = (TextView) view.findViewById(R.id.search_history_item_address_name);
                String name = textView.getText().toString();
                inputText.setText(name);
                Toast.makeText(SearchAddressActivity.this, name, Toast.LENGTH_SHORT).show();

            }
        });

        // 第一次进入查询所有的历史记录
        queryData("");

    }

    ......
    
    //采用本地数据库存储
    /**
     * 插入数据
     */
    private void insertData(String tempName) {
        db = helper.getWritableDatabase();
        db.execSQL("insert into history(name) values('" + tempName + "')");
        db.close();
    }

    /**
     * 模糊查询数据
     */
    private void queryData(String tempName) {
        Cursor cursor = helper.getReadableDatabase().rawQuery(
                "select id as _id,name from history where name like '%" + tempName + "%' order by id desc ", null);


        // 创建adapter适配器对象
        baseAdapter = new SimpleCursorAdapter(this, R.layout.search_history_item, cursor, new String[] { "name" },
                new int[] { R.id.search_history_item_address_name }, CursorAdapter.FLAG_REGISTER_CONTENT_OBSERVER);

        //添加footerView
        View footerView = LayoutInflater.from(this).inflate(R.layout.delete_search_history_bt,null);
        search_history_listView.addFooterView(footerView,null,false);

        footerView.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                deleteData();
                toast("清除成功");
            }
        });
        // 设置适配器
        search_history_listView.setAdapter(baseAdapter);
        baseAdapter.notifyDataSetChanged();
        if(baseAdapter.getCount()==0){
            //无历史搜索记录
            search_history_view.setVisibility(View.GONE);
        }

    }

    /**
     * 检查数据库中是否已经有该条记录
     */
    private boolean hasData(String tempName) {
        Cursor cursor = helper.getReadableDatabase().rawQuery(
                "select id as _id,name from history where name =?", new String[]{tempName});
        //判断是否有下一个
        return cursor.moveToNext();
    }

    /**
     * 清空数据
     */
    private void deleteData() {
        db = helper.getWritableDatabase();
        db.execSQL("delete from history");
        db.close();
        loge("搜索历史数据删除成功");
        queryData("");
        
    }
```

  数据库的操作网上很多教程和文章,这里就不多加解释,主要说一下逻辑吧。首先,我们输入关键词搜索到相应的目标地点后,点击回调中,即插入一条该地名的数据。当然,为了防止重复,我们需要判断一下数据库中是否已经存在该数据。我们刚进入搜索界面的时候,需要查询数据库中所有的数据并展示。

- 搜索历史数据的展示和删除:

  展示的话没什么太大的问题,一般采用listview展示并为item加上点击事件就OK了,主要是要设置好展示数据和刷新数据的逻辑。这里主要提一下如何实现下方"清除搜索历史"的友好展示以及逻辑。

  如果有人问你如何快速实现listview下面的button依附效果,你会怎么回答?常见的回答一般有两种:

1.在listview下面放一个button,然后外面套一层ScrollerView

2.使用listview的addFooterView()给其添加底部布局

  虽然两种方式很容易想到,但是对于我们这些新手来说,动手实现起来多少有些弯弯绕绕。比如第一种方式,我们需要考虑如何处理嵌套滑动的问题,至于第二种,无非是研究listview之footerview的用法,当然,抛开各自的难点,第二种方式无非更加优雅一些,所以,这里只讨论如何使用该方式实现。
首先看一下footerview的布局:
>search_history_item:
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:gravity="center_horizontal">
    <TextView
        android:id="@+id/item_search_history_delete"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="15dp"
        android:gravity="center"
        android:text="清楚搜索历史"
        android:textSize="12sp"
        android:layout_marginBottom="15dp"
        android:textColor="#828282"/>

</LinearLayout>
```
listview添加footerview比较简单,只通过简单的两行代码:
```
//添加footerView
View footerView = LayoutInflater.from(this).inflate(R.layout.delete_search_history_bt,null);
search_history_listView.addFooterView(footerView,null,false);
```
便可以实现,但是footerview的点击事件如何获取呢?很多人说直接用onItenClickListener()呀,但是,大家可以通过log或者toast看看,点击footerview是否真的响应了。答案是 - 并没有。我们应该尽量避免在onITemClickListener回调方法中实现footerview点击事件,因为position并没有变化,上限依旧是原来的adapter.getCount()。我们可以先禁止footerview在item中的点击响应,即addFooterView()方法第三个参数设为false,然后给footerView单独设置点击事件:
```
footerView.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                deleteData();
                toast("清除成功");
            }
        });
```

  到这里,我们就完整实现了地图的搜索功能了,虽然实现的方式比较简单,但还是学到一些东西的。后面有时间会将该部分功能做个demo单独分享出来。另外大家如果有什么问题和优化建议,欢迎留言反馈,不胜感激😆。

  最后,请教一下大家mac如何录制gif呢,为什么我用licecap录制出来的是黑屏呢(⊙﹏⊙).