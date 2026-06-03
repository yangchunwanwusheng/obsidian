---
type: lesson
tags: [语音面试, WebSocket, TTS, ASR, 实时通信, 豆包, Fun-ASR]
created: 2026-05-25
updated: 2026-05-25
difficulty: advanced
prerequisites: [WebSocket协议, 音频处理基础, 流式通信]
topic: 开源项目深度拆解
status: completed
series: { name: "ai-interview-伯乐深度拆解", part: 7 }
---

# 语音面试系统深度拆解

> 语音面试是伯乐的技术亮点之一。它通过 WebSocket 实现了实时双向语音交互，涉及三个并发流的协调：ASR（语音转文字）、Agent（LLM 推理）、TTS（文字转语音）。

---

## 一、整体架构

```
┌──────────────────────────────────────────────────────────────┐
│                        浏览器 (前端)                          │
│                                                              │
│  ┌──────────┐    ┌──────────────┐    ┌──────────────────┐   │
│  │ 麦克风    │    │  AudioContext │    │  WebSocket Client │   │
│  │ (输入)    │    │  (播放输出)    │    │  (双向通信)       │   │
│  └────┬─────┘    └──────▲───────┘    └────────┬─────────┘   │
│       │                 │                      │             │
│       ▼                 │                      │             │
│  MediaRecorder     Audio Buffer           ws://api:5050/     │
│  (PCM 16kHz)       (PCM → 播放)          voice/interview     │
└───────┼─────────────────┼──────────────────────┼─────────────┘
        │                 │                      │
        │    ┌────────────┼──────────────────────┘
        │    │            │
        ▼    │            ▼
┌──────────────────────────────────────────────────────────────┐
│                     后端 (FastAPI WebSocket)                  │
│                                                              │
│  ┌─────────────────┐                                        │
│  │ voice_interview  │   src/services/voice_interview_service  │
│  │    _service.py   │   (~1500行)                            │
│  └────────┬────────┘                                        │
│           │                                                   │
│     ┌─────┼─────────────────────┐                            │
│     ▼     ▼                     ▼                            │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                   │
│  │ Fun-ASR  │  │Interview │  │ 豆包 TTS │                   │
│  │ (阿里云)  │  │  Agent   │  │ (字节)   │                   │
│  │          │  │          │  │          │                   │
│  │ 语音→文字│  │ 文字→文字 │  │ 文字→语音 │                   │
│  └──────────┘  └──────────┘  └──────────┘                   │
└──────────────────────────────────────────────────────────────┘
```

---

## 二、三阶段流水线

### 2.1 阶段一：ASR（语音转文字）

```python
# 使用阿里云 DashScope Fun-ASR 实时语音识别

async def process_audio_stream(websocket):
    """接收浏览器麦克风数据，实时转写"""
    
    # ① 建立 Fun-ASR WebSocket 连接
    asr_client = await create_funasr_connection()
    
    # ② 接收浏览器音频数据
    async for message in websocket:
        if message["type"] == "audio":
            # ③ 转发给 ASR 服务
            await asr_client.send(message["audio_data"])
            
            # ④ 获取实时转写结果
            result = await asr_client.receive()
            # result.text: "我觉得微服务架构的核心优势在于..."
            # result.is_final: True/False（句子是否结束）
            
            if result.is_final:
                # ⑤ 最终修正文案提交给 Agent
                await submit_to_agent(result.text)
```

### 2.2 VAD（语音活动检测）断句

```python
# 关键参数：连续静音时长
SILENCE_THRESHOLD_MS = 3000  # 3秒无声触发断句

# Fun-ASR 的 VAD 工作原理：
# 1. 持续接收音频流
# 2. 检测到语音 → 开始转写
# 3. 检测到连续 3 秒无声 → 判定句子结束 → is_final=True
# 4. 将最终文案提交给 Agent

# 为什么面试场景用 3 秒？
# - 太短 (1s)：思考停顿被误判为句子结束，频繁打断
# - 太长 (5s)：用户等太久，体验差
# - 3 秒：中文技术面试的最佳平衡点
```

### 2.3 阶段二：Agent 推理

```python
async def submit_to_agent(text: str):
    """将转写文本提交给 InterviewAgent"""
    
    # 复用现有的对话逻辑
    async for response_chunk in agent.stream_messages(
        messages=[HumanMessage(content=text)],
        context=interview_context,
    ):
        # 积累完整回复
        full_response.append(response_chunk)
    
    # 将完整回复发送给 TTS
    await synthesize_speech("".join(full_response))
```

### 2.4 阶段三：TTS（文字转语音）

```python
# 使用字节跳动豆包 TTS（双向流式 WebSocket）

async def synthesize_speech(text: str):
    """将 Agent 回复转为语音"""
    
    # 豆包 TTS 配置（在 prompt 中已说明固定音色）
    tts_config = {
        "app_id": DOUBAO_VOICE_APP_ID,
        "api_key": DOUBAO_VOICE_API_KEY,
        "speaker": "zh_male_m191_uranus_bigtts",   # 男声
        "resource_id": "seed-tts-2.0",              # TTS 模型版本
    }
    
    # 流式合成：逐句生成音频，边生成边推送
    async for audio_chunk in doubao_tts.stream_synthesize(
        text=text,
        speaker=tts_config["speaker"],
    ):
        # 立即推送给浏览器播放
        await websocket.send({
            "type": "audio",
            "data": audio_chunk,
        })
```

---

## 三、关键设计决策

### 3.1 为什么用 WebSocket 而不是 WebRTC？

| 特性 | WebSocket | WebRTC |
|------|-----------|--------|
| 部署复杂度 | ✅ 简单 | ❌ 需要 STUN/TURN |
| 防火墙穿透 | ✅ HTTP升级 | ❌ 经常被阻 |
| 延迟 | 中等 (TCP) | 低 (UDP) |
| 面试场景适合度 | ✅ 延迟可接受 | ❌ 过度设计 |

> **结论**：面试场景中，几百毫秒的延迟是可接受的。WebSocket 的简单性和兼容性远优于 WebRTC。

### 3.2 为什么不直接在 Agent 循环中调用 TTS？

```
❌ 错误方案：
Agent 每生成一个 token → TTS 合成 → 播放
问题：token 级别太碎，TTS 合成质量差，语音不连贯

✅ 正确方案：
Agent 生成完整句子 → TTS 逐句合成 → 逐句播放
优点：保持语音的自然停顿和语调
```

### 3.3 音色组合的固定性

```python
# 伯乐当前固定音色组合：
speaker = "zh_male_m191_uranus_bigtts"
resource_id = "seed-tts-2.0"

# 为什么固定？
# 这组 speaker 和 resource_id 必须匹配
# 使用不兼容的 speaker/resource_id 组合会导致：
# → WebSocket 可以建立
# → 但不会返回可播放的音频分片（静默失败！）
```

---

## 四、前端音频处理

### 4.1 AudioContext 恢复

```javascript
// 浏览器自动播放策略要求用户交互后才能播放音频
let audioContext = null;

async function startVoiceInterview() {
    // ① 用户点击按钮触发
    audioContext = new (window.AudioContext || window.webkitAudioContext)();
    
    // ② 恢复音频上下文（必须由用户手势触发）
    await audioContext.resume();
    
    // ③ 请求麦克风权限
    const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
    
    // ④ 建立 WebSocket 连接
    connectWebSocket(stream);
}
```

### 4.2 音频缓冲与播放

```javascript
// 低延迟音频播放策略
const BUFFER_THRESHOLD = 2;  // 缓冲2个chunk后开始播放
let audioBuffer = [];

ws.onmessage = (event) => {
    const audioChunk = event.data;  // PCM 数据
    
    audioBuffer.push(audioChunk);
    
    if (audioBuffer.length >= BUFFER_THRESHOLD) {
        // 开始播放
        while (audioBuffer.length > 0) {
            const chunk = audioBuffer.shift();
            playAudioChunk(chunk);
        }
    }
};

function playAudioChunk(pcmData) {
    const buffer = audioContext.createBuffer(1, pcmData.length, 16000);
    buffer.getChannelData(0).set(pcmData);
    
    const source = audioContext.createBufferSource();
    source.buffer = buffer;
    source.connect(audioContext.destination);
    source.start();
}
```

---

## 五、生产环境注意事项

详见 [[09-deployment-and-production#五、WebSocket 语音面试的生产问题|部署与生产落地实战 — WebSocket 语音面试]]

核心要点：
- 配置 Nginx 支持 WebSocket 升级（`proxy_set_header Upgrade $http_upgrade`）
- WebSocket 心跳保活（30秒 ping/pong）
- Fun-ASR 和豆包 TTS 的 API key 不提交到 Git
- 浏览器 AudioContext 必须由用户手势触发
- VAD 断句阈值按面试场景调优（默认 3 秒）

---

## 下一步学习

阅读 [[08-frontend-architecture|前端架构深度剖析]]，理解 Vue3 + Pinia + Ant Design 的协作模式。
