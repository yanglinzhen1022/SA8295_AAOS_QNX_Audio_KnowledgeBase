# SA8295 AAOS + QNX AudioReach 音频知识库

面向分层学习与源码阅读：从应用 API、Framework、AudioFlinger 和策略，到 Audio HAL、PAL、AGM/GSL、QNX 与设备侧实现，再把各层放回播放、录音、车载路由和故障定位场景。

**当前整理主线：本地 Android 12L / API 32 + SA8295 + QNX，产品使用 AudioReach。** 本地主线不是 Android 14；新版本机制需与对应版本源码对照，不与主线混写。

> **阅读状态：** 本库正在逐篇源码核对与重构。目前优先阅读下列 Native PCM / Policy、AAOS 路由/焦点、HAL → PAL → AGM 与双域校准主线文章；其余原章节保留供查阅，尚未完成本轮核对，可能含旧版本混写、错误映射或未验证流程。目录存在不代表内容已经核验，静态源码分析也不代表实机行为已经验证。

[术语表](GLOSSARY.md) ｜ [类名反向索引](CLASS_INDEX.md) ｜ [主题阅读入口](#主题去重导航矩阵single-source-of-truth)

## 从这里开始：Native PCM → Policy → HAL / PAL

| 顺序 | 文章 | 要回答的问题 |
|---|---|---|
| 1 | [5.1 AudioFlinger 职责与对象](05_AudioFlinger/05_5.1_AudioFlinger-音频数据面引擎.md) | Track、线程、I/O 与 HAL 对象为什么不是一一对应？ |
| 2 | [4.4 共享内存协议](04_Native_Framework_Layer/04_4.4_共享内存机制深度解析.md) | 谁推进 front/rear？为什么共享不等于零拷贝？ |
| 3 | [5.9 普通 PCM 播放](05_AudioFlinger/05_5.9_音乐播放全栈调用链.md) | 创建、FAST 协商、start、write 与实际消费怎样衔接？ |
| 4 | [5.10 普通 PCM 录音](05_AudioFlinger/05_5.10_录音全栈调用链.md) | 输入重选、两阶段启动、分发和静音为何不能照抄播放？ |
| 5 | [6.1 APS](06_Audio_Policy_Engine/06_6.1_AudioPolicyService-控制面入口.md) → [6.2 APM](06_Audio_Policy_Engine/06_6.2_AudioPolicyManager-策略核心实现.md) | 权限、动态策略、设备/I/O 选择和活动状态分别由谁管理？ |

### 加入车载控制：AAOS → AudioControl

| 顺序 | 文章 | 要回答的问题 |
|---|---|---|
| 1 | [9.1 AAOS 对象模型](09_AAOS_Car_Audio/09_9.1_AAOS音频系统总览.md) → [9.11 动态 AudioMix](09_AAOS_Car_Audio/09_9.11_CarAudioDynamicRouting-动态路由构建.md) | Zone、Context、usage 与 BUS 怎样关联，规则由谁注册？ |
| 2 | [9.2 CarAudioService](09_AAOS_Car_Audio/09_9.2_CarAudioService-车载音频核心服务.md) → [9.5 音量组](09_AAOS_Car_Audio/09_9.5_CarVolumeGroup-车载音量组.md) | 用户/UID affinity、索引、设备 gain 与 mute 分别怎样变化？ |
| 3 | [10.1 AudioControl v1](10_AudioControl_HAL/10_10.1_AudioControl_HAL总览.md) → [10.6 AIDL wrapper](10_AudioControl_HAL/10_10.6_AudioControlWrapperAidl-AIDL适配层架构.md) | 焦点、duck/mute 和死亡恢复的责任边界在哪里？ |

进一步深入：[9.3 Zone](09_AAOS_Car_Audio/09_9.3_CarAudioZone-多Zone音频管理.md) → [9.18 配置解析](09_AAOS_Car_Audio/09_9.18_CarAudioZonesHelper-XML配置解析.md)；[9.10 焦点分区](09_AAOS_Car_Audio/09_9.10_CarZonesAudioFocus-多Zone焦点分发器.md) → [9.4 焦点状态](09_AAOS_Car_Audio/09_9.4_CarAudioFocus-车载焦点管理.md) → [12.6 交互矩阵](12_Audio_Focus_Deep_Dive/12_12.6_AAOS焦点交互矩阵.md) → [10.7 HAL 外部请求](10_AudioControl_HAL/10_10.7_HalAudioFocus-外部焦点请求管理.md)。

### 继续向下：HAL → PAL → AGM

| 顺序 | 文章 | 要回答的问题 |
|---|---|---|
| 1 | [8.1 Audio HAL 边界：从 HIDL 接口到 AudioReach 实现](08_HAL_Layer/08_8.1_Audio_HAL双轨架构.md) | HIDL、FMQ、C HAL 与厂商实现怎样衔接？ |
| 2 | [15.10 HAL-PAL 适配层：从一次 PCM write 理解对象与生命周期](15_PAL_Architecture/15_15.10_HAL-PAL适配层架构.md) | 什么时候 open/start？standby 为什么会 close？正返回为何不证明已发声？ |
| 3 | [15.2 PAL 到 AGM 的层次](15_PAL_Architecture/15_15.2_架构层次图.md) → [15.12 tinyalsa 插件](15_PAL_Architecture/15_15.12_QC_audio-alsa_ALSA层实现.md) | pcm/mixer 怎样进入 AGM？用户态 ops 与内核 ioctl 有什么区别？ |
| 4 | [17.1 AGM 会话与音频图](17_AGM_GSL_Architecture/17_17.1_AGM会话与音频图.md) | metadata/AIF 为什么先于 open？图、设备和会话怎样启动与清理？ |
| 5 | [16.6 AudioReach 校准](16_Vendor_QNX_Architecture/16_16.6_ACDB校准体系.md) → [16.7 双域数据库](16_Vendor_QNX_Architecture/16_16.7_ACDB校准数据Android与QNX双域共享.md) | 图查询、初始库、VM 握手与追加数据库有什么不同？ |
| 6 | [17.2 GSL 跨 VM 控制与缓冲区](17_AGM_GSL_Architecture/17_17.2_GSL跨VM控制与缓冲区.md) → [16.12 双域全景](16_Vendor_QNX_Architecture/16_16.12_Android+QNX双域架构总结.md) | 命令和完成为何走不同通道？HOST/GUEST、export ID 与 offset 怎样配合？ |
| 7 | [17.3 GSL 图生命周期与 DSP 命令](17_AGM_GSL_Architecture/17_17.3_GSL图生命周期与DSP命令.md) | 共享 SG 如何启停？本地状态、GPR 应答、数据完成为什么不能混同？ |
| 8 | [16.17 QNX 服务](16_Vendor_QNX_Architecture/16_16.17_QNX_audio_service_vm_VM音频服务.md) → [16.19 GPR/GLink](16_Vendor_QNX_Architecture/16_16.19_QNX_apr_lib_APR协议库.md) | 服务线程怎样初始化？连接、投递、回收与 DSP 应答如何区分？ |
| 随读随查 | [15.99 Stream、Session、Device：工厂映射与适用边界](15_PAL_Architecture/15_15.99_附录_Stream-Session-Device映射关系.md) | 三类对象如何分别选择？Non-Tunnel 和检测引擎有哪些边界？ |

PAL 资源与配置专题：[15.6 ResourceManager](15_PAL_Architecture/15_15.6_ResourceManager-PAL核心管理模块.md) → [15.7 PayloadBuilder](15_PAL_Architecture/15_15.7_PayloadBuilder-载荷构建器.md) → [15.9 配置地图](15_PAL_Architecture/15_15.9_配置文件列表.md)，解释 FE/设备配置、selector/KV、模块参数和安装选择之间的关系。

以上主线文章配有源码片段、调用关系、时序或状态图。范围以各篇声明为准，不用一个普通 PCM 示例代替所有流类型。

## 分层学习顺序

推荐按知识依赖阅读，不必机械地按目录编号顺读；目录编号已于 2026-11-06 重排（原 17 调试章移至末尾，见上文注）。

### 第一阶段：建立音频对象与控制关系

**01 → 02 → 03 → 04 → 05 → 06 → 07**

先理解应用创建什么、Framework 管什么、AudioFlinger 怎样处理数据、Policy 怎样选择路由，以及效果处理放在哪里。策略、焦点和音量是控制关系，不能都画成串联的 PCM 搬运层。

### 第二阶段：跨过标准接口进入平台实现

**08 → 15 → 17 → 16 → 18**

先读上列 HAL-PAL 文章，区分 HIDL/C HAL/PAL 边界，再深入 PAL 内部、AGM/GSL 和 QNX，最后到 18 篇补上 Android 内核侧（FE 声卡与 APR-VM）。控制调用、共享缓冲区、PCM API 与跨 VM 消息需要分别追踪，不能用一条粗箭头替代。

### 第三阶段：理解 AAOS 与跨层主题

**09 → 10 → 11，再按问题读 12 / 13 / 14 → 19**

把 CarAudioService、多区域、AudioControl、平台配置与焦点/音量/设备联系起来，再用 19 篇场景专题把各层按真实场景串起来。核心 Audio HAL 与 AudioControl HAL 是不同接口，不要仅凭其中一者的 HIDL/AIDL 版本推断另一者。升级评估时读 20 篇的版本边界台账。

### 第四阶段：按现象回到各层

**21，并回查相应层的实现**

调试章节放在原理之后学习。先区分 Java AudioService、AudioFlinger、AudioPolicy 和 HAL/PAL 的观察范围，再判断问题属于哪层；不把某条日志或某个成功返回码当作完整链路成功。

以上路线中，除明确列出的已重写文章外，原文仍在整理。新版本主题可作为查阅线索，不作为本地 Android 12L 的实现依据。

## 章节目录

| 章节 | 学习层次 | 主要问题 |
|---|---|---|
| [01 架构总览](01_Architecture_Overview/) | 全局模型 | 对象、层次、控制流和数据流怎样关联 |
| [02 应用层](02_Application_Layer/) | 应用 API | AudioTrack、AudioRecord、AAudio 与媒体播放入口 |
| [03 Java Framework](03_Java_Framework_Layer/) | Java 服务 | AudioService、焦点、播放状态管理 |
| [04 Native Framework](04_Native_Framework_Layer/) | Native 客户端/接口 | AudioSystem、客户端代理、跨进程衔接 |
| [05 AudioFlinger](05_AudioFlinger/) | 音频引擎 | Track、线程、混音、播放与录音处理 |
| [06 Audio Policy](06_Audio_Policy_Engine/) | 策略 | 输出选择、路由、策略与音量组 |
| [07 Effects](07_Effects_Framework/) | 音效 | 效果链、处理位置与生命周期 |
| [08 Audio HAL](08_HAL_Layer/) | 标准接口边界 | 本地主线的 HIDL、FMQ、C HAL 与 Patch |
| [09 AAOS Car Audio](09_AAOS_Car_Audio/) | 车载服务 | 区域、焦点、音量组与动态路由 |
| [10 AudioControl HAL](10_AudioControl_HAL/) | 车载控制接口 | 车载系统与 Android 音频控制的交互 |
| [11 Vendor 定制](11_Vendor_Layer/) | 平台配置 | Android 音频配置与厂商扩展边界 |
| [12 Audio Focus 专题](12_Audio_Focus_Deep_Dive/) | 跨层控制 | 焦点状态及抢占后的动作 |
| [13 Volume / Device 专题](13_Volume_Device_Deep_Dive/) | 跨层控制 | 音量与设备切换 |
| [14 Bluetooth Audio](14_Bluetooth_Audio/) | 外部音频通路 | 蓝牙音频角色、路由和数据通路；注意版本差异 |
| [15 PAL 架构](15_PAL_Architecture/) | 平台抽象 | HAL-PAL、Stream/Session/Device、资源管理 |
| [16 Vendor + QNX](16_Vendor_QNX_Architecture/) | 平台与跨域实现 | 16.6/16.7 校准、16.11/16.12 平台接口、16.16–16.19 后端/服务/GPR 与兼容入口已调整；其余原文待核对 |
| [17 AGM / GSL](17_AGM_GSL_Architecture/) | 图管理 | 17.1 AGM、17.2 GSL VM、17.3 宿主图/GPR/数据路径；前端、RTC/动态模块及底层传输仍有边界 |
| [18 Android 内核音频](18_Kernel_Audio/) | 内核层 | techpack/audio 构建、ASoC 声卡、FE/Platform、QDSP6 客户端与 APR-VM（HAB）、tinyalsa 插件分发；Elite/AR 启动双分支与 AR 主线全插件路由的静态证明链 |
| [19 端到端场景](19_End_to_End_Scenarios/) | 跨层综合 | 多音区并发、语音录音、蓝牙通话、异常恢复四场景，逐步回链各层 |
| [20 版本演进](20_Android_Version_Evolution/) | 版本对照 | 12L 本树与 13+/14+ 机制的逐项边界台账与迁移阅读提示 |
| [21 调试与 OEM 指南](21_Debug_and_OEM_Guide/) | 问题定位 | 用各层状态、日志和工具验证假设 |

> 注：2026-11-06 章节重编号——原 17 调试与 OEM 指南移至末尾为第 21 章，原 18/19/20/21 章依次前移为 17/18/19/20 章；全部目录、文件名与交叉引用已同步更新（映射与核对记录见 [docs/superpowers/reports/2026-11-05-final-completion-report.md](docs/superpowers/reports/2026-11-05-final-completion-report.md) §8）。

## 本地主线的几个必要边界

- Android 版本依据本地版本定义中的 API 32；产品源码中 `msmnile_gvmq/manifest.xml` 声明核心 audio/effects HIDL 6.0、AudioControl AIDL v1。**声明不等于最终运行服务已核验。**
- AudioReach 是本次学习与重构范围，不展开其他厂商音频架构分支。
- PAL 的 Stream、Session、Device 是不同抽象；`NON_TUNNEL` 在本地 Session 工厂中选择 `SessionAgm`，不能把全平台写成统一的 `SessionAlsaPcm`。
- 源码根目录：Android 为 `Android/lagvm/LINUX/android/`；QNX 音频为 `Qnx/apps/qnx_ap/AMSS/multimedia/audio/`。文章内给出相对根目录的文件/函数位置，不把它们写成失效的知识库内部链接。
- 本库不提供从静态代码即可成立的整机延迟、物理接线或运行效果保证；涉及这些问题，应补充相应配置与实测条件。

## 主题去重导航矩阵（Single Source of Truth）

下表约定深入讲解的组织位置，而非为未整理原文作正确性背书。先建立主题模型，再回到各层看它怎样参与；跨层专题不重复复制所有层内实现。

| 主题 | 优先入口 | 关联层 |
|---|---|---|
| 客户端共享内存 | [4.4](04_Native_Framework_Layer/04_4.4_共享内存机制深度解析.md) | AudioTrack、AudioRecord、Proxy |
| AF 视角的普通播放/录音 | [5.9](05_AudioFlinger/05_5.9_音乐播放全栈调用链.md) / [5.10](05_AudioFlinger/05_5.10_录音全栈调用链.md) | Track、I/O、线程与 HAL；不代替跨域端到端专题 |
| 策略入口与选择 | [6.1](06_Audio_Policy_Engine/06_6.1_AudioPolicyService-控制面入口.md) / [6.2](06_Audio_Policy_Engine/06_6.2_AudioPolicyManager-策略核心实现.md) | APS、APM、Engine、动态 mix |
| HAL 边界与 FMQ | [8.1](08_HAL_Layer/08_8.1_Audio_HAL双轨架构.md) | AudioFlinger、libaudiohal、厂商 HAL |
| HAL-PAL 适配与普通 PCM 生命周期 | [15.10](15_PAL_Architecture/15_15.10_HAL-PAL适配层架构.md) | 08、PAL 内部 |
| Stream / Session / Device 映射 | [15.99](15_PAL_Architecture/15_15.99_附录_Stream-Session-Device映射关系.md) | HAL flags、设备与 PAL 工厂 |
| AAOS 区域与动态路由 | [9.1](09_AAOS_Car_Audio/09_9.1_AAOS音频系统总览.md) / [9.11](09_AAOS_Car_Audio/09_9.11_CarAudioDynamicRouting-动态路由构建.md) | 9.2 服务协调、Native Policy、BUS 地址 |
| 车载控制与 duck/mute | [10.1](10_AudioControl_HAL/10_10.1_AudioControl_HAL总览.md) / [10.6](10_AudioControl_HAL/10_10.6_AudioControlWrapperAidl-AIDL适配层架构.md) | [9.4 单区域焦点](09_AAOS_Car_Audio/09_9.4_CarAudioFocus-车载焦点管理.md)、[12.6 矩阵](12_Audio_Focus_Deep_Dive/12_12.6_AAOS焦点交互矩阵.md)；12 其余原文仍待核对 |
| 车载音量组 | [9.5](09_AAOS_Car_Audio/09_9.5_CarVolumeGroup-车载音量组.md) | 9.2、AudioPort gain、AudioControl muting；[13 跨层专题](13_Volume_Device_Deep_Dive/) 原文仍待核对 |
| 效果处理 | [07](07_Effects_Framework/) | 02、05、HAL；原文待核对 |
| PAL 到 AGM 的 PCM/控制桥 | [15.12](15_PAL_Architecture/15_15.12_QC_audio-alsa_ALSA层实现.md) | tinyalsa、PCM/mixer plugin、计量与状态 |
| AGM 会话与图 | [17.1](17_AGM_GSL_Architecture/17_17.1_AGM会话与音频图.md) | metadata、AIF、device、GSL API |
| AudioReach 校准与双域数据库 | [16.6](16_Vendor_QNX_Architecture/16_16.6_ACDB校准体系.md) / [16.7](16_Vendor_QNX_Architecture/16_16.7_ACDB校准数据Android与QNX双域共享.md) | GKV/CKV/TKV、文件列表、QNX 追加库、路径与句柄 |
| GSL VM 控制与数据缓冲 | [17.2](17_AGM_GSL_Architecture/17_17.2_GSL跨VM控制与缓冲区.md) | HAB 命令/事件、HOST/GUEST、export/import、完成与清理 |
| GSL 图与 DSP 命令 | [17.3](17_AGM_GSL_Architecture/17_17.3_GSL图生命周期与DSP命令.md) | handle/GKV/共享 SG、状态与应答、data mode 与 buffer 所有权 |
| QNX 服务与底层报文 | [16.17](16_Vendor_QNX_Architecture/16_16.17_QNX_audio_service_vm_VM音频服务.md) / [16.19](16_Vendor_QNX_Architecture/16_16.19_QNX_apr_lib_APR协议库.md) | 并行初始化、resource manager、GPR/GLink、packet 所有权 |
| Android + QNX 平台全景 | [16.12](16_Vendor_QNX_Architecture/16_16.12_Android+QNX双域架构总结.md) | 执行域、构建/启动边界与不能由静态代码推导的结论 |
| 内核音频通路（Guest VM） | [18.1](18_Kernel_Audio/18_18.1_内核音频通路总览.md) | 18.2–18.6：模块清单、声卡注册、FE/Platform、APR-VM HAB、tinyalsa 插件分发 |
| 端到端场景 | [19.1](19_End_to_End_Scenarios/19_19.1_场景阅读指南与全链路全景.md) | 19.2–19.5：并发播放/录音仲裁/蓝牙通话/异常恢复；不引入新机制，逐步回链各层 |
| 新版本机制与版本边界 | [20.1](20_Android_Version_Evolution/20_20.1_版本边界总表与迁移提示.md) / [01.1.7](01_Architecture_Overview/01_1.7_AOSP14_Audio新特性架构演进.md) | 逐机制台账（12L 证据 ↔ 13+/14+ 归属与出处）；特性讲解在 01.1.7 |

后续继续深化 AGM/GSL。Kernel Audio（18 篇）、端到端场景（19 篇）和 Android 版本演进（20 篇）已建成；仍坚持不为未核验主题生成占位章节，不把设计/实施记录放进学习主线。
