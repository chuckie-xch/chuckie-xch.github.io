---
title: 修改yum源
date: 2018-01-08 14:00:53
tags: linux
---
# 修改yum源

## 1. 备份系统自带yum源文件:/etc/yum.repos.d/CentOS-Base.repo

> cd /etc/yum.repos.d/
> cp CentOS-Base.repo CentOS-Base.repo_bak

## 2. 下载ailiyun/163的yum源配置文件到/etc/yum.repos.d/

> wget -O /etc/yum.repos.d/CentOS-Base.repo 
> http://mirrors.aliyun.com/repo/Centos-7.repo

## 3. 刷缓存yum makecache并更新

> yum -y update


