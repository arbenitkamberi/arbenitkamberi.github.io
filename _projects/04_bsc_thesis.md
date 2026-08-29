---
layout: page
title: "BSc thesis: A Rust Frontend for GenMC"
description: Extending the GenMC stateless model checker to verify concurrent Rust programs.
img: assets/img/ETH.jpg
importance: 1
category: work
related_publications: true
---

My Bachelor thesis at the Department of Computer Science, ETH Zürich, advised by
Prof. Dr. Michalis Kokologiannakis.

Stateless model checkers are widely used for verifying concurrent programs, but
their application to Rust has thus far been limited. This work extends
[GenMC](https://github.com/MPI-SWS/genmc), a state-of-the-art stateless model checker for concurrent C/C++ programs under weak memory models, so that it
can also verify Rust programs.

Because GenMC operates on the LLVM-IR produced during compilation, the frontend
adds support for LLVM-IR constructs emitted by the Rust compiler and provides
custom implementations of Rust's standard-library concurrency APIs so they can be
intercepted by the checker. Rust tests were run and cross-checked against the
equivalent C programs, and as a case study several concurrent data structures
from [Crossbeam](https://docs.rs/crossbeam/latest/crossbeam/), a popular open-source Rust crate, were verified.

The full write-up, including the approaches used to bridge the gap between the
Rust and C/C++ toolchains, is available in the thesis {% cite kamberi2025rustgenmc %}.
