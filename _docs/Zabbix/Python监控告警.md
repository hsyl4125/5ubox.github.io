---
title: 钉钉/微信消息告警
category: 监控告警
order: 1
date: 2020-10-10 11:00:00 +0800
tags: zabbix,钉钉,微信
---
> 作者:马鹏飞         邮箱:mapengfei4125@163.com         微信:ttsysprep

### 钉钉告警

1. 自动生成配置文件，第一次运行会在程序所在目录生成以程序名命名的配置文件

   ```shell
   python3 dingding.py 
   ```

2. 修改配置文件内容

   ```shell
   [Webhook]
   #将此URL修改为钉钉自定义机器人接口URL
   url = https://oapi.dingtalk.com/robot/send?access_token=390e9376820B5559b32bc0f49494193223660222d8ab4d1092b620a5beaeac86
   
   [ProxyServer]
   #Socket5代理服务器默认禁用，如需启用修改为“state = on”
   state = off
   #代理服务器IP，默认支持两台Socket5代理服务器同时提供服务，当servera无法使用时自动切换到serverb发送消息
   servera = xxx.xxx.xxx.xxx
   serverb = xxx.xxx.xxx.xxx
   port = 1080
   
   [Log]
   #日志文件位置，默认存在程序相同目录下
   logdir = /Users/mapengfei/Documents/DingDing
   level = INFO
   
   [license]
   #License高级功能，需购买License
   cdkey = none
   
   
   ```

   

3. 参数用空格隔开，发送告警消息到钉钉

   ```shell
   python3 dingding.py 发送消息的标题  发送消息的内容
   ```
   
   <img src="/images/zabbix/images/example1.jpeg" style="zoom:50%;" />

### 微信告警

开发中，敬请期待……

### 高级功能

开发中，敬请期待……

