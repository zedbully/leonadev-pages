---
title: "SO 文件名不等于 SO 载体可读：御盾 S21-R43 native 载体静态与动态烟测"
permalink: /articles/2026-06-26-yudun-s21-r43-native-carrier-static-dynamic-smoke/
---

# SO 文件名不等于 SO 载体可读：御盾 S21-R43 native 载体静态与动态烟测

原文已发布在御盾技术内容中心：[SO 文件名不等于 SO 载体可读：御盾 S21-R43 native 载体静态与动态烟测](https://dun.leonadev.com/article/yudun-s21-r43-native-carrier-static-dynamic-smoke)。

## 本轮主目标

本轮测评不重复上一轮 JADX/原始算法还原问题，而是把目标收窄到 native/SO 载体：攻击者看到包内存在 SO 外观或 native 承载物时，能不能直接当成普通 ELF 读取业务逻辑；同批可安装候选能否完成基础安装和入口拉起烟测。

| 目标 | 本轮状态 | 公开结论 |
|---|---|---|
| SO/native 载体可读性 | 本轮主目标 | 文件名或载体角色不能直接等同于普通 ELF 可读性，主要 native 表面经过符号收敛。 |
| 动态安装烟测 | 已执行 | 同批可安装候选完成安装和入口拉起，观测窗口内未形成启动阻断。 |
| 原始算法还原 | 非本轮目标 | 不重复上一轮 JADX 结论，只记录 native 与 assets 分工。 |
| 二次打包闭合 | 后续专项 | 本轮不公开改包、重签、回包或运行闭合过程。 |
| DEX/SO 脱壳 | 后续专项 | 本轮只记录静态承载结构和动态烟测，不声明脱壳结论。 |

## 公开证据摘要

- 最新构建族中 release 候选用于静态发布面评估，同批 debug 候选用于安装和入口烟测，避免把不同前置条件混成一个结论。
- release 候选呈现单 DEX、多个 native 载体和 assets 载体材料，Java 反编译面不是唯一保护面。
- Manifest 层观察到组件工厂介入，说明启动链存在保护入口前置。
- 主要 native ELF 为 ARM64 stripped 表面，普通符号表未作为可读导航面保留。
- 大体量 assets 载体承担核心保护角色，但不按普通 ELF 直接展开。
- 字符串面没有形成可公开的 IP 字面量泄露证据，少量敏感词模式仅保留内部复核。
- 同批可安装候选完成安装和入口拉起，但不扩大成完整业务链路通过。

## 公开边界

本文不公开本地路径、真实 APK 名、文件摘要、签名、包名、组件名、SO 名、assets 名、设备、命令、日志、进程、符号、偏移、section 原名或可复现攻击步骤。公开内容只保留测评目标、脱敏证据、正向结果和后续专项方向。

## 相关自有站页面

- [御盾 App 加固产品页](https://dun.leonadev.com/article/yudun-app-hardening-product)
- [App 加固 PoC 验收指南](https://dun.leonadev.com/article/app-hardening-poc-acceptance-guide)
- [御盾性能与兼容性中心](https://dun.leonadev.com/article/yudun-performance-compatibility-center)
- [市场 App 加固产品对比页](https://dun.leonadev.com/article/mobile-app-hardening-market-comparison-framework)
