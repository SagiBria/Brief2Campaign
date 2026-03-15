# Brief2Campaign — Turn Marketing Briefs into Production-Ready Ad Campaigns

An 8-step AI marketing automation pipeline that transforms marketing briefs into campaign-ready visuals and ad creatives using [Bria.ai](https://bria.ai) for commercially-safe image generation.

**Works with:** Claude Code, Cursor, Cline, Codex, and [37+ other agents](https://skills.sh)

## Quick Start

### 1. Install

```shell
npx skills add <your-org>/brief2campaign
```

### 2. Get your API keys

| Key | Where to get it |
|-----|-----------------|
| **Bria API Key** | [platform.bria.ai](https://platform.bria.ai/console/account/api-keys) |
| **Anthropic API Key** | [console.anthropic.com](https://console.anthropic.com/) |
| **ImgBB API Key** | [api.imgbb.com](https://api.imgbb.com/) (only needed for ad generation) |

### 3. Set the keys

```shell
export BRIA_API_KEY="your-key-here"
export ANTHROPIC_API_KEY="your-key-here"
export IMGBB_API_KEY="your-key-here"
```

Or add them to a `.env` file in the project root.

### 4. Use it

Ask your agent to generate a campaign from a brief. The skill handles the rest — parsing, image generation, AI review, product placement, and ad creation.

> "Run the Brief2Campaign pipeline with this brief: Summer coffee promotion targeting young professionals..."

## How It Works

```
Marketing Brief (text, PDF, or DOCX)
    ↓
[Step 1] Parse Brief → structured campaign spec
    ↓
[Step 2] Generate Images → enriched prompts → FIBO 4MP images
    ↓
[Step 3] AI Review → 6-dimension scoring against brief
    ↓
[Step 4] Edit & Refine → targeted fixes on failed images
    ↓
[Step 5] Product Placement → composite products into scenes — optional
    ↓
[Step 6] Image Selection → agent shows images, user picks favorites
    ↓
[Step 7] Ad Templates → generate branded ad creatives — optional
    ↓
[Step 8] Ad Heading Review → verify text placement
    ↓
Production-Ready Campaign Assets
```

## Pipeline Commands

### Generate images (Steps 1-5)

```bash
python bria_marketing_agent.py generate --brief brief.txt --output output
```

Creates images and saves `output/candidates.json` for selection.

### Finalize with ads (Steps 7-8)

```bash
python bria_marketing_agent.py finalize \
  --candidates output/candidates.json \
  --selected edited_0 edited_2 \
  --brand-id 162 --templates 1274
```

### Full pipeline (auto-select all)

```bash
python bria_marketing_agent.py run --brief brief.txt --brand-id 162 --templates 1274
```

## What You Can Control

| Capability | How | Example |
|------------|-----|---------|
| **Brief input format** | Text, PDF, DOCX, or TXT | Paste text or upload a campaign brief document |
| **Scene descriptions** | Natural language in brief | "Close-up latte on wooden table, morning light" |
| **Brand identity** | Colors, fonts, tone in brief | Brand colors extracted and enforced across all visuals |
| **Aspect ratios** | Multiple per campaign | Generate every scene in 1:1, 16:9, and 9:16 |
| **Product placement** | Upload product images | Remove background and place products in lifestyle scenes |
| **Ad templates** | Bria template + brand IDs | Auto-map copy to template text slots |
| **Tailored models** | Fine-tuned Bria models | Use a custom model with configurable influence weight |
| **Quality threshold** | 6-dimension AI scoring | Images must score ≥ 8 overall, no dimension below 5 |
| **Image selection** | Human-in-the-loop | Review, approve, or request regeneration |

## Output Structure

```
output/
├── 1_generated/              # Raw FIBO-generated images
├── 2_final/                  # Images after AI review + editing
├── 3_products/               # Product placement results
├── 4_ads/                    # Final ad creatives
├── candidates.json           # Candidate list for selection
└── pipeline_results.json     # Complete metadata and results
```

## Requirements

- **Python 3.12+** with dependencies in `requirements.txt`
- **Bria API key** — Free at [platform.bria.ai](https://platform.bria.ai/console/account/api-keys)
- **Anthropic API key** — From [console.anthropic.com](https://console.anthropic.com/)
- **ImgBB API key** — From [api.imgbb.com](https://api.imgbb.com/) (needed for ad generation only)

## Documentation

- [SKILL.md](SKILL.md) — Skill definition and usage guide for AI agents
- [Pipeline Steps](references/pipeline-steps.md) — Detailed breakdown of all 8 pipeline steps
- [Brief Examples](references/brief-examples.md) — Example marketing briefs

## License

MIT
