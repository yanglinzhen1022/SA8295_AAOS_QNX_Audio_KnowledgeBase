# 第五篇：AudioFlinger

> [← 上一篇](../04_Native_Framework_Layer/README.md) | [返回导航](../README.md) | [下一篇 →](../06_Audio_Policy_Engine/README.md)

---

## 推荐阅读

本章以本地 Android 12L/API 32 产品实现为主线，按以下顺序建立模型：

1. [5.1 AudioFlinger 职责与对象生命周期](05_5.1_AudioFlinger-音频数据面引擎.md)：Track、I/O、线程与 HAL 是不同对象。
2. [4.4 共享内存协议](../04_Native_Framework_Layer/04_4.4_共享内存机制深度解析.md)：客户端供数/取数与发布顺序。
3. [5.9 普通 PCM 播放](05_5.9_音乐播放全栈调用链.md)：选择、创建、start、write、混音与 NormalSink。
4. [5.10 普通 PCM 录音](05_5.10_录音全栈调用链.md)：FAST 重选、两阶段启动、转换分发与 silenced。
5. [6.1 APS](../06_Audio_Policy_Engine/06_6.1_AudioPolicyService-控制面入口.md) / [6.2 APM](../06_Audio_Policy_Engine/06_6.2_AudioPolicyManager-策略核心实现.md)，然后进入 [8.1 HAL](../08_HAL_Layer/08_8.1_Audio_HAL双轨架构.md)。

上述入口已按本地源码重写。以下其余旧文仍待本轮核对，可能混有不同版本能力；存在源码分支不代表当前产品已启用，源码检查也不代表实机验证。

## 文章目录

- [5.1 AudioFlinger — 音频数据面引擎](05_5.1_AudioFlinger-音频数据面引擎.md)
- [5.2 Thread体系 — AudioFlinger的核心执行单元](05_5.2_Thread体系-AudioFlinger的核心执行单元.md)
- [5.3 PlaybackThread核心循环](05_5.3_PlaybackThread核心循环.md)
- [5.4 FastMixer — 低延迟混音路径](05_5.4_FastMixer-低延迟混音路径.md)
- [5.5 Buffer管理与共享内存](05_5.5_Buffer管理与共享内存.md)
- [5.6 PatchPanel — 音频路由管理](05_5.6_PatchPanel-音频路由管理.md)
- [5.7 Track/Record — 音频流端点](05_5.7_TrackRecord-音频流端点.md)
- [5.8 Thread继承体系与匹配规则](05_5.8_Thread继承体系与匹配规则.md)
- [5.9 普通 PCM 播放：从 AudioTrack 到 AudioFlinger 输出](05_5.9_音乐播放全栈调用链.md)
- [5.10 普通 PCM 录音：AudioRecord、RecordTrack 与输入线程](05_5.10_录音全栈调用链.md)
- [5.11 AudioMixer与Resampler — 混音引擎核心](05_5.11_AudioMixer与Resampler-混音引擎核心.md)
- [5.12 DeviceEffectManager — 设备级音效管理](05_5.12_DeviceEffectManager-设备级音效管理.md)
- [5.13 MelReporter — MEL声暴露报告](05_5.13_MelReporter-MEL声暴露报告.md)
- [5.14 SoundDoseManager — CSD声剂量管理](05_5.14_SoundDoseManager-CSD声剂量管理.md)
- [5.15 PatchCommandThread — Patch异步命令线程](05_5.15_PatchCommandThread-Patch异步命令线程.md)
- [5.16 BufLog — 缓冲区调试日志](05_5.16_BufLog-缓冲区调试日志.md)
- [5.17 SpatializerThread与BitPerfectThread — 特殊输出线程](05_5.17_SpatializerThread与BitPerfectThread-特殊输出线程.md)
