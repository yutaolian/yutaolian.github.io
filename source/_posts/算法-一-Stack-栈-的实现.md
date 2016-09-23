---
title: '算法(一):Stack(栈)的实现'
date: 2016-09-10 14:52:55
tags:
	- 算法
	- Stack
	- 栈
---
### 0.简介
阅读过Java Stack源码的同学都只知道，Stack(栈)是继承自Vector(通常称为向量)而Vector是基于数组实现的，所以Stack也是是基于数组实现的。
### 1.特点
Stack(栈)作为一种简单的数据结构,其特点就是先进后出即First In Last Out(FILO)。其与另外一中数据结构Queue(队列)相反。队列是先进先出的。举个例子：栈比作一根一头封闭的管道，数据比作一个个小球，小球进入管道后先进去的在最里。而后进管道的小球在最外。先把所有的小球都拿出来的时候，其出来的顺序与进入的顺序是相反的。（队列下一片文章介绍）
 ![](/img/stack/stack.jpg)
### 2.基本方法
```
push() //压栈
pop()	//出栈
peek() //或的栈顶元素
isEmpty() //栈是否为空
	
```
### 3.代码实现(java 实现)
先上代码再分析

```
import java.util.Arrays;

public class MyStack<T> {
	//数据
	 Object[] data;
	//大小
	private int size;
	//默认容量
	private int capacity;
	//构造函数
	public MyStack() {
		this.size = 0;
		this.capacity = 10;
		this.data = new Object[capacity];
	}
	//扩容方法
	private void ensureCapacity(){
		capacity = capacity * 2;
		data = Arrays.copyOf(data, capacity);
	}
	//push
	public void push(T element){
		//是否需要扩容
		if (size < capacity) {
			data[size++] = element;
		}else{
			ensureCapacity();
			data[size++] = element;
		}
	}
	//pop
	public T pop(){
		if (size > 0) {
			Object obj = data[size-1];
			data[--size] = null;
			return (T) obj;
		}else{
			System.out.println("stack empty");
			return null;
		}
	}
	//isEmpty
	public boolean isEmpty(){
		return size == 0;
	}
	//栈顶元素
	public T peek(){
		if (size > 0) {
			return (T) data[size - 1];
		}
		return null;
	}
	//测试方法
	public static void main(String[] args) {
		MyStack<String> myStack = new MyStack<String>();
		
		for (int i = 0; i < 20; i++) {
			String string = "stack--"+i;
			myStack.push(string);
			System.out.println("push---"+string);
		}
		for (int i = 0; i < 6; i++) {
			System.out.println("pop---"+myStack.pop());

			System.out.println("peek---"+myStack.peek());
			
		}
		System.out.println("peek---"+myStack.peek());
		System.out.println(myStack.isEmpty());
		for (int i = 0; i < 6; i++) {
			System.out.println("pop---"+myStack.pop());
		}
			System.out.println(myStack.isEmpty());
		
	}

}

```

本代码只是简单实现一个栈的基本功能（不完善）感兴趣的同学可以参考Java Util包下的Stack和Vector具体实现方式。看了Java源码才发现自己代码与源码的差距简直是差太多了。不够可以发现Stack和Vector都是县城安全的（pop peek等方法都有synchronized关键字修饰）。栈的代码实现比较简单不做过多的分析了。之后可以一起阅读Stack和Vector的源码。

