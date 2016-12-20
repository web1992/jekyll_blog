---
layout: post
title:  "java i++ 与 ++i 的区别"
date:  2016-03-28 15:25:00 +0800
categories: java
tags: java
keywords: java,web1992
---


####java i++ 与 ++i 的区别

本文从java的字节码角度，去探索一下i++ 与++i的本质的区别

<!--more-->
####java代码1


```java
	public class Test {

	   public static void main(String[] args) {

		    int q = 0;
			q = q++;
			System.out.println(q);
	  }

```

>输出结果 0
>

####java代码2
                                                  
                                                     
```java                      
        public class Test {                          
                                                     
           public static void main(String[] args) {  
                                                     
                        int q = 0;                   
                        q = ++q;                     
                        System.out.println(q);       
          }                                          
                                                     
``` 

>输出结果 1
> 

####出现这个现象的原因是由java的编译之后的操作码决定的（JVM Opcode Reference）

> 运行下面命令，查看反编译之后的操作码（指令码）
>
> javap -c Test
>

####代码1的反编译

```java
	Code:
    	0: iconst_0
    	1: istore_1
    	2: iload_1
    	3: iinc          1, 1
    	6: istore_1
    	7: getstatic     #2   
   		10: iload_1
   		11: invokevirtual #3   
   		14: return
```

####代码2的反编译： 

```java
	Code:
	0: iconst_0
	1: istore_1
	2: iinc          1, 1
	5: iload_1
	6: istore_1
	7: getstatic     #2    
	10: iload_1
	11: invokevirtual #3    
	14: return 
```
   

                  
反编译之后的代码区别：

![](http://i.imgur.com/zlue6od.png)
![](http://i.imgur.com/7aBMw2D.png)

>关于iinc 指令解释，可参照 JVM Opcode Reference
>
>[http://homepages.inf.ed.ac.uk/kwxm/JVM/codeByFn.html#A2b](http://homepages.inf.ed.ac.uk/kwxm/JVM/codeByFn.html#A2b)

>iinc
>
>**Increment local var.**

**iinc 的意思对本地变量进行自增+1，注意这里指的是本地变量 local var**

**关于java中栈帧，参照下图：(可自行Google java操作数栈)**


**这里需要明白几个概念：**

1. 局部变量表，用来存在局部变量的
 
2. 操作数栈 是进行算术运算的地方

3. 在进行算术运算时，局部变量表与操作数栈是存在数据交互的

可参考：[ "栈帧、局部变量表、操作数栈 http://wangwengcn.iteye.com/blog/1622195"](http://wangwengcn.iteye.com/blog/1622195)
> 以上是操作数栈的相关概念

<br />
> 下面对反编译的字节码的解释
> 

代码1的指令码执行顺序

	2: iload_1				//  从局部变量表中，加载索引为1的local var 到`操作数栈`
    3: iinc          1, 1   //  局部变量表中索引为1的local var自增+1,本地变量的值是1，此时 `操作数栈`依然为0
    6: istore_1 		    //   pop int, store into local variable 1，从  `操作数栈`取值，存储在本地变量中，覆盖了原有已经自增+1的变量

>所以结果是：0


<br />
代码2的指令码执行顺序

	2: iinc          1, 1 // 局部变量表中索引为1的local var自增+1,本地变量的值是1
	5: iload_1			  // 从局部变量表中，加载索引为1的local var 到`操作数栈`，加载的数据值为1
	6: istore_1			  // pop int, store into local variable 1，从  `操作数栈`取值，存储在本地变量中

>所以结果是：1


