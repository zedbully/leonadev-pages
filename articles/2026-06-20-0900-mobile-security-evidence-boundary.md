---
platform: github-pages
status: ready-for-public-pages
source_batch: 2026-06-20-0900-mobile-security-evidence-boundary
title: "2026-06-20 09:00 移动安全证据边界内容包"
permalink: /articles/2026-06-20-0900-mobile-security-evidence-boundary/
---

# 2026-06-20 09:00 移动安全证据边界内容包

本文是本轮公开技术内容索引，覆盖 5 篇自有站正式文章。正文只保留公开化工程事实、脱敏证据和验收方法。

## 本轮自有站文章入口

- 1. [能跑通的包为什么还不能上线：Android 加固发布门禁如何挡住 false pass？](https://dun.leonadev.com/article/android-false-pass-release-gate-same-candidate)
- 2. [iOS 包能安装不代表可交付：protected IPA、runner 和 evidence bundle 为什么必须绑在一起？](https://dun.leonadev.com/article/ios-release-gate-evidence-binding-not-install-success)
- 3. [把 SecretKey 塞进 APK 的设备指纹，为什么从第一天就失守？](https://dun.leonadev.com/article/android-backend-wrapper-secret-boundary-not-in-apk)
- 4. [网络失败不是越狱：iOS evidence envelope 为什么要把 transport 和风险证据拆开？](https://dun.leonadev.com/article/ios-transport-diagnostic-not-jailbreak-evidence)
- 5. [memfd 命中就封号吗？Android 游戏反外挂为什么还要等服务端 verdict？](https://dun.leonadev.com/article/android-native-mapping-game-cheat-server-verdict)

## 技术摘要

### 1. 能跑通的包为什么还不能上线：Android 加固发布门禁如何挡住 false pass？

产品与平台：御盾 Android

核心判断：Android 加固发布不是证明“包能打开”，而是证明同一候选产物在静态、动态、SO/provider、风险快照、业务 smoke 和性能口径上没有被错误晋级。

事实依据：

- 1. QA no-mix readiness 矩阵：矩阵把当前候选和旧候选分开记录，明确旧候选的静态通过不能导入当前候选的闭合链。 支撑判断：发布门禁必须禁止跨候选拼接证据，防止 false pass 进入上线流程。
- 2. QA no-mix readiness 矩阵：当前候选存在 authority support_waiting、final signer、lineage、source-stamp 输入缺失和 downstream rerun 要求。 支撑判断：构建 lineage 与签名材料缺失时，不能因为 APK 文件存在就认定可验收。
- 3. QA no-mix readiness 矩阵：动态记录显示安装与启动可到达应用，但进程存活不足、运行时异常存在，RiskSnapshot 和 key_delta 仍为空。 支撑判断：能启动不是动态闭合；风险快照缺失时业务 smoke 和性能 smoke 应保持 support_waiting。
- 4. QA no-mix readiness 矩阵：SO/provider 侧存在 classloader mismatch、native bound 时序和 provider readback 阻塞。 支撑判断：SO/provider 未通过时，不能用静态 DEX 或页面 smoke 代替 native 运行链证明。
- 5. 架构合同审计：审计结论为 must_rebuild：已有候选证明包装修复和静态 native-bridge 顺序，但没有证明后续 one-hop retry patch 集成到重建候选。 支撑判断：静态合同通过不能替代补丁集成和重建产物验证。
- 6. 架构合同审计：审计记录明确没有写入 GateEvidence，也没有声称 QA gate pass。 支撑判断：门禁系统应允许“有发现但不晋级”的状态，避免报告文字被误读为通过。

工程清单：

- no-mix 候选隔离：把当前候选、旧候选、支持性证据和阻塞性证据分开，任何跨候选拼接都不得晋级。
- 动态存活与风险快照：启动成功后继续检查进程存活、运行时异常、RiskSnapshot、key_delta 和材料生命周期。
- SO/provider 闭合：classloader、native bridge、provider readback 与材料绑定必须在同一候选上成立。
- 业务 smoke 延后：业务和性能 smoke 只有在静态、SO/provider、动态和风险前置证据闭合后才允许运行。

公开边界：不公开源码、私有路径、密钥、测试设备、内部账号、客户信息、函数名、偏移、原始 token/assertion、完整攻击复现链路或可直接绕过检测的实现细节。

### 2. iOS 包能安装不代表可交付：protected IPA、runner 和 evidence bundle 为什么必须绑在一起？

产品与平台：御盾 iOS

核心判断：iOS 加固验收的核心不是“IPA 能安装”，而是每个 gate artifact 都能绑定到同一个 protected IPA、同一次 runner/session 和同一个 evidence bundle。

事实依据：

- 1. iOS Release Gate 证据绑定校验：必要 gate artifact 不能只孤立输出 PASS，必须用脱敏字段绑定到同一个 protected IPA、同一次 runner/session、同一个 evidence bundle。 支撑判断：iOS 发布验收要证明同一产物链，而不是展示多个无关联报告。
- 2. iOS Release Gate 证据绑定校验：即使 6 个合成 gate artifact 都是 PASS 且绑定字段一致，当前阶段仍要求 commercialIosHardeningReady=false。 支撑判断：证据绑定只是基础设施，不能替代真实 runner 隔离、签名材料隔离、真机验收和逆向验收。
- 3. iOS Release Gate 证据绑定校验：顶层 evidenceBinding 支持 BOUND_CONSISTENT_NOT_COMMERCIAL_READY、UNBOUND、PARTIAL、INCONSISTENT 等状态。 支撑判断：状态口径要区分一致但未商业 ready、缺字段、部分绑定和跨 gate 不一致。
- 4. iOS Release Gate 证据绑定校验：每个 gate 必须提供 evidenceProducedAt 和自己的 gate 专属 evidence ref；缺失或跨 gate 不一致会 fail-closed。 支撑判断：发布门禁要阻断“看起来有报告但无法追溯”的交付。
- 5. iOS Real Device Gate 合同：真机 gate 缺少真实 protected IPA、签名安装 gate、受控 runner、实体设备或云真机证据时，必须输出 BLOCKED 且 commercialReady=false。 支撑判断：安装成功不能独立代表真机商业验收，输入缺失应直接阻断。
- 6. iOS Real Device Gate 合同：真机 smoke 子项包括 protectedIpaHash、installRecord、launchRecord、mainPathSmoke、protectedFunctionHit、protectedResourceHit、dispatcherBindingHit、shortWindowMaterialization、crashExitCode 和 deviceOsArchMatrix。 支撑判断：商业验收必须覆盖运行路径和保护命中，而不是只看安装结果。

工程清单：

- protected IPA 绑定：所有 gate 必须绑定同一个 protected IPA，不允许中间包、模拟器包、未签名包和真实交付包混用。
- runner/session 绑定：runner 或 session 只输出不可逆绑定或脱敏引用，确保 gate 之间可追溯但不可复用。
- evidence bundle 一致性：签名、真机、静态逆向、动态逆向、性能和 CI 证据都应指向同一证据包。
- 真机 smoke 子项：安装、启动、主路径、受保护命中、dispatcher/binding、短窗口材料和崩溃摘要必须逐项记录。

公开边界：不公开源码、私有路径、密钥、测试设备、内部账号、客户信息、函数名、偏移、原始 token/assertion、完整攻击复现链路或可直接绕过检测的实现细节。

### 3. 把 SecretKey 塞进 APK 的设备指纹，为什么从第一天就失守？

产品与平台：守界 Android

核心判断：设备证据 SDK 的边界不是把签名能力带进 APK，而是让 APK 只交 evidence，客户后端用 SecretKey 查询 verdict、拉取报告、提交反馈并做业务决策。

事实依据：

- 1. Backend Wrapper Contract：wrapper 的作用是帮助客户后端签名请求、查询 verdict/evidence、拉取 support bundle、提交 feedback label 和脱敏输出。 支撑判断：wrapper 是后端能力，不是移动 SDK 能力。
- 2. Backend Wrapper Contract：wrapper 明确不得运行在 Android App 内，不得嵌入 tenant SecretKey、provider credential、token 或真实 AppKey secret。 支撑判断：把密钥放进 APK 会把服务端信任根暴露给客户端逆向。
- 3. Backend Wrapper Contract：后端签名要求 timestamp、nonce 和请求体摘要，nonce 必须由安全随机源生成，timestamp 必须来自后端时钟。 支撑判断：请求签名要绑定服务端时间与随机性，不能依赖可被篡改的移动端时间。
- 4. Backend Wrapper Contract：HTTP 认证或 clock-skew 要作为 transport error 暴露，而不是解释成设备风险结论。 支撑判断：传输失败和设备环境证据必须分开，避免网络或认证错误触发误判。
- 5. Android Evidence Privacy Boundary：Android SDK 只允许输出 opaque BoxId、诊断状态、脱敏 support bundle 字段和 evidence 上传元数据，不允许输出 allow/reject/block/isFraud。 支撑判断：最终业务动作必须留在客户后端，SDK 不应成为裁判。
- 6. Device Identity Protocol：riskTags 只由 server verdict path 产生；client header evidence 默认 source=client_header、trust=low。 支撑判断：客户端上传的身份和证据只能作为低信任输入，不能更新权威身份或业务动作。

工程清单：

- 后端签名边界：SecretKey、nonce、timestamp、请求体摘要和 HMAC 签名只在客户后端生成。
- BoxId 查询路径：App 只传递 BoxId 或 evidence hint，客户后端调用 verdict/evidence/support bundle。
- transport diagnostic 分离：认证、时钟、TLS、超时和服务端错误进入传输诊断，不作为设备风险族。
- feedback 闭环：客户后端提交 fraud、false_positive、false_negative 等反馈标签，持续修正证据解释。

公开边界：不公开源码、私有路径、密钥、测试设备、内部账号、客户信息、函数名、偏移、原始 token/assertion、完整攻击复现链路或可直接绕过检测的实现细节。

### 4. 网络失败不是越狱：iOS evidence envelope 为什么要把 transport 和风险证据拆开？

产品与平台：守界 iOS

核心判断：iOS transport failure 只能说明采集或上报链路异常，不能被解释成 jailbreak、tamper、hook 或设备不可信。

事实依据：

- 1. iOS Evidence Extension Plan：iOS 最小闭环是 App 调用 sense，服务端返回 BoxId，客户后端查询 verdict 和 evidence report，最终业务决策留在客户后端。 支撑判断：transport 异常只能影响证据链新鲜度，不能让客户端本地裁决。
- 2. iOS Evidence Extension Plan：公开 API 允许 diagnosticSnapshot、supportBundle、lastServerEvidence，但禁止 isJailbroken、isFridaDetected、isTampered、shouldBlock 和 riskLevel。 支撑判断：SDK 不应把诊断状态或风险事实压成本地封禁按钮。
- 3. iOS Evidence Extension Plan：Evidence taxonomy 将 identity、runtime、jailbreak、Frida/injection、tamper、attestation、transport 分成独立 family。 支撑判断：网络错误、越狱线索、注入线索和证明状态应分别上报。
- 4. iOS Evidence Extension Plan：transport diagnostic 包括 network_timeout、auth_failed、server_5xx、timestamp_skew、tls_failure，并明确 transport failure 不解释为 jailbreak、tamper 或 hook。 支撑判断：传输失败应进入重试、降级或补证据流程，而不是触发风险封禁。
- 5. iOS Evidence Extension Plan：Cross-platform envelope 要求 unknown family 不能导致 server 500，client evidence 默认 trust=low，只进入 telemetry/provenance。 支撑判断：证据系统要能扩展字段并保持低信任输入边界。
- 6. 后端 wrapper/隐私边界：认证或时钟错误应作为 transport errors 表达；support bundle 和日志不得输出完整 BoxId、raw token、SecretKey 或原始设备标识。 支撑判断：错误解释和脱敏输出必须同时设计，才能降低误报和泄漏风险。

工程清单：

- transport family 独立：timeout、auth_failed、server_5xx、timestamp_skew 和 tls_failure 只表达传输状态。
- risk family 分层：jailbreak、Frida/injection、tamper、attestation 与 runtime evidence 分开上报。
- low-trust client evidence：客户端 evidence 默认低信任，权威结果来自服务端 policy、verifier 或可信私有路径。
- support bundle 脱敏：诊断包只输出 hash、hint、summary、status，不输出完整 BoxId、raw token、SecretKey 或原始标识。

公开边界：不公开源码、私有路径、密钥、测试设备、内部账号、客户信息、函数名、偏移、原始 token/assertion、完整攻击复现链路或可直接绕过检测的实现细节。

### 5. memfd 命中就封号吗？Android 游戏反外挂为什么还要等服务端 verdict？

产品与平台：守界 Android

核心判断：native mapping 命中是重要反外挂证据，但它必须与 BoxId、账号、会话、对局、历史反馈和服务端 verdict 结合，不能在客户端直接封号。

事实依据：

- 1. Device Identity Protocol：native payload 会被解码为 nativeFindingIds、nativeFactTags 和 nativeHighestSeverity，例如 memfd executable、deleted executable、Frida evidence、unidbg runtime fact。 支撑判断：native mapping 是证据族，需要服务端解释，不能在客户端本地封号。
- 2. Device Identity Protocol：nativeRiskTags 是兼容别名，实际语义仍应映射到 nativeFactTags；riskTags 只由 server verdict path 产生。 支撑判断：字段命名不能误导业务方把 native facts 当成最终风险标签。
- 3. Device Identity Protocol：legacy client-originated values 必须记录为低信任 telemetry，不能产生 authoritative risk tags、block tags、reject actions 或 canonical identity 更新。 支撑判断：客户端上报可以参与证据图谱，但不能更新权威身份或处罚动作。
- 4. Android Evidence Privacy Boundary：Android evidence family 包括 hook、Frida、native mapping、runtime injection、Root/Magisk、ROM、attestation 和 transport diagnostics。 支撑判断：游戏反外挂需要多证据族组合，不能只看单一 native mapping 命中。
- 5. Android Evidence Privacy Boundary：服务端可聚合 Device Evidence Graph、velocity windows、customer evidence reports 和 feedback evaluation reports，并保留 source、trust、provenance。 支撑判断：处罚前要看设备图谱、速度窗口、账号关系和客户反馈，而不是本地瞬时事实。
- 6. 动态风险门禁记录：Hook/debug/injection 路径被反复标记为禁止或仅允许 debug-safe support，缺少完整 runtime closure 时保持 blocked。 支撑判断：反外挂取证不能为了证明检测命中而引入新的材料泄露或越界采集。

工程清单：

- native fact taxonomy：memfd、deleted executable、Frida、unidbg、tamper 等事实记录为 nativeFindingIds、nativeFactTags、nativeHighestSeverity。
- server verdict 解释：riskTags、处罚标签、挑战动作和拒绝动作只由服务端根据业务上下文产生。
- 游戏场景分级：登录、对局、结算、排行榜、交易和奖励使用不同阈值与复核流程。
- feedback 反哺：误封、确认作弊、人工复核和运营标签写回 evidence graph，持续调整策略。

公开边界：不公开源码、私有路径、密钥、测试设备、内部账号、客户信息、函数名、偏移、原始 token/assertion、完整攻击复现链路或可直接绕过检测的实现细节。

## 深度技术笔记

### 1. 能跑通的包为什么还不能上线：Android 加固发布门禁如何挡住 false pass？：从事实到工程闭环

这篇文章属于 御盾 Android 的单一风险边界。它保留事实、判断、动作和边界的链路。

案例摘要：某 Android 加固候选包在测试机上能启动，静态检查也有一部分绿色记录。交付团队如果只看“能打开”和“部分 pass”，就会把候选推进上线。但脱敏 no-mix 矩阵显示，静态 pass 来自旧候选，当前候选缺 signer、lineage、source-stamp，动态存活和 RiskSnapshot 也没有闭合。 这个案例的风险在于 false pass 会污染后续判断。客户看到的是一个“已经验收”的保护包，实际却缺少 SO/provider、动态存活、业务 smoke 和性能 smoke 的同一候选证据。 御盾 Android 应把 false pass 当成发布事故前置项处理：每次上线只接受同一候选的完整证据链，旧候选、静态片段、人工说明和未集成补丁都不能成为商业 ready。

证据解释：证据 1 来自QA no-mix readiness 矩阵，观察到矩阵把当前候选和旧候选分开记录，明确旧候选的静态通过不能导入当前候选的闭合链。，因此支撑“发布门禁必须禁止跨候选拼接证据，防止 false pass 进入上线流程。”。公开时必须遵守边界：不公开候选散列、任务编号、原始 evidence id、设备、命令或日志。 证据 2 来自QA no-mix readiness 矩阵，观察到当前候选存在 authority support_waiting、final signer、lineage、source-stamp 输入缺失和 downstream rerun 要求。，因此支撑“构建 lineage 与签名材料缺失时，不能因为 APK 文件存在就认定可验收。”。公开时必须遵守边界：只公开缺口类别，不输出签名材料、构建清单、包路径或内部字段。 证据 3 来自QA no-mix readiness 矩阵，观察到动态记录显示安装与启动可到达应用，但进程存活不足、运行时异常存在，RiskSnapshot 和 key_delta 仍为空。，因此支撑“能启动不是动态闭合；风险快照缺失时业务 smoke 和性能 smoke 应保持 support_waiting。”。公开时必须遵守边界：不公开异常栈、设备、日志、类名、函数名或触发步骤。 证据 4 来自QA no-mix readiness 矩阵，观察到SO/provider 侧存在 classloader mismatch、native bound 时序和 provider readback 阻塞。，因此支撑“SO/provider 未通过时，不能用静态 DEX 或页面 smoke 代替 native 运行链证明。”。公开时必须遵守边界：不公开 provider 名称、桥接调用、原始日志、路径或 native 实现细节。 证据 5 来自架构合同审计，观察到审计结论为 must_rebuild：已有候选证明包装修复和静态 native-bridge 顺序，但没有证明后续 one-hop retry patch 集成到重建候选。，因此支撑“静态合同通过不能替代补丁集成和重建产物验证。”。公开时必须遵守边界：不公开源码路径、行号、类名、散列、构建清单或具体调用序列。 证据 6 来自架构合同审计，观察到审计记录明确没有写入 GateEvidence，也没有声称 QA gate pass。，因此支撑“门禁系统应允许“有发现但不晋级”的状态，避免报告文字被误读为通过。”。公开时必须遵守边界：只公开状态口径，不输出 DB 行、事件编号或内部审计命令。

工程路径：no-mix 候选隔离 负责把当前候选、旧候选、支持性证据和阻塞性证据分开，任何跨候选拼接都不得晋级。；动态存活与风险快照 负责启动成功后继续检查进程存活、运行时异常、RiskSnapshot、key_delta 和材料生命周期。；SO/provider 闭合 负责classloader、native bridge、provider readback 与材料绑定必须在同一候选上成立。；业务 smoke 延后 负责业务和性能 smoke 只有在静态、SO/provider、动态和风险前置证据闭合后才允许运行。。这些动作需要和 release gate、verdict API、support bundle、feedback label、rollback action 配合。

### 2. iOS 包能安装不代表可交付：protected IPA、runner 和 evidence bundle 为什么必须绑在一起？：从事实到工程闭环

这篇文章属于 御盾 iOS 的单一风险边界。它保留事实、判断、动作和边界的链路。

案例摘要：某 iOS protected IPA 在一台设备上安装成功，团队准备把截图和安装记录作为交付证据。审计时发现，安装记录没有绑定同一 protected IPA 摘要，runner/session 不能追溯，静态逆向和动态逆向 gate 也没有相同 evidence bundle。 如果真机 smoke 没有证明受保护函数命中、dispatcher/binding 命中、短窗口材料化和崩溃摘要，安装成功可能掩盖保护路径根本未执行。 御盾 iOS 的做法是把 evidence binding 放在 release gate 前面：字段缺失、部分绑定、跨 gate 不一致或真实输入缺失，都保持 BLOCKED。

证据解释：证据 1 来自iOS Release Gate 证据绑定校验，观察到必要 gate artifact 不能只孤立输出 PASS，必须用脱敏字段绑定到同一个 protected IPA、同一次 runner/session、同一个 evidence bundle。，因此支撑“iOS 发布验收要证明同一产物链，而不是展示多个无关联报告。”。公开时必须遵守边界：不公开 runner 原值、证据包原值、设备标识、原始日志、符号、地址或私钥。 证据 2 来自iOS Release Gate 证据绑定校验，观察到即使 6 个合成 gate artifact 都是 PASS 且绑定字段一致，当前阶段仍要求 commercialIosHardeningReady=false。，因此支撑“证据绑定只是基础设施，不能替代真实 runner 隔离、签名材料隔离、真机验收和逆向验收。”。公开时必须遵守边界：只公开 ready 边界，不输出内部 gate JSON、脚本或样本。 证据 3 来自iOS Release Gate 证据绑定校验，观察到顶层 evidenceBinding 支持 BOUND_CONSISTENT_NOT_COMMERCIAL_READY、UNBOUND、PARTIAL、INCONSISTENT 等状态。，因此支撑“状态口径要区分一致但未商业 ready、缺字段、部分绑定和跨 gate 不一致。”。公开时必须遵守边界：不公开 protected IPA 摘要、runner 绑定摘要或 bundle 真实引用。 证据 4 来自iOS Release Gate 证据绑定校验，观察到每个 gate 必须提供 evidenceProducedAt 和自己的 gate 专属 evidence ref；缺失或跨 gate 不一致会 fail-closed。，因此支撑“发布门禁要阻断“看起来有报告但无法追溯”的交付。”。公开时必须遵守边界：不公开 evidence ref 原文、设备唯一标识、完整日志或动态调试脚本。 证据 5 来自iOS Real Device Gate 合同，观察到真机 gate 缺少真实 protected IPA、签名安装 gate、受控 runner、实体设备或云真机证据时，必须输出 BLOCKED 且 commercialReady=false。，因此支撑“安装成功不能独立代表真机商业验收，输入缺失应直接阻断。”。公开时必须遵守边界：不公开 IPA 文件、设备账号、UDID、runner session 或原始 crash log。 证据 6 来自iOS Real Device Gate 合同，观察到真机 smoke 子项包括 protectedIpaHash、installRecord、launchRecord、mainPathSmoke、protectedFunctionHit、protectedResourceHit、dispatcherBindingHit、shortWindowMaterialization、crashExitCode 和 deviceOsArchMatrix。，因此支撑“商业验收必须覆盖运行路径和保护命中，而不是只看安装结果。”。公开时必须遵守边界：只公开子项类别，不输出函数地址、opcode、符号名、业务数据或调试脚本。

工程路径：protected IPA 绑定 负责所有 gate 必须绑定同一个 protected IPA，不允许中间包、模拟器包、未签名包和真实交付包混用。；runner/session 绑定 负责runner 或 session 只输出不可逆绑定或脱敏引用，确保 gate 之间可追溯但不可复用。；evidence bundle 一致性 负责签名、真机、静态逆向、动态逆向、性能和 CI 证据都应指向同一证据包。；真机 smoke 子项 负责安装、启动、主路径、受保护命中、dispatcher/binding、短窗口材料和崩溃摘要必须逐项记录。。这些动作需要和 release gate、verdict API、support bundle、feedback label、rollback action 配合。

### 3. 把 SecretKey 塞进 APK 的设备指纹，为什么从第一天就失守？：从事实到工程闭环

这篇文章属于 守界 Android 的单一风险边界。它保留事实、判断、动作和边界的链路。

案例摘要：某团队为了省后端接入工作，计划把设备证据服务的 SecretKey 放进 Android App，由 App 直接调用 verdict 并根据返回结果 block 用户。这个设计看似减少接口，但等于把服务端信任根、策略入口和业务动作都放到最容易被逆向和 patch 的位置。 脱敏 wrapper 合同给出的正确路径是：App 获取 BoxId 或上报 evidence，客户后端持有 SecretKey，后端查询 verdict、拉取 evidence report、提交 feedback label，再由客户业务系统决定观察、挑战、限速、复核或拒绝。 守界 Android 的产品边界必须在接入文档中写清。SDK 越克制，后端越有解释空间；密钥越少出现在客户端，攻击者越难把移动端变成绕过服务端的入口。

证据解释：证据 1 来自Backend Wrapper Contract，观察到wrapper 的作用是帮助客户后端签名请求、查询 verdict/evidence、拉取 support bundle、提交 feedback label 和脱敏输出。，因此支撑“wrapper 是后端能力，不是移动 SDK 能力。”。公开时必须遵守边界：不公开真实 endpoint、SecretKey、签名样本、nonce、完整 BoxId 或客户策略。 证据 2 来自Backend Wrapper Contract，观察到wrapper 明确不得运行在 Android App 内，不得嵌入 tenant SecretKey、provider credential、token 或真实 AppKey secret。，因此支撑“把密钥放进 APK 会把服务端信任根暴露给客户端逆向。”。公开时必须遵守边界：只公开边界，不输出凭据格式、生产配置或内部鉴权细节。 证据 3 来自Backend Wrapper Contract，观察到后端签名要求 timestamp、nonce 和请求体摘要，nonce 必须由安全随机源生成，timestamp 必须来自后端时钟。，因此支撑“请求签名要绑定服务端时间与随机性，不能依赖可被篡改的移动端时间。”。公开时必须遵守边界：不公开签名样例、密钥、请求体原文、Authorization 或 HMAC 结果。 证据 4 来自Backend Wrapper Contract，观察到HTTP 认证或 clock-skew 要作为 transport error 暴露，而不是解释成设备风险结论。，因此支撑“传输失败和设备环境证据必须分开，避免网络或认证错误触发误判。”。公开时必须遵守边界：不公开真实状态码上下文、租户、请求日志或 raw body。 证据 5 来自Android Evidence Privacy Boundary，观察到Android SDK 只允许输出 opaque BoxId、诊断状态、脱敏 support bundle 字段和 evidence 上传元数据，不允许输出 allow/reject/block/isFraud。，因此支撑“最终业务动作必须留在客户后端，SDK 不应成为裁判。”。公开时必须遵守边界：不公开完整 BoxId、raw device ID、install ID、serial、Android ID 或 provider token。 证据 6 来自Device Identity Protocol，观察到riskTags 只由 server verdict path 产生；client header evidence 默认 source=client_header、trust=low。，因此支撑“客户端上传的身份和证据只能作为低信任输入，不能更新权威身份或业务动作。”。公开时必须遵守边界：不公开 header 原文、native payload、完整 finding 映射或 canonical ID。

工程路径：后端签名边界 负责SecretKey、nonce、timestamp、请求体摘要和 HMAC 签名只在客户后端生成。；BoxId 查询路径 负责App 只传递 BoxId 或 evidence hint，客户后端调用 verdict/evidence/support bundle。；transport diagnostic 分离 负责认证、时钟、TLS、超时和服务端错误进入传输诊断，不作为设备风险族。；feedback 闭环 负责客户后端提交 fraud、false_positive、false_negative 等反馈标签，持续修正证据解释。。这些动作需要和 release gate、verdict API、support bundle、feedback label、rollback action 配合。

### 4. 网络失败不是越狱：iOS evidence envelope 为什么要把 transport 和风险证据拆开？：从事实到工程闭环

这篇文章属于 守界 iOS 的单一风险边界。它保留事实、判断、动作和边界的链路。

案例摘要：某 iOS App 在弱网或服务端升级期间频繁出现 sense timeout，业务方想把 timeout 直接当成“疑似越狱或注入”处理。这个方案会把网络、认证、TLS、时钟和服务端错误全部混进风险判断，正常用户会被误伤。 守界 iOS 的 envelope 把 transport 单独列为 evidence family。network_timeout、auth_failed、server_5xx、timestamp_skew 和 tls_failure 只说明上报链路状态；jailbreak、Frida、tamper、attestation 各自有独立字段。 public SDK 不输出 shouldBlock，也不暴露 raw token、完整 BoxId 或 SecretKey；客户后端可以根据业务价值决定是否重试、延迟、要求补证据或临时观察。

证据解释：证据 1 来自iOS Evidence Extension Plan，观察到iOS 最小闭环是 App 调用 sense，服务端返回 BoxId，客户后端查询 verdict 和 evidence report，最终业务决策留在客户后端。，因此支撑“transport 异常只能影响证据链新鲜度，不能让客户端本地裁决。”。公开时必须遵守边界：不公开真实 endpoint、AppKey、完整 BoxId、客户策略或生产请求。 证据 2 来自iOS Evidence Extension Plan，观察到公开 API 允许 diagnosticSnapshot、supportBundle、lastServerEvidence，但禁止 isJailbroken、isFridaDetected、isTampered、shouldBlock 和 riskLevel。，因此支撑“SDK 不应把诊断状态或风险事实压成本地封禁按钮。”。公开时必须遵守边界：不公开私有 detector、路径库、权重、绕过细节或测试设备。 证据 3 来自iOS Evidence Extension Plan，观察到Evidence taxonomy 将 identity、runtime、jailbreak、Frida/injection、tamper、attestation、transport 分成独立 family。，因此支撑“网络错误、越狱线索、注入线索和证明状态应分别上报。”。公开时必须遵守边界：不公开 raw IDFV、raw keychain id、完整证书、完整签名材料或私有规则。 证据 4 来自iOS Evidence Extension Plan，观察到transport diagnostic 包括 network_timeout、auth_failed、server_5xx、timestamp_skew、tls_failure，并明确 transport failure 不解释为 jailbreak、tamper 或 hook。，因此支撑“传输失败应进入重试、降级或补证据流程，而不是触发风险封禁。”。公开时必须遵守边界：不公开真实状态码上下文、请求体、header、IP 或生产域名。 证据 5 来自iOS Evidence Extension Plan，观察到Cross-platform envelope 要求 unknown family 不能导致 server 500，client evidence 默认 trust=low，只进入 telemetry/provenance。，因此支撑“证据系统要能扩展字段并保持低信任输入边界。”。公开时必须遵守边界：不公开完整 envelope 真实样本、客户数据或 provider credential。 证据 6 来自后端 wrapper/隐私边界，观察到认证或时钟错误应作为 transport errors 表达；support bundle 和日志不得输出完整 BoxId、raw token、SecretKey 或原始设备标识。，因此支撑“错误解释和脱敏输出必须同时设计，才能降低误报和泄漏风险。”。公开时必须遵守边界：不公开 Authorization、nonce、signature、raw token、SecretKey 或完整标识。

工程路径：transport family 独立 负责timeout、auth_failed、server_5xx、timestamp_skew 和 tls_failure 只表达传输状态。；risk family 分层 负责jailbreak、Frida/injection、tamper、attestation 与 runtime evidence 分开上报。；low-trust client evidence 负责客户端 evidence 默认低信任，权威结果来自服务端 policy、verifier 或可信私有路径。；support bundle 脱敏 负责诊断包只输出 hash、hint、summary、status，不输出完整 BoxId、raw token、SecretKey 或原始标识。。这些动作需要和 release gate、verdict API、support bundle、feedback label、rollback action 配合。

### 5. memfd 命中就封号吗？Android 游戏反外挂为什么还要等服务端 verdict？：从事实到工程闭环

这篇文章属于 守界 Android 的单一风险边界。它保留事实、判断、动作和边界的链路。

案例摘要：某手游在客户端检测到 memfd executable 或 deleted executable 后，准备直接封号。短期看这能快速打击一部分外挂，但也会带来误封：调试工具、兼容层、云手机、厂商安全组件、测试环境和被动注入都可能让 native mapping 呈现异常。 守界 Android 的反外挂证据链把 native facts 放进服务端图谱。客户端上报 nativeFindingIds、nativeFactTags、nativeHighestSeverity 和 BoxId hint；客户后端结合账号、对局、排行榜、交易、速度窗口、设备历史和人工复核决定动作。 这个模型不会削弱反外挂，反而让处罚更稳。攻击者需要同时伪造设备证据、账号行为和服务端历史；正常用户出现单点异常时也有申诉和回滚空间。

证据解释：证据 1 来自Device Identity Protocol，观察到native payload 会被解码为 nativeFindingIds、nativeFactTags 和 nativeHighestSeverity，例如 memfd executable、deleted executable、Frida evidence、unidbg runtime fact。，因此支撑“native mapping 是证据族，需要服务端解释，不能在客户端本地封号。”。公开时必须遵守边界：不公开 native payload、完整 finding 映射、样本、脚本或检测实现。 证据 2 来自Device Identity Protocol，观察到nativeRiskTags 是兼容别名，实际语义仍应映射到 nativeFactTags；riskTags 只由 server verdict path 产生。，因此支撑“字段命名不能误导业务方把 native facts 当成最终风险标签。”。公开时必须遵守边界：不公开 header 原文、签名字段、完整 BoxId 或 canonical identity。 证据 3 来自Device Identity Protocol，观察到legacy client-originated values 必须记录为低信任 telemetry，不能产生 authoritative risk tags、block tags、reject actions 或 canonical identity 更新。，因此支撑“客户端上报可以参与证据图谱，但不能更新权威身份或处罚动作。”。公开时必须遵守边界：不公开 legacy 字段原文、客户数据或服务端策略。 证据 4 来自Android Evidence Privacy Boundary，观察到Android evidence family 包括 hook、Frida、native mapping、runtime injection、Root/Magisk、ROM、attestation 和 transport diagnostics。，因此支撑“游戏反外挂需要多证据族组合，不能只看单一 native mapping 命中。”。公开时必须遵守边界：不公开完整包列表、raw device ID、完整 build fingerprint 或 provider token。 证据 5 来自Android Evidence Privacy Boundary，观察到服务端可聚合 Device Evidence Graph、velocity windows、customer evidence reports 和 feedback evaluation reports，并保留 source、trust、provenance。，因此支撑“处罚前要看设备图谱、速度窗口、账号关系和客户反馈，而不是本地瞬时事实。”。公开时必须遵守边界：不公开客户图谱、账号、对局、订单或处罚记录。 证据 6 来自动态风险门禁记录，观察到Hook/debug/injection 路径被反复标记为禁止或仅允许 debug-safe support，缺少完整 runtime closure 时保持 blocked。，因此支撑“反外挂取证不能为了证明检测命中而引入新的材料泄露或越界采集。”。公开时必须遵守边界：不公开 hook 工具、调试脚本、目标函数、设备或运行输出。

工程路径：native fact taxonomy 负责memfd、deleted executable、Frida、unidbg、tamper 等事实记录为 nativeFindingIds、nativeFactTags、nativeHighestSeverity。；server verdict 解释 负责riskTags、处罚标签、挑战动作和拒绝动作只由服务端根据业务上下文产生。；游戏场景分级 负责登录、对局、结算、排行榜、交易和奖励使用不同阈值与复核流程。；feedback 反哺 负责误封、确认作弊、人工复核和运营标签写回 evidence graph，持续调整策略。。这些动作需要和 release gate、verdict API、support bundle、feedback label、rollback action 配合。

## 统一发布检查

- 每篇文章正文不少于 10000 个中文字符。
- 每篇文章都有至少 5 条“事实依据与脱敏证据”。
- 每篇文章只聚焦单一产品、单一平台或单一场景。
