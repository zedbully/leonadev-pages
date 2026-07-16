---
title: "启动后还能闭合吗：御盾 Android 加固包 native dispatch 与 owner loader 动态复核"
permalink: /articles/2026-07-16-yudun-android-native-dispatch-owner-loader-runtime-evidence/
---

# 启动后还能闭合吗：御盾 Android 加固包 native dispatch 与 owner loader 动态复核

原文已发布在御盾技术内容中心：[启动后还能闭合吗：御盾 Android 加固包 native dispatch 与 owner loader 动态复核](https://dun.leonadev.com/article/yudun-android-native-dispatch-owner-loader-runtime-evidence)。

结论：这轮御盾 Android 加固包复核的重点，不是静态工具能不能打开 APK，而是启动后 native dispatch、owner loader 和受控材料化链路能否形成可复核闭合。静态证据显示入口、DEX、native 与 assets 已经分层；补充安装环境的动态记录进一步证明启动后存在 native 包络加载、owner loader 可解析和页面存活证据。

## 摘要

很多 Android 加固验收报告会停在第一步：用反编译工具打开 APK，看 Manifest、DEX、SO 和资源目录，然后根据能看到多少 Java 表面判断强弱。这一步是必要的，但不是最终结果。真正接近实战的问题是：应用启动以后，静态看到的壳层、native 载体和 assets 材料是否能被运行时正确接管？入口代理是否只是写在 Manifest 里的名字，还是实际进入了 native dispatch 归属链？owner loader 是不是完成了材料选择、装载、可解析和 Activity 侧闭合？

本轮复核先做静态分析，再做动态复核。静态阶段观察到：发布包具备主流签名校验面；Manifest 关闭备份与调试，native 库不走普通默认解压；启动入口由应用级代理组件接管；包内存在多 DEX、arm64 native 载体和 assets 侧不透明材料；反编译视角看到的是壳层、桥接与运行时调度表面，而不是直接展开的完整业务源码。动态阶段第一次环境被旧安装状态污染，不能采信。用户补充已安装环境后，重新采集到启动、进程存活、窗口焦点、native 包络加载、owner payload 选择、owner loader 返回可解析、alias owner loader 安装、native owner remap 成功等记录，且复核时没有捕捉到 FATAL、ANR 或强制结束信号。

因此，这篇文章只回答一个问题：御盾 Android 加固包在“静态分层已经成立”的前提下，运行时 owner loader 闭合是否有证据支撑。本文不还原原始算法，不写脱壳教程，不公开应用标识、命令、设备、路径、样本唯一标识、类名、方法名、SO 名或日志原文，也不宣称所有脱壳工具都无法恢复。本文的价值在于把静态观察、动态采信边界和运行时闭合证据放在同一张验收表里。

## 读者对象

这篇文章适合三类读者。第一类是正在评估 Android App 加固产品的安全负责人，他们需要知道 PoC 验收不能只看“JADX 截图”和“DEX 文件数量”，还要看启动后的 owner loader 是否能闭合。第二类是 Android 工程团队，他们关心加固后是否影响启动、组件路由、native 加载和 Activity 进入。第三类是逆向测评人员，他们需要把攻击工具矩阵、现象复核链和主因链写清楚，而不是看到一个现象就跳到下一个工具。

如果您的验收目标是“还原原始算法”“二次打包回装”“危险环境矩阵”“DEX dump”“SO dump”或“注入 Hook 观测”，这些都应该单独开专项。把所有目标塞进一篇文章，会让证据链变散，也容易把未验证结论写成已通过结论。本轮主线只有一个：启动后的 native dispatch 与 owner loader 闭合。

## 核心结论

- 静态阶段确认发布包具备主流签名校验面，适合进入完整性与二次打包方向的后续验收。
- Manifest 层关闭备份和调试，native 解压策略不是普通默认展开，入口由应用级代理组件接管。
- 包内存在多 DEX、arm64 native 载体和 assets 侧不透明材料，静态视图呈分层结构。
- 反编译视角主要看到壳层、桥接、运行时调度与读回注册表面，没有直接形成完整业务源码地图。
- 首次动态环境被旧安装状态污染，因此不作为本轮动态证据。
- 用户补充安装环境后，启动动作能进入目标应用，延迟复核时进程仍存在，窗口焦点落在应用页面。
- 运行时记录显示 native 包络加载、核心材料解封和 fileless/memfd 形态加载进入成功阶段。
- Activity 与主进程侧均出现 native root 注册成功记录，说明入口代理进入了 native owner 归属链。
- owner payload 经历选择、摘要创建、metadata 准备、payload 拷贝和 owner parent 准备等阶段。
- owner loader 返回可解析状态，class shell owner loader 被接受，alias owner loader 完成安装。
- 二次复核没有捕捉到 FATAL、ANR、强制结束、进程死亡或 native 崩溃类启动失败信号。
- 本轮可以写“启动闭合与 owner loader 路径复核成立”，但不能写“所有脱壳、重打包和注入场景均已覆盖”。

## 测评目标与非目标

| 目标 | 本轮状态 | 已做动作 | 可公开结论 | 边界 |
|---|---|---|---|---|
| 入口代理复核 | 已执行 | 解析 Manifest 与启动链路。 | 入口由应用级代理组件接管。 | 不公开真实组件名。 |
| native dispatch 归属 | 已执行 | 复核动态加载与 native root 注册记录。 | 运行时进入 native 归属链。 | 不公开日志 tag、SO 名和方法名。 |
| owner loader 闭合 | 已执行 | 复核 owner payload、loader 和可解析状态。 | owner loader 路径形成可采信闭合。 | 不公开材料名、路径和类名。 |
| 启动存活 | 已执行 | 复核进程、窗口焦点和异常过滤。 | 补充环境中未见启动崩溃类信号。 | 不公开设备与进程标识。 |
| 原始算法还原 | 非目标 | 未执行。 | 本文不做算法还原结论。 | 避免变成攻击教程。 |
| DEX/SO 脱壳 | 非目标 | 未执行。 | 留作后续专项。 | 不声明脱壳工具全部失败。 |
| 二次打包回装 | 非目标 | 未执行。 | 留作后续签名与完整性专项。 | 不写已覆盖。 |
| 注入 Hook 对抗 | 非目标 | 未执行。 | 留作后续动态观测专项。 | 不公开可复现注入步骤。 |

本轮主目标是“启动后能否进入 native dispatch 与 owner loader 闭合”。非目标包括原始算法还原、二次打包、危险环境安装、DEX dump、SO dump、注入 Hook 和调试跟踪。后续文章可以围绕这些非目标继续做，但不能把它们提前写成本轮结论。

## 事实依据与脱敏证据

| # | 证据来源类型 | 脱敏后的观察事实 | 支撑的工程判断 | 公开化边界 |
|---:|---|---|---|---|
| 1 | 签名校验输出 | APK 通过 Android 主流签名方案校验。 | 样本适合进入签名完整性与二次打包闭合的后续复核。 | 不公开证书、摘要、签名人信息。 |
| 2 | Manifest 静态解析 | 应用关闭备份与调试，native 库不走普通默认解压。 | 发布面没有暴露基础调试开关，native 载体加载受控。 | 不公开真实属性原文和组件名。 |
| 3 | Manifest 与反编译交叉观察 | 启动入口保持标准 launcher 形态，但初始化链路由代理组件接管。 | 入口不是裸业务 Activity 直出，适合做入口代理验收。 | 不公开 Activity、Application、Factory 名称。 |
| 4 | APK 结构与反编译输出 | 包内存在多 DEX、arm64 native 载体和 assets 侧不透明材料。 | 防护结构呈 DEX、native、assets 分层，而不是单一 DEX 展开。 | 不公开文件名、路径和材料名。 |
| 5 | JADX 反编译表面 | 可见内容集中在壳层、桥接、调度和读回注册表面。 | 静态 Java 视角没有直接形成完整业务源码地图。 | 不公开源码、类名和方法名。 |
| 6 | 首次动态环境检查 | 旧安装状态造成动态结果不可采信。 | 不能把污染环境的启动日志写成本轮 APK 通过。 | 不公开应用标识、命令、设备和失败原文。 |
| 7 | 补充安装环境复测 | 启动后进程延迟复核仍存在，窗口焦点落在目标应用页面。 | 补充环境中的启动存活证据可采信。 | 不公开窗口记录、进程号和设备标识。 |
| 8 | 运行时日志脱敏摘要 | native 包络加载、核心材料解封、fileless/memfd 加载进入成功阶段。 | native dispatch 运行时路径不是静态摆设。 | 不公开日志 tag、SO 名、路径和原文。 |
| 9 | owner loader 阶段记录 | owner payload 完成选择、metadata 准备、payload 拷贝和 parent 准备。 | 材料化链路具备阶段性闭合证据。 | 不公开材料路径、类名、loader 名。 |
| 10 | loader 可解析复核 | owner loader 返回可解析，class shell owner loader 被接受，alias owner loader 安装。 | Activity 侧运行时可解析闭合成立。 | 不公开方法名、符号和调用链。 |
| 11 | 异常过滤复核 | 未捕捉到 FATAL、ANR、强制结束、进程死亡或 native 崩溃类启动失败信号。 | 可以写启动闭合观察，不写更大范围通过。 | 不公开 logcat 原文。 |

## 原始报告事实映射（APK 分析事实映射）

| 分析事实 | 公开转述 | 支撑判断 | 未公开边界 |
|---|---|---|---|
| 签名校验通过主流方案。 | 发布包具备签名校验基础。 | 可进入完整性验收。 | 证书和摘要不公开。 |
| Manifest 关闭备份与调试。 | 基础发布面收敛。 | 降低调试和数据暴露面。 | Manifest 原文不公开。 |
| 入口代理与应用级工厂介入。 | 启动链路被保护侧接管。 | 支撑入口代理验收。 | 类名和组件名不公开。 |
| DEX、native、assets 分层。 | 防护载体跨层分布。 | 不能只用 DEX 判断强度。 | 文件名和路径不公开。 |
| 首次动态环境不可采信。 | 污染环境被排除。 | 防止错误结论。 | 应用标识和失败原文不公开。 |
| 补充环境启动后进程存活。 | 具备动态启动证据。 | 可以进入运行闭合判断。 | 设备和进程标识不公开。 |
| native 包络加载与材料解封。 | native dispatch 进入运行阶段。 | 支撑 native 归属链判断。 | 日志原文不公开。 |
| owner loader 可解析。 | Activity 侧 loader 闭合。 | 支撑运行时可解析结论。 | loader 名和类名不公开。 |
| 异常过滤未见启动失败信号。 | 启动阶段没有观察到崩溃闭锁。 | 支撑“补充环境启动闭合”。 | 不扩展为全场景通过。 |

```yaml
public_evidence:
  assessment_goal: "native dispatch and owner loader startup closure"
  static_stage:
    signing: "main Android signature schemes verified"
    manifest_surface: "backup and debug disabled; protected startup routing observed"
    carrier_layout: "multi-dex, arm64 native carrier, opaque asset material"
    decompile_surface: "shell and bridge surface; no direct full business source map"
  dynamic_stage:
    first_environment: "not accepted because existing install state polluted the result"
    supplemental_environment:
      installed_visible: true
      launch_event: "observed"
      process_after_delay: "present"
      focused_window: "target application"
      native_envelope: "loaded and decrypted"
      memfd_style_loading: "observed as successful runtime stage"
      owner_payload: "selected and materialized"
      owner_loader: "returned resolvable loader"
      alias_loader: "installed"
      native_owner_remap: "ok"
      fatal_or_anr: "not observed in filtered startup window"
  public_boundaries:
    - "no application identifier"
    - "no adb endpoint"
    - "no raw command"
    - "no class or method names"
    - "no raw log"
    - "no file path or sample fingerprint"
```

## 动态时间线

| 阶段 | 观察动作 | 观察结果 | 判断 | 公开边界 |
|---:|---|---|---|---|
| T0 | 静态前置复核 | 签名、Manifest、DEX、native、assets 已完成分层观察。 | 可以进入动态启动复核。 | 不公开原始工具输出。 |
| T1 | 首次动态尝试 | 环境存在旧安装状态污染。 | 该环境结果不可采信。 | 不公开失败原文。 |
| T2 | 用户补充安装环境 | 目标应用处于可见安装状态。 | 可以重新采集动态证据。 | 不公开 ADB 端口和设备。 |
| T3 | 启动动作 | 启动事件被系统接受。 | 应用进入启动流程。 | 不公开应用标识和命令。 |
| T4 | 延迟复核 | 进程仍存在，窗口焦点落在应用页面。 | 启动存活成立。 | 不公开进程和窗口原文。 |
| T5 | native 包络 | 包络加载、核心材料解封、fileless/memfd 运行阶段出现成功记录。 | native dispatch 进入运行态。 | 不公开 SO 名和路径。 |
| T6 | native root 注册 | 主进程和 Activity 侧出现注册成功记录。 | 入口代理进入 native owner 归属链。 | 不公开 tag 和类名。 |
| T7 | owner 材料化 | payload 选择、metadata 准备、payload 拷贝、parent 准备等阶段出现。 | owner loader 具备材料化证据。 | 不公开材料名。 |
| T8 | 可解析闭合 | owner loader 返回可解析，alias loader 安装，native owner remap 成功。 | Activity 侧闭合可采信。 | 不公开调用链。 |
| T9 | 异常过滤 | 未见 FATAL、ANR、强制结束、进程死亡或 native 崩溃类启动失败信号。 | 可以写启动闭合，不扩展为全场景通过。 | 不公开 logcat 原文。 |

## 攻击工具矩阵

| 阶段 | 常见工具或方法 | 本轮观察 | 复核意义 | 公开边界 |
|---|---|---|---|---|
| APK 容器枚举 | APK 解包、条目枚举 | 能看到 DEX、native、assets 分层。 | 建立静态载体地图。 | 不公开条目名。 |
| Java 反编译 | JADX 类工具 | 主要看到壳层、桥接和调度表面。 | 判断静态 Java 视图的边界。 | 不公开源码。 |
| Manifest 路由 | Manifest 解析 | 入口由代理链路接管，基础调试面收敛。 | 判断启动是否裸直出。 | 不公开组件名。 |
| native 观察 | SO 摘要、运行日志 | native 包络加载和材料解封进入成功阶段。 | 判断 native dispatch 是否运行。 | 不公开 SO 名和日志。 |
| owner loader 复核 | 动态日志阶段归纳 | payload 材料化、loader 可解析、alias loader 安装。 | 判断运行时闭合。 | 不公开 loader 名和类名。 |
| 启动稳定性 | 进程、焦点、异常过滤 | 进程存活，焦点进入应用，未见启动崩溃类信号。 | 判断启动闭合是否可采信。 | 不公开设备、进程、窗口原文。 |
| 后续专项 | DEX dump、SO dump、重签回装、注入 Hook | 本轮未执行。 | 防止把非目标写成通过项。 | 不公开攻击步骤。 |

这个矩阵的作用不是教人攻击，而是告诉验收人员每个工具应该回答哪个问题。反编译工具回答“静态 Java 表面是什么”；Manifest 解析回答“入口和组件面是什么”；动态日志回答“运行时是否进入保护侧链路”；异常过滤回答“启动阶段是否出现崩溃信号”。只有这些问题串起来，才有资格写“启动闭合”。

## 现象复核链

| 观察 | 复核方式 | 判断 | 边界 |
|---|---|---|---|
| 静态视图看到分层载体。 | 对照 Manifest、DEX、native 与 assets 摘要。 | 防护结构不是单一 DEX 表面。 | 不公开路径。 |
| 反编译仍有 Java 表面。 | 与桥接、壳层、调度表面交叉判断。 | 可见表面不等于完整业务恢复。 | 不公开源码。 |
| 首次动态有环境污染。 | 检查安装状态与启动来源。 | 不采信该次动态结果。 | 不公开失败细节。 |
| 补充环境中应用可启动。 | 延迟复核进程与窗口焦点。 | 启动存活成立。 | 不公开设备和进程。 |
| native 包络加载成功。 | 过滤运行时阶段记录。 | native dispatch 进入执行路径。 | 不公开 tag 和 SO。 |
| owner payload 完成材料化。 | 观察选择、metadata、拷贝、parent 准备阶段。 | owner loader 具备材料来源。 | 不公开材料名。 |
| loader 返回可解析。 | 复核可解析状态、alias loader 安装和 remap 成功。 | Activity 侧闭合成立。 | 不公开类名和调用链。 |
| 未见启动崩溃类信号。 | 过滤 FATAL、ANR、强制结束、进程死亡与 native 崩溃。 | 启动阶段没有观察到失败闭锁。 | 不扩展为全场景通过。 |

## 主因链

本轮能形成正向动态结论，核心原因不是“静态工具看不懂”，而是静态与动态的证据互相对上了。静态层先显示入口代理、DEX/native/assets 分层和壳层桥接表面；动态层再看到 native 包络加载、核心材料解封、owner payload 材料化和 owner loader 可解析。两边合起来，才能说明加固包不是只在静态结构上“摆壳”，而是在启动阶段把入口、native dispatch 和 owner loader 串成了运行链。

首次动态环境不可采信也很重要。安全报告经常出问题，就是因为把旧安装包、缓存状态、错误包或污染日志当成本轮样本结果。本轮把污染环境剔除，等用户补充已安装环境后才采集动态证据。这个过程反而增强了报告可信度：它说明文章不是为了写好话而忽略边界，而是在证据可采信之后才写正向结论。

因此，本文能写的主因链是：入口代理收束启动入口，native 包络承接运行侧材料，owner payload 在启动过程中完成选择和材料化，owner loader 返回可解析状态，alias loader 与 native owner remap 完成 Activity 侧闭合，异常过滤没有看到启动失败信号。本文不能写的主因链是：所有脱壳工具失败、所有二次打包失败、所有危险环境通过、所有 Hook 框架失效。这些结论需要后续专项。

## 技术拆解

### 1. 为什么不能只看静态反编译

Android 加固后的静态反编译结果通常会出现两种误读。第一种误读是“还能看到 Java 文件，所以没加固好”。第二种误读是“看不到业务源码，所以所有攻击都挡住了”。这两种都不专业。静态反编译只能说明当前 Java 视角的表面状态，不能直接说明运行时闭合，也不能替代 DEX dump、SO dump、重打包或注入专项。

本轮静态阶段的价值，是把启动入口、DEX 分布、native 载体、assets 材料和壳层桥接面拆开。它告诉我们：后续动态应该盯住 native dispatch 和 owner loader，而不是继续停留在“JADX 看到多少行”的层面。

### 2. native dispatch 的验收重点

native dispatch 的核心验收点不是“有没有 SO 文件”，而是启动后有没有进入 native 运行链。公开文章不需要暴露 SO 名、符号或路径，只需要说明运行时阶段是否出现包络加载、材料解封、受控加载和 native root 注册等脱敏现象。本轮补充环境中，这些阶段记录能够串起来，说明 native dispatch 不是静态摆设。

### 3. owner loader 的验收重点

owner loader 的验收重点是“材料能不能被正确归属并解析”。如果静态看到 assets 中有不透明材料，但动态没有材料选择、metadata、payload 拷贝、parent 准备和可解析返回，就不能说运行闭合。反过来，本轮动态记录中这些阶段都有对应脱敏现象，说明 owner loader 具备从材料选择到 Activity 侧可解析的闭合证据。

### 4. 为什么要保留首次动态失败边界

首次动态环境被旧安装状态污染，这在真实测评里很常见。如果直接把后续启动日志当成当前 APK 的结果，文章就会变成没有事实依据的漂亮话。本文保留这个边界，并把它写成证据链的一部分：不可采信的环境被剔除，可采信的环境重新采集。这样读者能看到测评过程，而不是只看到结论。

### 5. 启动闭合不等于全场景安全

启动闭合只覆盖一段范围：应用能从入口进入，native dispatch 与 owner loader 能完成运行时闭合，启动窗口内未见崩溃类信号。它不等于 DEX dump 失败，不等于 SO dump 失败，不等于二次打包失败，也不等于所有 Hook 框架失败。专业文章必须把这些边界写清楚，否则后续客户做 PoC 时会发现结论无法复现。

## 工程落地

如果企业要把这类复核纳入 App 加固 PoC，可以按下面的顺序落地。

第一步，先确认样本与环境。不要在旧安装状态、缓存污染、签名冲突或多版本共存的情况下写动态结论。每次动态前，都应该说明环境是否可采信。本文第一次动态结果被剔除，就是这个原则的例子。

第二步，做静态分层。至少确认 Manifest 入口、备份与调试开关、native 解压策略、DEX 数量、native 载体、assets 材料、反编译表面和敏感字符串面。静态分层的目的不是下最终结论，而是决定动态阶段应该盯哪里。

第三步，做启动闭合。启动后不要只看“界面起来了”，还要看进程是否存活、焦点是否进入目标应用、native 包络是否加载、owner payload 是否材料化、loader 是否可解析、异常过滤是否干净。每个观察都要有边界。

第四步，做专项扩展。启动闭合通过后，再做 DEX dump、SO dump、二次打包、危险环境、注入 Hook、调试跟踪等专项。每个专项单独写目标和非目标，避免一篇报告里混结论。

```text
poC_runtime_closure_checklist:
  - confirm_current_candidate_and_clean_environment
  - static_manifest_and_carrier_mapping
  - launch_and_recheck_process_after_delay
  - verify_focus_enters_target_page
  - summarize_native_envelope_loading_without_raw_logs
  - summarize_owner_payload_materialization_without_paths
  - verify_owner_loader_resolvable_state
  - filter_fatal_anr_force_finish_process_death
  - state_boundaries_before_publication
```

## 攻防视角

从攻击者视角看，第一阶段会先建立静态地图：入口在哪里，Java 表面有哪些，native 文件在哪里，assets 是否有材料，字符串是否泄露业务线索。御盾这轮加固包的静态表现，让这张地图不能直接落到完整业务源码。攻击者如果继续推进，就必须进入动态观察、dump、重打包或注入专项。

从防守者视角看，不能因为静态地图不完整就提前庆祝。真正关键的是运行时是否能把受保护材料接起来。本轮动态复核看到 native 包络、owner payload、owner loader 和 Activity 侧可解析闭合，这才是比“JADX 看不到完整源码”更有价值的证据。

从采购和验收视角看，这类报告最重要的是可复核边界。一个可信的加固验收结论，必须写清楚哪些通过、哪些未测、哪些不能公开、哪些需要下一轮专项。本文只给出启动闭合结论，不替代脱壳、二次打包和注入专项。

## 风险边界

本文所有公开结论都来自脱敏后的静态与动态观察，不包含原始路径、应用标识、设备、命令、日志、类名、方法名、SO 名、assets 名、样本唯一标识或可复现攻击链。本文不会公开任何能帮助攻击者复现绕过、脱壳或注入的细节。

本文的动态结论只覆盖补充安装环境中的启动闭合与 owner loader 路径复核。它不覆盖所有 Android 版本、所有 ROM、所有模拟器、所有 root 环境、所有 Hook 框架、所有脱壳工具、所有重签场景和所有业务路径。后续如果要扩展范围，需要按专项继续采集证据。

本文也不把“没有观察到崩溃”写成“永不崩溃”。启动窗口内未见 FATAL、ANR、强制结束和 native 崩溃，只能说明本轮启动复核没有捕捉到这些失败信号。长期稳定性、兼容性和性能仍需要持续矩阵。

## 常见误区

| 误区 | 为什么不严谨 | 正确做法 |
|---|---|---|
| 看到 Java 文件就说加固无效 | 壳层、桥接和运行时调度本来就可能需要 Java 表面。 | 判断是否形成完整业务源码地图。 |
| 看到 DEX 小就直接说很安全 | DEX 小只说明 DEX 视图小，不代表所有攻击都失败。 | 结合 native、assets 和动态闭合。 |
| 启动一次就写全场景通过 | 启动只覆盖一段路径。 | 把启动闭合、脱壳、重打包、注入分开写。 |
| 不区分污染环境 | 旧安装包日志可能误导结论。 | 先确认环境可采信，再写动态结果。 |
| 把日志原文贴到公开稿 | 日志可能暴露应用标识、类名、路径和实现细节。 | 使用脱敏阶段、现象和判断。 |
| 用营销话术代替证据 | 客户 PoC 会要求复核链。 | 使用证据表、时间线和边界说明。 |

## FAQ

### 这篇文章能证明御盾挡住了所有脱壳工具吗？

不能。本文证明的是本轮补充安装环境中的启动闭合与 owner loader 路径复核，不覆盖所有 DEX dump、SO dump 或内存脱壳工具。脱壳专项需要单独执行。

### 为什么文章不公开应用标识、日志和命令？

因为这些信息可能把公开测评变成可复现攻击材料。公开稿只需要说明阶段、现象、判断和边界，不需要暴露可定位样本和操作细节。

### 第一次动态环境为什么不采信？

因为存在旧安装状态污染，启动日志不能可靠归属于本轮 APK。安全测评必须把这种情况剔除，否则结论会失真。

### owner loader 闭合为什么重要？

它说明受保护材料不是停留在静态包内，而是在启动过程中完成选择、材料化、可解析和 Activity 侧接管。这个证据比单纯展示反编译截图更接近运行时真实状态。

### 下一步最应该测什么？

建议优先做 DEX dump/脱壳专项和二次打包/重签闭合专项。前者验证运行时材料是否容易被恢复，后者验证签名、完整性和启动闭锁是否形成闭环。

## 内链

- [御盾 App 加固产品页](https://dun.leonadev.com/article/yudun-app-hardening-product)
- [App 加固 PoC 验收指南](https://dun.leonadev.com/article/app-hardening-poc-acceptance-guide)
- [性能与兼容性中心](https://dun.leonadev.com/article/yudun-performance-compatibility-center)
- [市场 App 加固产品对比页](https://dun.leonadev.com/article/mobile-app-hardening-market-comparison-framework)
