# Movement Coach Website — Design Prototypes

Three variants built for Alex Wright's hand-balancing / movement coaching website.

## Goals (ranked)
1. Show name prominently at the top
2. Crop first photo tighter on the body (less wall)
3. Simplify menu items

## Variants

| Variant | Name Treatment | Photo Treatment | Feel |
|---------|---------------|-----------------|------|
| A · Bold Name + Crop | 1.75rem top-left, bold | 3/4 portrait, `object-position: 50% 20%` | Classic, safe |
| B · Name Overlay | 1rem overlaid on photo (weakest) | Full-bleed 85vh, gradient overlay | Bold, cinematic |
| C · Clean Minimal | 2rem centered, own line (strongest) | 3/4 portrait, same as A | Calm, boutique |

## Claude Code Design Review (2026-09-02)

**Recommendation: Variant C**

- Only variant that clearly nails goal #1 (name prominence) — centered 2rem name at top is strongest
- Ties A for cleanest photo crop (both use controlled portrait frame)
- B is most striking but disqualifies itself by shrinking name to corner label

**Unmet goals across all three:**
- Goal #3 (menu simplicity) — all variants carry same 5-item menu. Recommend trimming to 3–4: About · Teach · Gallery · Contact
- Goal #2 (photo crop) — could go further with `object-position: 50% 15%` or slight zoom

**If client prefers dramatic/athletic over calm/editorial:** B is fallback, but only after bumping overlaid name to ~1.75rem+

## Files
- `index.html` — switcher/comparison page
- `variant-a.html` — conservative
- `variant-b.html` — bold
- `variant-c.html` — minimal

All photos reference GitHub-hosted originals via absolute URLs.
