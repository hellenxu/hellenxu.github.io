SVG Animation
=========
VectorDrawable, which is introduced in API 21 and above, helps creating a drawable based on an XML vector graphic.
Before Android Studio 1.5.1, we need other svg editor to transform a svg format picture into an XML file. Luckily,
with 1.5.1 and above, we have Vector Asset Studio to import a scalable vector graphic. Now, let's common elements we
may use in VectorDrawable.

Part One: Common elements in Vector Asset
--------
```xml
<vector xmlns:android="http://schemas.android.com/apk/res/android"
        android:width="200dp"
        android:height="200dp"
        android:viewportWidth="100.0"
        android:viewportHeight="100.0">
    <group>
        <path android:name="line1"
            android:strokeColor="@android:color/holo_blue_dark"
            android:strokeWidth="5"
            android:strokeLineCap="round"
            android:pathData="M 20,80 L 50,80 80,80"/>

        <path android:name="line2"
            android:strokeColor="@android:color/holo_blue_dark"
            android:strokeWidth="5"
            android:strokeLineCap="round"
            android:pathData="M 20,20 L 50,20 80,20"/>
    </group>
</vector>
```

+ <vector>
> __name__: defines the name of this vector drawable
> __width__: used to define the intrinsic width of the drawable.
> __height__: used to define the intrinsic height of the drawable.
> __viewportWidth__: used to define the width of the viewport space.
> __viewport__: used to define the height of the viewport space.
> __tint__: the Porter_Duff blending mode for the tint color. The default value is src_in.
> __autoMirrored__: indicates if the drawable needs to be mirrored when its layout direction is right to left.
> __alpha__: the opacity of this drawable.

+ <group>
> name: Defines the name of the group.
> rotation: The degrees of rotation of the group.
> pivotX: The X coordinate of the pivot for the scale and rotation of the group. This is defined in the viewport space.
> pivotY: The Y coordinate of the pivot for the scale and rotation of the group. This is defined in the viewport space.
> scaleX: The amount of scale on the X Coordinate.
> scaleY: The amount of scale on the Y coordinate.
> translateX: The amount of translation on the X coordinate. This is defined in the viewport space.
> translateY: The amount of translation on the Y coordinate. This is defined in the viewport space.

+ <path>
> name: Defines the name of the path.
> pathData: Defines path data using exactly same format as "d" attribute in the SVG's path data. This is defined in the viewport space.
> fillColor: Defines the color to fill the path (none if not present).
> strokeColor: Defines the color to draw the path outline (none if not present).
> strokeWidth: The width a path stroke.
> strokeAlpha: The opacity of a path stroke.
> fillAlpha: The opacity to fill the path with.
> trimPathStart: The fraction of the path to trim from the start, in the range from 0 to 1.
> trimPathEnd: The fraction of the path to trim from the end, in the range from 0 to 1.
> trimPathOffset: Shift trim region (allows showed region to include the start and end), in the range from 0 to 1.
> strokeLineCap: Sets the linecap for a stroked path: butt, round, square.
> strokeLineJoin: Sets the lineJoin for a stroked path: miter,round,bevel.
> strokeMiterLimit: Sets the Miter limit for a stroked path.

Part Two: Common SVG Commands
--------
Click (here)[https://www.w3.org/TR/SVG11/paths.html#PathData] to see common svg commands.

Part Three: SVG animation sample
--------

