---
name: n1n-media
description: 通过 n1n API 生成图片。触发：用户说"生成图片/生图/画画/画图"，或发送图片并提出修改要求（"替换角色"/"改成XX"）。使用 Gemini（gemini-3.1-flash-image-preview）图生图。
---

# N1N Media Generator

**核心规则**：不自主降级，不自行换模型，不改写用户提示词。

## 存储路径
`/Users/yue/.openclaw/media/tool-image-generation/`

## API Keys（默认优先）

| Key | 分组 | 说明 |
|-----|------|------|
| `sk-C0tPF9DXyc6MZkqVXNaVR4IgMkWIl5EaP9zG6ppLme9cBj2F` | 默认 | 先手，先用这个 |
| `sk-8f5bM2Pp0vfVhUCp0VlbqKIgTbleNYagbRjAAZ6maOF15nlT` | 优质Gemini | 备手，默认失败后换用 |

## Base URL（默认优先）

先用 `https://api.n1n.ai/v1`，收到 404/错误时才换：
1. `https://api.n1n.ai/v1`（默认）
2. `https://api.n1n.ai/v1/chat/completions`
3. `https://api.n1n.ai`

---

## 🖼️ 图片生成

### 流程

1. **取参考图**：取当前消息里的图片，不用老图
2. **组装请求**：用户提示词 1:1 填入 text，不改写
3. **发请求**：默认 key + 默认 Base URL；收到失败响应时依次切换备选
4. **保存并汇报**

### 图生图请求体

```json
{
  "contents": [{"parts": [
    {"text": "<用户原始提示词>"},
    {"inlineData": {"mimeType": "image/png", "data": "<图片base64>"}}
  ]}],
  "generationConfig": {
    "responseModalities": ["TEXT", "IMAGE"],
    "imageConfig": {"aspectRatio": "9:16"}
  }
}
```

**curl 调用**：
```bash
curl -s -X POST "https://api.n1n.ai/v1beta/models/gemini-3.1-flash-image-preview:generateContent" \
  -H "Authorization: Bearer sk-C0tPF9DXyc6MZkqVXNaVR4IgMkWIl5EaP9zG6ppLme9cBj2F" \
  -H "Content-Type: application/json" \
  -d @/tmp/img_req.json \
  -o /tmp/img_resp.json \
  --max-time 120
```

**保存图片**：
```python
python3 -c "
import json, base64, time
d = json.load(open('/tmp/img_resp.json'))
for p in d['candidates'][0]['content']['parts']:
    if 'inlineData' in p:
        img = base64.b64decode(p['inlineData']['data'])
        path = f'/Users/yue/.openclaw/media/tool-image-generation/gemini-{int(time.time())}.png'
        open(path, 'wb').write(img)
        print('saved', len(img), 'bytes to', path)
"
```

### 汇报格式

```
🖼️ 图片生成完成
- 模型：gemini-3.1-flash-image-preview
- 分组：<分组>
- 尺寸：9:16
- 文件：<文件名>
```
然后附上 `MEDIA:` 行。若中途换了 key 或 Base URL 必须注明。

---

## 错误处理（级联）

```
默认 key + 默认 URL → 失败 → 换优质Gemini key → 失败 → 换 Base URL → 失败 → 报错结束
```

收到 HTTP 错误 / 超时 / 404 时才切换，不提前预检。切换顺序：
1. 换优质Gemini key（同 URL）
2. 换 Base URL（用当前 key）
3. 均失败则报错告知用户，**不换其他模型**