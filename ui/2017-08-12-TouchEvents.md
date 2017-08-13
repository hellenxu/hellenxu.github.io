Touch Events
======================
### Facts you may know
1, Both activity and view have onTouchEvent(MotionEvent event) and dispatchTouchEvent(MotionEvent event), but not applied for Fragment;
2, Besides onTouchEvent and dispatchTouchEvent, ViewGroup also has onInterceptTouchEvent(MotionEvent event);

### How do touch events propagate generally?
TouchEvent propagation goes from top to bottom. Mostly, propagation starts from dispatchTouchEvent of Activity until it finds the right target child if the parent(ViewGroup) does not hijack this event, and from then all touch events are dispatched to the target view. Otherwise, if the parent does want to intercept the event by returning true from onInterceptTouchEvent, ACTION_CANCEL will be sent to children views so they can abandon their touch event processing. From that time, the parent will handle all touch events in onTouchListener.onTouch() or onTouchEvent() method.

### Cases of touch events
+ Case I: no one consumes touch events.

+ Case 2: View handles touch events.

+ Case 3: ViewGroup handles touch events.

+ Case 4: Activity handles touch events.


### Conclusion

