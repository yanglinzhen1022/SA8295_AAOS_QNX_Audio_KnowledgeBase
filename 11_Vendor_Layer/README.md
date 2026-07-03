# 第十一篇：Vendor Layer — OEM音频定制层

> [← 上一篇](../10_AudioControl_HAL/README.md) | [返回导航](../README.md) | [下一篇 →](../12_Audio_Focus_Deep_Dive/README.md)

---

## 概述

Vendor Layer是OEM定制AAOS音频系统的核心层，涵盖6大XML配置文件的完整标签体系、解析流程、OEM定制指南、AIDL/HIDL HAL实现架构。本篇从源码级深度解析每个配置标签、属性取值、Car端与Phone端差异，并提供OEM 7步定制流程和完整验证清单。

## 目录

### 配置文件体系
- [11.1 配置文件体系总览](11_11.1_配置文件体系总览.md) — 6大XML配置文件角色、关系、依赖
- [11.2 audio_policy_configuration.xml — 核心配置](11_11.2_audio_policy_configuration.xml-核心配置.md) — module/mixPort/devicePort/route详解
- [11.3 audio_policy_volumes.xml — 音量曲线](11_11.3_audio_policy_volumes.xml-音量曲线.md) — volume/ref段点/设备类别曲线
- [11.4 audio_policy_engine_configuration.xml — 策略引擎配置](11_11.4_audio_policy_engine_configuration.xml-策略引擎配置.md) — ProductStrategy/VolumeGroup/焦点交互
- [11.5 car_audio_configuration.xml — AAOS车载配置](11_11.5_car_audio_configuration.xml-AAOS车载配置.md) — Zone/VolumeGroup/Context/Bus映射

### 解析与定制
- [11.6 配置解析流程](11_11.6_配置解析流程.md) — Serializer/CarAudioZonesHelper解析链
- [11.7 OEM定制指南](11_11.7_OEM定制指南.md) — 7步定制流程/Bus分配/HAL实现模板
- [11.8 audio_policy_configuration.xml 属性详解](11_11.8_audio_policy_configuration.xml_属性详解.md) — 全标签属性/flags映射/Car vs Phone对比
- [11.9 car_audio_configuration.xml 深度解析](11_11.9_car_audio_configuration.xml_深度解析.md) — CarAudioContext/Duck策略/v3 zoneConfigs

### 定制矩阵与HAL
- [11.10 OEM定制点完整矩阵](11_11.10_OEM定制点完整矩阵.md) — 6大配置+HAL+Framework全部定制点
- [11.11 AIDL/HIDL Vendor HAL实现架构](11_11.11_AIDLHIDL_Vendor_HAL实现架构.md) — IModule 30+方法/Vendor扩展/编译配置

---

> [← 上一篇](../10_AudioControl_HAL/README.md) | [返回导航](../README.md) | [下一篇 →](../12_Audio_Focus_Deep_Dive/README.md)