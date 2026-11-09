# GSL VM Control and Buffers Implementation Plan

> **For agentic workers:** 使用 executing-plans，在原分支由作者执行；整体完成后按 completion-review-gate 进行独立 agent review，不提前将本批当作整体完成。

**Goal:** 用可见 QNX 后端和前端接口证据解释 GSL VM 命令、事件、句柄及缓冲区生命周期，修正双域总结中的执行域和零拷贝概括。
**Architecture:** 新建有实质内容的 17.2 作为跨 VM 控制与数据边界主文；16.12 保留平台全景与正确入口，16.14 的 GSL 本地内部原文另批处理，不冒充已核对。
**Tech Stack:** C、GSL/HAB API、QNX 服务初始化和构建声明、Markdown/Mermaid、/tmp 静态检查。
**Spec:** `docs/superpowers/specs/2026-10-30-audioreach-knowledge-base-design.md`

## Global Constraints

- Android 12L/API 32 + SA8295 + QNX AudioReach；不扩展 Elite，不修改产品源码。
- 原分支 docs/audioreach-hal-pal；不创建 worktree、merge、push、不执行产品/实机操作。
- 前端只有预编译库与头，不补造实现；区分后端可见处理与完整前端发送链。
- 总截止参考 2026-10-31T19:00:34+08:00，未完成允许停止并如实记录；不把静态检查当渲染/编译/硬件结果。

## Review Focus

1. 命令/应答、事件、OOB 描述和 PCM 缓冲不是同一条逻辑通道；TX/RX 采用谁的视角要明确。
2. handle、export_id、QNX vaddr、GVM addr、offset、GSL alloc_info 单位和用途不同；不画成通用可解引用指针。
3. HOST/GUEST 内存来源及协议版本是条件，不统一成 PVM 永远分配或 GVM 永远零拷贝。
4. API 返回状态、更新后的 count、异步完成事件与声学结果不同；失败分支和锁范围按源码。
5. 清理/SSR/断连有不同入口；资源列表清除、服务恢复和旧 PCM 恢复不能混同。

## Task 1: 17.2 跨 VM 控制和缓冲生命周期

**Create:** `17_AGM_GSL_Architecture/17_17.2_GSL跨VM控制与缓冲区.md`。
**Read:** QNX `audio_ar/audio_driver/gsl_be/src/gsl_be.c`、`gsl_vm_be.c` 与 inc 下四个头；`audio_reach/gsl/api/gsl_intf.h`、QNX ar_osal 内存 API；Android gsl_fe 预编译头与 AGM HIDL 两侧已有链。
**Consumes:** 17.1 的 graph/GSL API 边界与 16.7 的 INIT/数据库边界。
**Produces:** 命令通道模型、图句柄映射、两种内存模式、读写/事件、清理解释及源片段。

- [x] 阅读 task/session 建立、HAB open/query/recv/send、opcode 分发、graph open/close 和版本条件。
- [x] 阅读 HOST request/release 与 GUEST provide/retrieve、buffer 查找/地址转换、read_write 两版、回调事件及错误路径；逐字段记录单位与持有关系。
- [x] 创建 `/tmp/qc-gsl-vm-doc-check.py`，以 17.2 缺失和旧 16.12 作为基线，检查链接、片段、围栏、来源及五项 Review Focus 概念，保存初始失败。
- [x] 写 17.2 正文，不虚构前端实现；运行 `python3 /tmp/qc-gsl-vm-doc-check.py --task 1`，零失败。

## Task 2: 双域总结与导航收口

**Modify:** `16_Vendor_QNX_Architecture/16_16.12_Android+QNX双域架构总结.md`、16/17 README、总 README、CLASS_INDEX。
**Read:** QNX `audio_ar/audio_service/src/coreinit.c`、audio_service 相关构建/入口；Android HAL/PAL/AGM 已核对构建条件。
**Produces:** 逻辑层与运行域分开的平台全景、真实证据边界，以及 17.2 的读者入口。

- [x] 对照旧 16.12 的执行域、调用链和数据路径声明；读取 coreinit 的本地 GSL 与后端初始化边界，未确认的进程/装配不补造。
- [x] 重写 16.12 平台总结，以 17.2 承载跨域机制深度，不复制完整协议；保留旧文件名。
- [x] 更新四个导航，只标记确实完成的文章；保留 16.14 待核对状态。
- [x] 运行 `python3 /tmp/qc-gsl-vm-doc-check.py --navigation` 与校准/AGM/focus/AAOS/Native/HAL-PAL 全部回归和 diff --check，全部通过；提交前再检查暂存编码/围栏/新增链接。
- [x] 作者自审并记录局限，8 个指定文档已提交为 `896dc6b`；暂存编码/围栏/新增链接与工作区一致性检查通过，提交后工作区干净。继续资源配置、本地 GSL 与内核等整体剩余主题。

## Execution Record

- 起点 d6d844e；校准计划的提交完成记录在本批工作区补记。
- 已定位后端两个 C 文件，read_write 有两版；session 默认 HOST，V2 可设置 mem_type；audio_service/src/coreinit.c 调用 gsl_be_init。
- Plan self-review：两个产物分别回答机制与全景；不提前迁移 16.14 的内部 GSL 讲解，避免丢失尚需核对的有用深度。该批不覆盖全部库目标。
- 初始检查 `/tmp/qc-gsl-vm-before.log`：2 目标、4 链接、0 匹配范围内代码块、4 Mermaid 块、7 失败项，含缺失 17.2 和必要概念断言。旧文引用块内片段不属于该扫描器代码块覆盖范围；7 不作为语义错误总数。
- Task 1：17.2 单文 5 链接、3 摘录、3 图、0 失败；两正文 20 链接、4 摘录、5 图、0 失败，0 带锚点入链。
- 作者自审：MM_AUD_1 命令/普通应答与 MM_AUD_2 事件不是播放/录音划分；READ/WRITE 应答在外层跳过，失败也不自动补通知；pending 增长与异步完成分别发生，不能当声学计数。
- 作者自审：HOST/GUEST 与消息版本是不同条件；common.mk 覆盖头默认为 V2；OOB 与 PCM 缓冲的分配/导入队列分开；逻辑大小、页对齐步长、export_id、offset 和本地 alloc_handle 分开说明，V2 metadata 声明不等于实际传递。
- 作者自审：GVM -ENODEV 清整个 session；SSR reset_only 清图而非重建所有 HAB；close 与 unexport/unimport 错误不保证全事务。局部 payload 生命周期和分配失败 rc 问题仅按静态代码提示，不修改产品。
- 作者自审：coreinit 创建 detached 后端线程，有 ENABLE_AUDIO_BACKEND 及 HYP710/QHS220 构建条件；不声称代码顺序证明全部进程 ready。本地 GSL 不在 DSP 框内；安全音频优先级和无故障隔离不从域划分推导。
- 最后回归：GSL VM 6 文件/176 链接/4 摘录/5 图；校准 6/165/9/4；AGM 9/200/10/7；focus 11/212/18/12 与 169 矩阵单元；AAOS 10/197/16/15；Native 11/201/16/20；HAL-PAL 7 文件/9 摘录。全部 0 失败，git diff --check 通过。
- 限制：作者自审非独立 review；没有编译、渲染、前端/IPC/HAB/QNX/DSP 运行、错误注入或性能测试。整体目标尚未完成，最终新 agent review 尚未启动。
