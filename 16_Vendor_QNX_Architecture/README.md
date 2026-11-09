# 第十六篇：SA8295 Vendor+QNX双域音频架构深度解析

> [← 上一篇：PAL架构深度解析](../15_PAL_Architecture/README.md) | [返回导航](../README.md) | [下一篇：调试方法与OEM定制指南 →](../21_Debug_and_OEM_Guide/README.md)

---

## 阅读范围

AGM 会话/图深度说明已移至 [17.1](../17_AGM_GSL_Architecture/17_17.1_AGM会话与音频图.md)，16.10 保留兼容入口。[16.6 校准模型与初始化](16_16.6_ACDB校准体系.md) → [16.7 双域数据库与路径](16_16.7_ACDB校准数据Android与QNX双域共享.md) 已按本地 AudioReach 源码重写，区分本地初始库、VM 握手和远端追加库。

[16.12 双域全景](16_16.12_Android+QNX双域架构总结.md) 也已重写，跨 VM 深入机制见 [17.2](../17_AGM_GSL_Architecture/17_17.2_GSL跨VM控制与缓冲区.md)。[16.11](16_16.11_SessionGsl与GSL接口.md) 已按声明/工厂/构建边界重写，[16.14](16_16.14_GSLGraph_Service_Layer内部架.md) 保留 GSL 兼容入口，普通图/命令/数据路径详见 [17.3](../17_AGM_GSL_Architecture/17_17.3_GSL图生命周期与DSP命令.md)。16.16–16.19 已按 AudioReach 后端/服务/图管理入口/GPR 分工调整，文件名保留历史标识；不再以旧 AMS/APR 主体讲当前通路。除上述入口外，本章原文尚待当前 Android 12L + SA8295 AudioReach 主线逐篇核对；尤其不能将 IPC 名字、配置目录或库名当作执行域/实机效果证明。已整理文章不代表已完成前端预编译实现、实机服务启动或板级验证。

## 目录

- [16.1 概述](16_16.1_概述.md)
- [16.2 auto-audiod守护进程](16_16.2_auto-audiod守护进程.md)
- [16.3 AutoPower与VHAL集成](16_16.3_AutoPower与VHAL集成.md)
- [16.4 Silent Boot监控](16_16.4_Silent_Boot监控.md)
- [16.5 audio-chime早期提示音](16_16.5_audio-chime早期提示音.md)
- [16.6 ACDB校准体系](16_16.6_ACDB校准体系.md)
- [16.7 ACDB校准数据Android与QNX双域共享](16_16.7_ACDB校准数据Android与QNX双域共享.md)
- [16.8 ALSA UCM配置](16_16.8_ALSA_UCM配置.md)
- [16.9 auto-casa-xml配置](16_16.9_auto-casa-xml配置.md)
- [16.10 AGM 平台定位与迁移入口](16_16.10_AGMAudio_Graph_Manager深度解.md) → [17.1 会话与音频图](../17_AGM_GSL_Architecture/17_17.1_AGM会话与音频图.md)
- [16.11 GSL接口与PAL Session的图管理](16_16.11_SessionGsl与GSL接口.md)（文件名保留历史标识；本地工厂边界见 [15.99](../15_PAL_Architecture/15_15.99_附录_Stream-Session-Device映射关系.md)，不据此断言类的历史废弃过程）
- [16.12 Android+QNX双域架构总结](16_16.12_Android+QNX双域架构总结.md)
- [16.13 Primary HAL AudioReach版深度解析](16_16.13_Primary_HALAudioReach版深度解.md)
- [16.14 GSL 平台定位与兼容入口](16_16.14_GSLGraph_Service_Layer内部架.md) → [17.3 图生命周期与 DSP 命令](../17_AGM_GSL_Architecture/17_17.3_GSL图生命周期与DSP命令.md)
- [16.15 常见问题与解答Q&A](16_16.15_常见问题与解答Q&A.md)
- [16.16 QNX AudioReach 后端与用户态/内核边界](16_16.16_QNX_audio_driver_vm_VM音频驱动层.md)
- [16.17 AR audio_service：装配、并行初始化与 SSR](16_16.17_QNX_audio_service_vm_VM音频服务.md)
- [16.18 旧 AMS 兼容入口与当前图管理路线](16_16.18_QNX_ams_lib_音频管理服务库.md)
- [16.19 AudioReach GPR/GLink：路由、连接与所有权](16_16.19_QNX_apr_lib_APR协议库.md)
- [16.20 A2B总线音频（兼容入口）](16_16.20_QNX_audio_a2b_A2B总线音频.md)
- [16.21 audio_dac PCM1681 DAC控制](16_16.21_QNX_audio_dac_DAC音频驱动.md)
- [16.22 audio_expander I2C音频扩展器](16_16.22_QNX_audio_expander_音频扩展器.md)