# 术语与缩略语表（GLOSSARY）

> SA8295 AAOS + QNX 双域音频知识库高频术语速查。"主要章节"列指该术语深入讲解所在章，便于按需跳转。[← 返回总目录](README.md)

---

## 一、平台与虚拟化

| 缩写/术语 | 全称 | 说明 | 主要章节 |
|-----------|------|------|----------|
| SA8295 | Snapdragon Automotive 8295 | 高通车载 SoC 平台 | 16 |
| AAOS | Android Automotive OS | 车载 Android 系统 | 09 |
| GVM | Guest Virtual Machine | 运行 Android 的虚拟机（Guest 域） | 01/16 |
| PVM | Primary Virtual Machine | 运行 QNX 的主虚拟机（Primary 域，DSP 唯一控制方） | 16 |
| QNX | QNX Neutrino RTOS | 实时操作系统，掌管 ADSP | 16 |
| ADSP | Audio DSP | 音频数字信号处理器 | 16 |
| MM-HAB | Multimedia Hypervisor Advanced Bus | GVM↔PVM 跨 VM 音频通信通道 | 16 |
| Hypervisor | — | 虚拟化管理层，隔离 GVM/PVM | 01/16 |

## 二、Android 音频框架

| 缩写/术语 | 全称 | 说明 | 主要章节 |
|-----------|------|------|----------|
| AF | AudioFlinger | Android 音频混音/输出核心服务 | 05 |
| APM | AudioPolicyManager | 音频策略管理（路由/音量决策） | 06 |
| MFC | MediaFocusControl | 媒体焦点控制（焦点栈仲裁） | 03/12 |
| PAM | PlaybackActivityMonitor | 播放活动监控（duck/fade/mute） | 03/12 |
| HAL | Hardware Abstraction Layer | 硬件抽象层（AIDL/HIDL 双轨） | 08 |
| AIDL | Android Interface Definition Language | 新一代 HAL 接口语言 | 08 |
| HIDL | HAL Interface Definition Language | 旧 HAL 接口语言（PAL IPC 仍用） | 08/15 |
| MMAP | Memory-Mapped | AAudio 低延迟内存映射路径 | 02 |
| CSD | Computed Sound Dose | 计算声剂量（IEC 62368-1） | 13 |
| MEL | Momentary Exposure Level | 瞬时暴露级（声剂量计算量） | 13 |

## 三、车载音频（AAOS）

| 缩写/术语 | 全称 | 说明 | 主要章节 |
|-----------|------|------|----------|
| Zone | Audio Zone | 车载音频分区（主驾/副驾/后排） | 09 |
| Ducking | — | 焦点抢占时压低背景音 | 09/10/12 |
| Fade | — | 音量渐入渐出 | 12 |
| VHAL | Vehicle HAL | 车辆硬件抽象层 | 16 |

## 四、高通 Vendor / PAL / AudioReach

| 缩写/术语 | 全称 | 说明 | 主要章节 |
|-----------|------|------|----------|
| PAL | Platform Audio Layer | 高通平台音频层（Stream/Device/Session） | 15 |
| AGM | Audio Graph Manager | 音频图管理器 | 16 |
| GSL | Graphite Service Layer | 图服务层（AudioReach 图运行时） | 16 |
| SPF | Signal Processing Framework | 信号处理框架（DSP 侧） | 16 |
| APM(SPF) | Audio Processing Manager | SPF 侧音频处理管理（区别于 Android APM） | 16 |
| ACDB | Audio Calibration DataBase | 音频校准数据库 | 16 |
| GKV | Graph Key Vector | 图键向量（选择音频图） | 15 |
| CKV | Calibration Key Vector | 校准键向量（检索校准数据） | 15 |
| TKV | Tag Key Vector | 标签键向量（模块级参数） | 15 |
| SSR | SubSystem Restart | 子系统（DSP）重启恢复 | 15 |
| UCM | Use Case Manager | ALSA 用例配置 | 16 |
| Session | PAL Session | PAL 会话对象（实走 SessionAlsaPcm 等 4 分支） | 15 |
| SessionGsl | — | 历史遗留（`.cpp` 已移除），实际不走此分支 | 15/16 |
| CAPIv2 | Common Audio Processing Interface v2 | DSP 侧编解码/处理接口 | 15 |
| CASA | — | 高通校准配置工具 | 15 |
| APR | Asynchronous Packet Router | QNX 侧异步包路由协议库 | 16 |

## 五、蓝牙音频

| 缩写/术语 | 全称 | 说明 | 主要章节 |
|-----------|------|------|----------|
| A2DP | Advanced Audio Distribution Profile | 蓝牙高质量音频传输 | 14 |
| LE Audio | Bluetooth Low Energy Audio | 低功耗蓝牙音频 | 14 |
| LC3 | Low Complexity Communication Codec | LE Audio 编解码器 | 14 |
| BAP/CAP | Basic/Common Audio Profile | LE Audio 基础/通用配置 | 14 |
| SCO/HFP | Synchronous Connection-Oriented / Hands-Free Profile | 通话音频/免提 | 14 |