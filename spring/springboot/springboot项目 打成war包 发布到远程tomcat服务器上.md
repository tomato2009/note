#### springboot项目 打成war包 发布到远程tomcat服务器上

IDE：intellij IDEA

参考文章：

~~~http
https://blog.csdn.net/yy226953/article/details/78851341
~~~

#####1 将pom.xml中的打包方式修改为war

~~~xml
	<groupId>com.tomatoman</groupId>
	<artifactId>boot-mvc</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>war</packaging>
~~~

##### 2.在pom.xml中添加依赖，将scope状态修改为provided

```
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
            <scope>provided</scope>
        </dependency>
```



##### 3 修改src/main/java下的application启动项 

~~~java
package com.tomatoman.bootmvc;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.web.servlet.support.SpringBootServletInitializer;
import org.springframework.web.WebApplicationInitializer;
import org.springframework.web.bind.annotation.RequestMapping;

@SpringBootApplication
public class BootMvcApplication extends SpringBootServletInitializer implements WebApplicationInitializer {

	@Override
	protected SpringApplicationBuilder configure(SpringApplicationBuilder application){

		return application.sources(BootMvcApplication.class);
	}

	public static void main(String[] args) {
		SpringApplication.run(BootMvcApplication.class, args);
	}
}
~~~



##### 4 使用maven 出war包

* 先clean，后package
* 在target目录下可以到看到 war包



##### 5 SecureCRT 上传包到远程服务器

* 使用rz命令 
* 上传到tomcat目录webapps目录下

启动tomcat，会自动解压缩war



直接在浏览器远程访问服务：

~~~http
http://ip:port/war包名/其他页面路径
~~~



如spring-boot.war 下有个Controller  RequestMapping("hello")

访问路径为：

~~~http
http://ip:port/spring-boot/hello
~~~

