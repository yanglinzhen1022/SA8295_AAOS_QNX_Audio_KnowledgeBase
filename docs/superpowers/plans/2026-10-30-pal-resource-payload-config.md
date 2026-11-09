# PAL Resource, Payload and Configuration Implementation Plan

> **For agentic workers:** 由作者使用 executing-plans 在原分支连续执行；整体验收遵循 completion-review-gate，不以单批完成代替整体目标。

**Goal:** 重写 PAL ResourceManager、PayloadBuilder、配置三篇，解释资源分配、设备参数选择、KV 与模块 payload 的真实职责及配置来源。
**Architecture:** 15.6 管运行期资源和状态，15.7 管选择器/KV/模块参数构建，15.9 管配置选择与安装边界；共同承接 15.12、17.1，但不重复 AGM 或 GSL VM 深度。
**Tech Stack:** PAL C++、Expat、tinyalsa、AR XML/Android.mk、Markdown/Mermaid、临时 Python 静态校验。
**Spec:** `docs/superpowers/specs/2026-10-30-audioreach-knowledge-base-design.md`

## Global Constraints

- 主线 Android 12L/API 32 + SA8295 AudioReach；源码只读，文档保留原文件名。
- 原分支 docs/audioreach-hal-pal；不用 worktree、merge、push，不构建或运行产品。
- 不能从配置文件存在推断安装选择、DSP 图或板级接线；不能把 audio_platform_info 的其他 HAL 路径强套到 PAL。
- 不扩展 Elite；不虚构统一零拷贝、统一流状态、固定优先级或性能数据。
- 静态检查工具仅放 /tmp；所有提交门禁失败时立即停止命令链。
- 整体完成后新起独立 reviewer 并循环修复；2026-10-31T19:00:34+08:00 后若未完成允许停止并如实记录。

## Review Focus

1. FE ID 分配/归還、BE 名称/设备映射、Stream 注册和设备引用是不同对象关系，不等于 APM 路由决策。
2. RM 构造、init、配置解析、声卡/路由句柄创建的失败行为不能写成始终成功；单例可访问不等于所有资源 ready。
3. XML selector 匹配顺序与缺省/失败行为必须按实际代码，KV 不是任意硬编码字典；重复 key 与 selector 去重不是同一层操作。
4. payload buffer 大小、对齐、MIID/MID/parameter ID、释放责任及状态返回不能混同；挑选一个实际模块路径贯通到 Session。
5. 同名配置、平台后缀、构建复制规则与实际安装分开；保留适用域，修正不存在的路径和无条件功能启用说法。

## Task 1: ResourceManager 运行期资源

**Modify:** `15_PAL_Architecture/15_15.6_ResourceManager-PAL核心管理模块.md`。
**Sources:** `vendor/qcom/opensource/pal/resource_manager/src/ResourceManager.cpp`、`inc/ResourceManager.h`；SessionAlsaPcm/SessionAlsaUtils/StreamPCM 的调用点。
**Produces:** 初始化与单例、FE 池、Stream 注册/注销、BE/设备配置、路由/并发边界。

- [x] 阅读 ResourceManager 构造/getInstance/init_audio/init、allocateFrontEndIds/freeFrontEndIds、registerStream/deregisterStream、getDeviceConfig 及相关调用点。
- [x] 编写 `/tmp/qc-pal-resource-doc-check.py` 检查三篇旧文与本批来源、摘录、链接、围栏及 Review Focus 概念；保存初始失败。
- [x] 重写 15.6，通过具体普通 PCM 场景解释资源职责，同时指出 Voice/Compressed/Non-Tunnel 等分支不能套用同一资源池；保留必要状态与调用图。
- [x] 运行 `python3 /tmp/qc-pal-resource-doc-check.py --task 1`，预期零失败；作者核对锁、失败回退与 ID 生命周期。

## Task 2: PayloadBuilder 与 KV/参数

**Modify:** `15_PAL_Architecture/15_15.7_PayloadBuilder-载荷构建器.md`。
**Sources:** `vendor/qcom/opensource/pal/session/src/PayloadBuilder.cpp`、`session/inc/PayloadBuilder.h`；usecaseKvManager.xml；Session 调用点及 DSP 参数头。
**Produces:** XML→selector→GKV/CKV/TKV 的边界、模块 payload 样例、内存与错误责任。

- [x] 阅读 init/XML callbacks/retrieveKVs/populateStreamKV/populateDeviceKV/populateCalKeyVector，区分双向与单向重载。
- [x] 跟踪 payloadVolumeConfig 或 payloadMFCConfig 的实际头、分配、对齐、填字段、发送与释放调用点，不自造示意 API。
- [x] 重写 15.7，完整摘录短的关键段，并将伪代码/概念布局标作 text，不冒充源码。
- [x] 运行 `python3 /tmp/qc-pal-resource-doc-check.py --task 2`，预期零失败；自审 selector 默认行为与 payload 单位。

## Task 3: 配置选择与读者导航

**Modify:** `15_PAL_Architecture/15_15.9_配置文件列表.md`、15 README、总 README、CLASS_INDEX。
**Sources:** RM 的 SNDPARSER/init_audio/XmlParser、PayloadBuilder USECASE_XML_FILE；AR HAL `configs/msmnile_au/msmnile_au.mk` 与实际 card-defs/mixer/usecase/resource XML。
**Produces:** 按消费者而非罗列名称的配置地图，以及具体文件选择过程。

- [x] 找到实际 resourcemanager/usecase/mixer/card-defs 文件及其解析入口/平台选择，核对 mk；未找到的安装关联明确不确定性。
- [x] 重写 15.9，分开 Car audio 配置、Policy 配置、PAL resource/KV、tinyalsa 卡定义、mixer path、ACDB 与 QNX 配置。
- [x] 更新三个导航，未改写 PAL 旧文继续标明待核对。
- [x] 运行 `python3 /tmp/qc-pal-resource-doc-check.py --navigation` 及 GSL VM/校准/AGM/focus/AAOS/Native/HAL-PAL 回归；检查暂存文件编码、围栏、链接及 diff。
- [x] 作者自审、记录局限并指定文档提交；继续本地 GSL/内核与剩余层。

## Execution Record

- 起点 896dc6b；上批提交完成记录在本批补记。
- 当前三篇旧文共 1613 行；此数仅为阅读范围，不作为质量指标。
- 已定位 RM 的默认 virtual card 100、card-defs 解析、声卡后缀决定 mixer/resourcemanager XML；PayloadBuilder 使用 /vendor/etc/usecaseKvManager.xml（非 Android 分支另有路径）。细节尚需逐函数核对，不预先写成结论。
- Plan self-review：分离资源、载荷、配置三个学习职责；检查覆盖 ID、选择器、内存及安装条件，不扩大为重写 10k 行 RM 的所有特殊流。用户已授权连续执行，无新审批门槛。
- Pre-flight：Task 1/3 共享 RM XML 选择逻辑，先读真实选择过程、分别解释消费者与配置语义；Task 2/3 共享 usecase XML，正文只在 15.7 深入解析，15.9 提供来源和安装边界。未发现接口矛盾。
- Ruling：沿用 docs/superpowers 计划中的检查清单和执行记录作为 ledger，不另建隐藏 sdd 工作区或运行 task-start 脚本；因为用户限定文档修改、维护记录位置及原工作区。代价是需手工核对提交与复验记录，不能依赖脚本自动标记。
- Ruling：本批无人类运行的产品代码变更；临时工具仅做文档链接、摘录与格式一致性检查，不把词语命中当行为测试，也不为满足 TDD 执行禁止的产品测试。语义准确性仍由源码阅读和最终独立 review 承担。
- Ruling：最终 review 按用户新要求在整体完成后新起 agent 并循环修改/复审，不采用技能的一次修复且不复审上限；不提前将单批结束当整体验收。
- 已找到车载资源文件与 usecase XML 的专用位置：vendor/qcom/proprietary/mm-audio-auto/auto-casa-xml；后续以其真实内容及 Android.mk 对照，不拿 monaco/taro 配置替代 8295。
- 初始静态基线：3 文件 / 26 链接 / 19 摘录 / 15 Mermaid 块 / 0 anchored incoming / 33 失败，保存在 /tmp/qc-pal-resource-before.log；包含新增概念覆盖检查，不是 33 个独立技术错误。
- Task 1：15.6 重写完成；单例构造与 pal_init 后续步骤、devInfo/deviceInfo、不同 FE 池、注册时机、设备字段覆盖、EC 计数、检测流并发和 SSR 队列均已按相应函数核对。静态检查 1 文件 / 8 链接 / 1 摘录 / 2 图 / 0 失败。
- Task 1 作者复核保留了 Non-Tunnel 从 max+i 且 i=0、Voice 按值 vector 分配与成员池归还、分配 TODO lock/erase 后迭代器、未知注册类型仍进总列表的边界；没有把源码意图升级为无冲突、线程安全或原子恢复保证。
- Task 2：15.7 重写完成；核对 selector 数量升序、候选 pair 比较、外层多项追加、CustomConfig fallback、单/双向重载及被忽略返回值；纠正 CKV 不含采样率/通道数的绝对说法。静态检查 1 文件 / 9 链接 / 3 摘录 / 2 图 / 0 失败。
- Task 2 作者复核追到 Volume 的 tag→MIID→payload→setParam→freeCustomPayload；明确 new[]/free 配对风险、setMixerParameter 名称分配失败释放实参后调用者仍持指针、发送后 Session 返回 0。MFC 的 calloc 与 Volume 不混用。只记录静态风险，不修改产品或声称复现崩溃。
- Task 3：15.9 与导航完成；区分 RM 固定 SNDPARSER、resource/mixer 的 SKU+声卡后缀、PB 固定 USECASE_XML_FILE、snd-card-parser property；区分 gvmq mixer 复制与 gvmgh Car/Policy 复制条件。保留真实 device_profile 标签和 8/32 声道配置，而非沿用虚构 XML/平台型号表。静态单篇 1 文件 / 9 链接 / 2 摘录 / 1 图 / 0 失败。
- 全部静态回归：PAL 6/175 links/6 excerpts/5 blocks；GSL VM 6/182/4/5；校准 6/171/9/4；AGM 9/209/10/7；focus 11/218/18/12 且 169 矩阵单元；AAOS 10/203/16/15；Native 11/207/16/20；HAL-PAL 7 文件/9 摘录。全部 0 失败，diff --check 通过。
- 作者自审不是独立 review；最终独立整体验收尚未启动。没有 Mermaid 渲染、产品构建、实机、IPC、DSP、故障注入或内存检测器运行。8 个拟提交 Markdown 已通过暂存 UTF-8/U+FFFD、围栏奇偶、新增本地链接、工作区/暂存一致性及双 diff 检查。提交前重跑暂存门禁和 PAL navigation 检查均通过；提交 547cc1f（8 文件，451 insertions/1438 deletions），提交后工作区干净。继续 GSL 图/GPR/data path 批次；整体未完成。
