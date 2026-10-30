# Framework / Native PCM Implementation Plan

> **For agentic workers:** 使用 executing-plans 在当前会话实施；用户已授权连续推进，不再等待每批确认。复核采用作者自审，不声称独立审阅。

**Goal:** 将 HAL-PAL 样章向上连接到客户端、共享内存、AudioFlinger 播放/录音与策略选择，形成可连续学习的 Android 12L PCM 主线。
**Architecture:** 保留原路径；已有大篇章中跨版本或未证实的细节不直接沿用，按本地调用方/被调用方重建关键机制。分层解释优先，不为覆盖率批量生成概述或跳转页。
**Tech Stack:** Markdown、Mermaid、本地 C++/Java/Binder/HIDL 源码、临时 Python 文档检查。
**Spec:** `docs/superpowers/specs/2026-10-30-audioreach-knowledge-base-design.md`

## Global Constraints

- Android 12L/API 32 + SA8295 + QNX AudioReach；产品源码只读。
- 沿用 `docs/audioreach-hal-pal` 分支；只提交知识库，不合并、不推送、不执行实机操作。
- 源码摘录必须匹配，简化代码标为伪代码；图表不补造接口或性能数字。
- 本批是总目标的一个子项目；后续不再请求计划或目录审批，但未完成部分不得记为完成。

## Review Focus

1. 本地已有 AIDL 形式的 AudioFlinger 服务接口，不把“核心 HAL 使用 HIDL”等同于 Framework 没有 AIDL。
2. 创建 Track、选择 output/input、创建 HAL 流与 start 是不同步骤；播放与录音不可镜像照抄。
3. 普通流共享内存与 HAL FMQ 是不同缓冲区；区分 front/rear 所有权、发布顺序、字节/帧及回绕。
4. FAST 是协商结果，失败可重新选择/创建；write/read 可短返回，DEAD_OBJECT 与数据丢失的恢复语义要明确。
5. AudioPolicy 不承载 PCM；getOutputForAttr 与 startOutput、getInputForAttr 与 startInput 分开；动态策略优先路径不能被固定设备表掩盖。

## Task 1: 客户端共享内存

**Files:** `04_Native_Framework_Layer/04_4.4_共享内存机制深度解析.md`
**Consumes:** AudioTrack/AudioRecord、AudioTrackShared.h/.cpp、AudioFlinger TrackBase 分配路径。
**Produces:** client/server proxy、cblk 与 audio buffer、生产消费方向、futex/flags 与恢复边界。

- [x] 阅读 `frameworks/av/media/libaudioclient/{AudioTrack,AudioRecord,AudioTrackShared}.cpp` 与 `frameworks/av/include/private/media/AudioTrackShared.h`；追踪服务端 `Tracks.cpp` 中 buffer 分配。
- [x] 保存旧文已知错误反例并运行临时检查，确认旧文不满足新主线要求。
- [x] 重写正文，加入方向图、发布/消费流程与原文摘录；不宣称端到端零拷贝。
- [x] 核对 PCM 内存角色、字节/帧、失败与唤醒条件，运行格式/链接/摘录检查。

## Task 2: AudioFlinger 播放与录音主线

**Files:** `05_AudioFlinger/05_5.9_音乐播放全栈调用链.md`、`05_AudioFlinger/05_5.10_录音全栈调用链.md`
**Consumes:** Task 1；`AudioFlinger.cpp`、`Threads.cpp`、`Tracks.cpp`（含 TrackHandle、RecordHandle 实现）与 libaudioclient。
**Produces:** 普通 PCM 创建、协商、start、write/read、线程处理、stop/释放及恢复；向下回链 8.1/15.10。

- [x] 阅读 client create/start/write/read、服务端 createTrack/createRecord、线程选择、FAST 接受条件与失败重试。
- [x] 追踪普通 MixerThread/RecordThread 的实际数据处理、HAL read/write，以及 Track/RecordTrack 启停与策略交互。
- [x] 分别重写播放与录音，保留必要时序、代码、异常和调试观察；移除伪全栈及固定延迟表。
- [x] 核对每条调用与返回，验证文档并复核输入输出差异。

## Task 3: Policy 与服务职责

**Files:** `06_Audio_Policy_Engine/06_6.2_AudioPolicyManager-策略核心实现.md`、`06_Audio_Policy_Engine/06_6.1_AudioPolicyService-控制面入口.md`、`05_AudioFlinger/05_5.1_AudioFlinger-音频数据面引擎.md`
**Consumes:** Task 2；AudioPolicyInterfaceImpl.cpp、AudioPolicyManager.cpp、AudioPolicyClientImpl.cpp 及 AIDL 声明。
**Produces:** 服务/策略/引擎/输出描述符边界，与 PCM 数据面独立的路由控制。

- [x] 对照本地接口和实现阅读 output/input 选择、动态 policy mix、启动/停止/释放及 client callback。
- [x] 删除与本地无关的新版本路由机制、伪造签名和静态设备优先级表；保留可证实机制。
- [x] 写清图、代码、解释与失败条件；补全当前主线而不扩写未读的策略细节。
- [x] 更新 04/05/06 README、总 README、CLASS_INDEX 的已整理阅读入口。
- [x] 运行本批检查、自审；记录实际结果。
- [x] 指定文件提交后继续下一个子项目：提交 `1f116fd`，后续计划为 `2026-10-30-aaos-routing-control.md`。

## Verification

- `git diff --check`。
- 临时检查器仅扫描本批：UTF-8、围栏、本地链接、已知错误反例；C++ 原文块按空白规范化匹配本地源码。
- 技术自审针对 Review Focus，不用格式检查冒充语义验证。
- 无 `mmdc` 则不声称图形渲染成功；不安装工具、不构建产品、不测硬件。

## Execution Record

- 起点：`4fb381f`，HAL-PAL 样章已完成；当前工作区干净。
- 用户追加授权：持续执行直至总目标完成，后续不再逐批询问。
- 总目标尚余：Framework/Native 核心；AAOS/AudioControl/焦点/音量/配置；PAL/AGM/GSL 深入；QNX/Kernel/跨 VM/校准；播放录音/蓝牙/语音/音效/恢复/调试；新版本独立补充与全库收尾。
- 本批不把 6 篇正文完成当作上述总目标完成。其他章节按知识依赖继续研究和改写。
- Pre-flight：Task 1 提供共享协议模型，Task 2 消费该模型并分别解释播放/录音，Task 3 解释其中的控制调用；交叉引用沿用原文件，无重命名冲突。
- Ruling：维护进度继续放在本计划，不另建面向读者的审计/台账系统；沿用用户指定原分支，不建 worktree、不合并、不推送。
- Ruling：修正计划中的源码路径；共享内存头文件实际为 `frameworks/av/include/private/media/AudioTrackShared.h`，TrackHandle/RecordHandle 实现在 `Tracks.cpp`，不存在本计划原先列出的两个独立 .cpp。按实际定义阅读，不补造文件。
- Ruling：不能按“新版印象”删除本地已有实现。APM/AF 中确有 Spatializer 条件分支；保留条件提示，但不宣称产品已启用，不伪装为 Android 14 core HAL。
- 自审重点：确认输出 NormalSink/MonoPipe/FastMixer 与直接输出分支，录音两种重试及 STARTING_1/STARTING_2，APS stopOutput 的排队语义、录音 silenced，以及 APM 动态主/次输出与显式设备优先关系。此次为作者自审，没有独立审阅者。
- 临时检查器 `/tmp/qc-native-doc-check.py` 首次运行旧六篇：363 个未满足项（包括无法解析的源码相对链接、未匹配的原文块与缺少的新解释项；不等于 363 个独立技术错误），输出在 `/tmp/qc-native-before.log`。
- 改写后的检查发现一处 replacement character，已修正；最终检查 11 个阅读文件、151 个本地链接、16 个 C++ 原文摘录、20 个 Mermaid 围栏，0 个检测失败。原文按空白规范化匹配本地源文件。
- 对全库检索进入这六篇的带锚点链接：找到 0 条；本批出站链接与简单标题锚点均纳入检查，不等于全库链接审计。
- 原 HAL-PAL 样章回归：7 个文件、9 个 C++ 摘录，0 个检测失败；`git diff --check` 通过。
- 未找到 mmdc，没有执行 Mermaid 图形渲染、产品构建、运行服务核验或实机播放/录音/延迟测试。图形渲染与动态行为仍是验证限制。
- 六篇内容已建立 Native PCM/Policy 主线；其余 04/05/06 原文章仍待专项整理，不能因导航更新就视为已核对。
