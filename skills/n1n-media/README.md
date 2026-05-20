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

**所有生成图片必须保存到此路径**，webchat 只认此目录下的文件作为 MEDIA 附件。

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

### 完整工作流（Agent 必须按顺序执行）

#### 第一步：收到图片时立即保存

用户发送图片后，**立即**复制到永久目录，防止 inbound 清空丢失：

```bash
cp <inbound路径> /Users/yue/.openclaw/media/tool-image-generation/n1n-ref-<timestamp>.png
```

**⚠️ inbound 是临时目录，消息处理完会被清空，绝不能依赖后续读取！**

#### 第二步：组装请求

- 用户提示词原封不动填入 `text`，不改写
- 参考图 base64 直接写入请求的 `inlineData`，**不要让子 Agent 自己读文件**（子 Agent 是 isolated 模式，看不到父 session 的文件）

#### 第三步：发请求

默认 key + 代理 + `--http1.1`，收到 403/quota 失败时换备用 key。

#### 第四步：保存结果

**必须**保存到 `tool-image-generation/` 目录。

#### 第五步：汇报格式

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
生成要求：保留图1的主体，将UI布局改为图2风格...

参考图路径（必须是实际存在的文件）：
- 图1主体：/Users/yue/.openclaw/media/tool-image-generation/n1n-ref-xxx.png
- 图2 UI：/Users/yue/.openclaw/media/tool-image-generation/n1n-ref-yyy.png

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

## 📁 目录结构

```
n1n-media/
├── SKILL.md            # 核心技能文件（必须）
├── README.md           # 本文档
└── references/          # 参考文档（可选）
    ├── control-ui-skill-copy.txt
    └── models.md
```