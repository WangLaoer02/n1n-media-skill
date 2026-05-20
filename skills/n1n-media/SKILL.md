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
| `sk-8f5bM2Pp0vfVhUCp0VlbqKIgTbleNYagbRjAAZ6maOF15nlT` | 备手 | 默认失败后换用 |

## 网络问题排查记录

**Mac 直连 api.n1n.ai 不通**（timeout），原因：本地代理（CMYNetwork）接管了 443 出口流量。

**解决方案**：所有 curl 请求必须加代理和 `--http1.1`：
```
-x http://127.0.0.1:26136 --http1.1
```

不加代理 → exit code 28 超时
不加 `--http1.1` → SSL_ERROR_SYSCALL（exit code 35）

## Base URL

```
https://api.n1n.ai/v1beta/models/gemini-3.1-flash-image-preview:generateContent
```

---

## 🖼️ 图片生成（图生图）

### 流程

1. **取参考图**：从当前消息的 inbound 目录读取图片路径，不用老图
2. **组装请求**：用户提示词 1:1 填入 text，不改写；参考图 base64 放 inlineData
3. **发请求**：默认 key + 代理 + `--http1.1`；收到 403/quota 失败时换备用 key
4. **保存并汇报**

### 请求体格式

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

### curl 调用（Mac 必须加代理）

```bash
curl -s --max-time 180 -x http://127.0.0.1:26136 -X POST \
  "https://api.n1n.ai/v1beta/models/gemini-3.1-flash-image-preview:generateContent" \
  -H "Authorization: Bearer sk-C0tPF9DXyc6MZkqVXNaVR4IgMkWIl5EaP9zG6ppLme9cBj2F" \
  -H "Content-Type: application/json" \
  -d @/tmp/img_req.json \
  -o /tmp/img_resp.json \
  --http1.1
```

### 保存图片

```python
python3 -c "
import json, base64, time
d = json.load(open('/tmp/img_resp.json'))
for p in d['candidates'][0]['content']['parts']:
    if 'inlineData' in p:
        img = base64.b64decode(p['inlineData']['data'])
        path = f'/Users/yue/.openclaw/media/tool-image-generation/n1n-{int(time.time())}.png'
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
然后附上 `MEDIA:` 行。若中途换了 key 必须注明。

---

## 错误处理（级联）

```
默认 key（走代理）→ 失败 → 换备用 key（走代理）→ 均失败 → 报错结束
```

错误类型判断：
- `exit code 28` = 超时 / 直连不通 → 确认代理是否开启
- `exit code 35` = SSL 错误 → 需加 `--http1.1`
- HTTP 403 + `"code":"local:insufficient_quota"` = 额度欠费
- HTTP 200 但 candidates 为空 = 可能是别的问题

**不换其他模型**，只换 key 或确认网络配置。