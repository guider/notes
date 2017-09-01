####  


![20170506149404022924991.png](http://o7lzjfyxl.bkt.clouddn.com/20170506149404022924991.png)

  setContentView到底做了些什么，为什么调用后就可以显示出我们想要的布局页面？
	
	PhoneWindow倒是什么东西？Window和它是什么关系？
	
	DecorView是干什么用的？和我们的布局又有什么样的关系
	
	requestFeature为什么要在setContentView之前调用？

![20170506149404008044401.png](http://o7lzjfyxl.bkt.clouddn.com/20170506149404008044401.png)
	
	2、LayoutInflater 


	到底怎么把xml添加到decorview？
	
		include 为什么不能xml资源布局的根节点？
		
		merge 为什么作为xml资源布局的根节点？
	 
	
![20170506149403998388910.png](http://o7lzjfyxl.bkt.clouddn.com/20170506149403998388910.png)
	
   小结：每一个Activity都有一个关联的Window对象，用来描述应用程序窗口。
		 每一个窗口内部又包含了一个DecorView对象，Decorview对象用来描述窗口的视图--xml布局
	
	上述是创建DecorView的过程
	
	3、DecorView如何添加到Window
	看图片，最终调用了ViewRootImpl.setView
	在setview方法里调用了view.assignParent(this);,将Decorview的mParent设置成ViewRootImpl
	这也就是为什么View再用requestLayout方法的时候最终会走到ViewRootImpl的requestLayout


![20170506149404009584656.png](http://o7lzjfyxl.bkt.clouddn.com/20170506149404009584656.png)