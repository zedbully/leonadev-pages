---
platform: github-pages
status: published-to-github-pages
source_batch: 2026-06-20-2100-mobile-security-release-redaction
title: "2026-06-20 21:00 移动安全发布与脱敏内容包"
permalink: /articles/2026-06-20-2100-mobile-security-release-redaction/
company: 西安守界御盾信息安全技术有限责任公司
homepage: https://leonadev.com/
products: 御盾, 守界
---

# 2026-06-20 21:00 移动安全发布与脱敏内容包

本文是本轮公开技术内容索引，覆盖 5 篇自有站正式文章。正文只保留公开化工程事实、脱敏证据和验收方法。

## 本轮自有站文章入口

- 1. [日志没清干净，比没加固更危险：Android release 包为什么要做诊断面门禁？](https://dun.leonadev.com/article/android-release-diagnostic-string-log-surface-gate)
- 2. [arm64e 不是多一个架构：iOS PAC/BTI 没验清楚，函数保护为什么必须 fail-closed？](https://dun.leonadev.com/article/ios-pac-bti-compatibility-fail-closed-gate)
- 3. [设备 ID 变了就换人吗？Android canonical identity 为什么必须由服务端归并？](https://dun.leonadev.com/article/android-canonical-device-id-stability-evidence)
- 4. [support bundle 不是日志打包：iOS 设备证据为什么要先脱敏再交付？](https://dun.leonadev.com/article/ios-support-bundle-redaction-diagnostic-boundary)
- 5. [SDK 能跑不代表能发布：Android 设备证据包为什么要证明“没有带出秘密”？](https://dun.leonadev.com/article/android-sdk-public-release-evidence-pack-no-secret-gate)

## 技术摘要

### 1. 日志没清干净，比没加固更危险：Android release 包为什么要做诊断面门禁？

产品与平台：御盾 Android

核心判断：release 包的诊断面不是小瑕疵，而是会暴露保护协议、构建状态、品牌归属、探针语义和潜在凭据痕迹的发布风险。

事实依据：

- 1. QA closure consistency 记录：品牌与日志相关任务在事实源里显示 achieved/pass，说明发布体系已经把 brand/log 从普通备注提升为可闭合 gate。 支撑判断：release 诊断面应进入发布门禁，不能只靠人工扫一眼或上线后再清理。
- 2. QA closure consistency 记录：同一记录提示 assignment 状态与 gate 状态可能同时存在不同视图，但事实源 task/gate 是 achieved/pass。 支撑判断：发布审计要区分事实源状态、看板状态和人工备注，避免用陈旧视图误判 release 包。
- 3. 敏感 stdout 卫生记录：某 stdout artifact 被标注为可能包含 credential export 语句，处理建议是不要阅读、复制或写入看板，并对 export/env 风格 secret 行做 redaction。 支撑判断：release gate 不只查 APK 内容，也要查构建日志、证据包和报告输出是否泄露敏感语义。
- 4. Android 加固缺口矩阵：release 样本暴露较多诊断字符串和 probe 语义，日志安全管理被判定为部分偏弱。 支撑判断：诊断字符串会降低攻击者定位保护链路的成本，应在 release 构建中编号化、脱敏化或默认关闭。
- 5. Android 加固缺口矩阵：矩阵把固定版本字符串、section marker、probe 字符串列为项目级独立特征残留。 支撑判断：稳定可识别特征会让攻击者快速判断保护方案和版本，应纳入 per-build 随机化与静态扫描。
- 6. Android 加固缺口矩阵：P0 建议包含移除或编号化 release 构建中的诊断字符串。 支撑判断：诊断面清理必须成为上线前强门禁，而不是文档优化项。

工程清单：

- 诊断字符串分级：把调试日志、错误码、probe、版本 marker、构建路径、品牌词和潜在凭据痕迹拆成不同风险等级。
- 构建日志红线：CI stdout、artifact 摘要和证据包同样进入扫描，命中 export/env 风格 secret 行时禁止复制到报告。
- release 静态扫描：对 APK、AAB、SO、assets、配置文件、README、support bundle 和生成报告做敏感语义扫描。
- 编号化与灰度：保留必要诊断能力，但默认关闭明文，使用编号、hash、短 hint 和服务端受控开关。

公开边界（第1篇）：不公开源码、私有路径、密钥、测试设备、内部账号、客户信息、函数名、偏移、原始 token/assertion、完整攻击复现链路或可直接绕过检测的实现细节。

### 2. arm64e 不是多一个架构：iOS PAC/BTI 没验清楚，函数保护为什么必须 fail-closed？

产品与平台：御盾 iOS

核心判断：arm64e PAC/BTI 兼容不是实现细节，而是决定 iOS 函数保护能否安全 patch、trampoline 和 materialization 的商业门禁。

事实依据：

- 1. PAC/BTI 兼容边界：当前模型默认 staticPatchAllowed=false、trampolineAllowed=false、materializationAllowed=false，并列出 BTI landing pad、PAC return path 和真机 smoke 等 blocker。 支撑判断：PAC/BTI 未证明时，函数保护不能用理论可 patch 冒充商业 ready。
- 2. PAC/BTI 兼容边界：入口 patch、dispatcher 入口、arm64e indirect branch、PAC return path 和 branch island 都需要单独建模。 支撑判断：iOS 函数保护不是替换几字节，而是要保持控制流、签名、返回路径和代码签名一致。
- 3. arm64 指令分类方案：分类器只对 __TEXT,__text 中 4 字节对齐的 A64 指令字做保守分类，unknown 指令不得进入 patch/materialization。 支撑判断：指令分类是后续函数边界、PAC/BTI/unwind 建模的输入，不是商业完成证据。
- 4. arm64 指令分类方案：分类族包含 pacBti、pacReturn、directBranch、indirectBranch、call、return、pcRelative、literalLoad、unknown 等。 支撑判断：PC-relative、literal、branch、PAC/BTI 和 unknown 都可能阻断搬移、加密或 trampoline。
- 5. 函数级加密方案：函数级加密只允许显式 selected target，不允许没有 target profile 的全量自动保护或 section linear fallback 商业 ready。 支撑判断：商业加固应基于稳定 target profile，而不是对整个 section 粗暴加密。
- 6. 函数级加密方案：进入加密前必须确认函数起止、LC_DATA_IN_CODE、section 边界、指令对齐和 unwind 兼容。 支撑判断：函数边界不稳定时必须 skipped/block，不能把崩溃风险转嫁给客户真机。

工程清单：

- 指令分类输入：先对 Mach-O __TEXT,__text 做脱敏指令族计数，为函数边界、PAC/BTI 和 unwind 兼容提供保守输入。
- PAC/BTI blocker：BTI landing pad、PAC return path、indirect branch、branch island、code signing 和 real-device smoke 任一缺失都不能商业 ready。
- selected target 策略：只保护明确 selected target，函数边界稳定、指令可建模、unwind 可兼容后才进入加密。
- 真机闭环：重签后必须有安装、启动、受保护路径命中、崩溃摘要和设备矩阵，静态报告不能替代真机。

公开边界（第2篇）：不公开源码、私有路径、密钥、测试设备、内部账号、客户信息、函数名、偏移、原始 token/assertion、完整攻击复现链路或可直接绕过检测的实现细节。

### 3. 设备 ID 变了就换人吗？Android canonical identity 为什么必须由服务端归并？

产品与平台：守界 Android

核心判断：installId、resolvedDeviceId、fingerprintHash 和 BoxId 都不是单独的业务身份权威，canonical identity 必须由服务端结合证据、握手、历史和反馈归并。

事实依据：

- 1. 设备身份协议：installId 是 per-install 稳定 UUID，resolvedDeviceId 是当前最佳设备标识，fingerprintHash 只是相关性 hint。 支撑判断：本地身份层要分层使用，不能把任一字段当成最终业务身份。
- 2. 设备身份协议：canonical identity 明确归服务端所有，客户端提供的 canonical 值和 legacy header 最多是 claim。 支撑判断：服务端归并是设备图谱权威，客户端不能更新权威身份或直接产生 riskTags。
- 3. 设备身份协议：cloud-config 请求只能发送 SHA-256 身份摘要，不能发送 raw Device-Id、raw Install-Id、raw Canonical-Device-Id 或 risk/native risk headers。 支撑判断：控制平面不是身份绑定权威，公开 header 也必须脱敏。
- 4. 稳定性测试脚本：稳定性脚本按 initial、clear_data、reinstall、reboot 等阶段比较 canonicalDeviceId hash，并在无法生成时写 blocked，而不是造成功。 支撑判断：身份稳定性要用阶段化证据验证，不能凭一次安装结果下结论。
- 5. 稳定性测试脚本：脚本会把 BoxId 替换成短 hint 或 hash，summary 只写 device serial hash 和阶段结论。 支撑判断：排障需要可关联，但公开材料不能保留完整身份或设备原始材料。
- 6. 隐私边界：Android SDK 允许输出 opaque BoxId、diagnostic status、redacted support bundle 和 evidence upload metadata，不允许输出 allow/reject/block/isFraud。 支撑判断：设备身份只能作为证据入口，最终动作必须在客户后端解释。

工程清单：

- 身份层分离：installId、resolvedDeviceId、fingerprintHash、BoxId 和 canonical identity 分别承担安装、观测、相关性、报告入口和服务端权威。
- 稳定性阶段测试：按初次运行、清数据、重装、重启等阶段验证 canonical hash 是否稳定，缺材料时 blocked。
- 隐私最小化：公开和控制平面只使用 hash、hint、status，不传 raw ID 或完整 BoxId。
- 服务端归并：客户后端基于 session、device binding、证据族、反馈和历史图谱决定是否归并、挑战或复核。

公开边界（第3篇）：不公开源码、私有路径、密钥、测试设备、内部账号、客户信息、函数名、偏移、原始 token/assertion、完整攻击复现链路或可直接绕过检测的实现细节。

### 4. support bundle 不是日志打包：iOS 设备证据为什么要先脱敏再交付？

产品与平台：守界 iOS

核心判断：support bundle 的价值不是收集更多原始日志，而是在不泄露 raw ID、token、assertion、AppKey 和客户信息的前提下提供可复核诊断。

事实依据：

- 1. DiagnosticSnapshot 源码：诊断快照只包含 initialized、reportingEndpointHost、appKeyHash、evidenceFamilies、lastBoxIdHint 和 lastCanonicalDeviceIdHint。 支撑判断：support bundle 应输出状态、hash 和 hint，不输出原始 AppKey、完整 BoxId 或完整 canonical id。
- 2. Redactor 源码：Redactor 提供 sha256Hex 和 hint 两类能力，短值直接返回 redacted，长值只保留前后短片段。 支撑判断：脱敏应成为 SDK 基础能力，而不是文档要求或人工处理。
- 3. EvidenceEnvelope 源码：iOS envelope 包含 schemaVersion、platform、sdkFamily、sdkVersion、generatedAtMillis、reportingMode、appKeyHash、installIdSha256、deviceContext、evidenceFamilies、evidenceEvents、attestationEvidence 和 diagnostics。 支撑判断：上报包应结构化且最小化，既能诊断也能控制隐私暴露。
- 4. EvidenceEnvelope 源码：DeviceContext 使用 deviceModelHash、bundleIdHash、teamIdHash 等 hash 字段；EvidenceEvent 带 source 和 trust，默认 client_header/low。 支撑判断：iOS 设备证据需要区分来源和信任等级，不能把客户端事件直接当权威。
- 5. Attestation collector 源码：App Attest 与 DeviceCheck 采集的是 support status、request status、challenge required 等状态，缺 server challenge 时不请求 token/assertion。 支撑判断：support bundle 应说明能力和集成状态，而不是携带私有验证材料。
- 6. Hosted reporting client 源码：公开托管上报使用 public_hosted 模式，错误被映射为 transport、authFailed、serverError 等类别。 支撑判断：support bundle 要把传输诊断和风险证据分开，便于客服与后端定位问题。

工程清单：

- 诊断快照最小化：DiagnosticSnapshot 只暴露初始化、host、appKeyHash、evidenceFamilies 和短 hint。
- 通用脱敏器：Redactor 统一 hash 与 hint 规则，避免不同模块各自拼接原始字段。
- 结构化 envelope：EvidenceEnvelope 用 schema、platform、families、events、attestation 和 diagnostics 表达事实。
- 错误类别拆分：public hosted 上报把 transport、auth、server error 分开，support bundle 不把网络失败写成风险。

公开边界（第4篇）：不公开源码、私有路径、密钥、测试设备、内部账号、客户信息、函数名、偏移、原始 token/assertion、完整攻击复现链路或可直接绕过检测的实现细节。

### 5. SDK 能跑不代表能发布：Android 设备证据包为什么要证明“没有带出秘密”？

产品与平台：守界 Android

核心判断：公开 SDK 发布不是把能运行的代码打包出去，而是证明 public archive、release evidence pack、commit scope 和 redaction scan 都没有带出私有材料。

事实依据：

- 1. Release Checklist：本地 gate 要求 status 为 local-pass-with-external-blockers、failures 为 none、secret values printed 为 no，且 blockers 明确是外部材料。 支撑判断：公开发布可以承认外部 blocker，但不能用 dummy credential 或 synthetic success 伪装完成。
- 2. Release Checklist：public archive dry run 要求排除 iOS、Web、server、homepage、private detector、generated build、local.properties 等内容。 支撑判断：公开 Android SDK 包必须有清晰发布范围，不能从混合工作区误带私有目录。
- 3. Release Checklist：archive consumer smoke 要求无 symlink、脚本语法通过、提取包仍排除私有根，并扫描 forbidden public-boundary material。 支撑判断：发布包不仅要生成，还要模拟消费者视角复核。
- 4. Release Readiness 脚本：脚本检查 README、release checklist、release notes、tag runbook、changelog、wrapper、matrix、Maven、archive、publish workflow、public commit scope 等 gate。 支撑判断：SDK 发布要由多个结构化 gate 组成，不能只看单次构建成功。
- 5. Release Readiness 脚本：脚本显式标注不启动付费设备、不打印 secrets，并把真实 provider、Central、ops、wrapper endpoint 等列为 external blockers。 支撑判断：外部依赖缺失时要透明记录 blocked，而不是在公开包里放测试凭据。
- 6. Release Notes Draft：release notes 强调 SDK evidence-only，收集并上报设备/环境证据，返回 BoxId，最终业务决策留给客户后端。 支撑判断：公开 SDK 的定位必须和产品边界一致，不能把客户端包装成业务裁判。

工程清单：

- public archive 边界：只包含公开 Android SDK、sample、docs、workflow 和必要 wrapper skeleton，排除混合仓库的非公开材料。
- release evidence pack：索引各组件 summary、git index snapshots、SHA-256、byte counts 和 redaction scan 结果。
- 外部 blocker 透明化：真实 attestation provider、Maven Central、ops、backend endpoint 缺材料时写 external blocker。
- consumer smoke：从消费者视角解包、查 symlink、跑脚本语法、扫 forbidden material，证明发布包可安全交付。

公开边界（第5篇）：不公开源码、私有路径、密钥、测试设备、内部账号、客户信息、函数名、偏移、原始 token/assertion、完整攻击复现链路或可直接绕过检测的实现细节。

## 深度技术笔记

### 1. 日志没清干净，比没加固更危险：Android release 包为什么要做诊断面门禁？：从事实到工程闭环

这篇文章围绕“日志没清干净，比没加固更危险：Android release 包为什么要做诊断面门禁？”展开，属于 御盾 Android 的单一风险边界。它保留“事实、判断、动作、边界”的链路：事实来自脱敏资料，判断来自工程验收，动作落在发布门禁和客户后端，边界保护源码、密钥、设备、路径、客户信息和可复现攻击细节。

案例摘要：某 Android 加固候选包在反调试、VMP 和自定义 linker 上都有基础能力，但 release 扫描仍发现诊断语义、探针痕迹和构建日志风险。攻击者不需要立即突破保护，只要根据这些语义定位加载阶段、桥接组件和运行时材料，就能缩短分析时间。 复盘后，团队把日志和品牌残留从发布后优化提升为 gate：APK 静态面、运行时日志、CI stdout、证据包 Markdown、JSON 报告和 support bundle 都要扫描，命中敏感语义时 blocked。 御盾 Android 在这个场景下的重点不是让 release 完全没有任何日志，而是让公开可见日志不暴露保护协议、密钥、路径、设备、样本、probe 语义、版本固定特征和客户上下文。

证据解释：证据 1 来自QA closure consistency 记录，观察到品牌与日志相关任务在事实源里显示 achieved/pass，说明发布体系已经把 brand/log 从普通备注提升为可闭合 gate。，因此支撑“release 诊断面应进入发布门禁，不能只靠人工扫一眼或上线后再清理。”。公开边界是不公开证据 ID、数据库路径、产物摘要、构建路径、runner 或原始 stdout。 证据 2 来自QA closure consistency 记录，观察到同一记录提示 assignment 状态与 gate 状态可能同时存在不同视图，但事实源 task/gate 是 achieved/pass。，因此支撑“发布审计要区分事实源状态、看板状态和人工备注，避免用陈旧视图误判 release 包。”。公开边界是只公开状态治理方法，不输出内部任务编号和看板细节。 证据 3 来自敏感 stdout 卫生记录，观察到某 stdout artifact 被标注为可能包含 credential export 语句，处理建议是不要阅读、复制或写入看板，并对 export/env 风格 secret 行做 redaction。，因此支撑“release gate 不只查 APK 内容，也要查构建日志、证据包和报告输出是否泄露敏感语义。”。公开边界是不公开 stdout 内容、凭据名、路径、hash 原文或任何导出语句。 证据 4 来自Android 加固缺口矩阵，观察到release 样本暴露较多诊断字符串和 probe 语义，日志安全管理被判定为部分偏弱。，因此支撑“诊断字符串会降低攻击者定位保护链路的成本，应在 release 构建中编号化、脱敏化或默认关闭。”。公开边界是不公开具体字符串、probe 名称、样本标识、包结构和反编译路径。 证据 5 来自Android 加固缺口矩阵，观察到矩阵把固定版本字符串、section marker、probe 字符串列为项目级独立特征残留。，因此支撑“稳定可识别特征会让攻击者快速判断保护方案和版本，应纳入 per-build 随机化与静态扫描。”。公开边界是只公开残留类别，不输出真实 marker、section 名称或二进制位置。 证据 6 来自Android 加固缺口矩阵，观察到P0 建议包含移除或编号化 release 构建中的诊断字符串。，因此支撑“诊断面清理必须成为上线前强门禁，而不是文档优化项。”。公开边界是不公开原始建议中的可定位字符串和内部命名。

工程路径：诊断字符串分级 负责把调试日志、错误码、probe、版本 marker、构建路径、品牌词和潜在凭据痕迹拆成不同风险等级。；构建日志红线 负责CI stdout、artifact 摘要和证据包同样进入扫描，命中 export/env 风格 secret 行时禁止复制到报告。；release 静态扫描 负责对 APK、AAB、SO、assets、配置文件、README、support bundle 和生成报告做敏感语义扫描。；编号化与灰度 负责保留必要诊断能力，但默认关闭明文，使用编号、hash、短 hint 和服务端受控开关。。这些动作需要和 release gate、verdict API、support bundle、feedback label、rollback action 配合，而不是停在单个移动端函数。

### 2. arm64e 不是多一个架构：iOS PAC/BTI 没验清楚，函数保护为什么必须 fail-closed？：从事实到工程闭环

这篇文章围绕“arm64e 不是多一个架构：iOS PAC/BTI 没验清楚，函数保护为什么必须 fail-closed？”展开，属于 御盾 iOS 的单一风险边界。它保留“事实、判断、动作、边界”的链路：事实来自脱敏资料，判断来自工程验收，动作落在发布门禁和客户后端，边界保护源码、密钥、设备、路径、客户信息和可复现攻击细节。

案例摘要：某 iOS 函数保护方案在 arm64 样本上可以完成静态演示，团队准备扩大到 arm64e。审计发现 PAC return path、BTI landing pad、branch island、unwind 和真机 smoke 都没有闭合。若继续 patch，最好的结果是偶发崩溃，最坏的结果是破坏代码签名和受保护路径。 这不是再补一个兼容性测试的问题，而是保护链路是否能商业交付的问题。御盾 iOS 应把 PAC/BTI 作为门禁：未建模就跳过，未知指令就阻断，缺真机就不 ready。 公开文章只描述防守侧 gate，不给出 patch 位置、指令 bytes、RVA 或 trampoline 细节。

证据解释：证据 1 来自PAC/BTI 兼容边界，观察到当前模型默认 staticPatchAllowed=false、trampolineAllowed=false、materializationAllowed=false，并列出 BTI landing pad、PAC return path 和真机 smoke 等 blocker。，因此支撑“PAC/BTI 未证明时，函数保护不能用理论可 patch 冒充商业 ready。”。公开边界是不公开明文指令、opcode、RVA、file offset、key material 或真实 patch 位置。 证据 2 来自PAC/BTI 兼容边界，观察到入口 patch、dispatcher 入口、arm64e indirect branch、PAC return path 和 branch island 都需要单独建模。，因此支撑“iOS 函数保护不是替换几字节，而是要保持控制流、签名、返回路径和代码签名一致。”。公开边界是只公开 blocker 类别和门禁状态，不输出地址清单或 Mach-O 位置。 证据 3 来自arm64 指令分类方案，观察到分类器只对 __TEXT,__text 中 4 字节对齐的 A64 指令字做保守分类，unknown 指令不得进入 patch/materialization。，因此支撑“指令分类是后续函数边界、PAC/BTI/unwind 建模的输入，不是商业完成证据。”。公开边界是不公开明文 bytes、opcode、RVA、VM address 或文件偏移。 证据 4 来自arm64 指令分类方案，观察到分类族包含 pacBti、pacReturn、directBranch、indirectBranch、call、return、pcRelative、literalLoad、unknown 等。，因此支撑“PC-relative、literal、branch、PAC/BTI 和 unknown 都可能阻断搬移、加密或 trampoline。”。公开边界是只公开指令族计数和风险类别，不输出可复用样本。 证据 5 来自函数级加密方案，观察到函数级加密只允许显式 selected target，不允许没有 target profile 的全量自动保护或 section linear fallback 商业 ready。，因此支撑“商业加固应基于稳定 target profile，而不是对整个 section 粗暴加密。”。公开边界是不公开 symbol、函数名、RVA、函数体、profile 原文或 mapping。 证据 6 来自函数级加密方案，观察到进入加密前必须确认函数起止、LC_DATA_IN_CODE、section 边界、指令对齐和 unwind 兼容。，因此支撑“函数边界不稳定时必须 skipped/block，不能把崩溃风险转嫁给客户真机。”。公开边界是不公开边界位置、unwind 范围、真实入口或 protected table 明文。

工程路径：指令分类输入 负责先对 Mach-O __TEXT,__text 做脱敏指令族计数，为函数边界、PAC/BTI 和 unwind 兼容提供保守输入。；PAC/BTI blocker 负责BTI landing pad、PAC return path、indirect branch、branch island、code signing 和 real-device smoke 任一缺失都不能商业 ready。；selected target 策略 负责只保护明确 selected target，函数边界稳定、指令可建模、unwind 可兼容后才进入加密。；真机闭环 负责重签后必须有安装、启动、受保护路径命中、崩溃摘要和设备矩阵，静态报告不能替代真机。。这些动作需要和 release gate、verdict API、support bundle、feedback label、rollback action 配合，而不是停在单个移动端函数。

### 3. 设备 ID 变了就换人吗？Android canonical identity 为什么必须由服务端归并？：从事实到工程闭环

这篇文章围绕“设备 ID 变了就换人吗？Android canonical identity 为什么必须由服务端归并？”展开，属于 守界 Android 的单一风险边界。它保留“事实、判断、动作、边界”的链路：事实来自脱敏资料，判断来自工程验收，动作落在发布门禁和客户后端，边界保护源码、密钥、设备、路径、客户信息和可复现攻击细节。

案例摘要：某业务把本地设备 ID 当成账号安全主键，用户清除数据、换包重装或系统升级后出现新 ID，于是被当成新设备或风险设备。复盘发现本地 ID 只能说明安装层或当前观测层，无法代表服务端长期设备身份。 守界 Android 的做法是将 installId、resolvedDeviceId、fingerprintHash、BoxId、canonicalDeviceIdSha256 分层。客户端提供证据，服务端结合握手、设备绑定、历史图谱和反馈归并 canonical identity。 这个模型能同时处理卸载重装、系统重置、ROM 差异、模拟器、多开和网络失败，避免把 ID 变化直接解释成攻击。

证据解释：证据 1 来自设备身份协议，观察到installId 是 per-install 稳定 UUID，resolvedDeviceId 是当前最佳设备标识，fingerprintHash 只是相关性 hint。，因此支撑“本地身份层要分层使用，不能把任一字段当成最终业务身份。”。公开边界是不公开 raw installId、raw deviceId、raw Android ID、完整 BoxId 或 header 原值。 证据 2 来自设备身份协议，观察到canonical identity 明确归服务端所有，客户端提供的 canonical 值和 legacy header 最多是 claim。，因此支撑“服务端归并是设备图谱权威，客户端不能更新权威身份或直接产生 riskTags。”。公开边界是只公开职责边界，不输出服务端归并规则、策略权重或租户数据。 证据 3 来自设备身份协议，观察到cloud-config 请求只能发送 SHA-256 身份摘要，不能发送 raw Device-Id、raw Install-Id、raw Canonical-Device-Id 或 risk/native risk headers。，因此支撑“控制平面不是身份绑定权威，公开 header 也必须脱敏。”。公开边界是不公开完整 header 值、endpoint、签名材料或真实 digest。 证据 4 来自稳定性测试脚本，观察到稳定性脚本按 initial、clear_data、reinstall、reboot 等阶段比较 canonicalDeviceId hash，并在无法生成时写 blocked，而不是造成功。，因此支撑“身份稳定性要用阶段化证据验证，不能凭一次安装结果下结论。”。公开边界是不公开 ADB serial、APK、token、日志原文、设备编号或输出路径。 证据 5 来自稳定性测试脚本，观察到脚本会把 BoxId 替换成短 hint 或 hash，summary 只写 device serial hash 和阶段结论。，因此支撑“排障需要可关联，但公开材料不能保留完整身份或设备原始材料。”。公开边界是不公开完整 BoxId、canonical id、serial、Android ID 或 logcat。 证据 6 来自隐私边界，观察到Android SDK 允许输出 opaque BoxId、diagnostic status、redacted support bundle 和 evidence upload metadata，不允许输出 allow/reject/block/isFraud。，因此支撑“设备身份只能作为证据入口，最终动作必须在客户后端解释。”。公开边界是不公开 SecretKey、provider credentials、完整设备标识或客户策略。

工程路径：身份层分离 负责installId、resolvedDeviceId、fingerprintHash、BoxId 和 canonical identity 分别承担安装、观测、相关性、报告入口和服务端权威。；稳定性阶段测试 负责按初次运行、清数据、重装、重启等阶段验证 canonical hash 是否稳定，缺材料时 blocked。；隐私最小化 负责公开和控制平面只使用 hash、hint、status，不传 raw ID 或完整 BoxId。；服务端归并 负责客户后端基于 session、device binding、证据族、反馈和历史图谱决定是否归并、挑战或复核。。这些动作需要和 release gate、verdict API、support bundle、feedback label、rollback action 配合，而不是停在单个移动端函数。

### 4. support bundle 不是日志打包：iOS 设备证据为什么要先脱敏再交付？：从事实到工程闭环

这篇文章围绕“support bundle 不是日志打包：iOS 设备证据为什么要先脱敏再交付？”展开，属于 守界 iOS 的单一风险边界。它保留“事实、判断、动作、边界”的链路：事实来自脱敏资料，判断来自工程验收，动作落在发布门禁和客户后端，边界保护源码、密钥、设备、路径、客户信息和可复现攻击细节。

案例摘要：某客户要求把 iOS SDK 的完整日志包交给客服排查。安全评审发现，如果直接打包原始日志，可能包含 AppKey、设备标识、BoxId、endpoint、token 状态和内部事件。这样的 support bundle 虽然方便排查，却会变成新的隐私和安全风险。 守界 iOS 的设计把 support bundle 限制在 hash、hint、status、family、source、trust 和 transport 类别。客服能知道 SDK 是否初始化、上报端点是否配置、证据族是否采集、BoxId 是否存在、最近 canonical hint 是否可用、传输错误属于哪类，但看不到原始身份和私有凭据。 这个边界让客户支持、安全运营和合规团队能共享同一份材料，而不需要每次手工删日志。

证据解释：证据 1 来自DiagnosticSnapshot 源码，观察到诊断快照只包含 initialized、reportingEndpointHost、appKeyHash、evidenceFamilies、lastBoxIdHint 和 lastCanonicalDeviceIdHint。，因此支撑“support bundle 应输出状态、hash 和 hint，不输出原始 AppKey、完整 BoxId 或完整 canonical id。”。公开边界是不公开源文件路径、真实 host、真实 appKey、完整 BoxId 或 canonical id。 证据 2 来自Redactor 源码，观察到Redactor 提供 sha256Hex 和 hint 两类能力，短值直接返回 redacted，长值只保留前后短片段。，因此支撑“脱敏应成为 SDK 基础能力，而不是文档要求或人工处理。”。公开边界是不公开原始输入值、真实摘要、客户标识或完整 hint。 证据 3 来自EvidenceEnvelope 源码，观察到iOS envelope 包含 schemaVersion、platform、sdkFamily、sdkVersion、generatedAtMillis、reportingMode、appKeyHash、installIdSha256、deviceContext、evidenceFamilies、evidenceEvents、attestationEvidence 和 diagnostics。，因此支撑“上报包应结构化且最小化，既能诊断也能控制隐私暴露。”。公开边界是不公开 raw installId、raw AppKey、Team 原文、bundleId 原文或完整 payload。 证据 4 来自EvidenceEnvelope 源码，观察到DeviceContext 使用 deviceModelHash、bundleIdHash、teamIdHash 等 hash 字段；EvidenceEvent 带 source 和 trust，默认 client_header/low。，因此支撑“iOS 设备证据需要区分来源和信任等级，不能把客户端事件直接当权威。”。公开边界是只公开字段类别，不输出真实 hash、模型、bundle 或 Team。 证据 5 来自Attestation collector 源码，观察到App Attest 与 DeviceCheck 采集的是 support status、request status、challenge required 等状态，缺 server challenge 时不请求 token/assertion。，因此支撑“support bundle 应说明能力和集成状态，而不是携带私有验证材料。”。公开边界是不公开 raw token、assertion、challenge、key id、Apple 私有材料。 证据 6 来自Hosted reporting client 源码，观察到公开托管上报使用 public_hosted 模式，错误被映射为 transport、authFailed、serverError 等类别。，因此支撑“support bundle 要把传输诊断和风险证据分开，便于客服与后端定位问题。”。公开边界是不公开真实 endpoint、请求体、响应体、header 值或客户配置。

工程路径：诊断快照最小化 负责DiagnosticSnapshot 只暴露初始化、host、appKeyHash、evidenceFamilies 和短 hint。；通用脱敏器 负责Redactor 统一 hash 与 hint 规则，避免不同模块各自拼接原始字段。；结构化 envelope 负责EvidenceEnvelope 用 schema、platform、families、events、attestation 和 diagnostics 表达事实。；错误类别拆分 负责public hosted 上报把 transport、auth、server error 分开，support bundle 不把网络失败写成风险。。这些动作需要和 release gate、verdict API、support bundle、feedback label、rollback action 配合，而不是停在单个移动端函数。

### 5. SDK 能跑不代表能发布：Android 设备证据包为什么要证明“没有带出秘密”？：从事实到工程闭环

这篇文章围绕“SDK 能跑不代表能发布：Android 设备证据包为什么要证明“没有带出秘密”？”展开，属于 守界 Android 的单一风险边界。它保留“事实、判断、动作、边界”的链路：事实来自脱敏资料，判断来自工程验收，动作落在发布门禁和客户后端，边界保护源码、密钥、设备、路径、客户信息和可复现攻击细节。

案例摘要：某 SDK 在本地 sample app 中可以跑通 BoxId 返回和诊断输出，团队准备直接打包给客户。发布审查发现，能跑通只是功能结果，公开交付还要证明 archive 没有混入 server、iOS、Web、homepage、private detector、generated local files、local.properties、SecretKey 或 provider 凭据。 守界 Android v0.4 的 release checklist 把这件事拆成 release readiness、public archive dry run、archive consumer smoke、publish workflow dry run、release candidate manifest、evidence pack schema、public commit scope 和 post-release consumption。任何一个 gate 失败，都不能靠人工说明继续。 这个案例说明，SDK 发布的安全性不只在 SDK 代码里，也在发布流程里。公开包一旦带出私有材料，后续再修复也很难收回。

证据解释：证据 1 来自Release Checklist，观察到本地 gate 要求 status 为 local-pass-with-external-blockers、failures 为 none、secret values printed 为 no，且 blockers 明确是外部材料。，因此支撑“公开发布可以承认外部 blocker，但不能用 dummy credential 或 synthetic success 伪装完成。”。公开边界是不公开报告路径、凭据、token、客户端点或真实 blocker 材料。 证据 2 来自Release Checklist，观察到public archive dry run 要求排除 iOS、Web、server、homepage、private detector、generated build、local.properties 等内容。，因此支撑“公开 Android SDK 包必须有清晰发布范围，不能从混合工作区误带私有目录。”。公开边界是只公开范围规则，不输出本地路径、文件列表原文或私有目录。 证据 3 来自Release Checklist，观察到archive consumer smoke 要求无 symlink、脚本语法通过、提取包仍排除私有根，并扫描 forbidden public-boundary material。，因此支撑“发布包不仅要生成，还要模拟消费者视角复核。”。公开边界是不公开 archive 位置、内部扫描命中原文或客户材料。 证据 4 来自Release Readiness 脚本，观察到脚本检查 README、release checklist、release notes、tag runbook、changelog、wrapper、matrix、Maven、archive、publish workflow、public commit scope 等 gate。，因此支撑“SDK 发布要由多个结构化 gate 组成，不能只看单次构建成功。”。公开边界是不公开完整 stdout、环境变量、凭据、工作目录或本地 artifact。 证据 5 来自Release Readiness 脚本，观察到脚本显式标注不启动付费设备、不打印 secrets，并把真实 provider、Central、ops、wrapper endpoint 等列为 external blockers。，因此支撑“外部依赖缺失时要透明记录 blocked，而不是在公开包里放测试凭据。”。公开边界是不公开 provider、Central、ops 或 endpoint 凭据。 证据 6 来自Release Notes Draft，观察到release notes 强调 SDK evidence-only，收集并上报设备/环境证据，返回 BoxId，最终业务决策留给客户后端。，因此支撑“公开 SDK 的定位必须和产品边界一致，不能把客户端包装成业务裁判。”。公开边界是不公开客户策略、risk engine、SecretKey 或私有服务端实现。

工程路径：public archive 边界 负责只包含公开 Android SDK、sample、docs、workflow 和必要 wrapper skeleton，排除混合仓库的非公开材料。；release evidence pack 负责索引各组件 summary、git index snapshots、SHA-256、byte counts 和 redaction scan 结果。；外部 blocker 透明化 负责真实 attestation provider、Maven Central、ops、backend endpoint 缺材料时写 external blocker。；consumer smoke 负责从消费者视角解包、查 symlink、跑脚本语法、扫 forbidden material，证明发布包可安全交付。。这些动作需要和 release gate、verdict API、support bundle、feedback label、rollback action 配合，而不是停在单个移动端函数。

## 统一发布检查
- 每篇文章正文不少于 10000 个中文字符。
- 每篇文章都有至少 5 条“事实依据与脱敏证据”。
- 每篇文章只聚焦单一产品、单一平台或单一场景。
