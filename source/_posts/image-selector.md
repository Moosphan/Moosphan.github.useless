---
title: Android 实现本地图片和视频选择器功能
date: 2018-9-22 15:11:35
tags: 
- Android
- Image selector
categories: Android
toc: true
---
哈喽，大家好，好久不见了，很久没有更新 Android 方面的技术文章了，最近在忙公司的 AR 类的新产品，其中涉及到本地图片和视频的选择和上传功能。至于为什么不用系统提供的图片和视频选择器，原因你懂的，系统提供的选择器只能通过 Intent 方式去获取，这意味着需要离开当前页面前往系统的媒体库，选择完毕后在onActivityResult 方法中拿到结果。这显然存在很多弊端：

- UI的定制化很差
- 需要离开当前页面，体验不好
- 不同机型可能会出现各种问题
- 系统选择器并不支持多选功能
<!--more-->


​其实，我们最希望的是拿到手机中的图片和视频数据，至于UI的绘制和交互细节都由我们自己来定制。你说你想用 ListView 或者 RecyclerView 来展示所有图片和视频，ok，当然可以，那是你的自由！让我们先来看一下最终实现的效果图吧：



![图片选择器效果图](http://upload-images.jianshu.io/upload_images/5256969-f244456c0dbb2dea.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)                ![视频选择器效果图](http://upload-images.jianshu.io/upload_images/5256969-b6ec3435d7b516a7.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



不要直接一看效果图以为还是前往的另一个页面，那和其他图片选择器有什么分别？客官先别急，这里的效果图只是为了美观而已，反正数据给你了，想怎么安排UI就看你们设计喵了😄～，比如可以这样：



![定制化UI效果图](http://upload-images.jianshu.io/upload_images/5256969-3fa6f98868927420.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



看到这你可能会以为很复杂，其实不然，代码量很少，而且涉及到的核心知识点如：获取系统图片和视频数据、单选和多选功能，相信大家一看就明了。好了，喝口茶，且听我慢慢道来。



#### 获取手机所有图片和视频数据

一般地，获取手机内部图片和视频数据有两种方式：通过遍历文件夹获取图片和视频资源，或者通过ContentResolver来获取。虽然第一种方式拿到的图片比较齐全，但文件遍历操作过于耗时，这里我推荐采用第二种方式。ContentResolver即内容解析器，可以对ContentProvider中的数据库进行增删改查操作，其中主要包含联系人、短信、相册、视频、音频等一系列数据。我们来看看具体获取系统图片数据实现代码吧：

```
/**
 * <pre>
 *     @author moosphon  (about me: <a>https://github.com/Moosphan<a/>)
 *     @date   2018/09/16
 *     @desc   get all pictures of the phone.
 * <pre/>
 */
fun getLocalPictures(mContext: Context?): List<ImageMediaEntity>? {
    val images = ArrayList<ImageMediaEntity>()
    val resolver = mContext?.contentResolver
    var cursor: Cursor? = null
    queryImageThumbnails(resolver!!, arrayOf(MediaStore.Images.Thumbnails.IMAGE_ID, MediaStore.Images.Thumbnails.DATA))
    try {
        cursor = resolver.query(MediaStore.Images.Media.EXTERNAL_CONTENT_URI,
                arrayOf(MediaStore.Images.ImageColumns.DATA,
                        MediaStore.Images.ImageColumns._ID,
                        MediaStore.Images.ImageColumns.SIZE,
                        MediaStore.Images.ImageColumns.MIME_TYPE),
                null, null, null)
        return if (cursor == null || !cursor.moveToFirst()) {
            null
        } else {
            do {
                val picPath = cursor.getString(cursor.getColumnIndex(MediaStore.Images.Media.DATA))
                val id = cursor.getString(cursor.getColumnIndex(MediaStore.Images.Media._ID))
                val size = cursor.getString(cursor.getColumnIndex(MediaStore.Images.Media.SIZE))
                val mimeType = cursor.getString(cursor.getColumnIndex(MediaStore.Images.Media.MIME_TYPE))
                val image = ImageMediaEntity.Builder(id, picPath)
                        .setMimeType(mimeType)
                        .setSize(size)
                        .setThumbnailPath(mThumbnailMap?.get(id))
                        .build()
                images.add(image)
                mThumbnailMap = null
            }while (cursor.moveToNext())

            return images
        }
    } finally {
        if (cursor != null) {
            cursor.close()
        }
    }
}


 /**
  * search for thumbnails for local images
  *
  * @author moosphon
  */
  private fun queryImageThumbnails(cr: ContentResolver, projection: Array<String>) {
      var cur: Cursor? = null
      try {
          cur = MediaStore.Images.Thumbnails.queryMiniThumbnails(cr, MediaStore.Images.Thumbnails.EXTERNAL_CONTENT_URI,
                MediaStore.Images.Thumbnails.MINI_KIND, projection)
          if (cur != null && cur.moveToFirst()) {
              do {
                  val imageId = cur.getString(cur.getColumnIndex(MediaStore.Images.Thumbnails.IMAGE_ID))
                  val imagePath = cur.getString(cur.getColumnIndex(MediaStore.Images.Thumbnails.DATA))
                  mThumbnailMap = mapOf(imageId to imagePath)
              } while (cur.moveToNext() && !cur.isLast)
          }
      } finally {
          cur?.close()
      }
 }


```

可以通过代码看到，我们借助于 `ContentResolver.query` 方法来查询匹配的图片数据，我们可以设置需要获取的图片的数据字段，如 `MediaStore.Images.ImageColumns.DATA` 就表示图片存储的路径信息，其他的可以获取的信息还有图片ID、图片大小、图片类型等，大家可以参照代码去网上查看具体含义，这里不再赘述。此外，系统还为我们存储了图片以及视频的缩略图数据，我们为了提高图片加载速度，可以通过获取和展示缩略图的形式来增强体验效果。获取图片缩略图的方式采用系统自带的，也比较简单，大家可以自行查看一下文档。

另外，大家可能会发现 `ImageMediaEntity` 这个类，明白人应该很快就会知道这个数据类主要存储一些图片相关的数据。的确，这个是我个人封装的一层针对图片的数据类，而它还有个父类，名叫 `BaseMediaEntity` ,我们来看看里面都有些啥：

```
/**
 * base entity data for local media
 *
 * @author Moosphon
 */
public abstract class BaseMediaEntity implements Parcelable{
    protected enum TYPE{
        IMAGE,
        VIDEO
    }

    protected String path;
    protected String id;
    protected String size;
    public Boolean isSelected = false;


    public BaseMediaEntity() {

    }

    public BaseMediaEntity(String path, String id) {
        this.path = path;
        this.id = id;
    }

    public BaseMediaEntity(Parcel in) {
        this.path = in.readString();
        this.id   = in.readString();
        this.size = in.readString();
    }

    public abstract TYPE getMediaType();

    public String getPath() {
        return path;
    }

    public void setPath(String path) {
        this.path = path;
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getSize() {
        return size;
    }

    public void setSize(String size) {
        this.size = size;
    }

    public Boolean getSelected() {
        return isSelected;
    }

    public void setSelected(Boolean selected) {
        isSelected = selected;
    }

    @Override
    public int describeContents() {
        return 0;
    }

    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeString(this.path);
        dest.writeString(this.id);
        dest.writeString(this.size);
    }
}
```

可以看到，这是我们抽离出的公共基类，因为图片和视频等多媒体数据都有公共的数据字段id、path和size，差异性由它的子类来实现就OK了。至于  `ImageMediaEntity` 和  `VideoMediaEntity` 具体代码就先省略不放了，影响篇幅长度，最后面会有完整的sample代码。



看完了本地图片数据的获取，自然而然就能知道视频数据也是采用相同的方式获取，没错，这里就直接上代码了，其实实现方式是一样的：

```
/**
 * <pre>
 *     @author moosphon  (about me: <a>https://github.com/Moosphan<a/>)
 *     @date   2018/09/16
 *     @desc   get all videos of the phone.
 * <pre/>
 */
fun getLocalVideos(mContext: Context?) : List<VideoMediaEntity>?{
    val videos = ArrayList<VideoMediaEntity>()
    val resolver = mContext?.contentResolver
    var cursor: Cursor? = null
    try {
        cursor = resolver?.query(MediaStore.Video.Media.EXTERNAL_CONTENT_URI,
                arrayOf(MediaStore.Images.ImageColumns.DATA,
                        MediaStore.Video.Media._ID,
                        MediaStore.Video.Media.DISPLAY_NAME,
                        MediaStore.Video.Media.RESOLUTION,
                        MediaStore.Video.Media.SIZE,
                        MediaStore.Video.Media.DURATION,
                        MediaStore.Video.Media.DATE_MODIFIED),
                MediaStore.Video.Media.MIME_TYPE + "=?", arrayOf("video/mp4"), null)
        return if (cursor == null || !cursor.moveToFirst()) {
            null
        } else {
            while (cursor.moveToNext()){
                // video path
                val path = cursor.getString(cursor.getColumnIndexOrThrow(MediaStore.Video.Media.DATA))
                // video id
                val id = cursor.getLong(cursor.getColumnIndexOrThrow(MediaStore.Video.Media._ID))
                // video display name
                val name = cursor.getString(cursor.getColumnIndexOrThrow(MediaStore.Video.Media.DISPLAY_NAME))
                // video resolution
                val resolution = cursor.getString(cursor.getColumnIndexOrThrow(MediaStore.Video.Media.RESOLUTION))
                // video size
                val size = cursor.getLong(cursor.getColumnIndexOrThrow(MediaStore.Video.Media.SIZE))
                // video duration
                val duration = cursor.getLong(cursor.getColumnIndexOrThrow(MediaStore.Video.Media.DURATION))
                val date = cursor.getLong(cursor.getColumnIndexOrThrow(MediaStore.Video.Media.DATE_MODIFIED))

                val video = VideoMediaEntity.Builder(id.toString(), path)
                        .setTitle(name)
                        .setDateTaken(date.toString())
                        .setDuration(duration.toString())
                        .setSize(size.toString())
                        .build()
                videos.add(video)
            }

            return videos
        }
    } finally {
        if (cursor != null) {
            cursor.close()
        }
    }
}
```



通过上面代码我们可以发现，这几乎和获取图片数据的代码一样啊，没错，是几乎一样，但留意的人会发现，这里我调用 `ContentResolver.query` 时多传了一个selection参数，它是query方法的第三个参数，主要用来设置一些查询的条件，已达到过滤功能，大家可以根据自己需要自行设置，这里我只是想拿到mp4格式的视频数据。还有人可能会问：为什么我这里没有获取视频的缩略图数据呢？系统虽为我们也提供了获取视频缩略图的方式，但是，并不是所有的视频都存在视频缩略图，这就造成你想加载视频的缩略图的时候会出现大片空白数据问题。同时，可能会有人想借助于其他方式获取，但主流的几种方式都比较耗时，不建议在正式项目中采用。其实，通过查看很多优秀的开源视频选择器框架发现，很多都采用了分批加载功能，比如手机中一共有一千个视频数据，如果一次性获取显然很耗时，而且体验不好，我们可以分批获取数据，每页100条限制，这就极大的节省了获取数据的时间，然后再在列表滑动到底部时加载下一批数据。这里我暂时使用的是 Glide 来加载我们的视频数据，后续会寻找更佳方案代替。



下面，我们来看看图片视频的多选、单选效果实现。用过 RecyclerView 和 CheckBox 组合的开发者都知道，RecyclerView复用性会导致 CheckBox 选择状态混乱，即onCheckChanged方法的“神秘回调”，解决方案也有很多种，网上有些方案没有解决问题的也有很多。常见的方案有：自定义 checkbox、通过 checkbox 的 onclick 事件来处理选中状态，adapter数据刷新或者 checkbox 每次选中前移除上次的选中事件等等，我只选两种进行简单说明。为了节省时间，我这里将实现图片多选和视频的单选功能，它们 checkbox 问题的处理各自采用不同的方式。

我们先来看看图片多选功能实现，前方高能，代码来袭：

```
/**
 * <pre>
 *    author: moosphon
 *    date:   2018/09/16
 *    desc:   本地视频的适配器
 * <pre/>
 */
class LocalImageAdapter: RecyclerView.Adapter<LocalImageAdapter.LocalImageViewHolder>() {
    lateinit var context: Context
    private var mSelectedPosition: Int = 0
    var listener: OnLocalImageSelectListener? = null
    private lateinit var data: List<ImageMediaEntity>
    /** 存储选中的图片 */
    private var chosenImages : HashMap<Int, String>  = HashMap()
    /** 存储选中的状态 */
    private var checkStates  : HashMap<Int, Boolean> = HashMap()


    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): LocalImageViewHolder {
        context = parent.context
        val view = LayoutInflater.from(parent.context).inflate(R.layout.rv_item_local_video_layout, parent, false)
        return LocalImageViewHolder(view)
    }

    override fun getItemCount(): Int {
        return data.size
    }

    override fun onBindViewHolder(holder: LocalImageViewHolder, position: Int) {
        val thumbnailImage: ImageView = holder.view.find(R.id.local_video_item_thumbnail)
        val checkBox: CheckBox = holder.view.find(R.id.local_video_item_cb)
        /** 通过map存储checkbox选中状态,放置rv复用机制导致的状态混乱状态 */
        checkBox.setOnCheckedChangeListener(null)
        checkBox.isChecked = checkStates.containsKey(position)
        val options = RequestOptions()
                .diskCacheStrategy(DiskCacheStrategy.NONE)
                .error(R.mipmap.ic_launcher)
                .placeholder(R.mipmap.ic_launcher)

        Glide.with(context)
                .asBitmap()
                .load(data[position].thumbnailPath)
                .apply(options)
                .thumbnail(0.2f)
                .into(thumbnailImage)
        checkBox.setOnCheckedChangeListener{
            _, isChecked ->
            if (isChecked){
                checkStates[position] = true
                // 将当前选中的图片存入map
                chosenImages[position] = data[position].path

            }else{
                // 从选中列表中移除
                checkStates.remove(position)
                chosenImages.remove(position)
            }
            if (listener != null){
                val selectedImages  = ArrayList<String>()
                for (v in chosenImages.values){
                    selectedImages.add(v)
                }
                listener!!.onImageSelect(holder.view, position, selectedImages)

            }
        }


    }

    fun setData(data: List<ImageMediaEntity>){
        this.data = data
        for (i in 0 until data.size) {
            if (data[i].isSelected) {
                mSelectedPosition = i
            }
        }
    }



    class LocalImageViewHolder(val view: View) : RecyclerView.ViewHolder(view)
    /** 自定义的本地视频选择监听器 */
    interface OnLocalImageSelectListener{
        fun onImageSelect(view: View, position:Int, images: List<String>)
    }

}
```

可以看到，我们这里通过 HashMap 存储已选中 `CheckBox` 的状态，并在 `checkBox.setOnCheckedChangeListener` 前移除上一次 `CheckBox` 的监听器，然后再在 `onCheckChanged` 方法中判断当前选中状态，如果选中，那么map存入 `CheckCox` 选中状态，否则移除当前位置的value数据，这样，就解决了 滑动`RecyclerView` 后 `CheckBox` 状态混乱问题。同时，我们用 `Map` 存储每个选中后的图片路径信息，然后在自己的回调中返回这些选中的图片，最后在 `Activity` 或者 `Fragment` 中展示就可以了。

实现了图片的多选效果，我们就来看看视频单选的实现吧：

```
/**
 * <pre>
 *    author: moosphon
 *    date:   2018/09/16
 *    desc:   本地视频的适配器
 * <pre/>
 */
class LocalVideoAdapter: RecyclerView.Adapter<LocalVideoAdapter.LocalVideoViewHolder>() {
    lateinit var context: Context
    private var mSelectedPosition: Int = -1
    var listener: OnLocalVideoSelectListener? = null
    private lateinit var data: List<VideoMediaEntity>
    private var checkState: HashSet<Int> = HashSet()

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): LocalVideoViewHolder {
        context = parent.context
        val view = LayoutInflater.from(parent.context).inflate(R.layout.rv_item_local_video_layout, parent, false)
        return LocalVideoViewHolder(view)
    }

    override fun getItemCount(): Int {
        return data.size
    }

    override fun onBindViewHolder(holder: LocalVideoViewHolder, position: Int) {
        val thumbnailImage: ImageView = holder.view.find(R.id.local_video_item_thumbnail)
        val checkBox: CheckBox = holder.view.find(R.id.local_video_item_cb)
        checkBox.isChecked = checkState.contains(position)
        val options = RequestOptions()
                .diskCacheStrategy(DiskCacheStrategy.NONE)
                .error(R.mipmap.ic_launcher)
                .placeholder(R.mipmap.ic_launcher)


        Glide.with(context)
                .asBitmap()
                .load(data[position].path)
                .apply(options)
                .thumbnail(0.2f)
                .into(thumbnailImage)
        checkBox.setOnClickListener {

            if (mSelectedPosition!=position){
                //先取消上个item的勾选状态
                checkState.remove(mSelectedPosition)
                notifyItemChanged(mSelectedPosition)
                //设置新Item的勾选状态
                mSelectedPosition = position
                checkState.add(mSelectedPosition)
                notifyItemChanged(mSelectedPosition)
            }else if(checkBox.isChecked){
                checkState.add(position)

            }else if(!checkBox.isChecked){

                checkState.remove(position)
            }
            if (listener != null){
                listener!!.onVideoSelect(holder.view, position)

            }
        }
    }

    fun setData(data: List<VideoMediaEntity>){
        this.data = data
        for (i in 0 until data.size) {
            if (data[i].isSelected) {
                mSelectedPosition = i
            }
        }
    }





    class LocalVideoViewHolder(val view: View) : RecyclerView.ViewHolder(view)
    /** 自定义的本地视频选择监听器 */
    interface OnLocalVideoSelectListener{
        fun onVideoSelect(view:View, position:Int)
    }

}
```

此处主要利用 `checkBox.setOnClickListener` 以及 `HashSet` 来处理单选事件，先通过一个mSelectedPosition字段来保存当前选中的 `Checkbox` 的位置，然后在点击事件中进行分情况处理，由于这里是单选，所以在设置新的选中状态前移除上一次的`CheckBox` 选中状态。代码没什么复杂的，主要是一种思路，具体逻辑理清楚就好了，这里大家可以自己琢磨一下。



Github传送门：https://github.com/Moosphan/LocalVideoImage-selector

欢迎大家提出改进意见或者帮助我一起完善下去～