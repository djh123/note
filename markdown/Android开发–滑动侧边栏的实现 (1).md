#Android开发--滑动侧边栏的实现
##在Android应用开发中，滑动侧边栏经常使用，今天我也试着自己进行了一个简单的实践，虽然功能还不是很强大，但是可以保留下来为以后的开发使用，有需要时在进行简单的修改。实现一个滑动侧边栏思路也很简单：

- 1.重写一个SlidingMenu类继承ViewGroup，病危该ViewGroup添加两个子布局，分别为菜单和主界面显示；

- 2.为了得到一个滑动的效果，选择Scroller帮助我们实现，配合ViewGroup下的computeScroll方法实现界面的更新；

- 3.利用一个boolean来记录菜单是否打开，在菜单打开的状态下向右滑动不会响应，在菜单关闭的情况向左滑动不会响应；

- 4.为了得到一个良好的交互，我们可以为界面滑动与手指移动的距离定义一个比例，如每次触摸事件发生，界面移动的距离仅为手指移动距离的一半；
#### 下面是两张效果图，界面没怎么布局，大家凑合看

![enter image description here](http://www.2cto.com/uploadfile/Collfiles/20141230/20141230084941212.jpg)
![enter image description here](http://www.2cto.com/uploadfile/Collfiles/20141230/20141230084942213.jpg)


``` java
package com.example.test;

import android.content.Context; import android.view.MotionEvent; import android.view.View; import android.view.ViewGroup; import android.widget.Scroller;

public class SlidingMenu extends ViewGroup {

private static final String TAG = SlidingMenu.class.getName();

private enum Scroll_State {
    Scroll_to_Open, Scroll_to_Close;
}

private Scroll_State state;
private int mMostRecentX;
private int downX;
private boolean isOpen = false;

private View menu;
private View mainView;

private Scroller mScroller;

private OnSlidingMenuListener onSlidingMenuListener;

public SlidingMenu(Context context, View main, View menu) {
    super(context);
    // TODO Auto-generated constructor stub
    setMainView(main);
    setMenu(menu);
    init(context);
}

private void init(Context context) {
    mScroller = new Scroller(context);
}

@Override
protected void onLayout(boolean arg0, int l, int t, int r, int b) {
    // TODO Auto-generated method stub
    mainView.layout(l, t, r, b);
    menu.layout(-menu.getMeasuredWidth(), t, 0, b);
}

public void setMainView(View view) {
    mainView = view;
    addView(mainView);
}

public void setMenu(View view) {
    menu = view;
    addView(menu);
}

@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    // TODO Auto-generated method stub
    super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    mainView.measure(widthMeasureSpec, heightMeasureSpec);
    menu.measure(widthMeasureSpec - 150, heightMeasureSpec);
}

@Override
public boolean onTouchEvent(MotionEvent event) {
    switch (event.getAction()) {
    case MotionEvent.ACTION_DOWN:
        mMostRecentX = (int) event.getX();
        downX = (int) event.getX();
        break;
    case MotionEvent.ACTION_MOVE:
        int moveX = (int) event.getX();
        int deltaX = mMostRecentX - moveX;
        // 如果在菜单打开时向右滑动及菜单关闭时向左滑动不会触发Scroll事件
        if ((!isOpen && (downX - moveX) < 0)
                || (isOpen && (downX - moveX) > 0)) {
            scrollBy(deltaX / 2, 0);
        }
        mMostRecentX = moveX;
        break;
    case MotionEvent.ACTION_UP:
        int upX = (int) event.getX();
        int dx = upX - downX;
        if (!isOpen) {// 菜单关闭时
            // 向右滑动超过menu一半宽度才会打开菜单
            if (dx > menu.getMeasuredWidth() / 3) {
                state = Scroll_State.Scroll_to_Open;
            } else {
                state = Scroll_State.Scroll_to_Close;
            }
        } else {// 菜单打开时
            // 当按下时的触摸点在menu区域时，只有向左滑动超过menu的一半，才会关闭
            // 当按下时的触摸点在main区域时，会立即关闭
            if (downX < menu.getMeasuredWidth()) {
                if (dx < -menu.getMeasuredWidth() / 3) {
                    state = Scroll_State.Scroll_to_Close;
                } else {
                    state = Scroll_State.Scroll_to_Open;
                }
            } else {
                state = Scroll_State.Scroll_to_Close;
            }
        }
        smoothScrollto();
        break;
    default:
        break;
    }
    return true;
}

private void smoothScrollto() {
    int scrollx = getScrollX();
    switch (state) {
    case Scroll_to_Close:
        mScroller.startScroll(scrollx, 0, -scrollx, 0, 500);
        if (onSlidingMenuListener != null && isOpen) {
            onSlidingMenuListener.close();
        }
        isOpen = false;
        break;
    case Scroll_to_Open:
        mScroller.startScroll(scrollx, 0,
                -scrollx - menu.getMeasuredWidth(), 0, 500);
        if (onSlidingMenuListener != null && !isOpen) {
            onSlidingMenuListener.close();
        }
        isOpen = true;
        break;
    default:
        break;
    }
}

@Override
public void computeScroll() {
    if (mScroller.computeScrollOffset()) {
        scrollTo(mScroller.getCurrX(), 0);
    }
    invalidate();
}

public void open() {
    state = Scroll_State.Scroll_to_Open;
    smoothScrollto();
}

public void close() {
    state = Scroll_State.Scroll_to_Close;
    smoothScrollto();
}

public boolean isOpen() {
    return isOpen;
}

public void setOnSlidingMenuListener(
        OnSlidingMenuListener onSlidingMenuListener) {
    this.onSlidingMenuListener = onSlidingMenuListener;
}

public interface OnSlidingMenuListener {
    public void open();

    public void close();
}
}
```
####在MainActivity中进行调用

```java
package com.example.test;

 import android.os.Bundle; 
 import android.support.v4.app.FragmentActivity; 
 import android.view.LayoutInflater; 
 import android.view.View; 
 import android.view.View.OnClickListener; 
 import android.widget.Button;

public class MainActivity extends Activity {

private Button openButton;
private Button closeButton;
private SlidingMenu mSlidingMenu;

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    mSlidingMenu = new SlidingMenu(this, LayoutInflater
            .from(this).inflate(R.layout.fragment1, null), LayoutInflater
            .from(this).inflate(R.layout.fragment2, null));
    setContentView(mSlidingMenu);//注意setContentView需要换为我们的SlidingMenu
    openButton = (Button) findViewById(R.id.button1);
    closeButton = (Button) findViewById(R.id.button_close);
    openButton.setOnClickListener(new OnClickListener() {

        @Override
        public void onClick(View arg0) {
            // TODO Auto-generated method stub
            mSlidingMenu.open();
        }
    });
    closeButton.setOnClickListener(new OnClickListener() {

        @Override
        public void onClick(View arg0) {
            // TODO Auto-generated method stub
            mSlidingMenu.close();
        }
    });
}
}
```