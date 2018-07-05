---
layout: post
title:  "JettyWebServer"
categories: niagara
---

# web-WebService

This service encapsulates access to the HTTP server as well as the servlet infrastructure used to expose custom applications over HTTP. Only one WebService is supported in a station.

## BWebServer

___BWebServer___ serves as the base for any class wishing to provide web server capabilities. A web server must reside as a child of a ___BWebService___.

```java
public abstract class BWebServer extends BComponent implements BIService
{
  protected abstract void doStartWebServer() throws Exception;
  protected abstract void doStopWebServer() throws Exception;

  /**
   * Schedule a restart for 500ms from now. Any pending restart is cancelled and a new one is scheduled.
   */
  protected final synchronized void scheduleRestart()
  {
    scheduleRestart(BRelTime.make(500));
  }

  protected synchronized void scheduleRestart(BRelTime delay)
  {
    restartTicket.cancel();
    restartTicket = Clock.schedule(this, delay, restart, null);
  }

  @Override
  public IFuture post(Action action, BValue argument, Context cx)
  {
    if(restart==action && isRunning())
    {
      post(new Invocation(BWebServer.this, action, argument, cx));
      return null
    }
    else
    {
      return super.post(action,argument, cx);
    }
  }

 /**
   * Post some work to the Web Server Config Thread.
   *
   * @param r The work to be done.
   */
  public final Future<?> post(Runnable r)
  {
    if (executorService == null)
    {
      throw new IllegalStateException("Cannot post to Web Server Config " +
          "Thread. Thread not running: " + r.toString());
    }

    return executorService.submit(r);
  }

  ...
    executorService = ExecutorUtil.newSingleThreadBackgroundExecutor("webServerConfig", 1, TimeUnit.MINUTES);
  ...
}
```

### BJettyWebServer

>Eclipse Jetty provides a Web server and javax.servlet container, plus support for HTTP/2, WebSocket, OSGi, JMX, JNDI, JAAS and many other integrations.

基于Jetty的BWebServer实现，创建Server，加载Connectors，加载Handlers，加载Servlets等，最后启动服务start。

```java
public final class BJettyWebServer extends BWebServer {

  public void doStartWebServer() throws Exception {
    ...
  }

  private void configureNiagaraWebApp(ServletContextHandler context) {
    ...
    context.addFilter(AddSubjectFilter.class, "/*", (EnumSet)null);
    context.addFilter(TridiumSecurityFilter.class, "/*", (EnumSet)null);
    context.addFilter(LocaleFilter.class, "/*", (EnumSet)null);
    context.addFilter(ContextFilter.class, "/*", (EnumSet)null);
    AccessController.doPrivileged(() -> {
      context.addServlet(new ServletHolder(new PreloginServlet()), "/prelogin");
      context.addServlet(new ServletHolder(new LoginServlet()), "/login");
      context.addServlet(new ServletHolder(new LoginFileServlet()), "/login/*");
      context.addServlet(new ServletHolder(new LogoutServlet()), "/logout");
      return null;
    });
    context.addServlet(new ServletHolder(new WebStartServlet()), "/webstart/*");
    context.setSessionHandler(new NiagaraSessionHandler());
    context.getSessionHandler().setSessionManager(this.sessManager);
  }
}
```

Jetty限制可以从浏览器或其他客户端回发到服务器的数据量，允许的默认最大值是200000 bytes和1000 keys，可以通过修改system.properties来调整它们的大小：

```text
org.eclipse.jetty.server.Request.maxFormContentSize=250000
org.eclipse.jetty.server.Request.maxFormKeys=2000
```

#### NiagaraLoginService

```java
public UserIdentity login(String username, Object credentials, ServletRequest request) {
  if (username == null) {
    return null;
  } else {
    BUserService userService = (BUserService)Sys.getService(BUserService.TYPE);
    NiagaraHttpSession session = (NiagaraHttpSession)credentials;
    CallbackHandler handler = (CallbackHandler)session.getAttribute("callbackHandler");
    BAuthenticationScheme scheme = (BAuthenticationScheme)session.getAttribute("authenticationScheme");
    BUser user = userService.getUser(username);

    try {
      BAuthenticationService authnService = (BAuthenticationService)Sys.getService(BAuthenticationService.TYPE);
      user = authnService.authenticate(session, user, handler, scheme);
    } catch (AuthenticationException ex) {
      if (ex.getLoginFailureCause() != null) {
        session.setAttribute("loginFailureCause", ex.getLoginFailureCause());
      }

      log.info(ex.toString() + ": most likely a username/password mismatch");
      return null;
    }

    return new NiagaraUserIdentity(user);
  }
}
```

#### LoginService

>The Login service provides an abstract mechanism for an Authenticator to check credentials and to create a UserIdentity using the set IdentityService.

#### NCSA Log

>The Common Log Format, also known as the NCSA Common log format, is a standardized text file format used by web servers when generating server log files.

Field|Description
---|---
Remote host address|The IP address of the client that made the request.
Remote log name|Not used. This value is always a hyphen.
User name|The name of the authenticated user that accessed the server. Anonymous users are indicated by a hyphen. The best practice is for the application always to provide the user name.
Date, time, and Greenwich mean time (GMT) offset|The local date and time at which the activity occurred. The offset from Greenwich mean time is also indicated.
Request and Protocol version|The HTTP protocol version that the client used.
Service status code|The HTTP status code.
Bytes sent|The number of bytes sent by the server.

如果在JettyWebServer -> NCSA Log里配置Extended Format为true，那么会增加下面两个字段：

Field|Description
---|---
Referer|This identifies the address of the webpage (i.e. the URI or IRI) that linked to the resource being requested.
User agent|Most Web browsers use a User-Agent string value as follows: Mozilla/[version] ([system and browser information]) [platform] ([platform details]) [extensions].

启用NCSA Log后默认会在站点的webLogs目录下生成yyyy_mm_dd.request.log文件，参看下面的Log示例：

```log
10.78.148.251 - Admin [04/Jul/2018:10:48:16 +0800] "POST https://10.78.148.251/vrf/SearchStationCascade HTTP/1.1" 200 24 "https://10.78.148.251/vrf/web/machine-mgr.html" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.99 Safari/537.36"
10.78.148.251 - Admin [04/Jul/2018:10:48:16 +0800] "POST https://10.78.148.251/vrf/SearchUserInfo HTTP/1.1" 200 60 "https://10.78.148.251/vrf/web/machine-mgr.html" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.99 Safari/537.36"
```
