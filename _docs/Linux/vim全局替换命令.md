---
title: vim全局替换命令
category: Linux
order: 1
---

**语法为 :[addr]s/源字符串/目的字符串/[option]**

**全局替换命令为：:%s/源字符串/目的字符串/g**

```shell
[addr] 表示检索范围，省略时表示当前行。
如："1，20" ：表示从第1行到20行；
"%" ：表示整个文件，同"1,$"；
". ,$" ：从当前行到文件尾；
s : 表示替换操作
[option] : 表示操作类型
如：g 表示全局替换; 
c 表示进行确认
p 表示替代结果逐行显示（Ctrl + L恢复屏幕）；
省略option时仅对每行第一个匹配串进行替换；
如果在源字符串和目的字符串中出现特殊字符，需要用"\"转义
```

下面是一些例子：

```shell
#将That or this 换成 This or that
:%s/\(That\) or \(this\)/\u\2 or \l\1/

#将句尾的child换成children
:%s/child\([ ,.;!:?]\)/children\1/g

#将mgi/r/abox换成mgi/r/asquare
:g/mg\([ira]\)box/s//mg//my\1square/g    <=>  :g/mg[ira]box/s/box/square/g

#将多个空格换成一个空格
:%s/  */ /g

#使用空格替换句号或者冒号后面的一个或者多个空格
:%s/\([:.]\)  */\1 /g

#每行的行首都添加一个字符串
:%s/^/要插入的字符串

#每行的行尾都添加一个字符串
:%s/$/要插入的字符串

#Vim 删除每行行尾的空格
:%s/\s+$//g
```

