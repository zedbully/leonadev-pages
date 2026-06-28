---
title: "DEX 很小不是异常：御盾 release 的 native/assets 分层载体实测"
permalink: /articles/2026-06-28-yudun-s21-carrier-distribution-static-evidence/
---

# DEX 很小不是异常：御盾 release 的 native/assets 分层载体实测

原文已发布在御盾技术内容中心：[DEX 很小不是异常：御盾 release 的 native/assets 分层载体实测](https://dun.leonadev.com/article/yudun-s21-carrier-distribution-static-evidence)。

## 本轮主目标

本轮自主检索到 2026-06-28 晚间的新 release 候选，分析方向从上午的静态明文锚点切换到 DEX、native 与 assets 分层载体。目标是回答一个 PoC 验收中常见的问题：当 DEX 体量很小、JADX 仍能看到少量 Java 表面、主要体量落在 native/assets 层时，应该如何解释这个结果。

| 目标 | 本轮状态 | 公开结论 |
|---|---|---|
| 候选新鲜度 | 已确认 | 使用晚间新候选，不复用上午明文锚点证据。 |
| DEX 体量 | 已执行 | 只有一个 DEX，体量不是整包主要来源。 |
| native/assets 载体 | 已执行 | 主要体量集中在 native 与 assets 分层载体。 |
| Manifest 入口 | 已执行 | 公开组件和权限面保持较窄。 |
| JADX 表面 | 已复核 | 可见部分壳层/框架表面，不能解释成完整业务恢复。 |
| 动态运行 | 未形成公开证据 | 安装前置不足，本轮不写动态通过。 |

## 公开证据摘要

- APK 约 78.29 MB，共 24 个条目，适合做载体分层静态复核。
- DEX 只有一个，未压缩体量约 252 KB。
- lib 层有两个 native 文件，其中一个运行载体约 14.90 MB。
- assets 层有两个条目，其中一个核心载体约 62.85 MB。
- 整包压缩率约 0.2%，关键载体以存储形态存在。
- Manifest 公开组件面只有一个启动 activity，没有 provider、service、receiver 和权限声明。
- application 层存在组件工厂介入，native 解压策略不是普通默认展开。
- JADX 产出 53 个 Java 文件、约 13,797 行表面，同时工具结束时有错误。
- content URI、IP 字面量、凭据类关键词、注入调试类关键词计数均为 0。
- URL 类别只有 2 条，归类为编译工具链来源信息，不是业务端点。

## 公开边界

本文不公开可定位样本、完整条目清单、源码、类路径、原始字符串、安装校验原文或可复现攻击步骤。公开内容只保留测评目标、脱敏证据、正向结果和未执行边界。

## 相关页面

- [御盾 App 加固产品页](https://dun.leonadev.com/article/yudun-app-hardening-product)
- [App 加固 PoC 验收指南](https://dun.leonadev.com/article/app-hardening-poc-acceptance-guide)
- [御盾性能与兼容性中心](https://dun.leonadev.com/article/yudun-performance-compatibility-center)
- [市场 App 加固产品对比页](https://dun.leonadev.com/article/mobile-app-hardening-market-comparison-framework)
