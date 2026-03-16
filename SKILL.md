---
name: brief2campaign
description: >
  Run the Brief2Campaign AI marketing pipeline that transforms marketing briefs into
  production-ready ad campaigns using Bria APIs. Use this skill whenever the user
  wants to run a marketing campaign pipeline, generate campaign images from a brief, create
  ad creatives from a marketing brief, or anything involving Brief2Campaign, Bria image
  generation for marketing, or the brief-to-campaign workflow. Also trigger when the user
  mentions "campaign pipeline", "marketing pipeline", "generate ads from brief",
  "brief to campaign", "FIBO generate", "product placement pipeline", or wants to turn a
  marketing brief (text, PDF, or DOCX) into campaign visuals and ad templates.
allowed-tools: Read, Write, Bash, Edit, Glob, Grep
---

# Brief2Campaign Pipeline

An 8-step AI marketing pipeline that turns marketing briefs into production-ready campaign visuals
and ad creatives using Bria APIs for image generation, editing, product placement, and ad creation.

**You are the UI.** Display images inline using the Read tool on local `.png` files. Show progress
and review scores directly in the conversation. Handle image selection by presenting candidates and
asking the user.

## Setup

### 1. Clone or Update the Repo

The pipeline code lives on GitHub. Clone it if not present, or pull updates:

```bash
REPO_DIR="$HOME/.brief2campaign"
if [ -d "$REPO_DIR/.git" ]; then
  cd "$REPO_DIR" && git pull
else
  git clone https://github.com/SagiBria/Brief2Campaign.git "$REPO_DIR"
fi
```

All subsequent commands should run from `$HOME/.brief2campaign`.

### 2. Check API Keys

The pipeline requires these environment variables. Check if they're set:

```bash
echo "BRIA=$BRIA_API_KEY" && echo "ANTHROPIC=$ANTHROPIC_API_KEY" && echo "IMGBB=$IMGBB_API_KEY"
```

If any key is missing, ask the user and help them set it:
- **BRIA_API_KEY** — Get from [platform.bria.ai](https://platform.bria.ai/console/account/api-keys)
- **ANTHROPIC_API_KEY** — Get from [console.anthropic.com](https://console.anthropic.com/)
- **IMGBB_API_KEY** — Get from [api.imgbb.com](https://api.imgbb.com/) (only needed for ad generation)

Save keys to the `.env` file in `$HOME/.brief2campaign/.env`.

### 3. Install Dependencies

```bash
cd "$HOME/.brief2campaign"
python3 -m venv .venv 2>/dev/null
source .venv/bin/activate
pip install -r requirements.txt
```

## How the Pipeline Works

```
Brief (text/PDF/DOCX) → [1] Parse → [2] Generate → [3] Review → [4] Edit → [5] Product Placement → [6] Select → [7] Ads → [8] Review
```

| Step | What happens | Output |
|------|-------------|--------|
| 1. Parse Brief | Extracts campaign spec (scenes, colors, tone, copy) | Structured spec |
| 2. Generate | Enriches prompts → FIBO generates 4MP images | `output/1_generated/` |
| 3. Review | AI scores each image on 6 dimensions (pass ≥ 8) | Scores + feedback |
| 4. Edit | Re-edits failed images (up to 3 retries) | `output/2_final/` |
| 5. Products | Places products in scenes (if product images provided) | `output/3_products/` |
| 6. Select | **You show images and ask user to pick favorites** | User selection |
| 7. Ads | Generates branded ad creatives from templates | `output/4_ads/` |
| 8. Review | Checks ad heading placement | Verified ads |

## Running the Pipeline

The pipeline has 3 commands: `generate`, `finalize`, and `run`.

### Phase 1: Generate Images (Steps 1-5)

Run the `generate` command. This creates images and saves a `candidates.json` for selection:

```bash
cd "$HOME/.brief2campaign" && source .venv/bin/activate
python bria_marketing_agent.py generate \
  --brief /path/to/brief.txt \
  --output output
```

**With inline brief text:**
```bash
python bria_marketing_agent.py generate \
  --brief-text "Campaign: Summer Coffee\nScenes:\n1. Latte on cafe table\n2. Interior shot" \
  --output output
```

**With product images (enables product placement):**
```bash
python bria_marketing_agent.py generate \
  --brief brief.txt \
  --products ~/bottle.png ~/can.png \
  --output output
```

**With PDF/DOCX brief:**
```bash
python bria_marketing_agent.py generate --brief campaign_brief.pdf --output output
```

**Timeout:** This step takes 2-5 minutes depending on the number of scenes. Use a 600s timeout.

**After it completes:**
1. Read `output/candidates.json` to get the candidate list
2. Read each candidate's local image file with the Read tool to display it inline
3. Present candidates to the user with their IDs, scores, and scene descriptions
4. Ask the user which images they want to keep

### Phase 2: Image Selection (Step 6)

After showing the candidates, collect the user's selection. Map their choices to candidate IDs
from `candidates.json` (e.g., `edited_0`, `edited_1`, `product_0`).

**candidates.json structure:**
```json
{
  "campaign_name": "Your Daily Pause",
  "candidates": [
    {
      "id": "edited_0",
      "category": "edited",
      "local": "output/2_final/final_s1_1x1.png",
      "scene": "Close-up latte on wooden table...",
      "score": 9,
      "ratio": "1:1"
    }
  ],
  "spec": { "brand_id": "162", "template_ids": ["1274"], ... }
}
```

If the user doesn't want ad creatives, the selected images in `output/2_final/` are the final
deliverables. No need to run `finalize`.

### Phase 3: Generate Ads (Steps 7-8) — Optional

If the user wants branded ad creatives (requires `brand_id` + `template_ids`):

```bash
python bria_marketing_agent.py finalize \
  --candidates output/candidates.json \
  --selected edited_0 edited_2 \
  --brand-id 162 \
  --templates 1274
```

Brand ID and template IDs can come from the brief (extracted during parsing) or from the user.
After completion, read and display the ad images from `output/4_ads/`.

### Full Pipeline (No Pause)

For a headless run that auto-selects all passing images:

```bash
python bria_marketing_agent.py run \
  --brief brief.txt \
  --brand-id 162 \
  --templates 1274 \
  --output output
```

Use this when the user says "just run everything" or doesn't need to pick specific images.

## Pipeline Input Options

| Flag | Description |
|------|-------------|
| `--brief`, `-b` | Brief file path (`.txt`, `.pdf`, `.docx`) |
| `--brief-text` | Brief as inline text string |
| `--sample`, `-s` | Use built-in sample brief |
| `--products`, `-p` | Product image paths/URLs (enables Step 5) |
| `--references`, `-r` | Reference images for style guidance |
| `--brand-id` | Bria Brand ID (enables Step 7) |
| `--templates`, `-t` | Bria Template IDs (enables Step 7) |
| `--output`, `-o` | Output directory (default: `output`) |
| `--tailored-model` | Fine-tuned Bria model ID |
| `--tailored-influence` | Tailored model weight (0.0-1.0) |

## Output Structure

```
output/
├── 1_generated/              # Raw generated images
├── 2_final/                  # Images after AI review + editing
├── 3_products/               # Product placement results
├── 4_ads/                    # Ad creatives
├── candidates.json           # Candidate list for selection
└── pipeline_results.json     # Complete results metadata
```

## Displaying Results

**Always use the Read tool** to display images inline to the user. Example flow:

1. After `generate` completes, read `output/candidates.json`
2. For each candidate, read the local `.png` file to show it
3. Present with context: scene description, review score, aspect ratio
4. After `finalize`, read ad images from `output/4_ads/`

## Reference

For detailed pipeline step documentation, see `references/pipeline-steps.md`.
For example marketing briefs, see `references/brief-examples.md`.

## Troubleshooting

- **"BRIA_API_KEY missing"** — Check `.env` file has the key set
- **Image review fails repeatedly** — After 3 retries, the image is flagged. Brief may be too specific
- **Ad generation timeout** — Bria ads can take up to 2 minutes. The pipeline polls automatically
- **"No matching candidates"** — Check that `--selected` IDs match IDs in `candidates.json`
- **PDF brief not parsing** — Ensure `anthropic` package is installed (PDF uses Anthropic's native PDF support)
