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
+ Step two: create an inner class which extends ViewDragHelper.Callback.
```java
public class ViewDragLinearLayout extends LinearLayout {
    ...
    private final class DragCallback extends ViewDragHelper.Callback{
        @Override
        public boolean tryCaptureView(View child, int pointerId){
            return true;
        }
    }
    ...
}
```