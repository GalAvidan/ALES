# Skill: Color Grade

## Purpose

All rendered scenes must have the project LUT applied before delivery. This ensures a consistent
look across scenes rendered at different times or by different tools.

## LUT File

- **Path:** `src/assets/luts/brand-grade-v2.cube`
- **Format:** .cube (32×32×32)
- **Application point:** Apply after compositing, before encode.

## Grade Intent

The LUT produces:
- Slightly desaturated shadows (cool, clean feel)
- Warm highlights (product feels inviting)
- Neutral midtones (skin tones and product colors read accurately)

## Rules

1. Always apply the LUT at 100% opacity. Do not blend it.
2. The LUT is designed for Rec.709 input. If source footage is in a different color space,
   convert to Rec.709 *before* applying the LUT.
3. Scene 5 (Logo Hold) is exempt — it is a pure white background with vector assets.
   Do not apply the LUT to the logo hold scene.
4. After applying the LUT, check that the product's primary color matches the brand reference:
   `src/assets/brand/product-color-reference.png`. If it does not match within ΔE < 3, flag and ask.
