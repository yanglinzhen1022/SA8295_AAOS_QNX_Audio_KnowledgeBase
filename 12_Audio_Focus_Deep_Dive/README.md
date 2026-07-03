# 第十二篇：Audio Focus 深度解析

> [← 上一篇](../11_Vendor_Layer/README.md) | [返回导航](../README.md) | [下一篇 →](../13_Volume_Device_Deep_Dive/README.md)

---

Audio Focus是Android音频系统的核心协调机制，决定多个应用同时请求音频输出时的优先级排序、Loss传播和框架级执行（Duck/FadeOut/Mute）。本篇从源码级别深度解析AOSP14/AAOS14的焦点系统。

## 目录

| 章节 | 文件 | 核心内容 |
|------|------|----------|
| 12.1 | [Focus栈模型与核心数据结构](12_12.1_Focus栈模型与核心数据结构.md) | MediaFocusControl类结构、FocusRequester字段详解、Stack模型、多焦点列表、外部策略数据结构、同步锁机制 |
| 12.2 | [完整焦点状态机含Fade Limbo状态](12_12.2_完整焦点状态机含Fade_Limbo状态.md) | 9种状态全景图、focusLossForGainRequest映射算法、Limbo悬停态、状态转换时序 |
| 12.3 | [requestAudioFocus完整流程](12_12.3_requestAudioFocus完整流程.md) | 9阶段决策(Binder/AppOps/栈/外部策略/授权)、返回值、时序图 |
| 12.4 | [焦点Loss传播与Loss类型映射](12_12.4_焦点Loss传播与Loss类型映射.md) | propagateFocusLossFromGain_syncAf三阶段、映射算法+升级链、批量移除 |
| 12.5 | [框架级焦点执行机制](12_12.5_框架级焦点执行机制.md) | ENFORCE开关、frameworkHandleFocusLoss决策矩阵、Duck/FadeOut/Mute执行链 |
| 12.6 | [AAOS焦点交互矩阵](12_12.6_AAOS焦点交互矩阵.md) | CONCURRENT/EXCLUSIVE/REJECT三种结果、CarAudioFocus判定、IAudioControl HAL路径 |
| 12.7 | [abandonAudioFocus流程](12_12.7_abandonAudioFocus流程.md) | removeFocusStackEntry参数差异、RING/CALL退出处理、栈顶恢复 |
| 12.8 | [Audio Focus全栈调用链](12_12.8_Audio_Focus全栈调用链.md) | 标准Android和AAOS两条完整调用链、五层架构、Binder IPC接口、延迟分析 |
| 12.9 | [通话Muting机制](12_12.9_通话Muting机制.md) | USAGES_TO_MUTE_IN_RING_OR_CALL、runAudioCheckerForRingOrCallAsync、mutePlayersForCall、100ms延迟设计 |
| 12.10 | [FadeOutManager深度解析](12_12.10_FadeOutManager深度解析.md) | FADEOUT_VSHAPE三段曲线、canCauseFadeOut/canBeFadedOut资格判定、FadedOutApp内部类、mFadedApps管理 |
| 12.11 | [PlaybackActivityMonitor Duck执行深度](12_12.11_PlaybackActivityMonitor_Duck执行深度.md) | DUCK_VSHAPE vs STRONG_DUCK_VSHAPE、duckPlayers筛选排除、DuckingManager/DuckedApp、强Duck判定 |
| 12.12 | [FocusRequester内部机制](12_12.12_FocusRequester内部机制.md) | 三组核心字段、focusLossForGainRequest映射、handleFocusLoss三层决策链、IPC回调 |
| 12.13 | [多用户焦点隔离](12_12.13_多用户焦点隔离.md) | 用户切换焦点清理、mAudioFocusLock用户维度保护、外部策略多用户处理 |
| 12.14 | [焦点与音量的联动机制](12_12.14_焦点与音量的联动机制.md) | 三条路径(Duck/FadeOut/Mute)、VolumeShaper四种配置、PlayerFocusEnforcer委托链、AAOS DSP路径 |

## 核心源码文件

| 文件 | 行数 | 职责 |
|------|------|------|
| AudioManager.java | 9786 | 焦点API入口 |
| MediaFocusControl.java | 1360 | 焦点管理核心 |
| FocusRequester.java | 540 | 焦点请求者封装 |
| PlaybackActivityMonitor.java | 1665 | Duck/FadeOut/Mute执行器 |
| FadeOutManager.java | 290 | 淡出曲线管理 |
| AudioService.java | 13267 | 服务中枢 |