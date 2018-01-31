---
title: 一个比较实用的maven发布项目到服务器的插件
date: 2017-11-20 18:07:51
tags: mysql
---



wagon-maven-plugin,用来把打包之后的文件发送到服务器上
```xml
<plugin>
	<groupId>org.codehaus.mojo</groupId>
	<artifactId>wagon-maven-plugin</artifactId>
	<version>1.0</version>

	<configuration>
		<source>1.8</source>
		<target>1.8</target>
		<fromFile>target/ROOT.war</fromFile>
		<url>scp://用户名@服务器ip/服务器路径/jetty/webapps</url>
	</configuration>
</plugin>
```
就是参照了linux的scp命令,当然如果使用的是mac的话,可以在终端直接实现.