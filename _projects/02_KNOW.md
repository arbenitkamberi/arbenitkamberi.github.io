---
layout: page
title: "Computer vision for industrial defect detection"
description: Research internship at Know Center Research GmbH, Graz. Video analysis for detecting and classifying anomalous elongated structures in an industrial process.
img: assets/img/know-center-logo-rgb.png
importance: 1
category: work
related_publications: false
---

A research internship at
[Know Center Research GmbH](https://www.know-center.at/) in Graz, working on an
automated defect-detection pipeline for thermal-camera video of an industrial
process. The task is to detect thin, elongated structures ("strands") in the
scene and classify anomalies into two visually similar but geometrically
distinct failure modes: a strand that is **deformed** (bent along its length)
versus one that is **fragmented** (short and detached). Because the two look
alike, a single global threshold is error-prone and for any approach, 
tightening the rule for one class introduces mistakes on the other.

The perception layer is built on
[**SAM 3.1**](https://ai.meta.com/sam/) for promptable video segmentation,
running on **NVIDIA H100** GPUs. SAM 3.1 is deliberately tuned to
over-segment, meaning to propose more candidate defects than really exist, and the
geometric passes downstream decide what is real.

**From filter/rescue to independent passes.** I reworked the core methodology
from a rigid chain of filter and rescue operations into a set of independent
*passes*. Each pass measures its own geometric properties of a candidate
(thickness, curvature, orientation, convex hull) and emits its own local
verdict. The verdicts are only combined at the end, where the two anomaly
classes can be judged with separate criteria. Every stage keeps a full history
of each candidate's status and reason, which feeds both debugging and a visual
debug view.

**Amodal shape completion.** When two strands overlap, the segmentation model
returns a mask with two disconnected components for the same object. I added an
amodal filling step that estimates the end tangents of each fragment, joins them
with a smooth Hermite/Bézier spline, and fills the gap to the measured local
thickness. This is done only when the implied curve is gentle, so that occlusion is
reconstructed without inventing curvature. This gives the downstream length,
curvature, and orientation passes the whole structure instead of an arbitrary
fragment. Recent amodal-segmentation research is not yet usable in this case, but tracked as a future drop-in replacement once it targets SAM 3.1.

**More robust curvature estimation.** The curvature pass was rewritten in three
places:

- *Thinning.* The Zhang–Suen skeleton was prone to spur "hairs" and staircase
  artefacts. Adding pre- and post-processing steps from the Stentiford and Holt
  algorithms (*Algorithms for Image Processing and Computer Vision*, Ch. 6) removes them cleanly.
- *Turning-angle computation.* Rasterised skeletons zig-zag one pixel at a time,
  which inflated total turning angle enormously (a near-vertical line measured
  ~52°). Arc-length resampling followed by block averaging over a fixed physical
  window removes the rasterisation bias.
- *Temporal decision rule.* The pass now samples every fifth frame and only
  reports a deformation that persists across several consecutive samples,
  logging the frame range.

**Debug render grid.** A single `--debug` run produces a combined grid (and a
per-frame folder) showing, side by side, the calibrated thermal input with all
overlaid segmentation masks and object IDs, auxiliary masks for other scene
objects, and each pass's detections with its intermediate geometry: filled
masks, skeletons with interpolation points, convex hulls. This turned out to be
the most valuable tool of the project for understanding pipeline behaviour at a
glance. The terminal output was likewise extended to report the exact frames or
frame ranges each pass acted on.

**Practical tooling.** I added a CPU-offload option so the pipeline fits within
the VRAM limits of an older development GPU, and a single-frame extraction
utility for quickly reproducing and inspecting individual problem frames. A
small set of representative clips and frames was curated for regression-style
debugging.

**Outcome.** Each pass is now well hardened for the property it is meant to
measure, and the pass-based design makes exactly the kind of order-independent,
class-specific judgement this problem needs (deciding per anomaly class which
geometric criteria apply, rather than forcing one global rule to serve both) possible.
