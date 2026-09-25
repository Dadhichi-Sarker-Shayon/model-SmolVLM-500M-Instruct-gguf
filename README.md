---
base_model: HuggingFaceTB/SmolVLM-500M-Instruct
tags:
- gguf
- llama.cpp
- ollama
- quantization
- smolvlm
- q4_k_m
- q8_0
- text-generation
- text-tower
- 500m
- wikitext-2
license: apache-2.0
pipeline_tag: text-generation
---

# SmolVLM-500M-Instruct Q4_K_M GGUF — verified edge build (text tower)

<div align="center">

<img alt="Model" src="https://img.shields.io/badge/model-SmolVLM--500M--Instruct-8A2BE2?style=for-the-badge">
<img alt="Published formats" src="https://img.shields.io/badge/GGUF-Q8_0%20%7C%20Q4_K_M-FFD21E?style=for-the-badge">
<img alt="Scope" src="https://img.shields.io/badge/scope-TEXT%20TOWER%20ONLY-E8590C?style=for-the-badge">
<img alt="Vision" src="https://img.shields.io/badge/vision-NOT%20INCLUDED-8B0000?style=for-the-badge">
<img alt="Size" src="https://img.shields.io/badge/Q4%20size-303MB-0069B4?style=for-the-badge">
<img alt="License" src="https://img.shields.io/badge/license-Apache--2.0-7C3AED?style=for-the-badge">

</div>

Base: `HuggingFaceTB/SmolVLM-500M-Instruct` (Apache-2.0) · Quant: llama.cpp `0.4.1-dev` · imatrix on 3000-line wikitext-2 (seed 42, 625 chunks, 4 threads)

⚠️ **Scope correction:** this repository holds the **text tower only** (291 tensors, llama arch — verified: image input is rejected). The SigLIP vision encoder and `mmproj` are NOT included, so this quant cannot see images. The PPL numbers below are valid for the text tower. 

## Format status

| File | Status | Note |
|---|---|---|
| `smol-Q4_K_M.gguf` | Published | Main release, ~303 MB |
| `smol-Q8_0.gguf` | Published | ~0.4 GB |
| F16 | Reference only | 0.8 GB source-side reference used for the PPL baseline; not published |
| `mmproj` (vision projector) | Not available | Vision conversion failed validation; see the MM repository |

Note: SmolVLM's 960-dim layers don't divide evenly for Q4_K, so llama.cpp fell back parts to q5_0/q8_0 (5.89 bits/weight). Valid imatrix quant, disclosed.

## Eval (wikitext, 128 held-out lines, 2048 ctx)

| model | PPL | size |
|---|---|---|
| F16 | 13.5363 | 0.8 GB |
| Q8_0 | 13.5752 (+0.29%) | 0.4 GB |
| Q4_K_M | 13.7871 (**+1.85%**) | 0.3 GB |

Gate was `<5%` — passed, best of our three builds. Task evals not run: perplexity only, stated honestly. Files: `smol-Q4_K_M.gguf`, `smol-Q8_0.gguf`, `imatrix.dat`, `calib.txt`, `REPRO.json`, `SHA256.txt`.

## Verified text outputs

Verbatim `smol-Q4_K_M.gguf` completions, `--temp 0 -no-cnv`, 24 new tokens, prompt form `Question: ...\nAnswer:`. SmolVLM is instruction-tuned, so it restates the answer several times before drifting into a new question; repeats are trimmed.

| Question | Model answer |
|---|---|
| What is the capital of Japan? | `Tokyo` |
| What is the capital of Italy? | `The capital of Italy is Rome.` |
| What is the capital of Egypt? | `The capital of Egypt is Cairo.` |
| What is the largest ocean on Earth? | `Pacific Ocean` |
| Which planet is closest to the Sun? | `Mercury.` |
| How many days are in a leap year? | `366` |
| How many continents are there? | `7` |
| What is the chemical symbol for gold? | `Au` (then adds `Gold` and `The answer is: Gold`) |

Answers are correct but extremely terse, and the same answer is repeated two to four times. This is the 500M text tower of a VLM: it is a small text baseline, not a vision model and not a chat assistant.

## Use (text interface)

Ollama (see `Modelfile`):
```
ollama create smolvlm-500m-q4 -f Modelfile
ollama run smolvlm-500m-q4 "Summarize photosynthesis in 2 lines."
```

llama.cpp:
```
./llama-cli -m smol-Q4_K_M.gguf -p "Summarize photosynthesis in 2 lines." -c 2048
```

303MB: runs anywhere, fully offline. Do NOT pass images — no vision encoder in this file (see scope note).

## Limitations

- Text tower only: no vision encoder, no `mmproj`, image input is rejected by the runtime.
- Perplexity-only validation; verify on your task before production use.
- English wikitext calibration: other languages/domains may vary.
- Repeats its answer up to four times; add a stop sequence or use the instruct template if that matters.
- Vision quality of the upstream VLM is untested here and should not be inferred from this repository.
