# QNX AudioReach Service and GPR Implementation Plan

> **For agentic workers:** 使用 executing-plans，由父会话在原分支连续研究/编写；每步以复核源码和静态门禁推进，不派发 writer。整体完成后独立整库 review 按 completion-review-gate 执行。

**Goal:** 把 QNX 服务初始化、AudioReach 驱动目录职责与 GPR/GLink 传输分清，去掉旧文把 Elite APR 栈拼接到 AudioReach 主线的解释。
**Architecture:** 原 16.17 文件名保持不变，正文改讲 audio_ar/audio_service；16.19 保留 APR 历史路径入口身份，实质转为当前 GPR 路由/所有权；16.16 讲 AudioReach 后端边界并回链 17.2/17.3，不重复 VM 协议。16.18 若仅旧 AMS 内容，则保留说明和现有可用主文导读，不补造其 AR 对应实现。
**Tech Stack:** QNX C、GPR/GLink、AudioReach 服务、Markdown/Mermaid、/tmp Python 静态检查。
**Spec:** docs/superpowers/specs/2026-10-30-audioreach-knowledge-base-design.md

## Global Constraints

- Android 12L/API 32 + SA8295 + QNX AudioReach；不读取 Elite 实现来扩充主线，不开展 Elite 专题。
- 只改知识库文档，原分支 docs/audioreach-hal-pal；不 worktree/merge/push/构建/实机操作。
- 保存 01–17 路径，维护计划不进读者导航；临时检查器只在 /tmp。
- 函数、线程、用户态库、resource manager、设备节点、内核驱动、DSP 各有边界。ThreadCtl、目录名 driver/island 或 `/dev/` 不证明组件执行于内核。
- 库的初始化成功/线程创建成功/远端 channel 连接/端口注册/命令处理成功各自独立。
- 脚本 set -e；使用 python3；静态检查不冒充运行/内存/性能验证。
- 整体未完成；独立 review 尚未开始；允许未完成停止时间 2026-10-31T19:00:34+08:00。

## Review Focus

1. audio_service 主线程/异步 PIL、FastRPC/backend、OEM 线程的真实先后与 join/detach；不串成全同步就绪链。
2. initFlags、-f、SECPOL、ifs_base、路径生命周期与错误处理；注释/日志不替代执行语句。
3. GPR 目的 domain 与注册 port、GLink port name、packet pool 和音频 PCM buffer 不混用。
4. async_send/tx_done/APM basic response/audio buffer DONE 不混用；回收必须按发送/接收 packet 来源分析。
5. link up/open/connected、rx intent 失败、断开重开与 GSL SSR 恢复分层；不制造自动完整恢复保证。

## Task 1: QNX 服务与 AudioReach 后端边界

**Modify:** 16_Vendor_QNX_Architecture/16_16.17_QNX_audio_service_vm_VM音频服务.md、16_16.16_QNX_audio_driver_vm_VM音频驱动层.md；必要时将 16_16.18_QNX_ams_lib_音频管理服务库.md 调整为历史入口/当前阅读导航，明确不再展开 Elite。
**Sources:** Qnx/apps/qnx_ap/AMSS/multimedia/audio/audio_ar/audio_service/src/main.c、coreinit.c；该服务 common.mk、aarch64/o-le-s8295-ar-qvmhost/Makefile；实际找到的 sysctrl/Resource_Daemonize 实现；audio_driver 的当前 AR 入口及 gsl_be 对接，不凭目录推导 AVMM/VAPM 必经。
**Consumes:** 17.2 VM backend 深度、17.3 GSL graph 深度、16.12 双域全景。
**Produces:** 服务启动时序、主线程/后端工作线程与 resource manager 边界，旧 driver_vm/AMS 路径的准确主线定位。

- [x] 读取旧 16.17 全文、16.16/16.18 的入口及主题/源码归属，核对当前 AR service 完整入口、直接依赖与目标构建条件；非主线细节按下方 ruling 处理。
- [x] /tmp/qc-qnx-service-gpr-doc-check.py 检查目标 links/源码路径/短摘录/围栏/核心概念；保存改写前失败基线 /tmp/qc-qnx-service-gpr-before.log。
- [x] 重写 16.17：启动主线、并行初始化、条件后端、错误/路径生命周期与观察层次；以少量精确摘录支撑。
- [x] 重写 16.16 与处理 16.18：不把 AR 目录下库集误称统一内核 VM 驱动；保留真正适用的机制并回链深讲，非主线说明简洁。
- [x] 运行 python3 /tmp/qc-qnx-service-gpr-doc-check.py --task 1，静态零失败；作者检查图中每个线程与返回通道。

## Task 2: GPR/GLink 与导航收束

**Modify:** 16_Vendor_QNX_Architecture/16_16.19_QNX_apr_lib_APR协议库.md、16 README、root README、CLASS_INDEX；17.3 仅必要下游回链。
**Sources:** audio_ar/audio_driver/audio_reach/gpr 的 platform/qnx/gpr_init_wrapper.c、core/src/gpr_drv.c、gpr_drv_island.c、选中 session 实现、datalinks/gpr_glink/src/gpr_glink.c/gpr_glink_i.h、api/gpr_packet.h/ipc_dl_api.h；实际 get_glink_port_data_info 实现及构建选择。
**Consumes:** 17.3 GSL packet/signal 与 Task 1 服务/底层初始化区分。
**Produces:** GPR 按域分发与本地端口回调、GLink 生命周期、packet ownership、传输完成与 DSP response 的分层教材。

- [x] 追完整 send/free/receive/send_done/register 与 GLink state/tx/rx 路径；核对平台 datalink 表和 domain 参数。
- [x] 重写 16.19：保留路径，不沿用 APR 包/域编号/GSL→APR 假链；配路由图和发送/回收时序图。
- [x] 更新导航及已核对范围，17.3 回链下层传输；其余旧文不可标成已核对。
- [x] 新检查器 --navigation，加 GSL graph/PAL/VM/calibration/AGM/focus/AAOS/Native/HAL-PAL 全回归，静态零失败。
- [ ] 作者自审、暂存门禁和 diff 检查后仅提交指定文档；记录提交与剩余范围，继续下一批。

## Execution Record

- 起点 4d6ba19；GSL graph 批次提交 9 文件、406 insertions/824 deletions，门禁及新批/旧批静态回归通过；提交后工作区干净。
- Pre-flight：Task 2 的 GPR 传输不能重复 Task 1 的 service 编排，也不能取代 17.3 的 GSL command response；共享的是上下层边界，不共享虚构的统一进程/内核模型。
- Ruling：将旧 16.19 的主要篇幅用于 AR GPR，而不新建相同职责的 17.4；旧名保留兼容，标题/导读清楚说明当前内容。16.16/17/18 同样不继续教授用户已排除的 Elite。
- 初步源码发现（尚未全部写入正文）：AR main 有 `0 < open(DEVICE_NAME)` 检查而不是完整单实例原语；core init 先置 inited 后调用 base；PIL 线程虽有 skip 日志仍执行 pil_start_subsystem；base join PIL 后检查 adsp_pil_load_status，FastRPC/OEM 后续 detach；FastRPC 失败日志后仍可尝试 gsl_be_init。ifs_base 分支将块内数组地址存入静态指针，生命周期须说明；不做运行复现。
- 初步 GPR：QNX wrapper 指定 APPS 本地域、ADSP glink/APPS local 表；ipc_dl_glink_init 未传播 gpr_glink_init 返回值；GLink tx_done 是 packet 所有权回收，不是 APM 应答。尚需继续完整追收包和 free。
- 搜索 sysctrl/Resource_Daemonize 于 audio_driver/utils 未命中导致一次探索命令退出 1，set -e 阻止后续构建条件搜索；这不是验证通过，需重新扩大定位并单独读取构建文件。未修改产品代码。
- Plan self-review：每个 Review Focus 分别落在服务时序/错误表与 GPR 生命周期/所有权；不把服务启动成功作为声音或安全隔离证明，不补造缺失 CSD2 全部实现专题。
- Ruling：旧 16.16/16.18/16.19 的非主线细节仅检查主题归属与混接风险，不继续逐段研究 Elite 产品实现；原细节保留在 Git 历史，当前用 AR 后端/服务/GPR 及有实质主文的兼容导读替代。代价是这些旧专题不再作为本地主线教材，但符合用户明确排除 Elite 的范围。
- Ruling：额外更新 16.14 与 18 README 的下游回链/剩余范围，避免已有入口仍说 GPR/GLink 尚无讲解；不扩展修改未核对原文。
- Baseline：4 files/36 links/35 excerpts/32 Mermaid blocks/68 failures（/tmp/qc-qnx-service-gpr-before.log）；含旧摘录与主线关键概念检查，非 68 个独立技术错误。
- Task 1 初次通过：3 files/44 links/3 excerpts/2 diagrams/0 failures。Task 2 正文通过：4/62/5/5/0。写入时两处 U+FFFD（含一次文件名链接）当场修复，后续静态门禁再检查。
- 作者自审补充：sysctrl 起初在错误目录搜索，后定位 platform/utilities/sysctrl 与 platform/qal/clients/dll_utils；未找到的猜测路径未写入正文。Resource_Daemonize 忽略后台化返回；sysctrl devctl 无用户 hook 时 ENOSYS、write 忽略 dispatcher 返回；process exit code 不等于启动成功。CSD2 入口真实文件为 csd2_api.c，不是先前尝试的 csd2.c。
- GPR 自审：common.mk 选 array_based/cmn 并定义 audioss_proc 禁 QoS；默认两静态 packet 池，不混成 PCM。GLink init/连接/intent/tx_done/业务应答分层；记录分发失败仍返回 EOK、默认回调 end_command 后上层再次 free 的静态风险，不声称运行复现。
- 2026-10-30T21:14:57+08:00 全回归：QNX 8/248/5/9；GSL graph 7/235/4/6；PAL 6/186/6/5；VM 6/202/4/5；校准 6/191/9/4；AGM 9/229/10/7；focus 11/229/18/12 +169 cells；AAOS 10/214/16/15；Native 11/218/16/20；HAL-PAL 7 files/9 excerpts。均零失败，相关带 anchor 入链为零；diff --check 通过。
- 提交前暂存门禁：12 个 Markdown 文件通过 UTF-8/U+FFFD/围栏/新增本地链接/暂存工作区一致性与 diff 检查，新批 navigation 再次零失败；提交号与提交后状态下一批补记。
- 局限：仅源码/Markdown 静态验证与作者自审。没有 Mermaid 渲染、产品构建、HAB/GLink/DSP 执行、内存测试、错误注入、安全隔离或硬件测试；整库改造与独立 review 均未完成。
