---
title: "御盾 r325 Android 二次打包防护证据链"
permalink: /articles/2026-06-22-yudun-r325-repack-failclosed-evidence-chain/
---

# 御盾 r325 Android 二次打包防护证据链

这是一篇公开索引页，用于指向 dun.leonadev.com 上的新专家文章。文章基于御盾 r325 Android 加固候选包的脱敏静态/动态测评，讨论为什么二次打包防护不能只看系统签名，而要同时验证包内封印、启动链代理、运行时类加载、native bridge、SO 载体化、资产一致性和 fail-closed 运行闭合。

## 自有站原文

- [只验签为什么挡不住二次打包：御盾 r325 静态/动态测评里的 fail-closed 证据链](https://dun.leonadev.com/article/yudun-r325-android-repack-failclosed-evidence-chain)

## 公开证据摘要

| 证据维度 | 公开观察 | 支撑判断 |
|---|---|---|
| 系统签名与包内封印 | 候选包保留 Android 标准签名基础，同时存在独立包内完整性封印资产。 | 二次打包防护不能只依赖系统签名。 |
| 启动链代理 | Application、launcher 和 ComponentFactory 被纳入代理启动链。 | 防护应早于业务界面暴露。 |
| Java/DEX 与运行时加载 | 受保护字符串、ClassLoader、ByteBuffer、Zip、Digest 与 native bridge 共同出现。 | 防护链覆盖类加载与运行时上下文。 |
| SO 载体化 | native 载体具备基础硬化，静态可见面被收敛。 | 替换 native 库不能只看单点入口。 |
| 资产一致性 | 配置块、桥接表、完整性封印和 native 载体由资产承载。 | assets 也是二次打包验收面。 |
| 运行闭合 | 进入代理启动路径后，目标进程快速收口，业务面不持续暴露。 | fail-closed 要看运行态现象。 |

## 公开边界

本页不公开包标识、证书主体、签名摘要、hash、路径、命令、真实组件名、类名、方法名、JNI 符号、section、偏移、日志原文、设备标识或可复现攻击链。完整技术展开请阅读自有站原文。
