---
layout: default
title: XMLSerializer的一个bug
#excerpt: 

---

　　相同的代码，读取未格式化xml和已格式化xml，未格式化的解析起来报错，代码很简单

# 1.java解析代码

```java
    // getResponseContent(fileName) 从指定文件名中读取文件内容作为字符串
    String responseXML = getResponseContent("content");

    // xml字符串 转换为 json字符串
    String responseJsonStr = new net.sf.json.xml.XMLSerializer().read(responseXML).toString();

    // json字符串 转换 json对象
    JSONObject responseJson = net.sf.json.JSONObject.fromObject(responseJsonStr);

    // 获取指定节点
    JSONArray details = responseJson.getJSONArray("details");
```



# 2.解析未格式化xml

未格式化的xml：

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?><searchdetail totalRecords="1" pageSize="1" requestId="201910311452000033"><details><transaction><player playerId="abc" partnerId="bcd"/><detail transactionId="43308923710" transactionDate="20191031 03:53:00" currency="CNY" game="FuStarH5" transactionSubType="Wager" handId="13809462143" amount="-2.50"/></transaction></details></searchdetail>
```

结果异常：

```java
Exception in thread "main" net.sf.json.JSONException: JSONObject["details"] is not a JSONArray.
    at net.sf.json.JSONObject.getJSONArray(JSONObject.java:2038)
    at io..M.main(M.java:53)
```



# 3.解析已格式化xml

已格式化的xml：

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<searchdetail totalRecords="1" pageSize="1" requestId="201910311451000013">
    <details>
        <transaction>
            <player playerId="abc" partnerId="bcd"/>
            <detail transactionId="43308923710" transactionDate="20191031 03:53:00" currency="CNY" game="FuStarH5" transactionSubType="Wager" handId="13809462143" amount="-2.50"/>
        </transaction>
    </details>
</searchdetail>
```

结果正常