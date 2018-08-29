# 容器化Elastic stack (ELK)

基于Elastic的官方Docker镜像:

* [elasticsearch](https://github.com/elastic/elasticsearch-docker)
* [logstash](https://github.com/elastic/logstash-docker)
* [kibana](https://github.com/elastic/kibana-docker)

其他特性分支:

* [`x-pack`](https://github.com/deviantony/docker-elk/tree/x-pack): X-Pack 支持
* [`searchguard`](https://github.com/deviantony/docker-elk/tree/searchguard): Search Guard 支持

## 目录

1. [要求](#要求)
   * [主机配置](#主机配置)
   * [SELinux](#selinux)
2. [使用](#使用)
   * [启动](#启动)
   * [初始设置](#初始设置)
3. [配置](#配置)
   * [Kibana配置](#Kibana配置)
   * [Logstash配置](#Logstash配置)
   * [Elasticsearch配置](#Elasticsearch配置)
   * [Elasticsearch cluster配置](#Elasticsearch cluster配置)
4. [存储](#storage)
   * [Elasticsearch数据持久化](#Elasticsearch数据持久化)
5. [扩展](#扩展)
   * [添加插件](#添加插件)
6. [JVM调整](#JVM调整)
   * [内存设置](#内存设置)
7. [升级](#升级)
   * [升级版本](#升级版本)

## 要求

### 主机配置

1. 安装 [Docker](https://www.docker.com/community-edition#/download) version **17.03+**
2. 安装 [Docker Compose](https://docs.docker.com/compose/install/) version **1.6.0+**

### SELinux


```console
$ chcon -R system_u:object_r:admin_home_t:s0 docker-elk/
```


## 使用

### 启动


```console
$ echo "nameserver 9.9.9.9" > /etc/resolv.conf
$ git clone https://github.com/Zer0d0y/docker-elk.git
$ docker-compose build && docker-compose up -d
```

访问 Kibana web UI
[http://localhost:5601](http://localhost:5601)

默认容器开放以下端口：
* 5000: Logstash TCP input.
* 9200: Elasticsearch HTTP
* 9300: Elasticsearch TCP transport
* 5601: Kibana

测试：

```console
$ nc localhost 5000 < /path/to/logfile.log
```

## 初始设置

### 创建默认Kibana index pattern

#### 1.通过Kibana web UI

**NOTE**: 在通过Kibana Web UI配置Logstash索引模式之前，您需要将数据注入Logstash。 然后你要做的就是点击* Create *按钮。

参考：[Connect Kibana with
Elasticsearch](https://www.elastic.co/guide/en/kibana/current/connect-to-elasticsearch.html) 

#### 2.使用命令行

通过Kibana API创建索引模式:

```console
$ curl -XPOST -D- 'http://localhost:5601/api/saved_objects/index-pattern' \
    -H 'Content-Type: application/json' \
    -H 'kbn-version: 6.4.0' \
    -d '{"attributes":{"title":"logstash-*","timeFieldName":"@timestamp"}}'
```

## 配置

### Kibana配置

Kibana默认配置文件： `kibana/config/kibana.yml`.

### Logstash配置

Logstash默认配置文件： `logstash/config/logstash.yml`.

### Elasticsearch配置

Elasticsearch默认配置文件： `elasticsearch/config/elasticsearch.yml`.

### Elasticsearch cluster配置

参考：[Scaling out
Elasticsearch](https://github.com/deviantony/docker-elk/wiki/Elasticsearch-cluster)

## 存储

### Elasticsearch数据持久化

存储在Elasticsearch中的数据将在容器重启后保留，但在容器删除后不会保留。


```yml
elasticsearch:

  volumes:
    - /path/to/storage:/usr/share/elasticsearch/data
```

把数据存储到： `/path/to/storage`.

**NOTE:** [Elasticsearch镜像使用非特权用户 `elasticsearch` user][esuser] , 所以挂载的数据目录所有者uid必须为 `1000`.

[esuser]: https://github.com/elastic/elasticsearch-docker/blob/016bcc9db1dd97ecd0ff60c1290e7fa9142f8ddd/templates/Dockerfile.j2#L22


## 扩展

### 添加插件

步骤:

1. 在相应的`Dockerfile`添加`RUN`指令(如： `RUN logstash-plugin install logstash-filter-json`)
2. 将关联的插件代码配置添加到service配置中 (如： Logstash input/output)
3. 执行命令`docker-compose build`

## JVM调整

### 内存设置

Elasticsearch和Logstash的JVM Heap大小默认使用主机内存的1/4[1/4 of the total host
memory](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/parallel.html#default_heap_size)

Elasticsearch和Logstash的启动脚本可以从环境变量的值附加额外的JVM options:

| Service       | Environment variable |
|---------------|----------------------|
| Elasticsearch | ES_JAVA_OPTS         |
| Logstash      | LS_JAVA_OPTS         |

修改`docker-compose.yml`文件

```yml
logstash:

  environment:
    LS_JAVA_OPTS: "-Xmx1g -Xms1g"
```

## 升级

### 版本升级

修改项目根目录下 `.env`文件，然后执行以下命令:

```console
$ docker-compose build
$ docker-compose up
```

参考：[upgrade instructions](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-upgrade.html)
