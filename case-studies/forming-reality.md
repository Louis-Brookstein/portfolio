# Concept generator for Forming Reality

**A brief-to-client-deck pipeline for a shopfittings manufacturer.**
Sketches and a phone photo go in; a dimensioned 3D spec, photoreal renders, a
bill of materials and a PowerPoint come out. Live in production since September
2026.

`TypeScript · Three.js · Claude · Gemini · Fastify · Vite · Docker · Caddy`

---

## The problem

Forming Reality design and build retail display units — the fixtures you walk
past in a chemist or a shoe shop. Their business development lead would come
back from a client meeting with a sketch, some dimensions scribbled on a
drawing, and a photo of the unit currently on the shop floor. Turning that into
something he could show the client meant queueing for a designer. Days, usually,
for work that often changed again after the next call.

The ask was narrow and I kept it narrow: **let one salesperson produce a
credible concept himself, in an afternoon, without learning CAD.**

## What it does

**1 · Brief → dimensioned spec.** Upload the sketches, drawings and photos, add
a few lines of context. Claude reads them and produces a structured spec —
every part, every dimension, every material — plus up to three questions about
what it couldn't determine. The questions matter: the alternative is a model
that quietly invents a 150mm base frame.

**2 · Spec → block model.** The spec drives a Three.js scene rendered headlessly
from eight fixed cameras. Grey blocks, no materials. This is the geometric
source of truth.

<img src="../images/forming-reality/01-block-model.png" width="700" alt="Grey block model of a three-tier retail display unit">

**3 · Block model → painted render.** Gemini paints the block render, keeping
its geometry and viewpoint, applying the materials and product from the spec.

<img src="../images/forming-reality/02-painted-render.jpg" width="700" alt="The same unit rendered photorealistically in brushed steel with product">

**4 · Placed in store.** The painted unit is composited into a described retail
environment.

<img src="../images/forming-reality/03-in-store.jpg" width="700" alt="The finished unit shown in a shoe shop interior">

**5 · Technical output.** Dimensioned elevations and plans as SVG, a bill of
materials with a weight estimate from per-material densities, and a client-ready
`.pptx`.

<img src="../images/forming-reality/04-elevation.png" width="520" alt="Dimensioned front elevation drawing">

Throughout, he edits by typing what he wants in plain English — *"the shelf
is 25 thick, not 100"*, *"Side B crates run front to back"*. An adjustment
router turns that into a patch against the spec, re-renders, and verifies the
result against the instruction.

---

*Built for Forming Reality. Source is private.*
