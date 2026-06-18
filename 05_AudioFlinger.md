# 第五篇：AudioFlinger

> [← 上一篇：Native Framework](04_Native_Framework_Layer.md) | [返回导航](README.md) | [下一篇：Audio Policy Engine →](06_Audio_Policy_Engine.md)

---

## 5.1 AudioFlinger — 音频数据面引擎

### 模块职责
AudioFlinger是Android音频系统的数据面引擎，运行在mediaserver进程中，负责：
- 混音：将多个AudioTrack的PCM数据混合到一个输出流
- 采集：从HAL读取输入数据分发给多个AudioRecord
- 效果处理：管理EffectChain对音频数据进行效果处理
- 路由：通过PatchPanel管理音频端口之间的连接
- 音量：应用音量和ducking衰减

### 所属层级
Native Service → `frameworks/av/services/audioflinger/`

### 初始化入口
AudioFlinger在`main_mediaserver.cpp`中注册为Binder服务：
```
main_mediaserver → new AudioFlinger() → IServiceManager.addService("media.audio_flinger", audioFlinger)
```

---

## 5.2 Thread体系 — AudioFlinger的核心执行单元

### Thread继承图谱

```mermaid
classDiagram
    ThreadBase <|-- PlaybackThread
    PlaybackThread <|-- MixerThread
    PlaybackThread <|-- DirectOutputThread
    PlaybackThread <|-- OffloadThread
    PlaybackThread <|-- SpatializerThread
    PlaybackThread <|-- BitPerfectThread
    PlaybackThread <|-- DuplicatingThread
    ThreadBase <|-- RecordThread
    ThreadBase <|-- MmapThread
    MmapThread <|-- MmapPlaybackThread
    MmapThread <|-- MmapCaptureThread
    PlaybackThread *-- Track
    Track <|-- TrackHandle
    RecordThread *-- Record
    Record <|-- RecordHandle
```

### Thread类型决策逻辑（[`openOutput_l()`](frameworks/av/services/audioflinger/AudioFlinger.cpp:2982)）

| flags条件 | Thread类型 | 场景 | 特点 |
|-----------|-----------|------|------|
| AUDIO_OUTPUT_FLAG_MMAP_NOIRQ | MmapPlaybackThread | AAudio MMAP模式 | 共享内存直传，不经过AF混音 |
| AUDIO_OUTPUT_FLAG_BIT_PERFECT | BitPerfectThread | AAOS位完美输出 | 不修改PCM数据格式/位深 |
| AUDIO_OUTPUT_FLAG_SPATIALIZER | SpatializerThread | 空间音频 | 空间化处理后再输出 |
| AUDIO_OUTPUT_FLAG_COMPRESS_OFFLOAD | OffloadThread | 码流卸载 | MP3/AAC等压缩格式直传HAL |
| AUDIO_OUTPUT_FLAG_DIRECT | DirectOutputThread | 直通输出 | 非混音，单Track直接输出 |
| 默认(无特殊flags) | MixerThread | 标准混音 | 多Track混音后输出 |

**为什么需要这么多Thread类型？**

| 问题 | 解决 | Thread |
|------|------|--------|
| 多App同时播放需要混合 | 混音多个Track到一个输出 | MixerThread |
| 延迟敏感场景（专业音频） | 绕过AF混音直传DSP | MmapPlaybackThread |
| 压缩码流无需解码混音 | 直接传压缩数据到DSP解码 | OffloadThread |
| 多声道/高采样率输出 | 不混音保留原始格式 | DirectOutputThread |
| 车载位完美音频要求 | 不做任何格式转换 | BitPerfectThread(AAOS14新增) |
| 空间音频处理 | 专门线程处理空间化 | SpatializerThread |
| 多输出同步（如HDMI+扬声器） | 复制到多个输出 | DuplicatingThread |

---

## 5.3 PlaybackThread核心循环

### threadLoop — AudioFlinger的心跳

[`PlaybackThread::threadLoop()`](frameworks/av/services/audioflinger/Threads.cpp:3853)是AudioFlinger最核心的循环：

```mermaid
graph TB
    Start["threadLoop() 开始"] --> Sleep["sleepWait_l() 等待唤醒"]
    Sleep --> Check["检查活跃Track列表"]
    Check -->|"有活跃Track"| Prepare["prepareTracks_l()"]
    Check -->|"无活跃Track"| Sleep
    Prepare -->|"Track就绪"| Mix["threadLoop_mix() 混音"]
    Prepare -->|"Track未就绪"| Sleep
    Mix --> Effect["EffectChain.process_l()"]
    Effect --> Write["threadLoop_write() → HAL"]
    Write --> Update["更新cblk.server位置"]
    Update --> Sleep
```

### prepareTracks_l() — 混音准备

关键逻辑：
1. 遍历所有活跃Track，检查cblk是否有新数据（`framesReady = cblk.user - cblk.server`）
2. 设置Track的volume（ducking衰减、焦点音量等）
3. 标记Track为READY或INVALID
4. 返回就绪Track数量，0则不执行mix+write

### threadLoop详细执行流程

[`Threads.cpp:3853-4200`](frameworks/av/services/audioflinger/Threads.cpp:3853)

```mermaid
flowchart TD
    A["threadLoop() 入口"] --> B["cacheParameters_l()<br/>缓存采样率/frameCount等"]
    B --> C["acquireWakeLock()"]
    C --> D["checkSilentMode_l()"]
    D --> E{"主循环 for(;;)"}
    
    E --> F["processConfigEvents_l()<br/>处理配置变更事件"]
    F --> G["collectTimestamps_l()"]
    G --> H{"有SignalPending?"}
    H -->|是| I["清除信号标志"]
    H -->|否, 等待异步回调| J["computeWaitTimeNs_l()"]
    J --> K["mWaitWorkCV.waitRelative()"]
    K --> L{"mActiveTracks空 且 超时?"}
    L -->|是| M["threadLoop_standby()"]
    M --> N["mWaitWorkCV.wait() 长休眠"]
    N --> E
    
    L -->|否| O["prepareTracks_l()"]
    O --> P{"mMixerStatus?"}
    P -->|MIXER_TRACKS_READY| Q["threadLoop_mix()"]
    P -->|MIXER_IDLE| R["threadLoop_sleepTime()"]
    Q --> S["EffectChain.process_l()"]
    R --> T{"mSleepTimeUs==0?"}
    T -->|是| U["填充0到sink buffer<br/>记录underrun"]
    T -->|否| V["继续sleep"]
    S --> W["mono_blend + balance处理"]
    U --> W
    W --> X["threadLoop_write() → HAL"]
    X --> Y["更新mBytesWritten/mFramesWritten"]
    Y --> E
```

**关键步骤详解**：

#### 1. prepareTracks_l() — 决定哪些Track参与混音

遍历mActiveTracks列表，对每个Track执行：

```cpp
// 检查共享内存中有多少帧数据可读
framesReady = track->framesReady();
if (framesReady == 0) {
    // 无数据：标记为UNDERRUN或IDLE
    if (track->isStopping()) {
        // Offload Track停止中且无数据 → 完成
        track->mState = TrackBase::STOPPED;
    }
} else {
    // 有数据：设置volume和format
    // 检查ducking/焦点衰减 → 设置实际volume
    // 标记为READY
}
```

> **volume计算顺序**：master volume × stream volume × duck attenuation × balance → 最终每个Track的左右声道gain

#### 2. threadLoop_mix() — 执行混音

```cpp
// AudioMixer::process()
// 对每个READY的Track:
//   1. 从共享内存读取PCM数据(通过AudioTrackServerProxy)
//   2. 重采样到输出采样率(如需要)
//   3. 应用音量gain
//   4. 混音到mMixerBuffer(float格式)
// 输出: mMixerBuffer → memcpy到mSinkBuffer或mEffectBuffer
```

#### 3. threadLoop_write() — 写入HAL

```cpp
// 根据Thread类型:
// MixerThread: mOutputSink->write(mSinkBuffer, frames)
// DirectOutputThread: mOutputSink->write(trackBuffer, frames)
// OffloadThread: mOutputSink->write(compressedBuffer, frames)
```

#### 4. Underrun检测与计数

[`Threads.cpp:4130-4138`](frameworks/av/services/audioflinger/Threads.cpp:4130)

```cpp
// 当mSleepTimeUs==0且无数据可mix时，填充0到sink buffer
for (const auto& track : activeTracks) {
    if (track->mFillingUpStatus == Track::FS_ACTIVE
            && !track->isStopped() && !track->isPaused()) {
        // 记录underrun帧数
        track->mAudioTrackServerProxy->tallyUnderrunFrames(mNormalFrameCount);
    }
}
```

### AudioFlinger::createTrack()完整流程

[`AudioFlinger.cpp:1105-1310`](frameworks/av/services/audioflinger/AudioFlinger.cpp:1105)

```mermaid
sequenceDiagram
    participant Client as AudioTrack(Native)
    participant AF as AudioFlinger
    participant APM as AudioPolicyManager
    participant Thread as PlaybackThread
    
    Client->>AF: createTrack(CreateTrackRequest)
    AF->>AF: UID/PID校验(防冒用)
    AF->>AF: sessionId分配
    AF->>APM: getOutputForAttr(attr, config, flags)
    APM->>APM: 根据usage选择输出设备<br/>选择/复用PlaybackThread<br/>FAST flag兼容性检查
    APM-->>AF: outputId, streamType, portId, selectedDeviceId
    AF->>AF: checkPlaybackThread_l(outputId)
    AF->>AF: registerPid(clientPid) → Client对象
    AF->>Thread: createTrack_l(client, attr, ...)
    Thread->>Thread: 分配共享内存(cblk+buffer)<br/>创建Track对象<br/>检查FAST flag可行性<br/>调整frameCount/notificationFrames
    Thread-->>AF: sp<Track>
    AF->>AF: updateSecondaryOutputsForTrack_l()<br/>移动EffectChain到新Thread
    AF->>AF: 处理mPendingSyncEvents
    AF-->>Client: TrackHandle(IAudioTrack) + cblk + buffers
```

**关键决策点**：

1. **getOutputForAttr**：AudioPolicyManager决定路由到哪个PlaybackThread。同类型的AudioAttributes可能路由到同一个MixerThread（共享混音），也可能路由到独立的DirectOutputThread。

2. **FAST flag处理**：即使Client请求FAST，AudioFlinger也需要检查：
   - 当前Thread是否支持FastMixer
   - Fast Track数量是否已满
   - Track格式是否兼容
   不满足则降级为Normal Track

3. **EffectChain迁移**：如果同sessionId的EffectChain在其他Thread上，createTrack会将其迁移到新Thread，确保Effect与Track在同一个Thread处理。

4. **SecondaryOutputs**：某些场景（如HDMI+扬声器同步输出），一个Track需要同时写入多个输出。通过updateSecondaryOutputsForTrack_l()设置。

### AudioFlinger::createRecord() — 录音端创建

[`AudioFlinger.cpp:2390-2575`](frameworks/av/services/audioflinger/AudioFlinger.cpp:2390)

与createTrack对称，但有独特机制：

**FAST flag降级重试**（createRecord独有）：
```cpp
for (;;) {
    lStatus = AudioSystem::getInputForAttr(..., output.flags, ...);
    recordTrack = thread->createRecordTrack_l(...);
    if (lStatus == BAD_TYPE) {
        continue;  // FAST不支持 → 去掉FAST重新请求
    }
    break;
}
```

### threadLoop_mix() — AudioMixer混音

AudioMixer是MixerThread的核心组件：
- 支持多Track同时混音（最多32个Track）
- 支持float/16bit/24bit格式输入，输出为float
- 每个Track有独立的volume、format、channel配置
- 混音结果写入mSinkBuffer

---

## 5.4 FastMixer — 低延迟混音路径

### 模块职责
FastMixer是MixerThread内部的低延迟子线程，在SCHED_FIFO调度策略下运行，减少混音延迟。

### FastMixer架构

```mermaid
classDiagram
    FastThread <|-- FastMixer
    FastMixer --> AudioMixer
    FastMixer --> NBAIO_Sink
    FastMixer --> FastMixerStateQueue
```

[`FastMixer`](frameworks/av/services/audioflinger/FastMixer.h:34)继承FastThread，核心成员：
- `mMixer`: AudioMixer实例
- `mOutputSink`: NBAIO_Sink写入HAL
- `mSQ`: FastMixerStateQueue，与NormalMixer通信
- `mSinkBuffer`: 混音输出buffer
- `mMixerBuffer`: AudioMixer内部buffer（float格式）

### FastMixer vs NormalMixer

| 维度 | FastMixer | NormalMixer |
|------|-----------|-------------|
| 调度策略 | SCHED_FIFO（实时） | SCHED_OTHER（普通） |
| 优先级 | 高 | 低 |
| 锁 | 无锁（atomic操作） | Mutex锁 |
| 延迟 | <10ms | 20-50ms |
| Track限制 | Fast Track（特殊条件） | 所有Track |
| 效果 | 不支持 | 支持 |

**Fast Track条件**：Track必须满足特定格式/采样率/通道要求，才能在FastMixer路径上混音。不满足条件的Track走NormalMixer路径。

---

## 5.5 Buffer管理与共享内存

### Buffer分配策略

| 场景 | App buffer大小 | AF buffer大小 | 说明 |
|------|---------------|---------------|------|
| MODE_STREAM | 由App指定(getMinBufferSize) | AF内部buffer | 双buffer结构 |
| MODE_STATIC | App一次性提供 | 无额外AF buffer | App buffer直接映射 |
| FastMixer | 较小(2-3 periods) | 小buffer | 降低延迟 |
| NormalMixer | 较大(4-8 periods) | 大buffer | 增加容错 |
| Offload | 压缩码流buffer | 无混音buffer | 直传HAL |

### Underrun处理

当App写入速度不够快导致PlaybackThread无数据时：
1. MixerThread：填充0（静音）→ 继续混音输出 → 记录underrun计数
2. FastMixer：标记underrun → 尽快恢复
3. DirectOutputThread：暂停输出等待数据
4. 通知App：通过`IAudioTrack`回调发送UNDERRUN事件

---

## 5.6 PatchPanel — 音频路由管理

### 模块职责
[`PatchPanel`](frameworks/av/services/audioflinger/PatchPanel.h:24)管理AudioFlinger内部的音频端口连接，实现软件路由。

### 核心功能

| 方法 | 说明 |
|------|------|
| `createAudioPatch()` | 创建音频路由（源端口→目标端口） |
| `releaseAudioPatch()` | 释放路由 |
| `listAudioPorts()` | 列出所有音频端口 |
| `getAudioPort()` | 获取端口属性 |
| `getDownstreamSoftwarePatches()` | 获取下游软件路由 |

### SoftwarePatch — 软件路由

当音频需要从一个HAL模块路由到另一个HAL模块时（如FM广播→扬声器），AudioFlinger创建SoftwarePatch：
- 创建一个RecordThread从源设备读取
- 创建一个PlaybackThread向目标设备写入
- 两者通过内部Track连接，形成"软件桥"

```mermaid
graph LR
    FM_HAL["FM HAL"] -->|"read"| RecordThread -->|"内部Track"| PlaybackThread -->|"write"| Speaker_HAL["Speaker HAL"]
```

---

## 5.7 Track/Record — 音频流端点

### Track（PlaybackThread内部对象）

Track是PlaybackThread内部的音频流端点，代表一个App的AudioTrack在AudioFlinger侧的镜像：

```mermaid
classDiagram
    PlaybackThread *-- Track
    Track <|-- TrackHandle
    Track --> audio_track_cblk_t
    Track --> AudioTrackClientProxy
    TrackHandle --> IAudioTrack_Bn
```

- **Track**: 管理共享内存读取、音量设置、状态控制
- **TrackHandle**: Binder服务端，实现IAudioTrack接口，App通过此接口控制
- **AudioTrackClientProxy**: 从共享内存读取App写入的PCM数据

### Record（RecordThread内部对象）

与Track对称，是AudioRecord在AF侧的镜像：
- 从HAL读取 → 写入共享内存 → App从共享内存读取

---

## 5.8 Thread继承体系与匹配规则

### 5.8.1 AudioFlinger线程类型完整继承体系

```mermaid
classDiagram
    class ThreadBase {
        +mType : type_t
        +mIoHandle : audio_io_handle_t
        +mStandby : bool
        +mSampleRate : uint32_t
        +mFormat : audio_format_t
        +mChannelMask : audio_channel_mask_t
        +threadLoop() : bool
        +processConfigEvents()
        +readlRead()
    }
    class PlaybackThread {
        +mTracks : SortedVector~Track~
        +mActiveTracks : SortedVector~Track~
        +mSuspendedTracks : SortedVector~Track~
        +mMixBuffer : void*
        +mMasterVolume : float
        +prepareTracks_l() : int
        +threadLoop_mix()
        +threadLoop_write()
        +createTrack_l()
    }
    class MixerThread {
        +mFastMixer : FastMixer*
        +mNormalMixer : AudioMixer*
        +mFastTrackNb : int
        +onFirstRef()
    }
    class DirectOutputThread {
        +mSinkBuffer : void*
        +mFormat : audio_format_t
        +selectTrack_l()
    }
    class OffloadThread {
        +mHwPaused : bool
        +mPaused : bool
        +mDraining : bool
        +processDrain_l()
        +handleStreamEnd()
    }
    class FastMixer {
        +mState : state_t
        +mCommandQueue : FIFO
        +mNormalSink : NBAIO_Sink*
        +onFirstRef()
    }
    class RecordThread {
        +mActiveTracks : SortedVector~RecordTrack~
        +mInput : audio_input_handle_t
        +mRsmpInBuffer : void*
        +threadLoop_read()
        +createRecordTrack_l()
    }
    class MmapPlaybackThread {
        +mHalStream : sp~IMmapStream~
        +createMmapBuffer()
    }
    class MmapCaptureThread {
        +mHalStream : sp~IMmapStream~
        +createMmapBuffer()
    }
    class DuplicatingThread {
        +mOutputPaths : SortedVector~audio_io_handle_t~
        +addOutputPath()
        +removeOutputPath()
    }
    class SpatializerThread {
        +mSpatializer : sp~Spatializer~
        +processEffect_l()
    }
    class BitPerfectThread {
        +mBitPerfect : bool
    }

    ThreadBase <|-- PlaybackThread
    ThreadBase <|-- RecordThread
    ThreadBase <|-- MmapPlaybackThread
    ThreadBase <|-- MmapCaptureThread
    PlaybackThread <|-- MixerThread
    PlaybackThread <|-- DirectOutputThread
    PlaybackThread <|-- OffloadThread
    PlaybackThread <|-- DuplicatingThread
    PlaybackThread <|-- SpatializerThread
    PlaybackThread <|-- BitPerfectThread
    MixerThread *-- FastMixer : mFastMixer
```

### 5.8.2 MixerThread双路径架构详解

```mermaid
flowchart TB
    subgraph "MixerThread内部双路径"
        TRACKS["mActiveTracks"] --> FASTCHECK{"Track有FLAG_FAST?"}
        FASTCHECK -->|"是"| FASTPATH["FastMixer路径<br/>SCHED_FIFO优先级"]
        FASTCHECK -->|"否"| NORMALPATH["NormalMixer路径<br/>SCHED_OTHER优先级"]

        subgraph "FastMixer路径(~10ms延迟)"
            FASTPATH --> FASTCMD["mCommandQueue写入<br/>FastMixer命令"]
            FASTCMD --> FASTMIX["FastMixer混音<br/>OBTAIN模式"]
            FASTMIX --> FASTSINK["mNormalSink<br/>NBAIO_Sink写入HAL"]
        end

        subgraph "NormalMixer路径(~40ms延迟)"
            NORMALPATH --> NMIX["AudioMixer混音<br/>SCHED_OTHER线程"]
            NMIX --> NWRITE["threadLoop_write()<br/>写入HAL mSinkBuffer"]
        end
    end
```

| 参数 | FastMixer | NormalMixer |
|------|-----------|------------|
| 线程优先级 | SCHED_FIFO | SCHED_OTHER |
| 缓冲区大小 | 小(~2-4ms) | 大(~20-40ms) |
| 混音方式 | 简单copy | AudioMixer多轨 |
| 适用Track | TRANSFER_CALLBACK/SYNC | TRANSFER_SHARED(write) |
| 延迟 | ~10ms | ~40ms |

> **FastMixer限制**: 最多支持`mFastTrackNb`(通常8个)Fast Track，超出则降级到NormalMixer。Fast Track不支持VolumeShaper和大部分Effect。

### 5.8.3 各Thread类型的适用场景

| Thread类型 | 适用场景 | 输出Flag | 特殊能力 | 延迟范围 |
|-----------|---------|----------|---------|---------|
| MixerThread | 通用混音 | NONE/FAST/DEEP_BUFFER | 双路径(Fast+Normal) | 10ms(Fast)/40ms(Normal) |
| DirectOutputThread | 独占输出(不支持混音) | DIRECT | 单Track独占一个输出 | ~10ms |
| OffloadThread | 码流Offload(MP3/AAC到DSP) | COMPRESS_OFFLOAD | DSP解码，CPU省电 | ~100ms |
| DuplicatingThread | 多设备同步输出 | NONE | 复制数据到多个下游Thread | 取最慢下游 |
| SpatializerThread | 空间音频(Spatial Audio) | SPATIALIZER | 头相关传输函数(HRTF) | ~20ms |
| BitPerfectThread | 位完美输出(无重采样) | BIT_PERFECT | 不做任何处理直通 | 取源延迟 |
| MmapPlaybackThread | MMAP低延迟播放 | MMAP_NOIRQ | 共享内存直通(无threadLoop) | <3ms |
| RecordThread | 录音输入 | NONE/FAST | 重采样+Effect处理 | 输入延迟 |
| MmapCaptureThread | MMAP低延迟录音 | MMAP_NOIRQ | 共享内存直通 | <3ms |

### 5.8.4 Thread与Track的匹配规则

```mermaid
flowchart TB
    ATTR["AudioTrack.Builder<br/>setPerformanceMode/Flag"] --> PM{"PerformanceMode?"}

    PM -->|"LOW_LATENCY"| FASTFLAG["FLAG_FAST(0x4)"]
    PM -->|"POWER_SAVING"| DEEPFLAG["FLAG_DEEP_BUFFER(0x8)"]
    PM -->|"NONE"| TRANSFER{"TransferMode?"}
    TRANSFER -->|"TRANSFER_CALLBACK/SYNC"| MAYBEFAST["可能获得FAST"]
    TRANSFER -->|"TRANSFER_SHARED"| NORMALTRACK["NormalMixer Track"]

    FASTFLAG --> FASTCHECK2{"AF.createTrack_l()<br/>FAST资格检查"}
    FASTCHECK2 -->|"通过"| FASTTRACK["FastMixer Track<br/>SCHED_FIFO"]
    FASTCHECK2 -->|"被拒(BAD_TYPE)"| FALLBACK2["降级到NormalMixer Track"]

    DEEPFLAG --> DEEPBUF["DeepBuffer Track<br/>更大缓冲区"]

    subgraph "Offload路径"
        OFFLOADCHECK{"isOffloadSupported()?"}
        OFFLOADCHECK -->|"格式支持"| OFFLOAD["OffloadThread Track<br/>DSP解码"]
        OFFLOADCHECK -->|"不支持"| DIRECTORMIX["Direct或Mixer路径"]
    end

    subgraph "Direct路径"
        DIRECTCHECK{"需要独占?"}
        DIRECTCHECK -->|"DIRECT Flag"| DIRECT["DirectOutputThread Track"]
        DIRECTCHECK -->|"否"| MIXER["MixerThread Track"]
    end
```

---

## 5.9 音乐播放全栈调用链

### 5.9.1 AudioTrack创建时序

```mermaid
sequenceDiagram
    participant App as App(应用)
    participant AT_J as AudioTrack(Java)
    participant JNI as JNI Bridge
    participant AT_N as AudioTrack(Native)
    participant AF as AudioFlinger
    participant APS as AudioPolicyService
    participant APM as AudioPolicyManager
    participant Eng as EngineBase
    participant PT as PlaybackThread
    participant HAL as StreamOutHal

    App->>AT_J: new AudioTrack.Builder()...build()
    Note over AT_J: 1. PerformanceMode→Flags映射<br/>2. Offload兼容性检查<br/>3. bufferSize计算
    AT_J->>JNI: native_setup(...)
    JNI->>AT_N: AudioTrack::set(...)
    Note over AT_N: 1. Transfer Type决策<br/>2. FAST flag资格检查
    AT_N->>AT_N: createTrack_l()
    AT_N->>AF: IAudioFlinger.createTrack() [Binder]
    
    AF->>AF: UID/PID校验
    AF->>APS: getOutputForAttr()
    APS->>APM: getOutputForAttr()
    APM->>Eng: getProductStrategyForAttributes()
    Eng-->>APM: strategy_media
    APM->>Eng: getOutputDevicesForAttributes()
    Eng-->>APM: {speaker} + AUDIO_OUTPUT_FLAG_FAST
    APM-->>APS: outputId, portId, selectedDeviceId
    APS-->>AF: 路由结果
    
    AF->>PT: createTrack_l() → 共享内存(cblk+buffer)+Track
    PT-->>AF: sp<Track> + cblk + buffers
    AF-->>AT_N: TrackHandle + cblk + buffers
    AT_N-->>AT_J: mNativeTrackInJavaObj
```

### 5.9.2 播放数据流时序

```mermaid
sequenceDiagram
    participant App, AT_N, Cblk, PT, Mixer, Effect, HAL

    App->>AT_N: start()
    AT_N->>PT: IAudioTrack.start() [Binder]
    PT->>PT: Track加入mActiveTracks

    loop 每个App写入周期
        App->>AT_N: write(pcmData)
        AT_N->>Cblk: obtainBuffer → memcpy → releaseBuffer
    end

    loop 每个混音周期(~20ms NormalMixer, ~10ms FastMixer)
        PT->>PT: prepareTracks_l()
        PT->>Mixer: AudioMixer.process() → 混音
        PT->>Effect: EffectChain.process_l()
        PT->>HAL: mOutputSink->write()
        PT->>Cblk: mFront += consumed [唤醒App]
    end
```

---

## 5.10 录音全栈调用链

从App层AudioRecord到HAL层的完整录音数据流路径。

### 5.10.1 AudioRecord创建时序

```mermaid
sequenceDiagram
    participant App, AR_J, AR_N, AF, APS, APM, RT, HAL
    App->>AR_J: new AudioRecord(AudioAttributes, AudioFormat, bufferSize)
    AR_J->>AR_N: native_setup() [JNI]
    AR_N->>AF: IAudioFlinger.openInput() [Binder]
    AF->>APM: openInput(module, config, source)
    APM->>APM: getInputForAttr() → 选择RecordThread类型
    APM-->>AF: inputHwDev + config
    AF->>RT: new RecordThread(inputHwDev)
    AF->>HAL: openInputStream(config)
    HAL-->>AF: streamIn + config
    AF-->>AR_N: IAudioRecord [Binder代理]
    AR_N->>AR_N: 创建audio_track_cblk_t共享内存
    AR_N-->>AR_J: INIT_SUCCESS
```

### 5.10.2 录音数据流时序

```mermaid
sequenceDiagram
    participant App, AR_N, Cblk, RT, Resampler, Effect, HAL, Mic
    App->>AR_N: startRecording()
    AR_N->>RT: IAudioRecord.start() [Binder]
    RT->>RT: RecordThread加入mActiveTracks

    loop 每个录音周期(~20ms)
        HAL->>Mic: 读取PCM数据
        Mic->>HAL: 中断/DMA传输
        HAL->>RT: streamIn.read(inputBuffer)
        RT->>Effect: EffectChain.process_l() [AEC/NS处理]
        RT->>Resampler: Resampler.process() [采样率转换]
        RT->>Cblk: memcpy到共享内存 + 更新mFront
        Cblk-->>AR_N: mFront更新 [唤醒App]
    end

    loop 每个App读取周期
        App->>AR_N: read(buffer, size)
        AR_N->>Cblk: obtainBuffer → memcpy → releaseBuffer
        AR_N-->>App: 返回PCM数据
    end
```

### 5.10.3 RecordThread类型与路由

| RecordThread类型 | 输入设备 | Flag | 特殊处理 |
|-----------------|---------|------|---------|
| RecordThread | 麦克风/有线耳机 | NONE | 标准录音+重采样+Effect |
| MmapCaptureThread | 麦克风(MMAP) | MMAP_NOIRQ | 共享内存直通，无threadLoop |

**录音路由决策**（AudioPolicyManager）:

| AudioSource | 路由优先级 | HAL预处理 |
|-------------|----------|----------|
| `VOICE_RECOGNITION` | 主MIC > 有线 > BT SCO | **关闭AGC+AEC** |
| `VOICE_COMMUNICATION` | BT SCO > 有线 > 主MIC | **AEC+NS** |
| `MIC` | 主MIC > 有线 > BT SCO | AGC可选 |
| `CAMCORDER` | 主MIC(与摄像头同向) | 无 |

> **关键区别**: 播放路径是"App→AF→HAL"(推模型)，录音路径是"HAL→AF→App"(拉模型)。RecordThread从HAL读取数据推入共享内存，App从共享内存拉取数据。

---

## 5.11 AudioMixer与Resampler — 混音引擎核心

### 5.11.1 AudioMixer继承体系

```mermaid
classDiagram
    class AudioMixerBase {
        +mTrackNames : SortedVector~int~
        +mConfig : mixer_config_t
        +mEnabled : SortedVector~int~
        +process()
        +invalidate()
        +setBufferProvider(id, provider)
    }
    class AudioMixer {
        +mState : State*
        +mHook : process_hook_t
        +process()
        +setParameter()
        +setPostProcessing()
    }
    AudioMixerBase <|-- AudioMixer
```

**源码位置**: [`AudioMixerBase.h`](frameworks/av/media/libaudioprocessing/include/media/AudioMixerBase.h) / [`AudioMixer.h`](frameworks/av/media/libaudioprocessing/include/media/AudioMixer.h)

### 5.11.2 混音处理流程

```mermaid
flowchart TB
    PREPARE["prepareTracks_l()"] --> CHECK_ACTIVE{"有活跃Track?"}
    CHECK_ACTIVE -->|"否"| IDLE["mHook = process_NoOp<br/>静音输出"]
    CHECK_ACTIVE -->|"1个Track"| SINGLE{"Track需要音量调节?"}
    CHECK_ACTIVE -->|"多个Track"| MULTI["process__genericNoResampling<br/>或process__genericResampling"]
    SINGLE -->|"不需要"| COPY["process__OneTrackCopy<br/>直接拷贝(零处理)"]
    SINGLE -->|"需要"| VOL["process__OneTrack16BitStereo<br/>音量乘法"]
    MULTI --> RESAMPLE_CHECK{"需要重采样?"}
    RESAMPLE_CHECK -->|"否"| NO_RESAMPLE["逐Track: volume乘法→累加"]
    RESAMPLE_CHECK -->|"是"| RESAMPLE["逐Track: Resampler.process()→volume乘法→累加"]
```

**Hook机制**: AudioMixer使用函数指针Hook动态选择处理路径，避免每次混音都做条件判断。`setParameter()`时根据Track参数更新Hook指向最优实现。

### 5.11.3 AudioResampler继承体系

```mermaid
classDiagram
    class AudioResampler {
        +mSampleRate : int
        +mQuality : quality_t
        +init()
        +resample()
        +setSampleRate()
        +setQuality()
    }
    class AudioResamplerDyn {
        +mCoefReader : CoefReader
        +mInBuffer : BufferProvider
        +resample()
    }
    class AudioResamplerCubic {
        +mX0, mX1, mX2, mX3 : int16_t
        +resample()
    }
    class AudioResamplerSinc {
        +mFirCoef : int32_t*
        +resample()
    }
    AudioResampler <|-- AudioResamplerDyn : 默认(高质量)
    AudioResampler <|-- AudioResamplerCubic : 低质量
    AudioResampler <|-- AudioResamplerSinc : 中质量
```

**源码位置**: [`libaudioprocessing/`](frameworks/av/media/libaudioprocessing/)

| Resampler类型 | 质量 | CPU开销 | 算法 | 适用场景 |
|--------------|------|---------|------|---------|
| `Cubic` | LOW | 最低 | 三次插值 | 后台播放/省电 |
| `Sinc` | MED | 中等 | Sinc插值(固定系数) | 一般播放 |
| `Dyn` | HIGH/VERY_HIGH | 最高 | 多相FIR滤波(动态系数) | 专业音频/HiFi |

> **性能关键路径**: AudioMixer.process()在每个混音周期执行(NormalMixer ~20ms, FastMixer ~10ms)，是音频系统CPU消耗最大的路径。Hook机制+SIMD优化(NEON)确保实时性。

---

> [← 上一篇：Native Framework](04_Native_Framework_Layer.md) | [返回导航](README.md) | [下一篇：Audio Policy Engine →](06_Audio_Policy_Engine.md)