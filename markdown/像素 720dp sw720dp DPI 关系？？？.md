###像素 720dp sw720dp DPI 关系？？？

四种屏幕尺寸分类：: small, normal, large, and xlarge
四种密度分类: ldpi (low), mdpi (medium), hdpi (high), and xhdpi (extra high)
需要注意的是: xhdpi是从 Android 2.2 (API Level 8)才开始增加的分类.
xlarge是从Android 2.3 (API Level 9)才开始增加的分类.
DPI是**“dot per inch”** 的缩写，每英寸像素数。

一般情况下的普通屏幕：ldpi是120，mdpi是160，hdpi是240，xhdpi是320。

根据dpi值准备5套图片资源，drawable，drawalbe-ldpi,drawable-mdpi,drawable-hdpi,drawable-xhdpi

在每英寸160点的显示器上，1dp = 1px。 以这个基准可以算出 其它的比例。

比如我手上有 1280*720的红米手机  4.7寸的   可知 每英寸像素 DPI 320  可以通过程序获得。  那么 1dp=2px；

sw720 的dp 的意思是最小720dp   红米手机的宽720px/2=360 所有  红米手机 应该是sw360   android 在运行程序的时候应该会找 drawable-xhdpi 文件下的图片 和 values-sw340dp 下的资源

**在代码中获取屏幕像素、屏幕密度** 
```
DisplayMetrics metric = new DisplayMetrics(); 
getWindowManager().getDefaultDisplay().getMetrics(metric); 
int width = metric.widthPixels; // 屏幕宽度（像素） 
int height = metric.heightPixels; // 屏幕高度（像素） 
float density = metric.density; // 屏幕密度（0.75 / 1.0 / 1.5） 
int densityDpi = metric.densityDpi; // 屏幕密度DPI（120 / 160 / 240） 
```

**Android支持下列所有单位。**
px（像素）：屏幕上的点。
in（英寸）：长度单位。
mm（毫米）：长度单位。
pt（磅）：1/72英寸。
dp（与密度无关的像素）：一种基于屏幕密度的抽象单位。在每英寸160点的显示器上，1dp = 1px。
dip：与dp相同，多用于android/ophone示例中。
sp（与刻度无关的像素）：与dp类似，但是可以根据用户的字体大小首选项进行缩放。