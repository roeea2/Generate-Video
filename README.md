# Generate Video — Claude Code Skill

A guided, step-by-step Claude Code skill that generates cinematic AI videos using the **HiggsField MCP**. The skill walks you through four questions, writes a cinematic script for your approval, and then generates the video — returning a direct download link.

---

## Features

- Step-by-step guided input (genre, topic, reference image, model)
- Cinematic script generation with director-level prompting
- Script approval before any generation is triggered
- Reference image upload and identity preservation via HiggsField MCP
- Automatic polling until the video is ready
- Direct MP4 download link on completion
- Supports multiple models: Seedance 2.0, Kling 3.0, Veo 3, Grok Video

---

## Requirements

- [Claude Code](https://claude.ai/code) CLI or desktop app
- HiggsField MCP configured in your Claude Code settings
- An active HiggsField account with credits

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

Type `/generate-video` in Claude Code. The skill will ask:

1. **Genre** — e.g. `action`, `drama`, `sci-fi`, `horror`, `epic`, `noir`
2. **Topic / Concept** — describe the scene or story
3. **Reference image(s)** — file path(s) or `none`
4. **Model** — defaults to Seedance 2.0 (best for identity preservation)

After collecting your answers, the skill presents a cinematic script for your approval. Once approved, it generates the video via HiggsField and returns a download link.

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
