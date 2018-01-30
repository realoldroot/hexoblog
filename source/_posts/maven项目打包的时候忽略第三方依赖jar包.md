---
title: maven项目打包的时候忽略第三方依赖jar包
date: 2017-11-28 18:20:50
tags: maven
---



因为项目需要经常性的修改-打包-上传到服务器，包含lib的war包比较大，第三方jar包基本不会大动，所以就想把这些jar包移出war包，放在jetty里面，这样每次只需要编译打包源码即可，war包大小极度减小，方便上传。
maven配置

```
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-war-plugin</artifactId>
    <version>3.1.0</version>
    <configuration>
        <warName>ROOT</warName>
        <packagingExcludes>
            %regex[WEB-INF/lib/(?!lock-).*.*.jar]
        </packagingExcludes>
    </configuration>
</plugin>
```
打包的时候排除/WEB-INF/lib/下面的jar包，lock- 是我自己的jar包，需要保留，所以使用正则处理了一下。
其他的第三方jar包全放在/jetty/lib/ext 中。