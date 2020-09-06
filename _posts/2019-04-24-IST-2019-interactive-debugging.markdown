---
layout: paper
title: "Interactive semi-automated specification mining for debugging: An experience report"
category: publication
permalink: /publications/:title:output_ext
tag:
- publication
- software engineering

date: 2019-04-24 17:10
doi: 10.1016/j.infsof.2019.05.002
arxiv: 1905.02245
venue: "Information and Software Technology Volume 113, September 2019, Pages 20-38"
---

**Also presented at ASE 2019's [journal-first presentations track](https://2019.ase-conferences.org/details/ase-2019-Journal-First-Presentations/13/Interactive-semi-automated-specification-mining-for-debugging-An-experience-report).**

## Abstract
### Context
Specification mining techniques are typically used to extract the specification of a software in the absence of 
(up-to-date) specification documents. This is useful for program comprehension, testing, and anomaly detection. However, 
specification mining can also potentially be used for debugging, where a faulty behavior is abstracted to give developers 
a context about the bug and help them locating it.

### Objective
In this project, we investigate this idea in an industrial setting. We propose a very basic semi-automated specification 
mining approach for debugging and apply that on real reported issues from an AutoPilot software system from our industry 
partner, MicroPilot Inc. The objective is to assess the feasibility and usefulness of the approach in a real-world 
setting.

### Method
The approach is developed as a prototype tool, working on C code, which accept a set of relevant state fields and 
functions, per issue, and generates an extended finite state machine that represents the faulty behavior, abstracted 
with respect to the relevant context (the selected fields and functions).

### Results
We qualitatively evaluate the approach by a set of interviews (including observational studies) with the company’s 
developers on their real-world reported bugs. The results show that (a) our approach is feasible, (b) it can be
automated to some extent, and (c) brings advantages over only using their code-level debugging tools. We also compared 
this approach with traditional fully automated state-merging algorithms and reported several issues when applying those 
techniques on a real-world debugging context.

### Conclusion
The main conclusion of this study is that the idea of an “interactive” specification mining rather than a fully 
automated mining tool is NOT impractical and indeed is useful for the debugging use case.


