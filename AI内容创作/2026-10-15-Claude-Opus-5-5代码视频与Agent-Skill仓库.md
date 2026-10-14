---
title: "Claude Opus 5.5 代码视频与 Agent Skill 仓库调研"
date: 2026-10-15
category: "AI内容创作"
tags:
  - AI视频剪辑
  - Agent Skill
  - Claude Opus 5.5
  - 代码动画
  - 提示词
  - Canvas
  - ffmpeg
---

# Claude Opus 5.5 代码视频与 Agent Skill 仓库调研

## 1. 范围与核查方式

调研日期：**2026-10-15**。目标是确认用户列出的五个名字对应什么仓库、能提供什么、怎样使用、能否直接出片，以及哪些内容只是作者声明或展示。

使用 Agent Reach 的 GitHub CLI 后端搜索仓库，读取 GitHub API 元数据、固定提交的完整文件树、README、JSON 索引、提示词样本、Skill 入口、安装脚本、渲染源码、依赖声明及最近 Issues/Release。临时资料存放在系统临时目录；本次没有安装这些项目，没有执行第三方安装脚本，没有独立复现成片，也没有逐条观看视频或重新核查其 X 原帖。下文明确区分仓库统计、维护者声明和本次分析。

### 名称对应

| 用户提供的名字 | 本次定位的仓库 | 判定 |
| --- | --- | --- |
| awesome-opus5-5-videos | [yihui-dev/awesome-opus5-5-videos](https://github.com/yihui-dev/awesome-opus5-5-videos) | 名称完全匹配 |
| lemo-opuscar | [lemomo-ai/lemo-opuscar](https://github.com/lemomo-ai/lemo-opuscar) | 采用作者主仓库，不采用派生项目 |
| awesome-claude-video | [opusvideo/awesome-claude-video](https://github.com/opusvideo/awesome-claude-video) | 名称完全匹配；不混同 awesome-claude-video-skills |
| awesome-opus5-5-video | 未定位独立的同名公开仓库 | GitHub 普通搜索及 `in:name` 搜索均只返回复数主仓库和其插件派生项目；可能重复或名称有误，但不能据此认定不存在 |
| awesome-opus-video-skills | [ismoshushi/awesome-opus-video-skills](https://github.com/ismoshushi/awesome-opus-video-skills) | 名称完全匹配 |

单数项目作为待确认项保留，未用近似名字的项目补位。

## 2. 主要结论

**真正适合拿来做风格化代码短片的是 Lemo-Opuscar；另外三个已定位项目主要用于找案例、提示词或候选工具。** 它们可以组成“案例参考 → 选择风格或 Skill → 编写动画 → 渲染与检查”的工作流，但不是四套可以互换的视频生成器。

| 仓库 | 实际类型 | 核心内容 | 是否自带出片工具 | 首要用途 |
| --- | --- | --- | --- | --- |
| awesome-opus5-5-videos | 作品与提示词合集 | 703 条记录/提示词文件，418 条标有 Remake | 否 | 找视觉参考、学习提示词 |
| lemo-opuscar | 可安装 Skill、导演指南与制片工具库 | 43 套常规风格、样片源码、渲染/语音/配乐工具，另有 OPUSCAR 98 图鉴 | 是 | 生成可修改源码与 MP4 |
| awesome-claude-video | 案例图鉴与实现指南 | 59 个案例，出处、工具披露、预览、复现清单 | 否 | 看不同制作路线及证据 |
| awesome-opus-video-skills | 第三方工具索引 | 117 项工具/Skill，用途、栈、安装方式、许可 | 否；需进入原项目 | 选型和发现可安装 Skill |

共同的核心路线是 **Agent 写渲染代码，再用浏览器或动画引擎生成帧，最后合成视频**。这是 Canvas、SVG、Three.js、p5.js、GSAP、Remotion 等编程动画路线。部分案例另外使用 AI 图片、视频生成模型、TTS、现成音乐和实际素材，因此不能把“Claude 辅助制作”一律理解为“全部像素、声音和素材均由纯代码生成”。

仓库名称中的 Opus 5.5 是项目的定位或作者归因。本次没有做模型性能对比，不据此认定其他模型无法运行这些代码或达到相同效果。

## 3. awesome-opus5-5-videos：大规模提示词参考库

### 内容与数据

主仓库为 `yihui-dev/awesome-opus5-5-videos`。文件树主要是 `README.md`、`LICENSE`、`data/videos.json` 和 `prompts/`，没有视频渲染器、安装包或 `SKILL.md`。

本次读取 `data/videos.json` 并与文件树交叉统计：

| 指标 | 固定提交统计 |
| --- | ---: |
| JSON 记录数 | 703 |
| `prompts/*.md` 文件数 | 703 |
| `has_remake=true` | 418 |
| `prompt_partial=true` | 207 |
| motion / 3d / explainer / interactive | 369 / 117 / 116 / 101 |

README 主展示区选取 100 个 Remake，分别为动态图形 58、讲解 16、3D 14、游戏与交互 12；这组数字是展示子集，不能与全部 703 条分类统计混用。

记录包含作者、原帖链接、技术标签、提示词、是否为部分提示词、Skillry 地址等。`tech_tags` 和样本中的 “Remake built with” 可能描述复刻版技术栈，不能直接拿来证明原作者用的是同一技术。418 是仓库标注存在 Remake 的数量，本次未独立验证网站全部复刻版本可播放或与原作一致。

### 怎么用与实际价值

README 的操作是打开提示词文件、复制 Prompt、交给运行 Opus 5.5 的 Agent，要求生成单 HTML，再录制页面。适合寻找产品发布片、UI 动效、科普、粒子、3D 或像素风参考。

抽查 `prompts/moritzkremb-466494.md`：要求找一个熟悉的 SaaS、获取真实品牌素材，再做专业产品发布动态图形，复刻标签为 SVG/GSAP。它提供创作方向，没有给完整镜头表、渲染工程和品牌素材包。

抽查 `prompts/anabology-491441.md`：包含已有 MP4/歌曲、私有 `/asic` 文件夹、ElevenLabs、图像与 Seedance 视频生成等上下文。说明某些看似可复制的提示词实际上依赖原作者的素材、私有 Skill、服务和创作预算；复制文字不能保证复现。该样本也证明集合中存在混合媒体流程。

**建议用途：作为提示词与原帖检索库。** 可读取 JSON 做分类、筛选完整度、保存出处；要做可运行模板，还需自己生成源码、明确资产和音频来源，并补渲染工具。MIT 适用于仓库许可范围；README 明确视频与提示词属于各创作者，不能用仓库 MIT 推导第三方作品可任意使用。

## 4. Lemo-Opuscar：这一组中最完整的制片工作流

### 风格与工程内容

主仓库为 `lemomo-ai/lemo-opuscar`，作者 Lemomo。README 明确其不是 awesome 外链合集，而是作者用 Agent 制作的风格与样片库。

文件树有 1,760 个文件，其中 43 个常规风格 `STYLE.md`；额外的第 44 个是 `styles/_template/STYLE.md`，不能计入可选风格。风格覆盖手绘、东方传统、版画、动态排版、科普发布、卡通、游戏、时代电影、3D 材质等。

另有 `styleboard/opuscar98_styles.json`：98 条电影风格记录，按 `render` 字段统计为 2D 77、2.5D 4、3D 17。README 将 OPUSCAR 98 描述为 6 分 31 秒的特别放映。这 98 条提供风格提示词、镜头与编排信息，**不能等同于额外 98 套独立、完整的常规 Skill/demo 工程**。

关键组成：

| 内容 | 作用 |
| --- | --- |
| `plugin/skills/lemo-opuscar/SKILL.md` | 获取共享库、安装依赖、交给库内制作指南 |
| `DIRECTOR.md` / `TECHNIQUE.md` | 故事、节奏、镜头、声音、制作和交付规则 |
| `styles/<slug>/STYLE.md` / `DEMO.md` / `demo/` | 风格语言、样片说明、源码示例 |
| `core/render/` | 静帧、视频、事件、文字可读时长、字幕、合成工具 |
| `core/tts/` | Kokoro、edge-tts、Whisper 回译校对与词级时间信息 |
| `core/audio/` | 程序音效、采样乐器、混音辅助 |

Skill 入口明确面向新建短片、宣传片、讲解或动画，**不用于编辑或转换已有视频文件**。因此它不能直接替代剪映草稿工具或真实素材自动混剪器。

### 安装方式与 Codex 使用边界

Claude Code 的 README 命令为：

```sh
claude plugin marketplace add lemomo-ai/lemo-opuscar
claude plugin install lemo-opuscar@lemolab
```

首次调用在默认 `~/lemo-opuscar` 获取共享库，成片工程放在发起目录中的独立文件夹。另一种是完整 clone，在仓库内制作 `films/<name>/`。

README 对其他 Agent 的建议是复制 **`plugin/skills/lemo-opuscar/`** 到其 skills 目录，并保留脚本。Claude 的 plugin 命令不能作为 Codex 的安装命令。复制入口后，还要能运行 shell 脚本、获取共享库，并让 Agent 阅读库中的制作指南；本次未进行 Codex 安装实测。作者声明风格针对 Opus 5.5 调整，其他模型效果不保证，这是维护者口径。

### 依赖、渲染和音频

README 要求 Node 20+、ffmpeg、Python 3.11+ 或 uv；语音依赖文件进一步注明 Kokoro-ONNX 需要 Python 3.11–3.13。核心 npm 依赖为 `playwright-core` 与 `three`。

依赖分层为：核心 NumPy/SciPy/soundfile/soxr/Pillow，语音 Kokoro-ONNX/edge-tts/faster-whisper，音乐 numba 等。安装脚本另外下载 Chromium headless shell，语音模式获取模型；乐器、HDRI 等可以按需获取。因此“不用视频生成模型”不等于“无安装、无模型下载、无网络、无计算成本”。

页面渲染契约：

```javascript
window.READY = true; // 字体、图片等资源就绪
window.DUR = 30;    // 秒
window.render(t);   // 指定时间画帧，需确定性
```

`video.mjs` 逐帧输出视频，默认 1920×1080 / 24 fps；`still.mjs` 生成检查帧，`readcheck.mjs` 根据 `window.TEXTS(t)` 检查文字在画面内的持续时间，`mux.sh` 合成音频并做 −14 LUFS、真峰值 −1.2 dB 的响度处理。WebGL 渲染器会检查软件渲染并提示性能影响。

Kokoro 是本地语音路线，但先要有模型与依赖；中文 edge-tts 是在线路线，不能归为离线。Whisper 回译用于校对发音/文本一致性，不能证明故事或知识内容真实。字体子集也有覆盖限制，`core/README.md` 指出 ZCOOL 子集只有约 200 字，需要检查新稿字符是否完整。

### 源码样例说明了什么

`styles/dark-keynote/demo/build.sh` 展示时间线 → Kokoro 配音 → Whisper 校对 → 原创配乐 → 事件/字幕 → 音画 cue 检查 → 混音 → 渲染 → mux → 成片抽查的实际命令链。这比单纯的作品预览更适合研究工程化出片。

但 Skill 的 `setup.sh demo <slug>` 注明 demo 下载用来阅读技术参考，不是保证可直接渲染的快捷模板。不能把有 `build.sh` 推导为 Skill 模式里执行一次即可无条件复现所有样片。此样例末尾 `final_asr.py ... || true` 也意味着最终抽查失败可能不阻断脚本，仍应人工查看输出。

### Windows 与更新行为

安装和样片命令依赖 `sh`、POSIX 工具及 `.venv/bin/python`，没有核实原生 PowerShell 完整路径。Git Bash 可以提供 shell，但不会自动把 Windows Python 虚拟环境变成 POSIX 布局。对当前 Windows 环境，建议优先评估 WSL/Linux 运行完整产线；若坚持原生 Windows，需要适配脚本、虚拟环境路径及子进程。3D 场景还要实际核查 Chromium/GPU 后端。

安装脚本对它创建、标记 `lemo.managed=true` 的共享 clone 会 fetch `main` 并 `reset --hard FETCH_HEAD`；用户所在的现有库 clone 则按脚本规则复用。**不应把自定义影片源码或手改库文件当作托管缓存保存**。需要长期复现时，另存工程、锁定库提交和依赖，并保留素材来源。

作者估计单片约 30–60 分钟、token 用量较大，本次未验证时长或费用。项目许可证为 MIT，样片中的字体、采样、模型素材仍保留各自许可，按 CREDITS 核对。

## 5. awesome-claude-video：规模较小、证据字段更细的案例图鉴

主仓库为 `opusvideo/awesome-claude-video`。完整文件树仅 14 个文件，以中英文 README、`cases.json`、`IMPLEMENTATION_GUIDES.md`、贡献说明和封面为主，没有统一运行工程或可安装 Skill。

JSON 共 59 个案例：产品广告 8、动态图形/UI 4、科普 11、角色故事 13、艺术/3D/模拟 12、音乐/纪录片/剪辑 5、模型比较 6。记录包含作者、原帖、视频链接、浏览量核查时间、模型归因、其他模型、披露工具、提示词链接、源码链接和核查状态。仅 3 条 `source_code_url` 非空，不能说它提供了 59 套源码工程；其他实现线索也可能出现在 `resources`，需逐条阅读。

维护者的 `verification_scope` 明确说明：核查 X oEmbed 文本与视频元数据，部分原帖曾在浏览器查看；模型是创作者归因，视频未独立复现或全面事实核查。这种结构有利于区分“有出处”和“已经复现”。本次只是重新读取这些字段，没有再次验证每个外部原帖。

`IMPLEMENTATION_GUIDES.md` 明确是根据公开信息重新组织的制作清单。例如无缝 UI morph 给出单 HTML、确定性的 `seek(t)`、Playwright、FFmpeg、弹簧和配乐节拍等路线；其他案例涉及 Manim/Kokoro、Remotion、Blender、After Effects、HyperFrames、Runway MCP、真实素材剪辑等。它适合研究**不同产线**，不能解释成所有片子都是纯 HTML、没有其他模型。

GitHub API `license=null`，文件树也没有 LICENSE；贡献说明指出作品归原作者、收录不授予新许可。能公开阅读不代表可任意转载视频或作为已授权工程使用。

## 6. awesome-opus-video-skills：寻找具体工具的入口

主仓库为 `ismoshushi/awesome-opus-video-skills`，文件树只有 10 个文件，包括中英文 README、网页、LICENSE 和一段展示视频。`skills.md` 只是指向 README 的迁移说明，**不是 `SKILL.md` 入口**。仓库本身不托管第三方 Skill 源码，也没有统一渲染器。

本次按 README 速览表逐行统计，与明细标题一致：

| 分类 | 条目数 | 用途 |
| --- | ---: | --- |
| ① 代码出片 | 70 | Canvas/GSAP/p5.js/Remotion/HyperFrames 等 |
| ② 产品片、成片编辑 | 24 | 实际素材、口播、剪辑、产品展示 |
| ③ 看视频、视频转 Skill | 10 | 抽帧、转录、分析、反向还原流程 |
| ④ 调用外部视频模型 | 13 | 外部 API、视频模型、ComfyUI 等 |
| 合计 | 117 | 目录条目数，不等于已实测安装数 |

字段包括用途、技术栈、安装方式、许可证和是否明确提到 Opus 5.5。`✅` 表示原项目明确点名该模型，不是本次的兼容性实测；`—` 也不能直接证明全 Agent/全模型兼容。星数是索引维护者不同日期的历史快照，应与本报告实时 API 数字分开。

其宣称“只收开源、可安装 Skill”，但表中存在明确标注“非可安装 skill”的 `Vincentwei1021/mg-styles-15`，以及三个未标注标准许可证的条目：`misbahsy/claude-horizon-animation`、`WaterDJiang/Wattter_openSkills`、`Moh4696/claude-video-vision`。这是索引标准与实际收录之间的例外，不能直接称为 117 个已授权、可一键安装的 Skill。

### 怎样利用这个索引

先按目标选择类别，再到具体项目核查 README、`SKILL.md`、安装脚本、demo、系统依赖、服务费用和许可证。索引中的 `npx skills add <owner>/<repo>`、复制技能目录、Claude plugin 等方式互不通用，以原项目入口和当前 Agent 为准。

例如 Lemo 适合代码风格短片；索引列出的 Remotion 工具适合组件化视频，HyperFrames/GSAP 类适合 HTML 动效；真实素材剪辑和视频理解属于不同类别。这里是筛选方向，除 Lemo 外，本次未把 117 个链接逐个展开、审计或试运行，因此不给单项质量排名。

索引自身为 MIT，不能替代第三方仓库许可。收录 Lemo 与链接 `awesome-claude-video` 表明这几个资源有交集，联合使用时应去重，不要把它们的条目数相加当成独立项目数量。

## 7. 维护快照与许可证

以下是 2026-10-15 获取的 GitHub API 快照。提交日期以 API 返回的 UTC 日期记录；star/fork 是动态数量，`updated_at` 可能受非源码活动影响，故维护判断使用实际最新提交时间。

| 仓库 | Star / Fork | 最新提交日期（UTC） | 固定提交 | 许可证 | Release 情况 |
| --- | --- | --- | --- | --- | --- |
| yihui-dev/awesome-opus5-5-videos | 5,002 / 524 | 2026-10-13 | `92aee2f586ac0715cb3cbcb6384834f98a33383d` | MIT；原作者作品另算 | 未查到 Release |
| lemomo-ai/lemo-opuscar | 2,319 / 246 | 2026-10-13 | `fdc46d7f0e20cc953a71d56241a92d1b4cb7298e` | MIT；第三方素材另算 | `films`、`web` 滚动媒体发布，另有 `thanks` |
| opusvideo/awesome-claude-video | 200 / 10 | 2026-10-06 | `4b74fdb099ade960e72fa303bb87720303ccfbb1` | 未发现标准 LICENSE | 未查到 Release |
| ismoshushi/awesome-opus-video-skills | 14 / 2 | 2026-10-14 | `8371f56dba3fe847bc86ff792fa890a66cced97a` | 索引 MIT；条目另算 | 未查到 Release |

四个项目均未标记 archived。Lemo 的媒体 Release 不能当作稳定软件版本承诺；`package.json` 为 private，版本 0.1.0。其最新提交说明删除了 readcheck 测试文件；当前不能据此宣称有完整自动化测试套件。最近的 readcheck PR 记录了边界修复与当时的验证，但这不等于本次独立运行通过。

## 8. 面向实际工作的选择建议

| 想完成的事情 | 推荐入口 | 需要补齐的环节 |
| --- | --- | --- |
| 看大量动效、抄提示词方向 | awesome-opus5-5-videos | 补素材、源码、时间线与导出；优先检查 `prompt_partial` |
| 看产品、科普、故事分别怎么做 | awesome-claude-video | 阅读工具披露和原帖，筛选有源码的案例 |
| 做新故事的风格化短片 | lemo-opuscar | 系统依赖、Windows 路径、TTS/字体、渲染和内容检查 |
| 找适合当前 Agent 的视频 Skill | awesome-opus-video-skills | 进入原仓库确认真实安装入口及示例 |
| 已有真人素材混剪、剪映草稿、口型同步 | 上述索引的剪辑/外部模型类别 | 选择专门工具；Lemo 不负责这些任务 |

对于首次尝试 Lemo，建议给一个 15–30 秒、2D 风格、文本量少的明确主题，要求交付源码、视频、字幕、运行步骤和素材说明；先验静帧与中文字体，再查声音和节拍，最后完整渲染。这是建议的验证路线，本次没有代用户安装、制作或运行影片。

初步选型顺序：**需要产出时先研究 Lemo；需要发现更多工具时查 Skill 索引；作品合集与案例图鉴负责补参考与工作流依据。** 若目标是原生 Windows 一键成片，当前证据还不能证明 Lemo 开箱即用。

## 9. 待确认与来源

唯一未定位的目标是 `awesome-opus5-5-video`（单数）。需要准确 owner/repo 或 URL 后才能将其作为第五个独立项目评价。本报告保留四个已确认仓库的完整结论。

主要证据均可通过以下固定提交链接复查：

- 提示词库：[README](https://github.com/yihui-dev/awesome-opus5-5-videos/blob/92aee2f586ac0715cb3cbcb6384834f98a33383d/README.md)、[videos.json](https://github.com/yihui-dev/awesome-opus5-5-videos/blob/92aee2f586ac0715cb3cbcb6384834f98a33383d/data/videos.json)、[SaaS 提示词](https://github.com/yihui-dev/awesome-opus5-5-videos/blob/92aee2f586ac0715cb3cbcb6384834f98a33383d/prompts/moritzkremb-466494.md)、[混合媒体提示词](https://github.com/yihui-dev/awesome-opus5-5-videos/blob/92aee2f586ac0715cb3cbcb6384834f98a33383d/prompts/anabology-491441.md)。
- Lemo：[中文 README](https://github.com/lemomo-ai/lemo-opuscar/blob/fdc46d7f0e20cc953a71d56241a92d1b4cb7298e/README.zh-CN.md)、[Skill](https://github.com/lemomo-ai/lemo-opuscar/blob/fdc46d7f0e20cc953a71d56241a92d1b4cb7298e/plugin/skills/lemo-opuscar/SKILL.md)、[安装脚本](https://github.com/lemomo-ai/lemo-opuscar/blob/fdc46d7f0e20cc953a71d56241a92d1b4cb7298e/plugin/skills/lemo-opuscar/scripts/setup.sh)、[核心工具](https://github.com/lemomo-ai/lemo-opuscar/blob/fdc46d7f0e20cc953a71d56241a92d1b4cb7298e/core/README.md)、[样片构建](https://github.com/lemomo-ai/lemo-opuscar/blob/fdc46d7f0e20cc953a71d56241a92d1b4cb7298e/styles/dark-keynote/demo/build.sh)、[OPUSCAR 98 数据](https://github.com/lemomo-ai/lemo-opuscar/blob/fdc46d7f0e20cc953a71d56241a92d1b4cb7298e/styleboard/opuscar98_styles.json)。
- 案例图鉴：[中文 README](https://github.com/opusvideo/awesome-claude-video/blob/4b74fdb099ade960e72fa303bb87720303ccfbb1/README.zh-CN.md)、[cases.json](https://github.com/opusvideo/awesome-claude-video/blob/4b74fdb099ade960e72fa303bb87720303ccfbb1/cases.json)、[实现指南](https://github.com/opusvideo/awesome-claude-video/blob/4b74fdb099ade960e72fa303bb87720303ccfbb1/IMPLEMENTATION_GUIDES.md)、[贡献与许可说明](https://github.com/opusvideo/awesome-claude-video/blob/4b74fdb099ade960e72fa303bb87720303ccfbb1/CONTRIBUTING.md)。
- Skill 索引：[中文 README](https://github.com/ismoshushi/awesome-opus-video-skills/blob/8371f56dba3fe847bc86ff792fa890a66cced97a/README.md)、[迁移说明 skills.md](https://github.com/ismoshushi/awesome-opus-video-skills/blob/8371f56dba3fe847bc86ff792fa890a66cced97a/skills.md)、[贡献规则](https://github.com/ismoshushi/awesome-opus-video-skills/blob/8371f56dba3fe847bc86ff792fa890a66cced97a/CONTRIBUTING.md)。

Agent Reach 版本更新检查因其 GitHub API 限流未能完成，不影响上述 GitHub CLI 数据读取；本次没有更新工具。
