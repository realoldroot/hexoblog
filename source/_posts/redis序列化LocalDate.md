---
title: 解决redis序列化java8 LocalDateTime错误的问题
date: 2017-08-15 15:02
tags: java
---

redis序列化选择方式

```xml
<!-- 缓存序列化方式 -->
    <!--对key的默认序列化器。默认值是StringSerializer-->
    <bean id="keySerializer" class="org.springframework.data.redis.serializer.StringRedisSerializer" />
    <!--是对value的默认序列化器，默认值是取自DefaultSerializer的JdkSerializationRedisSerializer。-->
    <bean id="valueSerializer" class="org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer" >
    <bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
        <property name="connectionFactory"   ref="connectionFactory" />
        <!-- 这里修改了redis默认的序列化方式 -->
        <property name="keySerializer" ref="keySerializer" />
        <property name="valueSerializer" ref="valueSerializer" />
        <property name="hashKeySerializer" ref="keySerializer" />
        <property name="hashValueSerializer" ref="valueSerializer" />
    </bean>
```

<!-- more -->

要序列化class Demo

```java
public class Demo {
    private Long id;
    private String name;
    private LocalDateTime time;
    ......
    }
```
操作存入redis中

```java
@Test
public void test1() {
    Demo demo = new Demo();
    demo.setId(10000000001L);
    demo.setName("测试序列化");
    demo.setTime(LocalDateTime.now());
    redisTemplate.opsForValue().set("test", demo);
 }
```
在redis中查看

```json
{
  "@class": "com.karmay3d.Demo",
  "id": 10000000001,
  "name": "测试序列化",
  "time": {
    "dayOfMonth": 15,
    "dayOfWeek": "TUESDAY",
    "dayOfYear": 227,
    "month": "AUGUST",
    "monthValue": 8,
    "year": 2017,
    "hour": 14,
    "minute": 45,
    "second": 51,
    "nano": 921000000,
    "chronology": {
      "@class": "java.time.chrono.IsoChronology",
      "id": "ISO",
      "calendarType": "iso8601"
    }
  }
}
```
LocalDateTime 存储的方式有问题。然后再从redis中取出Demo

```java
@Test
    public void test2() {
        Demo demo = redisTemplate.opsForValue().get("test");
        System.out.println(demo.getId());
    }
```
报异常

```log
org.springframework.data.redis.serializer.SerializationException: Could not read JSON: Can not construct instance of java.time.LocalDateTime: no suitable constructor found, can not deserialize from Object value (missing default constructor or creator, or perhaps need to add/enable type information?)
 at [Source: [B@68d651f2; line: 1, column: 81] (through reference chain: com.karmay3d.Demo["time"]); nested exception is com.fasterxml.jackson.databind.JsonMappingException: Can not construct instance of java.time.LocalDateTime: no suitable constructor found, can not deserialize from Object value (missing default constructor or creator, or perhaps need to add/enable type information?)
 at [Source: [B@68d651f2; line: 1, column: 81] (through reference chain: com.karmay3d.Demo["time"])
......
Caused by: com.fasterxml.jackson.databind.JsonMappingException: Can not construct instance of java.time.LocalDateTime: no suitable constructor found, can not deserialize from Object value (missing default constructor or creator, or perhaps need to add/enable type information?)
 at [Source: [B@68d651f2; line: 1, column: 81] (through reference chain: com.karmay3d.Demo["time"])
	at com.fasterxml.jackson.databind.JsonMappingException.from(JsonMappingException.java:270)
	at com.fasterxml.jackson.databind.DeserializationContext.instantiationException(DeserializationContext.java:1456)
......
```
这个问题真的是纠结了几天。。。解决办法
maven需要的jar包

```xml
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
<dependency>
    <groupId>com.fasterxml.jackson.datatype</groupId>
    <artifactId>jackson-datatype-jsr310</artifactId>
    <version>${jackson.version}</version>
</dependency>
```
LocalDateTime属性加上注解
```java
@JsonDeserialize(using = LocalDateTimeDeserializer.class)
@JsonSerialize(using = LocalDateTimeSerializer.class)
```

```java
public class Demo {
    private Long id;
    private String name;
    @JsonDeserialize(using = LocalDateTimeDeserializer.class)
    @JsonSerialize(using = LocalDateTimeSerializer.class)
    private LocalDateTime time;
    ......
    }
```
redis再次存入之后结构

```json
{
  "@class": "com.karmay3d.Demo",
  "id": 10000000001,
  "name": "测试序列化",
  "time": [2017,8,15,14,57,37,525000000]
}
```
之后反序列化就可以取出Demo对象了。

[解决这个问题查找的问答](https://stackoverflow.com/questions/13592467/can-not-deserialize-instance-of-org-joda-time-datetime-or-localdate-out-of-start)