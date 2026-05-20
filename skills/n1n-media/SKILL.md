---
name: n1n-media
description: 通过 n1n API 生成图片。触发：用户说"生成图片/生图/画画/画图"，或发送图片并提出修改要求（"替换角色"/"改成XX"）。使用 Gemini（gemini-3.1-flash-image-preview）图生图。
---

# N1N Media Generator

**核心规则**：不自主降级，不自行换模型，不改写用户提示词。

## 存储路径
`/Users/yue/.openclaw/media/tool-image-generation/`

**所有生成图片必须保存到此路径**，webchat 只认此目录下的文件作为 MEDIA 附件。

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

- 不加代理 → exit code 28 超时
- 不加 `--http1.1` → SSL_ERROR_SYSCALL（exit code 35）

## Base URL

```
https://api.n1n.ai/v1beta/models/gemini-3.1-flash-image-preview:generateContent
```

---

## 🖼️ 图片生成（图生图）

### 完整工作流（必须按顺序执行）

#### 第一步：收到图片时立即保存到永久目录

用户发送图片后，**立即**复制到永久目录，防止 inbound 清空丢失：

```bash
# 从 inbound 读取用户发送的图（路径在消息 metadata 中）
# 复制到永久目录，文件名用 message_id 或 timestamp 区分
cp <inbound路径> /Users/yue/.openclaw/media/tool-image-generation/n1n-ref-<timestamp>.png
```

**⚠️ inbound 是临时目录，消息处理完会被清空，绝不能依赖后续读取！**

#### 第二步：组装请求

- 用户提示词 1:1 填入 `text`，不改写
- 参考图 base64 直接写入请求的 `inlineData`，**不要让子 Agent 自己读文件**（子 Agent 是 isolated 模式，看不到父 session 的文件）

请求体格式：
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

#### 第三步：发请求

- 默认 key + 代理 + `--http1.1`
- 收到 403/quota 失败时换备用 key

#### 第四步：保存结果

图片**必须**保存到允许目录：
```python
python3 -c "
import json, base64, time, os
d = json.load(open('/tmp/img_resp.json'))
for p in d['candidates'][0]['content']['parts']:
    if 'inlineData' in p:
        img = base64.b64decode(p['inlineData']['data'])
        # ⚠️ 必须存到 tool-image-generation/
        path = '/Users/yue/.openclaw/media/tool-image-generation/n1n-{int(time.time())}.png'
        open(path, 'wb').write(img)
        print('saved', len(img), 'bytes to', path)
"
```

#### 第五步：汇报（MEDIA 格式）

```
🖼️ 图片生成完成
- 模型：gemini-3.1-flash-image-preview
- 分组：<分组>
- 尺寸：9:16
- 文件：<文件名>
```

**然后单独一行发送 MEDIA（不要加任何前缀文字）：**
```
MEDIA:/Users/yue/.openclaw/media/tool-image-generation/xxx.png
```

---

## ⚠️ SubAgent 图生图任务规范

当需要用子 Agent 并行生成多张图时：

### 正确的 task 写法

```
生成要求：保留图1的黄色外卖车主体，将UI布局改为图2风格...

参考图路径（必须是实际存在的文件）：
- 图1主体：/Users/yue/.openclaw/media/tool-image-generation/n1n-ref-xxx.png
- 图2 UI：/Users/yue/.openclaw/media/tool-image-generation/n1n-ref-yyy.png

生成要求：...

保存路径：/Users/yue/.openclaw/media/tool-image-generation/result.png
```

### 禁止行为

- ❌ 不要让子 Agent 自己从 inbound 读图（已清空）
- ❌ 不要让子 Agent 自己从 Downloads 找图（会乱找）
- ❌ 不要让子 Agent 自己找参考图路径
- ❌ 不要假设 isolated 子 Agent 能访问父 session 的任何文件
- ❌ 不要把图片存到 `/tmp`（webchat 不认）

### 正确做法

- ✅ 把图片 base64 **直接内嵌在 task 里**传给子 Agent
- ✅ 或把图片复制到 `tool-image-generation/` 后把**绝对路径**写进 task
- ✅ 结果图片路径明确指定为 `tool-image-generation/xxx.png`

---

## 错误处理（级联）

```
默认 key（走代理）→ 失败 → 换备用 key（走代理）→ 均失败 → 报错结束
```

| 错误类型 | 判断方法 | 处理 |
|----------|----------|------|
| 超时/直连不通 | exit code 28 | 确认代理是否开启 |
| SSL 错误 | exit code 35 | 需加 `--http1.1` |
| 额度欠费 | HTTP 403 + `"code":"local:insufficient_quota"` | 换备用 key |
| 其他失败 | HTTP 200 但 candidates 为空 | 报错结束 |

**不换其他模型**，只换 key 或确认网络配置。