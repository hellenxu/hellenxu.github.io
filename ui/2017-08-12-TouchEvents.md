Touch Events
======================
### Facts you may know
1. Both activity and view have onTouchEvent(MotionEvent event) and dispatchTouchEvent(MotionEvent event), but not applied for Fragment;
2. Besides onTouchEvent and dispatchTouchEvent, ViewGroup also has onInterceptTouchEvent(MotionEvent event);

### How do touch events propagate generally?
TouchEvent propagation goes from top to bottom. Mostly, propagation starts from dispatchTouchEvent of Activity until it finds the right target child if the parent(ViewGroup) does not hijack this event, and from then all touch events are dispatched to the target view. Otherwise, if the parent does want to intercept the event by returning true from onInterceptTouchEvent, ACTION_CANCEL will be sent to children views so they can abandon their touch event processing. From that time, the parent will handle all touch events in onTouchListener.onTouch() or onTouchEvent() method.

### Cases of touch events
+ Case 1: no one consumes touch events.

    `08-04 14:17:45.981 3807-3807/six.ca.androidadvanced I/System.out: activity - dispatch: down
     08-04 14:17:45.981 3807-3807/six.ca.androidadvanced I/System.out: ViewGroup - dispatch: down
     08-04 14:17:45.981 3807-3807/six.ca.androidadvanced I/System.out: ViewGroup - intercept: down
     08-04 14:17:45.982 3807-3807/six.ca.androidadvanced I/System.out: view - dispatch: down
     08-04 14:17:45.982 3807-3807/six.ca.androidadvanced I/System.out: view - onTouchEvent: down
     08-04 14:17:45.982 3807-3807/six.ca.androidadvanced I/System.out: ViewGroup - onTouchEvent: down
     08-04 14:17:45.982 3807-3807/six.ca.androidadvanced I/System.out: activity - onTouchEvent: down
     08-04 14:17:46.014 3807-3807/six.ca.androidadvanced I/System.out: activity - dispatch: move
     08-04 14:17:46.014 3807-3807/six.ca.androidadvanced I/System.out: activity - onTouchEvent: move
     08-04 14:17:46.037 3807-3807/six.ca.androidadvanced I/System.out: activity - dispatch: move
     08-04 14:17:46.037 3807-3807/six.ca.androidadvanced I/System.out: activity - onTouchEvent: move
     08-04 14:17:46.047 3807-3807/six.ca.androidadvanced I/System.out: activity - dispatch: move
     08-04 14:17:46.047 3807-3807/six.ca.androidadvanced I/System.out: activity - onTouchEvent: move
     08-04 14:17:46.048 3807-3807/six.ca.androidadvanced I/System.out: activity - dispatch: up
     08-04 14:17:46.048 3807-3807/six.ca.androidadvanced I/System.out: activity - onTouchEvent: up`

In this situation, both of parent and child don’t consume ACTION_DOWN, and lastly, Activity becomes the receiver of the following events, e.g. ACTION_DOWN, ACTION_MOVE and ACTION_UP.

+ Case 2: View handles touch events.(without interception from ViewGroup)
    - View#onTouchEvent return true.
    `08-04 14:38:32.617 3807-3807/six.ca.androidadvanced I/System.out: activity - dispatch: down
     08-04 14:38:32.618 3807-3807/six.ca.androidadvanced I/System.out: ViewGroup - dispatch: down
     08-04 14:38:32.618 3807-3807/six.ca.androidadvanced I/System.out: ViewGroup - intercept: down
     08-04 14:38:32.618 3807-3807/six.ca.androidadvanced I/System.out: view - dispatch: down
     08-04 14:38:32.618 3807-3807/six.ca.androidadvanced I/System.out: view - onTouchEvent: down
     08-04 14:38:32.714 3807-3807/six.ca.androidadvanced I/System.out: activity - dispatch: move
     08-04 14:38:32.715 3807-3807/six.ca.androidadvanced I/System.out: ViewGroup - dispatch: move
     08-04 14:38:32.715 3807-3807/six.ca.androidadvanced I/System.out: ViewGroup - intercept: move
     08-04 14:38:32.715 3807-3807/six.ca.androidadvanced I/System.out: view - dispatch: move
     08-04 14:38:32.715 3807-3807/six.ca.androidadvanced I/System.out: view - onTouchEvent: move
     08-04 14:38:32.757 3807-3807/six.ca.androidadvanced I/System.out: activity - dispatch: up
     08-04 14:38:32.757 3807-3807/six.ca.androidadvanced I/System.out: ViewGroup - dispatch: up
     08-04 14:38:32.757 3807-3807/six.ca.androidadvanced I/System.out: ViewGroup - intercept: up
     08-04 14:38:32.757 3807-3807/six.ca.androidadvanced I/System.out: view - dispatch: up
     08-04 14:38:32.757 3807-3807/six.ca.androidadvanced I/System.out: view - onTouchEvent: up`

    This case is common known. View consumes ACTION_DOWN by returning true from its onTouchEvent, and then other following events will come to its door.
    
    - View#dispatchTouchEvent return true no matter onTouchEvent return true or calling super.onTouchEvent
    
        `08-04 15:29:06.372 11197-11197/six.ca.androidadvanced I/System.out: activity - dispatch: down
         08-04 15:29:06.373 11197-11197/six.ca.androidadvanced I/System.out: ViewGroup - dispatch: down
         08-04 15:29:06.374 11197-11197/six.ca.androidadvanced I/System.out: ViewGroup - intercept: down
         08-04 15:29:06.374 11197-11197/six.ca.androidadvanced I/System.out: view - dispatch: down
         08-04 15:29:06.391 11197-11197/six.ca.androidadvanced I/System.out: activity - dispatch: move
         08-04 15:29:06.392 11197-11197/six.ca.androidadvanced I/System.out: ViewGroup - dispatch: move
         08-04 15:29:06.392 11197-11197/six.ca.androidadvanced I/System.out: ViewGroup - intercept: move
         08-04 15:29:06.392 11197-11197/six.ca.androidadvanced I/System.out: view - dispatch: move
         08-04 15:29:06.462 11197-11197/six.ca.androidadvanced I/System.out: activity - dispatch: up
         08-04 15:29:06.462 11197-11197/six.ca.androidadvanced I/System.out: ViewGroup - dispatch: up
         08-04 15:29:06.462 11197-11197/six.ca.androidadvanced I/System.out: ViewGroup - intercept: up
         08-04 15:29:06.462 11197-11197/six.ca.androidadvanced I/System.out: view - dispatch: up`
    
    Under this situation, View doesn’t pass touch events to its method onTouchEvent() or onTouch() of its onTouchListener.

+ Case 3: ViewGroup handles touch events.
    - ViewGroup#dispatchTouchEvent returns true
    
        `08-04 15:41:41.748 3094-3094/six.ca.androidadvanced I/System.out: activity - dispatch: down
         08-04 15:41:41.748 3094-3094/six.ca.androidadvanced I/System.out: ViewGroup - dispatch: down
         08-04 15:41:41.775 3094-3094/six.ca.androidadvanced I/System.out: activity - dispatch: move
         08-04 15:41:41.775 3094-3094/six.ca.androidadvanced I/System.out: ViewGroup - dispatch: move
         08-04 15:41:41.814 3094-3094/six.ca.androidadvanced I/System.out: activity - dispatch: up
         08-04 15:41:41.815 3094-3094/six.ca.androidadvanced I/System.out: ViewGroup - dispatch: up`
   
    Touch events stop passing to other methods of ViewGroup and other inner views, if ViewGroup#dispatchTouchEvent returns true.

    - ViewGroup#onInterceptTouchEvent returns true and no one consumes touch events
    
        `08-04 14:47:23.416 3807-3807/six.ca.androidadvanced I/System.out: activity - dispatch: down
         08-04 14:47:23.417 3807-3807/six.ca.androidadvanced I/System.out: ViewGroup - dispatch: down
         08-04 14:47:23.419 3807-3807/six.ca.androidadvanced I/System.out: ViewGroup - intercept: down
         08-04 14:47:23.420 3807-3807/six.ca.androidadvanced I/System.out: ViewGroup - onTouchEvent: down
         08-04 14:47:23.420 3807-3807/six.ca.androidadvanced I/System.out: activity - onTouchEvent: down
         08-04 14:47:23.517 3807-3807/six.ca.androidadvanced I/System.out: activity - dispatch: move
         08-04 14:47:23.517 3807-3807/six.ca.androidadvanced I/System.out: activity - onTouchEvent: move
         08-04 14:47:23.517 3807-3807/six.ca.androidadvanced I/System.out: activity - dispatch: up
         08-04 14:47:23.517 3807-3807/six.ca.androidadvanced I/System.out: activity - onTouchEvent: up`
        
        In this case, ViewGroup intercepts ACTION_DOWN by returning true from onInterceptOnTouchEvent(), when pressing down. After that, ViewGroup#onTouchEvent only receives ACTION_DOWN and other events stop going through, just fly to Activity because ViewGroup doesn’t consume touch events.
        
    - Both ViewGroup#onInterceptTouchEvent and ViewGroup#onTouchEvent return true.
        
        `08-04 15:14:58.963 11197-11197/six.ca.androidadvanced I/System.out: activity - dispatch: down
         08-04 15:14:58.964 11197-11197/six.ca.androidadvanced I/System.out: ViewGroup - dispatch: down
         08-04 15:14:58.964 11197-11197/six.ca.androidadvanced I/System.out: ViewGroup - intercept: down
         08-04 15:14:58.964 11197-11197/six.ca.androidadvanced I/System.out: ViewGroup - onTouchEvent: down
         08-04 15:14:59.049 11197-11197/six.ca.androidadvanced I/System.out: activity - dispatch: move
         08-04 15:14:59.049 11197-11197/six.ca.androidadvanced I/System.out: ViewGroup - dispatch: move
         08-04 15:14:59.049 11197-11197/six.ca.androidadvanced I/System.out: ViewGroup - onTouchEvent: move
         08-04 15:14:59.050 11197-11197/six.ca.androidadvanced I/System.out: activity - dispatch: up
         08-04 15:14:59.050 11197-11197/six.ca.androidadvanced I/System.out: ViewGroup - dispatch: up
         08-04 15:14:59.050 11197-11197/six.ca.androidadvanced I/System.out: ViewGroup - onTouchEvent: up`
    
        Apparently, in this case, children views cannot receive any touch events and ViewGroup became the receiver of all touch events. onInterceptTouchEvent() only gets ACTION_DOWN, while dispatchTouchEvent() and onTouchEvent are able to receive three types of touch events: ACTION_DOWN, ACTION_MOVE and ACTION_UP.
        
        
+ Case 4: Activity handles touch events.

    Activity#dispatchTouchEvent() returns true and no matter onTouchEvent return true or calling super.onTouchEvent.

    `08-11 13:43:47.314 3028-3028/six.ca.androidadvanced I/System.out: activity - dispatch: down
     08-11 13:43:47.347 3028-3028/six.ca.androidadvanced I/System.out: activity - dispatch: move
     08-11 13:43:47.412 3028-3028/six.ca.androidadvanced I/System.out: activity - dispatch: up`

    As for this scenario, Activity takes care of all touch events and ViewGroup and View beneath it will not receive any touch events.


### Conclusion
After going through several cases regarding touch event propagation, I drew a flowchart which includes those cases above, hoping this will set a light to others who feel confused when it comes to touch event propagation.

![](/imgs/20170812_touch_events_purple.png)
