---
name: analyzing-designs-for-code
description: Use when translating a design image/screenshot/mockup into exact UI code — measuring colors, sizes, spacing, ratios, or when a rendered UI "looks off vs design" and you need precise hex/dimensions instead of guessing.
---

# Analyzing Designs for Code

## Overview
Never eyeball a design. Extract exact pixels with PIL/numpy, work in **ratios** (not fixed px), confirm the spec, then code once. A wrong guess costs a full build/OTA cycle.

## Workflow
1. **Crop** the relevant region from the full design board (designs are often multi-phone boards — isolate the target phone first).
2. **Upscale NEAREST** (`Image.NEAREST`, 8–12×) to SEE true structure before measuring. This reveals nested shapes (e.g. two concentric circles vs one bordered circle) that look identical at thumbnail size.
3. **Measure** colors + ratios (below).
4. **Confirm** hex/ratios with the user when ambiguous.
5. **Code once**, then **re-measure the rendered screenshot** to verify against design.

## Measuring colors
- Sample **solid centers**, never anti-aliased edges (AA blends colors and poisons thresholds).
- Use `Counter(map(tuple, pixels))` to get dominant colors; ignore the board/background color.
- For layered/two-tone icons, do a **radial profile** from the centroid outward — each color band = one layer.

## Measuring ratios (the important part)
- **Compare ratios, not fixed pt.** Design px ≠ device pt; render scale differs. Measure `element / container` (e.g. icon glyph height ÷ bar height) and reproduce that ratio in code.
- Convert design→pt only via `element/screen_height × device_pt` when you need an absolute.
- Detect element bounding boxes with a color mask, then `xs.min()/max()`, `ys.min()/max()` for size; centroid for centering; row/column scans for gaps & alignment.
- **PNG assets have transparent padding.** Before sizing an `<Image>`, measure glyph fill = `alpha_bbox / canvas`. To hit a target visible-glyph ratio R: `imageBox = R × container / fillRatio`.

## Example (PIL + numpy)
```python
from PIL import Image
import numpy as np
im = Image.open("crop.png").convert("RGB")
a = np.array(im).astype(int)
# radial profile to find concentric layers
cy, cx = a.shape[0]//2, a.shape[1]//2
for r in range(0, 24):
    ring = [a[int(cy+r*np.sin(t)), int(cx+r*np.cos(t))]
            for t in np.linspace(0, 2*np.pi, 24)]
    m = np.mean(ring, 0).astype(int)
    print(r, "#%02x%02x%02x" % tuple(m))
# bounding box of a color mask → size & ratio
mask = (a[:,:,1]-a[:,:,0] > 25) & (a[:,:,1]-a[:,:,2] > 25)  # greenish
ys, xs = np.where(mask)
print("diam", xs.max()-xs.min()+1, ys.max()-ys.min()+1)
```

## Common mistakes
- Assuming a thin **border** when the design is two **nested filled circles** — upscale NEAREST to check.
- Sampling on AA edges → muddy hex. Sample plateaus.
- Comparing absolute px between design and a 2x/3x screenshot — always normalize to a ratio.
- Forgetting PNG transparent padding → icon renders smaller than intended.
- Trusting one scanline through a circle that isn't the true center — scan the widest row.

## Verify before claiming done
Re-open the *rendered* screenshot and re-measure the same ratio. Design ratio == app ratio → done.
