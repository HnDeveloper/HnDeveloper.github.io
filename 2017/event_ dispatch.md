> 作者：谢昆

# touch事件的分发机制


一段伪代码反应整个touch事件的分发
```
public boolean dispatchTouchEvent(MotionEvent event) { 
    boolean consume = false;
    if (onInteceptTouchEvent(event)) { 
        consume = onTouchEvent(event); 
    } else { 
        consume = child.dispatchTouchEvent(event); 
    } 
    return consume;
}
```
### View的事件分发

![image](http://img.blog.csdn.net/20130616154134500)


```
public boolean dispatchTouchEvent(MotionEvent event) {  
    if (mOnTouchListener != null && (mViewFlags & ENABLED_MASK) == ENABLED &&  
            mOnTouchListener.onTouch(this, event)) {  
        return true;  
    }  
    return onTouchEvent(event);  
}  
```
1. 第一个参数是我们设置的onTouchListener
2. 第二个参数(mViewFlags & ENABLED_MASK) == ENABLED是判断当前点击的控件是否是enable的，按钮默认都是enable的，因此这个条件恒定为true
3. 第三个参数就是mOnTouchListener.onTouch(this, event)方法的返回值


onTouchEvent(MotionEvent event)的源码：
```
public boolean onTouchEvent(MotionEvent event) {  
    final int viewFlags = mViewFlags;  
    if ((viewFlags & ENABLED_MASK) == DISABLED) {  
        // A disabled view that is clickable still consumes the touch  
        // events, it just doesn't respond to them.  
        return (((viewFlags & CLICKABLE) == CLICKABLE ||  
                (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE));  
    }  
    if (mTouchDelegate != null) {  
        if (mTouchDelegate.onTouchEvent(event)) {  
            return true;  
        }  
    }  
    if (((viewFlags & CLICKABLE) == CLICKABLE ||  
            (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE)) {  
        switch (event.getAction()) {  
            case MotionEvent.ACTION_UP:  
                boolean prepressed = (mPrivateFlags & PREPRESSED) != 0;  
                if ((mPrivateFlags & PRESSED) != 0 || prepressed) {  
                    // take focus if we don't have it already and we should in  
                    // touch mode.  
                    boolean focusTaken = false;  
                    if (isFocusable() && isFocusableInTouchMode() && !isFocused()) {  
                        focusTaken = requestFocus();  
                    }  
                    if (!mHasPerformedLongPress) {  
                        // This is a tap, so remove the longpress check  
                        removeLongPressCallback();  
                        // Only perform take click actions if we were in the pressed state  
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
                    }  
                    if (mUnsetPressedState == null) {  
                        mUnsetPressedState = new UnsetPressedState();  
                    }  
                    if (prepressed) {  
                        mPrivateFlags |= PRESSED;  
                        refreshDrawableState();  
                        postDelayed(mUnsetPressedState,  
                                ViewConfiguration.getPressedStateDuration());  
                    } else if (!post(mUnsetPressedState)) {  
                        // If the post failed, unpress right now  
                        mUnsetPressedState.run();  
                    }  
                    removeTapCallback();  
                }  
                break;  
            case MotionEvent.ACTION_DOWN:  
                if (mPendingCheckForTap == null) {  
                    mPendingCheckForTap = new CheckForTap();  
                }  
                mPrivateFlags |= PREPRESSED;  
                mHasPerformedLongPress = false;  
                postDelayed(mPendingCheckForTap, ViewConfiguration.getTapTimeout());  
                break;  
            case MotionEvent.ACTION_CANCEL:  
                mPrivateFlags &= ~PRESSED;  
                refreshDrawableState();  
                removeTapCallback();  
                break;  
            case MotionEvent.ACTION_MOVE:  
                final int x = (int) event.getX();  
                final int y = (int) event.getY();  
                // Be lenient about moving outside of buttons  
                int slop = mTouchSlop;  
                if ((x < 0 - slop) || (x >= getWidth() + slop) ||  
                        (y < 0 - slop) || (y >= getHeight() + slop)) {  
                    // Outside button  
                    removeTapCallback();  
                    if ((mPrivateFlags & PRESSED) != 0) {  
                        // Remove any future long press/tap checks  
                        removeLongPressCallback();  
                        // Need to switch from pressed to not pressed  
                        mPrivateFlags &= ~PRESSED;  
                        refreshDrawableState();  
                    }  
                }  
                break;  
        }  
        return true;  
    }  
    return false;  
}
```

首先在第14行我们可以看出，如果该控件是可以点击的就会进入到第16行的switch判断中去，而如果当前的事件是抬起手指，则会进入到MotionEvent.ACTION_UP这个case当中。在经过种种判断之后，会执行到第38行的performClick()方法

performClick()的源码：
```
public boolean performClick() {  
    sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_CLICKED);  
    if (mOnClickListener != null) {  
        playSoundEffect(SoundEffectConstants.CLICK);  
        mOnClickListener.onClick(this);  
        return true;  
    }  
    return false;  
}  
```

setOnClickListener(OnClickListener l)的源码：
```
public void setOnClickListener(OnClickListener l) {  
    if (!isClickable()) {  
        setClickable(true);  
    }  
    mOnClickListener = l;  
}
```

### ViewGroup的事件分发

![image](http://img.blog.csdn.net/20130629154352906?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2lueXU4OTA4MDc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)


```
public boolean dispatchTouchEvent(MotionEvent ev) {  
    final int action = ev.getAction();  
    final float xf = ev.getX();  
    final float yf = ev.getY();  
    final float scrolledXFloat = xf + mScrollX;  
    final float scrolledYFloat = yf + mScrollY;  
    final Rect frame = mTempRect;  
    boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;  
    if (action == MotionEvent.ACTION_DOWN) {  
        if (mMotionTarget != null) {  
            mMotionTarget = null;  
        }  
        if (disallowIntercept || !onInterceptTouchEvent(ev)) {  
            ev.setAction(MotionEvent.ACTION_DOWN);  
            final int scrolledXInt = (int) scrolledXFloat;  
            final int scrolledYInt = (int) scrolledYFloat;  
            final View[] children = mChildren;  
            final int count = mChildrenCount;  
            for (int i = count - 1; i >= 0; i--) {  
                final View child = children[i];  
                if ((child.mViewFlags & VISIBILITY_MASK) == VISIBLE  
                        || child.getAnimation() != null) {  
                    child.getHitRect(frame);  
                    if (frame.contains(scrolledXInt, scrolledYInt)) {  
                        final float xc = scrolledXFloat - child.mLeft;  
                        final float yc = scrolledYFloat - child.mTop;  
                        ev.setLocation(xc, yc);  
                        child.mPrivateFlags &= ~CANCEL_NEXT_UP_EVENT;  
                        if (child.dispatchTouchEvent(ev))  {  
                            mMotionTarget = child;  
                            return true;  
                        }  
                    }  
                }  
            }  
        }  
    }  
    boolean isUpOrCancel = (action == MotionEvent.ACTION_UP) ||  
            (action == MotionEvent.ACTION_CANCEL);  
    if (isUpOrCancel) {  
        mGroupFlags &= ~FLAG_DISALLOW_INTERCEPT;  
    }  
    final View target = mMotionTarget;  
    if (target == null) {  
        ev.setLocation(xf, yf);  
        if ((mPrivateFlags & CANCEL_NEXT_UP_EVENT) != 0) {  
            ev.setAction(MotionEvent.ACTION_CANCEL);  
            mPrivateFlags &= ~CANCEL_NEXT_UP_EVENT;  
        }  
        return super.dispatchTouchEvent(ev);  
    }  
    if (!disallowIntercept && onInterceptTouchEvent(ev)) {  
        final float xc = scrolledXFloat - (float) target.mLeft;  
        final float yc = scrolledYFloat - (float) target.mTop;  
        mPrivateFlags &= ~CANCEL_NEXT_UP_EVENT;  
        ev.setAction(MotionEvent.ACTION_CANCEL);  
        ev.setLocation(xc, yc);  
        if (!target.dispatchTouchEvent(ev)) {  
        }  
        mMotionTarget = null;  
        return true;  
    }  
    if (isUpOrCancel) {  
        mMotionTarget = null;  
    }  
    final float xc = scrolledXFloat - (float) target.mLeft;  
    final float yc = scrolledYFloat - (float) target.mTop;  
    ev.setLocation(xc, yc);  
    if ((target.mPrivateFlags & CANCEL_NEXT_UP_EVENT) != 0) {  
        ev.setAction(MotionEvent.ACTION_CANCEL);  
        target.mPrivateFlags &= ~CANCEL_NEXT_UP_EVENT;  
        mMotionTarget = null;  
    }  
    return target.dispatchTouchEvent(ev);  
}
```


在第13行可以看到一个条件判断，如果disallowIntercept和!onInterceptTouchEvent(ev)两者有一个为true，就会进入到这个条件判断中。接着如果进入条件语句就回去遍历查找点击的是哪一个子view，进而调用子view的dispatchTouchEvent(ev)方法将事件传递下去。

那如果遍历没有找到目标的子view，则会继续往下面走，会在第50行调用super.dispatchTouchEvent(ev)，这里就是View中的dispatchTouchEvent(ev)，这样就执行完整个的流程。

![image](http://img.blog.csdn.net/20130629200236578?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc2lueXU4OTA4MDc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)


### activity对事件的分发过程

activity的dispatchTouchEvent(MotionEvent ev)方法：
```
public boolean dispatchTouchEvent(MotionEvent ev) {
    if (ev.getAction() == MotionEvent.ACTION_DOWN) {
        onUserInteraction();
    }
    if (getWindow().superDispatchTouchEvent(ev)) {
        return true;
    }
    return onTouchEvent(ev);
}
```

主要就通过window来分发事件，如果没有view消费掉事件，就调用activity的onTouchEvent(ev)

Window是一个抽象类，他的实现类就是PhoneWindow，PhoneWindow中的superDispatchTouchEvent(ev)的方法：

```
public boolean superDispatchTouchEvent(MotionEvent ev){
    return mDecor.superDispatchTouchEvent(ev);
}
```
