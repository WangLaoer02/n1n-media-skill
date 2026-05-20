---
name: github-managed-repos
description: 管理王东悦的 GitHub 仓库信息、凭证和常见错误规范。当涉及 GitHub 操作（push/pull/clone/查看仓库/上传文件）或需要查找 GitHub 相关配置时触发。
---

# GitHub 仓库管理

## 🔑 凭证信息

| 项目 | 内容 |
|------|------|
| **用户名** | WangLaoer02 |
| **Token** | `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` |
| **Remote URL** | `https://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@github.com/WangLaoer02/n1n-media-skill.git` |

**Token 权限**：私人库（private repo），已测试可正常 push/pull。

---

## 📁 已管理的仓库

### n1n-media-skill
- **URL**：`https://github.com/WangLaoer02/n1n-media-skill`
- **路径**：`~/.openclaw/workspace/skills/n1n-media/`
- **内容**：
  ```
  skills/
  ├── huoying-asset-manager/
  │   └── SKILL.md
  └── n1n-media/
      ├── SKILL.md          ← 核心技能文件
      ├── README.md         ← 安装说明（供其他 Agent 学习）
      └── references/
          ├── control-ui-skill-copy.txt
          └── models.md
  ```
- **安装命令**：
  ```bash
  git clone https://github.com/WangLaoer02/n1n-media-skill.git \
    ~/.openclaw/workspace/skills/n1n-media
  ```

---

## ⚠️ OpenClaw Agent 常见错误（必须熟记）

### 错误1：inbound 目录被清空
**问题**：用户发送的图片进入 `inbound/` 目录，但处理完后会被清空。如果在后续步骤（如子 Agent 执行时）才去读图，会发现文件不存在。

**正确做法**：收到用户图片后**立即**复制到永久目录，不要依赖后续再从 inbound 读取。

```bash
# 立即复制到永久目录
cp <inbound路径> /Users/yue/.openclaw/media/tool-image-generation/n1n-ref-<timestamp>.png
```

---

### 错误2：SubAgent 是 isolated 模式，看不到父 session 的文件
**问题**：用 `sessions_spawn(context: "isolated")` 启动的子 Agent 完全隔离，读不到父 session 的任何文件（包括图片、配置等）。子 Agent 可能会自己去乱找文件作为参考。

**正确做法**：
- 把图片路径**明确写入**子 Agent 的 task，而不是让它自己找
- 把图片 base64 **直接内嵌**在 task 里传给子 Agent
- 或者先把图片复制到子 Agent 能访问的固定路径

---

### 错误3：MEDIA 路径不在 webchat 允许列表
**问题**：webchat 只认 `/Users/yue/.openclaw/media/tool-image-generation/` 目录下的文件作为图片附件。存到 `/tmp` 或其他目录会导致图片显示 "Unavailable / Outside allowed folders"。

**正确做法**：
- 所有生成的图片**必须**保存到 `/Users/yue/.openclaw/media/tool-image-generation/`
- 发送时 `MEDIA:` 单独一行，**不带任何前缀文字**

```markdown
✅ 正确：
MEDIA:/Users/yue/.openclaw/media/tool-image-generation/xxx.png

❌ 错误（哥们收不到）：
这是一张图
MEDIA:/Users/yue/.openclaw/media/tool-image-generation/xxx.png

❌ 错误（路径不在允许列表）：
MEDIA:/tmp/xxx.png
```

---

### 错误4：GitHub 仓库有重复文件
**问题**：不同时间通过不同方式（直接 push / n1n-media.skill 包 / skills/ 子目录）上传到 GitHub，根目录可能留下旧文件和 `skills/` 下的新文件重复，造成冲突和混淆。

**正确做法**：
- 定期检查 `https://api.github.com/repos/WangLaoer02/<repo>/git/trees/HEAD?recursive=1` 确认仓库只有一套内容
- 删除重复的旧文件（用 GitHub API DELETE 或 git filter-branch）
- 保持 `skills/<skill-name>/SKILL.md` 是唯一权威文件

---

### 错误5：SubAgent 图生图 task 写法不规范
**问题**：子 Agent 的 task 里没有写明具体的参考图路径，让它自己找 → 它会乱找。

**正确的 task 写法**：
```
参考图路径（必须是实际存在的文件）：
- 图1主体：/Users/yue/.openclaw/media/tool-image-generation/n1n-ref-xxx.png
- 图2 UI：/Users/yue/.openclaw/media/tool-image-generation/n1n-ref-yyy.png

生成要求：保留图1的黄色外卖车主体，将UI布局改为图2风格...

保存路径：/Users/yue/.openclaw/media/tool-image-generation/result.png
```

---

## 🛠️ GitHub 常用操作

### 推送到 GitHub
```bash
cd /Users/yue/.openclaw/workspace/skills/<skill-name>
git add <files>
git commit -m "<message>"
git push  # remote 已配置好 token，无需手动输入
```

### 查看仓库结构
```bash
curl -s "https://api.github.com/repos/WangLaoer02/<repo>/git/trees/HEAD?recursive=1" \
  | python3 -c "import json,sys; d=json.load(sys.stdin); [print(f['path']) for f in sorted(d.get('tree',[]),key=lambda x:x['path'])]"
```

### 查看远程仓库 token
```bash
git remote -v
```

### 从远程拉取最新
```bash
cd /Users/yue/.openclaw/workspace/skills/<skill-name>
git pull --rebase origin main && git push
```

---

## 📌 MEMORY.md 更新习惯

遇到以下情况后**必须**更新 MEMORY.md：
1. 新增了 GitHub 仓库 → 记录仓库名、路径、内容
2. 新增了 API token → 记录分组和用途
3. 犯了某个错误并解决 → 把原因和解决方案记到本 skill 的"常见错误"部分
4. 某个 skill 的目录结构变了 → 同步更新本 skill 的"已管理仓库"部分