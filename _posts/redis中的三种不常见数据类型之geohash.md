---
title: redis中的三种不常见数据类型之geohash
top: false
cover: false
toc: true
mathjax: true
date: 2020-10-24 23:02:06
password:
summary:
tags: geohash
categories: redis
typora-root-url: ../../source
---

>这次介绍一个在地理定位系统中使用的比较多的一种数据类型——geohash。geo是geography的缩写，说明和地理位置有关，hash可以认为是一种特殊的编码格式。

# geohash

## 介绍

geohash是redis在3.2.0及以上版本，中的一个特殊的集合，提供存储经纬度，计算距离范围等功能。和bitmap，hyperloglog不同的是，geohash在redis中存储的类型是zset。也就是说除了支持geohash相关的api之外，完全兼容zset的所有api。

### geo

该指令用于添加地理位置信息。



