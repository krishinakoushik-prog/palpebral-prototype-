# Palpebra

**On-device anemia risk screening, from a photo of your eyelid.**

Built for the iQOO hackathon (Hyderabad) — HealthTech track.

## What this is

Palpebra reads conjunctival pallor — the same sign clinicians check by
pulling down your lower eyelid and looking at the color — from a phone
camera, and scores anemia risk with a lightweight model that runs entirely
in the browser. No cloud calls, no image ever leaves the device.

Anemia affects a large share of Indian women and children, but screening
needs a blood test most people never get around to. This is a first pass at
closing that gap: not a diagnosis, but a nudge toward getting tested when it
matters.

## How it works

1. **Capture** — live camera with a guide overlay, or upload an existing photo
2. **Region of interest** — pixels inside the guide, ellipse-masked so
   surrounding skin doesn't skew the average
3. **Features** — three color signals pulled from that region:
   - CIELAB `a*` (the red–green axis)
   - HSV saturation
   - Redness ratio, `R / (R+G+B)`
4. **Model** — the three features run through a logistic regression, output
   as a continuous risk probability rather than a hardcoded if/else cutoff
5. **Result** — risk band (Low / Medium / High) plus the raw signal values,
   so it isn't a black box

Everything above runs client-side in plain JavaScript and the Canvas API —
no backend, no API keys, no build step.

## Status: model weights are provisional

The three-feature pipeline is real, and the scoring math is verified
monotonic — a paler reading always scores higher risk, never the reverse.
What isn't done yet: training on actual labeled patient data.

[CP-AnemiC](https://data.mendeley.com/) is a public, academically-reusable
dataset of 710 hemoglobin-annotated conjunctival images (children, Ghana)
built for exactly this task. A 2025 study quantized a MobileNet trained on
it to FP16 for edge deployment and held 92.5% accuracy — evidence this
class of model runs on phone-class hardware.

**To make it real:** pull CP-AnemiC, extract the same three features (or
fine-tune a small CNN), fit real logistic weights, and drop them into the
`WEIGHTS` object in `index.html`. The architecture doesn't change.

## Roadmap

- Fit real logistic (or small-CNN) weights against CP-AnemiC instead of
  today's provisional starting values
- Port the scoring pipeline to run natively through on-device NPU inference
  rather than in-browser JavaScript
- Add a short onboarding step (name, age, gender) so scoring can be
  calibrated against age- and sex-specific hemoglobin reference ranges,
  since normal Hb thresholds differ across those groups
- Generate a downloadable PDF summary per scan — the reading, the signals
  behind it, and a recommended next step

## Running it

It's one file. Camera access needs an actual HTTPS context (or
`localhost`) — it won't request permission from a plain double-clicked
`file://` page — so serve it rather than opening it directly:

```bash
npx serve .
```

## Deploying

Zero-config static site — works as-is on Netlify, Vercel, GitHub Pages, or
any static host. No framework preset needed; both serve `index.html`
automatically.

**Live demo:** [palpebra-prototype.netlify.app](https://palpebra-prototype.netlify.app/)

## Disclaimer

This is a screening signal, not a diagnosis. Color alone doesn't confirm
anemia. A Medium or High reading means "get an actual CBC blood test," not
"you have anemia."

## License

MIT — see [LICENSE](LICENSE).
