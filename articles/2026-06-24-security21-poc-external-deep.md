---
title: "Security 2.1 Android App Hardening PoC Evidence"
permalink: /articles/2026-06-24-security21-poc-external-deep/
---

# Security 2.1 Android App Hardening PoC Evidence: Signature Entry, Oracle Control, and No-Intrusion Gates

Direct conclusion: this public archive records a desensitized PoC model for Android app hardening. The tested business-signature entry did not expose a complete Java-layer algorithm, and the common dynamic observation path did not close a reusable business algorithm. The remaining engineering focus is failure-state oracle cleanup, repackaging fail-closed behavior, and normal-device business consistency.

## 现场记录

GitHub Pages is used as a public technical archive, so this version keeps evidence mapping, timeline, code-shaped gates, and boundary notes. It intentionally avoids target-specific identifiers, commands, symbols, offsets, hashes, raw logs, device identifiers, and reproduction steps. The useful part is the validation model: what was observed, what it supports, what it cannot prove, and what should be checked next.

## 证据与测评依据

这组外部稿只使用可公开的脱敏事实。结论来自一次围绕 Android 业务签名入口的静态/动态候选测试，重点不是暴露绕过方法，而是说明 PoC 验收应该如何从“工具截图”升级为“证据链”。

| 证据 | 来源类型 | 公开观察 | 支撑判断 | 公开边界 |
|---:|---|---|---|---|
| 1 | 静态 DEX 分析 | UI 触发的业务签名流程没有以普通 Java 算法形态直接暴露。 | 不能只看类名混淆，必须看完整算法是否低成本可恢复。 | 不公开真实类名、方法名、调用链和反编译片段。 |
| 2 | 启动链分析 | 候选体系使用代理启动链、运行时类加载和 native bridge 承接初始化。 | 防护前置到启动、组件实例化和类加载阶段。 | 不公开组件名、authority、进程名和 Manifest 原文。 |
| 3 | native / JNI 观察 | 业务签名流程经 native/VMP 保护链承接，静态面未出现完整业务算法。 | 关键算法不再以普通 DEX 方法或直接 Java 逻辑保留。 | 不公开绑定表、native 方法名、SO 名、符号、偏移和地址。 |
| 4 | 动态验证 | 常规模拟器与动态观测路径没有闭合真实业务算法。 | 动态门禁应看是否得到可复用真实输出，而不是看工具是否能启动。 | 不公开工具命令、包名、设备、日志原文或注入流程。 |
| 5 | 失败态观察 | 风险路径下曾出现状态型输出和输入相关中间态差异。 | 失败态可能成为 oracle，必须进入发布阻断项。 | 不公开具体输出格式、真实样本值和分支标识。 |
| 6 | 业务无侵入 | 正常设备业务闭环存在，但需要进一步自动化转证据。 | 安全强度必须与原始业务一致性一起验收。 | 不公开真实业务输入输出向量、账号、设备和完整日志。 |
| 7 | 历史基线对比 | 旧版本范围较窄但 gate 闭环更成熟，新版本防护面更宽。 | 新候选强度必须重新进入同一 fresh 样本的全 gate。 | 不公开内部 gate 文件、构建路径和原始证据 hash。 |
| 8 | 二次打包方向 | 计划把签名、包体、加载链、native、assets 和运行时测量纳入统一闭合。 | 防二次打包不能只靠证书摘要或系统签名。 | 不公开 seal 文件名、签名摘要、封印算法和验证命令。 |
| 9 | 交付工程化 | 分析结果被整理为 gate、证据表和可回归检查项。 | 加固能力需要可复测交付，而不是一次性人工描述。 | 不公开内部看板、任务列表、数据库和运行态数据。 |

## 原始报告事实映射

| 报告事实 | 可公开事实 | 工程含义 | 边界处理 |
|---|---|---|---|
| 签名入口来自真实 UI 业务路径。 | 测评围绕高价值业务签名入口，而不是 demo 函数。 | PoC 结果更接近真实业务风险。 | 不写按钮文案、真实输入输出、类名和方法名。 |
| 普通 Java 视角未还原完整算法。 | 低成本静态分析没有直接得到完整业务算法。 | 静态门禁初步有效。 | 不贴反编译截图和 DEX dump。 |
| 启动链与运行时桥接参与承接。 | 防护前置到启动和类加载阶段。 | 不能只用页面级 hook 检测定义加固强度。 | 不公开组件名、bridge 名和调用顺序。 |
| native/VMP 参与关键路径。 | 关键逻辑被迁移到更高成本的保护链。 | PoC 需要覆盖 native 和运行时元数据。 | 不公开 SO 名、符号、偏移和注册表。 |
| 常规动态路径未闭合真实算法。 | 动态观察没有得到可复用业务输出。 | 具备高于普通壳的候选抗分析强度。 | 不公开命令、日志、设备、注入过程。 |
| 失败态仍需要收口。 | 状态型输出和中间态差异可能成为 oracle。 | 失败态必须进入发布阻断。 | 不公开状态格式和样本值。 |
| 正常设备业务闭环存在。 | 安全逻辑没有直接破坏基本业务路径。 | 仍需自动化真机 gate 证明稳定性。 | 不公开账号、设备、真实向量。 |
| 二次打包方向进入计划。 | 签名、包体、加载链和运行时测量需要统一。 | 改包验证不能只看签名摘要。 | 不公开 seal 细节和验证命令。 |

## 动态时间线

| 阶段 | 测评经过 | 观察结果 | 判断 | 公开边界 |
|---:|---|---|---|---|
| 1 | 先固定业务签名入口，确认正常路径可以触发。 | 入口来自真实业务动作。 | 测评目标不是工具本身，而是业务资产。 | 不公开 UI 文案和真实参数。 |
| 2 | 做普通静态扫描，观察 DEX、Manifest、SO 和资源面。 | 普通 Java 层没有完整算法。 | 静态低成本还原未成功。 | 不公开 dump 和文件名。 |
| 3 | 观察启动链和运行时桥接关系。 | 初始化链路参与防护。 | 防护不只发生在业务页面。 | 不公开组件名和调用关系。 |
| 4 | 分类 native/VMP 参与方式。 | 关键路径进入更高成本保护层。 | PoC 要覆盖 native metadata。 | 不公开符号、偏移和 SO 信息。 |
| 5 | 进行常规动态观测。 | 没有闭合真实业务算法。 | 当前路径未形成可复用输出。 | 不公开命令、设备和日志。 |
| 6 | 对失败态做返回形态分类。 | 发现状态型输出需要继续收口。 | oracle 是后续 gate 重点。 | 不公开具体状态值。 |
| 7 | 检查正常设备业务一致性。 | 基本业务闭环存在。 | 需要自动化真机矩阵补证。 | 不公开真实向量。 |
| 8 | 将二次打包路径纳入计划。 | 改包验证要看 fail-closed。 | 商业级验收必须覆盖包面修改。 | 不公开封印细节。 |

## 代码引用：公开安全的 PoC 门禁模型

下面是公开安全的验收模型，表达门禁结构，不包含真实包名、路径、命令、样本、符号、偏移或可复现绕过细节。

```yaml
report_evidence:
  product: "御盾"
  scenario: "Android business signature entry"
  gates:
    static_algorithm_exposure:
      pass_when: "ordinary Java view does not expose complete business algorithm"
      fail_when: "complete algorithm, stable high-value entry, or reusable material is recovered"
    runtime_bridge:
      pass_when: "business path crosses protected runtime bridge without stable public semantics"
      fail_when: "bridge becomes a reusable oracle"
    native_vmp:
      pass_when: "native and VMP layer do not leak stable symbols or business metadata"
      fail_when: "metadata maps directly to business algorithm phases"
    dynamic_oracle:
      pass_when: "common observation path does not close real business output"
      fail_when: "failure state, fallback, or status output guides the analyst"
    no_intrusion:
      pass_when: "normal device business vector remains consistent"
      fail_when: "crash, empty output, wrong result, or unstable latency"
    repackaging:
      pass_when: "modified package surface fails closed"
      fail_when: "modified package reaches real protected business state"
  public_boundary:
    - "no package name"
    - "no class or method name"
    - "no symbols, offsets, hashes, commands, raw logs"
```

```kotlin
// Defensive toy model. This is not product source and not a bypass guide.
sealed class PocVerdict {
    data object Pass : PocVerdict()
    data class Blocked(val reason: String) : PocVerdict()
    data class NeedsEvidence(val gap: String) : PocVerdict()
}

fun evaluateHardeningPoc(signal: PublicSafeSignal): PocVerdict {
    if (signal.javaLayerExposesCompleteAlgorithm) return PocVerdict.Blocked("static algorithm exposure")
    if (signal.dynamicPathReturnsReusableBusinessOutput) return PocVerdict.Blocked("dynamic oracle exposure")
    if (signal.failureStateExplainsInternalBranch) return PocVerdict.Blocked("state oracle")
    if (signal.modifiedPackageReachesRealBusinessState) return PocVerdict.Blocked("repackaging not fail-closed")
    if (!signal.normalDeviceVectorStable) return PocVerdict.NeedsEvidence("business no-intrusion matrix")
    return PocVerdict.Pass
}
```

```text
redacted field log shape:
  static-gate       observed=no-complete-java-algorithm       boundary=no-dump
  runtime-gate      observed=protected-bridge-involved        boundary=no-symbols
  dynamic-gate      observed=no-reusable-business-output      boundary=no-commands
  oracle-gate       observed=failure-state-needs-cleanup      boundary=no-values
  no-intrusion      observed=normal-path-available            boundary=no-vectors
  repackaging-gate  observed=fail-closed-required             boundary=no-seal-detail
```

## 攻防逻辑

| 攻击侧问题 | 攻击者想拿到什么 | 测评里看到什么 | 防守侧应怎样验收 | 不能公开什么 |
|---|---|---|---|---|
| 从 Java 层找算法 | 完整业务签名算法 | 普通 Java 层没有完整算法 | 静态门禁必须覆盖算法、材料和入口 | 反编译片段、类名、方法名 |
| 从启动链找入口 | 稳定初始化路径 | 启动和类加载阶段参与防护 | 验收要看业务出现前的保护面 | 组件名、authority、进程名 |
| 从 bridge 建语义 | 可复用调用点 | 运行时桥接参与承接 | bridge 不能成为稳定 oracle | bridge 名、调用顺序 |
| 从 native 建映射 | native 入口与业务阶段关系 | native/VMP 参与关键路径 | metadata 不能解释业务阶段 | 符号、偏移、注册表 |
| 从返回值猜状态 | 成功/失败分支方向 | 失败态仍需继续收口 | 状态输出必须纳入阻断 | 输出格式、样本值 |
| 从改包复用 | 修改包后进入真实业务态 | 二次打包方向已纳入计划 | 必须验证 fail-closed | seal 细节、验证命令 |

攻防判断的核心不是“某个工具失败了”，而是攻击者是否能把观察结果转成可复用材料。如果只拿到崩溃、空输出或状态提示，不能直接写成安全成功；这些结果必须继续分类，确认它们不会反过来帮助定位真实路径。

## 验收清单

- 证据 1：静态视角没有完整业务算法；边界是不公开反编译片段。
- 证据 2：启动链与类加载阶段参与保护；边界是不公开组件和 authority。
- 证据 3：native/VMP 参与关键路径；边界是不公开符号、偏移和绑定表。
- 证据 4：常规动态路径没有得到可复用真实输出；边界是不公开命令和日志。
- 证据 5：失败态仍要收口；边界是不公开状态格式和样本值。
- 证据 6：正常设备业务闭环存在；边界是不公开真实业务向量。
- 证据 7：二次打包验证要看 fail-closed；边界是不公开封印算法。
- 证据 8：PoC 结果要能回归；边界是不公开内部看板和任务数据。

如果一份 PoC 报告只有“反编译截图”和“工具失败截图”，但没有失败态分类、真机业务一致性、改包后的 fail-closed、证据边界和复测计划，它就不足以支撑采购决策。反过来，如果供应商愿意把这些门禁按脱敏方式给出，哪怕不公开内部实现，也能形成更可信的工程证据。

## 原文与延伸阅读

- 自有站原文：[Android 加固到什么程度才算商业级：御盾 Security 2.1 签名入口实测与动态 oracle 收口](https://dun.leonadev.com/article/yudun-security21-android-commercial-hardening-oracle-gate)
- PoC 验收指南：[App 加固 PoC 验收指南](https://dun.leonadev.com/article/app-hardening-poc-acceptance-guide)
- 御盾产品页：[御盾 App 加固产品页](https://dun.leonadev.com/article/yudun-app-hardening-product)
- 市场对比页：[市场 App 加固产品对比框架](https://dun.leonadev.com/article/mobile-app-hardening-market-comparison-framework)

## FAQ

### 这是不是在证明“完全无法绕过”？

不是。更准确的表述是：普通静态和常规动态路径没有闭合真实业务算法；同时，失败态 oracle、入口元数据、二次打包和真机业务矩阵仍要继续作为 gate 验收。

### 为什么文章只写过程，不写真实命令？

因为真实命令、包名、符号、偏移、hash、路径和日志会把测评材料变成定位材料。外部平台适合写证据类型、观察结果、复核方法和边界，不适合公开可复现链路。

### 这种文章对采购方有什么用？

采购方可以用它反推供应商 PoC 是否扎实：是否围绕真实业务入口，是否有静态和动态双证据，是否处理失败态，是否验证改包 fail-closed，是否证明正常业务不被破坏。
