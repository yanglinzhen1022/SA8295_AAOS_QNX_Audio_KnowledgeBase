# 第十三篇：Volume & Device 深度解析

> [← 上一篇](../12_Audio_Focus_Deep_Dive/README.md) | [返回导航](../README.md) | [下一篇 →](../14_Bluetooth_Audio/README.md)

---

## 概述

本篇深度解析AOSP14音频子系统中的**音量控制（Volume）**、**设备管理（Device）**和**听力保护（CSD）**三大核心机制，涵盖Java Framework层到Native C++层的完整技术链路。

**核心源码规模**：
- [`AudioService.java`](frameworks/base/services/core/java/com/android/server/audio/AudioService.java) — 10861行（wc -l，12L），Volume/Device核心逻辑
- [`AudioDeviceBroker.java`](frameworks/base/services/core/java/com/android/server/audio/AudioDeviceBroker.java) — 1961行（wc -l，12L），设备路由管理
- [`AudioDeviceInventory.java`](frameworks/base/services/core/java/com/android/server/audio/AudioDeviceInventory.java) — 1430行（wc -l，12L），设备连接状态管理
- （12L：安全音量在 `AudioService.java` 内实现，无独立 SoundDoseHelper 类）
- （Android 14+ 的 `MelProcessor.cpp`/`MelAggregator.cpp` 不在本树，见13.6/13.7兼容入口）

---

## 目录

### Volume控制
- [13.1 Volume状态机](13_13.1_Volume状态机.md) — VolumeStreamState内部类、adjustVolume/setStreamVolume完整流程、安全音量状态机、固定音量设备、音量双路径应用

### Device管理
- [13.2 Device状态机](13_13.2_Device状态机.md) — AudioDeviceBroker/AudioDeviceInventory架构、设备连接/断开流程、BtHelper蓝牙设备、绝对音量同步
- [13.3 Focus+Device+Volume联合交互场景](13_13.3_Focus+Device+Volume联合交互场景.md) — 三者联动深度分析、音量随设备切换、Focus影响路由
- [13.4 外部音频设备 — USB/HDMI/有线设备管理](13_13.4_外部音频设备-USBHDMI有线设备管理.md) — USB/HDMI设备路由、有线设备优先级、外部设备音量策略

### CSD听力保护（Android 14+，兼容入口）
- [13.5 SoundDose与CSD声剂量管理](13_13.5_SoundDose与CSD声剂量管理.md) — 兼容入口：Android 14+机制不在本树；12L安全音量等价能力对照
- [13.6 MelProcessor — MEL计算引擎](13_13.6_MelProcessor-MEL计算引擎.md) — 兼容入口：Android 14+，不在本树
- [13.7 MelAggregator — CSD聚合器](13_13.7_MelAggregator-CSD聚合器.md) — 兼容入口：Android 14+，不在本树

---

> [← 上一篇](../12_Audio_Focus_Deep_Dive/README.md) | [返回导航](../README.md) | [下一篇 →](../14_Bluetooth_Audio/README.md)
