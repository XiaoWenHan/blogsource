title: "开发手记——基于XMPP的Android即时通讯APP（二）"
date: 2015-02-09 19:17:25
tags:
categories: Android开发

---

隔了几天，把应用的登录、注册部分做的比较完善了，当然这只是个人感觉哈。
今天要说的，**都是干货**！  
首先，**没有大片代码** ；其次，**在网上一般找不到** ；最后，真的让你**节约开发时间**！这也是为什么时隔好几天才会发第二篇连载。  
既然说，没有大片代码，一般的登录、注册流程这里就不提了，百度谷歌一搜一大堆，而且基本都能用。这里说几点注意：  



**关于后台服务**：  
官方的建议，要使用“START_STICKY”这种类型的后台服务。为什么要用这种服务，官方的说法很明朗：  

![](http://i.imgur.com/CCtCF7k.png)

这点和Android Developer官网解释保持一致，需要注意的是，为了照顾4.4.2版本的Bug，需要在程序中做些处理。  
上面的截图引用自：https://github.com/Flowdalic/asmack/wiki/Should-applications-using-aSmack-use-foreground-Services%3F  



**关于断线重连**：  
新版本的aSamck4.0.6包中可以断线重连了，这一点和之前的版本还是有一点区别的。  

**关于用户登陆的判断**：  
可以通过XMPPConnection对象获取连接状态，具体说明如下：  
xmppConnection.isConnected() ：布尔值表示是否连接到服务器（注意：此时用户不一定登录）；  
xmppConnection.isAuthenticated() ： 布尔值表示是否登录成功（即用户名+密码验证通过），此时代表已登录成功，且与服务器保持连接。  

**关于与服务器的交互操作**：  
用线程，Handler，切记！即使是在Service中。  

**调试小技巧**：  
为了看清和服务器交互的内容（xml），可以在和服务器连接之前对ConnectionConfiguration做如下设置：  

    SmackConfiguration.DEBUG_ENABLED = true;  
	connectionConfiguration = new ConnectionConfiguration(PrefUtil.getServerIP(context), PrefUtil.getServerPort(context));  
	connectionConfiguration.setReconnectionAllowed(true);  
	connectionConfiguration.setSendPresence(true);  
	connectionConfiguration.setRosterLoadedAtLogin(true);  
	connectionConfiguration.setSecurityMode(SecurityMode.disabled);  
	connectionConfiguration.setDebuggerEnabled(BuildConfig.DEBUG);  
	xmppConnection = new XMPPTCPConnection(connectionConfiguration);  
	xmppConnection.connect();   

其中connectionConfiguration是ConnectionConfiguration对象，PrefUtil是首选项工具类。

**注册时摸不到头脑**？  
4.0.6版本的aSmack包貌似用于注册的Registration类无法setUsername和setPassword，怎么办？下方贴出这部分代码：    

    Registration registration = new Registration();  
	registration.setType(IQ.Type.SET);  
	registration.setTo(xmppConnection.getServiceName());  
	Map<String, String> attributes = new HashMap<String, String>();  
	attributes.put("username", username);  
	attributes.put("password", password);  
	registration.setAttributes(attributes);  
	PacketFilter filter = new AndFilter(new PacketIDFilter(registration.getPacketID()), new PacketTypeFilter(IQ.class));  
	PacketCollector collector = xmppConnection.createPacketCollector(filter);  
	xmppConnection.sendPacket(registration);  
	IQ result = (IQ) collector.nextResult(xmppConnection.getPacketReplyTimeout());  
	collector.cancel();  
	if (result == null || result.getType() == IQ.Type.ERROR) {  
    	DebugUtil.Log("An error occured during registion, maybe the same username");  
	    return false;  
	} else {  
    	DebugUtil.Log("Registion has OK!");  
	    return true;  
	}  

如上所示，用HashMap来存放用户名和密码，然后作为Attributes，set到registration对象中就注册成功了。  

**需要切换用户**？  
当应用需求切换当前登陆账户时，务必要先调用XMPPConnection中的disconnect()方法断开和服务器的连接（貌似没有单独的注销账户方法），然后重新connect()，再login()。  

最后说一下项目进展：  
目前完成了登录、注册和个人信息修改的功能。登录可以多账户，保存各账户的密码（如果用户要求），自动登录；由于应用提供了服务器的手动配置，因此在用户注册前可以切换到其他的服务器；上述服务器配置在登录界面可做修改，并存入首选项，应用启动时会检查是否已经配置；个人信息修改今天刚刚做好，手动小测了一下，还可以，但是鉴于没有做过严格的测试，所以先不提。  
项目开源路径：  
[https://wh1990xiao2005@bitbucket.org/wh1990xiao2005/anychat.git](https://wh1990xiao2005@bitbucket.org/wh1990xiao2005/anychat.git)