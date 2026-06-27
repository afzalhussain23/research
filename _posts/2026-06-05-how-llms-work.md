---
layout: post
title: how LLMs actually work
date: 2026-06-05 08:00:00
description: a simple breakdown of inference, training, and finetuning
tags: llm ai
categories: notes
---

# Inference

I always wondered how Large Language Models (LLMs) actually work. When I ask a question in ChatGPT, how does it generate the answer?

At the core, LLMs are surprisingly simple. They consist of just two pieces:

- **Weights:** Think of it as a database with massive numbers that store some information.
- **A Program:** The code that tells the system how to use those weights to produce an outcome. This is what we call "Inference."

<div class="row mt-3 justify-content-center">
    <div class="col-8 col-md-6">
        {% include figure.liquid loading="eager" path="assets/img/how-llms-work-llm.svg" class="img-fluid" %}
    </div>
</div>

Obviously this explanation is oversimplified, since delivering a ChatGPT-scale service is a humongous task on its own. But let's keep it simple for now.

# Training

So how do we get the weights? That comes from a process called "Training," and it is much more involved. Essentially, it compresses internet data into something like a zip file, but not the kind we create on our computers.

- **Normal compression:** When we make a compressed file, we can extract it and get back the information exactly as it was.
- **LLM compression:** They can't deterministically reproduce the text they were trained on. Instead, it's a _lossy compression_, where the model is asked to predict the next word in a sequence.

By forcing the network to solve this puzzle billions of times, it effectively "zips" the world's knowledge into its weights.

<div class="row mt-3 justify-content-center">
    <div class="col-12 col-md-9">
        {% include figure.liquid loading="eager" path="assets/img/how-llms-work-training.svg" class="img-fluid" %}
    </div>
</div>

# Finetuning

But training alone isn't enough to produce a chatbot like ChatGPT. What we get after training is a "base model."

- **The problem:** It can predict the next word, but it doesn't behave like a helpful assistant.
- **The fix:** We align the model to act as an assistant by further training it on smaller, high-quality datasets of Q&A pairs along with human feedback.

Basically, it's a way of teaching the model how to behave as a conversational partner. I think this is the breakthrough that enabled ChatGPT back in November 2022.
