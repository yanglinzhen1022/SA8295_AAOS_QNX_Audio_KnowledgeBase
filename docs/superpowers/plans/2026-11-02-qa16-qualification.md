# 16.15 Q&A 资格化 + 16.1 残留修正批

executing-plans，父会话唯一 writer；前批提交 1493578。复用 /tmp/qc-audiod-doc-check.py（articles 增 16.15，roots 增 gsl_be）。

## 审查重点
1. 16.15 全页改为三级证据标注（源码/构建部署/运行时），纠正：udevadm（Android guest 无）、PCM 设备号（MERC 48/49、A2B 50/53）、MultiMedia23=设备55、chime TAG `early-audio-chime`、bike_bell.wav 路径、UCM 与 mixer_paths 混写、QNX 链路混入 Elite 组件（QNX AM/ams/apr → 16.18/16.19/16.20 兼容口径）、APR 地址域说法改 GPR、删 persist.audio.ssr_recovery / persist.audio.silent_boot / persist.audio.gsl.debug / ACDB ID 对照表 / 0x0001-0x0007 错误码表（均为无出处内容，标为负结论）、PAL TAG 修正为 `PAL: SessionAlsaPcm`/`PAL: PayloadBuilder`、multiple_mix_dsp 挂上 resourcemanager_gvmauto8295_adp_star.xml 证据。
2. 16.1 残留：QNX 域 mermaid 去 QNX AM/ARM/resourcemanager 混写；安全音频链改 audio_service+CSD2/GSL 骨架并把"专用Graph/优先级/独立TDM"降级为设计层描述；启动树去 QNX Audio Manager 就绪项；组件表 ams/apr 行标注 Elite 遗留。

## Tasks
- [x] Task1 16.15 重写（1287→281 行，18 问保留教学价值）
- [x] Task2 16.1 六处修正
- [x] Task3 回归+提交

## 结果记录
- 终态 article 11 文件/52 摘录 0 失败；navigation 15 文件/376 链接/13 图 0 失败。
- 检查器增 gsl_be root 后 GSL_VM opcode 摘录可验证；行号后缀（:67 等）从反引号路径中移除以免破坏存在性检查。
