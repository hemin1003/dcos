## 服务的管理

卸载服务

docker run mesosphere\/janitor \/janitor.py -r &lt;service\_name&gt;-role -p &lt;service\_name&gt;-principal -z dcos-service-&lt;service\_name&gt;

you may still use \`dcos package uninstall &lt;framework-name&gt;\` to uninstall the framework, probably.

If that doesn’t work, then you’ll have to use the \`\/teardown\` endpoint.

You may need an auth\_token for the janitor.

to do that run \`docker run mesosphere\/janitor \/janitor.py -r &lt;service\_name&gt;-role -p &lt;service\_name&gt;-principal -z dcos-service-&lt;service\_name&gt; --auth\_token=&lt;token&gt;\`

dcos config show core.dcos\_acs\_token


Cassandra default: 

```
docker run mesosphere/janitor /janitor.py -r cassandra-role -p cassandra-principal -z dcos-service-cassandra
```

HDFS default: 

```
docker run mesosphere/janitor /janitor.py -r hdfs-role -p hdfs-principal -z dcos-service-hdfs
```

Kafka default: 

```
docker run mesosphere/janitor /janitor.py -r kafka-role -p kafka-principal -z dcos-service-kafka
```

Custom values: 

```
docker run mesosphere/janitor /janitor.py -r <custom_role> -p <custom_principal> -z dcos-service-<custom_service_name>
```

服务的自动更新

可以将服务配置保存到Git仓库（Gitlab）中，当服务配置发生变更时，可以使用Gitlab CI 或 Jenkins的任务部署变更的配置并调用Marathon的API重新启动服务。




