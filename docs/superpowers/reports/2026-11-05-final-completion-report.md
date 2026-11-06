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

- **fw-checker 全 18 章**：0 failures（合计 259 文件、~4700 链接、~2100 行号锚点、~4800 fence；UTF-8/U+FFFD/fence 配对/lineref 标识符规则全过）。
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
