#### thymeleaf ：<script>中获取获取一些上下文参数

##### 1,获取sessionId

~~~javascript
<script type="text/javascript" th:inline="javascript">
    //得到sessionId
    /*<![CDATA[*/
        sessionId = /*[[${#session.id}]]*/ '';
    /*]]>*/
    
//后续可以吧sessionId当做一个 JavaScript全局变量用
</script>
~~~



注意[[${#session.id}]]中的#号，否则获取不到sessionId

##### 2，获取session中通过setAttribute设置的值

~~~javascript
<script type="text/javascript" th:inline="javascript">
    //得到sessionId
    /*<![CDATA[*/
        sessionId = /*[[${session.uname}]]*/ '';
    /*]]>*/
    
    
</script>
~~~



#####3，获取contextPath

###### 3.1 方式一

~~~javascript
<script th:inline="javascript">
        /*<![CDATA[*/
        contextPath = /*[[@{/}]]*/ '';
        /*]]>*/

var url = contextpath + "login.html";
alert(url);
</script>
~~~



###### 3.2方式二

某些时候，使用<script th:inline="javascript"> 回和一些框架参数冲突，比如 和layui的 form.verify,可以使用该方式

~~~javas
<script >
     var url = "[[${#request.getContextPath()}]]/cpk/reset";
     alert(url);
</script>
~~~

