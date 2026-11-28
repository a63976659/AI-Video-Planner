# AI Video Planner

[简体中文](./README.md) · [English](./README_EN.md)

> **Lead｜Break one long video into segments you can actually execute.**
>
> A ComfyUI plugin: AI handles planning and prompts, the official MiniMax H3 nodes handle generation, and this plugin handles splitting, continuation, joining and post-production handoff.

<img width="3830" height="1877" alt="AI视频规划师界面" src="https://github.com/user-attachments/assets/58fd4209-1137-4232-b772-d519727b6062" />

*Fig 1.1｜The AI Video Planner interface: from brief and shot list to timeline and video production.*

> **Good video = a clear goal × the right structure × executable shots × controlled rhythm.**

## Contents

> **00 / Lead｜Pick a route first, then read the details.** This guide runs in the order environment → creation → materials → reuse → saving → optimisation. On a first read, go straight through from chapter 01.

| Chapter | Topic | Read it when |
|---|---|---|
| **01** | [Install](#install) | Installing for the first time, or after a plugin update |
| **02** | [Get started](#get-started) | You want to complete one video end to end |
| **03** | [Skills creative references](#skills-creative-references) | You want to understand or extend the creative rules |
| **04** | [Materials and models](#materials-and-models) | Configuring AI services, reference images or materials |
| **05** | [Edit one segment and reuse results](#edit-one-segment-and-reuse-results) | Already generated, and you only want to change part |
| **06** | [Save and restore](#save-and-restore) | Saving, sharing or restoring a project |
| **07** | [Upscaling and acceleration](#upscaling-and-acceleration) | You want better picture or faster generation |
| **08** | [Common questions](#common-questions) | You hit a save, request or generation problem |
| **09** | [Guides and feedback](#guides-and-feedback) | Customising references or filing an issue |
| **10** | [Changelog](#changelog) | Checking what changed |
| **Appendix** | [Acknowledgments](#acknowledgments) · [Support the author](#support-the-author) | Sources, licences and how to back the project |

---

## Install

> **01 / Lead｜Get the environment running before chasing length or picture quality.** This chapter covers installation, dependencies and startup only; confirm the models and nodes with a short clip first.

You need ComfyUI with its official MiniMax H3 nodes, an H3 model, a Qwen3-VL encoder, and video and audio VAEs. Start with a short clip to check your setup before increasing duration or resolution.

1. Download this repository into `ComfyUI/custom_nodes/ComfyUI-AI-Video-Planner/`, or use the commands below.
2. Install dependencies from the plugin folder using **the Python environment that runs ComfyUI**. For a portable installation, use its bundled Python.
3. Restart ComfyUI and refresh the browser. Search for **AI视频规划师** and check that the matching panel appears at the bottom of the page.

```bash
cd ComfyUI/custom_nodes
git clone https://github.com/a63976659/ComfyUI-AI-Video-Planner
cd ComfyUI-AI-Video-Planner
python -m pip install -r requirements.txt
```

Restart ComfyUI and refresh the browser after updates too.

<details>
<summary>Optional features and dependencies</summary>

For video dependencies, uncomment the relevant entries in `requirements.txt` and install them. Each local voice model has its own `.venv-shared` while reusing the current ComfyUI Python, Torch and GPU runtime.

| Feature | What you need |
|---|---|
| Read reference videos | `imageio`, plus `imageio-ffmpeg` or `av` |
| Video editing and export | Full FFmpeg and `ffprobe`, including `libx264`, AAC, `libass` and Chinese fonts |
| Qwen3-TTS local voices | [Qwen-TTS setup](文档/技术实现/Qwen-TTS本地配音.md), with models in `ComfyUI/models/TTS/` |
| IndexTTS-2.5 local voices | [IndexTTS-2.5 setup](文档/技术实现/IndexTTS-2.5本地配音.md), with models in `ComfyUI/models/TTS/IndexTTS-2/` |
| Breeze-TTS-2 local voices | [Breeze-TTS-2 setup](文档/技术实现/Breeze-TTS-2本地配音.md), with models in `ComfyUI/models/TTS/Breeze-TTS-2/` |

All three local voice models use the same [installer and shared-host environment rules](文档/技术实现/本地配音便携环境.md). Each model can be installed or checked separately, and paths are updated automatically after moving the installation.

</details>

> **One-line takeaway｜Confirm the environment with a short clip, then add duration and resolution.**

---

## Get started

> **02 / Lead｜Four steps close the smallest complete creative loop.** Load a workflow, plan, build the timeline, then produce the video. Every step saves, so nothing has to be right on the first pass.

### 1. Load a workflow

For your first run, load the [basic workflow](./workflows/AI长视频规划师V3.6.json), select your downloaded models, then open the **AI视频规划师** panel at the bottom.

| Tab | Purpose |
|---|---|
| **AI规划 — AI Planning** | Upload materials and describe what you want; AI plans the shots |
| **提示词时间轴 — Prompt Timeline** | Adjust shot order and duration, then generate prompts |
| **提示词编辑 — Prompt Editor** | Edit segments on the node and choose generation modes and references |
| **视频制作 — Video Production** | Receive generated videos, trim, adjust sound, add subtitles and music, then export |

**Method card｜The minimum creative loop**

| Module | Content |
|---|---|
| **Core** | Finish one playable, saveable short clip before adding complexity |
| **Formula** | Load a workflow → generate the shot list → adjust and fill the node → run and produce the video |
| **Example** | On a first run, use few materials and a short duration to validate the whole flow |
| **Fits** | Environment checks, onboarding, quick retries after swapping models |

![Creative loop overview](./文档/插图/创作总流程_EN.svg)

*Fig 2.1｜The minimum loop from materials and shot list to prompts and finished video.*

### 2. Plan your shots

Upload materials in **AI规划**, choose a purpose or leave automatic detection selected, and enter your idea, source text and anything that must be kept. Type `@` to reference an uploaded item.

**Total duration is entered in minutes and defaults to 0 for automatic allocation.** A nonzero value sets a target total duration; the tool reports when the content or dialogue cannot fit.

Click **生成规划** to run **Analyze materials → Plan episodes → Generate references → Generate shots**. Image generation is skipped when automatic reference images are disabled. Completed stages and shots are saved; click a stage to inspect its results.

You can also skip AI planning and add segments and prompts manually on the node timeline.

### 3. Edit and fill the node

Review the draft in **提示词时间轴**. Drag cards to change order or duration, split or delete shots, and add segments as needed. Zoom and horizontal navigation help with longer timelines.

Enable the segments you want to process and click **AI生成提示词**. The backend sends one request for each target segment and returns results segment by segment. You can stop generation and keep completed results. When ready, click **填充节点**, then check each segment’s prompt, mode, references and first/last frames in **提示词编辑**.

- **上一步 / 下一步** keep up to 20 steps in the local `历史编辑缓存/编辑历史/` folder. Loading a workflow restores history only from a matching cache; workflow files do not include history.
- Edit and export subtitles in **视频制作**.
- **提示词优化** rewrites the current segment’s text. **审查标签** checks references that have no matching material.

### 4. Run and finish the video

Click ComfyUI’s **Run / Queue Prompt**. Each completed segment is saved to disk, then all segments are decoded into frames and joined. Progress appears in the bottom panel.

**视频制作** automatically receives videos generated by the current workflow. Trim and reorder clips, adjust existing sound, add subtitles and background music, then preview and export. The first imported video sets the canvas; later clips use the same canvas settings.

The subtitle editor provides three entries: **当前片段字幕 (current-clip subtitles)**, **全片字幕 (full-film subtitles)**, and **文字叠加 (text overlays)**. Style controls share the top row with the tabs, and subtitle/text cards use a compact layout with start/end fields arranged in pairs. Start and end times for all three types are limited to two decimal places; extra digits are truncated rather than rounded. Clip duration, voice-over timing, and audio-track timing are unchanged.

See the [video editing guide (Chinese)](./文档/技术实现/2026-10-05-视频制作后期调整说明.md) for details.

<details>
<summary>Connect the node manually</summary>

| Input | Connection |
|---|---|
| `fl2va模型` | Model for text, image or first/last-frame video; connect when needed |
| `ref2va模型` | Model for reference-based or video-to-video tasks; connect when needed |
| `CLIP编码器` | Qwen3-VL encoder of the minimax type; required |
| `视频VAE` / `音频VAE` | Video and audio models; both required |
| `SelfLift配置` | Optional; see upscaling and acceleration below |

Outputs include images, audio, frame rate, total frames and a report for preview or saving. **初采产物** connects to the optional **放大和重绘** node.

</details>

> **One-line takeaway｜Get a short video through first, then add duration, resolution and post-production complexity.**

---

## Skills creative references

> **03 / Lead｜Let each piece of creative guidance do its own job.** Task skills decide how content is understood, shared capabilities provide general methods, specialty rules fix specific problems, and formula cards add structure on request.

`Skills/` is the creative-rules library shipped with the plugin. During planning the AI picks shared capabilities, specialty rules and task skills automatically from the video brief — you never choose skills by hand.

![Skills layers](./文档/插图/Skills分层_EN.svg)

*Fig 3.1｜Creative references are assembled by task and stage; the program owns requests, scheduling, checks and saving.*

### What it does well

- **Assembled on demand** — material type, target genre, rewrite mode, planning stage and trigger conditions decide what is loaded. Core text is loaded once without duplication; supporting details are read per task.
- **Layered rules** — task skills, shared capabilities, specialty rules and formula cards are maintained separately, so there are general methods and topic- or problem-specific capabilities side by side.
- **Clear boundaries** — your requirements, source-text facts, product facts, duration and the sound plan outrank formulas or style suggestions; materials, coverage, capacity and formatting are still validated by the program.
- **Defined responsibility** — Skills only provide creative method; they run no code. Model requests, task scheduling, result validation and save/restore belong to the plugin.
- **Easy to extend** — add specialty rules, references or formula cards inside the existing folders without touching other request types.

### Capabilities

- Recognises ideas, novels, scripts, explainers and product briefs, then sets the planning route and rewrite approach.
- Arranges content structure, episodes, scenes, shot order, duration, dialogue capacity and what is actually executable.
- Designs characters, locations, props, action causality, camera language, reference sheets and spatial rehearsal.
- Handles character and location continuity, dialogue and sound, model fit and finished-video problem prevention.
- When a fixed narrative skeleton is needed it can supply macro structures such as the **模版公式** (template formula), without overriding your established facts or main structure.

### Coverage

- **Task skills**: idea development, novel adaptation, script fidelity, explainer, product showcase, requirement discovery.
- **Shared capabilities**: storyboard structure, general shot design, character and location continuity, duration and feasibility.
- **Specialty rules**: world rules and space, character design, content creativity, action and combat, reference-sheet rehearsal, scene-level narrative, finished-video prevention, model adaptation, object transformation, performance and sound, camera language, genre expression.
- **Formula cards**: the general video formula, with room for more cards later.

Ordinary stories and ordinary ads do not get a formula applied automatically; formula cards are used only when **模版公式** is checked or you explicitly ask for a formula, narrative skeleton or structure template.

Folder layout and customisation are covered in [Skills 目录说明](./Skills/目录说明.md) and [Skills 目录与装配说明](./文档/技术实现/Skills目录与装配说明.md) (Chinese).

> **One-line takeaway｜Skills supply the method; the program handles execution and validation.**

---

## Materials and models

> **04 / Lead｜Connect the service, the materials and the generation entry points first.** Models understand and generate; the plugin puts materials in the right place and checks whether the capability matches.

### Configure the AI service

Open **ComfyUI Settings → 🎬 AI视频规划师 → API 模型配置**. Enter the service address, model name and API key, test the connection, and save. Preset services and custom cloud or local services with an OpenAI-compatible interface are supported.

**Opening-frame evidence repair** uses the currently selected **AI规划模型** and no longer loads the local `LLM/Qwen3.5-9B` judgement model. Only when the first narrative shot's positioning clue cannot be confirmed directly from its opening frame does the plugin make one targeted request. It accepts only a verbatim, continuous quote of 1–360 characters from that opening frame, writes it back, and reruns the existing validation; otherwise the original error remains. `/lvp/llm_test` only tests connectivity for the selected API model.

You can choose separate models for **AI规划模型**, **时间轴改写模型** and **提示词优化模型**. Unassigned functions use the default model. The planning model must support your material types; both prompt rewriting tools send text only.

Adjust the request timeout for slower models. Timeline rewriting and editor optimization each wait at most 600 seconds per request, or the configured timeout if shorter.

### Generate planning reference images with a ComfyUI workflow

1. Prepare a local ComfyUI workflow that outputs a single image. Use **Save Image** or **Preview Image** as the output; when the workflow has several image outputs, name the target node **LVP 输出图**.
2. Save the JSON with **Export (API)** and put the file in this plugin’s [workflows/生图](./workflows/生图) folder.
3. In **API 模型配置 → 图像模型**, choose **ComfyUI 本地工作流**, select the **文生图工作流** and save. With **生成参考图** enabled on the planning page, the job runs in the current ComfyUI queue and the pictures join the planning materials.

Each character gets one horizontal four-view identity sheet from a single image-model run, left to right: a front face-and-neck close-up, a full-body front view, a full-body profile with the nose pointing left, and a full-body back view. The prompt keeps the three full-body views at the same scale with head and feet aligned, arms relaxed and opened about 15°. The four views serve as identity reference; first and last frames use separate shot images. Landscape canvases are recommended for text-to-image and image-to-image workflows.

Each prop gets one image per run. When the story only shows a photo or poster, generate one complete front view. When the physical structure matters, show front, side and back in order. When the story opens, closes or folds it, show each key state in separate rows of the same image, and shots reference the state in use. Photos or patterns printed on a prop turn with the object and keep their own composition.

Configure **图生图工作流** as well when the workflow needs to draw from an uploaded image.

- The planner follows the sampler’s forward links to find a single unique text-encoding entry. Common CLIP, SDXL, SD3, FLUX and Qwen image encoder nodes take the text directly; nodes with several text boxes receive it in each forward box. Negative prompts stay as the workflow set them. When the Qwen Image 2.1 negative box is left empty, character and prop sheets exclude text, borders and panel lines by default; scene images exclude text, watermarks, annotations and collages.
- When the encoder node has a recognisable image port, the planner connects your selected images in order, reusing existing load and processing chains, and empty ports may stay connected. If reference images only travel an independent branch, or their source cannot be determined, name the matching **Load Image** nodes **LVP 参考图 1**, **LVP 参考图 2** and so on, one per image selected that run.
- When the forward entry cannot be resolved uniquely, or forward text passes through processing links you want to keep, set the target text input to `{{LVP_PROMPT}}`.

Auto-fill applies only to the job queued in that run. Models, sizes and sampling parameters in the workflow run exactly as the file specifies.

### Planning materials and segment references

| Location | Purpose |
|---|---|
| **AI Planning uploads** | Help AI understand and plan content; accepts images, video, audio and text |
| **Segment references** | Used for video generation; H3 supports up to 9 reference images, 3 reference videos and 3 reference audio files per segment; first/last frames depend on the task |
| **Video Production clips** | Hold generated results for editing; received videos and voices are added as production clips |

**Clearing planning materials removes their bindings in the draft.** Uploading files with the same names does not restore those references automatically. Before rewriting prompts or filling the node, check each segment’s materials, first/last frames and reference tags. Clearing the list does not delete original files already saved on disk.

With the official DeepSeek service, images are uploaded and reused within the same planning task, supporting **up to 600 images per request**. Uploaded files are set to **expire after 7 days**; the tool attempts early cleanup when the task finishes, fails or is cancelled. Format, size and total upload limits still depend on the service. Upload failures show the reason without automatically switching to another sending method. Official guides: [Image understanding](https://api-docs.deepseek.com/zh-cn/guides/vision/) · [File uploads](https://api-docs.deepseek.com/zh-cn/guides/files_api/).

The Bailian Beijing service supports uploading complete images, videos and audio using your existing key, without a separate storage account. Other services have different capabilities; see the [material upload guide (Chinese)](./文档/技术实现/2026-10-07-规划素材原件发送.md).

<details>
<summary>Shared materials and source-video audio</summary>

These controls apply to reference-based video, video-to-video and reference-plus-video tasks:

- **全段共用**: read video frames, their soundtracks and ordinary reference audio in consecutive segment windows. When off, each segment takes its own duration from the beginning. Frame-grid padding repeats only the last frame in that window. Explicit dialogue and background tracks keep their own arrangement.
- **禁用**: open a video preview and enable this switch to exclude only that video’s soundtrack from model references. Its images remain available.
- **原声**: enable this switch in a video or audio preview to use that material’s original audio as the output soundtrack. Reference audio remains enabled unless the video’s separate **禁用** switch is on. Requires a node frame rate of 24; sound is trimmed or padded with silence to match the picture.

Only one video or audio in a material group can supply the output soundtrack; selecting another replaces the previous choice. Audio previews have no disable switch. Turning off **原声** restores normal output behavior. Videos with sound occupy audio reference numbers; check the prompt’s `@` list.

See the [reference video and audio guide (Chinese)](./文档/技术实现/2026-10-05-参考视频音轨接入修复.md).

</details>

> **One-line takeaway｜Planning materials inform the content, segment references drive generation, production clips serve post-production.**

---

## Edit one segment and reuse results

> **05 / Lead｜Shrink the redo to the segment that actually changed.** Keep everything else constant so existing results can be reused; add continuation or transitions later, and then redo only what the dependency chain requires.

Existing results are reused when parameters, materials, models and **重新生成编号** (regeneration number) match. **A segment’s Run switch allows generation; it does not force a new result. A disabled segment still joins the video if a matching result exists.**

**Method card｜Local rework**

| Module | Content |
|---|---|
| **Core** | Regenerate only the target segment; every other segment reuses its matching result |
| **Formula** | Edit the target segment → enable Run for it only → keep the regeneration number → run |
| **Example** | With **渐变过渡** off, change only the second segment’s prompt and leave the rest alone |
| **Fits** | Partial rework, dialogue tweaks, swapping the material of one segment |

![Single-segment reuse](./文档/插图/单段结果复用_EN.svg)

*Fig 5.1｜The target segment is regenerated, other segments reuse matching results, and everything is joined into the finished video.*

### Change just one segment

This applies when the original video was generated with **渐变过渡 turned off**:

1. Change the target segment’s prompt or its own references. Keep other segments, ordering, durations and global settings unchanged.
2. Enable Run only for the target segment and disable the others. **Keep 重新生成编号 unchanged.**
3. Run the workflow. The target segment is regenerated and the others reuse matching results; this run already produces the complete video.

Other segments need saved matching results, or they will be omitted. Enabling all segments again still reuses matching results. Reading results from disk also requires decoding and joining; the time depends on video length and resolution.

### Regeneration number and segment continuity

| Setting | Effect |
|---|---|
| **重新生成编号 +1** | Regenerate enabled segments on the next run; applies to the whole node and does not start a run immediately |
| **渐变过渡 off** | Generate segments independently for easier individual edits |
| **渐变过渡 on** | Carry a stretch of the previous ending picture (length set by 续拍秒数: 0.2 / 0.9 / 1.6 / 2.3 s, default 0.9) and about 1 s of sound into the next segment, and only match the opening warmth and brightness of the join to that ending — the previous picture is not blended in; the segment is generated slightly longer and its head is trimmed, so what reaches the final video still equals the seconds set on the timeline; editing an earlier segment may regenerate later ones |

**渐变过渡** does not crossfade two overlapping pictures. **Toggling it changes the saved-result matching conditions for all segments, requiring regeneration.** Changing it after a run starts affects later runs only.

续拍秒数 is no longer a control on the node; pick it from the button to the right of 渐变过渡 on the timeline toolbar. You can set it while 渐变过渡 is off — it simply does not take part in generation then.

Every run writes one measured line per join into the report: cut jump, opening momentum, replay degree, colour-temperature compensation. Compare that line before and after changing the setting instead of judging by impression.
To keep the pictures for comparison, set the environment variable `H3_接缝导出=1` before starting ComfyUI; the 12 frames on each side of every join are saved as images under `%TEMP%/长视频规划师_接缝诊断/`. That folder is not cleaned up automatically — delete it yourself when done.

The cache toolbar shows disk usage and can delete saved segment results. Deletion cannot be undone. Results are stored in `%TEMP%/长视频规划师_段缓存/` by default.

> **One-line takeaway｜Local rework means changing only the target segment while every other match condition stays put.**

---

## Save and restore

> **06 / Lead｜Keep the creative state and the results together.** Workflows save the node and page content, plans save segments and material references, and the original files still need separate safekeeping.

| Method | Use it for |
|---|---|
| **Save the ComfyUI workflow** | Node connections, settings and page edits, including planning requests not yet submitted |
| **计划数据 → 保存 / 加载** | Reuse the node’s settings, segments, references and planning draft; excludes the regeneration number |
| **载入规划** | Restore original shots and stage results from history; replaces the draft, with 上一步 available to undo |
| **Settings → 数据管理** | Import, export and manage previous plans and projects |

Plans are stored in the plugin’s `计划数据/` folder. Share image, video and audio files separately; material references do not package the files themselves.

**预设** adds a reusable prompt before every segment’s text. Put `.txt` files in `预设/` or one level of subfolders, then collapse and reopen the panel to refresh the list.

**Start over:** the node timeline’s **重置** button clears the current node’s creative content after confirmation. Connections, shared model settings and disk files are kept. Stop active tasks or wait for them to finish first.

> **One-line takeaway｜The workflow owns the current project, the plan owns the reusable setup, and the original files own the material itself.**

---

## Upscaling and acceleration

> **07 / Lead｜Prove the basic flow first, then raise quality and speed according to your hardware.** Upscaling, redraw and SelfLift each have their own settings and cache; none is needed on a first install.

These features are optional. Complete the basic workflow first.

| Feature | How to use it |
|---|---|
| **放大和重绘** | Connect 初采产物 to refine or upscale pictures; has its own saved results and regeneration number |
| **SelfLift加速** | Connect SelfLift配置 to the main node; start at a smaller size and finish at full resolution. Use `euler` and leave at least one step for each stage |

SelfLift uses lip-sync sampling by default. Enable **关闭对口型** to the left of a segment’s **运行** switch to keep its single valid reference audio fixed during both sampling stages.

Download the [H3 upscaling weights](https://huggingface.co/LBH-123-AI/Minimax_h3_latent_Upscaler) yourself and put them in `ComfyUI/models/latent_upscale_models/`. Pixel upscalers go in `ComfyUI/models/upscale_models/`. Lanczos upscaling requires no model. Without an upscaling model, SelfLift requires **允许插值降级**; use that mode for previews.

Example workflows: [Redraw and upscale acceleration](./workflows/重绘放大加速版V3.6.json). Speed, quality and memory use depend on hardware and settings.

<details>
<summary>Optional 初采交接与清理 node</summary>

Connect **AI视频规划师 → 初采交接与清理 → 放大和重绘 → Save**.

It saves first-pass data, requests memory cleanup, then starts the second pass. Only one clear handoff chain is supported; batch runs, partial execution and connections across stages are unsupported. Keep the queue exclusive and avoid submitting jobs from other windows during this process.

Stopping keeps data already written to disk, but refreshing the page does not automatically resume the old handoff task. Combined testing on a real GPU, browser and complete save chain is still pending. The final complete video still requires memory.

</details>

> **One-line takeaway｜Test a short clip first, then choose an acceleration route based on VRAM, duration and picture goals.**

---

## Common questions

> **08 / Lead｜Decide which step the problem belongs to first.** Saving, requests, generation and memory each have their own boundary; keeping completed results is usually faster than restarting the whole video.

**How do I keep content after a refresh?** Save the current ComfyUI workflow, and save important setups as plans too. Loading an older workflow or plan restores its older content. Keep original materials in their expected locations.

**What if generation stops?** Completed video segments remain available for reuse; unfinished segments must be generated again. AI planning can continue unfinished work when requested. Failed prompt rewriting keeps the original text and earlier completed results without automatic resending; disable segments you do not want rewritten before retrying.

**Can I switch windows during prompt generation?** The background task can continue and reconnect when you return. If you actually change the node, segments or draft, outdated results may be discarded to protect the new content.

**What does HTTP 413 mean?** The request exceeds a limit of the service or an intermediate server. Check supported material types, size limits and the upload method. Large image collections can use the supported official DeepSeek file-upload route.

**What does an incomplete response mean?** The model service or connection ended before the full response arrived. Check the service and network, then retry.

**How much memory does a long video need?** It depends on the model, duration and resolution. Joining retains the whole video’s frames in memory, so long videos may need tens of GB. Start with a lower resolution and a shorter duration.

**How do resolution and frame rate work?** Aspect ratio and the megapixel setting determine picture size; start low. H3 generates at 24 frames per second. Changing the output frame rate changes playback speed, rather than the motion designed during generation.

**Can I connect just one model input?** Yes. Connect the model needed by your current tasks; missing required models are reported clearly.

> **One-line takeaway｜Keep what is finished, locate the failing stage, then retry only that part.**

---

## Guides and feedback

> **09 / Lead｜When you need customisation or troubleshooting, start at the matching entry point.** Creative references, provenance and licences, and technical notes are maintained separately; attach full context to a report and hide your keys.

Planning and prompt rewriting automatically load the creative references included with the plugin. No manual skill selection is needed. For customization, see the [planning references](./Skills/目录说明.md), [prompt rewriting rules](./AI生成提示词/SKILL.md) and [directory guide](./文档/技术实现/Skills目录与装配说明.md). Sources and licenses are listed in the [source records](./文档/参考/Skills来源清单.json). These guides are in Chinese.

Report problems in [Issues](https://github.com/a63976659/ComfyUI-AI-Video-Planner/issues) with steps to reproduce, the full error, your ComfyUI version, GPU and model details. Remove API keys and private content first.

> **One-line takeaway｜Read the matching guide first, then report with reproducible steps and sanitised details.**

---

## Changelog

> **10 / Lead｜Read capability changes in time order.** This records what users can notice; the chapters above stay authoritative for current usage.

<details>
<summary>Show update history</summary>

See the guide above for current instructions.

- **2026-11-28** Synchronised the subtitle-layout and time-precision guidance across the user guide, test notes, and troubleshooting notes.
- **2026-11-27** The subtitle editor in 视频制作 now uses a compact layout with no overlapping time fields and narrower font-size and position controls; start and end times for current-clip subtitles, full-film subtitles, and text overlays are limited to two decimal places, with extra digits truncated.
- **2026-11-26** AI planning now keeps opening cues consistent with opening images, uses the configured planning model for one targeted repair instead of a local judging model, and still reports the original error if repair fails; dialogue-based rearranging now lets a very short line take 1 second instead of at least 5 seconds.
- **2026-11-25** Source-video playback now continues into the next clip automatically, the production timeline can be dragged, and subtitles longer than 23 characters split at punctuation in both previews and exported videos.
- **2026-11-24** Source-video previews now show timed subtitles using the project's subtitle style.
- **2026-11-11** Added notes on the changes from the past few days.
- **2026-11-10** Videos with a fixed length now leave time for later scenes, and you can pick up where you left off using the pictures and scenes you already prepared.
- **2026-11-09** Each clip can continue the previous shot or start a new one, and rewriting its description keeps the planned length.
- **2026-11-08** This day's changes still need to be confirmed.
- **2026-11-07** One picture can show the front, side, and back of a prop, including how it looks open and closed.
- **2026-11-06** You can make pictures of characters, places, and props on your own computer, including views of a character from different angles.
- **2026-11-05** Video descriptions focus on how people move and what changes, with fewer details repeated from the pictures.
- **2026-11-04** When you switch between video setups, the bottom of the screen shows the work and progress for the one you're on.
- **2026-11-03** You can make videos through an online service and choose clip by clip whether mouth movements should follow the voice.
- **2026-11-02** This day's changes still need to be confirmed.
- **2026-11-01** The guide now credits three voice tools and explains what each does.
- **2026-10-31** You can zoom in on very short clips and see all the planned shots for a scene at once; AI prompt generation now sends a separate request for each clip and returns each result separately, runs sequentially by default with optional concurrency, and keeps finished results when stopped.
- **2026-10-30** A video plan comes back in one piece, names the files it uses, and lists speech separately from background sounds.
- **2026-10-29** Smooth transitions bring neighboring clips closer in brightness and color and give each clip its planned length.
- **2026-10-28** The tool fixes small mistakes like extra commas and tells you what went wrong if a clip's description still fails.
- **2026-10-26** A new voice option runs on your own computer and can add laughter, sighs, or coughs to lines.
- **2026-10-25** You can use an existing recording for a character's voice and start listening wherever you choose.
- **2026-10-24** A separate voice window lets you edit lines, make new recordings, or choose ones you've saved.
- **2026-10-23** Changing a clip's length moves the following clips into place and updates the total time.
- **2026-10-22** Planning looks up guidance that fits your idea and uses the same material when you pick up where you left off.
- **2026-10-21** If planning stops with an error, you can keep the finished work and pick up where you left off.
- **2026-10-20** You can voice a conversation with up to three people in one go and adjust the pauses between their lines.
- **2026-10-19** When you make a video from pictures, the tool picks the opening picture automatically.
- **2026-10-18** You can make voices on your own computer and save favorite voices, recordings, and profile pictures for next time.
- **2026-10-17** You can find and listen to voices before adding them to dialogue or narration.
- **2026-10-14** Your computer saves up to 20 recent edits so you can step back if you make a mistake.
- **2026-10-13** The tool keeps writing video descriptions when you switch pages and shows the finished work when you return.
- **2026-10-12** When you set a total video length, planning sets aside time for later scenes.
- **2026-10-11** The tool reads your material, plans scenes, prepares pictures, and works out what to show and how long each part should be.
- **2026-10-10** The editing page puts the preview in the middle and lets you scroll the settings and clips separately.
- **2026-10-09** Undo and redo let you go back and forth after editing or asking the tool to rewrite your descriptions.
- **2026-10-08** Set the video's total length yourself or enter zero to let the tool work it out from the content.
- **2026-10-07** You can use original pictures, recordings, videos, and full articles in a plan and recover finished work after an error.
- **2026-10-06** A reset button lets you clear the current work and start over.
- **2026-10-05** Finished videos go straight to the editing page, where voices and subtitles follow clips as you trim or reorder them.
- **2026-10-04** Write your idea in one box to see each step's results and make pictures of characters, places, and props.
- **2026-10-03** The tool can split long documents into episodes and has a separate page for watching and saving finished videos.
- **2026-10-02** You can use speaking speed and pauses to work out how long each part needs and let the content set the full video length.
- **2026-10-01** Planning chooses writing guidance to suit your idea and uses colors to show what each clip is for.
- **2026-09-30** Click a picture, video, recording, or document to preview it, or type @ to pick a file while writing what you want.
- **2026-09-29** The bottom panel has separate pages for your request, your clips, and their descriptions, and finished descriptions appear as they're written.
- **2026-09-28** You can turn off subtitles in the clip list and save that choice with the plan.
- **2026-09-27** Fixed jobs getting stuck between steps and added a choice to redraw the picture while making it bigger.
- **2026-09-26** You can save time by making a smaller picture and enlarging it, and choose whether to crop your picture when making a video.
- **2026-09-25** You can touch up or enlarge a finished clip and save it before moving to the next step.
- **2026-09-24** Clips join more smoothly, and the sound runs for the same length as the video.
- **2026-09-23** When you use a plan to make a video, each clip shows the files you chose for that run.
- **2026-09-21** You can zoom in on the clip list with the mouse wheel and see the little preview update when you change a picture.
- **2026-09-20** You can zoom in up to 40 times while editing to line up small clips more easily.
- **2026-09-19** The tool remakes the clips affected by changes to files or earlier clips, and lets you retry just the ones you pick.
- **2026-09-18** Clips stay visible with their details, and you can choose which ones the tool should write descriptions for.
- **2026-09-17** You can trim, split, remove, or reorder clips and ask the tool to write a description for each one.
- **2026-09-16** Enter a topic and the tool plans what to show in each part of your video.
- **2026-09-15** Clips are saved first and joined at the end, with smoother joins and less strain on your computer.
- **2026-09-14** You can mix different kinds of clips, and the tool picks the right way to make each one.
- **2026-09-13** The picture size follows your setting, with a smaller size used by default.
- **2026-09-12** Fixed a problem opening sound files and another that set up new clips incorrectly.

**Still planned**

- Keep editing a clip's description after the clip is made.

> From September 16 to 18, the tool also added ways to save and open a whole plan as a file, followed by a button to clear old results on September 19.

</details>

> **One-line takeaway｜Review changes here; for current steps, go back to the matching chapter.**

---

## Acknowledgments

> **Appendix A / Lead｜This plugin stands on many open-source projects and community experience.** The lists below are grouped by host, model, voice, peer projects, references and dependencies.

<details>
<summary>Open-source projects and contributors</summary>

This plugin stands on the shoulders of many excellent open-source projects, models and tools. Listed below (by category) are the things we referenced, borrowed from or depend on, directly or indirectly. Thanks to every author and community — if anything was missed, please open an Issue.

### Host & official nodes

- **[Comfy-Org / ComfyUI](https://github.com/Comfy-Org/ComfyUI)** — the host environment. The whole "plan → split → run each segment → stitch back" is built on ComfyUI's V3 node interface, its executor and its caching machinery; the bottom panel, settings page and theme colours all reuse the ComfyUI front end.
- **ComfyUI's official MiniMax H3 nodes** — all actual video/audio generation, sampling and decoding  are done by the official nodes. This plugin deliberately never changes or forks the host; it only arranges things around them and crops/stitches their outputs.

### Generation & understanding models

- **[MiniMax H3](https://github.com/MiniMax-AI/MiniMax-H3) (MiniMax / 稀宇科技)** — the base model for multi-segment video/audio generation; the six task modes and the 24-frames-per-second time base all come from it.
- **[Qwen3-VL](https://github.com/QwenLM) (Alibaba Qwen team)** — the text encoder that lets the model understand your prompt.
- **[Kimi](https://github.com/moonshotai) (Moonshot AI) / [DeepSeek](https://github.com/deepseek-ai) and other compatible services** — provide the large-model reasoning behind "one-click planning" and per-segment prompt rewriting (over the network, zero VRAM). The preset model is Kimi-K3; custom cloud/local services ([Ollama](https://github.com/ollama/ollama), [LM Studio](https://lmstudio.ai)) are also supported.

### Local voice models

- **[Qwen3-TTS](https://github.com/QwenLM/Qwen3-TTS) (Alibaba Qwen team)** — provides preset voices, voice design and voice cloning for local speech generation.
- **[IndexTTS-2.5](https://github.com/index-tts/index-tts) (IndexTeam)** — provides reference-audio voice cloning and emotion control for local speech generation; the voice library includes reference recordings from its official demo.
- **[Breeze-TTS-2](https://github.com/breezeblue-ai/breeze-tts) (BreezeBlue)** — provides voice design, voice cloning and performance control for local speech generation.

### Peer & reference open-source projects

Much of the design drew on the open-source H3 ecosystem and similar AI-video tools (see the per-project analyses under `文档/参考/`). Special thanks to:

- **[ComfyUI-Artificial-Intelligence](https://github.com/a63976659/ComfyUI-Artificial-Intelligence)** — the knowledge-base loader, the unified list that keeps a key per model, connectivity testing and error-message extraction were referenced/ported from it.
- **[AIMixer 导演台](https://github.com/AIMixer/ComfyUI_MiniMaxH3_Director)**, **[H3-Motion-Context](https://github.com/NikoDemon80/ComfyUI-H3-Motion-Context)**, **[H3-Multishot](https://github.com/jlucasmcrell/ComfyUI-H3-Multishot)**, **[H3-Codex-Drama](https://github.com/chiphoton/MiniMax-H3-Codex-Drama)**, **[H3-SelfLift](https://github.com/facok/comfyui-SelfLift)** — community nodes on the same MiniMax H3 pipeline; passing the internal data straight between segments, the audio continuation window, the `context_length` values, and the "check a sample segment first / don't touch the host" ideas are mutually corroborated with them.
- **[MoneyPrinterTurbo](https://github.com/harry0703/MoneyPrinterTurbo) (harry0703)**, **[video-shotcraft](https://github.com/Vincentwei1021/video-shotcraft) / [video-talkcraft](https://github.com/Vincentwei1021/video-talkcraft) / [anything2explainer](https://github.com/Vincentwei1021/anything2explainer) (based on [Remotion](https://github.com/remotion-dev/remotion))** — the reference frame for one-stop short-video pipelines and the "skill / recipe card / multi-layer checking" approach.
- **AIVideo_node**, **[NodeCraft AI](https://github.com/a63976659/ComfyUI-NodeCraft-AI) knowledge base** — sources of experience for large-model retry handling, settings persistence and linked UI patterns.

### Skills & knowledge-base material

- Content-planning references are stored in `Skills/`; timeline rewriting methods are stored in `AI生成提示词/`. See [the Skills directory guide](Skills/目录说明.md) for its structure and usage rules.

### Libraries & tools we depend on

- **[PyTorch](https://github.com/pytorch/pytorch) (torch)** — array maths, stitching and silence padding, i.e. all the numeric work.
- **[FFmpeg](https://ffmpeg.org) / [imageio-ffmpeg](https://github.com/imageio/imageio-ffmpeg)** — audio conversion and reading frames from reference videos.
- **[soundfile](https://github.com/bastibe/python-soundfile)** — reading audio length and samples.
- **[NumPy](https://github.com/numpy/numpy) / [Pillow](https://github.com/python-pillow/Pillow) / [aiohttp](https://github.com/aio-libs/aiohttp)** — array, image and async network capability provided by the ComfyUI main environment.
- **[PyAV](https://github.com/PyAV-Org/PyAV) (av)** — optional video-reading backend.
- **[pytest](https://github.com/pytest-dev/pytest)** — all front and back-end tests.

### Others

- The short-video storyboarding approach (hook / opening / setup / progression / conflict / climax / twist / ending / call-to-action as the jobs each segment does) draws on patterns widely circulated in the creator community.
- Thanks to every community member who filed bugs, suggestions and code changes.

</details>

---

## Support the author

> **Appendix B / Lead｜Help good tools get seen.** A star, sharing your work, or a voluntary sponsor all help the project keep going.

If this plugin helps you:

- **Star this repo**
- **Share your work** — videos made with this plugin are welcome in the Issues.
- **Sponsor**

<img width="400" height="500" alt="Donation QR code 1" src="https://github.com/user-attachments/assets/47f64c6e-cc45-4395-9b9e-29f464d12df2" />

*Fig A.1｜Donation QR code 1.*

<img width="400" height="500" alt="Donation QR code 2" src="https://github.com/user-attachments/assets/7bfb521a-6f24-4104-89c0-016dcf74834b" />

*Fig A.2｜Donation QR code 2.*

### Thanks for your support

- Thank you to **石头** (QQ: 34720803) for your suggestions, help, and red packet donation.
- Thank you to **Sorry全场** (QQ: 473148949) for your red packet donation.

> **One-line takeaway｜If the plugin helped you, a star, a shared video or feedback all count.**

---

<p align="center">
  <sub>Made with ❤️ for the ComfyUI community</sub>
</p>
