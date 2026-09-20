---
title: "Local LLMs: Naming"
date: 2026-09-20T10:46:18+03:00
draft: true
---
My first introduction to local large language models has been about a year ago when I tried to look into the coding models. Having used Claude Code
as my daily tool the quality of the local models was really bad. They were no good for anything. I failed to come up with a good enough use-case.
Some engineers suggested to use it for creating commit messages or pull request description. But why? When I wrote code myself, I could also write
PR description or commit message. And so I left this topic for some time.

About half a year later I heard a Podcast episode on small language models. Guests discussed tiny models for the edge devices, e.g.
a standing desk knowing when I need a stretch and raising the height, or lighting adapting to the light condition. The conversation went on to
discuss other topics like privacy and automation. An interesting topic was a discussion on the physical materials: a custom-trained model for
manufacturing, how different materials interact with each other under different conditions. This was the moment that I learned about the ways
these tools could be used for anything other than writing code.

The moment I finally started looking into local models is when Mistral released their Shieldstral model. It is a specialised model for content moderation.
"Shieldstral frames content moderation as a binary question-answering task". In simple words, the model receives input and responds with yes/no
and a score. For example, does this text promote violence? Does this image contain drugs? Finally I knew how this could be applied for my daily
work, and built a tool around this model. This work sent me on an interesting journey that I will describe in a series of posts here.

## The naming confusion

When you hear names like Shieldstral, Astra, DeepSeek V4 Flash, GLM 5.3, there is little to no confusion. Model name, model version, and that is it.
The moment you want to run a model locally, things start to get more complicated. Take Qwen 3.8 for example. There is `qwen3.8-27B`, `qwen3.8-27B-FP8`,
`qwen3.8-2.4T-A95B`, `qwen3.8-27B-MTPLX`, `qwen3.8-27B-GGUF`, `qwen3.8-27B-GSQ-RCO-GGUF`, or `gemma-4-26B-A4B-it-4bit`. The moment you are presented
with such list it feels very frightening, at least to me. What are all these abbreviations? What should I choose if I want to have the same exerience
as I do with Claude Code and a Haiku model? Turns out that each section encodes something important. We'll go through the filename convention from left
to right, the same way you would read it.

## The parameter count

Let's start with the most important part here, the parameter count. Usually this is the section that immideately follows the model name, e.g. `qwen3.8-2.4T`
or `qwen3.8-27B`. In the first one the model has 2.4 trillion parameters, and in the latter there are only 27 billion. Knowing number of parameters
and whether the model is dense or a mixture of experts, you can quickly learn how much memory it would need and whether you will be able to run it
comfortably on your hardware.

## Model architecture

Often times you will see something like `A3B`, a letter "A" followed by the number of parameters. For example, a Google's Gemma 4 model `gemma-4-26B-A4B`
has a total of 26 billion parameters, but only 4 billion of these are ever active at any point. This is a [mixture of experts](https://huggingface.co/blog/moe)
model which uses a sort of a router to activate only 4B parameters for each token. This brings the compute savings over a dense model where all 26B params
activate for each token.

A small detour from the naming into the architecture itself: for the MoE models given some not so trivial engineering it is possible to stream only the
"currently-needed" experts from disk or network which allows to use 1-2.8T parameter models on computers where the model cannot fit into the VRAM. Some
people managed to run Kimi K3 with 2.8T models on an macbook M1 MAX and stream it from four SSDs at about 15 tokens per minute. [source](https://github.com/gavamedia/deltafin)

## Quantization

When you hear that a model has been quantized, it means that its precision has been reduced. The precision is usually indicated by the `FP` prefix followed
by a number. An `FP8` means that model weights are stored as 8-bit floating point numbers. Quantization can vary from FP32 all the way to integer where you
lose any precision. FP is not the only option you can see. For integer precision you could see `Q4_K_M`, `INT8`, `Q8_0` which all are about integer precision.

Why is quantization useful at all? With the loss of precision you also get a smaller model size. By default each model weight uses 2 bytes, i.e. FP16. For
a 26B model at full precision the file size is ~50GB. The same model at 4-bit is ~15GB - something you can already run on a good laptop. Depending on the
use-case you might tolerate the loss of precision to a varying degree. At 4-bit quantization the loss is often small enough that you can barely notice it.

## Fine-tuning

Let's look at the different types of a Gemma 4 model: `A4B`, `A4B-it`, `A4B-it-assistant`. The first one is the so called base model. This is the raw
pretrained model trained to predict text. It cannot hold any conversation with you like ChatGPT. Why would you need such a model? You use it to further
fine-tune for your own needs. It can understand text, and you want it to spot patterns like a vulnerable network response.

The next one is the **instruction-tuned** model which is the same base model that has been further trained to follow instructions. You ask it a question,
and you get a meaningful response back. This is pretty much your ChatGPT model that you can hold conversations with.

Finally comes the **assistant**. Contrary to what you might guess from the name, this is not your butler aimed at helping you prepare the breakfast. It is
a small draft model meant for speculative decoding. Instead of generating tokens one at a time with the full 27B model, the tiny assistant model **predicts**
what the next handful of tokens a full model would likely generate. The job of the "big" model is to then verify whether the predicted tokens are correct or
not. When the prediction is correct, you can get up to 2X boost in the token generation speed.

## What is MLX?

I use Mac as my daily driver. And you can find the `MLX` abbreviation either in the model name itself or as a prefix on Hugging Face, e.g.
`mlx-community/Qwen3.8-27B-4bit`. MLX is a machine learning framework build by Apple for its chips. You generally want to run models from the
mlx community hub on your Apple hardware.

## A working example

Let's take [gemma-4-26B-A4B-it-4bit](https://huggingface.co/mlx-community/gemma-4-26b-a4b-it-4bit) which I used recently on my machine. This model
was converted to the MLX format by the MLX community so that it can run on the Apple hardware. The model contains 26B total parameters, and only 4B
of them are active at any point in time. This means that this is a mixture of experts model. It has been instruction-tuned so you can chat with it.
Finally, it has been quantized to 4 bit precision. The full model size is ~15GB which when used took 15.6GB of VRAM at its peak and generated
~40 tokens per second. At this speed and size I could comfortably use the model for my work and have enough memory left for other apps.

This should be enough information to decode most of the names. In the next post dive into how to run these models and what it costs.
