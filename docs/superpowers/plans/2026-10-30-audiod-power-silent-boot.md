# audiod / AutoPower / Silent Boot 实施计划

执行方式：executing-plans，父会话唯一 writer，无需批次审批；整体完成后按 completion-review-gate 做独立整库 review。主线 Android12L/API32 + SA8295 + QNX AudioReach；只改知识库，分支 docs/audioreach-hal-pal；前批提交 c865473。

## 审查重点

1. 声卡发现过滤（msm/apq/sa 前缀）、共享 prev_state/bootup_complete、首次 ONLINE 与重启分支；不写成完整 per-card 状态机。
2. HIDL transport/result 分层、HAL 死亡重连后状态需重发；发 SND_CARD_STATUS ≠ AR HAL/PAL 已消费/恢复。
3. hostless 条件路由/PCM 设备表、NULL/未 ready 重试、错误回滚、enable 标志与硬件分离；wakelock 只在 non-VHAL power worker。
4. AutoPower 仅订阅 AP_POWER_STATE_REPORT（EVENTS_FROM_ANDROID），onPropertySet 仅 DEEP_SLEEP_EXIT/SHUTDOWN_POSTPONE 两动作；订阅失败缓存的 mVehicle 指针与重试。
5. Silent Boot 条件编译来源（非本平台配置）、epoll 边缘触发无首次读、单字节解析、锁内 pthread_create 失败出口；plugin 消息 ≠ mute/DSP 效果。

## Tasks

- [x] Task1 16.2：audiod 进程/rc、声卡 FD 与 poll 主循环、重启属性分支、notifyAudioDevice HIDL、hostless helper（tinyalsa 分支为主，LINUX_ENABLED 仅提差异）。
- [x] Task2 16.3：AutoPower 构造即起线程、initVehicle 重试、死亡重连、onPropertySet 与 _all 变体、VHAL_EXT 对 power 监控的替代关系。
- [x] Task3 16.4：silent_mode_monitor_thread 全链（open→dlopen plugin→epoll→fetch→send_msg）、宏展开为 no-op 的默认路径、与 16.2 启动点关系。
- [x] Task4 导航：16 README/CLASS_INDEX/根 README、16.1 概述若引用旧页需同步；链接与锚点。

## 验证

- 检查器 /tmp/qc-audiod-doc-check.py（链接解析、源码摘录行匹配、Mermaid 代码块闭合、U+FFFD）；基线日志 /tmp/qc-audiod-before.log。
- 旧 10 文件回归 + diff --check + /tmp/qc-staged-doc-gate.py 后本地提交。

## 结果记录

- 基线 /tmp/qc-audiod-before.log：37 失败（旧 16.2/16.3/16.4 摘录不匹配、旧路径引用 device/google/trout 等）。
- 重写后 article 检查 0 失败；navigation（含 16.1、README、CLASS_INDEX）0 失败。
- 检查器纠错实例：DEEP_SLEEP_ENTRY 分支漏抄、silent_mode_epoll_fd 误写、epoll/fetch 摘录 `...` 省略不匹配，均已按源修正。
- 16.1 联动修正：audiod/AutoPower/SilentBoot/audio-chime 所属域 QNX→Android(LA VM)；删除虚构 auto-audiod.cfg、slog2info -l auto-audiod 行；SSR 时序改由 16.17 audio_service 主导；gsl_vm_msg.h 链接改为反引号路径。
- 待后续批次：16.5 audio-chime、16.1 中 "QNX Audio Manager" 等剩余未核条目。
