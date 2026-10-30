# 第十八篇：AGM / GSL 架构

> [返回总导航](../README.md) · [前置：PAL](../15_PAL_Architecture/README.md) · [关联：QNX 与平台实现](../16_Vendor_QNX_Architecture/README.md)

本章承接 PAL 的 Session 和 tinyalsa 插件，讲解 AudioReach 图管理边界。主线为本地 Android 12L/API 32 + SA8295；不将库依赖、IPC、跨 VM 通信和 DSP 执行画成同一种边界。

## 当前阅读路线

1. [15.2 PAL 层次与控制/数据分工](../15_PAL_Architecture/15_15.2_架构层次图.md)。
2. [15.12 tinyalsa 与 AGM 插件](../15_PAL_Architecture/15_15.12_QC_audio-alsa_ALSA层实现.md)：从 PCM/mixer 操作进入 AGM。
3. [18.1 AGM 会话与音频图](18_18.1_AGM会话与音频图.md)：会话池、metadata/AIF、open/config/prepare/start、读写与清理。

4. [16.6 AudioReach 校准](../16_Vendor_QNX_Architecture/16_16.6_ACDB校准体系.md) → [16.7 双域数据库加载](../16_Vendor_QNX_Architecture/16_16.7_ACDB校准数据Android与QNX双域共享.md)：GKV/CKV/TKV、初始化文件列表、QNX 追加数据库与路径边界。

18.1 已按本地服务核心及 HIDL 两侧源码整理；校准两篇补充了数据库侧机制。这不等于完整 GSL 跨 VM PCM buffer/协议、DSP 模块或板级线路已验证。16 章其他原文仍待核对；不创建空的后续文章。

原 [16.10 AGM 入口](../16_Vendor_QNX_Architecture/16_16.10_AGMAudio_Graph_Manager深度解.md) 保留兼容链接，深入讲解统一放在本章。
