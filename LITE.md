# Gradient Studio Lite

A configuration of Gradient Studio scoped to the COJE project. The full tool is
kept intact alongside it for future work; nothing here removes capability from
that build.

## What changed, and why

**Palette** — the four room palettes are unchanged. The traditional set is halved
from ten to five: Yamabuki, Kakishibu, Akane, Budou, Sumi. The references sit in a
narrow warm band (measured hues of 24 to 51 degrees across all seven), so the
blues, greens and pinks were doing nothing for this project. Kakishibu returns
because persimmon is squarely in that band.

**Frame** — 4:5, 16:9, 1:1, 9:16 and 14x10. Phone is gone; 14x10 is the Members
Deck's print size, and at 1680x1200 a 2x export lands at 3360x2400 — 240dpi across
14 inches.

**Field** — five forms: Field, Wave, Drape, Swell, Orb. This is where the work
happens, so Hand is folded in: the warp controls and Seed sit with the rest of the
field's controls rather than in a panel of their own.

**Light** — unchanged.

**Motion** — Spin removed. On a soft field it reads as a smudge rather than as
movement, and none of the references show anything like it.

**Finish** — no rim, no edge overlay. **Midtones** is added, completing Highlights / Midtones / Shadows. It is a
bell centred where the middle stop sits, tapering to nothing at both ends so it
cannot fight the other two, and like Highlights it is a gain rather than an added
lightness — the middle of the ramp brightens without draining its colour. Measured
at full: midtones move 30 levels, shadows 2.5.


## Field geometry: size and position

Margin and Width were two names for one move — both shrank the box symmetrically
about its centre. Measured, Margin X 22% and Width 74% produced lit areas 716px and
675px wide: the same thing twice.

They are now genuinely different jobs:

| control | what it does |
|---|---|
| **Width / Height** | the field's size against the frame, 12% to 200% |
| **Offset X / Y** | where it sits, plus or minus 50% of the frame |

Anything margin used to do — including bleeding off an edge — is a size over 100%.
Off-centre compositions are reachable now, and were not before: Offset Y moves the
field 324px while changing its size by 1px.

**Randomize rolls both axes from one base**, then varies each within 15% of it, so a
roll can come out a band or a column but usually reads as a field. Across 40 rolls no
value lands outside its own slider range, and none produces a sliver.

That check exists because of the bug it would have caught: when margin became offset,
the old margin roll was renamed onto Width and Height, feeding margin-sized numbers
into a size control. Every roll came out between -3% and 12% — some of them negative.
A rename that compiles is not a rename that works.

## The base gradient

Defaults are matched to the reference itself, not to an average of the seven. That
reference's lit mass measures **0.88 of the frame wide by 0.55 tall, covering 43.7%**,
with the frame edge 12.7 levels above the darkest value and a top edge that wanders
1.7px.

Three settings had to move together, which is why calibrating any one of them alone
kept failing:

- **Width 90%, height 80%.** The earlier 76% came from calibrating the edge gap
  alone. It held the ground but made the field narrower than anything the references
  show — 0.74 against 0.88.
- **Blur 4%, down from 10%.** Width alone could not close the gap: at 90% wide the
  lit mass matched but the edge gap doubled to +25, because the blur spreads the
  field's light past its own boundary and a wide field leaves less frame to absorb
  it. The blur had to come down with the size.
- **Warp 6%, down from 35%.** With the blur at 10% the default warp was hidden; at
  4% it was suddenly visible, and the base read as a wobbly shape where the reference
  is a clean soft rectangle. Measured, 35% wandered 10.4px against the reference's
  1.7px. 6% measures 2.2px — still organic, not yet a shape.

The result measures 0.90 by 0.52, covers 44.0%, holds an 11.4 edge gap and wanders
2.0px. Every control keeps its full reach from there.


## Frames

**14x10** is added for the Members Deck's print size. At 1680x1200 a 2x export lands
at 3360x2400 — 240dpi across 14 inches.

## Drape

The hem is **one continuous curve** now. Each bay used to be its own cosine pinned
back to the hem line at both ends, so where two bays met the curve returned to the
line and set off again — a hard split at every peak instead of a rise and a fall.
Summing three harmonics gives the same uneven bays with nothing to butt against: the
hem is one function across the whole span, so it can only rise and fall smoothly.
Measured along the shaped edge, the median step is 2px and the sharpest 11px.

## Field direction

There used to be two directional systems that did not agree. A form's orientation was
welded to the light — Drape's hem always hung at the dark end, so moving the light
rotated the cloth, and on a diagonal the form snapped to whichever cardinal was
nearer. Warp From meanwhile worked in frame terms and ignored the light entirely. Two
controls, two coordinate systems, and no way to say *hang the cloth from the top and
light it from the side*.

They are one control now. **Field direction** anchors the form and biases the warp to
the same edge, so the shape reads as coming from one place. **Follow light** keeps the
old behaviour for anyone who wants them locked together.

**The control names the edge being shaped, not the edge being pinned.** That was the
second half of the fix and the more important one: pinned the other way round, Drape's
scallops landed on whichever side the light was not, and a shape in the dark has
nothing to show — the control measured as working while looking broken. Anchor it Top
now and the top edge is the one that scallops, wherever the light happens to be.

Verified: Drape lands on the named edge **4 out of 4** and does not move when the
light does. Swell manages 3 of 4. **Wave does not follow it** and is not meant to —
it has no single shaped edge, it is a band that sways on both sides at once, so
Field direction changes its proportions rather than picking an edge. Orb ignores it
too, being a radial mass.

## Warp: four controls instead of one

Organic did three jobs at once, which is why it was hard to read. It is now:

| control | what it does |
|---|---|
| **From** | which side the deformation enters from |
| **Amount** | how far the edge travels |
| **Detail** | how many waves fit around the shape |
| **Variety** | how irregular they are — at zero the harmonics sit in phase and the result is a regular, almost architectural undulation |

Three things give the warp its range. The wave count is drawn from a **range** rather
than fixed, so two seeds at the same Detail give four lobes and six rather than the
same five. The harmonics are no longer whole multiples of each other — at 1.9 and 3.3
times the base they never line up twice around the shape, so the outline stops
repeating itself. And a per-seed **skew** makes crests and troughs unequal, rounded
on one side and pinched on the other, which is what stops a warped edge reading as a
plain sine. Six seeds now give six clearly different warps.

`From` is keyed to **the edge's own outward normal**. Two earlier attempts were
wrong: keying it to position along the outline put the deformation on whichever side
the shape happened to start from, and keying it to direction from the centre failed
on a wide field, where the top corners point sideways and half the top edge was
attributed to left and right. The normal is (0,-1) all along the top however wide the
field is.

**Warp amplitude scales with blur.** It has to out-run the blur or it never reaches
the picture: at the old coefficient the default warp displaced about 25px against a
48px blur radius, so most of it was averaged away — which is why Amount felt weak and
From looked like it did nothing at all.

**Warp and Light are told about each other.** An edge facing the light shows every
millimetre it moves; an edge facing away is displacing dark into dark and shows
almost nothing — measured, the lit side registered 11.5 against the unlit side's 2.0,
so `From Bottom` on a top-lit gradient looked broken when it was working perfectly.
The unlit side is now given more travel to compensate. Across four light directions
the spread between the strongest and weakest warp direction fell from about 6x to
**1.9-2.3x**, so every direction reads whichever way the light runs.

**Shade is renamed Shadow**, under its own heading with Amount, so it reads as a
shadow source rather than as a mystery.


## Two fixes worth recording

**The depth layer was lightening the ground.** `shadowTone` derived its colour by
moving away from the ground's lightness — which on a dark palette meant *adding*
0.034 of it. The layer named shadow depth was lifting the deepest part of the
picture. It now always goes down, keeping the small hue shift that stops the dark
reading as one flat colour.

**Margin opens at 9%, not 2%.** At 2% the blurred field bleeds to the frame edge and
the ground never reads: the border sat **36 levels above the darkest value** in the
picture, so the deepest tone ended up inside the field rather than around it — the
darks pulled to the middle. At 9% that gap closes to about 9, which is where the
references sit (+1 to +18).

## Drape

Drape reached 21% of the field at most, with two identical swags — not enough to
change a composition, which is the whole point of the form. It now reaches **62%**,
the number of bays comes from the seed, and neither their widths nor their depths
match: one bay hangs to the floor while the next barely dips. Two matched arcs never
look like hanging cloth; uneven ones do.

## Hold

A soft form needs blur, and blur carries the field's light outward by roughly its own
radius, so raising Blur to get Drape or Dome soft enough pushed light to the frame and
the border went with it — the border-to-darkest gap climbed from +8.7 at 4% blur to
+27.0 at 22%. The geometry pays for the blur: each edge pulls in by the blur radius,
and the gap holds at +8.5 to +12.3 across the whole range. Nothing is drawn to keep
the border dark; the field stops far enough back that the ground can be seen.

An edge already past the frame is left alone, which is what the retired **Bleed**
control used to say. It said nothing Width, Height and Offset did not already say
better: a size over 100% runs the field off the frame and an offset runs it off one
side. Bleed was a third way to ask for the same thing.


## The ramp keeps the box the user asked for

Hold and Bleed move the shape's edges. If the gradient were mapped across the moved
box too, bleeding one edge would stretch the ramp and shift the light everywhere
else — measured, bleeding the top pulled the bottom of the lit mass up by **6.4% of
the frame**. The shape now uses the adjusted box and the ramp uses the original, so
the light stays where it was put and only the named edge changes. The same test now
reads **0.1%**, and holds at width 45%, 90% and 140%.

## The effect side is drawn at its own blur

A form's character lives on one edge — Drape's hem, Swell's belly, Dome's dome — and
at a single global blur that edge is exactly as crisp as the three edges facing the
frame, which is not how the references read. A directional blur would be the obvious
answer and is not available: asymmetric radii do not survive Figma.

So the field is drawn **twice** from the same outline and the same ramp, each pass
faded along the light axis so only one is present at either end. The soft pass carries
**2.6x the blur** and owns the effect side; the crisp pass keeps its old radius and
owns the other end. The frame-facing edges are unchanged.

Every form shapes the **far** edge — the dark end at the default Follow light — so
the soft pass always belongs to the dark end and there is no per-form exception.
Field direction is the one way to move the shaping to the light end.

That consistency needed Drape and Swell to reach deeper. Shaping the far edge means
shaping the dark end, and a shape there has nothing to show: measured, at their old
depths Drape's silhouette was **identical to a plain Field**. Both now climb far
enough that their scallops cut into the lit band, which is what Dome does and why Dome
always read.

Orb is excluded, as it has no shaped edge to speak of — its falloff is radial and
already carries its own softness.

Export grows from about 7 KB to 11-12 KB, since every field is two layers now.

## Warp drives each form, not a wobble over the top

Amount, Detail and Variety used to feed a generic outline wobble that sat on top of
whatever form was selected, so every field looked like the same noise applied to a
different shape and Amount read as "add randomness". They now name each form's own
parameters: Detail sets Drape's bay count and Dome's breadth, Variety decides how far
the seed may move those and how uneven they come out, Amount is the form's depth. At
Variety 0 the count is exactly what Detail asks for. The generic wobble remains as a
surface texture at a third of its old weight.

## The reverse-arch reference — what it actually is, and why the tool cannot make it

Five mechanisms were tried for this reference and all five were wrong: rim light,
Edge, a cut Dome, an edge light at the frame, and sinking the ramp below the ground.
The picture was finally measured properly rather than guessed at.

**A vertical ramp alone explains it.** Residual RMS 11.2 against a picture standard
deviation of 35.3, and fitting a dark elliptical mass barely improves that (11.2 to
10.3). There is no dome, no arch, no shadow shape. The residual is flat across the
whole middle and spikes only in the outermost columns.

**What those columns are:** left +32.0 and right +30.4 against the picture just
inside, top +10.5, bottom -0.1. In the lower half that margin is rgb(60,30,25), a
warm brown, while the middle is rgb(1.6,1.1,1.2). It is not a light. It is the
**backdrop showing behind an inset field**, and the field's ramp goes darker than
the backdrop does.

**Why the tool cannot currently do it.** Read as our architecture, the reference's
backdrop is #41221B at luminance 40 and its field ramps down to luminance 0. In our
palettes the ground is *both* the backdrop and the ramp's dark end, and it is the
darkest colour in the palette — luminance 15 for the Dining Room. A backdrop darker
than the field's darkest point can never show as a margin, whatever is drawn over it.

**The fix is structural.** The three palette stops would have to mean something
different: the backdrop becomes a lifted ground, and the ramp ends near black
independently of it. A `depth` control that does this is written and withheld — it
moved the key measurement from -6.5 to -3.8 where the reference is +34.8, which is
the right dial at a fraction of the range it needs. Opening that range means changing
what every palette means and re-rendering everything the tool has produced, so it is
not a change to slip in quietly.


## Orb

The old Orb was a mass larger than the frame with a slow falloff — measured against
the reference it sat at **115 where the reference was at 40**, three quarters of the
way out, so the whole picture read as lit and nothing was left for the dark to do.

The reference profile is a dense core, a wide coloured halo, and a fall to near-black
inside the frame: luminance 154, 140, 112, 73, 40, 25 at increasing radius — with
chroma *rising* outward, 43% to 54%, because the core is close to white and the halo
is where the colour lives.

Orb is built to that now: a smaller radius, the core held flat before it drops, the
middle stop given the widest band of the three, and the ground reached before the
corner. Mean error against the reference profile fell from **49.7 to 21.2**, and on
Dining Room — whose ground sits closest to the reference's — it is **9.6**.

The residual is the palette, not the shape: each ground sets the floor the halo falls
to, so Gold Bar bottoms out at 67 and Rosa at 24 where the reference is 25. That is
correct behaviour rather than an error to tune away.

**The core is not the issue if Orb reads flat.** Measured against the reference's
core of 155.8, ours runs 156 to 172 depending on palette — at or above it. What
differs is the surround: the reference falls to 47.8 while Gold Bar holds 96.6, so
the reference reads 3.26x core-to-surround against our 1.78x. Perceived brightness is
contrast, not level. Pulling the surround down — a smaller Width and Height, a darker
palette, or Shadows in Finish — makes the core read brighter than raising it does.

## Calibration

Defaults open where the references sit rather than where the dark rounds left the
full tool — margin 2%, middle point 48%, edge light 60%.

| palette | Lite mean / hue | reference mean / hue |
|---|---|---|
| Dining Room | 91 / 55 | 95 / 44 |
| Gold Bar | 119 / 38 | 140 / 36 |
| Member's Bar | 100 / 67 | 88 / 51 |
| Rosa Lounge | 66 / 29 | 68 / 24 |

Gold Bar still runs about 20 levels under its reference; Exposure covers the gap.

## Unchanged from the full tool

Export behaviour, the Figma constraints in `NOTES.md`, undo and redo, Randomize,
image sampling, and the grain engine. Exports are 14 KB across all five frame sizes
with no transforms.

## What was tried and removed

Four things were built, measured, shown, and taken back out. They are recorded here
because the reasons are worth keeping, not because anything should be rebuilt:

- **Rim light** (four sliders and a side selector) — a band drawn along the frame.
  Always read as imposed, because it was.
- **Edge** — the same intent as a soft radial in the palette's own colours. Better
  behaved, still not wanted.
- **Bloom** and **Shadow** — extra light and extra dark laid over the field.

The pattern across all four: every one of them added something *on top of* the
gradient. What the references do is all in the gradient itself — the field's shape,
where the light falls across it, and how the ramp is tuned. The tool is 19 sliders
now and the useful surface is larger than it was at 25.
