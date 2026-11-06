# SA8295 AAOS QNX Audio KnowledgeBase — 最终完成报告

日期：2026-11-06（初版 11-05，gap 闭环更新）｜ 分支：`docs/audioreach-hal-pal` ｜ HEAD：本提交（gap 闭环收口）｜ 工作区：干净
主线口径：**Android 12L / API 32 + SA8295 + QNX AudioReach（PAL/AGM/GSL）**

## 1. 交付物

- 259 个 Markdown 文件（18 章 + 11_Vendor_Layer + README/CLASS_INDEX/GLOSSARY + docs/），约 13MB。
- 分层教材结构（非清单式）：01 架构总览 → 02 应用层 → 03 Java框架 → 04 Native框架 → 05 AudioFlinger → 06 策略引擎 → 07 音效框架 → 08 HAL → 09 AAOS车载 → 10 AudioControl HAL → 11 厂商层 → 12 焦点深潜 → 13 音量设备深潜 → 14 蓝牙 → 15 PAL → 16 QNX厂商架构 → 17 调试指南 → 18 AGM/GSL。
- 版本边界纪律：13+/14+ 机制一律标注或转兼容条目；12L 不存在的机制以"否定性发现"记录（零匹配 grep 证据）；Elite/兼容链（apr_lib/ams_lib/audio_a2b/QNX Audio Manager）仅作兼容入口。
- 摘录纪律：fence 头 `// File:NNN` + 逐字摘录/简化示意限定；链接文本携带真实标识符（fw-checker lineref 规则）。

## 2. 转换历史（67 commits，其中转换期 50）

| 阶段 | 提交范围 | 内容 |
|------|----------|------|
| A. 重建（源驱动重写） | `3da205f`…`253e04d`（14） | 以本地 AudioReach/产品源为准重写 HAL-PAL 学习样本、native PCM、AAOS 路由/焦点、PAL 资源/载荷、GSL 图/GPR、audiod/AutoPower/Silent Boot、chime、UCM/DAC/expander、ACDB 校准、Q&A 证据分级 |
| B. 12L 校准 | `88f0f83`…`9c5233b`（7）+ 评审门禁计划 `5ca45cc` | ch02/03/07/11/13/14/17 校准至本树；蓝牙/LE Audio/13.5-13.7 等转兼容入口 |
| C. 独立评审迭代 | `3f6eb22`…`fc98ff8`（29） | 见 §3 R1/R2/ch05 |
| D. 覆盖缺口闭环 | `6e23da9` 之后 `96c11fd`、`fa7488a` + 本提交（3） | 见 §3 gap/delta 轮 |

## 3. 独立评审覆盖与结果（gap 闭环后：全库文件均至少一轮独立评审）

评审方式：只读 reviewer 子代理逐文件对照产品源码（`/home/ylz/code/qc/Android/lagvm/LINUX/android` + `Qnx/apps/qnx_ap/AMSS/multimedia/audio`），产出 P0/P1/P2 + Verdict；作者对每条发现先源码复核再修复（评审者错误一律以源码 grep 为准回退）。

**覆盖勘误（2026-11-06）**：初版本节标题宣称"全库 100% 章节覆盖"——该声明不准确。R1/R2/ch05 轮为**章节级**覆盖，覆盖审计发现 **22 个文件从未被独立评审**（ch11：11.6–11.11+README；ch15：15.2/15.6/15.7/15.9/15.10/15.12–15.15/15.99+README；ch18：18.1–18.3+README）。下表 gap/delta 轮已将该缺口完全闭环。

| 轮次 | 覆盖 | 结果→修复提交 |
|------|------|---------------|
| R1 基线 | ch08；ch09/10 | `3f6eb22`、`332ecb1` |
| R2/R2b 逐章 | ch01、02、03(×2)、04、06、07、11、12、13、14、15、16+GLOSSARY、17 | `93631e0`、`6c327bc`、`ac9f53e`+`51c2e86`、`338fc3d`、`1197013`、`a3eafa6`、`0870bb2`、`3fd9500`、`77939c2`、`ced3d67`+`107b476`、`1cd3ea0`、`c57ee08`、`35a7716`+`4a4624c` |
| ch05 五轮循环 | 18 文件全部 | `a63d325`（基线）→ rev05c→`ba98c2a` → rev05d1/d2→`a259296` → rev05e1/e2→`a61a862` → **rev05f OK-with-notes**→`fc98ff8`（收口） |
| **gap 轮（22 文件缺口）** | gap11a-retry（11.6/11.7/11.8）、gap11b（11.9/11.10/11.11/README）、gap15c（15.2/15.6/15.7）、gap15d（15.9/15.10/15.12/15.99/README）、gap15e（15.13/15.14/15.15）、gap18f（ch18 全部） | 11.6-11.10 与 15.15 **BLOCK**（虚构机制/符号，如 zoneConfig 组级 context、useHalAudioRouting、HIDL"产品 7.0"、maxOpenSessionsCount、PAL_AUDIO_FMT_SBC/LDAC/APTX、MODULE_ID_APTX_ENC、SessionBT、INTERACTION_MATRIX 等）；15.13/15.14 P1；gap15d/gap18f 零发现 → **`96c11fd`**（全部 P0/P1/P2 修复，均先经源码 grep 复验） |
| **delta 轮（96c11fd 复核）** | delta11a（11.6/11.7）、delta11b（11.8）、delta11c（11.9/11.10/11.11/11.1/README）、delta15（15.13–15.15） | delta11b **BLOCK**（P0：common_au deep_buffer"逐字"摘录实为未生效 msmnile_au 变体）+ delta11c **BLOCK**（P1×4：emulator fence 截断、manifest 实为 6.0、"均 1:1:1"、QUAT_TLAIN 虚构标记）+ delta11a/delta15 OK-with-notes（P1：行号漂移簇、DTSHD_PARSER_ENABLED 负结论失实等） → **`fa7488a`**（P0×1+P1×10+P2×30+ 修复） |
| **delta2 轮（fa7488a 定点复核）** | delta2a（ch11 八文件 50 项清单）、delta2b（ch15 三文件 9 项清单） | **双 OK**：全部内容性声明（API/常量/结构/负结论/逐字摘录/表内容）经源码逐条验证为真，0 P0/P1；残留 P2（锚点微差/口径统一）→ 本提交清理（18 处：区间右端、v3 归属统一为 14+、mermaid HID 标签/AIDL 悬空边、sequence participant 语法、kill→force-stop、zoneConfig 同名局部变量限定、libbatterylistener 旗标注记等） |

**最终覆盖状态**：259 个文件全部经过至少一轮独立评审；gap 范围 22 文件经 gap+delta（部分再 delta2）共 2–3 轮；所有 BLOCK 判定均已修复并通过下一轮独立复核（delta2 双 OK）。

ch05 循环收口判定：rev05f 复核 a61a862 全部 P0/P1 修复正确、无新 P0/P1，残留 3 项 P2 行号尾差已修复并逐条源码验证（Effects.cpp:3258-3262、流程图 G→I 汇合边 cpp:96-105、节流 fence :4165-4171）。

**评审者误报（已由源码复核回退/纠正，体现"源码 grep 优先"纪律）**：FastTrack 字段 :49-56（原本正确，回退）、sendConfigEvent_l :340（正确，未改）、isPlaybackRestricted :262（原本正确，撤销）、RecordTrack "IDLE→ACTIVE fast/shared" 边（rev05d1 误报引入的错误，rev05e1 确认虚构，已删）、gap15e 对 15_15.15:278 的 P2（文档本已正确，跳过）。gap/delta 轮的 P0/P1 经复核全部属实（无一条回退）。

## 4. 最终验证状态（全部通过，静态）

- **fw-checker 全 18 章**（增补 19–21 章后为 21 章，见 §7）：0 failures（合计 259 文件、~4700 链接、~2100 行号锚点、~4800 fence；UTF-8/U+FFFD/fence 配对/lineref 标识符规则全过）。
- **主题 checker ×11**（aaos/agm/audiod/calibration/focus/gsl-graph/gsl-vm/hal-pal/native/pal-resource/qnx-service-gpr）：0 failures。
- **staged gate**（UTF-8、替换字符、fence 配对、本地链接目标、暂存=工作区、`git diff --check`）：通过；工作区干净。

## 5. 局限与未验证项（如实声明）

1. **静态检查≠语义/渲染/构建/运行时证明**：Mermaid 未渲染验证（mmdc 不可用，仅语法静态检查）；未做产品构建；无真机/运行时行为验证。
2. 部分产品源码目录无 Git 元数据、Android 根无 `.repo` manifest —— 行号锚点以当前工作树为准，未钉住具体 revision。
3. `gsl_fe` 源码未定位（仅 prebuilt 头文件 `prebuilt_HY11/.../gsl_fe/gsl_intf.h`）；相关内容以头文件与调用方为准。
4. 运行时/设备相关条目（如实际路由结果、性能数字、时延）文档中未断言，或明确标注为未验证。

## 6. 成本核算

- LLM API 账户：`yanglinzhen.3`（JoyCode SaaS，本地 LiteLLM 代理）；模型 GLM-5.3-jcloud（上下文 200000）。
- 累计 token（不含缓存）：≈24.7M。

## 7. 增补（2026-11-06）：补建设计规格 §4.1 的 19/20/21 章

终报原以“18 章完成”收口；设计规格 §4.1 信息架构表中规划但未建的三个专题于本次增补完成（分支 `docs/kernel-e2e-evolution`，从 master `1c4b87a` 切出）：

| 章 | 文件 | 内容与边界 |
|---|---|---|
| **19 Android 内核音频** | 6 文+README | techpack/audio 构建链（gvmauto.conf :4-19 在树 vs sa8295gvmauto.conf 未接线双入口）、ASoC 声卡（sa8155.c :10668-10676 与 AR 分支加载的 dummy 变体；DT 双节点在树 direwolf-vm-audio.dtsi）、FE/Platform 对象组、APR-VM 走 HAB MM_AUD_1/2（apr_vm.c :1366-1379，Elite 分支路径）、Elite/AR 启动双分支（init.target.rc :36-45）；AR 主线 PCM 全插件路由静态可证（card-defs.xml card 100 全 PLUGIN + PAL SND_CARD_VIRTUAL 100）；诚实边界：实机启动属性取值不可静态证明、audio_*.ko 改名映射文件未定位 |
| **20 端到端场景** | 5 文+README | 四场景（多音区并发/语音录音/蓝牙通话/异常恢复），全部回链既有已核验章节，仅新增 common_au/car_audio_configuration.xml 的产品锚点（zone0 四音量组 :34-63、输入 Bus :65/:98/:124 等，已逐行验证） |
| **21 版本演进** | 1 文+README | 全库 118 处版本边界的汇总台账（30+ 项，逐项回链出处章）；证据分级 A/B/C 声明“未对照 AOSP 13/14 源码复核”；迁移提示明确标注未验证 |

同步更新：顶层 README（章节目录/学习顺序/去重矩阵）、CLASS_INDEX（内核层+场景/版本条目）、01.1.5（techpack 路径行）。集成文件（README/CLASS_INDEX/01.1.5）的改动均在增补提交内自检（fw-checker 0 failures、链接 0 broken）。写作过程中的自检修正（先行于独立评审）：gvmauto.conf 摘录行号/开关数（:4-19、14 个）、QUIN_GVM 片段归属（autogvm_GKI.config:1 而非 autogvm_QGKI.config:61）、routing-auto/platform_init_auto 选择条件（findstring QUIN_GVM/PLATFORM_AUTO 而非 SND_SOC_AUTO）、auto init 差异（voice/voip 而非 slim）、19.3 sa8155 摘录补全 custom 条目、21.1 出处回链 8 处改指、audio_ar 打包清单补 libautohal_pal。本表独立评审闭环已于同日完成，记录见下。

### 评审闭环记录（2026-11-06，分支 docs/kernel-e2e-evolution）

**第一轮（独立评审，全 BLOCK）**：3 位 fresh-context reviewer 逐章对照源码——k19（3×P0 + 4×P1 + 11×P2，核心：19.6 虚构库名 libsndcarddefs 与“卡定义不在树/路由不可静态证明”负结论，实为 libsndcardparser 在树且 card-defs.xml 双分支接线链完整）；k20（2×P0 + 5×P1 + 13×P2，核心：虚构 PAL_STREAM_VOICE_RECOGNITION 与 commute 设备分类、焦点矩阵执行者与 duck 执行层归因错误）；k21（10×P1 + 10×P2，均在 21.1 版本列与出处回链）。作者对每条发现先直接源码 grep 复核（全部属实）后修复。

**修复提交 `4d370ec`**（18 文件，+250/−147）：19.6 全重写（libsndcardparser/card-defs 双分支接线链）、19.1–19.5 锚点与边界联动修正、20.2 焦点重写、20.3/20.4 删虚构常量、21.1 全部版本列修正、集成文件同步。

**第二轮（delta 复审，全部完成）**：d19fix **PASS with notes**（原 3×P0+4×P1+11×P2 全部 FIXED 且经独立源码复核；新发现仅 4×P2 纳米级锄误）；d20fix **PASS with notes**（原 2×P0+5×P1+13×P2 全部 FIXED；新发现 1×P1：20.2 重写引入的“每 Track duck 仅在 App 收到 CAN_DUCK 时触发”条件句与源码相反，及 1×P2 锚点错接）；d21fix **PASS**（原 10×P1+10×P2 全部 FIXED，8 处新论断全部独立核验属实，零新发现）。

**收尾提交 `ccf51a5`**（6 文件，12 行）：20.2 §3/§6 按复审员处方改写（入口 = FocusRequester.handleFocusLoss，框架 duck 成功时 App 反而收不到回调；AAOS 派发走 dispatchFocusChange 直达 App 不经 duckPlayers）；其余为锄误级锄点/计数修正（19.6 mermaid 19 模块、Android.mk:4、card-defs-dummy 精确描述、19.2 codecs Kbuild :89-116、19.1 WSA 表述加限定、19.5 端点统一、13_13.4 Engine.cpp 锚点归还）。收尾提交的全部引用锚点均经作者直接源码复核（CarAudioFocus.java:138-140、MediaFocusControl.java:766-786、FocusRequester.java:397/:400-410/:487-515、agm/snd_parser/Android.mk:4/:6/:10、dsp/Kbuild:153-168、dsp/codecs/Kbuild:89-116、EngineBase.cpp:70、AudioPolicyManager.cpp:100-140、enginedefault Engine.cpp:346-356、direwolf-vm-audio.dtsi:1251/:1261）。

**最终状态**：三轮独立评审（初评→修复→delta 复审）+ 收尾锄误全部闭环；静态检查 fw-doc-check 21 文件/380 链接/0 失败 + staged gate 通过；剩余未验证项（实机启动属性、HAL/DSP 实际压低效果、mmdc 渲染、构建）均在文中以边界声明标注。
