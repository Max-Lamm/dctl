# maxlammSat

**Advanced Saturation Control DCTL for DaVinci Resolve**

A precision saturation tool built for colorists who need more control than Resolve's built-in saturation offers. maxlammSat uses RGB luma-based saturation (not HSV) for natural, artifact-free results, with luminance zone targeting, a saturation limiter, additive/subtractive mix control, and gamut-aware soft clipping.

---

## Features

- **RGB Luma-Based Saturation** – `out = luma + (in - luma) × factor` preserves hue and avoids HSV neon artifacts
- **Multiplicative Gain** – scale from 0 (monochrome) through 1 (neutral) to 2 (double saturation)
- **Luminance Zone Control** (Low / Mid / High) – target saturation to shadows, midtones, or highlights
- **Sat Limiter** – protect high-saturation or low-saturation areas from over-processing
- **Sat Mix** – blend between subtractive (proportional) and additive (uniform) zone behavior
- **Gamut-Aware Soft Clamp** – per-channel roll-off prevents clipping while preserving hue
- **Show Curve** – real-time saturation transfer overlay with zone curves

## Parameters

| Slider | Range | Default | Description |
|---|---|---|---|
| **Gain** | 0.0 … 2.0 | 1.0 | Multiplicative saturation. 0 = mono, 1 = neutral, 2 = double |
| **Low** | -1 … +1 | 0.0 | Saturation offset in shadows |
| **Mid** | -1 … +1 | 0.0 | Saturation offset in midtones |
| **High** | -1 … +1 | 0.0 | Saturation offset in highlights |
| **Sat Limiter** | -1 … +1 | 0.0 | Neg: protect saturated pixels. Pos: protect neutral pixels |
| **Sat Mix** | -1 … +1 | 0.0 | -1 = subtractive, 0 = balanced, +1 = additive |
| **Show Curve** | on/off | off | Saturation transfer curve overlay |
| **Show Masks** | on/off | off | *Reserved* |
| **Hue Mask** | 0 … 1 | 0.0 | *Reserved* |
| **Luma Mask** | 0 … 1 | 0.5 | *Reserved* |

## Processing Pipeline

```
Input RGB
  │
  ├── Compute Rec.709 luminance
  ├── Measure pixel saturation (RGB distance from neutral)
  │
  ├── Gain (multiplicative base factor)
  │
  ├── Zone adjustment (Low/Mid/High weighted by luma)
  │     ├── modulated by Sat Limiter
  │     └── blended via Sat Mix
  │
  ├── Apply: out = luma + (in - luma) × factor
  │
  ├── Gamut-aware soft clamp (per-channel roll-off)
  │
  └── Output RGB
```

## Why RGB Luma-Based (Not HSV)

HSV saturation treats all channels uniformly, which leads to neon/plastic artifacts on saturated colors and discontinuities at hue boundaries. RGB luma-based saturation scales the distance of each channel from the luminance neutral point, which preserves the perceptual color balance and produces results identical to how professional color tools like Resolve's own saturation work internally.

At Gain = 0, you get exact Rec.709 luminance (pure monochrome). At Gain = 2, color distances from neutral are doubled proportionally. No hue shift, no artifacts.

## Soft Clamp

Instead of clamping abstract saturation values, maxlammSat operates gamut-aware: it computes per-channel headroom (how far each RGB channel can go before hitting 0 or 1), then applies a hyperbolic shoulder when the saturation factor approaches that limit. This means colors roll off smoothly toward the gamut boundary without hard clipping, and the hue is preserved throughout the roll-off.

## Luminance Zones

Zone targeting uses Rec.709 luminance for perceptually accurate separation:

```
Low   ████████░░░░░░░░░░░░  peaks at 0.0, fades by ~0.45
Mid   ░░░░████████████░░░░  peaks at 0.5, active ~0.15–0.85
High  ░░░░░░░░░░░░████████  peaks at 1.0, fades in from ~0.55
```

## Curve Overlay

When **Show Curve** is enabled:

- **Orange** — main transfer curve (midtone luminance)
- **Blue** — low zone curve (visible when Low is active)
- **Red** — high zone curve (visible when High is active)
- **Diagonal** — identity (no change)
- **Dark red line** — soft clamp threshold at 75%

## Installation

1. Copy `maxlammSat.dctl` into your DaVinci Resolve DCTL directory:
   - **macOS:** `/Library/Application Support/Blackmagic Design/DaVinci Resolve/LUT/DCTL/`
   - **Windows:** `C:\ProgramData\Blackmagic Design\DaVinci Resolve\Support\LUT\DCTL\`
   - **Linux:** `/opt/resolve/LUT/DCTL/` or `~/.local/share/DaVinciResolve/LUT/DCTL/`
2. Restart DaVinci Resolve (or refresh LUTs)
3. Apply via **Effects > ResolveFX Color > DCTL** on any node

## Roadmap

- [ ] Hue Mask – isolate changes to specific hue ranges
- [ ] Luma Mask – soft luminance masking with adjustable falloff
- [ ] Show Masks – visual overlay for active masks

## Technical Details

- **Saturation model:** RGB luma-based (`out = luma + (in - luma) × factor`)
- **Luminance:** Rec.709 (`0.2126R + 0.7152G + 0.0722B`)
- **Soft clamp:** Per-channel gamut-aware hyperbolic shoulder
- **Zone transitions:** Smoothstep blending

---

*Maximilian Lamm © 2026*
