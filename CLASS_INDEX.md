# 源码类名 → 讲解章节 反向索引（CLASS_INDEX）

> 对照源码阅读时，从核心类/接口名快速定位讲解所在章。按层分组，`章节` 列指主要讲解章。[← 返回总目录](README.md) ｜ 目录级速查见 [01.5 关键源码目录速查](01_Architecture_Overview/01_1.5_关键源码目录速查.md)

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
| IModule / IStreamOut / IStreamIn (AIDL) | 08 |
| StreamDescriptor / AudioGain / AudioPort / AudioPatch | 08 |
| ISoundDose / ITelephony (AIDL) | 08 |
| libaudiohal 适配层 | 08 |
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
| PAL API / Stream / Device | 15 |
| Session::makeSession (4 分支) / SessionAlsaPcm / SessionAlsaCompress / SessionAlsaVoice / SessionAgm | 15 |
| SessionGsl（历史遗留，`.cpp` 已移除，实走 SessionAlsaPcm） | 15 / 16 |
| ResourceManager / PayloadBuilder (GKV/CKV/TKV) | 15 |
| ContextManager / SndCardMonitor | 15 |
| AGM / GSL / gsl_fe / gsl_vm_be | 16 |
| ACDB / acdb-loader | 16 |
| auto-audiod / AutoPower / Silent Boot / audio-chime | 16 |
| audio_service_vm / ams_lib / apr_lib / audio_dac / audio_expander | 16 |