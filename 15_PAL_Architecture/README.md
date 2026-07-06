# 第十五篇：PAL(Porting Audio Layer)架构深度解析

> [← 上一篇：蓝牙音频](../14_Bluetooth_Audio/README.md) | [返回导航](../README.md) | [下一篇：Vendor+QNX双域架构 →](../16_Vendor_QNX_Architecture/README.md)

---

## 目录

### 第一部分：架构总览与基础定义

- [15.1 概述与目录](15_15.1_概述与目录.md) — PAL定义、历史演进、SA8295虚拟化、源码路径、核心类概览
- [15.2 架构层次图](15_15.2_架构层次图.md) — 六层架构分层、层间接口协议、SA8295跨VM架构、数据流向
- [15.3 API分类总览](15_15.3_API_分类总览.md) — 22个PAL C API分类详解、参数结构体、回调机制、错误码

### 第二部分：类型定义体系

- [15.4 流类型与设备类型](15_15.4_流类型_pal_stream_type_t.md) — pal_stream_type_t(27种)、pal_device_id_t(43种)、pal_audio_fmt_t、pal_stream_attributes

### 第三部分：核心类层次

- [15.5 核心类层次图：Stream-Device-Session](15_15.5_核心类层次图-Stream-Device-Session.md) — Stream基类+9子类、Device基类+18子类、Session基类+5子类+3引擎、状态机、工厂方法、SA8295跨VM路径

### 第四部分：核心管理模块

- [15.6 ResourceManager架构地位](15_15.6_架构地位.md) — 单例模式、流注册/注销、设备路由、FE/BE分配、SSR处理、并发策略
- [15.7 PayloadBuilder核心职责](15_15.7_核心职责.md) — GKV/CKV/TKV构建、模块配置Payload、DSP模块API、kvh2xml映射
- [15.8 HIDL接口定义](15_15.8_HIDL_接口定义.md) — IPAL.hal接口、服务端/客户端实现、IPC机制、SA8295安全模型、HIDL→AIDL演进
- [15.9 配置文件列表](15_15.9_配置文件列表.md) — 7个SoC平台配置、XML结构详解、device_profile/EC ref/VSID、平台差异

### 第五部分：HAL适配与插件体系

- [15.10 HAL-PAL适配层架构](15_15.10_架构.md) — AudioDevice/StreamOut/StreamIn/AudioVoice、参数映射、audio_extn扩展、SA8295车机特化
- [15.11 编解码器插件 (plugins/codecs/)](15_15.11_编解码器插件_pluginscodecs.md) — BT Offload编解码器插件体系
- [15.12 QC audio-alsa：ALSA用户空间封装层](15_15.12_QC_audio-alsa_ALSA层实现.md) — tinyalsa封装与ALSA控制
- [15.13 QC GEF：通用音效框架](15_15.13_QC_GEF_通用音效框架.md) — Generic Effect Framework
- [15.14 QC audio-parsers：音频帧格式解析器](15_15.14_QC_audio-parsers_音频解析器.md) — 压缩音频帧解析
- [15.15 QC CAPIv2：编解码器统一接口标准](15_15.15_QC_CAPIv2_编解码接口.md) — Codec API v2接口
- [15.16 QC CASA：校准配置与数据管理工具](15_15.16_QC_CASA_校准配置工具.md) — ACDB校准数据管理
- [15.17 QC Sound Trigger HAL：语音触发HAL实现](15_15.17_QC_Sound_Trigger_HAL.md) — SVA语音唤醒

### 附录

- [15.99 Stream-Session-Device映射关系](15_15.99_附录_Stream-Session-Device映射关系.md) — 三维映射表、音频路径类型、EC参考映射、并发流组合、SA8295车机特有映射