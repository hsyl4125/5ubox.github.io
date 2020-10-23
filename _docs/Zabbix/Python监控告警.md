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

### 源代码

```python
'''
1. 用于Markdown方式发送告警信息
2. 自动生成配置文件与程序同名的.ini配置文件
3. 作者：马鹏飞
4. 作者邮箱：mapengfei4125@163.com
5. 微信：ttsysprep
6. 博客：blog.5ubox.cn
>>> import dingding.py
>>> python3 dingding.py  "消息标题"  "Markdown消息内容"
'''
from requests import post
import json
import urllib3
import time
import os.path
import configparser
from sys import argv
from logging import getLogger, FileHandler, Formatter
# 读取配置
__config__ = configparser.ConfigParser()
__app_path__, __app_filename__ = os.path.split(os.path.realpath(__file__))
__config_filename__ = __app_path__ + os.sep + os.path.splitext(__app_filename__)[0] + '.ini'
if not os.path.exists(__config_filename__):
    # 这里的url替换成你的
    __config__['Webhook'] = {'url': 'https://oapi.dingtalk.com/robot/send?access_token=390e9376820a5559bd2bc0f49494193223660222d8ab4d1092b620a5beaeac86'}
    __config__['ProxyServer'] = {'state': 'off', 'ServerA': 'xxx.xxx.xxx.xxx', 'ServerB': 'xxx.xxx.xxx.xxx', 'port': '1080'}
    __config__['Log'] = {'logdir': __app_path__, 'level': 'INFO'}
    __config__['license'] = {'CDKEY': 'none'}
    with open(__config_filename__, 'w', encoding='utf-8') as configfile:__config__.write(configfile)
__config__.read(__config_filename__, encoding='utf-8')


# 设置日志级别
__logger__ = getLogger()
__loglevel__ = __config__["Log"]["level"]  # 获取日志等级
__logger__.setLevel(__loglevel__)  # Log等级总开关
__logfile_name__ = os.path.basename(__file__).split(".")[0]  # 获取文件名
__logfile_header_time__ = time.strftime('%Y%m%d%H', time.localtime(time.time()))
__logdir__ = __config__["Log"]["logdir"]
__logfile__ = __logdir__ + "/" + __logfile_name__ + __logfile_header_time__ + ".log"
__loghandler__ = FileHandler(__logfile__, mode='a', encoding='utf8')
__formatter__ = Formatter('%(asctime)s %(levelname)s %(message)s')
__loghandler__.setFormatter(__formatter__)
__logger__.addHandler(__loghandler__)

# 发送markdown消息,消息标题，消息内容


def sendmsg(msg_title, msg):
    ProxyState = __config__["ProxyServer"]["state"]
    if ProxyState == 'on':
        ProxyServerA = __config__["ProxyServer"]["ServerA"]
        ProxyServerB = __config__["ProxyServer"]["ServerB"]
        ProxyPort = __config__["ProxyServer"]["port"]
        __proxiesA__ = {
            'http': 'socks5h://'+ProxyServerA + ':' + ProxyPort,
            'https': 'socks5h://'+ProxyServerA + ':' + ProxyPort
        }
        __proxiesB__ = {
            'http': 'socks5h://'+ProxyServerB + ':' + ProxyPort,
            'https': 'socks5h://'+ProxyServerB + ':' + ProxyPort
        }
        __logger__.debug('配置文件ProxyServerA地址：' + ProxyServerA)
        __logger__.debug('配置文件ProxyServerB地址：' + ProxyServerB)
        __logger__.debug('配置文件ProxyServer端口:' + ProxyPort)
    else:
        __proxiesA__ = ''
        __proxiesB__ = ''
        __logger__.debug('ProxyState处于未启用状态，要启用请修改配置项为"on"')

    # 整理要发送的数据
    data = {
        "msgtype": "markdown",
        "markdown": {
            "title": msg_title,
            "text": msg
        },
        "at": {
            "atMobiles": '17732637993',
            "isAtAll": 'false'
        }
    }
    # 发送信息到钉钉服务器
    urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)
    headers = {'Content-Type': 'application/json'}
    url = __config__["Webhook"]["url"]
    for proxies in (__proxiesA__, __proxiesB__):
        try:
            r = post(url=url, data=json.dumps(data),
                     headers=headers, proxies=proxies, timeout=2)
            if r.status_code == 200:
                __logger__.info(r.text)
                d = json.loads(r.text)
                if d['errmsg'] == 'ok':
                    __logger__.info('消息成功发送')
                    return 1
                elif d['errcode'] == '310000':
                    __logger__.error("钉钉发送消息需要包含关键词，请查看https://ding-doc.dingtalk.com/doc#/serverapi2/qf2nxq")
                else:
                    __logger__.debug(d['errmsg'])
            else:
                __logger__.error(d['errmsg'])
        except Exception as e:
            __logger__.debug(e)
            __logger__.error("消息发送失败，请检查网络")


if __name__ == '__main__':
    if len(argv) >= 2:
        if len(argv)>=3:
            __title__ = argv[1]
            __content__ = argv[2]
            __logger__.debug("获取到的参数：" + __title__ + ";" + __content__)
            sendmsg(__title__,__content__)
        else:
            __title__ = argv[1]
            __content__ = argv[1]
            __logger__.info("参数建议：消息标题，消息内容")
            sendmsg(__title__, __content__)
    else:
        __logger__.info("本程序支持参数，例如：dingding.py 测试 测试消息")
        sendmsg("消息标题","消息内容")
```

技术支持：

<img src="/images/zabbix/images/weixin.png" style="zoom:50%;" />