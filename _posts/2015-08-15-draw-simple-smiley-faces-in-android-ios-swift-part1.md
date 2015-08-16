---
layout:     post
title:      Creating simple smiley face in Android and IOS.
date:       2015-08-15 20:38:19
author:     Franklin
summary:    An article about how to draw a simple smiley face on Android and IOS.
categories: draw smiley
#thumbnail:  heart
tags:
 - Creating smiley
 - Android
 - IOS swift
 - git
---

In this article i am going to show you how to draw a simple smiley face in Android using [Canvas][1] and in IOS using [UIBezierPath][2].

## Android

After following this article you should be able to create something like

![desk](http://i.imgur.com/Dbxr4ss.png)
![desk](http://i.imgur.com/0u5H5JU.png)

_<ins>Step 1 - Planing the smiley</ins>_

1. Draw a circle with maximum radius, it will act as face.
2. Draw two small circles inside the face circle, it will act as eyes
3. Draw an arc under the eyes, it will act as the mouth.
4. Make sure, it works even if the orientation changes (both landscape and portrait modes.)

_<ins>Step 2 - Create Required files.</ins>_

1. Create Main Activity (Generate both xml and java)

2. Create FaceView class and extend from View.

_<ins>Step 3 - Initializing the FaceView.</ins>_

1. Create a FaceView constructor and initialize Paint object.

2. Setup default values like color, stroke values, radius

_<ins>Step 4 - Override onDraw Function.</ins>_

1. Set color property to paint object
2. Set stroke width to the paint object
3. Set drawing style as stroke

{% highlight java %}
mPaint.setColor(Color.parseColor(COLOR_HEX));
mPaint.setStrokeWidth(strokeWidth);
mPaint.setStyle(Paint.Style.STROKE);
{% endhighlight %}

_<ins>Step 5 - Draw Face circle.</ins>_

1. Get the total width, height taken by our FaceView using getMeasuredWidth and getMeasuredHeight function.
2. DrawCircle function requires (x, y) points , radius and a paint object.
3. Calculate the center x position by getMeasuredWidth / 2
4. Calculate the center y position by getMeasuredHeight / 2
5. Calculate the radius from the minimum of y position or x position to support both landscape and portrait modes.
6. Call canvas.drawCircle function with the x position, y position, radius and paint object.

{% highlight java %}
xPosition = getMeasuredWidth() / 2;
yPosition = getMeasuredHeight() / 2;
radius = xPosition < yPosition ? xPosition : yPosition ;
radius -= defaultMargin;
canvas.drawCircle(xPosition, yPosition, radius, mPaint);
{% endhighlight %}

_<ins>Step 5 - Draw eyes.</ins>_

* Find the eyeYposition. eyeYPosition will remail same for both left and right. The eyeYposition should be a little more than the calculated yPosition of the face. The Face's yPosition is half the height, but eyes should be placed a little above. so i use the following formula to calculate the eyeYPosition

{% highlight java %}
eyeYPosition = (float) (yPosition / 1.2);
{% endhighlight %}

* Find the leftEyeXPosition. xPosition is half the width, so the left eye position should take half the xPosition. i use the following formula to calculate leftEyeXPosition to make sure it works even the orientation changes.

{% highlight java %}
leftEyeXPosition = xPosition < yPosition ? xPosition / 2 : (float) (xPosition / 1.3);
{% endhighlight %}

* Find the rightEyeXPosition. xPosition is half the width, so the right eye position should take little more than the xPosition. i use the following formula to calculate rightEyeXPosition to make sure it works even the orientation changes.

{% highlight java %}
rightEyeXPosition = xPosition < yPosition ? xPosition + xPosition / 2 : xPosition + xPosition / 4;
{% endhighlight %}

* We have calculated the eye positions, now we can draw left, right eyes using drawCircle function.

{% highlight java %}
canvas.drawCircle(leftEyeXPosition, eyeYPosition, eyeRadius, mPaint);
canvas.drawCircle(rightEyeXPosition, eyeYPosition, eyeRadius, mPaint);
{% endhighlight %}

_<ins>Step 5 - Draw Mouth.</ins>_

Drawing Mouth is little tricky.

1 Mark the boundaries using RectF

  * Find the top left -  (leftEyeXPosition, yPosition + some value)
  * Find the bottom right - (rightEyeXPosition, yPosition + some value)

    {% highlight java %}
    RectF oval = new RectF(leftEyeXPosition, yPosition + yPosition / 8, rightEyeXPosition, (float) (yPosition + yPosition / 2.5)); // left top right bottom
    {% endhighlight %}

2 Draw an arc within the boundaries.

  * For happy face start from 0 degree and draw another 180 degree (sweep angle).

  {% highlight java %}
  canvas.drawArc(oval, 10, 150, false, mPaint);
  {% endhighlight %}

  * For sad face start from 180 degree and draw another 180 degree (sweep angle).

  {% highlight java %}
  canvas.drawArc(oval, 200, 140, false, mPaint);
  {% endhighlight %}



_<ins>The final code of FaceView.</ins>_

{% highlight java %}
public class FaceView extends View {

    private static final String COLOR_HEX = "#0000FF"; // RRGGBB - BLUE
    private final Paint mPaint;
    private float xPosition;
    private float yPosition;
    private float radius;
    private float strokeWidth = 10;
    private float defaultScale = 0.90f;
    private float eyeRadius = 60;
    private float eyeYPosition;
    private float leftEyeXPosition;
    private float rightEyeXPosition;

    public FaceView(Context context, AttributeSet attrs) {
        super(context, attrs);
        mPaint = new Paint();
        mPaint.setAntiAlias(true);
    }

    @Override
    protected void onDraw(Canvas canvas) {

        super.onDraw(canvas);
        mPaint.setColor(Color.parseColor(COLOR_HEX));
        mPaint.setStrokeWidth(strokeWidth);
        mPaint.setStyle(Paint.Style.STROKE);
        canvas.drawPaint(mPaint);

        // drawing outer circle
        // lets setup x cord, y cord, radius
        // x, y position should point to center.
        // radius should be half the width / height
        xPosition = getMeasuredWidth() / 2;
        yPosition = getMeasuredHeight() / 2;
        radius = xPosition < yPosition ? xPosition : yPosition ;
        radius * = defaultScale;
        canvas.drawCircle(xPosition, yPosition, radius, mPaint);

        // Drawing Eyes.

        // lets find eye y position
        eyeYPosition = (float) (yPosition / 1.2);

        // lets find eye x position
        leftEyeXPosition = xPosition < yPosition ? xPosition / 2 : (float) (xPosition / 1.3);

        // lets find right eye x position
        rightEyeXPosition = xPosition < yPosition ? xPosition + xPosition / 2 : xPosition + xPosition / 4;

        // left eye
        canvas.drawCircle(leftEyeXPosition, eyeYPosition, eyeRadius, mPaint);

        // right eye
        canvas.drawCircle(rightEyeXPosition, eyeYPosition, eyeRadius, mPaint);

        // lets draw mouth.
        RectF oval = new RectF(leftEyeXPosition, yPosition + yPosition / 8, rightEyeXPosition, (float) (yPosition + yPosition / 2.5)); // left top right bottom

//        canvas.drawArc(oval, 200, 140, false, mPaint); // sad face.
        canvas.drawArc(oval, 10, 150, false, mPaint); // happy face.
    }
}
{% endhighlight %}

_<ins>Step 3</ins>_
Add the FaceView in MainActivity's xml file.

{% highlight xml %}
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools" android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity">

    <com.laminin.happiness.FaceView
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

</LinearLayout>
{% endhighlight %}

## IOS - swift

Now lets see how to do the same in IOS, I am going to use swift. The final output will be something like this.

![desk](http://i.imgur.com/xaFGAb6.png)
![desk](http://i.imgur.com/XMyzHtC.png)


_<ins>Step 1 - Planing the smiley</ins>_

1. Create the required files (Stroyboard, ViewController, FaceView)
2. Set up the storyboard and view controller to draw FaceView
3. Draw a circle with maximum radius, it will act as face.
4. Draw two small circles inside the face circle, it will act as eyes
5. Draw an arc under the eyes, it will act as the mouth.
6. Make sure, it works even if the orientation changes (both landscape and portrait modes.)

_<ins>Step 2 - Creating required files</ins>_

1. Open storyboard - navigate to object library,  drag a View into the storyboard view controller.
2. Stretch the view to fit on the view controller.
3. Select the view and click the Reset to suggested constrains from the resolve auto layout button.
4. Create FaceView as a subclass of CocoaTouch UIView class.
5. Select the view from storyboard by shift-ctrl-click on the storyboard.
6. Click identity inspector and change the class to FaceView

_<ins>Step 3 - Draw the Face</ins>_

* Setting up computed property to get the face center.

{% highlight swift %}
var faceCenter: CGPoint {
    // we want the center of the face view, not the super view.
    return convertPoint(center, fromView: superview)
}
{% endhighlight %}

* Setting up computed property for face radius. Face should take as maximum space as possible.

{% highlight swift %}
var faceRadius: CGFloat {
    // return the minimum value of width and height.
    return min(bounds.size.width, bounds.size.height) / 2 // converting diameter into radius.
}
{% endhighlight %}

* Create a facePath inside drawRect function using faceCenter, faceRadius.

{% highlight swift %}
let facePath = UIBezierPath(arcCenter: faceCenter, radius: faceRadius, startAngle: 0, endAngle: CGFloat(2 * M_PI), clockwise: true)
{% endhighlight %}

* Setting up line width using property observer

{% highlight swift %}
    var linewidth: CGFloat = 3 { didSet { setNeedsDisplay() } } // every time some one change the value we need to re draw .
{% endhighlight %}

* Setting up color using property absorber.

{% highlight swift %}
    var color : UIColor = UIColor.blueColor() { didSet { setNeedsDisplay() } } // when somebody changes the color we have to redraw.
{% endhighlight %}


* Set the color inside your drawRect function

{% highlight swift %}
    color.set()
{% endhighlight %}

* Draw the face. using facePath.stroke()

{% highlight swift %}
    facePath.stroke()
{% endhighlight %}

* The drawRect function looks something like

{% highlight swift %}
override func drawRect(rect: CGRect) {
    let facePath = UIBezierPath(arcCenter: faceCenter, radius: faceRadius, startAngle: 0, endAngle: CGFloat(2 * M_PI), clockwise: true)
    facePath.lineWidth = lineWidth
    color.set()
    facePath.stroke()
}
{% endhighlight %}


_<ins>Step 4 - Some more changes</ins>_

1 Redraw when the bounds changes. - To avoid the stretching while changing the orientation.

Click the view, select the Attribute inspector, change the mode to Redraw.

2 Reduce the radius to 90% of the width / height

{% highlight swift %}
// setup property obsover to take 90% scale
  var scale : CGFloat = 0.90 { didSet { setNeedsDisplay() } } // redraw when a change occure.
{% endhighlight %}


3 Multiply 0.90 with the computed radius value as follows.

{% highlight swift %}
var faceRadius: CGFloat {
    // return the minimum value of width and height.
    return min(bounds.size.width, bounds.size.height) / 2  * scale // converting diameter into radius.
}
{% endhighlight %}

_<ins>Step 5 - Drawing eyes.</ins>_

* To create eye we need some constants. we put them inside a struct.

{% highlight swift %}
private struct Scaling {
    static let FaceRadiusToEyeRadiousRatio: CGFloat = 10
    static let FaceRadiusToEyeOffsetRatio: CGFloat = 3
    static let FaceRadiusToEyeSeparationRatio: CGFloat = 1.5
    static let FaceRadiusToMouthWidthRatio: CGFloat = 1
    static let FaceRadiusToMouthHeightRatio: CGFloat = 3
    static let FaceRadiusToMouthOffsetRatio: CGFloat = 3
}
{% endhighlight %}


*  To draw eye.
{% highlight swift %}
  let eyeRadius = faceRadius / Scaling.FaceRadiusToEyeRadiousRatio
  let eyeVerticalOffset = faceRadius / Scaling.FaceRadiusToEyeOffsetRatio
  let eyeHorizontalSeparation = faceRadius / Scaling.FaceRadiusToEyeSeparationRatio
  var eyeCenter = faceCenter
  eyeCenter.y -= eyeVerticalOffset
{% endhighlight %}

* Drawing left eye
{% highlight swift %}
  eyeCenter.x -= eyeHorizontalSeparation / 2
  let path = UIBezierPath(arcCenter: eyeCenter, radius: eyeRadius, startAngle: 0, endAngle: CGFloat(2 * M_PI), clockwise: true)
  path.lineWidth = lineWidth
  path.stroke()
{% endhighlight %}

* Drawing right eye
{% highlight swift %}
  eyeCenter.x += eyeHorizontalSeparation / 2
  let path = UIBezierPath(arcCenter: eyeCenter, radius: eyeRadius, startAngle: 0, endAngle: CGFloat(2 * M_PI), clockwise: true)
  path.lineWidth = lineWidth
  path.stroke()
{% endhighlight %}


_<ins>Step 5 - Drawing Mouth.</ins>_
{% highlight swift %}
  let mouthWidth = faceRadius / Scaling.FaceRadiusToMouthWidthRatio
  let mouthHeight = faceRadius / Scaling.FaceRadiusToMouthHeightRatio
  let mouthVerticalOffset = faceRadius / Scaling.FaceRadiusToMouthOffsetRatio

  let smileHeight = CGFloat(max(min(fractionOfMaximumSmile, 1), -1)) * mouthHeight

  let start = CGPoint(x: faceCenter.x - mouthWidth / 2, y: faceCenter.y + mouthVerticalOffset)
  let end = CGPoint(x: start.x + mouthWidth, y: start.y)
  let cp1 = CGPoint(x: start.x + mouthWidth / 3, y: start.y + smileHeight)
  let cp2 = CGPoint(x: end.x - mouthWidth / 3 , y: cp1.y)

  let path = UIBezierPath()
  path.moveToPoint(start)
  path.addCurveToPoint(end, controlPoint1: cp1, controlPoint2: cp2)
  path.lineWidth = lineWidth
  path.stroke()
{% endhighlight %}


_<ins>Step 5 - Final FaceView class.</ins>_

The final FaceView class looks something like this.

{% highlight swift %}

import UIKit

@IBDesignable
class FaceView: UIView {

    @IBInspectable
    var lineWidth: CGFloat = 3 { didSet { setNeedsDisplay() } }
    @IBInspectable
    var color =  UIColor.blueColor() { didSet { setNeedsDisplay() } }
    @IBInspectable
    var scale: CGFloat = 0.90 { didSet { setNeedsDisplay() } }

    private struct Scaling {
        static let FaceRadiusToEyeRadiousRatio: CGFloat = 10
        static let FaceRadiusToEyeOffsetRatio: CGFloat = 3
        static let FaceRadiusToEyeSeparationRatio: CGFloat = 1.5
        static let FaceRadiusToMouthWidthRatio: CGFloat = 1
        static let FaceRadiusToMouthHeightRatio: CGFloat = 3
        static let FaceRadiusToMouthOffsetRatio: CGFloat = 3
    }

    private enum Eye { case Left, Right }

    var faceCenter: CGPoint {
        return convertPoint(center, fromView: superview)
    }

    var faceRadius: CGFloat {
        return min(bounds.size.width, bounds.size.height) / 2 * scale
    }

    private func bezierPathForEye(whichEye: Eye) -> UIBezierPath {
        let eyeRadius = faceRadius / Scaling.FaceRadiusToEyeRadiousRatio
        let eyeVerticalOffset = faceRadius / Scaling.FaceRadiusToEyeOffsetRatio
        let eyeHorizontalSeparation = faceRadius / Scaling.FaceRadiusToEyeSeparationRatio

        var eyeCenter = faceCenter
        eyeCenter.y -= eyeVerticalOffset
        switch whichEye{
        case .Left: eyeCenter.x -= eyeHorizontalSeparation / 2
        case .Right: eyeCenter.x += eyeHorizontalSeparation / 2
        }

        let path = UIBezierPath(arcCenter: eyeCenter, radius: eyeRadius, startAngle: 0, endAngle: CGFloat(2 * M_PI), clockwise: true)
        path.lineWidth = lineWidth

        return path
    }

    private func bezierPathForSmile(fractionOfMaximumSmile: Double) -> UIBezierPath {
        let mouthWidth = faceRadius / Scaling.FaceRadiusToMouthWidthRatio
        let mouthHeight = faceRadius / Scaling.FaceRadiusToMouthHeightRatio
        let mouthVerticalOffset = faceRadius / Scaling.FaceRadiusToMouthOffsetRatio

        let smileHeight = CGFloat(max(min(fractionOfMaximumSmile, 1), -1)) * mouthHeight

        let start = CGPoint(x: faceCenter.x - mouthWidth / 2, y: faceCenter.y + mouthVerticalOffset)
        let end = CGPoint(x: start.x + mouthWidth, y: start.y)
        let cp1 = CGPoint(x: start.x + mouthWidth / 3, y: start.y + smileHeight)
        let cp2 = CGPoint(x: end.x - mouthWidth / 3 , y: cp1.y)

        let path = UIBezierPath()
        path.moveToPoint(start)
        path.addCurveToPoint(end, controlPoint1: cp1, controlPoint2: cp2)
        path.lineWidth = lineWidth

        return path
    }

    override func drawRect(rect: CGRect) {
        let facePath = UIBezierPath(arcCenter: faceCenter, radius: faceRadius, startAngle: 0, endAngle: CGFloat(2 * M_PI), clockwise: true)
        facePath.lineWidth = lineWidth
        color.set()
        facePath.stroke()

        bezierPathForEye(Eye.Left).stroke()
        bezierPathForEye(Eye.Right).stroke()

        let smileness = 0.75 // range starts form -1 to 1, positive values for happy, negative for sand
        let smilePath = bezierPathForSmile(smileness)
        smilePath.stroke()
    }
}

{% endhighlight %}

__Get the source code__

You can clone this project form [github](https://github.com/Franklin2412/smiley-part1)

or clone the project using following command.

{% highlight sh %}
  git clone git@github.com:Franklin2412/smiley-part1.git
{% endhighlight %}


[1]:http://developer.android.com/intl/ko/training/custom-views/custom-drawing.html
[2]:https://developer.apple.com/library/ios/documentation/2DDrawing/Conceptual/DrawingPrintingiOS/BezierPaths/BezierPaths.html
