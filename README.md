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


restTemplate做前台服务转发          https://blog.csdn.net/yiifaa/article/details/77939282
