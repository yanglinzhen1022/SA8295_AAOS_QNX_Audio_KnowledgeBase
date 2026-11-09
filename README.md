# SA8295 AAOS + QNX AudioReach 音频知识库

面向分层学习与源码阅读：从应用 API、Framework、AudioFlinger 和策略，到 Audio HAL、PAL、AGM/GSL、QNX、Android 内核与设备侧实现，再把各层放回播放、录音、车载路由、版本迁移和故障定位场景。

**主线：本地 Android 12L / API 32 + SA8295 + QNX，产品使用 AudioReach。**

## 目录索引

| # | 章节 | 学习层次 | 主要内容 |
|---|---|---|---|
| 01 | [架构总览](01_Architecture_Overview/) | 全局模型 | 对象、层次、控制流和数据流怎样关联 |
| 02 | [应用层](02_Application_Layer/) | 应用 API | AudioTrack、AudioRecord、AAudio 与媒体播放入口 |
| 03 | [Java Framework](03_Java_Framework_Layer/) | Java 服务 | AudioService、焦点、播放状态管理 |
| 04 | [Native Framework](04_Native_Framework_Layer/) | Native 接口 | AudioSystem、客户端代理、跨进程衔接 |
| 05 | [AudioFlinger](05_AudioFlinger/) | 音频引擎 | Track、线程、混音、播放与录音处理 |
| 06 | [Audio Policy](06_Audio_Policy_Engine/) | 策略 | 输出选择、路由、策略与音量组 |
| 07 | [Effects](07_Effects_Framework/) | 音效 | 效果链、处理位置与生命周期 |
| 08 | [Audio HAL](08_HAL_Layer/) | 标准接口边界 | 本地主线的 HIDL、FMQ、C HAL 与 Patch |
| 09 | [AAOS Car Audio](09_AAOS_Car_Audio/) | 车载服务 | 区域、焦点、音量组与动态路由 |
| 10 | [AudioControl HAL](10_AudioControl_HAL/) | 车载控制接口 | 车载系统与 Android 音频控制的交互 |
| 11 | [Vendor 定制](11_Vendor_Layer/) | 平台配置 | Android 音频配置与厂商扩展边界 |
| 12 | [Audio Focus 专题](12_Audio_Focus_Deep_Dive/) | 跨层控制 | 焦点状态及抢占后的动作 |
| 13 | [Volume / Device 专题](13_Volume_Device_Deep_Dive/) | 跨层控制 | 音量与设备切换 |
| 14 | [Bluetooth Audio](14_Bluetooth_Audio/) | 外部音频通路 | A2DP/HFP-SCO/ASHA 主线；LE Audio 为兼容入口 |
| 15 | [PAL 架构](15_PAL_Architecture/) | 平台抽象 | HAL-PAL、Stream/Session/Device、资源管理 |
| 16 | [Vendor + QNX](16_Vendor_QNX_Architecture/) | 平台与跨域实现 | audiod、ACDB 校准、UCM/平台配置与 QNX 服务/GPR |
| 17 | [AGM / GSL](17_AGM_GSL_Architecture/) | 图管理 | AGM 会话、GSL 跨 VM 控制与图生命周期 |
| 18 | [Android 内核音频](18_Kernel_Audio/) | 内核层 | techpack 构建、ASoC 声卡、FE/Platform、APR-VM 与 tinyalsa 插件分发 |
| 19 | [端到端场景](19_End_to_End_Scenarios/) | 跨层综合 | 多音区并发、语音录音、蓝牙通话与异常恢复 |
| 20 | [版本演进](20_Android_Version_Evolution/) | 版本对照 | 12L 与 13+/14+ 逐项边界台账与迁移提示 |
| 21 | [调试与 OEM 指南](21_Debug_and_OEM_Guide/) | 问题定位 | 用各层状态、日志和工具验证假设 |

## 分层学习顺序

推荐按知识依赖阅读，不必机械地按目录编号顺读。

### 第一阶段：建立音频对象与控制关系

**01 → 02 → 03 → 04 → 05 → 06 → 07**

先理解应用创建什么、Framework 管什么、AudioFlinger 怎样处理数据、Policy 怎样选择路由，以及效果处理放在哪里。策略、焦点和音量是控制关系，不能都画成串联的 PCM 搬运层。

关键篇目：[5.1 AudioFlinger 职责与对象](05_AudioFlinger/05_5.1_AudioFlinger-音频数据面引擎.md) · [4.4 共享内存协议](04_Native_Framework_Layer/04_4.4_共享内存机制深度解析.md) · [5.9 播放全栈调用链](05_AudioFlinger/05_5.9_音乐播放全栈调用链.md) · [5.10 录音全栈调用链](05_AudioFlinger/05_5.10_录音全栈调用链.md) · [6.1/6.2 策略入口](06_Audio_Policy_Engine/06_6.1_AudioPolicyService-控制面入口.md)

### 第二阶段：跨过标准接口进入平台实现

**08 → 15 → 17 → 16 → 18**

先跨过 HAL 标准边界（08 与 15.10），区分 HIDL/C HAL/PAL 边界，再深入 PAL 内部、AGM/GSL 和 QNX，最后到 18 篇补上 Android 内核侧（FE 声卡与 APR-VM）。控制调用、共享缓冲区、PCM API 与跨 VM 消息需要分别追踪，不能用一条粗箭头替代。

关键篇目：[8.1 HAL 双轨边界](08_HAL_Layer/08_8.1_Audio_HAL双轨架构.md) · [15.10 HAL-PAL 适配层](15_PAL_Architecture/15_15.10_HAL-PAL适配层架构.md) · [15.12 tinyalsa→AGM 桥](15_PAL_Architecture/15_15.12_QC_audio-alsa_ALSA层实现.md) · [17.1 AGM 会话与音频图](17_AGM_GSL_Architecture/17_17.1_AGM会话与音频图.md) · [17.2 GSL 跨 VM 控制](17_AGM_GSL_Architecture/17_17.2_GSL跨VM控制与缓冲区.md) · [16.12 双域全景](16_Vendor_QNX_Architecture/16_16.12_Android+QNX双域架构总结.md) · [18.1 内核通路总览](18_Kernel_Audio/18_18.1_内核音频通路总览.md)

专题：[16.6 ACDB 校准体系](16_Vendor_QNX_Architecture/16_16.6_ACDB校准体系.md) · [17.3 GSL 图生命周期](17_AGM_GSL_Architecture/17_17.3_GSL图生命周期与DSP命令.md) · [16.17 QNX audio_service](16_Vendor_QNX_Architecture/16_16.17_QNX_audio_service_vm_VM音频服务.md) · 随读随查 [15.99 Stream/Session/Device 映射](15_PAL_Architecture/15_15.99_附录_Stream-Session-Device映射关系.md)

### 第三阶段：理解 AAOS 与跨层主题

**09 → 10 → 11，再按问题读 12 / 13 / 14 → 19**

把 CarAudioService、多区域、AudioControl、平台配置与焦点/音量/设备联系起来，再用 19 篇场景专题把各层按真实场景串起来。核心 Audio HAL 与 AudioControl HAL 是不同接口，不要仅凭其中一者的 HIDL/AIDL 版本推断另一者。升级评估时读 20 篇的版本边界台账。

关键篇目：[9.1 AAOS 对象模型](09_AAOS_Car_Audio/09_9.1_AAOS音频系统总览.md) · [9.11 动态路由构建](09_AAOS_Car_Audio/09_9.11_CarAudioDynamicRouting-动态路由构建.md) · [9.4 车载焦点](09_AAOS_Car_Audio/09_9.4_CarAudioFocus-车载焦点管理.md) · [12.6 焦点交互矩阵](12_Audio_Focus_Deep_Dive/12_12.6_AAOS焦点交互矩阵.md) · [10.1 AudioControl 总览](10_AudioControl_HAL/10_10.1_AudioControl_HAL总览.md) · [19.1 场景阅读指南](19_End_to_End_Scenarios/19_19.1_场景阅读指南与全链路全景.md)

### 第四阶段：按现象回到各层

**21，并回查相应层的实现**

调试章节放在原理之后学习。先区分 Java AudioService、AudioFlinger、AudioPolicy 和 HAL/PAL 的观察范围，再判断问题属于哪层；不把某条日志或某个成功返回码当作完整链路成功。

关键篇目：[21.1 dumpsys audio 全量状态](21_Debug_and_OEM_Guide/21_21.1_dumpsys_audio-音频系统全量状态.md) · [21.3 常见问题定位](21_Debug_and_OEM_Guide/21_21.3_常见问题定位.md)

## 版权与使用范围

本库为个人学习笔记，内容基于产品源码的静态分析，包含少量带出处的代码摘录；仅供个人学习参考，未获任何厂商授权，请勿用于商业用途或对外分发。
