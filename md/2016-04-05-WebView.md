###WebView的基本用法
1. `AndroidManifest.xml` 添加网络访问权限 `<uses-permission android:name="android.permission.INTERNET" /> `

2. XML添加一个 `<WebView />`,Java 中`WebView webView = (WebView) findViewById(R.id.webView);`
3. 然后加载网页，

	```
	//加载网页
	webView.loadUrl("http://www.douban.com");
	
	//加载本地网页
	webView.loadUrl("file:///android_asset/test.html")；
	
	//加载字符串
	String htmlString = "<p>This is HTML text</p>";
    webView.loadData(htmlString, "text/html", "utf-8");
	```
根据XX不同，分为WebViewClient、WebChromeClient、WebSettings。WebViewClient处理各种通知、请求事件;WebChromeClient辅助处理Javascript的对话框、加载进度等；WebSettings负责Settings，像缓存等，此处只简单介绍不按上文结构。



### shouldOverrideUrlLoading()

初次 使用上面的loadUrl时会调用，`return true`则在当前webview里处理页面(单页面)。
    
    
    
    @Override
    public boolean shouldOverrideUrlLoading(WebView view, String url) {
        view.loadUrl(url);
        return true;
    }
    
          
还可以根据后缀名过滤apk，自己处理下载等事件。


###JS调用Android


启用JS
```webView.getSettings().setJavaScriptEnabled(true);```

添加JS接口:
```webView.addJavascriptInterface(new JsInterface(this), "AndroidWebView");```

重写：

```
    private class JsInterface {
        private Context mContext;
        public JsInterface(Context context) {
            this.mContext = context;
        }
        //在js中调用window.AndroidWebView.putTokentoJs()触发此方法。
        //4.2+ 需要加下面这行
        @JavascriptInterface
        public String putTokentoJs() {
            return token;
        }
    }
```    
###Android调用JS
4.4以上与上文类似，4.3及以下略不同：
[Android中Java和JavaScript交互](http://droidyue.com/blog/2014/09/20/interaction-between-java-and-javascript-in-android/)

####onJSAlert、onJsPrompt、onJSConfirm

```
@Override
public boolean onJsAlert(WebView view, String url, String message, JsResult result) {
    //balabalabala
    new AlertDialog.Builder(getActivity()).setMessage(message).setNegativeButton("YES", null).show();
    //result.confirm();
    return true;
}
```
另外logcat也可以打印console，不过碰到[object Object]就无能为力了，不过可以用下面的Chrome远程调试。
###Chrome远程调试WebView

1. Chrome32+；USB线连接电脑和Android手机；Android4.4+ 安装了Chrome；
2. 在手机中开启开发者模式（4.4+点七次系统版本号），开启 USB调试；
3. 在电脑端打开```chrome://inspect```，选中 发现USB硬件 ；
4. 开启WebView调试还要在App下添加下面这句：
   
    ```
	if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
	    WebView.setWebContentsDebuggingEnabled(true);
    }
    ```
    
5. 发现硬件后inspect 就可以像调试普通网页一样调试Android中的WebView了。

其他用法见链接，此处只说WebView：

[Remote Debugging Devices](https://developers.google.com/web/tools/chrome-devtools/debug/remote-debugging/remote-debugging)

[Remote Debugging on Android with Chrome](https://developer.chrome.com/devtools/docs/remote-debugging)


###WebSettings Webkit中的实现
[Android WebView 开发详解(三)](http://blog.csdn.net/typename/article/details/40302351)

###WevView缓存
[Android webView 缓存 Cache + HTML5离线功能](http://blog.csdn.net/moubenmao_jun/article/details/9076917)


###重写返回键：goBack()和再按一次退出


```
    private boolean mIsExit = false;
    
    @Override
    public boolean onKeyDown(int keyCode, KeyEvent event) {
    	//如果是KEYCODE_BACK 且 canGoBack()就goBack()
        if ((keyCode == KeyEvent.KEYCODE_BACK) && webView.canGoBack()) {
            webView.goBack();
            toolbar.setVisibility(View.GONE);
            bottomBar.setVisibility(View.VISIBLE);
            mReward.setVisibility(View.GONE);
            return true;
        } else if ((keyCode == KeyEvent.KEYCODE_BACK) && !mIsExit) {
        //如果只是KEYCODE_BACK则提示是否退出，1000内再按就退出 
            mIsExit = true;
            new Handler().postAtTime(new Runnable() {
                @Override
                public void run() {
                 mIsExit = false;
                }
            }, SystemClock.uptimeMillis() + 1000);
            	Toast.makeText(this, "再按一次退出程序", Toast.LENGTH_SHORT).show();
            	return false;
        }
        finish();
        return super.onKeyDown(keyCode, event);
    }
```
###项目中的使用
除了上面提到的，下面几个也用在了项目中：

- `onProgressChanged` 此处根据加载进度写一个动画效果。
- `onReceivedTitle` 据此用来作为Toolbar的标题或者后续的使用。
- `webview.reload()` 评论成功后刷新。
- `onReceivedError()` 用来处理错误，比如404替换成错误页。
- `onPageStarted()` 在这里用`getUrl())`,从中过滤出文章ID之类的，同时根据`getUrl().length()`判断是主页还是其他页面，从而决定Toolbar的显示样式。

- 从`getUrl()`过滤出文章id：
  
  ```               
    String[] parts = url.split("newsId=");
    parts = parts[1].split("&token");
    docid = parts[0];                
  ```





####参考连接
[WebView - Android SDK | Android Developers](http://androiddoc.qiniudn.com/reference/android/webkit/WebView.html)

[7.5.1 WebView(网页视图)基本用法](http://www.runoob.com/w3cnote/android-tutorial-webview.html)

