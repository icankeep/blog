---
title: '安装npm和nodejs'
date: 2019-05-14
permalink: /posts/2019/05/安装npm和nodejs/
excerpt: 'npm和nodejs的安装'
tags:
  - npm
  - nodejs
---

# 安装最新版nodejs

## 更新ubuntu软件源
```shell
sudo apt-get update
sudo apt-get install -y python-software-properties software-properties-common
sudo add-apt-repository ppa:chris-lea/node.js
sudo apt-get update
```

## 安装nodejs
```shell
sudo apt-get install nodejs
sudo apt install nodejs-legacy
sudo apt install npm
```

## 更新npm的包镜像源，方便快速下载
```shell
sudo npm config set registry https://registry.npm.taobao.org
sudo npm config list
```

## 全局安装n管理器(用于管理nodejs版本)
```shell
sudo npm install n -g
```

## 安装最新的nodejs（stable版本）
```shell
sudo n stable
sudo node -v
```

Q: apt-get update找不到文件
// error code

W: The repository 'http://ppa.launchpad.net/chris-lea/node.js/ubuntu xenial Release' does not have a Release file.

N: Data from such a repository can't be authenticated and is therefore potentially dangerous to use.

N: See apt-secure(8) manpage for repository creation and user configuration details.

E: Failed to fetch http://ppa.launchpad.net/chris-lea/node.js/ubuntu/dists/xenial/main/binary-amd64/Packages  404  Not Found

E: Some index files failed to download. They have been ignored, or old ones used instead.

// A: 修改node相关文件：/etc/apt/sources.list.d/chris-lea-ubuntu-node_js-xenial.list

deb http://ppa.launchpad.net/chris-lea/node.js/ubuntu trusty main

deb-src http://ppa.launchpad.net/chris-lea/node.js/ubuntu trusty main

Q: node -v异常

```shell
node -v
-bash: /usr/local/bin/node: Permission denied

sudo node -v
Segmentation fault (core dumped)
```

// A: 重新安装nodejs

手动移除 /usr/local/lib/node_modules整个目录

手动移除 /usr/local/bin/node

手动移除 /usr/local/bin/n

手动移除 /usr/local/bin/npm

ubuntu软件源

修改/etc/apt/sources.list为以下软件源（改前请备份）

163和里面在部分网络下会一直下载失败，可以改回官方源下载源列表

http://wiki.ubuntu.org.cn/模板:16.04source

Ubuntu官方

deb http://cn.archive.ubuntu.com/ubuntu/ xenial main restricted universe multiverse

deb http://cn.archive.ubuntu.com/ubuntu/ xenial-security main restricted universe multiverse

deb http://cn.archive.ubuntu.com/ubuntu/ xenial-updates main restricted universe multiverse

deb http://cn.archive.ubuntu.com/ubuntu/ xenial-backports main restricted universe multiverse

deb http://cn.archive.ubuntu.com/ubuntu/ xenial-proposed main restricted universe multiverse

deb-src http://cn.archive.ubuntu.com/ubuntu/ xenial main restricted universe multiverse

deb-src http://cn.archive.ubuntu.com/ubuntu/ xenial-security main restricted universe multiverse

deb-src http://cn.archive.ubuntu.com/ubuntu/ xenial-updates main restricted universe multiverse

deb-src http://cn.archive.ubuntu.com/ubuntu/ xenial-backports main restricted universe multiverse

deb-src http://cn.archive.ubuntu.com/ubuntu/ xenial-proposed main restricted universe multiverse
