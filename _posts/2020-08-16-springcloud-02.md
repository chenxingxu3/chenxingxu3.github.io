---
layout:     post
title:      02.SpringCloud学习笔记
subtitle:   Cloud相关组件的停更及替换
date:       2020-08-16
author:     Chen Xingxu
header-img: img/post-bg-road.jpg
catalog:    true
tags:
    - SpringCloud
---
# 02.SpringCloud学习笔记--Cloud相关组件的停更及替换

## 服务注册中心

| 组件      | 状态     |
| --------- | -------- |
| Eureka    | 停用     |
| ZooKeeper | 可用     |
| Consul    | 可用     |
| **Nacos** | **推荐** |

## 服务调用

| 组件                      | 状态 |
| ------------------------- | ---- |
| Ribbon                    | 可用 |
| spring-cloud-loadbalancer | 可用 |

## 服务调用 2

| 组件      | 状态 |
| --------- | ---- |
| Feign     | 停用 |
| OpenFeign | 可用 |

## 服务降级

| 组件         | 状态     |
| ------------ | -------- |
| Hystrix      | 停用     |
| Resilience4J | 可用     |
| **Sentinel** | **推荐** |

## 服务网关

| 组件        | 状态     |
| ----------- | -------- |
| Zuul        | 停用     |
| Zuul2       | 不确定   |
| **Gateway** | **推荐** |

## 服务配置

| 组件      | 状态     |
| --------- | -------- |
| Config    | 不推荐   |
| Apollo    | 可用     |
| **Nacos** | **推荐** |

## 服务总线

| 组件      | 状态     |
| --------- | -------- |
| Bus       | 不推荐   |
| **Nacos** | **推荐** |