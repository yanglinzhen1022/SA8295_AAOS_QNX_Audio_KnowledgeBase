# AAOS Routing and AudioControl Implementation Plan

> **For agentic workers:** 使用 executing-plans 在当前会话执行；用户授权连续推进，不再逐批请求确认。作者自审，不声称独立审阅。

**Goal:** 将 Native PCM/Policy 主线连接到车载 Zone/Context/BUS、CarAudioService、音量组与本地 AudioControl AIDL 边界。
**Architecture:** 保留原目录与文件名，在对应层讲机制并回链，不复制完整 Framework→DSP 链。先解释动态策略构建，再解释音量与外部控制；本地没有的新版对象不移植进主线。
**Tech Stack:** Java/C++/AIDL/XML 源码阅读，Markdown/Mermaid，临时 Python 文档检查。
**Spec:** `docs/superpowers/specs/2026-10-30-audioreach-knowledge-base-design.md`

## Global Constraints

- Android 12L/API 32 + SA8295 + QNX AudioReach；产品源码只读。
- 原仓库 `docs/audioreach-hal-pal` 分支；不新建 worktree、不合并、不推送。
- 源码声明、Java wrapper 能力、默认 HAL 示例、产品实现与运行状态分别说明。
- 不提供虚构 BUS 接线、音区硬件拓扑、衰减值或延迟；保留必要图、源码摘录与异常处理。
- 本批维护计划不加入读者导航；六篇完成不等于总目标完成。

## Review Focus

1. Zone、Context、usage、BUS address、volume group 不是同一个 ID；每个区域的映射及合法性按本地类核对。
2. 动态 AudioPolicy 注册、UID/用户关联、焦点策略与 PCM 流向分别说明；focus 不是音频数据路由。
3. setGroupVolume 的索引范围、gain 单位、静音/持久化与设备应用不能凭新版类补造。
4. AudioControl 本地 AIDL v1 的接口范围与 wrapper 能力表必须匹配；不加入 gain/module callbacks 等未支持接口。
5. duck/mute 是设备地址级通知与 HAL 责任边界；不宣称 Java wrapper 已完成真实硬件衰减。HAL death 和 RemoteException 不等于业务成功。

## Task 1: AAOS 对象与动态策略

**Files:** `09_AAOS_Car_Audio/09_9.1_AAOS音频系统总览.md`、`09_AAOS_Car_Audio/09_9.11_CarAudioDynamicRouting-动态路由构建.md`。
**Consumes:** CarAudioService、CarAudioZone、CarAudioContext、CarAudioDynamicRouting、CarAudioZonesHelper/Validator；Native APM 主线。
**Produces:** zone/context/device/group 概念关系，AudioMix 构建和注册，动态路由条件及 BUS 边界。

- [x] 阅读上述 Java 类、相关 car 配置与 AudioPolicy/AudioMix Builder 的实际使用，定位旧文新版混入点。
- [x] 写临时检查并先对旧文运行：识别本地路径、接口/类名、必需条件解释、代码摘录和链接问题。
- [x] 改写两篇，给出对象图、构建流程和实际 Java 片段；保留原文章入口。
- [x] 核对混音规则与 BUS/Context 关系，运行格式/源码摘录检查。

## Task 2: 服务初始化与音量组

**Files:** `09_AAOS_Car_Audio/09_9.2_CarAudioService-车载音频核心服务.md`、`09_AAOS_Car_Audio/09_9.5_CarVolumeGroup-车载音量组.md`。
**Consumes:** Task 1；CarAudioService、CarVolumeGroup、CarAudioDeviceInfo、CarAudioSettings、CarVolumeGroupMuting。
**Produces:** dynamic/legacy 条件、初始化依赖、用户/UID 与 zone 关系、音量索引到设备 gain、静音与持久化。

- [x] 阅读初始化、setGroupVolume、设备 gain 应用、静音/用户切换及失败分支。
- [x] 对旧文执行检查，区分现有本地类与未存在的新版辅助类。
- [x] 改写服务与音量组文章，用图和精确代码说明调用顺序、单位和边界。
- [x] 核对无音/静音问题应观察的控制状态，不附加实机结果；验证本任务文档。

## Task 3: AudioControl AIDL 边界与导航

**Files:** `10_AudioControl_HAL/10_10.1_AudioControl_HAL总览.md`、`10_AudioControl_HAL/10_10.6_AudioControlWrapperAidl-AIDL适配层架构.md`；09/10 README、总 README、CLASS_INDEX。
**Consumes:** 本地 frozen AIDL v1、AudioControlWrapperAidl/Wrapper、HalAudioFocus、CarDucking/Muting 与产品 manifest。
**Produces:** 核心 Audio HAL/AudioControl 分工，focus listener、duck/mute、fade/balance、能力/死亡处理边界及连续阅读入口。

- [x] 阅读接口快照、wrapper、接收通知实现和可定位的产品 AudioControl 服务；区分默认示例与产品端（未确认实际产品服务实现）。
- [x] 对旧文执行检查，拒绝将新版 gain/module 回调混入本地 v1。
- [x] 重写两篇，加入方向清晰的焦点与 duck/mute 图、异常/死亡处理和精确摘录。
- [x] 更新导航，仅将本批实际完成文章列入已整理主线。
- [x] 执行本批 UTF-8/围栏/链接/摘录检查、前两批回归、git diff --check；作者自审并记录限制。
- [ ] 指定文件提交后继续焦点/配置或后续平台批次，不停下来请求确认。

## Verification

- `/tmp/qc-aaos-doc-check.py`：本批文档与导航、本地简单锚点、源码文件/Java/AIDL 摘录；不提交检查工具。
- 原文块按空白规范化匹配本地源文件；伪代码显式标注，检查不替代技术复核。
- Mermaid 只统计/检查围栏，无渲染工具不声称图形通过；不安装工具、不构建产品、不运行实机命令。
- 检查进入改写文章的带锚点链接；保留未改写旧文的状态提示。

## Execution Record

- 起点：`1f116fd`，Native PCM/Policy 批次已提交，工作区干净。
- Pre-flight：Task 1 的对象/路由模型被 Task 2 服务/音量引用，Task 3 解释 Task 2 使用的 AudioControl 通知；保留路径避免重命名冲突。
- 源树初步定位：本地存在 CarAudioZone、CarVolumeGroup、CarAudioDynamicRouting；目录中未见 CarAudioZoneConfig、CarAudioGainMonitor、CarAudioMirrorRequestHandler 等旧文所述文件。正文以实际调用定义为准，不仅凭目录名作全产品能力结论。
- Ruling：沿用用户允许的作者自审和本计划内进度记录，不引入工作树/审计工程；误判风险是缺少独立审阅，明确披露。
- 基线检查：`python3 /tmp/qc-aaos-doc-check.py` 对六篇旧文得到 245 项失败（196 个本地链接、70 个代码块、40 个 Mermaid 块）；日志 `/tmp/qc-aaos-before.log`。这些是检查失败项，不是 245 个独立语义错误。
- Task 1/2/3 正文已改写，6 篇共 16 个精确源码摘录、15 个 Mermaid 块；保持原入口，无改名。正文不提供未经确认的产品实现或硬件效果。
- Ruling：保留本地 getChannelCount→setChannelMask 的原文并解释 count/mask 语义差异，不改只读产品源码、不宣称实际注册结果；若后续上下游有特殊约定，需补充验证。
- 配置核对：AR common_au XML 的 displays 标签在当前 parseAudioZone 中不单独处理；源文件不能证明显示器映射已生效。其他产品 makefile 的旧厂商服务条目也不等价于 gvmq 实际运行服务。
- 作者自审（非独立审阅）：逐项复核 Review Focus，包括 group gain 汇总非范围交集、设备失败不向 group 返回、setGroupVolume 回调先行、mute 信号独立、occupant/UID 互斥、HAL v1 六个 oneway 方法与 wrapper 异常差异。
- 自审修正：服务组静音条件原文过于笼统，改为dynamic routing 与资源开关的逻辑与、仅启用且版本不符才抛异常；HAL focus 图原先未区分请求结果，补充仅 GRANTED 存储，FAILED/DELAYED 反馈 LOSS。两项文字断言先失败，再修正。
- 格式修正：移除写入时产生的替换字符。静态检查包含本批及导航路径、简单锚点、UTF-8 替换字符、围栏、必需边界描述和空白规范化源码匹配；不等于 Java/AIDL 编译或 Mermaid 渲染。
- 提交前复验：`python3 /tmp/qc-aaos-doc-check.py --navigation` → 10 个阅读文件、158 个本地链接、16 个源码摘录、15 个 Mermaid 块、0 个带锚点入链、0 失败；`python3 /tmp/qc-native-doc-check.py --navigation` → 11 个文件、169 个本地链接、16 个 C++ 摘录、20 个 Mermaid 块、0 失败；`python3 /tmp/qc-hal-pal-doc-check.py SA8295_AAOS_QNX_Audio_KnowledgeBase` → 7 个文件、9 个 C++ 摘录、0 失败；`git diff --check` 通过。
- 没有硬件、服务运行、延迟、Binder death 或图形渲染测试。HalAudioFocus reset 的集合更新、Binder death recipient 身份仍须专门验证，不把方法名当完整恢复保证。
