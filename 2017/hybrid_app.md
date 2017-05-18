> 主讲人：吴彬


要学习某个东西之前，我们首先要了解这个东西是什么？然后我们要了解这东西有什么用,有什么好处和弊端?最后我们要知道这东西怎么用?
简单点就是 ------是什么？有什么用?怎么用?
那么进入正题

# 一、什么是Hybrid 开发?

Hybrid App开发（混合模式移动应用开发）是指开发介于web-app、native-app这两者之间的app，这种app兼具“Native App良好用户交互体验的优势”和“Web App跨平台开发的优势”。


# 二、存在即合理，Hybrid开发的意义

首先，我们看下Hybrid app和web-app、native-app之间的一些区别


可见Hybrid app开发相比于其他开发还是好处多多的，那么它相比于Native开发的好处有哪些呢?

使用Native开发的方式人员要求高，只是一个简单的功能就需要iOS程序员和Android程序员各自完成；

使用Native开发的方式版本迭代周期慢，每次完成版本升级之后都需要上传到App Store并审核，升级，重新安装等，升级成本高；

使用hybrid开发的方式简单方便，同一套代码既可以在IOS平台使用，也可以在Android平台使用，提高了开发效率与代码的可维护性；

使用hybrid开发的方式升级简单方便，只需要服务器端升级一下就好了，对用户而言完全是透明了，免去了Native升级中的种种不便；

# 三、如何进行Hybrid 开发?

Hybrid 开发可以使用第三方框架(此处不细讲，有兴趣的朋友可以自行了解)，也可以用Android自带的webView来加载H5
那么我们来讲讲android 自带的webView和Js之间的互调吧。

首先，别忘记加网络权限和引入webView.

Native调用JS

在4.4之前，调用的方式：
```java
	//ui线程中运行,因为mWebView是UI控件
 	runOnUiThread(new Runnable() {  
        @Override  
        public void run() {  
	    //缺点:无法取得返回值
            mWebView.loadUrl("javascript: 方法名('参数,需要转为字符串')");  
            Toast.makeText(Activity名.this, "调用方法...", Toast.LENGTH_SHORT).show();  
        }  
	});    
```

4.4以后(包括4.4)，使用以下方式:
```java
	mWebView.evaluateJavascript("javascript: 方法名('参数,需要转为字符串')", new ValueCallback() {
	@Override
	public void onReceiveValue(String value) {
	//这里的value即为对应JS方法的返回值
	}
	});
```
传统的Native调用JS还有个缺点就是不适合传输大量数据(大量数据建议用接口方式获取)。
JS调用Native

Native中通过addJavascriptInterface添加暴露出来的JS桥对象,然后再该对象内部声明对应的API方法。
WebSettings webSettings = mWebView.getSettings();  
 //Android容器允许JS脚本
webSettings.setJavaScriptEnabled(true);
//Android容器设置侨连对象,api17前有可被hacker攻击的漏洞,api17(4.4)之后调用需要在调用方法加入加入@JavascriptInterface注解，
//如果代码无此申明，那么也就无法使得js生效，也就是说这样就可以避免恶意网页利用js对安卓客户端的窃取和攻击。
mWebView.addJavascriptInterface(this, "JSBridge");


首先看下这个方法
public void addJavascriptInterface(Object object, String name)
我是在Activity里 写的这段代码，那么这里第一个参数传入this，即Activity实例,第二个参数则是给第一个参数起的别名,方便JS里调用。

然后在Activity里暴露两个方法
```java
		//在Android4.2以上(api17后),暴露的api要加上注解@JavascriptInterface，否则会找不到方法
        @JavascriptInterface
        public String getStudentName(){  
            return "张三";  
        }  

        @JavascriptInterface
        public String getMyName(final String name){  
            return "my name is:" + param;  
        }
```
 然后我们在HTML里调用Native方法
```java
	//调用方法一
	window.JSBridge.getStudentName(); //返回:'张三'
	//调用方法二
	window.JSBridge.getMyName('韩梅梅');//返回:'my name is 韩梅梅'
```
OK,以上就是我们传统的调用方式，但是显而易见，传统的调用存在许多的不足，所以出现了许多好用的第三方框架，那么接下来，我们讲解一个常用框架JSBridge

# 四、JSBridge

OK，让我们提出我们的疑问，什么是JSBridge?JSBridge有什么好处(相比于传统的Native和JS互调)?JSBridge是怎么设计出来的呢？JSBridge怎么使用呢?
### 1、什么是JSBridge?

JSBridge就是js和Native之前的桥梁，是用来定义Native和JS通信的,Native只通过一个固定的桥对象调用JS,JS也只通过固定的桥对象调用Native。
### 2、为什么使用JSBridge？

Android4.2以下,addJavascriptInterface方式有安全漏掉
iOS7以下,JS无法调用Native
JSBridge有一套现有的成熟方案,可以完美兼容各种版本，对以前老版本技术的兼容。

### 3、JSBridge的原理与实现

我们知道，Native要调用JS是非常简单的，只要使用WebView.loadUrl(“JavaScript:function()”)即可。那么JS怎么调Native呢？很明显，上面讲的已经行不通了，毕竟有兼容性问题和安全问题。那么怎么办呢？我们得寻找一个突破口吧，大家仔细回一下，WebView有一个方法，叫setWebChromeClient，可以设置WebChromeClient对象，而这个对象中有三个方法，分别是onJsAlert,onJsConfirm,onJsPrompt，当js调用window对象的对应的方法，即window.alert，window.confirm，window.prompt，WebChromeClient对象中的三个方法对应的就会被触发，我们是不是可以利用这个机制，自己做一些处理？答案是肯定的。
至于js这三个方法的区别，可以详见w3c JavaScript 消息框 。一般来说，我们是不会使用onJsAlert的，为什么呢？因为js中alert使用的频率还是非常高的，一旦我们占用了这个通道，alert的正常使用就会受到影响，而confirm和prompt的使用频率相对alert来说，则更低一点。那么到底是选择confirm还是prompt呢，其实confirm的使用频率也是不低的，比如你点一个链接下载一个文件，这时候如果需要弹出一个提示进行确认，点击确认就会下载，点取消便不会下载，类似这种场景还是很多的，因此不能占用confirm。而prompt则不一样，在Android中，几乎不会使用到这个方法，就是用，也会进行自定义，所以我们完全可以使用这个方法。该方法就是弹出一个输入框，然后让你输入，输入完成后返回输入框中的内容。因此，占用prompt是再完美不过了。

OK，找到突破口后，我们就开始实现通信，那么怎么实现呢？既然要通信，那么我们是不是得有通信协议呢？我们可以参考http 的样式http://host:port/path?param=value 来 规定一个属于我们自己的协议:

	jsbridge://className:port/methodName?jsonObj

这样我们Native拿到这串字符串后，通过解析，就可以知道调用哪个类的哪个方法，此处jsonObj用来封装请求的参数。

细心的朋友一点发现了里面有个port，那么这个port是拿来干什么的呢?
其实js层调用native层方法后，native需要将执行结果返回给js层，不过你会觉得通过WebChromeClient对象的onJsPrompt方法将返回值返回给js不就好了吗，其实不然，如果这么做，那么这个过程就是同步的，如果native执行异步操作的话，返回值怎么返回呢？这时候port就发挥了它应有的作用，我们在js中调用native方法的时候，在js中注册一个callback，然后将该callback在指定的位置上缓存起来，然后native层执行完毕对应方法后通过WebView.loadUrl调用js中的方法，回调对应的callback。那么js怎么知道调用哪个callback呢？于是我们需要将callback的一个存储位置传递过去，那么就需要native层调用js中的方法的时候将存储位置回传给js，js再调用对应存储位置上的callback，进行回调。

上面是js向native的通信协议，那么另一方面，native向js的通信协议也需要制定，一个必不可少的元素就是返回值，这个返回值和js的参数做法一样，通过json对象进行传递，该json对象中有状态码code，提示信息msg，以及返回结果result，如果code为非0，则执行过程中发生了错误，错误信息在msg中，返回结果result为null，如果执行成功，返回的json对象在result中。下面是两个例子，一个成功调用，一个调用失败。
```json
	{
    "code":500,
    "msg":"method is not exist",
    "result":null
	}


	{
    "code":0,
    "msg":"ok",
    "result":{
        "key1":"returnValue1",
        "key2":"returnValue2",
        "key3":{
            "nestedKey":"nestedValue"
            "nestedArray":["value1","value2"]
        }
    }
	}
```
Ok,这样我们在native里调用
mWebView.loadUrl("javascript:JSBridge.onFinish(port,jsonObj);");

即可，在调用JS暴露的方法时顺便把port带上，为什么要带port,可以参考上面一段解释。

好了，那么我们可以根据以上的内容先来设计JS了(看注释的重点)
```javascript
	(function (win) {
    var hasOwnProperty = Object.prototype.hasOwnProperty;
    var JSBridge = win.JSBridge || (win.JSBridge = {});
    var JSBRIDGE_PROTOCOL = 'JSBridge';
    var Inner = {
        callbacks: {},
        call: function (obj, method, params, callback) {
            console.log(obj+" "+method+" "+params+" "+callback);
            var port = Util.getPort();
            console.log(port);
            //重点
            this.callbacks[port] = callback;
            var uri=Util.getUri(obj,method,params,port);
            console.log(uri);
            window.prompt(uri, "");
        },
        onFinish: function (port, jsonObj){
           //重点
            var callback = this.callbacks[port];
            callback && callback(jsonObj);
            delete this.callbacks[port];
        },
    };
    var Util = {
        getPort: function () {
            return Math.floor(Math.random() * (1 << 30));
        },
        getUri:function(obj, method, params, port){
            params = this.getParam(params);
            var uri = JSBRIDGE_PROTOCOL + '://' + obj + ':' + port + '/' + method + '?' + params;
            return uri;
        },
        getParam:function(obj){
            if (obj && typeof obj === 'object') {
                return JSON.stringify(obj);
            } else {
                return obj || '';
            }
        }
    };
    for (var key in Inner) {
        if (!hasOwnProperty.call(JSBridge, key)) {
            JSBridge[key] = Inner[key];
        }
    }
	})(window);
```

我们在call()方法里随机生成领域各port后将callback存入callbacks数组中的port位置,根据之前我们定义的协议生成uri,调用window.prompt(uri,"");
我们还需要一个方法来处理js回调，所以有了onFinish方法，方法里面做的事:首先根据port从callbacks中取出callback执行，然后将callbacks的port位置元素删除。

好了，JS这边处理完了，那我们native这边是不是要暴露方法给JS调用呢? 没错，我们需要一个类似android 原生的@javascriptInterface引用，就是暴露给JS调用的方法，所以我们有了如下的方法
JSBridge.register("jsName",javaClass.class)
这个javaClass就是满足某种规范的类，该类中有满足规范的方法，我们规定这个类需要实现一个空接口，为什么呢?主要作用就混淆的时候不会发生错误，还有一个作用就是约束JSBridge.register方法第二个参数必须是该接口的实现类。那么我们定义这个接口
public interface IBridge{
}
类规定好了，类中的方法我们还需要规定，为了调用方便，我们规定类中的方法必须是static的，这样直接根据类而不必新建对象进行调用了（还要是public的），然后该方法不具有返回值，因为返回值我们在回调中返回，既然有回调，参数列表就肯定有一个callback，除了callback，当然还有前文提到的js传来的方法调用所需的参数，是一个json对象，在java层中我们定义成JSONObject对象；方法的执行结果需要通过callback传递回去，而java执行js方法需要一个WebView对象，于是，满足某种规范的方法原型就出来了。
public static void methodName(WebView web view,JSONObject jsonObj,Callback callback){

}

OK，那我们看看这个register方法究竟做了些什么事吧
```java
	public class JSBridge {
    	private static Map<String, HashMap<String, Method>> exposedMethods = new HashMap<>();

    	public static void register(String exposedName, Class<? extends IBridge> clazz) {
        if (!exposedMethods.containsKey(exposedName)) {
            try {
                exposedMethods.put(exposedName, getAllMethod(clazz));
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }

    private static HashMap<String, Method> getAllMethod(Class injectedCls) throws Exception {
        HashMap<String, Method> mMethodsMap = new HashMap<>();
        Method[] methods = injectedCls.getDeclaredMethods();
        for (Method method : methods) {
            String name;
            if (method.getModifiers() != (Modifier.PUBLIC | Modifier.STATIC) || (name = method.getName()) == null) {
                continue;
            }
            Class[] parameters = method.getParameterTypes();
            if (null != parameters && parameters.length == 3) {
                if (parameters[0] == WebView.class && parameters[1] == JSONObject.class && parameters[2] == Callback.class) {
                    mMethodsMap.put(name, method);
                }
            }
        }
        return mMethodsMap;
    }
	}
```
简单来说，我们这个方法做的事就是先判断这个类的别名(此处是jsName)是否存在，不存在的话就往里面添加该类暴露出来的方法(三个参数要满足上面的规范的方法)

好了，我们暴露出去方法了，那么这时候js调用后，从我们的突破口，没错，就是onJsPrompt方法里，我们要做的事就是调用我们的native方法,所以有了以下代码
```java
	public class JSBridgeWebChromeClient extends WebChromeClient {
    	@Override
    	public boolean onJsPrompt(WebView view, String url, String message, String defaultValue, JsPromptResult result) {
       	 	result.confirm(JSBridge.callJava(view, message));
        	return true;
    	}
	}
```
别忘了给webView设置

	mWebView.setWebChromeClient(new JSBridgeWebChromeClient());

我们callJava方法里做了什么事呢?其实就是解析url，查找对应的暴露方法调用
```java
	public static String callJava(WebView webView, String uriString) {
        String methodName = "";
        String className = "";
        String param = "{}";
        String port = "";
        if (!TextUtils.isEmpty(uriString) && uriString.startsWith("JSBridge")) {
            Uri uri = Uri.parse(uriString);
            className = uri.getHost();
            param = uri.getQuery();
            port = uri.getPort() + "";
            String path = uri.getPath();
            if (!TextUtils.isEmpty(path)) {
                methodName = path.replace("/", "");
            }
        }


        if (exposedMethods.containsKey(className)) {
            HashMap<String, Method> methodHashMap = exposedMethods.get(className);
            //重点
            if (methodHashMap != null && methodHashMap.size() != 0 && methodHashMap.containsKey(methodName)) {
                Method method = methodHashMap.get(methodName);
                if (method != null) {
                    try {
                        //重点
                        method.invoke(null, webView, new JSONObject(param), new Callback(webView, port));
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            }
        }
        return null;
    }
```



好了，最后，我们当然是来讲Callback啦，看重点注释就可以了，其实就是回调js的onFinish方法。
```java
	public class Callback {
    	private static Handler mHandler = new Handler(Looper.getMainLooper());
    	//重点
    	private static final String CALLBACK_JS_FORMAT = "javascript:JSBridge.onFinish('%s', %s);";
    	private String mPort;
    	//为了防止内存泄露，这里使用弱引用
    	private WeakReference<WebView> mWebViewRef;

    	public Callback(WebView view, String port) {
        	mWebViewRef = new WeakReference<>(view);
        	mPort = port;
    }


    	public void apply(JSONObject jsonObject) {
        	final String execJs = String.format(CALLBACK_JS_FORMAT, mPort, String.valueOf(jsonObject));
        	if (mWebViewRef != null && mWebViewRef.get() != null) {
            	mHandler.post(new Runnable() {
               		@Override
                	public void run() {
                   	//重点
                    	mWebViewRef.get().loadUrl(execJs);
                	}
            	});

        	}

    	}
	}
```

好了，最后我们来测测吧，也就是我们说的使用
```java
	public class BridgeImpl implements IBridge {
    	public static void showToast(WebView webView, JSONObject param, final Callback callback) {
        	String message = param.optString("msg");
        	Toast.makeText(webView.getContext(), message, Toast.LENGTH_SHORT).show();
	        if (null != callback) {
	            try {
	                JSONObject object = new JSONObject();
	                object.put("key", "value");
	                object.put("key1", "value1");
	                callback.apply(getJSONObject(0, "ok", object));
	            } catch (Exception e) {
	                e.printStackTrace();
	            }
	        }
	    }

	    private static JSONObject getJSONObject(int code, String msg, JSONObject result) {
	        JSONObject object = new JSONObject();
	        try {
	            object.put("code", code);
	            object.put("msg", msg);
	            object.putOpt("result", result);
	            return object;
	        } catch (JSONException e) {
	            e.printStackTrace();
	        }
	        return null;
	    }
	}
```

别忘了注册

	JSBridge.register("bridge", BridgeImpl.class);

JS这边我只放重点的那行代码

	<button onclick="JSBridge.call('bridge','showToast',{'msg':'Hello JSBridge'},function(res){alert(JSON.stringify(res))})">

接着就是使用WebView将该index.html文件load进来测试了

	mWebView.loadUrl("file:///android_asset/index.html");

点击按钮后的结果截图

![](http://i.imgur.com/HFLSNEm.png)

好了，那我们来试试子线程回调，在在BridgeImpl中加入测试方法
```java
	public static void testThread(WebView webView, JSONObject param, final Callback callback) {
	        new Thread(new Runnable() {
	            @Override
	            public void run() {
	                try {
	                    Thread.sleep(3000);
	                    JSONObject object = new JSONObject();
	                    object.put("key", "value");
	                    callback.apply(getJSONObject(0, "ok", object));
	                } catch (InterruptedException e) {
	                    e.printStackTrace();
	                } catch (JSONException e) {
	                    e.printStackTrace();
	                }
	            }
	        }).start();
	    }

```
JS中加入
```javascript

	<button onclick="JSBridge.call('bridge','testThread',{},function(res){alert(JSON.stringify(res))})">
```
点击后，3秒弹出

![](http://i.imgur.com/t3lQyb7.png)

有些框架里面的突破口不在onJsPrompt方法里，在shouldOverrideUrlLoading方法里，不过原理应该差不多。

好了，完结，撒花，放鞭炮，收工！














