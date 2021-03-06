---
## 为什么我们需要微服务架构改造

### 单体架构面临的问题

- 业务发展越来越快，业务需求层出不穷，但单体应用业务高度耦合，越来越复杂，再加上相关文档的缺失，技术实现没有规范，因此除了初创团队技术负责人，其他新成员根本无法理清其中的复杂业务逻辑，理解学习业务成本过高，这样就陷入了一个怪圈，业务需求无法得到快速实现，开发团队不断加人，然后开发进度并不能显著提高，系统上线时间不断delay，老板和业务团队抱怨更大。
- 由于不断有新业务实现堆砌到单体应用中，业务系统越来越复杂，耦合性越来越高，经常改一处而动全身，回归测试工作量大，系统上线风险高，系统稳定性也越来越差，而开发团队花大量的时间来处理线上问题，从而进一步影响业务上线周期。
- 开发团队仅仅为了快速实现功能而简陋设计，系统没有进行详细的论证设计，也没有很好的对业务进行抽象，导致很多功能代码根本不能复用，大量拷贝粘贴重复工作，大量使用存储过程，从而进一步降低了系统的鲁棒性。
- 系统可扩展性较差，系统性能越来越差，很多时候靠服务器升级或增加部署实例已经无法有效提高系统吞吐量和响应时间。

### 微服务架构希望解决的问题

- 微服务首先要解决的问题就是服务拆分，主要包括两个方面即系统按照具体业务领域范畴垂直拆分和考虑业务复用情况的分层设计。
- 根据垂直业务拆分可以把原先铁板一块的系统分为几个边界相对清晰的业务领域由不同的团队负责。
- 把一些相对稳定的业务形态抽象为基础业务（如用户中心、关系中心、库存服务、主播中心等），把一些业务无关的公共服务服务拆分出来（如推送服务、im服务等）。
- 这样把一个大而复杂的单体应用系统拆分为一个个小而具体的服务，由不同的团队或人员分别开发和上线，耦合性大大降低，大量的基础服务和公共服务可直接复用，团队得到很好扩展，上线交付效率大大提高，风险降低，稳定性也得到保障。

### 微服务改造存在的风险

- 需要构建微服务基础技术栈，团队需要的技能较高，学习成本高，转变设计开发思维较难
- 已有系统耦合性高，改造难度大、风险较大
- 运维成本增加，持续构建持续交付能力不足，业务日志和监控几乎没有
- 改造落地中存在一些难点如分布式事务，olap运营管理系统中大量批量和复杂查询问题等

