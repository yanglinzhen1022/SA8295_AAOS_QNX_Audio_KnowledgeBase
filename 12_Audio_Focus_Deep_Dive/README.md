# 第十二篇：Audio Focus 深度解析

> [← 上一篇](../11_Vendor_Layer/README.md) | [返回导航](../README.md) | [下一篇 →](../13_Volume_Device_Deep_Dive/README.md)

焦点协调的是音频请求关系、loss/gain 通知及相关执行；不能简单当作固定优先级或硬件发声权限。当前按 Android 12L/API 32 + SA8295 AudioReach 重构。

## 已整理的 AAOS 主线

1. [9.10 CarZonesAudioFocus](../09_AAOS_Car_Audio/09_9.10_CarZonesAudioFocus-多Zone焦点分发器.md)：先确定焦点区域。
2. [9.4 CarAudioFocus](../09_AAOS_Car_Audio/09_9.4_CarAudioFocus-车载焦点管理.md)：holders/losers、blockers、单个 delayed 请求及恢复。
3. [12.6 AAOS 交互矩阵](12_12.6_AAOS焦点交互矩阵.md)：完整本地矩阵、方向性和并发条件。
4. [10.7 HalAudioFocus](../10_AudioControl_HAL/10_10.7_HalAudioFocus-外部焦点请求管理.md)：外部音源如何请求、收到反馈及清理。

> 本章 12.1/12.3/12.4/12.5/12.7/12.9/12.11/12.12/12.13/12.14 已完成 12L 校准（行号勘误 + 强 Duck 版本边界：-35dB 强 Duck 为 14+ 机制不在本树，12L 仅 DUCK_VSHAPE -14dB）；12.2/12.8/12.10 等其余旧文仍待核对，不将标题和旧代码块视为本地已验证行为。

## 完整目录

| 文章 | 当前范围 |
|---|---|
| [12.1 Focus 栈模型与核心数据结构](12_12.1_Focus栈模型与核心数据结构.md) | 待核对：普通 Framework 数据结构 |
| [12.2 完整焦点状态机](12_12.2_完整焦点状态机含Fade_Limbo状态.md) | 待核对：版本与状态边界 |
| [12.3 requestAudioFocus 流程](12_12.3_requestAudioFocus完整流程.md) | 已校准行号（12L L874-1054） |
| [12.4 Loss 传播与映射](12_12.4_焦点Loss传播与Loss类型映射.md) | 已校准行号（:248） |
| [12.5 Framework 执行机制](12_12.5_框架级焦点执行机制.md) | 已校准（强 Duck 为 14+ 边界） |
| [12.6 AAOS 焦点交互矩阵](12_12.6_AAOS焦点交互矩阵.md) | 已按本地 FocusInteraction 重写 |
| [12.7 abandonAudioFocus](12_12.7_abandonAudioFocus流程.md) | 已校准行号（:1058） |
| [12.8 Audio Focus 跨层链路](12_12.8_Audio_Focus全栈调用链.md) | 待核对 |
| [12.9 通话 Muting](12_12.9_通话Muting机制.md) | 待核对 |
| [12.10 FadeOutManager](12_12.10_FadeOutManager深度解析.md) | 待核对 |
| [12.11 PlaybackActivityMonitor Duck](12_12.11_PlaybackActivityMonitor_Duck执行深度.md) | 待核对 |
| [12.12 FocusRequester](12_12.12_FocusRequester内部机制.md) | 待核对 |
| [12.13 多用户焦点隔离](12_12.13_多用户焦点隔离.md) | 待核对 |
| [12.14 焦点与音量联动](12_12.14_焦点与音量的联动机制.md) | 待核对 |

焦点通知、Java 播放器状态、设备地址级 duck/mute 和真正的音频数据消费是不同观察层。已整理文章也只对照静态实现，不声称实机焦点或恢复测试通过。
