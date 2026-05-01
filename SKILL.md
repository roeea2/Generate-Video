---
name: generate-video
description: Guided HiggsField video generation — collects genre, topic, references, model, and duration, then plans a full storyboard, generates a 3×3 keyframe grid for visual consistency, slices it into per-scene start frames, creates each 10-second video from its keyframe, and delivers the final combined video.
user-invocable: true
allowed-tools:
  - Read
  - Bash(curl *)
  - Bash(ls *)
  - Bash(file *)
  - Bash(ffmpeg *)
  - Bash(printf *)
  - Bash(convert *)
  - mcp__claude_ai_HiggsField__models_explore
  - mcp__claude_ai_HiggsField__generate_image
  - mcp__claude_ai_HiggsField__media_upload
  - mcp__claude_ai_HiggsField__media_confirm
  - mcp__claude_ai_HiggsField__generate_video
  - mcp__claude_ai_HiggsField__job_status
  - mcp__claude_ai_HiggsField__job_display
---

# /generate-video — Guided HiggsField Video Generator

You are a creative video director and prompt engineer. Guide the user step by step through creating a cinematic AI-generated video via the **HiggsField MCP**.

The core architecture uses a **keyframe grid**: before any video is generated, you produce a single 3×3 image where each cell is the visual starting frame of one 10-second scene. Slicing this grid into individual keyframes and using each as `start_image` for its scene is what guarantees character identity, color temperature, lighting, and environment stay consistent across the whole video.

Arguments passed: `$ARGUMENTS`

---

## Phase 1: Step-by-Step Input Collection

Ask **one question at a time**, in order. Wait for the user's answer before moving to the next.

---

### Step 1 — Genre

Ask:

> **Let's create your video!**
>
> First — what **genre** or mood would you like?
>
> Examples: `action` · `drama` · `comedy` · `horror` · `romance` · `sci-fi` · `fantasy` · `documentary` · `thriller` · `epic` · `noir` · `animation`

---

### Step 2 — Topic / Concept

Ask:

> Great choice! Now — what should the video be **about**? Describe the concept, scene, or story.
>
> Example: *"A lone astronaut discovers a glowing ancient artifact on Mars and reaches out to touch it"*

---

### Step 3 — Reference Image(s)

Ask:

> Do you have any **reference images** to ground the video (e.g. a photo of a character, object, or place)?
>
> If yes, give me the file path(s) — e.g. `~/Pictures/photo.jpg`
> If no, just type `none`.

---

### Step 4 — Model

Before asking, call `mcp__claude_ai_HiggsField__models_explore` with `action: "recommend"`, `type: "video"`, and `input: "image"` (or `"text"` if no reference images).

Then ask:

> Almost there! Which **model** would you like to use?
>
> My recommendation: **Seedance 2.0** (`seedance_2_0`) — best for identity preservation and reference-driven scenes.
>
> Other options: `kling3_0` · `veo3` · `grok_video`
>
> Press **Enter** or type `default` to use Seedance 2.0.

If the user presses Enter or says "default", use `seedance_2_0`.

---

### Step 5 — Duration

Ask:

> How long should the video be? (10s to 120s)
>
> Examples: `10s` · `30s` · `1 minute` · `1.5 minutes` · `2 minutes`

Calculate:
- `total_seconds` = the number of seconds requested
- `N_scenes` = ceil(total_seconds / 10), clamped to max 12
- The grid is always **3×3 = 9 cells**; scenes beyond 9 use a second grid pass if needed
- Empty grid cells (when N < 9) are noted as unused

---

## Phase 2: Storyboard Planning

Build the **Continuity Sheet** first — this defines the shared visual identity locked across every scene:

| Property | Value |
|---|---|
| **Character A** | Exact appearance: shape, colors, markings, glow, texture |
| **Character B** | Exact appearance: shape, colors, markings, glow, texture |
| **Color temperature** | Warm / cool / neutral + dominant hues |
| **Lighting** | Key light direction, fill, ambient glow, shadow style |
| **Environment** | Setting, floor, background, atmospheric effects, particle details |

Then plan all N scenes as a complete narrative arc. For each scene write:
- **Beat**: what happens in these 10 seconds (action, emotion, movement)
- **Camera**: angle and movement — **must rotate across scenes, no two consecutive scenes share the same angle**
- **Opens**: exact visual state at the START (carried from the prior scene's end — never a fresh setup)
- **Ends**: exact visual moment this scene closes on (becomes next scene's Opens)

Present for approval:

---

**Here's your [total_seconds]s storyboard — does this look good?**

**Continuity Sheet:**
- Character A: [description]
- Character B: [description]
- Color temp / lighting: [values]
- Environment: [description]

| # | Beat | Camera | Opens | Ends |
|---|------|--------|-------|------|
| 1 | ... | Wide establishing | Fresh start | ... |
| 2 | ... | Close-up tracking | [prior end] | ... |
| N | ... | Low-angle | [prior end] | Final |

Reply **approve** to generate keyframes — or tell me what to change.

---

Wait for approval. Revise if needed. Never proceed without approval.

---

## Phase 3: Generate Keyframe Grid

Once the storyboard is approved, generate a single **3×3 keyframe grid image** where each cell is the opening visual frame of its scene. Because all 9 cells are generated together in one image, they share the same style, color palette, character designs, and lighting — this is what guarantees consistency across the whole video.

### 3a. Upload reference images (skip if none)

For each reference image path:
1. Verify the file exists: `ls <path>` and `file <path>`
2. Call `mcp__claude_ai_HiggsField__media_upload` with filename and correct content_type (`.jpg`→`image/jpeg`, `.png`→`image/png`, `.webp`→`image/webp`)
3. Run the curl PUT command from the upload response
4. Call `mcp__claude_ai_HiggsField__media_confirm` with `type: "image"` and the `media_id`
5. Save each confirmed `media_id` as `ref_media_ids[]`

### 3b. Generate the keyframe grid image

Call `mcp__claude_ai_HiggsField__generate_image` with:
- `aspect_ratio`: "1:1" (square — produces a balanced 3×3 grid)
- `medias`: all reference images with `role: "image"`
- `prompt`: a detailed 3×3 storyboard grid description:

```
A clean 3×3 storyboard grid on a pure black background, 9 equal panels separated by thin white borders, each panel numbered 1-9 (top-left to bottom-right, left to right across rows).

[Continuity Sheet values: describe exact character appearances, color temperature, lighting, environment — these apply to every panel]

Panel 1 (top-left): [Scene 1 Opens description — exact characters, positions, action, environment]
Panel 2 (top-center): [Scene 2 Opens description]
Panel 3 (top-right): [Scene 3 Opens description]
Panel 4 (middle-left): [Scene 4 Opens description]
Panel 5 (middle-center): [Scene 5 Opens description]
Panel 6 (middle-right): [Scene 6 Opens description]
Panel 7 (bottom-left): [Scene 7 Opens or BLACK EMPTY PANEL]
Panel 8 (bottom-center): [Scene 8 Opens or BLACK EMPTY PANEL]
Panel 9 (bottom-right): [Scene 9 Opens or BLACK EMPTY PANEL]

All panels must use identical character designs, color palette, and lighting. Cinematic storyboard style.
```

Poll `mcp__claude_ai_HiggsField__job_status` until completed. Display with `mcp__claude_ai_HiggsField__job_display`.

Download: `curl -s -o /tmp/keyframe_grid.png "<rawUrl>"`

### 3c. Slice the grid into individual keyframes

Do NOT use `convert` (ImageMagick) — it may not be available and does not handle grid borders reliably.

Instead, use Python + PIL to detect the actual white border lines in the image and crop each panel precisely:

```python
python3 -c "
from PIL import Image
import numpy as np

img = Image.open('/tmp/keyframe_grid.png').convert('RGB')
arr = np.array(img)

# Find bright column and row separators (white border lines)
brightness_cols = arr.mean(axis=(0, 2))
brightness_rows = arr.mean(axis=(1, 2))

# Detect border runs: sequences of columns/rows above threshold
threshold = 200

def find_borders(brightness, size):
    in_border = False
    borders = []  # list of (start, end) for each border band
    start = 0
    for i in range(size):
        if brightness[i] > threshold:
            if not in_border:
                start = i
                in_border = True
        else:
            if in_border:
                borders.append((start, i - 1))
                in_border = False
    if in_border:
        borders.append((start, size - 1))
    return borders

col_borders = find_borders(brightness_cols, img.width)
row_borders = find_borders(brightness_rows, img.height)

# Convert borders into panel content regions
def to_regions(borders, size):
    regions = []
    prev_end = borders[0][1] + 1  # skip outer left/top border
    for b_start, b_end in borders[1:]:
        if b_start > prev_end:
            regions.append((prev_end, b_start - 1))
        prev_end = b_end + 1
    return regions

col_regions = to_regions(col_borders, img.width)
row_regions = to_regions(row_borders, img.height)

print('Col regions:', col_regions)
print('Row regions:', row_regions)

# Crop each panel left-to-right, top-to-bottom
idx = 0
for (y1, y2) in row_regions:
    for (x1, x2) in col_regions:
        panel = img.crop((x1, y1, x2 + 1, y2 + 1))
        panel.save(f'/tmp/keyframe_{idx}.png')
        print(f'keyframe_{idx}.png: {panel.size} from ({x1},{y1}) to ({x2},{y2})')
        idx += 1
"
```

This auto-detects the actual panel boundaries regardless of border thickness, producing `/tmp/keyframe_0.png` through `/tmp/keyframe_N.png` in row-major order (left-to-right, top-to-bottom).

After slicing, **read each keyframe image** to visually verify it contains the correct scene content. If any panel looks wrong (wrong scene, black empty panel used, border artifacts), re-run with adjusted threshold or manually specify crop coordinates.

Only use keyframes for scenes 0 through N_scenes-1 (skip empty black panels).

### 3d. Upload each keyframe

For each keyframe i (0 to N_scenes-1):
1. `mcp__claude_ai_HiggsField__media_upload` with `filename: "keyframe_i.png"`, `content_type: "image/png"`
2. Run the curl PUT to upload `/tmp/keyframe_i.png`
3. `mcp__claude_ai_HiggsField__media_confirm` with `type: "image"`
4. Save as `keyframe_media_ids[i]`

### 3e. Show keyframes for approval

Tell the user:

---

**Here are your [N] scene keyframes — do these opening frames look right?**

*(The grid above shows the starting frame for each scene. Each cell will become the first frame of its 10-second video.)*

Reply **approve** to start generating videos — or describe what to fix and I'll regenerate the grid.

---

Wait for approval. If the user wants changes, go back to 3b and regenerate the grid.

---

## Phase 4: Generate Videos Scene by Scene

Once keyframes are approved, generate each 10-second scene in order.

For each scene i (1 to N_scenes):

**Tell the user:** "Generating scene [i] of [N_scenes]…"

Call `mcp__claude_ai_HiggsField__generate_video` with:
- `model`: chosen model
- `prompt`: the scene's cinematic prompt — constructed as follows:
  - **Always opens mid-action or mid-state** — never re-establishes the setup from zero. Use language like "Continuing mid-battle…", "The fight rages on as…", "Picking up immediately from…"
  - Re-states the exact `Opens` visual state from the storyboard (what the keyframe shows)
  - Describes the scene's beat and camera movement in full
  - Ends with a description matching the scene's `Ends` state
  - **Fully re-describes both characters and the environment** with the same specifics as the Continuity Sheet — the model has no memory of prior scenes
  - 4–8 sentences, rich with sensory and visual detail
- `aspect_ratio`: "16:9"
- `duration`: 10
- `resolution`: "1080p"
- `genre`: genre value (seedance_2_0 only — omit for other models)
- `medias`:
  - `{ "value": keyframe_media_ids[i-1], "role": "start_image" }` — locks the exact opening frame
  - `{ "value": ref_id, "role": "image" }` for each original reference image — reinforces character identity throughout

Poll `mcp__claude_ai_HiggsField__job_status` (sync: true) until `"completed"` or `"failed"`.

Call `mcp__claude_ai_HiggsField__job_display` with the job id.

Download: `curl -s -o /tmp/scene_NNN.mp4 "<rawUrl>"` (zero-padded, e.g. `scene_001.mp4`)

**Tell the user:** "Scene [i]/[N_scenes] done."

If a scene fails, explain the error and retry with an adjusted prompt before continuing.

---

## Phase 5: Deliver Final Video

Once all scenes are downloaded:

1. Write the concat list:
```bash
printf "file '/tmp/scene_001.mp4'\nfile '/tmp/scene_002.mp4'\n..." > /tmp/concat_final.txt
```

2. Concatenate:
```bash
ffmpeg -f concat -safe 0 -i /tmp/concat_final.txt -c copy /tmp/final_video.mp4 -y
```

3. Copy to project directory:
```bash
cp /tmp/final_video.mp4 ./final_[genre]_[total_seconds]s.mp4
```

4. Present the result:

---

**Your [total_seconds]-second video is complete!**

**Local file:** [final_genre_Xs.mp4](./final_genre_Xs.mp4)

**Scenes:** [N] × 10s · [model] · 1080p 16:9 · Genre: [genre]

---

---

## Rules

- Ask **one question at a time** — never bundle questions
- Never generate keyframes without storyboard approval
- Never generate videos without keyframe approval
- Always use HiggsField MCP tools — never simulate or skip them
- Seedance 2.0 is the default model — only the `genre` param applies to it
- **The keyframe grid is mandatory** — it is the single source of truth for visual consistency. Never skip it.
- **Always pass the scene's keyframe as `start_image`** — this locks the opening frame
- **Always pass ALL original reference images as `role: "image"`** to every scene — they reinforce character identity throughout the clip
- **Every scene prompt must open mid-action** — never "faces off", "squares up", "stands before". Use "continuing", "mid-battle", "picking up from"
- **Every scene prompt must fully re-describe characters and environment** — the video model has no memory between generations
- **Rotate camera angles across scenes** — no two consecutive scenes use the same angle
- **Continuity Sheet values are frozen** — character descriptions, color temperature, lighting, and environment must be word-for-word identical across all scene prompts
- If IP detection triggers on a `start_image`, describe the character more abstractly (shape/color, not brand name) and retry
- The grid always has 9 cells; leave unused cells black — do not shrink the grid
