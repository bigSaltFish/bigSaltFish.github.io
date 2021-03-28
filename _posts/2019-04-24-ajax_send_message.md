---
layout: default
title: ajax传递参数的一个坑
#excerpt: 
---

　　上周有一个需求，需要使用ajax请求后端，参数是比较复杂，有普通字符串也有对象数组。之前都是使用form表单来实现的，效果是这样的  

![期待的效果]({{site.url}}/assets/2019-04-24-ajax_send_message/expect.jpg) 

　　后来，需求变更了，需要提交局部表单的效果。既然是局部的内容提交，那就不能使用form来实现了，我就想着用js手动拼接对象，传输给后台。代码如下：  

```javascript
	var arr = {
            list : [ {
                id : 0,
                name : "我是名字",
                age : 18
            }, {
                id : 1,
                name : "我是名字",
                age : 18
            }, {
                id : 2,
                name : "我是名字",
                age : 18
            } ]
        };

        $.ajax({
            url : "",
            data : arr,
            success : function() {

            }
        });
```


　　然而，实际结果却是这样的：  

![期待的效果]({{site.url}}/assets/2019-04-24-ajax_send_message/unexpect.jpg)

　　数组元素的属性后面，跟上了不期待的中括号，因为后面带上了中括号，所以后台就解析不了。当时是为了赶进度，转成json字符串到后端，后端再解析，绕过了这个问题。  

　　现在有时间了，就来分析一下

# 结论

　　先说结论，只需要把参数作为字符串操作：  

```javascript
var jsonParam = decodeURIComponent($.param(json)).replace(/\[([^ \[\]]*?[^ \[\]\d]+?[^ \[\]]*?)\]/g, ".$1");
```

　　把这个处理过后的字符串作为参数，传递给后端，就能正常使用了。　　

# 原因

## 为什么字符串不会被带中括号？

　　为什么使用“json字符串”传输到后端就是没问题的呢？因为，在使用ajax函数的时候，它会去对参数进行处理，如果参数是String类型，就不处理。所以，当转换成json字符串和使用decodeURIComponent的时候，就是正常的。    

　　jquery-3.2.1.js 中 第9040行

```javascript
	// Convert data if not already a string
	if ( s.data && s.processData && typeof s.data !== "string" ) {
		s.data = jQuery.param( s.data, s.traditional );
	}
```

## 为什么对象数组的属性会被带上中括号？

　　为什么对象数组的属性也会被带上中括号呢？因为在处理参数的时候，它会判断是否是数组，如果是数组，它会遍历数组，每个属性塞上中括号。  

　　jquery-3.2.1.js 中 第8383行，else语句块中

```javascript
	// Serialize array item.
	jQuery.each( obj, function( i, v ) {
		if ( traditional || rbracket.test( prefix ) ) {

			// Treat each array item as a scalar.
			add( prefix, v );

		} else {

			// Item is non-scalar (array or object), encode its numeric index.
			buildParams(
				prefix + "[" + ( typeof v === "object" && v != null ? i : "" ) + "]",
				v,
				traditional,
				add
			);
		}
	} );
```

　　至于jquery为什么会使用中括号来处理参数，有可能是基于某些情况的考虑吧。。。
