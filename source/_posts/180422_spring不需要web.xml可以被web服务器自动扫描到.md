---
title: spring不需要web.xml可以被web服务器自动扫描到
date: 2018-04-22 10:12:20
permalink: spring-doesnot-require-webxml-can-be-automatically-scanned-by-web-server
categories: QA
tags:
 - FAQ
 - corn
 - spring
---

## 回顾

回顾下最开始使用 `SpringWeb` 的时候,那时候到处是 `spring-xxx.xml`。最为熟悉的不过于 `web.xml` 对 `SpringWeb` 启动的配置如下:

```xml
   <servlet>
     <servlet-name>chapter2</servlet-name>
     <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
     <load-on-startup>1</load-on-startup>
     <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:spring-config.xml</param-value>
     </init-param>
   </servlet>
```

```xml
   <listener>      
     <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
   </listener>
```


- `listener` 下的 `ContextLoaderListener` 是一种引入方式，默认读取 `/WEB-INF/applicationContext.xml` 实际上 `ContextLoaderListener` 实现了 `javax.servlet.ServletContextListener`,让 `web` 服务扫描到。`ContextLoaderListener` 的  `contextInitialized` 方法就会被调用然后启动 读取 `Spring` 相关的文件,解析加载相关的内容。
- 若是 `spring-web` 项目，`DispatcherServlet` 也是一种引入方式，默认读取 `/WEB-INF/${servlet-name}-servlet.xml`

```java
package javax.servlet;

import java.util.EventListener;

/**
 * Implementations of this interface receive notifications about changes to the
 * servlet context of the web application they are part of. To receive
 * notification events, the implementation class must be configured in the
 * deployment descriptor for the web application.
 *
 * @see ServletContextEvent
 * @since v 2.3
 */

public interface ServletContextListener extends EventListener {

    /**
     ** Notification that the web application initialization process is starting.
     * All ServletContextListeners are notified of context initialization before
     * any filter or servlet in the web application is initialized.
     * @param sce Information about the ServletContext that was initialized
     */
    public void contextInitialized(ServletContextEvent sce);

    /**
     ** Notification that the servlet context is about to be shut down. All
     * servlets and filters have been destroy()ed before any
     * ServletContextListeners are notified of context destruction.
     * @param sce Information about the ServletContext that was destroyed
     */
    public void contextDestroyed(ServletContextEvent sce);
}
```


## 问题

后面发现不使用 `web.xml` 也能够加载 `Spring`,并且没法发现显示调用 `Spring` 相关的启动类。

`org.springframework.web.servlet.support.AbstractDispatcherServletInitializer`

```java
   /**
   * Base class for {@link org.springframework.web.WebApplicationInitializer}
   * implementations that register a {@link DispatcherServlet} in the servlet context.
   *
   * <p>Concrete implementations are required to implement
   * {@link #createServletApplicationContext()}, as well as {@link #getServletMappings()},
   * both of which get invoked from {@link #registerDispatcherServlet(ServletContext)}.
   * Further customization can be achieved by overriding
   * {@link #customizeRegistration(ServletRegistration.Dynamic)}.
   *
   * <p>Because this class extends from {@link AbstractContextLoaderInitializer}, concrete
   * implementations are also required to implement {@link #createRootApplicationContext()}
   * to set up a parent "<strong>root</strong>" application context. If a root context is
   * not desired, implementations can simply return {@code null} in the
   * {@code createRootApplicationContext()} implementation.
   */
  public abstract class AbstractDispatcherServletInitializer extends AbstractContextLoaderInitializer {
```

看注释就知道 这个类的功能和 `ContextLoaderListener` 的功能类似，用来创建上下文。

通过调试我们找到了原始的加载核心(`tomcat` 内核代码)

```java
   /**
    * ServletContainerInitializers (SCIs) are registered via an entry in the
    * file META-INF/services/javax.servlet.ServletContainerInitializer that must be
    * included in the JAR file that contains the SCI implementation.
    * <p>
    * SCI processing is performed regardless of the setting of metadata-complete.
    * SCI processing can be controlled per JAR file via fragment ordering. If an
    * absolute ordering is defined, the only those JARs included in the ordering
    * will be processed for SCIs. To disable SCI processing completely, an empty
    * absolute ordering may be defined.
    * <p>
    * SCIs register an interest in annotations (class, method or field) and/or
    * types via the {@link javax.servlet.annotation.HandlesTypes} annotation which
    * is added to the class.
    *
    * @since Servlet 3.0
    */
    public interface ServletContainerInitializer {

        /**
        * Receives notification during startup of a web application of the classes
        * within the web application that matched the criteria defined via the
        * {@link javax.servlet.annotation.HandlesTypes} annotation.
        *
        * @param c     The (possibly null) set of classes that met the specified
        *              criteria
        * @param ctx   The ServletContext of the web application in which the
        *              classes were discovered
        *
        * @throws ServletException If an error occurs
        */
        void onStartup(Set<Class<?>> c, ServletContext ctx) throws ServletException;
    } 
```

从注释看我们知道是要去加载 `jar` 下面的 `META-INF/services/javax.servlet.ServletContainerInitializer` 文件，让我们打开 `spring-web-x.x.x.jar` 能找到这个文件,里面的内容是 `org.springframework.web.SpringServletContainerInitializer` 也就是 `springweb` 的初始加载核心。`tomcat` 启动回去扫描相关的类，启动 `SpringWeb`。

## 扩展: 文件加载

文件结构
 
`src/main/java/Test.java`
`src/resource/META-INF/services/javax.servlet.ServletContainerInitializer`

```java
   import java.net.URL;
   import java.util.Enumeration;

   public class Test {

      public static void main(String[] args) {
          ClassLoader classLoader = Test.class.getClassLoader();
          URL url = classLoader.getResource("META-INF/services/javax.servlet.ServletContainerInitializer");
          System.out.println(url);
          System.out.println("==================================");
          try {
              Enumeration<URL> enumeration = classLoader.getResources("META-INF/services/javax.servlet.ServletContainerInitializer");
              while (enumeration.hasMoreElements()) {
                  System.out.println(enumeration.nextElement());
              }
          } catch (Exception e) {
              e.printStackTrace();
          }

      }
  }
```

打印结果:

```
file:/E:/comp_code_workspace/demo/target/classes/META-INF/services/javax.servlet.ServletContainerInitializer
==================================
file:/E:/comp_code_workspace/demo/target/classes/META-INF/services/javax.servlet.ServletContainerInitializer
jar:file:/D:/mvn_repository/org/apache/tomcat/embed/tomcat-embed-websocket/8.5.15/tomcat-embed-websocket-8.5.15.jar!/META-INF/services/javax.servlet.ServletContainerInitializer
jar:file:/D:/mvn_repository/org/springframework/spring-web/4.3.9.RELEASE/spring-web-4.3.9.RELEASE.jar!/META-INF/services/javax.servlet.ServletContainerInitializer
jar:file:/D:/mvn_repository/ch/qos/logback/logback-classic/1.1.11/logback-classic-1.1.11.jar!/META-INF/services/javax.servlet.ServletContainerInitializer
```

这个设计的好处出非常多，常用的是：某个框架提供一个约定的配置形式，有利于开发者扩展。目前 `Spring` 相关插件就是依据这个而来，最常见的是 `META-INF/spring.handlers` 和 `META-INF/spring.schemas`(虽然这个是比较老的 `xml`解析相关)。