# 第十八篇：AGM / GSL 架构

> [返回总导航](../README.md) · [前置：PAL](../15_PAL_Architecture/README.md) · [关联：QNX 与平台实现](../16_Vendor_QNX_Architecture/README.md)

本章承接 PAL 的 Session 和 tinyalsa 插件，讲解 AudioReach 图管理边界。主线为本地 Android 12L/API 32 + SA8295；不将库依赖、IPC、跨 VM 通信和 DSP 执行画成同一种边界。

## 当前阅读路线

1. [15.2 PAL 层次与控制/数据分工](../15_PAL_Architecture/15_15.2_架构层次图.md)。
2. [15.12 tinyalsa 与 AGM 插件](../15_PAL_Architecture/15_15.12_QC_audio-alsa_ALSA层实现.md)：从 PCM/mixer 操作进入 AGM。
3. [18.1 AGM 会话与音频图](18_18.1_AGM会话与音频图.md)：会话池、metadata/AIF、open/config/prepare/start、读写与清理。

18.1 已按本地服务核心及 HIDL 两侧源码整理。它不包含已完成的 GSL 跨 VM buffer/协议、QNX 后端、DSP 模块或板级线路验证。相关原文仍在 16 章作为待核对资料保留；不创建空的后续文章。

原 [16.10 AGM 入口](../16_Vendor_QNX_Architecture/16_16.10_AGMAudio_Graph_Manager深度解.md) 保留兼容链接，深入讲解统一放在本章。
