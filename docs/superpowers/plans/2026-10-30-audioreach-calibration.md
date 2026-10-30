# AudioReach Calibration Implementation Plan

> **For agentic workers:** 使用 executing-plans 在原分支由作者连续执行与自审。已授权不重复请求审批；本批由作者执行。整体完成后的独立 agent review 与时间界限遵循 `2026-10-30-completion-review-gate.md`。

**Goal:** 重写校准两篇，明确 AudioReach 数据库选择、AGM/GSL 加载、QNX 多数据库与远端路径映射，删除以 Elite loader 推演 AR 的主链。
**Architecture:** 16.6 讲数据库与 GKV/CKV/TKV 的作用、AGM 初始化、GSL/ACDB 查询；16.7 讲源文件部署、远端数据库注册与路径边界。AGM 对象生命周期引用 18.1，不复制。
**Tech Stack:** C/C++、GSL/ACDB headers、Android.mk/QNX filesets、Markdown/Mermaid、/tmp Python 检查。
**Spec:** `docs/superpowers/specs/2026-10-30-audioreach-knowledge-base-design.md`

## Global Constraints

- Android 12L/API 32 + SA8295 + QNX AudioReach；只修改知识库。
- 保留原路径，分支 docs/audioreach-hal-pal；不新建 worktree、merge、push，不构建或运行产品/硬件。
- 不扩展 Elite。旧文的 legacy 参数表不是本地主线，不能为保留篇幅继续混写。
- 前端仅有预编译库与头时，明确实现不可见；不拿 Android args/gsl 的完整本地实现替代 gsl_fe。
- fileset、mk 拷贝意图、文件存在、数据库加载和 DSP 生效分开；不导出校准二进制内容，不宣称性能结果。
- 临时工具只在 /tmp，失败门禁用 set -e/显式 &&，不能失败后继续提交。

## Review Focus

1. GKV 图选择、CKV 校准、TKV 标签与旧 ACDB numeric device ID 的角色不能混为固定映射。
2. graph_init 扫描路径、属性、文件过滤及 delta 构造按源码解释；空目录/缺失文件不是必然默认成功。
3. QNX add-database opcode 的 payload、路径前缀和 handle 与 gsl_init 初始数据库是不同操作。
4. 后端将远端绝对路径改为 /vm/%03d 前缀，并不证明对应挂载/文件权限/内容已存在；不能据此说同步拷贝或 QNX ACDB 与 Android 完全相同。
5. 校准参数存在、数据库被注册、graph 查询和模块应用是不同阶段；前端不可见的控制流与错误返回明确限定。

## Task 1: AudioReach 校准基础与数据库初始化

**Modify:** `16_Vendor_QNX_Architecture/16_16.6_ACDB校准体系.md`。
**Consumes:** 18.1 metadata 与 graph_open；`vendor/qcom/opensource/agm/service/src/graph.c`、`service/Android.mk`；QNX `audio_ar/audio_driver/audio_reach/gsl/src/gsl_main.c` 与 ACDB 调用头；gsl_fe 导出头。
**Produces:** AR 校准模型、初始化文件列表/delta、图查询边界，必要图与真实片段。

- [x] 阅读 graph_init/get_acdb_files_from_directory 与本地编译条件；对照 QNX gsl_init、graph 查询调用和实际结构。
- [x] 编写 `/tmp/qc-calibration-doc-check.py`，检查本批链接、源路径、原文摘录、围栏和 Review Focus 必要概念；先在旧文运行并保存失败记录。
- [x] 重写 16.6：不保留 loader→AGM_IOCTL 等无来源链；用源片段解释文件列表、GSL/ACDB 与 KV，区分可见源码和预编译前端。
- [x] 运行 `python3 /tmp/qc-calibration-doc-check.py --task 1`，零失败；作者核对路径、空/失败分支及参数用途。

## Task 2: QNX 远端数据库与部署边界

**Modify:** `16_Vendor_QNX_Architecture/16_16.7_ACDB校准数据Android与QNX双域共享.md`、16/18 README、总 README、CLASS_INDEX。
**Consumes:** Task 1；`Qnx/apps/qnx_ap/AMSS/multimedia/audio/audio_ar/audio_driver/gsl_be/src/gsl_vm_be.c`、`inc/gsl_vm_msg.h`；QNX GSL gsl_add_database；`Qnx/apps/qnx_ap/target/filesets/qc.audio.common.build`；Android AR 配置与 mk。
**Produces:** QNX 初始库与 GVM 注册库分离、opcode/路径/handle、安装与运行证据界限。

- [x] 阅读 ADD_ACDB_DATABASE 接收 payload、路径重写、gsl_add_database 与删除/会话清理；检查 fileset 宏条件和 Android 配置路径来源。
- [x] 重写 16.7，保留源配置路径示例，不杜撰复制协议、VM 编号、实际加载拓扑或 mmap 数据共享；未知前端步骤在图中明确。
- [x] 更新阅读路线，只将两篇已完成内容加入已核对主线；其余 QNX 原文仍待核对。
- [x] 运行 `python3 /tmp/qc-calibration-doc-check.py --navigation`、AGM/focus/AAOS/Native/HAL-PAL 回归及 `git diff --check`，均通过。提交前另检查暂存 Markdown 编码/围栏/新增链接。
- [x] 记录作者自审与未验证事项，8 个指定文档已提交为 `d6d844e`；暂存编码、围栏、新增链接和工作区一致性检查通过，提交后工作区干净。继续 GSL 前后端、资源配置与内核主题。

## Execution Record

- 起点：`5cc852d`，AGM 桥接正文 `d80d8df`，附加旧文围栏修正已提交；工作区干净。
- 已定位：gsl_fe 在 prebuilt_HY11 的 msmnile_gvmq 目录有 lib32/lib64 与 gsl_intf.h；当前定位未发现前端源码。Android args/gsl 与 QNX audio_reach/gsl 是另外的实现，不能替代前端调用证据。
- 已定位：QNX ADD_ACDB_DATABASE=38；gsl_vm_be_add_acdb_database 重写 /vm/%03d 路径后调用 gsl_add_database；fileset 的 AR+VARIANT_8540 从 adp_8295_ar 安装数据库、workspace 与 delta。
- Self-review：本批只覆盖已批准设计中的校准子主题；不代替完整 GSL/内核/恢复与整体库完成。两篇分别负责加载机制和跨域部署，避免重复类说明；依用户约束采用静态校验、作者自审和原工作区，不做产品测试。
- 旧文基线 `/tmp/qc-calibration-before.log`：2 文件、20 链接、8 摘录、5 图、20 失败项。含旧 legacy 摘录不匹配本批 AR 源池和必要概念缺失，不当作 20 个独立技术错误。
- Task 1 检查：1 文件、5 链接、4 摘录、2 图、0 失败。Task 2 两正文：2 文件、9 链接、9 摘录、4 图、0 失败；均无带锚点入链。
- 作者自审：扫描为 strstr，不是严格后缀；num_files 上限检查变量与 i 增长不同；空目录可扫描成功，但 QNX acdb_init 拒绝空列表；delta 长度与普通条目长度计量不同。均按源码保留边界，未修改产品。
- 作者自审：QNX CSD2 初始文件集不同于 VM INIT，后者只校验版本；ADD_ACDB_DATABASE 的 OOB 是路径描述，payload_size 未在 handler 用于完整验证；/vm/%03d 与 +7 长度不代表范围校验或已存在挂载。
- 作者自审：QNX ar_mem_cpy 按最小长度复制，后端传入 filename 容量而拷贝整个条目，不保证完整更新尾部；CSD2 环境重定向存在块内数组指针生命周期问题；数据库删除返回值在 session cleanup 未处理。这些为静态分析，不是错误注入结果。
- 作者自审：保留 AR ATS 主题但纠正“固定 TCP/IP 通道”概括，diag/TCP server 有宏条件，初始化服务错误路径不同；前端只有预编译证据不画成已验证实线调用链。
- 最后整批检查：校准 6 文件/157 链接/9 摘录/4 Mermaid 块；AGM 9/192/10/7；focus 11/208/18/12 并核对 169 矩阵单元；AAOS 10/193/16/15；Native 11/197/16/20；HAL-PAL 7 文件/9 摘录。全部 0 失败，git diff --check 通过。
- 限制：没有图形渲染、产品构建、单元/IPC/实机/调音/性能测试；本批作者自审非独立 review。整体完成后按用户新要求另启独立 reviewer，当前尚未到整体完成阶段。
