# 城市选择器 #

今天我们一起实现一个城市选择器。

代码下载：
城市选择器 - 下载频道 - CSDN.NET
http://download.csdn.net/detail/baidu_31093133/9675482

效果图预览

![](http://i.imgur.com/5dMe6Hc.gif)

# 主要包含以下内容： #
1、自动定位所在城市  
2、热门城市列表展示  
3、所有城市列表的展示  
4、输入城市名或者城市拼音搜索对应城市  
5、右侧的slidebar城市列表导航栏

请大家先下载Demo然后再一边看demo一边看博客。因为博客里很多代码因为比较简单就不贴了。

# 首先我们先搭建基本的UI： #

分析效果图，我们需要一个顶部title view，一个搜索框，一个定位功能的view，一个展示热门城市的view，一个侧边栏view和一个listview。

###顶部title View:

这里有一些需要注意的地方：
我们在新建工程的时候，android studio会自动生成一个style作为我们的主题:  

    <style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
    </style>    

    android:theme="@style/AppTheme"  

这个默认的主题是带有actionbar的，如果我们要去掉这个actionbar，首先需要把DarkActionBar改为NoActionBar，因为使用AppCompatActivity的时候，Activity必须使用Theme.AppCompat主题及其子主题，所以我们的自定义的HD_NoActionBar样式必须继承这个主题：

     <style name="AppTheme" parent="Theme.AppCompat.Light.NoActionBar">
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
    </style>
    <style name="HD_NoActionBar" parent="AppTheme">
        <item name="android:windowNoTitle">true</item>
        <item name="android:windowActionBar">false</item>
    </style>

然后引用这个style:

    android:theme="@style/AppTheme.NoActionBar"

接下来写我们的头布局 title_view.xml：  

    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical">

    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize"
        android:background="@color/light_blue">

        <ImageView
            android:id="@+id/back"
            style="@style/Widget.AppCompat.ActionButton"
            android:layout_width="wrap_content"
            android:layout_height="match_parent"
            android:paddingLeft="16dp"
            android:paddingRight="16dp"
            android:scaleType="center"
            android:src="@mipmap/ic_back"
            tools:ignore="ContentDescription" />

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_centerInParent="true"
            android:text="@string/select_city"
            android:textSize="20sp"
            android:textColor="@color/white" />
    </RelativeLayout>

    <View
        android:layout_width="match_parent"
        android:layout_height="1dp"
        android:background="@color/white"/>
</LinearLayout>

布局返回按钮用一个ImageView,title用一个Textview。

然后在我们的主布局里使用<include>标签引入头布局：  

    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/activity_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
	android:background="@color/light_blue"
    android:orientation="vertical">

    <include layout="@layout/title_view" />

</LinearLayout>

现在的效果是这样的：

![title](http://i.imgur.com/Sr9mtCA.png)

### 搜索框布局 search view

search_view/xml:

    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="36dp"
    android:layout_marginBottom="8dp"
    android:layout_marginLeft="16dp"
    android:layout_marginRight="16dp"
    android:layout_marginTop="8dp"
    android:gravity="center_vertical"
    android:background="@drawable/search_box_bg">

    <ImageView
        android:layout_width="wrap_content"
        android:layout_height="match_parent"
        android:paddingLeft="8dp"
        android:paddingRight="8dp"
        android:src="@mipmap/ic_search"
        android:scaleType="center"
        tools:ignore="ContentDescription" />

    <EditText
        android:id="@+id/et_search"
        android:layout_weight="1"
        android:layout_width="0dp"
        android:layout_height="match_parent"
        android:background="@null"
        android:gravity="center_vertical"
        android:hint="@string/hint_search_box"
        android:textColorHint="@color/deep_blue"
        android:inputType="text"
        android:singleLine="true"
        android:textColor="@color/deep_blue"
        android:textSize="14sp"
        tools:ignore="RtlHardcoded" />

    <ImageView
        android:id="@+id/iv_search_clear"
        android:layout_width="wrap_content"
        android:layout_height="match_parent"
        android:paddingLeft="8dp"
        android:paddingRight="8dp"
        android:src="@mipmap/ic_search_clear"
        android:visibility="gone"
        tools:ignore="ContentDescription" />
</LinearLayout>

然后在主布局里引入这个布局：

    <include layout="@layout/search_view"/>

搜索框的布局也非常简单，就不说明了。

现在的效果：
![](http://i.imgur.com/F7d376p.png)

###城市列表

接下来的定位城市、热门城市、以及所有城市的列表我们使用一个Listview搞定，让Listview加载三种不同的布局来展示。

定位城市和所有城市列表好说，这个热门城市的UI该怎么做呢？我们准备使用gridview来做，在listview里嵌套gridview会遇到gridview只能显示一行的问题，我们先重现这个问题，然后再分析怎么解决。

listview需要一个adapter适配器，adapter需要一个数据源，我们的数据源存放在一个db数据库里，所以我们要构建一个数据库操作类，从数据库中取出这些城市然后展示出来。这一段的代码比较多，前方高能预警(*^__^*) 

我们把要做的事情按步骤划分：

1、导入数据库文件  
2、构建City对象，用户存储城市信息  
3、创建DBManager用来操作数据库，将查询到的数据传递给adapter  
4、编写定位城市、热门城市、所有城市三种不同的item布局  
5、编写adapter，在adapter里加载三种item布局  
6、编写gridview热门城市的item布局
7、实现gridview的adapter

  
####1、建立assets文件，并把db文件放在assets目录下：
![](http://i.imgur.com/r9I56f5.png)

####2、City对象

City.java:  

    public class City {
    private String name;
    private String pinyin;

    public City() {}

    public City(String name, String pinyin) {
        this.name = name;
        this.pinyin = pinyin;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPinyin() {
        return pinyin;
    }

    public void setPinyin(String pinyin) {
        this.pinyin = pinyin;
    }
}


####数据库操作类：

DBManager.java:
  
    public class DBManager {
    private static final String ASSETS_NAME = "china_cities.db";
    private static final String DB_NAME = "china_cities.db";
    private static final String TABLE_NAME = "city";
    private static final String NAME = "name";
    private static final String PINYIN = "pinyin";
    private static final int BUFFER_SIZE = 1024;
    private String DB_PATH;
    private Context mContext;

    //初始化
    public DBManager(Context context) {
        this.mContext = context;
        DB_PATH = File.separator + "data"
                + Environment.getDataDirectory().getAbsolutePath() + File.separator
                + context.getPackageName() + File.separator + "databases" + File.separator;
    }
    //保存数据库到本地
    @SuppressWarnings("ResultOfMethodCallIgnored")
    public void copyDBFile(){
        File dir = new File(DB_PATH);
        if (!dir.exists()){
            dir.mkdirs();
        }
        File dbFile = new File(DB_PATH + DB_NAME);
        if (!dbFile.exists()){
            InputStream is;
            OutputStream os;
            try {
                is = mContext.getResources().getAssets().open(ASSETS_NAME);
                os = new FileOutputStream(dbFile);
                byte[] buffer = new byte[BUFFER_SIZE];
                int length;
                while ((length = is.read(buffer, 0, buffer.length)) > 0){
                    os.write(buffer, 0, length);
                }
                os.flush();
                os.close();
                is.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
    /**
     * 读取所有城市
     * @return
     */
    public List<City> getAllCities(){
        SQLiteDatabase db = SQLiteDatabase.openOrCreateDatabase(DB_PATH + DB_NAME, null);
        Cursor cursor = db.rawQuery("select * from " + TABLE_NAME, null);
        List<City> result = new ArrayList<>();
        City city;
        while (cursor.moveToNext()){
            String name = cursor.getString(cursor.getColumnIndex(NAME));
            String pinyin = cursor.getString(cursor.getColumnIndex(PINYIN));
            city = new City(name, pinyin);
            result.add(city);
        }
        cursor.close();
        db.close();
        Collections.sort(result, new CityComparator());
        return result;
    }

    /**
     * 通过名字或者拼音搜索
     * @param keyword
     * @return
     */
    public List<City> searchCity(final String keyword){
        SQLiteDatabase db = SQLiteDatabase.openOrCreateDatabase(DB_PATH + DB_NAME, null);
        Cursor cursor = db.rawQuery("select * from " + TABLE_NAME +" where name like \"%" + keyword
                + "%\" or pinyin like \"%" + keyword + "%\"", null);
        List<City> result = new ArrayList<>();
        City city;
        while (cursor.moveToNext()){
            String name = cursor.getString(cursor.getColumnIndex(NAME));
            String pinyin = cursor.getString(cursor.getColumnIndex(PINYIN));
            city = new City(name, pinyin);
            result.add(city);
        }
        cursor.close();
        db.close();
        Collections.sort(result, new CityComparator());
        return result;
    }

    /**
     * a-z排序
     */
    private class CityComparator implements Comparator<City> {
        @Override
        public int compare(City lhs, City rhs) {
            String a = lhs.getPinyin().substring(0, 1);
            String b = rhs.getPinyin().substring(0, 1);
            return a.compareTo(b);
        }
    }
}

这个类使用SQLiteDatabase来管理数据库，同事写了一个排序类CityComparator用来对城市按照首字母进行排序

###定位城市的布局：
view_locate_city.xml:  

    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:paddingBottom="8dp"
    tools:ignore="RtlHardcoded">

    <TextView
        style="@style/LetterIndexTextViewStyle"
        android:text="@string/located_city"/>

    <LinearLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginLeft="16dp"
        android:background="@color/content_bg">

        <LinearLayout
            android:id="@+id/layout_locate"
            android:layout_width="wrap_content"
            android:layout_height="40dp"
            android:minWidth="96dp"
            android:paddingLeft="8dp"
            android:paddingRight="8dp"
            android:gravity="center"
            android:clickable="true"
            android:background="@drawable/overlay_bg">

            <ImageView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:src="@mipmap/ic_locate"
                tools:ignore="ContentDescription" />

            <TextView
                android:id="@+id/tv_located_city"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_marginLeft="8dp"
                android:text="@string/locating"
                android:textSize="16sp"
                android:textColor="@color/white"/>
        </LinearLayout>
    </LinearLayout>
</LinearLayout>
>效果图
![](http://i.imgur.com/bjIQyY8.png)

然后是所有城市的布局：  


    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    tools:ignore="RtlHardcoded">

    <TextView
        android:id="@+id/tv_item_city_listview_letter"
        style="@style/LetterIndexTextViewStyle"
        android:textSize="18sp"
        android:clickable="false"/>

    <TextView
        android:id="@+id/tv_item_city_listview_name"
        android:layout_width="match_parent"
        android:layout_height="48dp"
        android:paddingLeft="16dp"
        android:paddingRight="@dimen/side_letter_bar_width"
        android:background="?android:attr/selectableItemBackground"
        android:clickable="true"
        android:gravity="center_vertical"
        android:textSize="@dimen/city_text_size"
        android:textColor="@color/light_blue"/>
    <View
        android:layout_width="match_parent"
        android:layout_height="1px"
        android:layout_marginLeft="16dp"
        android:layout_marginRight="@dimen/side_letter_bar_width"
        android:background="@color/divider"/>
</LinearLayout>

使用两个TextView一个用来显示城市的首字母，一个用来显示城市名字

上面的布局都很简单，接下来就是热门城市了：

如果我们使用gridview不做任何处理的话，最终效果是这样的：
![](http://i.imgur.com/fXyS6HB.png)
不止在listview，gridview在其它任何可以滚动的view里都会出现这个问题，解决这个问题我们有固定的方案，那就是自定义一个gridview然后重写onMeasure方法，在onMeasure方法里，让gridview测量子view的高度，并全部显示出来。

代码其实非常简单： 

    public class WrapHeightGridView extends GridView {
    public WrapHeightGridView(Context context) {
        this(context, null);
    }

    public WrapHeightGridView(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public WrapHeightGridView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

    //如果把GridView放到一个垂直方向滚动的布局中，设置其高度属性为 wrap_content ,
    // 则该GridView的高度只有一行内容，其他内容通过滚动来显示。
    // 如果你想让该GridView的高度为所有行内容所占用的实际高度，则可以通过覆写GridView的 onMeasure 函数来修改布局参数
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        int heightSpec = MeasureSpec.makeMeasureSpec(Integer.MAX_VALUE >> 2, MeasureSpec.AT_MOST);
        super.onMeasure(widthMeasureSpec, heightSpec);
    }
}  

这里重点理解makeMeasureSpec这个方法，public static int makeMeasureSpec(int size, int mode)
这个是由我们给出的尺寸大小和模式生成一个包含这两个信息的int变量。这个int值有32位，其中高2位表示模式，低30位表示值。我们把Integer.MAX_VALUE >> 2右移两位然后和MeasureSpec.AT_MOST合成一个新的int值。Integer.MAX_VALUE是INT类型的最大值是0xFFFFFFFF，所以这个值右移两位就表示新的合成的int值得低30位都是1。也就是说我们取最大值作为控件高度的最大值。


这样就可以了，效果：

![](http://i.imgur.com/WQ2IM9B.png)

接下来看一下adapter是怎么实现的：

CityListAdapter.java:

    package test.study.select_city.adapter;

public class CityListAdapter extends BaseAdapter{
    private static final int VIEW_TYPE_COUNT = 3;

    private Context mContext;
    private LayoutInflater inflater;
    private List<City> mCities;
    private HashMap<String, Integer> letterIndexes;
    private String[] sections;
    private OnCityClickListener onCityClickListener;
    private int locateState = LocateState.LOCATING;
    private String locatedCity;

    public CityListAdapter(Context mContext, List<City> mCities) {
        this.mContext = mContext;
        this.mCities = mCities;
        this.inflater = LayoutInflater.from(mContext);
        if (mCities == null){
            mCities = new ArrayList<>();
        }
        mCities.add(0, new City("定位", "0"));
        mCities.add(1, new City("热门", "1"));
        int size = mCities.size();
        letterIndexes = new HashMap<>();
        sections = new String[size];
        for (int index = 0; index < size; index++){
            //当前城市拼音首字母
            String currentLetter = PinyinUtils.getFirstLetter(mCities.get(index).getPinyin());
            //上个首字母，如果不存在设为""
            String previousLetter = index >= 1 ? PinyinUtils.getFirstLetter(mCities.get(index - 1).getPinyin()) : "";
            if (!TextUtils.equals(currentLetter, previousLetter)){
                letterIndexes.put(currentLetter, index);
                sections[index] = currentLetter;
            }
        }
    }

    /**
     * 更新定位状态
     * @param state
     */
    public void updateLocateState(int state, String city){
        this.locateState = state;
        this.locatedCity = city;
        notifyDataSetChanged();
    }

    /**
     * 获取字母索引的位置
     * @param letter
     * @return
     */
    public int getLetterPosition(String letter){
        Integer integer = letterIndexes.get(letter);
        return integer == null ? -1 : integer;
    }

    @Override
    public int getViewTypeCount() {
        return VIEW_TYPE_COUNT;
    }

    @Override
    public int getItemViewType(int position) {
        return position < VIEW_TYPE_COUNT - 1 ? position : VIEW_TYPE_COUNT - 1;
    }

    @Override
    public int getCount() {
        return mCities == null ? 0: mCities.size();
    }

    @Override
    public City getItem(int position) {
        return mCities == null ? null : mCities.get(position);
    }

    @Override
    public long getItemId(int position) {
        return position;
    }

    @Override
    public View getView(final int position, View view, ViewGroup parent) {
        CityViewHolder holder;
        int viewType = getItemViewType(position);
        switch (viewType){
            case 0:     //定位
                view = inflater.inflate(R.layout.view_locate_city, parent, false);
                ViewGroup container = (ViewGroup) view.findViewById(R.id.layout_locate);
                TextView state = (TextView) view.findViewById(R.id.tv_located_city);
                switch (locateState){
                    case LocateState.LOCATING:
                        state.setText(mContext.getString(R.string.locating));
                        break;
                    case LocateState.FAILED:
                        state.setText(R.string.located_failed);
                        break;
                    case LocateState.SUCCESS:
                        state.setText(locatedCity);
                        break;
                }
                container.setOnClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        if (locateState == LocateState.FAILED){
                            //重新定位
                            if (onCityClickListener != null){
                                onCityClickListener.onLocateClick();
                            }
                        }else if (locateState == LocateState.SUCCESS){
                            //返回定位城市
                            if (onCityClickListener != null){
                                onCityClickListener.onCityClick(locatedCity);
                            }
                        }
                    }
                });
                break;
            case 1:     //热门
                view = inflater.inflate(R.layout.view_hot_city, parent, false);
                WrapHeightGridView gridView = (WrapHeightGridView) view.findViewById(R.id.gridview_hot_city);
                final HotCityGridAdapter hotCityGridAdapter = new HotCityGridAdapter(mContext);
                gridView.setAdapter(hotCityGridAdapter);
                gridView.setOnItemClickListener(new AdapterView.OnItemClickListener() {
                    @Override
                    public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
                        if (onCityClickListener != null){
                            onCityClickListener.onCityClick(hotCityGridAdapter.getItem(position));
                        }
                    }
                });
                break;
            case 2:     //所有
                if (view == null){
                    view = inflater.inflate(R.layout.item_city_listview, parent, false);
                    holder = new CityViewHolder();
                    holder.letter = (TextView) view.findViewById(R.id.tv_item_city_listview_letter);
                    holder.name = (TextView) view.findViewById(R.id.tv_item_city_listview_name);
                    view.setTag(holder);
                }else{
                    holder = (CityViewHolder) view.getTag();
                }
                if (position >= 1){
                    final String city = mCities.get(position).getName();
                    holder.name.setText(city);
	//如果当前的item的城市的首字母和上一个城市的首字母相同，就不显示首字母否则就显示。  
	//这样就可以实现让所有城市根据首字母分类的效果了。
                    String currentLetter = PinyinUtils.getFirstLetter(mCities.get(position).getPinyin());
                    String previousLetter = position >= 1 ? PinyinUtils.getFirstLetter(mCities.get(position - 1).getPinyin()) : "";
                    if (!TextUtils.equals(currentLetter, previousLetter)){
                        holder.letter.setVisibility(View.VISIBLE);
                        holder.letter.setText(currentLetter);
                    }else{
                        holder.letter.setVisibility(View.GONE);
                    }
                    holder.name.setOnClickListener(new View.OnClickListener() {
                        @Override
                        public void onClick(View v) {
                            if (onCityClickListener != null){
                                onCityClickListener.onCityClick(city);
                            }
                        }
                    });
                }
                break;
        }
        return view;
    }

    public static class CityViewHolder{
        TextView letter;
        TextView name;
    }

    public void setOnCityClickListener(OnCityClickListener listener){
        this.onCityClickListener = listener;
    }

    public interface OnCityClickListener{
        void onCityClick(String name);
        void onLocateClick();
    }
}


Listview加载不同的布局，请参考我的另一篇博客：
最主要的就是getItemViewType方法。

> listview加载不同布局 - 秦时明月 - 博客频道 - CSDN.NET
http://blog.csdn.net/baidu_31093133/article/details/51804923

在adapter里使用LocateState类来标示不同的定位状态。代码比较多，但是都很好理解就不再解释了（其实是懒。。。）

###gridview热门城市的item布局
就是一个Textview而已
###gridview热门城市的adapter
请参考demo，就是一个简单的adapter

###然后是侧边导航栏的实现：

写一个自定义view，
SideBar.java  

    public class SideBar extends View {
    private static final String[] b = {"定位", "热门", "A", "B", "C", "D", "E", "F", "G", "H", "I", "J", "K", "L", "M", "N", "O", "P", "Q", "R", "S", "T", "U", "V", "W", "X", "Y", "Z"};
    private int choose = -1;
    private Paint paint = new Paint();
    private boolean showBg = false;
    private OnLetterChangedListener onLetterChangedListener;
    private TextView overlay;

    public SideBar(Context context, AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
    }

    public SideBar(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public SideBar(Context context) {
        super(context);
    }

    /**
     * 设置悬浮的textview
     * @param overlay
     */
    public void setOverlay(TextView overlay){
        this.overlay = overlay;
    }

    @SuppressWarnings("deprecation")
    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        if (showBg) {
            canvas.drawColor(Color.TRANSPARENT);
        }

        int height = getHeight();
        int width = getWidth();
        int singleHeight = height / b.length;//单行字母高度
        for (int i = 0; i < b.length; i++) {
            paint.setTextSize(getResources().getDimension(R.dimen.side_letter_bar_letter_size));
            paint.setColor(getResources().getColor(R.color.deep_blue));
            paint.setAntiAlias(true);
            if (i == choose) {
                paint.setColor(getResources().getColor(R.color.gray_deep)); 
		
//                paint.setFakeBoldText(true);  //加粗 

            }
	    
            float xPos = width / 2 - paint.measureText(b[i]) / 2;
            float yPos = singleHeight * i + singleHeight;
            canvas.drawText(b[i], xPos, yPos, paint);
            paint.reset();
        }

    }

    @Override
    public boolean dispatchTouchEvent(MotionEvent event) {
        final int action = event.getAction();
        final float y = event.getY();
        final int oldChoose = choose;
        final OnLetterChangedListener listener = onLetterChangedListener;
        final int c = (int) (y / getHeight() * b.length);//获取字母的index

        switch (action) {
            case MotionEvent.ACTION_DOWN:
                showBg = true;
                if (oldChoose != c && listener != null) {
                    if (c >= 0 && c < b.length) {
                        listener.onLetterChanged(b[c]);
                        choose = c;
                        invalidate();
                        if (overlay != null){
                            overlay.setVisibility(VISIBLE);
                            overlay.setText(b[c]);
                        }
                    }
                }

                break;
            case MotionEvent.ACTION_MOVE:
                if (oldChoose != c && listener != null) {
                    if (c >= 0 && c < b.length) {
                        listener.onLetterChanged(b[c]);
                        choose = c;
                        invalidate();
                        if (overlay != null){
                            overlay.setVisibility(VISIBLE);
                            overlay.setText(b[c]);
                        }
                    }
                }
                break;
            case MotionEvent.ACTION_UP:
                showBg = false;
                choose = -1;
                invalidate();
                if (overlay != null){
                    overlay.setVisibility(GONE);
                }
                break;
        }
        return true;
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        return super.onTouchEvent(event);
    }

    public void setOnLetterChangedListener(OnLetterChangedListener onLetterChangedListener) {
        this.onLetterChangedListener = onLetterChangedListener;
    }

    public interface OnLetterChangedListener {
        void onLetterChanged(String letter);
    }

}

我们绘制了slidebar，并且重写了它的ontouch事件。当手指摁下和滑动的时候在布局中央显示当前城市的首字母，显示首字母的控件是一个Textview,这个Textview通过setOverlay方法传递进来。抬起的时候不显示。

现在的效果是这样：

![](http://i.imgur.com/LzJyzOV.png)

UI总算实现的差不多了，接下来还有城市定位功能，搜索功能以及右侧导航功能的要实现。

###搜索功能：

主要就是给edittext设置一个TextChangedListener，让它根据输入去数据库中查找数据并将数据传递给adapter，然后通过notifyDataSetChanged方法来更新UI。

代码请参照demo

###右侧导航功能
当我们的手指在slidebar上滑动的时候会触发它的ontouch事件，然后通过回调将当前的字母传递回来，接着我们将点击的字母传递给mCityAdapter的getLetterPosition方法，来得到当前字母的位置，并通过mListview.setSelection(position);方法来改变listview的显示位置

代码参考demo

###定位功能

这个需要接入百度地图的sdk

这个大家根据百度地图开发者中心的手册一点点来就可以了。

传送门：

> android-locsdk/guide/key - Wiki
http://lbsyun.baidu.com/index.php?title=android-locsdk/guide/key

谢谢大家。(*^__^*)




