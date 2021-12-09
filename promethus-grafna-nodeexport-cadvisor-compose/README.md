# docker-compose部署Prometheus+Grafana监控系统
## 1、说明

```tex
Prometheus负责收集数据，Grafana负责展示数据。其中采用Prometheus 中的 Exporter含：
1）Node Exporter，负责收集 host 硬件和操作系统数据。它将以容器方式运行在所有 host 上。
2）cAdvisor，负责收集容器数据。它将以容器方式运行在所有 host 上。
3）Alertmanager，负责告警。它将以容器方式运行在所有 host 上。
```

## 2、前置工作

```tex
mkdir -p /usr/local/src/config
cd /usr/local/src/config #以下文件都是创建在这个目录下
```

### 2.1 prometheus.yml

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
 
# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets: ['192.168.159.129:9093']
      # - alertmanager:9093
 
# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  - "node_down.yml"
  # - "first_rules.yml"
  # - "second_rules.yml"
 
# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'
    static_configs:
- targets: ['192.168.159.129:9090']
 
  - job_name: 'cadvisor'
    static_configs:
    - targets: ['192.168.159.129:8080']
 
  - job_name: 'node'
    scrape_interval: 8s
    static_configs:
      - targets: ['192.168.159.129:9100']
```

### 2.2 alertmanager.yml

```yaml
global:
  smtp_smarthost: 'smtp.163.com:25'　　#163服务器
  smtp_from: 'tsiyuetian@163.com'　　　　　　　　#发邮件的邮箱
  smtp_auth_username: 'tsiyuetian@163.com'　　#发邮件的邮箱用户名，也就是你的邮箱
  smtp_auth_password: 'TPP***'　　　　　　　　#发邮件的邮箱密码
  smtp_require_tls: false　　　　　　　　#不进行tls验证
 
route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 10m
  receiver: live-monitoring
 
receivers:
- name: 'live-monitoring'
  email_configs:
  - to: '1933306137@qq.com'　　　　　　　　#收邮件的邮箱
```

### 2.3 node_down.yml 报警规则

```yaml
groups:
- name: node_down
  rules:
  - alert: InstanceDown
    expr: up == 0
    for: 1m
    labels:
      user: test
    annotations:
      summary: "Instance {{ $labels.instance }} down"
      description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 1 minutes."
```

### 2.4  docker-compose.yml

```yaml
version: '3.7'
 
networks:
    monitor:
        driver: bridge
 
services:
    prometheus:
        image: prom/prometheus
        container_name: prometheus
        hostname: prometheus
        restart: always
        volumes:
            - /usr/local/src/config/prometheus.yml:/etc/prometheus/prometheus.yml
            - /usr/local/src/config/node_down.yml:/etc/prometheus/node_down.yml
        ports:
            - "9090:9090"
        networks:
            - monitor
 
    alertmanager:
        image: prom/alertmanager
        container_name: alertmanager
        hostname: alertmanager
        restart: always
        volumes:
            - /usr/local/src/config/alertmanager.yml:/etc/alertmanager/alertmanager.yml
        ports:
            - "9093:9093"
        networks:
            - monitor
 
    grafana:
        image: grafana/grafana
        container_name: grafana
        hostname: grafana
        restart: always
        ports:
            - "3000:3000"
        networks:
            - monitor
 
    node-exporter:
        image: quay.io/prometheus/node-exporter
        container_name: node-exporter
        hostname: node-exporter
        restart: always
        ports:
            - "9100:9100"
        networks:
            - monitor
 
    cadvisor:
        image: google/cadvisor:latest
        container_name: cadvisor
        hostname: cadvisor
        restart: always
        volumes:
            - /:/rootfs:ro
            - /var/run:/var/run:rw
            - /sys:/sys:ro
            - /var/lib/docker/:/var/lib/docker:ro
        ports:
            - "8080:8080"
        networks:
            - monitor
```

### 2.5 **启动docker-compose**

```properties
#启动容器：
docker-compose -f /usr/local/src/config/docker-compose-monitor.yml up -d
#删除容器：
docker-compose -f /usr/local/src/config/docker-compose-monitor.yml down
#重启容器：
docker restart id
```

## 3、**附录：单独命令启动各容器**

```tex
#启动prometheus
docker run -d -p 9090:9090 --name=prometheus \
-v /usr/local/src/config/prometheus.yml:/etc/prometheus/prometheus.yml \
-v /usr/local/src/config/node_down.yml:/etc/prometheus/node_down.yml \
prom/prometheus
 
# 启动grafana
docker run -d -p 3000:3000 --name=grafana grafana/grafana
 
#启动alertmanager容器
docker run -d -p 9093:9093 -v /usr/local/src/config/config.yml:/etc/alertmanager/config.yml --name alertmanager prom/alertmanager
 
#启动node exporter
docker run -d \
  -p 9100:9100 \
  -v "/:/host:ro,rslave" \
  --name=node_exporter \
  quay.io/prometheus/node-exporter \
  --path.rootfs /host
 
#启动cadvisor
docker run                                    \
--volume=/:/rootfs:ro                         \
--volume=/var/run:/var/run:rw                 \
--volume=/sys:/sys:ro                         \
--volume=/var/lib/docker/:/var/lib/docker:ro  \
--publish=8080:8080                           \
--detach=true                                 \
--name=cadvisor                               \
google/cadvisor:latest
```

## 4、[常用的Grafna支持Promethus插件编号](https://grafana.com/grafana/dashboards)

```properties
1、主机资源监控插件 
https://grafana.com/grafana/dashboards/8919
2、Dcoker容器资源监控插件
10956、11558、6756、315、893
```

