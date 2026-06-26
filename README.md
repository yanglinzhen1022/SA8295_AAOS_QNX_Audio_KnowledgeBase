# AOSP14 Audio Knowledge Base

Android 14 (AAOS) 音频子系统深度解析知识库，共17篇，覆盖从架构总论到调试指南的完整体系。阅读顺序：01-14→16→17→15(Debug放最后)。

---

## 目录

| 序号 | 文件 | 主题 | 关键类/流程 |
|------|------|------|-------------|
| 01 | [01_Architecture_Overview.md](01_Architecture_Overview.md) | 架构总论(621行) | 7层分层图、5大状态机、Binder IPC地图、AOSP14新特性演进、播放/录音全栈数据流、系统启动时序、交叉引用导航 |
| 02 | [02_Application_Layer.md](02_Application_Layer.md) | 应用层API(2612行) | AudioTrack/Record时序、AAudio MMAP、AAudio服务端oboeservice(AAudioService/EndpointManager/CommandQueue/MMAP+Shared双路径/SharedRingBuffer)、AudioTrack pause()/flush()深度解析+setVolume()/VolumeShaper音量渐变、MediaPlayer/NuPlayer、MediaCodec/Extractor/MetadataRetriever |
| 03 | [03_Java_Framework_Layer.md](03_Java_Framework_Layer.md) | Java框架层(746行) | AudioService、MFC焦点栈、PAM duck/fade/mute、AudioMode状态机 |
| 04 | [04_Native_Framework_Layer.md](04_Native_Framework_Layer.md) | Native框架层(1004行) | AudioSystem路由缓存、AudioTrackClientProxy、Binder IPC 5步时序、AudioMmapStreamInterface、IAudioFlingerService AIDL、IAudioTrackCallback/IAudioRecordCallback、audio_utils(ChannelMix/resampler/fifo) |
| 05 | [05_AudioFlinger.md](05_AudioFlinger.md) | AudioFlinger引擎(1255行) | Thread体系、FastMixer、createTrack时序、AudioMixer引擎、DeviceEffectManager、MelReporter/SoundDoseManager、PatchCommandThread、BufLog、SpatializerThread+BitPerfectThread |
| 06 | [06_Audio_Policy_Engine.md](06_Audio_Policy_Engine.md) | 音频策略引擎(1045行) | getOutputForAttrInt、ProductStrategy、VolumeGroup曲线、EngineConfigurable(PFW)、EngineConfig、EngineInterface、VolumeCurve插值、LastRemovableMediaDevices、AudioPolicyManagerObserver |
| 07 | [07_Effects_Framework.md](07_Effects_Framework.md) | 音效框架(2001行) | EffectChain process_l、Insert/Auxiliary、Spatializer、AEC/NS算法、LoudnessEnhancer、EffectHandle Binder代理、DeviceEffectProxy设备级代理、libeffects内置音效Native实现(DynamicsProcessing/HapticGenerator/LVM Bundle/AIDL迁移) |
| 08 | [08_HAL_Layer.md](08_HAL_Layer.md) | HAL抽象层(1320行) | AIDL IModule 8子模块、StreamDescriptor、AudioGain模型、IModule/ISoundDose/ITelephony AIDL接口、libaudiohal适配层(DeviceHalAidl/StreamHalAidl FMQ/EffectsFactoryHalAidl/EffectProxy/16个AidlConversion) |
| 09 | [09_AAOS_Car_Audio.md](09_AAOS_Car_Audio.md) | 车载音频系统(841行) | CarAudioService、多Zone架构、交互矩阵、CarVolumeGroup、CarAudioZoneConfig、CarZonesAudioFocus、CarAudioDynamicRouting、CarVolume、CarAudioGainMonitor、CarDucking、MirrorRequestHandler、MediaRequestHandler、CoreAudioHelper |
| 10 | [10_AudioControl_HAL.md](10_AudioControl_HAL.md) | 车载音频控制HAL(687行) | evaluateFocusRequest、Ducking/Muting流程、AudioControlWrapperAidl、HalAudioFocus外部焦点、IAudioGainCallback增益回调、IModuleChangeCallback模块变更 |
| 11 | [11_Vendor_Layer.md](11_Vendor_Layer.md) | Vendor定制层(496行) | audio_policy_configuration.xml属性详解、car_audio_configuration深度解析、OEM定制矩阵、AIDL/HIDL HAL架构 |
| 12 | [12_Audio_Focus_Deep_Dive.md](12_Audio_Focus_Deep_Dive.md) | Audio Focus深度解析(735行) | Focus栈模型、duck/fade/mute执行、通话Muting、FadeOutManager、PlaybackActivityMonitor Duck配置、FocusRequester Limbo状态、多用户隔离 |
| 13 | [13_Volume_Device_Deep_Dive.md](13_Volume_Device_Deep_Dive.md) | Volume & Device深度解析(714行) | Volume状态机、SoundDose CSD算法(IEC 62368-1)、MelProcessor/MelAggregator、Device路由迁移、USB/HDMI管理 |
| 14 | [14_Bluetooth_Audio.md](14_Bluetooth_Audio.md) | 蓝牙音频深度解析(747行) | A2DP Codec/Offload、LE Audio BAP/CAP/ASCS/MICP/CSIP/TMAP、LC3编码、SCO/HFP状态机、LE Audio Broadcast |
| 15 | [15_Debug_and_OEM_Guide.md](15_Debug_and_OEM_Guide.md) | 调试方法与OEM定制指南(最后篇) | dumpsys audio、logcat过滤、问题定位、systrace/perfetto追踪、AF详细dump解读、AAOS CarAudio调试、AudioPolicy配置验证、延迟调试深度指南、录音调试、HAL调试、音效调试、OEM深度定制 |
| 16 | [16_PAL_Architecture.md](16_PAL_Architecture.md) | PAL架构深度解析(1462行) | PAL API层、Stream/Device/Session类层次、ResourceManager单例、PayloadBuilder(GKV/CKV/TKV)、IPC机制(HIDL)、XML配置、HAL-PAL适配层、Plugins、ContextManager、SndCardMonitor、核心时序图(播放/路由/SSR/语音/EC) |
| 17 | [17_Vendor_QNX_Architecture.md](17_Vendor_QNX_Architecture.md) | Vendor+QNX双域架构深度解析 | auto-audiod守护进程、AutoPower+VHAL集成、Silent Boot监控、audio-chime早期提示音、ACDB校准体系(loader/mcs/fts/rtac)、QNX侧ACDB校准数据、ALSA UCM配置、auto-casa-xml配置、**AGM(Audio Graph Manager)深度解析(Session/Graph/Device对象模型/APM交互/IPC/AudioReach vs Legacy对比)**、SessionGsl+GSL接口、**Primary HAL(AR版)深度解析(AudioDevice/AudioStream/AudioVoice/延迟参数/VSID/audio_extn扩展)**、**GSL内部架构深度解析(Graph状态机/GKV Node/GPR通信/DataPath Buffer管理)**、Android+QNX双域架构总结、Q&A |

---

## 架构分层总览

```
┌──────────────────────────────────────────────────────────────┐
│                   Layer 7: Application                       │
│   AudioTrack / AudioRecord / AAudio / MediaPlayer / ExoPlayer│
├──────────────────────────────────────────────────────────────┤
│                 Layer 6: Java Framework                       │
│   AudioService / MediaFocusControl / PlaybackActivityMonitor │
│   AudioDeviceBroker / AudioMode(6种模式) / Capture仲裁(5层) │
├──────────────────────────────────────────────────────────────┤
│                Layer 5: Native Framework                      │
│   libaudioclient(Binder IPC) / AudioSystem(路由缓存)          │
├──────────────────────────────────────────────────────────────┤
│                  Layer 4: AudioFlinger                        │
│   Thread体系 / AudioMixer+Resampler / PatchPanel / Track     │
├──────────────────────────────────────────────────────────────┤
│              Layer 3: Audio Policy Engine                     │
│   AudioPolicyManager / ProductStrategy / VolumeGroup / Engine │
├──────────────────────────────────────────────────────────────┤
│              Layer 2: Effects Framework                       │
│   EffectChain / EffectModule / Spatializer                   │
├──────────────────────────────────────────────────────────────┤
│              Layer 1: HAL + Vendor + AAOS                     │
│   Audio HAL(AIDL/HIDL) / AudioControl HAL / Config / BT HAL │
├──────────────────────────────────────────────────────────────┤
│            Layer 0: Vendor Platform (SA8295)                  │
│   PAL / AGM / GSL / APM(SPF) / ACDB / auto-audiod / audio-chime / UCM / QNX  │
└──────────────────────────────────────────────────────────────┘
```

---

## 阅读建议

- **入门路径**: 01 → 02 → 03 → 05 → 08
- **4大状态机**: 03(AudioMode) → 12(Focus) → 13(Volume+Device)
- **策略与路由**: 01 → 06 → 13 → 08
- **车载专项**: 01 → 09 → 10 → 11 → 15(Debug放最后)
- **蓝牙专项**: 02 → 03 → 14
- **全栈调用链**: 05(播放/录音) → 12(Focus链路) → 13(Volume/Device链路)
- **调试定位**: 05 → 15(最后阅读)
- **Vendor平台**: 08 → 16 → 17 (PAL→AGM→GSL→APM→ACDB→auto-audiod→QNX)
- **双域架构**: 16(PAL) → 17(Vendor+QNX) → 09(AAOS车载)
- **AudioReach架构**: 16(PAL) → 17.10(AGM) → 17.13(Primary HAL AR版) → 17.14(GSL内部架构) → 17.11(SessionGsl)

---

> 本知识库基于 Android 14 (AOSP) + 高通SA8295平台源码分析，覆盖AOSP标准架构与Vendor平台特有实现，适用于 AAOS 车载音频系统开发与定制。