---
name: historical-figure-video-kit
description: Turn a historical figure's name into a complete local short-video package and saved CapCut Mate draft: title, eight life subtitles, eight image prompts, eight images, seven ordered videos, captions, music, and a draft link.
metadata:
  short-description: 输入历史人物姓名，自动完成文案、画面、图片、视频并保存本地
---

# 历史名人短视频全流程技能包

当用户输入一个历史人物姓名，直接完成从资料核验到本地素材归档的完整流程，不要先追问。用户另有标题、段数、画风、模型、时长或保存目录要求时，以本轮要求为准。

## 交付内容

在本地创建 `历史人物爆款短视频/人物名/` 文件夹，至少保存：

- `人物名-短视频脚本.md`：视频标题、8 段生平字幕、8 条对应画面提示词。
- `图片/01-人物名.jpg` 至 `08-人物名.jpg`：按生平顺序生成的竖版图片。
- `视频/01-人物名-第1段.mp4` 至 `07-人物名-第7段.mp4`：相邻图片首尾帧生成的视频。
- `剪映草稿记录.md`：草稿 ID、最终草稿链接、时间线、字幕和音乐记录。
- `生成记录.md`：模型、参数、任务状态、失败重试记录和文件清单。

如果用户指定了已有文件夹，直接使用该目录，不要另建平行目录。

## 内容生成规则

1. **标题**：简洁、有叙事性，避免夸张和无史实依据的断言。
2. **生平字幕**：固定 8 段，按时间顺序覆盖成长、求学或立志、转折、代表性成就、重大变故、晚年与身后影响。每段使用 `［年龄］事件`，文字短小，适合配音和屏幕阅读。
3. **画面提示词**：与 8 段字幕一一对应；每条不少于 50 个汉字，写明人物阶段、身份服饰、姿态、单一场景、道具、建筑环境、光线、构图和氛围。只写一个静态瞬间，不写连续动作、分镜序列或多个场景。每条末尾逐字加 `真实古代纪录片风格。`
4. **史实**：核验生卒年、年龄、官职、战争和争议事件；不确定年龄用“约”或年龄范围；不把传说写成确证史实；视觉化想象不得冒充史实；避免现代物品和时代错置。

## 图片生成

读取 [references/image_generation.md](references/image_generation.md)。默认使用 Seedream 5.0 Flash，竖版 `1152x2048`，一条提示词生成一张图。每张图按字幕编号命名并下载到 `图片/`。生成失败时记录错误并按原提示词重试一次，不要用占位图冒充结果。

API Key 只从用户指定的配置文件或环境变量读取，绝不写入技能包、输出文件或回复。若 Key 缺失，说明需要用户配置后停止媒体生成，但仍可先保存文案。

## 视频生成

读取 [references/video_generation.md](references/video_generation.md)。图片全部成功后，按严格相邻顺序生成 7 段：

- 第 1 段：图片 1 → 图片 2
- 第 2 段：图片 2 → 图片 3
- ……
- 第 7 段：图片 7 → 图片 8

每个任务使用 `first_frame` 和 `last_frame` 两个 `image_url`，5 秒、`ratio=adaptive`、默认关闭音频和水印。7 个任务可以并行提交和轮询，但文件编号必须保持顺序。模型未开通或任务失败时，先重试失败段；仍失败则保留已成功文件和明确的失败记录，不声称全部完成。

## 剪映草稿

读取 [references/capcut_api_workflow.md](references/capcut_api_workflow.md)。当 7 段视频生成成功后，自动创建并保存一个 1080×1920 的 CapCut Mate 草稿：

1. `create_draft` 创建竖屏草稿。
2. `add_videos` 按 01–07 顺序加入视频。每段默认 5 秒，使用微秒时间线：第 n 段的 `start=(n-1)*5000000`、`end=n*5000000`，总时长 35000000 微秒；`scene_timelines` 与 `video_infos` 一一对应。
3. 第一次 `add_captions` 添加标题，覆盖 0–35000000 微秒，放在上方：`font=江湖体`、`font_size=15`、`line_spacing=10`、`text_color=#ffffff`、`border_color=#000000`、`transform_y=1369`。
4. 第二次 `add_captions` 添加生平字幕，按视频内容覆盖对应时间段，放在下方：`font=江湖体`、`font_size=12`、`line_spacing=10`、`text_color=#ffde00`、`border_color=#000000`、`transform_y=-794.5`。8 条生平经历与 7 段视频不等长时，将最后一段视频拆成两个字幕区间，确保 8 条经历全部出现且不超出 35 秒。
5. `add_audios` 添加背景音乐，时间线固定为 0–35000000 微秒。中国历史男性人物从已验证可下载的男性音乐中随机选一首；中国历史女性、历史名将和国外历史人物使用对应音乐列表。先下载音频到人物本地文件夹并核对格式、时长，再提交其可访问 URL。音频时长不足时按 API 支持的方式循环或裁剪到总时长，并在记录中注明。
6. `save_draft` 保存草稿，记录并返回 `draft_url` 和 `draft_id`。不要只返回 API 文件列表链接；用户需要的是可在剪映小助手中打开的草稿链接或 draft_id。

若剪映素材 URL 已过期，先查询即梦任务状态恢复原始视频 URL；只有在无法恢复时才尝试无需注册的临时直链服务。临时上传失败不应阻塞文案和本地素材交付，并须在记录中说明。

用户反馈剪映小助手下载失败、媒体丢失或无声时，读取 [references/capcut_api_workflow.md](references/capcut_api_workflow.md) 的“失效素材与重新生成”流程。不要仅修改本机草稿后仍返回旧的云端链接；小助手重新获取旧草稿会覆盖本地修改。需要新链接时，验证或恢复所有媒体 URL，重新执行 `create_draft → add_videos → add_captions` 两次 `→ add_audios → save_draft`，核对新草稿后返回新的 `draft_id` 和完整链接。

## 保存与汇报

生成完成后核对文件数量、文件格式和顺序，写入 `生成记录.md`，再向用户返回本地文件夹链接和完成情况。不要在回复中展示 API Key、任务请求头或完整敏感配置。

## 默认输出版式

```text
# 人物名｜短视频脚本

## 视频标题
...

## 生平经历与字幕
1. ［年龄］事件
...

## 逐段静态画面提示词
1. 对应字幕：...
   画面提示词：...真实古代纪录片风格。
...
```
