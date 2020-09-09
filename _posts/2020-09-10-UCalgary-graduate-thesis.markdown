---
layout: paper
title: "MSc thesis: Black-box Behavioral Model Inference for Autopilot Software Systems"
category: publication
permalink: /publications/:title:output_ext
tag:
- publication
- deep learning
- software engineering
- thesis
date: 2020-09-10 15:00
doi: null
arxiv: null
poster: null
venue: University of Calgary Vault
---

## Abstract
Inferring behavior model of a running software system is quite useful for several automated software engineering tasks, such as program comprehension, anomaly detection, and testing. Most existing dynamic model inference techniques are white-box, i.e. they require source code to be instrumented to get run-time traces. However, in many systems, instrumenting the entire source code is not possible (e.g., when using black-box third-party libraries) or might be very costly. 
Unfortunately, most black-box techniques that detect states over time are either univariate, or make assumptions on the data distribution, or have limited power for learning over a long period of past behavior. 
To overcome the above issues, in this thesis, I proposed a hybrid deep neural network that accepts as input a set of time series, one per input/output signal of the system, and applies a set of convolutional and recurrent layers to learn the non-linear correlations between signals and the patterns, over time. 
I have applied this approach on two real UAV autopilot case studies: one from our industry partner, MicroPilot (MP in short), with half a million lines of C code, and one widely used open source solution: Paparazzi. 
I ran more than 1200 system-level tests in total (to generate the input data) and inferred the system's internal state, over time.
In case of Paparazzi, as it did not include system tests like MP, I created a tool that generates and executes meaningful test scenarios.
Comparison with several traditional time series change point detection techniques showed that this approach improves their performance by up to 102% in MP's case and 94% in Paparazzi's, in terms of finding state change points, measured by F1 score. I also showed that this state classification algorithm provides on average 90.45% F1 score for MP and 82.23% for Paparazzi, which improves traditional classification algorithms by up to 17% in MP's case and 20% in Paparazzi's.

In addition, by creating a hyper-parameter tuning pipeline using grid search technique, despite having a way smaller training set in the second case study (7 times smaller compared to the first one), I managed to get a better performance, up to 48% better, out of the neural network model as measured by 8 metrics.
The tuning performance is compared to using the same hyper-parameters that worked for MP's case, for Paparazzi.

## Related Projects
### [Hybrid-net](/projects/#inferring-state-models-from-black-box-software-using-hybrid-deep-neural-networks)
The main repository containing the whole learning pipeline. The code is in Python using Keras on Tensor Flow.
[repository](https://github.com/sea-lab/hybrid-net)

### [Paparzzi Tester](/projects/#paparzzi-tester)
:airplane:️ A fuzz testing tool for generating and performing system tests for Paparazzi auto pilot
[repository](https://github.com/MJafarMashhadi/pprz_tester)

### Paparazzi
I contributed code to Paparazzi autopilot: a free and open-source hardware and software project for unmanned (air) vehicles.
[repository](https://github.com/MJafarMashhadi/paparazzi)

### pprzlink
I contributed code and documentation to pprzlink: Message and communication library for the Paparazzi UAV system
[repository](https://github.com/MJafarMashhadi/pprzlink)

## Presentation Slides

<iframe src="https://www.slideshare.net/slideshow/embed_code/key/KYetgcTa9ummEU" width="427" height="356" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="https://www.slideshare.net/secret/KYetgcTa9ummEU" title="Black-box Behavioral Model Inference for Autopilot Software Systems" target="_blank">Black-box Behavioral Model Inference for Autopilot Software Systems</a> </strong> from <strong><a href="https://www.slideshare.net/MohammadJafarMashhad" target="_blank">Mohammad Jafar Mashhadi</a></strong> </div>
