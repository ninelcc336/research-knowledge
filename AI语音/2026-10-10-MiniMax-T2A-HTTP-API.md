---
title: "MiniMax T2A HTTP 语音合成 API 接入与计费调研"
date: 2026-10-10
category: "AI语音"
tags:
  - MiniMax
  - TTS
  - 语音合成
  - API
  - Skill
  - CLI
  - MCP
  - 计费
---

# MiniMax T2A HTTP 语音合成 API 接入与计费调研

## 1. 调研范围与结论

- 调研日期：2026-10-10
- 主要接口：[同步语音合成 HTTP API](https://platform.minimax.cn/docs/api-reference/speech-t2a-http)
- 相关产品页：[MiniMax Audio 文字转语音](https://www.minimax.cn/audio/text-to-speech)
- 相关官方仓库：[MiniMax-AI/cli](https://github.com/MiniMax-AI/cli)
- 相关官方文档：[MiniMax MCP 使用指南](https://platform.minimax.cn/docs/guides/mcp-guide)

**一句话结论：** MiniMax T2A HTTP API 是一个使用 Bearer API Key、向 `POST /v1/t2a_v2` 发送 JSON 的同步 TTS 接口，支持非流式和流式输出、40 种语言、系统/复刻/设计音色以及较丰富的音频控制参数。生产接入应使用开放平台的普通按量 API Key、语音资源包，或在确认资源覆盖范围后使用 M Plan 订阅 Key；MiniMax Audio 网页每月的“1000 声贝”不能依据现有官方资料视为 API 免费额度。

“CRI”不是 MiniMax 当前官方接入资料中的名称。结合官方资料，用户所说的“Skill 或 CRI”很可能是“Skill 或 CLI”。本报告因此完整说明官方 CLI + Skill，并补充官方 MCP 方案；如果 CRI 指某个特定运行时或内部协议，需要再按该协议的定义单独适配。

## 2. API 基本信息

### 2.1 接入地址

| 场景 | 地址 |
| --- | --- |
| 默认 HTTP 地址 | `https://api.minimax.cn/v1/t2a_v2` |
| 华北建议地址 | `https://api-bj.minimaxi.com/v1/t2a_v2` |
| WebSocket 流式地址 | `wss://api.minimax.cn/ws/v1/t2a_v2` |
| 华北 WebSocket 地址 | `wss://api-bj.minimaxi.com/ws/v1/t2a_v2` |

华北地区官方建议使用 `api-bj.minimaxi.com`，主要是首包延迟更低。HTTP API 与 WebSocket API 的鉴权方式相同，但协议和请求格式不同，不应混用示例。

### 2.2 鉴权与请求头

在开放平台的“账户管理 > 接口密钥”创建或查看 API Key，请求头如下：

```http
Authorization: Bearer <MINIMAX_API_KEY>
Content-Type: application/json
```

API Key 只应放在服务端、CLI 环境变量或 MCP 服务进程环境变量中，不应写入浏览器前端、移动端包或提交到代码仓库。

### 2.3 支持模型

当前文档列出的同步 T2A 模型包括：

- `speech-2.8-hd`
- `speech-2.8-turbo`
- `speech-2.6-hd`
- `speech-2.6-turbo`
- `speech-02-hd`
- `speech-02-turbo`
- `speech-01-hd`
- `speech-01-turbo`

通常可按以下思路选择：

- **HD**：优先音质、韵律和情绪表现。
- **Turbo**：优先速度和成本，适合实时对话、批量播报和成本敏感场景。
- 新接入优先从 `speech-2.8-hd` 或 `speech-2.8-turbo` 开始，再用实际文本和目标语言做 A/B 测试。

## 3. HTTP 请求与响应

### 3.1 最小请求

```bash
curl --request POST \
  --url https://api.minimax.cn/v1/t2a_v2 \
  --header "Authorization: Bearer ${MINIMAX_API_KEY}" \
  --header "Content-Type: application/json" \
  --data '{
    "model": "speech-2.8-turbo",
    "text": "欢迎使用 MiniMax 语音合成。",
    "stream": false,
    "voice_setting": {
      "voice_id": "male-qn-qingse",
      "speed": 1,
      "vol": 1,
      "pitch": 0
    },
    "audio_setting": {
      "sample_rate": 32000,
      "bitrate": 128000,
      "format": "mp3",
      "channel": 1
    }
  }'
```

非流式成功响应的关键结构如下：

```json
{
  "data": {
    "audio": "<hex encoded audio>",
    "status": 2
  },
  "extra_info": {
    "audio_length": 3200,
    "audio_sample_rate": 32000,
    "audio_size": 51200,
    "bitrate": 128000,
    "audio_format": "mp3",
    "audio_channel": 1,
    "usage_characters": 18
  },
  "trace_id": "...",
  "base_resp": {
    "status_code": 0,
    "status_msg": "success"
  }
}
```

`data.audio` 默认是十六进制字符串。客户端需要把它解码为二进制后按请求的格式写入文件；如果非流式请求设置 `output_format` 为 `url`，则返回音频 URL，官方文档说明该 URL 有效期为 24 小时。生产系统应优先下载并持久化文件，不要长期保存临时 URL。

### 3.2 主要字段

| 字段 | 说明 |
| --- | --- |
| `model` | 模型 ID，必填 |
| `text` | 待合成文本，小于 10,000 字符；超过 3,000 字符建议流式输出 |
| `stream` | 是否流式返回，默认 `false` |
| `voice_setting.voice_id` | 系统音色、复刻音色或文生音色 ID |
| `voice_setting.speed` | 语速，`0.5` 到 `2` |
| `voice_setting.vol` | 音量，`0` 到 `10` 的开区间/闭区间范围，即大于 0 且不超过 10 |
| `voice_setting.pitch` | 语调，`-12` 到 `12` |
| `voice_setting.emotion` | `happy`、`sad`、`angry`、`fearful`、`disgusted`、`surprised`、`calm`、`fluent`、`whisper`，具体模型有差异 |
| `audio_setting.format` | `mp3`、`pcm`、`flac`、`wav`、`pcmu_raw`、`pcmu_wav`、`opus` |
| `audio_setting.sample_rate` | 常用值包括 8k、16k、22.05k、24k、32k、44.1k |
| `audio_setting.bitrate` | MP3 比特率，可选 32k、64k、128k、256k |
| `audio_setting.channel` | `1` 单声道或 `2` 双声道 |
| `language_boost` | 语言/方言增强，也可使用 `auto` |
| `pronunciation_dict.tone` | 自定义发音，例如 `处理/(chu3)(li3)` |
| `timbre_weights` | 最多混合 4 个音色，每个音色配置权重 |
| `subtitle_enable` | 开启字幕信息 |
| `subtitle_type` | `sentence`、`word` 或流式适用的 `word_streaming` |
| `text_normalization_mode` | `basic` 或 `quality`；`quality` 仅对部分非流式模型生效 |
| `output_format` | 非流式时可选 `hex` 或 `url`，流式只支持 `hex` |
| `aigc_watermark` | 是否在非流式音频末尾添加节奏标识 |

文本还支持换行、`<#x#>` 停顿标记、带声调数字的拼音/IPA/粤语拼音，以及 `speech-2.8` 系列支持的语气词标签，例如 `(laughs)`、`(sighs)` 和 `(breath)`。应把这些标记视为模型输入语法，而不是普通文本内容。

### 3.3 流式处理注意点

流式响应可能通过 JSON 或 `text/event-stream` 分块返回，每个分块会带 `data.audio` 和状态。`status=1` 表示合成中，`status=2` 表示结束。

官方 `stream_options.exclude_aggregated_audio` 默认是 `false`，因此最后一个 chunk 默认会包含拼接后的完整音频 hex。客户端有两种正确做法：

1. 实时播放 `status=1` 的中间 chunk；保存完整文件时，只使用最后 `status=2` 包中的完整音频，不再追加已播放的分片；或
2. 将 `exclude_aggregated_audio` 设为 `true`，再自行拼接所有音频分片。

不能在默认配置下把“所有 chunk”无条件拼接，否则最后一包会导致重复音频。实时播放还要确认音频格式、播放器和分片边界是否匹配；只需要低延迟时可考虑官方 WebSocket 方案。

### 3.4 错误处理

HTTP 200 不代表业务成功。必须同时检查 `base_resp.status_code`：

| 业务码 | 含义 |
| ---: | --- |
| `0` | 成功 |
| `1000` | 未知错误 |
| `1001` | 超时 |
| `1002` | 触发限流 |
| `1004` | 鉴权失败 |
| `1039` | TPM 限流 |
| `1042` | 非法字符超过 10% |
| `2013` | 请求参数异常 |

建议记录 `trace_id`、模型、计费字符数和业务码，但不要记录完整 API Key 或原始敏感文本。对 `1002`、`1039` 使用指数退避；对 `1004` 先检查 Key 类型、区域和请求地址；对 `2013` 直接修正请求，不要盲目重试。

## 4. Skill、CLI 与 MCP 接入

### 4.1 官方 CLI + Skill：适合 Agent 调用

MiniMax 官方提供 `mmx` CLI。官方文档说明：

```bash
# Node.js 18+
npm install -g mmx-cli

# 使用 API Key 登录，Key 不要直接写入报告或代码仓库
mmx auth login --api-key "$MINIMAX_API_KEY"

# 查看当前认证和用量
mmx auth status
mmx quota

# 直接合成语音
mmx speech synthesize \
  --text "欢迎使用 MiniMax 语音合成。" \
  --out welcome.mp3

# 查看可用音色
mmx speech voices
```

如果希望 Claude Code、OpenClaw、Cursor 等 Agent 在自然语言任务中知道如何调用 `mmx`，安装官方 Skill：

```bash
npx skills add MiniMax-AI/cli -y -g
```

登录时应由用户本人在本地终端运行 `mmx auth login`，不要把真实 Key 发送给 Agent。上面的非交互示例适用于已通过环境变量注入凭证的脚本。

安装后需要重启 Agent。Skill 本身只是把命令用法、参数和工作流提供给 Agent，并不替代鉴权，也不改变 API 的计费规则。官方 CLI 当前支持语音合成、音色列表、文本、图像、视频、视觉理解和搜索等能力；TTS 的常用命令是 `mmx speech synthesize`。

若只想封装这一条 HTTP API，也可自建一个窄范围 Skill：`SKILL.md` 定义何时调用、音色/模型选择和输出约定，`scripts/tts.mjs` 或 `scripts/tts.py` 负责读取环境变量、发送请求、校验 `base_resp`、解码 hex 并保存音频。输出只返回文件路径、时长、`usage_characters` 和 `trace_id`，不要把整段 hex 送回 Agent 上下文。该方案是本报告的工程建议，不是 MiniMax 官方已有 Skill 的实现声明。

### 4.2 官方一键配置向导：把 MiniMax 写入 Agent

官方 CLI 文档提供了 `agent setup`，可配置 Claude Code、Codex CLI、OpenCode、Grok CLI、Hermes Agent 和 Pi：

```bash
npx -y mmx-cli@latest agent setup
```

非交互示例：

```bash
npx -y mmx-cli@latest agent setup \
  --agent codex \
  --api-key "$MINIMAX_API_KEY" \
  --region cn
```

向导会询问 Key 类型。官方说明它既接受订阅 Key，也接受按量计费 API Key；两者的额度和扣费路径不同。`--dry-run` 可用于预览配置变更。不要把 `--api-key` 后面的真实 Key 写进 shell 历史、CI 日志或 Agent 对话；生产环境更适合使用环境变量或交互式登录。

**推荐选择：**

- 只想让 Agent 生成语音：安装 CLI + 官方 Skill。
- 想让多个编码 Agent 使用 MiniMax 模型：使用 `mmx agent setup`。
- 只做后端产品集成：直接调用 T2A HTTP API，不需要安装 Skill。

### 4.3 官方 MCP：如果用户所说的 CRI 实际指 MCP/工具协议

MiniMax 官方提供 Python 和 JavaScript 版本的 MCP Server，工具包括 `text_to_audio`、`list_voices`、`voice_clone`、`voice_design` 等。官方文档推荐新项目优先考虑 CLI，但 MCP 适合已经使用 Claude Desktop、Cursor、Windsurf、OpenAI Agents 或其他 MCP 客户端的场景。

Python MCP 的示例配置：

```json
{
  "mcpServers": {
    "MiniMax": {
      "command": "uvx",
      "args": ["minimax-mcp"],
      "env": {
        "MINIMAX_API_KEY": "<按量 API Key 或文档支持的订阅 Key>",
        "MINIMAX_MCP_BASE_PATH": "<本地输出目录>",
        "MINIMAX_API_HOST": "https://api.minimax.cn",
        "MINIMAX_API_RESOURCE_MODE": "local"
      }
    }
  }
}
```

JavaScript MCP 的启动形态是：

```json
{
  "mcpServers": {
    "minimax-mcp-js": {
      "command": "npx",
      "args": ["-y", "minimax-mcp-js"],
      "env": {
        "MINIMAX_API_HOST": "https://api.minimax.cn",
        "MINIMAX_API_KEY": "<API Key>",
        "MINIMAX_MCP_BASE_PATH": "<本地输出目录>",
        "MINIMAX_RESOURCE_MODE": "local"
      }
    }
  }
}
```

MCP 服务本质上仍是调用 MiniMax API。它只是把 TTS 包装成 Agent 可发现的工具，不会让 API 变成免费，也不会自动获得 Audio 网页的声贝额度。若 `CRI` 是另一个内部运行时协议，最稳妥的实现方式是让 CRI 工具服务端调用同一段 T2A HTTP 客户端代码，并把 API Key 放在 CRI 服务端环境中。

## 5. 免费、付费与额度

### 5.1 开放平台普通 API Key：按量计费

当前官方定价页给出的 TTS 刊例价为：

| 接口 | 模型 | 价格 |
| --- | --- | ---: |
| 同步 T2A | `speech-2.8-hd` | ¥3.50 / 万字符 |
| 同步 T2A | `speech-2.8-turbo` | ¥2.00 / 万字符 |
| 异步长文本 T2A | `speech-2.8-hd` | ¥3.50 / 万字符 |
| 异步长文本 T2A | `speech-2.8-turbo` | ¥2.00 / 万字符 |

计费按输入字符计算：1 个汉字计 2 个字符；英文字母、标点、空格、回车等各计 1 个字符。最终应以响应中的 `extra_info.usage_characters` 和控制台账单为准。

按量计费流程是：

1. 在开放平台“接口密钥”创建普通 API Key。
2. 在账户余额中充值。
3. 调用接口时按实际使用量扣账户余额。
4. 在控制台查看用量和账单，并可设置余额预警。

**小规模使用成本示例：** 不含标点的 1,000 个汉字约计 2,000 个计费字符，Turbo 约 ¥0.40，HD 约 ¥0.70；10,000 个汉字约为 ¥4.00 / ¥7.00。这是总用量成本示例，不表示 10,000 个汉字可放进一次同步请求。系统音色不会触发自建音色首用费；快速复刻和音色设计当前均为 ¥9.90 / 音色，首次正式用于合成时收费，接口内试听另按文档计费。

官方速率限制页当前列出 T2A v2 的 RPM：免费用户 10，充值用户 20。这里的“免费用户”是资源保障/速率限制分类，不等于每月赠送免费 TTS 字符；不能将 10 RPM 理解为免费调用额度。是否获得 API 试用金或优惠券，应在账号控制台确认，本次公开文档未确认固定的月度免费 TTS 额度。

### 5.2 语音资源包：预付字符额度

官方语音资源包区分 HD 和 Turbo，且同步 T2A、异步长文本 T2A 都在支持范围内：

| 类型 | 价格 | 字符数 | 有效期 | 适用模型 |
| --- | ---: | ---: | --- | --- |
| HD 套餐一 | ¥630 | 2,000,000 | 1 个月 | HD |
| HD 套餐二 | ¥5,950 | 20,000,000 | 3 个月 | HD |
| HD 套餐三 | ¥56,000 | 200,000,000 | 1 年 | HD |
| Turbo 套餐一 | ¥360 | 2,000,000 | 1 个月 | Turbo |
| Turbo 套餐二 | ¥3,400 | 20,000,000 | 3 个月 | Turbo |
| Turbo 套餐三 | ¥32,000 | 200,000,000 | 1 年 | Turbo |

资源包是 API 侧明确的预付额度，HD 与 Turbo 不能混为一个通用字符池。官方说明资源包余量不继承，过期清零；购买前应核对模型系列、有效期和接口范围。

### 5.3 M Plan 订阅 Key 与普通 API Key

当前官方 M Plan FAQ 明确区分：

- **订阅 Key**：用于 M Plan 套餐额度和已购积分，额度覆盖文本、图像、语音等资源，具体消耗受套餐窗口与支持范围影响。
- **普通 API Key**：用于开放平台按量计费，消耗账户余额。

二者不能混用。官方 FAQ 还说明，已有按量计费价格的 API 端点可以按对应价格扣减 M Plan 套餐内额度；额度不足时，可按规则使用已购积分。生产环境官方建议使用按量计费，而 M Plan 更偏向个人开发者和 Agent 交互式使用。

因此，若使用 `mmx` 或把 MiniMax 接入 Codex/Claude Code：

- 用订阅 Key，通常走 M Plan/积分路径，不会自动扣开放平台余额。
- 用普通 API Key，走开放平台按量计费，会扣余额。
- 不要只看 Key 都以 `sk-` 开头就假设额度相同，应在控制台确认 Key 类型和区域。

### 5.4 Audio 网页免费与付费区别

下表依据 Audio 官方页面实际加载的 `prod-zh-0.1.44` 中文资源中的 FAQ 和权益文案整理。静态资源可能同时包含新旧套餐和活动文本，其证据强度低于登录后当前套餐页；不据此保证某个账号此刻拥有全部权益。

| 项目 | 免费/非会员 | 付费会员 |
| --- | --- | --- |
| TTS 合成 | 消耗产品声贝，受剩余额度限制 | 更多套餐声贝，具体数量以账号套餐为准 |
| 音色克隆 | 文案为“免费用户仅可预览音色” | 可保存、使用克隆音色，卡槽数依套餐而定 |
| 情绪与音色效果调试 | 仅买声贝不会获得会员专属功能；另有“限时免费”文案，活动需看实际页面 | 按套餐解锁相关功能 |
| 克隆/音色设计试听 | 每日 5 次免费预览，两页面共用，超出消耗声贝 | FAQ 使用“用户”而未单独给会员更多免费次数，其他权益以套餐为准 |
| 语音商用 | FAQ 允许个人用途，但要求加标识或标注音频来源 | FAQ 写明语音商用需“标准版会员及以上” |
| 生成速度 | 普通资源 | 权益文案包含更快生成速度 |
| 额外充值 | 可补充声贝，不等于购买会员 | 声贝与会员功能分开管理 |

网页文案的单位是 HD `1 字符 = 1 声贝`、Turbo `1 字符 = 0.6 声贝`；FAQ 另说明中文汉字/日文字符与积分存在双倍计量，实际预览扣减以网页为准。这些是网页产品内部规则，不是声贝与 API 字符费用之间的转换率。充值声贝文案标注购买后 2 个月到期，也不能套用到 API 余额、API 资源包或赠送声贝。

当前语音商用 FAQ 原文为：

> 个人用途（如社交媒体发布或非营利性的会议）您可以免费使用 MiniMax 语音生成的音频，只需添加我们的标识或标注音频来源。如需将音频用于商业用途（如创收内容或商业项目），请订阅我们的标准版会员及以上。

这仅是 Audio 网页产品的授权说明，不代表开放平台 API 商用要求购买 Audio 会员。API 授权应按开放平台的服务条款判断。

## 6. “每月 1000 声贝”能否用于 API？

### 6.1 结论

**目前不能按可用处理。** 更准确地说：官方公开的 API 定价、资源包、M Plan FAQ 和 API 文档都没有说明 MiniMax Audio 网页的“声贝”可以兑换、共享或抵扣开放平台 T2A API 的字符费用。因此，工程和预算上应把两者视为两个产品额度池：

- MiniMax Audio 网页：面向网页创作体验的产品额度，单位是“声贝”；
- MiniMax 开放平台 API：面向开发者接入，按量计费、API 资源包或 M Plan/积分规则结算，单位主要是字符数。

本次未登录页面仅显示“登录获得免费积分”，未确认固定的“每月 1000 声贝”政策。用户看到的 1000 声贝可以是具体账号权益，但本文不将其写成所有用户统一赠送标准。

除非登录后的 Audio 产品明确出现“用于 API”或官方客服对账号给出书面确认，否则不能把 1000 声贝换算成 API 字符，也不能据此保证 API 请求不会扣费。

### 6.2 为什么不能直接换算

1. **计量单位不同**：API TTS 按输入字符计量，Audio 产品用“声贝”计量。公开资料没有两者的换算公式。
2. **产品入口不同**：API 使用开放平台接口密钥；Audio 网页使用产品侧登录和额度体系。
3. **官方 API 价格页没有列出声贝抵扣**：API 的免费用户说明只涉及速率限制，不是每月 1000 声贝。
4. **官方资源包也没有写入 Audio 网页免费额度**：语音资源包是单独购买的 API 侧产品，按 HD/Turbo 和字符数管理。

### 6.3 建议验证方式

如果必须确认某个具体账号是否存在跨产品权益，建议只做一次短文本验证，并在发起前检查：

- Audio 产品的额度说明是否明确出现“API 可用”；
- 开放平台控制台余额、资源包或 M Plan 用量是否发生变化；
- 请求响应中的 `usage_characters`、控制台账单和 Audio 声贝余额是否有对应变化；
- 必要时向 MiniMax 官方支持提供账号主体、产品入口和 API Key 类型，要求确认“声贝是否能抵扣 `t2a_v2`”。不要在工单中提交完整 Key。

在没有得到明确确认前，应按 API 正常计费来估算成本。若目标只是每月少量个人播报，直接使用 Audio 网页更简单；若目标是自动化、批量处理、服务端生产或 Agent 工具调用，应使用开放平台 API/资源包/M Plan 的明确路径。

## 7. 推荐落地方案

| 目标 | 推荐方式 | 额度/计费 |
| --- | --- | --- |
| 服务端自动生成短音频 | 直接调用 HTTP T2A | 普通 API Key 按量计费，或 API 语音资源包 |
| Agent 根据自然语言生成音频 | `mmx` CLI + 官方 Skill | 订阅 Key 或按量 Key，按所选 Key 的规则结算 |
| 已有 MCP 客户端和工具编排 | 官方 MiniMax MCP | MCP 只是封装，底层仍按 API/M Plan 规则计费 |
| 大于 3,000 字符且需要尽快出音频 | HTTP 流式或 WebSocket | 处理分片，避免默认最后分片重复拼接 |
| 长篇文章、书籍、有字幕 | T2A Async v2 | 创建任务、查询状态、按 `file_id` 下载；下载 URL 仅短期有效 |
| 生产系统 | 普通 API Key + 余额预警，或购买匹配的语音资源包 | 不依赖网页 Audio 免费声贝，单独核算成本 |

## 8. 官方来源

- [同步语音合成 HTTP API](https://platform.minimax.cn/docs/api-reference/speech-t2a-http)
- [同步语音合成接入地址与流式指南](https://platform.minimax.cn/docs/guides/speech-t2a-websocket)
- [定价概览](https://platform.minimax.cn/docs/pricing/overview)
- [按量计费](https://platform.minimax.cn/docs/guides/pricing-paygo)
- [速率限制](https://platform.minimax.cn/docs/guides/rate-limits)
- [账户相关](https://platform.minimax.cn/docs/faq/about-account)
- [M Plan 常见问题](https://platform.minimax.cn/docs/m-plan/faq)
- [M Plan 概览](https://platform.minimax.cn/docs/m-plan/intro)
- [MiniMax CLI](https://platform.minimax.cn/docs/token-plan/minimax-cli)
- [MiniMax CLI GitHub](https://github.com/MiniMax-AI/cli)
- [MiniMax MCP 使用指南](https://platform.minimax.cn/docs/guides/mcp-guide)
- [MiniMax Audio 文字转语音](https://www.minimax.cn/audio/text-to-speech)
- [Audio 官方中文部署资源：FAQ、声贝及会员权益文案](https://cdn.hailuoai.com/hailuo-voice-web/prod-zh-0.1.44/_next/static/chunks/6002-6a6757c6bbd92374.js)
- [查询可用音色 ID](https://platform.minimax.cn/docs/api-reference/voice-management-get)
- [创建异步语音任务](https://platform.minimax.cn/docs/api-reference/speech-t2a-async-create)

## 9. 资料限制

MiniMax Audio 文字转语音页是客户端渲染页面，公开 HTML 主要包含应用壳和脚本；本次另核查了页面加载的官方中文部署资源，可确认声贝消费、免费试听、克隆与会员权益文案，但资源可能含未启用的历史/活动变体，未登录浏览器未验证具体套餐或月度赠送数。未找到可独立引用的“1000 声贝可用于 API”或“禁止互通”条款，因此本文表述为“官方资料未证明互通，工程上按不可用处理”。

官方材料也存在版本差异：CLI GitHub README 仍称 Requires Token Plan，新 CLI 文档已经称 M Plan；MCP 指南默认模型为旧版 `speech-02-hd`，且情绪表含 `whipser` 拼写和与 API 页面不同的模型支持说明。接入应以当前 HTTP API 参数文档和安装版本的 `--help` 为准。新版 M Plan 与旧 Token Plan 页面有不同套餐及积分价格，本报告不将它们混合成统一价目表。

异步接口也不能简单理解为 `text` 字段支持 100 万字符：直接 `text` 最长 5 万，文件输入小于 100 万；创建任务后需查询、下载，URL 有效期为 9 小时。同步接口详情写“小于 10000”，概览写“最长 10000”，本报告按详情的严格边界描述。

本次未持有或使用用户 API Key，没有发起计费语音生成请求；命令和配置为官方文档核对结果，不是已完成账号端到端测试。价格、模型、套餐和产品权益可能变化，实际接入前应重新核对对应官方页面。
