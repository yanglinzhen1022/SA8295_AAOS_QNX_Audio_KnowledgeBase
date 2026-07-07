# 第十六篇：SA8295 Vendor+QNX双域音频架构深度解析

> [← 上一篇：PAL架构深度解析](../15_PAL_Architecture/README.md) | [返回导航](../README.md) | [下一篇：调试方法与OEM定制指南 →](../17_Debug_and_OEM_Guide/README.md)

---

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
- [16.10 AGM Audio Graph Manager深度解析](16_16.10_AGMAudio_Graph_Manager深度解.md)
- [16.11 GSL接口与PAL Session的图管理](16_16.11_SessionGsl与GSL接口.md)（注：文件名保留历史标识 SessionGsl，SessionGsl 类已废弃，实走 SessionAlsaPcm/SessionAgm）
- [16.12 Android+QNX双域架构总结](16_16.12_Android+QNX双域架构总结.md)
- [16.13 Primary HAL AudioReach版深度解析](16_16.13_Primary_HALAudioReach版深度解.md)
- [16.14 GSL Graph Service Layer内部架构](16_16.14_GSLGraph_Service_Layer内部架.md)
- [16.15 常见问题与解答Q&A](16_16.15_常见问题与解答Q&A.md)
- [16.16 QNX audio_driver_vm VM音频驱动层](16_16.16_QNX_audio_driver_vm_VM音频驱动层.md)
- [16.17 QNX audio_service_vm VM音频服务](16_16.17_QNX_audio_service_vm_VM音频服务.md)
- [16.18 QNX ams_lib 音频管理服务库](16_16.18_QNX_ams_lib_音频管理服务库.md)
- [16.19 QNX apr_lib APR协议库](16_16.19_QNX_apr_lib_APR协议库.md)
- [16.20 QNX audio_a2b A2B总线音频](16_16.20_QNX_audio_a2b_A2B总线音频.md)
- [16.21 QNX audio_dac DAC音频驱动](16_16.21_QNX_audio_dac_DAC音频驱动.md)
- [16.22 QNX audio_expander 音频扩展器](16_16.22_QNX_audio_expander_音频扩展器.md)