---
layout:     post
title:      "IDEA+Maven+Embedded Jetty+Jersey构建Restful服务并打包成jar包发布"
date:  2017-08-05 12:00:00
author:     "孟琦Poet"
tags:
    - Jetty
    - Java
---

# 一、简要介绍
---
最近做的项目用到了嵌入式Jetty当服务器，并用Jersey来构建Restful api，看了老师的项目文件发现还有pom.xml文件，才知道Maven。但因为不是一个组的老师，而且那个老师貌似前端精通的多一点，Maven什么的也不是很了解，从老师那里学的东西也不是很多。因为项目相关，最后还是自己Google各种资料，一点一滴从零开始学习。国内关于嵌入式Jetty的资料真的少，大部分都是翻译官方文档，很难找到完整的案例。因此，想在这里把自己学到的关于Embedded Jetty、Jersy的知识记录下来，希望提能给想入坑的提供点经验，少走点弯路。


# 二、环境需求
---
文中介绍的东西都是基于**JDK1.8**、**Maven3**、**IDEA2017.2**，*Jetty*版本为**9.4.3.v20170317**，*Jersey*版本为**2.25.1**。
# 三、具体步骤
---
## 1、新建maven空项目
打开IDEA，新建一个Project，选择Maven项目，然后填写一些基本信息，最后新建的项目如下图所示
![新建Maven项目](http://upload-images.jianshu.io/upload_images/1826540-7a5065a7863824a3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看到新建的项目除了`/src/main/java`代码包和`/src/main/resources`资源包之外，还多了一个`pom.xml`的文件，Maven项目中对项目的管理配置都在`pom.xml`文件中进行，主要包括添加jar包依赖，对项目进行清理打包构建等等。
## 2、添加辅助框架
右键项目名称，然后点击`Add Framework Support`。
![添加辅助框架](http://upload-images.jianshu.io/upload_images/1826540-942db863da21b0b6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
左侧选中`WebService`，右侧选中`Set up library later`，然后点击`OK`。
![添加网络服务框架](http://upload-images.jianshu.io/upload_images/1826540-68fd9d5da304f8b4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
添加完成后，会在项目的结构树中看到多出`/web`目录，并在`/src/main/java`包中多出一个`example`，红色的`/target`目录是编译文件夹，编译后的class文件都在里面。
![项目结构树](http://upload-images.jianshu.io/upload_images/1826540-802249ad0aa2bc3a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
## 3、添加jar包依赖
我们将`example`包整个删除，然后打开`pom.xml`，添加Rest项目需要的jar包依赖，在Maven项目中，当你需要引入jar包时，只需要在`pom.xml`文件中填上所需jar包的Maven坐标即可，Maven坐标由三部分组成，分别是`groupId`,`artifactId`,`version`，形式如下：
```
         <groupId>org.glassfish.jersey.core</groupId>
         <artifactId>jersey-server</artifactId>
         <version>2.25.1</version>
```
当在Maven项目中引入jar包依赖时，只需要按照如下形式，把坐标放在一个`<dependency></dependency>`标签中即可，多个依赖放在一个`<dependencys></dependencys>`标签中。

本项目中所用到的所有依赖关系如下所示：
```
  <dependencies>
        <dependency>
            <groupId>org.glassfish.jersey.core</groupId>
            <artifactId>jersey-server</artifactId>
            <version>2.25.1</version>
        </dependency>
        <dependency>
            <groupId>org.glassfish.jersey.containers</groupId>
            <artifactId>jersey-container-servlet-core</artifactId>
            <version>2.25.1</version>
        </dependency>
        <dependency>
            <groupId>org.glassfish.jersey.containers</groupId>
            <artifactId>jersey-container-jetty-http</artifactId>
            <version>2.25.1</version>
            <exclusions>
                <exclusion>
                    <groupId>org.eclipse.jetty</groupId>
                    <artifactId>jetty-util</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.glassfish.jersey.media</groupId>
            <artifactId>jersey-media-moxy</artifactId>
            <version>2.25.1</version>
        </dependency>
        <dependency>
            <groupId>org.eclipse.jetty</groupId>
            <artifactId>jetty-server</artifactId>
            <version>9.4.3.v20170317</version>
        </dependency>
        <dependency>
            <groupId>org.eclipse.jetty</groupId>
            <artifactId>jetty-servlet</artifactId>
            <version>9.4.3.v20170317</version>
        </dependency>
        <dependency>
            <groupId>org.eclipse.jetty</groupId>
            <artifactId>jetty-webapp</artifactId>
            <version>9.4.3.v20170317</version>
        </dependency>
        <dependency>
            <groupId>org.glassfish.jersey.media</groupId>
            <artifactId>jersey-media-multipart</artifactId>
            <version>2.25.1</version>
        </dependency>
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-web-api</artifactId>
            <version>7.0</version>
        </dependency>
    </dependencies>
```
## 4、搭建Embedded Jetty服务器
在`/src/main/java`中新建包`com.heu.cs.jettyserver`，在包中新建类`JettyServer`，写入代码
```
package com.heu.cs.jettyserver;

import org.eclipse.jetty.server.*;
import org.eclipse.jetty.server.handler.HandlerCollection;
import org.eclipse.jetty.servlet.DefaultServlet;
import org.eclipse.jetty.servlet.ServletContextHandler;
import org.eclipse.jetty.servlet.ServletHolder;

public class JettyServer {

    public static void main(String[] args) throws Exception {
        Server jettyServer = new Server();
        HttpConfiguration http_config = new HttpConfiguration();
        /**
         *http_config可以对服务器进行配置，比如设置https,BufferSize等等
         *        http_config.setSecureScheme("https");
         *        http_config.setSecurePort(8443);
         *        http_config.setOutputBufferSize(32768);
         *        http_config.setRequestHeaderSize(8192);
         *        http_config.setResponseHeaderSize(8192);
         */
        http_config.setSendServerVersion(true);
        http_config.setSendDateHeader(false);
        
        /**
         * 新建http连接来设置访问端口，超时时间等等。
         */
        ServerConnector httpServer = new ServerConnector(jettyServer,
                new HttpConnectionFactory(http_config));
        httpServer.setPort(7012);
        httpServer.setIdleTimeout(120000);
        jettyServer.addConnector(httpServer);

        /**
         * 设置整个web服务的根url，/ 表示 localhost:7012/  之后地址的是可访问的
         */
        ServletContextHandler context = new ServletContextHandler(ServletContextHandler.SESSIONS);
        context.setContextPath("/");

        /**
         * 添加动态servlet路径，处理我们自己写的动态的servlet
         */
        ServletHolder jerseyServlet = context.addServlet(
                org.glassfish.jersey.servlet.ServletContainer.class, "/webapi/*");
        jerseyServlet.setInitOrder(1);
        // Tells the Jersey Servlet which REST api/class to load.设置动态servlt加载的包
        jerseyServlet.setInitParameter("jersey.config.server.provider.packages", "com.heu.cs.api");
        //也可单独设置加载某个类，
         jerseyServlet.setInitParameter("jersey.config.server.provider.classnames",
                "UploadFileService;org.glassfish.jersey.media.multipart.MultiPartFeature");


        /**
         * 添加默认的servlet路径，处理不在动态servlet路径中的地址，一般都是一些可供访问的静态html,css,js资源文件
         */
        ServletHolder staticServlet = context.addServlet(DefaultServlet.class, "/static/*");
        staticServlet.setInitParameter("resourceBase", "src/main/resources");
        staticServlet.setInitParameter("pathInfoOnly", "true");


        /**
         * Embedded Jetty还可以直接当作服务器，在上面部署已经发布的war包，这方面的资料国内挺多的，就不累述
         * 
         * 其次，在我们的项目中是没有用到web.xml文件来进行webappde的配置，因为上面的设置并不能使得服务器访问web.xml，
         * 如果需要用到web.xml，则需要new一个WebAppContext,并对其进行配置，同时在下面的handlers中加上webAppContext
         *  WebAppContext webAppContext = new WebAppContext();
         *  设置描述符位置
         *  webAppContext.setDescriptor("./web/WEB-INF/web.xml");
         *  设置Web内容上下文路径
         *  webAppContext.setResourceBase("./web");
         *  设置上下文路径
         *  webAppContext.setContextPath("/");
         *  webAppContext.setParentLoaderPriority(true);
         */
        
        HandlerCollection handlers = new HandlerCollection();
        // handlers.setHandlers(new Handler[]{context,webAppContext});
        handlers.setHandlers(new Handler[]{context});
        jettyServer.setHandler(handlers);
        try {
            jettyServer.start();
            jettyServer.join();
        } finally {
            jettyServer.destroy();
        }
    }
}
```
## 5、访问静态资源
至此，一个简单的Embedded Jetty服务器已经搭建完成，当我们需要启动服务器时，只需秩序`JettyServer`中的`main`方法即可。
项目中静态资源的访问根URL是`http://localhost:7012/static/`，我们在`/src/main/resources`目录下新建一些js、css、html文件，然后启动服务器
![静态资源文件](http://upload-images.jianshu.io/upload_images/1826540-d3e2cccccb11d178.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
然后在浏览器中输入`http://localhost:7012/static/`即可访问到这些文件

![访问静态资源](http://upload-images.jianshu.io/upload_images/1826540-4c7c1f55300c1b25.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 6、访问动态Api
我们在前面实现了访问静态资源，如果说想实现访问动态资源，譬如我想访问动态URL路径下的某个Api`http://localhost:7012/webapi/demo`，该如何实现呢？
在`JettyServer`中我们已经设置了动态资源的访问根URL`http://localhost:7012/webapi`，并设置了访问动态资源时加载的包为`com.heu.cs.api`，也就是说，当我们访问动态资源时，服务器会**向这个包里寻找我们要访问的路径及其响应**。
下面我们在这个包里新建一个类名为`Demo`，其中代码如下
```
package com.heu.cs.api;

import javax.servlet.http.HttpServletRequest;
import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.core.Context;
import javax.ws.rs.core.UriInfo;


/**
 * Created by memgq on 2017/8/5.
 */
@Path("demo")
public class Demo {
    @Context
    UriInfo uriInfo;

    @Context
    HttpServletRequest request;

    @GET
    @Produces("text/plain;charset=utf-8")
    public String index() {
        return "这是通过动态servlet生成的内容";
    }

    @GET
    @Path("secondlevel")
    @Produces("text/plain;charset=utf-8")
    public String second() {
        return "rest多级路由";
    }
}
```
在`Demo`这个类中，我们使用到了Jersey**注解开发**，其中`@Path`代表访问路径，`class`上为一级路径，里面可以设置二级路径，`@GET、@POST`为访问方法，`@Produces`表示响应响应回去的类型，关于Jersey常用的注解开发及注解类型，网上资料多尔全面，这里也不累述。
在`Demo`中，我们添加了一个一级URL`/demo`和一个二级URL`/demo/secounlevel`，在前面加上根URL`http://localhost:7012/webapi`，我们即可访问到自己用生成的动态资源

![访问动态资源](http://upload-images.jianshu.io/upload_images/1826540-c2df5888035511fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![访问多级URL](http://upload-images.jianshu.io/upload_images/1826540-03e5c3db8bed52cf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 四、Embedded Jetty项目打包
---
当我们完成webapp开发后，想要将这个项目发布出去，该怎么办？在这里Maven给我们提供了打包插件`maven-shade-plugin`，通过在`pom.xml`文件中对项目进行配置，即可完成打包工作，将项目打包成可执行的jar包发布出去。`pom.xml`中的配置如下

```
<build>
        <finalName>RestDemo</finalName><!-- 打包后的名字-->
    
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>3.0.0</version>

                <configuration>
                    <createDependencyReducedPom>true</createDependencyReducedPom>
                    <filters>
                        <filter>
                            <artifact>*:*</artifact>
                            <excludes>
                                <exclude>META-INF/*.SF</exclude>
                                <exclude>META-INF/*.DSA</exclude>
                                <exclude>META-INF/*.RSA</exclude>
                            </excludes>
                        </filter>
                    </filters>

                </configuration>

                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <transformers>
                                <transformer
                                        implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer" />
                                <transformer
                                        implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <manifestEntries>
                                        <Main-Class>com.heu.cs.jettyserver.JettyServer</Main-Class><!-- 执行主函数-->
                                    </manifestEntries>
                                </transformer>
                            </transformers>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.7</source>
                    <target>1.7</target>
                </configuration>
            </plugin>
        </plugins>
    </build>

```
打包时，鼠标放在IDEA左下角的正方形图标，然后点击Maven Projects

![打开Maven](http://upload-images.jianshu.io/upload_images/1826540-833760594fa3825c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
在IDEA右侧弹出Maven窗口

![Maven](http://upload-images.jianshu.io/upload_images/1826540-ded5c5fc65c50786.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
关闭`JettyServer.main`，然后依次点击Maven中的`clean->install`，最后打包完成的jar包会放在`/target`目录下
![打包后的jar包](http://upload-images.jianshu.io/upload_images/1826540-db8ae64f5d2aa2b9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
然后打开CMD，进入次目录下，键入`java -jar RestDemo.jar`即可执行程序。
![执行jar程序](http://upload-images.jianshu.io/upload_images/1826540-dfe44802364995d9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 注意事项
* 通过这种方法打包的jar包并不包括Resources中的静态资源文件，静态资源文件需要提取出来放到与jar包相同的目录中，打包后仅包含`/src/main/java`中的编译文件。因此在复制静态资源文件时需要按照`/src/main/resources`这样的路径拷贝过来。

# 五、总结
---
我用这个嵌入式Jetty当服务器开发有半年左右，因为基础差，学到的东西不是很多，前期什么都不懂，好多简单的东西都要琢磨很久，走了很多弯路，尤其是打包那一块，对Maven也不熟悉，所以折腾了很久，才找到这个简陋的方法。希望记录下来自己用到的东西，能够帮助到其他人。


