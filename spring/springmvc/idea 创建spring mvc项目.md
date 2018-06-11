---
typora-root-url: pic
---

#### idea 创建spring mvc项目

##### 1 创建项目



![springmvc_20180611160524](../../../maven/pic/springmvc_20180611160524.png)



![springmvc20180611160604](../../../maven/pic/springmvc20180611160604.png)



![springmcv_20180611160645](../../../maven/pic/springmcv_20180611160645.png)



##### 2 创建Controller文件

src目录创建目录，并创建HelloController.java

~~~java
package com.tomatoman.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class HelloController {

    @RequestMapping("hello")
    public String hello() {
        return "hello";
    }
}
~~~



##### 3 配置xml文件

###### 3.1 web.xml:<url-pattern>/</url-pattern> 修改为拦截所有请求 /



~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/applicationContext.xml</param-value>
    </context-param>
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
~~~



###### 3.2 配置 dispatcher-servlet.xml 

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"

       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"

       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd

       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/mvc
       http://www.springframework.org/schema/mvc/spring-mvc-3.0.xsd">

        <!-- 默认使用基于注释的适配器和映射器 -->
    <mvc:annotation-driven/>
    <!-- 只把动态信息当做controller处理，忽略静态信息 -->
    <mvc:default-servlet-handler/>

    <!-- 注解扫描 -->
    <context:component-scan base-package="com.tomatoman.controller"/>

    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
</beans>
~~~



###### 3.3 创建jsp目录及文件

WEB-INF下创建jsp 目录，并创建文件 hello.jsp

~~~jsp
<%--
  Created by IntelliJ IDEA.
  User: DELL
  Date: 2018/6/11
  Time: 15:41
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
  <head>
    <title>Hello</title>
  </head>
  <body>
  Hello
  </body>
</html>

~~~



##### 4 配置tomcat

![springmvc_20180611162409](../../../maven/pic/springmvc_20180611162409.png)



![springmvc_20180611162621](../../../maven/pic/springmvc_20180611162621.png)



![springmvc20180611162710](../../../maven/pic/springmvc20180611162710.png)



#####5 点击项目构建，在`Problem`中点击`fix`,添加所需的`jar`文件到工程中 

![20161122182640369](../../../maven/pic/20161122182640369.png)



#####6 项目结构

![springmvc_20180611163224](../../../maven/pic/springmvc_20180611163224.png)



##### 7 访问

localhost:8080/hello