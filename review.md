# Python在Netflix

[Python at Netflix](https://link.medium.com/nvCPoeUE5Y)

正如我们许多人准备去参加PyCon大会，我想分享一些Python在Netflix使用的实践。我们在整个环节都使用了Python，从决定如何操作CDN到如何提供所有的视频给一亿四千八百万会员。我们使用和捐助了很多开元的Python包，有一些会在下面提到。如果你对这些感兴趣，可以打开[网站](https://pycon.org/)找到我们。我们捐赠了一部分Netflix的经典海报给PyLadies Auction然后期望在那也能看到你。

## Open Connect

[Open Connect](https://openconnect.netflix.com)是Netflix的内容分发网络(CDN)。一个简单、不明确的途径理解Netflix基础设施就是当你在远端按下Play按钮之前AWS会发生什么，尽管在Open Connect网络之后已经发生了。Open Connect CDN的网络服务会将内容分发给离得最近的终端用户，除了可以提高流媒体速度也可以减少Netflix和ISP合作伙伴的网络消耗。

各种各样的软件系统都需要设计、构建和操作这个CDN基础设施，有相当一部分的都是用Python完成的。网络设备构建的一大部分CDN都是用Python应用管理的。这个应用跟踪我们的网络详细信息：什么设备，什么型号，哪个硬件组件，分布在哪个站点。这些设备的配置被其他包含真实来源、管理配置的应用的系统控制和备份。监控和其他操作的集群设备也是另外一个Python应用。Python很长一段时间在网络编程方面成为流行的编程语言是因为它是一个直观的语言可以让工程师快速的解决网络问题。后来，越来越多的库被开发出来，让这个语言更好的使用和学习。

## Demand Engineering

Demand Engineering负责了局部故障切换、分流、满载操作和Netflix cloud的效能。我们可以骄傲的说我们团队的工具主要是用Python开发的。精心策划的故障切换服务使用了numpy和scipy来执行数字解析，boto3用来改变AWS基础设施，rq用来运行异步工作负荷然后我们将他们包装到一个轻量级的叫做Flash API。偶然掉进了bpython的shell会节省很多时间。

我们有Jupyter Notebooks大量的用户和nteract来分析可使用的数据和原型分析工具用来帮助我们发现容量回收。

## 核心(CORE)

核心团队使用Python在警报和统计分析工作中。我们依赖了很多统计和计算的库(numpy,scipy,ruptures,pandas)来帮助自动分析1000多个警告系统的错误指标。我们开发了一系列使用在团队内外的分布式关联系统，可以并行的处理大量的分析工作以便快速的交付结果。

Python也经常用来处理自动任务，数据探测和清洗，和方便的可视化工作。

## 监控，告警和自动修复(Monitoring,alerting and autoremediation)

有洞察力的工程团队可以可靠的构建和操作运营情况、告警、诊断、自动修复工具。与越来越流行的Python，团队提供了越来越多的服务给Python客户端。我们的例子是Spectator这个Python客户端库，这个库为检测仪表装置代码记录一系列时间和空间。我们构建的Python库也影响了其他的Netflix平台级服务。另外，Winston和Bolt产品也是使用Python框架构建的(Gunicorn + Flask + Flask-RESTPlus)。

## 信息安全(Information Security)

信息安全团队使用Python来完成Netflix高杠杆率的目标：安全自动化、风险分类、自动修复和风险点识别。我们有一部分成功的Python开源代码，包括Security Monkey(我们团队最活跃的开源项目)。我们利用Bless来影响Python保护我们的SSH资源。我们安全团队使用Repokid影响Python来帮助IAM管控权限的基础设施。我们使用Lemur来帮助生成LTS认证。

我们最近的一些项目包含了Prism：一个批量处理框架来帮助安全工程师测量铺过的路，危险因素，识别代码中的风险。我们当前在Prism使用了Python和Ruby的库。Diffy这个规则分发工具就是完全用Python写的。我们也会使用Python构建的Lanius来发现敏感数据。

## 个性化算法(Personalization Algorithms)

我们广泛的使用Python在个性化机器学习基础设施，训练一些机器学习模块用在Netflix的关键点上：从推荐算法到个性化主页到营销算法。比如，一些算法使用了TensorFlow，Keras和PyTorch来学习深度神经网络，XGBoost和LightGBM来学习Gradient Boosted Decision Trees(例如numpy,scipy,sklearn,matplotlib,pandas,cvxpy)。因为我们不断尝试新特性，我们使用Jupyter Notebooks来驱动我们的实验。我们还开发了一部分高级的库用来帮助融合到我们的微服务中。

## 机器学习基础设施(Machine Learning Infrastructure)

除了个性化之外，Netflix使用机器学习到上百个案例中。大多数的应用都是用Metaflow驱动的，一个Python框架可以简单的从原型阶段到执行ML工程。

Metaflow推动了Python的发展：我们很好的优化和并行Python代码取10Gbps的数据，在内存中处理上百万的数据，精心计算数以万计的CPU核心。

## 笔记本(Notebooks)

我们在Netflix热衷于Jupyter notebooks，之前我们也写了reasons and nature of this investment。

但是Python扮演了提供服务的重要角色。当我们用Jupyter ecosystem开发、调试、探索和对比不同Python是主要的编程语言。我们使用Python来构建Jupyter服务的用户扩展，比如日志、存档、发布和克隆notebook在我们的用户代表。

我们通过不同的Jupyter内核提供了很多Python用法，使用Python来管理部署那些内核规范。

## 编制(Orchestration)

大数据编制团队有责任提供所有服务和工具来安排和执行ETL和自组织网管道。

编制服务中大多数组件使用Python开发的。从我们的调度程序开始，使用了Jupyter Notebooks的papermill来提供模板化工作类型(Spark,Presto,...)。这允许我们的用户有标准的、轻松的途径来表达需要执行的工作。我们使用notebooks作为真实的执勤记录地点当需要人工干预--例如：重启在过去一个小时失败的任何事情。

内部的，我们也用Python编写了一个事件驱动平台。我们从若干个系统创建了事件流来统一到一个工具中。这允许我们定义条件来过滤事件，来驳回或者路由它们。作为结果，我们在数据平台分离微服务和可视化展示。

我们的团队也构建了pygenie客户端其接口是Genie，一个联合任务执行服务。我们给这个库增加扩展应用到业务约定和整合到Netflix平台中。这些库是大数据平台编程的主要使用接口。

最后，我们的团队承诺捐赠给papermill和scrapbook开源项目。我们的工作既有属于我们自己的也有外部的。这些努力让我们从开源社区获得了一些荣誉然后我们也非常乐意去捐助那些项目。

## 实验平台(Experimentation Plaform)

科学计算团队尝试创建科学平台和分析AB tests。科学家和工程师可以贡献新的技术创新在三个方面，数据，科学和视觉化。

Metrics Repo是一个基于PyPika的Python框架，它允许贡献者写出可复用的参数化SQL查询。它提供了任何新解析的切入点。

Causal模型库是Python & R构建的科学计算和因果接口框架。它影响了PyArrow和RPy2因此在统计计算方面可以无缝的和其他语言集成。

可视化库基于Plotly。当Plotly是一个广泛的可视化规范，这有各种各样的工具允许贡献者从我们平台消费后输出。

## 合作生态(Partner Ecosystem)

合作生态组扩大Python的使用用来测试Netflix应用程序。Python构成了CI的基础设施，包括管理服务，spinnaker控制，测试查询和过滤，设备和容器上的定时测试任务。在Python中使用TensorFlow决定哪个测试在哪个设备上容易出问题和附加的事后分析。

## 视频编码和媒体云(Video Encoding and Media Cloud Engineering)

我们的团队负责Netflix的产品样本的编码，除此之外负责机器学习完成添加到产品样本中。我们使用Python为50多个项目，例如vmaf和mezzfs，我们使用媒体并行计算平台构建计算机视觉解决方案，叫做Archer，然后我们将Python使用到多个内部项目中。

## Netflix动漫和NVFX

在我们创建的动漫和VFX内容中Python对于大部分主要的应用来说是工业级的标准，所以它是轻量级的。我们所有的Maya和Nuke一体化都是用Python，我们全部的好用的工具都是用Python。我们仅仅开始把工具放到云端，期待用Python部署我们的AMIs容器。

## 内容机器学习，科学解析

内容机器学习团队大量的使用Python构建机器学习模型预测受众大小，观众类型，需求指标等。
