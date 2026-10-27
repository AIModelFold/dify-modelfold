# Use ModelFold in Dify

[中文](README.zh-CN.md)

ModelFold is one OpenAI-style API for chat, image, video, and audio models. In Dify, add it as the **OpenAI-API-compatible** model provider and your chat apps can call ModelFold models with your own API key.

Site: https://ai.modelfold.online

This guide matches the official `langgenius/openai_api_compatible` plugin schema and a live call to `https://ai.modelfold.online/v1` on 2026-10-27. It was **not** clicked through inside Dify Cloud or a local Docker Dify (see [What was checked](#what-was-checked)).

## 1. Create an API key

1. Sign up at https://ai.modelfold.online.
2. Open **Account → API Keys**: https://ai.modelfold.online/account
3. Create a key. It looks like `mf-` followed by a random string. Copy it once and store it yourself. This repository never contains a real key.

New accounts receive signup credits (the site default is 100; the amount that lands in the wallet is what you can spend). Every successful call deducts credits. If the upstream model fails, ModelFold refunds that call.

## 2. What to point Dify at

| | |
| --- | --- |
| API base URL | `https://ai.modelfold.online/v1` |
| Chat | `POST /v1/chat/completions` |
| List models | `GET /v1/models` |
| Images | `POST /v1/images/generations` |
| Auth | `Authorization: Bearer <your key>` |

Dify appends `/chat/completions` itself. Put the base URL only. Do not paste `/chat/completions` into the form.

Chat on this base URL accepts `model`, `messages`, `max_tokens` (default 1024), and `temperature`. It always returns one JSON object (`object: chat.completion`). It does not emit server-sent events. It does not accept image parts inside `messages`, tools / function calls, or `stream: true` as a real stream. Leave those switches off.

```bash
export MODELFOLD_API_KEY="your-key"
curl -sS https://ai.modelfold.online/v1/models \
  -H "Authorization: Bearer $MODELFOLD_API_KEY"

curl -sS https://ai.modelfold.online/v1/chat/completions \
  -H "Authorization: Bearer $MODELFOLD_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"gpt-6-luna","messages":[{"role":"user","content":"Reply with the single word pong."}],"max_tokens":32}'
```

Use a chat model id from `GET /v1/models` where `modality` is `chat`. An image id on this endpoint returns `400 unsupported_model`.

## 3. Install the provider

Same steps on **Dify Cloud** (https://cloud.dify.ai) and on a **self-hosted** Dify. A self-hosted instance only needs outbound HTTPS to `ai.modelfold.online`.

1. In the workspace, open **Settings → Model Provider**. Some current builds label this **Integrations → Model Provider**. You need owner or admin on the workspace.
2. If **OpenAI-API-compatible** is not on the page, install it from the plugin marketplace. The publisher is `langgenius`, plugin id `openai_api_compatible`.
3. On that card, choose **Add Model**.

## 4. Fill in the model

Add one row per model id. Suggested first model: `gpt-6-luna` (lowest chat price in the catalog below).

| Field | Value |
| --- | --- |
| Model type | LLM |
| Model name | `gpt-6-luna` (exact id) |
| Model display name | optional, for example `GPT-6 Luna (ModelFold)` |
| API Key | the key only, without the word `Bearer` |
| API Base URL | `https://ai.modelfold.online/v1` |
| model name for API endpoint | leave empty |
| Completion mode | Chat |
| Model context size | `128000` |
| Upper bound for max tokens | `4096` |
| Function Call Type | Not Support |
| Stream function calling | Not Support |
| Vision Support | Not Support |
| Video / Audio / Document Support | Not Support |
| Thinking Mode Support | Not supported |
| Web Search Support | Not supported |
| Structured Output | Not supported |
| Compatibility mode | OpenAI compatible (strict) |
| API Type | Chat Completions API (`/chat/completions`) |
| Token parameter name | Use max_tokens |
| Include Usage in Stream | Disabled |
| Stream mode auth | Not use |
| User Identity Support | Not Support |

Notes on those fields:

- **Model context size** is only Dify's history budget. `GET /v1/models` does not return a context window. `128000` is a working cap so Dify trims long chats. A smaller number is fine.
- **Upper bound for max tokens** is the highest value the app's Max Tokens slider can reach. ModelFold pre-authorizes credits from `max_tokens`. On a new account, keep the app slider at 512 or 1024 so a large slider does not return `402` before the model runs.
- **Token parameter name** must be **Use max_tokens**. On Auto, model names that start with `gpt-5`, `gpt-6`, `o1`, or `o3` make this plugin send `max_completion_tokens`. ModelFold reads `max_tokens` and ignores the other name (the call still runs, at the default of 1024).
- **API Type** must stay on Chat Completions. Responses API calls `/v1/responses`, which this base URL does not serve.
- **Function calling and vision** are off because this chat endpoint does not implement them. Do not use the model in an Agent app that expects tools, and do not attach images to the chat.

Save. The plugin's save check sends a short non-streaming `ping` (Stream mode auth = Not use). That check matches what ModelFold implements. A failed save is almost always the URL or the key; see [Troubleshooting](#6-troubleshooting).

Repeat Add Model for each id you want in the picker. The key and base URL are the same; only the model name changes.

### Chat model ids

Checked against `GET /v1/models` on 2026-10-27 (35 chat models). Prices are credits per 1,000,000 tokens from the ModelFold docs the same day. The catalog changes; list models again before you depend on a name. Model pages: `https://ai.modelfold.online/models/<id>`.

| Model id | Input / 1M | Output / 1M |
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

### Image models are a different endpoint

The OpenAI-API-compatible plugin can add LLM, embeddings, rerank, speech-to-text, and text-to-speech. It has no image-generation type, and ModelFold does not serve embeddings, rerank, or speech on `/v1`. Do not add image ids as LLM models.

Image generation is `POST https://ai.modelfold.online/v1/images/generations` with `model`, `prompt`, and optional `resolution` or `image_url`. On 2026-10-27 the live list had 64 image models. Examples:

| Model id | Starting credits |
| --- | ---: |
| `google/imagen4-fast` | 6 |
| `google/imagen4` | 12 |
| `gpt-image/1.5-text-to-image` | 6 |
| `flux-2/pro-text-to-image` | 8 |

Call that URL from a Dify **HTTP Request** node (POST, JSON body, header `Authorization: Bearer <key>`, read timeout 120 seconds). The response shape is `{ "ok": true, "data": [{ "url": "https://..." }] }`.

## 5. Use the model

### Chat assistant

1. Studio → Create from blank → **Chatbot**.
2. In the model picker on the prompt panel, choose **OpenAI-API-compatible / gpt-6-luna** (or the display name you set).
3. Set Max Tokens to `512`.
4. Publish, open Preview, and send a short question.

### Workflow LLM node

1. Studio → Create from blank → **Workflow**.
2. Put an **LLM** node between Start and End.
3. Select the same ModelFold model. User prompt: the start-node question variable. Leave vision off.
4. Run with a short question.

You can import [`examples/modelfold-llm.workflow.yml`](examples/modelfold-llm.workflow.yml) (Studio → Import DSL). The LLM node expects a provider model whose name is exactly `gpt-6-luna`. If the node shows the model as missing, open it and select the model you added. The file has no API key.

### Workflow that does not depend on streaming

Dify usually asks the provider plugin to stream when a chat or an LLM node runs. ModelFold answers with one JSON body, so that path can fail after the model saves successfully. The HTTP workflow below is the call that matches the API: one POST, no stream.

1. Import [`examples/modelfold-hello.workflow.yml`](examples/modelfold-hello.workflow.yml).
2. Open the app → environment variables → set `MODELFOLD_API_KEY` to your key.
3. Run. Input is `question`. Outputs are `status_code` and `body`. The assistant text is `choices[0].message.content` inside `body`.

To change models, edit the JSON body and replace `gpt-6-luna` with another chat id.

## 6. Troubleshooting

| What you see | What to change |
| --- | --- |
| `401` / unauthorized | The key is missing, revoked, or has extra text. Paste the key only. In the HTTP example the header must be `Bearer` plus a space plus the key. |
| `404`, URL contains `/v1/v1` | The plugin added `/v1` again. Set API Base URL to `https://ai.modelfold.online`. |
| `404`, URL is `/chat/completions` with no `/v1` | An older plugin joined the path and dropped `v1`. Set API Base URL to `https://ai.modelfold.online/v1/` (trailing slash). |
| `404`, URL ends with `/chat/completions/chat/completions` | You pasted the full path. Use only `https://ai.modelfold.online/v1`. |
| `400` `unsupported_model` | The model name is not a chat id, or an image id was used on the chat endpoint. Copy the id from `GET /v1/models`. |
| `402` insufficient credits | The wallet cannot cover the pre-authorization. Lower Max Tokens or add credits at https://ai.modelfold.online/account. |
| `429` | 60 requests per minute for the account. Wait and retry. |
| Save works, chat or LLM node errors while parsing a stream | Expected with today's chat API: Dify sent `stream: true`, ModelFold returned one JSON object. Use [`examples/modelfold-hello.workflow.yml`](examples/modelfold-hello.workflow.yml). Keep Include Usage in Stream = Disabled and Stream mode auth = Not use. |
| Agent tools never run | Function calling is not supported on this endpoint. Use a workflow LLM or HTTP node for plain text. |
| Empty or very long answers on `gpt-5*` / `gpt-6*` | Token parameter name is still Auto, so the plugin sent `max_completion_tokens`. Set it to **Use max_tokens**, then save the model again. |

Other error types from the gateway: `invalid_messages` (400, empty `messages`), `upstream_failed` (502, refunded), `upstream_unavailable` (503).

## 7. Billing

Chat is settled on real token counts after a pre-authorization: `credits = ceil((input tokens × input price + output tokens × output price) / 1,000,000)`. Image and video calls charge per generation. Prices and the credit packs are on https://ai.modelfold.online. A short `gpt-6-luna` reply often costs 1 credit.
