**引用文章**
[https://blog.csdn.net/weixin_44275820/article/details/120548004](https://blog.csdn.net/weixin_44275820/article/details/120548004)
[初次搭建Grafana 开源的Loki 日志系统](https://blog.csdn.net/weixin_60092693/article/details/127236297)
[http://www.360doc.com/content/22/0616/11/10087950_1036241750.shtml](http://www.360doc.com/content/22/0616/11/10087950_1036241750.shtml)
[loki的原理及使用](https://blog.csdn.net/shiyuezhong/article/details/111942669)


# 1  简介
`Loki` 是 Grafana Labs 团队最新的开源项目，是一个水平可扩展，高可用性，多租户的日志聚合系统。它的设计非常经济高效且易于操作，因为它不会为日志内容编制索引，而是为每个日志流编制一组标签。项目受 Prometheus 启发，官方的介绍就是：`Like Prometheus, but for logs`，类似于 Prometheus 的日志系统。
官网：[https://github.com/grafana/loki](https://github.com/grafana/loki)
文档：[https://grafana.com/docs/loki/latest/](https://grafana.com/docs/loki/latest/)

与其他日志聚合系统相比，Loki具有下面的一些特性：

- 不对日志进行全文索引。通过存储压缩非结构化日志和仅索引元数据，Loki 操作起来会更简单，更省成本。
- 通过使用与 Prometheus 相同的标签记录流对日志进行索引和分组，这使得日志的扩展和操作效率更高。
- 特别适合储存 Kubernetes Pod 日志; 诸如 Pod 标签之类的元数据会被自动删除和编入索引。
- 受 Grafana 原生支持。
## 1.2  Loki 组成
![[Obsidian/附件/Loki -日志收集.png]]

- loki 是主服务器，负责存储日志和处理查询
   1. Loki可以水平扩展、高可用以及支持多租户的日志聚合系统
   2. 使用和Prometheus相同的服务发现机制，将标签添加到日志流中而不是构建全文索引
   3. Promtail接收到的日志和应用的metrics指标就具有相同的标签集

![[Obsidian/附件/Loki -日志收集-1.png]]
Loki本身又可以划分为3个组件，**查询器（querier）**、**日志存储器（inester）**、**前置查询器（query-frontend）**、**写入分发器（distributor）**

**写入分发器（distributor）**
分配器负责处理客户端写入的日志，一旦分配器接收到日志数据，就会分成若干批次，并将他们并行的发送到多个采集器上去。
分配器通过gRPC和采集器进行通信。分配器是无状态的，可以根据实际需要对分配器进行扩所容。
**日志存储器（inester）**
采集器负责将日志写入长期存储的后端(S3,SynamoDB等)，采集器会校验采集的日志是否乱序。另外，采集器验证接收到的日志行是按时间戳递增的顺序接收的，如果时间戳不正确，日志行将被拒绝且返回错误。
**查询器（querier）**
查询器服务负责处理LogQL查询语句来评估存储在长期存储中的日志数据。

- promtail 是代理，负责收集日志并将其发送给 loki
   1. 将容器日志发送到Loki或者Grafana服务上的日志收集工具。
   2. 发现采集目标以及给日志流添加上Label，然后发送给Loki。
   3. Promtail的服务发现基于[Prometheus](https://so.csdn.net/so/search?q=Prometheus&spm=1001.2101.3001.7020)的服务发现机制实现的，可以查看configmap loki-promtail了解细节

![image.png](https://cdn.nlark.com/yuque/0/2023/png/28922312/1677574075497-84c676cc-f633-4572-8f2e-1b5f5a9c2d4b.png#averageHue=%23f9f9f9&clientId=u23b0bd89-f9fb-4&from=paste&id=u2441223d&name=image.png&originHeight=618&originWidth=1208&originalType=url&ratio=1.5&rotation=0&showTitle=true&size=147453&status=done&style=none&taskId=u254b5618-4865-4124-ab6c-651eb915f06&title=Promtail%20%E5%A4%84%E7%90%86%E6%B5%81%E7%A8%8B "Promtail 处理流程")

---

- Grafana 用于 UI 展示
   1. Grafana是一个用于监控和可视化观测的开源平台，支持非常丰富的数据源
   2. 在Loki技术栈中它专门用来展示来自Prometheus和Loki等数据源的时间序列数据
   3. 允许进行查询、可视化、报警等操作，可以用于创建、探索和共享数据Dashboard
# 2  安装
**版本**

| **软件** | **版本** |
| --- | --- |
| Grafana | 8.5.3 |
| Loki | 2.50 |

### 2.1 下载安装包
```shell
# 下载loki服务端
wget https://github.com/grafana/loki/releases/download/v2.5.0/logcli-linux-amd64.zip
# （无wget）curl -O -L "https://github.com/grafana/loki/releases/download/v2.5.0/loki-linux-amd64.zip"

# 下载客户端promtail
wget https://github.com/grafana/loki/releases/download/v2.5.0/promtail-linux-amd64.zip
# （无wget）curl -O -L "https://github.com/grafana/loki/releases/download/v2.5.0/promtail-linux-amd64.zip"
# （docker版本）docker pull "grafana/promtail:2.5.0"

# grfana
wget https://mirrors.tuna.tsinghua.edu.cn/grafana/yum/rpm/grafana-8.5.3-1.x86_64.rpm --no-check-certificate

unzip logcli-linux-amd64.zip
chmod a+x logcli-linux-amd64

unzip promtail-linux-amd64.zip
chmod a+x promtail-linux-amd64
```
### 2.2 创建配置文件
#### loki
```shell
$ mkdir /opt/app/{promtail,loki} -pv

# promtail配置文件
$ cat <<EOF> /opt/app/loki/loki.yaml
auth_enabled: false

server:
  http_listen_port: 3100
  grpc_listen_port: 9096

ingester:
  wal:
    enabled: true
    dir: /opt/app/loki/wal
  lifecycler:
    address: 127.0.0.1
    ring:
      kvstore:
        store: inmemory
      replication_factor: 1
    final_sleep: 0s
  chunk_idle_period: 1h       # Any chunk not receiving new logs in this time will be flushed
  max_chunk_age: 1h           # All chunks will be flushed when they hit this age, default is 1h
  chunk_target_size: 1048576  # Loki will attempt to build chunks up to 1.5MB, flushing first if chunk_idle_period or max_chunk_age is reached first
  chunk_retain_period: 30s    # Must be greater than index read cache TTL if using an index cache (Default index read cache TTL is 5m)
  max_transfer_retries: 0     # Chunk transfers disabled

schema_config:
  configs:
    - from: 2020-10-24
      store: boltdb-shipper
      object_store: filesystem
      schema: v11
      index:
        prefix: index_
        period: 24h

storage_config:
  boltdb_shipper:
    active_index_directory: /opt/app/loki/boltdb-shipper-active
    cache_location: /opt/app/loki/boltdb-shipper-cache
    cache_ttl: 24h         # Can be increased for faster performance over longer query periods, uses more disk space
    shared_store: filesystem
  filesystem:
    directory: /opt/app/loki/chunks

compactor:
  working_directory: /opt/app/loki/boltdb-shipper-compactor
  shared_store: filesystem

limits_config:
  reject_old_samples: true
  reject_old_samples_max_age: 168h

chunk_store_config:
  max_look_back_period: 0s

table_manager:
  retention_deletes_enabled: false
  retention_period: 0s


ruler:
  storage:
    type: local
    local:
      directory: /opt/app/loki/rules
  rule_path: /opt/app/loki/rules-temp
  alertmanager_url: http://localhost:9093
  ring:
    kvstore:
      store: inmemory
  enable_api: true
EOF

# 解压包
unzip loki-linux-amd64.zip 
mv loki-linux-amd64 /opt/app/loki/loki
      
# service文件
$ cat <<EOF >/etc/systemd/system/loki.service
[Unit]
Description=loki server
Wants=network-online.target
After=network-online.target

[Service]
ExecStart=/opt/app/loki/loki -config.file=/opt/app/loki/loki.yaml
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=loki
[Install]
WantedBy=default.target
EOF

systemctl daemon-reload
systemctl restart loki
systemctl status loki
```
#### promtail
```shell
# promtail配置文件
$ cat <<EOF> /opt/app/promtail/promtail.yaml
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /var/log/positions.yaml # This location needs to be writeable by promtail.

client:
  url: http://localhost:3100/loki/api/v1/push

scrape_configs:
 - job_name: system
   pipeline_stages:
   static_configs:
   - targets:
      - localhost
     labels:
      job: varlogs
      host: yourhost
      __path__: /var/log/*.log
EOF

# 解压安装包
unzip promtail-linux-amd64.zip
mv promtail-linux-amd64 /opt/app/promtail/promtail

# service文件
$ cat <<EOF >/etc/systemd/system/promtail.service
[Unit]
Description=promtail server
Wants=network-online.target
After=network-online.target

[Service]
ExecStart=/opt/app/promtail/promtail -config.file=/opt/app/promtail/promtail.yaml
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=promtail
[Install]
WantedBy=default.target
EOF
  
systemctl daemon-reload
systemctl restart promtail
systemctl status promtail
```
# 3   使用
## 配置loki数据源
![image.png](https://cdn.nlark.com/yuque/0/2023/png/28922312/1677659065896-7e43baa7-c56c-4b2e-96fd-4bd7d7153374.png#averageHue=%2317191e&clientId=u23b0bd89-f9fb-4&from=paste&height=489&id=uc574ac3c&name=image.png&originHeight=734&originWidth=1815&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=43714&status=done&style=none&taskId=u8954190b-1338-4c56-ad27-b7995759274&title=&width=1210)
在数据源列表中选择 Loki，配置 Loki 源地址：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/28922312/1677659082317-df1f4d63-86a0-48f5-b14f-10b2c49a15a9.png#averageHue=%2321242a&clientId=u23b0bd89-f9fb-4&from=paste&height=325&id=u0f2508b9&name=image.png&originHeight=488&originWidth=1487&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=48191&status=done&style=none&taskId=u464994ea-bbe4-4796-9c4d-a9ef2cce63e&title=&width=991.3333333333334)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/28922312/1677659161455-45b18309-b3cf-4bbe-a0c1-c1bf59fb0461.png#averageHue=%23191c20&clientId=u23b0bd89-f9fb-4&from=paste&height=546&id=u91e4b91e&name=image.png&originHeight=819&originWidth=625&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=47288&status=done&style=none&taskId=u3f4f1175-773e-44c0-b3ff-45055f94c76&title=&width=416.6666666666667)
源地址配置 [http://loki:3100](http://loki:3100/) 即可，保存。
## explore
在添加完Loki的数据源之后，可以很方便地使用 grafana 的 explore 进行日志的查询。
切换到 grafana 左侧区域的 Explore，即可进入到 Loki 的页面：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/28922312/1675336910100-18ab5306-5c1f-4471-ac5c-6c5d44cf6ba1.png#averageHue=%23745f32&clientId=u76d7f503-79ee-4&from=paste&id=u9be7012b&name=image.png&originHeight=960&originWidth=1910&originalType=url&ratio=1&rotation=0&showTitle=false&size=366379&status=done&style=none&taskId=u4a11b9bc-5d75-4d87-be0d-0ff2a82a0e1&title=)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/28922312/1675336919359-f38f445d-14a8-4fd9-8fd6-ab791e0aec82.png#averageHue=%23f3f4f4&clientId=u76d7f503-79ee-4&from=paste&id=ucb1909d9&name=image.png&originHeight=964&originWidth=1916&originalType=url&ratio=1&rotation=0&showTitle=false&size=151053&status=done&style=none&taskId=u374875dd-64bc-4568-8aa5-9d328b44c0a&title=)
然后点击 Log labels 就可以把当前系统采集的日志标签给显示出来，可以根据这些标签进行日志的过滤查询
![image.png](https://cdn.nlark.com/yuque/0/2023/png/28922312/1675336991808-4808903d-f421-4970-addc-e23e19cb9135.png#averageHue=%23edeced&clientId=u76d7f503-79ee-4&from=paste&id=ub434e95e&name=image.png&originHeight=963&originWidth=1915&originalType=url&ratio=1&rotation=0&showTitle=false&size=160594&status=done&style=none&taskId=u481a6739-f172-499b-995e-95e46c8c169&title=)
点击某个label值，grafana会自动生成对应的LogQL的查询语句， 即可显示对应查询的结果
上面能显示日志统计的数量，下面是log的具体列表
![](https://cdn.nlark.com/yuque/0/2023/webp/28922312/1675387810543-568a99d8-89ad-404a-ba77-86ceeeca566b.webp#averageHue=%231b1f23&clientId=u522b7173-8140-4&from=paste&id=u3762ab82&originHeight=505&originWidth=1440&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u8ce5edd3-c5d0-48cf-8c8d-8fe6ceb7469&title=)
如果是 LogQL 的 Metric Query，则会显示对应的曲线
![](https://cdn.nlark.com/yuque/0/2023/webp/28922312/1675387837449-09636732-2185-448a-9000-0a405d29b741.webp#averageHue=%231a1e22&clientId=u522b7173-8140-4&from=paste&id=u6846863c&originHeight=594&originWidth=1440&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=uf1023e63-84a7-4b12-8b09-a25cc9ed3aa&title=)

:::info
需要添加新的标签，要修改 promtail.yaml 文件。
:::
```yaml
# Promtail Server Config
server:
  http_listen_port: 9080
  grpc_listen_port: 0

# Postitions
positions:
  filename: /tmp/positions.yaml

# loki服务器地址
clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
- job_name: system
  pipeline_stages:
  - json:
      expressions:
        content:
  - labels:
      Status: # 将Status字段从json格式抽取出来
  - output:
      source: content

  static_configs:
  - targets:
      - localhost
    labels:
      job: varlogs
      host: gt31
      zone: GL
      __path__: /var/log/operation.LOG # 指定本机采集的文件路径
```


# 4 查询语言 LogQL
**引用文章**受 PromQL 的启发，Loki 自己推出的查询语言，称为 LogQL，它就像一个分布式的 grep，可以聚合查看日志。和 PromQL 一样，LogQL 也是使用标签和运算符进行过滤的，主要有两种类型的查询功能： 

- 查询返回日志行内容
- 通过过滤规则在日志流中计算相关的度量指标
## 4.1 日志查询
一个基本的日志查询由两部分组成。

- log stream selector（日志流选择器）
- log pipeline（日志管道）

![](https://cdn.nlark.com/yuque/0/2023/png/28922312/1675320216650-9cc76815-6064-4fe4-9c19-02301d4f9546.png#averageHue=%23fafaf9&clientId=uc54f23d4-fafd-4&from=paste&height=363&id=u31eaa283&originHeight=363&originWidth=1080&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u5d61cbb0-d7c7-4a7a-83aa-9d9b241d392&title=&width=1080)
由于 Loki 的设计，所有 LogQL 查询必须包含一个日志流选择器。
**日志流选择器**决定了有多少日志流（日志内容的唯一来源，如文件）将被搜索到，一个更细粒度的日志流选择器将搜索到流的数量减少到一个可管理的数量。所以传递给日志流选择器的标签将影响查询执行的性能。
而日志流选择器后面的**日志管道**是可选的，日志管道是一组阶段表达式，它们被串联在一起应用于所过滤的日志流，每个表达式都可以过滤、解析和改变日志行内容以及各自的标签。

下面的例子显示了一个完整的日志查询的操作：
```
{container="query-frontend",namespace="loki-dev"} |= "metrics.go" | logfmt | duration > 10s and throughput_mb < 500
```
该查询语句由以下几个部分组成：

- 一个日志流选择器 {container="query-frontend",namespace="loki-dev"}，用于过滤 loki-dev 命名空间下面的 query-frontend[容器](https://cloud.tencent.com/product/tke?from=10680)的日志
- 然后后面跟着一个日志管道 |= "metrics.go" | logfmt | duration > 10s and throughput_mb < 500，这管道表示将筛选出包含 metrics.go 这个词的日志，然后解析每一行日志提取更多的表达并进行过滤
> 为了避免转义特色字符，你可以在引用字符串的时候使用单引号，而不是双引号，比如 `\w+1` 与 "\w+" 是相同的。

## **4.2 Log Stream Selector**
日志流选择器决定了哪些日志流应该被包含在你的查询结果中，选择器由**一个或多个键值对**组成，其中每个键是一个**日志标签**，每个值是该标签的值。
![](https://cdn.nlark.com/yuque/0/2023/webp/28922312/1675325327706-3fb21dfa-d88b-488c-aeda-4850a2d5a389.webp#averageHue=%23f9f9f9&clientId=uc9db39ed-d119-4&from=paste&id=RfWH7&originHeight=527&originWidth=582&originalType=url&ratio=1&rotation=0&showTitle=false&status=done&style=none&taskId=u36952d11-b1d4-4920-9327-39ae6321655&title=)
日志流选择器是通过将键值对包裹在一对大括号中编写的，比如：
```yaml
{app="mysql",name="mysql-backup"}
```
上面这个示例表示，所有标签为 app 且其值为 [mysql](https://cloud.tencent.com/product/cdb?from=10680) 和标签为 name 且其值为 mysql-backup 的日志流将被包括在查询结果中。
其中标签名后面的 = 运算符是一个标签匹配运算符，LogQL 中一共支持以下几种标签匹配运算符：

- =：完全匹配
- !=：不相等
- =~：正则表达式匹配
- !~：正则表达式不匹配

例如：

- {name=~"mysql.+"}
- {name!~"mysql.+"}
- {name!~mysql-\d+}

适用于 [Prometheus](https://cloud.tencent.com/product/tmp?from=10680) 标签选择器的规则同样适用于 Loki 日志流选择器。
**偏移量修饰符**
偏移修饰符允许改变查询中范围向量的时间偏移。例如，以下表达式对 MySQL 作业的最近 10 分钟到 5 分钟（而不是最近 5 分钟）内的所有日志进行计数。注意，偏移量修饰符总是需要紧跟在范围向量选择器之后。
## **4.3 Log Pipeline**
日志管道可以附加到日志流选择器上，以进一步处理和过滤日志流。它通常由一个或多个表达式组成，每个表达式针对每个日志行依次执行。如果一个表达式过滤掉了日志行，则管道将在此处停止并开始处理下一行。一些表达式可以改变日志内容和各自的标签，然后可用于进一步过滤和处理后续表达式或指标查询。

一个日志管道可以由以下部分组成。

- 日志行过滤表达式
- 解析器表达式
- 标签过滤表达式
- 日志行格式化表达式
- 标签格式化表达式
- Unwrap 表达式

其中 unwrap 表达式是一个特殊的表达式，只能在度量查询中使用
### 日志行过滤表达式
日志行过滤表达式用于对匹配日志流中的聚合日志进行分布式 grep。
编写入日志流选择器后，可以使用一个**搜索表达式**进一步过滤得到的[日志数据](https://cloud.tencent.com/solution/cloudlog?from=10680)集，搜索表达式可以是文本或正则表达式，比如：

- {job="mysql"} |= "error"
- {name="kafka"} |~ "tsdb-ops.*io:2003"
- {name="cassandra"} |~ "error=\\w+"
- {instance=~"kafka-[23]",name="kafka"} != "kafka.server:type=ReplicaManager"

上面示例中的 |=、|~ 和 != 是**过滤运算符**，支持下面几种：

- |=：日志行包含的字符串
- !=：日志行不包含的字符串
- |~：日志行匹配正则表达式
- !~：日志行与正则表达式不匹配

过滤运算符可以是链式的，并将按顺序过滤表达式，产生的日志行必须满足每个过滤器，比如：
```shell
{job="mysql"} |= "error" != "timeout"
```
当使用 |~和 !~ 时，可以使用 Golang 的 RE2 语法的正则表达式，默认情况下，匹配是区分大小写的，可以用 (?i) 作为正则表达式的前缀，切换为不区分大小写。
虽然日志行过滤表达式可以放在管道的任何地方，但最好把它们放在开头，这样可以提高查询的性能，当某一行匹配时才做进一步的后续处理。例如，虽然结果是一样的，但下面的查询 {job="mysql"}|="error"|json |= "error" | json | line_format "{{.err}}" 会比 {job="mysql"} | json | line_format "{{.message}}" |= "error" 更快，**日志行过滤表达式是继日志流选择器之后过滤日志的最快方式**。
### **解析器表达式**
解析器表达式可以解析和提取日志内容中的标签，这些提取的标签可以用于标签过滤表达式进行过滤，或者用于指标聚合。
提取的标签键将由解析器进行自动格式化，以遵循 Prometheus 指标名称的约定（它们只能包含 ASCII 字母和数字，以及下划线和冒号，不能以数字开头）。
例如下面的日志经过管道 | json 将产生以下 Map 数据：
```shell
{ "a.b": { "c": "d" }, "e": "f" }
-->
{a_b_c="d", e="f"}
```
在出现错误的情况下，例如，如果该行不是预期的格式，该日志行不会被过滤，而是会被添加一个新的 __error__ 标签。
需要注意的是如果一个提取的标签键名已经存在于原始日志流中，那么提取的标签键将以 _extracted 作为后缀，以区分两个标签，你可以使用一个标签格式化表达式来强行覆盖原始标签，但是如果一个提取的键出现了两次，那么只有最新的标签值会被保留。
目前支持 json、logfmt、regexp 和 unpack 这几种解析器。
我们应该尽可能使用 json 和 logfmt 等预定义的解析器，这会更加容易，而当日志行结构异常时，可以使用 regexp，可以在同一日志管道中使用多个解析器，这在你解析复杂日志时很有用。
#### JSON
json 解析器有两种模式运行。

1. 没有参数。如果日志行是一个有效的 json 文档，在你的管道中添加 | json 将提取所有 json 属性作为标签，嵌套的属性会使用 _ 分隔符被平铺到标签键中。
> 注意：数组会被忽略。

例如，使用 json 解析器从以下文件内容中提取标签。
```shell
{
  "protocol": "HTTP/2.0",
  "servers": ["129.0.1.1", "10.2.1.3"],
  "request": {
    "time": "6.032",
    "method": "GET",
    "host": "foo.grafana.net",
    "size": "55",
    "headers": {
      "Accept": "*/*",
      "User-Agent": "curl/7.68.0"
    }
  },
  "response": {
    "status": 401,
    "size": "228",
    "latency_seconds": "6.031"
  }
}
```
可以得到如下所示的标签列表：
```shell
"protocol" => "HTTP/2.0"
"request_time" => "6.032"
"request_method" => "GET"
"request_host" => "foo.grafana.net"
"request_size" => "55"
"response_status" => "401"
"response_size" => "228"
"response_size" => "228"
```

2. 带有参数的。在你的管道中使用  | json label="expression", another="expression" 将只提取指定的 json 字段为标签，你可以用这种方式指定一个或多个表达式，与 label_format 相同，所有表达式必须加引号。

当前仅支持字段访问（my.field, my["field"]）和数组访问（list[0]），以及任何级别嵌套中的这些组合（my.list[0]["field"]）。
例如，|json first_server="servers[0]", ua="request.headers[\"User-Agent\"] 将从以下日志文件中提取标签：
```shell
{
  "protocol": "HTTP/2.0",
  "servers": ["129.0.1.1", "10.2.1.3"],
  "request": {
    "time": "6.032",
    "method": "GET",
    "host": "foo.grafana.net",
    "size": "55",
    "headers": {
      "Accept": "*/*",
      "User-Agent": "curl/7.68.0"
    }
  },
  "response": {
    "status": 401,
    "size": "228",
    "latency_seconds": "6.031"
  }
}
```
提取的标签列表为：
```shell
"first_server" => "129.0.1.1"
"ua" => "curl/7.68.0"
```
如果表达式返回一个数组或对象，它将以 json 格式分配给标签。例如，|json server_list="services", headers="request.headers 将提取到如下标签：
```shell
"server_list" => `["129.0.1.1","10.2.1.3"]`
"headers" => `{"Accept": "*/*", "User-Agent": "curl/7.68.0"}`
```
#### logfmt
logfmt 解析器可以通过使用 |logfmt 来添加，它将从 logfmt 格式的日志行中提前所有的键和值。
例如，下面的日志行数据：
```shell
at=info method=GET path=/ host=grafana.net fwd="124.133.124.161" service=8ms status=200
```
将提取得到如下所示的标签：
```shell
"at" => "info"
"method" => "GET"
"path" => "/"
"host" => "grafana.net"
"fwd" => "124.133.124.161"
"service" => "8ms"
"status" => "200"
```
#### **regexp**
与 logfmt 和 json（它们隐式提取所有值且不需要参数）不同，regexp 解析器采用单个参数 | regexp "<re>" 的格式，其参数是使用 Golang RE2 语法的正则表达式。
**引用文章**
正则表达式必须包含至少一个命名的子匹配（例如(?P<name>re)），每个子匹配项都会提取一个不同的标签。
例如，解析器
 | regexp "(?P<method>\\w+) (?P<path>[\\w|/]+) \\((?P<status>\\d+?)\\) (?P<duration>.*)" 将从以下行中提取标签：
```shell
POST /api/prom/api/v1/query_range (200) 1.5s
```
提取的标签为：
```shell
"method" => "POST"
"path" => "/api/prom/api/v1/query_range"
"status" => "200"
"duration" => "1.5s"
```
#### **unpack**
unpack 解析器将解析 json 日志行，并通过打包阶段解开所有嵌入的标签，一个特殊的属性 _entry 也将被用来替换原来的日志行。
例如，使用 | unpack 解析器，可以得到如下所示的标签：
```shell
{
  "container": "myapp",
  "pod": "pod-3223f",
  "_entry": "original log message"
}
```
允许提取 container 和 pod 标签以及原始日志信息作为新的日志行。
> 如果原始嵌入的日志行是特定的格式，你可以将 unpack 与 json 解析器（或其他解析器）相结合使用。

### **标签过滤表达式**
标签过滤表达式允许使用其原始和提取的标签来过滤日志行，它可以包含多个谓词。
一个谓词包含一个标签标识符、操作符和用于比较标签的值。
例如 cluster="namespace" 其中的 cluster 是标签标识符，操作符是 =，值是"namespace"
LogQL 支持从查询输入中自动推断出的多种值类型：

- String（字符串）用双引号或反引号引起来，例如"200"或`us-central1`。
- Duration（时间）是一串十进制数字，每个数字都有可选的数和单位后缀，如 "300ms"、"1.5h" 或 "2h45m"，有效的时间单位是 "ns"、"us"（或 "µs"）、"ms"、"s"、"m"、"h"。
- Number（数字）是浮点数（64 位），如 250、89.923。
- Bytes（字节）是一串十进制数字，每个数字都有可选的数和单位后缀，如 "42MB"、"1.5Kib" 或 "20b"，有效的字节单位是"b"、"kib"、"kb"、"mib"、"mb"、"gib"、"gb"、"tib"、"tb"、"pib"、"bb"、"eb"

字符串类型的工作方式与 Prometheus 标签匹配器在日志流选择器中使用的方式完全一样，这意味着你可以使用同样的操作符（=、!=、=~、!~）。
使用 Duration、Number 和 Bytes 将在比较前转换标签值，并支持以下比较器。

- == 或 = 相等比较
- != 不等于比较
- > 和 >= 用于大于或大于等于比较
- < 和 <= 用于小于或小于等于比较

例如 logfmt | duration > 1m and bytes_consumed > 20MB 过滤表达式。
如果标签值的转换失败，日志行就不会被过滤，而会添加一个 __error__ 标签，要过滤这些错误，请看管道错误部分。
你可以使用 and和 or 来连接多个谓词，它们分别表示**且**和**或**的二进制操作，and 可以用逗号、空格或其他管道来表示，标签过滤器可以放在日志管道的任何地方。
以下所有的表达式都是等价的:
```shell
| duration >= 20ms or size == 20kb and method!~"2.."
| duration >= 20ms or size == 20kb | method!~"2.."
| duration >= 20ms or size == 20kb,method!~"2.."
| duration >= 20ms or size == 20kb method!~"2.."
```
默认情况下，多个谓词的优先级是从右到左，你可以用圆括号包装谓词，强制使用从左到右的不同优先级。
例如，以下内容是等价的：
```shell
| duration >= 20ms or method="GET" and size <= 20KB
| ((duration >= 20ms or method="GET") and size <= 20KB)
```
它将首先评估 duration>=20ms or method="GET"，要首先评估 method="GET" and size<=20KB，请确保使用适当的括号，如下所示。
```shell
| duration >= 20ms or (method="GET" and size <= 20KB)
```
### **日志行格式表达式**
日志行格式化表达式可以通过使用 Golang 的 text/template 模板格式重写日志行的内容，它需要一个字符串参数 | line_format "{{.label_name}}" 作为模板格式，所有的标签都是注入模板的变量，可以用 {{.label_name}} 的符号来使用。
例如，下面的表达式：
```shell
{container="frontend"} | logfmt | line_format "{{.query}} {{.duration}}"
```
将提取并重写日志行，只包含 query 和请求的 duration。你可以为模板使用双引号字符串或反引号 `{{.label_name}}` 来避免转义特殊字符。
此外 line_format 也支持数学函数，例如：
如果我们有以下标签 ip=1.1.1.1, status=200 和 duration=3000(ms), 我们可以用 duration 除以 1000 得到以秒为单位的值：
```shell
{container="frontend"} | logfmt | line_format "{{.ip}} {{.status}} {{div .duration 1000}}"
```
上面的查询将得到的日志行内容为1.1.1.1 200 3
### **标签格式表达式**
| label_format表达式可以重命名、修改或添加标签，它以逗号分隔的操作列表作为参数，可以同时进行多个操作。
当两边都是标签标识符时，例如 dst=src，该操作将把 src 标签重命名为 dst。
左边也可以是一个模板字符串，例如 dst="{{.status}} {{.query}}"，在这种情况下，dst 标签值会被 Golang 模板执行结果所取代，这与 | line_format 表达式是同一个模板引擎，这意味着标签可以作为变量使用，也可以使用同样的函数列表。
在上面两种情况下，如果目标标签不存在，那么就会创建一个新的标签。
重命名形式 dst=src 会在将 src 标签重新映射到 dst 标签后将其删除，然而，模板形式将保留引用的标签，例如 dst="{{.src}}" 的结果是 dst 和 src 都有相同的值。
一个标签名称在每个表达式中只能出现一次，这意味着 | label_format foo=bar,foo="new" 是不允许的，但你可以使用两个表达式来达到预期效果，比如 | label_format foo=bar | label_format foo="new"
# 4 查询示例
**多重过滤**
过滤应该首先使用标签匹配器，然后是行过滤器，最后使用标签过滤器：
```shell
{cluster="ops-tools1", namespace="loki-dev", job="loki-dev/query-frontend"} |= "metrics.go" !="out of order" | logfmt | duration > 30s or status_code!="200"
```
**多解析器**
比如要提取以下格式日志行的方法和路径：
```shell
level=debug ts=2020-10-02T10:10:42.092268913Z caller=logging.go:66 traceID=a9d4d8a928d8db1 msg="POST /api/prom/api/v1/query_range (200) 1.5s"
```
你可以像下面这样使用多个解析器：
```shell
{job="cortex-ops/query-frontend"} | logfmt | line_format "{{.msg}}" | regexp "(?P<method>\\w+) (?P<path>[\\w|/]+) \\((?P<status>\\d+?)\\) (?P<duration>.*)"`
```
首先通过 logfmt 解析器提取日志中的数据，然后使用 | line_format 重新将日志格式化为 POST /api/prom/api/v1/query_range (200) 1.5s，然后紧接着就是用 regexp 解析器通过正则表达式来匹配提前标签了。
**格式化**
下面的查询显示了如何重新格式化日志行，使其更容易阅读。
```shell
{cluster="ops-tools1", name="querier", namespace="loki-dev"}
  |= "metrics.go"
  |!= "loki-canary"
  | logfmt
  | query != ""
  | label_format query="{{ Replace .query \"\\n\" \"\" -1 }}"
  | line_format "{{ .ts}}\t{{.duration}}\ttraceID = {{.traceID}}\t{{ printf \"%-100.100s\" .query }} "
```
其中的 label_format 用于格式化查询，而 line_format 则用于减少信息量并创建一个表格化的输出。比如对于下面的日志行数据：
```shell
level=info ts=2020-10-23T20:32:18.094668233Z caller=metrics.go:81 org_id=29 traceID=1980d41501b57b68 latency=fast query="{cluster=\"ops-tools1\", job=\"cortex-ops/query-frontend\"} |= \"query_range\"" query_type=filter range_type=range length=15m0s step=7s duration=650.22401ms status=200 throughput_mb=1.529717 total_bytes_mb=0.994659
level=info ts=2020-10-23T20:32:18.068866235Z caller=metrics.go:81 org_id=29 traceID=1980d41501b57b68 latency=fast query="{cluster=\"ops-tools1\", job=\"cortex-ops/query-frontend\"} |= \"query_range\"" query_type=filter range_type=range length=15m0s step=7s duration=624.008132ms status=200 throughput_mb=0.693449 total_bytes_mb=0.432718
```
经过上面的查询过后可以得到如下所示的结果：
```shell
2020-10-23T20:32:18.094668233Z	650.22401ms	    traceID = 1980d41501b57b68	{cluster="ops-tools1", job="cortex-ops/query-frontend"} |= "query_range"
2020-10-23T20:32:18.068866235Z	624.008132ms	traceID = 1980d41501b57b68	{cluster="ops-tools1", job="cortex-ops/query-frontend"} |= "query_range"
```
# 5 日志度量
LogQL 同样支持通过函数方式将日志流进行度量，通常我们可以用它来计算消息的错误率或者排序一段时间内的应用日志输出 Top N。
## 区间向量
LogQL 同样也支持有限的区间向量度量语句，即是**将日志条目作为一个整体来计算数值**。使用方式和 PromQL 类似，常用函数主要是如下 4 个：

- rate(log-range)：计算每秒的日志条目
- count_over_time(log-range)：对指定范围内的每个日志流的条目进行计数
- bytes_rate(log-range)：计算日志流每秒的字节数
- bytes_over_time(log-range)：对指定范围内的每个日志流的使用的字节数

比如计算 nginx 的 qps：
```shell
rate({filename="/var/log/nginx/access.log"}[5m])) 
```
计算 kernel 过去 5 分钟发生 oom 的次数：
```shell
count_over_time({filename="/var/log/message"} |~ "oom_kill_process" [5m]))
```
## **聚合函数**
LogQL 也支持聚合运算，我们可用它来聚合单个向量内的元素，从而产生一个具有较少元素的新向量，当前支持的聚合函数如下：

- sum：求和
- min：最小值
- max：最大值
- avg：平均值
- stddev：标准差
- stdvar：标准方差
- count：计数
- bottomk：最小的 k 个元素
- topk：最大的 k 个元素

聚合函数我们可以用如下表达式描述：
```shell
<aggr-op>([parameter,] <vector expression>) [without|by (<label list>)] 
```
对于需要对标签进行分组时，我们可以用 without 或者 by 来区分。比如计算 nginx 的 qps，并按照 pod 来分组：
```shell
sum(rate({filename="/var/log/nginx/access.log"}[5m])) by (pod) 复制
```
只有在使用 bottomk 和 topk 函数时，我们可以对函数输入相关的参数。比如计算 nginx 的 qps 最大的前 5 个，并按照 pod 来分组：
```shell
topk(5,sum(rate({filename="/var/log/nginx/access.log"}[5m])) by (pod)))
```
## **二元运算**
### **数学计算**
Loki 存的是日志，都是文本，怎么计算呢？显然 LogQL 中的数学运算是面向区间向量操作的，LogQL 中的支持的二进制运算符如下：

- +：加法
- -：减法
- *：乘法
- /：除法
- %：求模
- ^：求幂

比如我们要找到某个业务日志里面的错误率，就可以按照如下方式计算：
```shell
sum(rate({app="foo", level="error"}[1m])) / sum(rate({app="foo"}[1m]))
```
### **逻辑运算**
集合运算仅在区间向量范围内有效，当前支持

- and：并且
- or：或者
- unless：排除

比如：
```shell
rate({app=~"foo|bar"}[1m]) and rate({app="bar"}[1m])
```
### **比较运算**
LogQL 支持的比较运算符和 PromQL 一样，包括：

- ==：等于
- !=：不等于
- >：大于
- >=: 大于或等于
- <：小于
- <=: 小于或等于

通常我们使用区间向量计算后会做一个阈值的比较，这对应告警是非常有用的，比如统计 5 分钟内 error 级别日志条目大于 10 的情况：
```shell
count_over_time({app="foo", level="error"}[5m]) > 10
```
我们也可以通过布尔计算来表达，比如统计 5 分钟内 error 级别日志条目大于 10 为真，反正则为假：
```shell
count_over_time({app="foo", level="error"}[5m]) > bool 10
```
## 注释
LogQL 查询可以使用 # 字符进行注释，例如：
```shell
{app="foo"} # anything that comes after will not be interpreted in your query
```
对于多行 LogQL 查询，可以使用 # 排除整个或部分行：
```shell
{app="foo"}
| json
# this line will be ignored
| bar="baz" # this checks if bar = "baz"
```
