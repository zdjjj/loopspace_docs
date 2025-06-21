# API 集成指南

## 概述

本指南将帮助开发者了解如何与 Loopspace 平台进行深度集成。

## 认证机制

### OAuth 2.0 认证

Loopspace 使用标准的 OAuth 2.0 协议进行用户认证：

```javascript
// 获取访问令牌
const tokenResponse = await fetch("/api/oauth/token", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    grant_type: "authorization_code",
    client_id: "your_client_id",
    client_secret: "your_client_secret",
    code: "authorization_code_from_redirect",
    redirect_uri: "your_redirect_uri",
  }),
});
```

### API 密钥认证

对于服务器到服务器的调用，可以使用 API 密钥：

```javascript
const headers = {
  Authorization: "Bearer your_api_key",
  "Content-Type": "application/json",
};
```

## 核心 API 端点

### 音频管理

#### 上传音频文件

```javascript
POST /api/v1/audio/upload

// 表单数据
FormData:
- file: 音频文件 (mp3, wav, flac)
- title: 音频标题
- description: 描述信息
- tags: 标签数组
```

#### 获取音频信息

```javascript
GET /api/v1/audio/{audioId}

Response:
{
  "id": "audio_123",
  "title": "示例音频",
  "duration": 180,
  "waveform": [...],
  "metadata": {
    "bitrate": 320,
    "sampleRate": 44100
  }
}
```

### 用户管理

#### 获取用户资料

```javascript
GET /api/v1/users/{userId}

Response:
{
  "id": "user_456",
  "username": "artist_name",
  "avatar": "https://...",
  "stats": {
    "followersCount": 1000,
    "tracksCount": 25
  }
}
```

## Webhook 集成

### 配置 Webhook

在开发者控制台中配置 Webhook URL：

```json
{
  "url": "https://your-server.com/webhook",
  "events": ["audio.uploaded", "user.followed", "purchase.completed"],
  "secret": "your_webhook_secret"
}
```

### 处理 Webhook 事件

```javascript
app.post("/webhook", (req, res) => {
  const signature = req.headers["x-loopspace-signature"];
  const payload = req.body;

  // 验证签名
  const expectedSignature = crypto
    .createHmac("sha256", process.env.WEBHOOK_SECRET)
    .update(JSON.stringify(payload))
    .digest("hex");

  if (signature === expectedSignature) {
    // 处理事件
    handleWebhookEvent(payload);
    res.status(200).send("OK");
  } else {
    res.status(401).send("Unauthorized");
  }
});
```

## SDK 使用

### JavaScript SDK

```bash
npm install @loopspace/sdk
```

```javascript
import { LoopspaceClient } from "@loopspace/sdk";

const client = new LoopspaceClient({
  apiKey: "your_api_key",
  environment: "production", // or 'sandbox'
});

// 上传音频
const audio = await client.audio.upload({
  file: audioFile,
  title: "我的新作品",
  tags: ["electronic", "chill"],
});

// 获取用户数据
const user = await client.users.get("user_id");
```

### Python SDK

```bash
pip install loopspace-python
```

```python
from loopspace import Client

client = Client(api_key='your_api_key')

# 获取热门音频
popular_tracks = client.audio.get_popular(limit=20)

# 搜索音频
search_results = client.audio.search(
    query='电子音乐',
    filters={'genre': 'electronic'}
)
```

## 最佳实践

### 错误处理

```javascript
try {
  const result = await client.audio.upload(audioData);
} catch (error) {
  if (error.code === "RATE_LIMIT_EXCEEDED") {
    // 处理限流
    await new Promise((resolve) =>
      setTimeout(resolve, error.retryAfter * 1000)
    );
    // 重试
  } else if (error.code === "FILE_TOO_LARGE") {
    // 文件太大，进行压缩
    const compressedFile = await compressAudio(audioData.file);
    return client.audio.upload({ ...audioData, file: compressedFile });
  }
}
```

### 分页处理

```javascript
// 获取所有用户的音频
async function getAllUserTracks(userId) {
  let allTracks = [];
  let page = 1;
  let hasMore = true;

  while (hasMore) {
    const response = await client.users.getTracks(userId, {
      page,
      limit: 50,
    });

    allTracks = allTracks.concat(response.data);
    hasMore = response.hasMore;
    page++;
  }

  return allTracks;
}
```

### 缓存策略

```javascript
// 使用 Redis 缓存用户数据
const redis = require("redis");
const client = redis.createClient();

async function getCachedUser(userId) {
  const cached = await client.get(`user:${userId}`);
  if (cached) {
    return JSON.parse(cached);
  }

  const user = await loopspaceClient.users.get(userId);
  await client.setex(`user:${userId}`, 300, JSON.stringify(user)); // 5分钟缓存

  return user;
}
```

## 常见问题

### Q: API 调用频率限制是多少？

A: 免费版每分钟 60 次，专业版每分钟 1000 次。

### Q: 支持的音频格式有哪些？

A: 支持 MP3、WAV、FLAC、AAC 格式，最大文件大小 100MB。

### Q: 如何处理大文件上传？

A: 使用分片上传功能，将大文件分割成小块逐个上传。

## 联系支持

如有技术问题，请联系：

- 📧 Email: dev@loopspace.com
- 💬 Discord: [开发者社区](https://discord.gg/loopspace)
- 📚 文档: [完整 API 文档](https://docs.loopspace.com)
