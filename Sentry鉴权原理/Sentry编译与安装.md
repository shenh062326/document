## Sentry开发与安装配置
* 代码https://github.com/apache/sentry.git
* 编译 mvn install -DskipTests，安装包在 sentry/sentry-dist/target/
* 配置 sentry-site.xml, 默认为本地derby数据库，也可以使用mysql，mysql是共用的。
```
<?xml version="1.0" encoding="utf-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
  <property>
    <name>sentry.service.admin.group</name>  
    <value>admin</value>
  </property>  
  <property>
    <name>sentry.service.allow.connect</name>  
    <value>hive,admin</value>
  </property>  
  <property>
    <name>sentry.service.reporting</name>  
    <value>JMX</value>
  </property>  
  <property>
    <name>sentry.service.server.rpc-address</name>  
    <value>localhost</value>
  </property>  
  <property>
    <name>sentry.service.server.rpc-port</name>  
    <value>8038</value>
  </property>  
  <property>
    <name>sentry.store.group.mapping</name>  
    <value>org.apache.sentry.provider.common.HadoopGroupMappingService</value>
  </property>  
  <property>
    <name>sentry.hive.server</name>  
    <value>server1</value>
  </property>  
  <!-- 配置Webserver -->  
  <property>
    <name>sentry.service.web.enable</name>  
    <value>true</value>
  </property>  
  <property>
    <name>sentry.service.web.port</name>  
    <value>51000</value>
  </property>  
  <property>
    <name>sentry.service.web.authentication.type</name>  
    <value>NONE</value>
  </property>  
  <property>
    <name>sentry.service.web.authentication.kerberos.principal</name>  
    <value> </value>
  </property>  
  <property>
    <name>sentry.service.web.authentication.kerberos.keytab</name>  
    <value> </value>
  </property>  
  <!--配置认证-->  
  <property>
    <name>sentry.service.security.mode</name>  
    <value>none</value>
  </property>  
  <property>
    <name>sentry.service.server.principal</name>  
    <value> </value>
  </property>  
  <property>
    <name>sentry.service.server.keytab</name>  
    <value> </value>
  </property>  

  <property>
    <name>sentry.store.jdbc.url</name>
    <value>jdbc:derby:;databaseName=metastore_db;create=true</value>
  </property>
  <property>
    <name>sentry.store.jdbc.driver</name>
    <value>org.apache.derby.jdbc.EmbeddedDriver</value>
  </property>
  <property>
    <name>sentry.store.jdbc.user</name>
    <value>sentry</value>
  </property>
  <property>
    <name>sentry.store.jdbc.password</name>
    <value>test</value>
  </property>

<!--
  <property>
    <name>sentry.store.jdbc.url</name>  
    <value>jdbc:mysql://db.mid.baba.net:3306/test</value>
  </property>
  <property>
    <name>sentry.store.jdbc.driver</name>  
    <value>com.mysql.jdbc.Driver</value>
  </property>  
  <property>
    <name>sentry.store.jdbc.user</name>  
    <value>test</value>
  </property>  
  <property>
    <name>sentry.store.jdbc.password</name>  
    <value>test</value>
  </property>
-->

    <property>
       <name>sentry.service.client.server.rpc-port</name>
       <value>8038</value>
    </property>
    <property>
       <name>sentry.service.client.server.rpc-addresses</name>
       <value>localhost</value>
    </property>
    <property>
       <name>sentry.service.client.server.rpc-connection-timeout</name>
       <value>200000</value>
    </property>

    <!--以下是客户端配置-->
    <property>
        <name>sentry.provider</name>
        <value>org.apache.sentry.provider.file.HadoopGroupResourceAuthorizationProvider</value>
    </property>
    <property>
        <name>sentry.hive.provider.backend</name>
        <value>org.apache.sentry.provider.db.SimpleDBProviderBackend</value>
    </property>
    <property>
        <name>sentry.metastore.service.users</name>
        <value>hive</value><!--queries made by hive user (beeline) skip meta store check-->
    </property>
      <property>
        <name>sentry.hive.server</name>
        <value>server1</value>
      </property>
     <property>
        <name>sentry.hive.testing.mode</name>
        <value>true</value>
     </property>

</configuration>
```
* 初始化DB命令：
```
derby
bin/sentry --command schema-tool --conffile conf/sentry-site.xml --dbType derby --initSchema

mysql
bin/sentry --command schema-tool --conffile conf/sentry-site.xml --dbType mysql --initSchema
```

* 启动sentry
```
bin/sentry --command service --conffile conf/sentry-site.xml
```
* 登录网页 localhost:51000

* 配置HiveServer

其中hive版本需要2.0.0级以上，hive-site.xml如下：
```
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
<property>
    <name>hive.metastore.warehouse.dir</name>
    <value>/tmp/</value>
</property>
<property>
  <name>sentry.hive.testing.mode</name>
  <value>true</value>
</property>
    <property>
      <name>hive.server2.authentication</name>
      <value>NONE</value>
    </property>

    <property>
      <name>hive.metastore.pre.event.listeners</name>
      <value>org.apache.sentry.binding.metastore.MetastoreAuthzBinding</value>
    </property>
    <property>
      <name>hive.metastore.event.listeners</name>
      <value>org.apache.sentry.binding.metastore.SentryMetastorePostEventListener</value>
    </property>

    <property>
      <name>hive.server2.enable.impersonation</name>
      <value>true</value>
    </property>
    <property>
        <name>hive.security.authorization.task.factory</name>
        <value>org.apache.sentry.binding.hive.SentryHiveAuthorizationTaskFactoryImpl</value>
    </property>
    <property>
        <name>hive.server2.session.hook</name>
        <value>org.apache.sentry.binding.hive.HiveAuthzBindingSessionHook</value>
    </property>
    <property>
        <name>hive.sentry.conf.url</name>
        <value>file:///Users/hongshen/runtime/apache-sentry-1.8.0_ant-bin/conf/sentry-site.xml</value>
    </property>
</configuration>
```
* 复制jar包到HiveServer的lib下,包括
```
hongdeMacBook-Pro:sentry2hiveJar hongshen$ ll
total 7200
drwxr-xr-x  22 hongshen  staff      748 Nov  8 17:13 ./
drwxr-xr-x  14 hongshen  staff      476 Nov  8 16:42 ../
-rw-r--r--   1 hongshen  staff   111969 Nov  8 17:13 commons-pool2-2.4.2.jar
-rw-r--r--   1 hongshen  staff   163852 Nov  8 16:53 sentry-binding-hive-1.8.0_ant.jar
-rw-r--r--   1 hongshen  staff    36569 Nov  8 16:53 sentry-binding-hive-common-1.8.0_ant.jar
-rw-r--r--   1 hongshen  staff    14932 Nov  8 16:53 sentry-binding-hive-conf-1.8.0_ant.jar
-rw-r--r--   1 hongshen  staff    26496 Nov  8 16:53 sentry-binding-hive-follower-1.8.0_ant.jar
-rw-r--r--   1 hongshen  staff    77506 Nov  8 16:53 sentry-core-common-1.8.0_ant.jar
-rw-r--r--   1 hongshen  staff    32294 Nov  8 16:53 sentry-core-model-db-1.8.0_ant.jar
-rw-r--r--   1 hongshen  staff    20518 Nov  8 16:53 sentry-core-model-indexer-1.8.0_ant.jar
-rw-r--r--   1 hongshen  staff    22799 Nov  8 16:53 sentry-core-model-kafka-1.8.0_ant.jar
-rw-r--r--   1 hongshen  staff    21502 Nov  8 16:53 sentry-core-model-search-1.8.0_ant.jar
-rw-r--r--   1 hongshen  staff    21777 Nov  8 16:53 sentry-core-model-sqoop-1.8.0_ant.jar
-rw-r--r--   1 hongshen  staff    13274 Nov  8 17:11 sentry-policy-common-1.8.0_ant.jar
-rw-r--r--   1 hongshen  staff    10634 Nov  8 17:11 sentry-policy-engine-1.8.0_ant.jar
-rw-r--r--   1 hongshen  staff    13462 Nov  8 17:11 sentry-policy-indexer-1.8.0_ant.jar
-rw-r--r--   1 hongshen  staff    14093 Nov  8 16:53 sentry-provider-cache-1.8.0_ant.jar
-rw-r--r--   1 hongshen  staff    25054 Nov  8 16:53 sentry-provider-common-1.8.0_ant.jar
-rw-r--r--   1 hongshen  staff  2561286 Nov  8 16:53 sentry-provider-db-1.8.0_ant.jar
-rw-r--r--   1 hongshen  staff    23302 Nov  8 16:53 sentry-provider-file-1.8.0_ant.jar
-rw-r--r--   1 hongshen  staff    18037 Nov  8 16:53 shiro-config-core-1.4.0.jar
-rw-r--r--   1 hongshen  staff   410541 Nov  8 16:53 shiro-core-1.4.0.jar
```

* 启动HiveServer
```
bin/hiveserver2 start --hiveconf hive.root.logger=INFO,console
```

* 启动beeline连上HiveServer
```
bin/beeline -u "jdbc:hive2://localhost:10000/" -n hive -p hive -d org.apache.hive.jdbc.HiveDriver
```
* 提交sql验证
使用超级账户“hive”,创建并赋权，然后使用test验证是否有权限。参考http://blog.javachen.com/2015/04/30/test-hive-with-sentry.html
```
create database sensitive;

create table sensitive.events (
    ip STRING, country STRING, client STRING, action STRING
  ) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';

load data local inpath '/tmp/events.csv' overwrite into table sensitive.events;
create database filtered;
create view filtered.events as select country, client, action from sensitive.events;
create view filtered.events_usonly as select * from filtered.events where country = 'US';

create role admin_role;
GRANT ALL ON SERVER server1 TO ROLE admin_role;
GRANT ROLE admin_role TO GROUP admin;
GRANT ROLE admin_role TO GROUP hive;

create role test_role;
GRANT ALL ON DATABASE filtered TO ROLE test_role;
GRANT ROLE test_role TO GROUP test;

show roles;
SHOW GRANT ROLE test_role;
SHOW GRANT ROLE admin_role;
...
```
