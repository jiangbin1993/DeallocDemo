# DeallocDemo
关于控制器和子视图无法释放的解决办法demo

> 最近在项目中发现了一个控制器和子视图释放不掉的问题。写此文记录一下，希望能帮到遇到和我一样问题的人。

附上demo下载地址：https://github.com/jiangbin1993/DeallocDemo.git

#### 问题一：在切换根控制器的时候，老的根控制器并没有释放掉。
##### 原因：在切换根控制器之前在根控制器上做了modal操作
```
LoginViewController *vc = [[LoginViewController alloc] init];
UINavigationController *nav = [[UINavigationController alloc] initWithRootViewController:vc];
[[UIApplication sharedApplication].keyWindow.rootViewController presentViewController:nav animated:YES completion:nil];
```

##### 解决方法：
在切换根控制器之前先dismiss这些控制器
```
AppDelegate *delegate = (AppDelegate *)[UIApplication sharedApplication].delegate;
// 必须先把根控制器modal出来的控制器dismiss掉再重设根控制器 不然之前的根控制器会释放不掉
[delegate.window.rootViewController dismissViewControllerAnimated:NO completion:nil];
delegate.window.rootViewController = nil;
JJTabbarViewController *vc = [[JJTabbarViewController alloc] init];
UINavigationController *nav = [[UINavigationController alloc] initWithRootViewController:vc];
delegate.window.rootViewController = nav;
```

#### 问题二：老控制器释放了，但是老控制器的子视图没有释放

原因一：子视图里面的定时器没有销毁
解决方法： 在控制器的dealloc方法里销毁掉子视图的定时器

原因二： 这个项目的控制器结构是appdelegate.window的根控制器是一个UINavigationController ，这个导航控制器的根视图是UITabbarController。tabbar控制器下是几个子控制器。在切换根控制器时会导致tabbar控制器当前正在显示的子控制器的子视图无法释放。

![项目结构图.png](https://upload-images.jianshu.io/upload_images/2541004-2195c5dc0b8671f1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




##### 解决方法： 在tabbar控制器所有的子控制器dealloc方法里移除掉所有的子视图。

在demo中一开始不移除子视图，在首页弹出登陆控制器，然后切换根控制器，发现所有的控制器和首页控制器之外的子视图都释放了，唯独首页控制器的子视图并没有释放。

**控制台打印 ：** 

*2019-02-25 18:10:23.588 TabbarDeallocDemo[96176:723960] tabbar控制器销毁了
2019-02-25 18:10:23.588 TabbarDeallocDemo[96176:723960] 首页控制器销毁
2019-02-25 18:10:23.588 TabbarDeallocDemo[96176:723960] 我的控制器销毁
2019-02-25 18:10:23.589 TabbarDeallocDemo[96176:723960] 我的contentview销毁*

然后在子控制器的dealloc方法中移除所有子视图重新运行，执行相同操作。这时所有的控制器和子视图都释放了。
```
- (void)dealloc {
for (id view in self.view.subviews) {
[view removeFromSuperview];
}
}
```

**控制台打印：**
*2019-02-25 18:02:35.175 TabbarDeallocDemo[96145:720401] tabbar控制器销毁了
2019-02-25 18:02:35.175 TabbarDeallocDemo[96145:720401] 我的控制器销毁
2019-02-25 18:02:35.176 TabbarDeallocDemo[96145:720401] 我的contentview销毁
2019-02-25 18:02:35.176 TabbarDeallocDemo[96145:720401] 首页控制器销毁
2019-02-25 18:02:35.176 TabbarDeallocDemo[96145:720401] 首页contentview销毁*
