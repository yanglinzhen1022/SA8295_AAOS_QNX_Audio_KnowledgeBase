# 第十四篇：Bluetooth Audio 深度解析（本树 12L：A2DP / HFP-SCO / ASHA；LE Audio 为兼容入口）

> **树边界**：本树蓝牙 = `system/bt` 原生栈 + 经典蓝牙应用 `packages/apps/Bluetooth`（A2dp/HearingAid/LE Audio Java Profile 服务在此）；**无** 13+ 的 mainline 模块 `packages/modules/Bluetooth` 与 14+ 的 AIDL audio `IBluetooth*`。LE Audio 在 12L 为早期未完成实现（见 14.3）；14.7–14.10 的 13+/14+ 机制（LC3 协商、BASS/Broadcast、VCP）按版本边界保留为兼容入口，原文可从 Git 历史恢复。

> [← 上一篇](../13_Volume_Device_Deep_Dive/README.md) | [返回导航](../README.md) | [下一篇 →](../15_PAL_Architecture/README.md)

---

## 目录

- [14.1 蓝牙音频协议总览](14_14.1_蓝牙音频协议总览.md)
- [14.2 A2DP — 高级音频分发协议](14_14.2_A2DP-高级音频分发协议.md)
- [14.3 LE Audio — 低功耗蓝牙音频（兼容入口：本树仅HAL接口雏形）](14_14.3_LE_Audio-低功耗蓝牙音频.md)
- [14.4 SCO/HFP — 通话语音](14_14.4_SCOHFP-通话语音.md)
- [14.5 Hearing Aid — 助听器](14_14.5_Hearing_Aid-助听器.md)
- [14.6 蓝牙音频设备与AudioDeviceBroker交互](14_14.6_蓝牙音频设备与AudioDeviceBroker交互.md)
- [14.7 LE Audio Profile深度解析（兼容入口，Android 13+）](14_14.7_LE_Audio_Profile深度解析.md)
- [14.8 LC3编码参数与配置（兼容入口，liblc3=Android 13+）](14_14.8_LC3编码参数与配置.md)
- [14.9 LE Audio广播音频(Broadcast Audio)（兼容入口，Android 14）](14_14.9_LE_Audio广播音频Broadcast_Audio.md)
- [14.10 LE Audio与Classic Bluetooth共存策略（兼容入口，核心机制=Android 13/14）](14_14.10_LE_Audio与Classic_Bluetooth共存策略.md)
- [14.11 蓝牙音频对比总结](14_14.11_蓝牙音频对比总结.md)
