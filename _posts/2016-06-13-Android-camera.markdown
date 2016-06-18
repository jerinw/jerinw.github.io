---
layout:     post
title:      "Android Camera"
subtitle:   "安卓调用本地摄像头并自动对焦拍照"
date:       2016-06-13 15:02:00
author:     "Jerin"
header-img: "img/tag-bg.jpg"
catalog: true
tags:
    - 技术
---

# 前言

最近在做安卓程序开发，一个车牌识别的项目，其实车牌识别已经挺成熟了，只是：

1.安卓平台的识别比较少

2.对中文的支持不太好​​​

​之前试过两个开源的识别项目，一个是国外的Openalpr，看起来做得很好的样子，适用的平台也很多，虽然从字符上看还是支持中文的，但是实际检测的时候从来测不出中文。一个是国内的easypr，号称有95%的识别率，试了一下发现识别率应该没有这么高，但是还可以吧，起码支持中文，是cpp写的。

于是就打算​把easypr移植到安卓上，Github上有现成版本可以参考，但是只能识别本地一张固定名称的照片，特别傻，于是就打算调用一下摄像头，直接手机上点一下识别了。

核心的识别部分还是用的CPP的源码​​，NDK的配置什么就不讲了。



---

## 正文

​### A.权限

首先在manifest.xml里加入摄像头权限和读写SD卡权限

```
    <uses-permission android:name="android.permission.CAMERA"/>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS"/> 
```

### B.加入控件

layout的xml文件里加surfaceview，摄像头捕获的内容就显示在这个控件里。fill_parent表示控件有多大内容就有多大，比如这个activity里没有别的控件了，那这个摄像头捕获的图像就满屏幕，跟照相机一样，如果有别的控件，就填满除这个控件外剩下的空间。

```
	<SurfaceView
	    android:id="@+id/surfaceview"
	    android:layout_width="fill_parent"
	    android:layout_height="fill_parent" />
```

![](/img/2016-06-13/2.jpg)

### C.懒得写了，贴源码

```
package com.example.carplate;

import java.io.FileOutputStream;
import java.io.IOException;
import java.io.UnsupportedEncodingException;

import org.opencv.android.BaseLoaderCallback;
import org.opencv.android.LoaderCallbackInterface;
import org.opencv.android.OpenCVLoader;
import org.opencv.core.Mat;
import org.opencv.highgui.Highgui;

import android.os.Bundle;
import android.os.Environment;
import android.app.Activity;
import android.content.Context;
import android.content.res.Configuration;
import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.hardware.Camera;
import android.hardware.Camera.PictureCallback;
import android.view.MotionEvent;
import android.view.SurfaceHolder;
import android.view.SurfaceView;
import android.view.View;
import android.widget.ImageView;
import android.widget.TextView;

public class MainActivity extends Activity implements SurfaceHolder.Callback, Camera.PictureCallback {
	private ImageView imageView = null;
	private Bitmap bmp = null;
	private TextView m_text = null;
	private String path = null;
	
	private static Context context = null;  
	private SurfaceView surfaceview;  
	private SurfaceHolder surfaceholder;  
	private Camera camera = null;
	private boolean af;
	
	static {
	    if (!OpenCVLoader.initDebug()) {
	    } else {
	        System.loadLibrary("imageproc");
	    }
	}
	

//	protected void onCreate(Bundle savedInstanceState) {
//		super.onCreate(savedInstanceState);
//		setContentView(R.layout.activity_main);
//		imageView = (ImageView) findViewById(R.id.image_view);
//		m_text = (TextView) findViewById(R.id.myshow);
//		bmp = BitmapFactory.decodeResource(getResources(), R.drawable.plate_locate);
//		imageView.setImageBitmap(bmp);
//		path = Environment.getExternalStorageDirectory().getAbsolutePath(); 
//	}
	@Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity_main);  
        context = this;  
        
        m_text = (TextView) findViewById(R.id.myshow);  
        surfaceview = (SurfaceView)findViewById(R.id.surfaceview);  
        surfaceholder = surfaceview.getHolder();  
        surfaceholder.setType(SurfaceHolder.SURFACE_TYPE_PUSH_BUFFERS);  
        surfaceholder.addCallback(MainActivity.this);  //这里的safeactivity改成了mainactivity
        path = Environment.getExternalStorageDirectory().getAbsolutePath();
        //path新加，为了后来拍完照片存到识别的目录
        
    }  

	private BaseLoaderCallback mLoaderCallback = new BaseLoaderCallback(this) {
		@Override
		public void onManagerConnected(int status) {
			switch (status) {
			case LoaderCallbackInterface.SUCCESS: {
			}
				break;
			default: {
				super.onManagerConnected(status);
			}
				break;
			}
		}
	};
	
/**
 *识别的模块，点下那个识别的button以后要根据这两个训练好的svm.xml和ann.xml识别这张plate_locate.jpg。其中path为SD卡根目录
 */
	public void click(View view) throws UnsupportedEncodingException {
		System.out.println("entering the jni");
		String svmpath = path+"/svm.xml";
		String annpath = path+"/ann.xml";
		String imgpath = path+"/plate_locate.jpg";
		byte[] resultByte = CarPlateDetection.ImageProc(imgpath,svmpath,annpath);
		String result = new String(resultByte,"GBK");
		System.out.println(result);
		m_text.setText(result);
	}

	@Override
	protected void onResume() {
		super.onResume();
		mLoaderCallback.onManagerConnected(LoaderCallbackInterface.SUCCESS);
	}
	
/**
 *摄像头捕获图像预览的模块  
 */
	@Override  
	public void surfaceChanged(SurfaceHolder arg0, int arg1, int arg2, int arg3) {  
	    System.out.println("surfacechanged");  
	}  
	@Override  
	public void surfaceCreated(SurfaceHolder holder) {  
	    System.out.println("surfacecreated");  
	    //获取camera对象  
	    camera = Camera.open();  
	    try {  
	        //设置预览监听  
	        camera.setPreviewDisplay(holder);  
	        Camera.Parameters parameters = camera.getParameters();  
	          
	        if (this.getResources().getConfiguration().orientation   
	                    != Configuration.ORIENTATION_LANDSCAPE) {  
	            parameters.set("orientation", "portrait");  
	            camera.setDisplayOrientation(90);  
	            parameters.setRotation(90);  
	        } else {  
	            parameters.set("orientation", "landscape");  
	            camera.setDisplayOrientation(0);  
	            parameters.setRotation(0);  
	        }  
	        camera.setParameters(parameters);  
	        //启动摄像头预览  
	        camera.startPreview();  
	        System.out.println("camera.startpreview");  
	          
	    } catch (IOException e) {  
	        e.printStackTrace();  
	        camera.release();  
	        System.out.println("camera.release");  
	    }  
	}  
	@Override  
	public void surfaceDestroyed(SurfaceHolder arg0) {  
	    System.out.println("surfaceDestroyed");  
	    if (camera != null) {  
	        camera.stopPreview();  
	        camera.release();
	        camera =null;
	    }  
	}  
	
/**
 * 拍照的模块，点击屏幕自动对焦，松开手指时拍照，并保存在SD卡根目录
 */
    @Override
    public boolean onTouchEvent(MotionEvent event) {//屏幕触摸事件
    	if (event.getAction() == MotionEvent.ACTION_DOWN) {//按下时自动对焦
            camera.autoFocus(null);
            af =true;
        }
        if (event.getAction() == MotionEvent.ACTION_UP & af ==true) {//放开后拍照
            camera.takePicture(null, null,this);
            af =false;
        }
        return true;
    }
 
    public void onPictureTaken(byte[] data, Camera camera) {//拍摄完成后保存照片
    	try {
            String picpath = path +"/plate_locate.jpg";
            data2file(data, picpath);
        } catch (Exception e) {
        }
        camera.startPreview();
    }
 
    private void data2file(byte[] w, String fileName) throws Exception {//将二进制数据转换为文件的函数
        FileOutputStream out =null;
        try {
            out =new FileOutputStream(fileName);
            out.write(w);
            out.close();
        } catch (Exception e) {
            if (out !=null)
                out.close();
            throw e;
        }
    }
 

}
```

## 后记


其实就是在一个车牌识别的软件里加一个摄像头的功能，使其可以直接拍照识别，其中CarPlateDetection.ImageProc这个函数写在另外的文件里，是调用一些cpp文件实现的。