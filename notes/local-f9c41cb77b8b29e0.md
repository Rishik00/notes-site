---
title: "2607.01849v1"
authors: ""
arxiv: "local-f9c41cb77b8b29e0"
status: published
---

# Making models better at STRUDL

## One-line takeaway

Making models better at writing strudl code given a midi program. Could've been a 10 page paper but appreciate the detail here.&#x20;

## Problem

Strudl is a domain specific language used for music synthesizing. Its easy to use but is very low resource because not many people use it. Naturally, LLMs are bad at it. Approach proposed here to improve strdl generation using MIDI as a source language.&#x20;

## Core idea

Simple, 2 stage pipeline for SFT to give good general knowledge of MIDI and strudl and RL to improve the model's generations capability. And introducing a new synthetically geenrated dataset called strudl-synth.&#x20;

## Method

### Dataset pipeline

1. Generating strudl programs under a fixed task prompt using opus. The resulting programs were checked using AST and ReGex for cleanliness.&#x20;
2. The programs were then rendered using a headless strudl runtime.&#x20;

The generation pipeline was seeded with "music seeds":&#x20;

1. EveryNoise: a finegrained vocab for spotify listening data.&#x20;
2. MidiCaps from MIDI datasets
3. TheoryTab, a collection of popular song analyses.&#x20;



### Training steps

1. SFT on strudl synth and MIDI code to give the models general understanding of the conversion.&#x20;
2. During RL stage, unpaired MIDI (or just MIDI) is given to the model and is asked to produce the corresponding strudl code. It's rewarded for compiling correctly and correctness. algorithm used is GRPO.&#x20;

Justification is also fair and simple: because they're dealing with 2 DSL like structures the intial rollouts from direct RL will be very bad. So they optimised using SFT first and then RL'ed it to death.&#x20;

Rewards were designed for the following aspects:&#x20;

1. Faithfulness is measured by onset F1 between the reproduction and the original sample. A predicted sample is corrected only if (1) matches the pitch of a reference note (2) has an onset within 50ms and (3) is assigned the same general MIDI program number.&#x20;
2. Readability via an LLM judge witha  rubric.&#x20;

Metrics are onset F1, Frame F1, multi instrument onset F1. Diversity is measured by CodeBLEU self-similarity across compiled outputs.&#x20;

They trained 4B (mainly Qwen 3 4B) and 8B (Qwen 3 8B) models.&#x20;

## Results

* For baselines, they ask opus 4.6, GPT 5.5, Gemini 3.5 flash to write strudl programs using a fixed prompt. Turns out the trained models beat the above baselines by a good margin across readability and faithfulness.
* &#x20;SFT helps for a task like this! further ablations proved that RL without SFT was basically less than half the original scores when RL was performed before SFT.&#x20;

## Limitations

* It'd be interesting to see how current gen models would perform to this.&#x20;
* The dataset distribution between multi instrument songs might be too skewed.&#x20;
* Only 2 models tested from the same family.&#x20;

## Useful quotes

Nil

## Follow-up questions

1. Can we specialize this with one instrument rather than songs?&#x20;
2. Is there a better reward design? If we remove readablility does it affect the overall compilability of the program?&#x20;
3. Can we test it on T5 class models?&#x20;