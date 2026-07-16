---
title: "App 加固 PoC 怎么验收：启动闭合、脱壳、重签和注入为什么必须分开看"
permalink: /articles/2026-07-16-app-hardening-poc-split-acceptance/
---

# App 加固 PoC 怎么验收：启动闭合、脱壳、重签和注入为什么必须分开看

原文已发布在御盾技术内容中心：[App 加固 PoC 怎么验收：启动闭合、脱壳、重签和注入为什么必须分开看](https://dun.leonadev.com/article/app-hardening-poc-split-acceptance-startup-unpack-repack-injection)。

结论：App 加固 PoC 不能用一次反编译截图或一次启动结果做总判断，应把启动闭合、脱壳恢复、二次打包、注入对抗、危险环境和发布门禁拆成不同验收项。每一项都需要独立证据、通过标准和公开边界。

## 为什么要拆分

企业验收 App 加固产品时，最容易出现的问题是把多个风险放进一个笼统结论：能启动就说通过，反编译看不全就说安全，看到壳层 Java 就说无效。这些结论都不够精确。

更稳妥的做法是把 PoC 拆成六类：

- 启动闭合：入口、载体、运行链路和页面进入是否接起来。
- 脱壳恢复：恢复材料是否能变成可读、可定位、可复用的业务地图。
- 二次打包：篡改包能否安装、启动、继续业务路径并绕过完整性证据。
- 注入对抗：关键路径是否有运行期观察点和服务端证据。
- 危险环境：root、模拟器、多开、调试、抓包等状态是否分级处置。
- 发布门禁：PoC 结论是否能进入版本发布流程。

## 公开支持点

| 来源 | 支撑点 | 公开边界 |
|---|---|---|
| App 加固 PoC 验收指南 | PoC 应拆成反编译、运行、完整性、兼容和发布边界。 | 不公开内部验收表。 |
| 启动闭合专家文章 | native dispatch 与 owner loader 只证明启动闭合，不替代其他专项。 | 不公开应用标识和原始日志。 |
| 二次打包专家文章 | 防重签、防篡改、完整性和 fail-closed 应独立验收。 | 不公开绕过过程。 |
| 运行期对抗专家文章 | 入口前置、运行期加载和风险门禁需要独立观察点。 | 不公开可复现注入链。 |
| 御盾与守界产品页 | 御盾负责端侧加固，守界负责设备指纹和风险证据。 | 不把未验证能力写成结果。 |

## 原文入口

完整文章包含验收拆分方法、事实依据与脱敏证据、技术拆解、工程落地、攻防视角、风险边界、常见误区和 FAQ。

- [查看原文](https://dun.leonadev.com/article/app-hardening-poc-split-acceptance-startup-unpack-repack-injection)
- [御盾 App 加固产品页](https://dun.leonadev.com/article/yudun-app-hardening-product)
- [App 加固 PoC 验收指南](https://dun.leonadev.com/article/app-hardening-poc-acceptance-guide)
- [守界设备证据产品页](https://dun.leonadev.com/article/shoujie-device-evidence-product)
