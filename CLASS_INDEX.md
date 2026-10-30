# 源码类名 → 讲解章节 反向索引（CLASS_INDEX）

> 对照源码阅读时，从核心类/接口名快速定位讲解所在章。按层分组，`章节` 列指主要讲解章。[← 返回总目录](README.md) ｜ 目录级速查见 [01.5 关键源码目录速查](01_Architecture_Overview/01_1.5_关键源码目录速查.md)

> **范围提示：** 本地主线为 Android 12L + SA8295 AudioReach。下方本批新增的 8.1、15.10、15.99 直达链接对应已重写文章；其他条目沿用旧索引，可能属于新版本或尚待源码核对，不代表本地均有实现。

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
| AudioTrack.cpp / AudioRecord.cpp | 04 |
| AudioSystem | 04 |
| AudioTrackClientProxy / AudioRecordClientProxy | 04 |
| IAudioFlingerService (AIDL) | 04 / 05 |
| ChannelMix / resampler / fifo (audio_utils) | 04 / 13(MEL) |

## AudioFlinger（05）

| 类 / 接口 | 章节 |
|-----------|------|
| AudioFlinger / ThreadBase / MixerThread / RecordThread | 05 |
| FastMixer | 05 |
| AudioMixer / AudioResampler | 05 |
| Track / RecordTrack | 05 |
| PatchPanel / PatchCommandThread | 05 |
| MelReporter / SoundDoseManager | 05 / 13 |
| SpatializerThread / BitPerfectThread | 05 |
| DeviceEffectManager | 05 / 07 |

## Audio Policy / Engine（06）

| 类 / 接口 | 章节 |
|-----------|------|
| AudioPolicyService / AudioPolicyManager | 06 |
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
| IAudioControl (AIDL) / IFocusListener | 10 |
| IAudioGainCallback / IModuleChangeCallback | 10 |
| AudioControlWrapperAidl / HalAudioFocus | 10 |

## AAOS 车载（09）/ 焦点·音量·设备深入（12–13）

| 类 / 接口 | 章节 |
|-----------|------|
| CarAudioService / CarAudioZone / CarAudioZoneConfig | 09 |
| CarAudioFocus / CarZonesAudioFocus | 09 / 12 |
| CarVolumeGroup / CarVolume | 09 / 13 |
| CarAudioDynamicRouting / CarAudioGainMonitor / CarDucking | 09 |
| FocusRequester / FadeOutManager | 12 |
| VolumeController / MelProcessor / MelAggregator | 13 |

## 蓝牙（14）

| 类 / 接口 | 章节 |
|-----------|------|
| A2dpService / LeAudioService / LeAudioCodecConfig | 14 |
| BtHelper / HearingAidService | 14 |

## Vendor / PAL / QNX（15–16）

| 类/ 接口 | 章节 |
|-----------|------|
| AudioDevice / StreamPrimary / StreamOutPrimary / StreamInPrimary (AR HAL) | [15.10 HAL-PAL 适配](15_PAL_Architecture/15_15.10_HAL-PAL适配层架构.md) |
| pal_stream_open / start / write / stop / close | [15.10 PCM 生命周期与错误返回](15_PAL_Architecture/15_15.10_HAL-PAL适配层架构.md) |
| Stream::create / StreamPCM / StreamCompress / StreamInCall / StreamNonTunnel | [15.99 Stream 工厂](15_PAL_Architecture/15_15.99_附录_Stream-Session-Device映射关系.md) |
| Session::makeSession / SessionAlsaPcm / SessionAlsaCompress / SessionAlsaVoice / SessionAgm | [15.99 Session 四分支](15_PAL_Architecture/15_15.99_附录_Stream-Session-Device映射关系.md) |
| StreamSoundTrigger / SoundTriggerEngineGsl / StreamACD / ContextDetectionEngine | [15.99 检测引擎与 Session 的持有关系](15_PAL_Architecture/15_15.99_附录_Stream-Session-Device映射关系.md) |
| Device::getInstance / Speaker (PAL) | [15.99 Device 分派与复用](15_PAL_Architecture/15_15.99_附录_Stream-Session-Device映射关系.md) |
| ResourceManager / PayloadBuilder (GKV/CKV/TKV) | 15 |
| ContextManager / SndCardMonitor | 15 |
| AGM / GSL / gsl_fe / gsl_vm_be | 16 |
| ACDB / acdb-loader | 16 |
| auto-audiod / AutoPower / Silent Boot / audio-chime | 16 |
| audio_service_vm / ams_lib / apr_lib / audio_dac / audio_expander | 16 |