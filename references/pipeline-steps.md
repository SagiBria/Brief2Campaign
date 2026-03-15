# Brief2Campaign Pipeline Steps

The pipeline runs 8 sequential steps. Steps 5 (Product Placement) and 7 (Ad Templates) are
conditional — they only run if the user provides the relevant inputs.

## Step 1: Parse Brief

**Input:** Marketing brief (text, PDF, DOCX, or TXT)
**Process:** Claude extracts structured requirements into a BriefSpec JSON
**Output:** Campaign name, brand colors/fonts, tone, audience, aspect ratios, scene descriptions, ad copy, product info, brand/template IDs

The brief can be free-form text. Claude is flexible about format and will extract what it can.

## Step 2: FIBO Generate

**Input:** Enriched prompts derived from brief
**Process:**
1. Claude creates an enriched text prompt from the scene description + brand guidelines
2. Bria VLM Bridge converts the prompt to VGL (Visual Generation Language) JSON with 100+ visual attributes
3. FIBO generates 4MP images from the structured representation

**Output:** Generated images + VGL structured prompts
**Optional:** Use a tailored (fine-tuned) Bria model with configurable influence weight (0.0-1.0)

## Step 3: AI Review & QA

**Input:** Generated images + brief spec
**Process:** Claude vision reviews each image against the brief on 6 weighted dimensions:
- Scene accuracy (25%)
- Object physics (20%)
- Composition (15%)
- Brand alignment (20%)
- Technical quality (10%)
- Marketing impact (10%)

**Output:** Score (1-10), pass/fail, edit instructions
**Pass threshold:** Overall score >= 8 AND every individual dimension >= 5

## Step 4: FIBO Edit / Refine

**Input:** Failed images + edit instructions from review
**Process:** Targeted edits via Bria APIs:
- `/image/edit` — Instruction-based editing
- `/image/edit/gen_fill` — Masked generation fill
- `/image/edit/relight` — Lighting adjustments

**Max retries:** 3 per image before flagging for human review
**Output:** Edited images matching brand guidelines

## Step 5: Product Placement (Conditional)

**Skipped if:** No product images provided

**Input:** Product images + scene descriptions/reference images
**Process:**
1. Remove background from product image
2. Upload to ImgBB for a public URL
3. Place product in scene via `lifestyle_shot_by_text` or `lifestyle_shot_by_image`

**Output:** Product composited in lifestyle scene

## Step 6: Image Selection (Human-in-the-Loop)

**The `generate` command exits here**, saving candidates to `candidates.json`.

**Input:** All candidate images (edited + product-placed)
**Process:** The agent displays candidate images inline and asks the user to select which images
to keep for ad generation.
**Options:**
- Select specific images to proceed → run `finalize` command
- Regenerate → run `generate` command again
- Use all images → run `run` command for auto-selection

**Output:** Selected candidate IDs for the `finalize` command

In `run` mode (full pipeline), all passing images are automatically selected.

## Step 7: Ad Templates (Conditional)

**Skipped if:** No template IDs provided

**Input:** Template ID, brand ID, hero image, ad copy text elements
**Process:**
1. Fetch template to discover text slots
2. Map `ad_copy` roles (heading, sub_heading, cta) to template slots
3. Generate ads with `expand_image` operation
4. Poll until ad images are ready (can take up to ~2 minutes)

**Output:** Final ad creatives in multiple formats

## Step 8: Ad Heading Review

**Input:** Ad images + expected heading texts
**Process:** Claude vision QA checks:
- All text headings fully visible inside image borders
- Properly positioned without overlapping
- Text is readable

**Max retries:** 3 fix attempts per ad using FIBO Edit
**Output:** Verified ads with correct heading placement
