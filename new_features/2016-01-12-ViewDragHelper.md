ViewDragHelper
=========
ViewDragHelper is a very useful utility class to write custom ViewGroup, which is also used in SlidingPaneLayout and DrawerLayout handling movement events.
Now we are going to talk about somethings about it as follows:
> 1. How to use ViewDragHelper
> 2. Common methods of ViewDragHelper
> 3. Source code of ViewDragHelper

Part One: How to use ViewDragHelper
-----
Usually, ViewDragHelper is used within a ViewGroup, as a tool to handle dragging events. Here, we use a custom ViewGroup that extends LinearLayout to learn how to use ViewDragHelper.

+ Step one: create a custom ViewGroup ViewDragLinearLayout as well as an instance of ViewDragHelper in constructor function.
```java
public class ViewDragLinearLayout extends LinearLayout {
    private static final String TAG = ViewDragLinearLayout.class.getSimpleName();
    private final ViewDragHelper mDragHelper;

    public ViewDragLinearLayout(Context context){
        this(context, null);
    }

    public ViewDragLinearLayout(Context context, AttributeSet attrs){
        this(context, attrs, 0);
    }

    public ViewDragLinearLayout(Context context, AttributeSet attrs, int defStyleAttr){
        super(context, attrs, defStyleAttr);
            mDragHelper = ViewDragHelper.create(this, 1.0f, new DragCallback());
    }
}
```
+ Step two: create an inner class which extends ViewDragHelper.Callback, and then override three methods: 1) tryCaptureView; 2) clampViewPositionVertical; 3) clampViewPositionHorizontal.
```java
public class ViewDragLinearLayout extends LinearLayout {
    ...
    private final class DragCallback extends ViewDragHelper.Callback{
        @Override
        public boolean tryCaptureView(View child, int pointerId){
            return true;
        }
        
        @Override
        public int clampViewPositionHorizontal(View child, int left, int dx) {
            final int leftPadding = getPaddingLeft();
            final int maxLeft = getWidth() -leftPadding - child.getWidth();
            return Math.max(leftPadding, Math.min(maxLeft, left));
        }
        
        @Override
        public int clampViewPositionVertical(View child, int top, int dy) {
            final int topPadding = getPaddingTop();
            final int maxTop = getHeight() - topPadding - child.getHeight();
            return Math.max(topPadding, Math.min(maxTop, top));
        }
    }
    ...
}
```
+ Step three: use this custom ViewGroup, ViewDragLinearLayout, in a layout file.
```xml
	<com.six.newfeatures.DraggingView xmlns:android="http://schemas.android.com/apk/res/android"
                  android:layout_width="match_parent"
                  android:layout_height="match_parent"
                  android:orientation="vertical">
    	<TextView android:layout_width="80dp"
    	          android:layout_height="80dp"
    	          android:background="@android:color/holo_blue_light"/>
    </com.six.newfeatures.DraggingView>
```

Now, it is time to launch the app and see how ViewDragHelper works. 
Click [here](https://github.com/hellenxu/new_features/blob/master/app/src/main/java/com/six/newfeatures/DraggingView.java) to see the whole sample.
