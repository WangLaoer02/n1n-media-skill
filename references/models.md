# N1N 模型参考

## n1n API Key
`sk-C0tPF9DXyc6MZkqVXNaVR4IgMkWIl5EaP9zG6ppLme9cBj2F`
Base URL: `https://api.n1n.ai`

---

## 图片模型（按类型分类）

### Gemini 系列（原生格式）
| 模型 | 端点 | 调用方式 | 状态 |
|------|------|----------|------|
| `gemini-3.1-flash-image-preview` | `/v1beta/models/...:generateContent` | curl + base64 解码 | ✅ 已测试 |
| `gemini-3-pro-image-preview` | `/v1beta/models/...:generateContent` | curl + base64 解码 | 待测 |

### Grok 系列（OpenAI 兼容格式）
| 模型 | 端点 | 调用方式 | 状态 |
|------|------|----------|------|
| `grok-imagine-image` | `/v1/images/generations` | curl + URL 下载 | ✅ 已测试 |
| `grok-imagine-image-pro` | `/v1/images/generations` | curl + URL 下载 | 待测 |
| `grok-3-image` | `/v1/images/generations` | curl + URL 下载 | 待测 |
| `grok-4-image` | `/v1/images/generations` | curl + URL 下载 | 待测 |
| `grok-4.1-image` | `/v1/images/generations` | curl + URL 下载 | 待测 |
| `grok-4.2-image` | `/v1/images/generations` | curl + URL 下载 | 待测 |

### FLUX 系列（OpenAI 兼容格式）
| 模型 | 端点 | 调用方式 | 状态 |
|------|------|----------|------|
| `flux-schnell` | `/v1/images/generations` | curl + URL 下载 | ⚠️ 上游可能饱和 |
| `flux-dev` | `/v1/images/generations` | curl + URL 下载 | ⚠️ 上游可能饱和 |
| `flux-pro` | `/v1/images/generations` | curl + URL 下载 | 待测 |
| `flux-2-pro` | `/v1/images/generations` | curl + URL 下载 | 待测 |
| `flux-2-flex` | `/v1/images/generations` | curl + URL 下载 | 待测 |
| `flux-pro-1.1-ultra` | `/v1/images/generations` | curl + URL 下载 | 待测 |
| `flux.1.1-pro` | `/v1/images/generations` | curl + URL 下载 | 待测 |

### OpenAI GPT-Image 系列（OpenAI 兼容格式）
| 模型 | 端点 | 调用方式 | 状态 |
|------|------|----------|------|
| `gpt-image-2` | `/v1/images/generations` | curl + URL 下载 | 待测 |
| `gpt-image-1.5` | `/v1/images/generations` | curl + URL 下载 | 待测 |
| `gpt-image-1` | `/v1/images/generations` | curl + URL 下载 | 待测 |
| `gpt-image-1-mini` | `/v1/images/generations` | curl + URL 下载 | 待测 |

### 阿里 Qwen 系列
| 模型 | 端点 | 调用方式 | 状态 |
|------|------|----------|------|
| `qwen-image-max` | `/v1/images/generations` | curl + URL 下载 | 待测 |
| `qwen-image-max-2025-12-30` | `/v1/images/generations` | curl + URL 下载 | 待测 |
| `qwen-image-edit-2509` | `/v1/images/generations` | curl + URL 下载 | 待测 |

### 字节 Kling 系列
| 模型 | 端点 | 调用方式 | 状态 |
|------|------|----------|------|
| `kling-image` | `/v1/images/generations` | curl + URL 下载 | 待测 |
| `kling-omni-image` | `/v1/images/generations` | curl + URL 下载 | 待测 |

### 其他
| 模型 | 端点 | 调用方式 | 状态 |
|------|------|----------|------|
| `dall-e-3` | `/v1/images/generations` | curl + URL 下载 | ⚠️ 上游可能饱和 |
| `z-image-turbo` | `/v1/images/generations` | curl + URL 下载 | 待测 |

### OpenClaw 原生工具（已配置）
| 模型 | 调用方式 | 状态 |
|------|----------|------|
| `MiniMax image-01` | `image_generate` 工具 | ✅ 已配置 |

---

## 视频模型（按类型分类）

### OpenClaw 原生工具（已配置）
| 模型 | 调用方式 | 状态 |
|------|----------|------|
| `MiniMax-Hailuo-2.3` | `video_generate` 工具 | ✅ 已配置 |
| `MiniMax-Hailuo-02` | `video_generate` 工具 | 待测 |

### n1n API 视频模型
| 模型 | 端点 | seconds 参数 | 状态 |
|------|------|-------------|------|
| `sora-2` | `/v1/videos` | 4/8/12 | ⚠️ 上游可能饱和 |
| `sora-2-pro-all` | `/v1/videos` | 待测 | 待测 |
| `veo3` | `/v1/videos` | 待测 | 待测 |
| `veo3.1` | `/v1/videos` | 待测 | 待测 |
| `veo3.1-pro` | `/v1/videos` | 待测 | 待测 |
| `veo3.1-fast` | `/v1/videos` | 待测 | 待测 |
| `veo3.1-4k` | `/v1/videos` | 待测 | 待测 |
| `veo3.1-pro-4k` | `/v1/videos` | 待测 | 待测 |
| `veo_3_1` | `/v1/videos` | 待测 | 待测 |
| `veo_3_1-lite` | `/v1/videos` | 待测 | 待测 |
| `kling-video` | `/v1/videos` | 待测 | 待测（需特殊auth） |
| `kling-omni-video` | `/v1/videos` | 待测 | 待测 |
| `wan2.5-i2v-preview` | `/v1/videos` | 待测 | 待测 |
| `wan2.6-i2v` | `/v1/videos` | 待测 | 待测 |
| `runwayml-gen4_turbo-10` | `/v1/videos` | 待测 | 待测 |
| `runwayml-gen4_turbo-5` | `/v1/videos` | 待测 | 待测 |
| `grok-video-3` | `/v1/videos` | 10（字符串） | 待测 |
| `grok-video-3-10s` | `/v1/videos` | 待测 | 待测 |

---

## 常用调用模板

### 图片 - Gemini 原生（默认）
```bash
curl -s -X POST "https://api.n1n.ai/v1beta/models/gemini-3.1-flash-image-preview:generateContent" \
  -H "Authorization: Bearer sk-C0tPF9DXyc6MZkqVXNaVR4IgMkWIl5EaP9zG6ppLme9cBj2F" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{"parts": [{"text": "PROMPT"}]}],
    "generationConfig": {
      "responseModalities": ["TEXT", "IMAGE"],
      "imageConfig": {"aspectRatio": "1:1"}
    }
  }' | python3 -c "
import json,sys,base64,time
d=json.load(sys.stdin)
for p in d['candidates'][0]['content']['parts']:
    if 'inlineData' in p:
        img=base64.b64decode(p['inlineData']['data'])
        open(f'/Users/yue/.openclaw/media/tool-image-generation/gemini-{int(time.time())}.png','wb').write(img)
        print(len(img),'bytes saved')
"
```

### 图片 - Grok（备选稳定）
```bash
curl -s -X POST "https://api.n1n.ai/v1/images/generations" \
  -H "Authorization: Bearer sk-C0tPF9DXyc6MZkqVXNaVR4IgMkWIl5EaP9zG6ppLme9cBj2F" \
  -H "Content-Type: application/json" \
  -d '{"model": "grok-imagine-image", "prompt": "PROMPT", "n": 1, "size": "1024x1024"}' \
  | python3 -c "
import json,sys,time,urllib.request
d=json.load(sys.stdin)
url=d['data'][0]['url']
urllib.request.urlretrieve(url, f'/Users/yue/.openclaw/media/tool-image-generation/grok-{int(time.time())}.png')
print('saved')
"
```

### 视频 - sora-2
```bash
curl -s -X POST "https://api.n1n.ai/v1/videos" \
  -H "Authorization: Bearer sk-C0tPF9DXyc6MZkqVXNaVR4IgMkWIl5EaP9zG6ppLme9cBj2F" \
  -H "Content-Type: application/json" \
  -d '{"model": "sora-2", "prompt": "PROMPT", "seconds": 4}'
```

### 视频 - grok-video-3（seconds 必须是字符串）
```bash
curl -s -X POST "https://api.n1n.ai/v1/videos" \
  -H "Authorization: Bearer sk-C0tPF9DXyc6MZkqVXNaVR4IgMkWIl5EaP9zG6ppLme9cBj2F" \
  -H "Content-Type: application/json" \
  -d '{"model": "grok-video-3", "prompt": "PROMPT", "seconds": "10"}'
```