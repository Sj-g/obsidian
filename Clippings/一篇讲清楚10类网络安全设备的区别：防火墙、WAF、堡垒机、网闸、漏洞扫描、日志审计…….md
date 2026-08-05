---
title: "一篇讲清楚10类网络安全设备的区别：防火墙、WAF、堡垒机、网闸、漏洞扫描、日志审计……"
source: "https://mp.weixin.qq.com/s/fU6YXAF1M2iJ7dxvpzKRww?poc_token=HEpAc2qj2N5KlneAj2X5oP13derR-Yw30Ad_cEK1"
author:
  - "[[安安]]"
published:
created: 2026-08-05
description: "做运维、网工、搞安全，甚至刚入行的IT朋友，十有八九都被一堆设备名字绕晕过——防火墙、WAF、堡垒机、网闸、漏"
tags:
  - "clippings"
---
安安 网网又安安 *2026年7月11日 17:00*

做运维、网工、搞安全，甚至刚入行的IT朋友，十有八九都被一堆设备名字绕晕过——防火墙、WAF、堡垒机、网闸、漏扫、日志审计、数据库审计……

明明都是“安全设备”，可它们各守哪道关？谁管边界，谁管内鬼，谁管事后追责？

其实，看安全设备有个极简心法：先看保护对象，再看部署位置。

今天直接把常见的10类网络安全设备一一拆解，帮你理清它们各自的“管辖范围”和“站位逻辑”。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/3oIZz0jun02I1FFHbvlE3vhFLsZhiaQHeMlomFoJBsXdYBcVusXHJMpU9jUaxQGQQt9HmEW7MrGu4MKrsBGxyaLc5RKCFLDjbPALeHZcria4o/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0) ![图片](https://mmbiz.qpic.cn/mmbiz_png/3oIZz0jun01WQx1bJFgvfR93Dia32dHo0dvVttPJDkowSpZS1yJtcBf0TGK5Xn6qaqW3WFxjX5cgzxf6Q2HXYosOU69kkwXdWHnJOQpT8QLI/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=2) ![图片](https://mmbiz.qpic.cn/mmbiz_png/3oIZz0jun02yia0JAfEFXguP8a6G3mS8T6Vcz9lGnouQu4uuYI2D9q6CXicT0LFuqx5chfnzVjVudQib9NdDCTEIDLmPZuxuic5n55dEt2MD7iaQ/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=4) ![图片](https://mmbiz.qpic.cn/mmbiz_png/3oIZz0jun02knScniaia2GONogQgNJxNQHiblyl68Yrw5FNMtbIFKbzz45DMGvkVk20IoJ9F8amR3wo4yraavBW9SU192ibCpv8iaBicYJI392AqQ/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=6)