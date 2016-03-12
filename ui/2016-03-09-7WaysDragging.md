Seven ways to drag views
======================
No talking, just show the code;)

###Method No.1 View.layout(int left, int top, int right, int bottom)

```java
	public SixDraggingView extends View {
		...
		private int lastX;
		private int lastY;
		@Override
		public boolean onTouchEvent(MotionEvent event) {
			switch(event.getAction()){
				case MotionEvent.ACTION_DOWN:
					lastX = (int)event.getX();
					lastY = (int)event.getY();
				break;
				case MotionEvent.ACTION_MOVE:
					int offsetX = (int) (event.getX() - lastX);
					int offsetY = (int) (event.getY() - lastY);
					layout(getLeft() + offsetX, getTop() + offsetY, getRight() + offsetX, getBottom() + offsetY);
				break;
				case MotionEvent.ACTION_UP:
				...
				break;
				default:
				break;
			}
		}
		...
	}
```


###Method No.2 View.offsetLeftAndRight(int offset) && View.offsetTopAndBottom(int offset)

```java
	public SixDraggingView extends View {
	...
		private int lastX;
		private int lastY;	
		@Override
		public boolean onTouchEvent(MotionEvent event) {
			switch(event.getAction()){
				case MotionEvent.ACTION_DOWN:
					lastX = (int)event.getX();
					lastY = (int)event.getY();
					break;					
				case MotionEvent.ACTION_MOVE:
					int offsetX = (int) (event.getX() - lastX);
					int offsetY = (int) (event.getY() - lastY);
					offsetLeftAndRight(offsetX);
					offsetTopAndBottom(offsetY);
					break;																		
				case MotionEvent.ACTION_UP:
					...
					break;
				default:
				break;
			}
		}
	...
	}
```

###Method No.3 View.setLayoutParams(LayoutParams params)
```java
	public SixDraggingView extends View {
	...
		private int lastX;
		private int lastY;	
		@Override
		public boolean onTouchEvent(MotionEvent event) {
			switch(event.getAction()){
				case MotionEvent.ACTION_DOWN:
					lastX = (int)event.getX();
					lastY = (int)event.getY();
					break;					
				case MotionEvent.ACTION_MOVE:
					int offsetX = (int) (event.getX() - lastX);
					int offsetY = (int) (event.getY() - lastY);
					ViewGroup.MarginLayoutParams layoutParams = (ViewGroup.MarginLayoutParams) getLayoutParams();
					layoutParams.leftMargin = getLeft() + offsetX;
					layoutParams.topMargin = getTop() + offsetY;
					break;																		
				case MotionEvent.ACTION_UP:
					...
					break;
				default:
				break;
			}
		}
	...
	}
```

###Method No.4 View.scrollTo(int x, int y) or scrollBy(int x, int y)
```java
	public SixDraggingView extends View {
	...
    	private int lastX;
    	private int lastY;
    	@Override
    	public boolean onTouchEvent(MotionEvent event) {
    		switch(event.getAction()){
    			case MotionEvent.ACTION_DOWN:
    				lastX = (int)event.getX();
    				lastY = (int)event.getY();
    				break;
    			case MotionEvent.ACTION_MOVE:
    				int offsetX = (int) (event.getX() - lastX);
    				int offsetY = (int) (event.getY() - lastY);
    				scrollBy(offsetX, offsetY);
    				break;
    		}
    	}
	...
	}
```

###Method No.5 Scroller
```java
	public SixDraggingView extends View {
        ...
        private Scroller mScroller;
        public SixDraggingView(Context context, AttributeSet attrs, int defStyleAttr) {
                super(context, attrs, defStyleAttr);
                mScroller = new Scroller(context);
        }

        @Override
        public boolean onTouchEvent(MotionEvent event) {
        	 switch(event.getAction()){
                 case MotionEvent.ACTION_DOWN:
                 	lastX = (int)event.getX();
                 	lastY = (int)event.getY();
                 	break;
                 case MotionEvent.ACTION_MOVE:
                 	int offsetX = (int) (event.getX() - lastX);
                 	int offsetY = (int) (event.getY() - lastY);
                 	((View) getParent()).scrollBy(-offsetX, -offsetY);
                 	break;
                 case MotionEvent.ACTION_UP:
                 	View viewGroup = (View) getParent();
                    mScroller.startScroll(viewGroup.getScrollX(), viewGroup.getScrollY(), -viewGroup.getScrollX(), -viewGroup.getScrollY());
                    invalidate();
                    break;
             }
        }

        @Override
        public void computeScroll() {
        	super.computeScroll();
            if (mScroller.computeScrollOffset()) {
            	//Scroller.getCurrX && getCurrY is the current offset in scrolling.
                ((View) getParent()).scrollTo(mScroller.getCurrX(), mScroller.getCurrY());
                invalidate();
                }
            }
        ...
	}
```

###Method No.6 ViewDragHelper

###Method No.7 Animation

