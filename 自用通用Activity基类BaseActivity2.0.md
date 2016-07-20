先来说一下这个是干嘛的吧...
长时间写各式各样的Activity写烦了，到处要打日志到处要吐司提示好麻烦，于是自己整理了个Activity基类来实现一些常用的功能。
之前那个《通用BaseActivity实现暗色状态栏》http://www.jianshu.com/p/fdf5c9fd21fc 是这个的1.0版本，这回的2.0做了很多增强。
用法很简单，将开发的Activity exthends这个BaseActivity就可以用了。
以下是一点说明：
1.要实现通知栏透明（Android4.4.4+），请在你的xml布局中放置一个以下的布局块，将它放到layout顶端，其他布局跟随这个LinearLayout即可。
```
    <LinearLayout
        android:id="@+id/sys_statusBar"
        android:layout_width="fill_parent"
        android:layout_height="40dp"
        android:orientation="vertical"
        android:visibility="gone">
    </LinearLayout>
```
然后在初始化绑定完毕组件后，调用setStatusBarHeightByLayout(LinearLayout linearLayout)传入上边那个sys_statusBar就可以了。在Android4.4.4以下不支持透明通知栏的情况下这个布局会隐藏，支持的情况下这个布局会显示并且高度为通知栏的高度，此时Activity实际占用界面面积是全屏的，所以sys_statusBar实际上充当了占位布局占用了状态栏的位置，如果设置了layout主布局的背景，是可以透明到状态栏的。
至于为什么不将sys_statusBar设置个固定高度的原因是，部分Android改的系统，例如miui，状态栏高度是不同于原生安卓的，所以需要动态获取状态栏高度，使用占位布局强行将界面中其他元素下移一定的位置，实现状态栏透明的原理。

2.我默认透明后是灰色状态栏图标文字样式（Android6.0可见，如果不需要，请删除代码中有关“dark”的部分）

3.要简易吐司一些文字（短时间的提醒），直接使用toast(Object obj);即可，相对于原始的吐司，优势在于方便以及不冲突，也就是说可以连续吐司，不会滞后消息。

4.简易log打印，直接使用log(String str);即可，还是那句话，方便。

5.打开软键盘和关闭软键盘直接用openIMM(EditText editText,boolean openIMM);即可，传入一个edittext和boolean决定开关就可以了，很方便。

6.以后还会陆续添加新的功能。

上代码：
```
public class BaseActivity extends Activity {

    protected SystemBarTintManager mTintManager;

    private static final String KEY_MIUI_VERSION_CODE = "ro.miui.ui.version.code";
    private static final String KEY_MIUI_VERSION_NAME = "ro.miui.ui.version.name";
    private static final String KEY_MIUI_INTERNAL_STORAGE = "ro.miui.internal.storage";

    public static boolean isMIUI() {
        try {
            final BuildProperties prop = BuildProperties.newInstance();
            return prop.getProperty(KEY_MIUI_VERSION_CODE, null) != null || prop.getProperty(KEY_MIUI_VERSION_NAME, null) != null || prop.getProperty(KEY_MIUI_INTERNAL_STORAGE, null) != null;
        } catch (final IOException e) {
            return false;
        }
    }

    public static boolean isFlyme() {
        try {
            final Method method = Build.class.getMethod("hasSmartBar");
            return method != null;
        } catch (final Exception e) {
            return false;
        }
    }


    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
            setTranslucentStatus(true);
        }
        mTintManager = new SystemBarTintManager(this);
    }

    protected void setTranslucentStatus(boolean on) {
        Log.i(">>>",Build.VERSION.SDK_INT+"");
        if (isMIUI())setStatusBarDarkMode(true,this);
        if (isFlyme())setStatusBarDarkIcon(getWindow(),true);
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            Window window = getWindow();
            window.clearFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS
                    | WindowManager.LayoutParams.FLAG_TRANSLUCENT_NAVIGATION);
            window.getDecorView().setSystemUiVisibility(View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN
                    | View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
                    | View.SYSTEM_UI_FLAG_LAYOUT_STABLE);
            window.addFlags(WindowManager.LayoutParams.FLAG_DRAWS_SYSTEM_BAR_BACKGROUNDS);
            window.setStatusBarColor(Color.TRANSPARENT);
            window.setNavigationBarColor(Color.TRANSPARENT);

            getWindow().getDecorView().setSystemUiVisibility(View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN|View.SYSTEM_UI_FLAG_LIGHT_STATUS_BAR);
            getWindow().setStatusBarColor(Color.TRANSPARENT);
            return;
        }else if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT){
            Window win = getWindow();
            WindowManager.LayoutParams winParams = win.getAttributes();
            final int bits = WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS;
            if (on) {
                winParams.flags |= bits;
            } else {
                winParams.flags &= ~bits;
            }
            win.setAttributes(winParams);
            return;
        }
    }

    public void setStatusBarDarkMode(boolean darkmode, Activity activity) {
        Class<? extends Window> clazz = activity.getWindow().getClass();
        try {
            int darkModeFlag = 0;
            Class<?> layoutParams = Class.forName("android.view.MiuiWindowManager$LayoutParams");
            Field field = layoutParams.getField("EXTRA_FLAG_STATUS_BAR_DARK_MODE");
            darkModeFlag = field.getInt(layoutParams);
            Method extraFlagField = clazz.getMethod("setExtraFlags", int.class, int.class);
            extraFlagField.invoke(activity.getWindow(), darkmode ? darkModeFlag : 0, darkModeFlag);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static boolean setStatusBarDarkIcon(Window window, boolean dark) {
        boolean result = false;
        if (window != null) {
            try {
                WindowManager.LayoutParams lp = window.getAttributes();
                Field darkFlag = WindowManager.LayoutParams.class.getDeclaredField("MEIZU_FLAG_DARK_STATUS_BAR_ICON");
                Field meizuFlags = WindowManager.LayoutParams.class.getDeclaredField("meizuFlags");
                darkFlag.setAccessible(true);
                meizuFlags.setAccessible(true);
                int bit = darkFlag.getInt(null);
                int value = meizuFlags.getInt(lp);
                if (dark) {
                    value |= bit;
                } else {
                    value &= ~bit;
                }
                meizuFlags.setInt(lp, value);
                window.setAttributes(lp);
                result = true;
            } catch (Exception e) {
                Log.e("MeiZu", "setStatusBarDarkIcon: failed");
            }
        }
        return result;
    }

    public static class BuildProperties {

        private final Properties properties;

        private BuildProperties() throws IOException {
            properties = new Properties();
            properties.load(new FileInputStream(new File(Environment.getRootDirectory(), "build.prop")));
        }

        public boolean containsKey(final Object key) {
            return properties.containsKey(key);
        }

        public boolean containsValue(final Object value) {
            return properties.containsValue(value);
        }

        public Set<Map.Entry<Object, Object>> entrySet() {
            return properties.entrySet();
        }

        public String getProperty(final String name) {
            return properties.getProperty(name);
        }

        public String getProperty(final String name, final String defaultValue) {
            return properties.getProperty(name, defaultValue);
        }

        public boolean isEmpty() {
            return properties.isEmpty();
        }

        public Enumeration<Object> keys() {
            return properties.keys();
        }

        public Set<Object> keySet() {
            return properties.keySet();
        }

        public int size() {
            return properties.size();
        }

        public Collection<Object> values() {
            return properties.values();
        }

        public static BuildProperties newInstance() throws IOException {
            return new BuildProperties();
        }

    }

    public void setStatusBarHeightByLayout(LinearLayout linearLayout) {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
            LinearLayout linear_bar = linearLayout;
            linear_bar.setVisibility(View.VISIBLE);
            int statusHeight = getStatusBarHeight();
            android.widget.LinearLayout.LayoutParams params = (android.widget.LinearLayout.LayoutParams) linear_bar.getLayoutParams();
            params.height = statusHeight;
            linear_bar.setLayoutParams(params);
        }
    }//end setStatusBarHeight()


    /**
     * 获取状态栏的高度
     * @return
     */
    private int getStatusBarHeight(){
        try
        {
            Class<?> c=Class.forName("com.android.internal.R$dimen");
            Object obj=c.newInstance();
            Field field=c.getField("status_bar_height");
            int x=Integer.parseInt(field.get(obj).toString());
            return  getResources().getDimensionPixelSize(x);
        }catch(Exception e){
            e.printStackTrace();
        }
        return 0;
    }

    protected final static String NULL = "";
    private Toast toast;
    protected void runOnMain(Runnable runnable) {
        runOnUiThread(runnable);
    }

    //简易吐司
    public void toast(final Object obj) {
        try {
            runOnMain(new Runnable() {

                @Override
                public void run() {
                    if (toast == null)
                        toast = Toast.makeText(BaseActivity.this, NULL,Toast.LENGTH_SHORT);
                    toast.setText(obj.toString());
                    toast.show();
                }
            });
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    //简易Log
    public void log(final Object obj){
        try {
            runOnMain(new Runnable() {

                @Override
                public void run() {
                    if (BuildConfig.DEBUG) {
                        Log.i("log", obj.toString());
                    }
                }
            });
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public void openIMM(EditText editText,boolean openIMM){
        if (openIMM) {
            editText.setFocusable(true);
            editText.requestFocus();
            editText.setFocusableInTouchMode(true);
            InputMethodManager imm = (InputMethodManager) getSystemService(Context.INPUT_METHOD_SERVICE);
            imm.showSoftInput(editText, InputMethodManager.SHOW_FORCED);
        }else{
            InputMethodManager imm = (InputMethodManager) getSystemService(Context.INPUT_METHOD_SERVICE);
            imm.hideSoftInputFromWindow(editText.getWindowToken(), 0);
        }
    }
}
```
