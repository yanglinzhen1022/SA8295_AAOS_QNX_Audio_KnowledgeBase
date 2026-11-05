# 第十二篇：Audio Focus 深度解析

> [← 上一篇](../11_Vendor_Layer/README.md) | [返回导航](../README.md) | [下一篇 →](../13_Volume_Device_Deep_Dive/README.md)

焦点协调的是音频请求关系、loss/gain 通知及相关执行；不能简单当作固定优先级或硬件发声权限。当前按 Android 12L/API 32 + SA8295 AudioReach 重构。

## 已整理的 AAOS 主线

1. [9.10 CarZonesAudioFocus](../09_AAOS_Car_Audio/09_9.10_CarZonesAudioFocus-多Zone焦点分发器.md)：先确定焦点区域。
2. [9.4 CarAudioFocus](../09_AAOS_Car_Audio/09_9.4_CarAudioFocus-车载焦点管理.md)：holders/losers、blockers、单个 delayed 请求及恢复。
3. [12.6 AAOS 交互矩阵](12_12.6_AAOS焦点交互矩阵.md)：完整本地矩阵、方向性和并发条件。
4. [10.7 HalAudioFocus](../10_AudioControl_HAL/10_10.7_HalAudioFocus-外部焦点请求管理.md)：外部音源如何请求、收到反馈及清理。

> 本章全部 14 篇已完成 12L 校准（R2b 复审问题清单逐项修复：正文内联 L 标签重锚、13+ 方法名清除——pushBelowLockedFocusOwners 12L 无 AndPropagate 后缀且不向下传播、mNotifyFocusOwnerOnDuck 为 boolean、canReassignAudioFocus 只查栈顶、abandon 的 signal/notifyFocusFollowers 实参语义、duckUid 2 参/DuckedApp(uid) 单参/固定 DUCK_VSHAPE 与 DUCK_ID、MUTE_AWAIT_CONNECTION 不在本树、forceDuck 来源为无障碍、mRingOrCallActive 由 IN_VOICE_COMM_FOCUS_ID 置位、非逐字代码块已换逐字摘录或明确标注 14+ 示意）。

## 完整目录

| 文章 | 当前范围 |
|---|---|
| [12.1 Focus 栈模型与核心数据结构](12_12.1_Focus栈模型与核心数据结构.md) | 已校准（12L） |
| [12.2 完整焦点状态机](12_12.2_完整焦点状态机含Fade_Limbo状态.md) | 已校准（Limbo 12L 即有） |
| [12.3 requestAudioFocus 流程](12_12.3_requestAudioFocus完整流程.md) | 已校准行号（12L L874-1054） |
| [12.4 Loss 传播与映射](12_12.4_焦点Loss传播与Loss类型映射.md) | 已校准行号（:248） |
| [12.5 Framework 执行机制](12_12.5_框架级焦点执行机制.md) | 已校准（强 Duck 为 14+ 边界） |
| [12.6 AAOS 焦点交互矩阵](12_12.6_AAOS焦点交互矩阵.md) | 已按本地 FocusInteraction 重写 |
| [12.7 abandonAudioFocus](12_12.7_abandonAudioFocus流程.md) | 已校准行号（:1058） |
| [12.8 Audio Focus 跨层链路](12_12.8_Audio_Focus全栈调用链.md) | 已校准（evaluateFocusRequestLocked/onAudioFocusChange(usage, zoneId, focusChange)；延迟表改量级示意） |
| [12.9 通话 Muting](12_12.9_通话Muting机制.md) | 已校准（runAudioCheckerForRingOrCallAsync :1114） |
| [12.10 FadeOutManager](12_12.10_FadeOutManager深度解析.md) | 已校准（DELAY_FADE_IN_OFFENDERS_MS=MSL_L_FORGET_UID） |
| [12.11 PlaybackActivityMonitor Duck](12_12.11_PlaybackActivityMonitor_Duck执行深度.md) | 已校准（duckUid 2 参等 12L 逐字） |
| [12.12 FocusRequester](12_12.12_FocusRequester内部机制.md) | 已校准（afi 构造器 :115-132、flag 位值） |
| [12.13 多用户焦点隔离](12_12.13_多用户焦点隔离.md) | 已校准（mDuckers 字段名等） |
| [12.14 焦点与音量联动](12_12.14_焦点与音量的联动机制.md) | 已校准（MUTE_AWAIT_CONNECTION 负结论、VolumeShaper 链直达 App） |

焦点通知、Java 播放器状态、设备地址级 duck/mute 和真正的音频数据消费是不同观察层。已整理文章也只对照静态实现，不声称实机焦点或恢复测试通过。
