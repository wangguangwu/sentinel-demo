# Sentinel Demo Project

基于 Spring Cloud Alibaba 的微服务示例项目，演示了如何使用 Sentinel 实现服务的限流、熔断和容错功能。

## 项目架构

项目包含三个主要模块：

1. **producer-service** (端口: 8081)
   - 生产者服务，提供数据生成接口
   - 使用 @SentinelResource 实现限流和熔断
   - 集成 Nacos 进行服务注册

2. **consumer-service** (端口: 8082)
   - 消费者服务，通过 Feign 调用生产者服务
   - 实现了服务降级和熔断策略
   - 使用 Sentinel 进行流量控制

3. **gateway-service** (端口: 8080)
   - Spring Cloud Gateway 网关服务
   - 实现了路由转发和负载均衡
   - 集成 Sentinel 实现网关层面的流控

## 技术栈

- Java 17
- Spring Boot 3.2.1
- Spring Cloud 2023.0.0
- Spring Cloud Alibaba 2022.0.0.0
- Nacos: 服务注册与发现
- Sentinel: 流量控制和熔断
- OpenFeign: 服务调用
- Prometheus & Grafana: 监控和可视化

## 快速开始

### 1. 环境准备

确保已安装：
- Java 17
- Maven 3.6+
- Docker & Docker Compose

### 2. 启动基础服务

进入 docker 目录并启动所需的基础服务：

```bash
cd docker
docker-compose up -d
```

这将启动：
- Nacos: http://localhost:8848/nacos (用户名/密码: nacos/nacos)
- Sentinel Dashboard: http://localhost:8858 (用户名/密码: sentinel/sentinel)
- Prometheus: http://localhost:9090
- Grafana: http://localhost:3000 (用户名/密码: admin/admin)

### 3. 编译和启动服务

```bash
# 编译整个项目
mvn clean package

# 启动服务（按顺序）
java -jar producer-service/target/producer-service-0.0.1-SNAPSHOT.jar
java -jar consumer-service/target/consumer-service-0.0.1-SNAPSHOT.jar
java -jar gateway-service/target/gateway-service-0.0.1-SNAPSHOT.jar
```

### 4. 测试服务

通过网关访问服务：

1. 生产者服务：
```bash
curl -X POST http://localhost:8080/producer/produce -d "test-data"
```

2. 消费者服务：
```bash
curl http://localhost:8080/consumer/consume?data=test-data
```

## 功能特性

1. **服务限流**
   - 支持 QPS 限流
   - 支持线程数限流
   - 支持基于调用关系的限流

2. **熔断降级**
   - 支持基于异常比例的熔断
   - 支持基于响应时间的熔断
   - 支持自定义熔断策略

3. **系统负载保护**
   - 支持基于系统负载的保护机制
   - 支持基于 CPU 使用率的保护

4. **实时监控**
   - Prometheus 采集监控指标
   - Grafana 面板展示
   - Sentinel 控制台实时监控

## 监控面板

1. **Sentinel Dashboard**
   - 访问地址：http://localhost:8858
   - 实时查看流控规则和监控数据

2. **Nacos 控制台**
   - 访问地址：http://localhost:8848/nacos
   - 查看服务注册情况和配置管理

3. **Prometheus & Grafana**
   - Prometheus：http://localhost:9090
   - Grafana：http://localhost:3000

## Docker 环境说明

本项目使用 Docker Compose 来管理和运行所需的服务。以下是各个服务的说明：

### 1. 服务组件

#### Nacos
- 版本：2.4.3
- 端口：8848
- 用途：服务注册与发现
- 访问地址：http://localhost:8848/nacos
- 默认账号：nacos/nacos

#### Sentinel Dashboard
- 版本：1.8.8
- 端口：8858
- 用途：流量控制、熔断降级
- 访问地址：http://localhost:8858
- 默认账号：sentinel/sentinel

#### Prometheus
- 版本：v3.1.0-rc.0
- 端口：9090
- 用途：监控指标收集
- 访问地址：http://localhost:9090
- 配置文件：`docker/prometheus/prometheus.yml`

#### Grafana
- 版本：11.4.0
- 端口：3000
- 用途：监控数据可视化
- 访问地址：http://localhost:3000
- 默认账号：admin/admin

### 2. 目录结构

```bash
docker/
├── docker-compose.yml            # Docker Compose 配置文件
├── grafana/
│   ├── dashboards/              # Grafana 仪表盘配置
│   │   └── sentinel-dashboard.json
│   ├── provisioning/
│   │   └── datasources/         # 数据源配置
│   │       └── prometheus.yml
│   └── data/                    # Grafana 数据目录
├── prometheus/
│   ├── prometheus.yml           # Prometheus 配置文件
│   └── data/                    # Prometheus 数据目录
└── nacos/
    └── logs/                    # Nacos 日志目录
```

### 3. 快速开始

1. 启动服务：
```bash
cd docker
docker-compose up -d
```

2. 验证服务状态：
```bash
docker-compose ps
```

3. 查看服务日志：
```bash
# 查看所有服务日志
docker-compose logs

# 查看特定服务日志
docker-compose logs nacos
docker-compose logs sentinel
```

4. 停止服务：
```bash
docker-compose down
```

### 4. 监控配置

#### Prometheus 指标采集
Prometheus 配置了以下监控目标：
- Sentinel Dashboard: http://host.docker.internal:8858/actuator/prometheus
- Producer Service: http://host.docker.internal:8081/actuator/prometheus
- Consumer Service: http://host.docker.internal:8082/actuator/prometheus

#### Grafana 仪表盘
- 默认提供了一个基础的 Sentinel 监控仪表盘
- 包含 QPS、响应时间等基础指标
- 可以根据需要在 Grafana 中添加更多监控面板

### 5. 注意事项

1. 端口占用：
   - 确保 8848, 8858, 9090, 3000 端口未被占用
   - 如需修改端口，请更新 docker-compose.yml 文件

2. 数据持久化：
   - Prometheus 和 Grafana 的数据都已配置持久化存储
   - 数据分别存储在 `prometheus/data` 和 `grafana/data` 目录

3. 网络配置：
   - 所有服务都在 `sentinel-net` 网络下
   - 服务间可以通过服务名互相访问

4. 安全性：
   - 所有服务都配置了默认的用户名和密码
   - 建议在生产环境中修改默认密码

### 6. 故障排除

1. 服务无法启动：
   ```bash
   # 检查服务状态
   docker-compose ps
   
   # 查看详细日志
   docker-compose logs [service_name]
   ```

2. 端口冲突：
   ```bash
   # 检查端口占用
   lsof -i :[port]
   ```

3. 数据目录权限问题：
   ```bash
   # 设置正确的权限
   chmod -R 777 docker/grafana/data docker/prometheus/data
   ```

## Docker 环境配置

在启动 Docker 环境之前，需要准备以下配置文件：

### 1. Nacos 配置

在 `docker/nacos/init.d/` 目录下创建 `custom.properties` 文件：
```properties
management.endpoints.web.exposure.include=*
nacos.core.auth.enabled=false
spring.datasource.platform=mysql
db.num=1
db.url.0=jdbc:mysql://mysql:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
db.user.0=nacos
db.password.0=nacos
```

### 2. Prometheus 配置

在 `docker/prometheus/` 目录下创建 `prometheus.yml` 文件：
```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'producer-service'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['host.docker.internal:8081']

  - job_name: 'consumer-service'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['host.docker.internal:8082']

  - job_name: 'gateway-service'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['host.docker.internal:8080']
```

### 3. Grafana 配置

1. 创建 Grafana 数据源配置：
   在 `docker/grafana/provisioning/datasources/` 目录下创建 `prometheus.yml` 文件：
```yaml
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
    editable: true
```

2. 创建 Grafana 仪表盘：
   将 `sentinel-dashboard.json` 文件放在 `docker/grafana/dashboards/` 目录下

### 4. 目录结构准备

在启动 Docker 之前，需要创建以下目录结构：
```bash
docker/
├── grafana/
│   ├── dashboards/
│   │   └── sentinel-dashboard.json
│   ├── provisioning/
│   │   └── datasources/
│   │       └── prometheus.yml
│   └── data/
├── mysql/
│   ├── data/
│   └── init/
├── nacos/
│   ├── logs/
│   └── init.d/
│       └── custom.properties
└── prometheus/
    ├── data/
    └── prometheus.yml
```

可以使用以下命令创建必要的目录：
```bash
mkdir -p docker/grafana/{dashboards,provisioning/datasources,data}
mkdir -p docker/mysql/{data,init}
mkdir -p docker/nacos/{logs,init.d}
mkdir -p docker/prometheus/data
```

### 5. 权限设置

确保数据目录具有正确的权限：
```bash
chmod -R 777 docker/grafana/data
chmod -R 777 docker/mysql/data
chmod -R 777 docker/nacos/logs
chmod -R 777 docker/prometheus/data
```

## 性能测试

### 1. JMeter 测试计划

项目包含了预配置的 JMeter 测试计划，位于 `jmeter/sentinel-demo-test-plan.jmx`。该测试计划包含以下测试场景：

1. **生产者服务测试**
   - 并发用户数：50
   - 循环次数：100
   - 爬坡时间：10秒
   - 测试接口：POST /producer/produce

2. **消费者服务测试**
   - 并发用户数：50
   - 循环次数：100
   - 爬坡时间：10秒
   - 测试接口：GET /consumer/consume

### 2. 运行压测

1. 下载并安装 [Apache JMeter](https://jmeter.apache.org/download_jmeter.cgi)

2. 启动 JMeter GUI：
```bash
jmeter
```

3. 加载测试计划：
   - 文件 -> 打开
   - 选择 `jmeter/sentinel-demo-test-plan.jmx`

4. 运行测试：
   - 点击工具栏中的绿色启动按钮
   - 或使用快捷键 Ctrl + R (Windows) / Cmd + R (Mac)

5. 命令行方式运行（推荐用于实际压测）：
```bash
jmeter -n -t jmeter/sentinel-demo-test-plan.jmx -l test-results.jtl -e -o test-report
```

### 3. 查看测试结果

测试计划包含两种结果视图：

1. **结果树**：
   - 显示每个请求的详细信息
   - 包括响应时间、状态码等

2. **汇总报告**：
   - 显示聚合统计信息
   - 包括平均响应时间、吞吐量等

### 4. 压测注意事项

1. **监控资源使用**：
   - 在压测过程中监控 Sentinel Dashboard
   - 观察服务的 CPU 和内存使用情况
   - 查看 Prometheus 和 Grafana 监控指标

2. **调整限流规则**：
   - 根据压测结果适当调整 Sentinel 限流规则
   - 可在 Sentinel Dashboard 中实时修改规则

3. **压测建议**：
   - 先进行小规模压测，确认系统稳定性
   - 逐步增加并发用户数和请求频率
   - 压测时注意观察错误率和响应时间变化

4. **结果分析**：
   - 关注 95th 百分位响应时间
   - 监控错误率和异常情况
   - 分析 Sentinel 限流和熔断情况

## 注意事项

1. 确保 Nacos 和 Sentinel Dashboard 正常运行
2. 服务启动顺序：Nacos > Sentinel > Producer > Consumer > Gateway
3. 首次访问服务可能需要等待服务注册完成

## 问题排查

1. 如果服务注册失败，检查 Nacos 服务是否正常运行
2. 如果限流规则不生效，检查 Sentinel Dashboard 连接是否正常
3. 如果服务调用失败，检查服务注册状态和网络连接
