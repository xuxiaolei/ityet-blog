---
layout: post
title: 实现集团产品统一认证
categories: auth
description: 统一认证
keywords: auth,Java
---
一个公司产品矩阵比较丰富的时候，用户在不同系统之间来回切换，固然对产品用户体验上较差，并且增加用户密码管理成本。

也没有很好地利用内部流量进行用户打通，并且每个产品的独立体系会导致产品安全度下降。

因此实现集团产品的单点登录对用户使用体验以及效率提升有很大的帮助。那么如何实现统一认证呢？我们先了解一下传统的身份验证方式。

## 传统 Session 机制及身份认证方案

### Cookie 与服务器的交互


众所周知，http 是无状态的协议，因此客户每次通过浏览器访问 web。

页面，请求到服务端时，服务器都会新建线程，打开新的会话，而且服务器也不会自动维护客户的上下文信息。

比如我们现在要实现一个电商内的购物车功能，要怎么才能知道哪些购物车请求对应的是来自同一个客户的请求呢？


因此出现了 session 这个概念，session 就是一种保存上下文信息的机制，他是面向用户的，每一个 SessionID 对应着一个用户，并且保存在服务端中。

session 主要以 cookie 或 URL 重写为基础的来实现的，默认使用 cookie 来实现，系统会创造一个名为 JSESSIONID 的变量输出到 cookie 中。

JSESSIONID 是存储于浏览器内存中的，并不是写到硬盘上的，如果我们把浏览器的cookie 禁止，则 web 服务器会采用 URL 重写的方式传递 Sessionid，我们就可以在地址栏看到 sessionid=KWJHUG6JJM65HS2K6 之类的字符串。

通常 JSESSIONID 是不能跨窗口使用的，当你新开了一个浏览器窗口进入相同页面时，系统会赋予你一个新的 sessionid，这样我们信息共享的目的就达不到了。

### 服务器端的 session 的机制

当服务端收到客户端的请求时候，首先判断请求里是否包含了 JSESSIONID 的 sessionId，如果存在说明已经创建过了，直接从内存中拿出来使用，如果查询不到，说明是无效的。

如果客户请求不包含 sessionid，则为此客户创建一个 session 并且生成一个与此 session 相关联的 sessionid，这个 sessionid 将在本次响应中返回给客户端保存。

对每次 http 请求，都经历以下步骤处理：

*   服务端首先查找对应的 cookie 的值（sessionid）。

*   根据 sessionid，从服务器端 session 存储中获取对应 id 的 session 数据，进行返回。

*   如果找不到 sessionid，服务器端就创建 session，生成 sessionid 对应的 cookie，写入到响应头中。

session 是由服务端生成的，并且以散列表的形式保存在内存中。

### 基于 session 的身份认证流程

基于 seesion 的身份认证主要流程如下：


因为 http 请求是无状态请求，所以在 Web 领域，大部分都是通过这种方式解决。但是这么做有什么问题呢？我们接着看。

## 集群环境下的 Session 困境及解决方案

随着技术的发展，用户流量增大，单个服务器已经不能满足系统的需要了，分布式架构开始流行。


通常都会把系统部署在多台服务器上，通过负载均衡把请求分发到其中的一台服务器上，这样很可能同一个用户的请求被分发到不同的服务器上。

因为 session 是保存在服务器上的，那么很有可能第一次请求访问的 A 服务器，创建了 session，但是第二次访问到了 B 服务器，这时就会出现取不到 session 的情况。

我们知道，Session 一般是用来存会话全局的用户信息（不仅仅是登陆方面的问题），用来简化/加速后续的业务请求。

传统的 session 由服务器端生成并存储，当应用进行分布式集群部署的时候，如何保证不同服务器上 session 信息能够共享呢？

### Session 共享方案

Session 共享一般有两种思路：

*   session 复制

*   session 集中存储

##### ①session 复制

session 复制即将不同服务器上 session 数据进行复制，用户登录，修改，注销时，将 session 信息同时也复制到其他机器上面去。


这种实现的问题就是实现成本高，维护难度大，并且会存在延迟登问题。

##### ②session 集中存储

集中存储就是将获取 session 单独放在一个服务中进行存储，所有获取 session 的统一来这个服务中去取。

这样就避免了同步和维护多套 session 的问题。一般我们都是使用 redis 进行集中式存储 session。

## 多服务下的登陆困境及 SSO 方案

### SSO 的产生背景

如果企业做大了之后，一般都有很多的业务支持系统为其提供相应的管理和 IT 服务，按照传统的验证方式访问多系统，每个单独的系统都会有自己的安全体系和身份认证系统。

进入每个系统都需要进行登录，获取 session，再通过 session 访问对应系统资源。

这样的局面不仅给管理上带来了很大的困难，对客户来说也极不友好，那么如何让客户只需登陆一次，就可以进入多个系统，而不需要重新登录呢？


“单点登录”就是专为解决此类问题的。其大致思想流程如下：通过一个 ticket 进行串接各系统间的用户信息。

### SSO 的底层原理 CAS

#### ①CAS 实现单点登录流程

我们知道对于完全不同域名的系统，cookie 是无法跨域名共享的，因此 sessionId 在页面端也无法共享，因此需要实现单店登录，就需要启用一个专门用来登录的域名如（ouath.com）来提供所有系统的 sessionId。

当业务系统被打开时，借助中心授权系统进行登录，整体流程如下：

*   当 b.com 打开时，发现自己未登陆，于是跳转到 ouath.com 去登陆

*   ouath.com 登陆页面被打开，用户输入帐户/密码登陆成功

*   ouath.com 登陆成功，种 cookie 到 ouath.com 域名下

*   把 sessionid 放入后台 redis，存放<ticket，sesssionid>数据结构，然后页面重定向到 A 系统

*   当 b.com 重新被打开，发现仍然是未登陆，但是有了一个 ticket 值

*   当 b.com 用 ticket 值，到 redis 里查到 sessionid，并做 session 同步，然后种 cookie 给自己，页面原地重定向

*   当 b.com 打开自己页面，此时有了 cookie，后台校验登陆状态，成功

整个交互流程图如下：


#### ②单点登录流程演示

##### CAS 登录服务 demo 核心代码如下：

用户实体类：

```java
public class UserForm implements Serializable{
private static final long serialVersionUID = 1L;

private String username;
private String password;
private String backurl;

public String getUsername() {
return username;
}

public void setUsername(String username) {
this.username = username;
}

public String getPassword() {
return password;
}

public void setPassword(String password) {
this.password = password;
}

public String getBackurl() {
return backurl;
}

public void setBackurl(String backurl) {
this.backurl = backurl;
}

}
```

登录控制器：

```java
@Controller
public class IndexController {
@Autowired
private RedisTemplate redisTemplate;

@GetMapping("/toLogin")
public String toLogin(Model model,HttpServletRequest request) {
Object userInfo = request.getSession().getAttribute(LoginFilter.USER_INFO);
//不为空，则是已登陆状态
if (null != userInfo){
String ticket = UUID.randomUUID().toString();
redisTemplate.opsForValue().set(ticket,userInfo,2, TimeUnit.SECONDS);
return "redirect:"+request.getParameter("url")+"?ticket="+ticket;
}
UserForm user = new UserForm();
user.setUsername("laowang");
user.setPassword("laowang");
user.setBackurl(request.getParameter("url"));
model.addAttribute("user", user);

return "login";
}

@PostMapping("/login")
public void login(@ModelAttribute UserForm user,HttpServletRequest request,HttpServletResponse response) throws IOException, ServletException {
System.out.println("backurl:"+user.getBackurl());
request.getSession().setAttribute(LoginFilter.USER_INFO,user);

//登陆成功，创建用户信息票据
String ticket = UUID.randomUUID().toString();
redisTemplate.opsForValue().set(ticket,user,20, TimeUnit.SECONDS);
//重定向，回原url  ---a.com
if (null == user.getBackurl() || user.getBackurl().length()==0){
response.sendRedirect("/index");
} else {
response.sendRedirect(user.getBackurl()+"?ticket="+ticket);
}
}

@GetMapping("/index")
public ModelAndView index(HttpServletRequest request) {
ModelAndView modelAndView = new ModelAndView();
Object user = request.getSession().getAttribute(LoginFilter.USER_INFO);
UserForm userInfo = (UserForm) user;
modelAndView.setViewName("index");
modelAndView.addObject("user", userInfo);
request.getSession().setAttribute("test","123");
return modelAndView;
}
}
```

登录过滤器：

```java
public class LoginFilter implements Filter {
public static final String USER_INFO = "user";
@Override
public void init(FilterConfig filterConfig) throws ServletException {

}

@Override
public void doFilter(ServletRequest servletRequest,
ServletResponse servletResponse, FilterChain filterChain)
throws IOException, ServletException {

HttpServletRequest request = (HttpServletRequest) servletRequest;
HttpServletResponse response = (HttpServletResponse)servletResponse;

Object userInfo = request.getSession().getAttribute(USER_INFO);;

//如果未登陆，则拒绝请求，转向登陆页面
String requestUrl = request.getServletPath();
if (!"/toLogin".equals(requestUrl)//不是登陆页面
&amp;&amp; !requestUrl.startsWith("/login")//不是去登陆
&amp;&amp; null == userInfo) {//不是登陆状态

request.getRequestDispatcher("/toLogin").forward(request,response);
return ;
}

filterChain.doFilter(request,servletResponse);
}

@Override
public void destroy() {

}
}
```

配置过滤器：

```java
@Configuration
public class LoginConfig {

//配置filter生效
@Bean
public FilterRegistrationBean sessionFilterRegistration() {

FilterRegistrationBean registration = new FilterRegistrationBean();
registration.setFilter(new LoginFilter());
registration.addUrlPatterns("/*");
registration.addInitParameter("paramName", "paramValue");
registration.setName("sessionFilter");
registration.setOrder(1);
return registration;
}
}
```

登录页面：

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>enjoy login</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
<div text-align="center">
    <h1>请登陆</h1>
    <form action="#" th:action="@{/login}" th:object="${user}" method="post">
        <p>用户名: <input type="text" th:field="*{username}" /></p>
        <p>密  码: <input type="text" th:field="*{password}" /></p>
        <p><input type="submit" value="Submit" /> <input type="reset" value="Reset" /></p>
        <input type="text" th:field="*{backurl}" hidden="hidden" />
    </form>
</div>

</body>
</html>
```

##### web 系统 demo 核心代码如下：

过滤器：

```java
public class SSOFilter implements Filter {
private RedisTemplate redisTemplate;

public static final String USER_INFO = "user";

public SSOFilter(RedisTemplate redisTemplate){
this.redisTemplate = redisTemplate;
}
@Override
public void init(FilterConfig filterConfig) throws ServletException {

}

@Override
public void doFilter(ServletRequest servletRequest,
ServletResponse servletResponse, FilterChain filterChain)
throws IOException, ServletException {

HttpServletRequest request = (HttpServletRequest) servletRequest;
HttpServletResponse response = (HttpServletResponse)servletResponse;

Object userInfo = request.getSession().getAttribute(USER_INFO);;

//如果未登陆，则拒绝请求，转向登陆页面
String requestUrl = request.getServletPath();
if (!"/toLogin".equals(requestUrl)//不是登陆页面
&amp;&amp; !requestUrl.startsWith("/login")//不是去登陆
&amp;&amp; null == userInfo) {//不是登陆状态

String ticket = request.getParameter("ticket");
//有票据,则使用票据去尝试拿取用户信息
if (null != ticket){
userInfo = redisTemplate.opsForValue().get(ticket);
}
//无法得到用户信息，则去登陆页面
if (null == userInfo){
response.sendRedirect("http://127.0.0.1:8080/toLogin?url="+request.getRequestURL().toString());
return ;
}

/**
* 将用户信息，加载进session中
*/
UserForm user = (UserForm) userInfo;
request.getSession().setAttribute(SSOFilter.USER_INFO,user);
redisTemplate.delete(ticket);
}

filterChain.doFilter(request,servletResponse);
}

@Override
public void destroy() {

}
}
```

控制器：

```java
@Controller
public class IndexController {
@Autowired
private RedisTemplate redisTemplate;

@GetMapping("/index")
public ModelAndView index(HttpServletRequest request) {
ModelAndView modelAndView = new ModelAndView();
Object userInfo = request.getSession().getAttribute(SSOFilter.USER_INFO);
UserForm user = (UserForm) userInfo;
modelAndView.setViewName("index");
modelAndView.addObject("user", user);

request.getSession().setAttribute("test","123");
return modelAndView;
}
}
```

首页：

```HTML
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>enjoy index</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
<div th:object="${user}">
    <h1>cas-website：欢迎你"></h1>
</div>
</body>
</html>
```

#### ③CAS 的单点登录和 OAuth2 的区别

**OAuth2：** 三方授权协议，允许用户在不提供账号密码的情况下，通过信任的应用进行授权，使其客户端可以访问权限范围内的资源。

**CAS：** 中央认证服务（Central Authentication Service），一个基于 Kerberos 票据方式实现 SSO 单点登录的框架，为 Web 应用系统提供一种可靠的单点登录解决方法（属于 Web SSO ）。

CAS 的单点登录时保障客户端的用户资源的安全 ；OAuth2 则是保障服务端的用户资源的安全 。

CAS 客户端要获取的最终信息是，这个用户到底有没有权限访问我（CAS 客户端）的资源；OAuth2 获取的最终信息是，我（oauth2 服务提供方）的用户的资源到底能不能让你（oauth2 的客户端）访问。

因此，需要统一的账号密码进行身份认证，用 CAS；需要授权第三方服务使用我方资源，使用 OAuth2。