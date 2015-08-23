---
layout:     post
title:      Creating simple smiley face in Android and IOS - Part 2.
date:       2015-08-22 21:38:19
author:     Franklin
summary:    An article about how to draw a simple smiley face on Android and IOS with touch events.
categories: draw smiley
tags:
 - Creating smiley
 - Android
 - IOS swift
 - Pan Pinch events
---

Here we add more feature to our [Smiley][0]. If you have not read [part1][0], it is recommended to read [part1][0] first.

Here i am going to show you how to detect gestures (Pan and Pinch) on the smiley face in Android using [gestures][1] and in IOS using [UIPanGestureRecognizer][2] and [UIPinchGestureRecognizer][3].

## Android

### Detecting Pan - Drag  gesture

Before applying gesture detector we need to make some changes to our smiley. Our current smiley's size of eye and mouth are fixed. We need to apply some ratio to eye and mouth with face radius, so any change in face radius will map the size of eye and mouth.

__The following are the ratio to map eye, mouth with face radius.__

{% highlight java %}
private static final float FACE_RADIUS_TO_EYE_RADIUS_RATIO = 10;
private static final float FACE_RADIUS_TO_EYE_OFFSET_RATIO = 3;
private static final float FACE_RADIUS_TO_EYE_SEPARATION_RATIO = 2.5f;

private static final float FACE_RADIUS_TO_MOUTH_WIDTH_RATIO = 1;
private static final float FACE_RADIUS_TO_MOUTH_HEIGHT_RATIO = 2;
private static final float FACE_RADIUS_TO_MOUTH_X_OFFSET_RATIO = 2;
private static final float FACE_RADIUS_TO_MOUTH_Y_OFFSET_RATIO = 10;
{% endhighlight %}

__After applying the ratio, onDraw function will look something like this.__

{% highlight java %}
faceRadius = Math.min(xPosition, yPosition) * defaultScale;

canvas.drawCircle(xPosition, yPosition, faceRadius, mPaint);

// lets draw eyes

// lets set the eye radius:
eyeRadius = faceRadius / FACE_RADIUS_TO_EYE_RADIUS_RATIO;

// lets find eye y position
eyeYPosition = yPosition - (faceRadius / FACE_RADIUS_TO_EYE_OFFSET_RATIO);

// lets find eye x position
leftEyeXPosition = xPosition + (faceRadius / FACE_RADIUS_TO_EYE_SEPARATION_RATIO);

// lets find right eye x position
rightEyeXPosition = xPosition - (faceRadius / FACE_RADIUS_TO_EYE_SEPARATION_RATIO);

// left eye
canvas.drawCircle(leftEyeXPosition, eyeYPosition, eyeRadius, mPaint);

// right eye
canvas.drawCircle(rightEyeXPosition, eyeYPosition, eyeRadius, mPaint);

// lets draw mouth.
mouthWidth = faceRadius / FACE_RADIUS_TO_MOUTH_WIDTH_RATIO;
mouthMaximumHeight = faceRadius / FACE_RADIUS_TO_MOUTH_HEIGHT_RATIO;

if (mouthHeight == 0.0f) { // initial
    mouthHeight = mouthMaximumHeight;
}

mouthRectLeft = xPosition - (faceRadius / FACE_RADIUS_TO_MOUTH_X_OFFSET_RATIO);
mouthRectTop = yPosition + (faceRadius / FACE_RADIUS_TO_MOUTH_Y_OFFSET_RATIO);
mouthRectRight = mouthRectLeft + mouthWidth;
mouthRectBottom = mouthRectTop + Math.abs(mouthHeight);

RectF ovalRect = new RectF(mouthRectLeft, mouthRectTop, mouthRectRight, mouthRectBottom); // left top right bottom

if (mouthHeight < 0) { // bottom - up - happy
    canvas.drawArc(ovalRect, 0, 180, false, mPaint);
} else if (mouthHeight > 0) { // up - bottom - sad
    canvas.drawArc(ovalRect, 180, 180, false, mPaint);
}
{% endhighlight %}


__Pan/drag gesture detector__

1. Override onTouchEvent(MotionEvent ev) function to capture the events.
2. Under ActionDown event store the pointerIndex, pointerId, x position, y position.
  {% highlight java %}
  pointerIndex = MotionEventCompat.getActionIndex(event);
  x = MotionEventCompat.getX(event, pointerIndex);
  y = MotionEventCompat.getY(event, pointerIndex);

  // Remember where we started (for dragging)
  mLastTouchX = x;
  mLastTouchY = y;

  // Save the ID of this pointer (for dragging)
  mActivePointerId = MotionEventCompat.getPointerId(event, 0);
  {% endhighlight %}
3. Under ActionMove event get the xPosition, yPosition.
4. Calculate the distance moved from the lastTouchPostion
5. If the touch event occur inside face bound change the mouth height and then redraw.
6. Call invalidate function to redraw.

{% highlight java %}
x = MotionEventCompat.getX(event, pointerIndex);
y = MotionEventCompat.getY(event, pointerIndex);

// Calculate the distance moved
final float dx = x - mLastTouchX;
final float dy = y - mLastTouchY;

// here we need to set the height of the mouth.
// lets find y boundary to detect this motion.

if (y > yPosition - faceRadius && y < yPosition + faceRadius) {
    mouthHeight = mouthMaximumHeight * (dy / 1000);
    invalidate();
}
{% endhighlight %}

__Pinch gesture detector__

Pinch gesture is detected using [multi touch][4] listeners. We use pinch gesture detector to expand or shrink the face view.

+ Identify the multi touch events using pointer count
{% highlight java %}
boolean singleTouch = event.getPointerCount() > 1 ? false : true;
{% endhighlight %}

+ Under action down event calculate the pointerIndex, activePointerId, xPosition, yPosition of primary touch
{% highlight java %}
pointerIndex = MotionEventCompat.getActionIndex(event);
mActivePointerId = MotionEventCompat.getPointerId(event, 0);
x = MotionEventCompat.getX(event, pointerIndex);
y = MotionEventCompat.getY(event, pointerIndex);
{% endhighlight %}

+ Calculate the pointerIndex, activePointerId, xPosition, yPosition of secondary touch
{% highlight java %}
secondaryPointerId = MotionEventCompat.getPointerId(event, 1); // is the secondary finger pointer id.
secondaryPointerIndex = MotionEventCompat.findPointerIndex(event, secondaryPointerId);
secondaryX = MotionEventCompat.getX(event, secondaryPointerIndex);
secondaryY = MotionEventCompat.getY(event, secondaryPointerIndex);
{% endhighlight %}

+ Calculate the distance between primary and secondary touch positions.
{% highlight java %}
initialDistance = (float) Math.sqrt(((secondaryX - x) * (secondaryX - x)) + ((secondaryY - y) * (secondaryY - y)));
{% endhighlight %}

+ Repeat the same steps under Action Move.
{% highlight java %}
mActivePointerId = MotionEventCompat.getPointerId(event, 0);
pointerIndex = MotionEventCompat.findPointerIndex(event, mActivePointerId);
x = MotionEventCompat.getX(event, pointerIndex);
y = MotionEventCompat.getY(event, pointerIndex);

secondaryPointerId = MotionEventCompat.getPointerId(event, 1);
secondaryPointerIndex = MotionEventCompat.findPointerIndex(event, secondaryPointerId);
secondaryX = MotionEventCompat.getX(event, secondaryPointerIndex);
secondaryY = MotionEventCompat.getY(event, secondaryPointerIndex);

if (initialDistance == 0) {
    initialDistance = (float) Math.sqrt(((secondaryX - x) * (secondaryX - x)) + ((secondaryY - y) * (secondaryY - y)));
}
{% endhighlight %}

+ Calculate the current distance
{% highlight java %}
distance = (float) Math.sqrt(((secondaryX - x) * (secondaryX - x)) + ((secondaryY - y) * (secondaryY - y)));
currentDistance = Math.abs(initialDistance - distance);
{% endhighlight %}

+ Calculate the face radius and redraw the face.
{% highlight java %}
faceRadius = Math.max(minimumFaceRadius, Math.min(maximumFaceRadius, currentDistance));
invalidate();
{% endhighlight %}

__Final FaceView class should look something like this.__
{% highlight java %}
public class FaceView extends View {

    private static final String COLOR_HEX = "#0000FF"; // RRGGBB
    private final Paint mPaint;
    private float xPosition;
    private float yPosition;
    private float faceRadius;
    private static final float strokeWidth = 4;
    private float defaultScale = 0.90f;

    private float eyeRadius = 30; // default value.
    private float eyeYPosition;
    private float leftEyeXPosition;
    private float rightEyeXPosition;


    private static final float FACE_RADIUS_TO_EYE_RADIUS_RATIO = 10;
    private static final float FACE_RADIUS_TO_EYE_OFFSET_RATIO = 3;
    private static final float FACE_RADIUS_TO_EYE_SEPARATION_RATIO = 2.5f;

    private static final float FACE_RADIUS_TO_MOUTH_WIDTH_RATIO = 1;
    private static final float FACE_RADIUS_TO_MOUTH_HEIGHT_RATIO = 2;
    private static final float FACE_RADIUS_TO_MOUTH_X_OFFSET_RATIO = 2;
    private static final float FACE_RADIUS_TO_MOUTH_Y_OFFSET_RATIO = 10;

    private Float mouthWidth;
    private Float mouthHeight = 0.0f;
    private Float mouthMaximumHeight;

    private Float mouthRectLeft;
    private Float mouthRectTop;
    private Float mouthRectRight;
    private Float mouthRectBottom;

    float mLastTouchX = 0;
    float mLastTouchY = 0;
    int mActivePointerId = 0;

    float maximumFaceRadius = 0;
    float minimumFaceRadius = 20;

    float initialDistance = 0;
    float currentDistance = 0;

    int index;
    int action;
    int pointerId;

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


        if (maximumFaceRadius == 0) {
            faceRadius = Math.min(xPosition, yPosition) * defaultScale;
            maximumFaceRadius = faceRadius;
        }

        canvas.drawCircle(xPosition, yPosition, faceRadius, mPaint);

        // lets draw eyes

        // lets set the eye radius:
        eyeRadius = faceRadius / FACE_RADIUS_TO_EYE_RADIUS_RATIO;

        // lets find eye y position
        eyeYPosition = yPosition - (faceRadius / FACE_RADIUS_TO_EYE_OFFSET_RATIO);

        // lets find eye x position
        leftEyeXPosition = xPosition + (faceRadius / FACE_RADIUS_TO_EYE_SEPARATION_RATIO);

        // lets find right eye x position
        rightEyeXPosition = xPosition - (faceRadius / FACE_RADIUS_TO_EYE_SEPARATION_RATIO);
        // left eye
        canvas.drawCircle(leftEyeXPosition, eyeYPosition, eyeRadius, mPaint);

        // right eye
        canvas.drawCircle(rightEyeXPosition, eyeYPosition, eyeRadius, mPaint);

        // lets draw mouth.

        mouthWidth = faceRadius / FACE_RADIUS_TO_MOUTH_WIDTH_RATIO;
        mouthMaximumHeight = faceRadius / FACE_RADIUS_TO_MOUTH_HEIGHT_RATIO;

        if (mouthHeight == 0.0f) { // initial
            mouthHeight = mouthMaximumHeight;
        }

        mouthRectLeft = xPosition - (faceRadius / FACE_RADIUS_TO_MOUTH_X_OFFSET_RATIO);
        mouthRectTop = yPosition + (faceRadius / FACE_RADIUS_TO_MOUTH_Y_OFFSET_RATIO);
        mouthRectRight = mouthRectLeft + mouthWidth;
        mouthRectBottom = mouthRectTop + Math.abs(mouthHeight);

        RectF ovalRect = new RectF(mouthRectLeft, mouthRectTop, mouthRectRight, mouthRectBottom); // left top right bottom


        if (mouthHeight < 0) { // bottom - up - happy
            canvas.drawArc(ovalRect, 0, 180, false, mPaint);
        } else if (mouthHeight > 0) { // up - bottom - sad
            canvas.drawArc(ovalRect, 180, 180, false, mPaint);
        }
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {

        boolean singleTouch = event.getPointerCount() > 1 ? false : true;

        index = event.getActionIndex();
        action = event.getActionMasked();
        pointerId = event.getPointerId(index);


        final int pointerIndex;
        final float x;
        final float y;


        int secondaryPointerId = 0;
        final int secondaryPointerIndex;
        final float secondaryX;
        final float secondaryY;

        float distance;

        switch (action) {
            case MotionEvent.ACTION_DOWN:

                if (singleTouch) { // for pan/drag event.

                    pointerIndex = MotionEventCompat.getActionIndex(event);
                    x = MotionEventCompat.getX(event, pointerIndex);
                    y = MotionEventCompat.getY(event, pointerIndex);

                    // Remember where we started (for dragging)
                    mLastTouchX = x;
                    mLastTouchY = y;
                    // Save the ID of this pointer (for dragging)
                    mActivePointerId = MotionEventCompat.getPointerId(event, 0);

                } else { // for pinch  event.
                    // mActivePointerId is the primary finger pointer id.
                    pointerIndex = MotionEventCompat.getActionIndex(event);
                    mActivePointerId = MotionEventCompat.getPointerId(event, 0);
                    x = MotionEventCompat.getX(event, pointerIndex);
                    y = MotionEventCompat.getY(event, pointerIndex);

                    secondaryPointerId = MotionEventCompat.getPointerId(event, 1); // is the secondary finger pointer id.
                    secondaryPointerIndex = MotionEventCompat.findPointerIndex(event, secondaryPointerId);
                    secondaryX = MotionEventCompat.getX(event, secondaryPointerIndex);
                    secondaryY = MotionEventCompat.getY(event, secondaryPointerIndex);

                    // distance between primary and secondary fingers
                    // squareroot(square(x2-x1) + square(y2-y1))

                    initialDistance = (float) Math.sqrt(((secondaryX - x) * (secondaryX - x)) + ((secondaryY - y) * (secondaryY - y)));

                }
                break;
            case MotionEvent.ACTION_MOVE:

                if (singleTouch) {

                    // lets implement drag/pan event.
                    // we need to change the height of the mouth and redraw the circle.

                    // Find the index of the active pointer and fetch its position

                    pointerIndex = MotionEventCompat.findPointerIndex(event, mActivePointerId);

                    if (pointerIndex >= 0) { // to avoid pointer index exception.
                        x = MotionEventCompat.getX(event, pointerIndex);
                        y = MotionEventCompat.getY(event, pointerIndex);

                        // Calculate the distance moved
                        final float dx = x - mLastTouchX;
                        final float dy = y - mLastTouchY;

                        // here we need to set the height of the mouth.
                        // lets find y boundary to detect this motion.

//                        if (y > yPosition - faceRadius && y < yPosition + faceRadius) { // change the mouth height only if the event occur inside the face.
//                            mouthHeight = mouthMaximumHeight * (dy / 1000);
//                            invalidate(); // redraw
//                        }
                        mouthHeight = mouthMaximumHeight * (dy / 1000);
                        invalidate(); // redraw

                    }
                } else {
                    // lets find the both the finger index.
                    // pointer index is the primary finger index

                    mActivePointerId = MotionEventCompat.getPointerId(event, 0);
                    pointerIndex = MotionEventCompat.findPointerIndex(event, mActivePointerId);
                    x = MotionEventCompat.getX(event, pointerIndex);
                    y = MotionEventCompat.getY(event, pointerIndex);

                    secondaryPointerId = MotionEventCompat.getPointerId(event, 1);
                    secondaryPointerIndex = MotionEventCompat.findPointerIndex(event, secondaryPointerId);
                    secondaryX = MotionEventCompat.getX(event, secondaryPointerIndex);
                    secondaryY = MotionEventCompat.getY(event, secondaryPointerIndex);


                    if (initialDistance == 0) {
                        initialDistance = (float) Math.sqrt(((secondaryX - x) * (secondaryX - x)) + ((secondaryY - y) * (secondaryY - y)));
                    }
                    distance = (float) Math.sqrt(((secondaryX - x) * (secondaryX - x)) + ((secondaryY - y) * (secondaryY - y)));

                    currentDistance = Math.abs(initialDistance - distance);

                    faceRadius = Math.max(minimumFaceRadius, Math.min(maximumFaceRadius, currentDistance));
                    invalidate();
                }
                break;

            case MotionEvent.ACTION_UP:
                initialDistance = 0;
                break;
            case MotionEvent.ACTION_CANCEL:
                initialDistance = 0;
                break;
        }

        return true;

    }
}
{% endhighlight %}

##IOS

IOS gives us lots of gesture recognizer function to detect gestures, so we dont have to deal with raw data. Lets add a pinch gesture recognizer to faceView to control zoom in and zoom out and a pan gesture recognizer to faceView to control the happiness.

_Pinch gesture_

+ Add pinch gesture recognizer to FaceView inside HappinessViewController.
{% highlight swift %}
faceView.addGestureRecognizer(UIPinchGestureRecognizer(target: faceView, action: "scale:"))
{% endhighlight %}

+ Action is the handler, when faceView detect an pinch gesture it will call scale function. Since the scale ends with a ':' it will take UIPinchGestureRecognizer as an input.

+ Define the scale function inside your FaceView. When the state of the gesture changes apply the required scale .
{% highlight swift %}
func scale(gesture: UIPinchGestureRecognizer){
    if gesture.state == .Changed {
        scale *= gesture.scale
        gesture.scale = 1
    }
}
{% endhighlight %}

_Pan gesture._

We are going to set up pan gesture from our story board.

1. Select your story board. click on the FaceView, now drag the pan gesture from the object library to the view.

2. Click the pan gesture icon from the top of the story board and ctrl-clik drag to HappinessViewController.

3. Make the connection as action, name as changeHappiness and the type as UIPanGestureRecognizer.

4. When the pan gesture in changed state, find the translation using .translationInView function.

5. Change the happiness value.

{% highlight swift %}
@IBAction func changeHappiness(gesture: UIPanGestureRecognizer) {
    switch gesture.state{
    case .Ended: fallthrough
    case .Changed:
        let translation = gesture.translationInView(faceView)
        let happinessChange = -Int(translation.y / Constants.HappinessGestureScale)

        if happinessChange != 0 {
            happiness += happinessChange
            gesture.setTranslation(CGPointZero, inView: faceView)
        }
    default: break
    }
}
{% endhighlight %}


_The final HappinessViewController looks like_
{% highlight swift %}
import UIKit

class HappinessViewController: UIViewController, FaceViewDataSource {

    private struct Constants {
        static let HappinessGestureScale: CGFloat = 4
    }

    var happiness: Int = 100 { // 0 is very sad, 100 is ecstatic
        didSet{
            happiness = min(max(happiness, 0), 100)
            print("Happiness = \(happiness)")
            updateUI()
        }
    }

    @IBAction func changeHappiness(gesture: UIPanGestureRecognizer) {
        switch gesture.state{
        case .Ended: fallthrough
        case .Changed:
            let translation = gesture.translationInView(faceView)
            let happinessChange = -Int(translation.y / Constants.HappinessGestureScale)

            if happinessChange != 0 {
                happiness += happinessChange
                gesture.setTranslation(CGPointZero, inView: faceView)
            }
        default: break
        }
    }

    @IBOutlet weak var faceView: FaceView! {
        didSet{
            faceView.dataSource  = self
            faceView.addGestureRecognizer(UIPinchGestureRecognizer(target: faceView, action: "scale:"))
//            faceView.addGestureRecognizer(UIPanGestureRecognizer(target: self, action: "changeHappiness:"))
        }
    }

    func smilinessForFaceView(sender: FaceView) -> Double {
        return Double (happiness - 50 ) / 50
    }

    func updateUI(){
        faceView.setNeedsDisplay()
    }
}

{% endhighlight %}


_The final FaceView looks like_

{% highlight swift %}
import UIKit

protocol FaceViewDataSource: class { // FaceViewDataSource can only be implemented by class type
    func smilinessForFaceView(sender: FaceView) -> Double
}

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

    weak var dataSource: FaceViewDataSource? // weak becase the datasource points to FaceView.

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

    func scale(gesture: UIPinchGestureRecognizer){
        if gesture.state == .Changed {
            scale *= gesture.scale
            gesture.scale = 1
        }
    }

    override func drawRect(rect: CGRect) {

        let facePath = UIBezierPath(arcCenter: faceCenter, radius: faceRadius, startAngle: 0, endAngle: CGFloat(2 * M_PI), clockwise: true)
        facePath.lineWidth = lineWidth
        color.set()
        facePath.stroke()

        bezierPathForEye(Eye.Left).stroke()
        bezierPathForEye(Eye.Right).stroke()

        let smileness = dataSource?.smilinessForFaceView(self) ?? 0.0 // ? optional chaining - if dataSource? is nil then everything comes after it will be nil; ??  if lhs is nil then use 0.0 else use lhs
        let smilePath = bezierPathForSmile(smileness)
        smilePath.stroke()

    }
}

{% endhighlight %}

[0]: http://laminin.github.io/draw/smiley/2015/08/15/draw-simple-smiley-faces-in-android-ios-swift-part1/
[1]: https://developer.android.com/training/gestures/index.html
[2]: https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIPanGestureRecognizer_Class/
[3]: https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIPinchGestureRecognizer_Class/
[4]: https://developer.android.com/training/gestures/multi.html
