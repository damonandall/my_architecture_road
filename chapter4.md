---
## 用户中心诞生记

### 背景介绍

用户中心改造之前，用户数据仅仅指用户的基本信息，而用户的虚拟币信息以及用户的各种道具信息都分别开发实现，并不存在完整意义上的用户中心，这些大量的核心用户信息散落在各个模块，各个模块存储和缓存的实现也比较随意没有统一的规划。而经过我们调研，大部分业务都会围绕用户核心数据（用户基本信息、用户资产信息、用户各种虚拟道具信息）展开，这就导致业务开发中需要调用这些凌乱的模块，将结果计算聚合返回，非常容易出错，性能也较差。总结一下改造之前用户核心数据的管理存在以下问题：

- 数据存储分散，有的存在oracle有的存在postgre有的存在MongoDB，不利于管理
- 缓存实现混乱，缓存策略、缓存数据结构、失效策略和失效时间各异，经常发生数据不一致，甚至数据丢失的问题
- 虚拟道具缺乏统一抽象，而是每种道具单独开发实现，造成整个虚拟道具管理体系及其混乱，而且大量重复工作，很难维护
- 用户核心数据分散，业务高峰时需要高并发访问时需要聚合大量子服务获取合并信息，延时高，不利于业务服务性能的提高

### 改造方案

基于之前用户数据管理存在的问题，我们的改造方案为：

- 聚合业务中经常同时访问的用户核心数据到用户中心统一管理，提高用户中心的内聚性
- 对各种用户虚拟道具功能统一抽象为道具基本信息和道具功能，道具基本信息附属于用户中心，由道具中心实现道具配置的高速缓存查询
- 对各用户核心数据进行统一规划存储和缓存，保证高性能和可扩展

### 实现原则

- 极简化设计，仅满足支持大规模高并发的OLTP类的CRUD业务需求，批量更新操作、复杂条件查询等OLAP业务通过全局库或异构数据方案解决
- 查询接口设计支持各种粒度用户数据查询需求，尽量满足业务调用一次查询得到所需要的全部用户核心数据
- 设计中最大化提高性能并考虑未来的容量的可扩展性

### 上线方案

整个用户中心的改造涉及接口变更，模块功能抽象聚合，数据存储结构变更等，改造完直接上线几乎不可能，因此需要规划制定一套完善周全的上线流程来保证业务的平稳过度。整个上线过程主要分为以下几个步骤：
1. 针对重新设计的用户中心接口做两个版本的实现，版本A兼容老的存储结构，版本B使用新设计的存储结构
2. 对用户中心接口中增删改接口两个版本实现中增加发现变更事件消息的步骤发送至各自的MQ的topic中
3. 实现两个版本的同步程序分布订阅对方变更事件的topic实现数据存储及缓存的双向同步功能
4. 用户中心版本A上线
5. 业务检查所有业务程序涉及用户核心数据的操作代码统一改造为调用新版用户中心版本A
6. 检查所有业务接入完成后，进行数据割接（割接方案后面一节详述），同时开启双向数据同步
7. 逐步上线用户中心版本B，直到完成对所有版本A的替换
8. 确认业务没有异常、数据无误后关闭同步程序

### 数据割接方案

由于涉及用户核心数据，而且数据量大（亿级），而整个割接过程中业务不能停，割接要保证数据一致性、完整性。为此我们制定了一套数据割接方案，步骤如下：
1. 实现一个数据割接程序，该割接程序可以对用户全量数据进行割接，也可以对指定的用户ID列表的数据割接
2. 实现一个用户数据变更订阅程序，通过订阅用户数据变更事件收集某段时间内发生数据变化的用户ID列表
3. 数据割接前首先开启用户数据变更订阅程序，收集割接期间发生数据变更的用户ID列表
4. 开启用户数据全量割接程序直至割接完成
5. 开启用户中心双向同步程序（其实此时只有老版本会有变更需要同步）
6. 将订阅到变更的用户ID列表swap为一个空的列表，然后对原列表的用户ID进行增量数据割接
7. 反复操作步骤6，直到变更列表为一个很小的量级后，人工介入割接并核对校验数据
8. 对新老数据一致性进行反复校验，直至确保无误
9. 停止变更订阅程序，数据割接完成
