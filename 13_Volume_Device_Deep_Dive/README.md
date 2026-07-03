# 第十三篇：Volume & Device 深度解析

> [← 上一篇](../12_Audio_Focus_Deep_Dive/README.md) | [返回导航](../README.md) | [下一篇 →](../14_Bluetooth_Audio/README.md)

---

## 概述

本篇深度解析AOSP14音频子系统中的**音量控制（Volume）**、**设备管理（Device）**和**听力保护（CSD）**三大核心机制，涵盖Java Framework层到Native C++层的完整技术链路。

**核心源码规模**：
- [`AudioService.java`](frameworks/base/services/core/java/com/android/server/audio/AudioService.java) — 13267行，Volume/Device核心逻辑
- [`AudioDeviceBroker.java`](frameworks/base/services/core/java/com/android/server/audio/AudioDeviceBroker.java) — 2357行，设备路由管理
- [`AudioDeviceInventory.java`](frameworks/base/services/core/java/com/android/server/audio/AudioDeviceInventory.java) — ~2350行，设备连接状态管理
- [`SoundDoseHelper.java`](frameworks/base/services/core/java/com/android/server/audio/SoundDoseHelper.java) — 1157行，CSD声剂量Java层
- [`MelProcessor.cpp`](system/media/audio_utils/MelProcessor.cpp) — 414行，MEL计算Native层
- [`MelAggregator.cpp`](system/media/audio_utils/MelAggregator.cpp) — 282行，CSD聚合Native层

---

## 目录

### Volume控制
- [13.1 Volume状态机](13_13.1_Volume状态机.md) — VolumeStreamState内部类、adjustVolume/setStreamVolume完整流程、安全音量状态机、固定音量设备、音量双路径应用

### Device管理
- [13.2 Device状态机](13_13.2_Device状态机.md) — AudioDeviceBroker/AudioDeviceInventory架构、设备连接/断开流程、BtHelper蓝牙设备、绝对音量同步
- [13.3 Focus+Device+Volume联合交互场景](13_13.3_Focus+Device+Volume联合交互场景.md) — 三者联动深度分析、音量随设备切换、Focus影响路由
- [13.4 外部音频设备 — USB/HDMI/有线设备管理](13_13.4_外部音频设备-USBHDMI有线设备管理.md) — USB/HDMI设备路由、有线设备优先级、外部设备音量策略

### CSD听力保护
- [13.5 SoundDose与CSD声剂量管理](13_13.5_SoundDose与CSD声剂量管理.md) — IEC 62368-1标准、安全音量状态机、CSD双路径(Framework+HAL)、1X/5X警告分级
- [13.6 MelProcessor — MEL计算引擎](13_13.6_MelProcessor-MEL计算引擎.md) — A-weighting IIR滤波器、process()算法、MelWorker异步回调、RS1/RS2瞬时检测
- [13.7 MelAggregator — CSD聚合器](13_13.7_MelAggregator-CSD聚合器.md) — 多流MEL合并算法、CSD归一化计算、7天滚动窗口、跨重启恢复

---

> [← 上一篇](../12_Audio_Focus_Deep_Dive/README.md) | [返回导航](../README.md) | [下一篇 →](../14_Bluetooth_Audio/README.md)
