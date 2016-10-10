title: iOS（iOS7+）开发中的常用的Tips
date: 2015-09-01 22:55:25
tags: iOS
---

##### 0.设置状态栏颜色反转

```bash
    [self.navigationBar setBarStyle:UIBarStyleBlack];
```

##### 1.将导航栏的背景设为透明（默认是半透明状态）

```java
  [self.navigationController.navigationBar setBackgroundImage:[[UIImage alloc]init] forBarMetrics:UIBarMetricsDefault];
```
##### 2.xcode6配置pch文件

   ![](/img/iOS/pch.png)
