---
layout: post
title:  "java classloader"
date:  2016-03-06 22:40:00 +0800
categories: java
tags: java
keywords: java,web1992
---


> java中类加载器的使用
>

[1,类加载器](#1)

[2,类加载器的层次结构](#2)

[3,Class对象的创建过程](#3)

[4,根据java规范（约定）编写自己的ClassLoader](#4)

[5,对自己的`.class` 文件进行解密&解密](#5)

[6,问题思考](#6)

[7,问题解决](#7)

[8,Java jni ](#8)

[9,使用c/c++编写实现自己的ClassLoader](#9)

[10,其他](10)

###1,类加载器


- 引导类加载器 bootstrap
- 扩展类加载器 extension
- 应用程序类加载器（系统类加载器）System
- plugin 类加载器

<!--more-->

2,类加载器的层次结构
###
![](http://i.imgur.com/l2Qgtuz.png)

3,Class对象的创建过程
###
- `.java` 文件-> `.class` 文件
- `.class` 文件 -> `byte[]` 数组
- `byte[]` -> Clazz 对象
- Class.class.newInstance();

> 一小段源码（来自 java.lang.ClassLoader）
>

```java
	protected final Class<?> defineClass(String name, byte[] b, int off, int len)
        throws ClassFormatError
    {
        return defineClass(name, b, off, len, null);
    }
```

4,根据java规范（约定）编写自己的`ClassLoader`
###

```java
	  // 网络类加载器子类必须定义方法 findClass 和 loadClassData，以实现从网络加载类。
	 // 下载组成该类的字节后，它应该使用方法 defineClass 来创建类实例
	 class NetworkClassLoader extends ClassLoader {
			 String host;
			 int port;

			 public Class findClass(String name) {
				 byte[] b = loadClassData(name);
				 return defineClass(name, b, 0, b.length);
			 }

			 private byte[] loadClassData(String name) {
				 // load the class data from the connection
				  . . .
			 }
		 }
```
	
5, 对自己的`.class` 文件进行解密&解密
###

- 读取自己的 `.class` 文件
- byte[] readClassFile(String File)// 读取已经编译好的class文件
- byte[] newByte=EncryptClazz.encryptByte(data);// 加密
- 使用自己的 MyClassLoader，读取加密后的文件
- byte[] newByte=EncryptClazz.encryptByte(data);// 解密
- Class<?> defineClass(String name, byte[] b, int off, int len) //生成Class对象

6,问题思考
###
> 我们的代码最终会放到服务器上，进行运行,那么必须对我们的代码进行解密->运行
>
> 如果用人通过一定手段，获取了你的加密算法，那么很容易对你的程序进行解密
>
> 这时我们加密就没有意义了
>
> 如上面 MyClassLoader中会获取 解密后的byte[] 数组，那么就可以把这个写入到文件
>
> 这样就获得了源码。

7,问题解决
###
- 对3中自定义的 MyClassLoader 进行优化

- 读取自己的 `.class` 文件
- byte[] readClassFile(String File)// 读取已经编译好的class文件
- byte[] newByte=EncryptClazz.encryptByte(data);// 加密
- 使用自己的 MyClassLoader，读取加密后的文件

把这个

>byte[] newByte=EncryptClazz.encryptByte(data);// 解密
>
>Class<?> defineClass(String name, byte[] b, int off, int len) //生成Class对象

变成
> Class<?> makeClass(byte[]);
>

我们不再使用 jdk 中的 defineClass创建Class对象 ，而是自己实现 Class 的创建
这样对于 byte[] 对象的解密过程就隐藏起来了

8,java jni 
###
使用java jni 技术，调用c/c++ 方法，对 byte[] 进行解密，并创建Class对象，返还给java程序

- 定义一个 native 方法

> private native Class<?> makeClass(String name, byte[] data);
>

>举个栗子，编写native方法，调用c/c++ 方法
>

```java
	// HelloWorld.java
	public class HelloWorld {

		static {
			System.loadLibrary("Hello");
		}

		public native void DisplayHello();

		public static void main(String[] args) {

			new HelloWorld().DisplayHello();
		}

	}
```
	
>javac HelloWorld.java
>
>javah HelloWorld
>

```c
	// HelloWorld.h
	/* DO NOT EDIT THIS FILE - it is machine generated */
	#include <jni.h>
	/* Header for class HelloWorld */

	#ifndef _Included_HelloWorld
	#define _Included_HelloWorld
	#ifdef __cplusplus
	extern "C" {
	#endif
	/*
	 * Class:     HelloWorld
	 * Method:    DisplayHello
	 * Signature: ()V
	 */
	JNIEXPORT void JNICALL Java_HelloWorld_DisplayHello
	  (JNIEnv *, jobject);

	#ifdef __cplusplus
	}
	#endif
	#endif
```
	
> 根据javah 生成的头文件，编写自己的hello.cpp，编译-> 生成so 文件（可以理解问java 的jar）
>
```c
	// hello.cpp
	#include <jni.h>
	#include "HelloWorld.h"
	#include <stdio.h>
	JNIEXPORT void JNICALL Java_HelloWorld_DisplayHello
	(JNIEnv *env, jobject obj)
	{
				printf("From hello.cpp :");
				printf("Hello world ! \n");
				return;
	}
```

```sh
	g++  -I/usr/local/java/include -I/usr/local/java/include/linux  -o libHello.so -g -shared -fPIC  hello.cpp

	java -Djava.library.path=. HelloWorld
```

输出：
>From hello.cpp :Hello world !
>

9, 使用c/c++编写实现自己的ClassLoader
###
> 未完待续...
>
> private native Class<?> makeClass(String name, byte[] data);
> 
>

>jvm  源码ClassLoader.c 中 有一个 Java_java_lang_ClassLoader_defineClass1 方法
>
>对应这java ClassLoader中 

>from Java

```c++
	private native Class defineClass1(String name, byte[] b, int off, int len, ProtectionDomain pd, String source, boolean verify);
```

>jdk7 源码下载地址:https://jdk7.java.net/source.html	
>hg下载:http://hg.openjdk.java.net/jdk7u/jdk7u/
>
>form C++

```c++
	JNIEXPORT jclass JNICALL
	Java_java_lang_ClassLoader_defineClass1(JNIEnv *env,
											jobject loader,
											jstring name,
											jbyteArray data,
											jint offset,
											jint length,
											jobject pd,
											jstring source)
```

> C++ 源码具体实现 

```c++
	JNIEXPORT jclass JNICALL
	Java_java_lang_ClassLoader_defineClass1(JNIEnv *env,
											jobject loader,
											jstring name,
											jbyteArray data,
											jint offset,
											jint length,
											jobject pd,
											jstring source)
	{
		// code ..
	}
```

	
	
	
10, 知识准备Method.invoke 使用例子
###


```java
		package com.web.clazz;

		import java.lang.reflect.Method;

		/**
		 * Created by erbao.wang on 2016/3/8.
		 *
		 * @desc
		 */
		public class ClazzMain {


			public static void main(String[] args) throws Exception {

				ClassLoader myClassLoader = Thread.currentThread().getContextClassLoader();

				Class clazz = myClassLoader.loadClass("com.web.clazz.Demo");
				Object obj = clazz.newInstance();

				// doSomeThing 方法是公用的，非静态方法，需要实例去调用
				Method methodDoSomeThing = clazz.getMethod("doSomeThing", null);

				//实例方法,需要类的实例对象，第一个参数不能为空
				methodDoSomeThing.invoke(obj, null);


				Method methodMain = clazz.getMethod("main", new Class[]{String[].class});
				// 静态方法,不需要类的实例对象，第一个参数可为空
				methodMain.invoke((Object) null, (Object) null);

			}
		}


		class Demo {

			public static void main(String[] args) {
				System.out.println("main...");
			}


			public void doSomeThing() {
				System.out.println("doSomeThing...");
			}
		}	
```
	
>输出结果
>

	doSomeThing...
	main...