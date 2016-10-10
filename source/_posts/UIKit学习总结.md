title: UIKit学习总结
date: 2015-08-30 22:30:50
tags: iOS
---

### UIKit  应该是iOS开发中最重的工具包了吧。
##### 0.关于UIKit中比较重要的类

&nbsp;&nbsp;&nbsp;&nbsp;一定要去官网[UIKit Framework Reference](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIKit_Framework/index.html#//apple_ref/doc/uid/TP40006955)找最权威的资料。其树形结构已经很直观告诉其中类之间的关系

##### 1.UIWindow

&nbsp;&nbsp;&nbsp;&nbsp;之所以将这个类放在第一位是因为在每个iOS项目的AppDelegate文件中有这么一个成员变量。而且我们还会个它设置一个rootViewController

``` bash
    self.window.rootViewController = xxViewController
```

&nbsp;&nbsp;&nbsp;&nbsp;而这个window就是指的当前设备的屏幕。就是把viewController中管理的页面数据加载到设备上去。

##### 2.UIViewController

&nbsp;&nbsp;&nbsp;&nbsp;大家都知道iOS开发算是把MVC模式用到了极致，viewController就是管理view的控制器,我们可以在controller中构建页面样式，加载页面数据，做这个页面展示需要做的所有工作，然后把控制器交给window(设备)就好。当然根控制器只能有一个但是我们可以通过控制器之间的切换来示不同的页面。

&nbsp;&nbsp;&nbsp;&nbsp;常见的viewController

 ![](/images)

&nbsp;&nbsp;&nbsp;&nbsp;其中红色的框中的controller为常用的视图控制器，当然也可以不是用UIViewController的这个子控制器，直接使用UIViewController然后在其中添加各种view.(其实各种控制器只是默认在在UIViewController中加好了对应的view，实现了对应协议的方法而已）

##### 2.UIView
&nbsp;&nbsp;&nbsp;&nbsp;图上已经标出几个比较常用的view
  ![](/img/iOS/uiview.png)

##### 3.UIGestureRecognizer
&nbsp;&nbsp;&nbsp;&nbsp;关于手势识别的view
   ![](/img/iOS/gesture.png)
