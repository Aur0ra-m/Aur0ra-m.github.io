---
title: "APIKiller -- 一款漏洞扫描（神）器"
date: 2023-02-14T08:00:00+08:00
draft: false
Categories: ["Golang", "安全开发"]
---

# APIKiller -- 一款漏洞扫描（神）器

> Github项目直通车：[APIKiller API Security Tool](https://github.com/Aur0ra-m/APIKiller)

## 项目背景

去年有幸进入字节无恒实验室实习，并负责抖音的安全SDLC工作，在期间挖的众多漏洞中发现很多问题都是IDOR这类的越权问题，当时想的就是能不能自动化识别啥的，毕竟一直这样也挺低效的（其实字节是有IDOR相关的DAST工具，只是有些没覆盖到），所以项目名称原本叫IDORKiller。但是最近写着写着，想着索性做成工程化的工具，于是在原有的基础上，进行整体结构上的优化，使其能够保证高效、易拓展的特性，于是就有了现在的APIKiller项目。


## 项目介绍

![img.png](/images/imgs/3_APIKiller/img.png)

## Feature
- 支持HTTP/HTTPS流量检测
- 多来源检测
  - 支持流量监听
  - 支持历史流量回扫\[目前只支持burpsuite存储流量\]
- 多功能扫描模块
  - 越权检测模块，高效精准，支持多情景检测
    - 具备多种鉴权姿势的账号设定
    - 多维度、特征化判断引擎
  - 40x bypass 模块
    - 支持常见的架构层的解析不一致导致的风险检测
    - 支持常见后门检测
  - csrf检测模块
    - 支持token检测
    - 常见的referer、origin检测
  - 【欢迎大家积极提PR】
- 多功能Filter处理，默认自带多个filter
  - 针对性扫描，例如只对 baidu.com域名进行扫描
  - 去重扫描，提高效率
  - 自动过滤静态文件(js,gif,jpg,png,css,jpeg,xml,img,svg...)
- API 运维
  - 提供简易的API Security运维平台
- 多方式漏洞发现提醒
  - Lark飞书
  - 钉钉
  - ...
- 对抗常见风控手段
  - 频控
- **【重磅】以上都可以快速进行拓展&二次开发**

> 由于多处核心组件采用了抽象的方式，所以可以给有需要的使用者进行快速的二次开发（二次开发的文档，后续将会补齐）

### 越权检测模块【由于设计需要，此模块有所修改】

> 越权检测既定分为三个子模块：未授权检测模块、单角色检测模块、多角色检测模块；但单角色检测模块和未授权检测模块存在较大优化空间，所以当前采用简易模式，仅能够适用于甲方日常，如果想用去扫描，则可以手动开启，或加以改造

![img_1.png](/images/imgs/3_APIKiller/img_1.png)


- 越权判断引擎

![img_2.png](/images/imgs/3_APIKiller/img_2.png)

- 未授权检测模块

![img_3.png](/images/imgs/3_APIKiller/img_3.png)

- 单角色检测模块
![img_5.png](/images/imgs/3_APIKiller/img_5.png)


- 多角色检测模块

![img_4.png](/images/imgs/3_APIKiller/img_4.png)

### 40x bypass模块
1. 核心检测逻辑
![img_6.png](/images/imgs/3_APIKiller/img_6.png)
2. 判定引擎
![img_7.png](/images/imgs/3_APIKiller/img_7.png)

### CSRF检测模块
1. 核心检测逻辑
![img_8.png](/images/imgs/3_APIKiller/img_8.png)

2. same-site检测逻辑
![img_9.png](/images/imgs/3_APIKiller/img_9.png)


## API运营平台

![img_10.png](/images/imgs/3_APIKiller/img_10.png)

