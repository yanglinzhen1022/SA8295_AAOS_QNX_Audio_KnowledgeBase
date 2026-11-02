# 第九篇：AAOS Car Audio

> [← 上一篇](../08_HAL_Layer/README.md) | [返回导航](../README.md) | [下一篇 →](../10_AudioControl_HAL/README.md)

---

## 本地主线阅读顺序

本章按 Android 12L/API 32 + SA8295 AudioReach 整理。已重写以下主线文章，建议按职责依赖阅读：

1. [9.1 对象与系统关系](09_9.1_AAOS音频系统总览.md)：区分 zone、occupant、usage、Context、BUS 和音量组。
2. [9.11 动态路由构建](09_9.11_CarAudioDynamicRouting-动态路由构建.md)：Context 展开 usage、按地址构建 AudioMix，再注册策略。
3. [9.2 CarAudioService](09_9.2_CarAudioService-车载音频核心服务.md)：初始化、用户/UID affinity、音量与控制协作。
4. [9.5 CarVolumeGroup](09_9.5_CarVolumeGroup-车载音量组.md)：索引、millibel、设备 gain、mute 与持久化。
5. [9.3 CarAudioZone](09_9.3_CarAudioZone-多Zone音频管理.md) → [9.18 XML 解析](09_9.18_CarAudioZonesHelper-XML配置解析.md)：深入对象、版本、Context 与输入设备校验。
6. [9.10 焦点区域分发](09_9.10_CarZonesAudioFocus-多Zone焦点分发器.md) → [9.4 单区域焦点](09_9.4_CarAudioFocus-车载焦点管理.md) → [12.6 交互矩阵](../12_Audio_Focus_Deep_Dive/12_12.6_AAOS焦点交互矩阵.md)：方向性、并发条件、blockers 与 delayed。

之后读 [10.1 AudioControl](../10_AudioControl_HAL/10_10.1_AudioControl_HAL总览.md) 和 [10.6 AIDL wrapper](../10_AudioControl_HAL/10_10.6_AudioControlWrapperAidl-AIDL适配层架构.md)，向下对齐 [6.2 APM](../06_Audio_Policy_Engine/06_6.2_AudioPolicyManager-策略核心实现.md)。

> 以下完整目录保留旧入口；除上述主线外尚未完成本轮核对。镜像、CarAudioZoneConfig、CarAudioGainMonitor 等旧文不能直接作为本地实现依据。声明/配置、策略注册与真实设备效果也应分开看。

## 完整目录（含兼容入口）

> 9.9/9.13/9.15/9.16/9.17 为**兼容入口**（描述的类不在本树：CarAudioZoneConfig/CarAudioGainMonitor/CarAudioMirrorRequestHandler/MediaRequestHandler/CoreAudioHelper 均为 13+/14+ 机制，原文可从 Git 历史恢复）；9.6/9.7/9.8/9.12/9.14 已按 12L 源码补版本边界与勘误。

- [9.1 AAOS音频系统总览](09_9.1_AAOS音频系统总览.md)
- [9.2 CarAudioService — 车载音频核心服务](09_9.2_CarAudioService-车载音频核心服务.md)
- [9.3 CarAudioZone — 多Zone音频管理](09_9.3_CarAudioZone-多Zone音频管理.md)
- [9.4 CarAudioFocus — 车载焦点管理](09_9.4_CarAudioFocus-车载焦点管理.md)
- [9.5 CarVolumeGroup — 车载音量组](09_9.5_CarVolumeGroup-车载音量组.md)
- [9.6 CarAudioContext — 车载音频上下文深度解析](09_9.6_CarAudioContext-车载音频上下文深度解析.md)
- [9.7 Audio Mirroring — 音频镜像](09_9.7_Audio_Mirroring-音频镜像.md)
- [9.8 AAOS多Zone全栈调用链](09_9.8_AAOS多Zone全栈调用链.md)
- [9.9 CarAudioZoneConfig — Zone配置管理](09_9.9_CarAudioZoneConfig-Zone配置管理.md)
- [9.10 CarZonesAudioFocus — 多Zone焦点分发器](09_9.10_CarZonesAudioFocus-多Zone焦点分发器.md)
- [9.11 CarAudioDynamicRouting — 动态路由构建](09_9.11_CarAudioDynamicRouting-动态路由构建.md)
- [9.12 CarVolume — 音量优先级算法](09_9.12_CarVolume-音量优先级算法.md)
- [9.13 CarAudioGainMonitor — HAL Gain事件分发](09_9.13_CarAudioGainMonitor-HAL_Gain事件分发.md)
- [9.14 CarDucking — AAOS系统级Ducking](09_9.14_CarDucking-AAOS系统级Ducking.md)
- [9.15 CarAudioMirrorRequestHandler — 音频镜像请求管理](09_9.15_CarAudioMirrorRequestHandler-音频镜像请求管理.md)
- [9.16 MediaRequestHandler — 媒体音频请求管理](09_9.16_MediaRequestHandler-媒体音频请求管理.md)
- [9.17 CoreAudioHelper — Core Audio路由适配](09_9.17_CoreAudioHelper-Core_Audio路由适配.md)
- [9.18 CarAudioZonesHelper — XML配置解析](09_9.18_CarAudioZonesHelper-XML配置解析.md)
