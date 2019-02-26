---
layout: post
title: 大数据部技术分享第四期
icon: star-o
---
2019年2月月22日13:00--17:00朱小杰老师和沈志宏老师分享关于“大数据流水线系统piflow0.5”、“融合型图数据管理引擎PiDB0.1”两方面的技术知识。

pidb是围绕图数据技术和人工智能技术的数据综合管理工具，基于Neo4j的改进方案支持文件存储，融合AI算法，支持存储、管理、查询和简单分析处理。
## 介绍内容

- [问题背景](#问题背景)
- [解决方案](#解决方案)
- [框架设计](#框架设计)
- [实现效果](#实现效果)
- [下一步计划](#下一步计划)

## 问题背景

- 学术知识图谱的建立
  - 以专家为节点建立基于属性图模型的学术领域的知识图谱。
  - 需要存储专家的相关信息，包括姓名、研究方向、个人照片等。
  - 需要根据照片进行实体对齐，即判断照片是否是专家本人。

- 面临问题：
  - 属性图模型不直接支持图片、录音等非结构化数据的存储。
  - 现有的知识图谱管理系统不能直接处理非结构化数据。

- 目标：
  - 实现一个系统，能在知识图谱中引入非结构化数据，能从非结构化数据中抽取语义信息，用于图谱查询。

- 挑战：
  - 知识图谱对非结构化数据存储的支持：
    - 将非结构化数据与结构化数据融合管理
    - 引入非结构化数据后系统不能过分臃肿
  - 非结构化数据中信息的抽取，参与图谱的查询：
    - 非结构化数据中信息抽取算法多样
    - 依赖复杂
    - 计算耗时较大

## 解决方案

- 总体结构：把系统分为数据融合管理和信息抽取两个部分
  - 数据融合管理： 基于图数据库Neo4J，提供数据的存储管理、查询检索
  - 信息抽取：从非结构化数据中提取语义信息，供检索查询使用
  
## 框架设计
![](https://github.com/cas-bigdatalab/research-group-web/blob/master/img/pidb_architecture.png)

## 实现效果
create (zhai:Person {name: '翟天临', age:38, photo: Blob.fromURL('http://s12.sinaimg.cn/mw690/005AE7Quzy7rL8kA4Nt6b&690'), pet: Blob.fromURL('https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1550774432673&di=d624e593ae6da13442a3f1dcff874b86&imgtype=0&src=http%3A%2F%2Ft7.baidu.com%2Fit%2Fu%3D376404753%2C1656708515%26fm%3D191%26app%3D48%26wm%3D1%2C13%2C90%2C45%2C0%2C7%26wmo%3D10%2C10%26n%3D0%26g%3D0n%26f%3DJPEG%3Fsec%3D1853310920%26t%3D233e69181e084e53e12c9c52943e85d7'), car: Blob.fromURL('https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1550774659962&di=9b91a5eb8952aa965e66659e3669f191&imgtype=0&src=http%3A%2F%2Fdingyue.nosdn.127.net%2FrVge2NNqB6kKkTI65LNwiXoDG1mmOiBb3fR3C0CwepM8p1539733312296.jpg')}) create (sonIsComing:Show {name: '儿子来了', year:2019, photo: Blob.fromURL('http://s15.sinaimg.cn/mw690/005AE7Quzy7rL8j2jlIee&690')}) create (e:Event {name: '博士论文作假', actorName:'翟天临', actorPhoto: Blob.fromURL('https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1550775439865&di=0c07dfdefbcfbe6a1d0ba18e3634d298&imgtype=0&src=http%3A%2F%2Fpic.rmb.bdstatic.com%2F5fef7010e532b8757c51071e927b6477.jpeg')})

create (zhai:Person {name: '翟天临', age:38, photo: Blob.fromURL('http://s12.sinaimg.cn/mw690/005AE7Quzy7rL8kA4Nt6b&690'), pet: Blob.fromURL('https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1550774432673&di=d624e593ae6da13442a3f1dcff874b86&imgtype=0&src=http%3A%2F%2Ft7.baidu.com%2Fit%2Fu%3D376404753%2C1656708515%26fm%3D191%26app%3D48%26wm%3D1%2C13%2C90%2C45%2C0%2C7%26wmo%3D10%2C10%26n%3D0%26g%3D0n%26f%3DJPEG%3Fsec%3D1853310920%26t%3D233e69181e084e53e12c9c52943e85d7'), car: Blob.fromURL('https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1550774659962&di=9b91a5eb8952aa965e66659e3669f191&imgtype=0&src=http%3A%2F%2Fdingyue.nosdn.127.net%2FrVge2NNqB6kKkTI65LNwiXoDG1mmOiBb3fR3C0CwepM8p1539733312296.jpg')}) create (sonIsComing:Show {name: '儿子来了', year:2019, photo: Blob.fromURL('http://s15.sinaimg.cn/mw690/005AE7Quzy7rL8j2jlIee&690')}) create (e:Event {name: '博士论文作假', actorName:'翟天临', actorPhoto: Blob.fromURL('https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1550775439865&di=0c07dfdefbcfbe6a1d0ba18e3634d298&imgtype=0&src=http%3A%2F%2Fpic.rmb.bdstatic.com%2F5fef7010e532b8757c51071e927b6477.jpeg')})

create (zhai:Person {name: '翟天临', age:38, photo: Blob.fromURL('http://s12.sinaimg.cn/mw690/005AE7Quzy7rL8kA4Nt6b&690'), pet: Blob.fromURL('https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1550774432673&di=d624e593ae6da13442a3f1dcff874b86&imgtype=0&src=http%3A%2F%2Ft7.baidu.com%2Fit%2Fu%3D376404753%2C1656708515%26fm%3D191%26app%3D48%26wm%3D1%2C13%2C90%2C45%2C0%2C7%26wmo%3D10%2C10%26n%3D0%26g%3D0n%26f%3DJPEG%3Fsec%3D1853310920%26t%3D233e69181e084e53e12c9c52943e85d7'), car: Blob.fromURL('https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1550774659962&di=9b91a5eb8952aa965e66659e3669f191&imgtype=0&src=http%3A%2F%2Fdingyue.nosdn.127.net%2FrVge2NNqB6kKkTI65LNwiXoDG1mmOiBb3fR3C0CwepM8p1539733312296.jpg')}) create (sonIsComing:Show {name: '儿子来了', year:2019, photo: Blob.fromURL('http://s15.sinaimg.cn/mw690/005AE7Quzy7rL8j2jlIee&690')}) create (e:Event {name: '博士论文作假', actorName:'翟天临', actorPhoto: Blob.fromURL('https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1550775439865&di=0c07dfdefbcfbe6a1d0ba18e3634d298&imgtype=0&src=http%3A%2F%2Fpic.rmb.bdstatic.com%2F5fef7010e532b8757c51071e927b6477.jpeg')})

match (n) return n

match (n:Person {name:'翟天临'}) return n.pet->animal,n.car->plateNumber

match (n:Person {name:'翟天临'}), (s:Show {name:'儿子来了'}) return n.photo <: s.photo

match (p:Person),(e:Event) return p.photo ~: e.actorPhoto

match (p:Person),(e:Event) where p.photo ~: e.actorPhoto create (p)-[:ActIn]->(e)

match (n) return n

## 下一步计划

- 100亿实体，10000亿非结构化数据对象管理
  - 数据分片
  - 批量写加速

- AI算法库扩展
- AI算法缓存优化
- 多级属性
  - 非结构化数据的信息提取永无止境

