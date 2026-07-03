# 第十篇：AudioControl HAL

> [← 上一篇](../09_AAOS_Car_Audio/README.md) | [返回导航](../README.md) | [下一篇 →](../11_Vendor_Layer/README.md)

---

## 目录

- [10.1 AudioControl HAL总览](10_10.1_AudioControl_HAL总览.md) — 版本演进、架构总览、双向交互模式、Feature矩阵
- [10.2 核心接口](10_10.2_核心接口.md) — IAudioControl AIDL 10个方法、IFocusListener 4个方法、IAudioGainCallback、IModuleChangeCallback、Reasons枚举
- [10.3 焦点回调流程](10_10.3_焦点回调流程.md) — HAL→CarSvc外部焦点请求、HalAudioFocus两层映射、ZoneId传递、Metadata降级
- [10.4 Ducking机制](10_10.4_Ducking机制.md) — CarDucking实现、CarDuckingInfo数据结构、CarDuckingUtils地址提取、CarFocusCallback触发
- [10.5 Muting机制](10_10.5_Muting机制.md) — CarVolumeGroupMuting、setRestrictMuting、carMuteChanged触发、MutingInfo结构
- [10.6 AudioControlWrapperAidl](10_10.6_AudioControlWrapperAidl-AIDL适配层架构.md) — AIDL适配层、3个内部Wrapper、版本协商、重试机制、binderDied恢复
- [10.7 HalAudioFocus](10_10.7_HalAudioFocus-外部焦点请求管理.md) — 外部焦点请求管理、SparseArray两层映射、请求/释放/死亡恢复全流程
- [10.8 AudioControlFactory与HIDL Wrapper](10_10.8_IAudioGainCallback-HAL增益回调链路.md) — 工厂模式服务发现、V1/V2与AIDL对比、Feature降级策略、死亡恢复差异
- [10.9 IModuleChangeCallback](10_10.9_IModuleChangeCallback-运行时模块变更通知.md) — 运行时模块变更通知、AudioPort→HalAudioDeviceInfo转换、增益阶段重计算、外推算法