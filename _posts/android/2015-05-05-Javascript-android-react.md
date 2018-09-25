---
layout: post
title:  "android js 互调"
categories: android
tags:  android
---

* content
{:toc}

最近在做原生和js端的互调的功能，自己改了个demo，给大家讲解下。

<!--more-->

先上js代码
```
<!DOCTYPE html>
<html>
<head>
	<meta charset="UTF-8">
	<title>方法调用基本流程测试</title>
</head>
<body>
<div id="helloweb"> 
	<div id="echoInfo">如果有数据返回，会显示在这儿</div>
	</div>
	<script type="text/javascript">
	
	function funFromjs(){
    	document.getElementById("helloweb").innerHTML="HelloWebView,i'm from js";
    }
		function echoInfo( container, obj ){
			var domContainer = document.getElementById('echoInfo');
			domContainer.innerHTML = JSON.stringify( obj );
		}

		//function windowCallback( str ){
			//echoInfo( 'echoInfo', str );
		//}

		 window.windowCallback = function( str ){
		 	echoInfo( 'echoInfo', str );
		 };

		var MfsJSBridge = MfsJSBridge || undefined;
		if( undefined != MfsJSBridge ){

			//看这里
			var params = {
				id : 1,
				name : '测试'
			};

			var strParams = JSON.stringify( params );

			MfsJSBridge.invoke( 'testFunc', strParams, 'windowCallback');

		}else{
			alert('未定义MfsJSBridge');

		}

	</script>

</body>
</html>
```
android webview 设置可用javascript
```java
//设置编码
 mWebView.getSettings().setDefaultTextEncodingName("utf-8");
 //支持js
mWebView.getSettings().setJavaScriptEnabled(true);
```
android 调js

```java
mBtn1.setOnClickListener(new View.OnClickListener() {

            @Override
            public void onClick(View v) {
              mWebView.loadUrl("javascript:funFromjs()");//其中funFromjs()为js的方法
              Toast.makeText(mContext, "调用javascript:funFromjs()", Toast.LENGTH_LONG).show();
            }
        });

```

js调原生，原生响应时间并回调数据

```java

 mWebView.addJavascriptInterface(new Object(){
        	//注意4.4以后加注解，位置在这个方法名上面，鉴于很多这个的例子，瞎、、写注解位置，并需要下                        //载积分写了这个
	        @JavascriptInterface
            public void invoke(String name ,String t,String callback) {
            	if(name.equals("testFunc")){ 
            	//其中t 为js带过来的数据          
            		Toast.makeText(mContext, t,Toast.LENGTH_LONG).show();
            		
            		String strJson = "{\"code\":122, \"msg\":\"1231\", \"data\":null}";

            		//回调数据给js 其中callback 为android 掉js 的方法名称。
            		mWebView.loadUrl("javascript:"+ callback +"('" + strJson + "')");
            	}
              //  Toast.makeText(mContext, name, Toast.LENGTH_LONG).show();
                
            }
        },"MfsJSBridge");
```
代码比较简单，最主要的是 @JavascriptInterface注解的位置大家注意下。

[源码下载](http://download.csdn.net/detail/forezp/9555368)


 




