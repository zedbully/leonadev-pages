---
title: "御盾 Security 2.1 Android 商业级加固测评"
permalink: /articles/2026-06-23-yudun-security21-commercial-hardening-oracle-gate/
---

# Android 加固到什么程度才算商业级：御盾 Security 2.1 签名入口实测与动态 oracle 收口

## 现场结论

这次测评不从“壳强不强”这种泛问题开始，而是从一个更容易暴露真实强度的点开始：一个 UI 触发的业务签名入口，在静态反编译、动态观测、native 注册、返回值分类和二次打包风险面前，是否还能被还原成可复用的真实算法。

结论先写清楚。御盾 Security 2.1 候选体系里，业务签名流程没有在普通 Java 层以完整算法形态出现，而是进入运行时桥接、native/VMP 和加载链；常规模拟器与动态观测路径没有闭合真实业务算法。这个结果说明它已经高于普通混淆和普通壳。但测评也发现，失败态状态输出、输入相关中间态、入口可定位性和 runtime metadata 仍需要继续收口。商业级加固不能把“没还原算法”当作终点，必须证明失败态也不会成为 oracle。

本文只保留公开安全的评测过程和判断模型，不放包名、类名、方法名、真实 native 名、符号、偏移、hash、路径、命令、设备、日志原文、真实业务输入输出向量或完整注入流程。

## 测评经过：从业务入口反推保护链

GitHub Pages 用于保存公开技术记录，因此这里保留完整过程、证据映射、动态时间线和代码化门禁模型，便于后续文章继续引用。

| 步骤 | 现场动作 | 观察到了什么 | 支撑什么判断 | 公开边界 |
|---:|---|---|---|---|
| 1 | 先选定一个 UI 业务签名入口。 | 评测不是从工具能否 attach 开始，而是从业务输出能否复现开始。 | 商业级评测要围绕高价值业务路径。 | 不公开按钮文案、真实输入输出、类名和方法名。 |
| 2 | 静态扫 DEX、Manifest、SO、assets 和字符串。 | 普通 Java 层没有完整签名算法。 | 静态面已经从普通 Java 暴露状态进入保护链状态。 | 不公开文件名、hash、字符串词表和反编译片段。 |
| 3 | 观察启动链、类加载和 native bridge。 | 业务入口经运行时桥接进入 native/VMP。 | 加固不是只做字符串替换，而是把业务入口接入运行时保护链。 | 不公开组件名、bridge 名和调用链。 |
| 4 | 动态观察返回形态。 | 常规动态路径没有得到可对任意输入复现的真实业务算法。 | 工具能看到状态，不等于算法已经还原。 | 不公开动态命令、设备、包名和日志。 |
| 5 | 对失败态做分类。 | 风险路径下存在状态型输出和输入相关中间态差异。 | 失败态可能成为 oracle，必须进入 2.1 收口。 | 不公开状态格式、真实样本值和分支标识。 |
| 6 | 对正常设备路径做边界确认。 | 正常设备业务闭环存在，但需要自动化证据。 | 安全强度和业务无侵入必须同时验收。 | 不公开设备、账号、输入输出和完整记录。 |
| 7 | 对二次打包方向做 gate 规划。 | 2.1 要把签名、包体、加载链、native、assets 和运行时测量统一起来。 | 防重签不能只靠证书摘要或系统签名。 | 不公开 seal 名、签名摘要、封印算法和验证命令。 |

上面的过程有一个关键判断：动态工具没有闭合真实算法，只是阶段性胜利；失败态如果还带有可解释状态，仍然会帮助分析者判断自己是否命中真实路径、fallback 路径或 materializer 分支。商业级加固要做的是让成功路径和失败路径都不可稳定复用。

## 证据与测评依据

| 证据 | 来源类型 | 脱敏观察 | 工程判断 | 公开边界 |
|---:|---|---|---|---|
| 1 | 静态 DEX 分析 | UI 触发的业务签名流程没有以普通 Java 算法形态直接暴露。 | 御盾不是只做普通 Java 混淆，而是把业务入口接入运行时保护链。 | 不公开类名、方法名、调用链和 DEX dump。 |
| 2 | Manifest / 启动链分析 | 候选体系使用代理启动链、运行时类加载和 native bridge 组合承接初始化。 | 防护需要前置到启动、组件实例化和类加载阶段。 | 不公开组件名、authority、进程名和 Manifest 原文。 |
| 3 | native / JNI 观察 | 业务签名流程经 native/VMP 保护链承接，静态面未出现完整业务算法。 | 关键算法不再以普通 DEX 方法或直接 Java 逻辑保留。 | 不公开 JNI 绑定表、native 方法名、SO 名、符号、偏移和注册细节。 |
| 4 | 动态验证 | 常规模拟器与动态观测路径没有闭合真实业务算法。 | 御盾具备高于普通壳和简单混淆的动态抗分析候选强度。 | 不公开动态工具命令、包名、设备、日志原文或注入流程。 |
| 5 | 失败态观察 | 风险路径下曾出现状态型输出和输入相关中间态差异。 | 商业级防护不仅要保护成功路径，也要避免失败态成为 oracle。 | 不公开具体输出格式和真实样本值。 |
| 6 | 业务无侵入 | 正常设备业务闭环存在，但仍需要进一步自动化转证据。 | 安全强度必须和原始业务不变同时验收。 | 不公开真实业务输入输出向量。 |
| 7 | 历史基线对比 | Security 1.5 范围更窄但 gate 闭环更成熟；Security 2.x 防护面更宽。 | 2.1 目标是把更宽防护面收敛成可复测商业闭环。 | 不公开内部 gate 文件、历史构建路径和任务编号。 |
| 8 | 二次打包方向 | 2.1 计划把签名、包体、加载链、native、assets 和运行时测量纳入统一闭合。 | 防二次打包不能只靠证书摘要或系统签名。 | 不公开 seal 文件名、签名摘要和验证命令。 |
| 9 | 交付工程化 | 2.1 计划把分析结果转成 GateEvidence 和自动化门禁。 | 产品能力要靠证据链交付，而不是靠单次人工报告。 | 不公开本地数据库、看板路径和内部任务全量列表。 |

## 原始报告事实映射

| 报告事实 | 公开安全表达 | 支撑的判断 | 不公开内容 |
|---|---|---|---|
| 测试围绕 UI 触发的业务签名计算入口展开。 | 从真实业务入口反推保护链，而不是只看工具输出。 | 高价值业务路径比普通文件扫描更能体现加固强度。 | 不公开按钮文案、真实向量、类名和方法名。 |
| 静态分析未直接恢复完整签名算法。 | 普通 Java 反编译不能直接得到算法主体。 | 业务入口已进入 DEX/native/VMP 保护链。 | 不公开反编译片段、dump 文件和字符串词表。 |
| 动态路径未复现真实业务算法。 | 常规动态观察没有得到可泛化的真实输入输出 oracle。 | 看到状态不等于算法还原。 | 不公开工具命令、日志原文、设备和运行流程。 |
| 风险路径出现状态型输出和中间态差异。 | 失败态仍可能泄露保护层状态。 | Security 2.1 必须把 oracle 清理作为核心目标。 | 不公开状态格式、分支标识和样本值。 |
| 正常设备业务闭环存在。 | 安全增强没有直接破坏正常业务，但还要进入自动化真机 gate。 | 商业级加固必须同时证明安全强度和业务无侵入。 | 不公开设备、账号、业务输入输出。 |
| Security 2.x 防护面更宽但 gate 仍需收敛。 | 候选强度需要转成同一 fresh 样本、同一 hash、全 gate 证据。 | 单次测评不能替代最终发布闭环。 | 不公开内部构建、hash、路径和任务编号。 |

```yaml
report_evidence:
  source: "Security 2.1 redacted Android hardening assessment"
  measurement_scope:
    - business_signature_entry
    - static_algorithm_exposure
    - runtime_bridge
    - native_vmp_chain
    - dynamic_oracle_resistance
    - repackaging_fail_closed_direction
    - business_no_intrusion
  public_evidence:
    java_algorithm: "complete algorithm not exposed in ordinary Java layer"
    dynamic_result: "common dynamic path did not close real business algorithm"
    oracle_risk: "state-like output and intermediate differences require hardening"
    business_path: "normal-device closure exists; automated gate required"
  redaction:
    - no_package_class_method
    - no_native_symbol_offset
    - no_hash_path_command_log
    - no_real_business_vector
```

## 动态时间线

| 阶段 | 观察 | 复核方式 | 结论 | 边界 |
|---:|---|---|---|---|
| 1 | 业务入口被选为主线。 | 以真实业务输出能否复现作为标准。 | 避免把工具失败当安全成功。 | 不公开入口名称。 |
| 2 | 普通 Java 层没有完整算法。 | 静态维度覆盖 DEX、Manifest、SO、assets、字符串。 | 静态直接还原未闭合。 | 不公开 dump 和文件名。 |
| 3 | 运行时桥接和 native/VMP 参与。 | 只看事件类别和链路角色。 | 保护链覆盖运行时上下文。 | 不公开 bridge 名和 native 名。 |
| 4 | 常规动态路径没有真实算法 oracle。 | 返回值按业务输出、状态输出、fallback、无效输出归类。 | 算法未被动态闭合。 | 不公开命令和日志。 |
| 5 | 失败态存在状态差异。 | 将状态型输出列入阻断项。 | oracle 清理是 2.1 核心。 | 不公开具体状态格式。 |
| 6 | 正常设备业务闭环存在。 | 后续转真机自动化 gate。 | 业务无侵入需要证据化。 | 不公开设备和向量。 |
| 7 | 2.1 需要全 gate。 | 同一 fresh 样本、同一 hash、同一设备矩阵复测。 | 候选强度转商业交付。 | 不公开内部构建信息。 |

## 代码引用：防守侧返回值分类

下面是防守侧伪代码，目的是说明测评原则，不是产品实现，也不是攻击脚本。

```python
def classify_public_result(input_family, output_shape):
    if output_shape == "verified_business_output":
        return "business-output"
    if output_shape == "state_like_or_branch_like":
        return "oracle-risk-block"
    if output_shape in {"empty", "crash", "timeout"}:
        return "invalid-not-success"
    if output_shape == "fallback_or_unknown":
        return "private-review-required"
    return "do-not-promote"
```

```text
release gate:
  static_algorithm_exposure: must_not_expose_complete_java_algorithm
  dynamic_oracle: must_not_return_state_like_failure
  business_no_intrusion: must_pass_real_device_vector
  repackaging: must_fail_closed_on_modified_package_surface
  evidence: must_use_same_candidate_same_hash_same_gate
```

这两个代码块对应今天的评估逻辑：如果输出只是状态、崩溃或 fallback，不能把它写成“防护成功”；如果正常业务路径没有真机证据，也不能把安全强度单独拿出来卖。

## 攻防逻辑：为什么 oracle 比 hook 失败更关键

| 攻击侧假设 | 攻击者想得到什么 | 今天的公开观察 | 防守侧动作 | 公开边界 |
|---|---|---|---|---|
| 从 Java 层直接读算法 | 完整签名算法或稳定业务入口 | 普通 Java 层没有完整算法 | 保持算法不在普通 DEX 暴露 | 不公开反编译片段 |
| 从 bridge 建立入口关系 | 可复用调用点 | bridge 仍有语义收敛空间 | 入口多态、延迟绑定、会话绑定 | 不公开类名和方法名 |
| 从 native 注册定位 | native 入口与业务入口关系 | native/VMP 保护链参与 | 注册去稳定化、entry epoch、调用链 token | 不公开符号和偏移 |
| 从返回值判断路径 | 真实输出或状态 oracle | 动态路径未闭合真实算法，但失败态需清理 | 状态输出阻断发布 | 不公开真实输出 |
| 从二次打包复用 | 持有签名信息后进入真实保护态 | 2.1 纳入 fail-closed 方向 | 包体、签名、加载链、assets 统一测量 | 不公开 seal 细节 |

真正的对抗点不只是“能不能 hook”。如果失败态会告诉攻击者路径状态，攻击者就可以用它做方向盘。商业级防护要让攻击者看见的状态尽量无意义，即使进入了某些观察点，也不能稳定复用到真实业务路径。

## 工程验收清单

| 门禁 | 通过标准 | 阻断条件 |
|---|---|---|
| 静态算法门禁 | 普通 Java 层无完整业务算法 | 直接恢复完整算法 |
| bridge 语义门禁 | bridge 不提供稳定业务语义 | 可稳定定位高价值入口 |
| native/VMP 门禁 | native 入口、metadata 和 handler 不提供结构线索 | 可稳定建立入口和 handler 关系 |
| 动态 oracle 门禁 | 失败态不可解释、不可复用 | 状态型输出或输入相关中间态可观察 |
| 业务无侵入门禁 | 真机业务向量一致 | 崩溃、无输出或业务错误 |
| 二次打包门禁 | 修改包面后 fail-closed | 改包后仍进入真实保护态 |
| 性能门禁 | 启动和首次计算在预算内 | 安全增强明显破坏体验 |

## 后续动态样本计划

下一轮建议只补三类证据。第一是真机业务向量自动化，把正常设备启动、业务入口、冷启动、重启、前后台和连续计算固定成 gate。第二是 oracle 负向样本，把状态型输出、fallback、中间态差异和异常日志全部作为阻断条件。第三是二次打包 fail-closed，把签名信息复制、包体重排、组件替换、SO/DEX/assets 修改都放进同一套候选样本复测。

## 现场补充记录：哪些结果不能被写成“已安全”

这轮测评里最需要压住的冲动，是把“没拿到算法”直接写成“已经安全”。公开技术记录要保留边界，所以这里把几个不能升级为最终结论的状态单独列出来。

第一，动态工具没有闭合真实算法，不等于所有动态路径都被封死。它只能说明当前常规路径没有拿到可泛化的真实业务输出。下一步还要看入口多态、延迟绑定、调用链 token、session epoch 和 metadata 黑盒是否能把分析成本继续推高。

第二，状态型输出不能作为业务输出。只要失败态能稳定反映 VM 状态、分支状态或 materializer 状态，攻击者就可以把它当作方向信号。防守侧应把状态型输出列为阻断项，而不是把它当成“保护层返回了东西”。

第三，崩溃、空输出、无响应不能直接算防护成功。商业级加固不仅要让攻击者拿不到材料，还要让正常设备正常完成业务。一个高强度但业务不可用的版本不能进入客户交付。

第四，正常设备可用还不够。用户手工确认可用只是候选证据，真正能进入产品页和销售材料的应是自动化真机 gate：启动、业务入口、冷启动、重启、前后台、首次计算、连续计算和异常路径都要有证据。

第五，历史强版本不能替代当前版本。Security 1.5 的成熟闭环是参考基线，但 Security 2.x 的防护面更宽，必须用当前 fresh 样本重新证明，而不能把旧闭环直接迁移成当前结论。

| 结果形态 | 能说明什么 | 不能说明什么 | 下一步证据 |
|---|---|---|---|
| 静态看不到 Java 算法 | 普通反编译未直接暴露算法 | 不能证明动态路径全部闭合 | native/VMP、metadata 和动态 gate |
| 常规动态路径未闭合 | 当前路径没有得到真实算法 | 不能证明所有 hook/trace 都失败 | 负向样本和入口多态验证 |
| 状态型失败输出 | 保护层状态可被观察 | 不能当成业务输出 | oracle 清理 gate |
| 正常设备可用 | 业务闭环方向正确 | 不能替代自动化证据 | 真机矩阵和性能预算 |
| 二次打包方向明确 | 防护设计有闭合目标 | 不能替代改包实测 | fail-closed 样本集 |

## 原文与延伸阅读

- 自有站原文：[Android 加固到什么程度才算商业级：御盾 Security 2.1 签名入口实测与动态 oracle 收口](https://dun.leonadev.com/article/yudun-security21-android-commercial-hardening-oracle-gate)
- 御盾 App 加固产品页：[https://dun.leonadev.com/article/yudun-app-hardening-product](https://dun.leonadev.com/article/yudun-app-hardening-product)
- App 加固 PoC 验收指南：[https://dun.leonadev.com/article/app-hardening-poc-acceptance-guide](https://dun.leonadev.com/article/app-hardening-poc-acceptance-guide)
- 市场 App 加固产品对比页：[https://dun.leonadev.com/article/mobile-app-hardening-market-comparison-framework](https://dun.leonadev.com/article/mobile-app-hardening-market-comparison-framework)

## FAQ

### 这是否说明算法已经绝对安全？

不能这么说。公开结论是：常规静态和动态路径没有闭合真实业务算法，但失败态 oracle、入口可定位性和全 gate 仍需要 Security 2.1 收口。

### 为什么不公开真实输入输出？

真实业务向量会帮助第三方建立黑盒对比，降低分析成本。公开内容只保留评估方法和脱敏结论。

### 工具失败能不能当安全成功？

不能。工具失败可能是路径没触达、脚本不匹配或环境问题。必须以真实业务输出能否被复现、失败态是否泄露状态、正常业务是否无侵入作为判断标准。
