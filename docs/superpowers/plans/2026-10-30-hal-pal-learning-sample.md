# HAL → PAL Learning Sample Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 将三篇 HAL/PAL 文章重构为本地 Android 12L + SA8295 AudioReach 的分层学习样板，具备源码调用图、关键实现解释与准确的边界说明。

**Architecture:** 保留原路径，重写错误密集的正文，同步必要导航。先读实现再形成文章；代码摘录、调用图与解释相互校验。维护材料不进入课程目录。

**Tech Stack:** Markdown、Mermaid、本地 Java/C/C++/HIDL 源码；临时 Python 检查仅用于格式、引用及已知错误回归，不新增知识库工具系统。

**Spec:** `docs/superpowers/specs/2026-10-30-audioreach-knowledge-base-design.md`

## Global Constraints

- 主线为本地 Android 12L / API 32、SA8295、QNX，实际使用 AudioReach；只研究 AudioReach，不展开 Elite。
- 保留并加强必要的源码调用流程图、时序图、状态图、关键源码片段及解释。
- 源码只读。知识库在 `docs/audioreach-hal-pal` 分支原地编辑，不创建新 worktree、不合并或推送。
- 不编造接口、路径、算法、配置、延迟数字或运行结果。
- 不建设基线台账、编译教程、审计系统；源码位置在文末精简呈现。
- 用户已授权自审后继续；执行方法为当前会话自行实现及复核，不另行启动代理。

## Review Focus

1. HIDL 已有 FMQ 与 Audio Patch：不能再次把它们描述成 AIDL 独有能力。
2. 声明版本、适配代码版本、实际服务版本不同：用明确范围描述，不从 Android 版本推断产品已运行哪个接口。
3. Stream::create 与 Session::makeSession 不是同一工厂；NON_TUNNEL 必须映射 SessionAgm；引擎不当作 Session 子类。
4. HAL write 失败可能由 onWriteError 转成字节数返回：不得把返回成功写成 DSP 已播放。
5. 录音、压缩、Voice、SoundTrigger 不能照搬普通 PCM 播放的启停顺序和设备映射。

## Source Roots

- Android: `/home/ylz/code/qc/Android/lagvm/LINUX/android`
- PAL: Android 根下 `vendor/qcom/opensource/pal`
- AR HAL: Android 根下 `vendor/qcom/opensource/audio-hal-ar/primary-hal/hal-pal`
- QNX 仅在本批跨域边界核对时引用，不扩写 QNX 内部架构。

## Task 1: 核对源码并重写 Audio HAL 入门

**Files:**
- Modify: `08_HAL_Layer/08_8.1_Audio_HAL双轨架构.md`

**Interfaces:**
- Consumes: libaudiohal HIDL 客户端、HIDL common adapter、AR HAL 的本地实现。
- Produces: HIDL / C HAL / AR HAL / PAL 边界解释，供 15.10 回链。

- [x] 阅读 `frameworks/av/media/libaudiohal/impl/DevicesFactoryHalHidl.cpp`、`StreamHalHidl.cpp` 的设备打开与 FMQ 写入路径。
- [x] 阅读 `hardware/interfaces/audio/core/all-versions/default/{DevicesFactory,Device,StreamOut}.cpp` 相关完整函数，并核对 `hardware/interfaces/audio/6.0/{IDevice,IStreamOut}.hal`。
- [x] 阅读 AR `AudioDevice.cpp` 的 `adev_open`、`adev_open_output_stream` 与 C 回调绑定。
- [x] 修改前记录旧文中“write/read回调 vs FMQ”“隐式路由”等反例；这是本批文档的失败回归基线，不是音频运行测试。
- [x] 完整重写文章：职责边界、设备打开时序、FMQ 数据图、WriteThread 调用代码、Patch、输入和 MMAP/压缩边界、AudioControl 区别、源码位置。
- [x] 检查新增 HIDL 图与代码逐项对应；文章不再包含旧的错误对比表及 AIDL 推荐路径主线。

## Task 2: 重建 PAL 工厂映射

**Files:**
- Modify: `15_PAL_Architecture/15_15.99_附录_Stream-Session-Device映射关系.md`

**Interfaces:**
- Consumes: `PalDefs.h`、`Stream.cpp`、`Session.cpp`、`Device.cpp` 和相关 Stream 构造/生命周期。
- Produces: Stream / Session / Device 的独立映射及适用条件，供 15.10 引用。

- [x] 完整阅读 `Stream::create`、`Session::makeSession`、`Device::getInstance/getObject` 与枚举。
- [x] 核对 StreamPCM、StreamNonTunnel、StreamCompress、StreamInCall、StreamSoundTrigger、StreamACD 的实际构造路径；注意条件编译、直接 Session 创建或引擎持有关系。
- [x] 修改前检查旧文 `PAL_STREAM_NON_TUNNEL → SessionAlsaPcm`、重复 Session 列、不存在的流枚举及静态优先级推论。
- [x] 完整重写：三类对象职责、Stream 分派表、四分支 Session 工厂代码、典型对象创建图、设备工厂与复用、Non-Tunnel 定义边界、源码位置。
- [x] 用源码重新核对表格所有枚举/类名；删除未经实现证明的 app_type、固定设备、延迟和优先级映射。

## Task 3: 重写 HAL-PAL 适配与生命周期

**Files:**
- Modify: `15_PAL_Architecture/15_15.10_HAL-PAL适配层架构.md`

**Interfaces:**
- Consumes: Task 1 的 HAL 边界、Task 2 的对象映射。
- Produces: 普通 PCM 播放深入样板及录音/特殊流的差异说明。

- [x] 阅读 AR `AudioStream.cpp` 的 C 回调、构造函数、Open/configurePalOutputStream/write/onWriteError/Standby 及关闭回调/析构；查清类型和设备映射。
- [x] 阅读输入流对应函数，按实际实现描述打开、启动、读取和错误处理，不套用输出路径。
- [x] 阅读 `Pal.cpp` 的 open/start/write/read/stop/close 与 StreamPCM 对应生命周期；确认何时创建对象、何时打开底层资源。
- [x] 重写职责图、open/write 分离的时序图、关键代码解释、错误流程图、standby/close、录音差异、平台边界及调试观察。
- [x] 核验图中每条函数调用与关键状态条件；明确 write 正返回不能证明已发声。

## Task 4: 同步导航、自审与提交

**Files:**
- Modify: `README.md`
- Modify: `08_HAL_Layer/README.md`
- Modify: `15_PAL_Architecture/README.md`
- Modify: `CLASS_INDEX.md`
- Update: 本计划的执行记录。

**Interfaces:**
- Consumes: Tasks 1–3 的已完成文章。
- Produces: 可直接阅读的入口与准确的批次状态，不替未修改文章背书。

- [x] 总 README 改为本地主线与清晰的分层学习入口，删除失效质量数字；标明原章节仍在逐步整理。
- [x] 更新 08/15 章文章名称、描述与优先阅读入口，避免改好的正文配旧的误导性导读。
- [x] CLASS_INDEX 为本批核心类添加直达文件链接，不把全库索引改动扩展为另一个大任务。
- [x] 运行以下格式/引用检查（只扫描本批修改，不把其他旧文章的既有问题混作本批结果）：

```bash
git diff --check
python3 /tmp/qc-hal-pal-doc-check.py /home/ylz/code/qc/SA8295_AAOS_QNX_Audio_KnowledgeBase
```

Expected: 无编码替代字符、代码围栏未闭合、Markdown 本地链接失效；已知错误回归不再出现。临时脚本需报告检查范围，不能宣称验证全部技术结论或 Mermaid 渲染。

- [x] 逐篇重新读正文并与本批源码记录核对，重点检查 Review Focus 五项；必要修订同步图、表、代码与文字。
- [x] 对声明为原文摘录的代码块检查本地源码匹配；简化代码明确标为伪代码。
- [x] 检查 Mermaid 渲染工具并记录限制：未发现 `mmdc`，本批未进行 Mermaid 图形渲染，不安装依赖或声称渲染成功。
- [ ] 指定本批文件提交，检查分支状态；报告完成范围、未验证内容及后续批次，不声称整库已完成。

## Execution Record

- 2026-10-30：设计自审修正三项风险；用户授权自审后继续。
- Ruling: 仅文档重构，不引入产品代码或自动校验工具项目；验证采用旧错误反例 → 正文修正 → 格式与技术复核。它不能代替实机测试。
- Ruling: 原目录新建独立分支 `docs/audioreach-hal-pal`；保持用户能直接阅读改动及本地源码路径稳定，不创建未经请求的新 worktree。
- Pre-flight: Task 1 HAL 边界 → Task 3 调用链；Task 2 工厂映射 → Task 3 对象解释；Task 4 必须在前三项后更新导航，暂无冲突。
- 执行方式沿用用户授权的自行 review：作者另作收尾自审，未启动独立 reviewer；其独立性弱于新上下文审阅，不作独立审阅通过声明。
- Ruling: 任务进度直接保留在本计划，临时检查放 `/tmp`，不增加工作区台账和工具项目；代价是临时检查脚本不是仓库交付物，后续需重新建立检查环境。
- Ruling: 计划中的“Close”按实际关闭 callback、AudioDevice 关闭方法及析构阅读，不假定存在同名 StreamOutPrimary::Close 方法；否则会重新制造虚构 API。
- 修改前检查：7 个阅读文件共报 15 项问题，覆盖旧错误对比、Non-Tunnel 映射、虚构枚举、失效源码链接和陈旧质量声明。首次误用了不存在的 `python` 命令，已改为计划中的 `python3`。
- 检查脚本复核：修正两类检查器误报——把“不是 SessionAlsaPcm”当成错误映射；把 PAL 根下的 `device/src/` 当成 Android 根路径。未为通过检查改掉正确技术说明。
- Task 1: complete — 8.1 重写为 200 行，4 个 Mermaid 代码块；HIDL 6.0 的 FMQ/Patch 与 C callback 边界已回查。
- Task 2: complete — 15.99 重写为 230 行，2 个 Mermaid 代码块；Stream 分派、四分支 Session 工厂、引擎持有关系与设备复用已回查。
- Task 3: complete — 15.10 重写为 377 行，4 个 Mermaid 代码块；普通 PCM、flags 判断顺序、BUS 映射、错误回退与输入差异已回查。
- 收尾自审：修正 PalDefs 文件定位为 PAL 根下 `PalDefs.h`；再次逐篇读取三篇正文，核对 Review Focus 五项。未发现尚未处理的本批重大技术问题；此结论不覆盖未改写章节或实机行为。
- 检查结果：`git diff --check` 通过；临时检查器扫描 7 个阅读文件，编码/围栏/本地链接与简单标题锚点/已知错误回归 0 失败；9 个 C++ 摘录按空白规范化后匹配本地源码，PAL 枚举/flags 与限定源码路径检查通过。
- 链接兼容性：保留三篇原文件名；全库定向检索未发现指向这三篇的旧标题锚点链接。本轮不是全库链接校验。
- 未执行：Mermaid 图形渲染、产品构建、实机音频/延迟测试、最终运行服务验证。未修正文档中的其他章节；已在导航注明其整理状态。
- 后续顺序仍按设计：Framework → AudioFlinger/Policy，再展开 AAOS 与更深的平台层；本批不顺带生成后续空文章。
