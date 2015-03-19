title: "开发手记——基于XMPP的Android即时通讯APP（三）"
date: 2015-03-03 20:59:25
tags:
categories: Android开发

---

首先祝各位读者新年快乐，博主在这里给大家拜万年啦！而且马上要到元宵节，顺祝大家元宵节快乐！  
上一次谈了注册和登录的编码技巧，这一次我们来谈谈加好友的技巧。  
**搜索用户**：  
XMPP协议为我们提供了完善的好友查找功能，而且通过aSmack的库，能够轻易实现模糊查找功能。为了保证应用程序的通用性。  
在搜索时，我们最好按如下的方法做：  

    UserSearchManager usm = new UserSearchManager(xmppConnection);  
	Form searchForm = usm.getSearchForm("search." + xmppConnection.getServiceName());  
	Form answerForm = searchForm.createAnswerForm();  
	answerForm.setAnswer("Username", true);  
	answerForm.setAnswer("search", searchContent);  
	ReportedData data = usm.getSearchResults(answerForm, "search." + xmppConnection.getServiceName());  
	data.getRows();  

注意：由于我们需要使用"search."+[服务器名]的字符串作为参数，以便让我们的应用更加通用。而xmppconnection对象提供了服务器名，因此，将上述参数设置为：

    "search." + xmppConnection.getServiceName()  

即可实现在任何xmpp服务器上进行用户查找的需求了。

**关于头像**：  
由于我们的程序最终要运行在Android设备上，因此，要对内存进行较为细致的控制，以防内存溢出。我在程序中设置了头像的上传大小为200*200，80%的压缩质量，png格式。并将其放在数据库中作为blob类型字段保存，权当缓存了。当然，每次登陆后会和服务器联系获取最新的头像和好友信息。  
关键的图片处理在此：  

    /** 
 	* 将制定路径下的图像文件转换为指定长宽的byte数据 
 	*  
 	* @param bitmapPath 
 	* @param width 
 	* @param height 
 	* @return 
 	*/  
	public static Bitmap resizeBitmap(String bitmapPath, int width, int height) {  
	    Options options = new Options();  
    	options.inJustDecodeBounds = true;  
    	BitmapFactory.decodeFile(bitmapPath, options);  
    	options.inSampleSize = calcSampleSize(options, width, height);  
	    options.inJustDecodeBounds = false;  
    	return BitmapFactory.decodeFile(bitmapPath, options);  
	}  
  
	// Bitmap转byte[]  
	public static byte[] bitmapToBytes(Bitmap bitmap) {  
	    ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();  
	    bitmap.compress(Bitmap.CompressFormat.PNG, 80, byteArrayOutputStream);  
    	return byteArrayOutputStream.toByteArray();  
	}  
  
	// 计算sampleSize  
	private static int calcSampleSize(Options options, int width, int height) {  
    	int returnData = 1;  
    	if (options.outHeight > height || options.outWidth > width) {  
    	    int heightRatio = Math.round((float) options.outHeight / (float) height);  
    	    int widthRatio = Math.round((float) options.outWidth / (float) width);  
    	    returnData = heightRatio > widthRatio ? heightRatio : widthRatio;  
    	}  
	    return returnData;  
	}  

最后提醒大家一句：别忘了在onDestroy()中recycle使用的Bitmap哟！另外，在读取用户VCard时，务必要记得优化图片的显示，否则，在PC端设置一张超大的图片作为头像，手机这边一登录没准直接就OOM了。  
**数据库配置**：  
这里提醒读者一句：所有存到数据库中的值一定要转码！一方面是为了数据安全，其实更多的另一方面是为了更好的程序健壮性。试想：如果一个账户的用户名是group、sort、order，直接傻眼了。  
我的方法是仅仅取字符串的byte[]，转换成了16进制，作为字符串存到数据库中。这样就避免了数据库操作报错崩溃的问题。当然，对于关键的聊天记录数据，也可以选择加密存储，然后解密查看。  
**键盘优化**：  

    android:windowSoftInputMode="stateHidden|adjustResize"  

把上述配置加至activity节点下，一方面可实现键盘出现时调整布局，另一方面是自动隐藏键盘。是个不错的用户体验，比默认的显示策略好很多。