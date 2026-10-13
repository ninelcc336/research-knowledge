---
title: "JianYing Editor Skill 仓库与使用方式调研"
date: 2026-10-13
category: "AI内容创作"
tags:
  - AI视频剪辑
  - 剪映
  - Agent Skill
  - Python
  - 视频草稿
  - 自动化导出
  - 多模态视频理解
---

# JianYing Editor Skill 仓库与使用方式调研

## 1. 调研范围

- 调研时间：2026-10-13
- 目标：确认最近常见的 Jianying Editor Skill 对应的仓库、安装方式、能做什么、如何用于实际剪辑，以及哪些宣传能力需要额外条件。
- 核查方式：使用 Agent Reach 的 GitHub CLI、网页读取和 B 站公开搜索，检查仓库源码、README、SKILL.md、示例、Issue、许可证与当前环境。
- 验证范围：本次为文档与源码核查，没有安装或执行该 Skill，也没有验证真实剪映工程打开、预览或 MP4 导出。B 站资料仅核对标题和简介，未观看完整视频。
- 主仓库核查快照：`32c56928ded4f9e2c2b80e099dc7abb793d2c30b`（2026-09-11 提交）。后续更新可能改变安装命令和兼容范围。

## 2. 一句话结论

`luoluoluo22/jianying-editor-skill` 是视频作者「我老萝卜」简介指向的主仓库：它通过 Python 和 `pyJianYingDraft` 的扩展，把素材、音频、字幕、配音、特效、转场、关键帧和录屏结果写入剪映桌面版草稿目录，再由剪映打开、预览、继续编辑和渲染。最实用的定位是让 Agent 自动搭一个可编辑粗剪工程，人工审片后导出；视频内容理解、精彩片段选择还需要外部模型或人工给出片段清单。

当前主仓库 GitHub 页面约有 3,960 stars、509 forks，仓库 API 的最近更新时间为 2026-10-13；主分支的 `VERSION` 为 `1.7.0`，最近提交日期为 2026-09-11。仓库 README 已改为“新版草稿可生成，Windows 自动导出仍以旧版本最稳”：Windows 和 macOS 都支持草稿生成，macOS 仍需在剪映中手动导出；国内版桌面剪映是目标，手机端和 CapCut 国际版不在支持范围内。

## 3. 先辨认你刷到的是哪个仓库

| 仓库 | 当前公开数据 | 主要特点 | 适合怎么理解 |
| --- | --- | --- | --- |
| [luoluoluo22/jianying-editor-skill](https://github.com/luoluoluo22/jianying-editor-skill) | 约 3.9k stars、509 forks；MIT；v1.7.0 | `JyProject` 高层封装、草稿生成、素材自包含、TTS、字幕、云素材、录屏智能变焦、Web-to-Video、Windows 导出 | 当前主线与资料最多，本文以它为主 |
| [isYangs/jianying-editor-skill](https://github.com/isYangs/jianying-editor-skill) | 约 218 stars、19 forks；MIT；v1.0.0；最近提交 2026-03-29 | 同样面向自然语言剪映自动化，README 给出 Python 3.8+、FFmpeg 与剪映 5.9 或更低版本要求 | 可作为同名项目的另一个分支，不要混用安装路径和文档 |
| [jjjane233/jianying-editor-skill-11.5](https://github.com/jjjane233/jianying-editor-skill-11.5) | 约 3 stars；基于 isYangs 适配 | 作者声称实测 Windows 剪映 `11.5.0.14471`，调用本机 DLL 读写原生加密草稿 | 特定新版的候选方案，验证范围和成熟度需单独评估 |

视频中常见的“剪映 5.9 + Trae + Skill”教程，多数指向主仓库。B 站公开搜索还能看到作者演示“录屏自动缩放”“影视解说”“游戏视频配音字幕”等场景；这些是示例工作流，不代表所有素材都能一次成功。

## 4. 它的工作原理

```text
自然语言需求
      ↓
Agent 生成 Python 剪辑脚本
      ↓
JyProject / pyJianYingDraft 写入剪映草稿 JSON 与素材副本
      ↓
剪映桌面版刷新并打开工程
      ↓
人工审片、修改、预览、导出
```

仓库不是通过剪映公开 API 操作完整 UI，而是生成剪映能读取的草稿结构；Windows 的自动导出另使用 `uiautomation` 模拟 UI。这样做的优点是时间轴和参数可以由脚本精确复现，缺点是剪映草稿格式、版本、路径和 UI 行为变化都会影响兼容性。

## 5. 能做什么，不能做什么

### 能力

- 导入视频、音频和图片，按源时间截取并排列到多条轨道。
- 生成普通字幕、SRT 字幕、花字、文字动画和关键帧。
- 调用剪映原生音色或 Edge TTS 生成旁白；`add_narrated_subtitles` 可把配音、拆句和字幕对齐放进一个流程。
- 从本地或已索引的剪映云素材库添加音乐、音效、视频、滤镜、特效和转场。
- 把 HTML/JavaScript/Canvas/SVG 动画通过 Playwright 录成视频素材。
- 录制屏幕，并根据鼠标操作生成缩放、红圈和关键帧效果。
- 用多模态模型先生成时间点/故事板 JSON，再用 `movie_commentary_builder.py` 或类似脚本落成剪映草稿。
- 对草稿做路径安全、素材暂存、媒体信息解析、草稿自修复和 Draft Inspector 检查。

### 边界

- 最终预览与渲染依赖剪映本身；Skill 生成的是可编辑工程。
- “看懂视频”不是本地 Skill 单独完成的能力。`smart_rough_cut.py` 示例还依赖外部的 `AntigravityClient` 和多模态模型；没有视觉模型时，它不能凭空理解长视频内容。
- 原视频的语音识别需要额外 ASR 工具或现成 SRT。TTS 生成配音并对齐字幕，不等于识别原素材中的讲话；`movie_commentary_builder.py` 主要消费故事板，当前代码没有直接完成 TTS 合成。
- 原生音色走内部 SAMI 服务，失败时回退 Edge TTS；网络、服务变化和账号条件会影响结果，不能保证每种剪映音色都可稳定使用。
- 剪映 GPU 实时能力，如智能抠图、美颜、剪映内置语音识别字幕，以及“一键成片”“图文成片”等完整 UI 功能，不能按普通 Python API 直接调用。
- 自动导出是 Windows UI 自动化，不是稳定的无头渲染器；剪映弹窗、焦点或控件变化都可能造成失败。
- 云端音乐、音效和素材需要本机已有的缓存/登录状态或仓库提供的索引与下载流程；不能把任意网络素材当成永久可用资源。

## 6. 推荐安装方式

### 在 Codex 或通用项目中

以下是 Windows 项目级手动安装方案。准备 Git、64 位 Python 3.12、FFmpeg 和 ffprobe，并让两个媒体命令在 PATH 中可调用；建议 3.12 是根据当前固定依赖选择的起点，不是本次实际安装通过的结论。先在剪映中新建并保存一次空工程，确认草稿目录。

在剪辑项目根目录执行，后续命令也保持在此目录。把 Skill 放在 `.agents/skills/jianying-editor`，业务脚本放在项目根目录或单独的 `scripts/`：

```powershell
git clone https://github.com/luoluoluo22/jianying-editor-skill.git .agents/skills/jianying-editor
py -3.12 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r .agents\skills\jianying-editor\requirements.txt
# 只有需要 Web-to-Video 时才安装浏览器
.\.venv\Scripts\python.exe -m playwright install chromium
```

仓库的路径探测代码会优先检查 `JY_SKILL_ROOT`，再检查 `.agents/skills/jianying-editor`、`.agent/skills/jianying-editor`、`.trae/skills/jianying-editor`、`.claude/skills/jianying-editor` 和 `skills/jianying-editor`。因此同一份源码可以被不同 Agent 编辑器复用。若 Python 启动器不是 `py`，使用指向完整 Python 安装的 `python`；不要使用 WindowsApps 的存根解释器。

这里能确认的是仓库脚本接受 `.agents` 路径；Codex 从技能发现到实际剪辑的完整流程没有实测，兼容性 Issue #25 仍未关闭。可以明确要求 Agent 读取该目录的 `SKILL.md`，并使用项目的 `.venv\Scripts\python.exe`。macOS 安装应另按平台文档核对依赖，本文 PowerShell 命令不能直接照搬。

### 剪映与草稿目录

- Windows 默认草稿目录：`%LOCALAPPDATA%\JianyingPro\User Data\Projects\com.lveditor.draft`
- macOS 默认草稿目录：`~/Movies/JianyingPro/User Data/Projects/com.lveditor.draft`
- 目录不一致时设置 `JY_PROJECTS_ROOT`，或在任务里明确告诉 Agent 实际路径。

草稿生成优先支持 `draft_info.json` 的新版结构，并在 v1.7 中加入了素材自包含和媒体丢失修复；这不等于所有剪映版本都能打开每种草稿。生成后剪映列表不刷新时，重启剪映或进入旧草稿再返回。

## 7. 最小验证流程

用测试素材做一个 5 秒视频 + 音频 + 两段文字的小样。注意：`api_validator.py` 会创建工程，默认名称为 `Diagnostic_Test`，并使用 `overwrite=True`；它不是只读检查。以下给它生成独特名称，且用 `--strict` 提前拦截缺少素材或 ffprobe 的情况：

```powershell
$skill = "$PWD\.agents\skills\jianying-editor"
$python = "$PWD\.venv\Scripts\python.exe"
$testDraft = "Diagnostic_" + (Get-Date -Format "yyyyMMdd_HHmmss")
& $python "$skill\scripts\api_validator.py" --project $testDraft --strict --json
# 确认诊断成功，再运行示例
& $python "$skill\examples\simple_clip_demo.py"
& $python "$skill\scripts\draft_inspector.py" list --limit 20
& $python "$skill\scripts\draft_inspector.py" summary --name "Hello_JianYing_V3"
```

示例依赖仓库的 `assets/video.mp4` 和 `assets/audio.mp3`，缺失时会直接返回。它会覆盖同名 `Hello_JianYing_V3`，运行前确认这个名称没有需要保留的内容，或先在示例副本里改名。更一般地，`JyProject` 默认覆盖同名工程，即便设为 `overwrite=False`，模板加载失败也有强制重建分支；正式工作应使用独特新名称和工程副本。

验收标准：脚本保存成功；草稿目录出现新项目；剪映能打开工程；视频、音频和文字轨道都存在；素材没有“媒体丢失”；画幅与预览符合预期。脚本退出码正常不代表剪映实际打开成功。先手动导出一个 MP4；Windows 上如需自动化，再单独测试：

```powershell
& $python "$skill\scripts\auto_exporter.py" "Hello_JianYing_V3" "$PWD\output.mp4" --res 1080 --fps 30 --json
```

自动导出测试期间不要操作鼠标键盘；macOS 不运行这一步，直接在剪映里手动导出。

## 8. 实际剪辑的推荐工作流

1. 先让 Agent 检查环境、剪映版本、草稿路径和素材清单。
2. 先做“原声粗剪 + 标题 + 本地 BGM”，确认剪映能打开和素材不丢失；需要逐句字幕时提供 SRT 或另接语音识别。
3. 再加 TTS、花字、滤镜、转场和关键帧；每次只增加一类能力，出错时容易定位。
4. 如果要做影视解说或素材匹配，先让多模态模型输出可审阅的故事板 JSON，再生成草稿；不要让 Agent 直接对长视频做不可审查的全自动修改。
5. 在剪映里检查片段边界、字幕断句、声音响度、BGM 遮盖和画幅，最后手动导出；自动导出在独立验证通过后再加入流程。

推荐给 Agent 的任务描述应包含输入目录、目标画幅、目标时长、保留/删除规则、字幕样式、BGM 来源、输出草稿名和验收项。例如：

> “读取并使用 `.agents/skills/jianying-editor/SKILL.md`，用 `.venv\Scripts\python.exe` 执行脚本。读取 `D:\素材\露营` 的视频和照片，先列出素材和建议片段清单，再生成 9:16、约 60 秒的 Vlog 草稿；优先原声，BGM 用本地 `music.mp3`，字幕用我提供的 SRT。新建独特名称的工程，完成后检查轨道、媒体路径和总时长，并告诉我在剪映中打开哪个草稿。素材不够时报告实际时长。”

如果 Agent 没有视频理解能力，把“建议片段清单”换成你指定的文件名与起止时间。v1.7 的视频素材暂存机制会复制文件进草稿目录，需预留相应磁盘空间；直接 `add_audio_safe` 不经过同样的暂存流程，音频路径也要核对。

## 9. 版本、Issue 与许可证判断

- 主仓库 README 目前同时表达“支持新版草稿”和“Windows 老版本自动导出最稳”。这两个结论并不矛盾：草稿写入层和 UI 导出层的兼容范围不同。
- 未关闭的 Issue 仍包括“新版草稿能否用 5.9 导出”“是否兼容 Codex”“剪映自动更新干扰”等问题；说明社区仍在处理版本与 Agent 适配细节。
- Issue 中维护者明确表示，本项目是剪辑执行器，视频理解需要多模态模型；这支持了“Skill 负责落地时间轴，模型负责分析内容”的分工判断。
- 主仓库根目录声明 MIT；`scripts/vendor/pyJianYingDraft` 另带 Apache-2.0。使用仓库代码时应保留相应版权和许可证文本。剪映云素材、TTS 服务、模型权重和你导入的素材仍有各自的使用条款。

## 10. 最终建议

- **适合现在试用**：Windows 或 macOS 桌面剪映用户，想让 Agent 自动搭时间轴、批量做字幕/配音、做教程录屏或生成可继续编辑的模板。
- **兼容选型**：先记录当前剪映完整版本，用主仓库测试一个新草稿和手动导出，不因为旧教程就直接降级。Windows 5.9 及以下的 UI 自动导出较有文档支持；新版是否可用以本机实测为准。
- **Windows 11.5 候选**：如果当前版本恰为 `11.5.0.14471`，可另评估 `jjjane233/jianying-editor-skill-11.5`。作者声称通过本机 `videoeditor.dll` 的 DraftIO 读写加密草稿，支持 `cut/build/patch/inspect/doctor/repair`；本次只核对其 README，没有复核 DLL 调用或实机结果。它仅声明这一版本和 64 位 Python 3.10+，不能外推到其他版本或 macOS。`cut --prompt` 是规则解析，`render` 是基础 FFmpeg 单轨渲染，完整成片仍应在剪映中手动导出；部分 AI/超分功能会上传素材并消耗账号点数，人机需分阶段编辑同一工程。
- **第一次正式任务**：只做素材导入、按明确片段清单粗剪、标题和本地 BGM；确认草稿和媒体路径稳定后，再增加 ASR 字幕、多模态选段、TTS、特效和自动导出。

## 11. 关键资料

- [主仓库 README](https://github.com/luoluoluo22/jianying-editor-skill)
- [主仓库 SKILL.md](https://github.com/luoluoluo22/jianying-editor-skill/blob/main/SKILL.md)
- [最小命令 SOP](https://github.com/luoluoluo22/jianying-editor-skill/blob/main/docs/minimal-command-sop.md)
- [核心操作规则](https://github.com/luoluoluo22/jianying-editor-skill/blob/main/rules/core.md)
- [音频与配音规则](https://github.com/luoluoluo22/jianying-editor-skill/blob/main/rules/audio-voice.md)
- [影视解说脚本](https://github.com/luoluoluo22/jianying-editor-skill/blob/main/scripts/movie_commentary_builder.py)
- [主仓库 Issue #25：Codex 兼容性](https://github.com/luoluoluo22/jianying-editor-skill/issues/25)
- [主仓库 Issue #26：新版草稿与 5.9 导出](https://github.com/luoluoluo22/jianying-editor-skill/issues/26)
- [主仓库 Issue #19：剪辑执行器与视频理解的分工](https://github.com/luoluoluo22/jianying-editor-skill/issues/19)
- [固定源码快照](https://github.com/luoluoluo22/jianying-editor-skill/tree/32c56928ded4f9e2c2b80e099dc7abb793d2c30b)
- [工程初始化与覆盖逻辑](https://github.com/luoluoluo22/jianying-editor-skill/blob/32c56928ded4f9e2c2b80e099dc7abb793d2c30b/scripts/core/project_base.py)
- [诊断脚本的草稿写入行为](https://github.com/luoluoluo22/jianying-editor-skill/blob/32c56928ded4f9e2c2b80e099dc7abb793d2c30b/scripts/api_validator.py)
- [Skill 路径探测代码](https://github.com/luoluoluo22/jianying-editor-skill/blob/32c56928ded4f9e2c2b80e099dc7abb793d2c30b/scripts/utils/skill_path.py)
- [Windows 11.5 适配候选 README](https://github.com/jjjane233/jianying-editor-skill-11.5)
- [B 站入门视频：我老萝卜](https://www.bilibili.com/video/BV1hLzCBzEDS/)
- [B 站 Trae 安装教程](https://www.bilibili.com/video/BV1SoFNzUES6/)

