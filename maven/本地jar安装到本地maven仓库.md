#### 1，maven jar管理：本地jar加入本地maven仓库、maven私服

#####1.1，把本地的jar安装到本地mav仓库

1，配置mvn环境变量

~~~she
D:\BC\apache-maven-3.3.9\bin
~~~

2，定位到本地jar目录下，执行mvn命令

~~~shel
mvn install:install-file -Dfile=cpkapi-2.0.8.jar -DgroupId=com.bochtech -DartifactId=cpkapi -Dversion=2.0.8 -Dpackaging=jar
~~~



其中：-DgroupId和-DartifactId的作用是指定了这个jar包在repository的安装路径，只是用来告诉项目去这个路径下寻找这个名称的jar包。

步骤2执行后就是指把cpkapi-2.0.8.jar安装到D:\.m2\repo\com\bochtech\cpkapi\2.0.8目录下，执行完命令后，如果需要在项目中使用这个jar，则在pom.xml中添加如下配置即可：

~~~xml
<!-- cpk api -->
        <dependency>
            <groupId>com.bochtech</groupId>
            <artifactId>cpkapi</artifactId>
            <version>2.0.8</version>
        </dependency>
~~~



##### 1.2，把本地jar上传到maven私服

前提是已经安装好了maven私服：Nexus

浏览器访问maven私服

~~~http
http://192.168.1.201:8081/nexus
~~~

右上角登录：默认用户名密码为admin/admin123

然后按图操作：点击左侧 Repositories

本地第三方jar一般放到 3r party仓库下

![20171030103625201](E:\note\maven\pic\20171030103625201.png)



点击3rd party，可以查看该仓库下的所有包Browser Storage， 点击Artifact Upload上传包

![20171030101058439](E:\note\maven\pic\20171030101058439.png)



![20171030103725977](E:\note\maven\pic\20171030103725977.png)



上传后可以查看到包

![20180608093214](E:\note\maven\pic\20180608093214.png)



在本地项目pom.xml 中加入

```
<dependency>
    <groupId>com.bcpkix</groupId>
    <artifactId>bcpkix</artifactId>
    <version>1.0</version>
</dependency>
```

同步maven，就可以看到私服上的包同步到本地maven仓库，并且本地项目可以访问到了