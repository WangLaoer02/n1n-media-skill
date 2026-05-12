---
name: n1n-media
description: N1N API 生图生视频技能 - 通过 n1n API 统一调用所有图像和视频生成模型。当用户想要生成图片、生成视频、图生视频、clip generation、image generation、video generation、画图、画画、生成画作，或任何涉及 AI 图像/视频创作的需求时，自动触发此技能。支持 Gemini、Grok、FLUX、Veo、Sora 等模型。图片默认 gemini-3.1-flash-image-preview，视频默认 veo3。
---

# N1N Media Generator

## 存储路径

- 图片：`/Users/yue/.openclaw/media/tool-image-generation/`
- 视频：`/Users/yue/.openclaw/media/tool-video-generation/`

## API Keys

| 分组 | Key | 用途 |
|------|-----|------|
| **默认分组** | `sk-C0tPF9DXyc6MZkqVXNaVR4IgMkWIl5EaP9zG6ppLme9cBj2F` | 默认生图、生视频 |
| **优质Gemini** | `sk-8f5bM2Pp0vfVhUCp0VlbqKIgTbleNYagbRjAAZ6maOF15nlT` | 快速模式（仅 Gemini） |

**Base URL**：`https://api.n1n.ai`

---

## 工作流程（精简版）

**Step 1 — 判断任务类型：**

| 用户说 | 任务类型 | 默认模型 |
|--------|----------|----------|
| 生图/画画/生成图片 | 🖼️ 图片 | `gemini-3.1-flash-image-preview` |
| 生视频/视频生成 | 🎬 视频 | `veo3` |

**Step 2 — 判断分组：**
- 用户说"快"/"着急"/"快速版" → 优质Gemini分组 key ⚡
- 其他 → 默认分组 key

**Step 3 — 确认模型（回复中告知用户）：**
> 即将用 `gemini-3.1-flash-image-preview`（**默认分组**）生成图片

**Step 4 — 执行生成（见下方详细调用）**

**Step 5 — 展示结果：**
```
🖼️ 图片生成完成
- 模型：gemini-3.1-flash-image-preview
- 分组：默认分组
- 尺寸：1:1
- 文件：/Users/yue/.openclaw/media/tool-image-generation/...
```
提供选项：**重新生成 / 调整提示词 / 换模型 / 换尺寸**

---

## 🖼️ 图片生成

### 默认：gemini-3.1-flash-image-preview

**调用（curl）：**
```bash
curl -s -X POST "https://api.n1n.ai/v1beta/models/gemini-3.1-flash-image-preview:generateContent" \
  -H "Authorization: Bearer sk-C0tPF9DXyc6MZkqVXNaVR4IgMkWIl5EaP9zG6ppLme9cBj2F" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{"parts": [{"text": "提示词"}]}],
    "generationConfig": {
      "responseModalities": ["TEXT", "IMAGE"],
      "imageConfig": {"aspectRatio": "1:1"}
    }
  }' > /tmp/img.json

# 保存图片
python3 -c "
import json,base64,time
d=json.load(open('/tmp/img.json'))
for p in d['candidates'][0]['content']['parts']:
    if 'inlineData' in p:
        img=base64.b64decode(p['inlineData']['data'])
        open(f'/Users/yue/.openclaw/media/tool-image-generation/gemini-{int(time.time())}.png','wb').write(img)
        print('saved', len(img), 'bytes')
"
```

**宽高比**：`1:1`（默认）/ `16:9` / `9:16` / `3:2` / `2:3`

### ⚡ 快速模式：gemini-3.1-flash-image-preview（优质Gemini分组）

端点完全相同，只需换 key：
```
-H "Authorization: Bearer sk-8f5bM2Pp0vfVhUCp0VlbqKIgTbleNYagbRjAAZ6maOF15nlT"
```
**注意**：优质Gemini分组**仅支持 Gemini**，不支持 grok/FLUX/GPT-Image。

### 备选：grok-imagine-image（默认分组，稳定快速）

当 Gemini 上游饱和时使用，返回 URL 直接下载，速度快。

**调用：**
```bash
curl -s -X POST "https://api.n1n.ai/v1/images/generations" \
  -H "Authorization: Bearer sk-C0tPF9DXyc6MZkqVXNaVR4IgMkWIl5EaP9zG6ppLme9cBj2F" \
  -H "Content-Type: application/json" \
  -d '{"model": "grok-imagine-image", "prompt": "提示词", "n": 1, "size": "1024x1024"}' \
  | python3 -c "
import json,sys,time,urllib.request
d=json.load(sys.stdin)
urllib.request.urlretrieve(d['data'][0]['url'],
  f'/Users/yue/.openclaw/media/tool-image-generation/grok-{int(time.time())}.png')
print('saved')
"
```

**已知问题（不推荐）：**
- `gpt-image-2`：OpenAI 上游极慢/饱和，响应超时，不可用
- `flux-schnell`/`dall-e-3`：上游负载饱和，不可用

---

## 🎬 视频生成

### 默认：veo3（n1n API，form-data）

**提交任务：**
```bash
curl -s -X POST "https://api.n1n.ai/v1/videos" \
  -H "Authorization: Bearer sk-C0tPF9DXyc6MZkqVXNaVR4IgMkWIl5EaP9zG6ppLme9cBj2F" \
  -F "model=veo3" \
  -F "prompt=提示词" \
  -F "seconds=5"
```

**轮询状态（等待完成）：**
```bash
curl -s -X GET "https://api.n1n.ai/v1/videos/{task_id}" \
  -H "Authorization: Bearer sk-C0tPF9DXyc6MZkqVXNaVR4IgMkWIl5EaP9zG6ppLme9cBj2F"
```
返回 `status: completed` 时，`result_url` 即为视频地址。

**下载保存：**
```bash
curl -L -o "/Users/yue/.openclaw/media/tool-video-generation/veo3.mp4" "{result_url}"
```

**支持时长**：4s / 5s / 6s / 8s（默认 5s）
**注意**：必须用 `-F` form-data 格式，不能用 JSON。

### 备选：sora-2-all

当 veo3 不可用时使用。**仅支持 10s 或 15s**。

```bash
curl -s -X POST "https://api.n1n.ai/v1/videos" \
  -H "Authorization: Bearer sk-C0tPF9DXyc6MZkqVXNaVR4IgMkWIl5EaP9zG6ppLme9cBj2F" \
  -H "Content-Type: application/json" \
  -d '{"model": "sora-2-all", "prompt": "提示词", "seconds": 10}'
```

### 备选：MiniMax-Hailuo-2.3（OpenClaw 工具）

当用户明确要求时使用，支持 6s/10s。
**注意**：额度用完返回 `usage limit exceeded`，下周一重置。

---

## 弯路记录（经验总结）

1. **GPT-Image-2 不可用**：OpenAI 上游极慢，所有分组均超时，不要尝试
2. **FLUX/dall-e-3 上游饱和**：默认分组频繁报负载饱和，不要优先尝试
3. **MiniMax 视频额度有限**：周限额，容易耗尽，veo3 更稳定
4. **Claude Code 无法调用**：需要独立登录，与本地 key 不通用
5. **grok-video-3 无可用渠道**：n1n 平台暂无可用渠道，不要尝试
6. **视频 duration 要匹配模型**：veo3 支持 4/5/6/8s，sora-2-all 只支持 10/15s，不要混用时长参数

## 注意

- Gemini 返回 base64，需解码保存；grok 返回 URL，直接下载
- 上游饱和 → 自动换备选模型，不要卡死重试
- 优质Gemini分组 key **不适用于 grok/FLUX/GPT-Image**，只能用 Gemini
- 视频轮询间隔建议 15~30s，避免过度请求