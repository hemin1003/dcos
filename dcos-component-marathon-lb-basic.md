## Marathon-LB基本概念

Marathon-LB通过脚本[`marathon_lb.py`](https://github.com/mesosphere/marathon-lb/blob/master/marathon_lb.py)连接到Marathon的API检索所有正在运行的应用程序，然后生成HAProxy配置并重新加载HAProxy。默认情况下，Marathon-LB绑定到每个应用程序的服务端口，并将传入请求发送给对应的应用程序实例。

应用程序通过在其Marathon程序定义中声明的服务端口（`servicePort`）来提供服务。此外，应用程序仅在具有与Marathon应用程序的标签（使用`HAPROXY_GROUP`）中定义的标记（或组）相同的LB上暴露服务。可以为特定标签标记的应用针对性的调整HAProxy参数。

要创建一个或多个虚拟主机，需要在应用程序的Marathon程序定义中设置HAPROXY\_{n}\_VHOST标签。配置了虚拟主机的应用除了在服务端口（`servicePort`）提供服务外，也通过80和443端口公开服务。可以在HAPROXY\_{n}\_VHOST中使用逗号作为主机域名之间的分隔符来指定多个虚拟主机。

所有应用程序也使用HTTP头`X-Marathon-App-Id`在端口9091上公开。参考[HAProxy配置模板](/dcos-component-marathon-lb-template.md)中的HAPROXY\_HTTP\_FRONTEND\_APPID\_HEAD配置。

可以通过`http://<public-node-ip>:9090/haproxy?stats`获取HAProxy的统计信息。可以通过`http://<public-node-ip>:9090/_haproxy_getconfig`获取当前HAProxy的配置。

部署

在选择Marathon-LB的部署方式前，首先了解一下Marathon-LB如何运行。Marathon-LB的入口程序是marathon-lb.py。可以选择**直接调用**或通过打包好的**Docker镜像（****[mesosphere\/marathon-lb](https://hub.docker.com/r/mesosphere/marathon-lb)****）**来启动。通过Marathon-LB容器的[Dockerfile](https://github.com/mesosphere/marathon-lb/blob/master/Dockerfile)定义可以看到，容器启动是通过在[run脚本](https://github.com/mesosphere/marathon-lb/blob/master/run)中调用marathon-lb.py并传递一组默认的参数：

脚本run：

```
......
exec /marathon-lb/marathon_lb.py \ 
    --syslog-socket $SYSLOG_SOCKET \ 
    --haproxy-config /marathon-lb/haproxy.cfg \ 
    --ssl-certs "${SSL_CERTS}" \ 
    --command "sv reload ${HAPROXY_SERVICE}" \ 
    $ARGS 
......
```

Dockerfile:

```
WORKDIR /marathon-lb 

ENTRYPOINT [ "/marathon-lb/run" ] 

CMD [ "sse", "--health-check", "--group", "external" ]
 
EXPOSE 80 443 9090 9091
```

在DC\/OS中，Marathon-LB以Docker镜像的方式部署，并且在Universe中存在预定义的包。可以通过DC\/OS CLI或WEB UI部署Marathon-LB。

dcos package install marathon-lb

如果要调整Marathon-LB的某些默认配置，可以通过下述命令检查当前Marathon-LB应用定义所支持的自定义参数：



