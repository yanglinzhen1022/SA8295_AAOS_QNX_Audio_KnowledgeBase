# 第十篇：AudioControl HAL

> [← 上一篇](../09_AAOS_Car_Audio/README.md) | [返回导航](../README.md) | [下一篇 →](../11_Vendor_Layer/README.md)

## 本地主线阅读顺序

本地产品 manifest 声明 AudioControl AIDL v1，核心 Audio HAL 则声明 HIDL 6.0，两者独立。优先读已重写的：

1. [10.1 AudioControl HAL 总览](10_10.1_AudioControl_HAL总览.md)：六个 v1 控制方法、双向焦点、地址级 duck/mute、fade/balance 以及默认示例边界。
2. [10.6 AudioControlWrapperAidl](10_10.6_AudioControlWrapperAidl-AIDL适配层架构.md)：固定能力表、类型转换、异常差异、Binder death 与上层重注册。

配合 [9.2 CarAudioService](../09_AAOS_Car_Audio/09_9.2_CarAudioService-车载音频核心服务.md) 与 [9.5 音量组](../09_AAOS_Car_Audio/09_9.5_CarVolumeGroup-车载音量组.md) 理解调用来源；AudioPort gain 与 AudioControl mute/duck 不是同一接口。

> 本章其余原文仍待核对，可能混入新版 gain/module/metadata 接口，不能据此扩展本地 AIDL v1 能力。控制通知正常返回也不证明设备已完成衰减、静音或死亡恢复。

## 完整目录（保留原文件入口）

- [10.1 AudioControl HAL 总览](10_10.1_AudioControl_HAL总览.md) — 已按本地 v1 重写。
- [10.2 核心接口](10_10.2_核心接口.md) — 待核对；原接口数量与新版回调不作为本地主线依据。
- [10.3 焦点回调流程](10_10.3_焦点回调流程.md) — 待核对。
- [10.4 Ducking 机制](10_10.4_Ducking机制.md) — 待核对。
- [10.5 Muting 机制](10_10.5_Muting机制.md) — 待核对。
- [10.6 AudioControlWrapperAidl](10_10.6_AudioControlWrapperAidl-AIDL适配层架构.md) — 已按本地 wrapper 重写。
- [10.7 HalAudioFocus](10_10.7_HalAudioFocus-外部焦点请求管理.md) — 待核对。
- [10.8 AudioControlFactory 与 HIDL Wrapper](10_10.8_IAudioGainCallback-HAL增益回调链路.md) — 保留历史文件名，内容与版本仍待核对。
- [10.9 IModuleChangeCallback](10_10.9_IModuleChangeCallback-运行时模块变更通知.md) — 新版本候选材料，不属于本地 v1 接口。
