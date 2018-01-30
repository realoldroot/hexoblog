---
title: springMvc直接接收json数据自动转化为Map<String,String>
date: 2017-05-05 18:44:25
tags: java springMvc
---



springMvc直接接收json数据自动转化为Map ，必须加上@RequestBody注解并且前台ajax发送请求的时候需要对数据进行格式化

```javascript
$.ajax({ 
    type : "POST", 
    url : "/search", 
    data :JSON.stringify(searchData), 
    contentType:"application/json",
    dataType : "json", 
    success : function(data) { } 
});
```

```java
@RequestMapping(value = "/search",method = RequestMethod.POST)
public void search (@RequestBody Map<String,String> map){
    System.out.println("传进来的参数：" + map);
}
```

重点在于 contentType:"application/json" 及配套使用的@RequestBody 