---
title: "Security 2.1 Failure-State Oracle Checklist"
permalink: /articles/2026-06-24-security21-oracle-checklist/
---

# Security 2.1 PoC Field Notes: Failure-State Oracle Checklist for Android Hardening

GitHub Pages 版作为公开归档，写法更像测试记录索引。自有站原文已经给出清单，这里补充外部分发版本：为什么 Android 加固 PoC 不应只证明“反编译失败”，还要证明失败态、fallback、空输出、异常和二次打包路径不会给出稳定可复用的判断信号。

本文复用御盾 Security 2.1 脱敏证据包，只保留可公开的工程判断和防守侧伪代码。

可作为 GitHub Pages 公开技术档案，不包含内部路径、样本细节或可复现攻击步骤。

自有站 canonical 原文：[Android 加固 PoC 别只看能不能反编译：失败态 oracle 验收清单](https://dun.leonadev.com/article/android-hardening-poc-failure-oracle-checklist-security21)

## 证据与测评依据

下面的证据来自 Security 2.1 Android 加固候选体系的脱敏测评。为了让文章保留真实测评感，我保留了“观察到了什么、怎样复核、支撑什么判断、公开边界是什么”四列；为了避免把文章变成可复现材料，我不写真实样本标识、路径、命令、类名、方法名、native 符号、偏移、hash、设备和日志原文。

| 证据 | 观察事实 | 复核方式 | 支撑判断 | 公开边界 |
|---:|---|---|---|---|
| 1 | UI 触发的业务签名流程没有以普通 Java 算法形态直接暴露。 | 静态面按 DEX、字符串、Manifest、assets 和 bridge 语义一起看。 | PoC 不能只看类名是否混淆，要看完整算法是否低成本可恢复。 | 不公开反编译片段、类名、方法名和调用链。 |
| 2 | 候选体系使用代理启动链、运行时类加载和 native bridge 承接初始化。 | 检查启动阶段和组件实例化阶段是否已进入保护链。 | 保护介入点应早于业务页面，而不是等用户点击后才做检测。 | 不公开组件名、authority、进程名和 Manifest 原文。 |
| 3 | 业务签名流程进入 native/VMP 保护链，静态面未出现完整业务算法。 | 观察 Java 入口是否只是桥接，关键语义是否转移到更高成本保护层。 | 关键算法不再以普通 DEX 方法或直接 Java 逻辑保留。 | 不公开 SO 名、符号、偏移、地址和注册细节。 |
| 4 | 常规模拟器与动态观测路径没有闭合真实业务算法。 | 只把“可对任意输入复现真实业务输出”当作动态成功。 | 当前常规路径没有形成可复用业务算法 oracle。 | 不公开工具命令、包名、设备、日志和注入过程。 |
| 5 | 风险路径下出现过状态型输出和输入相关中间态差异。 | 把返回形态分为真实业务、状态、fallback、空输出、异常和未知。 | 失败态本身可能给攻击者指路，不能被当作安全成功。 | 不公开状态格式、样本值和分支标识。 |
| 6 | 正常设备业务闭环存在，但还要自动化转证据。 | 对冷启动、连续计算、前后台切换和重启后的业务一致性做 gate。 | 安全强度不能以破坏真实业务为代价。 | 不公开账号、设备、真实输入输出向量。 |
| 7 | 防二次打包方向包含签名、包体、加载链、native、assets 和运行时测量。 | 改包样本不能只看是否启动，还要看是否进入真实保护态。 | 防重签不能只靠证书摘要或系统签名。 | 不公开 seal 文件名、摘要、封印算法和验证命令。 |
| 8 | Security 1.5 范围更窄但 gate 更成熟，Security 2.x 防护面更宽。 | 将候选强度和发布门禁分开评价。 | 2.1 的重点是把候选能力收敛成可交付证据。 | 不公开内部 gate 文件、构建路径和原始证据 hash。 |
| 9 | 报告材料包含可转门禁的工程化结构。 | 把每个结论写成 GateEvidence，而不是一句口头判断。 | 加固交付需要证据对象，不能只靠截图。 | 不公开内部任务编号、看板数据和运行态材料。 |

## 测评经过：我会按这条线复盘

这部分不是为了炫技，而是为了避免 PoC 结论跳步。很多失败结论并不等于安全结论，很多工具失败也不等于保护闭合。每一步都必须有观察对象、复核方式、判断依据和公开边界。

| 步骤 | 现场动作 | 观察结果 | 判断 | 边界 |
|---:|---|---|---|---|
| 1 | 先固定高价值业务签名入口。 | 入口来自真实业务流程，而不是 demo 方法。 | PoC 应围绕真实资产，不能只测玩具函数。 | 不公开 UI 文案和真实参数。 |
| 2 | 做包面静态检查。 | DEX、SO、assets、Manifest、字符串都进入观察范围。 | 反编译窗口只是静态面的一部分。 | 不公开文件清单和命中内容。 |
| 3 | 反查普通 Java 视角。 | 没有看到完整业务签名算法。 | 静态低成本还原没有闭合。 | 不公开反编译片段。 |
| 4 | 看启动链和运行时类加载。 | 代理启动链与 bridge 参与初始化。 | 保护链前置到业务页面之前。 | 不公开组件和 bridge 名。 |
| 5 | 看 native/VMP 是否参与。 | 关键路径进入更高成本保护层。 | PoC 要覆盖 native metadata 和运行时承载。 | 不公开符号、偏移、SO 名。 |
| 6 | 做常规动态观察。 | 没有得到可复用真实业务输出。 | 当前动态路径未闭合真实算法。 | 不公开命令、设备和日志。 |
| 7 | 单独分类失败态。 | 状态型输出、fallback、空输出、崩溃和未知返回被分开记录。 | 失败态不能被写成防护成功。 | 不公开状态值和样本值。 |
| 8 | 对照正常设备业务。 | 正常业务闭环存在。 | 还需要自动化真机矩阵证明无侵入。 | 不公开真实业务向量。 |
| 9 | 延伸到二次打包。 | 改包防护要覆盖签名、包体、加载链、native、assets 和运行时测量。 | 防重签不是一个摘要校验能解决。 | 不公开改包过程。 |
| 10 | 收敛交付结论。 | 当前适合写成商业候选强度与待收口项。 | 不把半截证据写成满分结论。 | 不公开内部构建和任务数据。 |

## 原始报告事实映射

| 报告事实 | 公开表达 | 支撑判断 | 不能公开的部分 |
|---|---|---|---|
| 测试围绕 UI 触发的业务签名入口展开。 | 评测对象是真实业务入口。 | 高价值业务路径比 demo 方法更能反映商业风险。 | UI 文案、真实参数、类名、方法名。 |
| 静态分析覆盖 DEX、SO、assets、Manifest 和字符串面。 | 静态不是只看 jadx。 | 加固验收要覆盖承载形态和可解释线索。 | 文件名、hash、具体字符串和扫描输出。 |
| Java 层没有完整签名算法。 | 普通 Java 视角没有直接恢复完整业务逻辑。 | 静态低成本还原未闭合。 | 反编译截图和调用链。 |
| 业务逻辑进入 native/VMP。 | 关键路径被转移到更高成本保护链。 | PoC 要覆盖 native 入口和 metadata。 | native 名、符号、偏移、注册表。 |
| 常规动态路径没有复现真实业务输出。 | 动态观察未形成可复用算法结果。 | 当前路径未闭合，但不能绝对化。 | 动态命令、设备、日志、注入流程。 |
| 返回值出现过状态型输出或中间态差异。 | 失败态可能成为 oracle。 | 失败路径必须进入 gate。 | 状态格式、样本值、分支标识。 |
| 正常设备业务闭环存在。 | 正常业务方向正确。 | 需要自动化真机矩阵证明无侵入。 | 真实业务向量和设备信息。 |
| 二次打包方向包含签名、包体、加载链和运行时测量。 | 改包防护要看组合证据。 | 不能只靠签名校验判断。 | seal 细节和验证命令。 |
| 报告给出 Security 2.1 迭代建议。 | 下一阶段重点是 oracle、入口、metadata、改包和真机 gate。 | 候选强度需要工程闭合。 | 内部任务文件和执行路径。 |

## 动态时间线

| 阶段 | 观察目标 | 复核方式 | 工程意义 | 公开边界 |
|---:|---|---|---|---|
| 1 | 业务签名入口。 | 从真实业务输出能否被复现反推保护链。 | PoC 应围绕真实资产。 | 不公开入口文案和向量。 |
| 2 | DEX 静态面。 | 判断普通 Java 层是否保留完整算法。 | 区分混淆和算法迁移。 | 不公开反编译片段。 |
| 3 | 启动链与类加载。 | 观察保护是否在业务页面前介入。 | 判断是否存在前置保护。 | 不公开组件和 authority。 |
| 4 | native/VMP 参与。 | 观察关键路径是否进入更高成本保护链。 | 判断 DEX 方法是否仍是主要承载。 | 不公开符号、偏移和 SO 名。 |
| 5 | 动态返回形态。 | 把输出分为真实业务、状态、fallback、空输出、异常。 | 判断是否形成动态 oracle。 | 不公开命令、日志和真实输出。 |
| 6 | 正常设备业务。 | 验证启动、重启、前后台和连续计算。 | 证明业务无侵入。 | 不公开设备和业务向量。 |
| 7 | 二次打包路径。 | 观察签名、包体、组件、SO/DEX/assets 修改后的结果。 | 判断是否 fail-closed。 | 不公开改包流程。 |
| 8 | 发布证据。 | 把结论写成 GateEvidence 并绑定候选范围。 | 避免拼接不同版本证据。 | 不公开构建和任务数据。 |

## 代码证据：公开安全脚手架

下面的代码块只表达防守侧验收逻辑，不包含真实样本参数，也不能用来复现目标。第一段用于把失败态分类，第二段用于把 PoC 结论写成可归档的证据对象。

```python
# public_safe_return_classifier.py
# Defensive PoC helper: classify public-shaped returns, not real vectors.
from dataclasses import dataclass

@dataclass
class Verdict:
    category: str
    release_gate: str
    reason: str

def classify_return(shape: str) -> Verdict:
    if shape in {"empty", "crash", "fallback"}:
        return Verdict("invalid", "block", "failure does not prove protection")
    if shape in {"state-like", "branch-hint", "intermediate"}:
        return Verdict("oracle-risk", "block", "failure state gives direction")
    if shape == "verified-business-output-on-real-device":
        return Verdict("business-ok", "needs-regression", "normal path remains valid")
    return Verdict("unknown", "manual-review", "do not mark as passed")
```

```yaml
report_evidence:
  product: "御盾"
  assessment: "Security 2.1 Android hardening PoC"
  public_evidence:
    java_layer: "complete business algorithm not observed in ordinary Java layer"
    runtime_chain: "runtime bridge and native/VMP protection chain involved"
    dynamic_path: "common dynamic observation did not close real business output"
    failure_state: "state-like output and intermediate differences require gate"
    delivery: "candidate strength must become same-scope gate evidence"
  not_public:
    - package_name
    - class_or_method_names
    - native_symbols
    - offsets
    - hashes
    - device_ids
    - raw_commands
    - raw_logs
```

```text
GateEvidence
  static_surface: no complete ordinary Java algorithm observed
  runtime_chain: bridge and native/VMP chain observed as involved
  dynamic_result: reusable business output not closed by common path
  failure_state: state-like or branch-like output blocks release
  no_intrusion: normal-device vector must stay stable
  repackaging: modified package must fail closed
  verdict: commercial candidate; final closure requires full gate
```

## 攻防逻辑：别把失败看成通过

| 攻击者看到的现象 | 可能继续怎么做 | 防守侧应如何判断 | 发布动作 |
|---|---|---|---|
| 普通 Java 层没有完整算法 | 转向 bridge、native metadata、字符串残留和服务端请求关系 | 只能证明静态低成本路径未闭合 | 继续 native 与动态 gate |
| 动态路径没有真实输出 | 继续观察空输出、fallback、崩溃和状态差异 | 不能直接算安全成功 | 做失败态分类 |
| 状态型输出稳定出现 | 用状态差异判断保护链内部阶段 | 这是 oracle 风险 | 阻断发布 |
| 正常设备业务可用 | 尝试寻找风险环境和改包路径差异 | 只能证明方向正确 | 转自动化真机矩阵 |
| 改包后随机失败 | 继续找可解释的 fail-open 点 | 随机失败不是工程闭合 | 做 fail-closed gate |
| 多版本证据拼在一起 | 利用版本差异绕过结论 | 证据必须同一候选范围 | 禁止拼接交付 |

我会把这里的核心判断写得很保守：静态看不到算法值得记录，动态没有闭合真实输出也值得记录，但只有当失败态不给方向、正常路径不受影响、改包路径 fail-closed、证据来自同一候选范围时，才适合写成商业级通过。

## 验收清单

- 证据 1：PoC 是否从真实业务签名入口开始，而不是从 demo 方法开始。
- 证据 2：普通 Java 层是否仍能恢复完整业务算法。
- 证据 3：启动链、类加载、native/VMP 是否被纳入观察。
- 证据 4：动态路径是否能对任意输入复现真实业务输出。
- 证据 5：失败态是否会稳定泄露 VM 状态、分支状态或中间状态。
- 证据 6：空输出、崩溃、fallback 是否被排除在“通过”之外。
- 证据 7：正常设备业务向量是否稳定，是否有自动化矩阵。
- 证据 8：二次打包是否证明 fail-closed，而不是随机失败。
- 证据 9：所有结论是否来自同一候选范围。
- 证据 10：外部材料是否清楚写出公开边界，避免泄露可复现链路。

## 不公开的内容

这类测评文章必须保留证据，但不能把内部测评变成定位材料。真实包名、类名、方法名、native 名、符号、偏移、hash、路径、设备、命令、日志原文、真实输入输出向量、patch 点和完整复现流程都不应该出现在外部平台。能公开的是测评顺序、证据类型、脱敏观察、工程判断、阻断条件和下一步 gate。

## 原文与延伸阅读

- 自有站原文：[Android 加固 PoC 别只看能不能反编译：失败态 oracle 验收清单](https://dun.leonadev.com/article/android-hardening-poc-failure-oracle-checklist-security21)
- PoC 验收指南：[App 加固 PoC 验收指南](https://dun.leonadev.com/article/app-hardening-poc-acceptance-guide)
- 产品页：[御盾 App 加固产品页](https://dun.leonadev.com/article/yudun-app-hardening-product)
- 选型页：[Android/iOS App 加固选型页](https://dun.leonadev.com/article/android-ios-app-hardening-product-selection)
- 市场对比：[市场 App 加固产品对比框架](https://dun.leonadev.com/article/mobile-app-hardening-market-comparison-framework)

## FAQ

### 这是不是证明任何逆向都不可能？
不是。公开结论应当保守：普通静态路径和常规动态路径没有闭合真实业务算法，但失败态 oracle、入口可定位性、二次打包和真机自动化仍需要继续收口。

### 为什么要把失败态写得这么重？
因为攻击者不一定需要马上拿到算法。只要状态差异能告诉他自己走到了哪一层，他就能继续缩小分析范围。失败态如果稳定、可分类、可复用，就应该进入阻断项。

### 为什么文章贴代码但不贴真实脚本？
代码用于说明防守侧验收逻辑，不用于复现目标。公开安全代码应该表达分类方法、门禁结构和证据对象，不应该包含目标参数、真实命令或内部实现。

### 企业做验收时最应该问什么？
不要只问“能不能反编译”。更应该问：真实业务入口是否被保护，动态路径是否无法闭合，失败态是否不给方向，正常业务是否稳定，改包路径是否 fail-closed，证据是否来自同一候选范围。
