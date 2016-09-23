---
title: '算法(二):实现一个getMin功能的栈'
date: 2016-09-10 15:59:55
tags:
	- 算法
	- Stack
	- 栈
---

> 题目:
> 在Stack的基础上实现一个可以获得Stack的最小元素的方法（getMin）,要求pop,push,getMin操作的时间复杂度是o(1)
> 

题目来源《程序员代码面试指南》作者：左程云

### 0.分析
题目要求时间复杂度为o(1),即可以直接获得Stack的最小元素，我们可以使用两个栈，一个栈（StackData）存储全部元素，一个栈(StackMin)来存储最小元素（其实不是一个最小元素，应该是比第一个压入StackData栈的小的所有元素），getMin即获得StackMin栈的栈顶元素，需要注意的就是在push和pop时都需要和StackMin栈的栈顶做比较,如果出栈的数据与StackMin栈顶元素相同那么StackMin也需要pop数据。
如图
![](/img/stack/getMinStack.png)

### 1.代码

```

import java.util.Stack;

public class MinSatck {
	
	private Stack<Integer> stackData;
	private Stack<Integer> stackMin;
	
	public MinSatck() {

		this.stackData = new Stack<>();
		this.stackMin = new Stack<>();
	}
	public void push(int item){
		
		if (stackMin.isEmpty()) {
			stackMin.push(item);
		}else if (item <= getMin()) {
			stackMin.push(item);
		}	
		
		stackData.push(item);
	}
	
	public int pop(){
		if (stackData.isEmpty()) {
			throw new RuntimeException("stack is empty");
		}else{
			int value = stackData.pop();
			if (value == getMin()) {
				stackMin.pop();
			}
			return value;
		}
	}
	
	public int getMin(){
		if (stackMin.isEmpty()) {
			throw new RuntimeException("stack is empty");
		}
		return stackMin.peek();
	}
	
}
```



