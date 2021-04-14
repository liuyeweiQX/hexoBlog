---
title: 初识Activiti工作流
date: 2021-02-06 16:14:30
tags: 
  - Activiti
categories: 
  - 工作流
description: 初识Activiti工作流
comments: true
top_img: https://cdn.jsdelivr.net/gh/liuyeweiQX/picBed/img/post/img16.jpg
cover: https://cdn.jsdelivr.net/gh/liuyeweiQX/picBed/img/post/img16.jpg
---
### 工作流介绍
工作流（Workflow），是对工作流程及其各操作步骤之间业务规则的抽象、概括、描述。
工作流要解决的主要问题：为实现某个业务目标，在多个参与者之间，利用计算机，按某种预定规则自动传递文档、信息或者任务。

### BPMN 2.0规范简述
BPMN规范，全称是Business Process Modeling Notation（1.0-2004），BPMN规范的发布为了让业务流程的全部参与人员对流程可以进行可视化管理，提供一套让所有参与人员都易于理解的语言和标记。
2.0 版本于2011年发布，全称改为Business Process Model And Notation（业务流程模型和符号）

BPMN2.0在以下方面扩展了BPMN1.2：
- 规范了流程元素的执行语法。
- 定义了流程模型和流程图的扩展机制。
- 细化了事件的组成。
- 扩展了参与者的交互定义。
- 定义了编排模型。

BPMN 2.0的**目的**是建议简单并且易懂的业务流程模型，但是同时又需要处理高度复杂的业务流程，因此要解决这两个矛盾的要求，需要在规范中定义标准的图形和符号。

BPMN中定义了5个基础的元素类别：
1. 流对象（Flow Objects）：在一个业务流程中，流对象是用于定义行为的图形元素，主要有事件（Events）、活动（Activities）和网关（Gateways）三种流对象。
2. 数据（Data）：主要有数据对象（Data Object）、数据输入（Data Inputs）、数据输出（Data Outputs）和数据存储（Data Stores）4种元素。
3. 连接对象（Connecting Objects）：用于连接流对象，主要有4种连接流对象的方式，包括顺序流（Sequence Flows）、消息流（Message Flows）、关联（Associations）和数据关联（Data Associations）
4. 泳道（Swimlanes）：泳道有两种途径组织基础的模型元素，分别是池（Pools）和道（Lanes）。
5. 制品（Artifacts）：制品主要用于为流程提供附件信息，当前制品包括组（Group）和注释（Text Annotation）。

BPMN元素的图形及其描述：
{% asset_img content1.png content1 %}       
{% asset_img content2.png content2 %} 

### Activiti数据库设计
**Activiti流程引擎的数据表分5大类**：
1. 通用数据表
2. 流程存储数据表
3. 身份数据表
4. 运行时数据表
5. 历史数据表
Activiti 6.0版本种加入了基于DMN规范的规则引擎，DMN规则引擎数据表

#### 通用数据表：
用于存放一些通用的数据，这些表本身不关心特定的流程或者业务，只用于存放这些业务或者流程所使用的通用资源。
通用数据表有两个，都以“ACT_GE”开头，GE是单词general的缩写。
- 资源表：表ACT_GE_BYTEARRAY用于保存与流程引擎相关的资源，只要调用了Activiti存储服务的API，涉及的资源均会被转换为byte数据保存到这个表中。使用这个表来保存字符串、流程文件
- 属性表：Activiti将全部的属性抽象为 key-value 对，每个属性都有名称和值，使用ACT_GE_PROPERTY来保存这些属性

#### 流程存储表：
使用仓储表来保存流程定义和部署信息这类数据，存储表名称以“ACT_RE”开头，“RE"是repository单词的缩写。
- 部署数据表：在Activiti中，一次部署可以添加多个资源，资源会被保存到资源表中（ACT_GE_BTYEARRAY），而对于部署，则部署信息会被保存到部署表中（ACT_RE_DEPLOYMENT）
- 流程定义表：Activiti在部署添加资源时，如果发布部署的文件是流程文件（.bpmn或者.BPMN20.xml），则除了会解析解析流程文件，将内容保存到资源表外，还会解析流程文件的内容，形成特定的流程定义数据，写入流程定义表（ACT_RE_PROCDEF）中。

#### 身份数据表：
Activiti的整个身份数据模块，可以独立于流程引擎而存在，有关身份数据的几张表，并没有保存与流程相关的数据及关联。身份表名称以ACT_ID开头，表名中的"ID"是单词identity的缩写。
- 用户表：流程引擎用户的信息被保存在ACT_ID_USER表中
- 用户账号（信息）表：Activiti将用户、用户账号和用户信息分为三种数据，其中用户表保存用户的数据，而用户账号和用户信息，则被保存到ACT_ID_INFO表中。
- 用户组表：使用ACT_ID_GROUP表来保存用户组的数据
- 关系表：一个用户组下有多个用户，一个用户可以属于不同的用户组，那么这种多对多的关系就使用关系表来进行描述，关系表为ACT_ID_MEMBERSHIP

#### 运行时数据表：
运行时数据表用来保存流程在运行过程中所产生的数据，例如流程实例、执行流、任务等。
运行时数据表的名称以ACT_RU开头，"RU"是单词runtime的缩写。
- 流程实例（执行流）表：流程启动后，会产生一个流程实例，同时会产生相应的执行流，流程实例和执行流数据均被保存在ACT_RU_EXECUTION表中，如果一个流程实例只有一条执行流，那么该表中只产生一条数据，该数据既表示执行流，也表示流程实例。
- 流程任务表：流程在运行过程中所产生的任务数据保存在ACT_RU_TASK表中
- 流程参数表：ACT_RU_VARIABLE表来存放流程中的参数，这类参数包括流程实例参数、执行流参数和任务参数，参数有可能会有多种类型
- 流程与身份关系表：用户组和用户之间的关系，使用ACT_ID_MEMBERSHIP表来保存，用户或者用户组与流程数据之间的关系，则使用ACT_RU_IDENTITYLINK表来保存，相比于ACT_ID_MEMBERSHIP表，ACT_RU_IDENTITYLINK表的字段更多一些。
- 工作数据表：在流程执行的过程中，会有一些工作需要定时或者重复执行，这类工作数据被保存到工作表中
- 事件描述表：如果流程到达某类事件节点，Activiti会往ACT_RU_EVENT_SUBSCR表中加入事件描述数据，这些事件描述数据将会决定流程事件的触发。

#### 历史数据表：
历史数据表就好像流程引擎的日志表，操作过的流程元素将会被记录到历史表中。历史数据表名称以ACT_HI开头，“HI”是单词history的缩写。
- 流程实例表：流程实例的历史数据会被保存到ACT_HI_PROCINST表中，只要流程被启动，就会将流程实例的数据写入表中。除了基本的流程字段外，与运行时数据表不同的是，历史流程实例表还会记录流程开始活动ID、结束活动ID等信息。
- 流程明细表：ACT_HI_DETAIL会记录流程执行过程中的参数或者表单数据，由于在流程执行过程中，会产生大量这类数据，因此默认情况下，Activiti不会保存流程明细数据，除非将流程引擎的历史数据（history）配置为null
- 历史任务表和历史行为表：当流程到达某个任务节点时，就会向历史任务表（ACT_HI_TASKINST）中写入历史任务数据，该表在运行时的任务表类似。历史行为表（ACT_HI_ACTINST）会记录每一个流程活动的实例，一个流程活动将会被记录为一条数据，根据该表可以追踪最完整的流程信息。
- 附件表和评论表：使用任务服务（TasjService）的API，可以添加附件和评论，这些附件和评论的数据将会被保存到ACT_HI_ATTACHMENT和ACT_HI_COMMENT表中。ACT_HI_COMMENT表实际不只保存评论数据，它还会保存某些事件数据，但它的表名为COMMENT，因此更倾向把它叫做评论表。虽然附件表和评论表的命名遵守历史数据表的命名规范（以ACT_HI开头），但是可以调用其他服务组件的API来往这两张表中写入数据。历史数据表实际上保存的是那种一经写入，就很少会发生变化（结构性变化）的数据。

#### DMN规范引擎表：
Activiti 6.0中加入了基于DMN规范的规则引擎模块，当前版本主要有三个数据表，保存规则引擎相关的数据。
- 决策部署表：保存决策数据，类似于流程定义部署，每一次部署，可以添加多份决策文件，向部署表中写入一条部署数据，对应数据表为ACT_DMN_DEPLOYMENT。
- 决策表：可以先将决策看作流程定义，决策文件中保存着决策表，部署时会解析决策文件中的决策模型并将其保存到ACT_DMN_DECISION_TABLE表中。
- 部署资源表：规则引擎相关的资源，例如决策文件、图片等，被保存在ACT_DMN_DEPLOYMENT_RESOURCE表中，该表类似流程引擎的资源表。

### Activiti流程引擎
**ProcessEngineConfiguration对象**代表一个Activiti流程引擎的全部配置，该类提供一系列创建 ProcessEngineConfiguration 实例的静态方法，这些方法用于读取和解析相应的配置文件，并返回 ProcessEngineConfiguration 的实例。

使用静态方法创建 ProcessEngineConfiguration 实例：
1. 读取默认的配置文件
ProcessEngineConfiguration 对象的 createProcessEngineConfigurationFromResourceDefault 方法， 使用Activiti默认的方式创建ProcessEngineConfiguration的实例。这里所说的默认方式，是**指由Activiti决定读取配置文件的位置、文件的名称和配置bean的名称这些信息**。Activiti默认到ClassPath下读取名为"activiti.cfg.xml”的Activiti配置文件，启动并获取名称为 “processEngineConfiguration”的bean的实例。解析XML与创建该bean实例的过程，由Spring 代为完成。
{% asset_img content3.png content3 %}

2. 读取自定义的配置文件
默认情况下Activiti将到ClassPath下读取activiti.cfg.xml文件，如果希望 Activiti 读取另外名称的配置文件，则可以使用 createProcessEngineConfigurationFromResource 方法创建ProcessEngineConfiguration,该方法参数为一个字符串对象，当调用该方法时，需要告诉 Activiti 配置文件的位置。
{% asset_img content4.png content4 %}

3. 读取输入流的配置
ProcessEngineConfiguration对象中提供了一个createProcessEngineConfigurationFromInputStream方法，该方法使得Activiti配置文件的加载不再局限于项目的ClassPath，只要得到配置文件的输入流， 即可创建 ProcessEngineConfiguration。createProcessEngineConfigurationFromInputStream方法，提供了两个重载的方法，可以指定在解析XML时bean的名称。 
{% asset_img content5.png content5 %}

4. 使用createStandaloneInMemProcessEngineConfiguration方法
使用该方法创建ProcessEngineConfiguration，并不需要指定任务参数，该方法直接返回一 个 StandalonelnMemProcessEngineConfiguration 实例，该类为 ProcessEngineConfiguration 的子类。使用该方法创建ProcessEngineConfiguration，并不会读取任何的Activiti配置文件，这就意味着流程引擎配置的全部属性都会使用默认值。与其他子类不一样的是, 创建的 StandalonelnMemProcessEngineConfiguration 实例，只特别指定了 databaseSchemaUpdate 属性和jdbcUrl属性。
{% asset_img content6.png content6 %}

5. 使用createStandaloneProcessEngineConfiguration方法
createStandaloneProcessEngineConfiguration 方法实际返回一个 StandaloneProcessEngineConfiguration 实例，需要注意的是，StandaloneInMemProcessEngineConfiguration类是 StandaloneProcessEngineConfiguration 类的子类。
{% asset_img content7.png content7 %}

### 数据源配置
Activiti在启动时，会读取相应的数据源配置，用于对数据库进行相应的操作。Activiti会先读取配置文件，然后取得配置的bean,并对其进行初始化。

**Activiti支持的数据库**
Activiti默认支持H2数据库，H2是一个开源的嵌入式数据库，是使用Java语言编写的。 使用H2数据库不需要安装服务器或者客户端，只需提供一个jar包即可使用。在实际的企业应用中，很少会使用这种轻量级的嵌入式数据库，因此H2数据库更适合单元测试。除了 H2 数据库，Activiti还对以下数据库提供支持。
- MySQL：主流数据库之一，它是一个开源的小型关系型数据库，由于它体积小、速度快，得到相当多开发者的青睐，并且最重要的是，它是免费的。
- Oracle：目前世界上最流行的商业数据库，价格昂贵，但是它高效的性能、可靠的数据管理，仍令不少企业心甘情愿为其掏钱。
- PostgreSQL：PostgreSQL是另外一款开源的数据库。
- DB2：由IBM公司研发的一款关系型数据库，其良好的伸缩性和高效性，让它成为继 Oracle之后又一强大的商业数据库。
- MSSQL：微软研发的一款数据库产品，目前也支持在Linux下使用。

**Activiti 与 Spring**
Spring是目前非常流行的一个轻量级J2EE框架，它提供了一套轻量级的企业应用解决方案，包括IoC容器、AOP面向切面技术以及WebMVC框架等。
使用Activiti的项目，并不意味着一定要使用Spring，Activiti可以在任何有或者没有Spring 的环境中使用。虽然Activiti并不需要使用Spring环境，但是Activiti在创建流程引擎时，使用了 Spring的XML解析与依赖注入功能ProcessEngineConfiguration对应的配置，即为Spring 中的一个bean。

**JDBC 配置**
使用JDBC连接数据库，需要使用jdbc url、 jdbc驱动、数据库用户名和密码。
{% asset_img content8.png content8 %}

**DBCP数据源配置**
DBCP是Apache提供的一-个数据库连接池。ProcessEngineConfiguration中提供了一个 dataSource属性，如果用户不希望直接将JDBC的相关连接属性交给Activiti，可以自己创建数据库连接，然后通过这个dataSource属性设置到ProcessEngineConfiguration中。为Activiti的 ProcessEngineConfiguration设置dataSource，可以采用配置或者编码的方式。
{% asset_img content9.png content9 %}

**C3P0数据源配置**
与DBCP类似，C3P0也是一个开源的数据库连接池，它们都被广泛地应用到开源项目以 及企业应用中。
{% asset_img content10.png content10 %}

**Activiti其他数据源配置**
如果不使用第三方数据源，直接使用Activiti提供的数据源，那么还可以指定其他一些数据库属性。Activiti默认使用的是myBatis的数据连接池，因此ProcessEngineConfiguration中也提供了一些MyBatis的配置。
jdbcMaxActiveConnections: 在数据库连接池内最大的活跃连接数，默认值为10。
jdbcMaxIdleConnections: 连接池最大的空闲连接数。
jdbcMaxCheckoutTime：当连接池内的连接耗尽，外界向连接池请求连接时，创建连接的等待时间，单位为毫秒，默认值为20000,即20秒。
jdbcMaxWaitTime：当整个连接池需要重新获取连接的时候，设置等待的时间，单位为毫秒，默认值为20000,即20秒。

**数据库策略配置**
ProcessEngineConfiguration提供了 databaseSchemaUpdate属性，该项可以设置流程引擎启动 和关闭时数据库执行的策略。在Activiti的官方文档中，databaseSchemaUpdate有以下三个值。
false： false为默认值，若设置为该值，Activiti在启动时，会对比数据库表中保存的版本，如果没有表或者版本不匹配，将在启动时抛出异常。
true： 若设置为该值，Activiti会对数据库中所有的表进行更新,如果表不存在，则Activiti会自动创建。
create-drop： Activiti启动时，会执行数据库表的创建操作。而Activiti关闭吋，执行数据库表的删除操作。
{% asset_img content11.png content11 %}

**databaseType 配置**
将databaseSchemaUpdate设置为create-drop或者drop-create 时，Activiti在启动和初始化时，会执行相应的创建表和删除表操作。Activiti支持多种数据库， 每种数据库的创建表与删除表的语法有可能不一样，因此，需要指定databaseType属性，来告诉Activiti，目前使用了何种数据库(当然，如果设置为true而数据库中没有表的话，也需要知道使用哪种数据库)。databaseType属性支持这些值：h2、mysql、oracle、postgres、mssql 和db2。没有指定值时，databaseType为null。指定databaseType属性，目的是为了确定执行创建（或删除）
表的SQL脚本。
{% asset_img content12.png content12 %}

### 流程引擎的创建
#### ProcessEngineConfiguration 的 buildProcessEngine 方法
使用ProcessEngineConfiguration的create方法可以创建ProcessEngineConfiguration 的实例。ProcessEngineConfiguration 中提供了一个 buildProcessEngine方法，该方法返回一个ProcessEngine实例。
{% asset_img content13.png content13 %}
得到流程引擎的相关配置后，buildProcessEngine方法会根据这些配置，初始化流程引擎 的相关服务和对象，包括数据源、事务、拦截器和服务组件等。这个流程引擎的初始化过程, 实际上也可以被看作是一个配置检查的过程。

#### ProcessEngines
除了 ProcessEngineConfiguration 的 buildProcessEngine 方法外，ProcessEngines 类提供了创建ProcessEngineConfiguration实例的方法。ProcessEngines是一个创建流程引擎与关闭流程引擎的工具类，所有创建(包括其他方式创建)的ProcessEngine实例均被注册到ProcessEngines 中。这里所说的注册，实际上是ProcessEngines类中维护一个Map对象，该对象的key为 ProcessEngine 实例的名称，value 为 ProcessEngine 的实例，当向 ProcessEngines 注册 ProcessEngine实例时，实际上是调用Map的put方法，将该实例缓存到Map中。
**init 与 getDefaultProcessEngine 方法**
ProcessEngines的init方法，会读取Activiti的默认配置文件，然后将创建的ProcessEngine 实例缓存到Map中。这里所说的默认配置文件，一般情况下是指ClassPath 下的activiti.cfg.xml, 如果与Spring进行了整合，则读取ClassPath下的activiti-context.xml文件。
{% asset_img content14.png content14 %}
此处的init方法并不会返回任何ProcessEngine实例，该方法只会加载classpath下全部的 Activiti配置文件并且将创建的ProcessEngine实例保存到ProcessEngines中。如果需要得到相 应的ProcessEngine实例，可以使用getProcessEngines方法获取ProcessEngines中全部的 ProcessEngine 实例 ，getProcessEngines 返回的是一个 Map，只需要根据 ProcessEngine 的名称， 即可得到相应的ProcessEngine实例。
另外，ProcessEngines 提供了一个 getDefaultProcessEngine 方法，用于返回 key 为 “default” 的ProcessEngine实例，该方法会判断ProcessEngines是否进行初始化，如果没有，则会调用 init方法进行初始化。

**registerProcessEngine 和 unregister 方法**
注册和注销方法，registerProcessEngine 方法向 ProcessEngines 中注册一个 ProcessEngine 实例，unregister方法则从ProcessEngines中注销一个ProcessEngine实例。注册与注销 ProcessEngine实例，均会根据该ProcessEngine实例的名称进行操作，因为Map的key使用的 是ProcessEngine的名称。
{% asset_img content15.png content15 %}
注意：unregister方法只是单纯地将ProcessEngine实例从Map移除，并不会调用 ProcessEngine 的 close 方法。

**retry 方法**
如果Activiti在加载配置文件时出现异常，则可以调用ProcessEngines的retry方法重新加载配置文件，重新创建ProcessEngine实例并加入到Map中。
{% asset_img content16.png content16 %}

**destroy 方法**
ProcessEngines的destroy方法，顾名思义，是对其所有维护的ProcessEngine实例进行销 毁，并且在销毁时调用ProcessEngine的close方法。
Destory方法在执行时，会调用所有ProcessEngine实例的close方法，该方法会将异步工作执行器（AsyncExecutor）关闭，如果为流程引擎配置的数据库策略为 create-drop，则会执行数据库表的 drop 操作。

#### ProcessEngine 对象
在Activiti中，一个ProcessEngine实例代表一个流程引擎，ProcessEngine保存着各个服务组件的实例，根据这些服务组件，可以操作流程实例、任务、系统角色等数据。
**服务组件**
当创建流程引擎实例后，ProcessEngine中会初始化一系列服务组件，这些组件提供了大部分控制流程引擎数据的业务方法，这些组件就好像J2EE中的Service层。可以使用 ProcessEngine中的getAXYService方法得到这些组件的实例。一个ProcessEngine维护以下业务组件实例。
- Repositoryservice：提供一系列管理流程定义和流程部署的API。
- Runtimeservice：在流程运行时对流程实例进行管理与控制。
- TaskService：对流程任务进行管理，例如任务提醒、任务完成和分配任务等。
- Identityservice：提供对流程角色数据进行管理的API,这些角色数据包括用户组、用 户以及它们之间的关系。
- Managementservice：提供对流程引擎进行管理和维护的服务。
- HistoryService：对流程的历史数据进行操作，包括查询、删除这些历史数据。
{% asset_img content17.png content17 %}

关闭流程引擎
ProcessEngines实例在进行销毁操作时，会调用所有ProcessEngine的close方法，还会对流程引擎进行关闭操作，这些操作包括关闭异步执行器 (AsyncExecutor)和执行数据库表删除(drop)。需要让其删除数据表，前提是要将流程引擎配置的databaseSchemaUpdate属性设置为create-drop。
{% asset_img content18.png content18 %}

流程引擎名称
每个ProcessEngine实例均有自己的名称，在ProcessEngines的 Map中，会使用该名称作为Map的key值。如果不为ProcessEngine设置名称，Activiti会默认将其设置为"default"。ProcessEngine本身没有提供设置名称的方法，该方法由 ProcessEngineConfiguration提供。
{% asset_img content19.png content19 %}

