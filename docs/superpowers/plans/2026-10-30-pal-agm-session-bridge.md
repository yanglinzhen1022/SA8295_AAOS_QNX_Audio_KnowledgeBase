# PAL to AGM Session Bridge Implementation Plan

> **For agentic workers:** 使用 executing-plans 在当前会话连续执行；作者自审。沿用已批准设计，不重复请求确认。

**Goal:** 纠正“PAL 一律直接调用 AGM / pcm_open 一律进入内核”的简化图，建立 SessionAlsaPcm→tinyalsa plugin→AGM session/graph 的本地学习主线。
**Architecture:** PAL 架构概览放 15.2；tinyalsa 操作表与 AGM 插件边界放 15.12；AGM 会话/图机制迁入有实质内容的 18.1，16.10 保留原路径并指向唯一深入正文。只在完成首篇正文时创建 18 章。
**Tech Stack:** C/C++/Android.mk/XML 源码阅读，Markdown/Mermaid，临时 Python 校验。
**Spec:** `docs/superpowers/specs/2026-10-30-audioreach-knowledge-base-design.md`

## Global Constraints

- Android 12L/API 32 + SA8295 + QNX AudioReach；产品源码只读，知识库原分支 docs/audioreach-hal-pal。
- 无新 worktree、merge、push、产品构建或实机操作；工具只放 /tmp。
- 保留旧文件入口，迁移后更新引用；不能将 AGM 在用户态的库名/目录名当作独立进程证明。
- ENABLE_HYP 的 libar-gsl_fe 选择与实际跨 VM 数据传输分开；不以控制消息箭头替代 PCM buffer 追踪。
- 不研究 Elite，不凭 API 名推断 ALSA 内核调用、DSP 拓扑或运行性能。

## Review Focus

1. Session 工厂、PCM ops/plugin 选择、设备节点配置及构建条件是不同层次，不把一种通路写成全部场景。
2. PCM read/write 的 frames/bytes/count 指针、状态、prepare/start 时机与返回值必须按双方实现核对。
3. PCM 和 mixer 控制插件各做什么，metadata 与 connect 和数据面不可混为一条串行调用链。
4. AGM session open/config/prepare/start/stop/close 与 graph 生命周期、引用和失败处理必须区分。
5. GSL 只读到接口/前端链接条件时明确边界，未知传输与 DSP 实现不补造；IPC 名称也不等同于进程或执行域。

## Task 1: PAL 架构与 PCM 插件桥

**Files:** `15_PAL_Architecture/15_15.2_架构层次图.md`、`15_PAL_Architecture/15_15.12_QC_audio-alsa_ALSA层实现.md`。
**Consumes:** 已完成 HAL-PAL 样板；SessionAlsaPcm/SessionAlsaUtils、vendor audio-alsa/tinyalsa、AGM tinyalsa pcm/mixer plugins、snd-card parser。
**Produces:** 普通 PCM 条件路径、函数分派、控制/数据分工、读写计量单位与失败边界。

- [x] 定位实际 tinyalsa fork/头文件、pcm ops 选择和配置入口；阅读 SessionAlsaPcm 调用与插件接收函数。
- [x] 建 `/tmp/qc-agm-doc-check.py`，对旧文先运行路径/围栏/摘录/概念边界检查。
- [x] 重写两篇；图中分别画标准 HAL/PAL、tinyalsa plugin 和条件 IPC 边界；用实际片段解释返回值和启动。
- [x] 执行 `python3 /tmp/qc-agm-doc-check.py --task 1`，零失败；作者核对实际调用双方。

## Task 2: AGM 会话与图，建立实质新章

**Files:** 新建 `18_AGM_GSL_Architecture/18_18.1_AGM会话与音频图.md`、`18_AGM_GSL_Architecture/README.md`；修改 `16_Vendor_QNX_Architecture/16_16.10_AGMAudio_Graph_Manager深度解.md`、15/16 README、总 README、CLASS_INDEX。
**Consumes:** Task 1 桥接；agm.c、session_obj.c、graph.c、metadata.c 与公开/私有头、service/Android.mk、IPC 调用两侧。
**Produces:** session/graph/device 关系，open/config/prepare/start/stop/close，metadata 与 AIF connect，GSL 可见边界。

- [x] 读公共 API 与实际分发、session/graph 生命周期和插件控制入口；识别直接库调用/IPC 的条件。
- [x] 对旧 16.10 的路径、结构、调用和进程描述运行失败检查；保留已确认的有价值深度，删除未经证实推断。
- [x] 完成 18.1 正文后才创建章导航；旧 16.10 改为范围明确的迁移入口，更新正文深入引用；16.9/16.11 相邻导航保留 16.10 兼容入口，不复制两套深度解释。
- [x] 执行 `python3 /tmp/qc-agm-doc-check.py --navigation`，检查本批源摘录、链接、锚点与新旧阅读路线，零失败。
- [ ] 执行 focus/AAOS/Native/HAL-PAL 四批回归和 `git diff --check`；记录作者自审与未验证事项，指定文件提交，继续配置/资源管理及跨 VM 研究。

## Execution Record

- 起点 `bbdbd82`：焦点与配置批次已提交，工作区干净。
- Pre-flight：Task 1 解释如何到达 AGM，Task 2 解释 AGM 内部；不能把 metadata 配置操作画在每次 PCM write 内。迁移 16.10 时保留文件路径。
- 初步定位：PayloadBuilder 在 `pal/session/src/PayloadBuilder.cpp`，不是 utils/src。AGM tinyalsa 插件有 pcm/mixer/compress 三类；dummy_impl 不作为真实调用语义依据。
- Ruling：本批先建立可验证的 PAL→AGM 主桥，不把 10k 行 ResourceManager 和全部 payload/config 深入强行塞入概览；其细节在后续具体计划继续。
- Task 1 旧文检查：2 文件、11 链接、8 摘录、11 Mermaid 块、18 检查失败项；日志 `/tmp/qc-agm-task1-before.log`。不是 18 个独立语义错误。
- Task 1 完成：2 文件、8 链接、7 原文摘录、4 Mermaid 块、0 带锚点入链、0 失败；`git diff --check` 通过。发现并清理 15.2 的 U+FFFD 字符。
- 作者自审（非独立 reviewer）：PLATFORM_VERSION 匹配 12 的 PAL/插件用 vendor libqti-tinyalsa；pcm ops 按节点二选一，而 mixer 汇总 hw/virt group，两者选择机制不同。SessionAlsaPcm::prepare 是空操作，实际 start→pcm_start→pcm_prepare→AGM prepare 不应省略。
- 计量边界：tinyalsa PCM API 成功返回 0；AGM 插件 count 指针的更新未回填 x->result；PAL write 末尾才回填 size，错误可绕过，read 不同。插件 errno=ret 与 tinyalsa EPIPE 比较需保留原实现限制，不宣称运行重启成功。
- 控制边界：SessionAlsaUtils 的部分 mixer setter 未检查返回；metadata/connect 与逐块 PCM 写入分离。libaudioalsa 原有 msm 控件知识保留但不套入 PAL 主链。
- IPC 初查：HwBinders 构建 libagmclient，write memcpy 到 HIDL vector；SwBinders 此处 agm_client/Android.mk 构建的是 agmclient 可执行项，不能仅凭目录名宣称同进程或同等可替换部署。
- Task 2 完成：读取会话池及 get/create、metadata/参数、connect/open、prepare/start/stop/close、read/write、graph 对应实现与 device 引用管理。先完成 18.1 再创建导航，16.10 只保留定位/迁移入口。
- Task 2 旧文检查日志 `/tmp/qc-agm-task2-before.log`：4 目标、10 失败项，含尚未创建 18.1 和预设迁移词缺失，不当作 10 个旧文技术错误。
- 作者自审：私有 metadata 定义在 agm_priv.h 且 sg_props 类型为 sg_prop；合并按 key 保留最早值；CLOSED connect 只标记 AIF_OPEN；无待连接设备的 open -EPIPE 不是 underrun；set_config 发生在 PCM 插件 open 后。
- 作者自审：SLIMBUS 先设备后 graph、其他端点相反；AGM device_start 只推进状态/引用而不调 pcm_start；stop 的 RX/TX 顺序不同；close 保留池对象而释放 graph；graph_close 在 GSL 错误后仍释放本地对象。状态与远端成功不等价。
- 作者自审：read/write 只在局部锁内拒绝 CLOSED，graph 层返回 status/count，HIDL 两侧有拷贝；不宣称并发 close 安全、零拷贝或完整跨 VM 传输。清理覆盖 ret、缓存参数却返回错误的局部路径明确说明，不修产品代码。
- 迁移涉及 16.1 的 AGM 执行域一行更正、16.6/16.11/16.13 各一处深入引用；这些文件其余原文未复核。旧相邻导航经 16.10 跳转仍有效。16 README 删除无依据的 SessionGsl 已废弃历史断言，指向已核对工厂。
- 复验：AGM → 9 文件、181 链接、10 摘录、7 Mermaid 块、0 带锚点入链、0 失败；focus → 11 文件、201 链接、18 摘录、12 图、169 矩阵单元、0 失败；AAOS → 10 文件、186 链接、16 摘录、15 图、0 失败；Native → 11 文件、190 链接、16 摘录、20 图、0 失败；HAL-PAL → 7 文件、9 摘录、0 失败。`git diff --check` 通过。
- 限制：作者自审非独立 review；没有 mmdc、图形渲染、产品构建、服务验证、错误注入、硬件或性能测试。整体资源管理、payload、GSL 跨 VM、QNX/内核、旧文和版本补充仍待完成。
