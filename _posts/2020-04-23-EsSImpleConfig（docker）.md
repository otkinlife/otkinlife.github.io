---
title: Es简要配置（Docker）
date: 2018/7/13 20:46:25
tags: 
    - Es
    - Docker
categories:
    - 工具
---

1. 配置中文分词需要安装中文分词插件
    -  下载插件（https://github.com/medcl/elasticsearch-analysis-ik/releases ）
    -  将插件解压到你的es根目录下的`plugins`目录下
    - 或者执行命令

    ```
    elasticsearch-plugin install https://github.com/medcl/ elasticsearch-analysis-ik/releases/download/v6.1.1/ elasticsearch-analysis-ik-6.1.1.zip
    ```
2. 镜像
    -  最新的es镜像地址（https://www.docker.elastic.co/#）
3. 升级es
    - 下载deb文件，安装
4. elasticsearch.yml配置

```
    #跨域配置
    http.cors.enabled: true 
    http.cors.allow-origin: "*"
    
    #集群节点配置
    
    #集群名
    cluster.name: xxx
    #节点名
    node.name: master
    #当前节点为主节点
    node.master: true
    
    #绑定ip
    network.host: 127.0.0.1
    
    #端口
    http.port: 9200
    
    #绑定master
    discovery.zen.ping.unicast.hosts: ["127.0.0.1"]
    
```
5. php-elastic sdk 文档
    - api(https://www.elastic.co/guide/en/elasticsearch/client/php-api/index.html)