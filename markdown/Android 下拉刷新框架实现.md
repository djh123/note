### Android 下拉刷新框架实现

前段时间项目中用到了下拉刷新功能，之前在网上也找到过类似的demo，但这些demo的质量参差不齐，用户体验也不好，接口设计也不行。最张没办法，终于忍不了了，自己就写了一个下拉刷新的框架，这个框架是一个通用的框架，效果和设计感觉都还不错，现在分享给各位看官。
####1. 关于下拉刷新
下拉刷新这种用户交互最早由twitter创始人洛伦•布里切特(Loren Brichter)发明，有理论认为，下拉刷新是一种适用于按照从新到旧的时间顺序排列feeds的应用，在这种应用场景中看完旧的内容时，用户会很自然地下拉查找更新的内容，因此下拉刷新就显得非常合理。大家可以参考这篇文章：有趣的下拉刷新，下面我贴出一个有趣的下拉刷新的案例。
![enter image description here](http://img.blog.csdn.net/20131012143637093?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGVlaG9uZzIwMDU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

图一、有趣的下拉刷新案例（一）
![enter image description here](http://img.blog.csdn.net/20131012143646500?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGVlaG9uZzIwMDU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

####2. 实现原理
>上面这些例子，外观做得再好看，他的本质上都一样，那就是一个下拉刷新控件通常由以下几部分组成：

**【1】Header**

>Header通常有下拉箭头，文字，进度条等元素，根据下拉的距离来改变它的状态，从而显示不同的样式

**【2】Content**
>这部分是内容区域，网上有很多例子都是直接在ListView里面添加Header，但这就有局限性，因为好多情况下并不一定是用ListView来显示数据。我们把要显示内容的View放置在我们的一个容器中，如果你想实现一个用ListView显示数据的下拉刷新，你需要创建一个ListView旋转到我的容器中。我们处理这个容器的事件（down, move, up），如果向下拉，则把整个布局向下滑动，从而把header显示出来。

**【3】Footer**
>Footer可以用来显示向上拉的箭头，自动加载更多的进度条等。
以上三部分总结的说来，就是如下图所示的这种布局结构：
![enter image description here](http://img.blog.csdn.net/20131012144804468?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGVlaG9uZzIwMDU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

>关于上图，需要说明几点：
1、这个布局扩展于LinearLayout，垂直排列
2、从上到下的顺序是：Header, Content, Footer
3、Content填充满父控件，通过设置top, bottom的padding来使Header和Footer不可见，也就是让它超出屏幕外
4、下拉时，调用scrollTo方法来将整个布局向下滑动，从而把Header显示出来，上拉正好与下拉相反。
5、派生类需要实现的是：将Content View填充到父容器中，比如，如果你要使用的话，那么你需要把ListView, ScrollView, WebView等添加到容器中。
6、上图中的红色区域就是屏的大小（严格来说，这里说屏幕大小并不准确，应该说成内容区域更加准确）

####3. 具体实现

明白了实现原理与过程，我们尝试来具体实现，首先，为了以后更好地扩展，设计更加合理，我们把下拉刷新的功能抽象成一个接口：

`1.IPullToRefresh<T extends View>`

它具体的定义方法如下：
```
[java] view plaincopy在CODE上查看代码片派生到我的代码片
public interface IPullToRefresh<T extends View> {  
    public void setPullRefreshEnabled(boolean pullRefreshEnabled);  
    public void setPullLoadEnabled(boolean pullLoadEnabled);  
    public void setScrollLoadEnabled(boolean scrollLoadEnabled);  
    public boolean isPullRefreshEnabled();  
    public boolean isPullLoadEnabled();  
    public boolean isScrollLoadEnabled();  
    public void setOnRefreshListener(OnRefreshListener<T> refreshListener);  
    public void onPullDownRefreshComplete();  
    public void onPullUpRefreshComplete();  
    public T getRefreshableView();  
    public LoadingLayout getHeaderLoadingLayout();  
    public LoadingLayout getFooterLoadingLayout();  
    public void setLastUpdatedLabel(CharSequence label);  
}  
```
这个接口是一个泛型的，它接受View的派生类，因为要放到我们的容器中的不就是一个View吗？

`2.PullToRefreshBase<T extends View>`

>这个类实现了IPullToRefresh接口，它是从LinearLayout继承过来，作为下拉刷新的一个抽象基类，如果你想实现ListView的下拉刷新，只需要扩展这个类，实现一些必要的方法就可以了。这个类的职责主要有以下几点：
处理onInterceptTouchEvent()和onTouchEvent()中的事件：当内容的View（比如ListView）正如处于最顶部，此时再向下拉，我们必须截断事件，然后move事件就会把后续的事件传递到onTouchEvent()方法中，然后再在这个方法中，我们根据move的距离再进行scroll整个View。
负责创建Header、Footer和Content View：在构造方法中调用方法去创建这三个部分的View，派生类可以重写这些方法，以提供不同式样的Header和Footer，它会调用createHeaderLoadingLayout和createFooterLoadingLayout方法来创建Header和Footer创建Content View的方法是一个抽象方法，必须让派生类来实现，返回一个非null的View，然后容器再把这个View添加到自己里面。
设置各种状态：这里面有很多状态，如下拉、上拉、刷新、加载中、释放等，它会根据用户拉动的距离来更改状态，状态的改变，它也会把Header和Footer的状态改变，然后Header和Footer会根据状态去显示相应的界面式样。

`3.PullToRefreshBase<T extends View>继承关系`
>这里我实现了三个下拉刷新的派生类，分别是ListView、ScrollView、WebView三个，它们的继承关系如下：

![enter image description here](http://img.blog.csdn.net/20131012194146843)

关于PullToRefreshBase类及其派和类，有几点需要说明：
对于ListView，ScrollView，WebView这三种情况，他们是否滑动到最顶部或是最底部的实现是不一样的，所以，在PullToRefreshBase类中需要调用两个抽象方法来判断当前的位置是否在顶部或底部，而其派生类必须要实现这两个方法。比如对于ListView，它滑动到最顶部的条件就是第一个child完全可见并且first postion是0。这两个抽象方法是：
```
[java] view plaincopy在CODE上查看代码片派生到我的代码片
/** 
 * 判断刷新的View是否滑动到顶部 
 *  
 * @return true表示已经滑动到顶部，否则false 
 */  
protected abstract boolean isReadyForPullDown();  
  
/** 
 * 判断刷新的View是否滑动到底 
 *  
 * @return true表示已经滑动到底部，否则false 
 */  
protected abstract boolean isReadyForPullUp();  
创建可下拉刷新的View（也就是content view）的抽象方法是
[java] view plaincopy在CODE上查看代码片派生到我的代码片
/** 
 * 创建可以刷新的View 
 *  
 * @param context context 
 * @param attrs 属性 
 * @return View 
 */  
protected abstract T createRefreshableView(Context context, AttributeSet attrs);  
```
`4.LoadingLayout`
LoadingLayout是刷新Layout的一个抽象，它是一个抽象基类。Header和Footer都扩展于这个类。这类抽象类，提供了两个抽象方法：
- **getContentSize**
这个方法返回当前这个刷新Layout的大小，通常返回的是布局的高度，为了以后可以扩展为水平拉动，所以方法名字没有取成getLayoutHeight()之类的，这个返回值，将会作为松手后是否可以刷新的临界值，如果下拉的偏移值大于这个值，就认为可以刷新，否则不刷新，这个方法必须由派生类来实现。
- **setState**
这个方法用来设置当前刷新Layout的状态，PullToRefreshBase类会调用这个方法，当进入下拉，松手等动作时，都会调用这个方法，派生类里面只需要根据这些状态实现不同的界面显示，如下拉状态时，就显示出箭头，刷新状态时，就显示loading的图标。
可能的状态值有：RESET, PULL_TO_REFRESH, RELEASE_TO_REFRESH, REFRESHING, NO_MORE_DATA

LoadingLayout及其派生类的继承关系如下图所示：

![enter image description here](http://img.blog.csdn.net/20131013000212234)

我们可以随意地制定自己的Header和Footer，我们也可以实现如图一和图二中显示的各种下拉刷新案例中的Header和Footer，只要重写上述两个方法getContentSize()和setState()就行了。HeaderLoadingLayout，它默认是显示箭头式样的布局，而RotateLoadingLayout则是显示一个旋转图标的式样。

`5.事件处理`
我们必须重写PullToRefreshBase类的两个事件相关的方法onInterceptTouchEvent()和onTouchEvent()方法。由于ListView，ScrollView，WebView它们是放到PullToRefreshBase内部的，所在事件先是传递到PullToRefreshBase#onInterceptTouchEvent()方法中，所以我们应该在这个方法中去处理ACTION_MOVE事件，判断如果当前ListView，ScrollView，WebView是否在最顶部或最底部，如果是，则开始截断事件，一旦事件被截断，后续的事件就会传递到PullToRefreshBase#onInterceptTouchEvent()方法中，我们再在ACTION_MOVE事件中去移动整个布局，从而实现下拉或上拉动作。

`6.滚动布局(scrollTo)`
如图三的布局结构可知，默认情况下Header和Footer是放置在Content View的最上面和最下面，通过设置padding来让他跑到屏幕外面去了，如果我们将整个布局向下滚动(scrollTo)一定距离，那么Header就会被显示出来，基于这种情况，所以在我的实现中，最终我是调用scrollTo来实现下拉动作的。

总的说来，实现的重要的点就这些，具体的一些细节在实现在会碰到很多，可以参考代码。

####4. 如何使用
使用下拉刷新的代码如下
```
[java] view plaincopy在CODE上查看代码片派生到我的代码片
@Override  
    public void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
          
        mPullListView = new PullToRefreshListView(this);  
        setContentView(mPullListView);  
          
        // 上拉加载不可用  
        mPullListView.setPullLoadEnabled(false);  
        // 滚动到底自动加载可用  
        mPullListView.setScrollLoadEnabled(true);  
          
        mCurIndex = mLoadDataCount;  
        mListItems = new LinkedList<String>();  
        mListItems.addAll(Arrays.asList(mStrings).subList(0, mCurIndex));  
        mAdapter = new ArrayAdapter<String>(this, android.R.layout.simple_list_item_1, mListItems);  
          
        // 得到实际的ListView  
        mListView = mPullListView.getRefreshableView();  
        // 绑定数据  
        mListView.setAdapter(mAdapter);         
        // 设置下拉刷新的listener  
        mPullListView.setOnRefreshListener(new OnRefreshListener<ListView>() {  
            @Override  
            public void onPullDownToRefresh(PullToRefreshBase<ListView> refreshView) {  
                mIsStart = true;  
                new GetDataTask().execute();  
            }  
  
            @Override  
            public void onPullUpToRefresh(PullToRefreshBase<ListView> refreshView) {  
                mIsStart = false;  
                new GetDataTask().execute();  
            }  
        });  
        setLastUpdateTime();  
          
        // 自动刷新  
        mPullListView.doPullRefreshing(true, 500);  
    }  
```
这是初始化一个下拉刷新的布局，并且调用setContentView来设置到Activity中。
在下拉刷新完成后，我们可以调用onPullDownRefreshComplete()和onPullUpRefreshComplete()方法来停止刷新和加载

####5. 运行效果
这里列出了demo的运行效果图。

![enter image description here](http://img.blog.csdn.net/20131013012938328?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGVlaG9uZzIwMDU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

![enter image description here](http://img.blog.csdn.net/20131013012947843?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGVlaG9uZzIwMDU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

####6. 源码下载
实现这个下拉刷新的框架，并不是我的原创，我也是参考了很多开源的，把我认为比较好的东西借鉴过来，从而形成我的东西，我主要是参考了下面这个demo：
https://github.com/chrisbanes/Android-PullToRefresh 这个demo写得不错，不过他这个太复杂了，我们都知道，一旦复杂了，万一我们要添加一些需要，自然也要费劲一些，我其实就是把他的简化再简化，以满足我们自己的需要。

[源码下载请猛点我](http://download.csdn.net/detail/leehong2005/6393505)

转载请说明出处
http://blog.csdn.net/leehong2005/article/details/12567757 
谢谢！！！


####7. Bug修复

已知bug修复情况如下，发现了代码bug的看官也可以给我反馈，谢谢~~~

- 1，对于ListView的下拉刷新，当启用滚动到底自动加载时，如果footer由隐藏变为显示时，出现显示异常的情况
这个问题已经修复了，修正的代码如下：
```
PullToRefreshListView#setScrollLoadEnabled方法，修正后的代码如下：
[java] view plaincopy在CODE上查看代码片派生到我的代码片
@Override  
public void setScrollLoadEnabled(boolean scrollLoadEnabled) {  
    if (isScrollLoadEnabled() == scrollLoadEnabled) {  
        return;  
    }  
      
    super.setScrollLoadEnabled(scrollLoadEnabled);  
      
    if (scrollLoadEnabled) {  
        // 设置Footer  
        if (null == mLoadMoreFooterLayout) {  
            mLoadMoreFooterLayout = new FooterLoadingLayout(getContext());  
            mListView.addFooterView(mLoadMoreFooterLayout, null, false);  
        }  
          
        mLoadMoreFooterLayout.show(true);  
    } else {  
        if (null != mLoadMoreFooterLayout) {  
            mLoadMoreFooterLayout.show(false);  
        }  
    }  
}  

```
LoadingLayout#show方法，修正后的代码如下：
```
[java] view plaincopy在CODE上查看代码片派生到我的代码片
/** 
 * 显示或隐藏这个布局 
 *  
 * @param show flag 
 */  
public void show(boolean show) {  
    // If is showing, do nothing.  
    if (show == (View.VISIBLE == getVisibility())) {  
        return;  
    }  
      
    ViewGroup.LayoutParams params = mContainer.getLayoutParams();  
    if (null != params) {  
        if (show) {  
            params.height = ViewGroup.LayoutParams.WRAP_CONTENT;  
        } else {  
            params.height = 0;  
        }  
          
        requestLayout();  
        setVisibility(show ? View.VISIBLE : View.INVISIBLE);  
    }  
}  
```
在更改LayoutParameter后，调用requestLayout()方法。
图片旋转兼容2.x系统
我之前想的是这个只需要兼容3.x以上的系统，但发现有很多网友在使用过程中遇到过兼容性问题，这次抽空将这个兼容性一并实现了。
       onPull的修改如下：
```java
[java] view plaincopy在CODE上查看代码片派生到我的代码片
@Override  
public void onPull(float scale) {  
    if (null == mRotationHelper) {  
        mRotationHelper = new ImageViewRotationHelper(mArrowImageView);  
    }  
      
    float angle = scale * 180f; // SUPPRESS CHECKSTYLE  
    mRotationHelper.setRotation(angle);  
}  

ImageViewRotationHelper主要的作用就是实现了ImageView的旋转功能，内部作了版本的区分，实现代码如下：

[java] view plaincopy在CODE上查看代码片派生到我的代码片
/** 
     * The image view rotation helper 
     *  
     * @author lihong06 
     * @since 2014-5-2 
     */  
    static class ImageViewRotationHelper {  
        /** The imageview */  
        private final ImageView mImageView;  
        /** The matrix */  
        private Matrix mMatrix;  
        /** Pivot X */  
        private float mRotationPivotX;  
        /** Pivot Y */  
        private float mRotationPivotY;  
          
        /** 
         * The constructor method. 
         *  
         * @param imageView the image view 
         */  
        public ImageViewRotationHelper(ImageView imageView) {  
            mImageView = imageView;  
        }  
          
        /** 
         * Sets the degrees that the view is rotated around the pivot point. Increasing values 
         * result in clockwise rotation. 
         * 
         * @param rotation The degrees of rotation. 
         * 
         * @see #getRotation() 
         * @see #getPivotX() 
         * @see #getPivotY() 
         * @see #setRotationX(float) 
         * @see #setRotationY(float) 
         * 
         * @attr ref android.R.styleable#View_rotation 
         */  
        public void setRotation(float rotation) {  
            if (APIUtils.hasHoneycomb()) {  
                mImageView.setRotation(rotation);  
            } else {  
                if (null == mMatrix) {  
                    mMatrix = new Matrix();  
                      
                    // 计算旋转的中心点  
                    Drawable imageDrawable = mImageView.getDrawable();  
                    if (null != imageDrawable) {  
                        mRotationPivotX = Math.round(imageDrawable.getIntrinsicWidth() / 2f);  
                        mRotationPivotY = Math.round(imageDrawable.getIntrinsicHeight() / 2f);  
                    }  
                }  
                  
                mMatrix.setRotate(rotation, mRotationPivotX, mRotationPivotY);  
                mImageView.setImageMatrix(mMatrix);  
            }  
        }  
    }  
```
最核心的就是，如果在2.x的版本上，旋转ImageView使用Matrix。

PullToRefreshBase构造方法兼容2.x
在三个参数的构造方法声明如下标注：
    @SuppressLint("NewApi")
    @TargetApi(Build.VERSION_CODES.HONEYCOMB)



大家如果还有什么问题，欢迎留言~~~


版权声明：本文为博主原创文章，未经博主允许不得转载。