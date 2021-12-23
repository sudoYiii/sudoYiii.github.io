---
title: IDEA配置Tomcat
date: 2021/11/18 18:28:00
categories: 
- 安装教程
tags: 
- Java
- Tomcat
---

#### 点击右上角的File->New->Project，新建一个Java项目。

![01](/images/20211118/01.png)

#### 右键工程添加框架。

![02](/images/20211118/02.png)

#### 选择Web Application。

![03](/images/20211118/03.png)

#### 配置Project Structure

![04](/images/20211118/04.png)

#### 配置Sources，在项目WEB-INF下创建两个文件夹classes和lib。

![05](/images/20211118/05.png)

#### 配置Paths，将两个output path修改为刚才创建的classes的地址 

![06](/images/20211118/06.png)

#### 配置Dependencies，点击＋号选择JARs or Directories，选择刚才创建的lib路径。

![07](/images/20211118/07.png)

![08](/images/20211118/08.png)

#### 选择Jar Directory，OK后将Export框打上勾。

![09](/images/20211118/09.png)

#### 配置Tomcat，点击run->Edit Configurations。

![10](/images/20211118/10.png)

#### 点击+号找到Tomcat Server->Local。

![11](/images/20211118/11.png)

#### 在Development中点击+号加一个Artifact。

#### 配好之后面板会有些变化，证明Tomcat已经配好了 。

#### 在index.jsp中随便写点东西运行测试。