# 5.1.1. 磁盘报警处理

## 5.1.1.1 磁盘报警说明

#### 报警说明

系统每xx时间周期性检查磁盘使用率，并把磁盘使用率和设定的报警阈值（默认80%）进行比较，当检测到磁盘使用率超过或等于阈值时产生报警。



#### 报警信息

| 报警标题      | 报警级别 | 是否需要手动干预 |
| ------------- | -------- | ---------------- |
| HighDiskUsage | 重要     | 是               |



#### 报警参数

| 参数名称    | 参数含义     |
| ----------- | ------------ |
| alertname   | 警告名称     |
| instance    | 报警节点     |
| monitor     | 报警监控     |
| mountpoint  | 磁盘报警地址 |
| description | 报警描述     |
| summary     | 报警总结     |



#### 可能原因

- 报警阈值配置的不合理。（默认80%）
- 磁盘资源无法满足当前业务数据量，磁盘使用率达到上限，需要考虑扩展磁盘。
- 短时间内产生大量日志，自动清理机制无法及时处理，需要手动清理。



## 5.1.1.2 磁盘报警案例

### 5.1.1.2.1 /var/lib/docker 磁盘报警

#### 对系统的影响

该节点上的应用不可更新和新增。



#### 检查步骤

第一步：检查 /var/lib/docker 的数据保存策略是否正常Work，默认3天。



#### 操作步骤

第一步：使用SSH工具，以**root**用户登录报警所在节点。

第二步：执行 df -h，查看系统磁盘分区的使用信息。

第三步：执行 docker system prune -f 清理磁盘。

```sql
[dcos@agent010 log]$ docker system prune -f
```

第四步：如果第三步执行完了，磁盘使用仍然超过阈值，执行  docker system prune -a 。

```sql
[dcos@agent010 log]$ docker system prune -a
```

第五步：如果第四步执行完了，磁盘使用仍然超过阈值，考虑扩容 /var/lib/docker。



### 5.1.1.2.2 /var/log 磁盘报警

#### 对系统的影响

该节点上应用不可用，且ETL作业不可运行。



#### 检查步骤

第一步：检查 /var/log 的数据保存策略是否正常Work，默认3天。



#### 操作步骤

第一步：使用SSH工具，以**root**用户登录报警所在节点。

第二步：执行 du -h --max-depth=1 /var/log，查看系统磁盘分区的使用信息。

```sql
[dcos@agent010 log]$ du -h --max-depth=1 /var/log
0	/var/log/chrony
4.0K	/var/log/tuned
6.0G	/var/log/glusterfs
629M	/var/log/mesos
8.0K	/var/log/rexray
0	/var/log/admin-message-board
0	/var/log/admin-kerberos-http
15G	/var/log/admin-bdos-core
8.0K	/var/log/admin-ranger-admin
36M	/var/log/kafka-connect
17M	/var/log/test-linktime-hs2-group
456K	/var/log/admin-hue
1.3G	/var/log/admin-linktime-hs2-group
491M	/var/log/admin-basic-etl-worker
0	/var/log/admin-etl-master
```

第三步：手动 rm -f 可删除的历史日志数据。

第四步：如果第三步执行完了，磁盘使用仍然超过阈值，考虑扩容  /var/log。



### 5.1.1.2.3 /usr/local/hadoop 磁盘报警

#### 对系统的影响

该节点上NodeManager不可用，会影响ETL运行。



#### 检查步骤

第一步：检查Spark History Service的数据保存策略是否正常Work，默认3天。

第二步：检查Hadoop下的 /tmp 目录的数据保存策略是否正常Work，默认3天。

第三步：检查Hive表的保存策略是否正常Work，根据数据类型自定义。



#### 操作步骤

第一步：使用SSH工具，以**root**用户登录报警所在节点。

第二步：执行 /etc/hadoop/bin/hadoop dfs -du -h /

```sql
[dcos@agent001 ~]$ /etc/hadoop/bin/hadoop dfs -du -h /
DEPRECATED: Use of this script to execute hdfs command is deprecated.
Instead use the hdfs command for it.

0        /centre
49.9 G   /historyservice
950.2 K  /hive
0        /kafka
0        /kafka-http-gateway
58.5 K   /kafka-topic-log
0        /sqoop
0        /system
0        /test
30.2 G   /tmp
13.7 K   /udf
0        /upload
306.3 G  /user
```

第三步：进一步查看各个目录下 /historyservice 、/tmp、/user 的数据，手动 rm -f 可删除的HDFS文件。

第四步：如果第三步执行完了，磁盘使用仍然超过阈值，考虑扩容  /usr/local/hadoop。



### 5.1.1.2.4 /opt/linktimecloud 磁盘报警

#### 对系统的影响

应用不可用，会影响ETL运行。



#### 检查步骤

第一步：检查 /opt/linktimecloud 中自定义应用的数据保存策略是否正常Work，默认3天。



#### 操作步骤

第一步：使用SSH工具，以**root**用户登录报警所在节点。

第二步：执行 du -h --max-depth=1 /opt/linktimecloud

```
[dcos@agent001 ~]$ du -h --max-depth=1 /opt/linktimecloud
180G	/opt/linktimecloud/bdos
185M	/opt/linktimecloud/linktime-mysql
42K	/opt/linktimecloud/kerberos
8.0K	/opt/linktimecloud/postgresql
37K	/opt/linktimecloud/prometheus
18M	/opt/linktimecloud/grafana
23K	/opt/linktimecloud/alertmanager
76K	/opt/linktimecloud/hue
247M	/opt/linktimecloud/labeling
81M	/opt/linktimecloud/zk_backup
4.0K	/opt/linktimecloud/hadoop
4.6M	/opt/linktimecloud/data-science
4.0K	/opt/linktimecloud/.trashcan
180G	/opt/linktimecloud
```

第三步：执行 du -h --max-depth=1 /opt/linktimecloud/bdos/web/upload

```
[dcos@agent001 upload]$ du -h --max-depth=1 /opt/linktimecloud/bdos/web/upload
24K	/opt/linktimecloud/bdos/web/upload/public
9.7G	/opt/linktimecloud/bdos/web/upload/developers
59M	/opt/linktimecloud/bdos/web/upload/admin
1.3G	/opt/linktimecloud/bdos/web/upload/backup-admin
11G	/opt/linktimecloud/bdos/web/upload
```

第四步：进一步查看各个目录下 /developers 、/admin 的数据，手动 rm -f 可删除的上传文件。

第五步：如果第四步执行完了，磁盘使用仍然超过阈值，考虑扩容  /opt/linktimecloud。

