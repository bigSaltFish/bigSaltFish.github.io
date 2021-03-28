---
layout: default
title: HttpClient使用setEntity传递参数 无法获取参数？
#excerpt: 
---



　　今天在接入一个第三方支付，在接入的时候，碰到一个问题：第三方在回调我方的时候，第三方居然只有一个json字符串。一般情况，都是Key:Value的形式，根据Key去获取对应的Value，所以我就有点蒙逼了。   

　　去联系第三方，让他们提供demo，然而，人家说没有demo，核心代码都已经在文档里了。   

　　我去翻了文档，看着回调这一块，孤零零的文字描述，连伪代码都没有，心里一万头草泥马。。。   

　　既然对方不提供demo，那只能自己多试试了。首先，写一个发起http请求的方法  

```java
    public static void httpTest(String url) throws ClientProtocolException, IOException {
        String token = "R5amyr6NyXCtWdScmNiuvVwBCJztfByZDUGaE2V0NwOUheW4XYlvUusYkrViTYt584RgcyXRhjxAJZG3rFlPLg";
        String json = "{\"action_name\":\"QR_LIMIT_SCENE\",\"action_info\":{\"scene\":{\"scene_id\":234}}}";
        HttpClient httpClient = new DefaultHttpClient();
        HttpPost post = new HttpPost(url);
        StringEntity postingString = new StringEntity(json);// json传递
        post.setEntity(postingString);
        post.setHeader("Content-type", "application/json");
        HttpResponse response = httpClient.execute(post);
        String content = EntityUtils.toString(response.getEntity());
        // Log.i("test",content);
        System.out.println("content:" + content);
        result = content;
    }
```

　　发起的写完了，再写一个接收的方法。接收，使用SpringMvc或者Struts都可以，这里就不贴了。在接收端，使用HttpServletRequest对象，遍历了getParameterMap的返回值，然而，并没有看到我期待的值。  

　　搞不懂，去google，搜到的结果，也不尽人意。   

　　去卫生间放水的路上，突然灵光乍现：当年刚工作的时候，有一次是接入亚马逊的接口，跟现在的情形，很相似，努力回忆起来，当初也是在getParamer的方式获取不到，最后好像是从request的inputStream解析成字符串才搞定的。难道，这次也是？ 完事后，回到工位马上进行验证。  

　　这里涉及到输入流转字符串，我使用的是Spring全家桶的StreamUtils工具类，代码如下  

```java
String reqBbody = StreamUtils.copyToString(this.getRequest().getInputStream(), StandardCharsets.UTF_8);
```

　　再次访问，后台输出，果然能看到setEntity里的参数了。  

　　ok，问题解决了，但是，为什么会是这样呢？