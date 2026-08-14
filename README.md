# Gradient Studio — Lite

A browser tool for making the COJE Members Club gradients. The whole thing is one
self-contained `index.html` — no build step, no dependencies, no package install.
Open the file locally or serve it from any static host and it runs.

This is a separate tool from [Gradient Studio](https://github.com/brendanwbrady/gradient-studio),
not a fork of it. The full tool keeps the wider feature set for other projects; this
one is scoped to COJE and deliberately much smaller.

---

## What it makes

It starts from the approved COJE gradient — a soft-edged rectangle of light on a dark
ground — and lets organic shadow eat into it. At rest the output *is* the approved
gradient. The shadow controls add interest without taking it out of family.

Every output is the same anatomy: a ground colour, the lit rectangle carrying a
three-stop gradient, an optional shadow eroding its edge, and grain on top. That
structure is fixed on purpose — it is the guardrail that keeps results in family.

### Export

| Format | Notes |
| --- | --- |
| **Copy SVG** | Paste straight onto a Figma canvas. Vector shapes, gradients, blur and grain arrive as editable layers. |
| **SVG** | Same output as a downloaded file. |
| **PNG 1× / 2× / 4×** | Rasterised from the identical SVG, so the two never drift apart. |

Frames: 4:5 (1080×1350), 16:9, 1:1, 9:16, and 14×10 (1680×1200). The last is the
Members Deck print size — a 2× export lands at 3360×2400, or 240dpi across 14 inches.

---

## The panels

**Palette** — the four room palettes, measured from the gradients in the deck. An
image can be sampled to pull a palette from a reference.

**Frame** — the five sizes above.

**Field** — the size and position of the lit rectangle, its blur, and the shadow:
its shape (Swell, Scallop, Wedge, Fray, Drift), how deep it reaches, how many bites,
how wide, how soft, and which edges it enters from. *Follow light* derives those edges
from the light direction; naming a side by hand overrides it and is then held.

**Light** — direction, where the ramp's midpoint sits, and how fast it falls away.

**Motion** — an optional drift, and its direction.

**Finish** — Exposure, Saturation, Contrast, Highlights, Midtones, Shadows, and grain.
These are calibration rather than exploration, so Randomize leaves them alone.

---

## Deploys

Hosting is entirely inside GitHub — GitHub Pages, driven by two workflows in
`.github/workflows/`. There is no external service and no build step.

**One-time setup**

1. Push this repo, including `.github/workflows/`.
2. The Deploy workflow runs and creates a `gh-pages` branch.
3. Settings → Pages → Source: *Deploy from a branch* → `gh-pages` → `/ (root)`.

The live site then serves from:

```
https://<owner>.github.io/gradient-studio-lite/
```

**What runs**

| Workflow | Trigger | Result |
| --- | --- | --- |
| `deploy.yml` | push to `main` | publishes to the site root |
| `preview.yml` | pull request opened or updated | publishes to `/preview/pr-<N>/` and comments the URL on the PR |
| `preview.yml` | pull request closed | deletes that preview folder |

Both authenticate with the `GITHUB_TOKEN` the runner creates automatically. No
personal access token, no secrets to configure.

**Worth knowing**

- Previews are per **pull request**, not per branch. Push a branch and open a pull
  request — draft is fine — to get a URL.
- GitHub Pages on a **private** repository requires a paid GitHub plan. On a free
  plan the repository has to be public for any of this to publish.
- Deploys take under a minute; Pages can take another minute to serve the new file.

---

## Working on it

Edit `index.html` and reload. That is the whole loop.

Read `NOTES.md` before touching the export. It records what the renderer has to do to
survive a paste into Figma, and several of its rules are not deducible from the code —
they were each found by a bug.

`REWORK.md` explains why the Field and Light panels are shaped the way they are, with
the measurements behind each decision. `LITE.md` covers what this build drops from the
full tool.

One thing worth knowing before adding a feature: six have been built and then removed
from this tool — rim light, an edge overlay, bloom, a shadow layer, a cut Dome form,
and an edge light at the frame. They shared a fault. Each added something *on top of*
the gradient, and the approved gradients do everything *inside* it.
