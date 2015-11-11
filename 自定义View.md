#Custom View#

**概要**
		


- 创建Custom View
- 重写onDrawn，包括Measure，Layout过程
- 实现View的交互
	

##创建Custom View##
####继承自View####
在Android Framework中所有的控件都继承自View，所以Custom View可以直接继承自View，或是现有的控件，如Button等。为了Android Develop Tools能够和Custom View进行交互，Custom View中至少有一种带有Context和AttributeSet参数的构造方法，这个构造方法可以允许布局编辑器创建和编辑Custom View的实例。

     class CustomView extends View {
		public CustomView(Context context, AttributeSet attrs) {
			super(context, attrs);
		}
	}

####自定义属性####
- 在<declare-styleable>中自定义属性
- 确定这些属性的值
- 运行时解析这些属性值
- 将这些属性值应用到Custom View中

在*res/values/attrs.xml*中定义一个*<declare-styleable>*，如下代码：

    <resources>
		<declare-styleable name="CustomView">
			<attr name="showText" format="boolean" />
        	<attr name="labelPosition" format="enum">
            	<enum name="left" value="0" />
            	<enum name="right" value="1" />
        	</attr>
		</declare_styleable>	
	</resources>

创建后，就可以在xml布局中引用这些属性，但是这些属性的xml namespace不能用*http://schemas.android.com/apk/res/android*，应该改成*http://schemas.android.com/apk/res/[your package name]*，如下所示：

    <?xml version="1.0" encoding="utf-8"?>
	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
   	   xmlns:custom="http://schemas.android.com/apk/res/com.kevin.customviews">
 	<com.kevin.customviews.CustomView
     	custom:showText="true"
     	custom:labelPosition="left" />
	</LinearLayout>
这里注意：


1. 当用gradle build时，这里命名空间必须用*http://schemas.android.com/apk/res-auto*。
2. 当定义的CustomView是一个内部类时，比如其外部类为OutView，则上面的com.kevin.customviews.CustomView应该改为*com.kevin.customviews.OutView$CustomView*。

####使用自定义属性####
Android在将xml转为R文件时，会将自定义属性生成常量。在创建CustomView时，只需将布局文件中关于CustomView的属性值传进构造函数，即从布局xml中导入设置的属性，转为AttributSet，然后作为参数传入CustomView的构造函数中，在构造函数中的操作如下：

    public CostumView(Context context, AttributeSet attrs) {
		super(context, attrs);
		TypedArray a = context.getTheme().obtainStyledAttributes(
			attrs,
			R.styleable.CustomView,
			0, 0);
		try {
			mShowText = a.getBoolean(R.styleable.CustomView_showText, false);
			mTextPos = a.getInteger(R.styleable.CustomView_labelPosition, 0);
		} finally {
			a.recycle();
		}
	}
注意：源码中可知，有一个关于TypeArray实例的pool，数量有限，用完时需调用其recycle()方法，方便后面的使用。

####增加属性和事件的代码控制####
自定义的属性在view初始化后，只能读取，想在运行时动态的初始化，就得自己增加一些设置属性的方法（get和set方法）：
    public boolean isShowText() {
		return mShowText;
	}
	
	public void setShowText(boolean showText) {
		mShowText = showText;
		invalidate();
		requestLayout();
	}
其中，在改变view的任何属性后，都要通过invalidate()方法来通知system，让system重新绘制（redrawn）。相同的，如果某个属性的改变影响了view的大小和形状，那么就要调用requestLayout()方法。

也可以自定义event和listener。在实际自定义View中，所有影响view的外观的最好都要实现其属性和事件的控制代码。




##自定义Drawing##
在绘制自定义View中，最重要的就是重写onDrawn(Canvas canvas)方法，其Canvas参数就是来绘制view自身的。

####创建绘制对象####
在android.graphics框架中，将绘制分成两个部分：

- 使用Canvas来控制绘制什么
- 使用Paint来控制怎么绘制

在绘制之前，都要先创建一个或多个Paint对象，如下面的init()所示。

    private void init() {
   		mTextPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
   		mTextPaint.setColor(mTextColor);
   		if (mTextHeight == 0) {
       		mTextHeight = mTextPaint.getTextSize();
   		} else {
       		mTextPaint.setTextSize(mTextHeight);
  		}

   		mPiePaint = new Paint(Paint.ANTI_ALIAS_FLAG);
   		mPiePaint.setStyle(Paint.Style.FILL);
   		mPiePaint.setTextSize(mTextHeight);

   		mShadowPaint = new Paint(0);
   		mShadowPaint.setColor(0xff101010);
   		mShadowPaint.setMaskFilter(new BlurMaskFilter(8, BlurMaskFilter.Blur.NORMAL));
	}

注意：init方法是在view的构造函数中调用，这是因为view一般都要经常重新绘制，而每次绘制的初始化操作都很expensive，所以提前在构造方法中创建Paint实例。

####处理布局变化####
为了能够正确的绘制view，得先知道view的size和shape等信息。一般情况下，大部分的measure方法是不需要重写的，只需重写下onSizeChanged()方法就可以了。可以在onSizeChanged()方法中计算positions，dimensions等，该方法在view第一次赋予size或后续size变化时，得到调用。

    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);

        // Account for padding
        float xpad = (float) (getPaddingLeft() + getPaddingRight());
        float ypad = (float) (getPaddingTop() + getPaddingBottom());

        // Account for the label
        if (mShowText) xpad += mTextWidth;

        float ww = (float) w - xpad;
        float hh = (float) h - ypad;

        // Figure out how big we can make the pie.
        float diameter = Math.min(ww, hh);
	}
注意：计算时需要先减去padding。

如果需要控制view的layout parameters，可以重写onMeasure方法，该方法的参数View.MeasureSpec传递父view希望我们view的大小。

    @Override
	protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
	   // Try for a width based on our minimum
	   int minw = getPaddingLeft() + getPaddingRight() + getSuggestedMinimumWidth();
	   int w = resolveSizeAndState(minw, widthMeasureSpec, 1);
	
	   // Whatever the width ends up being, ask for a height that would let the pie
	   // get as big as it can
	   int minh = MeasureSpec.getSize(w) - (int)mTextWidth + getPaddingBottom() + getPaddingTop();
	   int h = resolveSizeAndState(MeasureSpec.getSize(w) - (int)mTextWidth, heightMeasureSpec, 0);
	
	   setMeasuredDimension(w, h);
	}

其中，setMeasuredDimension方法是必须调用的。

####重写onDrawn####
重写onDrawn的一般套路：

- 绘制text，使用Canvas的drawText()。改变text字体，使用Paint的setTypeface()，改变text颜色，使用Paint的setColor。
- 绘制基本shape，使用Canvas的drawRect(),drawOval(),drawnArc()。改变shape是否填充，使用Paint的setStyle()。
- 绘制复杂shape，使用Path类。通过添加直线、曲线来定义一个shape，然后采用Canvas的drawPath()来绘制这个shape。如同基本的shape，也可以用Paint的setStyle来设置是否填充，画轮廓。
- 绘制渐变填充，使用LinearGradient（继承自android.graphics.Shader）。调用Painter的setShader()来使用LinearGradient填充shape。
- 绘制bitmap，使用Canvas的drawBitmap()。
下面有一段例子：
    protected void onDraw(Canvas canvas) {
	   super.onDraw(canvas);
	
	   // Draw the shadow
	   canvas.drawOval(
	           mShadowBounds,
	           mShadowPaint
	   );
	
	   // Draw the label text
	   canvas.drawText(mData.get(mCurrentItem).mLabel, mTextX, mTextY, mTextPaint);
	
	   // Draw the pie slices
	   for (int i = 0; i < mData.size(); ++i) {
	       Item it = mData.get(i);
	       mPiePaint.setShader(it.mShader);
	       canvas.drawArc(mBounds,
	               360 - it.mEndAngle,
	               it.mEndAngle - it.mStartAngle,
	               true, mPiePaint);
	   }
	
	   // Draw the pointer
	   canvas.drawLine(mTextX, mPointerY, mPointerX, mPointerY, mTextPaint);
	   canvas.drawCircle(mPointerX, mPointerY, mPointerSize, mTextPaint);
	}

##处理view的交互##
####基本的Touch####
可以重写onTouchEvent(android.view.MotionEvent)，来定义touch的行为：

     @Override
	 public boolean onTouchEvent(MotionEvent event) {
	    return super.onTouchEvent(event);
	 }

####手势控制GestureDetector####
GestrueDetector多个构造方法都需要传入手势的监听listener,listener可以实现GestrueDetector.OnGestrueListener，不过更方便的是实现GestrueDetector.SimpleOnGestrueListener。

    class OneListener extends GestureDetector.SimpleOnGestrueListner {
		@Override
		public boolean onDown(MotionEvent e) {
			return true;
		}
	}
	mDetector = new GestureDetector(context, new OneListener());

除非不想捕捉手势，否则必须重写onDown()方法，返回true。因为所有的手势都是从onDown开始，如果onDown返回false，则后续所有的手势都会被忽略。

当创建了手势实例时，可以在view的onTouchEvent方法中，由手势实例来解释event，如果手势实例不能识别该event，则返回false。当然，可以自己定义手势实例可以捕捉那些event。
    
    @Override
	public boolean onTouchEvent(MotionEvent event) {
	   boolean result = mDetector.onTouchEvent(event);
	   if (!result) {
	       if (event.getAction() == MotionEvent.ACTION_UP) {
	           stopScrolling();
	           result = true;
	       }
	   }
	   return result;
	}

####模拟物理运动的效果####
当手指快速划过屏幕，然后将手指抬起，这样的效果在屏幕上就是快速滑动，然后慢慢停止下来。这样的效果可以通过Scroller的fling方法来实现。

    @Override
	public boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX, float velocityY) {
	   mScroller.fling(currentX, currentY, velocityX / SCALE, velocityY / SCALE, minX, minY, maxX, maxY);
	   postInvalidate();
	}

其中，velocity可以通过GestureDetector来获取，由于GestureDetector获取的速度（velocity）太快，一般都要除以SCALE（范围4到8）。此后，还需定时通过computeScrollOffset更新Scroller，computeScrollOffset通过读取当前时间和物理坐标来更新。这因为Scoller能够计算当前的x、y坐标，所以会将Scroller传给View的srollTo()函数，来确定是否滑动结束。

Scoller可以计算出滑动的位置，但是它不会自动的将这些位置应用在view中，所以想要滑动的动画看上去顺滑，必须自己手动获取位置，并将其应用在view中，可以通过以下方法：

- 在调用fling()后，调用postInvalidate()，来重新绘制view。这个方法要求在onDraw()中计算出滑动的偏移，并且在偏移后调用postInvalidate()。
- 建立一个ValueAnimator来模拟fling过程，并且增加对ValueAnimator的监听（addUpdateListener()）。

采用动画模拟，可以不需要调用postInvalidate重新绘制view。但是ValueAnimator只能在API 11以上才能使用。

    mScroller = new Scroller(getContext(), null, true);
       mScrollAnimator = ValueAnimator.ofFloat(0,1);
       mScrollAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
           @Override
           public void onAnimationUpdate(ValueAnimator valueAnimator) {
               if (!mScroller.isFinished()) {
                   mScroller.computeScrollOffset();
                   setPieRotation(mScroller.getCurrY());
               } else {
                   mScrollAnimator.cancel();
                   onScrollFinished();
               }
           }
       });


####使用动画Transitions####
推荐ValueAnimator和ViewPropertyAnimator。