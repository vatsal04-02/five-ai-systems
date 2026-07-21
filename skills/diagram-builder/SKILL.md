---
name: diagram-builder
description: Turn any concept, process, comparison, or system into a polished dark-mode diagram. Use when the user says "diagram this", "visualize this workflow", "map out my funnel", or needs a visual for a video, carousel, or doc. Outputs an editable .excalidraw file the user opens free at excalidraw.com — no API keys needed.
allowed-tools: Read, Write, Bash, Glob, Grep
---

# Diagram Builder

Describe a concept and get a designed, editable diagram: dark background, colored accent cards, emoji icons, hand-drawn warmth. Claude writes the diagram as Excalidraw JSON directly — no image API, no accounts, no cost. The user opens the file at excalidraw.com and can drag, resize, and re-word every element before exporting a PNG or SVG.

## When to Use

- Process flows ("diagram my content funnel", "visualize the onboarding steps")
- System maps (how tools, people, or services connect — hub-and-spoke works well here)
- Comparisons (before/after, old way vs. new way, side-by-side)
- Frameworks and mental models (layers, rings, pyramids, ladders)
- Anything headed for a video, carousel slide, or doc that needs a visual

## Prerequisites

**None.** The core path is Claude writing a `.excalidraw` JSON file. Excalidraw.com is free and needs no account. An optional AI-image rendering path exists (see bottom) — it needs a Gemini API key, but skip it unless the user asks for a finished PNG with zero editing.

## How It Works

1. **Clarify the content.** Know the 3-7 major pieces and how they relate before drawing. If the request is vague ("diagram Docker"), ask one or two questions: which aspect, and how much detail? Keep it to 3-7 nodes — more than that, split into two diagrams.
2. **Pick a layout.**
   - Flow / process → left-to-right cards with labeled arrows
   - System / ecosystem → hub-and-spoke: one center card, satellites around it, arrows radiating out
   - Comparison → two columns with a "VS" divider
   - Framework → stacked layers or concentric rings
3. **Plan coordinates on paper first.** Assign each section a fixed width and x position, then work top-to-bottom: `next_y = current_y + height + gap`. Gaps: 40-60px between major sections, 15-25px between siblings. Never freehand coordinates — that's where overlaps come from.
4. **Write the JSON** to `outputs/diagrams/[concept-slug].excalidraw` following the style rules below.
5. **Deliver.** Describe what the diagram shows, then tell the user how to open it (see Output).
6. **Iterate.** Moving an element means updating its x/y plus everything positioned relative to it. Changing text means updating both `text` and `originalText`. For diagrams over ~20 elements, build section by section to avoid coordinate drift.

## Style Rules (what makes these look good)

### Dark canvas, always

```json
"appState": {"gridSize": null, "viewBackgroundColor": "#141a28"}
```

- Background: `#141a28` (deep navy)
- Card fills: `#1a1f2e` | nested/inner cards: `#0d1320`
- Card strokes: the section's accent color (never gray on dark)
- Headings: `#f2f5fb` | descriptions: `#7c92b0` | closing/impact lines: `#38d9a9`
- If the user explicitly wants light mode: white background, `#1e1e1e` strokes and text, same accent colors.

### Accent colors — one per section

Each major section gets its own accent. Rotate through these; never one color for everything:

| Accent | Hex | Feels like |
|---|---|---|
| Punch red | `#C4533A` | Energy, key concepts, CTAs |
| Coral | `#ff8787` | Highlights, warnings |
| Blue | `#4dabf7` | Data, tech, inputs |
| Teal | `#38d9a9` | Outputs, results, growth |
| Purple | `#da77f2` | Ideas, content, creative |
| Amber | `#ffa94d` | Process, operations |

Color tells the story: everything in the "input" zone shares one accent, everything in the "output" zone another. The viewer should read the structure before reading a word.

### The shadow card (the signature move)

Every major card is TWO overlapping rounded rectangles — a colored "shadow" rect offset +10px in x and y BEHIND a dark card. The accent peeks out on the bottom-right edge and the whole diagram gains depth:

```
BACK:  x+10, y+10, backgroundColor = accent color, strokeColor = "transparent",
       opacity 50-70, roundness {"type": 3}
FRONT: x, y, backgroundColor = "#1a1f2e", strokeColor = accent color,
       roundness {"type": 3}
```

The shadow rect goes earlier in the `elements` array (lower z-order). No flat boxes sitting alone on the background.

### Emoji icons

One emoji per card as a text element, fontSize 22-40, centered near the top of the card. Pick one that instantly reads as the concept (🎬 video, 📱 mobile, 🧠 brain, ⚙️ system). One per card — more gets noisy.

### Typography

All text uses Virgil, the hand-drawn font (`fontFamily: 1`). Monospace Cascadia (`fontFamily: 3`) only for code or data values. Never Helvetica.

| Role | fontSize |
|---|---|
| Diagram title | 28-36 |
| Section header | 20-24 |
| Card label | 16-18 |
| Annotation / small note | 12-14 |

Labels stay short — 2-5 words. Longer explanations become smaller muted-color annotations. Set text width equal to the parent card width with `textAlign: "center"` and it self-centers.

**Contrast rule:** text on dark fills is `#f2f5fb` or `#7c92b0`, never the accent color of the card it sits on (accent-on-accent is unreadable). Accent color is for strokes, section titles, and arrows.

### Hand-drawn feel

`roughness: 1` on every shape. `roundness: {"type": 3}` on card rectangles. Arrows: `strokeWidth: 2`, accent or `#7c92b0` stroke, labeled with a small text element ~25px above the midpoint when the connection isn't obvious.

### Hard rules

1. No numbers, digits, or step ordinals in any text — spell out or cut.
2. Every word correctly spelled; simple common English.
3. 3-7 main cards per diagram.
4. Shadow + accent on every major card.
5. One accent color per section, one emoji per card.

## Element Schema

Every element needs ALL base fields — Excalidraw rejects incomplete elements:

```json
{
  "id": "unique-string", "type": "rectangle", "x": 0, "y": 0,
  "width": 200, "height": 140, "angle": 0,
  "strokeColor": "#4dabf7", "backgroundColor": "#1a1f2e",
  "fillStyle": "solid", "strokeWidth": 2, "strokeStyle": "solid",
  "roughness": 1, "opacity": 100, "groupIds": [], "frameId": null,
  "roundness": {"type": 3}, "boundElements": [], "updated": 1,
  "link": null, "locked": false
}
```

Text elements add: `"text"`, `"fontSize"`, `"fontFamily": 1`, `"textAlign"`, `"verticalAlign"`, `"containerId": null`, `"originalText"` (same as text), `"lineHeight": 1.25`.

Arrows add: `"points": [[0,0],[dx,dy]]`, `"lastCommittedPoint": null`, `"startBinding": null`, `"endBinding": null`, `"startArrowhead": null`, `"endArrowhead": "arrow"`.

File wrapper:

```json
{
  "type": "excalidraw", "version": 2, "source": "https://excalidraw.com",
  "elements": [ ... ],
  "appState": {"gridSize": null, "viewBackgroundColor": "#141a28"},
  "files": {}
}
```

Build order in the elements array (back to front): shadow rects → cards → inner content → emoji + text → arrows.

## Output

Save to `outputs/diagrams/[concept-slug].excalidraw` (create the folder if missing). Then tell the user:

> **How to view and edit your diagram:**
> - Go to excalidraw.com (free, no account needed)
> - Menu (top-left) → "Open" → select the `.excalidraw` file
> - Or copy the file's JSON and paste it on the canvas with Cmd/Ctrl+V
> - Everything is editable — drag to move, double-click to edit text
> - Export as PNG or SVG: Menu → "Export image" (turn on "background" to keep the dark canvas)

## Troubleshooting

- **File won't open on excalidraw.com** → almost always invalid JSON or a missing required field. Validate with `python3 -m json.tool file.excalidraw`, and check every element has the full base field set.
- **Text overflows its card** → widen the card or shorten the label; text width should equal card width.
- **Elements overlapping** → coordinates weren't planned. Redo the coordinate math for that section; children are absolute, not relative (`child_x = parent_x + padding`).
- **Diagram looks flat** → shadow rects missing or in the wrong z-order (they must come before their cards in the array).
- **Exported PNG has a white background** → enable the "background" toggle in Excalidraw's export dialog.

## Optional: AI-Rendered Images (needs an API key)

If the user wants a finished raster image with zero manual editing, Gemini's image model can render the same style directly. Skip this unless asked — the Excalidraw path is free and editable.

**Prerequisites:** `GEMINI_API_KEY` in a `.env` file (get one at aistudio.google.com — image generation costs cents per image). Python packages: `google-genai`, `python-dotenv`.

```python
from dotenv import load_dotenv; load_dotenv()
import os
from google import genai
from google.genai import types

client = genai.Client(api_key=os.getenv("GEMINI_API_KEY"))
prompt = (
    "Create a clean, modern diagram on a dark charcoal background. Rounded "
    "rectangles with soft muted fills, light cream typed sans-serif text, thin "
    "light-gray arrows, simple line-art icons in each box. Subtle thin hand-drawn "
    "annotation accents (sketchy circle around one key box, small handwritten "
    "note). No gradients, no photorealism, no numbers or digits anywhere, every "
    "word correctly spelled.\n\nDiagram to create: " + "YOUR TOPIC HERE"
)
resp = client.models.generate_content(
    model="gemini-3-pro-image-preview", contents=prompt,
    config=types.GenerateContentConfig(response_modalities=["IMAGE", "TEXT"]),
)
for part in resp.candidates[0].content.parts:
    if part.inline_data:
        with open("outputs/diagrams/diagram.png", "wb") as f:
            f.write(part.inline_data.data)
```

Generate 2-3 variations and let the user pick — image models vary run to run. Text accuracy is the weak spot; regenerate if any word is garbled.
