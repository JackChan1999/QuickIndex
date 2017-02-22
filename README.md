# QuickIndex
快速索引，黑马52期，http://blog.csdn.net/axi295309066/article/details/52507242

快速索引在应用中很常见，在联系人，微信，省市列表，应用管理，文件管理等应用场景都可以看到快速索引的身影，本篇博客将讲解快速索引的自定义，从中你可以学到获取汉字首字母的方法，绘制字母时，纵坐标的计算方法

![这里写图片描述](http://img.blog.csdn.net/20160911225554974)
##一、静态绘制
###**初始化数据**
创建自定义控件QuickIndexBar 继承View

```java
public class QuickIndexBar extends View {
        private Paint paint;
        // 字母数组
        private static final String[] LETTERS = new String[] { "A", "B", "C", "D",
                "E", "F", "G", "H", "I", "J", "K", "L", "M", "N", "O", "P", "Q",
                "R", "S", "T", "U", "V", "W", "X", "Y", "Z" };
        public QuickIndexBar(Context context) {
            this(context, null);
        }
        public QuickIndexBar(Context context, AttributeSet attrs) {
            this(context, attrs, 0);
        }
        public QuickIndexBar(Context context, AttributeSet attrs, int defStyle) {
            super(context, attrs, defStyle);
            paint = new Paint();
            // 设置抗锯齿，设置后画出来的边缘更加平滑
            paint.setAntiAlias(true);
            //设置字体为粗体
            paint.setTypeface(Typeface.DEFAULT_BOLD);
            // 设置字体颜色
            paint.setColor(Color.WHITE);
        }
        @Override
        protected void onDraw(Canvas canvas) {
            super.onDraw(canvas);
            //画文字的方法
            canvas.drawText("A", 10f, 10f, paint);
        }
    }
```
- 第3-6 行初始化字母数组
- 第7-12 行串连构造方法
- 第15-21 行初始化画笔
- 第27 行画文字的方法

将QuickIndexBar 布局到activity_main.xml 中

```xml
<RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <com.example.quickindexer.widget.QuickIndexBar
        android:id="@+id/quick_bar"
        android:layout_width="30dp"
        android:layout_height="match_parent"
        android:layout_alignParentRight="true"
        android:background="#ff0000"/>
</RelativeLayout>
```
###**计算字母坐标**
![这里写图片描述](http://img.blog.csdn.net/20160911224546002)

画文字的x，y 坐标是文字左下角的位置

1. 第一个字母的x 坐标则等于单元格宽度的一半减去文字宽度的一半，由于所有字母离左边的距离
一样，所以x 不变int x = cellWidth/2 - textWidth/2
2. 第一个字母的y 坐标则等于单元格高度的一半加上文字，第二个字母需要加上一个单元格的宽度
由此类推int y = cellHeight/2 +textHeight/2 +i*cellHeight
3. 单元格cellWidth 为QuickIndexBar 宽度，cellHeight 为QuickIndexBar 高度/字母数组的长度

```java
@Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        //遍历字母数组，计算坐标，进行绘制
        for (int i = 0; i < LETTERS.length; i++) {
            String letter = LETTERS[i];
            //计算x 坐标
            float x = cellWidth*0.5f - paint.measureText(letter)*0.5f;
            //计算y 坐标
            Rect bounds = new Rect();
            //获取文本的矩形区域
            paint.getTextBounds(letter, 0, letter.length(), bounds);
            float y = cellHeight*0.5f + bounds.height()+ i*cellHeight;
            //绘制文本
            canvas.drawText(letter, x, y, paint);
        }
    }
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        //控件高度
        int height = getMeasuredHeight();
        //控件宽度，也为单元格宽度
        cellWidth = getMeasuredWidth();
        //单元格宽度，控件高度除以字母数组长度，此处需要用float 类型
        //10/3 = 3.333 如果用int 接收则为3，此时高度比实际分配的高度小，所以用float 接收
        cellHeight = height*1.0f/LETTERS.length;
    }
```
- 第19-26 行获取单元的宽高，注意单元格高度需要用float 类型
- 第8-12 行通过paint 测量文字的宽高，并计算出每个字母的坐标

## **Android drawText获取text宽度的三种方式**
原文链接：http://blog.csdn.net/chuekup/article/details/7518239
```java
String str = "Hello";  
canvas.drawText( str , x , y , paint);  
  
//1. 粗略计算文字宽度  
Log.d(TAG, "measureText=" + paint.measureText(str));  
  
//2. 计算文字所在矩形，可以得到宽高  
Rect rect = new Rect();  
paint.getTextBounds(str, 0, str.length(), rect);  
int w = rect.width();  
int h = rect.height();  
Log.d(TAG, "w=" +w+"  h="+h);  
  
//3. 精确计算文字宽度  
int textWidth = getTextWidth(paint, str);  
Log.d(TAG, "textWidth=" + textWidth);  
  
    public static int getTextWidth(Paint paint, String str) {  
        int iRet = 0;  
        if (str != null && str.length() > 0) {  
            int len = str.length();  
            float[] widths = new float[len];  
            paint.getTextWidths(str, widths);  
            for (int j = 0; j < len; j++) {  
                iRet += (int) Math.ceil(widths[j]);  
            }  
        }  
        return iRet;  
    }  

//4. mPaint.getTextSize();
```

##**二、响应触摸事件**
重写onTouchEvent()方法，解析触摸事件

```java
//初始值需要设置为-1，不能为0，因为按下第一个字母的索引是0
    private int lastIndex = -1;
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        float y ;
        int currentIndex;
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:

                y = event.getY();
                //根据y 计算当前按下的字母索引
                //例如：y = 21,cellHeight = 10 =>(int) (y/cellHeight)= 2
                currentIndex = (int) (y/cellHeight);
                if(lastIndex != currentIndex){
                    //判断计算出来的索引值，避免数组越界
                    if(0 <= currentIndex && currentIndex < LETTERS.length){
                        String letter = LETTERS[currentIndex];
                        Utils.showToast(getContext(), letter);
                        //记录上次按下的索引
                        lastIndex = currentIndex;
                    }
                }
                break;
            case MotionEvent.ACTION_MOVE:
                y = event.getY();
                //根据y 计算当前按下的字母索引
                //例如：y = 21,cellHeight = 10 =>(int) (y/cellHeight)= 2
                currentIndex = (int) (y/cellHeight);
                //判断计算出来的索引值，避免数组越界
                if(0 <= currentIndex && currentIndex < LETTERS.length){
                    String letter = LETTERS[currentIndex];
                    Utils.showToast(getContext(), letter);
                    //记录上次按下的索引
                    lastIndex = currentIndex;
                }
                break;
            case MotionEvent.ACTION_UP:
                //手指抬起时需要将记录的值设为-1，否则再次按下该字母不会弹出toast
                lastIndex = -1;
                break;

            default:
                break;
        }
        //事件已被处理，返回true
        return true;
    }
```
- 第12 行通过触摸的y 值计算按下字母的索引值
- 第19 行记录上次按下字母的索引值，通过判断上次按下字母的索引与本次按下字母的索引是否相同，如果不同才弹出toast，避免在同一个字母上来回移动也一直弹出toast
- 第38 行手指抬起需要将lastIndex 还原为初始值
- 第45 行一定要返回true，代表事件已被消费
- 第17 行是单例Toast

```java
public class Utils {

        private static Toast toast;
        public static void showToast(Context context, String msg) {
            if (toast == null) {
                toast = Toast.makeText(context, "", Toast.LENGTH_SHORT);
            }
            toast.setText(msg);
            toast.show();
        }
    }
```
##**三、监听回调**
定义监听回调接口

```java
 private OnLetterUpdateListener onLetterUpdateListener;
    public OnLetterUpdateListener getOnLetterUpdateListener() {
        return onLetterUpdateListener;
    }
    public void setOnLetterUpdateListener(
            OnLetterUpdateListener onLetterUpdateListener) {
        this.onLetterUpdateListener = onLetterUpdateListener;
    }
    public interface OnLetterUpdateListener{
        public void onLetterUpdate(String letter);
    }
```
在弹出Toast 的地方替换成调用接口方法

```java
public boolean onTouchEvent(MotionEvent event) {
        float y ;
        int currentIndex;
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                y = event.getY();
                //根据y 计算当前按下的字母索引
                //例如：y = 21,cellHeight = 10 =>(int) (y/cellHeight)= 2
                currentIndex = (int) (y/cellHeight);
                if(lastIndex != currentIndex){
                    //判断计算出来的索引值，避免数组越界
                    if(0 <= currentIndex && currentIndex < LETTERS.length){
                        String letter = LETTERS[currentIndex];
                        if(onLetterUpdateListener != null){
                            onLetterUpdateListener.onLetterUpdate(letter);
                        }

                        //记录上次按下的索引
                        lastIndex = currentIndex;
                    }
                }
                break;
            case MotionEvent.ACTION_MOVE:
                y = event.getY();
                //根据y 计算当前按下的字母索引
                //例如：y = 21,cellHeight = 10 =>(int) (y/cellHeight)= 2
                currentIndex = (int) (y/cellHeight);
                //判断计算出来的索引值，避免数组越界
                if(0 <= currentIndex && currentIndex < LETTERS.length){
                    String letter = LETTERS[currentIndex];
                    if(onLetterUpdateListener != null){
                        onLetterUpdateListener.onLetterUpdate(letter);
                    }
                    //记录上次按下的索引
                    lastIndex = currentIndex;
                }
                break;
            case MotionEvent.ACTION_UP:
                //手指抬起时需要将记录的值设为-1，否则再次按下该字母不会弹出toast
                lastIndex = -1;
                break;
            default:
                break;
        }
        //事件已被处理，返回true
        return true;
    }
```
- 第14-16 调用回调接口方法，通知外面当前触摸的字母
- 第30-32 调用回调接口方法，通知外面当前触摸的字母

主Activity 中给QuickIndexBar 设置回调监听

```java
protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        QuickIndexBar quickBar = (QuickIndexBar) findViewById(R.id.quick_bar);
        quickBar.setOnLetterUpdateListener(new OnLetterUpdateListener() {
            @Override
            public void onLetterUpdate(String letter) {
                Utils.showToast(getApplicationContext(), letter);
            }
        });
    }
```

##**四、根据拼音排序**
###1、创建PinyinUtil.java
GitHub上有可以将汉字转拼音的开源项目，[TinyPinyin](https://github.com/promeG/TinyPinyin)，[pinyin4j](https://github.com/belerweb/pinyin4j)

- TinyPinyin：https://github.com/promeG/TinyPinyin
- pinyin4j：https://github.com/belerweb/pinyin4j

汉字转拼音需要导入pinyin4j-2.5.0.jar

```java
public class PinyinUtil {
        /**
         * 根据指定的汉字字符串, 返回其对应的拼音
         * @param string
         * @return
         */
        public static String getPinyin(String string) {
            // 黑-> HEI 马-> MA
            // 黑马*&^*
            // 黑123dfasdf 马
            HanyuPinyinOutputFormat format = new HanyuPinyinOutputFormat();
            // 不需要音标
            format.setToneType(HanyuPinyinToneType.WITHOUT_TONE);
            // 设置转换出大写字母
            format.setCaseType(HanyuPinyinCaseType.UPPERCASE);

            char[] charArray = string.toCharArray();

            StringBuilder sb = new StringBuilder();

            for (int i = 0; i < charArray.length; i++) {
                char c = charArray[i];

                // 如果是空格, 跳过当前循环
                if(Character.isWhitespace(c)){
                    continue;
                }

                if(c >= -128 && c < 127){
                    // 不可能是汉字, 直接拼接
                    sb.append(c);
                }else {
                    try {
                        // 获取某个字符对应的拼音. 可以获取到多音字. 单->DAN, SHAN
                        String s = PinyinHelper.toHanyuPinyinStringArray(c, format)[0];
                        sb.append(s);
                    } catch (BadHanyuPinyinOutputFormatCombination e) {

                        e.printStackTrace();
                    }
                }
            }
            return sb.toString();
        }
    }
```
###**2、填充ListView**
用于填充ListView 的姓名数组

```java
public class Cheeses {
        public static final String[] NAMES = new String[] { "宋江", "卢俊义", "吴用",
                "公孙胜", "关胜", "林冲", "秦明", "呼延灼", "花荣", "柴进", "李应", "朱仝", "鲁智
                深",
                "武松", "董平", "张清", "杨志", "徐宁", "索超", "戴宗", "刘唐", "李逵", "史进", "
                穆弘",
                "雷横", "李俊", "阮小二", "张横", "阮小五", " 张顺", "阮小七", "杨雄", "石秀", "
                解珍",
                " 解宝", "燕青", "朱武", "黄信", "孙立", "宣赞", "郝思文", "韩滔", "彭玘", "单廷珪
                ",
                "魏定国", "萧让", "裴宣", "欧鹏", "邓飞", " 燕顺", "杨林", "凌振", "蒋敬", "吕方
                ",
                "郭盛", "安道全", "皇甫端", "王英", "扈三娘", "鲍旭", "樊瑞", "孔明", "孔亮", "
                项充",
                "李衮", "金大坚", "马麟", "童威", "童猛", "孟康", "侯健", "陈达", "杨春", "郑天寿
                ",
                "陶宗旺", "宋清", "乐和", "龚旺", "丁得孙", "穆春", "曹正", "宋万", "杜迁", "薛永
                ", "施恩",
                "周通", "李忠", "杜兴", "汤隆", "邹渊", "邹润", "朱富", "朱贵", "蔡福", "蔡庆", "
                李立",
                "李云", "焦挺", "石勇", "孙新", "顾大嫂", "张青", "孙二娘", " 王定六", "郁保四", "
                白胜",
                "时迁", "段景柱" };
    }
```

将姓名转化为HaoHan 对象

```java
public class HaoHan implements Comparable<HaoHan>{
        private String name;
        private String pinyin;
        public HaoHan(String name) {
            super();
            this.name = name;
            this.pinyin = PinyinUtil.getPinyin(name);
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

        @Override
        public int compareTo(HaoHan another) {
            return this.pinyin.compareTo(another.pinyin);
        }
    }
```
第24-27 行实现Comparable 接口，用于排序操作
activity_main.xml 中添加ListView

```xml
<RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <ListView
        android:id="@+id/lv"
        android:layout_width="match_parent"
        android:layout_height="match_parent">
    </ListView>

    <com.example.quickindexer.widget.QuickIndexBar
        android:id="@+id/quick_bar"
        android:layout_width="30dp"
        android:layout_height="match_parent"
        android:layout_alignParentRight="true"
        android:background="#ff0000"/>
</RelativeLayout>
```
为ListView 创建数据适配器
ListView 条目的布局item_person.xml,每一个条目上都加上显示首字母的TextView

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <TextView
        android:id="@+id/tv_index"
        android:layout_width="match_parent"
        android:layout_height="40dp"
        android:background="#666666"
        android:gravity="center_vertical"
        android:paddingLeft="15dp"
        android:text="A"
        android:textColor="#FFFFFF"
        android:textSize="18sp"/>

    <TextView
        android:id="@+id/tv_name"
        android:layout_width="match_parent"
        android:layout_height="50dp"
        android:gravity="center_vertical"
        android:paddingLeft="15dp"
        android:text="宋江"
        android:textSize="22sp"/>

</LinearLayout>
```

数据适配器代码

```java
public class HaoHanAdapter extends BaseAdapter {
        private ArrayList<HaoHan> persons = new ArrayList<HaoHan>();
        private final Context context;

        public HaoHanAdapter(ArrayList<HaoHan> persons, Context context) {
            super();
            this.persons = persons;
            this.context = context;
        }
        @Override
        public int getItemViewType(int position) {
            // TODO Auto-generated method stub
            return super.getItemViewType(position);
        }
        @Override
        public int getViewTypeCount() {
            // TODO Auto-generated method stub
            return super.getViewTypeCount();

        }
        @Override
        public int getCount() {
            return persons.size();
        }
        @Override
        public View getView(int position, View convertView, ViewGroup parent) {
            View view;
            if(convertView == null){
                view = View.inflate(context, R.layout.item_person, null);
            }else {
                view = convertView;
            }

            TextView tv_index = (TextView) view.findViewById(R.id.tv_index);
            TextView tv_name = (TextView) view.findViewById(R.id.tv_name);

            HaoHan haoHan = persons.get(position);

            // 当前首字母
            String currentStr = haoHan.getPinyin().charAt(0) + "";
            tv_index.setText(currentStr);
            tv_name.setText(haoHan.getName());

            return view;
        }
        @Override
        public Object getItem(int position) {
            return null;
        }
        @Override
        public long getItemId(int position) {
            return 0;
        }
    }
```

##**五、根据首字母分组**
###修改数据适配器的getView()方法

```java
 @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        View view;
        if(convertView == null){

            view = View.inflate(context, R.layout.item_person, null);
        }else {
            view = convertView;
        }

        TextView tv_index = (TextView) view.findViewById(R.id.tv_index);
        TextView tv_name = (TextView) view.findViewById(R.id.tv_name);

        HaoHan haoHan = persons.get(position);

        // 当前首字母
        String currentStr = haoHan.getPinyin().charAt(0) + "";

        String indexStr = null;
        // 如果是第一个, 直接显示
        if(position == 0){
            indexStr = currentStr;
        }else {
            // 判断当前首字母和上一个条目的首字母是否一致, 不一致时候显示.
            String lastStr = persons.get(position - 1).getPinyin().charAt(0) + "";
            if(!TextUtils.equals(lastStr, currentStr)){
                // 不一致时候赋值indexStr
                indexStr = currentStr;
            }

        }

        tv_index.setVisibility(indexStr != null ? View.VISIBLE : View.GONE);
        tv_index.setText(currentStr);
        tv_name.setText(haoHan.getName());

        return view;
    }
```
第16-32 行如果是第一行直接显示首字母条目，如果不是第一行判断当前首字母和上一个条目的首字母是否一致, 不一致时候显示，一致则隐藏

##**六、ListView 和自定义控件结合**
修改回调接口代码，for 循环persons 集合找到与传回来的letter 值相同的索引值，ListView 直接滚动到对应位置即可
```java
QuickIndexBar quickBar = (QuickIndexBar) findViewById(R.id.quick_bar);
    quickBar.setOnLetterUpdateListener(new OnLetterUpdateListener() {
        @Override
        public void onLetterUpdate(String letter) {
            Utils.showToast(MainActivity.this, letter);
            for (int i = 0; i < persons.size(); i++) {


                String l = persons.get(i).getPinyin().charAt(0) + "";
                if(TextUtils.equals(letter, l)){
                    // 找到第一个首字母是letter 条目.
                    lv.setSelection(i);
                    break;
                }
            }
        }
    });
```

##**七、细节优化置**
置为当前选中的字母

```java
public boolean onTouchEvent(MotionEvent event) {
        float y ;
        int currentIndex;
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                y = event.getY();
                //根据y 计算当前按下的字母索引
                //例如：y = 21,cellHeight = 10 =>(int) (y/cellHeight)= 2
                currentIndex = (int) (y/cellHeight);
                if(lastIndex != currentIndex){
                    //判断计算出来的索引值，避免数组越界
                    if(0 <= currentIndex && currentIndex < LETTERS.length){
                        String letter = LETTERS[currentIndex];
                        if(onLetterUpdateListener != null){
                            onLetterUpdateListener.onLetterUpdate(letter);
                        }
                        //记录上次按下的索引
                        lastIndex = currentIndex;
                    }
                }
                break;
            case MotionEvent.ACTION_MOVE:
                y = event.getY();
                //根据y 计算当前按下的字母索引
                //例如：y = 21,cellHeight = 10 =>(int) (y/cellHeight)= 2
                currentIndex = (int) (y/cellHeight);
                //判断计算出来的索引值，避免数组越界
                if(0 <= currentIndex && currentIndex < LETTERS.length){
                    String letter = LETTERS[currentIndex];
                    if(onLetterUpdateListener != null){
                        onLetterUpdateListener.onLetterUpdate(letter);
                    }
                    //记录上次按下的索引
                    lastIndex = currentIndex;
                }
                break;
            case MotionEvent.ACTION_UP:
                //手指抬起时需要将记录的值设为-1，否则再次按下该字母不会弹出toast
                lastIndex = -1;
                break;

            default:
                break;
        }
        //重绘一次界面，会再次调用onDraw()方法，通过判断lastIndex 的值，把按下的字母置为灰色
        invalidate();
        //事件已被处理，返回true
        return true;
    }
```
第46 行为新增代码，此时lastIndex 为当前按下的字母索引，调用一次invalidate()方法，会再次调用onDraw()方法，在onDraw()方法中通过判断lastIndex 的值，把当前按下的字母置为灰色

```java
protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        //遍历字母数组，计算坐标，进行绘制
        for (int i = 0; i < LETTERS.length; i++) {
            String letter = LETTERS[i];
            //计算x 坐标
            float x = cellWidth*0.5f - paint.measureText(letter)*0.5f;
            //计算y 坐标
            Rect bounds = new Rect();
            //获取文本的矩形区域
            paint.getTextBounds(letter, 0, letter.length(), bounds);
            float y = cellHeight*0.5f + bounds.height()+ i*cellHeight;
            //把当前选中的字母置为灰色
            if(lastIndex == i){
                paint.setColor(Color.GRAY);
            }else{
                paint.setColor(Color.WHITE);
            }
            //绘制文本
            canvas.drawText(letter, x, y, paint);
        }
    }
```
第13-18 行如果当前字母的索引是选中的，则将画笔颜色改为灰色，没有选中的将画笔改为白色

把回调方法中Toast 的显示方式改为TextView 显示

```xml
<RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <ListView
        android:id="@+id/lv"
        android:layout_width="match_parent"
        android:layout_height="match_parent">
    </ListView>

    <TextView
        android:id="@+id/tv_center"
        android:layout_width="160dp"
        android:layout_height="100dp"
        android:layout_centerInParent="true"
        android:background="@drawable/shape_tv_center"
        android:gravity="center"
        android:text="A"
        android:textColor="#ffffff"
        android:textSize="32sp"
        android:visibility="gone"/>

    <com.example.quickindexer.widget.QuickIndexBar
        android:id="@+id/quick_bar"
        android:layout_width="30dp"
        android:layout_height="match_parent"
        android:layout_alignParentRight="true"
        android:background="#ff0000"/>
</RelativeLayout>
```
- 第11-21 行在屏幕中间添加一个提示框，默认情况为不显示
- 第16 行提示框的背景文件，需要在res 下新建一个drawable 文件夹，将shape_tv_center.xml 放在此文件夹下

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
       android:shape="rectangle" >
    <!-- android:shape="rectangle" 该形状为矩形-->
    <solid android:color="#66000000" /><!--填充颜色-->
    <corners android:radius="20dp" /><!--圆角半径-->
</shape>
```
修改回调接口中字母提示方式

```java
//屏幕中间的提示框
    tvCenter = (TextView) findViewById(R.id.tv_center);
    quickBar.setOnLetterUpdateListener(new OnLetterUpdateListener() {
        @Override
        public void onLetterUpdate(String letter) {
            //Utils.showToast(MainActivity.this, letter);
            //将Toast 改成文本提示框
            showLetter(letter);
            for (int i = 0; i < persons.size(); i++) {
                String l = persons.get(i).getPinyin().charAt(0) + "";
                if(TextUtils.equals(letter, l)){
                    // 找到第一个首字母是letter 条目.
                    lv.setSelection(i);
                    break;
                }
            }
        }
    });
    private Handler mHandler = new Handler();
    /**
     * 在屏幕中间显示一个字母提示
     * @param letter
     */
    private void showLetter(String letter) {
        tvCenter.setText(letter);
        tvCenter.setVisibility(View.VISIBLE);
        //移除所有的消息及任务
        mHandler.removeCallbacksAndMessages(null);
        //用消息机制延迟隐藏提示框
        mHandler.postDelayed(new Runnable() {
            @Override
            public void run() {
                //2 秒后隐藏提示框
                tvCenter.setVisibility(View.GONE);
            }
        }, 2000);
    }
```
- 第6-8 行将Toast 提示改为文本框提示
- 第28-36 行用消息机制延迟隐藏文本提示框，如果快速滑动showLetter()方法会频繁调用，会有多个延迟任务在消息队列中，其实当前延迟任务之前的任务都是没有必要执行的，所以可以先移除队列中所有的任务再将本次任务添加到队列中

# QuickIndexBar 
```java
/**
 * 快速索引
 * 
 * 用于根据字母快速定位联系人
 * @author AllenIverson
 *
 */
public class QuickIndexBar extends View {
	
	private static final String[] LETTERS = new String[]{
		"A", "B", "C", "D", "E", "F",
		"G", "H", "I", "J", "K", "L",
		"M", "N", "O", "P", "Q", "R",
		"S", "T", "U", "V", "W", "X",
		"Y", "Z"};

	private static final String TAG = "TAG";
	
	private Paint mPaint;

	private int cellWidth;

	private float cellHeight;
	
	/**
	 * 暴露一个字母的监听
	 */
	public interface OnLetterUpdateListener{
		void onLetterUpdate(String letter);
	}
	private OnLetterUpdateListener listener;
	
	public OnLetterUpdateListener getListener() {
		return listener;
	}
	/**
	 * 设置字母更新监听
	 * @param listener
	 */
	public void setListener(OnLetterUpdateListener listener) {
		this.listener = listener;
	}

	public QuickIndexBar(Context context) {
		this(context, null);
	}

	public QuickIndexBar(Context context, AttributeSet attrs) {
		this(context, attrs, 0);
	}

	public QuickIndexBar(Context context, AttributeSet attrs, int defStyle) {
		super(context, attrs, defStyle);
		
		mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
		mPaint.setColor(Color.WHITE);
		mPaint.setTypeface(Typeface.DEFAULT_BOLD);
	}
	
	@Override
	protected void onDraw(Canvas canvas) {
		
		for (int i = 0; i < LETTERS.length; i++) {
			String text = LETTERS[i];
			// 计算坐标
			int x = (int) (cellWidth / 2.0f - mPaint.measureText(text) / 2.0f);
			// 获取文本的高度
			Rect bounds = new Rect();// 矩形
			mPaint.getTextBounds(text, 0, text.length(), bounds);
			int textHeight = bounds.height();
			int y = (int) (cellHeight / 2.0f + textHeight / 2.0f + i * cellHeight);
			
			// 根据按下的字母, 设置画笔颜色
			mPaint.setColor(touchIndex == i ? Color.GRAY : Color.WHITE);
			
			// 绘制文本A-Z
			canvas.drawText(text, x, y, mPaint);
		}
	}
	
	int touchIndex = -1;
	@Override
	public boolean onTouchEvent(MotionEvent event) {
		int index = -1;
		switch (MotionEventCompat.getActionMasked(event)) {
			case MotionEvent.ACTION_DOWN:
				// 获取当前触摸到的字母索引
				index = (int) (event.getY() / cellHeight);
				if(index >= 0 && index < LETTERS.length){
					// 判断是否跟上一次触摸到的一样
					if(index != touchIndex) {
						if(listener != null){
							listener.onLetterUpdate(LETTERS[index]);
						}
						Log.d(TAG, "onTouchEvent: " + LETTERS[index]);
						
						touchIndex = index;
					}
				}
				
				break;
			case MotionEvent.ACTION_MOVE:
				index = (int) (event.getY() / cellHeight);
				if(index >= 0 && index < LETTERS.length){
					// 判断是否跟上一次触摸到的一样
					if(index != touchIndex){
						
						if(listener != null){
							listener.onLetterUpdate(LETTERS[index]);
						}
						Log.d(TAG, "onTouchEvent: " + LETTERS[index]);
						
						touchIndex = index;
					}
				}
				break;
			case MotionEvent.ACTION_UP:
				touchIndex = -1;
				break;
	
			default:
				break;
		}
		invalidate();
		
		return true;
	}
	
	@Override
	protected void onSizeChanged(int w, int h, int oldw, int oldh) {
		super.onSizeChanged(w, h, oldw, oldh);
		// 获取单元格的宽和高
		
		cellWidth = getMeasuredWidth();
		
		int mHeight = getMeasuredHeight();
		cellHeight = mHeight * 1.0f / LETTERS.length;
		
	}
}
```
# MainActivity

```java
public class MainActivity extends Activity {

	private ListView mMainList;
	private ArrayList<Person> persons;
	private TextView tv_center;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		
		QuickIndexBar bar = (QuickIndexBar) findViewById(R.id.bar);
		// 设置监听
		bar.setListener(new OnLetterUpdateListener() {
			@Override
			public void onLetterUpdate(String letter) {
//				Utils.showToast(getApplicationContext(), letter);
				
				showLetter(letter);
				// 根据字母定位ListView, 找到集合中第一个以letter为拼音首字母的对象,得到索引
				for (int i = 0; i < persons.size(); i++) {
					Person person = persons.get(i);
					String l = person.getPinyin().charAt(0) + "";
					if(TextUtils.equals(letter, l)){
						// 匹配成功
						mMainList.setSelection(i);
						break;
					}
				}
			}
		});
		
		mMainList = (ListView) findViewById(R.id.lv_main);
		
		persons = new ArrayList<Person>();
		
		// 填充数据 , 排序
		fillAndSortData(persons);
		
		mMainList.setAdapter(new HaoHanAdapter(MainActivity.this , persons));
		
		tv_center = (TextView) findViewById(R.id.tv_center);
		
		
	}

	private Handler mHandler = new Handler();
	
	/**
	 * 显示字母
	 * @param letter
	 */
	protected void showLetter(String letter) {
		tv_center.setVisibility(View.VISIBLE);
		tv_center.setText(letter);
		
		mHandler.removeCallbacksAndMessages(null);
		mHandler.postDelayed(new Runnable() {
			@Override
			public void run() {
				tv_center.setVisibility(View.GONE);				
			}
		}, 2000);
		
	}

	private void fillAndSortData(ArrayList<Person> persons) {
		// 填充数据
		for (int i = 0; i < Cheeses.NAMES.length; i++) {
			String name = Cheeses.NAMES[i];
			persons.add(new Person(name));
		}
		
		// 进行排序
		Collections.sort(persons);
	}
}
```
# HaoHanAdapter 
```java
public class HaoHanAdapter extends BaseAdapter {

	private Context mContext;
	private ArrayList<Person> persons;

	public HaoHanAdapter(Context mContext, ArrayList<Person> persons) {
		this.mContext = mContext;
		this.persons = persons;
	}

	@Override
	public int getCount() {
		// TODO Auto-generated method stub
		return persons.size();
	}

	@Override
	public Object getItem(int position) {
		return persons.get(position);
	}

	@Override
	public long getItemId(int position) {
		return position;
	}

	@Override
	public View getView(int position, View convertView, ViewGroup parent) {
		
		View view = convertView;
		if(convertView == null){
			view = view.inflate(mContext, R.layout.item_list, null);
		}
		ViewHolder mViewHolder = ViewHolder.getHolder(view);
		
		Person p = persons.get(position);
		
		String str = null;
		String currentLetter = p.getPinyin().charAt(0) + "";
		// 根据上一个首字母,决定当前是否显示字母
		if(position == 0){
			str = currentLetter;
		}else {
			// 上一个人的拼音的首字母
			String preLetter = persons.get(position - 1).getPinyin().charAt(0) + "";
			if(!TextUtils.equals(preLetter, currentLetter)){
				str = currentLetter;
			}
		}
		
		// 根据str是否为空,决定是否显示索引栏
		mViewHolder.mIndex.setVisibility(str == null ? View.GONE : View.VISIBLE);
		mViewHolder.mIndex.setText(currentLetter);
		mViewHolder.mName.setText(p.getName());
		
		return view;
	}
	
	static class ViewHolder {
		TextView mIndex;
		TextView mName;

		public static ViewHolder getHolder(View view) {
			Object tag = view.getTag();
			if(tag != null){
				return (ViewHolder)tag;
			}else {
				ViewHolder viewHolder = new ViewHolder();
				viewHolder.mIndex = (TextView) view.findViewById(R.id.tv_index);
				viewHolder.mName = (TextView) view.findViewById(R.id.tv_name);
				view.setTag(viewHolder);
				return viewHolder;
			}
		}
		
	}
}
```

# [FancyListIndexer](https://github.com/PoplarTang/FancyListIndexer)
![](https://camo.githubusercontent.com/9ee836fe3d74d706122869027f757f015edd3038/687474703a2f2f37786e7767632e636f6d312e7a302e676c622e636c6f7564646e2e636f6d2f6769745f6769746875625f46616e63794c697374496e64657865725f44656d6f312e676966)

![](https://camo.githubusercontent.com/563c6cb51774ce4ed768739b369e34da6cfed99f/687474703a2f2f37786e7767632e636f6d312e7a302e676c622e636c6f7564646e2e636f6d2f6769745f6769746875625f46616e63794c697374496e64657865725f50726f636564757265312e706e67)

```java
public class FancyIndexer extends View {
	
	
	public interface OnTouchLetterChangedListener {
		public void onTouchLetterChanged(String s);
	}
	
	private static final String TAG = "FancyIndexer";
	
	/////////////////////////////////////////////////////////////////////////
	
	//Properties
	// 向右偏移多少画字符， default 30
	float mWidthOffset = 30.0f;
	
	// 最小字体大小
	int mMinFontSize = 24;
	
	// 最大字体大小
	int mMaxFontSize = 48;
	
	// 提示字体大小
	int mTipFontSize = 52;
	
	// 提示字符的额外偏移
	float mAdditionalTipOffset = 20.0f;
	
	// 贝塞尔曲线控制的高度
	float mMaxBezierHeight = 150.0f;
	
	// 贝塞尔曲线单侧宽度
	float mMaxBezierWidth = 240.0f;
	
	// 贝塞尔曲线单侧模拟线量
	int  mMaxBezierLines = 32;
	
	// 列表字符颜色
	int  mFontColor = 0xffffffff;
	
	// 提示字符颜色
//	int  mTipFontColor = 0xff3399ff;
	int  mTipFontColor = 0xffd33e48;
	
	/////////////////////////////////////////////////////////////////////////
	
	private OnTouchLetterChangedListener mListener;
	
	private final String[] ConstChar = {"#","A","B","C","D","E","F","G","H","I","J","K","L"
			               ,"M","N","O","P","Q","R","S","T","U","V","W","X","Y","Z"};
	
	int mChooseIndex = -1;
	Paint mPaint = new Paint();
	PointF mTouch = new PointF();
	
	PointF[] mBezier1;
	PointF[] mBezier2;
	
	float mLastOffset[] = new float[ConstChar.length]; // 记录每一个字母的x方向偏移量, 数字<=0
	PointF mLastFucusPostion = new PointF();
	
	Scroller mScroller;
	boolean mAnimating = false;
	float mAnimationOffset;
	
	boolean mHideAnimation = false;
	int mAlpha = 255; 
	
	Handler mHideWaitingHandler = new Handler() {
		
		@Override
		public void handleMessage(Message msg) {
			if( msg.what == 1 )
			{
//				mScroller.startScroll(0, 0, 255, 0, 1000);
				mHideAnimation = true;
				mAnimating = false;
				FancyIndexer.this.invalidate();
				return;
			}
			super.handleMessage(msg);
		}
	};
	
	public FancyIndexer(Context context, AttributeSet attrs, int defStyle) {
		super(context, attrs, defStyle);
		initData(context, attrs);
	}

	public FancyIndexer(Context context, AttributeSet attrs) {
		super(context, attrs);
		initData(context, attrs);
	}

	public FancyIndexer(Context context) {
		super(context);
		initData(null, null);
	}
	
	private void initData(Context context, AttributeSet attrs) {
		
		if( context != null && attrs != null ) {
			
			TypedArray a = context.obtainStyledAttributes(attrs, R.styleable.FancyIndexer, 0, 0);
			
			mWidthOffset    = a.getDimension(R.styleable.FancyIndexer_widthOffset, mWidthOffset);
			mMinFontSize    = a.getInteger(R.styleable.FancyIndexer_minFontSize, mMinFontSize);
			mMaxFontSize    = a.getInteger(R.styleable.FancyIndexer_maxFontSize, mMaxFontSize);
			mTipFontSize    = a.getInteger(R.styleable.FancyIndexer_tipFontSize, mTipFontSize);
			mMaxBezierHeight = a.getDimension(R.styleable.FancyIndexer_maxBezierHeight, mMaxBezierHeight);
			mMaxBezierWidth = a.getDimension(R.styleable.FancyIndexer_maxBezierWidth, mMaxBezierWidth);			
			mMaxBezierLines =  a.getInteger(R.styleable.FancyIndexer_maxBezierLines, mMaxBezierLines);
			mAdditionalTipOffset = a.getDimension(R.styleable.FancyIndexer_additionalTipOffset, mAdditionalTipOffset);
			mFontColor = a.getColor(R.styleable.FancyIndexer_fontColor, mFontColor);
			mTipFontColor = a.getColor(R.styleable.FancyIndexer_tipFontColor, mTipFontColor);
			a.recycle();
		}
		mScroller = new Scroller( getContext() );
		mTouch.x = 0;
		mTouch.y = -10*mMaxBezierWidth;
		
		mBezier1 = new PointF[mMaxBezierLines];
		mBezier2 = new PointF[mMaxBezierLines];
		
		calculateBezierPoints();
		
	}
	
	@Override
	protected void onDraw(Canvas canvas) {
		
		// 控件宽高
	    int height = getHeight();
	    int width = getWidth();

	    // 单个字母高度
	    float singleHeight = height / (float)ConstChar.length;
	    
	    int workHeight = 0;
	    
	    if( mAlpha == 0 )
	    	return;
	    
	    mPaint.reset();
	    
	    int saveCount = 0;
	    
	    if( mHideAnimation )
	    {
	    	saveCount = canvas.save();
	    	canvas.saveLayerAlpha( 0, 0, width, height, mAlpha, Canvas.ALL_SAVE_FLAG );
	    }
	    
	    for(int i=0;i<ConstChar.length;i++) {
	    	
	       mPaint.setColor(mFontColor);
	       mPaint.setAntiAlias(true);
	       
	       float xPos = width - mWidthOffset;
	       float yPos = workHeight + singleHeight/2;
	       
	       //float adjustX = adjustXPos( yPos, i == mChooseIndex );
	       // 根据当前字母y的位置计算得到字体大小
	       int fontSize = adjustFontSize(i, yPos ); 
	       mPaint.setTextSize(fontSize);
    	   
	       // 添加一个字母的高度
	       workHeight += singleHeight;
	       
	       // 绘制字母
	       drawTextInCenter(canvas, ConstChar[i], xPos + ajustXPosAnimation(i,  yPos )  , yPos );
	       
	       // 绘制的字母和当前触摸到的一致, 绘制红色被选中字母
	       if(i == mChooseIndex) {
	    	   mPaint.setColor( mTipFontColor );
	    	   mPaint.setFakeBoldText(true);
	    	   mPaint.setTextSize( mTipFontSize );
	    	   yPos = mTouch.y;
	    	   
	    	   float pos = 0;
	    	   
	    	   if( mAnimating || mHideAnimation ) {
	    		   pos = mLastFucusPostion.x;
	    		   yPos = mLastFucusPostion.y;
	    	   } else {
	    		   pos = xPos + ajustXPosAnimation(i, yPos ) - mAdditionalTipOffset;
	    		   mLastFucusPostion.x = pos;
	    		   mLastFucusPostion.y = yPos;
	    	   }
	    		   
	    	   drawTextInCenter(canvas, ConstChar[i], pos, yPos );
//	    	   mPaint.setStrokeWidth(5);
//	    	   canvas.drawLine(0, yPos, width, yPos, mPaint);
	       }
	       mPaint.reset();
	    }
	    
	    if( mHideAnimation ) 
	    {
	    	canvas.restoreToCount(saveCount);
	    }
	   
	}
	
	/**
	 * @param canvas 画板
	 * @param string 被绘制的字母
	 * @param xCenter 字母的中心x方向位置
	 * @param yCenter 字母的中心y方向位置
	 */
	private void drawTextInCenter(Canvas canvas, String string, float xCenter, float yCenter) {

		FontMetrics fm = mPaint.getFontMetrics();
		//float fontWidth = paint.measureText(string);
		float fontHeight = mPaint.getFontSpacing();
		
		float drawY = yCenter + fontHeight/2 - fm.descent;
		
		if( drawY < -fm.ascent -fm.descent )
			drawY = -fm.ascent -fm.descent;
		
		if( drawY > getHeight() )
			drawY = getHeight() ;
		
		mPaint.setTextAlign(Align.CENTER);
		
		canvas.drawText(string, xCenter, drawY, mPaint);
	}
	
	private int adjustFontSize(int i, float yPos ) {
		
		// 根据水平方向偏移量计算出一个放大的字号
		float adjustX = Math.abs(ajustXPosAnimation(i,  yPos ));
		
		int adjustSize =(int)( (mMaxFontSize - mMinFontSize ) * adjustX / (float)mMaxBezierHeight) + mMinFontSize;
		
		return adjustSize;
	}
	
	/**
	 * x 方向的向左偏移量
	 * @param i	当前字母的索引
	 * @param yPos y方向的初始位置
	 * @return
	 */
	private float ajustXPosAnimation (int i, float yPos ) {

		float offset ;
		if( this.mAnimating || this.mHideAnimation ) {
			// 正在动画中或在做隐藏动画
			offset = mLastOffset[i];
			if( offset !=0.0f ) {
				offset += this.mAnimationOffset;
				if( offset > 0)
					offset = 0;
			}
		} else {
			
			// 根据当前字母y方向位置, 计算水平方向偏移量
			offset = adjustXPos( yPos );
			
			// 当前触摸的x方向位置
			float xPos = mTouch.x  ;
			
			float width = getWidth() - mWidthOffset;
			width = width - 60;
			
			// 字母绘制时向左偏移量 进行修正, offset需要是<=0的值
			if( offset != 0.0f  && xPos > width )
				offset +=  ( xPos - width );
			if( offset > 0)
				offset = 0;
			
			mLastOffset[i] = offset;
		}
		return offset;
	}
	
	private float adjustXPos(float yPos ) {
	
		float dis = yPos - mTouch.y; // 字母y方向位置和触摸时y值坐标的差值, 距离越小, 得到的水平方向偏差越大 
		if( dis > -mMaxBezierWidth  && dis < mMaxBezierWidth ) {
			// 在2个贝赛尔曲线宽度范围以内 (一个贝赛尔曲线宽度是指一个山峰的一边)

			// 第一段 曲线
			if( dis > mMaxBezierWidth/4 ) {
				for( int i = mMaxBezierLines-1; i>0 ; i-- ) {
					// 从下到上, 逐个计算
					
					if( dis == -mBezier1[i].y ) // 落在点上
						return mBezier1[i].x;
					
					// 如果距离dis落在两个贝塞尔曲线模拟点之间, 通过三角函数计算得到当前dis对应的x方向偏移量
					if( dis > -mBezier1[i].y && dis < -mBezier1[i-1].y ) {
						return (dis + mBezier1[i].y) * ( mBezier1[i-1].x - mBezier1[i].x ) / ( -mBezier1[i-1].y + mBezier1[i].y ) + mBezier1[i].x;
					}
				}
				return mBezier1[0].x;
			}
			
			// 第三段 曲线, 和第一段曲线对称
			if( dis < -mMaxBezierWidth/4 ) {
				for( int i = 0; i< mMaxBezierLines-1; i++ ) {
					// 从上到下
					
					if( dis == mBezier1[i].y ) // 落在点上
						return mBezier1[i].x;

					// 如果距离dis落在两个贝塞尔曲线模拟点之间, 通过三角函数计算得到当前dis对应的x方向偏移量
					if( dis > mBezier1[i].y && dis < mBezier1[i+1].y ) {
						return (dis - mBezier1[i].y )* (mBezier1[i+1].x - mBezier1[i].x ) / ( mBezier1[i+1].y - mBezier1[i].y ) + mBezier1[i].x;
					}
				}
				return mBezier1[mMaxBezierLines-1].x;
			}
			
			// 第二段 峰顶曲线
			for( int i = 0; i< mMaxBezierLines-1; i++ ) {
				
				if( dis == mBezier2[i].y )
					return mBezier2[i].x;

				// 如果距离dis落在两个贝塞尔曲线模拟点之间, 通过三角函数计算得到当前dis对应的x方向偏移量
				if( dis > mBezier2[i].y && dis < mBezier2[i+1].y ) {
					return ( dis - mBezier2[i].y) * ( mBezier2[i+1].x - mBezier2[i].x ) / (mBezier2[i+1].y - mBezier2[i].y ) + mBezier2[i].x;
				}
			}	
			return mBezier2[mMaxBezierLines-1].x;		

		}
		
		return 0.0f;
	}

	@Override
	public boolean dispatchTouchEvent(MotionEvent event) {
		final int action = event.getAction();
	    final float y = event.getY();
	    final int oldmChooseIndex = mChooseIndex;
	    final OnTouchLetterChangedListener listener = mListener;
	    final int c = (int) (y/getHeight()*ConstChar.length);
	    
		switch (action) {
			case MotionEvent.ACTION_DOWN:
				
				if( this.getWidth() > mWidthOffset ) {
					if ( event.getX() < this.getWidth() - mWidthOffset )
						return false;
				}
				
				mHideWaitingHandler.removeMessages(1);
				
				mScroller.abortAnimation();
				mAnimating = false;
				mHideAnimation = false;
				mAlpha = 255;
				
				mTouch.x = event.getX();
				mTouch.y = event.getY();
				
				if(oldmChooseIndex != c && listener != null){
					if(c > 0 && c< ConstChar.length){
						listener.onTouchLetterChanged(ConstChar[c]);
						mChooseIndex = c;
					}
				}
				invalidate();				
				break;
			case MotionEvent.ACTION_MOVE:
				mTouch.x = event.getX();
				mTouch.y = event.getY();
				invalidate();
				if(oldmChooseIndex != c && listener != null){

					if(c >= 0 && c< ConstChar.length){
						listener.onTouchLetterChanged(ConstChar[c]);
						mChooseIndex = c;
					}
				}
				break;
			case MotionEvent.ACTION_UP:
			case MotionEvent.ACTION_CANCEL:

				mTouch.x = event.getX();
				mTouch.y = event.getY();

				//this.mChooseIndex = -1;
				
				mScroller.startScroll(0, 0, (int)mMaxBezierHeight, 0, 2000);
				mAnimating = true;
				postInvalidate();
				break;
		}
		return true;
	}
	
	@Override
	public void computeScroll() {
		super.computeScroll();
		if (mScroller.computeScrollOffset()) {
			if( mAnimating ) {
				float x = mScroller.getCurrX();
				mAnimationOffset = x;				
			} else if( mHideAnimation ) {
				mAlpha = 255 - (int) mScroller.getCurrX();
			}
			invalidate();
		} else if( mScroller.isFinished() ) {
			if( mAnimating ) {
				mHideWaitingHandler.sendEmptyMessage(1);
			} else if( mHideAnimation ) {
				mHideAnimation = false;
				this.mChooseIndex = -1;
				mTouch.x = -10000;
				mTouch.y = -10000;
			}

		}
	}
	public void setOnTouchLetterChangedListener( OnTouchLetterChangedListener listener) {
		this.mListener = listener;
	}

	/**
	 * 计算出所有贝塞尔曲线上的点 
	 * 个数为 mMaxBezierLines * 2 = 64
	 */
	private void calculateBezierPoints() {
		
		PointF mStart = new PointF();   // 开始点
		PointF mEnd = new PointF();		// 结束点
		PointF mControl = new PointF(); // 控制点
		
		
		// 计算第一段红色部分 贝赛尔曲线的点
		// 开始点
		mStart.x = 0.0f;
		mStart.y = -mMaxBezierWidth;

		// 控制点
		mControl.x = 0.0f;
		mControl.y = -mMaxBezierWidth/2;
		
		// 结束点
		mEnd.x = - mMaxBezierHeight / 2;
		mEnd.y = - mMaxBezierWidth / 4;
		
		mBezier1[0] = new PointF();
		mBezier1[mMaxBezierLines-1] = new PointF();
		
		mBezier1[0].set(mStart);
		mBezier1[mMaxBezierLines-1].set(mEnd);
		
		for( int i = 1; i< mMaxBezierLines -1; i++ ) {
			
			mBezier1[i] = new PointF();
			
			mBezier1[i].x = calculateBezier( mStart.x, mEnd.x, mControl.x, i / (float) mMaxBezierLines );
			mBezier1[i].y = calculateBezier( mStart.y, mEnd.y, mControl.y, i / (float) mMaxBezierLines );
			
		}

		// 计算第二段蓝色部分 贝赛尔曲线的点
		mStart.y = -mMaxBezierWidth / 4;
		mStart.x = -mMaxBezierHeight / 2;
		
		mControl.y = 0.0f;
		mControl.x = -mMaxBezierHeight;

		mEnd.y = mMaxBezierWidth / 4;
		mEnd.x = -mMaxBezierHeight / 2;
		
		mBezier2[0] = new PointF();
		mBezier2[mMaxBezierLines-1] = new PointF();

		mBezier2[0].set(mStart);
		mBezier2[mMaxBezierLines-1].set(mEnd);
		
		for( int i = 1; i< mMaxBezierLines -1 ; i++ ) {
			
			mBezier2[i]= new PointF();
			mBezier2[i].x = calculateBezier( mStart.x, mEnd.x, mControl.x,  i / (float) mMaxBezierLines );
			mBezier2[i].y = calculateBezier( mStart.y, mEnd.y, mControl.y,  i / (float) mMaxBezierLines );
		}
	}

	/**
	 * 贝塞尔曲线核心算法
	 * @param start
	 * @param end
	 * @param control
	 * @param val
	 * @return
	 * 公式及动图, 维基百科: https://en.wikipedia.org/wiki/B%C3%A9zier_curve
	 * 中文可参考此网站: http://blog.csdn.net/likendsl/article/details/7852658
	 * 
	 */
	private float calculateBezier(float start, float end, float control, float val) {
		
		float t = val;
		float s = 1-t;
		
		float ret = start * s * s + 2 * control * s * t + end * t * t;
		
		return ret;
	}	
}
```

# 关于我

- Email：<815712739@qq.com>
- CSDN博客：[Allen Iverson](http://blog.csdn.net/axi295309066)
- 新浪微博：[AndroidDeveloper](http://weibo.com/u/1848214604?topnav=1&amp;wvr=6&amp;topsug=1&amp;is_all=1)

# License

    Copyright 2015 AllenIverson

    Copyright 2012 Jake Wharton
    Copyright 2011 Patrik Åkerfeldt
    Copyright 2011 Francisco Figueiredo Jr.

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.
