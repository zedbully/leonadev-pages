---
title: "2026-06-19 09:00 移动安全发布证据内容包"
permalink: /articles/2026-06-19-0900-mobile-security-release-evidence/
layout: default
---
# 2026-06-19 09:00 移动安全发布证据内容包

本文是本轮公开技术内容索引，适合发布到 GitHub Pages 作为长期沉淀页。它覆盖 5 篇自有站正式文章，并为外部平台发布提供入口。正文只保留公开化工程事实、脱敏证据和验收方法。

## 本轮自有站文章入口

- 1. [只验签为什么挡不住二次打包：Android 包 lineage 必须接上运行时证据](https://dun.leonadev.com/article/android-repackaging-package-lineage-runtime-evidence-gate)
- 2. [主 App 能启动不代表扩展安全：iOS Framework、Extension 和 App Clip 怎么做独立门禁？](https://dun.leonadev.com/article/ios-embedded-targets-extension-appclip-release-gate)
- 3. [没有 GMS 也不能假通过：大陆 Android attestation fallback 怎么分层？](https://dun.leonadev.com/article/android-mainland-attestation-fallback-evidence-gate)
- 4. [Bootloader 解锁不是封禁理由：Android ROM 证据为什么要和 Root/模拟器拆开？](https://dun.leonadev.com/article/android-rom-bootloader-evidence-not-policy)
- 5. [App Attest 接上了也不能在客户端判定：iOS 私有 verifier 为什么必须留在后端？](https://dun.leonadev.com/article/ios-appattest-private-verifier-release-gate)

## 技术摘要

### 1. 只验签为什么挡不住二次打包：Android 包 lineage 必须接上运行时证据

产品与平台：御盾 Android

核心判断：二次打包不是一个签名布尔值，而是一条从发布产物、安装包结构、运行时材料、设备环境到服务端合法版本集合的证据链。

事实依据：

- 1. 加固缺口矩阵：当前样本已经具备 DEX VMP、SO VMP 和自定义 linker 的基础链路，但高强度运行时签名验证仍只达到“部分”状态，未形成包签名、版本、设备风险与 VM/SO 材料的强绑定。 支撑判断：二次打包治理不能停在 APK 签名层面，必须把签名事实继续传递到运行时密钥、保护载荷和服务端合法版本集合。
- 2. 加固缺口矩阵：完整性校验仍存在依赖 CRC 的弱口径，CRC 只能证明传输或文件损坏，不能证明攻击者没有重写资源、DEX、SO 或配置。 支撑判断：包完整性需要 keyed MAC、签名绑定、分块校验、运行态自校验和服务端挑战，而不是只用本地 CRC 做强防篡改结论。
- 3. 签名算法还原复盘：脱敏复盘显示，接口签名入口迁入 native/VM 后，仍可能被运行期输入输出、dispatcher 行为和材料化窗口逐步关联。 支撑判断：客户端算法隐藏只能提高成本，不能阻止攻击者把重打包样本接回服务端接口；服务端必须要求新鲜证据和合法版本匹配。
- 4. 运行时门禁记录：动态门禁要求类加载上下文、provider 结果、冷启动路径和功能 smoke 同时成立，而不是只看启动页能否打开。 支撑判断：二次打包验收要把页面、组件、ClassLoader、native bridge、服务端回执放到同一条证据链里。
- 5. 加固缺口矩阵：风险环境识别仍是部分状态，root、Magisk、多开、AOSP、VPN 等事实尚未统一进入风险评分、key 派生或降级闭环。 支撑判断：如果重打包样本运行在高风险环境，保护链路应把环境事实用于材料失效、挑战升级或服务端复核。
- 6. 后端包装契约：后端 wrapper 的职责是签名 Leona API 请求、查询 verdict/evidence、提交 feedback、做脱敏；它不能跑在 Android App 内，也不能输出客户业务 allow/reject/block。 支撑判断：重打包治理的最终动作必须留在客户后端，客户端只报告证据和保护状态。

工程清单：

- 包 lineage 建模：把 package、签名方案、证书摘要、渠道、版本、资源摘要、DEX/SO 摘要、构建时间窗和灰度状态建成服务端版本集合，客户端只能上报事实，不能自封可信。
- 运行时证据绑定：把 ClassLoader、native bridge、provider、保护载荷、关键业务入口和服务端挑战绑定到同一次会话，让重打包样本不能只离线复制静态文件。
- 完整性升级路径：把 CRC 视为损坏检测，将强防篡改迁移到 keyed MAC、签名绑定、分块摘要、短窗口材料和运行态自校验。
- 服务端解释与灰度：把证据解释放到客户后端，区分观察、挑战、限速、复核、延迟和拒绝，不把所有异常都交给客户端阻断。

公开边界：不公开源码、私有路径、密钥、测试设备、内部账号、客户信息、函数名、偏移、原始 token/assertion、完整攻击复现链路或可直接绕过检测的实现细节。

### 2. 主 App 能启动不代表扩展安全：iOS Framework、Extension 和 App Clip 怎么做独立门禁？

产品与平台：御盾 iOS

核心判断：iOS 保护产物的商业验收不能只看主 App 启动，还必须逐个清点 Framework、Extension、App Clip 和 Watch companion 的签名、entitlements、平台 slice 与真机行为。

事实依据：

- 1. 签名安装兼容验收模板：真实验收必须使用同一次保护构建和同一签名流程产生的 protected IPA、签名证书摘要、provisioning profile、entitlements 摘要、Info.plist 摘要和 release manifest。 支撑判断：iOS 加固验收必须绑定同一产物链，不能用未签名中间包、模拟器包或人工说明替代。
- 2. 签名安装兼容验收模板：验收模板要求清点主 App、framework、extension、App Clip、Watch companion 和 debug artifact。 支撑判断：主 App 能启动不代表嵌入目标安全，所有目标都要独立做结构、签名、entitlements 和平台 slice 检查。
- 3. 签名安装兼容验收模板：没有 protected IPA、签名材料或真机时，所有工具项必须为 NOT_RUN，商业 ready 必须为 false。 支撑判断：发布系统要 fail-closed，不得把缺失证据包装成“通过”。
- 4. 真机门禁合同：真实通过必须覆盖安装、启动、主路径、受保护函数或资源命中、dispatcher/binding 命中、短窗口 materialization、退出码或崩溃摘要和设备矩阵。 支撑判断：iOS 加固验收不能停在安装成功，必须证明受保护路径在真实设备上命中。
- 5. 真机门禁合同：缺少任一真实输入、签名安装 gate、受控 runner、实体设备或必需 smoke 子项时，producer 必须输出 BLOCKED，并保持 commercialReady=false。 支撑判断：门禁结论应可审计、可阻断、可回滚，不能用静态分析结果替代真机运行证据。
- 6. iOS 隐私与 App Attest 边界：public iOS SDK 只能公开 support/status/challenge-required 等 evidence-only 状态，不持有 Apple 私有材料、raw token/assertion 或客户端业务裁决 API。 支撑判断：嵌入目标验收还要检查隐私与 attestation 边界，避免扩展目标错误携带私有验证能力。

工程清单：

- 嵌入目标 inventory：先列出主 App、Framework、Extension、App Clip、Watch companion 和调试残留，再逐个绑定签名、权限、平台和保护状态。
- 签名与权限差异：对比实际 entitlements 与 profile 覆盖关系，避免扩展目标拥有不必要权限，也避免保护重签后权限丢失。
- 真机 smoke 证据：把安装、启动、主路径、受保护路径、dispatcher/binding、短窗口材料和退出状态写成结构化证据。
- fail-closed 门禁：没有真实 protected IPA、签名材料、受控 runner 或真机证据时只允许 NOT_RUN/BLOCKED，不允许商业 ready。

公开边界：不公开源码、私有路径、密钥、测试设备、内部账号、客户信息、函数名、偏移、原始 token/assertion、完整攻击复现链路或可直接绕过检测的实现细节。

### 3. 没有 GMS 也不能假通过：大陆 Android attestation fallback 怎么分层？

产品与平台：守界 Android

核心判断：大陆 Android attestation 的难点不在于把所有设备判成可信，而在于明确 provider、fallback、证据新鲜度、服务端权威和运营可量化边界。

事实依据：

- 1. 大陆非 GMS 发布门禁：门禁适用于目标设备缺少 Google Play 服务、依赖 OEM attestation 或暂时允许 no_attestation fallback 的场景。 支撑判断：大陆 Android 设备证明必须先判断 provider 条件，不能照搬单一 GMS 路线。
- 2. 大陆非 GMS 发布门禁：最小闭环要求静态 build gate、样例大陆构建安装路径、emulator attestation summary E2E 和 fallback path validation。 支撑判断：debug 流程可以验证链路，但不能替代真实 staging gate；每个阶段都要写清证据状态。
- 3. 大陆非 GMS 发布门禁：真实 staging gate 要求私有 OEM bridge、私有后端 verifier、可信 provider allowlist、oem_attested 握手和下游 decisioning 反映 OEM posture。 支撑判断：OEM attestation 生产 ready 依赖外部私有材料，缺失时应标记 blocked_external_input。
- 4. 大陆非 GMS 发布门禁：停止条件包括 OEM verifier 缺失、allowlist 为空、只有 fake path 通过、失败 OEM 被静默当作 no_attestation、fallback 流量不可量化。 支撑判断：fallback 不是成功状态，必须在报表中与 verified traffic 区分。
- 5. 设备身份协议草图：Android SDK 维护 installId、resolvedDeviceId 和 fingerprintHash 等身份层，但 canonical identity 归服务端所有，客户端 canonical 只能作为 claim。 支撑判断：attestation 只是设备证据之一，不能替代服务端身份权威和历史图谱。
- 6. 后端包装契约：wrapper 只能在服务端签名请求、查询 verdict/evidence、提交 feedback 和脱敏；不能放进 Android App，也不能输出业务 allow/reject/block。 支撑判断：大陆 attestation 的解释和业务动作必须在客户后端完成，SDK 只交付 evidence。

工程清单：

- provider 分层：把 GMS、OEM、binding fallback 和 no_attestation 分成不同 provider/status，不在 UI 或日志中合并成一个通过状态。
- fallback 量化：在服务端记录 fallback 占比、渠道、版本、设备类别、失败原因和灰度策略，避免运营只看到总量。
- identity 权威边界：客户端 installId 和 fingerprintHash 是证据，canonical identity 与 riskTags 只能由服务端产生。
- 后端 wrapper 安全：wrapper 只在服务端持有凭据和签名能力，Android App 内不得嵌入 SecretKey 或最终业务策略。

公开边界：不公开源码、私有路径、密钥、测试设备、内部账号、客户信息、函数名、偏移、原始 token/assertion、完整攻击复现链路或可直接绕过检测的实现细节。

### 4. Bootloader 解锁不是封禁理由：Android ROM 证据为什么要和 Root/模拟器拆开？

产品与平台：守界 Android

核心判断：Bootloader 解锁、custom ROM、root、模拟器、hook 和 debug 是不同证据族，不能在客户端压成一个封禁结论。

事实依据：

- 1. ROM/Bootloader 矩阵：矩阵明确要求 public SDK 只采集和报告 custom ROM、root、bootloader、emulator posture 的 evidence，不在客户端做本地 allow/deny。 支撑判断：ROM 证据必须进入服务端解释，不能在客户端直接封禁。
- 2. ROM/Bootloader 矩阵：矩阵要求把 ROM/bootloader facts 与 emulator、root、hook、debug evidence 分开，覆盖 custom AOSP、社区 ROM、GSI/DSU、bootloader unlocked 和 clean OEM control。 支撑判断：不同证据族风险含义不同，合并会导致误报、策略不可解释和客户申诉困难。
- 3. ROM/Bootloader 矩阵：posture collector 的输出默认脱敏，不保存完整 ADB serial、Android ID、build fingerprint、bootloader version 或 root manager package name。 支撑判断：设备证据要能做关联和排障，但不能把原始标识当成公开稳定身份。
- 4. ROM/Bootloader 矩阵：derivedEvidence 只覆盖 ROM、bootloader、build-channel、GSI/Treble 和 root-manager posture；空值不证明设备是干净物理机。 支撑判断：没有命中某类 ROM 事实不等于低风险，还需要模拟器、云手机、hook 和 server verdict 补充。
- 5. 设备身份协议草图：nativeFindingIds、nativeFactTags 和 nativeHighestSeverity 是 evidence，只有 server verdict policy 分类后才成为 riskTags。 支撑判断：本地 native facts 不应直接触发客户业务封禁，必须经过服务端 policy 和业务场景解释。
- 6. 后端包装契约：wrapper 不能输出 allow/reject/block，最终 business decision 不属于 SDK 或 wrapper 的职责。 支撑判断：bootloader、ROM、root 和模拟器证据应由客户后端按登录、支付、游戏结算、企业数据导出等动作分级处理。

工程清单：

- 证据族拆分：verified boot、bootloader、verity、build channel、GSI/Treble、ROM hint、root manager、emulator、hook/debug 必须分开记录。
- 脱敏采集：完整 serial、Android ID、完整指纹和 raw package name 不进入公开报告，短 hash 只做关联线索，不做身份权威。
- server verdict：native facts 和 environment facts 上传后由服务端生成 riskTags，客户端不直接做业务动作。
- 误报复盘：把用户申诉、人工复核、业务损失和策略回滚写回证据图谱，避免长期堆叠粗暴封禁。

公开边界：不公开源码、私有路径、密钥、测试设备、内部账号、客户信息、函数名、偏移、原始 token/assertion、完整攻击复现链路或可直接绕过检测的实现细节。

### 5. App Attest 接上了也不能在客户端判定：iOS 私有 verifier 为什么必须留在后端？

产品与平台：守界 iOS

核心判断：App Attest 是重要证据，但 challenge、replay、key registration、assertion verification、Apple material 和最终业务动作必须留在服务端。

事实依据：

- 1. App Attest 边界文档：public iOS SDK 只能收集 DeviceCheck support、DeviceCheck token request status、App Attest support、key generation status、assertion status 和 challenge binding status 等 evidence-only 状态。 支撑判断：客户端看到的是能力和集成状态，不是设备可信的最终判定。
- 2. App Attest 边界文档：缺少 server challenge 时，token、key generation、assertion 等状态应保持 not_requested 或 server_challenge_required。 支撑判断：没有服务端挑战就不能宣称 App Attest 完整闭环，更不能在客户端产生通过结论。
- 3. App Attest 边界文档：server challenge、replay protection、key registration、assertion verification、Apple key、Team/bundle allowlist、verifier credential、tenant policy 和最终业务动作都属于私有服务端职责。 支撑判断：private verifier 必须留在后端，public SDK 不能携带生产验证材料。
- 4. iOS 隐私清单：公开 SDK 收集 app/auth context、install identity、device context、runtime evidence、attestation capability、transport diagnostics 和 response diagnostics，字段以 hash、hint、summary、status 为主。 支撑判断：设备证明要在隐私最小化边界内工作，support bundle 也不能泄露原始标识。
- 5. iOS 隐私清单：公开 SDK 不提供 allow/reject/block/jailbreak/Frida/tamper/risk-level API。 支撑判断：App Attest 结果应交给客户后端解释，不能被 App 内本地业务逻辑直接消费成封禁按钮。
- 6. 跨平台后端契约：iOS 与 Android/Web 共用 BoxId、/v1/verdict、evidence report、Device Evidence Graph 和 feedback 模型。 支撑判断：App Attest 状态要进入统一证据图谱和反馈闭环，而不是形成 iOS 孤岛。

工程清单：

- public SDK 状态模型：公开包只表达 support、not_requested、server_challenge_required、transport hint 等低信任状态。
- private verifier 边界：challenge、replay、key registration、assertion verification、Apple material、tenant policy 和最终动作全部留在后端。
- 隐私清单约束：raw IDFV、raw keychain id、完整 BoxId、raw token/assertion、AppKey/SecretKey 和 Apple 私有材料不进入公开包。
- 跨平台 evidence graph：App Attest 状态与 Android/Web 设备证据共用 verdict、report、graph 和 feedback，避免孤岛。

公开边界：不公开源码、私有路径、密钥、测试设备、内部账号、客户信息、函数名、偏移、原始 token/assertion、完整攻击复现链路或可直接绕过检测的实现细节。

## 深度技术笔记

### 1. 只验签为什么挡不住二次打包：Android 包 lineage 必须接上运行时证据：从事实到工程闭环

这篇文章的主线是“只验签为什么挡不住二次打包：Android 包 lineage 必须接上运行时证据”，属于 御盾 Android 的单一风险边界。它不把多个产品揉在一起，也不把平台差异抹平。对长期文档来说，最重要的是保留“事实、判断、动作、边界”的链路：事实来自脱敏资料，判断来自工程验收，动作落在发布门禁和客户后端，边界保护源码、密钥、设备、路径、客户信息和可复现攻击细节。

案例摘要：某 Android App 已经接入壳、字符串隐藏和 native 签名校验，发行团队认为“签名验证通过”就足以防住二次打包。脱敏复盘却发现，攻击者并不需要理解所有保护逻辑，只要保留启动链路、修补一个本地校验返回、替换资源入口，再让接口签名算法在运行时继续输出可用结果，就能让伪包获得足够业务能力。 这类问题的核心不是某个检测点没写，而是证据没有连起来。安装包签名、资源摘要、DEX 摘要、SO 摘要、渠道版本、运行时风险、ClassLoader 上下文、接口签名上下文和服务端合法版本集合，如果彼此断开，攻击者就能挑最容易 patch 的一段下手。 御盾 Android 在这个场景下的工程目标，是让重打包样本必须同时满足包 lineage、运行时材料、环境姿态、服务端挑战和版本集合，而不是只修一个布尔值。防守侧要把“看起来能启动”降级为低信任事实，把“证据链一致且新鲜”升级为高价值业务动作前置条件。

证据解释：证据 1 来自加固缺口矩阵，观察到当前样本已经具备 DEX VMP、SO VMP 和自定义 linker 的基础链路，但高强度运行时签名验证仍只达到“部分”状态，未形成包签名、版本、设备风险与 VM/SO 材料的强绑定。，因此支撑“二次打包治理不能停在 APK 签名层面，必须把签名事实继续传递到运行时密钥、保护载荷和服务端合法版本集合。”。公开时必须遵守边界：不公开包名、证书摘要、构建参数、样本散列、原始门禁 JSON 或内部通道。 证据 2 来自加固缺口矩阵，观察到完整性校验仍存在依赖 CRC 的弱口径，CRC 只能证明传输或文件损坏，不能证明攻击者没有重写资源、DEX、SO 或配置。，因此支撑“包完整性需要 keyed MAC、签名绑定、分块校验、运行态自校验和服务端挑战，而不是只用本地 CRC 做强防篡改结论。”。公开时必须遵守边界：只公开校验强度差异，不输出具体校验位置、密钥派生细节或替换路径。 证据 3 来自签名算法还原复盘，观察到脱敏复盘显示，接口签名入口迁入 native/VM 后，仍可能被运行期输入输出、dispatcher 行为和材料化窗口逐步关联。，因此支撑“客户端算法隐藏只能提高成本，不能阻止攻击者把重打包样本接回服务端接口；服务端必须要求新鲜证据和合法版本匹配。”。公开时必须遵守边界：不公开函数名、偏移、样本输入输出、脚本、常量或可复现分析流程。 证据 4 来自运行时门禁记录，观察到动态门禁要求类加载上下文、provider 结果、冷启动路径和功能 smoke 同时成立，而不是只看启动页能否打开。，因此支撑“二次打包验收要把页面、组件、ClassLoader、native bridge、服务端回执放到同一条证据链里。”。公开时必须遵守边界：不公开 runner、设备编号、命令、日志、内部路径或 gate 原始字段。 证据 5 来自加固缺口矩阵，观察到风险环境识别仍是部分状态，root、Magisk、多开、AOSP、VPN 等事实尚未统一进入风险评分、key 派生或降级闭环。，因此支撑“如果重打包样本运行在高风险环境，保护链路应把环境事实用于材料失效、挑战升级或服务端复核。”。公开时必须遵守边界：只公开环境事实参与方式，不公开检测规则、权重、绕过样本或设备信息。 证据 6 来自后端包装契约，观察到后端 wrapper 的职责是签名 Leona API 请求、查询 verdict/evidence、提交 feedback、做脱敏；它不能跑在 Android App 内，也不能输出客户业务 allow/reject/block。，因此支撑“重打包治理的最终动作必须留在客户后端，客户端只报告证据和保护状态。”。公开时必须遵守边界：不公开 SecretKey、Authorization、nonce 签名组合、生产 endpoint 或租户策略。

工程路径：包 lineage 建模 负责把 package、签名方案、证书摘要、渠道、版本、资源摘要、DEX/SO 摘要、构建时间窗和灰度状态建成服务端版本集合，客户端只能上报事实，不能自封可信。；运行时证据绑定 负责把 ClassLoader、native bridge、provider、保护载荷、关键业务入口和服务端挑战绑定到同一次会话，让重打包样本不能只离线复制静态文件。；完整性升级路径 负责把 CRC 视为损坏检测，将强防篡改迁移到 keyed MAC、签名绑定、分块摘要、短窗口材料和运行态自校验。；服务端解释与灰度 负责把证据解释放到客户后端，区分观察、挑战、限速、复核、延迟和拒绝，不把所有异常都交给客户端阻断。。这些动作需要和 release gate、verdict API、support bundle、feedback label、rollback action 配合，而不是停在单个移动端函数。

复盘指标建议：只验签为什么挡不住二次打包：Android 包 lineage 必须接上运行时证据 应持续跟踪命中率、版本分布、渠道分布、失败原因、证据新鲜度、服务端解释结果、人工复核结果、误报申诉率和策略回滚次数。指标能帮助团队判断问题来自保护缺失、接入错误、渠道差异、外部材料不足还是客户策略误配。

维护建议：后续如果继续扩展“只验签为什么挡不住二次打包：Android 包 lineage 必须接上运行时证据”这个主题，应优先补充新的脱敏案例、门禁失败样例、服务端字段变化和客户反馈闭环，而不是重复产品介绍。每一次更新都要重新扫描敏感信息、重复段落、内链可用性和外部参考链接，确保 GitHub Pages 只沉淀可长期公开的工程材料。

### 2. 主 App 能启动不代表扩展安全：iOS Framework、Extension 和 App Clip 怎么做独立门禁？：从事实到工程闭环

这篇文章的主线是“主 App 能启动不代表扩展安全：iOS Framework、Extension 和 App Clip 怎么做独立门禁？”，属于 御盾 iOS 的单一风险边界。它不把多个产品揉在一起，也不把平台差异抹平。对长期文档来说，最重要的是保留“事实、判断、动作、边界”的链路：事实来自脱敏资料，判断来自工程验收，动作落在发布门禁和客户后端，边界保护源码、密钥、设备、路径、客户信息和可复现攻击细节。

案例摘要：某企业 App 的主二进制经过保护后可以安装并启动，团队据此准备对外交付。但脱敏验收清单要求继续清点嵌入 Framework、Share Extension、Notification Extension、App Clip 和 Watch companion。检查结果显示，主 App 的签名状态不能自动代表每个嵌入目标的 entitlements、平台 slice、MinimumOSVersion 和签名完整性。 这类问题在 iOS 上很常见：业务入口可能分散在扩展、轻 App、后台能力和共享 framework 里，攻击者也可能从权限更宽、保护更弱或分发链更复杂的目标进入。只做主 App smoke，等于把一组二进制产物压缩成一个结论，无法解释真实风险。 御盾 iOS 的验收路线，是把每个嵌入目标当成独立保护对象：结构清点、签名链、profile 覆盖、entitlements 差异、Mach-O 平台 slice、真机安装、真实启动、受保护路径命中和崩溃摘要都要有脱敏证据。缺少证据时写 BLOCKED，比写一个没有根据的 PASS 更有工程价值。

证据解释：证据 1 来自签名安装兼容验收模板，观察到真实验收必须使用同一次保护构建和同一签名流程产生的 protected IPA、签名证书摘要、provisioning profile、entitlements 摘要、Info.plist 摘要和 release manifest。，因此支撑“iOS 加固验收必须绑定同一产物链，不能用未签名中间包、模拟器包或人工说明替代。”。公开时必须遵守边界：不公开证书摘要、profile UUID、TeamIdentifier、设备覆盖、私钥或真实 IPA 散列。 证据 2 来自签名安装兼容验收模板，观察到验收模板要求清点主 App、framework、extension、App Clip、Watch companion 和 debug artifact。，因此支撑“主 App 能启动不代表嵌入目标安全，所有目标都要独立做结构、签名、entitlements 和平台 slice 检查。”。公开时必须遵守边界：只公开目标类别和验收维度，不输出 bundle id、文件树、签名链或客户 profile。 证据 3 来自签名安装兼容验收模板，观察到没有 protected IPA、签名材料或真机时，所有工具项必须为 NOT_RUN，商业 ready 必须为 false。，因此支撑“发布系统要 fail-closed，不得把缺失证据包装成“通过”。”。公开时必须遵守边界：只公开状态口径，不输出内部 gate JSON、工具命令和工作区路径。 证据 4 来自真机门禁合同，观察到真实通过必须覆盖安装、启动、主路径、受保护函数或资源命中、dispatcher/binding 命中、短窗口 materialization、退出码或崩溃摘要和设备矩阵。，因此支撑“iOS 加固验收不能停在安装成功，必须证明受保护路径在真实设备上命中。”。公开时必须遵守边界：不公开 runner、设备标识、崩溃原文、受保护函数名或资源名。 证据 5 来自真机门禁合同，观察到缺少任一真实输入、签名安装 gate、受控 runner、实体设备或必需 smoke 子项时，producer 必须输出 BLOCKED，并保持 commercialReady=false。，因此支撑“门禁结论应可审计、可阻断、可回滚，不能用静态分析结果替代真机运行证据。”。公开时必须遵守边界：只公开 fail-closed 原则，不输出 runner session、设备 farm 记录或证据包 id。 证据 6 来自iOS 隐私与 App Attest 边界，观察到public iOS SDK 只能公开 support/status/challenge-required 等 evidence-only 状态，不持有 Apple 私有材料、raw token/assertion 或客户端业务裁决 API。，因此支撑“嵌入目标验收还要检查隐私与 attestation 边界，避免扩展目标错误携带私有验证能力。”。公开时必须遵守边界：不公开 Apple key、Team allowlist、raw token、assertion、SecretKey 或策略权重。

工程路径：嵌入目标 inventory 负责先列出主 App、Framework、Extension、App Clip、Watch companion 和调试残留，再逐个绑定签名、权限、平台和保护状态。；签名与权限差异 负责对比实际 entitlements 与 profile 覆盖关系，避免扩展目标拥有不必要权限，也避免保护重签后权限丢失。；真机 smoke 证据 负责把安装、启动、主路径、受保护路径、dispatcher/binding、短窗口材料和退出状态写成结构化证据。；fail-closed 门禁 负责没有真实 protected IPA、签名材料、受控 runner 或真机证据时只允许 NOT_RUN/BLOCKED，不允许商业 ready。。这些动作需要和 release gate、verdict API、support bundle、feedback label、rollback action 配合，而不是停在单个移动端函数。

复盘指标建议：主 App 能启动不代表扩展安全：iOS Framework、Extension 和 App Clip 怎么做独立门禁？ 应持续跟踪命中率、版本分布、渠道分布、失败原因、证据新鲜度、服务端解释结果、人工复核结果、误报申诉率和策略回滚次数。指标能帮助团队判断问题来自保护缺失、接入错误、渠道差异、外部材料不足还是客户策略误配。

维护建议：后续如果继续扩展“主 App 能启动不代表扩展安全：iOS Framework、Extension 和 App Clip 怎么做独立门禁？”这个主题，应优先补充新的脱敏案例、门禁失败样例、服务端字段变化和客户反馈闭环，而不是重复产品介绍。每一次更新都要重新扫描敏感信息、重复段落、内链可用性和外部参考链接，确保 GitHub Pages 只沉淀可长期公开的工程材料。

### 3. 没有 GMS 也不能假通过：大陆 Android attestation fallback 怎么分层？：从事实到工程闭环

这篇文章的主线是“没有 GMS 也不能假通过：大陆 Android attestation fallback 怎么分层？”，属于 守界 Android 的单一风险边界。它不把多个产品揉在一起，也不把平台差异抹平。对长期文档来说，最重要的是保留“事实、判断、动作、边界”的链路：事实来自脱敏资料，判断来自工程验收，动作落在发布门禁和客户后端，边界保护源码、密钥、设备、路径、客户信息和可复现攻击细节。

案例摘要：国内 Android 生态里，同一个 App 可能同时运行在带 GMS 的设备、无 GMS 的厂商系统、企业管控设备、社区 ROM、云手机、模拟器和多开环境中。一个只依赖 Play-style attestation 的设备证据系统，在大陆渠道会遇到大量 fallback；如果把 fallback 当作成功，运营看板会高估可信覆盖率。 守界 Android 的分层路线是：OEM attestation、binding-without-attestation、no_attestation、transport diagnostic、native evidence 和 server verdict 分开记录。每个证据都有 provider、status、failure reason、freshness、source、trust 和 provenance。客户后端看到的是结构化证据，不是一个难以解释的“可信/不可信”。 真实生产 ready 还需要私有 OEM bridge、私有 verifier、可信 provider allowlist 和下游 decisioning。没有这些材料时，正确状态是 blocked 或 fallback observation，而不是把 debug fake path 写成生产通过。这样做看起来更保守，但能避免设备图谱从第一天开始就混入不可解释的“假可信”。

证据解释：证据 1 来自大陆非 GMS 发布门禁，观察到门禁适用于目标设备缺少 Google Play 服务、依赖 OEM attestation 或暂时允许 no_attestation fallback 的场景。，因此支撑“大陆 Android 设备证明必须先判断 provider 条件，不能照搬单一 GMS 路线。”。公开时必须遵守边界：不公开内部命令、AppKey、设备、token、环境变量或运行记录路径。 证据 2 来自大陆非 GMS 发布门禁，观察到最小闭环要求静态 build gate、样例大陆构建安装路径、emulator attestation summary E2E 和 fallback path validation。，因此支撑“debug 流程可以验证链路，但不能替代真实 staging gate；每个阶段都要写清证据状态。”。公开时必须遵守边界：只公开门禁阶段，不输出脚本参数、日志、设备编号或样例凭据。 证据 3 来自大陆非 GMS 发布门禁，观察到真实 staging gate 要求私有 OEM bridge、私有后端 verifier、可信 provider allowlist、oem_attested 握手和下游 decisioning 反映 OEM posture。，因此支撑“OEM attestation 生产 ready 依赖外部私有材料，缺失时应标记 blocked_external_input。”。公开时必须遵守边界：不公开 verifier、allowlist、provider 凭据、租户配置或生产 rollout 数据。 证据 4 来自大陆非 GMS 发布门禁，观察到停止条件包括 OEM verifier 缺失、allowlist 为空、只有 fake path 通过、失败 OEM 被静默当作 no_attestation、fallback 流量不可量化。，因此支撑“fallback 不是成功状态，必须在报表中与 verified traffic 区分。”。公开时必须遵守边界：只公开停止条件类别，不输出真实流量、客户数据或运营看板字段。 证据 5 来自设备身份协议草图，观察到Android SDK 维护 installId、resolvedDeviceId 和 fingerprintHash 等身份层，但 canonical identity 归服务端所有，客户端 canonical 只能作为 claim。，因此支撑“attestation 只是设备证据之一，不能替代服务端身份权威和历史图谱。”。公开时必须遵守边界：不公开 raw Android ID、raw installId、签名 header、端点或完整 BoxId。 证据 6 来自后端包装契约，观察到wrapper 只能在服务端签名请求、查询 verdict/evidence、提交 feedback 和脱敏；不能放进 Android App，也不能输出业务 allow/reject/block。，因此支撑“大陆 attestation 的解释和业务动作必须在客户后端完成，SDK 只交付 evidence。”。公开时必须遵守边界：不公开 SecretKey、Authorization、nonce、body hash、生产服务配置或策略权重。

工程路径：provider 分层 负责把 GMS、OEM、binding fallback 和 no_attestation 分成不同 provider/status，不在 UI 或日志中合并成一个通过状态。；fallback 量化 负责在服务端记录 fallback 占比、渠道、版本、设备类别、失败原因和灰度策略，避免运营只看到总量。；identity 权威边界 负责客户端 installId 和 fingerprintHash 是证据，canonical identity 与 riskTags 只能由服务端产生。；后端 wrapper 安全 负责wrapper 只在服务端持有凭据和签名能力，Android App 内不得嵌入 SecretKey 或最终业务策略。。这些动作需要和 release gate、verdict API、support bundle、feedback label、rollback action 配合，而不是停在单个移动端函数。

复盘指标建议：没有 GMS 也不能假通过：大陆 Android attestation fallback 怎么分层？ 应持续跟踪命中率、版本分布、渠道分布、失败原因、证据新鲜度、服务端解释结果、人工复核结果、误报申诉率和策略回滚次数。指标能帮助团队判断问题来自保护缺失、接入错误、渠道差异、外部材料不足还是客户策略误配。

维护建议：后续如果继续扩展“没有 GMS 也不能假通过：大陆 Android attestation fallback 怎么分层？”这个主题，应优先补充新的脱敏案例、门禁失败样例、服务端字段变化和客户反馈闭环，而不是重复产品介绍。每一次更新都要重新扫描敏感信息、重复段落、内链可用性和外部参考链接，确保 GitHub Pages 只沉淀可长期公开的工程材料。

### 4. Bootloader 解锁不是封禁理由：Android ROM 证据为什么要和 Root/模拟器拆开？：从事实到工程闭环

这篇文章的主线是“Bootloader 解锁不是封禁理由：Android ROM 证据为什么要和 Root/模拟器拆开？”，属于 守界 Android 的单一风险边界。它不把多个产品揉在一起，也不把平台差异抹平。对长期文档来说，最重要的是保留“事实、判断、动作、边界”的链路：事实来自脱敏资料，判断来自工程验收，动作落在发布门禁和客户后端，边界保护源码、密钥、设备、路径、客户信息和可复现攻击细节。

案例摘要：某业务把 bootloader unlocked 写成直接封禁条件，结果在技术社区用户、企业测试设备和部分解锁维修设备上产生大量申诉。复盘发现，这些设备未必存在 hook 或自动化作弊；真正高风险的是 bootloader unlocked 与 root manager、可执行匿名映射、模拟器证据、异常账号迁移和高价值业务动作同时出现。 守界 Android 的处理方式是拆证据族。ROM 事实说明系统来源和构建形态，bootloader 说明验证链状态，root manager 说明权限扩展可能性，模拟器说明运行环境形态，hook/debug 说明运行期干预，server verdict 说明这些事实在当前业务动作里的解释。拆开之后，客户后端可以对低价值浏览做观察，对登录做挑战，对提现或游戏结算做更严格校验。 这种模型对误报治理也更友好。用户申诉时，后台能解释是 ROM 事实、root 事实、模拟器事实还是 hook 事实触发了动作；工程团队能定位是采集、上报、verdict、客户策略还是运营误配；安全团队能把确认作弊和确认误报写回反馈闭环。

证据解释：证据 1 来自ROM/Bootloader 矩阵，观察到矩阵明确要求 public SDK 只采集和报告 custom ROM、root、bootloader、emulator posture 的 evidence，不在客户端做本地 allow/deny。，因此支撑“ROM 证据必须进入服务端解释，不能在客户端直接封禁。”。公开时必须遵守边界：不公开测试设备、完整属性、采集脚本参数、artifact 路径或原始输出。 证据 2 来自ROM/Bootloader 矩阵，观察到矩阵要求把 ROM/bootloader facts 与 emulator、root、hook、debug evidence 分开，覆盖 custom AOSP、社区 ROM、GSI/DSU、bootloader unlocked 和 clean OEM control。，因此支撑“不同证据族风险含义不同，合并会导致误报、策略不可解释和客户申诉困难。”。公开时必须遵守边界：只公开证据族类别，不输出完整 build fingerprint、vendor fingerprint、boot image fingerprint 或包名。 证据 3 来自ROM/Bootloader 矩阵，观察到posture collector 的输出默认脱敏，不保存完整 ADB serial、Android ID、build fingerprint、bootloader version 或 root manager package name。，因此支撑“设备证据要能做关联和排障，但不能把原始标识当成公开稳定身份。”。公开时必须遵守边界：只公开脱敏原则，不输出 hash 前原文、完整指纹或设备材料。 证据 4 来自ROM/Bootloader 矩阵，观察到derivedEvidence 只覆盖 ROM、bootloader、build-channel、GSI/Treble 和 root-manager posture；空值不证明设备是干净物理机。，因此支撑“没有命中某类 ROM 事实不等于低风险，还需要模拟器、云手机、hook 和 server verdict 补充。”。公开时必须遵守边界：不公开具体规则、root manager 明细或内部补充证据。 证据 5 来自设备身份协议草图，观察到nativeFindingIds、nativeFactTags 和 nativeHighestSeverity 是 evidence，只有 server verdict policy 分类后才成为 riskTags。，因此支撑“本地 native facts 不应直接触发客户业务封禁，必须经过服务端 policy 和业务场景解释。”。公开时必须遵守边界：不公开 native payload、映射规则、检测实现或 endpoint。 证据 6 来自后端包装契约，观察到wrapper 不能输出 allow/reject/block，最终 business decision 不属于 SDK 或 wrapper 的职责。，因此支撑“bootloader、ROM、root 和模拟器证据应由客户后端按登录、支付、游戏结算、企业数据导出等动作分级处理。”。公开时必须遵守边界：不公开客户策略、权重、反馈标签、审核台字段或 SecretKey。

工程路径：证据族拆分 负责verified boot、bootloader、verity、build channel、GSI/Treble、ROM hint、root manager、emulator、hook/debug 必须分开记录。；脱敏采集 负责完整 serial、Android ID、完整指纹和 raw package name 不进入公开报告，短 hash 只做关联线索，不做身份权威。；server verdict 负责native facts 和 environment facts 上传后由服务端生成 riskTags，客户端不直接做业务动作。；误报复盘 负责把用户申诉、人工复核、业务损失和策略回滚写回证据图谱，避免长期堆叠粗暴封禁。。这些动作需要和 release gate、verdict API、support bundle、feedback label、rollback action 配合，而不是停在单个移动端函数。

复盘指标建议：Bootloader 解锁不是封禁理由：Android ROM 证据为什么要和 Root/模拟器拆开？ 应持续跟踪命中率、版本分布、渠道分布、失败原因、证据新鲜度、服务端解释结果、人工复核结果、误报申诉率和策略回滚次数。指标能帮助团队判断问题来自保护缺失、接入错误、渠道差异、外部材料不足还是客户策略误配。

维护建议：后续如果继续扩展“Bootloader 解锁不是封禁理由：Android ROM 证据为什么要和 Root/模拟器拆开？”这个主题，应优先补充新的脱敏案例、门禁失败样例、服务端字段变化和客户反馈闭环，而不是重复产品介绍。每一次更新都要重新扫描敏感信息、重复段落、内链可用性和外部参考链接，确保 GitHub Pages 只沉淀可长期公开的工程材料。

### 5. App Attest 接上了也不能在客户端判定：iOS 私有 verifier 为什么必须留在后端？：从事实到工程闭环

这篇文章的主线是“App Attest 接上了也不能在客户端判定：iOS 私有 verifier 为什么必须留在后端？”，属于 守界 iOS 的单一风险边界。它不把多个产品揉在一起，也不把平台差异抹平。对长期文档来说，最重要的是保留“事实、判断、动作、边界”的链路：事实来自脱敏资料，判断来自工程验收，动作落在发布门禁和客户后端，边界保护源码、密钥、设备、路径、客户信息和可复现攻击细节。

案例摘要：某 iOS 团队接入 App Attest 后，计划在 App 内把 assertion 成功直接映射成“可信设备”。安全评审指出，这个做法把 verifier、replay protection、Team/bundle allowlist 和业务策略的边界全部前移到客户端，会制造新的攻击目标，也会把隐私与凭据材料推向公开包。 守界 iOS 的 public SDK 只做 evidence-only：记录 support status、request status、challenge binding status、transport diagnostics 和 BoxId hint；private server 负责 challenge、replay、key registration、assertion verification、tenant policy、case review 和最终业务动作。这样即使客户端被 Hook，攻击者也拿不到生产 verifier 能力。 客户后端拿到 BoxId 和 evidence report 后，再结合账号、会话、业务动作、历史反馈和设备图谱决定处理方式。低价值访问可以观察，高价值支付可以要求新鲜 challenge，企业数据导出可以要求完整证据链。App Attest 很重要，但它应该在服务端证据链里发挥作用。

证据解释：证据 1 来自App Attest 边界文档，观察到public iOS SDK 只能收集 DeviceCheck support、DeviceCheck token request status、App Attest support、key generation status、assertion status 和 challenge binding status 等 evidence-only 状态。，因此支撑“客户端看到的是能力和集成状态，不是设备可信的最终判定。”。公开时必须遵守边界：不公开 raw token、raw assertion、challenge、key id、Team 或 verifier request/response。 证据 2 来自App Attest 边界文档，观察到缺少 server challenge 时，token、key generation、assertion 等状态应保持 not_requested 或 server_challenge_required。，因此支撑“没有服务端挑战就不能宣称 App Attest 完整闭环，更不能在客户端产生通过结论。”。公开时必须遵守边界：只公开状态语言，不输出挑战格式、replay ledger、client-data hash 或 verifier 细节。 证据 3 来自App Attest 边界文档，观察到server challenge、replay protection、key registration、assertion verification、Apple key、Team/bundle allowlist、verifier credential、tenant policy 和最终业务动作都属于私有服务端职责。，因此支撑“private verifier 必须留在后端，public SDK 不能携带生产验证材料。”。公开时必须遵守边界：不公开 Apple 私有材料、证书、profile、策略权重、case review 规则或运维 UI。 证据 4 来自iOS 隐私清单，观察到公开 SDK 收集 app/auth context、install identity、device context、runtime evidence、attestation capability、transport diagnostics 和 response diagnostics，字段以 hash、hint、summary、status 为主。，因此支撑“设备证明要在隐私最小化边界内工作，support bundle 也不能泄露原始标识。”。公开时必须遵守边界：不公开 raw IDFV、raw keychain id、完整 BoxId、raw AppKey、raw token、assertion 或 SecretKey。 证据 5 来自iOS 隐私清单，观察到公开 SDK 不提供 allow/reject/block/jailbreak/Frida/tamper/risk-level API。，因此支撑“App Attest 结果应交给客户后端解释，不能被 App 内本地业务逻辑直接消费成封禁按钮。”。公开时必须遵守边界：只公开职责边界，不公开客户业务策略或私有 detector。 证据 6 来自跨平台后端契约，观察到iOS 与 Android/Web 共用 BoxId、/v1/verdict、evidence report、Device Evidence Graph 和 feedback 模型。，因此支撑“App Attest 状态要进入统一证据图谱和反馈闭环，而不是形成 iOS 孤岛。”。公开时必须遵守边界：不公开 SecretKey、生产部署、租户策略、完整标识或反馈台字段。

工程路径：public SDK 状态模型 负责公开包只表达 support、not_requested、server_challenge_required、transport hint 等低信任状态。；private verifier 边界 负责challenge、replay、key registration、assertion verification、Apple material、tenant policy 和最终动作全部留在后端。；隐私清单约束 负责raw IDFV、raw keychain id、完整 BoxId、raw token/assertion、AppKey/SecretKey 和 Apple 私有材料不进入公开包。；跨平台 evidence graph 负责App Attest 状态与 Android/Web 设备证据共用 verdict、report、graph 和 feedback，避免孤岛。。这些动作需要和 release gate、verdict API、support bundle、feedback label、rollback action 配合，而不是停在单个移动端函数。

复盘指标建议：App Attest 接上了也不能在客户端判定：iOS 私有 verifier 为什么必须留在后端？ 应持续跟踪命中率、版本分布、渠道分布、失败原因、证据新鲜度、服务端解释结果、人工复核结果、误报申诉率和策略回滚次数。指标能帮助团队判断问题来自保护缺失、接入错误、渠道差异、外部材料不足还是客户策略误配。

维护建议：后续如果继续扩展“App Attest 接上了也不能在客户端判定：iOS 私有 verifier 为什么必须留在后端？”这个主题，应优先补充新的脱敏案例、门禁失败样例、服务端字段变化和客户反馈闭环，而不是重复产品介绍。每一次更新都要重新扫描敏感信息、重复段落、内链可用性和外部参考链接，确保 GitHub Pages 只沉淀可长期公开的工程材料。

## 统一发布检查

- 每篇文章正文不少于 10000 个中文字符。
- 每篇文章都有至少 5 条“事实依据与脱敏证据”。
- 每篇文章只聚焦单一产品、单一平台或单一场景。
- 外部平台稿按平台重写结构和开头，不直接堆叠自有站全文。
- GitHub Pages 只同步公开 Markdown、索引和静态站文件。
