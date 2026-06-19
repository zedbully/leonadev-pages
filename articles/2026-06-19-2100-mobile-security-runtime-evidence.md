---
platform: github-pages
status: ready-for-public-pages
source_batch: 2026-06-19-2100-mobile-security-runtime-evidence
title: "2026-06-19 21:00 移动安全运行证据内容包"
permalink: /articles/2026-06-19-2100-mobile-security-runtime-evidence/
---

# 2026-06-19 21:00 移动安全运行证据内容包

本文是本轮公开技术内容索引，适合发布到 GitHub Pages 作为长期沉淀页。它覆盖 5 篇自有站正式文章，并为外部平台发布提供入口。正文只保留公开化工程事实、脱敏证据和验收方法。

## 本轮自有站文章入口

- 1. [别把三条 SHA 拼成一个通过：Android 签名模式门禁为什么必须 SameSHA 闭合？](https://dun.leonadev.com/article/android-signmode-samesha-release-gate-no-cross-sha-pass)
- 2. [函数加密不是把字节藏起来：iOS Mach-O materialization 为什么要先 fail-closed？](https://dun.leonadev.com/article/ios-macho-function-materialization-fail-closed-gate)
- 3. [Root、Magisk、VPN、多开别塞进一个标签：Android 设备证据图谱该怎么拆？](https://dun.leonadev.com/article/android-root-magisk-vpn-clone-mount-evidence-graph)
- 4. [没有 GMS 时最怕“假通过”：Android OEM attestation fallback 应该怎么验收？](https://dun.leonadev.com/article/android-non-gms-attestation-fallback-not-pass)
- 5. [iOS 设备指纹不要做成本地风控开关：App Attest、越狱证据和 BoxId 如何进同一 envelope？](https://dun.leonadev.com/article/ios-evidence-envelope-appattest-verifier-boundary)

## 技术摘要

### 1. 别把三条 SHA 拼成一个通过：Android 签名模式门禁为什么必须 SameSHA 闭合？

产品与平台：御盾 Android

核心判断：签名模式不是“能启动就算通过”，而是同一候选产物在 DEX、SO-ART、静态资源、运行时材料、业务策略和 QA 闭环上同时成立。

事实依据：

- 1. 动态风险 SameSHA readiness 矩阵：矩阵显示某一候选只具备 DEX 字符串和运行时 readback 通过，SO-ART 与静态资源口径缺失，因此不允许启动 bounded verifier。 支撑判断：发布结论必须要求同一候选产物闭合 DEX、SO-ART、静态资源和动态验证，不能把局部通过当作完整通过。
- 2. 动态风险 SameSHA readiness 矩阵：另一个候选具备 ART/JNI bootstrap 通过，但 SOVMP 和 oracle 仍为 blocked/no-promote，且静态资源没有可接受证据。 支撑判断：native 启动链路能跑通不代表加固材料、oracle 和资源保护已经达到 release 口径。
- 3. 动态风险 SameSHA readiness 矩阵：某候选的品牌和日志静态项通过，但资源项仍显示明文资源数量，动态结论被标记 not_evaluated。 支撑判断：资源静态残留会阻断动态验证，发布系统不能先跑动态再回头解释静态缺口。
- 4. SIGNMODE 发布门禁索引：SIGNMODE 检查项明确区分 debug 签名与 release 签名：debug 可用于模拟器/root smoke，但不能让真实材料被动态分析或注入路径拿到；release 不允许模拟器/root 进入敏感路径。 支撑判断：签名模式门禁必须把环境、签名类型、材料可见性和业务路径绑定起来。
- 5. SIGNMODE 发布门禁索引：多轮记录反复出现 blocked/no-promote，原因集中在同一 SHA 缺少 SO-ART、Static、Business、Dynamic 或 QA 闭合证据。 支撑判断：阻塞不是失败噪声，而是发布口径在保护跨层一致性；系统应保留阻塞原因而不是强行通过。
- 6. 架构合同审计：合同审计确认 classes2 与 alias 包装修复有静态证据，但同一候选没有证明 SO/ART one-hop retry patch 已集成到重建产物。 支撑判断：静态包装修复不能替代 runtime bridge 集成证明；发布应要求重建候选与运行时证据一致。

工程清单：

- SameSHA 证据绑定：把 DEX、SO-ART、静态资源、动态验证和业务策略都绑定到同一个候选产物，禁止用不同构建的局部通过拼接发布结论。
- debug/release 策略分流：debug 签名只服务本地 smoke 与诊断，不进入真实敏感材料路径；release 签名必须在非调试环境、真实分发口径和服务端合法版本集合下验证。
- 运行时材料可见性：provider readback、短窗口材料、native bridge 和保护载荷必须有同一候选的 evidence，不能用静态修复报告代替运行时证明。
- fail-closed 门禁：缺少候选、签名清单、运行时证据或 QA 证据时输出 blocked/no-promote，避免人为把空白证据解释成通过。

公开边界：不公开源码、私有路径、密钥、测试设备、内部账号、客户信息、函数名、偏移、原始 token/assertion、完整攻击复现链路或可直接绕过检测的实现细节。

### 2. 函数加密不是把字节藏起来：iOS Mach-O materialization 为什么要先 fail-closed？

产品与平台：御盾 iOS

核心判断：iOS 函数级保护的关键不是“生成密文 blob”，而是 selected target、函数边界、指令分类、protected table、dispatcher、签名和真机证据全部可绑定。

事实依据：

- 1. iOS 函数级加密方案：selected target 只允许来自显式 target profile，覆盖主 App Mach-O、embedded framework、extension、App Clip 或 Watch companion 中被配置命中的函数。 支撑判断：iOS 加固不能默认全量自动保护，必须先定义明确目标和边界。
- 2. iOS 函数级加密方案：进入加密前必须确认函数起止地址稳定、不跨 data-in-code、不跨 section、满足 arm64/arm64e 对齐，并且 unwind 不被破坏。 支撑判断：函数边界不稳定时应跳过或阻断商业 ready，不能为了覆盖率强行 patch。
- 3. iOS 函数级加密方案：protected table 的函数记录要求 target id hash、arch tag、policy tag、encrypted body ref、trampoline binding、PAC/BTI tag、unwind tag 和 integrity tag。 支撑判断：保护记录应支持 dispatcher 和验收，不应泄露可复用明文映射。
- 4. Static Patch/Materialization 闸口：闸口默认 fail-closed，缺少 static patch writer、dispatcher materialization、runtime 注入、重签或真机 smoke 时商业 ready 必须为 false。 支撑判断：阻断状态是发布安全机制，不是功能失败；它避免把设计草案误标为商业完成。
- 5. Static Patch/Materialization 闸口：允许输出 gate 状态、selected target 计数、输入 gate 状态和布尔脱敏声明，禁止输出明文函数 range、RVA、file offset、opcode、instruction bytes、symbol names 和 keys。 支撑判断：报告应支持审计，但不能成为逆向资料。
- 6. Release Gate 证据绑定校验：Release Gate 要求 protected IPA、runner/session、evidence bundle、producedAt 和各 gate 专属 evidence ref 彼此一致；不一致或缺失时标记 UNBOUND、PARTIAL 或 INCONSISTENT。 支撑判断：即使合成 gate PASS，缺少真实 runner、签名隔离、真机验收和逆向验收时仍不能 commercial ready。

工程清单：

- selected target inventory：用显式 profile、Mach-O inventory、函数起点、符号或 mapping sidecar 选择目标，禁止没有边界的全量自动 patch。
- 边界与指令分类：确认函数边界、data-in-code、section、alignment、branch、literal、PAC/BTI 和 unwind 兼容，遇到未知项阻断。
- protected table 脱敏记录：只写 hash、tag、encrypted body ref、binding tag 和 integrity tag，不写明文入口、RVA、symbol、函数体或 key。
- Release Gate 绑定：把 protected IPA、runner/session、evidence bundle、签名、真机、静态逆向、动态逆向、性能和 CI 引用绑定成同一证据链。

公开边界：不公开源码、私有路径、密钥、测试设备、内部账号、客户信息、函数名、偏移、原始 token/assertion、完整攻击复现链路或可直接绕过检测的实现细节。

### 3. Root、Magisk、VPN、多开别塞进一个标签：Android 设备证据图谱该怎么拆？

产品与平台：守界 Android

核心判断：Root、Magisk、VPN、多开、mount、ROM、模拟器和 hook 是不同证据族，只有进入服务端图谱后才有业务解释价值。

事实依据：

- 1. Root/Magisk 风险门禁记录：相关门禁长期保持 blocked 状态，原因不是“不检测 root”，而是缺少同一候选、同一运行口径和服务端解释链路下的完整动态闭合。 支撑判断：Root/Magisk 证据应进入图谱和服务端 verdict，不能用单个本地命中替代发布通过。
- 2. VPN/多开/mount 风险门禁记录：VPN、clone、mount 等风险项被单独列为门禁，而不是与 root、hook 或模拟器混成一个环境标签。 支撑判断：不同环境事实应拆成独立 evidence_family，便于误报复盘和客户策略分级。
- 3. Hook/调试/注入禁用边界门禁：门禁记录多次强调禁止使用 hook/debug/injection 路径收集真实材料，相关 fixture 仅能作为 debug-safe support。 支撑判断：设备证据系统不能为了证明检测能力而引入新的材料泄露风险。
- 4. ROM and Bootloader Matrix：矩阵要求把 ROM/bootloader facts 与 emulator、root、hook、debug evidence 分开，custom ROM posture 可以映射到不同客户策略。 支撑判断：同一设备姿态在不同业务下可能有不同动作，SDK 不应本地做封禁。
- 5. Evidence and Privacy Boundary：Android SDK 只允许输出 opaque BoxId、诊断状态、脱敏 support bundle 字段和 evidence 上传元数据，不允许输出 allow/reject/block/isFraud。 支撑判断：最终业务动作属于客户后端，SDK 输出只能是证据。
- 6. Device Identity and Evidence Protocol：nativeFindingIds、nativeFactTags 和 nativeHighestSeverity 是 evidence；riskTags 只由 server verdict path 产生。 支撑判断：native 层发现不能直接成为客户端风险判定，必须经过服务端策略分类。

工程清单：

- 证据族拆分：Root、Magisk、VPN、多开、mount、ROM、bootloader、emulator、hook 和 debug 分别记录，不在客户端压成单一风险标签。
- BoxId 与图谱关联：BoxId 只做关联入口，native facts、environment evidence、server verdict、feedback label 和业务动作共同形成图谱。
- 隐私最小化：support bundle 只保留 hash、hint、count、family、status 和 provenance，不输出完整设备标识、完整包列表或原始凭据。
- 策略分级：登录、支付、提现、游戏结算、企业数据导出等业务动作使用不同阈值、挑战和复核流程。

公开边界：不公开源码、私有路径、密钥、测试设备、内部账号、客户信息、函数名、偏移、原始 token/assertion、完整攻击复现链路或可直接绕过检测的实现细节。

### 4. 没有 GMS 时最怕“假通过”：Android OEM attestation fallback 应该怎么验收？

产品与平台：守界 Android

核心判断：非 GMS 环境下的 attestation 不是“没有就放过”，而是 provider、binding-only、fallback traffic、verifier 和服务端 verdict 的分层验收。

事实依据：

- 1. Mainland non-GMS Attestation Release Gate：门禁适用于缺少 Google Play 服务、依赖 OEM attestation 或临时允许 no_attestation fallback 的大陆渠道。 支撑判断：国内 Android 设备证明必须先明确 provider 条件，不能用单一 GMS 路线覆盖全部渠道。
- 2. Mainland non-GMS Attestation Release Gate：最小闭环包括静态 build gate、样例大陆构建安装、emulator attestation summary E2E 和 fallback path validation。 支撑判断：debug 与样例路径只能验证流程，不能直接成为商业 ready。
- 3. Mainland non-GMS Attestation Release Gate：真实 staging gate 要求 private Android provider、private backend verifier、trusted provider allowlist、oem_attested 握手和下游 decisioning 反映 OEM posture。 支撑判断：OEM attestation 的生产通过依赖私有服务端材料和可信 provider，不是 SDK 单边能力。
- 4. Mainland non-GMS Attestation Release Gate：停止条件包括 OEM verifier 缺失、allowlist 为空、只有 fake path 通过、failed OEM 被静默当作 no_attestation、fallback 流量不可量化。 支撑判断：fallback 是可解释状态，不是通过状态；运营必须能看到 verified 与 fallback 的比例。
- 5. Device Identity and Evidence Protocol：installId、resolvedDeviceId 和 fingerprintHash 是身份层与关联 hint，canonical identity 由服务端拥有，客户端 canonical 只能作为 claim。 支撑判断：attestation 不应替代服务端 identity authority 和历史图谱。
- 6. 设备指纹市场分析：国内客户通常期待设备 ID、风险环境、风控平台和运营闭环的完整形态；单点检测不构成壁垒。 支撑判断：非 GMS attestation 只是设备智能的一部分，必须与 graph、feedback 和解释面结合。

工程清单：

- provider 状态模型：把 GMS、OEM、binding-only、no_attestation、transport failure 和 verifier missing 拆成不同状态。
- 真实 staging gate：生产验收必须包含私有 provider、私有 verifier、可信 allowlist、oem_attested 握手和客户后端 decisioning。
- fallback 量化：运营看板区分 verified traffic、fallback traffic、failed verifier、allowlist missing 和 network diagnostics。
- identity authority：canonical identity 和 riskTags 由服务端产生，客户端提供 installId/hash/fingerprint 只能作为低信任证据。

公开边界：不公开源码、私有路径、密钥、测试设备、内部账号、客户信息、函数名、偏移、原始 token/assertion、完整攻击复现链路或可直接绕过检测的实现细节。

### 5. iOS 设备指纹不要做成本地风控开关：App Attest、越狱证据和 BoxId 如何进同一 envelope？

产品与平台：守界 iOS

核心判断：iOS public SDK 应采集 evidence envelope 和 BoxId，而不是暴露 isJailbroken、isFridaDetected、shouldBlock 或 riskLevel 这类本地判定 API。

事实依据：

- 1. iOS Evidence Extension Plan：iOS v0.4 最小闭环是 App 调用 sense，服务端返回 BoxId，客户后端再查询 verdict 和 evidence report，最终业务决策留给客户后端。 支撑判断：iOS 设备指纹不能在客户端直接做 allow/reject/block。
- 2. iOS Evidence Extension Plan：公开 API 允许 diagnosticSnapshot、supportBundle 和 lastServerEvidence，但禁止 isJailbroken、isFridaDetected、isTampered、shouldBlock、riskLevel。 支撑判断：本地布尔判定容易被误用为最终策略，也会成为固定 patch target。
- 3. iOS Evidence Extension Plan：Evidence taxonomy 包含 identity、runtime、jailbreak、Frida/injection、tamper、attestation 和 transport diagnostic，字段默认是 evidence/telemetry。 支撑判断：不同 evidence family 必须分开上报，由服务端 policy 或 verifier 产生权威解释。
- 4. iOS Evidence Extension Plan：App Attest/DeviceCheck 只采集 token request status、key generation status、assertion status 和 challenge binding status；Apple verifier、Team allowlist 和 verification result 属于 server/private。 支撑判断：App Attest 是服务端验证链的一环，不是客户端本地可信开关。
- 5. iOS Evidence Extension Plan：Cross-platform envelope v1 规定 iOS 与 Android 共用 BoxId、canonical、provenance、riskTagsBySource、feedback label 和 Device Evidence Graph。 支撑判断：iOS 证据不应形成孤岛，应进入统一服务端图谱和反馈闭环。
- 6. 隐私与市场分析归纳：公开设备智能产品已经从设备 ID 扩展到环境、行为、网络、图谱、反馈和运营平台，单点客户端探针不构成壁垒。 支撑判断：守界 iOS 的价值应落在 evidence graph、feedback 和解释面，而不是几个本地检测函数。

工程清单：

- public SDK evidence-only：iOS SDK 提供 sense、diagnosticSnapshot、supportBundle 和 lastServerEvidence，不提供本地 block/risk 判定 API。
- cross-platform envelope：iOS 与 Android 共用 BoxId、provenance、riskTagsBySource、feedback label 和 Device Evidence Graph，字段只携带 hash、hint、summary 和 status。
- App Attest verifier 边界：challenge、replay、key registration、assertion verification、Team allowlist、verifier credential 和最终动作都留在服务端。
- 反馈闭环：客户后端把人工复核、误报、确认攻击和业务动作标签写回图谱，持续改进解释面。

公开边界：不公开源码、私有路径、密钥、测试设备、内部账号、客户信息、函数名、偏移、原始 token/assertion、完整攻击复现链路或可直接绕过检测的实现细节。

## 深度技术笔记

### 1. 别把三条 SHA 拼成一个通过：Android 签名模式门禁为什么必须 SameSHA 闭合？：从事实到工程闭环

这篇文章属于 御盾 Android 的单一风险边界。它保留“事实、判断、动作、边界”的链路：事实来自脱敏资料，判断来自工程验收，动作落在发布门禁和客户后端，边界保护源码、密钥、设备、路径、客户信息和可复现攻击细节。

案例摘要：一个 Android 加固产物在表面上看已经“多项通过”：DEX readback 有通过记录，ART/JNI bootstrap 也有通过记录，资源扫描里部分静态项看起来干净。发布负责人如果只看结果列，很容易把这些绿点合并成一个 release pass。脱敏门禁记录的价值就在于它阻止了这种合并：这些证据并不属于同一个候选产物，也不覆盖同一条运行链路。 攻击者最喜欢的正是这种跨产物拼接。只要某个渠道包、调试包或中间包被误认为 release 产物，后续的服务端合法版本集合、接口签名策略和运行时材料策略都会被污染。客户上线后看到的不是“某个检测点偶尔失败”，而是发布体系无法解释为什么一个包被信任。 御盾 Android 的工程实践应把 SIGNMODE 作为发布 contract，而不是一个埋在客户端的校验函数。每个候选必须有同一 SHA 的 DEX、SO-ART、静态资源、运行时材料、业务策略和 QA 证据，缺一项就写 blocked/no-promote。这个口径会让交付慢一点，但能避免把实验室片段包装成商业 ready。

证据解释：证据 1 来自动态风险 SameSHA readiness 矩阵，观察到矩阵显示某一候选只具备 DEX 字符串和运行时 readback 通过，SO-ART 与静态资源口径缺失，因此不允许启动 bounded verifier。，因此支撑“发布结论必须要求同一候选产物闭合 DEX、SO-ART、静态资源和动态验证，不能把局部通过当作完整通过。”。公开时必须遵守边界：不公开候选散列、证据编号、运行命令、日志路径、设备信息和原始数据库字段。 证据 2 来自动态风险 SameSHA readiness 矩阵，观察到另一个候选具备 ART/JNI bootstrap 通过，但 SOVMP 和 oracle 仍为 blocked/no-promote，且静态资源没有可接受证据。，因此支撑“native 启动链路能跑通不代表加固材料、oracle 和资源保护已经达到 release 口径。”。公开时必须遵守边界：只公开状态关系，不输出 SO/ART provider 名称、调用序列、fixture 或 runner 输出。 证据 3 来自动态风险 SameSHA readiness 矩阵，观察到某候选的品牌和日志静态项通过，但资源项仍显示明文资源数量，动态结论被标记 not_evaluated。，因此支撑“资源静态残留会阻断动态验证，发布系统不能先跑动态再回头解释静态缺口。”。公开时必须遵守边界：不公开明文资源名、路径、数量来源、样本文件或扫描脚本。 证据 4 来自SIGNMODE 发布门禁索引，观察到SIGNMODE 检查项明确区分 debug 签名与 release 签名：debug 可用于模拟器/root smoke，但不能让真实材料被动态分析或注入路径拿到；release 不允许模拟器/root 进入敏感路径。，因此支撑“签名模式门禁必须把环境、签名类型、材料可见性和业务路径绑定起来。”。公开时必须遵守边界：不公开 gate 原始索引、测试设备、签名材料、策略阈值和内部 evidence id。 证据 5 来自SIGNMODE 发布门禁索引，观察到多轮记录反复出现 blocked/no-promote，原因集中在同一 SHA 缺少 SO-ART、Static、Business、Dynamic 或 QA 闭合证据。，因此支撑“阻塞不是失败噪声，而是发布口径在保护跨层一致性；系统应保留阻塞原因而不是强行通过。”。公开时必须遵守边界：只公开阻塞类别，不输出轮次、候选散列、任务编号、命令和详细日志。 证据 6 来自架构合同审计，观察到合同审计确认 classes2 与 alias 包装修复有静态证据，但同一候选没有证明 SO/ART one-hop retry patch 已集成到重建产物。，因此支撑“静态包装修复不能替代 runtime bridge 集成证明；发布应要求重建候选与运行时证据一致。”。公开时必须遵守边界：不公开类名、源码路径、行号、构建清单散列、包内结构和具体 bridge 调用。

工程路径：SameSHA 证据绑定 负责把 DEX、SO-ART、静态资源、动态验证和业务策略都绑定到同一个候选产物，禁止用不同构建的局部通过拼接发布结论。；debug/release 策略分流 负责debug 签名只服务本地 smoke 与诊断，不进入真实敏感材料路径；release 签名必须在非调试环境、真实分发口径和服务端合法版本集合下验证。；运行时材料可见性 负责provider readback、短窗口材料、native bridge 和保护载荷必须有同一候选的 evidence，不能用静态修复报告代替运行时证明。；fail-closed 门禁 负责缺少候选、签名清单、运行时证据或 QA 证据时输出 blocked/no-promote，避免人为把空白证据解释成通过。。这些动作需要和 release gate、verdict API、support bundle、feedback label、rollback action 配合，而不是停在单个移动端函数。

复盘指标建议：别把三条 SHA 拼成一个通过：Android 签名模式门禁为什么必须 SameSHA 闭合？ 应持续跟踪命中率、版本分布、渠道分布、失败原因、证据新鲜度、服务端解释结果、人工复核结果、误报申诉率和策略回滚次数。

### 2. 函数加密不是把字节藏起来：iOS Mach-O materialization 为什么要先 fail-closed？：从事实到工程闭环

这篇文章属于 御盾 iOS 的单一风险边界。它保留“事实、判断、动作、边界”的链路：事实来自脱敏资料，判断来自工程验收，动作落在发布门禁和客户后端，边界保护源码、密钥、设备、路径、客户信息和可复现攻击细节。

案例摘要：某 iOS 加固路线已经能生成 selected target 的密文 blob，并把保护记录写入受保护 section。团队容易把“密文存在”当成函数级保护完成。但脱敏设计文档强调，密文只是中间结果；函数边界、PAC/BTI、unwind、dispatcher、入口改写、重签、真机安装和受保护路径命中都没有闭合时，商业 ready 必须保持 false。 iOS 的难点不只是 Mach-O 文件格式，而是平台约束叠加。一个函数如果边界不稳、覆盖 data-in-code、存在未建模的 PC-relative 依赖、破坏 unwind 或 arm64e PAC/BTI，强行 patch 可能让 App 在真实设备上崩溃，也可能让保护记录泄露足够信息给静态逆向。 御盾 iOS 应把 materialization gate 当成质量控制器：它允许明确告诉客户“哪些条件还没有商业闭合”，而不是提前宣称完成。对安全产品来说，能准确 fail-closed，比生成一个不可验证的保护包更可靠。

证据解释：证据 1 来自iOS 函数级加密方案，观察到selected target 只允许来自显式 target profile，覆盖主 App Mach-O、embedded framework、extension、App Clip 或 Watch companion 中被配置命中的函数。，因此支撑“iOS 加固不能默认全量自动保护，必须先定义明确目标和边界。”。公开时必须遵守边界：不公开目标函数名、RVA、symbol、profile 原文、二进制路径或客户包结构。 证据 2 来自iOS 函数级加密方案，观察到进入加密前必须确认函数起止地址稳定、不跨 data-in-code、不跨 section、满足 arm64/arm64e 对齐，并且 unwind 不被破坏。，因此支撑“函数边界不稳定时应跳过或阻断商业 ready，不能为了覆盖率强行 patch。”。公开时必须遵守边界：不公开边界扫描结果、地址、opcode、指令字节或调试符号。 证据 3 来自iOS 函数级加密方案，观察到protected table 的函数记录要求 target id hash、arch tag、policy tag、encrypted body ref、trampoline binding、PAC/BTI tag、unwind tag 和 integrity tag。，因此支撑“保护记录应支持 dispatcher 和验收，不应泄露可复用明文映射。”。公开时必须遵守边界：不公开 key material、明文函数体、明文 RVA、真实入口或映射表。 证据 4 来自Static Patch/Materialization 闸口，观察到闸口默认 fail-closed，缺少 static patch writer、dispatcher materialization、runtime 注入、重签或真机 smoke 时商业 ready 必须为 false。，因此支撑“阻断状态是发布安全机制，不是功能失败；它避免把设计草案误标为商业完成。”。公开时必须遵守边界：只公开 blocker 枚举，不公开 patch 位置、trampoline 位置、真实保护映射或中间产物。 证据 5 来自Static Patch/Materialization 闸口，观察到允许输出 gate 状态、selected target 计数、输入 gate 状态和布尔脱敏声明，禁止输出明文函数 range、RVA、file offset、opcode、instruction bytes、symbol names 和 keys。，因此支撑“报告应支持审计，但不能成为逆向资料。”。公开时必须遵守边界：不公开报告原文、内部 artifact、二进制样本或材料化布局。 证据 6 来自Release Gate 证据绑定校验，观察到Release Gate 要求 protected IPA、runner/session、evidence bundle、producedAt 和各 gate 专属 evidence ref 彼此一致；不一致或缺失时标记 UNBOUND、PARTIAL 或 INCONSISTENT。，因此支撑“即使合成 gate PASS，缺少真实 runner、签名隔离、真机验收和逆向验收时仍不能 commercial ready。”。公开时必须遵守边界：不公开 runner 原值、设备唯一标识、日志、符号、地址、脚本、私钥或证书材料。

工程路径：selected target inventory 负责用显式 profile、Mach-O inventory、函数起点、符号或 mapping sidecar 选择目标，禁止没有边界的全量自动 patch。；边界与指令分类 负责确认函数边界、data-in-code、section、alignment、branch、literal、PAC/BTI 和 unwind 兼容，遇到未知项阻断。；protected table 脱敏记录 负责只写 hash、tag、encrypted body ref、binding tag 和 integrity tag，不写明文入口、RVA、symbol、函数体或 key。；Release Gate 绑定 负责把 protected IPA、runner/session、evidence bundle、签名、真机、静态逆向、动态逆向、性能和 CI 引用绑定成同一证据链。。这些动作需要和 release gate、verdict API、support bundle、feedback label、rollback action 配合，而不是停在单个移动端函数。

复盘指标建议：函数加密不是把字节藏起来：iOS Mach-O materialization 为什么要先 fail-closed？ 应持续跟踪命中率、版本分布、渠道分布、失败原因、证据新鲜度、服务端解释结果、人工复核结果、误报申诉率和策略回滚次数。

### 3. Root、Magisk、VPN、多开别塞进一个标签：Android 设备证据图谱该怎么拆？：从事实到工程闭环

这篇文章属于 守界 Android 的单一风险边界。它保留“事实、判断、动作、边界”的链路：事实来自脱敏资料，判断来自工程验收，动作落在发布门禁和客户后端，边界保护源码、密钥、设备、路径、客户信息和可复现攻击细节。

案例摘要：一个游戏或金融 App 想要识别 Root、Magisk、VPN、多开和 mount 异常，最容易犯的错误是把所有事实压成一个“风险设备”标签。上线后运营会发现两个问题：真实攻击当然会命中，但开发者设备、企业设备、海外网络、隐私 ROM、维修设备和部分代理网络也会命中。没有证据族，就无法解释哪些命中应该观察，哪些应该挑战，哪些才应该拒绝。 守界 Android 的做法是把证据拆开。Root/Magisk 说明权限与隐藏模块风险，VPN 说明网络路径异常，多开说明运行容器或账号矩阵风险，mount 说明文件系统观察，ROM/bootloader 说明系统来源，hook/debug 说明运行期干预。BoxId 只是关联入口，server verdict 才负责把这些证据放进当前业务动作。 这样做还能保护隐私。公开 support bundle 里不需要完整设备标识、完整包列表、完整 build fingerprint 或真实网络配置；工程团队只需要 hash、hint、count、family、status 和 provenance，就能完成排障、复核和反馈回写。

证据解释：证据 1 来自Root/Magisk 风险门禁记录，观察到相关门禁长期保持 blocked 状态，原因不是“不检测 root”，而是缺少同一候选、同一运行口径和服务端解释链路下的完整动态闭合。，因此支撑“Root/Magisk 证据应进入图谱和服务端 verdict，不能用单个本地命中替代发布通过。”。公开时必须遵守边界：不公开检测规则、包名、设备、日志、候选散列、任务编号或绕过样本。 证据 2 来自VPN/多开/mount 风险门禁记录，观察到VPN、clone、mount 等风险项被单独列为门禁，而不是与 root、hook 或模拟器混成一个环境标签。，因此支撑“不同环境事实应拆成独立 evidence_family，便于误报复盘和客户策略分级。”。公开时必须遵守边界：只公开证据族，不输出规则细节、应用列表、mount 信息、网络配置或测试设备。 证据 3 来自Hook/调试/注入禁用边界门禁，观察到门禁记录多次强调禁止使用 hook/debug/injection 路径收集真实材料，相关 fixture 仅能作为 debug-safe support。，因此支撑“设备证据系统不能为了证明检测能力而引入新的材料泄露风险。”。公开时必须遵守边界：不公开 hook 工具、脚本、目标函数、运行输出或注入路径。 证据 4 来自ROM and Bootloader Matrix，观察到矩阵要求把 ROM/bootloader facts 与 emulator、root、hook、debug evidence 分开，custom ROM posture 可以映射到不同客户策略。，因此支撑“同一设备姿态在不同业务下可能有不同动作，SDK 不应本地做封禁。”。公开时必须遵守边界：不公开完整 build fingerprint、ADB serial、Android ID、bootloader version 或 root manager 原始包名。 证据 5 来自Evidence and Privacy Boundary，观察到Android SDK 只允许输出 opaque BoxId、诊断状态、脱敏 support bundle 字段和 evidence 上传元数据，不允许输出 allow/reject/block/isFraud。，因此支撑“最终业务动作属于客户后端，SDK 输出只能是证据。”。公开时必须遵守边界：不公开 SecretKey、provider credential、完整 BoxId、原始 token 或业务策略。 证据 6 来自Device Identity and Evidence Protocol，观察到nativeFindingIds、nativeFactTags 和 nativeHighestSeverity 是 evidence；riskTags 只由 server verdict path 产生。，因此支撑“native 层发现不能直接成为客户端风险判定，必须经过服务端策略分类。”。公开时必须遵守边界：不公开 native payload、完整 finding 映射、endpoint、签名 header 或原始身份材料。

工程路径：证据族拆分 负责Root、Magisk、VPN、多开、mount、ROM、bootloader、emulator、hook 和 debug 分别记录，不在客户端压成单一风险标签。；BoxId 与图谱关联 负责BoxId 只做关联入口，native facts、environment evidence、server verdict、feedback label 和业务动作共同形成图谱。；隐私最小化 负责support bundle 只保留 hash、hint、count、family、status 和 provenance，不输出完整设备标识、完整包列表或原始凭据。；策略分级 负责登录、支付、提现、游戏结算、企业数据导出等业务动作使用不同阈值、挑战和复核流程。。这些动作需要和 release gate、verdict API、support bundle、feedback label、rollback action 配合，而不是停在单个移动端函数。

复盘指标建议：Root、Magisk、VPN、多开别塞进一个标签：Android 设备证据图谱该怎么拆？ 应持续跟踪命中率、版本分布、渠道分布、失败原因、证据新鲜度、服务端解释结果、人工复核结果、误报申诉率和策略回滚次数。

### 4. 没有 GMS 时最怕“假通过”：Android OEM attestation fallback 应该怎么验收？：从事实到工程闭环

这篇文章属于 守界 Android 的非 GMS 设备证明风险边界。它保留“provider 状态、fallback 量化、服务端 verifier、运营复盘”的链路：事实来自脱敏资料，判断来自工程验收，动作落在发布门禁和客户后端，边界保护源码、密钥、设备、路径、客户信息和可复现攻击细节。

案例摘要：一个面向国内渠道的 Android SDK 如果把 attestation 设计成“有 Google Play 服务就走 Play，没有就默认通过”，上线后会迅速失真。真实设备、厂商系统、企业设备、云手机、模拟器、多开和自定义 ROM 的 provider 条件完全不同，fallback 流量如果不可量化，运营看板会把缺失证明误认为低风险。 守界 Android 的验收应该把状态拆开：gms_attested、oem_attested、binding_without_attestation、no_attestation、transport_error、verifier_missing、allowlist_missing。每个状态有 freshness、source、failure_reason 和 server verdict。客户后端根据业务动作决定观察、挑战、限速或拒绝，而不是由 SDK 本地给一个过度简化的通过。 生产 ready 的关键不在 demo 是否能跑，而在 private OEM bridge、private verifier、trusted provider allowlist 和下游 decisioning 是否真实存在。没有这些材料时，正确结论是 blocked_external_input 或 fallback_observed，不是 ready。

证据解释：证据 1 来自Mainland non-GMS Attestation Release Gate，观察到门禁适用于缺少 Google Play 服务、依赖 OEM attestation 或临时允许 no_attestation fallback 的大陆渠道。，因此支撑“国内 Android 设备证明必须先明确 provider 条件，不能用单一 GMS 路线覆盖全部渠道。”。公开时必须遵守边界：不公开 AppKey、脚本、环境变量、设备、endpoint 或生产 provider 配置。 证据 2 来自Mainland non-GMS Attestation Release Gate，观察到最小闭环包括静态 build gate、样例大陆构建安装、emulator attestation summary E2E 和 fallback path validation。，因此支撑“debug 与样例路径只能验证流程，不能直接成为商业 ready。”。公开时必须遵守边界：只公开阶段要求，不输出命令参数、token、样例日志或本地记录路径。 证据 3 来自Mainland non-GMS Attestation Release Gate，观察到真实 staging gate 要求 private Android provider、private backend verifier、trusted provider allowlist、oem_attested 握手和下游 decisioning 反映 OEM posture。，因此支撑“OEM attestation 的生产通过依赖私有服务端材料和可信 provider，不是 SDK 单边能力。”。公开时必须遵守边界：不公开 verifier、allowlist、provider secret、租户策略或生产验收报告。 证据 4 来自Mainland non-GMS Attestation Release Gate，观察到停止条件包括 OEM verifier 缺失、allowlist 为空、只有 fake path 通过、failed OEM 被静默当作 no_attestation、fallback 流量不可量化。，因此支撑“fallback 是可解释状态，不是通过状态；运营必须能看到 verified 与 fallback 的比例。”。公开时必须遵守边界：不公开真实渠道流量、客户看板、失败样本或运营指标原始值。 证据 5 来自Device Identity and Evidence Protocol，观察到installId、resolvedDeviceId 和 fingerprintHash 是身份层与关联 hint，canonical identity 由服务端拥有，客户端 canonical 只能作为 claim。，因此支撑“attestation 不应替代服务端 identity authority 和历史图谱。”。公开时必须遵守边界：不公开 raw installId、Android ID、完整 fingerprintHash、签名 header 或完整 BoxId。 证据 6 来自设备指纹市场分析，观察到国内客户通常期待设备 ID、风险环境、风控平台和运营闭环的完整形态；单点检测不构成壁垒。，因此支撑“非 GMS attestation 只是设备智能的一部分，必须与 graph、feedback 和解释面结合。”。公开时必须遵守边界：仅引用公开市场归纳，不复制竞品私有实现或客户数据。

工程路径：provider 状态模型 负责把 GMS、OEM、binding-only、no_attestation、transport failure 和 verifier missing 拆成不同状态。；真实 staging gate 负责生产验收必须包含私有 provider、私有 verifier、可信 allowlist、oem_attested 握手和客户后端 decisioning。；fallback 量化 负责运营看板区分 verified traffic、fallback traffic、failed verifier、allowlist missing 和 network diagnostics。；identity authority 负责canonical identity 和 riskTags 由服务端产生，客户端提供 installId/hash/fingerprint 只能作为低信任证据。。这些动作需要和 release gate、verdict API、support bundle、feedback label、rollback action 配合，而不是停在单个移动端函数。

复盘指标建议：没有 GMS 时最怕“假通过”：Android OEM attestation fallback 应该怎么验收？ 应持续跟踪命中率、版本分布、渠道分布、失败原因、证据新鲜度、服务端解释结果、人工复核结果、误报申诉率和策略回滚次数。

### 5. iOS 设备指纹不要做成本地风控开关：App Attest、越狱证据和 BoxId 如何进同一 envelope？：从事实到工程闭环

这篇文章属于 守界 iOS 的单一风险边界。它保留“事实、判断、动作、边界”的链路：事实来自脱敏资料，判断来自工程验收，动作落在发布门禁和客户后端，边界保护源码、密钥、设备、路径、客户信息和可复现攻击细节。

案例摘要：一个 iOS SDK 团队想快速补齐设备指纹能力，最直观的方案是暴露 isJailbroken、isFridaDetected、riskLevel 和 shouldBlock。这个方案看起来接入简单，却会把业务判定放在最容易被 patch 的客户端，也会让误报难以解释。用户被拒绝时，后台只能看到一个布尔值，看不到是越狱路径、Frida 线索、tamper 摘要、transport failure 还是 App Attest challenge 缺失。 守界 iOS 的 envelope 设计把每个事实放回证据族：identity 只上传 hash/hint，runtime 只上传状态摘要，jailbreak 和 Frida 只上传低价值公开探针的 evidence，tamper 只上传签名和资源摘要，attestation 只上传 status，transport 只上传诊断。客户后端拿到 BoxId 后再查询 verdict 和 evidence report。 App Attest 是其中最容易被误解的一环。token 和 assertion 的原始材料、challenge、replay、Team allowlist、private verifier 和最终业务动作必须在服务端。public SDK 可以说明“支持、未请求、缺少服务端 challenge、请求失败、transport 异常”，但不能自己宣布设备可信。

证据解释：证据 1 来自iOS Evidence Extension Plan，观察到iOS v0.4 最小闭环是 App 调用 sense，服务端返回 BoxId，客户后端再查询 verdict 和 evidence report，最终业务决策留给客户后端。，因此支撑“iOS 设备指纹不能在客户端直接做 allow/reject/block。”。公开时必须遵守边界：不公开真实 endpoint、AppKey、SecretKey、完整 BoxId 或客户策略。 证据 2 来自iOS Evidence Extension Plan，观察到公开 API 允许 diagnosticSnapshot、supportBundle 和 lastServerEvidence，但禁止 isJailbroken、isFridaDetected、isTampered、shouldBlock、riskLevel。，因此支撑“本地布尔判定容易被误用为最终策略，也会成为固定 patch target。”。公开时必须遵守边界：不公开私有 detector、完整路径库、权重、签名、绕过细节或测试设备。 证据 3 来自iOS Evidence Extension Plan，观察到Evidence taxonomy 包含 identity、runtime、jailbreak、Frida/injection、tamper、attestation 和 transport diagnostic，字段默认是 evidence/telemetry。，因此支撑“不同 evidence family 必须分开上报，由服务端 policy 或 verifier 产生权威解释。”。公开时必须遵守边界：不公开 raw IDFV、raw keychain id、完整证书、完整签名材料或完整 device model。 证据 4 来自iOS Evidence Extension Plan，观察到App Attest/DeviceCheck 只采集 token request status、key generation status、assertion status 和 challenge binding status；Apple verifier、Team allowlist 和 verification result 属于 server/private。，因此支撑“App Attest 是服务端验证链的一环，不是客户端本地可信开关。”。公开时必须遵守边界：不公开 raw token、raw assertion、challenge、key id、Team allowlist 或 verifier response。 证据 5 来自iOS Evidence Extension Plan，观察到Cross-platform envelope v1 规定 iOS 与 Android 共用 BoxId、canonical、provenance、riskTagsBySource、feedback label 和 Device Evidence Graph。，因此支撑“iOS 证据不应形成孤岛，应进入统一服务端图谱和反馈闭环。”。公开时必须遵守边界：不公开完整 envelope 样本中的真实身份字段、客户数据、生产 endpoint 或凭据。 证据 6 来自隐私与市场分析归纳，观察到公开设备智能产品已经从设备 ID 扩展到环境、行为、网络、图谱、反馈和运营平台，单点客户端探针不构成壁垒。，因此支撑“守界 iOS 的价值应落在 evidence graph、feedback 和解释面，而不是几个本地检测函数。”。公开时必须遵守边界：只公开公开资料归纳和产品边界，不复制竞品私有实现或客户样本。

工程路径：public SDK evidence-only 负责iOS SDK 提供 sense、diagnosticSnapshot、supportBundle 和 lastServerEvidence，不提供本地 block/risk 判定 API。；cross-platform envelope 负责iOS 与 Android 共用 BoxId、provenance、riskTagsBySource、feedback label 和 Device Evidence Graph，字段只携带 hash、hint、summary 和 status。；App Attest verifier 边界 负责challenge、replay、key registration、assertion verification、Team allowlist、verifier credential 和最终动作都留在服务端。；反馈闭环 负责客户后端把人工复核、误报、确认攻击和业务动作标签写回图谱，持续改进解释面。。这些动作需要和 release gate、verdict API、support bundle、feedback label、rollback action 配合，而不是停在单个移动端函数。

复盘指标建议：iOS 设备指纹不要做成本地风控开关：App Attest、越狱证据和 BoxId 如何进同一 envelope？ 应持续跟踪命中率、版本分布、渠道分布、失败原因、证据新鲜度、服务端解释结果、人工复核结果、误报申诉率和策略回滚次数。

## 统一发布检查

- 每篇文章正文不少于 10000 个中文字符。
- 每篇文章都有至少 5 条“事实依据与脱敏证据”。
- 每篇文章只聚焦单一产品、单一平台或单一场景。
- 外部平台稿按平台重写结构和开头，不直接堆叠自有站全文。
- GitHub Pages 只同步公开 Markdown、索引和静态站文件。
