# maxlammSat

**Advanced Saturation Control DCTL for DaVinci Resolve**

A precision saturation tool built for colorists who need more control than Resolve's built-in saturation offers. MonacoSat separates saturation adjustments by luminance zone, lets you choose between additive and subtractive saturation behavior, and includes a limiter to protect specific saturation ranges from being pushed too far.

![Maximilian Lamm](https://maxlamm.com)

---

## Features

- **Multiplicative Gain** – scale saturation from 0 (fully desat) through 1 (neutral) to 2 (double)
- **Luminance Zone Control** (Low / Mid / High) – target saturation changes to shadows, midtones, or highlights
- **Sat Limiter** – protect already-saturated or near-neutral areas from over-processing
- **Sat Mix** – blend between subtractive (proportional) and additive (uniform) saturation behavior
- **Soft Clamp** – saturation rolls off smoothly above 75%, preventing hard clipping or oversaturation
- **HSV+RGB Hybrid** – automatic counter-desaturation in RGB space tames neon/plastic artifacts from HSV boosts
- **Show Curve** – real-time overlay showing the saturation transfer function with soft clamp threshold

## Parameters

| Slider | Range | Default | Description |
|---|---|---|---|
| **Gain** | 0.0 … 2.0 | 1.0 | Multiplicative global saturation. 1 = no change |
| **Low** | -1 … +1 | 0.0 | Additive saturation adjustment in shadows |
| **Mid** | -1 … +1 | 0.0 | Additive saturation adjustment in midtones |
| **High** | -1 … +1 | 0.0 | Additive saturation adjustment in highlights |
| **Sat Limiter** | -1 … +1 | 0.0 | Neg: attenuate on high-sat pixels. Pos: attenuate on low-sat pixels |
| **Sat Mix** | -1 … +1 | 0.0 | -1 = subtractive, 0 = balanced, +1 = additive |
| **Show Curve** | on/off | off | Saturation transfer curve overlay |
| **Show Masks** | on/off | off | *Reserved* |
| **Hue Mask** | 0 … 1 | 0.0 | *Reserved* |
| **Luma Mask** | 0 … 1 | 0.5 | *Reserved* |

## Processing Pipeline

```
Input RGB
  │
  ├── RGB → HSV
  │
  ├── Gain (multiplicative: sat × gain)
  │
  ├── Zone adjustment (Low/Mid/High weighted by Rec.709 luma)
  │     └── modulated by Sat Limiter
  │     └── blended via Sat Mix (additive ↔ subtractive)
  │
  ├── Soft Clamp (shoulder curve above 75%)
  │
  ├── HSV → RGB
  │
  ├── RGB counter-desaturation (proportional to boost amount)
  │     └── blends toward luma-neutral to tame HSV artifacts
  │
  └── Output RGB
```

## Soft Clamp

Saturation values are free to move in the 0–75% range. Above 75%, a hyperbolic shoulder curve rolls values off smoothly toward 100% without ever exceeding it. This prevents color clipping and maintains natural-looking results even with aggressive settings.

## HSV+RGB Hybrid

Pure HSV saturation boosts tend to produce neon/plastic-looking colors because HSV treats all channels uniformly. MonacoSat automatically applies a subtle counter-desaturation in RGB space (weighted by Rec.709 luminance) proportional to the saturation boost. This happens invisibly — the user controls a single Gain slider while the hybrid blend works in the background.

The counter-desaturation strength is currently set to 30% of the HSV delta. It only activates when saturation is *increased* (never when desaturating).

## How Sat Mix Works

- **Subtractive (−1):** `new_sat = sat × (1 + adj)` — proportional, preserves color relationships
- **Additive (+1):** `new_sat = sat + adj` — uniform shift, can push neutrals into color
- **Balanced (0):** 50/50 blend

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
- **Blue** — low zone curve (visible when Low slider is active)
- **Red** — high zone curve (visible when High slider is active)
- **Diagonal** — identity reference (no change)
- **Dark red horizontal line** — soft clamp threshold at 75%

## Installation

1. Copy `MonacoSat.dctl` into your DaVinci Resolve DCTL directory:
   - **macOS:** `/Library/Application Support/Blackmagic Design/DaVinci Resolve/LUT/DCTL/`
   - **Windows:** `C:\ProgramData\Blackmagic Design\DaVinci Resolve\Support\LUT\DCTL\`
   - **Linux:** `/opt/resolve/LUT/DCTL/` or `~/.local/share/DaVinciResolve/LUT/DCTL/`
2. Restart DaVinci Resolve (or refresh LUTs)
3. Apply via **Effects > ResolveFX Color > DCTL** on any node

## Roadmap

- [ ] Hue Mask – isolate saturation changes to specific hue ranges
- [ ] Luma Mask – soft luminance masking with adjustable falloff
- [ ] Show Masks – visual overlay for active mask regions
- [ ] Configurable RGB counter-strength slider (currently fixed at 30%)

## Technical Details

- **Saturation model:** HSV with automatic RGB counter-desaturation
- **Zone weighting:** Rec.709 luminance (`0.2126R + 0.7152G + 0.0722B`)
- **Soft clamp:** Hyperbolic shoulder `f(x) = 0.75 + 0.25 × (x-0.75) / (x-0.75 + 0.25)`
- **Transitions:** Smoothstep-based zone blending

---

*maxlamm © 2026*
