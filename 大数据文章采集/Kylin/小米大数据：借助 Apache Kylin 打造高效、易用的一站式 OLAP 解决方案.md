# 小米大数据：借助 Apache Kylin 打造高效、易用的一站式 OLAP 解决方案

如今的小米不仅是一家手机公司，更是一家大数据与人工智能公司。随着小米公司各项业务的快速发展，数据中的商业价值也愈发突显。而与此同时，各业务团队在数据查询、分析等方面的压力同样正在剧增。因此，为帮助公司各业务线解决这些数据方面的挑战，小米大数据团队不断地尝试通过不同的技术手段打造新的解决方案。

小米大数据，目前主要负责设计、完善公司级数据仓库解决方案，提供完备及全链条的数据治理一站式平台，连通各业务线数据，开发通用的画像平台与分析引擎。小米大数据团队的目标，在于不断提升数据产品与服务品质，并依托数据科学、机器学习等技术赋能核心业务。

### 业务团队亟需统一、低门槛的 OLAP 解决方案

2012 年小米大数据团队成立之后，数据平台、用户画像等通用性的技术体系相继在公司内部建立了起来。然而由于业务需求的快速变化，新的挑战也不断随之出现，比如在多维数据分析及 OLAP 需求中所遇到的诸多困难，就是其中的典型。

OLAP 的价值可体现在实现精细化运营、提升数据处理效率、改善数据可视化效果等多个方面。但小米公司内部的业务种类异常繁杂，各业务团队为了具备多维数据分析能力而各自建立了独立的 OLAP 分析系统。这些 OLAP 引擎大多是采用指标数据先进入 MySQL，再在前端进行展示的方法，而这样就将面临以下问题：

1. 基于 MySQL 的架构，在大数据上的查询效率低下；
2. 业务间 OLAP 引擎不统一，数据管道冗长，数据复用率极低，开发工作周期变长，维护成本增加；
3. 缺乏统一的维表和事实表，同主题下数据统计口径不一致；
4. 新增业务需要投入较大的成本才能获得基础的 OLAP 能力。

经过充分的内部沟通之后，发现各业务团队的基础需求主要包括以下四点：

1. 报表能力；
2. 提供 OLAP 查询接口，支持各种即席分析；
3. 尽可能降低使用门槛（ETL 以及查询的门槛）；
4. 初级阶段只需支持离线分析需求。

举例来说，其中最常见的一类需求是——开发资源相当有限的新业务，如何能在 1 天时间内开发出关键指标的多维分析看板？在这种情况下又该如何系统性地设计、搭建技术架构与解决方案？

### 以小米大数据平台核心——数据金字塔体系为基础

为了进一步满足各业务线的实际需求，小米大数据认为有必要基于自行设计的数据治理体系——“数据金字塔”，来开发一套端到端的 OLAP 解决方案。

![小米大数据：借助Apache Kylin打造高效、易用的一站式OLAP解决方案](D:\superz\BigData-A-Question\大数据文章采集\Kylin\images\53536b835276a39e8f705e52cdf430f7.png)

*数据金字塔体系的结构*

数据金字塔，是小米大数据平台技术架构中的核心部分，其目标是承载、管理、促进小米公司内的数据生态环境。该体系可将数据由下至上分布在源数据层、中间层、汇总层、应用层，共四个层面中：

- 源数据层：对来源于业务线的数据进行采集、存放等最粗粒度的处理工作。这些业务数据进入源数据层之前，需要遵循科学的打点规范并准备好数据同步策略及工具。
- 中间层：对源数据层的数据进行 ETL，合乎规范的数据表将被存放在该层中。数据表包含事实表和维表，事实表用于记录业务过程的事实数据，而维表则记录维度关系。事实表和维表都需要遵循严格的命名与操作规范。
- 汇总层：公司级或业务级的主题数据都归属于该层。典型的主题表往往会对跨业务线的多张中间表进行汇总。例如小米用户画像，就是来源于公司内部多项业务数据的挖掘汇总而成。主题表是业务数据的高度概括，基本上能满足业务团队 80% 以上的数据需求。
- 应用层：结合中间层与汇总层中的数据，对其进行优化，并开发定制化的数据能力与工具，或提供业务级甚至公司级的数据服务。

公司各业务线的在线服务日志以及其他数据源的数据（MySQL 等等），可通过数据流服务下沉到 HDFS、Kudu、HBase 等引擎中，经过数据金字塔建模后再提供给业务团队使用。

源数据层中的数据可以类比为数据湖（Data Lake），包含多种原始数据。经过源数据层的加工，数据会向上进入中间层。在中间层中，将清洗脏数据，统一数据统计口径，再经历一系列的 Join 操作和其他 ETL 过程后，会生成汇总层数据。汇总层的数据通常会更面向主题，且具有一定的通用性，例如订单、画像等等。最后在应用层中，针对业务团队的分析需求，可直接基于汇总层与中间层数据的集合，构建个性化的数据集市。

![小米大数据：借助Apache Kylin打造高效、易用的一站式OLAP解决方案](D:\superz\BigData-A-Question\大数据文章采集\Kylin\images\87d34e5dc2a6d11a23f2a58c7252c892.png)

*梳理数据流程，简化数据模型*

因此，为了节约重复开发成本，基于上述数据治理体系，小米大数据开发了一系列经过优化的解决方案，其中就包括本文所涉及的核心内容——UnionSQL。

### 利用 Apache Kylin 的特性构建定制化 OLAP 解决方案

OLAP 常见的应用场景可分为数据看板、即席查询这两种，每种场景对于查询效率与数据新鲜度都有着不同的要求。而在最初阶段下，小米大数据的主要目标是集中精力解决对于离线数据 OLAP 能力的问题。

针对 OLAP 解决方案的技术选型问题，小米大数据鉴于之前在 Elasticsearch 上所积累的经验，对于数据量不太大的业务，首先尝试了基于 Elasticsearch+Logstash+Kibana 的解决方案。尽管 Elasticsearch 在查询效率方面表现不错，也对地理位置信息类数据进行了特殊优化，但是其本身更适用于原始数据的检索，而在数据摄入、查询语法等方面的表现也并不是很理想。在此之后，Apache Kylin、Druid、ClickHouse 等方案也成为了候选项，小米大数据在结合了实际业务需求与环境后，决定从以下方面进行考量：

1. 可满足大多数需求，支持常见的算子，以及数据的摄入、查询速度足够快；
2. 保证良好的 SLA；
3. 使用门槛相对较低。

![小米大数据：借助Apache Kylin打造高效、易用的一站式OLAP解决方案](D:\superz\BigData-A-Question\大数据文章采集\Kylin\images\8ec17ce552eb4bc9c0058fc825fb666f.png)

*Apache Kylin 架构*

作为候选方案之一的 Apache Kylin，基本支持常见的 SQL 语法，并能满足通常情况下的需求。数据摄入主要依赖 MapReduce、Spark 任务将 Hive 中的数据转换为对应 Cube 的 Segment（HFile），效率方面尚可，而在查询速度方面也能提供秒级支持。对于一些如 distinct 等复杂场景而言，Apache Kylin 也提供了高精度但低效，以及高效但存在可容忍误差，这两种计算方式，以适用不同的业务场景需求。

在 SLA 方面，鉴于之前小米相关团队在 Hadoop 技术栈上积累的经验，因此同样也能针对 Apache Kylin 而提供良好的 SLA 保障。此外，Apache Kylin 本身的设计与传统数据仓库相一致，学习构建 Cube 的门槛不高，而数据的查询基于 SQL 语法，同时还提供了 JDBC 接口和 Rest API，便于现有系统对接。同时 Apache Kylin 也能较好地与 Apache Superset 等开源可视化方案进行整合，易于进行数据可视化处理。目前小米公司的部分业务已实现在日志进入数据流之后，基于现有解决方案生成数据看板等可视化的功能。

由此可见，Apache Kylin 满足了上述考量要求，并最终作为 OLAP 引擎方案而进入了小米大数据平台的技术架构，而这套完整的 OLAP 解决方案则被命名为 UnionSQL。

### 越来越多的内部业务开始受益于 UnionSQL

在引入 Apache Kylin 作为 OLAP 引擎之后，就可将需要进行分析的数据抽象成星型模型，其中的受益之处包括：

1. 只需维护最细粒度的事实分析数据，进行简单的 ETL 处理；
2. 数据流变得更清晰；
3. 维护成本进一步降低。

截止到 2018 年第三季度，小米公司内部已有超过 50 个业务接入 UnionSQL 解决方案, 其中涉及手机、MIUI、小爱同学、新零售等相关核心业务，Cube 存储空间已超过 50TB，且 95% 的查询都能在 0.35 秒内返回。UnionSQL 的架构如下图所示：

![小米大数据：借助Apache Kylin打造高效、易用的一站式OLAP解决方案](D:\superz\BigData-A-Question\大数据文章采集\Kylin\images\51349321648c034e3751c968bb47c156.png)

SQL 计划器会将用户的查询进行解析与重排，而 SQL 转发器则会把改写后的结果分发给不同的引擎。例如，当最终用户想知道某个区域的实时运营活动点击率的时候，会基于 Lambda 架构，将历史数据的查询分发给 Apache Kylin，而实时数据的查询则分发给 Elasticsearch。

众所周知，点击率 = 点击 / 曝光，但如果直接在不同引擎中计算点击率，并将得到的结果相加，就会得到一个错误的点击率。因此，UnionSQL 引擎则会将原始 SQL 进行重写，再分别计算点击量和曝光量，最后在 UnionSQL 引擎中重新计算点击率，并将正确的结果返回给最终用户。

UnionSQL 的核心理念是“快”——上手快、数据更新快、查询响应快，同时还能越用越快。

![小米大数据：借助Apache Kylin打造高效、易用的一站式OLAP解决方案](D:\superz\BigData-A-Question\大数据文章采集\Kylin\images\11e3071c292d87719152b46197a6393a.png)

*UnionSQL 解决方案的诸多优势*

在上手方面，UnionSQL 基于 SQL 语法向最终用户提供服务，极大地降低了使用门槛，同时还内置支持 Lambda 架构以及多机房访问。最终用户无需了解多种查询引擎，只需通过 SQL 语句描述需求，UnionSQL 就能基于元数据将 SQL 改写后分发给正确的引擎，并以统一的数据格式返回给最终用户。

在数据更新方面，UnionSQL 基于公司内部的数据流服务与 Lambda 架构，成功将数据延迟控制在了 2 分钟时间以内。

在查询响应方面，UnionSQL 基于 Apache Kylin 等优秀的 OLAP 引擎，以及内置的 Cache/ 自动扩容能力，使 P95 查询低于 320 毫秒。

此外，UnionSQL 还能基于慢查询智能优化引擎，可发现问题并提供慢查询优化建议，进行不同引擎的切换，或 Apache Kylin 中 Cube 的构建优化等，实现查询得越多速度就越快。

![小米大数据：借助Apache Kylin打造高效、易用的一站式OLAP解决方案](D:\superz\BigData-A-Question\大数据文章采集\Kylin\images\9c180968ff35ac045f428e14beacb9b3.png)

*Lambda on OLAP 架构*

### 三类主要的 OLAP 落地场景

一般情况下，业务团队的 OLAP 需求可大体分为三类——用户画像、数据运营、数据分析。

在用户画像方面，小米拥有公司级的通用画像表，可为各业务提供人群画像支持。以小米之家为例, 该业务的数据进入数据金字塔的汇总层后，可以和通用画像表相结合, 对用户人群进行多维分析。

![小米大数据：借助Apache Kylin打造高效、易用的一站式OLAP解决方案](D:\superz\BigData-A-Question\大数据文章采集\Kylin\images\76de44f741d95211eb13e49e6947a9f0.png)

*用户画像分析的效果*

在数据运营方面，小米内部的每一项业务都可能会产生海量的数据，那么如何才能让运营人员便捷、 快速地查看整个业务的各项关键性指标以及历史趋势，正是业务团队的刚性需求。以小米音乐为例，运营人员需要每天看到用户活跃情况，以及热门歌曲、热门歌单、播放时长等相关指标。而通过 Apache Kylin 与 Apache Superset 的配合，就可以实现这些指标的快速可视化并展示给运营人员。

在数据分析方面，以小爱同学的相关业务为例，在一些运营活动中，会主动向用户推送具有引导性的内容。在 2018 年俄罗斯世界杯进行期间，小爱同学就加入了类似的运营干预。例如，用户向小爱同学询问与天气相关的问题，小爱同学在完成回答之后还将加上一个“小提示”，如“世界杯来了，足球知识早知道，坚决不做伪球迷，快对我说：什么是越位”等等。小米大数据团队内部将其称之为“素材”，而要想评估素材的效果，就需要通过数据分析来了解用户后续是否进行了小爱同学所提示的操作。

### 小米针对 Apache Kylin 进行了诸多定制化扩展

在小米公司内部，针对 Apache Kylin 的开发主要基于社区所发布的版本，根据公司内部业务的具体需求，再进行定制化的改进或扩展，以便更好地服务各业务团队。

在当前阶段下，对 Apache Kylin 的优化工作主要集中在监控与部署、同外部系统的集成与整合、以及服务限制，这三个方面上。

#### 一、监控与部署

##### 在监控方面：

1. 支持 Apache Kylin 将内部 Metrics 推送到 Faclon 监控系统上；
2. 实现作业构建成功时 HTTP 回调以及构建失败时发送短信通知；
3. 支持周期性的 Query 探测，可将 Kylin 集群的实时可用性推送到 Falcon 监控系统上，便于及时报警。

##### 在部署方面：

1. 修改了 Apache Kylin 的打包方式，去掉了默认的 Spark、Hadoop 依赖，添加了小米公司内部维护的 Hadoop、Spark、HBase、Hive 版本，并打包成了一个完整的发布包；
2. 与公司内部的自动化部署平台进行整合，将发布包和配置进行分离，支持自动化包与配置升级，加快开发与迭代的速度。

#### 二、同外部系统的集成与整合

##### 在 HBase 方面：

1. 支持 HBase 0.98，可设置 HBase 命名空间，通过 HBase Namespace Quota，避免 Apache Kylin 在共用 HBase 集群时创建太多的表；
2. 支持 HBase 使用独立的 HDFS 集群。

##### 在用户管理方面：

由于安全性问题，公司内部无法使用 LDAP 服务，因此修改了 Apache Kylin 的用户管理功能，将其用户密码加密存储在了 Kylin 的元数据中，并增加了新的用户管理功能，以便管理员直接添加用户或修改相关权限。

#### 三、服务限制

为避免最终用户误用，保证服务质量，而在 Apache Kylin 社区版本的基础上添加了一些必要限制：

1. 限制 Query 中 IN 语句里值的个数，避免过大 in values 从而导致请求变慢；
2. 限制 Query 的最大长度以及一个 Query 执行涉及的 Segment 个数；
3. 限制 Segment 构建数据的膨胀率，使膨胀率超过限制的构建直接失败，通过线下的方式再单独协助最终用户优化及调整 Cube；
4. 在线上环境中增加了对 Cube 的最大数据量限制， 避免 HDFS 过量使用；
5. 添加了构建过程中如 distinct 等聚合函数使用 buffer 大小的限制，避免在 HBase 上写入一个 Key-Value 存储过大的数据。

此外，对于一些具有通用性质的修改，小米内部相关团队也将会逐步反馈给社区。

### 未来将尝试进一步解决多租户支持问题

尽管 Apache Kylin 项目已发展地较为成熟、稳定，但其在小米公司内部环境下的推广与使用过程中，仍然还有一些问题需要进一步解决，比如针对多租户的支持问题。

目前 Apache Kylin 对多租户并不友好，仅支持不同构建作业运行在不同的 Hadoop 队列上。 在开启安全的 Hadoop、Hive、HBase 集群上，如果最终用户使用相同的 Kerberos 帐号， 以及相同的 HDFS 路径与和 HBase 命名空间，将很难避免不同项目之间互相产生影响。

未来计划以项目为基本单位，支持设置独立的 Kerberos 帐号，可设置 HDFS、HBase 空间和名称，隔离不同项目的 Kerberos 权限及存储空间的使用，避免出现不同用户之间的互相干扰。