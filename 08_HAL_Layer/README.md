# 第八篇：Audio HAL 接口与实现边界

> [上一篇：Effects](../07_Effects_Framework/README.md) | [返回导航](../README.md) | [下一篇：AAOS](../09_AAOS_Car_Audio/README.md)

## 本章怎样读

先读 **[8.1 HIDL → C HAL → AudioReach](08_8.1_Audio_HAL双轨架构.md)**，建立本地 Android 12L 主线的接口边界，再去 [15.10 HAL-PAL](../15_PAL_Architecture/15_15.10_HAL-PAL适配层架构.md) 跟踪实际流生命周期。

本地产品源码声明核心 Audio HAL HIDL 6.0，AudioControl AIDL v1；后者不是核心 Audio HAL。HIDL 已有 FMQ 和 Audio Patch，不用“回调 vs FMQ”或“隐式 vs 显式路由”来区分新旧接口。

> 本轮已重写 8.1。其余文章尚在核对，特别是 AIDL 专题不能直接当成本地实现；按版本整理完成前，仅作为原有资料入口。

## 本地主线入口

- [8.1 Audio HAL 边界：从 HIDL 接口到 AudioReach 实现](08_8.1_Audio_HAL双轨架构.md) — 设备与流打开、FMQ、C 回调、Patch、输入与特殊路径边界。
- [15.10 HAL-PAL 适配层](../15_PAL_Architecture/15_15.10_HAL-PAL适配层架构.md) — 普通 PCM 的 Open/start/write/standby 与错误返回。
- [15.99 PAL 对象工厂映射](../15_PAL_Architecture/15_15.99_附录_Stream-Session-Device映射关系.md) — 不同类型怎样选择 Stream、Session 与 Device。

## 原有分主题文章（待逐篇核对）

- [8.2 StreamOut/StreamIn — 音频数据流](08_8.2_StreamOutStreamIn-音频数据流.md)
- [8.3 Audio Patch — 硬件路由](08_8.3_Audio_Patch-硬件路由.md)
- [8.4 Audio Port — 音频端口模型](08_8.4_Audio_Port-音频端口模型.md)
- [8.5 HAL 参数机制](08_8.5_HAL参数机制.md)
- [8.6 Vendor 实现要点](08_8.6_Vendor实现要点.md)
- [8.7 AudioGain — HAL 增益控制模型](08_8.7_AudioGain-HAL增益控制模型.md)
- [8.11 libaudiohal — HAL 适配层](08_8.11_libaudiohal-HAL适配层架构深度解析.md)

## 新版本专题候选（不是本地主线，版本来源待整理）

- [8.8 IModule AIDL 接口](08_8.8_IModule_AIDL接口-HAL核心入口深度解析.md)
- [8.9 ISoundDose AIDL 接口](08_8.9_ISoundDose_AIDL接口-声剂量HAL接口.md)
- [8.10 ITelephony AIDL 接口](08_8.10_ITelephony_AIDL接口-电话音频HAL.md)
