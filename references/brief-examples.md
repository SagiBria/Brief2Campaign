# Brief Examples

Example marketing briefs for the Brief2Campaign pipeline. These demonstrate different levels of
detail and use cases.

## Example 1: Minimal Brief (Text Only)

```
Campaign: Summer Refresh 2026
Brand: FreshBrew Coffee
Brand Colors: #2D5016 (forest green), #F5E6D3 (cream), #8B4513 (coffee brown)
Tone: Warm, inviting, premium but approachable
Target Audience: Urban professionals 25-40
Required Formats: Instagram square (1:1)
Scenes:
1. Steaming coffee on a sunlit cafe table with pastries, morning light
2. Cold brew bottle on a picnic blanket in a park, summer afternoon
Headlines: "Refresh Your Ritual" / "Crafted for Your Best Moments"
```

**Usage:**
```bash
python bria_marketing_agent.py generate --brief-text "Campaign: Summer Refresh 2026..." -o output
```

---

## Example 2: Detailed Campaign Brief

```
Campaign Brief: "Your Daily Pause"

Brand
Urban neighborhood cafe (modern, warm, design-forward, premium but approachable)

Objective
Increase foot traffic and social engagement by positioning the cafe as the perfect daily escape.

Target Audience
Young professionals (25-40), freelancers, creatives, and remote workers who value aesthetics,
quality coffee, and cozy environments.

Key Message
Your day deserves a pause. Great coffee. Warm space. Simple moments.

Visual Style
* Square format (1:1 ratio)
* Warm natural light
* Neutral tones (beige, brown, cream, soft green)
* Minimal, Instagram-friendly composition
* Authentic, not overly staged
* Shallow depth of field

Image 1 - Hero Product Shot
Concept: "The Perfect Cup"
A beautifully crafted latte with detailed latte art on a wooden table near a window.
Visual Direction:
* Close-up shot
* Soft morning light hitting the cup
* Croissant or small pastry slightly blurred in background
* Calm, warm, inviting mood
* Negative space for text overlay

Image 2 - Lifestyle Moment
Concept: "Your Work Break"
A stylish young professional sitting at a cafe table with a laptop and coffee.
Visual Direction:
* Side angle shot
* Person mid-30s, relaxed, natural expression
* Coffee cup in foreground
* Indoor plants and warm interior in background

Image 3 - Emotional Ambience Shot
Concept: "Slow Afternoon"
Wide square shot of the cafe interior with warm lighting and a few customers.
Visual Direction:
* Golden hour lighting
* Warm hanging lights
* Wooden textures, plants
* Slight candid feel
* Cozy and intimate atmosphere
```

**Usage:**
```bash
python bria_marketing_agent.py generate --brief daily-pause-brief.txt -o output
```

---

## Example 3: Product Launch with Ads

```
Campaign: EcoBottle Launch
Brand: GreenSip
Brand Colors: #1B5E20 (deep green), #E8F5E9 (mint), #FFF8E1 (warm white)
Tone: Fresh, sustainable, modern
Target Audience: Health-conscious millennials 25-35

Product: Reusable water bottle (product photo provided separately)

Scenes:
1. Product on a modern kitchen counter, morning sunlight, plants nearby
2. Product in a gym bag pocket, fitness setting, energetic lighting
3. Product on an outdoor cafe table, urban setting, afternoon light

Headlines: "Sip Sustainably" / "Your Daily Companion"
CTA: "Shop Now"

Ad Templates:
- Brand ID: 162
- Template IDs: 1274, 1290

Required Formats: 1:1, 16:9
```

**Usage (with product images and ad templates):**
```bash
python bria_marketing_agent.py generate \
  --brief ecobottle-brief.txt \
  --products ~/photos/bottle.png \
  --brand-id 162 \
  --templates 1274 1290 \
  -o output

# After image selection:
python bria_marketing_agent.py finalize \
  --candidates output/candidates.json \
  --selected edited_0 edited_2 product_0 \
  --brand-id 162 \
  --templates 1274 1290
```
