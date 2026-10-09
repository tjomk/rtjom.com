---
title: "Local Llms Inference"
date: 2026-10-09T12:58:00+03:00
---
<center><img src="/images/local-llms-inference.jpg" alt="What is inference?" /></center>

In the [previous post](/blog/2026/09/local-llms-naming/) we discussed the different namings a single model can have, and how to navigate
between those. Now that you can understand what it means, it's time to run the model, and that is what inference is all about. The moment
you understand the amount of resources needed to run a relatively small model locally, you end up wondering what it takes to serve
frontier models to millions of people.

## What is inference

Every time you look up an open model that you can run on your hardware you hear about inference. Inference is just running a trained model
in order to make predictions on new data. A simple analogy: you learn basic arithmetic at school, say 2 + 2, and inference is applying that
knowledge at the store to work out how much you owe. But what actually happens when you "run" the model?

It starts by loading the weights into memory. The moment you start loading these you see your RAM filling up. The more memory you have, the
bigger the model you can load. Next step would be a user sending a request in the form of a text, and image, or any other way, and this input
gets tokenized. Tokenization splits the input into small pieces called tokens (usually parts of words) and converts each into a number the
model can work with.

Now that the input is converted to a suitable format, it is sent through the neural network in what is called a forward pass. The input flows
through every layer of the model, and the result is a probability for each possible next token. One token is picked from that distribution
(this is called sampling), and that is the first token of the answer. It is then appended to the input, and the whole process repeats,
producing one new token per pass until the model emits a special end-of-sequence token or hits a length limit. For a text model, these tokens
are then converted back into text.

This is, of course, very simplified. The prompt itself is processed in parallel in one go (prefill), and the model caches intermediate
results of earlier tokens (the KV cache) so it doesn’t recompute them on every pass.

## Why memory bandwidth matters more than compute

For local inference memory speed (and size) are often more important than the compute power of your GPU. Why is that?

Every time the model generates an output token, it touches every weight involved in that forward pass. Weight is read from the memory,
multiplied against the activations, and the process continues to the next layer or token. Every time this happens, weights get read and
then discarded right away. The math here is simple: just multiplication and addition. As a result most of the time we're waiting for data
to be fetched from memory, then perform simple arithmetic, and continue until the final result is generated. This means that the speed
with which tokens are generated, measured in tokens per second, is directly capped by how fast you can move weigths from memory to compute,
and not by how many (terra)FLOPs your GPU can do.

Here's a simple formula you can use to calculate the approximate speed:

> max tokens/sec ≈ memory bandwidth / bytes read per token

### A real-world example

I've been recently running Gemma 4 26B-A4B model with 4-bit quantization and got 40-42 tok/s on my MacBook with an M3 Pro chip and 36GB memory.
Apple says that my memory bandwidth is at around 150GB/s.

As the [previous post](/blog/2026/09/local-llms-naming/) explained, a mixture of experts (MoE) model only activates a part of its parameters
for each token, while a dense model uses all of them. This offers a considerable improvement in speed on the same hardware. Using the same
math as above: 26B x 0.5 byte = 13GB per token pass. We then use memory bandwidth 13GB ÷ 150GB/s ≈ 86.7ms/token which translates into 11.5
tokens/sec as the theoretical maximum that is almost 6.5x slower.

Let's also calculate this for a dense model, Qwen3.8-27B-4bit which averaged 6 tok/s: 27B params x 0.5 byte = 13.5GB per token pass.
We again use memory bandwidth to get the theoretical token generation speed: 13.5GB / 150GB/s ≈ 90 ms/tok which translates to 11 tok/s.
The real speed is about 2x slower.

Comparing the two runs: Qwen reads about 6.75x more bytes per token than Gemma, and it generates about 6-7x slower. Bytes read per token
predicts the relative speed well, even if the absolute ceiling is off by about 2x.

### Where does compute come into play

So why do people (and companies) buy up ever beefier GPUs? What kind of processes require more compute? It starts to matter in a few cases.
The first is serving many requests in parallel: the weights read from memory once can be reused for many users, so the arithmetic oprations
grow while the bytes moved stay roughly the same, and the bottleneck shifts from memory to compute. The second is processing long prompts:
all input tokens go through the model at once, which is compute-heavy, so a big prompt means a longer wait for the first token even on a
single-user machine. You notice this with coding agents, which send large prompts. Training also requires heavy compute, but that is a
different topic.

## Runtimes

To run a model locally, you need a runtime. And not every runtime can run any model you like. Not every model works on your hardware even though it
might fit into the memory. So what to choose?

When I first started playing with local models, [Ollama](https://ollama.com) was the first user-friendly option I came across. It’s a real plug
and play app that you download, install, select a model, and just run it. I haven’t used it since then, but the project is actively maintained,
and new features appear regularly. It started as a wrapper around `llama.cpp`, which we will discuss in a minute, and in 2026 it also added support
for Apple's MLX on Apple Silicon.

Recently several more MacOS-native (Swift) apps have appeared and gained popularity. I have been using [Nativ](https://github.com/Blaizzy/nativ)
which is running [mlx_vlm](https://github.com/Blaizzy/mlx-vlm) under the hood.

Both Ollama and Nativ are user-friendly options that is easy to start with. You just download the app, launch it, and begin to chat. Or point your
agent harness at its server endpoint, and launch tasks. But they do not always allow you to tune the parameters to your own needs. You might
also face problems running latest models: it might not start, crash, or just keeps timing out, and then you need to wait for an update if it ever
arrives.
As I mentioned, both Ollama and Nativ are user-friendly layers on top of lower-level runtimes (`llama.cpp` and `MLX` for Ollama, `mlx_vlm` for Nativ),
which you can also download and run directly.

I feel like `llama.cpp` and `vLLM` are the most popular runtimes, and they are updated daily. If you want to live on the bleeding edge and run models
that have just been released, you most-likely want to run either of these. They can also be configured to the extreme with tens or hundreds of different
flags in order to suit any hardware or LLM architecture. For my personal needs I use `mlx_vlm` which supports both text and vision. For text-only
models you can use `mlx_lm`. And I have [pi](https://pi.dev) as an agent harness.

### Running Qwen3.8 locally

When Qwen3.8 first came out, I tried to run it with Nativ. The model kept crashing and reloading until I figured out that it was not yet supported.
As I mentioned, Nativ uses `mlx_vlm` under the hood, and it has already been updated to run Qwen:

```bash
pip install -U mlx-vlm

APC_ENABLED=1 python -m mlx_vlm.server --host 127.0.0.1 --port 9080 \
  --model mlx-community/Qwen3.8-27B-4bit \
  --enable-thinking --thinking-budget 8192 \
  --log-level INFO
```

This will download the model from Hugging Face, store it in `~/.cache/huggingface`, and start an OpenAI-compatible web server serving the model.
You can then point harness at it. In Pi you need to add the new local provider to `models.json` so you can then select it via `/model`:
```json
{
  "providers" : {
    "local" : {
      "api" : "openai-completions",
      "apiKey" : "nativ",
      "baseUrl" : "http://127.0.0.1:9080/v1",
      "compat" : {
        "supportsDeveloperRole" : false,
        "supportsReasoningEffort" : false,
        "supportsUsageInStreaming" : true
      },
      "models" : [
        {
          "contextWindow": 65536,
          "maxTokens": 16384,
          "cost" : {
            "cacheRead" : 0,
            "cacheWrite" : 0,
            "input" : 0,
            "output" : 0
          },
          "id" : "mlx-community/Qwen3.8-27B-4bit",
          "input" : [
            "text"
          ],
          "name" : "Qwen3.8-27B-4bit",
          "reasoning" : true
        }
      ]
    }
  }
}
```

With these changes, you can select Qwen from the `/model` menu, and start coding. Or any other task you want.

## Will this run on my machine

Models that couldn't run on your machine yesterday with new advancements could tomorrow be easily run. Going from full precision to 4-bit quantization
may lose some quality in the output, but can also considerably shrink the model size to fit into memory. So how do you go from seeing `26B-A4B-4bit` to
knowing that you could use it?

The simplest way is to rely on the runtime GUI, both Ollama and Nativ (and others that I've tried) allow you to filter models that would fit your memory
and disk budget. Given the most likely source for searching and fetching these models is the HuggingFace, you can also create an account there and input
your hardware specs. Then in the search window you can just add your device as a filter option, and you will see only those models that you can realistically
run locally. This is what I am using most of the time.

I then search for models built by the `mlx-community`. These are meant to be run on the Apple hardware. I then check the model size on disk, and leave
some headroom: the context (KV cache) takes memory too and grows with context length, and OS lets the GPU use only a part of unified memory so that
other apps can continue working too. Both Qwen3.8 and Gemma 4 that I used are less than 20GB, so I can expect that these would fit nicely both on the
disk and in memory.

Finally you want to know whether the particular LLM architecture is supported by your runtime. If the model has just been released, most-likely it will
be supported by `llama.cpp` and `vLLM` in the coming days. And it might take a while for this support to land into the GUI apps like Ollama and Nativ.

## Conclusion

When I first tried local models I was confused as to why one would need them given that OpenAI and Claude are light years ahead in terms of the speed and
quality. Things have changed a lot since then. I still cannot run an open frontier model like GLM, DeepSeek, or Kimi, but with Qwen I can already do
quite a lot. What turned the needle for me was the release of [Shieldstral](https://mistral.ai/news/shieldstral/) by the French startup Mistral.

Shieldstral is a 3B model built for moderation tasks. The moment I heard about it, I immediately knew where it could be applied: AML. And I built a simple
prototype product around it which is now working in production. Since then something clicked in my head, and I started seeing more and more use-cases
for tiny, small, and medium language models everywhere. For many things you don't need a large language model. For many use-cases you also want privacy
and reliability of local inference.
