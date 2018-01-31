---
title: java8 LocalDate 类型 json 解析 日期格式处理
date: 2017-08-04 10:52
tags: java
---


在使用java8的过程中用到了新的日期类LocalDate、LocalDateTime类型，作为属性不经过任何处理转成json的时候会变成下面的样式。


<!-- more -->
```json
"applicationTime": {
        "month": "AUGUST",
        "year": 2017,
        "dayOfMonth": 2,
        "dayOfWeek": "WEDNESDAY",
        "dayOfYear": 214,
        "monthValue": 8,
        "hour": 16,
        "minute": 38,
        "second": 53,
        "nano": 0,
        "chronology": {
          "id": "ISO",
          "calendarType": "iso8601"
        }
      },
```
如何才能变成字符串的yyyy-MM-dd，需要导入几个类库

```xml
<properties>
	<jackson.version>2.8.5</jackson.version>
</properties>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-annotations</artifactId>
    <version>${jackson.version}</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>${jackson.version}</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
    <version>${jackson.version}</version>
</dependency>
<!-- 支持java8 localDate等新时间类型的序列化 -->
<dependency>
    <groupId>com.fasterxml.jackson.datatype</groupId>
    <artifactId>jackson-datatype-jsr310</artifactId>
    <version>${jackson.version}</version>
</dependency>
```
只有jackson-datatype-jsr310是用来格式化日期用的

```java
 @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd HH:mm:ss")
 private LocalDateTime applicationTime;
```
或者
```java
 @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
 private LocalDateTime applicationTime;
```
加上@JsonFormat注解，这样最后当对象转成json的时候就变成了

```json
"applicationTime": "2017-08-02 16:38:59",
```