# Generate Video — Claude Code Skill

A guided Claude Code skill that produces multi-scene cinematic AI videos using the **HiggsField MCP**. The skill collects your inputs, plans a full storyboard, generates a 3×3 keyframe grid for visual consistency, slices it into per-scene start frames, generates each 10-second clip from its keyframe, and delivers the final combined video.

---

## How It Works

The core architecture is the **keyframe grid**: before any video is generated, the skill produces a single 3×3 image where each cell is the exact opening frame of one 10-second scene. Since all 9 cells are generated together in one image, they share the same character designs, color palette, lighting, and environment. Each cell is then sliced out and used as `start_image` for its scene — locking the opening frame and eliminating character drift across the whole video.

---

## Features

- Step-by-step guided input (genre, topic, reference images, model, duration)
- Full storyboard planning with a Continuity Sheet (characters, color temperature, lighting, environment)
- 3×3 keyframe grid for visual consistency across all scenes
- Precise grid slicing via Python PIL border detection (works regardless of border thickness)
- Scene-by-scene video generation with `start_image` locking each opening frame
- Storyboard and keyframe approval gates before generation
- Automatic ffmpeg concat into a single final video
- Supports videos from 10s up to 2 minutes (up to 12 × 10s scenes)
- Supports multiple models: Seedance 2.0, Kling 3.0, Veo 3, Grok Video

---

## Requirements

- [Claude Code](https://claude.ai/code) CLI or desktop app
- HiggsField MCP configured in your Claude Code settings
- An active HiggsField account with credits
- Python 3 with `Pillow` and `numpy` (`pip install pillow numpy`)
- `ffmpeg` installed (`brew install ffmpeg` on macOS)

---

## Installation

Copy `SKILL.md` into your project's skills directory:

```bash
mkdir -p .claude/skills/generate-video
cp SKILL.md .claude/skills/generate-video/SKILL.md
```

Restart Claude Code — the skill will appear automatically.

---

## Usage

Type `/generate-video` in Claude Code. The skill asks **one question at a time**:

1. **Genre** — e.g. `action`, `drama`, `sci-fi`, `horror`, `epic`, `noir`
2. **Topic / Concept** — describe the scene or story
3. **Reference image(s)** — file path(s) or `none`
4. **Model** — defaults to Seedance 2.0
5. **Duration** — 10s to 2 minutes (e.g. `30s`, `1 minute`, `1.5 minutes`)

The skill then:
- Builds a storyboard with a Continuity Sheet and scene breakdown for your approval
- Generates a 3×3 keyframe grid for approval
- Generates each scene video using its keyframe as the opening frame
- Concatenates all scenes into the final video

---

## Supported Models

| Model | ID | Best For |
|---|---|---|
| Seedance 2.0 | `seedance_2_0` | Identity preservation, reference-driven scenes |
| Kling 3.0 | `kling3_0` | Multi-shot, cinematic, audio sync |
| Google Veo 3 | `veo3` | Broad cinematic range |
| Grok Video | `grok_video` | General purpose, versatile |

---

## License

Apache 2.0 — see [LICENSE](LICENSE).
