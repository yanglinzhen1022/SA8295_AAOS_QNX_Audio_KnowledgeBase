# 第四篇：Native Framework Layer

> [← 上一篇](../03_Java_Framework_Layer/README.md) | [返回导航](../README.md) | [下一篇 →](../05_AudioFlinger/README.md)

---

## 推荐阅读

本章主线为本地 Android 12L/API 32。先读已按源码重写的 [4.4 共享内存协议](04_4.4_共享内存机制深度解析.md)，掌握 cblk、client/server proxy、帧数与 futex；再接 [5.9 播放](../05_AudioFlinger/05_5.9_音乐播放全栈调用链.md) 和 [5.10 录音](../05_AudioFlinger/05_5.10_录音全栈调用链.md)。

Framework 的 AIDL 与核心 HAL 的 HIDL 是不同接口边界。

## 校准状态

| 文章 | 状态 |
|------|------|
| 4.1 JNI Bridge | 已按 12L 源码校准（回调=callback_t 自由函数，JniStorage/字段映射/摘录逐字化） |
| 4.2 libaudioclient | 已按 12L 源码校准（callback_t、set() 签名、proxy flush、调度组恢复） |
| 4.3 Binder IPC | 已按 12L 源码校准（IAudioTrack/IAudioRecord 为 AIDL、方法表重写、负结论标注） |
| 4.4 共享内存 | 已按 12L 源码重写（评审通过，无发现） |
| 4.5 AIDL 重构 | 已按 12L 源码校准（框架层 AIDL 为本树现行机制，非 AOSP14 现象） |
| 4.6 audio_utils | 已按 12L 源码校准（ChannelMix 运行时矩阵、fifo/cblk 独立、工具类真实 API） |

13+/14+ 机制（IAudioTrackCallback 事件形态、SoundDose、getMmapPolicyInfos 等）在文中均以负结论/版本边界标注。

## 文章目录

- [4.1 JNI Bridge — Java到Native的桥梁](04_4.1_JNI_Bridge-Java到Native的桥梁.md)
- [4.2 libaudioclient — Native音频客户端库](04_4.2_libaudioclient-Native音频客户端库.md)
- [4.3 Binder IPC机制](04_4.3_Binder_IPC机制.md)
- [4.4 共享内存：AudioTrack / AudioRecord 的生产与消费协议](04_4.4_共享内存机制深度解析.md)
- [4.5 AIDL IPC接口重构 — 从Binder C++到AIDL](04_4.5_AIDL_IPC接口重构-从Binder_C++到AIDL.md)
- [4.6 audio_utils核心工具类](04_4.6_audio_utils核心工具类.md)
