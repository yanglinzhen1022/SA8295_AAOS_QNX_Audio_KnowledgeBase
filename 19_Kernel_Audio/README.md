# 第十九篇：Android 内核音频（SA8295 Guest VM）

> [返回总导航](../README.md) · [前置：15.12 tinyalsa 与 AGM 插件](../15_PAL_Architecture/15_15.12_QC_audio-alsa_ALSA层实现.md) · [关联：16.12 双域架构](../16_Vendor_QNX_Architecture/16_16.12_Android+QNX双域架构总结.md)、[18.2 GSL 跨 VM](../18_AGM_GSL_Architecture/18_18.2_GSL跨VM控制与缓冲区.md)

本篇讲解 `kernel/msm-5.4/techpack/audio/` 在本产品（`msmnile_gvmq`，SA8295 Guest VM）中的角色：**内核保留两套声卡栈，由启动属性选择其一；AudioReach 主线下内核只提供枚举骨架，音频硬件与图执行在 QNX 宿主域**。四条主线结论（全部有源码锚点）：

1. 内核音频模块的构建入口是 `CONFIG_QTI_QUIN_GVM=y`（autogvm_GKI.config:1）→ `gvmauto.conf` 的 14 个 `=m` 开关；`msmnile_gvmq.mk:188-199` 列出 10 个 `audio_*.ko` 打包名，**实际加载由 `init.target.rc:36-45` 双分支控制**（Elite 默认分支 modprobe 19 个 `*_dlkm` 模块全集；AudioReach 分支仅 `stub_dlkm + spf_machine_dlkm`，[19.2](19_19.2_构建配置与模块清单.md)/[19.6](19_19.6_用户态消费与插件分发.md)）。
2. Guest VM 内核的 APR 传输是 **HAB `MM_AUD_1/MM_AUD_2`**（`apr_vm.c:1366-1379`），不是 rpmsg/glink（[19.5](19_19.5_QDSP6客户端与APR_VM传输.md)；注意该内核栈属 Elite 分支，AR 分支不加载）。
3. `pcm_open()` 走内核 FE 还是 AGM 用户态插件，由**启动分支 + 卡定义**共同决定：AR 分支指向 `card-defs.xml`（card 100，全部设备 PLUGIN 标记，在树内），PAL 打开虚拟卡 100 → **AudioReach 主线 PCM 全部走用户态 AGM 插件，不经内核 FE**（静态可证，[19.6](19_19.6_用户态消费与插件分发.md)）。
4. DT 在树内双节点（`direwolf-vm-audio.dtsi` 同时携带 sa8295 与 gvm-auto-spf 两个 sound 节点，5 个 direwolf-vm dtb 可证），实际注册哪张卡由启动分支决定（[19.3](19_19.3_ASoC声卡注册.md)）。

## 阅读顺序

| 顺序 | 文章 | 问题 |
|---|---|---|
| 1 | [19.1 内核音频通路总览](19_19.1_内核音频通路总览.md) | 内核栈在本产品的定位、两张通路图、证据边界 |
| 2 | [19.2 构建配置与模块清单](19_19.2_构建配置与模块清单.md) | CONFIG→Kbuild→打包名怎么对应？哪些是负结论？ |
| 3 | [19.3 ASoC 声卡注册](19_19.3_ASoC声卡注册.md) | machine 驱动、FE dailink、dummy 变体、DT 双节点与分支绑定 |
| 4 | [19.4 FE 与 Platform 驱动](19_19.4_FE与Platform驱动.md) | `audio_platform.ko`/`audio_native.ko` 里有什么 |
| 5 | [19.5 QDSP6 客户端与 APR VM 传输](19_19.5_QDSP6客户端与APR_VM传输.md) | q6adm/afe/asm 怎么出域？HAB 证据 |
| 6 | [19.6 用户态消费与插件分发](19_19.6_用户态消费与插件分发.md) | 启动双分支、卡定义双份、AR 主线路由的静态证明链 |

## 本篇边界（先读）

- 静态源码阅读，未做内核构建、模块加载或实机验证；
- DT（双 sound 节点）与卡定义 XML（card-defs.xml 全 PLUGIN / 空 card-defs-dummy.xml）均在树内；**不可静态证明的是实机启动属性 `ro.boot.audio` 的取值**，本库主线按 AudioReach 讲解（[19.6 §4](19_19.6_用户态消费与插件分发.md)）；
- HAB 驱动本体、QNX 侧 GPR/GLink、ACDB 数据库加载分别见 [18.2](../18_AGM_GSL_Architecture/18_18.2_GSL跨VM控制与缓冲区.md)、[16.19](../16_Vendor_QNX_Architecture/16_16.19_QNX_apr_lib_APR协议库.md)、[16.6](../16_Vendor_QNX_Architecture/16_16.6_ACDB校准体系.md)，本篇不重复展开。
