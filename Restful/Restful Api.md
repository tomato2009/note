####Restful Api

参考

~~~http
http://www.runoob.com/w3cnote/restful-architecture.html
~~~



##### 1 Restful 特点

- 在Restful之前的操作：
  - http://127.0.0.1/user/query/1 GET  根据用户id查询用户数据

  - http://127.0.0.1/user/save POST 新增用户、

  - http://127.0.0.1/user/update POST 修改用户信息

  - http://127.0.0.1/user/delete GET/POST 删除用户信息

    ​
- RESTful用法：

  - http://127.0.0.1/user/1 GET  根据用户id查询用户数据
  - http://127.0.0.1/user  POST 新增用户
  - http://127.0.0.1/user  PUT 修改用户信息
  - http://127.0.0.1/user  DELETE 删除用户信息




之前的操作有什么问题呢? 你每次请求的接口或者地址,都在做描述,例如

- 查询的时候用了query
- 新增的时候用了save

其实完全没有这个必要，Restfaul的时候，使用get请求，就是查询，使用post请求，就是新增的请求，我的意图很明显，完全没有必要做描述，这明显是restful的优势

- get请求：查询
- post请求：保存
- put请求：更新
- delete请求：删除

这就完全满足了api的需求



##### 2 使用springboot测试Restful Api

匹配规则看代码注释

~~~java
package com.tomatoman.restfulapi.controller;

import com.tomatoman.restfulapi.pojo.User;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;

/**
 * Spring mvc 对Restful的匹配是精准匹配优先，有点类似于nginx location匹配
 *
 * 客户端测试工具：postman，轻松模拟 get、post、put、delete请求
 */
@Controller
public class RestfulController {

    @RequestMapping("hello")
    @ResponseBody
    private String hello() {
        return "hello";
    }

    /**
     * 任何只有一级path 的请求，都可能被定位到该方法
     * 如访问：
     *  http://localhost:8089/123
     *  http://localhost:8089/ddd
     *
     * 但是如果有一个RequestMapping精准匹配上了，就不会响应此方法，而是调用精准匹配的方法
     * 如访问：http://localhost:8089/hello
     * 只会响应到 @RequestMapping("hello")
     * @param cmd
     * @return
     */
    @RequestMapping("{cmd}")
    @ResponseBody
    private String anything(@PathVariable String cmd) {
        return "{cmd} " + cmd;
    }

    /**
     * http://localhost:8089/123/qqq
     * http://localhost:8089/ddd/rtt
     * 任何只有两级path 的请求，都可能被定位到该方法
     *
     * 但是如果有一个RequestMapping精准匹配上了，就不会响应此方法，而是调用精准匹配的方法
     * 如访问：http://localhost:8089/hello/123
     * 只会响应到 @RequestMapping("hello/{id}")
     * @param cmd
     * @param cmd2
     * @return
     */
    @RequestMapping("{cmd}/{cmd2}")
    @ResponseBody
    private String anythingDouble(@PathVariable String cmd, @PathVariable String cmd2) {
        return "{cmd}/{cmd2} " + cmd + " " + cmd2;
    }

    /**
     * http://localhost:8089/hello/555
     * http://localhost:8089/hello/rtt
     * 任何只有两级path 的请求，且一级path为hello，都可能被定位到该方法
     *
     * 但是：
     * 如果有一个RequestMapping精准匹配上了，就不会响应此方法，而是调用精准匹配的方法
     * 如访问：http://localhost:8089/hello/hello
     * 只会响应到 @RequestMapping("hello/hello")
     *
     * 如果有一个RequestMapping 的Method 精准匹配上了，就不会响应此方法，而是调用精准匹配的方法
     * 如访问：http://localhost:8089/hello/hello  为GET请求
     * 只会响应到 @RequestMapping(value = "hello/{id}", method = RequestMethod.GET)
     * @param id
     * @return
     */
    @RequestMapping("hello/{id}")
    @ResponseBody
    private String hello(@PathVariable String id) {
        return "hello/{id} " + id;
    }

    /** 精准匹配
     * http://localhost:8089/hello/hello
     * @return
     */
    @RequestMapping("hello/hello")
    @ResponseBody
    private String helloDouble() {
        return "hello/hello ";
    }

    /**
     * 访问：http://localhost:8089/hello/123
     * http://localhost:8089/hello/ggg GET请求
     * @param id
     * @return
     */
    @RequestMapping(value = "hello/{id}", method = RequestMethod.GET)
    @ResponseBody
    private String helloGet(@PathVariable String id) {
        return "get  hello/{id} " + id;
    }

    /**
     * 访问：http://localhost:8089/hello
     * http://localhost:8089/hello POST请求
     * @return
     */
    @RequestMapping(value = "hello", method = RequestMethod.POST)
    @ResponseBody
    private String helloPost(User user) {
        return "post  hello " + user.toString();
    }

    /**
     * 访问：http://localhost:8089/hello
     * http://localhost:8089/hello PUT请求
     * @return
     */
    @RequestMapping(value = "hello", method = RequestMethod.PUT)
    @ResponseBody
    private String helloPut(User user) {
        return "put  hello " + user.toString();
    }

    /**
     * 访问：http://localhost:8089/hello/123
     * http://localhost:8089/hello/ggg DELETE请求
     * @param id
     * @return
     */
    @RequestMapping(value = "hello/{id}", method = RequestMethod.DELETE)
    @ResponseBody
    private String helloDelete(@PathVariable String id) {
        return "delete  hello/{id} " + id;
    }

    /**
     * 方便定义api版本，更具版本升级api
     * 访问：http://localhost:8089/v1/hello
     * http://localhost:8089/v2/hello
     * @return
     */
    @RequestMapping("{version}/hello")
    @ResponseBody
    private String helloVersion(@PathVariable String version) {
        return "{version}/hello " + version;
    }

    /**
     * 访问：http://localhost:8089/v1/hello/123
     * http://localhost:8089/v2/hello/fff
     * @return
     */
    @RequestMapping("{version}/hello/{id}")
    @ResponseBody
    private String helloVersionId(@PathVariable String version, @PathVariable String id) {
        return "{version}/hello/{id} " + version + " id " + id;
    }

    /**
     * 访问：
     * http://localhost:8089/hello/ff123
     * http://localhost:8089/hello/ffggg
     *
     *如果访问
     * http://localhost:8089/hello/zz123
     * 二级path不以ff开始，请求会响应到
     * @RequestMapping(value = "hello/{id}", method = RequestMethod.GET)
     * @return
     */
    @RequestMapping("hello/ff{id}")
    @ResponseBody
    private String helloVersionIdPrefix(@PathVariable String id) {
        return "prefix hello/{id} " + id;
    }

    /**
     * 访问：
     * http://localhost:8089/hello/db123ss
     * http://localhost:8089/hello/123ss
     *
     *如果访问
     * http://localhost:8089/hello/123zz
     * 二级path不以ff开始，请求会响应到
     * @RequestMapping(value = "hello/{id}", method = RequestMethod.GET)
     * @return
     */
    @RequestMapping("hello/{id}ss")
    @ResponseBody
    private String helloVersionIdSubfix(@PathVariable String id) {
        return "subfix hello/{id} " +   id;
    }

}

~~~



匹配规则总结:

**Spring mvc 对Restful的匹配是精准匹配优先，有点类似于nginx location匹配**