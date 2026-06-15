# 微服务

## 1. Netflix 组件

### Eureka
- 注册中心
- 作用：服务注册、服务发现

### Ribbon
- 客户端负载均衡组件
- 默认策略：轮询
- 常见策略：
  - 随机
  - 轮询
  - 权重
  - 最小连接数
  - 重试
  - 自定义

### Config
- 配置中心
- 用于统一管理服务配置

### Hystrix
- 容错组件
- 主要功能：
  - 熔断
  - 降级
  - 线程隔离
- 熔断状态：
  - Closed：关闭，正常放行请求
  - Open：开启，直接拒绝请求
  - Half Open：半开，尝试恢复服务
- 隔离方式：
  - 线程池隔离
  - 信号量隔离

### Zuul
- 网关组件
- Zuul 1.0：
  - 基于传统 Servlet
  - 阻塞式 IO
- Zuul 2.0：
  - 基于 Netty
  - NIO
  - 响应式

### Feign
- 声明式远程调用组件
- 基于接口和动态代理
- 可结合 Ribbon 实现负载均衡
- 可结合 Hystrix 实现服务容错

    feign:
      hystrix:
        enabled: true

### Bus
- 消息总线
- 通常结合 MQ 使用
- 用于配置刷新、消息广播

## 2. Spring Cloud 组件

### OpenFeign
- Feign 的升级版
- 用于服务间接口调用
- 支持声明式调用

### Gateway
- 新一代网关组件
- 基于 Spring WebFlux
- 支持：
  - 路由
  - 过滤器
  - 限流

### Consul
- 注册中心
- 支持：
  - 服务注册
  - 服务发现
  - 健康检查

## 3. Spring Cloud Alibaba 组件

### Nacos
- 注册中心 + 配置中心
- 注册中心：
  - 支持服务注册
  - 支持服务发现
- 配置中心：
  - 支持统一配置管理
  - 常用 bootstrap.yml 加载配置
- 实例类型：
  - 临时实例：服务主动发送心跳
  - 非临时实例：注册中心主动探测

常见配置：

    spring:
      cloud:
        nacos:
          config:
            extension-configs:
            shared-configs:

### Sentinel
- 熔断限流组件
- 主要功能：
  - 熔断
  - 限流
  - 信号量隔离
- 限流模式：
  - 直接
  - 关联
  - 链路
- 限流效果：
  - 快速失败
  - 预热
  - 排队等待
- 网关限流：
  - API 管理
  - 快速失败
  - 均匀排队

### Seata
- 分布式事务组件
- 常见模式：
  - AT
  - TCC
  - Saga
  - XA
- 核心思想：
  - 2PC 两阶段提交
