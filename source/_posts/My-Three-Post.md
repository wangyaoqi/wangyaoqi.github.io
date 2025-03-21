---
title:  Jira 容器化部署
date: 2024-07-13 20:57:40
tags:
---
# 使用 Jira 容器化部署一键

## Jira
团队项目协作，出色的工作流与插件

## 安装文档
https://github.com/haxqer/jira/blob/rm/README_zh.md


## 操作流程
jira
README | 中文文档

Long Term Support Version: v9.12.11
Latest Version: v9.17.0
新的使用方式 ，您可方便自行升级、修改各参数，支持https (感谢 xsharp).
Arm Version
新的Confluence/Jira版本仅支持数据中心许可证

默认端口: 8080

环境要求
docker: 17.09.0+
docker-compose: 1.24.0+
使用 docker-compose 启动 jira
启动 jira & mysql
    git clone https://github.com/haxqer/jira.git \
        && cd jira \
        && git checkout rm \
        && docker-compose pull \
        && docker-compose up
以守护进程的方式启动 jira & mysql
    docker-compose up -d
默认的 数据库(mysql8.0) 配置:
    driver=mysql8.0
    host=mysql-jira
    port=3306
    db=jira
    user=root
    passwd=123456
使用 docker 启动
启动 jira
    docker volume create jira_home_data && docker network create jira-network && docker run -p 8080:8080 -v jira_home_data:/var/jira --network jira-network --name jira-srv -e TZ='Asia/Shanghai' haxqer/jira:9.17.0
然后配置你的数据库
破解 jira
docker exec jira-srv java -jar /var/agent/atlassian-agent.jar \
    -p jira \
    -m Hello@world.com \
    -n Hello@world.com \
    -o your-org \
    -s you-server-id-xxxx
初次安装 jira 时，安装页面会显示你的 server-id

破解 jira 的插件
例如: 你想要破解 BigGantt 插件
从 jira marketplace 中安装 BigGantt 插件
查看 BigGantt 的 App Key 是 : eu.softwareplant.biggantt
然后执行 :
docker exec jira-srv java -jar /var/agent/atlassian-agent.jar \
    -p eu.softwareplant.biggantt \
    -m Hello@world.com \
    -n Hello@world.com \
    -o your-org \
    -s you-server-id-xxxx
在 System -> System Info 页面的 Server ID 字段即为你的 server-id

例如:

docker exec jira-srv java -jar /var/agent/atlassian-agent.jar \
    -p eu.softwareplant.biggantt \
    -m Hello@world.com \
    -n Hello@world.com \
    -o https://jira.haxqer.cm \
    -s BSFN-Q264-VGCZ-0AOE
最后粘贴生成的 licence
Datacenter license
添加 -d 参数即可生成 datacenter license

docker exec jira-srv java -jar /var/agent/atlassian-agent.jar \
    -d \
    -p jira \
    -m Hello@world.com \
    -n Hello@world.com \
    -o your-org \
    -s you-server-id-xxxx
如何升级
cd jira && git pull
docker pull haxqer/jira:rm && docker-compose stop
docker-compose rm
输入 y 之后重启 jira:

docker-compose up -d
Arm
已经测试过的机器:

Mac mini(M1,2020)
感谢:

odidev 提供的 arm image.
    git clone https://github.com/haxqer/jira.git \
        && cd jira \
        && git checkout rm && cd lts_arm \
        && docker-compose pull \
        && docker-compose up
破解 Jira Service Management(jsm)
感谢:

d1m0nstr for Jira Service Management
docker exec jira-srv java -jar /var/agent/atlassian-agent.jar \
    -p jsm \
    -m Hello@world.com \
    -n Hello@world.com \
    -o your-org/ \
    -s you-server-id