## 服务的管理

卸载服务



docker run mesosphere\/janitor \/janitor.py -r &lt;service\_name&gt;-role -p &lt;service\_name&gt;-principal -z dcos-service-&lt;service\_name&gt;

you may still use \`dcos package uninstall &lt;framework-name&gt;\` to uninstall the framework, probably.

If that doesn’t work, then you’ll have to use the \`\/teardown\` endpoint.

You may need an auth\_token for the janitor.

to do that run \`docker run mesosphere\/janitor \/janitor.py -r &lt;service\_name&gt;-role -p &lt;service\_name&gt;-principal -z dcos-service-&lt;service\_name&gt; --auth\_token=&lt;token&gt;\`

dcos config show core.dcos\_acs\_token

