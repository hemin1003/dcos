## Marathon-LB基本概念

Marathon-LB通过脚本`marathon_lb.py`连接到Marathon的API检索所有正在运行的应用程序，然后生成HAProxy配置并重新加载HAProxy。默认情况下，Marathon-LB绑定到每个应用程序的服务端口，并将传入请求发送给对应的应用程序实例。

应用程序通过在其Marathon程序定义中声明的服务端口（`servicePort`）来提供服务。此外，应用程序仅在具有与Marathon应用程序的标签（使用`HAPROXY_GROUP`）中定义的标记（或组）相同的LB上暴露服务。可以为特定标签标记的应用针对性的调整HAProxy参数。

要创建一个或多个虚拟主机，需要在应用程序的Marathon程序定义中设置HAPROXY\_{n}\_VHOST标签。配置了虚拟主机的应用除了在服务端口（`servicePort`）提供服务外，也通过80和443端口公开服务。可以在HAPROXY\_{n}\_VHOST中使用逗号作为主机域名之间的分隔符来指定多个虚拟主机。

所有应用程序也使用HTTP头`X-Marathon-App-Id`在端口9091上公开。参考[HAProxy配置模板](/dcos-component-marathon-lb-template.md)中的HAPROXY\_HTTP\_FRONTEND\_APPID\_HEAD配置。



