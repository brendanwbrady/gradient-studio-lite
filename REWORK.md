# Field and Light, reworked

A step back rather than another iteration. Written against the five gradients
currently in the deck, measured: a soft-edged rectangle of light inset about 6% at
the sides, occupying roughly the top 60% of the frame, on a dark ground, with an edge
softness of 4 to 9% of the width.

## The thinking

Everything built in the last several rounds — rim light, Edge, Bloom, Shade, Dome, an
edge light at the frame — **added a layer on top of the approved gradient**. Each one
therefore looked like something applied to it, and each was rejected for exactly that
reason. The brief is not a new picture; it is the approved picture with more going on
inside it.

So this rework does the opposite. It **subtracts from the one shape that was already
approved**. The rectangle's own edge is bitten into, and because the whole shape is
blurred as a single object, each bite reads as shadow reaching in rather than as a
form laid over. At rest the output *is* the approved rectangle. At full depth it is
still recognisably that rectangle, with something eating into it.

## Field

The field is **always the approved rectangle**. The chip row that used to pick a form
now picks the **shadow's character** — five ways of eating into that rectangle rather
than five different rectangles. Orb is retired with the rest; it was the last survivor
of the shape zoo.

| preset | what it is |
|---|---|
| **Swell** | one broad soft mass leaning in |
| **Scallop** | an even row of rounded bites |
| **Wedge** | steep on one flank, a long tail on the other |
| **Fray** | many shallow bites, a ragged edge |
| **Drift** | uneven — one dominant, the others incidental |

Each preset is a profile along the perimeter plus a spread of depths and widths and
how far the seed may vary them. The three sliders scale whatever the preset asks for,
so the preset sets the character and the sliders set the amount.

Every profile is **wider than it is deep** at the default depth, deliberately: a bite
deeper than it is wide comes to a point, and a point reads as a spike stabbing the
field rather than a shadow lying across it. That was the first tuning pass's mistake.

| control | what it does |
|---|---|
| Width, Height | the field's size against the frame |
| Offset X, Y | where it sits |
| Blur | its softness, as before |
| **Shadow · Depth** | how far the bites reach in |
| **Shadow · Count** | how many, 1 to 6 |
| **Shadow · Spread** | how wide each one is |
| **Shadow · Enters from** | which edges they come from, multi-select |
| Seed, Shuffle | which arrangement |

Each bite is a raised cosine along the perimeter, so it has no corners of its own and
no seam where it meets the straight part of a side. Bites taper to nothing near the
rectangle's corners and are never centred there — displacing a corner tears a notch
instead of taking a bite.

## The shadow is its own object

It was first cut out of the field's outline, which had two faults.

**A shadow could never be softer than the edge it ate into**, because the field's blur
was the only softness available.

**And reshaping the outline moved light as well as removing it.** Measured, every
preset ADDED up to 6 levels of light somewhere in the frame, wherever the eroded edge
fell outside the original rectangle.

The field is now drawn as the approved rectangle, untouched, and the shadow is the
region between that rectangle and the eroded outline — filled with a lowered ground,
blurred on its own terms, and **multiplied**. A multiply can only darken, so the
second fault cannot recur by construction: measured across 30 combinations of preset,
depth and field blur, light added is **0.0** everywhere.

Its blur is floored well above the field's, so the shadow is softer than the edge it
lies against whatever Blur is set to — about four times the field's radius at the
default. Measured, the shadow's own edge reads 1.8 to 2.9 against the field edge's
8.9. A **Softness** slider takes it further.

The tone is the ground lowered, not black: a shadow that reaches full black stops
reading as shadow and starts reading as a hole cut in the picture.

## Three faults found by measurement

**The shadow moved when unrelated settings changed.** It drew from the random stream
every other feature also draws from, so altering Blur or Middle point re-rolled the
bites — measured, unrelated settings shifted its footprint by up to 5%. It was also
built on the held box, which shrinks with Blur, so raising Blur slid the shadow as
well as softening it. It now takes a stream seeded from Seed alone and the box the
user asked for. Verified by diffing the shadow's own path in the export: byte
identical across Middle point, Falloff, Grain, Saturation and Blur, and changing only
for Width, which genuinely resizes the field it lives on.

**Count did not mean what it said.** It was multiplied by a per-preset factor, so
asking for three on Swell gave one. Count is literal now; the presets differ by the
width, depth, lean and lumpiness of each bite, not by how many the slider means.

**The shadow's blend had to satisfy two constraints at once**, and only `darken`
does:

- `multiply` made it invent a black the palette does not contain — ground multiplied
  by ground is ground squared, rgb(6,2,3) against a ground of rgb(26,12,18);
- painting it plainly bottomed out at the ground correctly, but *lightened* the few
  places already darker than the ground by up to 2.5 levels, because the depth blob
  puts them there.

Darken cannot go above what is underneath, so it never adds light, and cannot go
below the tone it is given, so the palette's ground is the deepest it reaches.
Measured across 15 combinations of preset and depth: light added **0.00**, and the
darkest pixel inside a shadow is rgb(26,11,17) against the ground's rgb(26,12,18).

## Amorphous, not geometric

A raised cosine on its own gives one belly and two matching flanks, which reads as a
scallop cut by a machine. Real cast shadows are lumpy — their outline wanders on
several scales at once, they are fatter on one side than the other, and no two lobes
match. Each bite now carries its own small set of harmonics at frequencies that are
not multiples of each other, faded out at the bite's rim so it still meets the
straight edge cleanly however lumpy its belly is. How lumpy is part of each preset:
Swell and Scallop stay smooth, Fray and Drift wander most.

**Bites are gated to the sides they were asked for.** Widths are a fraction of the
whole perimeter, so a broad bite would otherwise reach around a corner and eat a side
nobody selected — which is what tore the corners open in the first pass. Sharpest step
across all five presets on all four sides now measures 8.9, exactly the plain
rectangle's own figure: nothing the erosion does is sharper than the shape it started
from.

**Warp is gone** — Amount, Detail, Variety and Field direction. It was a wobble
applied to the whole outline, which is the same idea as erosion done less
controllably and with no sense of light or shadow.

## Palettes, read from the deck

The presets had never been updated to the gradients actually in the deck, and they had
drifted a long way through the darkening and lifting rounds. The deck's grounds sit
close to COJE's originals — #410813 against the original #420714 for Member's Bar —
while ours had been lifted to #2C050D.

All four are now measured from a deck gradient: ground from its darkest decile, mid
from the median band, light from its brightest few percent.

| | ground | mid | light |
|---|---|---|---|
| Dining Room | #0B0A0A | #1F1A14 | #332617 |
| Gold Bar | #3E1127 | #8C5431 | #D59C65 |
| Member's Bar | #410813 | #5E4656 | #747486 |
| Rosa Lounge | #381507 | #762722 | #CF8042 |

**The mapping of deck gradient to room name is a guess worth checking.** Member's Bar
and Rosa Lounge are unmistakable by colour family; Gold Bar and Dining Room less so. A
fifth deck gradient — #410813 / #673224 / #9A6A3B, warm brown on maroon — matches no
room name and is not included.

The traditional palettes are removed.

## The default

It opened with the shadow at 20% depth, which put a dark mass across half the frame.
That was a demonstration value that had been left in as a resting one. It opens at 13%
now, which departs from the plain approved rectangle by 0.99 — present, but the
gradient is still plainly the approved gradient.

## The shadow follows the light

**Follow light** is the default for which edges the shadow enters from. It is derived
from the light direction rather than chosen: light from the top puts the shadow along
the bottom, light from the right puts it along the left, and a diagonal puts it on
the two far edges at once, which is what a raking light does to a shape.

| light | shadow enters from |
|---|---|
| Top | Bottom |
| Top R | Bottom + Left |
| Right | Left |
| Btm R | Top + Left |
| Bottom | Top |
| Btm L | Top + Right |
| Left | Right |
| Top L | Bottom + Right |

**Naming a side by hand still overrides it**, and the choice is held when the light
moves afterwards. A shadow the light would not cast is sometimes exactly what a
composition wants, and the control should not argue. Clearing the last named side
returns to Follow light rather than leaving the shadow with nowhere to be.

**One thing that reads backwards until you look at why.** The shadow's visible mass
sits *toward* the light, not away from it — measured across all eight directions, its
centroid is on the lit side every time, and mirrors exactly when the light flips
(Top -0.057 against Bottom +0.056; Right +0.092 against Left -0.093). That is correct:
the bite enters from the far edge, where the ramp has already gone dark and darkening
changes nothing, and only becomes visible where its crown reaches into the light. A
shadow lying across a lit surface looks exactly like this.

## Light

Unchanged: Direction, Middle point, Falloff. It was not the problem.

## What this is not

It does not reproduce the reverse-arch reference. That picture needs a backdrop
lighter than the field's darkest point, which the palettes cannot express while the
ground is both backdrop and ramp end — the analysis is in `LITE.md` and the change
it would need is a palette-level decision, not a field control.
