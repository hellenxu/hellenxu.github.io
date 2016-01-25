ViewDragHelper
=========
ViewDragHelper is a very useful utility class to write custom ViewGroup, which is also used in SlidingPaneLayout and DrawerLayout handling movement events.
Now we are going to talk about somethings about it as follows:
> 1. How to use ViewDragHelper
> 2. Common methods of ViewDragHelper
> 3. How does ViewDragHelper work

Part Two: Common methods of ViewDragHelper
------
+ ViewDragHelper.abort()
This method cancel as well as abort all motion in progress and jump to the end of any animation.

+ ViewDragHelper.cancel()
Calling this method is equivalent to processing motion events which are canceled.

+ ViewDragHelper.captureChildView(View childView, int activePointerId)
You can capture one specific child view by calling this method, and I usually use it within the onFinishInflate method of a parent view. Also, you need to use this method with tryCaptureView of ViewDragHelper.Callback to completing capturing one specific child view rather than others.

```java
    @Override
    protected void onFinishInflate() {
        super.onFinishInflate();
        mDragView = getChildAt(0);
        mViewDragHelper.captureChildView(mDragView, mViewDragHelper.getActivePointerId());
    }

    private final class DragCallback extends ViewDragHelper.Callback {
    ...
        @Override
        public boolean tryCaptureView(View child, int pointerId) {
            return child.equals(mDragView);
        }
    ...
    }
```

+ ViewDragHelper.processTouchEvent(MotionEvent ev)
ViewDragHelper.processTouchEvent is used to process motion events received by the parent view, so we often call this method within the onTouchEvent of the parent view.
```java
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        mDragHelper.processTouchEvent(event);
        //returning true means that motion events are handled by the parent view
        return true;
    }
```

+ ViewDragHelper.shouldInterceptTouchEvent(MotionEvent ev)
Call within onInterceptTouchEvent of the parent view, checking whether the parent view should intercept the touch event stream.

```java
    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        switch(ev.getAction()) {
            case MotionEvent.ACTION_CANCEL:
            case MotionEvent.ACTION_DOWN:
                mDragHelper.cancel();
        }
        return mDragHelper.shouldInterceptTouchEvent(ev);
    }
```

+ ViewDragHelper.Callback

1. Callback.tryCaptureView

2. Callback.clampViewPositionHorizontal

3. Callback.clampViewPositionVertical