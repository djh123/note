###对TextView获取焦点和onclick冲突的解决

分析View源码中的onTouchEvent源码处理:(\android_src\frameworks\base\core\java\android\view):
```
case MotionEvent.ACTION_UP:
 boolean focusTaken = false;
 if (isFocusable() && isFocusableInTouchMode() && !isFocused()) {
   focusTaken = requestFocus();//设置焦点
 }
 if (!focusTaken) {
   // Use a Runnable and post this rather than calling
   // performClick directly. This lets other visual state
   // of the view update before click actions start.
    if (mPerformClick == null) {
         mPerformClick = new PerformClick();
    }
   if (!post(mPerformClick)) {
         performClick();
    }
  }
```

> `知道如果isFocusableInTouchMode是false,就是没有设置获取触摸焦点属性时候,我们的View会主动调用performClick触发onClick()回调，但是如果isFocusableInTouchMode是true，就不会主动触发onClick，而是像按键处理一样去设置焦点。`
当我们的控件(比如Button)设置了相关属性:
```
setFocusable(true)
setFocusableInTouchMode(true)
```
那么当我们按键处理的时候肯定是正常的，但是当我们点击该控件时候，肯定会调用**onFocusChange**回调，前面我们分析了是不会触发**onClick**回调的
也就是说，这种情况下我们只是把焦点移到该控件，需要我们再在该控件点击一下，才会触发**onClick**事件.
所以主界面我们做了一个处理,达到的效果就是我们用鼠标点击控件时候，不仅焦点移动到该控件，还会触发**onClick**事件：
默认没有设置**setFocusableInTouchMode**时候是直接会触发**onClick**事件，而且是没有焦点的概念。
**setFocusableInTouchMode(true)**后会设置焦点，但是不会触发**onClick**事件。
我们的做法就是在**onFocusChange**实现我们自己的处理，如下:
```
public void onFocusChange(View v, boolean hasFocus) {
   if (hasFocus) {  
    if (v.isInTouchMode())//判断是否是触摸，鼠标点击来切换焦点
    {  
     ((Button)v).performClick();  
    }  
   }
 }
}
```
>调用performClick方法主动触发控件的onClick事件(和View源码onTouchEvent的MotionEvent.ACTION_UP一样处理)。实际上我们也可以利用performClick方法来模拟任何控件的鼠标点击处理等(一般不需要我们这样做而已)