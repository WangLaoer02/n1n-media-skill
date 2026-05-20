# n1n-media Skill

> AI 图片生成技能 — 通过 n1n API 调用 Gemini 生成图片，兼容图生图（img2img）。
> 另一个 OpenClaw Agent 可通过安装此 Skill 来获得图片生成能力。

---

## 🗂️ 技能信息

| 项目 | 内容 |
|------|------|
| **名称** | `n1n-media` |
| **描述** | 通过 n1n API 生成图片。触发词：生成图片/生图/画画/画图，或发送图片并提出修改要求 |
| **模型** | `gemini-3.1-flash-image-preview` |
| **能力** | 文生图、图生图 |
| **仓库** | https://github.com/WangLaoer02/n1n-media-skill |

---

## 📦 安装方式

在已配置 n1n API key 的 OpenClaw 工作目录下执行：

```bash
# 方式一：通过 npx 安装（推荐）
npx skills add n1n-media --global -g

# 方式二：直接 clone 仓库
git clone https://github.com/WangLaoer02/n1n-media-skill.git \
  ~/.openclaw/workspace/skills/n1n-media
```

安装后 OpenClaw 会自动识别 `SKILL.md`，下次提到"生成图片/生图/画画"即触发此 Skill。

---

## 🔧 核心配置

### 存储路径
```
/Users/yue/.openclaw/media/tool-image-generation/
```

### API Keys（请替换为你的 keys）

| Key | 分组 | 说明 |
|-----|------|------|
| `sk-你的默认key` | 默认 | 先手，先用这个 |
| `sk-你的备用key` | 备手 | 默认失败后换用 |

### 网络（Mac 必须）

n1n API 直连不通，需走本地代理：

```bash
-x http://127.0.0.1:26136 --http1.1
```

- **不加代理** → exit code 28 超时
- **不加 `--http1.1`** → SSL_ERROR_SYSCALL exit code 35

### Base URL

```
https://api.n1n.ai/v1beta/models/gemini-3.1-flash-image-preview:generateContent
```

---

## 📝 使用方法

### 触发方式

| 用户说法 | 触发场景 |
|----------|----------|
| "生成图片"、"生图"、"画画"、"画图" | 文生图 |
| "替换角色"、"改成XX"、"换个风格"（发送图片） | 图生图 |

### 标准流程（Agent 必须遵循）

1. **取参考图**：从当前消息的 inbound 目录读取图片路径
2. **组装请求**：用户提示词原封不动填入 `text`；参考图 base64 放 `inlineData`
3. **发请求**：默认 key + 代理 + `--http1.1`
4. **保存图片**：解码 response 的 `inlineData`，写入 `tool-image-generation/` 目录
5. **汇报**：格式必须包含"模型/分组/尺寸/文件"，并附 `MEDIA:` 行

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

### curl 调用示例

```bash
curl -s --max-time 180 \
  -x http://127.0.0.1:26136 \
  -X POST "https://api.n1n.ai/v1beta/models/gemini-3.1-flash-image-preview:generateContent" \
  -H "Authorization: Bearer sk-你的key" \
  -H "Content-Type: application/json" \
  -d @/tmp/img_req.json \
  -o /tmp/img_resp.json \
  --http1.1
```

### 保存图片（Python）

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

### 汇报格式模板

```
🖼️ 图片生成完成
- 模型：gemini-3.1-flash-image-preview
- 分组：<分组>
- 尺寸：9:16
- 文件：<文件名>
```
然后附上 `MEDIA:` 行。

---

## ⚠️ 错误处理规则

```
默认 key（走代理）→ 失败 → 换备用 key（走代理）→ 均失败 → 报错结束
```

| 错误类型 | 判断方法 | 处理 |
|----------|----------|------|
| 超时/直连不通 | exit code 28 | 确认代理是否开启 |
| SSL 错误 | exit code 35 | 需加 `--http1.1` |
| 额度欠费 | HTTP 403 + `"code":"local:insufficient_quota"` | 换备用 key |
| 其他失败 | HTTP 200 但 candidates 为空 | 报错结束 |

**核心原则**：不换模型，不改用户提示词，只换 key 或确认网络。

---

## 🔑 配置自己的 API Keys

Skill 使用环境变量或配置文件管理 keys。要使用自己的 key，修改 SKILL.md 中的 API Keys 表格（直接写入完整 key，不要用占位符）。

如需多套 key 切换，可在 Skill 内置 key 失效时自动切换，详见 SKILL.md 的错误处理级联部分。

---

## 📁 目录结构

```
n1n-media/
├── SKILL.md            # 核心技能文件（必须）
└── references/          # 参考文档（可选）
    ├── control-ui-skill-copy.txt
    └── models.md
```