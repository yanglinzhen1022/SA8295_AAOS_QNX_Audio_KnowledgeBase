# AAOS Focus and Configuration Implementation Plan

> **For agentic workers:** 使用 executing-plans 在当前会话连续执行，遵守用户允许作者自审、不新建工作树、不再请求逐批批准的约束。进度记录在本计划，不建立交付的审计系统。

**Goal:** 解释本地 AAOS 配置对象、焦点分区、交互矩阵、holders/losers/延迟请求以及 HAL 外部请求的实际状态变化。
**Architecture:** 9.3/9.18 解释配置模型与解析；9.4/9.10 解释单区域算法及分发；12.6 只解释交互矩阵与场景，不复制完整调用链；10.7 深入 HAL 请求生命周期并回链 wrapper。
**Tech Stack:** Java/XML 源码阅读，Markdown/Mermaid，临时 Python 检查。
**Spec:** `docs/superpowers/specs/2026-10-30-audioreach-knowledge-base-design.md`

## Global Constraints

- Android 12L/API 32 + SA8295 + QNX AudioReach；源码只读，编辑仅在知识库。
- 保留 01–17 目录与文件名；原分支 docs/audioreach-hal-pal，无新工作树、merge、push、产品构建或实机操作。
- 每条图和片段按本地实现；不引入新版 OEM focus evaluator、zone config 动态切换、镜像或 HAL gain callbacks。
- 使用临时 `/tmp/qc-focus-doc-check.py`，不交付工具工程；围栏和源码匹配不代表图形渲染或运行验证。
- 作者自审不冒充独立 review；总体工程未完成，不用这一批充当全库验收。

## Review Focus

1. 非法 zone、bundle 附带 zone 与 UID zone 选择不同；权限必须按实际入口核对，不预设 bundle 分支额外鉴权；焦点分发不等于设备 affinity 修改。
2. 并发矩阵是方向性的，并发结论还受 duck/pause/request flags 影响，不等价于允许任意两路混音。
3. holders/losers/blockers、永久/暂时 loss、延迟请求、同 clientId 替换与恢复必须逐分支分析。
4. 配置版本 1/2、primary、Context 完整性、地址唯一性和输入补充必须区分解析器/验证器职责。
5. HAL 请求只保存 GRANTED；DELAYED 反馈 LOSS；reset 在遍历中移除 SparseArray 元素的行为不能按注释宣称总是完整清理。

## Task 1: Zone 和配置解析

**Files:** `09_AAOS_Car_Audio/09_9.3_CarAudioZone-多Zone音频管理.md`、`09_AAOS_Car_Audio/09_9.18_CarAudioZonesHelper-XML配置解析.md`。
**Consumes:** 已完成 9.1/9.11；CarAudioZone、CarAudioContext、CarAudioZonesHelper、CarAudioZonesValidator 与真实 AR XML。
**Produces:** 静态 zone/group/context/address/input 模型，版本/默认/错误处理和配置到验证边界。

- [x] 读四个 Java 类和 XML，对照旧文识别不存在的动态配置对象。
- [x] 写临时 checker，先在六篇旧文执行。检查 UTF-8、围栏、Markdown 目标/简单锚点、Java/XML 原文匹配，并添加主区域/版本/输入补充的必要概念检查。
- [x] 重写两篇，至少包含对象图、解析流程图、真实摘录及非法配置如何失败。
- [x] 执行 `python3 /tmp/qc-focus-doc-check.py --task 1`，两篇零失败；自审见执行记录。

## Task 2: 多区域分发、单区域算法与矩阵

**Files:** `09_AAOS_Car_Audio/09_9.4_CarAudioFocus-车载焦点管理.md`、`09_AAOS_Car_Audio/09_9.10_CarZonesAudioFocus-多Zone焦点分发器.md`、`12_Audio_Focus_Deep_Dive/12_12.6_AAOS焦点交互矩阵.md`。
**Consumes:** Task 1 区域模型；CarAudioFocus、CarZonesAudioFocus、FocusInteraction、FocusEntry。
**Produces:** 焦点 zone 选择、矩阵方向/条件、holders/losers/blockers、delayed 请求与恢复。

- [x] 阅读以上四个完整类及 AudioPolicy focus result callback 使用，先区分通知结果与实际播放器动作。
- [x] 为方向性矩阵、pause-on-duck、delayed、blockers、zone 校验加入必要概念断言；先对旧文执行失败检查。
- [x] 重写三篇，算法主线放 9.4，分发放 9.10，12.6 用源码矩阵和具体场景解释拒绝/独占/并发，不再复制整套 CarAudioService。
- [x] 执行 `python3 /tmp/qc-focus-doc-check.py --task 2`，三篇零失败；按 Review Focus 作者自审。

## Task 3: HAL 外部焦点、导航与批次提交

**Files:** `10_AudioControl_HAL/10_10.7_HalAudioFocus-外部焦点请求管理.md`；09/10/12 README、总 README、CLASS_INDEX。
**Consumes:** HalAudioFocus、AudioControlWrapperAidl、SparseArray 实现及本地 focus tests；前批 10.1/10.6。
**Produces:** zone/usage 两层请求表、请求/放弃/通知/reset 行为，恢复边界与连续入口。

- [x] 阅读请求结果、重复请求、abandon/remove、death reset、测试与 SparseArray keyAt/remove 行为，不改产品源码。
- [x] 先检查旧文，再重写 10.7，保留必要时序/状态图和精确片段；不虚构重试队列或透明恢复。
- [x] 更新导航，仅将实际完成的六篇加入主线；保留其他旧文待核对提示。
- [x] 执行本批全量 `python3 /tmp/qc-focus-doc-check.py --navigation`；运行 `/tmp/qc-aaos-doc-check.py --navigation`、`/tmp/qc-native-doc-check.py --navigation`、`/tmp/qc-hal-pal-doc-check.py SA8295_AAOS_QNX_Audio_KnowledgeBase`；各自零失败。
- [x] 检查全库带锚点入链、`git diff --check`、所有暂存 Markdown 编码/围栏；作者自审和限制已记录，指定文件提交 `bbdbd82`（14 文件）。继续 `docs/superpowers/plans/2026-10-30-pal-agm-session-bridge.md`。

## Execution Record

- 起点 `c403ab1`：AAOS Routing/AudioControl 六篇已提交。
- Pre-flight：Task 1 的 zone 模型供 Task 2 分发使用；Task 2 的焦点语义供 Task 3 HAL 请求解释使用。保持原路径，不重命名。
- Ruling：按用户授权连续执行，不重新做已批准总体设计的确认；检查器和计划只服务维护，不加入读者课程。
- 本批不覆盖普通 Android MediaFocusControl 栈全流程、全部 CarVolume 优先级、蓝牙、PAL/AGM/GSL 或全部配置专题；后续必须继续，不把这些范围默认为已完成。
- 旧文检查：6 篇、96 个链接、44 个源码块、24 个 Mermaid 块、119 个失败项；保存在 `/tmp/qc-focus-before.log`，不将失败项数称作独立语义错误数。
- Task 1 正文已完成未提交：`python3 /tmp/qc-focus-doc-check.py --task 1` → 2 篇、11 链接、7 源码摘录、4 Mermaid 块、0 带锚点入链、0 失败；`git diff --check` 通过。
- Task 1 作者自审：Zone 不拥有 ZoneConfig 或焦点处理器；活动用途来自播放属性而非地址反查；getter 非统一深快照；Context 完整性当前仅检查集合数量，未知名称映射 INVALID，合法性还受后续 getUsagesForContext 检查。保留代码实际边界，不将校验意图写成所有坏配置都早期拒绝。
- Ruling：新增解析器细节（V1 自动补四个 Context、未分配麦克风、未知标签 skip）放 9.18，不向总览复制整套算法；其代价是读者需要通过链接进入细节。
- Task 2：旧三篇检查 59 失败项；重写后 `--task 2` → 3 篇、14 链接、7 摘录、6 Mermaid 块、0 失败。追加逐单元匹配本地 13×13 矩阵，169 单元全部相符。
- Ruling：原 Review Focus 把 bundle zone 预称为特权覆盖不严谨；本地分发分支仅校验 zone 有效性。公开 service 查询权限与 Framework 焦点入口另行讲清，不补造额外 zone 权限分支；误判代价是误导安全边界，故正文明确范围。
- 作者自审（非独立审阅）：同 clientId 替换按 Context 而非严格 usage；DELAYED 返回前可能已改变其他条目；先尝试 delayed 再恢复 losers；普通恢复与 delayed 的 GAIN 分发失败处理不对称；去除图中把恢复通知画给 abandon 发起者的歧义。
- Task 3：HalAudioFocus 的 int usage 两层 SparseArray、只存 GRANTED、无记录 callback 不转发、abandon 先删除且失败不回填、重连接口后才回调 reset 均已对照。
- reset 限制：本地循环固定 usageCount，而 removeReturnOld 标记删除、keyAt/gc 压缩移动索引；三项静态路径可能跨过未处理 key，旧计数可超出有效索引。读到本地两项 reset 测试，但没有执行测试，不把测试存在称为通过。产品源码未改，不声称修复或实机复现。
- 关联更正：9.1 对 Context 完整性改为实际数量检查并链接 9.18；补入本批变更范围并由 AAOS 回归检查。
- 提交前复验：focus `--navigation` → 11 文件、192 链接、18 原文摘录、12 Mermaid 块、169 矩阵单元、0 带锚点入链、0 失败；AAOS → 10 文件、177 链接、16 摘录、15 图、0 失败；Native → 11 文件、181 链接、16 摘录、20 图、0 失败；HAL-PAL → 7 文件、9 摘录、0 失败。`git diff --check` 通过。
- 限制：没有独立 reviewer、Mermaid 渲染、产品构建、AndroidJUnit、Binder death 或设备运行验证。普通 Framework 焦点完整链、其他原文及整体目标继续待完成。
