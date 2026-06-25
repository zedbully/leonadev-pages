---
title: "JADX 能不能还原原始算法？御盾加固 APK 的静态还原难度实测"
permalink: /articles/2026-06-26-yudun-apk-algorithm-recovery-static-assessment/
---

# JADX 能不能还原原始算法？御盾加固 APK 的静态还原难度实测

原文已发布在御盾技术内容中心：[JADX 能不能还原原始算法？御盾加固 APK 的静态还原难度实测](https://dun.leonadev.com/article/yudun-apk-algorithm-recovery-static-assessment)。

## 本轮主目标

本轮不是泛泛扫描 APK，而是围绕一个明确目标做测评：低成本静态工具能否直接还原原始算法。静态分析覆盖 JADX 反编译输出、DEX 表面、native 载体、assets 载荷、Manifest 启动链和敏感明文收敛；动态分析只作为安装烟测补充，不把入口拉起扩大成完整业务闭环。

| 目标 | 本轮状态 | 公开结论 |
|---|---|---|
| 还原原始算法 | 本轮主目标 | JADX 未直接恢复清晰原始算法面，关键保护面转向壳层、native 与 assets。 |
| 二次打包 | 后续专项 | 当前文章只记录结构基础，不公开改包、重签或运行闭合步骤。 |
| 危险环境可安装 | 后续专项 | 本轮只做普通安装烟测，不声明 root、Hook、模拟器对抗通过。 |
| DEX 脱壳 | 后续专项 | 普通 DEX 面没有直接承载完整原始算法，但脱壳需另开专项。 |
| SO 脱壳 | 后续专项 | native 表面符号线索收敛，全局导出符号统计为 0；调试与脱壳细节不公开。 |

## 公开证据摘要

- 最新 release 输出被作为静态主候选，最近 signed delivery 输出被作为安装烟测候选，避免把未签名输出误判为动态问题。
- 静态候选呈现 DEX、native、assets 和启动链分层结构，普通 Java 面没有直接承载完整原始业务算法。
- Manifest 观察到代理 Application、代理 Activity 和组件工厂参与启动链，保护入口前置。
- arm64 native 载体为 stripped ELF，全局导出符号统计为 0，低成本符号定位线索被收敛。
- assets 中存在核心载荷与完整性记录，说明发布证据链不只依赖 DEX 混淆。
- 常见 secret、token、password、api-key 和 IP 字面量模式未命中。
- signed 候选完成干净安装和入口拉起，但本文不声明完整业务链路通过。

## 公开边界

本文不公开本地路径、真实包名、文件摘要、签名、组件名、SO 名、assets 名、设备、命令、日志、符号、偏移或可复现攻击步骤。公开内容只保留测评目标、测评方法、脱敏证据和正向结论边界。

## 相关自有站页面

- [御盾 App 加固产品页](https://dun.leonadev.com/article/yudun-app-hardening-product)
- [Android/iOS App 加固选型页](https://dun.leonadev.com/article/android-ios-app-hardening-product-selection)
- [App 加固 PoC 验收指南](https://dun.leonadev.com/article/app-hardening-poc-acceptance-guide)
- [御盾性能与兼容性中心](https://dun.leonadev.com/article/yudun-performance-compatibility-center)
