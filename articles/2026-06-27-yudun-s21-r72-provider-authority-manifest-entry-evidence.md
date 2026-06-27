---
title: "Provider authority 不该成为攻击地图：御盾 S21-R72 Manifest 入口收敛实测"
permalink: /articles/2026-06-27-yudun-s21-r72-provider-authority-manifest-entry-evidence/
---

# Provider authority 不该成为攻击地图：御盾 S21-R72 Manifest 入口收敛实测

原文已发布在御盾技术内容中心：[Provider authority 不该成为攻击地图：御盾 S21-R72 Manifest 入口收敛实测](https://dun.leonadev.com/article/yudun-s21-r72-provider-authority-manifest-entry-evidence)。

## 本轮主目标

本轮自主检索到 2026-06-27 的新 release 候选，分析方向从上一轮 SO/native 载体切换到 Manifest 与 Provider authority 暴露面。目标是验证公开 Manifest 是否还把 provider authority 或 content URI 字面量作为低成本路由锚点暴露出来。

| 目标 | 本轮状态 | 公开结论 |
|---|---|---|
| Provider authority 发布面 | 已执行 | release Manifest provider 与 authority 计数为 0。 |
| Manifest 入口面 | 已执行 | activity 与 intent-filter 摘要保持较窄，同时观察到组件工厂介入。 |
| 字符串路由锚点 | 已执行 | 未形成公开 `content://` URI 字面量证据，也未形成公开 IP 字面量证据。 |
| 动态安装烟测 | 未形成公开证据 | release 安装前置未满足，不写动态通过。 |
| 二次打包/危险环境 | 未执行 | 保留到后续专项。 |

## 公开证据摘要

- release 候选比上一轮 S21-R43 文章使用的样本更新，属于本轮新构建分析。
- release 静态发布面包含单 DEX、多 native 载体和 assets 载体。
- release Manifest provider 计数为 0，provider authority 计数为 0。
- release Manifest uses-permission 计数为 0，公开权限摘要较窄。
- application 层观察到 AppComponentFactory 介入，启动链不是裸入口。
- native 解压策略不是普通默认展开。
- release 静态字符串面未形成 `content://` URI 字面量证据。
- release 未满足安装前置，因此动态通过结论没有写入公开页。

## 公开边界

本文不公开可定位样本、路由身份、运行环境或复现流程的细节。公开内容只保留测评目标、脱敏证据、正向结果和未执行边界。

## 相关页面

- [御盾 App 加固产品页](https://dun.leonadev.com/article/yudun-app-hardening-product)
- [App 加固 PoC 验收指南](https://dun.leonadev.com/article/app-hardening-poc-acceptance-guide)
- [御盾性能与兼容性中心](https://dun.leonadev.com/article/yudun-performance-compatibility-center)
- [市场 App 加固产品对比页](https://dun.leonadev.com/article/mobile-app-hardening-market-comparison-framework)
