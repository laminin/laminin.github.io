Part-1 - Create a custom ui.

Drawing curved lines.
The recent project i was working on had curved shape filled with some background color. Though i was given a image(.png) of the shape, i wanted to draw it on canvas. So i decided to give it a try.

  <image> curved_line_image_given <image>

I decided to draw the following shape to make it more generic.

  <image> curved_line_image_i_draw <image>

Steps i followed.

1. create a class CurvedLineView which extends View.

public class CurvedLineView extends View {

}

2. Override the constructors. And read the attributes from xml.

public CurvedLineView(Context context, AttributeSet attrs, int defStyleAttr) {
    super(context, attrs, defStyleAttr);
    mActivity = (Activity) context;
}

3. Create a init() method, and initialize all the required and known values.

    private void init(){
        linePaint = new Paint();
        linePaint.setColor(fillColor);
        linePaint.setStyle(Paint.Style.STROKE);
        linePaint.setStrokeWidth(5);
        linePaint.setAntiAlias(true);

        fillPaint = new Paint();
        fillPaint.setColor(fillColor);
        fillPaint.setStyle(Paint.Style.FILL);

        upperPathPaint = new Paint();
        upperPathPaint.setColor(upperFillColor);
        upperPathPaint.setStyle(Paint.Style.FILL);

        lowerPathPaint = new Paint();
        lowerPathPaint.setColor(lowerFillColor);
        lowerPathPaint.setStyle(Paint.Style.FILL);

    }

4. Draw the line and paths.
5. color the required area.  
