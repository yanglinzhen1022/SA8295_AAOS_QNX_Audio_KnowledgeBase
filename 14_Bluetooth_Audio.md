# 第十四篇：Bluetooth Audio 深度解析

> [← 上一篇：Volume & Device](13_Volume_Device_Deep_Dive.md) | [返回导航](README.md) | [下一篇：Debug & OEM →](15_Debug_and_OEM_Guide.md)

---

蓝牙音频是Android Audio系统中最复杂的设备类型之一，涉及A2DP、LE Audio、SCO/HFP、Hearing Aid四种协议，每种都有独立的HAL接口、音量模型和路由策略。

## 14.1 蓝牙音频协议总览

```mermaid
graph TB
    subgraph "Bluetooth Audio协议栈"
        A2DP["A2DP<br/>高级音频分发<br/>高质量音乐流"]
        LEA["LE Audio<br/>低功耗音频<br/>AOSP12+支持"]
        SCO["SCO/HFP<br/>语音通话<br/>窄带/宽带/超宽带"]
        HA["Hearing Aid<br/>助听器<br/>ASHA协议"]
    end

    subgraph "Audio HAL(AIDL)"
        IBT["IBluetooth<br/>SCO+HFP配置"]
        IBA2["IBluetoothA2dp<br/>A2DP Offload"]
        IBLE["IBluetoothLe<br/>LE Audio Offload"]
    end

    subgraph "Framework"
        ADB["AudioDeviceBroker<br/>设备连接/音量"]
        BTH["BtHelper<br/>Profile代理"]
    end

    A2DP --> IBA2
    LEA --> IBLE
    SCO --> IBT
    HA --> ADB

    IBA2 --> ADB
    IBLE --> ADB
    IBT --> BTH
    BTH --> ADB
```

| 协议 | Audio设备类型 | 音频方向 | 用途 | 编解码 |
|------|-------------|---------|------|--------|
| A2DP | DEVICE_OUT_BLUETOOTH_A2DP | 输出 | 高质量音乐 | SBC/AAC/aptX/LDAC |
| A2DP Sink | DEVICE_IN_BLUETOOTH_A2DP | 输入 | 蓝牙录音源 | SBC |
| LE Audio | DEVICE_OUT_BLE_HEADSET / DEVICE_IN_BLE_HEADSET | 双向 | 低功耗音频 | LC3 |
| LE Broadcast | DEVICE_OUT_BLE_BROADCAST | 输出 | 广播音频 | LC3 |
| SCO | DEVICE_OUT_BLUETOOTH_SCO_HEADSET / DEVICE_IN_BLUETOOTH_SCO | 双向 | 通话语音 | CVSD/mSBC/LC3 |
| Hearing Aid | DEVICE_OUT_HEARING_AID | 输出 | 助听器 | ASHA自定义 |

## 14.2 A2DP — 高级音频分发协议

### 14.2.1 A2DP连接→Audio路由流程

```mermaid
sequenceDiagram
    participant BT as BluetoothA2dpService
    participant BTH as BtHelper
    participant ADB as AudioDeviceBroker
    participant ADI as AudioDeviceInventory
    participant APS as AudioPolicyService
    participant APM as AudioPolicyManager
    participant AF as AudioFlinger

    BT->>BTH: onActiveDeviceChanged(device, CONNECTED)
    BTH->>ADB: queueOnBluetoothActiveDeviceChanged()
    ADB->>ADB: createBtDeviceInfo(profile=A2DP, codec=SBC/AAC/LDAC)
    ADB->>ADI: postBluetoothDeviceConfigChange(BtDeviceInfo)
    ADI->>APS: setDeviceConnectionState(DEVICE_OUT_BLUETOOTH_A2DP, AVAILABLE)
    APS->>APM: setDeviceConnectionStateInt()
    APM->>APM: mAvailableOutputDevices.add(A2DP)
    APM->>APM: checkOutputsForDevice() → openOutput for A2DP
    APM->>APM: checkForDeviceAndOutputChanges()
    APM->>APM: checkOutputForAllStrategies() → MUSIC迁移到A2DP
    APM->>AF: openOutput(A2DP) → 创建新MixerThread
    APM->>APM: 活跃Track迁移到A2DP输出
```

### 14.2.2 A2DP音量机制 — AVRCP绝对音量

A2DP设备使用**AVRCP绝对音量**协议，手机端音量直接映射到耳机端：

```mermaid
flowchart TB
    VOL["音量调节(KeyEvent/API)"] --> A2DP_CHECK{"当前路由是A2DP?"}
    A2DP_CHECK -->|"是"| AVRCP["postSetAvrcpAbsoluteVolumeIndex()<br/>发送AVRCP绝对音量"]
    A2DP_CHECK -->|"否"| NORMAL["正常音量路径"]
    AVRCP --> BTVOL["BluetoothA2dp.setAvrcpAbsoluteVolume()<br/>音量值0-127"]
    BTVOL --> EARPHONE["蓝牙耳机端执行音量"]

    subgraph "固定音量设备(FIXED_VOLUME)"
        FIXED["isFixedVolumeDevice()==true<br/>A2DP通常为固定音量设备"]
        FIXED -->|"index=0"| MUTE2["静音(断开音频流)"]
        FIXED -->|"index>0"| MAX2["最大音量(音量由耳机控制)"]
    end
```

**关键源码位置**:
- [`AudioDeviceBroker.postSetAvrcpAbsoluteVolumeIndex()`](frameworks/base/services/core/java/com/android/server/audio/AudioDeviceBroker.java:1088): AVRCP音量发送
- [`AudioService`](frameworks/base/services/core/java/com/android/server/audio/AudioService.java:4516): A2DP音量路由判断

### 14.2.3 A2DP Codec协商

```mermaid
flowchart LR
    subgraph "Codec协商流程"
        CONNECT["A2DP连接"] --> QUERY["BluetoothA2dp.getCodecStatus()"]
        QUERY --> CAPS["获取双方Codec能力"]
        CAPS --> SELECT["BluetoothA2dp.setCodecConfig()<br/>选择最优Codec"]
        SELECT --> CONFIG["配置Codec参数<br/>(采样率/位深/声道)"]
    end

    subgraph "Codec优先级(默认)"
        P1["LDAC(990kbps)"]
        P2["aptX HD(576kbps)"]
        P3["aptX(384kbps)"]
        P4["AAC(256kbps)"]
        P5["SBC(328kbps)"]
    end
```

**Codec信息传递**: A2DP连接时，`BtHelper.getA2dpCodec(device)`获取当前Codec类型，传递给AudioPolicy用于配置输出流参数。

### 14.2.4 A2DP Offload

```mermaid
flowchart TB
    subgraph "A2DP Offload路径"
        APP["App: AudioTrack写PCM"] --> AF["AudioFlinger: 混音"]
        AF -->|"PCM数据"| BTA2["IBluetoothA2dp HAL"]
        BTA2 -->|"reconfigureOffload()"| DSP["DSP: 编码+发送"]
    end

    subgraph "IBluetoothA2dp接口(AIDL)"
        ENABLE["isEnabled() / setEnabled()"]
        SUPPORT["supportsOffloadReconfiguration()"]
        RECONFIG["reconfigureOffload(VendorParameter[])"]
    end
```

A2DP Offload将编码工作从CPU卸载到DSP，降低功耗。`reconfigureOffload()`允许运行时切换Codec参数。

### 14.2.5 A2DP Suspend机制

A2DP可以被挂起（Suspend），典型场景：SCO通话时需要暂停A2DP音频流。

```mermaid
flowchart TB
    SCO_CALL["SCO通话开始"] --> SUSPEND["AudioService.setA2dpSuspended(true)"]
    SUSPEND --> BROKER["AudioDeviceBroker.setA2dpSuspended()"]
    BROKER --> APS_SUSPEND["AudioSystem.setParameters('A2dpSuspend=true')"]
    APS_SUSPEND --> HAL_SUSPEND["HAL暂停A2DP流"]

    SCO_END["SCO通话结束"] --> RESUME["AudioService.setA2dpSuspended(false)"]
    RESUME --> BROKER2["AudioDeviceBroker.setA2dpSuspended()"]
    BROKER2 --> APS_RESUME["AudioSystem.setParameters('A2dpSuspend=false')"]
    APS_RESUME --> HAL_RESUME["HAL恢复A2DP流"]
```

**关键源码位置**:
- [`AudioDeviceBroker.setA2dpSuspended()`](frameworks/base/services/core/java/com/android/server/audio/AudioDeviceBroker.java:1004): A2DP挂起控制
- [`AudioService.setA2dpSuspended()`](frameworks/base/services/core/java/com/android/server/audio/AudioService.java:6416): 公共API

## 14.3 LE Audio — 低功耗蓝牙音频

### 14.3.1 LE Audio架构

```mermaid
graph TB
    subgraph "LE Audio协议栈"
        BAP["BAP<br/>Basic Audio Profile"]
        CAP["CAP<br/>Common Audio Profile"]
        ASCS["ASCS<br/>Audio Stream Control Service"]
        BASS["BASS<br/>Broadcast Audio Scan Service"]
    end

    subgraph "LE Audio在Audio系统的映射"
        CIS["CIS<br/>Connected Isochronous Stream<br/>单播音频"]
        BIS["BIS<br/>Broadcast Isochronous Stream<br/>广播音频"]
    end

    subgraph "设备类型"
        BLE_OUT["DEVICE_OUT_BLE_HEADSET<br/>LE Audio输出"]
        BLE_IN["DEVICE_IN_BLE_HEADSET<br/>LE Audio输入"]
        BLE_BC["DEVICE_OUT_BLE_BROADCAST<br/>LE Audio广播"]
    end

    BAP --> CIS
    BAP --> BIS
    CIS --> BLE_OUT
    CIS --> BLE_IN
    BIS --> BLE_BC
```

### 14.3.2 LE Audio连接→路由流程

```mermaid
sequenceDiagram
    participant BT as BluetoothLeAudioService
    participant BTH as BtHelper
    participant ADB as AudioDeviceBroker
    participant ADI as AudioDeviceInventory
    participant APM as AudioPolicyManager
    participant AF as AudioFlinger

    BT->>BTH: onActiveDeviceChanged(device, CONNECTED)
    BTH->>ADB: queueOnBluetoothActiveDeviceChanged()
    ADB->>ADB: createBtDeviceInfo(profile=LE_AUDIO, isLeOutput=true)
    ADB->>ADI: postBluetoothDeviceConfigChange()
    ADI->>APM: setDeviceConnectionState(DEVICE_OUT_BLE_HEADSET, AVAILABLE)
    APM->>APM: mAvailableOutputDevices.add(BLE_HEADSET)
    APM->>APM: checkForDeviceAndOutputChanges() → MUSIC迁移到LE Audio
    APM->>AF: openOutput(BLE_HEADSET)
```

### 14.3.3 LE Audio音量机制

```mermaid
flowchart TB
    VOL["音量调节"] --> LEA_CHECK{"当前路由是LE Audio?"}
    LEA_CHECK -->|"是"| LEVOL["postSetLeAudioVolumeIndex()<br/>LE Audio音量0-255"]
    LEA_CHECK -->|"否"| NORMAL["正常音量路径"]
    LEVOL --> BTH_SET["BtHelper.setLeAudioVolume()<br/>BluetoothLeAudio.setVolume()"]
    BTH_SET --> VCP["VCP(Volume Control Profile)<br/>LE Audio音量控制协议"]

    subgraph "LE Audio音量范围"
        RANGE["BT_LE_AUDIO_MIN_VOL = 0<br/>BT_LE_AUDIO_MAX_VOL = 255"]
    end
```

**关键源码位置**:
- [`AudioDeviceBroker.postSetLeAudioVolumeIndex()`](frameworks/base/services/core/java/com/android/server/audio/AudioDeviceBroker.java:1096): LE Audio音量发送
- [`BtHelper.setLeAudioVolume()`](frameworks/base/services/core/java/com/android/server/audio/BtHelper.java): VCP音量设置

### 14.3.4 LE Audio Suspend机制

与A2DP类似，LE Audio也支持Suspend：

```mermaid
flowchart TB
    SCO["SCO通话开始"] --> LE_SUSPEND["AudioService.setLeAudioSuspended(true)"]
    LE_SUSPEND --> BROKER["AudioDeviceBroker.setLeAudioSuspended()"]
    BROKER --> PARAMS["AudioSystem.setParameters()"]

    SCO_END["SCO通话结束"] --> LE_RESUME["AudioService.setLeAudioSuspended(false)"]
    LE_RESUME --> BROKER2["AudioDeviceBroker.setLeAudioSuspended()"]
```

**关键源码位置**: [`AudioDeviceBroker.setLeAudioSuspended()`](frameworks/base/services/core/java/com/android/server/audio/AudioDeviceBroker.java:1034)

### 14.3.5 LE Audio Offload

```mermaid
flowchart TB
    subgraph "IBluetoothLe接口(AIDL)"
        LE_ENABLE["isEnabled() / setEnabled()"]
        LE_SUPPORT["supportsOffloadReconfiguration()"]
        LE_RECONFIG["reconfigureOffload(VendorParameter[])"]
    end

    subgraph "LE Audio Offload路径"
        APP["App: AudioTrack写PCM"] --> AF["AudioFlinger: 混音"]
        AF -->|"PCM数据"| IBTLE["IBluetoothLe HAL"]
        IBTLE -->|"reconfigureOffload()"| DSP["DSP: LC3编码+发送"]
    end
```

### 14.3.6 LE Audio Broadcast

LE Audio Broadcast (Auracast) 允许一个源设备向多个接收设备广播音频：

```mermaid
graph TB
    SOURCE["广播源<br/>Broadcast Source"] --> BIS1["BIS #1<br/>LC3编码流"]
    SOURCE --> BIS2["BIS #2<br/>LC3编码流"]
    BIS1 --> RX1["接收设备1<br/>DEVICE_OUT_BLE_BROADCAST"]
    BIS1 --> RX2["接收设备2"]
    BIS2 --> RX3["接收设备3<br/>(不同语言)"]
```

## 14.4 SCO/HFP — 通话语音

### 14.4.1 SCO音频状态机

```mermaid
stateDiagram-v2
    state "INACTIVE" as INACTIVE
    state "ACTIVATE_REQ<br/>(等待Headset连接)" as ACTIVATE_REQ
    state "ACTIVE_EXTERNAL<br/>(BT端发起)" as ACTIVE_EXT
    state "ACTIVE_INTERNAL<br/>(App API发起)" as ACTIVE_INT
    state "DEACTIVATE_REQ<br/>(等待Headset连接)" as DEACTIVATE_REQ
    state "DEACTIVATING<br/>(等待BT意图)" as DEACTIVATING

    [*] --> INACTIVE
    INACTIVE --> ACTIVATE_REQ: startScoAudio()<br/>(Headset未连接)
    INACTIVE --> ACTIVE_EXT: Headset发起VR/通话
    INACTIVE --> ACTIVE_INT: AudioManager.startBluetoothSco()
    ACTIVATE_REQ --> ACTIVE_INT: Headset连接完成
    ACTIVE_EXT --> DEACTIVATING: BT端停止
    ACTIVE_INT --> DEACTIVATING: stopBluetoothSco()
    DEACTIVATING --> INACTIVE: SCO断开意图
```

### 14.4.2 IBluetooth SCO/HFP配置(AIDL)

```mermaid
classDiagram
    class IBluetooth {
        +setScoConfig(ScoConfig) ScoConfig
        +setHfpConfig(HfpConfig) HfpConfig
    }
    class ScoConfig {
        +isEnabled : Boolean?
        +isNrecEnabled : Boolean?
        +mode : Mode
        +debugName : String?
    }
    class HfpConfig {
        +isEnabled : Boolean?
        +sampleRate : Int?
        +volume : Float?
        VOLUME_MIN = 0
        VOLUME_MAX = 1
    }
    class ScoConfig_Mode {
        <<enumeration>>
        UNSPECIFIED
        SCO
        SCO_WB
        SCO_SWB
    }

    IBluetooth --> ScoConfig
    IBluetooth --> HfpConfig
    ScoConfig --> ScoConfig_Mode
```

**SCO模式说明**:
| 模式 | 带宽 | 采样率 | 编解码 | 音质 |
|------|------|--------|--------|------|
| SCO (NB) | 窄带 | 8kHz | CVSD | 基本通话质量 |
| SCO_WB (WB) | 宽带 | 16kHz | mSBC | 改善通话质量 |
| SCO_SWB (SWB) | 超宽带 | 32kHz | LC3 | 高清通话质量 |

## 14.5 Hearing Aid — 助听器

### 14.5.1 ASHA协议

ASHA (Audio Streaming for Hearing Aids) 是Google专为助听器设计的蓝牙协议：

```mermaid
flowchart TB
    subgraph "Hearing Aid音频路径"
        APP["App: AudioTrack"] --> AF["AudioFlinger: 混音"]
        AF -->|"DEVICE_OUT_HEARING_AID"| HAL["Audio HAL"]
        HAL --> BTHA["BluetoothHearingAid"]
        BTHA -->|"ASHA协议"| HA_DEVICE["助听器设备"]
    end

    subgraph "音量控制"
        HA_VOL["BtHelper.setHearingAidVolume()"]
        HA_VOL --> GAIN["HearingAid.setVolume()<br/>增益范围: -128~0 dB"]
    end
```

**关键参数**:
- 设备类型: `DEVICE_OUT_HEARING_AID`
- 音量范围: `BT_HEARING_AID_GAIN_MIN = -128 dB` ~ 0 dB
- 与LE Audio的关系: 未来ASHA将迁移到LE Audio的HAP (Hearing Access Profile)

## 14.6 蓝牙音频设备与AudioDeviceBroker交互

### 14.6.1 BtDeviceInfo创建

```mermaid
flowchart TB
    EVENT["蓝牙设备变化事件"] --> PROFILE{"Profile类型?"}
    PROFILE -->|"A2DP"| A2DP_DEV["audioDevice=DEVICE_OUT_BLUETOOTH_A2DP<br/>codec=getA2dpCodec()"]
    PROFILE -->|"A2DP_SINK"| A2DP_IN["audioDevice=DEVICE_IN_BLUETOOTH_A2DP"]
    PROFILE -->|"HEARING_AID"| HA_DEV["audioDevice=DEVICE_OUT_HEARING_AID"]
    PROFILE -->|"LE_AUDIO"| LEA_CHECK{"isLeOutput()?"}
    LEA_CHECK -->|"是"| LEA_OUT["audioDevice=DEVICE_OUT_BLE_HEADSET"]
    LEA_CHECK -->|"否"| LEA_IN["audioDevice=DEVICE_IN_BLE_HEADSET"]
    PROFILE -->|"LE_AUDIO_BROADCAST"| LEA_BC["audioDevice=DEVICE_OUT_BLE_BROADCAST"]
```

**关键源码位置**: [`AudioDeviceBroker.createBtDeviceInfo()`](frameworks/base/services/core/java/com/android/server/audio/AudioDeviceBroker.java:812)

### 14.6.2 蓝牙设备切换流程

```mermaid
sequenceDiagram
    participant BT as BluetoothService
    participant BTH as BtHelper
    participant ADB as AudioDeviceBroker
    participant ADI as AudioDeviceInventory
    participant APM as AudioPolicyManager

    BT->>BTH: onActiveDeviceChanged(newDevice)
    BTH->>ADB: queueOnBluetoothActiveDeviceChanged(BtDeviceChangedData)

    alt 设备未变化(同一设备更新)
        ADB->>ADI: postBluetoothDeviceConfigChange(STATE_CONNECTED)
    else 设备切换
        ADB->>ADI: 断开旧设备(STATE_DISCONNECTED)
        ADB->>ADI: 连接新设备(STATE_CONNECTED)
        ADI->>APM: setDeviceConnectionState(UNAVAILABLE) 旧设备
        ADI->>APM: setDeviceConnectionState(AVAILABLE) 新设备
        APM->>APM: 路由重评估 + Track迁移
    end
```

### 14.6.3 A2DP连接超时机制

```mermaid
flowchart TB
    A2DP_CONN["A2DP连接事件"] --> TIMEOUT["setA2dpTimeout()<br/>延迟8秒(BTA2DP_DOCK_TIMEOUT_MS)"]
    TIMEOUT --> CHECK{"8秒后检查"}
    CHECK -->|"A2DP仍连接"| NORMAL["正常使用"]
    CHECK -->|"A2DP已断开"| FALLBACK["回退到Speaker"]
    CHECK -->|"A2DP未完成握手"| MUTE_CHECK["BTA2DP_MUTE_CHECK_DELAY_MS=100ms<br/>检查是否需要取消静音"]
```

## 14.7 蓝牙音频对比总结

| 维度 | A2DP | LE Audio | SCO/HFP | Hearing Aid |
|------|------|----------|---------|-------------|
| 引入版本 | Android 1.5 | Android 12 | Android 1.5 | Android 9 |
| 音频方向 | 输出 | 双向 | 双向 | 输出 |
| 编解码 | SBC/AAC/aptX/LDAC | LC3 | CVSD/mSBC/LC3 | 自定义 |
| 延迟 | ~200ms | ~30ms | ~50ms | ~200ms |
| 音量模型 | AVRCP绝对音量(0-127) | VCP(0-255) | HFP volume(0-1) | 增益(-128~0dB) |
| HAL接口 | IBluetoothA2dp | IBluetoothLe | IBluetooth(ScoConfig) | 无专用接口 |
| Offload支持 | reconfigureOffload | reconfigureOffload | N/A | N/A |
| 固定音量设备 | 是(FLAG_FIXED_VOLUME) | 否 | 否 | 是 |
| 典型用途 | 音乐播放 | 音乐+通话+广播 | 通话语音 | 助听器 |

---

> [← 上一篇：Volume & Device](13_Volume_Device_Deep_Dive.md) | [返回导航](README.md) | [下一篇：Debug & OEM →](15_Debug_and_OEM_Guide.md)