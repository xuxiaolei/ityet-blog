---
layout: post
title: OkHttp 拦截器应用
categories: Java
description: OkHttp 拦截器应用
index_img: https://xn--6or.us.kg/api/?raw
date: 2017-12-17 09:09:09
tags: [Java,OkHttp,Interceptor]
---

1.  URL重定向
2.  请求体数据加密
3.  HEAD动态添加
4.  请求日志抓取

#### URL重定向

如何重定向，说白了就是更换个新的url，但是一般服务端做比较好，客户端就显得有些鸡肋。但是这个东西日常也会有用到，比如一些场景，测试生成环境的切换。业务多了，几个人混合开发的后台，每个人的代码不同意导致了baseurl还不同，这时候可以通过一个入口来修改就行，不然每个地方都去修改，接口量大的话会很麻烦。

*   自定义一个Interceptor（TestInterceptor后面都是基于这个来讲解）,直接创建个类实现Interceptor接口即可，然后在Okhttp初始化的是添加这个拦截器即可

``` java
public class TestInterceptor implements Interceptor {
    @Override
    public Response intercept(Chain chain) throws IOException {

        return null;
    }
}


```

然后在你需要初始化的okhttp的地方添加这个拦截器

``` java
public class TestActivity extends AppCompatActivity {
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        OkHttpClient build = new OkHttpClient.Builder()
                .addInterceptor(new TestInterceptor())
                .build();
    }
}


```

那么接下来我们需要干什么呢，拦截器已经写好了，既然定义是拦截器，肯定请求体信息都在那里。我们可以在接口Interceptor里面的方法intercept给了我们一个参数Chain，一切的源头都在这里面。来看看接下来如何进行骚操作来改变初始的URL达到重定向的功能

``` java
public class TestInterceptor implements Interceptor {
    private String newHost = "127.0.0.1";
    @Override
    public Response intercept(Chain chain) throws IOException {
        Request request = chain.request();
        HttpUrl url = request.url();
        //http://127.0.0.1/test/upload/img?userName=xiaoming&userPassword=12345
        String scheme = url.scheme();//  http https
        String host = url.host();//   127.0.0.1
        String path = url.encodedPath();//  /test/upload/img
        String query = url.encodedQuery();//  userName=xiaoming&userPassword=12345

        StringBuffer sb = new StringBuffer();
        String newUrl = sb.append(scheme).append(newHost).append(path).append("?").append(query).toString();

        Request.Builder builder = request.newBuilder()
                .url(newUrl);

        return chain.proceed(builder.build());
    }
}


```

上面只是简单的换了一下host retrofit可以轻易做到，在构造retrofit的时候可以修改baseUrl，但是如果要换path呢，这时候可能retrofit不是那么好做，而且你需要每个地方都要换，这时候你可以在这里当成一个统一的入口new一个新的path即可。简单吧，对于后面的每个字段说明意思我在后面也有备注，可以看到拆分后的样子

#### 请求体数据加密

既然要对请求体加密，那肯定要知道请求体在哪里，然后才能加密，其实都一样不论是加密url里面的query内容还是加密body体里面的都一样，只要拿到了对应的数据我们想怎么做怎么，有的接口比较奇葩，他需要根据请求体的内容进行签名认证。不论如何我们拿到了请求体当然想怎么样就怎么样，看下嘛的操作

1.  加密query内容

``` java
public class TestInterceptor implements Interceptor {
    private String newHost = "127.0.0.1";

    @Override
    public Response intercept(Chain chain) throws IOException {
        Request request = chain.request();
        HttpUrl url = request.url();

        //http://127.0.0.1/test/upload/img?userName=xiaoming&userPassword=12345
        String scheme = url.scheme();//  http https
        String host = url.host();//   127.0.0.1
        String path = url.encodedPath();//  /test/upload/img
        String query = url.encodedQuery();//  userName=xiaoming&userPassword=12345

        StringBuffer sb = new StringBuffer();
        sb.append(scheme).append(newHost).append(path).append("?");
        Set<String> queryList = url.queryParameterNames();
        Iterator<String> iterator = queryList.iterator();

        for (int i = 0; i < queryList.size(); i++) {

            String queryName = iterator.next();
            sb.append(queryName).append("=");
            String queryKey = url.queryParameter(queryName);
            //对query的key进行加密
            sb.append(CommonUtils.getMD5(queryKey));
            if (iterator.hasNext()) {
                sb.append("&");
            }
        }

        String newUrl = sb.toString();

        Request.Builder builder = request.newBuilder()
                .url(newUrl);

        return chain.proceed(builder.build());
    }
}


```

这样拼接的query内容就可以直接统一加密了。我这里只是简单的md5加密 如果说服务端和客户端定义了一套加密解密协议，就可以在这里进行特定的加密方式进行加密

2.  加密body体内容

``` java

public class TestInterceptor implements Interceptor {
    private String newHost = "127.0.0.1";

    public static String requestBodyToString(RequestBody requestBody) throws IOException {
        Buffer buffer = new Buffer();
        requestBody.writeTo(buffer);
        return buffer.readUtf8();
    }

    @Override
    public Response intercept(Chain chain) throws IOException {
        Request request = chain.request();
        HttpUrl url = request.url();

        //http://127.0.0.1/test/upload/img?userName=xiaoming&userPassword=12345
        String scheme = url.scheme();//  http https
        String host = url.host();//   127.0.0.1
        String path = url.encodedPath();//  /test/upload/img
        String query = url.encodedQuery();//  userName=xiaoming&userPassword=12345

        StringBuffer sb = new StringBuffer();
        sb.append(scheme).append(newHost).append(path).append("?");
        Set<String> queryList = url.queryParameterNames();
        Iterator<String> iterator = queryList.iterator();

        for (int i = 0; i < queryList.size(); i++) {

            String queryName = iterator.next();
            sb.append(queryName).append("=");
            String queryKey = url.queryParameter(queryName);
            //对query的key进行加密
            sb.append(CommonUtils.getMD5(queryKey));
            if (iterator.hasNext()) {
                sb.append("&");
            }
        }

        String newUrl = sb.toString();

        RequestBody body = request.body();
        String bodyToString = requestBodyToString(body);
        TestBean testBean = GsonTools.changeGsonToBean(bodyToString, TestBean.class);
        String userPassword = testBean.getUserPassword();
        //加密body体中的用户密码
        testBean.setUserPassword(CommonUtils.getMD5(userPassword));

        String testGsonString = GsonTools.createGsonString(testBean);
        RequestBody requestBody = RequestBody.create(MediaType.parse("application/json"), testGsonString);

        Request.Builder builder = request.newBuilder()
                .post(requestBody)
                .url(newUrl);

        return chain.proceed(builder.build());
    }
}


```

从上面可以看出我们先拿到body体的内容然后解析后，拿到对应的实体类，然后进行加密，再次创建一个新的body体里面然后post过去，即可达到body体加密，我这只是一种body加密方法，也有可能是拿到body然后进行加密然后对加密后的东西加入head 当成签名使用。

最终这种拦截器方式的加密也是一种统一代码入口的方式。

#### HEAD动态添加

在日常的开发中，可能每个接口对应的header不同有的多有的少。不可能说每个接口都写一个拦截器进行添加头部。这时候我们可以换个思维来考虑这个问题。怎么做呢，其实也是为了统一代码，同一种操作不要再多个地方进行，这样修改起来很麻烦。统一入口，统一出口。

``` java

public class TestInterceptor implements Interceptor {
    private String newHost = "127.0.0.1";
    private String path1 = "/test/upload/img";
    private String path2 = "/test/upload/voice";

    public static String requestBodyToString(RequestBody requestBody) throws IOException {
        Buffer buffer = new Buffer();
        requestBody.writeTo(buffer);
        return buffer.readUtf8();
    }

    @Override
    public Response intercept(Chain chain) throws IOException {
        Request request = chain.request();
        HttpUrl url = request.url();

        //http://127.0.0.1/test/upload/img?userName=xiaoming&userPassword=12345
        String scheme = url.scheme();//  http https
        String host = url.host();//   127.0.0.1
        String path = url.encodedPath();//  /test/upload/img
        String query = url.encodedQuery();//  userName=xiaoming&userPassword=12345

        StringBuffer sb = new StringBuffer();
        sb.append(scheme).append(newHost).append(path).append("?");
        Set<String> queryList = url.queryParameterNames();
        Iterator<String> iterator = queryList.iterator();

        for (int i = 0; i < queryList.size(); i++) {

            String queryName = iterator.next();
            sb.append(queryName).append("=");
            String queryKey = url.queryParameter(queryName);
            //对query的key进行加密
            sb.append(CommonUtils.getMD5(queryKey));
            if (iterator.hasNext()) {
                sb.append("&");
            }
        }

        String newUrl = sb.toString();

        RequestBody body = request.body();
        String bodyToString = requestBodyToString(body);
        TestBean testBean = GsonTools.changeGsonToBean(bodyToString, TestBean.class);
        String userPassword = testBean.getUserPassword();
        //加密body体中的用户密码
        testBean.setUserPassword(CommonUtils.getMD5(userPassword));

        String testGsonString = GsonTools.createGsonString(testBean);
        RequestBody requestBody = RequestBody.create(MediaType.parse("application/json"), testGsonString);

        Request.Builder builder = request.newBuilder()
                .post(requestBody)
                .url(newUrl);

        switch (path) {
            case path1:
                builder.addHeader("token","token");
                break;
            case path2:
                builder.addHeader("token","token");
                builder.addHeader("uid","uid");
                break;
        }

        return chain.proceed(builder.build());
    }
}


```

骚不骚，根据url中path的不同来动态的添加header，其实写代码吗？每个人实现的方式不同，只要路子对，怎么撸都行，看个人。

#### 请求日志抓取

这个其实已经有现成的log拦截器了，大家日常开发中已经有用过，但是如果说我们只想看到我想要的，过滤那些不要的东西。怎么办呢。只能自己来自定义了。把自己需要的东西打印出来。以免太多每次看很乱。这时候怎么通过拦截器完成这样的一个骚操作呢。也很简单，基于上面的基础我们应该该拿到的都拿到了。

```

public class TestInterceptor implements Interceptor {
    private String newHost = "127.0.0.1";
    private String path1 = "/test/upload/img";
    private String path2 = "/test/upload/voice";
    private String TAG = "TestInterceptor";
    public static String requestBodyToString(RequestBody requestBody) throws IOException {
        Buffer buffer = new Buffer();
        requestBody.writeTo(buffer);
        return buffer.readUtf8();
    }

    @Override
    public Response intercept(Chain chain) throws IOException {
        Request request = chain.request();
        Response response = chain.proceed(request);

        HttpUrl url = request.url();
        //http://127.0.0.1/test/upload/img?userName=xiaoming&userPassword=12345
        String scheme = url.scheme();//  http https
        String host = url.host();//   127.0.0.1
        String path = url.encodedPath();//  /test/upload/img
        String query = url.encodedQuery();//  userName=xiaoming&userPassword=12345

        RequestBody body = request.body();
        String bodyToString = requestBodyToString(body);

        Log.e(TAG,scheme);
        Log.e(TAG,host);
        Log.e(TAG,path);
        Log.e(TAG,query);

        if (response != null) {
            ResponseBody responseBody = response.body();
            long contentLength = responseBody.contentLength();
            String bodySize = contentLength != -1 ? contentLength + "-byte" : "unknown-length";

            Log.e(TAG,response.code() + ' '
                    + response.message() + ' '
                    + response.request().url()+' '
                    + bodySize
                 );

            Headers headers = response.headers();
            for (int i = 0, count = headers.size(); i < count; i++) {
                Log.e(TAG,headers.name(i) + ": " + headers.value(i));
            }

        }

        return chain.proceed(request);
    }
}


```