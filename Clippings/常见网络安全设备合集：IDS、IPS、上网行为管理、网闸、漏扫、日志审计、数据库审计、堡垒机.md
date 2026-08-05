---
title: "常见网络安全设备合集：IDS、IPS、上网行为管理、网闸、漏扫、日志审计、数据库审计、堡垒机"
source: "https://mp.weixin.qq.com/s/vbwzN_2OI2pPb0_3hR4ikQ"
author:
  - "[[知安的日记本]]"
published:
created: 2026-08-05
description: "每个安全设备都有不同的职责，按职责来理解就方便记忆：有的设备负责发现异常，有的负责行为管控，有的负责在线阻断，有的负责隔离交换，有的负责漏洞发现，有的负责审计留痕，还有的负责运维准入。本篇内容给你详细介绍"
tags:
  - "clippings"
---
知安的日记本 知安的日记本 *2026年5月31日 20:01*

看到 IDS、IPS、防火墙、网闸、漏扫、日志审计、数据库审计、堡垒机这些名字，感觉都和安全有关，很多人下意识会把所有设备都理解成拦截攻击的工具。

实际上每个安全设备都有不同的职责，按职责来理解就方便记忆：有的设备负责发现异常，有的负责行为管控，有的负责在线阻断，有的负责隔离交换，有的负责漏洞发现，有的负责审计留痕，还有的负责运维准入。

本篇内容采用图文的形式按“对象、位置、动作”三个维度，把常见安全设备做一次系统梳理。快速建立整体认知。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/5lO2GEAv1B5gGZtFV4wV87LAibWbjkVCzKFicObw1ZoJEQ5S7p3dOJiaO4TGEpvxv8dCXB6vEqHs8Shz2oNhkMKXc2Czq3FDzlAg0QEznrKFfg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0) ![图片](https://mmbiz.qpic.cn/mmbiz_png/5lO2GEAv1B5PDiagGdgvic9YFrpgtmRTfOLd3yE4szRXCgruLb7wVmqDxia0sVlibwJnktlxrrYcFnzRN4DsU9UkXJpiaoro5YMWAgbHNTkExlgM/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=2) ![图片](https://mmbiz.qpic.cn/mmbiz_png/5lO2GEAv1B58LaREsnicDWKjo3KaJ13CQHP23QoDF44e7ZXvhOjTf67Abe00kJj8k99GyotNExGW36RX0XN2f6EUnliba9BB8C8d0cpImIXak/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=4) ![图片](https://mmbiz.qpic.cn/mmbiz_png/5lO2GEAv1B4icBeD3aUCuibBibPS9DIcARhNTpKgF57fPPyK5qllaMRfI5Liaak2R66icUmZSDEc1ugrh3hjWLaVic06ViabTib8l4buzQAahbib2WLQ/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=6)

如果你也想从日复一日的网络运维、设备配置、故障排查里走出来，那不妨和我一样尝试去往网络安全方向发展，不仅能学到更多的技术实现更多自己的价值，最终获得更多的的自我价值收益、职业发展空间、薪资待遇、技术提升。如果现在的你实在没有方向而感到迷茫的话，那么下方公众号回复 888 可以添加我的微信，晚上有时间我带你解答交流学习。

网安干货 · 目录