---

title: Hello World！——内嵌Jetty
date: 2018-11-27 00:02:53
tags: [Hello World,Jetty,内嵌Jetty,]
categories: Hello World系列

---


Hello World！——内嵌Jetty
------------------------

[TOC]

Jetty是一个开源的轻量级Servlet容器，提供JSP和servlet运行环境。Jetty是纯Java编写的，可以直接使用JAR包方式启动，使用简单，架构也简单。

Jetty从设计之初就考虑作为组件提供，口号是"Don’t deploy your application in Jetty, deploy Jetty in your application!"(不在Jetty中部署你的应用，在你的应用中部署Jetty)。

Jetty的组件划分比较清晰， [Jetty源码](https://github.com/eclipse/jetty.project)挂在github上,有兴趣的可以下下来看看。 


## 一、Jetty提供的功能

Jetty托管在Eclipse基金会。Jetty Web Server可以作为Http服务器和Servlet容器使用， 提供静态和动态页面服务，可以独立部署也可以嵌入式使用。 

Jetty项目中，组合各组件可以提供以下功能：
- 异步Http服务器(Asynchronous HTTP Server)
- 标准Servelt容器(Standards based Servlet Container)
- websocket服务器(websocket server)
- Http/2服务器(http/2 server)
- 异步客户端(Asynchronous Client (http/1.1, http/2, websocket))
- 提供OSGI,JNDI,JMX,JASPI,AJP支持(OSGI, JNDI, JMX, JASPI, AJP support)


## 二、引入Jetty

在应用中内嵌Jetty通过maven的方式引入Jetty的jar包。
```xml
        <dependency>
			<groupId>org.eclipse.jetty</groupId>
			<version>9.4.14.v20181114</version>
			<artifactId>jetty-server</artifactId>
		</dependency>
```

jetty-server的jar包依赖关系如下, Jar包依赖非常简单： 
![jetty-server-dependency-hierarchy](https://raw.githubusercontent.com/run-zheng/my-demo-projects/9211dc989a169ac8775218aa4a983e2d4e68fbbc/hello-world-demo/hello-world-jetty/doc/jetty-server-dependency-hierarchy.png)

## 三、Hello world实例代码

演示内嵌Jetty，只需要创建Server，指定端口(8080)就可以将Jetty内嵌run起来，
但是因为没有处理器，默认的404页面： 

![jetty-server-hello-world-404-default](https://raw.githubusercontent.com/run-zheng/my-demo-projects/b98d523aee305567288c954a6dd0ae3c4a233ddc/hello-world-demo/hello-world-jetty/doc/jetty-server-hello-world-404-default.png)

最简单的是继承AbstractHandler实现一个Handler，处理请求和响应数据： 

```java
import java.io.IOException;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.eclipse.jetty.server.Request;
import org.eclipse.jetty.server.Server;
import org.eclipse.jetty.server.handler.AbstractHandler;

import lombok.extern.slf4j.Slf4j;

@Slf4j
public class JettyHelloWorldDemo {

	@Slf4j
	public static class Handler extends AbstractHandler {
		@Override
		public void handle(String target, Request baseRequest, 
				HttpServletRequest request, HttpServletResponse response)
				throws IOException, ServletException {
			log.info("Handler request start: {}", request.getRequestURL());
			//设置类型，指定编码utf8
			response.setContentType("text/html; charset=utf-8");
			//设置响应状态吗
			response.setStatus(HttpServletResponse.SC_OK);
			//写响应数据
			response.getWriter().write("<h1>Hello world!</h1>");
			//标记请求已处理，handle链
			baseRequest.setHandled(true);
			log.info("Handler request end");
		}
	}

	public static void main(String[] args) {
		//创建服务器
		Server server = new Server(8080);
		try {
			//设置handler
			server.setHandler(new Handler());
			//启动服务器
			server.start();
			//阻塞Jetty server的线程池，直到线程池停止
			server.join();
		} catch (Exception e) {
			log.error(e.getMessage(), e);
		}
	}
}
```

