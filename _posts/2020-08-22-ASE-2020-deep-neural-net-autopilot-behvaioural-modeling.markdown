---
layout: paper
title: "Hybrid Deep Neural Networks to Infer State Models of Black-Box Systems"
category: publication
permalink: /publications/:title:output_ext
tag:
- publication
- deep learning
- software engineering

date: 2020-08-22 17:10
doi: 10.1145/3324884.3416559
arxiv: 2008.11856
poster: null
venue: ASE 2020 - 35th IEEE/ACM International Conference on Automated Software Engineering
---

## Abstract
Inferring behavior model of a running software system is quite useful for several automated software engineering tasks
such as program comprehension, anomaly detection, and testing. Most existing dynamic model inference techniques are
white-box, i.e., they require source code to be instrumented to get run-time traces. However, in many systems,
instrumenting the entire source code is not possible (e.g. when using black-box third-party libraries) or might be very
costly. Unfortunately, most black-box techniques that detect states over time are either univariate, or make assumptions
on the data distribution, or have limited power for learning over a long period of past behavior. To overcome the above
issues, in this paper, we propose a hybrid deep neural network that accepts as input a set of time series, one per
input/output signal of the system, and applies a set of convolutional and recurrent layers to learn the non-linear
correlations between signals and the patterns, over time. We have applied our approach on a real UAV auto-pilot solution
from our industry partner with half a million lines of C code. We ran 888 random recent system-level test cases and
inferred states, over time. Our comparison with several traditional time series change point detection techniques showed
that our approach improves their performance by up to 102%, in terms of finding state change points, measured by F1 score.
We also showed that our state classification algorithm provides on average 90.45% F1 score, which improves traditional
classification algorithms by up to 17%.


## Presentation Slides
<iframe src="//www.slideshare.net/slideshow/embed_code/key/8ndk4DlWsgRUC0" width="595" height="485" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe>
