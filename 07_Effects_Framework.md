# 第七篇：Effects Framework

> [← 上一篇：Audio Policy Engine](06_Audio_Policy_Engine.md) | [返回导航](README.md) | [下一篇：HAL Layer →](08_HAL_Layer.md)

---

## 7.1 AudioEffect架构总览

### 模块职责
Effects Framework提供音频效果处理的统一框架，允许App和系统对音频流应用均衡器、低音增强、回声消除等效果。

### 分层架构

```mermaid
graph TB
    subgraph App
        AE_J["AudioEffect.java / Equalizer.java / ..."]
    end
    subgraph Native Client
        AE_N["AudioEffect.cpp (libaudioclient)"]
    end
    subgraph AudioFlinger
        EM["EffectModule"]
        EC["EffectChain"]
    end
    subgraph HAL
        EHAL["Effect HAL (vendor)"]
    end
    AE_J -->|"JNI"| AE_N -->|"Binder"| EM -->|"HAL"| EHAL
    EM --> EC
    EC --> PlaybackThread
```

---

## 7.2 EffectChain — 效果链

### 模块职责
[`EffectChain`](frameworks/av/services/audioflinger/Effects.h:448)是同一sessionId下所有EffectModule的集合，按顺序串联处理音频数据。

### EffectChain与Thread的关系

```mermaid
graph TB
    subgraph "PlaybackThread(MixerThread)"
        MIXER["AudioMixer混音输出"]
        CHAIN1["EffectChain(sessionId=0)<br/>全局效果链"]
        CHAIN2["EffectChain(sessionId=1234)<br/>App私有效果链"]
        CHAIN3["EffectChain(sessionId=5678)<br/>另一个App效果链"]
    end

    MIXER -->|"全局session=0"| CHAIN1
    MIXER -->|"session=1234数据"| CHAIN2
    MIXER -->|"session=5678数据"| CHAIN3
    CHAIN1 --> HAL["threadLoop_write()→HAL"]
    CHAIN2 --> HAL
    CHAIN3 --> HAL
```

> **关键**: session=0是全局效果链，作用于该Thread上所有Track。非0 sessionId的EffectChain只处理对应Track的数据。

### EffectChain内部处理流程 — [`process_l()`](frameworks/av/services/audioflinger/Effects.cpp:2273)

```mermaid
flowchart TB
    START["EffectChain.process_l()"] --> CHECK{"Offload/Mmap<br/>Thread?"}
    CHECK -->|"是"| SKIP["不处理效果<br/>(Offload线程不支持)"]
    CHECK -->|"否"| TRACKCHK{"当前session有<br/>活跃Track?"}
    TRACKCHK -->|"否+无tail"| SKIP2["跳过处理"]
    TRACKCHK -->|"否+有tail"| CLEAR["clearInputBuffer_l()<br/>清空输入buffer<br/>mTailBufferCount--"]
    TRACKCHK -->|"是"| UPDATE["mInBuffer->update()<br/>mOutBuffer->update()"]
    UPDATE --> LOOP["遍历mEffects列表"]
    LOOP -->|"每个EffectModule"| PROCESS["EffectModule.process()"]
    PROCESS --> NEXT["下一个EffectModule"]
    NEXT --> LOOP
    LOOP -->|"处理完毕"| COMMIT["mInBuffer->commit()<br/>mOutBuffer->commit()"]
    COMMIT --> STATE["遍历updateState()<br/>检查效果状态变化"]
    STATE --> VOLRESET{"需要重置音量?"}
    VOLRESET -->|"是"| RESETVOL["resetVolume_l()"]
    VOLRESET -->|"否"| DONE["完成"]
```

**Tail处理机制**: 某些效果(如Reverb)在Track停止后仍有残响(tail)。`mTailBufferCount`记录还需处理的帧数，确保残响自然衰减。

### Insert效果 vs Auxiliary效果

| 类型 | EFFECT_FLAG_TYPE | 在链中位置 | 输入buffer | 输出buffer | 说明 |
|------|-----------------|-----------|-----------|-----------|------|
| Insert(插入) | EFFECT_FLAG_TYPE_INSERT | 按优先级排序 | 前一个Effect的输出 | 下一个Effect的输入 | 串行处理，替换输入 |
| Auxiliary(辅助) | EFFECT_FLAG_TYPE_AUXILIARY | 链头部(mEffects[0]) | 独立mono buffer | 链输入buffer(mInBuffer) | 叠加到主信号上(如Reverb) |
| Pre-processing | EFFECT_FLAG_TYPE_PRE_PROC | 录音链中 | — | — | AEC/NS/AGC，录音方向 |
| Replace | EFFECT_FLAG_TYPE_REPLACE | 按优先级排序 | 链输入 | 链输出 | 完全替换信号(如Spatializer) |

**Insert效果排序规则** — [`getInsertIndex()`](frameworks/av/services/audioflinger/Effects.cpp:2382):
```
排序优先级(从前往后):
1. EFFECT_FLAG_TYPE_INSERT → 通用Insert效果(EQ/BassBoost)
2. EFFECT_FLAG_TYPE_AUXILIARY → 辅助效果(Reverb)
3. EFFECT_FLAG_TYPE_REPLACE → 替换效果(Spatializer)
4. 更高优先级的Insert排在前面
```

### 效果链Buffer管理

```mermaid
graph LR
    subgraph "Insert效果链(典型3效果)"
        IN["Chain mInBuffer<br/>(混音输出)"] --> EM1["EffectModule 1<br/>(AEC)"]
        EM1 -->|"写回mInBuffer"| IN
        EM1 --> EM2["EffectModule 2<br/>(NS)"]
        EM2 -->|"写回mInBuffer"| IN2["mInBuffer"]
        IN2 --> EM3["EffectModule 3<br/>(EQ)"]
        EM3 -->|"最后一个→写mOutBuffer"| OUT["Chain mOutBuffer<br/>(→HAL写入)"]
    end
```

> **优化**: 多数Insert效果的输出写回mInBuffer(in-place处理)，只有最后一个效果写到mOutBuffer，减少内存拷贝。

---

## 7.3 EffectModule — 单个效果实例

### 模块职责
EffectModule封装一个音频效果实例，通过Effect HAL与Vendor实现交互。

### EffectModule状态机

```mermaid
stateDiagram-v2
    state "IDLE" as IDLE
    state "REMOVED" as REMOVED
    state "ACTIVE(处理中)" as ACTIVE
    state "STOPPED" as STOPPED
    state "DESTROYED" as DESTROYED

    [*] --> IDLE: create_l()
    IDLE --> ACTIVE: start_l()
    ACTIVE --> STOPPED: stop_l()
    STOPPED --> ACTIVE: start_l()
    IDLE --> REMOVED: remove_l()
    ACTIVE --> REMOVED: remove_l()
    STOPPED --> REMOVED: remove_l()
    REMOVED --> DESTROYED: release()
```

### EffectModule.process()详解（源码: [`Effects.cpp:672`](frameworks/av/services/audioflinger/Effects.cpp:672)）

```mermaid
flowchart TB
    START["EffectModule.process()"] --> CHECK{"mState==DESTROYED<br/>或mEffectInterface==0?"}
    CHECK -->|"是"| RETURN["直接返回"]
    CHECK -->|"否"| TYPE{"效果类型?"}

    TYPE -->|"AUXILIARY<br/>(辅助效果)"| AUX["辅助效果处理"]
    TYPE -->|"INSERT/REPLACE<br/>(插入/替换)"| INSERT["Insert效果处理"]

    subgraph "Auxiliary效果(如Reverb)"
        AUX --> AUXCONV["输入buffer格式转换<br/>q4.27→float(或float→i16)"]
        AUXCONV --> AUXPROC["effect_engine->process()<br/>调用HAL实现"]
        AUXPROC --> AUXACC["累加到mInBuffer<br/>(叠加到主信号)"]
        AUXACC --> AUXCLEAR["清空aux输入buffer"]
    end

    subgraph "Insert效果(如EQ/AEC/NS)"
        INSERT --> IMPL{"isProcessImplemented()?"}
        IMPL -->|"是"| PROC["effect_engine->process()<br/>调用HAL实现"]
        IMPL -->|"否(bypass)"| BYPASS{"inChannelCount==<br/>outChannelCount?"}
        BYPASS -->|"是"| COPY["copyInputToOutput()<br/>直接拷贝in→out"]
        BYPASS -->|"否"| NOP["不处理(跳过)"]
    end

    PROC --> SATURATE["饱和处理(Saturation)<br/>float→i16 clamp"]
```

### 核心方法

| 方法 | 说明 | 源码位置 |
|------|------|---------|
| `create_l()` | 创建效果实例，加载HAL库 | Effects.cpp |
| `start_l()` | 启动效果处理(状态→ACTIVE) | Effects.cpp |
| `stop_l()` | 停止效果处理(状态→STOPPED) | Effects.cpp |
| `process()` | 处理一帧音频数据 | Effects.cpp:672 |
| `setVolume_l()` | 设置音量(效果可能修改音量曲线) | Effects.cpp |
| `setDevices_l()` | 设置输出设备(效果可能根据设备调整) | Effects.cpp |
| `setMode_l()` | 设置音频模式(NORMAL/RING/IN_CALL) | Effects.cpp |
| `setAudioSource_l()` | 设置录音源(AEC/NS需要知道录音源) | Effects.cpp |
| `updateState()` | 更新效果状态(参数变更) | Effects.cpp |

### FLOAT_EFFECT_CHAIN架构

AOSP14默认使用`FLOAT_EFFECT_CHAIN`，效果链内部使用float精度处理：

```
PCM16(Track输出) → float(EffectChain内部) → PCM16/PCM_FLOAT(HAL写入)
```

| 模式 | 内部格式 | 优势 |
|------|---------|------|
| FLOAT_EFFECT_CHAIN | float(32bit) | 高精度，无截断噪声 |
| 旧模式(PCM16链) | int16 | 兼容旧HAL |

> **Auxiliary效果**: 输入使用q4.27定点格式(避免AudioMixer累加溢出)，process()中转换为float处理

---

## 7.4 内置效果与Vendor效果

### 内置效果

| 效果 | Type UUID | 场景 |
|------|-----------|------|
| AcousticEchoCanceler | AEC | VoIP通话回声消除 |
| NoiseSuppressor | NS | VoIP通话降噪 |
| AutomaticGainControl | AGC | 自动增益控制 |
| Equalizer | EQ | 频率均衡 |
| BassBoost | — | 低音增强 |
| Virtualizer | — | 虚拟环绕声 |
| LoudnessEnhancer | — | 响度增强 |

### Vendor效果
Vendor在`audio_effects.xml`中声明自定义效果：
```xml
<effects>
    <effect name="custom_dsp_effect" uuid="xxx-xxx-xxx">
        <library name="custom_fx_lib" path="libfxcustom.so"/>
    </effect>
</effects>
```

### OEM定制点
- **自定义效果库**: 实现Effect HAL接口，在`audio_effects.xml`中声明
- **效果策略**: 通过`audio_effect_policy.xml`控制系统级效果的自动附加
- **预处理效果**: 为特定AudioSource附加AEC/NS/AGC（VoIP场景）

---

> [← 上一篇：Audio Policy Engine](06_Audio_Policy_Engine.md) | [返回导航](README.md) | [下一篇：HAL Layer →](08_HAL_Layer.md)