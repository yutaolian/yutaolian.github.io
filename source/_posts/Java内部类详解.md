---
title: Java内部类详解
date: 2016-09-03 09:27:47
tags:
	- Java
	- Java内部类	
---

## 0.什么是内部类
> 可以将一个类的定义放在另一个类的内部，这就是内部类。-《Java编程思想》

**让我们先来看一段代码**
 
```
    public class InnerClassTest {
    	
    	public class InnerClass{
    		public void innerMethod(){
    			System.out.println("this is a inner class method");
    		}
    	}
    	public static void main(String[] args) {
    		
    		InnerClassTest innerClassTest = new InnerClassTest();
    		InnerClass innerClass = innerClassTest.new InnerClass();
    		innerClass.innerMethod();	
    	}
    
    }
```
 
 上面的代码简单展示了内部类的定义和使用，
 我们可以查看编译后的文件
 ![](/img/innerclass/innerclass01.png)
## 1.内部类的种类
######一般内部类可分为四种：
* 成员内部类
* 局部内部类
* 嵌套内部类
* 匿名内部类

