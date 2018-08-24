# First
第一个仓库
小伙子，你那什么车啊？跳一下？
好啊，哎哟 不错哦
package com.myTools.softchat.service.impl;

import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;

import javax.websocket.OnClose;
import javax.websocket.OnMessage;
import javax.websocket.OnOpen;
import javax.websocket.Session;
import javax.websocket.server.ServerEndpoint;
import java.io.IOException;
import java.util.concurrent.CopyOnWriteArraySet;

@Component
@ServerEndpoint("/webSocket")
@Slf4j
public class WebSocket {

    private Session session;

    private static CopyOnWriteArraySet<WebSocket> webSocketSet = new CopyOnWriteArraySet<>();

    @OnOpen
    public void onOpen(Session session){
        this.session = session;
        webSocketSet.add(this);
        log.info("【WebSocket::onOpen】有新的连接,当前连接数:{}",webSocketSet.size());
    }

    @OnClose
    public void onClose(Session session){
        webSocketSet.remove(this);
        log.info("【WebSocket::onClose】断开连接,当前连接数:{}",webSocketSet.size());
    }

    @OnMessage
    public void onMessage(String message){
        log.info("【WebSocket::onMessage】收到客户端发来的消息:{}",message);
    }

    /**消息广播发送*/
    public void sendMessage(String message){
        log.info("【WebSocket::sendMessage】webSocketSet:{}",webSocketSet.size());
        for (WebSocket webSocket : webSocketSet) {
            log.info("【WebSocket::sendMessage】服务端广播消息,message:{}",message);
            try {
                webSocket.session.getBasicRemote().sendText(message);
            } catch (IOException e) {
                log.error("【WebSocket::sendMessage】广播消息发生异常:{}",e.getMessage());
                e.printStackTrace();
            }
        }
    }

}













<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>软聊</title>

    <link href="https://cdn.bootcss.com/bootstrap/3.0.1/css/bootstrap.css" rel="stylesheet"/>
    <script src="https://cdn.bootcss.com/jquery/1.12.4/jquery.min.js"></script>
    <script src="https://cdn.bootcss.com/bootstrap/3.3.5/js/bootstrap.min.js"></script>

    <script type="text/javascript">
        /*初始化执行脚本*/
        $(function () {
            queryMsg();
            $("#content").bind('keydown', function (event) {
                event = document.all ? window.event : event;
                if ((event.keyCode || event.which) == 13) {
                    send();
                }
            });
        });

        /*查询聊天记录脚本*/
        function queryMsg(page) {
            var userId = $("#onlySeeSelect").val();
            if(!page){ page = 1; }
            var size = $("#pageSize").val();
            $.ajax({
                type: 'POST',
                url: "/msg/page",
                data: {"currentPage":page ,"pageSize":size, "userId":userId},
                //返回数据的格式
                datatype: "html",//"xml", "html", "script", "json", "jsonp", "text".
                //在请求之前调用的函数
                beforeSend:function(){},
                //成功返回之后调用的函数
                success:function(data){
                    $("#msgul").html(data);
                },
                //调用执行后调用的函数
                complete: function(){},
                //调用出错执行的函数
                error: function(){}
            })
        }

        /*发送消息脚本*/
        function send() {
            var content = $("#content").val();
            if(content.trim()==''){
                $("#tips").html("请输入内容!");
                $('#tipsModal').modal('show');
                $("#content").val("");
                return;
            }
            $.ajax({
                type: 'POST',
                url: "/msg/send",
                data: {"content":content},
                datatype: "json",//"xml", "html", "script", "json", "jsonp", "text".
                beforeSend:function(){},
                success: function (data) {
                    $("#content").val("");
                },
                complete: function(){},
                error: function(){}
            })
        }

    </script>

    <#--websocket-->
    <script type="text/javascript">
        /*先判断WebSocket是否已内置于浏览器*/
        console.log('先判断WebSocket是否已内置于浏览器');
        var websocket = null;
        if ('WebSocket' in window) {
            websocket = new WebSocket('ws://localhost/webSocket')
        } else {
            alert('该浏览器不支持WebSocket!')
        }

        /*WebSocket初始化时*/
        console.log('WebSocket初始化时建立连接function');
        websocket.onopen = function (event) {
            console.log('建立WebSocket连接!')
        }

        /*WebSocket关闭时*/
        console.log('WebSocket关闭时function');
        websocket.onclose = function (event) {
            console.log('关闭WebSocket连接!')
        }
        /*监听到新消息时*/
        console.log('监听到新消息时function');
        websocket.onmessage = function (event) {
            console.log('收到消息:' + event.data)
            //做相关的反应操作:更新消息
            queryMsg();
        }
        /*WebSocket发生错误时*/
        console.log('WebSocket发生错误时function');
        websocket.onerror = function () {
            alert('WebSocket通信发生错误!');
        }
        /*本页面关闭时*/
        console.log('本页面关闭时function');
        window.onbeforeunload = function () {
            websocket.close();
        }
    </script>


</head>
<body>


    <div class="container">
        <div class="row clearfix">

            <!--导航栏开始-->
            <div class="col-md-12 column row">
                <nav class="navbar navbar-default" role="navigation">
                    <div class="navbar-header col-md-2 col-lg-2">
                        <button type="button" class="navbar-toggle" data-toggle="collapse" data-target="#bs-example-navbar-collapse-1">
                            <span class="sr-only">Toggle navigation</span>
                            <span class="icon-bar"></span>
                            <span class="icon-bar"></span>
                            <span class="icon-bar"></span>
                        </button>
                        <a class="navbar-brand" href="/chatroom">聊天室</a>
                    </div>
                    <div class="col-md-2 col-lg-3">
                        <ul class="nav navbar-nav">
                            <li><a href="/msg/history">历史记录</a></li>
                        </ul>
                    </div>
                    <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
                        <div class="navbar-form navbar-left">
                            <div class="form-group">
                                <select id="onlySeeSelect">
                                    <option value="">所有人</option>
                                    <#list all_user as u>
                                        <option value="${u.id}">${u.username}</option>
                                    </#list>
                                </select>
                            </div>
                            <button class="btn btn-default" onclick="queryMsg()">只看Ta</button>
                        </div>
                        <div class="navbar-form navbar-left">
                            <div class="form-group">
                                显示消息:
                                <select id="pageSize" onchange="queryMsg()">
                                    <option value="10">10</option>
                                    <option value="12" selected>12</option>
                                    <option value="15">15</option>
                                    <option value="20">20</option>
                                    <option value="30">30</option>
                                </select>
                                条
                            </div>
                        </div>
                        <ul class="nav navbar-nav navbar-right">
                            <#if (user)??>
                                <li>
                                    <a href="#">${user.username}</a>
                                </li>
                            <#else>
                                <li>
                                    <a href="#">${nickname}</a>
                                </li>
                                <li>
                                    <a href="/user/loginUI">登录</a>
                                </li>
                            </#if>
                            <li class="dropdown">
                                <a href="#" class="dropdown-toggle" data-toggle="dropdown">用户中心<strong class="caret"></strong></a>
                                <ul class="dropdown-menu">
                                    <li>
                                        <a href="/user/changePassword">修改密码</a>
                                    </li>
                                    <li>
                                        <a href="/contactAdmin">联系管理员</a>
                                    </li>
                                    <li class="divider"></li>
                                    <li>
                                        <a href="/user/logout">注销</a>
                                    </li>
                                </ul>
                            </li>
                        </ul>
                    </div>
                </nav>
            </div>
            <!--导航栏结束-->

            <!--聊天内容开始-->
            <div class="col-md-12 row" style="background-color: aliceblue">
                <div class="col-md-11">
                    <ul class="list-unstyled" id="msgul">数据加载失败</ul>
                </div>
            </div>
            <!--聊天内容结束-->


            <!--发送框开始-->
            <div class="col-md-12 row" style=" height: 120px;width: 100%; position: fixed; bottom: 0;">
                <hr/>
                <div class="col-xs-10">
                    <textarea id="content" name="content" class="form-control" placeholder="请输入..." rows="1" cols="100"></textarea>
                </div>
                <div class="col-xs-2">
                    <button type="button" class="btn btn-default" onclick="send()">发送</button>
                </div>
            </div>
            <!--发送框结束-->

            <#--提示模态框开始-->
            <div id="tipsModal" class="modal fade" id="failModal" role="dialog" aria-labelledby="myModalLabel" aria-hidden="true">
                <div class="modal-dialog">
                    <div class="modal-content">
                        <div class="modal-header">
                            <button type="button" class="close" data-dismiss="modal" aria-hidden="true">×</button>
                            <h4 class="modal-title" id="myModalLabel">
                                提示
                            </h4>
                        </div>
                        <div class="modal-body">
                            <span id="tips"></span>
                        </div>
                        <div class="modal-footer">
                            <button type="button" class="btn btn-default" data-dismiss="modal">关闭</button>
                        </div>
                    </div>
                </div>
            </div>
            <#--提示模态框结束-->
        </div>
    </div>


</body>
</html>





caizeming EN    首页|文档课程|社区求助论坛活动微博|专家博客|Wiki术语|业务系统外部资源
 团队图标
CloudDragon服务化工具链
当前位置
关键词
提交
首页
云龙百科
云龙知乎
公告栏
活动
文档
技术支持
更多
当前位置：首页 > 云龙百科 > 服务创建 > 操作指导 > 云龙 codegen版本特性
云龙 codegen版本特性    [ 您没有编辑权限 ]
目录
该工具的维护主体是云龙CloudInit服务团队，有问题可以咨询维护责任人（张西营 z00317404）
1     说明
2     2.1.13
3     2.1.14
4     2.1.17
[显示全部]
  [ 回目录 ]该工具的维护主体是云龙CloudInit服务团队，有问题可以咨询维护责任人（张西营 z00317404）   
  [ 回目录 ]1     说明   
本文是关于云龙SpringMVC/SpringBoot/SpringCloud开发框架的代码生成插件版本新特性说明，关于插件的yaml语法可参考如下文档。

      Restful API Doc Spec:  http://swagger.io/specification/
      RESTful API接口文件定义规范:   http://3ms.huawei.com/hi/group/2025819/wiki_4131055.html
特性列表 

  [ 回目录 ]2     2.1.13   
  [ 回目录 ]2.1     新增特性   
2.1.13
        支持Object
2.1.14
        支持HashMap<String,Object>
2.1.17
        支持tag优先级生成类文件
2.1.23
        支持接口参数异常处理
2.1.30
        去除client和model的checkstyle、findbugs、pmd告警
2.1.31
        实体类支持封装第三方类
        实体类支持封装数组
        接口支持第三方类作入参和返回类型
        解决中文注释乱码的问题
2.1.40
        增加打桩功能，加速开发进度 
2.1.41 
        修复bug: format为date-time时，增量生成API会误生成导入类model.date
2.1.45
        支持通过MultiparFile上传文件
2.1.46
        接口入参默认不按required排序
2.1.49
        枚举支持编程语言关键字
        集合类支持长度控制
2.1.52
        用到org.codehaus.jackson包的所有类切换到com.fasterxml.jackson包下的对应类
        解决mock业务层代码生成时无新增接口也改动源码的问题
        解决mock代码里返回类型为集合类时编译报错的问题
2.1.54
        支持Java基本类型生成封装类
        集合类默认初始化为null，而不是new一个对象
2.1.56
        支持生成XML注解
2.2.2
        实体类字段支持配置自定义注解
        实体类支持配置类级自定义注解
        接口入参支持配置自定义注解
        去掉生成的源码注释中带有的敏感信息
        修复MultipartFile上传文件生成接口报错的问题
2.2.4
        实体类字段配置第三方类的变更
        接口入参配置第三方类的变更
        接口返回对象配置第三方类的变更
2.2.6
        实体类支持指定包路径
2.2.8
        api支持配置是否抛出ServiceException异常
        api支持指定全局异常
        api支持配置是否生成Mock代码
        client代码支持配置生成到output目录下
        yaml文件支持按数组方式配置，且支持模糊匹配
        支持配置controller类是否生成ExceptionHandler方法
2.2.10
        实体类字段支持是否允许toString打印
        支持生成枚举类
        支持是否生成装配request对象的代码
2.2.12
        集合变量允许控制是否初始化
        修复配置不抛出异常时，生成 throws false; 的问题
2.2.21
        接口URL支持拼basePath做前缀
        解决模糊匹配yaml文件功能在linux环境上不生效的问题
2.2.23
        修复多个类的data属性引用同一个类的时候，只能生成一个Data类的问题
2.2.25
         通过x-tag标签，可以替换tags内容使一个yaml生成一个接口类
        @RequestHeader中加入属性name
        修复通过x-imports继承第三方类时重复导入包的问题
2.2.25
         通过showRecluseContract标签控制是否生成@RestController、@RequestMapping 、@RequestBody、@RequestParam、@PathVariale注解

  [ 回目录 ]2.2     特性说明   
2.2.1 说明

  对于接口definitions自定义类时，支持定义类属性为Object类。

 

yaml定义实例如下: 
definitions:
  ServiceInfo:
    properties:
      serviceId:
        type: integer
      serviceName:
        $ref: '#/definitions/Object'

生成代码如下:
public class ServiceInfo  implements Serializable
{
    
    @JsonProperty("serviceId")
    private int serviceId = 0;
    
    @JsonProperty("serviceName")
    private Object serviceName = null;
    
    public ServiceInfo()
    {
    	super();
    } 
    
    /**
    **/

    public int getServiceId() 
    {
        return serviceId;
    }
    public void setServiceId(int serviceId) 
    {
        this.serviceId = serviceId;
    }
    
    /**
    **/

    public Object getServiceName() 
    {
        return serviceName;
    }
    public void setServiceName(Object serviceName) 
    {
        this.serviceName = serviceName;
    }
}

  [ 回目录 ]3     2.1.14   
  [ 回目录 ]3.1     新增特性   
序号

特性

1

支持HashMap<String,Object>

 

  [ 回目录 ]3.2     特性说明   
3.2.1 说明

  对于接口definitions自定义类时，支持定义类属性为HashMap<String,Object>类。

yaml定义实例如下: 
definitions:
  ServiceInfo:
    properties:
      values:
        type: object
        additionalProperties:
          $ref: '#/definitions/Object'

生成代码如下:
public class ServiceInfo  implements Serializable
{
    
    @JsonProperty("values")
    private Map<String, Object> values = new HashMap<String, Object>();
    
    public ServiceInfo()
    {
    	super();
    } 
    
    /**
    **/

    public Map<String, Object> getValues() 
    {
        return values;
    }
    public void setValues(Map<String, Object> values) 
    {
        this.values = values;
    }
}

  [ 回目录 ]4     2.1.17   
  [ 回目录 ]4.1     新增特性   
序号

特性

1

支持tag优先级生成类文件

 

  [ 回目录 ]4.2     特性说明   
4.2.1 说明

  支持tag优先级生成接口定义和Controller类文件。

yaml定义实例如下: 
paths:
  /rest/v1:
    get:
      tags:
        - Lhh1704011524
      operationId: getServiceInfo
      produces:
        - application/json
      responses:
        200:
          description: Get Service Information
          schema:
            type: string
  /rest/v2:
    get:
      tags:
        - Lhh1704011524V2
      operationId: getServiceInfoV2
      produces:
        - application/json
      
      responses:
        200:
          description: Get Service Information
          schema:
            type: string

生成代码如下:




  [ 回目录 ]5     2.1.23   
  [ 回目录 ]5.1     新增特性   
序号

特性

1

支持接口参数异常处理

  [ 回目录 ]5.2     特性说明   
5.2.1 说明

      说明:支持处理的异常

1.      MethodArgumentTypeMismatchException(参数校验异常)

2.      MethodArgumentConversionNotSupportedException (参数校验异常)

3.      HttpMediaTypeNotAcceptableException

4.      MissingServletRequestParameterException

5.      NoSuchRequestHandlingMethodException

6.      HttpRequestMethodNotSupportedException

7.      HttpMessageNotWritableException

8.      HttpMessageNotReadableException

 

   异常处理返回格式:

    {

                "status": "error",

                "errorMsg": "Method Argument Type Mismatch Exception!"

   }

 

  [ 回目录 ]6     2.1.30   
  [ 回目录 ]6.1     新增特性   
序号

特性

1

去除client和model的checkstyle、findbugs、pmd告警

  [ 回目录 ]6.2     特性说明   
6.2.1 说明

      有些findbugs告警是清不掉的，即实体类如果有数组，如char[]，就会报相关告警。实际上，我们可以清除这些告警，但会引出codedex新告警。所以只好放弃。

 

  [ 回目录 ]7     2.1.31   
  [ 回目录 ]7.1     新增特性   
序号

特性

1

实体类支持封装第三方类

2

实体类支持封装数组

3

接口支持第三方类作入参和返回类型 

4

解决中文注释乱码的问题


由于此次版本不遵循swagger规范，请勿使用，请使用2.2.4版本。
  [ 回目录 ]7.2     特性说明   
7.2.1 说明

在以前，通过$ref我们只能引用在配置文件definitions标签下定义好的封装类型。
然而，很多时候，开发者不想拘束于这样的限制，希望引入第三方类。基于这样的需要，我们开发出新标签x-imports，通过这个标签把类显式导入，再在$ref内配置引用即可。
需要注意的是，在实体类内，x-imports标签需要放在与properties标签同一级的位置。在接口定义时，x-imports需要放在与parameters同一级的位置。支持同时导入多个类，导入多个类用英文分号隔开。
在配置导入之前，请确认要导入的类在当前环境下是否能找到。
如果一个类被多次配置导入，或者配置了导入但没有引用，又或者按单类型配置导入同时又按需类型配置导入，我们会优化导入内容。优化原则：导入就要被使用，不重复导入。这是为了防止构建服务时出现烦人的告警。
另外，String、Integer、Long这些在java.lang包下的封装类型是不需要显式导入就可以直接引用的。为了在构建服务时不出现关于java.lang多余导入的告警，代码生成时会忽略java.lang的相关导入。


7.2.2 实体类支持封装第三方类



 一个简单数据类型的配置示例：

   definitions:
  Response:
    description: 请求响应内容封装类
    x-imports: 'java.math.BigDecimal;'
    properties:
      money:
        $ref: '#/definitions/BigDecimal'
 

一个复杂数据类型的配置示例：(Map<String, Map<String, BigDecimal>>)

  definitions:
  Response:
    description: 请求响应内容封装类
    x-imports: 'java.math.BigDecimal;net.sf.json.*'
    properties:
      money:
        $ref: '#/definitions/BigDecimal'
      complexData:
        type: object
        additionalProperties:
          type: object
          additionalProperties:
            $ref: '#/definitions/JSONObject'
        description: 复杂数据类型
 

继承情况下的配置示例：

  definitions:
  Response:
    description: 请求响应内容封装类
    x-imports: 'java.math.BigDecimal;'
    properties:
      money:
        $ref: '#/definitions/BigDecimal'
      complexData:
        type: object
        additionalProperties:
          type: object
          additionalProperties:
            $ref: '#/definitions/BigDecimal'
        description: 复杂数据类型
  DataResponse:
    description: 数据封装
    allOf:
      - $ref: '#/definitions/Response'
      - type: object
        x-imports: 'net.sf.json.*'
        properties:
          message:
            $ref: '#/definitions/JSONObject'
          dataList:
            $ref: '#/definitions/JSONArray'
 

7.2.3 实体类支持封装数组



一维数组封装示例：

  definitions:
  Response:
    description: 请求响应内容封装类
    x-imports: 'java.math.BigDecimal;'
    properties:
      money:
        $ref: '#/definitions/BigDecimal[]'


三维数组封装示例：

  definitions:
  Response:
    description: 请求响应内容封装类
    x-imports: 'java.math.BigDecimal;'
    properties:
      money:
        $ref: '#/definitions/BigDecimal[][][]'


7.2.4 接口支持第三方类作入参和返回类型

 
paths:
   /v1/account/checkAccount:
    post:
      summary: 验证账号密码是否匹配
      description: 验证账号密码是否匹配
      tags:
        - AccountService
      operationId: checkAccount
      x-imports: 'net.sf.json.JSONObject;org.eclipse.jetty.security.jaspi.modules.UserInfo;'
      parameters:
        - name: userInfo
          in: body
          description: 账号信息
          required: true
          schema:
            $ref: '#/definitions/UserInfo'
      responses:
        '200':
          description: ''
          schema:
            $ref: '#/definitions/JSONObject'
        可能你会纳闷，导入是在接口定义内导入的，如果一个.java文件内有多个接口都引用同一个第三方类，是不是每个接口都要导入呢。答案是否定的，你只需要导入一次即可。当然，导入多次也没关系，只是代码生成时，重复的导入会被忽略掉，即只为你导入一次。

 

7.2.5 解决中文注释乱码的问题

        在以前，中文注释在生成的代码里常常表现乱码，此次版本更新解决乱码问题，看起来更友好。

 

  [ 回目录 ] 8     2.1.40    
  [ 回目录 ]8.1     新增特性   
       序号 特性 1 Mock业务类的生成 2 新增接口业务类方法的增量生成 3 Mock入参的配置 4 Mock结果的配置

  [ 回目录 ]   
  [ 回目录 ]8.2     特性说明   
8.2.1 说明

      Mock，有模拟的意思。假使你在A微服务上做开发，需要B服务的B1接口，但是B1接口未能尽快释放出来给你。怎么办，无奈地等待吗？不，浪费时间的事情我们尽量少做。如果我们知道入参和回调结果，是不是有种想模拟一个接口出来做应付使用？没错，这就是mock，打个桩，开始干活。



8.2.1 Mock业务类的生成

      相对于以前的生成器版本，使用此版本生成的源码中，工程目录下会生成${package}.mock.impl这样一个包，假设我在yaml里的配置如下：

   /v2/test:
    post:
      summary: 创建服务-生产消息
      description: 创建服务各阶段生产消息
      tags:
        - MockTestService2
      operationId: test2
      parameters: 
        - name: username
          in: query
          description: 
          required: false
          type: string
          x-mock: 'user1'
      responses:
        '200':
          description: ''
          x-mock: {"username":"sdfsfs","password":"sdfsgsfgsdfsf","status":"sdfsdgsfs"}
          schema:
            $ref: '#/definitions/UserInfo'
        '400':
          description: 请求参数非法
          schema:
            $ref: '#/definitions/FailedRsp'
      生成的源码长相如下：

-java代码
01
public class MockTestService2DelegateImplMock implements MockTestService2Delegate
02
 {
03
     public static final String mockResponse_test2_200 =
04
         "{\"username\":\"sdfsfs\",\"password\":\"sdfsgsfgsdfsf\",\"status\":\"sdfsdgsfs\"}";
05
     public static final String mockResponse_test2_400 = "";
06
 
07
     @Override
08
     public UserInfo test2(String username)
09
         throws ServiceException
10
     {
11
         return ObjectMapperUtil.string2ObjectWithoutThrow(
12
             mockResponse_test2_200,
13
             UserInfo.class);
14
     }
15
 }

      先不管x-mock标签，类级变量和方法体，请留意该mock类的实现接口和方法。



8.2.2 新增接口业务类的方法增量生成

      在以前，我们配置一个接口在对应的业务工程下没有实现类才会生成该类的源码，如果已经有了实现类，新增接口到该类下不会自动新增新接口对应的方法，必须手动新增。此版本解决这个问题，增量生成的源码默认使用mock业务。

-java代码
1
    @Override
2
     public UserInfo test2(String username)
3
         throws ServiceException
4
     {
5
        MockTestService2DelegateImplMock delegateImpl = new MockTestService2DelegateImplMock();
6
        return delegateImpl.test2(username);
7
     }

开始编码时，把方法体内的代码去掉，再编码。这就是mock业务应用的地方。



8.2.3 Mock入参的配置

      Mock入参，其实意思就是写死入参数值。你可能会问，不是可以配置默认值吗，为什么还需要Mock入参呢。默认值是没有传值时才会生效的，而Mock入参，则是无论有没有传值，都会使用Mock写死的值。那么Mock入参的应用场景和意义是什么？答，此处暂不讨论其应用场景及意义。

      假设你定义了一个实体类，长相如下：

-powershell代码
1
  UserInfo:
2
    properties:
3
      username:
4
        type: string
5
      password:
6
        type: string
7
      status:
8
        type: string
9
        maxLength: 255

      接口定义长相如下：

-powershell代码
01
  /v1/test1:
02
    post:
03
      summary: 创建服务-生产消息
04
      description: 创建服务各阶段生产消息
05
      tags:
06
        - MockTestService1
07
      operationId: test1
08
      parameters:
09
        - name: userInfo
10
          in: body
11
          description:
12
          required: true
13
          schema:
14
            $ref: '#/definitions/UserInfo'
15
          x-mock: {"username":"testusername","password":"testpassword","status":"ok"}
16
      responses:
17
        '200':
18
          description: ''
19
          schema:
20
            $ref: '#/definitions/ResponseUserInfo'
21
        '400':
22
          description: 请求参数非法
23
          schema:
24
            $ref: '#/definitions/FailedRsp'

      请注意，在parameters下x-mock标签的内容为mock入参。执行clean install命令后，如执行顺利，找到controller类，其中有一段代码长相如下：

-java代码
01
    public static final String mockParam_test1_userInfo =
02
        "{\"username\":\"testusername\",\"password\":\"testpassword\",\"status\":\"ok\"}";
03
         
04
 
05
    @Override
06
    public UserInfo test1(
07
            
08
            @Valid @RequestBody UserInfo userInfo)
09
        throws ServiceException
10
    {
11
        userInfo = ObjectMapperUtil.string2ObjectWithoutThrow(
12
            mockParam_test1_userInfo,UserInfo.class);
13
         
14
     
15
        return delegate.test1(userInfo);
16
    }

      首先，ObjectMapperUtil类是一起生成源码中的一个工具类，string2ObjectWithoutThrow方法的作用是将一串字符转换生成指定封闭类的实例。

      在接口被调通之后，接口入参在这里会被替换成mock入参。

      请注意，mock入参的值必需要能够被转换的字符。你不能定义入参类型为int，而mock入参赋值字母什么的。如果入参类型是封闭类，则mock入参需要赋值json字符串，而且属性需要能够匹配得上。

      啊不，其实如果你不喜欢处理json字符串，不防尝试如下配置。

-powershell代码
01
  /v1/test1:
02
    post:
03
      summary: 创建服务-生产消息
04
      description: 创建服务各阶段生产消息
05
      tags:
06
        - MockTestService1
07
      operationId: test1
08
      parameters:
09
        - name: userInfo
10
          in: body
11
          description:
12
          required: true
13
          schema:
14
            $ref: '#/definitions/UserInfo'
15
          x-mock:
16
            username: 'testusername'
17
            password: 'testpassword'
18
            status: 'ok'
19
      responses:
20
        '200':
21
          description: ''
22
 
23
          schema:
24
            $ref: '#/definitions/ResponseUserInfo'
25
        '400':
26
          description: 请求参数非法
27
          schema:
28
            $ref: '#/definitions/FailedRsp'




8.2.4 Mock结果的配置

      Mock结果的配置和Mock入参的配置相似，区别是Mock结果配置时需要将x-mock标签放在状态码下。而生成的相关java代码则在相应的mock类里可找到。

-java代码
01
  /v1/test1:
02
    post:
03
      summary: 创建服务-生产消息
04
      description: 创建服务各阶段生产消息
05
      tags:
06
        - MockTestService1
07
      operationId: test1
08
      parameters:
09
        - name: userInfo
10
          in: body
11
          description:
12
          required: true
13
          schema:
14
            $ref: '#/definitions/UserInfo'
15
          x-mock:
16
            username: 'testusername'
17
            password: 'testpassword'
18
            status: 'ok'
19
      responses:
20
        '200':
21
          description: ''
22
          x-mock: {"username":"sdfsfs","password":"sdfsgsfgsdfsf","status":"sdfsdgsfs"}
23
          schema:
24
            $ref: '#/definitions/UserInfo'
25
        '400':
26
          description: 请求参数非法
27
          schema:
28
            $ref: '#/definitions/FailedRsp'



8.2.5 Mock配置支持的数据类型

      Mock配置主要由org.codehaus.jackson.map.ObjectMapper类提供字符串到对象的转换。目前支持int,long,float,double,string,boolean,封装。这些数据类型是否能够满足你的需要？需要更多支持吗，date?date-time?binary?password?还有数组和集合类型呢。或者我们已经提供了支持自己都不知道呢，只是懒得去验证。



  [ 回目录 ]9     2.1.41   
  [ 回目录 ]9.1     新增特性   
      修复bug: format为date-time时，增量生成API时会误生成导入类model.date  

  [ 回目录 ]9.2     特性说明   
      format为date-time时，增量生成API时会误生成导入类model.date

  [ 回目录 ]    
  [ 回目录 ] 10     2.1.45    
  [ 回目录 ]9.1     新增特性   
      支持通过MultiparFile上传文件  

  [ 回目录 ]10.2     特性说明   
10.2.1 说明

  本次更新带来文件上传的支持，配置后，可使你的接口支持多文件上传。

10.2.2 使用步骤及示例(SpringMVC)

   1、在web.xml文件中配置servlet的标签内加入 multipart-config 配置，加入后代码类似：

-xml代码
01
  <servlet>
02
    <description>spring mvc servlet</description>
03
    <servlet-name>springMvc</servlet-name>
04
    <servlet-class>com.huawei.clouddragon.CloudDragonServlet</servlet-class>
05
    <init-param>
06
      <description>spring mvc conf</description>
07
      <param-name>contextConfigLocation</param-name>
08
      <param-value>classpath:spring/*.xml</param-value>
09
    </init-param>
10
    <load-on-startup>1</load-on-startup>
11
    <multipart-config>
12
        <max-file-size>52428800</max-file-size>
13
        <max-request-size>52428800</max-request-size>
14
        <file-size-threshold>0</file-size-threshold>
15
    </multipart-config>
16
  </servlet>
17
  <servlet-mapping>
18
    <servlet-name>springMvc</servlet-name>
19
    <url-pattern>/*</url-pattern>
20
  </servlet-mapping>

   2、在spring配置文件上引入mvc命名空间

      在beans标签内加入如下属性：

-xml代码
1
xmlns:mvc=http://www.springframework.org/schema/mvc

      在beans标签的xsi:schemaLocation属性内加入下面两行：

-xml代码
1
http://www.springframework.org/schema/mvc
2
http://www.springframework.org/schema/mvc/spring-mvc.xsd 

   3、引入mvc命名空间后，加入如下配置

-xml代码
1
<mvc:annotation-driven/>

   4、再在spring配置文件上加入这行代码

-xml代码
1
<bean id="multipartResolver" class="org.springframework.web.multipart.support.StandardServletMultipartResolver"></bean>

   如果加入这行代码后启动Web应用服务器时报错，请确认是否缺包。

   5、接口配置中的produces标签的值设置为：multipart/form-data

   6、接口入参中的in标签的值设置为：formData，type标签的值设置为：file

   7、完整接口配置示例

-powershell代码
01
  /v2/test2:
02
    post:
03
      summary: 测试文件传输
04
      description: 测试文件传输
05
      tags:
06
        - TestService2
07
      operationId: test1
08
      consumes:
09
        - application/json
10
      produces:
11
        - multipart/form-data
12
      parameters:
13
        - name: file1
14
          in: formData
15
          type: file
16
          description:
17
          required: false
18
        - name: file2
19
          in: formData
20
          type: file
21
          description:
22
          required: false
23
        - name: message1
24
          in: formData
25
          type: string
26
          description:
27
          required: false
28
        - name: message2
29
          in: formData
30
          type: string
31
          description:
32
          required: false
33
      responses:
34
        '200':
35
          description: ''
36
          schema:
37
            $ref: '#/definitions/ResultModel
38
        '400':
39
          description: 请求参数非法
40
          schema:
41
            $ref: '#/definitions/FailedRsp'

   8、上传文件时，如果获取到的文件名乱码，可以尝试这样解决：

-java代码
1
        String originName = file1.getOriginalFilename();
2
        String fileName = new String(originName.getBytes("GBK"), "UTF-8");

    9、得到MultipartFile对象后，如果你不知道怎么保存成文件，好吧，示例给你（file1为MultipartFile对象）：

-java代码
01
InputStream inputStream = file1.getInputStream();
02
OutputStream outputStream = new FileOutputStream(new File("D:\\txt.txt"));
03
byte[] buff = new byte[2048];
04
int len = 0;
05
while ((len = inputStream.read(buff)) != -1)
06
{
07
    outputStream.write(buff, 0, len);
08
}
09
outputStream.close();
10
inputStream.close();


10.2.3 使用步骤及示例(SpringBoot)
 SpringBoot工程的配置相对简单些，首先在yaml内配置好接口，部署到容器，看看MultipartFile对象是否有内容。如果没有，请引入一个spring配置，并在配置内加入：

-xml代码
1
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
2
    <property name="maxUploadSize" value="104857600"></property>
3
    <property name="maxInMemorySize" value="104857600"></property>
4
    <property name="defaultEncoding" value="UTF-8"></property>
5
</bean>

启动服务，如果报缺包，你可能需要加入如下依赖：

-xml代码
1
    <dependency>
2
        <groupId>commons-fileupload</groupId>
3
    <artifactId>commons-fileupload</artifactId>
4
    <version>1.3.1</version>
5
        </dependency>
6
</dependencies>


  [ 回目录 ]11     2.1.46   
  [ 回目录 ]11.1     新增特性   
      接口入参默认不按required排序  

  [ 回目录 ]11.2     特性说明   
      在配置生成器的pom.xml里，如果不配置sortParamsByRequiredFlag参数，则该参数会被默认赋值false。   另外，这个入参的意思是，接口入参的排序规则是否按照required来进行排序。假设有三个参数：   【key1】: required=false   【key2】: required=true   【key3】: required=false

      如果配置了sortParamsByRequiredFlag=true，那么生成的方法形似：operate(key2,key1,key3)。

      如果配置了sortParamsByRequiredFlag=false，那么生成的方法形似：operate(key1,key2,key3)。




  [ 回目录 ]12     2.1.49   
  [ 回目录 ]12.1     新增特性   
      枚举支持编程语言关键字  集合类支持长度控制  

  [ 回目录 ]12.2     特性说明   
12.2.1 枚举类型字段名支持java关键字

      在以前，配置的枚举字段名如果是编程语言关键字，编译会报错，现在解决这个问题。

12.2.2 集合类支持长度控制

      使用该特性，请使用 x-minLength 标签配置最小长度，使用 x-maxLength 标签配置最大长度。


  [ 回目录 ] 13     2.1.52   
  [ 回目录 ] 13.1     新增特性   
      用到org.codehaus.jackson包的所有类切换到com.fasterxml.jackson包下的对应类

      解决mock业务层代码生成时无新增接口也改动源码的问题

      解决mock代码里返回类型为集合类时编译报错的问题


如发现依赖缺少，在POM中添加jackson依赖

<dependency>
<groupId>com.fasterxml.jackson.core</groupId>
<artifactId>jackson-databind</artifactId>
<version>2.6.6</version>
</dependency>



  [ 回目录 ]13.2     特性说明   
      如题


  [ 回目录 ]14     2.1.54   
  [ 回目录 ]14.1     新增特性   
      支持Java基本类型生成封装类  集合类默认初始化为null，而不是new一个对象


  [ 回目录 ]14.2     特性说明   
支持Java基本类型生成封装类



       此版本以前，我们配置生成整形，单精度，双精度，布尔类型这些会生成int,float,double,boolean这些primivite类型，现在我们改为生成Integer,Float,Double,Boolean，并初始化为null。

      如果你仍然需要生成privamite类型，可以在pom文件配置插件入参处加上generatePrimitiveType，设置为true即可。配置可参考：

-xml代码
01
<plugin>
02
    <groupId>com.huawei.tools</groupId>
03
    <artifactId>service-codegen-maven-plugin</artifactId>
04
    <version>2.1.54</version>
05
    <configuration>
06
        <inputSpec>${project.basedir}/src/main/resources/rest-services.yaml</inputSpec>
07
        <language>spring4mvc</language>
08
        <invokerPackage>com.huawei.clouddragon.springcloudgencodetest</invokerPackage>
09
        <output>.</output>
10
        <sortParamsByRequiredFlag>false</sortParamsByRequiredFlag>
11
        <schema>both</schema>
12
        <generatePrimitiveType>true</generatePrimitiveType>
13
    </configuration>
14
    <executions>
15
        <execution>
16
            <id>model-generate</id>
17
            <phase>generate-sources</phase>
18
            <goals>
19
                <goal>generate</goal>
20
            </goals>
21
        </execution>
22
    </executions>
23
</plugin>


集合类默认初始化为null，而不是new一个对象
      此版本以前，配置生成list,map会初始化一个空集合，如：new ArrayList，new HashMap，现在，取消这种做法，初始化为null。


  [ 回目录 ]15     2.1.56   
  [ 回目录 ]15.1     新增特性   
      支持生成XML注解


  [ 回目录 ]15.2     特性说明   
说明

之前的版本中，实体类的注解是JSON注解，这里增加XML注解的支持，通过配置，用户可选择生成JSON注解还是XML注解。



类级注解的生成

要生成XML注解，需要在yaml中实体类配置下增加x-generateXml属性，设置为true，设置为false或不配置该属性则生成JSON注解。

要指定生成的XML内容中的root节点名，增加x-xmlRootName属性，并指定它的值。配置示例如下：

-powershell代码
01
ExampleModel:
02
  x-generateXml: true
03
  x-xmlRootName: root
04
 
05
  properties:
06
    names:
07
      description: '名字'
08
      type: array
09
      items:
10
        type: string
11
      x-minLength: 1
12
      x-maxLength: 200

将生成如下源码：
-java代码
1
@XmlAccessorType(XmlAccessType.FIELD)
2
@XmlRootElement(name = "root")
3
public class ExampleModel   
4
{
5
}


节点注解的生成

配置生成类级注解后，生成器会默认给各字段添加注解，暂不支持指定节点名。

生成的代码示例：

-java代码
01
@XmlAccessorType(XmlAccessType.FIELD)
02
@XmlRootElement(name = "root")
03
public class ExampleModel   
04
{
05
     
06
    @Size(max = 200, min = 1)
07
    @Valid
08
    @XmlElementWrapper(name = "names")
09
    @XmlElement(name = "names")
10
    private List<String> names = new ArrayList<String>();
11
     
12
    @Size(max = 200, min = 1)
13
    @Valid
14
    @XmlElement(name = "type")
15
    private String type = null;
16
     
17
    public enum LimitEnum
18
    {
19
         @JsonProperty("ERROR") ERROR,  @JsonProperty("OK") OK,  @JsonProperty("java") JAVA,  @JsonProperty("import") IMPORT,  @JsonProperty("class") CLASS, ;
20
    };
21
     
22
    @XmlElement(name = "limit")
23
    private LimitEnum limit = LimitEnum.JAVA;
24
     
25
    //get set method
26
}

  [ 回目录 ]16     2.2.2   
  [ 回目录 ]16.1     新增特性   
实体类字段支持配置自定义注解
实体类支持配置类级自定义注解
接口入参支持配置自定义注解
去掉生成的源码注释中带有的敏感信息
修复MultipartFile上传文件生成接口报错的问题

  [ 回目录 ]16.2     特性说明   
说明

在使用本插件过程中，你可能会有如下体会：

不了解一些用法，如配置实体类字段，接口入参的用法。
在配置实体类字段，接口入参时因业务需要而急需一些支持。
你们给我们提了需求，但需求实现起来需要时间，不能马上提供。
你们有特殊需求，生成器通过常规方法无法实现。
此版本能一定程度上排解上述难题。通过简单易用的配置，能实现比较复杂的需求。


x-onlyCustomAnnotation 扩展属性：配置此属性，赋予true值的话，表示只使用x-annotations属性配置的注解。
x-annotations 扩展属性：此属性用于配置要生成的自定义注解。

用法：

通过yaml配置一个实体类，会默认生成类级注解。要想取消这些注解，配置x-onlyCustomAnnotation为true即可实现。
x-annotations属性支持配置多个自定义注解，配置成列表即可。
通过x-annotations属性配置注解类名时可加入包名，如果不想这么做，可考虑在x-imports属性上配置引入。

暂不支持接口中的方法级注解的自定义配置以及接口类中的类级注解的自定义配置。


实体类支持配置类级自定义注解 示例

yaml配置：
-powershell代码
01
Response:
02
  description: 失败响应对象
03
  x-imports: "javax.xml.bind.annotation.XmlRootElement;javax.xml.bind.annotation.XmlElement;javax.validation.constraints.*;"
04
  x-onlyCustomAnnotation: true
05
  x-annotations:
06
  - "@XmlRootElement(name = \"root\")"
07
  properties:
08
    message:
09
      type: string
10
      x-onlyCustomAnnotation: true
11
      x-annotations:
12
      - "@Size(max = 25, min = 12)"
13
      - "@XmlElement"

效果：
-java代码
01
@XmlRootElement(name = "root")
02
public class Response  implements Serializable
03
{
04
    private static final long serialVersionUID = 1L;
05
     
06
    @Size(max = 25, min = 12)
07
    @XmlElement
08
    private String message = null;
09
     
10
    //other code
11
}



实体类字段支持配置自定义注解 示例
参考上一个实例


接口入参支持配置自定义注解 示例

yaml配置：

-powershell代码
01
paths:
02
  /rest/v1/healthcheck:
03
    get:
04
      summary: heartbeat心跳检查接口
05
      description: heartbeat心跳检查接口
06
      tags:
07
      - HealthCheck
08
      operationId: isAlive
09
      consumes:
10
      - application/json;charset=UTF-8
11
      produces:
12
      - application/xml;charset=UTF-8
13
      x-imports: "javax.validation.constraints.Size;org.springframework.web.bind.annotation.*;"
14
      parameters:
15
      - name: paramName
16
        in: query
17
        description:
18
        required: true
19
        type: string
20
        x-onlyCustomAnnotation: true
21
        x-annotations:
22
        - "@Size"
23
        - "@RequestParam(\"filterType\")"
24
      responses:
25
        '200':
26
          description: ''
27
          schema:
28
            $ref: '#/definitions/QueryCondition'


效果：

-java代码
01
@RequestMapping(
02
        value = "",
03
        produces = { "application/xml;charset=UTF-8" },
04
        method = RequestMethod.GET)
05
@Override
06
public QueryCondition isAlive(
07
        @Size
08
        @RequestParam("filterType") String paramName)
09
    throws ServiceException
10
{
11
     
12
 
13
    return delegate.isAlive(paramName);
14
}


  [ 回目录 ]17     2.2.4   
  [ 回目录 ]17.1     新增特性   
实体类字段配置第三方类的变更
接口入参配置第三方类的变更
接口返回对象配置第三方类的变更

从2.1.31版本开始，我们提供了配置以引用第三方类。然而，由于不遵循swagger规范，使用该特性配置出的yaml在swagger编辑器上会报错。
在保留原配置的基础上，我们提供一种新方法以适配该特性。

  [ 回目录 ]17.2     特性说明   
1、实体类字段配置第三方类的变更

-powershell代码
01
UserInfo:
02
  x-imports: "java.util.Map;java.util.List;net.sf.json.JSONObject"
03
  properties:
04
    username:
05
      type: string
06
    password:
07
      type: string
08
    status:
09
      type: string
10
      maxLength: 255
11
    customType:
12
      type: object
13
      x-type: "Map<String,List<Object>>"
14
    customType1:
15
      x-type: "JSONObject"
16
      type: object
17
    customType2:
18
      x-type: "net.sf.json.JSONObject"
19
      type: object
20
    customType3:
21
      x-type: "Integer"
22
      type: object


        如上述代码所示，一个字段要配置使用第三方类，可使用 x-type 标签指定，如果被引用的类不在 java.lang 包下，请使用 x-imports 标签显示引入。
2、接口入参配置第三方类的变更

-powershell代码
01
paths:
02
  /v1/test:
03
    post:
04
      summary: 测试接口
05
      description: 测试接口
06
      tags:
07
        - MessageService
08
      operationId: test1
09
      x-imports: "net.sf.json.JSONObject"
10
      parameters:
11
        - name: info
12
          in: body
13
          description:
14
          required: true
15
          type: object
16
          x-type: "Map<String,List<JSONObject>>"
17
      responses:
18
        '200':
19
          description: ''
20
          schema:
21
            $ref: '#/definitions/UserInfo'
22
        '400':
23
          description: 请求参数非法
24
          schema:
25
            $ref: '#/definitions/FailedRsp'

使用方法跟实体类的配置类似。

3、接口返回对象配置第三方类的变更

-powershell代码
01
/v2/test:
02
  post:
03
    summary: 测试接口
04
    description: 测试接口
05
    tags:
06
      - MessageService
07
    operationId: test2
08
    x-returnType: "Map<String,List<JSONObject>>"
09
    x-returnContainerType: Map
10
    x-imports: "net.sf.json.JSONObject;java.util.*"
11
    responses:
12
      '200':
13
        description: ''
14
        schema:
15
          type: object

         配置返回类型使用 x-returnType 标签，如果返回类型是复杂类型如集合类，则需要使用 x-returnContainerType 标签指定类型，否则会报错。其它类型不需要指定。

  [ 回目录 ]18     2.2.6   
  [ 回目录 ]18.1     新增特性   
前面的版本都是写死实体类的包路径，使得所有实体都存放在同一个包下。本版本支持实体类指定包路径，类文件分开存放，结构化更明显。
  [ 回目录 ]18.2     特性说明   
示例1（使用x-subPackage属性）：
-powershell代码
01
Response:
02
  x-subPackage: home.init
03
  properties:
04
    message:
05
      type: string
06
    status:
07
      type: string
08
      enum:
09
        - OK
10
        - ERROR

如果model/pom.xml文件下配置的 invokerPackage 标签的内容为 com.huawei.clouddragon.springcloudgencodetest ，那么该实体类会被生成到 com.huawei.clouddragon.springcloudgencodetest.model.home.init 包下。

示例2（使用x-package属性）：
-powershell代码
01
Response:
02
  x-package: com.huawei.test.model.home.init
03
  properties:
04
    message:
05
      type: string
06
    status:
07
      type: string
08
      enum:
09
        - OK
10
        - ERROR

该实体类会被生成到 com.huawei.test.model.home.init 包下。

请注意：
如果两个属性都存在，则插件会取 x-package 生成代码。
虽然支持指定不同的包，但是仍然不允许实体类同名。


  [ 回目录 ]19     2.2.8   
  [ 回目录 ]19.1     新增特性   
api支持配置是否抛出ServiceException异常
api支持指定全局异常
api支持配置是否生成Mock代码
client代码支持配置生成到output目录下
yaml文件支持按数组方式配置，且支持模糊匹配
支持配置controller类是否生成ExceptionHandler方法

  [ 回目录 ]19.2     特性说明   
1、api支持配置是否抛出ServiceException异常
以前，yaml配置的接口会生成类型于 public Object get() throws ServiceException; 的方法到 .java 文件。增加 throwServiceException 配置后，可控制是否生成 throws 。默认：true

2、api支持指定全局异常
如果你期望api抛出想要的异常类。可增加 globalException 配置项。throwServiceException 为 false 时有效。默认：不可用

3、api支持配置是否生成Mock代码
有些开发者并不喜欢生成Mock代码，通过增加 generateMock 配置项，可控制生成。默认：false

4、client代码支持配置生成到output目录下
把client的代码生成到 output 目录下，schema为client时有效。
特定情况下，用户只需要client代码，同时又不想把client生成到工程外层的client目录下，可以增加 clientToOutputLoc 配置项。默认：false

5、yaml文件支持按数组方式配置，且支持模糊匹配
按数组方式指定yaml文件，增加 inputSpecs 配置项以控制。当 inputSpec 标签和 inputSpecs 标签同时存在时，插件会使用 inputSpec 的配置。

6、支持配置controller类是否生成ExceptionHandler方法
通过配置generateExceptionHandler标签，表示生成的XXXXXController.java文件里是否带有ExceptionHandler方法。默认：true

配置示例：
-xml代码
01
            <plugin>
02
                <groupId>com.huawei.tools</groupId>
03
                <artifactId>service-codegen-maven-plugin</artifactId>
04
                <version>2.2.8</version>
05
                <configuration>
06
                    <output>.</output>
07
                    <paramCheck>true</paramCheck>
08
                    <jsonSerializeNotNull>false</jsonSerializeNotNull>
09
                    <language>spring4mvc</language>
10
                    <ignoreImplClass>false</ignoreImplClass>
11
                    <invokerPackage>com.huawei.clouddragon.springcloudgencodetest</invokerPackage>
12
                    <sortParamsByRequiredFlag>true</sortParamsByRequiredFlag>
13
                    <schema>both</schema>
14
                    <generatePrimitiveType>true</generatePrimitiveType>
15
                    <throwServiceException>false</throwServiceException>
16
                    <generateMock>false</generateMock>
17
                    <clientToOutputLoc>true</clientToOutputLoc>
18
<!--                     <inputSpec>${project.basedir}/src/main/resources/</inputSpec> -->
19
                    <inputSpecs>
20
                        <item>${project.basedir}/src/main/resources/codegen*/*.yaml</item>
21
                        <item>${project.basedir}/src/main/resources/rest-services-mock.yaml</item>
22
                    </inputSpecs>
23
                    <globalException>java.lang.Exception</globalException>
24
                    <generateExceptionHandler>false</generateExceptionHandler>
25
                </configuration>
26
                <executions>
27
                    <execution>
28
                        <id>model-generate</id>
29
                        <phase>generate-sources</phase>
30
                        <goals>
31
                            <goal>generate</goal>
32
                        </goals>
33
                    </execution>
34
                </executions>
35
            </plugin>

  [ 回目录 ]20     2.2.10   
  [ 回目录 ]20.1     新增特性   
实体类字段支持是否允许toString打印。
支持生成枚举类。
支持是否生成装配request对象的代码

  [ 回目录 ]20.2     特性说明   
1、实体类字段支持是否允许toString打印。
-powershell代码
01
UserInfo:
02
  x-imports: "java.util.Map;java.util.List;net.sf.json.JSONObject"
03
  properties:
04
    username:
05
      type: string
06
    password:
07
      type: string
08
    status:
09
      type: string
10
      maxLength: 255
11
      x-isSensitive: true


通过配置 x-isSensitive ，生成的实体类的toString方法将不包含该字段。

2、支持生成枚举类。
-powershell代码
01
Colors:
02
  type: string
03
  enum: &COLORS
04
    - black
05
    - white
06
    - red
07
    - green
08
    - blue
09
    - new
10
    - if
11
    - ab cd
12
    - cd-ef

上述配置生成的代码如下：
-java代码
01
public enum Colors
02
{
03
    @JsonProperty("black") BLACK("black"),
04
    @JsonProperty("white") WHITE("white"),
05
    @JsonProperty("red") RED("red"),
06
    @JsonProperty("green") GREEN("green"),
07
    @JsonProperty("blue") BLUE("blue"),
08
    @JsonProperty("new") NEW("new"),
09
    @JsonProperty("if") IF("if"),
10
    @JsonProperty("ab cd") AB_CD("ab cd"),
11
    @JsonProperty("cd-ef") CD_EF("cd-ef"),
12
    ;
13
     
14
    private Colors(String value)
15
    {
16
        this.value = value;
17
    }
18
     
19
    private String value;
20
 
21
    public String getValue()
22
    {
23
        return value;
24
    }
25
 
26
    public void setValue(String value)
27
    {
28
        this.value = value;
29
    }
30
}

有两点需要注意的是：
现阶段仅支持string类型；
在应用枚举方面暂只能当实体类来使用。

3、支持是否生成装配request对象的代码
-xml代码
1
<generateRequestAutowire>false</generateRequestAutowire>

插件增加上述配置，在service子工程下新生成的业务类将不包含 @Autowired private HttpServletRequest request; 这样一行代码。
该配置项默认值为：true

  [ 回目录 ]21     2.2.12   
  [ 回目录 ]21.1     新增特性   
集合变量允许控制是否初始化
修复配置不抛出异常时，生成 throws false; 的问题
  [ 回目录 ]21.2     特性说明   
通过增加如下配置，可控制实体类的集合变量（list、map）是否初始化。
-xml代码
1
<initContainerType>true</initContainerType>

该配置项默认为true


  [ 回目录 ]   
  [ 回目录 ]22     2.2.21   
  [ 回目录 ]22.1     新增特性   
接口URL支持拼basePath做前缀
解决模糊匹配yaml文件功能在linux环境上不生效的问题
  [ 回目录 ]22.2     特性说明   
1、接口URL支持拼basePath做前缀
要开启此特性，请在插件配置处增加如下配置项：
-xml代码
1
<effectBasePath>true</effectBasePath>

如果某yaml文件中的basePath为： /instance/instance1 ，而某接口URL为： /interface/v1，那么最终生成的URL为：/instance/instance1/interface/v1


2、解决模糊匹配yaml文件功能在linux环境上不生效的问题

假设模糊匹配配置如下：

-xml代码
1
<inputSpecs>
2
    <item>${project.basedir}/src/main/resources/*.yaml</item>
3
</inputSpecs>

该配置在windows环境能正常搜索yaml文件，而在linux环境上则无法正常工作。此版本解决了这个问题。

  [ 回目录 ]23     2.2.23   
  [ 回目录 ] 23.1     新增特性   
由于io.swagger原因，导致多个类的data属性引用同一个类的时候，只能生成一个Data类。修改后能正常生成多个Data类。
  [ 回目录 ]23.2     特性说明   
yaml中data的写法如下，QueryCountryRsp、QueryContryHelp的data都有引用同一个类能正常生成QueryCountryRspData、QueryContryHelpData类。
-xml代码
01
yaml:
02
definitions:
03
  QueryCountryRsp:
04
    type: object
05
    properties:
06
      error:
07
        $ref: '#/definitions/JosError'
08
      data:
09
        properties:
10
          countryList:
11
            type: array
12
            items:
13
              $ref: '#/definitions/Country'
14
     
15
  QueryContryHelp:
16
    type: object
17
    properties:
18
      error:
19
        $ref: '#/definitions/JacError'
20
      data:
21
        properties:
22
          countryList:
23
            type: array
24
            items:
25
              $ref: '#/definitions/Country'        

生成的代码如下：



  [ 回目录 ]24     2.2.25   
  [ 回目录 ]24.1     新增特性   
 通过x-tag标签，可以替换tags内容使一个yaml生成一个接口类。
-xml代码
01
yaml:
02
swagger: "2.0"
03
info:
04
  description: ""
05
  version: "v1"
06
  title: "TestAPI"
07
host: "localhost:9094"
08
x-tag: MyManagement
09
basePath: "/"
10
paths:      

生成的代码如下：



  [ 回目录 ]24.2     特性说明   
@RequestHeader中加入属性name
生成的代码如下：


  [ 回目录 ]24.3    修改说明   
  修复通过x-imports继承第三方类时重复导入包的问题
-xml代码
01
yaml:
02
definitions:  
03
  QueryLayouInfoResp:
04
    x-java-class: "com.huawei.music.capabiliy.web.resp.QueryLayouInfoResp"
05
    x-imports: "com.huawei.ttmusic.common.Result;com.huawei.ttmusic.common.BaseResp;com.huawei.ttmusic.common.ContentSimpleInfo;"
06
    allOf:
07
      - $ref: "#/definitions/BaseResp"
08
      - type: object
09
        properties:
10
          result:
11
            description: "处理结果"
12
            $ref: "#/definitions/Result"
13
          contentSimpleInfos:
14
            type: array
15
            items:
16
              $ref: "#/definitions/ContentSimpleInfo"
17
         

生成的代码如下：





  [ 回目录 ]25     2.2.26   
  [ 回目录 ]25.1     新增特性   
 通过showRecluseContract标签控制是否生成@RestController、@RequestMapping 、@RequestBody、@RequestParam、@PathVariale注解。默认为true，如果不需要生成如上注解则设置为false。

插件配置：


yaml内容：



生成的代码如下：




标签： SpringCloud;插件;代码生成器	分类：操作指导
分享 (1)
收藏 (5)
赞 (1)

 提交评论 同时发到我的微博   
评论列表

等级:
1楼刘引 发表于 2018-02-07 15:47
虽然看到的太晚了，还是看到了!
好多强大的技能，真棒！！！
赞(0) 回复

等级:
2楼翁宇锋 发表于 2018-02-07 16:33
回复 刘引 发表于 2018-02-07 15:47
虽然看到的太晚了，还是看到了!
好多强大的技能，真棒！！！
朋友，你走运了
赞(0) 回复

等级:
3楼刘引 发表于 2018-02-26 18:02
使用2.2.4 版本 codegen 
mock代码里返回类型为集合类时编译报错怎么解决？谢谢 ！
赞(0) 回复

等级:
4楼翁宇锋 发表于 2018-02-27 09:50
回复 刘引 发表于 2018-02-26 18:02
使用2.2.4 版本 codegen 
mock代码里返回类型为集合类时编译报错怎么解决？谢谢 ！
什么错误？
赞(0) 回复

等级:
5楼刘引 发表于 2018-02-27 10:07
回复 翁宇锋 发表于 2018-02-27 09:50
什么错误？
有一个接口返回值类型是List<ExcelObject>
插件在mock里自动生成了List<ExcelObject>.class，eclipse编译报错（ExcelObject cannot be resolved to a variable），
手动修改为 List.class后在codeClub CI时编译报错
赞(0) 回复

等级:
6楼翁宇锋 发表于 2018-02-27 10:15
回复 刘引 发表于 2018-02-27 10:07
有一个接口返回值类型是List<ExcelObject>
插件在mock里自动生成了List<ExcelObject>.class，eclipse编译报错（ExcelObject cannot be resolved to a variable），
手动修改为 List.class后在codeClub CI时编译报错
发yaml文件给我看看
赞(0) 回复

等级:
7楼刘引 发表于 2018-05-10 10:21
泛型要在swagger中怎么描述？多谢！
生成这种的 
@JsonProperty("result")
private List result = new ArrayList();
赞(0) 回复

等级:
8楼邵佐海 发表于 2018-05-17 11:41
回复 刘引 发表于 2018-05-10 10:21
泛型要在swagger中怎么描述？多谢！
生成这种的 
@JsonProperty("result")
private List result = new ArrayList();
感谢您的提问及关于泛型的解决方法。
x-imports: "java.util.List"
properties:
result:
type: object
x-type: "List"
x-imports加上可以解决 2.2.4版本
赞(0) 回复
统计词条
作者：罗红华
浏览：2143
编辑数：151 历史版本
最后编辑时间：2018-08-23 14:43
上一篇： 流水线执行任务显示成功但实际jenkins没有调起
下一篇： 在导入APITest用例时，访问不到测试服务的代码...
关于3MS
管理规定
平台管理规定
分类管理规定
信息安全规定
知识服务
服务清单
服务申请
帮助中心
常见问题
用户指引
运营指引
联系我们
优化需求
问题反馈
管理团队
Copyright © Huawei Technologies Co., Ltd. 2017. All rights reserved.
该工具的维护主体是云龙CloudInit服务团队，有问题可以咨询维护责任人（张西营 z00317404）
1     说明
2     2.1.13
2.1     新增特性
2.2     特性说明
3     2.1.14
3.1     新增特性
3.2     特性说明
4     2.1.17
4.1     新增特性
4.2     特性说明
5     2.1.23
5.1     新增特性
5.2     特性说明
6     2.1.30
6.1     新增特性
6.2     特性说明
7     2.1.31
7.1     新增特性
7.2     特性说明
8     2.1.40
8.1     新增特性
8.2     特性说明
9     2.1.41
9.1     新增特性
9.2     特性说明
10     2.1.45
9.1     新增特性
10.2     特性说明
11     2.1.46
11.1     新增特性
11.2     特性说明
12     2.1.49
12.1     新增特性
12.2     特性说明
13     2.1.52
13.1     新增特性
13.2     特性说明
14     2.1.54
14.1     新增特性
14.2     特性说明
15     2.1.56
15.1     新增特性
15.2     特性说明
16     2.2.2
16.1     新增特性
16.2     特性说明
17     2.2.4
17.1     新增特性
17.2     特性说明
18     2.2.6
18.1     新增特性
18.2     特性说明
19     2.2.8
19.1     新增特性
19.2     特性说明
20     2.2.10
20.1     新增特性
20.2     特性说明
21     2.2.12
21.1     新增特性
21.2     特性说明
22     2.2.21
22.1     新增特性
22.2     特性说明
23     2.2.23
23.1     新增特性
23.2     特性说明
24     2.2.25
24.1     新增特性
24.2     特性说明
24.3    修改说明
25     2.2.26
25.1     新增特性
