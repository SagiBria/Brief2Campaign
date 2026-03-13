# Brief2Campaign — Turn Marketing Briefs into Production-Ready Ad Campaigns

An 8-step AI marketing automation pipeline that transforms marketing briefs into campaign-ready visuals and ad creatives using [Claude](https://anthropic.com) for intelligence and [Bria.ai](https://bria.ai) for commercially-safe image generation.

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
| **ImgBB API Key** | [api.imgbb.com](https://api.imgbb.com/) |

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

The pipeline runs 8 sequential steps, orchestrated by Claude and powered by Bria's image generation APIs:

```
Marketing Brief (text, PDF, or DOCX)
    ↓
[Step 1] Parse Brief → structured campaign spec (Claude)
    ↓
[Step 2] Generate Images → enriched prompts → VGL → FIBO 4MP images (Bria)
    ↓
[Step 3] AI Review → 6-dimension scoring against brief (Claude Vision)
    ↓
[Step 4] Edit & Refine → targeted fixes on failed images (Bria Edit)
    ↓
[Step 5] Product Placement → composite products into scenes (Bria) — optional
    ↓
[Step 6] Image Selection → human-in-the-loop review — ⏸️ pipeline pauses
    ↓
[Step 7] Ad Templates → generate branded ad creatives (Bria Ads) — optional
    ↓
[Step 8] Ad Heading Review → verify text placement (Claude Vision)
    ↓
Production-Ready Campaign Assets
```

## What You Can Control

| Capability | How | Example |
|------------|-----|---------|
| **Brief input format** | Text, PDF, DOCX, or TXT | Paste text or upload a campaign brief document |
| **Scene descriptions** | Natural language in brief | "Close-up latte on wooden table, morning light, croissant in background" |
| **Brand identity** | Colors, fonts, tone in brief | Brand colors extracted and enforced across all visuals |
| **Aspect ratios** | Multiple per campaign | Generate every scene in 1:1, 16:9, and 9:16 simultaneously |
| **Product placement** | Upload product images | Remove background and place products in lifestyle scenes |
| **Ad templates** | Bria template + brand IDs | Auto-map copy to template text slots and generate branded ads |
| **Tailored models** | Fine-tuned Bria models | Use a custom model with configurable influence weight (0.0–1.0) |
| **Quality threshold** | 6-dimension AI scoring | Images must score ≥ 8 overall with no dimension below 5 |
| **Image selection** | Human-in-the-loop | Review, approve, or request regeneration before ad creation |

## AI Review Dimensions

Every generated image is scored by Claude Vision across 6 weighted dimensions:

| Dimension | Weight | What it checks |
|-----------|--------|----------------|
| Scene accuracy | 25% | Does the image match the requested scene? |
| Object physics | 20% | Are objects physically plausible? |
| Brand alignment | 20% | Does it match the brand's visual identity? |
| Composition | 15% | Well-composed for marketing use? |
| Technical quality | 10% | Clean render with no artifacts? |
| Marketing impact | 10% | Effective for the target audience? |

**Pass threshold:** Overall score ≥ 8 AND every individual dimension ≥ 5. Failed images are automatically re-edited up to 3 times.

## Example Use Cases

**Social media campaign?** Provide a brief with 3 scene descriptions and get Instagram-ready 1:1 images with AI-reviewed quality.

**Product launch?** Upload product photos + a brief, and get lifestyle shots with your product composited into on-brand scenes.

**Ad creatives?** Add a Bria brand ID and template IDs to generate ready-to-publish ads with your copy auto-mapped to template slots.

**Multi-format campaign?** Define multiple aspect ratios and the pipeline generates every scene in every ratio — one brief, many assets.

## Running the Pipeline

### Web UI

Start both servers and open the visual interface:

```bash
# Backend (FastAPI)
cd backend && pip install -r requirements.txt
uvicorn server:app --host 0.0.0.0 --port 8000

# Frontend (Next.js)
cd frontend && npm install && npm run dev
```

Open [http://localhost:3000](http://localhost:3000) — the UI provides brief input, file uploads, live progress visualization, image selection, and a results gallery.

### CLI

Run the pipeline directly from the command line:

```bash
# Interactive mode
python bria_marketing_agent.py

# From a brief file
python bria_marketing_agent.py --brief brief.txt

# With product images and reference images
python bria_marketing_agent.py -b brief.txt -p product.png -r ref1.jpg ref2.jpg

# With brand ID and ad templates
python bria_marketing_agent.py --brief brief.txt --brand-id 162 --templates 1274
```

### REST API

Start the backend and drive the pipeline programmatically:

```bash
# Start a pipeline run
curl -X POST http://localhost:8000/api/pipeline/start \
  -H "Content-Type: application/json" \
  -d '{"brief_text": "Your campaign brief...", "brand_id": "162", "template_ids": ["1274"]}'

# Monitor progress via SSE
curl -N http://localhost:8000/api/pipeline/status/{run_id}

# Submit image selection (Step 6)
curl -X POST http://localhost:8000/api/pipeline/{run_id}/select-images \
  -H "Content-Type: application/json" \
  -d '{"selected_image_ids": ["edited_0", "edited_2"]}'

# Get final results
curl http://localhost:8000/api/pipeline/results/{run_id}
```

## Output Structure

```
output/
├── 1_generated/              # Raw FIBO-generated images
├── 2_final/                  # Images after AI review + editing
├── 3_products/               # Product placement results
├── 4_ads/                    # Final ad creatives
└── pipeline_results.json     # Complete metadata and results
```

## Tech Stack

| Layer | Technology |
|-------|-----------|
| **AI / LLM** | Claude (Anthropic) — brief parsing, prompt enrichment, image review |
| **Image Generation** | Bria FIBO — text-to-image via VGL structured prompts |
| **Image Editing** | Bria Edit — instruction-based editing, gen fill, relighting |
| **Product Placement** | Bria Product API — background removal + lifestyle shots |
| **Ad Creation** | Bria Ads API — template-based ad generation |
| **Backend** | FastAPI + Uvicorn + SSE streaming |
| **Frontend** | Next.js 15 + React 19 + Zustand + Tailwind CSS |
| **Image Hosting** | ImgBB — public URL hosting for API interop |

## Requirements

- **Python 3.12+** with dependencies in `backend/requirements.txt`
- **Node.js 18+** for the frontend
- **Bria API key** — Free at [platform.bria.ai](https://platform.bria.ai/console/account/api-keys)
- **Anthropic API key** — From [console.anthropic.com](https://console.anthropic.com/)
- **ImgBB API key** — From [api.imgbb.com](https://api.imgbb.com/) (needed for product placement and ad generation)

## Documentation

- [SKILL.md](SKILL.md) — Skill definition and quick-start guide for AI agents
- [API Reference](references/api-reference.md) — All REST endpoints, request/response schemas, SSE events
- [Pipeline Steps](references/pipeline-steps.md) — Detailed breakdown of all 8 pipeline steps

## License

MIT
