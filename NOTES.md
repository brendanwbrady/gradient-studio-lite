# Engine notes

Constraints the render engine depends on. Each was arrived at by fixing a real
failure, and each is easy to break by accident because **the browser keeps rendering
correctly after the mistake** — the damage only shows up on import into Figma, or at a
different render scale.

Read before changing `buildSVG()` or anything it calls.

---

## 1. The noise filter must match Figma's own grammar exactly

Figma emits exactly two shapes of noise filter, and its importer recognises those two
and nothing else. An unrecognised primitive anywhere in the chain causes the **whole
filter to be dropped**, which shows up as grain silently vanishing on paste.

Permitted chains:

```
feFlood → feBlend → feTurbulence → feComponentTransfer → feComposite → feMerge
feFlood → feBlend → feTurbulence → feColorMatrix(luminanceToAlpha)
        → feComponentTransfer → feComposite → feFlood → feComposite → feMerge
```

A `feColorMatrix type="saturate"` was once inserted mid-chain to desaturate the
speckle. It is not a shape Figma emits, and it broke the import.

Where tonal control over grain is needed, it goes on the **plate** — the rect's fill,
opacity and blend mode are ordinary layer properties that survive import — never as a
filter primitive. The plate currently carries a three-stop gradient so grain eases off
at both ends of the tonal range.

Filter and gradient IDs follow Figma's own naming (`filter0_f_1234_56`,
`paint0_linear_1234_56`). Cheap to maintain, and it matches what the importer expects.

## 2. Noise frequency must stay clear of the pixel sampling limit

`baseFrequency` is in user units. At `2`, the noise period is half a pixel — right at
the sampling limit, where rendered strength swings unpredictably with render scale.

That single value was the root cause of a long run of unrelated-looking bugs: grain
"too strong" on load, grain "missing" from PNG export, grain inconsistent between
preview and export. All one problem.

**Keep the period above roughly one pixel.** The current `0.85` gives ~1.18px.
Measured grain now agrees to within 1% across the on-screen preview, the exported SVG
at native size, and the PNG export.

## 3. No transform attributes in exported geometry

Scale, rotation and position are resolved in JavaScript into absolute path
coordinates. Nothing in the export carries a `transform`, and no filtered group
contains another transformed group.

This applies to Smear, Veil and every motion sample. `bake()` and `rotPts()`
exist for exactly this purpose.

Consequence worth knowing: a `userSpaceOnUse` gradient is normally transformed along
with its element. With coordinates baked, gradient endpoints must be transformed
explicitly too — Smear and Veil do this so the fade stays aligned to the band.

## 4. Blur radii stay symmetric

`feGaussianBlur stdDeviation="180 12"` renders beautifully in a browser and is not
something Figma's uniform layer blur can represent.

Motion is therefore **geometric, not filtered**: the shape is sampled several times
along its travel and composited, each sample fainter than the last. Across and Along
offset the samples, Spin rotates them through an arc. Every blur in the file is a
single symmetric value.

The cost is layer count. Motion multiplies the field by five to nine samples; drift
has reached eighteen groups against four at rest. Motion is the expensive
control.

## 5. Blur eats the ends of a gradient ramp

A colour that exists only at the extreme end of a ramp sits at the very edge of the
shape, where blur immediately averages it against the ground. The authored colour then
never actually appears in the output.

**Falloff** exists to solve this: it holds both the light and the ground as plateaus
so each survives the blur, and bends the shoulders so the transition reads as decay
rather than as a linear blend. Measured on a light stop of luminance 197, peak output
went from 164 to 196 once this was in place.

## 6. Blend modes go on the group AND on the shape

Browsers and Figma disagree about where a blend belongs when the element sits inside
a filtered group, and each ignores the other's placement.

**A browser honours the blend on the GROUP.** Put it on the shape inside a filtered
group and it is dropped entirely — measured, moving it there rendered identically to
having no blend at all: luminance +2.30, saturation -5.66, grain +0.318 against the
correct render.

**Figma's own SVG exports put it on the SHAPE**, with the filter on the group. That is
what its importer reads, which is why a group-only blend came into Figma looking
lighter, flatter and grainier than the tool's preview — by roughly those same numbers.

Write it in **both places**. The browser takes the group's and ignores the shape's;
Figma takes the shape's. Verified pixel-identical in the browser against group-only:
mean difference 0.036, max 1.

The general lesson: when the preview and Figma disagree, diff our export against a
real Figma export of a similar artwork and look at *where* attributes sit, not just
whether they are present. Both files had the same blend modes; only the placement
differed.

## 7. Shadows lift a floor, highlights apply a gain

The two tonal controls are not mirror images and must not be built as one.

**Shadows are additive** on lightness. Lifting the floor is what the control is for,
and nothing multiplicative can raise a near-black ground at all — a gain applied to
zero is still zero.

**Highlights are a gain.** Adding lightness in HSL drags a colour toward white as it
brightens, which is not more light, it is a wash: measured, the old additive version
took Rosa's bright band from 61% chroma to 37% and Gold Bar's from 48% to 29% while
appearing to work perfectly on a luminance readout. Scaling the channels together
holds hue and saturation instead — the same colour, more of it.

Above unity the gain runs through the screen curve, `1-(1-x)^g`, rather than a
straight multiply. A straight multiply clips whichever channel hits 255 first, which
is itself a hue shift; the curve brightens without ever clipping one channel alone.
Measured at Highlights 100 with Exposure 60: no fully blown pixels and no
single-channel clipping anywhere in the frame.

Worth remembering that the fault was invisible to the obvious measurement. The
control moved the bright band by 55 to 108 levels in every palette, so a luminance
sweep said it worked. Only measuring **chroma** alongside luminance showed what was
actually wrong.

## 8. A new form needs a key nothing else answers to

`unitOutline` is a chain of `if(form===...)` branches, and the full tool's retired
forms are still in the file — unreachable only because nothing lists them. Adding a
second branch under a key that already exists puts it after the first, where it never
runs. A form written as `vault` sat behind the full tool's geometric Vault and
rendered that instead, silently: three different attempts at the shape all produced
byte-identical output, which is the symptom to watch for. Byte-identical results from
genuinely different code mean the code is not being reached.

## 9. A shadow only reads where there is light to take away

Two of the organic forms — **Slat** and **Rift** — subtract light rather than adding
it, multiplying in the ground colour. Both were first built entering from the dark
end of the light axis, which is where a shadow would physically fall and where they
were completely invisible: measured, Rift was darkening the ground at four times the
rate it touched the lit band. Placing them across the middle of the ramp, so their
edges cross the light, took that from 1.7 inside the lit area against 8.3 outside to
54 against 10.

The general rule for anything that occludes: measure the darkening **inside the lit
area** against outside it. If the ratio is the wrong way round the control will feel
dead no matter how strong it is.

## 10. The rim's corners are geometry, not blending

Four edge bands screening over each other double up where they overlap and blow the
corners, so they are **mitred** at 45 degrees and tile instead. Three things follow
from that, all of which showed up as a visible mask edge before they were fixed:

- **Mitre only where two lit edges meet.** An edge facing away from the light is
  skipped entirely, which left its neighbour's diagonal cut running across a bare
  corner with nothing beside it. Each end is now squared off unless the edge it
  abuts is also lit. The choice has to be binary — cutting both ends by the full
  depth tiles the corner exactly, and cutting neither does too, but cutting by a
  fraction leaves a wedge uncovered.
- **A wide facing lobe, not a bare dot product.** With a bare one an edge at right
  angles to the light drops to zero while its neighbour sits at full, and the mitre
  between them reads as a drawn line. `0.30 + 0.70*dot` keeps some light on every
  edge but the fully opposite one, so neighbours never differ by more than about
  three to one.
- **The glow's blur is floored against its own depth.** A wide band has a long
  diagonal at each corner, and a blur of a few percent of it leaves that diagonal
  crisp. The floor is 14% of the depth. The core keeps its own much smaller blur, so
  the hot line at the very edge stays sharp — this is why the rim is two passes.

Measured across 45 combinations of direction, width and feather: seams above the
grain floor fell from 16 to 6, worst from 33.5 to 15.0.

## 11. Motion is the expensive control

Motion is geometric, so every sample is a real layer. Drift reaches about 18 layers
and 43 KB, against 5 and 8 KB at rest. Everything still edits normally in Figma, but
that is the cost of the control.

**Motion copies must average, not stack.** Painted source-over at their own weights,
fifteen copies of one shape reach full alpha wherever they overlap, so a trail comes
out as a solid slab rather than a graded blur — and any edge running along the
direction of travel is drawn by every copy at once, which turns it into a hard seam.
A horizontal drift produced exactly that at the field's lower edge.

Each copy composites at `w / (running sum of w)`, which makes the result the weighted
mean of them all — the definition of a motion blur. The first copy lands at full alpha
and each later one contributes proportionally less. Measured on the seam: sharpest
horizontal step fell from 3.19 to 1.79, and motion stopped inflating the overall
brightness of the frame as a side effect.

Two further rules keep motion from breaking a hard-edged shape:

- The blur grows to at least **half the gap between copies**, so stamps fuse into a
  continuous trail rather than reading as discrete ghosts.

## 12. A metric can pass while the picture dies

Softening Dome was tuned against a crown-transition measurement, which rewards more
blur right up to the point where there is nothing left to soften. At 9.5x the radius
it read 65% against the reference's 67% — a near-perfect score on an even smudge with
no picture in it. The metric measured softness; it could not measure whether anything
survived.

Any metric that improves monotonically with one parameter needs a second measurement
that fails in the other direction — here, contrast, which collapsed while the
softness score climbed. Render and look before accepting a number that only ever
gets better.

## 13. Verify against measurement, not appearance

Several of the above were invisible by eye and obvious in numbers. Useful checks:

- **Grain stability** — high-pass sigma of preview vs PNG vs SVG-at-native-size.
  These three must agree.
- **Grain neutrality** — mean, shadow and contrast shift between grain 0 and grain
  20 should all be near zero. Grain that moves them is compressing the picture.
- **Noise period** — `baseFrequency ÷ Grain Scale` must stay under about 1.0, or the
  period drops below a pixel and the texture changes with render size. The Scale
  slider's floor exists for this reason.
- **Highlight fidelity** — brightest pixel in the output vs the authored light stop.
- **Export cleanliness** — count `transform=`, `stdDeviation="N N"`, and `<g...><g` in
  the exported SVG. All three should be zero.
- **Held settings** — after N randomizes, confirm frame and every Finish control is
  unmoved.

Contact sheets built from downscaled thumbnails introduce their own artefacts —
this has produced a false "hard edge at the frame" three separate times, and each
time the export was clean at 1:1. Verify suspected rendering faults at full
resolution before chasing them.
