+++
title = "基于ipset和ip黑名单批量封禁"
date = "2023-05-01T17:14:13+08:00"
author = "Black"
draft = false
authorTwitter = "" #do not include @
cover = ""
categories = ["Linux"]
tags = ["Linux", "ipset"]
keywords = ["Linux", "ipset"]
description = "通过iptable扩展模块ipset和shell脚本实现黑名单ip自动更新"
showFullContent = false
readingTime = false
hideComments = false
color = "" #color from the theme settings
+++

## 1.简介
前段时间研究服务器防火墙相关，偶然发现了中科大的黑名单ip列表和封锁ip的shell脚本[^1]，这里对其脚本进行了相应的优化，改为通过ipset实现ip黑名单列表自动更新和批量封禁,也可以使用自己的ip黑名单

## 2.创建ipset集合
ipset是iptable的扩展,可以创建一个ip地址的集合，进行批量管理，不像iptable规则只能针对单个ip进行管理
```bash
sudo ipset create banip hash:net hashsize 15000 maxelem 1000000 timeout 604800
```
### 2.1参数讲解
- `banip`： 自定义的ipset集合名称
- `hash`： ipset集合类型有`bitmap hash list`,其中`bitmap list`为固定大小，`hash`可以自动增加扩增
- `net`: ipset集合内数据类型，支持`ip net mac port iface`,这里因为我们的ip黑名单包含ip段，所以选择`net`类型
- `hashsize`: 集合的初始大小，默认是1024，满了会扩张为之前的两倍，这里因为要导入的ip黑名单很多所以指定的15000，
- `maxelem`: 集合的最大大小，默认为65536，这里指定的1000000
- `timeout`: 超时时间，设置为0则默认永久生效，这是设置为604800，也就是在604800秒以后自动从集合中移除
### 2.2其他指令
```bash
# []中内容为可选参数，指令输入不需要输入[]
# 查看全部指令
ipset --help
# 创建集合
ipset create 集合名称 类型 [可选参数]
# 添加ip到集合中
ipset add 集合名称 ip地址
# 从集合中移除一个ip
ipset del 集合名称 ip地址
# 测试一个ip是否在指定集合中
ipset test 集合名称 ip地址
# 查看集合内容
ipset list [集合名称]
# 清空集合
ipset flush [集合名称]
# 销毁一个集合
ipset destory [集合名称]
# 重命名集合名称
ipset rename 集合名称 新集合名称
# 交换两个集合名称
ipset swap 集合名称1 集合名称2
# 保存集合内容到文件
ipset save [集合名称] -f ipset.txt
# 从文件中恢复
ipset restore -f ipset.txt 
```

## 3.将ipset集合添加到iptable防火墙
```bash
sudo iptables -I INPUT -m set --match-set banip src -j DROP
```
### 3.1参数讲解
- `-I`: 插入规则,这里指定的INPUT链。用户访问时，默认走的INPUT链，如果服务器是通过docker进行宿主机端口映射，则是走的FORWORD链，可以在docker自动创建的DOCKER-USER链中添加该规则
- `-m`: 指定扩展模块set
- `--match-set`: 指定ipset集合banip
- `-j`: 指定动作，这里选择DROP

## 4.动态更新ipset集合
### 4.1 创建shell脚本
```bash
touch banip.sh
```
### 4.2 写入脚本内容
{{< code language="bash" title="脚本" id="1" expand="Show" collapse="Hide" isCollapsed="false" >}}
#!/bin/bash
export LC_ALL=C

# 黑名单链接
ruleurl="https://blackip.ustc.edu.cn/list.php?txt"
# ipv4正则表达式
ipv4="^(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$"
# ipv4地址段
ipv42="^(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\/(3[0-2]|[1-2][0-9]|[0-9])$"
# ipset 集合名称
setname="banip"
# 更新时间
updatetime=60
# 黑名单旧文件
oldfileurl="/apps/sh/old.blackip.txt"
# 黑名单新文件
newfileurl="/apps/sh/new.blackip.txt"

#检查sudo
checkroot() {
  [[ $EUID -ne 0 ]] && echo -e "${RED}请使用 root 用户运行本脚本！${PLAIN}" && exit 1
}

#清除规则列表
cleanrule() {
  echo "清除$setname规则列表"
  ipset flush $setname
}

#导入规则
importrule() {
  echo "向$setname导入规则"
  wget $ruleurl -O $oldfileurl 2>/dev/null
  for i in $(cat $oldfileurl); do
    if [[ "${i}" =~ $ipv4 ]] || [[ "${i}" =~ $ipv42 ]]; then
      ipset add $setname $i
    fi
  done
  ipset save -f /etc/ipset
}

#自动更新
updaterule() {
  echo "启动定时更新"
  while true; do
    date
    wget $ruleurl -O $newfileurl 2>/dev/null
    diff -u $oldfileurl $newfileurl | grep "^-[0-9]" | cut -c2- | while read ip; do
      if [[ "${ip}" =~ $ipv4 ]] || [[ "${ip}" =~ $ipv42 ]]; then
        echo del $ip
        ipset del $setname $ip
      fi
    done
    diff -u $oldfileurl $newfileurl | grep "^+[0-9]" | cut -c2- | while read ip; do
      if [[ "${ip}" =~ $ipv4 ]] || [[ "${ip}" =~ $ipv42 ]]; then
        echo add $ip
        ipset add $setname $ip
      fi
    done
    ipset save -f /etc/ipset
    mv -f $newfileurl $oldfileurl
    sleep $updatetime
  done
}

runall() {
  checkroot
  cleanrule
  importrule
  updaterule
}

runall
{{< /code >}}

### 4.3 后台运行
```bash
sudo sh -c 'nohup bash banip.sh > banip.log 2>&1 &'
```
#### 参数讲解
- `sh -c`: 便于sudo执行复杂长指令,将后面整句作为sudo执行
- `nohup`: 后台运行不挂断指令
- `> tuna-banip.log`: 其实为 1>tuna-banip.log的省略用法，命令输出写入指定日志文件中。1为标准输入流stdin,2为标准输出流stdout,3为标准错误流stderr
- `2>&1`: 将错误流重定向到标准输出流中，因为标准错误流没有缓冲区，而标准输出流有
- `&`: 结尾&表示后台运行

[^1]: [USTC IP Blacklist](https://blackip.ustc.edu.cn/intro.php)  

