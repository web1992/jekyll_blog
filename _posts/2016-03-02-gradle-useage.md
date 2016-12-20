---
layout: post
title:  "gradle useage memo"
date:  2016-03-02 17:25:00 +0800
categories: gradle
tags: gradle
keywords: gradle,web1992
---

### `gradle` useage memo (gradle用法备忘)

生成idea和eclipse项目

	gradle idea
	gradle eclipse

----------

<!--more-->

- [1,安装](#1)
- [2,使用daemon](#2)
- [3,列出项目使用的jar包](#3)
- [4,置依赖的多种方式](#4)
- [5,build Java Library （构架jar）](#5)
- [6,配置gradle task](#6)
- [7,gradle变量配置](#7)

###1 安装

	略
gradle 文档在gradle-xxx.zip/docs/userguide/userguide.html 中可以找到
gradle 官网引用了google 的文件，导致访问十分慢，

	
###2 使用daemon

```sh
		# 启用 gradle 后台进程，这样在本机进行开发时，进行build 时的速度提高（节省时间）
		touch ~/.gradle/gradle.properties && echo "org.gradle.daemon=true" >> ~/.gradle/gradle.properties
	
```


###3 列出项目使用的jar包

```groovy
		task listJars << {
			configurations.compile.each { File file -> println file.name }
		}	
```

执行task listJars

```groovy
		gradle -q listJars
```

###4 配置依赖的多种方式

```
		dependencies {
		    runtime group: 'org.springframework', name: 'spring-core', version: '2.5'
		    runtime 'org.springframework:spring-core:2.5',
		            'org.springframework:spring-aop:2.5'
		    runtime(
		        [group: 'org.springframework', name: 'spring-core', version: '2.5'],
		        [group: 'org.springframework', name: 'spring-aop', version: '2.5']
		    )
		    runtime('org.hibernate:hibernate:3.0.5') {
		        transitive = true
		    }
		    runtime group: 'org.hibernate', name: 'hibernate', version: '3.0.5', transitive: true
		    runtime(group: 'org.hibernate', name: 'hibernate', version: '3.0.5') {
		        transitive = true
		    }
		}
		
```
###5 build Java Library （构架jar）

- 使`java-library-distribution`插件
- 配置Zip文件名和目录
- 执行task, `java-library-distribution` 提供了 `gradle distZip` task

---


```groovy
		apply plugin: 'java-library-distribution'
			
		distributions {
	    	main {
	        	baseName = 'my-name'
	        	contents {
	            	from { 'src/dist' }
	        	}
	    	}
	}
```
----------

```sh
		gradle distZip
```

###6 配置gradle task

关于task的配置可参照 http://gradle.web1992.cn/more_about_tasks.html#defineAsExpression

```groovy
	task myCopy(type: Copy)
	
	myCopy {
	   from 'resources'
	   into 'target'
	   include('**/*.txt', '**/*.xml', '**/*.properties')
	}
	
	task taskX << {
	    println 'taskX'
	}
	
	task taskY << {
	    println 'taskY'
	}
	#配置依赖
	taskX.dependsOn taskY
```

###7 gradle变量配置（局部变量&全局变量）

```groovy
	#局部变量 def 定义
	def dest = "dest"
	
	task copy(type: Copy) {
	    from "source"
	    into dest
	}
	
	apply plugin: "java"
	#全局变量 ext 定义
	#子项目可访问 ext 全局变量
	ext {
	    springVersion = "3.1.0.RELEASE"
	    emailNotification = "build@master.org"
	}
	
	sourceSets.all { ext.purpose = null }
	
	sourceSets {
	    main {
	        purpose = "production"
	    }
	    test {
	        purpose = "test"
	    }
	    plugin {
	        purpose = "production"
	    }
	}
	
	task printProperties << {
	    println springVersion
	    println emailNotification
	    sourceSets.matching { it.purpose == "production" }.each { println it.name }
	}
	
	
```
	
	
###8 gradle项目之间的依赖

```groovy
	dependencies {
		compile project(':a1')
		compile project(':a2')
	}
    ### application 插件
	apply plugin: 'application'
	### 
	version '1.0.0'
	### 启动的Main 方法
	mainClassName = "com.demo.A2"
```

###9 whenReady 动态设置task 的值

```groovy
	task distribution << {
    		println "We build the zip with version=$version"
	}	

	task release(dependsOn: 'distribution') << {
    		println 'We release now'
	}

	gradle.taskGraph.whenReady {taskGraph ->
	    if (taskGraph.hasTask(release)) {
	        version = '1.0'
	    } else {
	        version = '1.0-SNAPSHOT'
	    }
	}	
```