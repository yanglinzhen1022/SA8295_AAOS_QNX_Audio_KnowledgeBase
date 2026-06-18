# AOSP14 Audio Knowledge Base

Android 14 (AAOS) 音频子系统深度解析知识库，共15篇，覆盖从架构总论到调试与OEM定制的完整体系。

---

## 目录

| 序号 | 文件 | 主题 | 核心内容 |
|------|------|------|----------|
| 01 | [01_Architecture_Overview.md](01_Architecture_Overview.md) | 架构总论 | 7层架构分层图、5大核心状态机(Focus/Volume/Device/AudioMode/Capture仲裁)、Binder IPC关系、模块地图、源码目录速查、设计哲学、版本演进 |
| 02 | [02_Application_Layer.md](02_Application_Layer.md) | 应用层API | AudioTrack创建/播放完整时序(含Offload)、AudioRecord采集流程、AudioManager、AudioFocusRequest、AAudio低延迟(MMAP)、MediaPlayer/SoundPool/MediaRecorder/ExoPlayer/OpenSL ES |
| 03 | [03_Java_Framework_Layer.md](03_Java_Framework_Layer.md) | Java框架层 | AudioService启动序列、MediaFocusControl焦点栈、PlaybackActivityMonitor(duck/fade/mute矩阵)、VolumeController双路径、AudioDeviceBroker设备管理、AudioMode状态机+CommunicationDevice、录音并发仲裁(Concurrent Capture)、RecordingActivityMonitor、通话Muting、MediaSessionService/SoundTriggerService |
| 04 | [04_Native_Framework_Layer.md](04_Native_Framework_Layer.md) | Native框架层 | libaudioclient Binder IPC(IAudioTrack/IAudioRecord/IAudioFlinger)、AudioSystem路由状态缓存、JNI桥接、共享内存布局、死亡通知 |
| 05 | [05_AudioFlinger.md](05_AudioFlinger.md) | AudioFlinger引擎 | Thread继承体系(7类Thread+FastMixer)、MixerThread双路径、Buffer管理、openOutput/createTrack完整时序、PatchPanel路由、Thread与Track匹配规则、播放全栈调用链、录音全栈调用链、AudioMixer+Resampler混音引擎 |
| 06 | [06_Audio_Policy_Engine.md](06_Audio_Policy_Engine.md) | 音频策略引擎 | getOutputForAttrInt()5步详解、ProductStrategy 6策略映射、VolumeGroup音量曲线、Engine路由决策、Device状态机+路由迁移、Focus Policy、AudioPolicyMix动态策略(投屏/录制) |
| 07 | [07_Effects_Framework.md](07_Effects_Framework.md) | 音效框架 | EffectChain process_l()完整流程、Insert vs Auxiliary对比、EffectModule状态机+process()详解、FLOAT_EFFECT_CHAIN、Spatializer |
| 08 | [08_HAL_Layer.md](08_HAL_Layer.md) | HAL抽象层 | AIDL IModule完整8子模块接口分类、openOutputStream/connectExternalDevice详解时序、StreamDescriptor结构、HIDL legacy、Audio Patch、AudioGain模型(JOINT/CHANNELS/RAMP) |
| 09 | [09_AAOS_Car_Audio.md](09_AAOS_Car_Audio.md) | 车载音频系统 | CarAudioService初始化时序、多Zone架构、CarAudioFocus交互矩阵、CarVolumeGroup三级控制、CarAudioContext映射、Audio Mirroring、AAOS多Zone全栈调用链 |
| 10 | [10_AudioControl_HAL.md](10_AudioControl_HAL.md) | 车载音频控制HAL | evaluateFocusRequestLocked完整流程、外部焦点请求、AAOS Ducking/Muting完整流程、Ducking vs Muting对比 |
| 11 | [11_Vendor_Layer.md](11_Vendor_Layer.md) | Vendor定制层 | 配置文件体系3层架构、audio_policy_configuration.xml完整解析、OEM定制点、car_audio_configuration.xml |
| 12 | [12_Audio_Focus_Deep_Dive.md](12_Audio_Focus_Deep_Dive.md) | Audio Focus深度解析 | Focus栈模型+requestAudioFocus完整流程+Loss类型映射、框架级焦点执行(duck/fade/mute)、AAOS交互矩阵、Focus全栈调用链、abandonAudioFocus、通话Muting |
| 13 | [13_Volume_Device_Deep_Dive.md](13_Volume_Device_Deep_Dive.md) | Volume & Device深度解析 | Volume状态机+SoundDose安全机制、音量调节完整流程+Stream别名映射、Volume全栈调用链+双路径应用、Device状态机+路由迁移+Fallback、USB/HDMI/有线设备管理、Device Routing全栈链路、联合交互场景 |
| 14 | [14_Bluetooth_Audio.md](14_Bluetooth_Audio.md) | 蓝牙音频深度解析 | A2DP连接/音量/Codec/Suspend/Offload、LE Audio架构/连接/音量/VCP/Suspend/Broadcast、SCO/HFP状态机/IBluetooth配置、Hearing Aid ASHA、AudioDeviceBroker蓝牙交互 |
| 15 | [15_Debug_and_OEM_Guide.md](15_Debug_and_OEM_Guide.md) | 调试方法与OEM定制指南 | dumpsys audio完整解析、logcat音频过滤、常见问题定位(播放/焦点/路由/延迟/蓝牙)、OEM定制指南(HAL/AudioControl/路由/音量/蓝牙)、性能优化 |

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
└──────────────────────────────────────────────────────────────┘
```

---

## 阅读建议

- **入门路径**: 01 → 02 → 03 → 05 → 08
- **4大状态机**: 03(AudioMode) → 12(Focus) → 13(Volume+Device)
- **策略与路由**: 01 → 06 → 13 → 08
- **车载专项**: 01 → 09 → 10 → 11 → 15
- **蓝牙专项**: 02 → 03 → 14
- **全栈调用链**: 05(播放/录音) → 12(Focus链路) → 13(Volume/Device链路)
- **调试定位**: 05 → 15

---

> 本知识库基于 Android 14 (AOSP) 源码分析，适用于 AAOS 车载音频系统开发与定制。