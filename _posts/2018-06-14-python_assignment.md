---
layout : default
title : python中的赋值，值传递？
excerpt : 几行很简单的代码,python里的赋值
---
几行很简单的代码
	
	def func(m):
	    m[0] = 10
	    m = [100,200,300]
	    return m
	
	m = [1,2,3]
	func(m)
	print(m)

猜测这个输出是什么呢？

A.【1，2，3】
B.【20，2，3】
C.【100，200，300】

乍一看，我还以为是选C，m不是被重新赋值了吗！but,运行后，证明我是错的
控制台输出的是B

为什么是B呢？
我加几行代码帮助理解

	def func(m):
	    print(id(m))
	    m[0] = 10
	    print(id(m))
	    m = [100,200,300]
	    print(id(m))
	    return m
		
	m = [1,2,3]
    print(id(m))
	func(m)
	print(m)
执行代码，输出如下

    36446536
	36446536
	36446536
	36552392
	[10, 2, 3]

在第一个和第二个print可以表明传递进来的对象是同一个36446536，经过赋值m[0]=10后，对象还是36446536，
在第四个print表明，此时的m已经不是传进来的那个“m”了

这个情况呢，用内存模型的角度来解释，就很好理解
![赋值调用方法]({{site.url}}/assets/2018-06-14-python_assignment/memery.jpg)
1.先定义m=[1,2,3]，有一个栈变量m和堆【1，2，3】，栈变量m指向堆【1，2，3】
2.调用func(m)，则在栈里，多一个“伪m”也指向堆【1，2，3】
3.执行m=[100,200,300]，则“伪m”修改指向伪堆【100，200，300】
但是呢，这样理解是不对的，因为python里只有值传递，按照上述内存模型的方式理解，则变成了引用传递了，but，如果按照值传递的思路，那如何解释前两次print(id)都是同一个36446536呢？


引用官方的原话：
>Remember that arguments are passed by assignment in Python.
>要记住，Python 里的参数是通过赋值传递的。