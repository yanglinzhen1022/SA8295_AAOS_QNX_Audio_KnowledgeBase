# 源码类名 → 讲解章节 反向索引（CLASS_INDEX）

> 对照源码阅读时，从核心类/接口名快速定位讲解所在章。按层分组，`章节` 列指主要讲解章。[← 返回总目录](README.md) ｜ 目录级速查见 [01.5 关键源码目录速查](01_Architecture_Overview/01_1.5_关键源码目录速查.md)

> **范围提示：** 本地主线为 Android 12L + SA8295 AudioReach。全库 21 章正文均已按本地源码重写并经独立评审闭环（记录见 [docs/superpowers/reports/2026-11-05-final-completion-report.md](docs/superpowers/reports/2026-11-05-final-completion-report.md)）；带直达链接的条目可跳转讲解文章，其余为章号引用。标注 13+/14+ 的机制为树外版本内容，不代表本地均有实现。

---

## 应用层 / Java 框架（02–03）

| 类 / 接口 | 章节 |
|-----------|------|
| AudioTrack (Java) / AudioRecord (Java) / AudioManager | 02 |
| AAudioStream / oboeservice | 02 |
| MediaPlayer / NuPlayer / MediaCodec / MediaExtractor | 02 |
| AudioService | 03 |
| MediaFocusControl | 03 / 12 |
| PlaybackActivityMonitor | 03 / 12 |
| AudioDeviceBroker | 03 / 13 / 14 |

## Native 框架（04）

| 类 / 接口 | 章节 |
|-----------|------|
| AudioTrack.cpp / AudioRecord.cpp | [5.9 播放](05_AudioFlinger/05_5.9_音乐播放全栈调用链.md) / [5.10 录音](05_AudioFlinger/05_5.10_录音全栈调用链.md) |
| AudioSystem | 04（完整客户端接口仍待整理）；策略协作见 [6.1](06_Audio_Policy_Engine/06_6.1_AudioPolicyService-控制面入口.md) |
| AudioTrackClientProxy / AudioRecordClientProxy / ServerProxy / audio_track_cblk_t | [4.4 共享内存协议](04_Native_Framework_Layer/04_4.4_共享内存机制深度解析.md) |
| IAudioFlingerService (AIDL) / AudioFlingerServerAdapter | [5.1 Framework 服务边界](05_AudioFlinger/05_5.1_AudioFlinger-音频数据面引擎.md) |
| ChannelMix / resampler / fifo (audio_utils) | 04 / 13(MEL) |

## AudioFlinger（05）

| 类 / 接口 | 章节 |
|-----------|------|
| AudioFlinger / ThreadBase / MixerThread / RecordThread | [5.1 对象与线程](05_AudioFlinger/05_5.1_AudioFlinger-音频数据面引擎.md)；具体周期见 5.9 / 5.10 |
| FastMixer | 05 |
| AudioMixer / AudioResampler | 05 |
| Track / TrackHandle | [5.9 播放端点生命周期](05_AudioFlinger/05_5.9_音乐播放全栈调用链.md) |
| RecordTrack / RecordHandle | [5.10 录音端点生命周期](05_AudioFlinger/05_5.10_录音全栈调用链.md) |
| PatchPanel / PatchCommandThread | 05 |
| MelReporter / SoundDoseManager | 05 / 13 |
| SpatializerThread / BitPerfectThread | 05 |
| DeviceEffectManager | 05 / 07 |

## Audio Policy / Engine（06）

| 类 / 接口 | 章节 |
|-----------|------|
| AudioPolicyService / AudioPolicyClient / IAudioPolicyService | [6.1 接口、权限与回调](06_Audio_Policy_Engine/06_6.1_AudioPolicyService-控制面入口.md) |
| AudioPolicyManager / TrackClientDescriptor / RecordClientDescriptor | [6.2 选择与资源生命周期](06_Audio_Policy_Engine/06_6.2_AudioPolicyManager-策略核心实现.md) |
| EngineBase / EngineInterface | 06 |
| ProductStrategy / VolumeGroup / VolumeCurve | 06 / 13 |
| EngineConfigurable (PFW) / EngineConfig | 06 |

## 音效框架（07）

| 类 / 接口 | 章节 |
|-----------|------|
| AudioEffect / EffectChain / EffectModule / EffectHandle | 07 |
| Spatializer | 07 / 05 |
| LoudnessEnhancer / AEC / NS | 07 |
| DeviceEffectProxy | 07 / 05 |

## HAL（08）/ AudioControl HAL（10）

| 类 / 接口 | 章节 |
|-----------|------|
| IDevicesFactory / IDevice / IStreamOut / IStreamIn (HIDL) | [8.1 本地 HAL 边界](08_HAL_Layer/08_8.1_Audio_HAL双轨架构.md) |
| DevicesFactoryHalHidl / DeviceHalHidl / StreamOutHalHidl | [8.1 libaudiohal 与 FMQ](08_HAL_Layer/08_8.1_Audio_HAL双轨架构.md) |
| WriteThread / HIDL adapter Device、StreamOut | [8.1 FMQ 到 C HAL](08_HAL_Layer/08_8.1_Audio_HAL双轨架构.md) |
| IModule / IStreamOut / IStreamIn (AIDL) | 08（新版本候选资料，非本地主线） |
| StreamDescriptor / AudioGain / AudioPort / AudioPatch | 08（需区分接口版本） |
| ISoundDose / ITelephony (AIDL) | 08（新版本候选资料，非本地主线） |
| IAudioControl (AIDL v1) / IFocusListener | [10.1 本地控制接口](10_AudioControl_HAL/10_10.1_AudioControl_HAL总览.md) |
| IAudioGainCallback / IModuleChangeCallback | 10（新版候选，非本地 AudioControl v1） |
| AudioControlWrapperAidl | [10.6 类型、能力与死亡处理](10_AudioControl_HAL/10_10.6_AudioControlWrapperAidl-AIDL适配层架构.md) |
| HalAudioFocus | [10.7 zone/usage 请求表与清理边界](10_AudioControl_HAL/10_10.7_HalAudioFocus-外部焦点请求管理.md) |

## AAOS 车载（09）/ 焦点·音量·设备深入（12–13）

| 类 / 接口 | 章节 |
|-----------|------|
| CarAudioService | [9.2 初始化、用户与控制协作](09_AAOS_Car_Audio/09_9.2_CarAudioService-车载音频核心服务.md) |
| CarAudioZone | [9.3 区域对象、设备与活动 Context](09_AAOS_Car_Audio/09_9.3_CarAudioZone-多Zone音频管理.md) |
| CarAudioZonesHelper / CarAudioZonesValidator | [9.18 V1/V2 配置解析与验证](09_AAOS_Car_Audio/09_9.18_CarAudioZonesHelper-XML配置解析.md) |
| CarAudioContext | 对象映射见 [9.1](09_AAOS_Car_Audio/09_9.1_AAOS音频系统总览.md)；9.6 专篇仍待核对 |
| CarAudioZoneConfig / CarAudioGainMonitor | 09（旧索引中的新版候选，非已确认本地实现） |
| CarAudioFocus / FocusEntry | [9.4 holders、losers、blockers 与 delayed](09_AAOS_Car_Audio/09_9.4_CarAudioFocus-车载焦点管理.md) |
| CarZonesAudioFocus | [9.10 区域选择与分发](09_AAOS_Car_Audio/09_9.10_CarZonesAudioFocus-多Zone焦点分发器.md) |
| FocusInteraction | [12.6 有方向的交互矩阵](12_Audio_Focus_Deep_Dive/12_12.6_AAOS焦点交互矩阵.md) |
| CarVolumeGroup / CarAudioDeviceInfo / CarAudioSettings | [9.5 索引、设备 gain 与持久化](09_AAOS_Car_Audio/09_9.5_CarVolumeGroup-车载音量组.md) |
| CarVolume | 09 / 13（待核对） |
| CarAudioDynamicRouting | [9.11 按地址构建 AudioMix](09_AAOS_Car_Audio/09_9.11_CarAudioDynamicRouting-动态路由构建.md) |
| CarDucking / CarVolumeGroupMuting | 控制边界见 [10.1](10_AudioControl_HAL/10_10.1_AudioControl_HAL总览.md)；详细原文待核对 |
| FocusRequester / FadeOutManager | 12 |
| VolumeController / MelProcessor / MelAggregator | 13 |

## 蓝牙（14）

| 类 / 接口 | 章节 |
|-----------|------|
| A2dpService / LeAudioService / LeAudioCodecConfig | 14 |
| BtHelper / HearingAidService | 14 |

## Vendor / PAL / QNX（15–17）

| 类/ 接口 | 章节 |
|-----------|------|
| AudioDevice / StreamPrimary / StreamOutPrimary / StreamInPrimary (AR HAL) | [15.10 HAL-PAL 适配](15_PAL_Architecture/15_15.10_HAL-PAL适配层架构.md) |
| pal_stream_open / start / write / stop / close | [15.10 PCM 生命周期与错误返回](15_PAL_Architecture/15_15.10_HAL-PAL适配层架构.md) |
| Stream::create / StreamPCM / StreamCompress / StreamInCall / StreamNonTunnel | [15.99 Stream 工厂](15_PAL_Architecture/15_15.99_附录_Stream-Session-Device映射关系.md) |
| Session::makeSession / SessionAlsaPcm / SessionAlsaCompress / SessionAlsaVoice / SessionAgm | [15.99 Session 四分支](15_PAL_Architecture/15_15.99_附录_Stream-Session-Device映射关系.md) |
| StreamSoundTrigger / SoundTriggerEngineGsl / StreamACD / ContextDetectionEngine | [15.99 检测引擎与 Session 的持有关系](15_PAL_Architecture/15_15.99_附录_Stream-Session-Device映射关系.md) |
| Device::getInstance / Speaker (PAL) | [15.99 Device 分派与复用](15_PAL_Architecture/15_15.99_附录_Stream-Session-Device映射关系.md) |
| ResourceManager | [15.6 资源、设备配置与状态分发](15_PAL_Architecture/15_15.6_ResourceManager-PAL核心管理模块.md) |
| PayloadBuilder / selector / GKV / CKV / TKV | [15.7 KV 匹配与模块 payload](15_PAL_Architecture/15_15.7_PayloadBuilder-载荷构建器.md) |
| SNDPARSER / USECASE_XML_FILE / RM getVendorConfigPath | [15.9 配置消费者与选择路径](15_PAL_Architecture/15_15.9_配置文件列表.md) |
| ContextManager / SndCardMonitor | 15 |
| tinyalsa pcm_ops / pcm_plugin_ops / AGM PCM 与 mixer 插件 | [15.12 操作分派与计量](15_PAL_Architecture/15_15.12_QC_audio-alsa_ALSA层实现.md) |
| AGM / session_obj / aif / graph_obj / device_obj | [17.1 metadata、会话与图生命周期](17_AGM_GSL_Architecture/17_17.1_AGM会话与音频图.md) |
| gsl_graph / gsl_graph_gkv_node / gsl_subgraph / gsl_data_path_info | [17.3 宿主图、GPR 与数据完成](17_AGM_GSL_Architecture/17_17.3_GSL图生命周期与DSP命令.md)、[16.11 PAL Session 与 GSL API](16_Vendor_QNX_Architecture/16_16.11_SessionGsl与GSL接口.md) |
| GSL / gsl_fe / gsl_vm_be / gsl_be_session | [17.2 控制与缓冲区](17_AGM_GSL_Architecture/17_17.2_GSL跨VM控制与缓冲区.md)、[16.12 双域全景](16_Vendor_QNX_Architecture/16_16.12_Android+QNX双域架构总结.md)；[16.7 VM INIT 与追加数据库](16_Vendor_QNX_Architecture/16_16.7_ACDB校准数据Android与QNX双域共享.md) |
| audio_service / sysctrl / pil_thread_adsp / fastrpc_be_thread | [16.17 AR 服务并行初始化与资源管理器](16_Vendor_QNX_Architecture/16_16.17_QNX_audio_service_vm_VM音频服务.md)、[16.16 后端边界](16_Vendor_QNX_Architecture/16_16.16_QNX_audio_driver_vm_VM音频驱动层.md) |
| GPR / gpr_local_send / gpr_ipc_receive / GLink | [16.19 域路由、端口回调、连接与报文所有权](16_Vendor_QNX_Architecture/16_16.19_QNX_apr_lib_APR协议库.md) |
| ACDB / graph_init / acdb_init / gsl_add_database | [16.6 图与校准查询](16_Vendor_QNX_Architecture/16_16.6_ACDB校准体系.md)、[16.7 双域数据库](16_Vendor_QNX_Architecture/16_16.7_ACDB校准数据Android与QNX双域共享.md)；不以 legacy acdb-loader 代表 AR 主链 |
| auto-audiod / AutoPower / Silent Boot / audio-chime | 16 |
| audio_service_vm / ams_lib / apr_lib / audio_dac / audio_expander | 16 |

## Android 内核（18）

| 类 / 驱动 | 章节 |
|-----------|------|
| sa8155 / gvm_auto_spf_dummy（machine） | [18.3 ASoC 声卡注册](18_Kernel_Audio/18_18.3_ASoC声卡注册.md) |
| msm-pcm-q6-v2 / msm-compress-q6-v2 / msm-dai-q6-v2 / msm-pcm-routing-auto | [18.4 FE 与 Platform 驱动](18_Kernel_Audio/18_18.4_FE与Platform驱动.md) |
| q6adm / q6afe / q6asm / q6core / rtac / msm_audio_ion_vm | [18.5 QDSP6 客户端与 APR VM](18_Kernel_Audio/18_18.5_QDSP6客户端与APR_VM传输.md) |
| apr_vm / apr_v2（APR over HAB MM_AUD_1/2） | [18.5](18_Kernel_Audio/18_18.5_QDSP6客户端与APR_VM传输.md)；QNX 侧同通道见 [17.2](17_AGM_GSL_Architecture/17_17.2_GSL跨VM控制与缓冲区.md) |
| snd_utils / pcm_plugin / agm_pcm_plugin（tinyalsa 插件分发） | [18.6 用户态消费与插件分发](18_Kernel_Audio/18_18.6_用户态消费与插件分发.md)、[15.12](15_PAL_Architecture/15_15.12_QC_audio-alsa_ALSA层实现.md) |

## 场景与版本（19–20）

| 条目 | 章节 |
|------|------|
| 多音区并发 / 语音录音 / 蓝牙通话 / 异常恢复（场景链路） | [19 篇](19_End_to_End_Scenarios/README.md) |
| 12L ↔ 13+/14+ 版本边界台账与迁移提示 | [20.1 版本边界总表](20_Android_Version_Evolution/20_20.1_版本边界总表与迁移提示.md) |