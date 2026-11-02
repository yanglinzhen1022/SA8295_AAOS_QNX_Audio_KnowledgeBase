# mm-audio-auto 收尾批：audio-chime / auto-casa-xml / Primary HAL(AR)

executing-plans，父会话唯一 writer；前批提交 8c52ea5。主线 Android12L + SA8295 + AR；只改知识库。检查器 /tmp/qc-audiod-doc-check.py 扩展复用（更名 qca mm-audio-auto 检查），基线先存 /tmp/qc-mmaudio-before.log。

## 审查重点
1. 16.5：仅 main 一条链；mixer 硬编码 card 0 与 -D 参数不同维度；SND_CARD_STATUS 回调与 16.2 发送方同键；启动证据只有 msmnile_au rc（on fs / bike_bell.wav / oneshot），主线 gvmq rc 无 chime 项；prebuilt 快照未见 libsndmonitor/libacdbloaderclient；plugin 仅调 audio_hal_plugin_init；错误路径标签链如实（ACDB dlopen 失败→free_mydata）。
2. 16.9：旧文结构与真实 xml 标签不符（config_params/bt_codecs 等不存在），按真实文件重写：resourcemanager_* 系列（PAL 配置，指向 15.6/15.9）、usecaseKvManager GKV、hw_ep_info 端点、kvh2xml 键定义；全部装 /vendor/etc（audio_ar_vendor_proprietary_product.mk + auto-casa-xml/Android.mk 板型门 sdmshrike/msmnile/MSMSTEPPE）；PAL 如何选择文件需源码证据。
3. 16.13：核心适配已在 15.10；本页收编"组装+audio_extn+Voice"：hal-pal Android.mk 编译条件、audio_extn 模块表（AutoExtn 分发+各 guard）、AudioVoice 高层（会话/VSID/setParameters 链），不重复 15.10 流细节。

## Tasks
- [x] Task1 16.5 重写（源码 audio_chime_app.c 全函数级核对 + init/sepolicy/prebuilt 证据）
- [x] Task2 16.9 重写（11 文件清单、标签结构、部署、与 15.6/15.9 分工）
- [x] Task3 16.13 重写（组装表、audio_extn 表、AudioVoice 机制层）
- [x] Task4 导航/交叉链接（16 README、16.1、CLASS_INDEX、根 README、15 章）

## 验证
- 新基线；全 13 检查器回归；diff --check；staged gate；本地提交。

## 结果记录

- 基线 /tmp/qc-mmaudio-before.log：23 失败。终态 article 0 失败；navigation（10 文件/286 链接/38 摘录/11 图）0 失败。
- 检查器纠错实例：getFileNameExtn 两处 strlcat 均为 XML_PATH_EXTN_MAX_SIZE（误写 MAX_LENGTH）；AudioVoice.h 摘录省略号、构造循环省略号；均按源修正。检查器源后缀过滤补 .xml/.mk。
- 关键新事实：chime 启动证据仅在 msmnile_au rc（on fs、bike_bell.wav、oneshot），主线 gvmq rc 无服务项；prebuilt 快照有 libaudiohalplugin.so、无 libsndmonitor/libacdbloaderclient；set_mixer_controls 硬编码 card 0 且成功路径不关 mixer；ACDB init 硬编码 sa6155 卡名；错误标签链非全量回滚。auto-casa-xml：PAL 以卡名扩展名选择 resourcemanager_*.xml、PayloadBuilder 固定路径读 usecaseKvManager.xml；hw_ep_info.xml/kvh2xml.xml 本地无按名读者（负结论入文）。Primary HAL(AR)：SRC 六文件、audio_extn 其余文件不在列表；gvmq/au 打包 audio.primary.msmnile.ar、DUAL 分支 .casa symlink、QSSI 分支 audioadsprpcd symlink；AudioVoice VSID×2 会话、两迁移状态机、call_screen/TTY 设备覆盖、开局后补发缓存效果。
- 16.1 修正：auto-casa-xml 所属域 QNX→Android(LA VM)；gsl_fe 行链接改 16.14/18.2；新增 Primary HAL (AR) 行。
