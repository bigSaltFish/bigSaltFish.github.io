---
layout: default
title: 文件上传的坑
#excerpt: 

---

　　前段时间我们系统被黑客攻击了，后来查明，是利用上传文件的漏洞。于是老大就让下面的人写个拦截器，对上传文件的内容进行审查。代码乍一看，是没啥问题的，但是，这几天我正好闲着，就多测了测，居然无意中发现了两个bug。  

　　下面的伪代码：  

```java
@Override
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
        throws IOException, ServletException {
    final HttpServletRequest req = (HttpServletRequest) request;
    final HttpServletResponse res = (HttpServletResponse) response;
    
    if (HttpMethod.POST.equalsIgnoreCase(req.getMethod())) {
        final MultiReadHttpServletRequest multiReadRequest = new MultiReadHttpServletRequest(
                (HttpServletRequest) request);
        if (multiReadRequest.getContentType() != null
                && multiReadRequest.getContentType().contains("multipart/form-data")) {
            MultipartHttpServletRequest multipartHttpServletRequest = multipartResolver
                    .resolveMultipart(multiReadRequest);
            Map<String, MultipartFile> fileMap = multipartHttpServletRequest.getFileMap();
           
			for (Map.Entry<String, MultipartFile> entity : fileMap.entrySet()) {
				// 文件审查                        
			}
        }
    }
    chain.doFilter(req, res);
}
```

# 第一个问题：只审查了POST请求。  

　　代码的第七行，只拦截POST请求。因为我们绝大部分情况下，上传文件都是使用post请求的，从大学课本里：上传文件是post的；从工作中：其他人的代码里，是post的。但是，为什么不可以使用GET请求呢？  

　　刚开始工作的那两年，找工作面试的时候，总是会被问到“GET请求和POST请求是什么区别”，面经里写的很清楚：GET请求的参数是放到URL中、长度限定是2M；而POST请求是放到body里的，长度无限制。  

　　貌似是没有地方写到：GET请求是不能上传文件的。  

　　我特意进行了测试，发现，GET请求来上传文件，是OK的。  



# 第二个问题：getFileMap()方法是不可靠的  

　　代码里使用“multipartHttpServletRequest.getFileMap()”来获取所有的文件，然后迭代，审查文件内容。然而，GetFileMap方法是不可靠的，因为如果一个文件，不设定文件名，那就会绕过审查方法。  

　　比如，使用postman工具-->Body-->选择form-data栏--> 选择key类型为File(只选择类型，不输入值，在Value栏选择文件)。  

![postman上传文件]({{site.url}}/assets/2019-04-01-file_upload_bug/postman_fileupload.jpg)

　　此时，就不会被getFileMap()方法抓住，会绕过审查。  

　　再来说一下，为什么getFileMap()方法是不可靠的。虽然我是无意中发现的，但是，既然发现了，那就得去扒一下，下次就可以愉快的吹逼了，哈哈  

　　扒源码的过程是痛苦了，历经千山万水，终于在FileUploadBase.java中，findNextItem()方法找到。下面是源码，去除了无关的代码  

```java
private boolean findNextItem() throws IOException {
	for (;;) {
		FileItemHeaders headers = getParsedHeaders(multi.readHeaders());
		if (currentFieldName == null) {
			// We're parsing the outer multipart
			String fieldName = getFieldName(headers);
			if (fieldName != null) {
				// ...
			}
		} else {
			String fileName = getFileName(headers);
			if (fileName != null) {
				// ...
			}
		}
		multi.discardBodyData();
	}
}
```

　　至于apache的文件上传，为什么忽略了文件名为null的情况。我猜测，可能是程序里操作文件，是需要根据文件名去操作的，假如没有文件名，就无法操作，自然也就可以忽略了。

　　tips:对于上传文件，应该在服务端统一处理，去除原文件名，用uuid命名，无后缀。记录到文件管理表，文件的原名、uuid和后缀