# 第七篇：Effects Framework

> [← 上一篇：第六篇：Audio Policy Engine](../06_Audio_Policy_Engine/README.md) | [返回导航](../README.md) | [下一篇：第八篇：HAL Layer →](../08_HAL_Layer/README.md)

---

## 状态说明

本篇 12 篇文章已按本地 **Android 12L/API 32** 源码完成本轮逐篇核对与重校（行号、摘录与负结论均以本树为准）：

- **版本边界**：Effect HAL 在 12L 仅 HIDL（libaudiohal/impl 无 AIDL 实现）；AIDL Effect HAL（`IFactory.aidl`/`IEffect.aidl` 硬件 HAL）与 `BundleTypes.h`、`NamedAllocRequest`、`onTransact` Binder 统计、checkPort 方向过滤等均为 Android 13+/14 机制，各文中以负结论标注。
- **12L 实况**：FLOAT_EFFECT_CHAIN 本树已默认启用；Spatializer 头部追踪被硬编码关闭（`mSupportsHeadTracking = false`，Spatializer.cpp:221）。
- 产品效果库与实现 UUID 见 7.9（msmnile_au `audio_effects.xml` 为准）。

## 目录

- [7.1 AudioEffect架构总览](07_7.1_AudioEffect架构总览.md)
- [7.2 EffectChain — 效果链](07_7.2_EffectChain.md)
- [7.3 EffectModule — 单个效果实例](07_7.3_EffectModule.md)
- [7.4 EffectModule内部架构详解](07_7.4_EffectModule内部架构详解.md)
- [7.5 EffectChain::process_l()深度解析](07_7.5_EffectChainprocess_l深度解析.md)
- [7.6 Spatializer空间音频架构详解](07_7.6_Spatializer空间音频架构详解.md)
- [7.7 AudioEffect Java API详解](07_7.7_AudioEffect_Java_API详解.md)
- [7.8 常见音效类型完整列表与参数](07_7.8_常见音效类型完整列表与参数.md)
- [7.9 内置效果与Vendor效果](07_7.9_内置效果与Vendor效果.md)
- [7.10 EffectHandle — App音效控制的Binder代理](07_7.10_EffectHandle.md)
- [7.11 DeviceEffectProxy — 设备级音效代理](07_7.11_DeviceEffectProxy.md)
- [7.12 内置音效Native实现 — libeffects](07_7.12_内置音效Native实现.md)
