# SA8295 AAOS + QNX 双域音频架构知识库

Android 14 (AAOS/GVM) + QNX (PVM) 双域架构、高通SA8295平台音频子系统深度解析，共17篇，覆盖从应用层API到QNX底层驱动、跨VM通信的完整体系。

> **基线**：AOSP Android 14 + 高通 SA8295 BSP。**规模**：17 章 / 237 文件 / ~11.5 万行。**质量**：Mermaid 图 1186（渲染 0 失败）、内部导航 ~1843 链接（0 断链）。**末次校验**：2026-07。
> **快速索引**：[术语缩略语表 GLOSSARY](GLOSSARY.md) ｜ [源码类名 → 讲解章节 反向索引](CLASS_INDEX.md) ｜ [主题去重导航矩阵](#主题去重导航矩阵single-source-of-truth)

---

## 目录

| 序号 | 文件 | 主题 | 关键类/流程 |
|------|------|------|-------------|
| 01 | [01_Architecture_Overview/](01_Architecture_Overview/) | 架构总论 | 7层分层图、5大状态机、Binder IPC地图、AOSP14新特性演进、播放/录音全栈数据流、系统启动时序、交叉引用导航 |
| 02 | [02_Application_Layer/](02_Application_Layer/) | 应用层API(14节) | AudioTrack/Record时序、AAudio MMAP、AAudio服务端oboeservice、AudioTrack pause()/flush()深度解析+setVolume()/VolumeShaper音量渐变、MediaPlayer/NuPlayer、MediaCodec/Extractor/MetadataRetriever |
| 03 | [03_Java_Framework_Layer/](03_Java_Framework_Layer/) | Java框架层 | AudioService、MFC焦点栈、PAM duck/fade/mute、AudioMode状态机 |
| 04 | [04_Native_Framework_Layer/](04_Native_Framework_Layer/) | Native框架层 | AudioSystem路由缓存、AudioTrackClientProxy、Binder IPC 5步时序、AudioMmapStreamInterface、IAudioFlingerService AIDL、IAudioTrackCallback/IAudioRecordCallback、audio_utils(ChannelMix/resampler/fifo) |
| 05 | [05_AudioFlinger/](05_AudioFlinger/) | AudioFlinger引擎 | Thread体系、FastMixer、createTrack时序、AudioMixer引擎、DeviceEffectManager、MelReporter/SoundDoseManager、PatchCommandThread、BufLog、SpatializerThread+BitPerfectThread |
| 06 | [06_Audio_Policy_Engine/](06_Audio_Policy_Engine/) | 音频策略引擎 | getOutputForAttrInt、ProductStrategy、VolumeGroup曲线、EngineConfigurable(PFW)、EngineConfig、EngineInterface、VolumeCurve插值、LastRemovableMediaDevices、AudioPolicyManagerObserver |
| 07 | [07_Effects_Framework/](07_Effects_Framework/) | 音效框架(12节) | EffectChain process_l、Insert/Auxiliary、Spatializer、AEC/NS算法、LoudnessEnhancer、EffectHandle Binder代理、DeviceEffectProxy设备级代理、libeffects内置音效Native实现 |
| 08 | [08_HAL_Layer/](08_HAL_Layer/) | HAL抽象层 | AIDL IModule 8子模块、StreamDescriptor、AudioGain模型、IModule/ISoundDose/ITelephony AIDL接口、libaudiohal适配层 |
| 09 | [09_AAOS_Car_Audio/](09_AAOS_Car_Audio/) | 车载音频系统 | CarAudioService、多Zone架构、交互矩阵、CarVolumeGroup、CarAudioZoneConfig、CarZonesAudioFocus、CarAudioDynamicRouting、CarVolume、CarAudioGainMonitor、CarDucking、MirrorRequestHandler、MediaRequestHandler、CoreAudioHelper |
| 10 | [10_AudioControl_HAL/](10_AudioControl_HAL/) | 车载音频控制HAL | evaluateFocusRequest、Ducking/Muting流程、AudioControlWrapperAidl、HalAudioFocus外部焦点、IAudioGainCallback增益回调、IModuleChangeCallback模块变更 |
| 11 | [11_Vendor_Layer/](11_Vendor_Layer/) | Vendor定制层 | audio_policy_configuration.xml属性详解、car_audio_configuration深度解析、OEM定制矩阵、AIDL/HIDL HAL架构 |
| 12 | [12_Audio_Focus_Deep_Dive/](12_Audio_Focus_Deep_Dive/) | Audio Focus深度解析 | Focus栈模型、duck/fade/mute执行、通话Muting、FadeOutManager、PlaybackActivityMonitor Duck配置、FocusRequester Limbo状态、多用户隔离 |
| 13 | [13_Volume_Device_Deep_Dive/](13_Volume_Device_Deep_Dive/) | Volume & Device深度解析 | Volume状态机、SoundDose CSD算法(IEC 62368-1)、MelProcessor/MelAggregator、Device路由迁移、USB/HDMI管理 |
| 14 | [14_Bluetooth_Audio/](14_Bluetooth_Audio/) | 蓝牙音频深度解析 | A2DP Codec/Offload、LE Audio BAP/CAP/ASCS/MICP/CSIP/TMAP、LC3编码、SCO/HFP状态机、LE Audio Broadcast |
| 15 | [15_PAL_Architecture/](15_PAL_Architecture/) | PAL架构深度解析 | PAL API层、Stream/Device/Session类层次、ResourceManager单例、PayloadBuilder(GKV/CKV/TKV)、IPC机制(HIDL)、XML配置、HAL-PAL适配层、Plugins、ContextManager、SndCardMonitor、核心时序图(播放/路由/SSR/语音/EC)、QC辅助模块(audio-alsa/GEF/audio-parsers/CAPIv2/CASA/SoundTrigger) |
| 16 | [16_Vendor_QNX_Architecture/](16_Vendor_QNX_Architecture/) | Vendor+QNX双域架构深度解析 | auto-audiod守护进程、AutoPower+VHAL集成、Silent Boot监控、audio-chime早期提示音、ACDB校准体系、QNX侧ACDB校准数据、ALSA UCM配置、auto-casa-xml配置、AGM深度解析、GSL接口与PAL Session图管理(注:SessionGsl为历史遗留,实走SessionAlsaPcm)、Primary HAL(AR版)深度解析、GSL内部架构深度解析、Android+QNX双域架构总结、Q&A、QNX VM隔离层(audio_driver_vm/audio_service_vm/ams_lib/apr_lib)、QNX辅助库(audio_a2b/audio_dac/audio_expander) |
| 17 | [17_Debug_and_OEM_Guide/](17_Debug_and_OEM_Guide/) | 调试方法与OEM定制指南(15节) | dumpsys audio、logcat过滤、问题定位、systrace/perfetto追踪、AF详细dump解读、AAOS CarAudio调试、AudioPolicy配置验证、延迟调试深度指南、录音调试、HAL调试、音效调试、OEM深度定制 |

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
- **车载专项**: 01 → 09 → 10 → 11 → 17(Debug放最后)
- **蓝牙专项**: 02 → 03 → 14
- **全栈调用链**: 05(播放/录音) → 12(Focus链路) → 13(Volume/Device链路)
- **调试定位**: 05 → 17(最后阅读)
- **Vendor平台**: 08 → 15 → 16 (PAL→AGM→GSL→APM→ACDB→auto-audiod→QNX)
- **双域架构**: 15(PAL) → 16(Vendor+QNX) → 09(AAOS车载)
- **AudioReach架构**: 15(PAL) → 16.10(AGM) → 16.13(Primary HAL AR版) → 16.14(GSL内部架构) → 16.11(GSL接口与PAL Session图管理；`SessionGsl`为历史遗留,实走`SessionAlsaPcm`)

---

## 主题去重导航矩阵（Single Source of Truth）

多个主题会跨章从不同视角出现（如"焦点"在应用/Java/车载/HAL 各层都有涉及）。为避免误读为"内容重复"，下表约定每个主题的**权威章节**（深入原理以此为准）与**其它视角章节**（仅从本层职责切入，细节回链权威章）。阅读时**先读权威章建立完整模型，再按需查看视角章**。

| 主题 | 权威章节（深入原理） | 其它视角章节（分层引用，细节回链权威章） |
|------|---------------------|------------------------------------------|
| 音频焦点 Focus | [12 Audio Focus 深度解析](12_Audio_Focus_Deep_Dive/) | 03(Java MFC 焦点栈)、09(车载多 Zone 焦点)、10(AudioControl HAL 外部焦点)、01(状态机总览) |
| duck/fade/mute 抢占动作 | [12 Audio Focus 深度解析](12_Audio_Focus_Deep_Dive/) | 09(车载 Ducking)、10(HAL Ducking/Muting)、03(PAM Duck 配置) |
| 音量 Volume | [13 Volume 状态机](13_Volume_Device_Deep_Dive/) + [06 VolumeGroup 曲线](06_Audio_Policy_Engine/) | 09(车载音量组)、05(AF 音量应用)、01(状态机总览) |
| 设备路由 Device | [13 Device 状态机/迁移](13_Volume_Device_Deep_Dive/) | 08(Audio Patch/Port 硬件路由)、06(动态路由策略)、09(车载动态路由) |
| 音效 Effect | [07 音效框架](07_Effects_Framework/) | 05(AF EffectChain 挂载/process_l)、02(应用侧 AudioEffect API)、08(HAL 侧) |
| 声剂量 SoundDose/CSD/MEL | [13 SoundDose 与 CSD](13_Volume_Device_Deep_Dive/) | 05(MelReporter/SoundDoseManager)、08(ISoundDose HAL)、04(audio_utils MEL 工具) |
| PAL Session/GSL 图管理 | [15 PAL 架构](15_PAL_Architecture/) | 16(AGM/GSL 内部与 QNX 侧实现)。注：`SessionGsl` 为历史遗留（`.cpp` 已移除），SA8295 实际走 `SessionAlsaPcm` 等 4 分支，详见 15.5/15.99 更正说明 |
| 跨 VM 通信 MM-HAB | [16 Vendor+QNX 双域架构](16_Vendor_QNX_Architecture/) | 15(PAL 经 AGM→gsl_fe→HAB→gsl_vm_be 调 DSP)、01(双域总览) |

> **去重原则**：本库不做物理内容合并（各篇保持独立可读），而以本矩阵约定权威源。若发现某"视角章节"重复展开了本应属于"权威章节"的深层原理，应精简为要点 + 回链，而非两处各维护一份。

---

> 本知识库基于 Android 14 (AOSP) + 高通SA8295平台源码分析，覆盖AOSP标准架构与Vendor平台特有实现，适用于 AAOS 车载音频系统开发与定制。