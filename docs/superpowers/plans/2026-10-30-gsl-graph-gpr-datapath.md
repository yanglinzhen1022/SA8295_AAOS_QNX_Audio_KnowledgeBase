# GSL Graph, GPR and DataPath Implementation Plan

> **For agentic workers:** 使用 executing-plans 在原分支连续执行；作者为唯一 writer。整体完成后才启动独立整库 review，并按 completion-review-gate 循环修复/复审。

**Goal:** 从本地 QNX AudioReach GSL 实现讲清图对象、子图共享、控制命令与数据完成，改正旧文将 GSL 视为 DSP 实现或严格单一状态机的说法。
**Architecture:** 新增 18.3 为 GSL 内部机制主文；18.1 继续负责 AGM，18.2 继续负责 VM 通道。16.14 保留迁移入口；16.11 解释 SessionGsl 头文件、真实工厂与 GSL API 边界，回链已完成文章。
**Tech Stack:** QNX C、GSL/GPR/SPF 接口、PAL C++、Markdown/Mermaid、/tmp Python 静态校验。
**Spec:** docs/superpowers/specs/2026-10-30-audioreach-knowledge-base-design.md

## Global Constraints

- 本地主线 Android 12L/API 32 + SA8295 + QNX AudioReach；不研究 Elite。
- 产品源码只读；原仓库 docs/audioreach-hal-pal，不创建 worktree、不 merge/push、不构建或实机操作。
- QNX GSL 是宿主侧实现，不能写成 DSP APM 的内部代码；Android gsl_fe 只有已找到的头与预编译证据，不用 args/gsl 替代。
- 超时常量、状态枚举与配置分支不是实际性能、恢复成功或全产品启用证明。
- 暂时检查器只在 /tmp；命令门禁使用 set -e；维护计划不进入读者导航。
- 最终独立 review 尚未启动；未完成允许停止时间为 2026-10-31T19:00:34+08:00。

## Review Focus

1. GSL handle、graph、GKV node、subgraph pool 与 DSP graph ID 不混用；共享 refcount 的变化需按实际命令结果说明。
2. open/prepare/start/stop/close 的状态与错误状态准确；不能根据 APM PREPARE 命令补造 GRAPH_PREPARED 枚举。
3. GPR 发送、basic response、事件回调和超时各自负责什么，锁与 signal token 不构成无限可靠重试。
4. 数据 buffer 所有权、同步/异步 read/write、完成回调、EOS、短传输和关闭清理按分支；不把事件当声学完成。
5. SessionGsl 头声明与工厂产物区分；原文源码样例、图及 API 需一起修正，迁移不丢失有用深度。

## Task 1: GSL 图与控制/数据机制

**Create:** 18_AGM_GSL_Architecture/18_18.3_GSL图生命周期与DSP命令.md。
**Sources:** Qnx/apps/qnx_ap/AMSS/multimedia/audio/audio_ar/audio_driver/audio_reach/gsl 下 api/gsl_intf.h、inc/gsl_graph.h、gsl_common.h、gsl_datapath.h；src/gsl_main.c、gsl_graph.c、gsl_subgraph_pool.c、gsl_common.c、gsl_datapath.c 与实际调用的 msg_builder/shmem 函数。
**Consumes:** 18.1 的 GSL API 入口与 18.2 的 VM 后端调用/缓冲边界。
**Produces:** GSL 主文，负责宿主图对象、命令/响应和数据路径，不重复 ACDB 注册或 VM 协议。

- [x] 读取旧 16.14 与目标源码函数；按图操作及数据操作分别记录真实调用、状态改变及错误出口。
- [x] 新建 /tmp/qc-gsl-graph-doc-check.py；对旧 16.11/16.14 与缺失目标检查链接、路径、摘录、围栏、Review Focus 关键词，记录失败基线，不当语义错误计数。
- [x] 写 18.3：对象关系、路径图、控制时序、buffer 完成时序和关键短摘录；公共 API 与内部实现分层解释。
- [x] 运行 python3 /tmp/qc-gsl-graph-doc-check.py --task 1，静态零失败；作者逐图对照源码，尤其 prepare、错误状态、回调及 buffer 生命周期。

## Task 2: 兼容入口与导航

**Modify:** 16_Vendor_QNX_Architecture/16_16.11_SessionGsl与GSL接口.md、16_16.14_GSLGraph_Service_Layer内部架.md、16/18 README、总 README、CLASS_INDEX。
**Sources:** PAL session/inc/SessionGsl.h、session/src/Session.cpp、Android.mk、AGM service/Android.mk；Task 1 来源。
**Produces:** 无重复 GSL 深讲的兼容入口，读者可从旧链接进入新正文。

- [x] 核对 SessionGsl 声明、实际工厂/构建与 PAL/AGM 接口，重写 16.11；不从未找到 .cpp 推导全产品不可能有实现。
- [x] 18.3 正文完成后将 16.14 改为有定位说明的迁移入口，更新导航并保留相邻链接；未发现指向本批正文的带 anchor 入链。
- [x] 更新 16/18/root/CLASS_INDEX 的职责和已重写范围。
- [x] 运行新检查器 --navigation，以及 PAL/GSL VM/校准/AGM/focus/AAOS/Native/HAL-PAL 全部回归；均为零失败。
- [ ] 作者自审，记录证据/局限，运行 /tmp/qc-staged-doc-gate.py 与 diff 检查后指定文档提交；继续 QNX/内核与剩余层，不宣布整库完成。

## Execution Record

- 起点 547cc1f；PAL 资源批次已提交且当时工作区干净，前批完成号在本批补记。
- Pre-flight：Task 2 消费 Task 1 已完成的 18.3；迁移必须后做。18.1/18.2/16.6/16.7 已有职责，避免在 18.3 重复。
- Ruling：沿用计划清单/执行记录作为 ledger，不建隐藏 sdd 工作区；用户已明确原工作区与文档维护位置。静态检查不替代产品测试，源码阅读与最后独立 review 负责语义核对。
- Plan self-review：五项关注分别由 Task 1 源码/图审查与 Task 2 工厂/链接审查承担；本批不覆盖全部 RTC/动态模块或内核驱动，避免把 15 个 GSL C 文件的存在等同于全部研究完成。
- Baseline：/tmp/qc-gsl-graph-before.log，3 files/21 links/13 excerpts/7 Mermaid blocks/32 failures；其中包括缺失目标与所需概念检查，不是 32 个独立技术错误。
- Task 1：初次静态检查 1 file/18 links/3 excerpts/4 Mermaid blocks/0 failures。写入时发现两处 U+FFFD，检查前立即修复；最终门禁再次检查编码。
- 作者语义自审：init 已设 OPENED，空 GKV 可成功；GRAPH_CLOSED 不是普通 close 的必经赋值；error state updater 不再改写且返回旧态；stop 先改 STOPPED、后发 SG 命令；多个 GKV 的 start 非通用事务。故用源码路径图替代旧文严格状态机。
- 数据自审：补入 STARTED 禁止重配、tag 查询恰好一个 MIID；区分 heap/SHMEM/EXTERN/PUSH_PULL、短传输、提交计数、回调临时描述符、non-shmem packet 生命周期，以及等待/清理错误未传播。全部为静态分析，不声称复现内存安全问题或掉音。
- Task 2 全回归：新批 7 files/222 links/4 excerpts/6 diagrams/0 failures；PAL 6/179/6/5；VM 6/193/4/5；校准 6/182/9/4；AGM 9/220/10/7；focus 11/222/18/12 +169 cells；AAOS 10/207/16/15；Native 11/211/16/20；HAL-PAL 7 files/9 excerpts。所有检查零失败；已检查带 anchor 入链均为零。
- 局限：未 Mermaid 渲染、产品构建、DSP/HAB 执行、故障注入、内存或硬件测试。最后整库独立 review 仍未启动；本批为作者自审。提交前暂存门禁检查 9 个 Markdown 文件通过，随后新批 navigation 检查再次零失败；提交号及提交后状态在下一批起点补记。
