# maxlammSat

**Advanced Saturation Control DCTL for DaVinci Resolve**

A precision saturation tool built for colorists who need more control than Resolve's built-in saturation offers. MonacoSat separates saturation adjustments by luminance zone, lets you choose between additive and subtractive saturation behavior, and includes a limiter to protect specific saturation ranges from being pushed too far.

![Maximilian Lamm](https://maxlamm.com)

---

## Features

- **Global Gain** – raise or lower saturation across the entire image
- **Luminance Zone Control** (Low / Mid / High) – target saturation changes to shadows, midtones, or highlights independently
- **Sat Limiter** – protect already-saturated or near-neutral areas from over-processing
- **Sat Mix** – blend between subtractive (proportional) and additive (uniform) saturation behavior
- **Show Curve** – on-screen overlay showing the saturation transfer function in real time

## Parameters

| Slider | Range | Description |
|---|---|---|
| **Gain** | -1 … +1 | Global saturation adjustment |
| **Low** | -1 … +1 | Saturation adjustment in shadow luminance range |
| **Mid** | -1 … +1 | Saturation adjustment in midtone luminance range |
| **High** | -1 … +1 | Saturation adjustment in highlight luminance range |
| **Sat Limiter** | -1 … +1 | Negative: attenuate effect on high-saturation pixels. Positive: attenuate effect on low-saturation pixels |
| **Sat Mix** | -1 … +1 | -1 = fully subtractive, 0 = balanced, +1 = fully additive |
| **Show Curve** | on/off | Display saturation transfer curve overlay |
| **Show Masks** | on/off | *Reserved for future use* |
| **Hue Mask** | 0 … 1 | *Reserved for future use* |
| **Luma Mask** | 0 … 1 | *Reserved for future use* |

## How Sat Mix Works

The Sat Mix slider controls *how* saturation is applied:

- **Subtractive (−1):** `new_sat = sat × (1 + adjustment)` — proportional scaling that preserves relative color relationships. Already-desaturated areas are affected less.
- **Additive (+1):** `new_sat = sat + adjustment` — uniform shift that pushes all pixels equally, including near-neutral tones.
- **Balanced (0):** A 50/50 blend of both methods.

## Luminance Zones

Zone targeting uses Rec.709 luminance weights for perceptually accurate separation:

```
Low   ████████░░░░░░░░░░░░  peaks at 0.0, fades by ~0.45
Mid   ░░░░████████████░░░░  peaks at 0.5, active ~0.15–0.85
High  ░░░░░░░░░░░░████████  peaks at 1.0, fades in from ~0.55
```

Zones overlap smoothly to avoid hard transitions.

## Curve Overlay

When **Show Curve** is enabled, a graph appears in the bottom-left corner:

- **Orange** — main transfer curve (midtone luminance)
- **Blue** — low zone curve (only visible when Low slider is active)
- **Red** — high zone curve (only visible when High slider is active)
- **Diagonal** — identity reference (no change)

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

## Technical Details

- **Color model:** HSV for saturation manipulation
- **Zone weighting:** Rec.709 luminance (`0.2126 R + 0.7152 G + 0.0722 B`)
- **Transitions:** Smoothstep-based zone blending for artifact-free results

---

*maxlamm © 2026*
