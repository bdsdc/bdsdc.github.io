---
title: "运维| filebeat协同victorialog收集nginx日志"
date: 2025-09-16
lastmod: 2025-09-16
author: 'bdsdc'
linkToMarkdown: true
categories: ['运维']
tags: ['log']
---

# Filebeat协同VictoriaLogs收集nginx日志

- VictoriaLogs 兼容支持多种数据输入软件，Filebeat 也支持多种数据输入。
- VictoriaLogs 的Web UI很简陋，所以要用Grafana。
- VictoriaLogs 是HTTP访问是无认证的，需要套其他软件来实现。（默认端口9428）
- VictoriaLogs 的数据过期时间是全局的，所以如果有需求，只能部署多个实例。


## filebeat rsyslog
Filebeat 相对Logstash 性能更好，也比VictoriaLogs 自带的rsyslog输入功能更多。

## filebat nginx

### filestream

```
filebeat.inputs:
  - type: filestream
    enabled: true
    paths:
      - /mnt/d/nginx/nginx_data/logs/*.log
    processors:
    - add_fields:
        fields:
          apps: nginx
output.elasticsearch:
  hosts: ["http://192.168.31.239:9428/insert/elasticsearch/"]
  parameters:
    _msg_field: "message"
    _time_field: "@timestamp"
    _stream_fields: "host.hostname,log.file.path"

```

### nginx modules

```
# vim /etc/filebeat/modules.d/nginx.yml 
# Module: nginx
# Docs: https://www.elastic.co/guide/en/beats/filebeat/9.1/filebeat-module-nginx.html

- module: nginx
  # Access logs
  access:
    enabled: true
    # Set custom paths for the log files. If left empty,
    # Filebeat will choose the paths depending on your OS.
    var.paths: ["/data/nginx/logs/*.log"]

  # Error logs
  error:
    enabled: true


    # Set custom paths for the log files. If left empty,
    # Filebeat will choose the paths depending on your OS.
    #var.paths:

  # Ingress-nginx controller logs. This is disabled by default. It could be used in Kubernetes environments to parse ingress-nginx logs
  ingress_controller:
    enabled: false

    # Set custom paths for the log files. If left empty,
    # Filebeat will choose the paths depending on your OS.
    #var.paths:

```

```
# vi /etc/filebeat/nginx-module-filebeat.yml 
filebeat.config.modules:
  path: ${path.config}/modules.d/*.yml
  reload.enabled: true
  reload.period: 30s
setup.template.settings:
  index.number_of_shards: 1
output.elasticsearch:
  hosts: ["http://localhost:9428/insert/elasticsearch/"]
  parameters:
    _msg_field: "message"
    _time_field: "@timestamp"
    _stream_fields: "host.hostname,log.file.path"
```
## filebeat启动

### filebeat 二进制
```
# filebeat -e -c /etc/filebeat/nginx-filebeat.yml 

```

### filebeat docker 
```


```
## victorialogs 
我们采用docker方式部署
```
services:
  victoria-logs-syslog:
    image: docker.io/victoriametrics/victoria-logs:latest
    container_name: victorialogs
    restart: always
    ports:
      - "9428:9428"
      - "29514:29514"
      - "514:514/udp"
    volumes:
      - /data/victorialogs/vlogs:/victoria-logs-data
    command:
      - '-syslog.listenAddr.udp=:514'
      - '-syslog.listenAddr.tcp=:29514'
      - '--retentionPeriod=300d'
```
## grafana 
这里我们采用docker 方式部署
```
version: "3.9"
services:
  grafana:
    image: grafana/grafana:12.0.2
    volumes:
      - "/mnt/d/grafana/logs:/var/log/grafana"
      - "/mnt/d/grafana/etc:/etc/grafana"
      - "/mnt/d/grafana/data:/var/lib/grafana"
      - "./provisioning:/etc/grafana/provisioning"

    environment:
      - GF_SECURITY_ADMIN_PASSWORD=123456
      - GF_INSTALL_PLUGINS=victoriametrics-logs-datasource
    ports:
      - "3000:3000"
    container_name: grafana
    restart: always

```

```
apiVersion: 1
datasources:
    # <string, required> Name of the VictoriaLogs datasource
    # displayed in Grafana panels and queries.
  - name: victorialogs
    # <string, required> Sets the data source type.
    type: victoriametrics-logs-datasource
    # <string, required> Sets the access mode, either
    # proxy or direct (Server or Browser in the UI).
    access: proxy
    # <string> Sets URL for sending queries to VictoriaLogs server.
    # see https://docs.victoriametrics.com/victorialogs/querying/
    url: http://192.168.31.239:9428

```









