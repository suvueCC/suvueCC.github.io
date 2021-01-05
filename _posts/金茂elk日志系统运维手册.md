## 一、elk stack简介  
“ELK”是三个开源项目的首字母缩写，这三个项目分别是：Elasticsearch、Logstash 和 Kibana。  
- elasticsearch 是一个搜索和分析引擎。  
- Logstash 是服务器端数据处理管道，能够同时从多个来源采集数据，转换数据，然后将数据发送到诸如 Elasticsearch 等“存储库”中。  
- Kibana 则可以让用户在 Elasticsearch 中使用图形和图表对数据进行可视化。
- Beats 轻量型的单一功能数据采集器，由于Logstash很耗性能，beats应运而生。  
然而ELK 这个名称又要变了，的确如此。把它叫做 BELK？BLEK？ELKB？当时的确有过继续沿用首字母缩写的想法。然而，扩展速度如此之快，一直采用首字母缩写的确不是长久之计。  
就这样，Elastic Stack 这个名字应运而生了！！！ 
和用户一直以来熟知并喜爱的开源产品一模一样，只是集成程度更高了，功能更加强大了，入门也更加容易了，而且可以带来无限可能。

## 二、架构说明
### 2.1 oracle日志收集架构图：
![202011281111未命名文件](https://gitee.com/zhaokeyan/pic_repo/raw/master/uPic/%202020%2011%2028%2011%2011%E6%9C%AA%E5%91%BD%E5%90%8D%E6%96%87%E4%BB%B6.png)  
### 2.2 idm生产环境的Nginx日志收集架构图：
![202011281126idm生产Nginx日志收集](https://gitee.com/zhaokeyan/pic_repo/raw/master/uPic/%202020%2011%2028%2011%2026idm%E7%94%9F%E4%BA%A7Nginx%E6%97%A5%E5%BF%97%E6%94%B6%E9%9B%86.png)  
### 2.3 F5生产日志收集架构图：  
![202011281135F5日志收集](https://gitee.com/zhaokeyan/pic_repo/raw/master/uPic/%202020%2011%2028%2011%2035F5%E6%97%A5%E5%BF%97%E6%94%B6%E9%9B%86.png)  
大致流程介绍：  
1、每个应用服务器上都安装Filebeat组件，由于Logstash在进行清洗和过滤数据的时候比较耗用系统资源，如果直接把Logstash和应用系统部署在一起，可能会影响应用系统的运行，所以日志采集组件基本都是由轻量级的Filebeat替代。  
2、Filebeat组件将收集到的日志发送到kafka集群消息队列，注意，idm生产日志通过了一层Nginx代理，再流入到kafka集群，Filebeat可以通过fields.app(自定义属性)设置当前收集日志属于哪个应用，通过fields.topic(自定义属性)设置当前收集日志发送给kafka的哪个主题。  
3、Logstash从Kafka消息队列中采集日志信息，通过input.Kafka输入流来采集不同主题的日志，然后发送到es集群，针对不同的应用创建不同的索引文件。  
4、通过Kibana管理平台进行日志的搜索、可视化分析，以及用户的创建及权限设置。  
## 三、环境说明  
  
| IP | SystemInfo | Roles
| :-----: | :-----: | :----: |
| 10.160.145.92 | CentOs 7.8 | ES(master) 、Elastcisearch-head插件|
| 10.160.145.93 | CentOs 7.8 | ES(slave) |
| 10.160.145.94 | CentOs 7.8 | ES(slave) | 
| 10.160.145.95 | CentOs 7.8 | Kafka、logstash、kibana、kafka可视化工具 | 
| 10.160.145.96 | CentOs 7.8 | Kafka、logstash | 
| 10.160.145.97 | CentOs 7.8 | Kafka、logstash | 
  
## 四、filebeat要点说明  
部署新的业务时，可以直接把filebeat整个目录拷贝过去，然后在新的机器上 rm -rf ${filebeat_home}/data/registry,
再在${filebeat_home}/filebeat.yml文件中配置自己想要的配置即可。  
一般常用的主要配置如下：  
```yaml
filebeat.inputs:
- type: log
  enabled: true
  paths:
# paths为需要采集的日志的路径，可配置多个（每行以'-'开头，需注意格式，也可使用"*.log"通配符）
    - /usr/local/nginx/logs/access.log
  fields:
# fields为自定义的属性，最后可传递到kibana上进行分类筛选使用
    app: IDMPortal02
    app_ip: 10.6.21.209
    app_port: 443
    kafka_topic: idmportal02-prod-ngxaccesslogv5
# tail_files表示是否从文件开头读取，历史日志的话一定是全部都要读取的，所以这里配置false，再次重启的话也不用进行更改，filebaet会
# 对每个文件标记上次读取的位置，所以tail_files属性只对第一次启动有效
  tail_files: false
# multiline为多行匹配配置
# multiline.pattern为多行匹配的正则，系统会认为一行日志就是一条日志，这往往不是我们想要的结果，
# 所以我们要根据实际情况编写正则，去匹配业务上的一条日志，比如Nginx是以时间开头，代表一条日志的开始，所以我们就要根据Nginx的时间格式，去写正则
# 切记：正则只支持R2,也就是简单的匹配规则，我的理解是这样的
  multiline.pattern: '^\d{1,3}\.\d{1,3}.\d{1,3}.\d{1,3}'
# 下面这两个配置可以不用动
  multiline.negate: true
  multiline.match: after

# 下面和上面是一样的，去采集别的类型日志
- type: log
  enabled: true
  paths:
    - /usr/local/nginx/logs/error.log
  fields:
    app: IDMPortal02
    app_ip: 10.6.21.209
    app_port: 443
    kafka_topic: idmportal02-prod-ngxerrorlogv5
  tail_files: false
  multiline.pattern: '^[0-9]{4}/[0-9]{2}/[0-9]{2} [0-9]{2}:[0-9]{2}:[0-9]{2}'
  multiline.negate: true
  multiline.match: after

# 输出到kafka
output.kafka:
  enabled: true
# DMZ区的业务系统，需要走Nginx代理，用下面的hosts配置
  hosts: ["10.6.21.65:9092"]
# 非DMZ区的业务系统，不需要代理，用下面的hosts配置
  hosts: ["10.160.145.95:9092","10.160.145.96:9092","10.160.145.97:9092"]
  topic: '%{[fields][kafka_topic]}'
  compression: gzip
  compression_level: 4
```  
我们使用了nginx代理了kafka集群，所以我们还需要配置一下/etc/hosts文件：  
DMZ区走nginx代理的，用下面的配置：  
```yaml
#elk kafka cluster  
10.6.21.65 KAFKA-LOGSTASH-01  
10.6.21.65 KAFKA-LOGSTASH-02  
10.6.21.65 KAFKA-LOGSTASH-03  
```  
非DMZ区的用下面的配置：  
```yaml
#elk kafka cluster  
10.160.145.95 KAFKA-LOGSTASH-01  
10.160.145.96 KAFKA-LOGSTASH-02  
10.160.145.97 KAFKA-LOGSTASH-03  
```  
配置完成之后，就可以启动了，下面是常用命令： 
启动   
```text
# cd ${filebeat_home}/ 
# nohup ./filebeat -e -c filebeat.yml -d "*" >/dev/null &
```  
停止  
```text
# kill -9 进程号
```  

注意事项：
1、比如filebeat想再次从头读取某一文件，可以 rm -rf ${filebeat_home}/data/registry目录  
2、filebeat启动日志可能会把业务系统的磁盘打满，所以启动命令中改为 >/dev/null  

## kafka要点说明  
安装目录  
```shell script
[root@kafka-logstash-01 kafka]# pwd  
/usr/local/elkstack/kafka
```  
常用命令（在/usr/local/elkstack/kafka下执行）  
```text
# 启动Zookeeper
[root@kafka-logstash-01 kafka]# bin/zookeeper-server-start.sh -daemon config/zookeeper.properties  
这里的-daemon参数，可以在后台启动Zookeeper，不必打印启动日志到控制台，下同，输出的信息在保存在执行目录的logs/zookeeper.out文件中。  
# 关闭 Zookeeper  
[root@kafka-logstash-01 kafka]# bin/zookeeper-server-stop.sh -daemon config/zookeeper.properties
# 启动kafka
[root@kafka-logstash-01 kafka]# bin/kafka-server-start.sh -daemon config/server.properties  
# 查看kafka日志  
[root@kafka-logstash-01 kafka]# tail -f logs/server.log
# 关闭kafka
[root@kafka-logstash-01 kafka]# bin/kafka-server-stop.sh config/server.properties
# 创建topic 名称为test 指定分区和副本数为3，也可不指定，在kaf配置文件中设置值
[root@kafka-logstash-01 kafka]# bin/kafka-topics.sh --create --zookeeper 10.160.145.95:2181 --replication-factor 3 --partitions 3 --topic test
# 查看topic列表  
[root@kafka-logstash-01 kafka]# bin/kafka-topics.sh --list --zookeeper 10.160.145.95:2181  
# 创建消费者,从头消费  
[root@kafka-logstash-01 kafka]# bin/kafka-console-consumer.sh --bootstrap-server 10.160.145.95:9092 --topic test --from-beginning
# 删除topic  
[root@kafka-logstash-01 kafka]# bin/kafka-topics.sh --zookeeper 10.160.145.95:2181 --delete  --topic test  
```  
重启kafka集群步骤：  
kafka01关闭-->kafka01启动-->kafka01启动完成-->kafka02关闭-->kafka02启动-->kafka02启动完成-->kafka03关闭-->kafka03启动-->kafka03启动完成  
重启zookeeper+kafka集群步骤：  
kafka01关闭-->kafka02关闭-->kafka03关闭-->zk01关闭-->zk02关闭-->zk03关闭-->确认都已关闭  
zk01启动-->zk02启动-->zk03启动-->kafka01启动-->kafka02启动-->kafka03启动-->查看是否启动成功  
kafka集群监控地址：  
http://10.160.145.95:8048  
账号：admin  
密码：123456  

## logstash要点说明  
安装目录  
```text
[root@kafka-logstash-01 logstash-7.3.0]# pwd
/usr/local/elkstack/logstash-7.3.0
```  
日志过滤配置文件的目录  
```text
[root@kafka-logstash-01 conf.d]# pwd
/usr/local/elkstack/logstash-7.3.0/config/conf.d
[root@kafka-logstash-01 conf.d]# ls
prod_h5.conf  prod_idmportal_nginx.conf  prod_oracl.conf  prod_platform_app.conf
```  
具体的配置说明太多了，这里就不展开了，可参考已有的去配置。  
常用命令：  
启动：  
```text
[root@kafka-logstash-01 conf.d]# cd /usr/local/elkstack/logstash-7.3.0
[root@kafka-logstash-01 conf.d]# bash start.sh
```  
停止：  
```text
kill -9 进程号  
```  
## es要点说明  
安装目录  
```text
[root@elasticsearch-01 elasticsearch-7.3.0]# pwd
/usr/local/elkstack/elasticsearch-7.3.0
```  
常用命令(在/usr/local/elkstack/elasticsearch-7.3.0下执行)  
启动（-d表示后台执行）：  
```text
[root@elasticsearch-01 elasticsearch-7.3.0]# su elkstack
[root@elasticsearch-01 elasticsearch-7.3.0]# ./bin/elasticsearch -d
```  
停止：  
```text
kill -9 进程号  
```  
删除索引(以oracle-orcl-alert-log-2020.11索引为例，可批量删除，例如oracle*，但不可oracle-*，这样删不掉)：  
```text
curl --user elastic:F2c@2020 -XDELETE 'http://10.160.145.92:9200/oracle-orcl-alert-log-2020.11'
```  
es-head插件地址（用于管理es集群的插件）：  
http://10.160.145.92:9100/?auth_user=elastic&auth_password=F2c@2020  

## kibana要点说明  
安装目录  
```text
[root@kafka-logstash-01 kibana-7.3.0-linux-x86_64]# pwd
/usr/local/elkstack/kibana-7.3.0-linux-x86_64
```  
常用命令（所有命令在安装目录下执行）  
启动：  
```text
nohup ./bin/kibana --allow-root > kibana_server.log 2>&1 &
```  
停止：  
```text
# 先根据端口号找到进程
fuser -n tcp 5601
# 获取进程id后杀死进程(假设进程号是1111)
kill -9 1111
```  
访问地址：  
http://10.160.145.95:5601/
用户名：elastic  
密码：F2c@2020  



