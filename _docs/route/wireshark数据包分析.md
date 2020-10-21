---

title: wireshark抓包过滤分析
category: 网络工程
order: 1
date: 2020-10-21 11:00:00 +0800
tags: wireshark,tcpdump,抓包,分析
---

## 捕获过滤

- 捕捉过滤器语法  

```  
语法：<Protocol> <Direction> <Host(s)> < Value> < Logical Operations> <Other expression>
Protocol（协议）: ether，fddi， ip，arp，rarp，decnet，lat， sca，moprc，mopdl， tcp ， udp 等，如果没指明协议类型，则默认为捕捉所有支持的协议。  
Direction（方向）:src， dst，src and dst， src or dst等，如果没指明方向，则默认使用 “src or dst” 作为关键字。  
Host(s): net, port, host, portrange等，默认使用”host”关键字，”src 10.1.1.1″与”src host 10.1.1.1″等价。  
Logical Operations（逻辑运算）:not, and, or 等，否(“not”)具有最高的优先级。或(“or”)和与(“and”)具有相同的优先级，运算时从左至右进行。  
```
### 常见使用的捕获过滤语句  

**只（不）捕获某主机的HTTP流量**
`host 192.168.5.231 and port 80 and http`  
只捕获主机192.168.5.231 的http流量。注意如果你的HTTP端口为8080，把80 改为8080。
`port 80 and http`  
捕获所有经过该接口的http流量。注意如果你的HTTP端口为8080，把80 改为8080。
`host 192.168.5.231 and not port 80`  
捕获主机192.168.5.231除 http 之外的其他所有流量，注意如果你的HTTP端口为8080，把80 改为8080  
`not port 80`  
捕获除 http 之外的其他所有流量，注意如果你的HTTP端口为8080，把80 改为8080。
`not port 80 and !http`  
捕获除 http 之外的其他所有流量，注意如果你的HTTP端口为8080，把80 改为8080。
`host 192.168.5.231`  
捕获源目主机均为192.168.5.231
`dst 192.168.5.231`  
捕获目的主机均为192.168.5.231
`src 192.168.5.231`  
捕获来源主机均为192.168.5.231
`net 192.168.5.0/24`  
捕获网段为d192.168.5的所有主机的所有流量
`host 192.168.5.231 and port 53`  
只捕获主机192.168.5.231 的dns流量。
`src 192.168.5.231 and port 53`  
只捕获主机192.168.5.231 对外的dns 的流量。
`dst 192.168.5.231 and port 53`  
只捕获dns服务器相应主机192.168.5.231的dns流量。
`port 53`  
捕获接口中的所有主机的dns流量

* 只（不）捕获APR流量

`host 192.168.5.231 and arp`  
只捕获主机192.168.5.231 的arp流量。
`host 192.168.5.231 and !arp`  
只捕获主机192.168.5.231 除arp外的所有流量。
`arp`  
捕获接口中的所有arp请求
`!arp`  
捕获接口中所有非arpq请求。
  
* 只捕获特定端口的流量

`tcp portrange 8000-9000 an port 80`  
捕获端口8000-9000之间和80端口的流量
`port 5060`  
捕获sip流量，因为sip的默认端口是5060。举一反三：`port 22`#捕获ssh流量

* **捕获电子邮件的流量**

`host 192.168.5.231 and port 25`  
捕获主机192.168.5.231 的POP3协议的流量。
`port 25 and portrange 110-143`  
因为电子邮件的协议：SMTP、POP3、IMAP4，所以捕获端口的流量。

* **捕获vlan 的流量**  

`vlan`  
捕获所有vlan 的流量
`vlan and (host 192.168.5.0 and port 80)`  
捕获vlan 中主机192.168.5.0 ，前提是有vlan，在wifi中不一定可以捕获到相应的流量，局域网（公司，学校里面的网络应该有vlan)  

* **更多的案例，可以参考**  
端口常识：https://svn.nmap.org/nmap/nmap-services#  
常见协议及其端口: http://tool.chinaz.com/port/#  

## 显示过滤器

捕获过滤器使用BPF语法，而显示过滤器使用wireshark专有格式。并且显示显示过滤器区分大小写，大部分使用的是小写。  

> 语法格式：`Protocol String1 String2 Comparision operator Value Logical Operations Other expression`

Protocol(协议)：该选项用来指定协议。该选项可以使用位于OSI模型第2-7层的协议。  
String1,String2(可选项)：协议的子类。  
Comparision operator: 指定运算比较符。  

| 英文写法 | C语言写法 | 含义     |
| -------- | --------- | -------- |
| eq       | ==        | 等于     |
| ne       | !=        | 不等于   |
| gt       | >         | 大于     |
| lt       | <         | 小于     |
| ge       | >=        | 大于等于 |
| le       | <=        | 小于等于 |

Logical expression: 指定逻辑运算符.  

| 英文写法 | C语言写法 | 含义     |
| -------- | --------- | -------- |
| and      | &&        | 逻辑与   |
| or       | \|\|      | 逻辑或   |
| xor      | ^^        | 逻辑异或 |
| not      | !         | 逻辑非   |

- 协议过滤器 
`arp`  显示所有ARP流量  
`ip` 显示所有IPv4流量  
`ipv6` 显示所有IPv6流量  
`tcp` 显示所基于TCP的流量数据  
  
- 应用过滤器  
`bootp` 显示所有DHCP流量  
`dns` 显示所有NDS流量，包括tcp传输和udp的dns请求和响应  
`tftp` 显示所有TFPT（简单文件传输协议）流量  
`http`显示所有HTTP命令、响应和数据传输包。但是不现实tcp握手包、tcp确认报和tcp断开包的流量数据。  
`icmp` 显示所有ICMP流量（ping命令发出的数据包）。  
  
- 字段存在过滤器 

`bootp.option.hostname`显示所有DHCP流量，包含主机名（DHCP是基于BOOTP）。  
`http.host`显示所有包含http主机名字段的数据包。通常是由一个客户端发给web服务器的请求。  
`ftp.request.command` 显示所有ftp命令数据，如USER、PASS、RETR命令。  

- 特有的过滤器  
`tcp.analysis.flags` 显示所有与tcp表示有关的包，包括丢包、重发和零窗口标志。  
`tcp.analysis.zero_window`  显示被标志的包，表示发送方的缓存空间已满  

### 常用显示过滤器及其表达式

- 数据链路层：  

筛选mac地址为04:f9:38:ad:13:26的数据包  
`eth.src == 04:f9:38:ad:13:26`

筛选源mac地址为04:f9:38:ad:13:26的数据包  
`eth.src == 04:f9:38:ad:13:26`

- 网络层：

筛选ip地址为192.168.1.1的数据包:`ip.addr == 192.168.1.1`
筛选192.168.1.0网段的数据:`ip contains “192.168.1”`
筛选192.168.1.1和192.168.1.2之间的数据包:`ip.addr == 192.168.1.1 && ip.addr == 192.168.1.2`  

筛选从192.168.1.1到192.168.1.2的数据包:`ip.src == 192.168.1.1 && ip.dst == 192.168.1.2`  

- 传输层：

筛选tcp协议的数据包:`tcp`  
筛选除tcp协议以外的数据包:`!tcp`  
筛选端口为80的数据包:`tcp.port == 80`  
筛选12345端口和80端口之间的数据包:`tcp.port == 12345 && tcp.port == 80`  
筛选从12345端口到80端口的数据包:`tcp.srcport == 12345 && tcp.dstport == 80`  

- 应用层：
`http.request`:表示请求头中的第一行（如GET index.jsp HTTP/1.1）  
`http.response`:表示响应头中的第一行（如HTTP/1.1 200 OK），其他头部都用`http.header_name`形式。  
筛选url中包含.php的http数据包:`-http.request.uri contains “.php”`  
筛选内容包含username的http数据包:`http contains “username”`  

