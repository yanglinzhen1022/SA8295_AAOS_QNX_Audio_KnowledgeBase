# UCM / audio_dac / audio_expander / a2b 批

executing-plans，父会话唯一 writer；前批提交 3878c70。检查器继续复用 /tmp/qc-audiod-doc-check.py（扩 articles/roots/terms），基线 /tmp/qc-ucm-dac-before.log。

## 审查重点
1. 16.8：UCM 主文件+独立用例文件结构、Syntax 4、LibraryConfig.agm（agm:1,0 PCM）、cset-tlv acdb-blob；部署证据是 CMake（HYP/LXC、quin-gvm-gen4→gvmauto8295 ucm-conf/acdb-blob/pp-control），Android 主线 mk 未见部署；消费者边界（LXC/LE Linux 音频栈）如实。
2. 16.21 audio_dac：audio_common 下 PCM1681 I2C 库（五 API、两 ioctl、默认增益 224、单例、无 SSR）；AR 消费者 csd2_dev_oem_lib 与 csd2_ipc/audio_chime 链接 libaudio_dacS 的证据。
3. 16.22 audio_expander：同为 audio_common、AR 同样链接；结构对称（aud_* 前缀）；差异点按源核实。
4. 16.20 a2b：源码仅存在于 audio_elite/audio_driver/audio_a2b → 本页转为资格限定入口（Elite 范畴、主线不研究；细节留在 Git 历史），检查主题混写。

## Tasks
- [x] Task1 16.8 重写（结构+部署边界+agm 插件衔接）
- [x] Task2 16.21/16.22 校验性修订（摘录可验证、补 AR 消费者、保留原有正确结构）
- [x] Task3 16.20 转兼容入口
- [x] Task4 导航与交叉链接（16 README、16.1、CLASS_INDEX）

## 验证
- 新基线；全回归；diff --check；staged gate；本地提交。

## 结果记录

- 基线 /tmp/qc-ucm-dac-before.log：14 失败。终态 article 与 navigation（14 文件/334 链接/48 摘录/12 图）均 0 失败。
- 检查器扩展：fence 语言增 conf；alsa-ucm 根收录 .conf 与 ucm-conf 无后缀文件；roots 增 audio_dac/audio_expander/csd2_dev_oem_lib。一次替换脚本未生效（roots 追加）由失败输出暴露后补齐——断言式替换必要。
- 关键事实：UCM 部署证据为 CMake（LXC/HYP、quin-gvm-gen4→gvmauto8295 ucm-conf+acdb-blob+pp-control），Android 主线 mk 无部署项；hifi EnableSequence cset/cset-tlv 指向 AGM 插件与 /etc/acdb-blob。audio_dac：默认增益 224 仅写 DAC1~4 且返回值丢弃；仅 MUTE/FMT 两命令；寄存器表 pcm1681_reg_map。audio_expander：三命令（DAC_ENABLE/SWITCH_SOURCE/SWITCH_ENABLE）、四引脚位、TDM_MUX 低有效；芯片型号源码未标注。AR 消费者：csd2_dev_oem_lib（i2c4，0x77 expander→0x4C DAC→dlopen libMercuryApp.so）与 csd2_ipc/audio_chime（输出切换 SOC_AUDIO/MERCURY、mute 自检）。audio_a2b：仅 Elite 树且纯 prebuilt（a2b-app+liba2bstack*/plugin）→ 16.20 转兼容入口；主线 A2B 证据在 PAL A2BSpeaker/audiod hostless 表/UCM 用例。
- 16.1 修正：UCM 行改 Linux/LXC guest+CMake；a2b 行标注 Elite-only 预编译；dac/expander 行改用户态库+AR 链接；UCM 路径行去 /usr/share/alsa/ucm 断言。
- cdc_dev_oem_plugin.c 注释 "sleep 60ms" 与实际 usleep(100*1000) 不一致——如实记录，不改源码。
