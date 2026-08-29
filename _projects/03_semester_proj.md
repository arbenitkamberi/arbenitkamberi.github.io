---
layout: page
title: "Optimizing GenMC's consistency checker"
description: Making the generated consistency checker at the core of the GenMC stateless model checker faster.
img: assets/img/ETH.jpg
importance: 1
category: work
related_publications: false
---

A project at the Department of Computer Science, ETH Zürich, again advised by
Prof. Dr. Michalis Kokologiannakis, following on from my
[Bachelor's thesis]({{ '/projects/' | relative_url }}).

[GenMC](https://github.com/MPI-SWS/genmc) is a stateless model checker for
concurrent programs that is parametric in the weak memory model it assumes. Every
time it extends an execution graph with a new event, it has to check that the
graph is still consistent under the chosen memory model, so the performance of
that check has a large effect on the performance of the tool as a whole. The
checker is not written by hand: [Kater](https://github.com/MPI-SWS/kater) takes a
memory-model specification and generates the C++ code that GenMC then uses to
decide consistency at runtime. The goal of this work is to improve the performance of that generated code.

My contributions target both the acyclicity checks and the inclusion (view)
checks that the generated checker is built from. On the acyclicity side this
includes reworking how the graph traversal is carried out so that it also
handles cases that previously pushed the checker past its limits, reducing the
bookkeeping the traversal has to redo on every check, and recognising when
an expensive check can be replaced by a much cheaper argument. On the inclusion
side it includes better handling of recursively defined relations and automatic sound transformations to reduce the required compution. These changes have translated into a noticeable speed-up for GenMC.

Although the work has been finished already, a full write-up of the project, with the details of each optimization and the
accompanying evaluation, is currently in progress. 
I will expand this page with more specifics once it is complete.
