---
title: Splunk搭建SIEM
tags:
  - Splunk
  - SIEM
categories: SOC
keywords: Splunk
description: 本文记录了作者在使用Splunk搭建SIEM时所需的知识点以及操作流程
top_img: https://pic.imgdb.cn/item/64aae46e1ddac507cc3c4359.jpg
cover: https://pic.imgdb.cn/item/64aae46e1ddac507cc3c4359.jpg
---
## Splunk 简介

### 总览

Splunk 是一款强大的数据分析和监控平台，专注于处理大规模、复杂的数据集。它能够实时、高效地索引、搜索、可视化和分析结构化和非结构化数据，为用户提供洞察业务、发现异常、解决问题的能力。
Splunk 支持从各种数据源中汇集数据，包括应用程序日志、网络流量、传感器数据等。用户可以通过简单直观的搜索查询语言，对数据进行高级搜索、分析和可视化。其强大的查询和分析功能，使得用户能够发现隐藏在数据中的关键信息、模式和趋势，用以优化业务运营、增强安全性和做出数据驱动的决策。
Splunk 还具备灵活的部署选项，可以在本地、云端或混合环境中部署，以满足不同组织的需求。综而言之，Splunk 是一个多功能、易用且高度可定制的数据分析平台，为企业提供了丰富的数据洞察，助力业务持续优化和创新。

### Splunk UF

Splunk Universal Forwarder（UF）是 Splunk 平台的一个重要组件，用于将数据从数据源（如服务器、终端设备、应用程序等）采集、转发并发送至 Splunk 索引器（Splunk Indexer）。UF 轻量且高效，设计用于在采集端点上安装和运行，以最小化资源占用和网络流量。

UF的主要功能包括：数据采集、日志转发、安全性、数据压缩、数据处理和数据路由。它能够采集多种数据类型，包括日志文件、事件数据、指标和其他格式的数据。一旦数据被采集，UF 会将其压缩并安全地发送到 Splunk 索引器进行索引、搜索和分析。

通过使用 Splunk UF ，组织可以实现实时数据采集，确保数据安全传输，降低网络流量，提高系统和网络性能。它允许企业集中管理数据采集配置，确保数据的完整性、可靠性和保密性。Splunk UF 是构建在 Splunk 平台上的重要工具，为实时数据采集和分析提供了可靠的基础。

### Splunk 下载

如果你是第一次访问 Splunk 网站，需要先注册一个 Splunk 用户，默认下载的是 60 天 Enterprise 试用版， 60 天试用之后将自动转化为 Free 版，转化位 Free 版后每日处理的日志量最高位 500M 。
3
[Splunk Enterprise 9.1.1 下载](https://www.splunk.com/en_us/download/splunk-enterprise.html#)

[Splunk Forwarder 9.1.1 下载](https://www.splunk.com/en_us/download/universal-forwarder.html)
## Splunk Server搭建

下载 Splunk 安装包之后，直接进行安装即可

