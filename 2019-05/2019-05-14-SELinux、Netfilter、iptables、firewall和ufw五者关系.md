---
title: 'SELinux、Netfilter、iptables、firewall和ufw五者关系'
date: 2019-05-14
permalink: /posts/2019/05/SELinux、Netfilter、iptables、firewall和ufw五者关系/
excerpt: '有点疑惑ufw和iptables的关系，就百度了一下，整出这五个，SELinux、Netfilter、iptables、firewall和ufw'
tags:
  - SELinux
  - Netfilter
  - iptables
  - firewall
  - ufw
---

## 一、五者是什么？
1. SELinux是美国国家安全局发布的一个强制访问控制系统
2. Netfilter是Linux 2.4.x引入的一个子系统，作为一个通用的、抽象的框架，提供一整套的hook函数的管理机制
3. iptables是Linux下功能强大的应用层防火墙工具。
4. firewall是centos7里面新的防火墙管理命令
5. ufw是Ubuntu下的一个简易的防火墙配置工具。


## 二、五者关系？
1. Netfilter管网络，selinux管本地。
2. iptables是用于设置防火墙，防范来自网络的入侵和实现网络地址转发、QoS等功能，而SELinux则可以理解为是作为Linux文件权限控制（即我们知道的rwx）的补充存在的
3. ufw是自2.4版本以后的Linux内核中一个简易的防火墙配置工具，底层还是调用iptables来处理的，iptables可以灵活的定义防火墙规则， 功能非常强大。但是产生的副作用便是配置过于复杂。因此产生一个相对iptables简单很多的防火墙配置工具：ufw
4. firewall是centos7里面新的防火墙管理命令，底层还是调用iptables来处理的，主要区别是iptables服务，每次更改都意味着刷新所有旧规则并从/etc/sysconfig/iptables读取所有新规则，firewall可以在运行时更改设置，而不丢失现有连接。
5. iptables是Linux下功能强大的应用层防火墙工具, 说到iptables必然提到Netfilter，iptables是应用层的，其实质是一个定义规则的配置工具，而核心的数据包拦截和转发是Netfiler。Netfilter是Linux操作系统核心层内部的一个数据包处理模块

- iptables和Netfilter关系图：

![iptables和Netfilter关系图](http://i1.bvimg.com/687356/227c71b0905cc644.png)
 
- iptables和firewall关系图：

![iptables和fillwall关系图](http://i1.bvimg.com/687356/88c4781962a34199.png)
