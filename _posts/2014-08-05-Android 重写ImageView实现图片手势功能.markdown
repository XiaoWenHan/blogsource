---
layout: post
title: "Android 重写ImageView实现图片手势功能"
date: 2014-08-05 17:07:04
categories: Android开发

---
通过这个XImageView.java类，可实现单点触摸移动，以及双指缩放图片功能。
使用时，只要在布局文件中引入，并在Java代码中按照原生ImageView设置要显示的图片即可。  

代码如下（仅供参考）：

    import android.annotation.SuppressLint;  
	import android.content.Context;  
	import android.graphics.Bitmap;  
	import android.graphics.BitmapFactory;  
	import android.graphics.Canvas;  
	import android.graphics.Matrix;  
	import android.util.AttributeSet;  
	import android.view.MotionEvent;  
	import android.widget.ImageView;  
  
	public class XImageView extends ImageView {  
  
	    public static Bitmap gintama;  
	    private Matrix matrix;  
	    private float point1_old_x, point1_old_y, point1_new_x, point1_new_y,point2_old_x, point2_old_y, point2_new_x, point2_new_y,distance_old, distance_new;  
	    private float point_old_x, point_old_y, point_new_x, point_new_y;  
	    private boolean isMultiTouch = false;  
  
	    public XImageView(Context context, AttributeSet attrs) {  
	        super(context, attrs);  
	        gintama = BitmapFactory.decodeResource(getResources(),  
                R.drawable.ic_launcher);  
	        matrix = new Matrix();  
	    }  
  
	    public XImageView(Context context) {  
	        super(context);  
	        gintama = BitmapFactory.decodeResource(getResources(),  
                R.drawable.ic_launcher);  
	        matrix = new Matrix();  
	    }  
  
	    // 回调方法，用于画控件上的图片  
	    @Override  
	    protected void onDraw(Canvas canvas) {  
	        canvas.save();  
	        canvas.drawBitmap(gintama, matrix, null);  
	        canvas.restore();  
	    }  
  
	    @SuppressLint("ClickableViewAccessibility")  
	    @Override  
	    public boolean onTouchEvent(MotionEvent event) {  
	        switch (event.getAction() & MotionEvent.ACTION_MASK) {  
	        case MotionEvent.ACTION_DOWN:  
	            // 单点移动 按下  
	            if (event.getPointerCount() == 1) {  
	                isMultiTouch = false;  
	                point_old_x = event.getX();  
	                point_old_y = event.getY();  
	            }  
	            break;  
	        case MotionEvent.ACTION_UP:  
	            // 单点移动 抬起  
	            if (event.getPointerCount() == 1) {  
	                isMultiTouch = false;  
	                point_old_x = point_new_x;  
	                point_old_y = point_new_y;  
	            }  
	            break;  
	        case MotionEvent.ACTION_POINTER_UP:  
	            // 双点操作 抬起  
	            if (event.getPointerCount() == 2) {  
	                isMultiTouch = true;  
	                point1_old_x = point1_new_x;  
	                point1_old_y = point1_new_y;  
	                point2_old_x = point2_new_x;  
	                point2_old_y = point2_new_y;  
	            }  
	            break;  
	        case MotionEvent.ACTION_POINTER_DOWN:  
	            // 双点操作 按下  
	            if (event.getPointerCount() == 2) {  
	                isMultiTouch = true;  
	                point1_old_x = event.getX(0);  
	                point1_old_y = event.getY(0);  
	                point2_old_x = event.getX(1);  
	                point2_old_y = event.getY(1);  
	                distance_old = (float) Math.sqrt(((point1_old_x - point2_old_x)* (point1_old_x - point2_old_x) + (point1_old_y - point2_old_y) * (point1_old_y - point2_old_y)));  
	            }  
	            break;  
	        case MotionEvent.ACTION_MOVE:  
	            // 双点操作 移动  
	            if (event.getPointerCount() == 2) {  
	                isMultiTouch = true;  
	                point1_new_x = event.getX(0);  
	                point1_new_y = event.getY(0);  
	                point2_new_x = event.getX(1);  
	                point2_new_y = event.getY(1);  
	                pinch();  
	            } else {  
	                // 单点移动 移动  
	                if (event.getPointerCount() == 1) {  
	                    if (!isMultiTouch) {  
	                        point_new_x = event.getX();  
	                        point_new_y = event.getY();  
	                        matrix.postTranslate((point_new_x - point_old_x),(point_new_y - point_old_y));  
	                        point_old_x = point_new_x;  
	                        point_old_y = point_new_y;  
	                        invalidate();  
	                    }  
	                }  
	            }  
	            break;  
	        }  
	        return true;  
	    }  
  
	    // 缩放图片方法  
	    private void pinch() {  
	        distance_new = (float) Math.sqrt(((point1_new_x - point2_new_x)* (point1_new_x - point2_new_x) + (point1_new_y - point2_new_y) * (point1_new_y - point2_new_y)));  
	        matrix.postScale((distance_new / distance_old),(distance_new / distance_old), this.getWidth() / 2,  this.getTop() / 2);  
	        invalidate();  
	        distance_old = distance_new;  
	    } 
	}  