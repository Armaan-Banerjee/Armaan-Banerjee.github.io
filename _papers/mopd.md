---
title: "MOPD: Multi-Teacher On-Policy Distillation for Capability Integration in LLM Post-Training"
date: 2026-11-03
authors: "Wenhan Ma, Jianyu Wei, Liang Zhao, Hailin Zhang, Bangjun Xiao, Lei Li, Qibin Yang, Bofei Gao, Yudong Wang, Rang Li, Jinhao Dong, Zhifang Sui, Fuli Luo"
paper_url: "https://arxiv.org/pdf/2606.30406"
---

## Abstract

- The paper proposes training multiple domain-specialised teachers with reinforcement learning, then distilling those teachers into a single student model

## Introduction

- Different tasks benefit from different reinforcement-learning pipelines: mathematics can use reinforcement learning with verifiable rewards (RLVR), while search agents can learn through reinforcement learning in a web environment
- Multi-Teacher On-Policy Distillation (MOPD) applies specialised reinforcement learning in each domain to create a collection of strong teachers, then distils their capabilities into one student model
- The approach provides a strong, dense training signal, allowing the student to inherit most of each teacher's capabilities
- Training uses trajectories sampled entirely from the student's distribution, which eliminates exposure bias
- Capabilities are integrated by routing each prompt to the appropriate domain teacher rather than using weight-space fusion, which merges the parameters of multiple neural networks

## Method

- Stage 1: fine-tune a base model to produce a supervised fine-tuning (SFT) checkpoint that covers the model's general capabilities
- Stage 2: starting from the Stage 1 checkpoint, train an expert for each domain using the reinforcement-learning method best suited to that domain
- Stage 3: initialise the student from the Stage 1 checkpoint and sample a batch of prompts; the student generates a trajectory for each prompt and records its per-token probability distributions
- Route each trajectory to the teacher corresponding to the prompt's domain, then update the student by minimising the reverse KL divergence between the student and teacher distributions for that trajectory

![The three-stage MOPD pipeline: supervised fine-tuning, domain-specialised reinforcement learning, and on-policy distillation into a student through reverse KL divergence]({{ '/assets/papers/mopd/figure-1.png' | relative_url }})

- Top-k distillation can compare only the highest-probability portion of each distribution, simplifying the implementation

## Experiments

- MOPD achieves a higher normalised score than the comparison methods, reaching 0.937 across AIME, IFE, and SWE

## Analysis

- Initialising both the teachers and student from the same base model helps stabilise distillation
- When the teachers were instead initialised from a significantly stronger base model, the initial KL divergence was five times higher
- Despite using stronger teacher models, this configuration did not outperform teachers and students derived from the same base model on the evaluated benchmarks
