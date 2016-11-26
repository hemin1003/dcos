## MLB高级特性

### 为Marathon-LB启用SSL

使用Let's Encrypt自动维护SSL Certificate，并配置marathon-lb使用cert启用SSL。

https:\/\/mesosphere.com\/blog\/2016\/04\/06\/lets-encrypt-dcos\/

https:\/\/github.com\/mesosphere\/letsencrypt-dcos

### 零停机部署

Marathon-lb is able to perform canary style blue\/green deployment with zero downtime. To execute such deployments, you must follow certain patterns when using Marathon.

The deployment method is described [in this Marathon document](https://mesosphere.github.io/marathon/docs/blue-green-deploy.html). Marathon-lb provides an implementation of the aforementioned deployment method with the script [`zdd.py`](https://github.com/mesosphere/marathon-lb/blob/master/zdd.py). To perform a zero downtime deploy using `zdd.py`, you must:

* Specify the `HAPROXY_DEPLOYMENT_GROUP` and `HAPROXY_DEPLOYMENT_ALT_PORT` labels in your app template
  * `HAPROXY_DEPLOYMENT_GROUP`: This label uniquely identifies a pair of apps belonging to a blue\/green deployment, and will be used as the app name in the HAProxy configuration
  * `HAPROXY_DEPLOYMENT_ALT_PORT`: An alternate service port is required because Marathon requires service ports to be unique across all apps

* Only use 1 service port: multiple ports are not yet implemented
* Use the provided `zdd.py` script to orchestrate the deploy: the script will make API calls to Marathon, and use the HAProxy stats endpoint to gracefully terminate instances
* The marathon-lb container must be run in privileged mode \(to execute `iptables` commands\) due to the issues outlined in the excellent blog post by the [Yelp engineering team found here](http://engineeringblog.yelp.com/2015/04/true-zero-downtime-haproxy-reloads.html)
* If you have long-lived TCP connections using the same HAProxy instances, it may cause the deploy to take longer than necessary. The script will wait up to 5 minutes \(by default\) for connections to drain from HAProxy between steps, but any long-lived TCP connections will cause old instances of HAProxy to stick around.

An example minimal configuration for a [test instance of nginx is included here](https://github.com/mesosphere/marathon-lb/blob/master/tests/1-nginx.json). You might execute a deployment from a CI tool like Jenkins with:

```
./zdd.py -j 1-nginx.json -m http://master.mesos:8080 -f -l http://marathon-lb.marathon.mesos:9090 --syslog-socket /dev/null

```

Zero downtime deployments are accomplished through the use of a Lua module, which reports the number of HAProxy processes which are currently running by hitting the stats endpoint at the `/_haproxy_getpids`. After a restart, there will be multiple HAProxy PIDs until all remaining connections have gracefully terminated. By waiting for all connections to complete, you may safely and deterministically drain tasks. A caveat of this, however, is that if you have any long-lived connections on the same LB, HAProxy will continue to run and serve those connections until they complete, thereby breaking this technique.

The ZDD script includes the ability to specify a pre-kill hook, which is executed before draining tasks are terminated. This allows you to run your own automated checks against the old and new app before the deploy continues.

### **Traffic Splitting Between Blue\/Green Apps**

Zdd has support to split the traffic between two versions of same app \(version 'blue' and version 'green'\) by having instances of both versions live at the same time. This is supported with the help of the `HAPROXY_DEPLOYMENT_NEW_INSTANCES` label.

When you run zdd with the `--new-instances` flag, it creates only the specified number of instances of the new app, and deletes the same number of instances from the old app \(instead of the normal, create all instances in new and delete all from old approach\), to ensure that the number of instances in new app and old app together is equal to `HAPROXY_DEPLOYMENT_TARGET_INSTANCES`.

Example: Consider the same nginx app example where there are 10 instances of nginx running image version v1, now we can use zdd to create 2 instances of version v2, and retain 8 instances of V1 so that traffic is split in ratio 80:20 \(old:new\).

Creating 2 instances with new version automatically deletes 2 instances in existing version. You could do this using the following command:

```
$ ./zdd.py -j 1-nginx.json -m http://master.mesos:8080 -f -l http://marathon-lb.marathon.mesos:9090 --syslog-socket /dev/null --new-instances 2
```

This state where you have instances of both old and new versions of same app live at the same time is called hybrid state.

When a deployment group is in hybrid state, it needs to be converted into completely current version or completely previous version before deploying any further versions, this could be done with the help of the `--complete-cur` and `--complete-prev` flags in zdd.

When you run the below command, it converts all instances to new version so that traffic split ratio becomes 0:100 \(old:new\) and it deletes the old app. This is graceful as it follows usual zdd procedure of waiting for tasks\/instances to drain before deleting them.

```
$ ./zdd.py -j 1-nginx.json -m http://master.mesos:8080 -f -l http://marathon-lb.marathon.mesos:9090 --syslog-socket /dev/null --complete-cur
```

Similarly you can use `--complete-prev` flag to convert all instances to old version \(and this is essentially a rollback\) so that traffic split ratio becomes 100:0 \(old:new\) and it deletes the new app.

Currently only one hop of traffic split is supported, so you can specify the number of new instances \(directly proportional to traffic split ratio\) only when app is having all instances of same version \(completely blue or completely green\). This implies `--new-instances` flag cannot be specified in hybrid mode to change traffic split ratio \(instance ratio\) as updating Marathon label \(`HAPROXY_DEPLOYMENT_NEW_INSTANCES`\) currently triggers new deployment in marathon which will not be graceful. Currently for the example mentioned, the traffic split ratio is 100:0 -&gt; 80:20 -&gt; 0:100, where there is only one hop when both versions get traffic simultaneously.

### 参考

https:\/\/docs.mesosphere.com\/1.9\/usage\/service-discovery\/marathon-lb\/advanced\/

