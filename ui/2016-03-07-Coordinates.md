Two coordinates in Android
==============
Usually we use two different coordinates during developing apps, they are:
> Screen Coordinates

> View Coordinates

Part One: Screen Coordinates
-------

![](/imgs/20160307_coordinates_01.png)

+ __Common Position:__

1) (0, 0) is on the top left corner;

2) (max_X, 0) is on the top right corner;

3) (0, max_Y) is on the bottom left corner;

4) (max_X, max_Y) is on the bottom right corner.

+ __Methods to get coordinates:__

1) View.getLocationOnScreen(int[] location);

2) MotionEvent.getRawX && MotionEvent.getRawY.


Part Two: View Coordinates
-------
View coordinate describes the position of a child view within a parent view, and the origin coordinate is no longer the top left corner of a phone screen. Instead, it is the top left corner of a parent view.

![](/imgs/20160307_coordinates_02.png)


Part Three: tips to distinguish methods in these two coordinates.
-------

![](/imgs/20160307_coordinates_03.png)

