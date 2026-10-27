# 在 Dify 里使用 ModelFold

[English](README.md)

ModelFold 用一套 OpenAI 风格的 API 提供对话、图像、视频和音频模型。在 Dify 里把它加为 **OpenAI-API-compatible**（兼容 OpenAI API）模型供应商后，聊天应用就可以用你自己的 API Key 调用 ModelFold 的对话模型。

站点：https://ai.modelfold.online

## 1. 准备 API Key

1. 在 https://ai.modelfold.online 注册。
2. 打开账户里的 API Keys：https://ai.modelfold.online/account
3. 新建一把 Key。外形是 `mf-` 加一段随机字符。只复制一次，自己保管。本仓库里没有任何真实 Key。

新账号会收到一笔注册赠送积分（站点默认 100，以到账金额为准）。每次调用成功都会扣积分。上游模型失败时，这一笔会退回。

## 2. 接口地址

| | |
| --- | --- |
| API base URL | `https://ai.modelfold.online/v1` |
| 对话 | `POST /v1/chat/completions` |
| 模型列表 | `GET /v1/models` |
| 图像 | `POST /v1/images/generations` |
| 鉴权 | 请求头 `Authorization: Bearer <你的 Key>` |

Dify 会自己在地址后面接 `/chat/completions`。表单里只填 base URL，不要把 `/chat/completions` 也贴进去。

这条对话接口只认 `model`、`messages`、`max_tokens`（默认 1024）和 `temperature`。它始终返回一整份 JSON（`object` 为 `chat.completion`），不返回 SSE 流。`messages` 里不能带图片，也不支持 tools / function call。把这些开关都关掉。

```bash
export MODELFOLD_API_KEY="你的Key"
curl -sS https://ai.modelfold.online/v1/models \
  -H "Authorization: Bearer $MODELFOLD_API_KEY"

curl -sS https://ai.modelfold.online/v1/chat/completions \
  -H "Authorization: Bearer $MODELFOLD_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"gpt-6-luna","messages":[{"role":"user","content":"Reply with the single word pong."}],"max_tokens":32}'
```

模型名必须是 `GET /v1/models` 里 `modality` 为 `chat` 的 id。把图像模型 id 发到对话接口会返回 `400 unsupported_model`。

## 3. 安装供应商

Dify 云版（https://cloud.dify.ai）和自部署是同一套界面。自部署的机器需要能访问 `https://ai.modelfold.online`。

1. 进入工作区的 **设置 → 模型供应商**。较新的界面也可能写成 **集成 → 模型供应商**。需要工作区所有者或管理员。
2. 列表里没有 **OpenAI-API-compatible** 时，到插件市场安装。发布者是 `langgenius`，插件 id 是 `openai_api_compatible`。
3. 在这张卡片上点 **添加模型**。

## 4. 每个字段填什么

每个模型 id 加一行。建议先加 `gpt-6-luna`（下面价目表里最便宜的对话模型）。

| 字段 | 填写 |
| --- | --- |
| 模型类型 | LLM |
| 模型名称 | `gpt-6-luna`（与 id 完全一致） |
| 模型显示名称 | 可选，例如 `GPT-6 Luna (ModelFold)` |
| API Key | 只贴 Key，不要带 `Bearer` |
| API Base URL | `https://ai.modelfold.online/v1` |
| API endpoint 中的模型名称 | 留空 |
| 对话类型（Completion mode） | 对话（Chat） |
| 模型上下文长度 | `128000` |
| 最大 token 上限 | `4096` |
| 函数调用类型 | 不支持 |
| 流式函数调用 | 不支持 |
| 视觉支持 | 不支持 |
| 视频 / 音频 / 文档支持 | 不支持 |
| 思考模式 | 不支持 |
| 联网搜索 | 不支持 |
| 结构化输出 | 不支持 |
| 兼容模式 | 严格（OpenAI 兼容） |
| API 类型 | Chat Completions API（`/chat/completions`） |
| token 参数名 | 使用 max_tokens |
| 流式中包含用量 | 禁用 |
| Stream mode auth | Not use（不使用） |
| 用户标识 | 不支持 |

说明：

- **模型上下文长度**只给 Dify 裁历史用。`GET /v1/models` 不返回上下文窗口。填 `128000` 作为工作上限即可，填小一点也可以。
- **最大 token 上限**是应用里 Max Tokens 滑杆的上限。ModelFold 会按 `max_tokens` 预扣积分。新账号建议把应用里的 Max Tokens 设成 512 或 1024，避免滑杆过大、模型还没跑就返回 `402`。
- **token 参数名**要选 **使用 max_tokens**。留在「自动」时，以 `gpt-5`、`gpt-6`、`o1`、`o3` 开头的模型名会让插件改发 `max_completion_tokens`。ModelFold 只读 `max_tokens`，另一个字段会被忽略（请求仍会成功，但按默认 1024 计）。
- **API 类型**必须是 Chat Completions。Responses API 会去调 `/v1/responses`，这个 base URL 没有该路径。
- **函数调用和视觉**关掉，因为这条对话接口没有实现。不要把模型用在需要工具的 Agent 里，也不要在对话里上传图片。

保存时，插件会发一条不走流式的短请求 `ping`（Stream mode auth 为 Not use）。这和 ModelFold 的实现一致。保存失败多半是地址或 Key，见 [排查](#6-常见报错)。

要在选择器里出现更多模型，就重复添加。Key 和地址不变，只改模型名称。

### 对话模型 id

2026-10-27 对照 `GET /v1/models` 核对，共 35 个。价格是同一天文档里的「每百万 token 积分」。目录会变，依赖某个名字之前请再拉一次列表。模型页：`https://ai.modelfold.online/models/<id>`。

| 模型 id | 输入 / 百万 | 输出 / 百万 |
| --- | ---: | ---: |
| `claude-fable-5` | 1200 | 6000 |
| `claude-haiku-4-5` | 82.5 | 427.5 |
| `claude-haiku-5-5` | 18 | 90 |
| `claude-opus-4-5` | 427.5 | 2145 |
| `claude-opus-4-6` | 427.5 | 2145 |
| `claude-opus-4-7` | 427.5 | 2145 |
| `claude-opus-4-8` | 600 | 3000 |
| `claude-opus-5` | 600 | 3000 |
| `claude-opus-5-5` | 480 | 2400 |
| `claude-sonnet-4-5` | 255 | 1282.5 |
| `claude-sonnet-4-6` | 255 | 1282.5 |
| `claude-sonnet-5` | 255 | 1282.5 |
| `claude-sonnet-5-5` | 240 | 1200 |
| `deepseek-v4-1-flash` | 36 | 142.5 |
| `gemini-3-pro` | 150 | 1050 |
| `gemini-3.1-pro` | 150 | 1050 |
| `gemini-3-6-flash` | 67.5 | 337.5 |
| `gemini-3-6-flash-openai` | 67.5 | 337.5 |
| `gemini-3-7-flash` | 67.5 | 337.5 |
| `gemini-3-7-flash-openai` | 67.5 | 337.5 |
| `gemini-3-8-flash` | 67.5 | 337.5 |
| `gemini-3-8-flash-openai` | 67.5 | 337.5 |
| `gemini-4-argon` | 240 | 1200 |
| `gemini-4-argon-openai` | 240 | 1200 |
| `gpt-5-2` | 131.256 | 1050 |
| `gpt-5-6-luna` | 16.8 | 100.8 |
| `gpt-5-6-sol` | 420 | 2520 |
| `gpt-5-6-terra` | 168 | 1008 |
| `gpt-6-1-sol` | 180 | 900 |
| `gpt-6-astra` | 840 | 4200 |
| `gpt-6-luna` | 9 | 45 |
| `gpt-6-sol` | 180 | 900 |
| `grok-4-6` | 240 | 720 |
| `grok-4-7` | 240 | 720 |
| `kimi-k3` | 720 | 3600 |

### 图像模型走另一个地址

OpenAI-API-compatible 插件只能加 LLM、向量、重排序、语音转文字和文字转语音。它没有图像生成类型。ModelFold 的 `/v1` 也不提供向量、重排序和语音。不要把图像 id 加成型 LLM。

图像生成是 `POST https://ai.modelfold.online/v1/images/generations`，正文里给 `model`、`prompt`，可选 `resolution` 或 `image_url`。2026-10-27 线上有 64 个图像模型，例如：

| 模型 id | 起价积分 |
| --- | ---: |
| `google/imagen4-fast` | 6 |
| `google/imagen4` | 12 |
| `gpt-image/1.5-text-to-image` | 6 |
| `flux-2/pro-text-to-image` | 8 |

在 Dify 里用 **HTTP 请求**节点调用：POST、JSON 正文、请求头 `Authorization: Bearer <Key>`，读取超时设 120 秒。成功时返回 `{ "ok": true, "data": [{ "url": "https://..." }] }`。

## 5. 在应用里选用

### 聊天助手

1. 工作室 → 从空白创建 → **聊天助手**。
2. 提示词面板上的模型选择器里选 **OpenAI-API-compatible / gpt-6-luna**（或你填的显示名）。
3. Max Tokens 设为 `512`。
4. 发布后打开预览，发一句短问题。

### 工作流里的 LLM 节点

1. 工作室 → 从空白创建 → **工作流**。
2. 在开始和结束之间加一个 **LLM** 节点。
3. 选同一个 ModelFold 模型。用户提示词引用开始节点的问题变量。视觉保持关闭。
4. 用一句短问题试跑。

也可以导入 [`examples/modelfold-llm.workflow.yml`](examples/modelfold-llm.workflow.yml)（工作室 → 导入 DSL）。这个 LLM 节点要求供应商里已经有一个名称正好是 `gpt-6-luna` 的模型。如果节点显示模型缺失，点开重新选一次。文件里没有 API Key。

### 不依赖流式输出的工作流

聊天助手和 LLM 节点运行时，Dify 通常会让插件走流式。ModelFold 回的是一整份 JSON，所以模型能保存成功，对话仍可能报解析错误。下面这个 HTTP 工作流就是接口实际接受的调用：一次 POST，不要求流。

1. 导入 [`examples/modelfold-hello.workflow.yml`](examples/modelfold-hello.workflow.yml)。
2. 打开应用的环境变量，把 `MODELFOLD_API_KEY` 设成你的 Key。
3. 运行。输入是 `question`，输出是 `status_code` 和 `body`。回答文字在 `body` 的 `choices[0].message.content`。

要换模型，改 JSON 正文里的 `gpt-6-luna`。

## 6. 常见报错

| 现象 | 处理 |
| --- | --- |
| `401` / unauthorized | Key 缺失、已吊销，或粘进了多余文字。只贴 Key。HTTP 示例里的请求头必须是 `Bearer`、一个空格、再加 Key。 |
| `404`，地址里出现 `/v1/v1` | 插件又加了一次 `/v1`。API Base URL 改成 `https://ai.modelfold.online`。 |
| `404`，地址是 `/chat/completions` 但没有 `/v1` | 旧版插件拼接时把 `v1` 弄丢了。API Base URL 改成 `https://ai.modelfold.online/v1/`（末尾保留斜杠）。 |
| `404`，地址末尾是 `/chat/completions/chat/completions` | 把完整路径贴进了 base URL。只留 `https://ai.modelfold.online/v1`。 |
| `400` `unsupported_model` | 名字不是对话模型 id，或把图像 id 用在了对话接口上。从 `GET /v1/models` 复制 id。 |
| `402` 积分不足 | 余额不够预扣。把 Max Tokens 调小，或到 https://ai.modelfold.online/account 充值。 |
| `429` | 同一账号每分钟 60 次。等一会儿再试。 |
| 保存成功，聊天或 LLM 节点报流式解析错误 | 和当前对话接口一致：Dify 发了 `stream: true`，ModelFold 回了一份 JSON。改用 [`examples/modelfold-hello.workflow.yml`](examples/modelfold-hello.workflow.yml)。流式中包含用量保持禁用，Stream mode auth 保持 Not use。 |
| Agent 的工具不执行 | 这条接口不支持函数调用。纯文本用工作流 LLM 或 HTTP 节点。 |
| `gpt-5*` / `gpt-6*` 回答特别长或特别短 | token 参数名还在「自动」，插件发了 `max_completion_tokens`。改成 **使用 max_tokens** 后重新保存。 |

网关还有这些错误：`invalid_messages`（400，`messages` 为空）、`upstream_failed`（502，已退款）、`upstream_unavailable`（503）。

## 7. 计费

对话先按 `max_tokens` 预扣，再按真实 token 结算：积分 = 向上取整(（输入 token × 输入单价 + 输出 token × 输出单价）/ 1,000,000)。图像和视频按次计费。价格和积分套餐见 https://ai.modelfold.online。`gpt-6-luna` 的一句短回复经常只扣 1 积分。
