## Hybrid 方案原理 安卓 WebView 和 h5 的交互原理（JSBridge 原理）

列举了 JSBridge 的几种内部实现逻辑。可以编译工程看代码

## 安卓调用 JS

- WebView 的 loadUrl

```
    mWebView.loadUrl("javascript:callJS()");
```

这个方法无法拿到返回值，webview 会刷新。

- WebView 的 evaluateJavascript
  这个方法可以拿到返回值

```
mWebView.evaluateJavascript("javascript:callJS()", new ValueCallback<String>() {
    @Override
    public void onReceiveValue(String value) {
                                //此处为 js 返回的结果
                                System.out.println("return: " + value);
    }
});

```

## JS 调用 Android

- 通过 WebView 的 addJavascriptInterface（）进行对象映射
  在 java 中有一个对象映射到 webview 里，webview 可以拿到这个 js 对象。注册在 window 上
  在 java 端的代码

```
mWebView.addJavascriptInterface(new AndroidtoJs(), "test");
```

在 webview 中的 JS 端

```
  function callAndroid(){
        if(window.test){
         test.hello("js调用了android中的hello方法");
        }else{
            console.log('error,test not exists.');
        }
    }

    let btn1 = document.getElementById("btn1");
    btn1.addEventListener('click',()=>{
        callAndroid();
    });
```

- 通过 WebViewClient 的 shouldOverrideUrlLoading ()方法回调拦截 url
  webview 中的 JS 发起 schema 请求，然后安卓端拦截到这个请求。

```
document.getElementById("btn1").addEventListener('click',()=>{
        /*约定的url协议为：js://webview?arg1=111&arg2=222*/
        document.location = "js://webview?arg1=111&arg2=222";
});
```

```
mWebView.setWebViewClient(new WebViewClient() {
            @Override
            public boolean shouldOverrideUrlLoading(WebView view, String url) {
                Uri uri = Uri.parse(url);
                if (uri.getScheme().equals("js")) {
                    if (uri.getAuthority().equals("webview")) {
                        HashMap<String, String> params = new HashMap<>();
                        Set<String> collection = uri.getQueryParameterNames();
                        System.out.println("js调用了Android的方法");
                        System.out.println("js调用了Android的方法:" + uri.toString());
                    }
                    return true;
                }
                return super.shouldOverrideUrlLoading(view, url);
            }
        });
```

- 通过 WebChromeClient 的 onJsAlert()、onJsConfirm()、onJsPrompt（）方法回调拦截 JS 对话框 alert()、confirm()、prompt（） 消息
  webview 中调用函数 prompt，然后安卓端拦截

```
function callAndroid(){
        var result=prompt("js://demo?arg1=111&arg2=222");
        alert("demo " + result);
    }

    let btn1 = document.getElementById("btn1");

    btn1.addEventListener('click',()=>{
        callAndroid();
    });
```

安卓端拦截请求
原理类似 也可以拦截到请求

```
   mWebView.setWebChromeClient(new WebChromeClient() {
            @Override
            public boolean onJsPrompt(WebView view, String url, String message, String defaultValue, JsPromptResult result) {
                System.out.println("js调用了Android的方法:"+message);
                Uri uri = Uri.parse(message);
                if (uri.getScheme().equals("js")) {
                    if (uri.getAuthority().equals("demo")) {
                        System.out.println("js调用了Android的方法");
                        HashMap<String, String> params = new HashMap<>();
                        Set<String> collection = uri.getQueryParameterNames();
                        result.confirm("js调用了Android的方法成功啦");

                    }
                    return true;
                }
                return super.onJsPrompt(view, url, message, defaultValue, result);
            }

```

参考资料
![https://github.com/wendux/DSBridge-Android/blob/master/readme-chs.md](https://github.com/wendux/DSBridge-Android/blob/master/readme-chs.md)
