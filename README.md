# mc-one-hop 12b

LoRA letter readout on frozen `google/gemma-4-12B-it` revision `707f0a3b8a3c7ad586ed01e27eafbad8a27dd0f7`. The adapter is rank 8, alpha 16, on language blocks 46 and 47 only. It is not merged into the base.

This repository is the weight file for a Benchmark Heaven evaluation. It is a different model from the frozen no-LoRA entry.

## Load

Use Transformers 5.15.1 or newer, `AutoModelForCausalLM`, and PEFT. Load the base at the revision above, then load this directory as a PEFT adapter. One 48 GB GPU in bf16 is enough. vLLM 0.27.1 cannot load `gemma4_unified`.

## Readout

One forward pass. Sum every vocabulary token that decodes to the same option letter, renormalize over the options, then raise each probability to `1/T`.

Temperatures, fit on a self-owned holdout and not on JevBench items:

- choice 2.90
- noul 3.10
- score 3.10

The system text is: "You are a calibration engine. You never answer in prose. You are given a state, a question and a numbered set of options, and you choose exactly one option. You reply with that option's LETTER and nothing else — a single character, no words, no punctuation, no explanation."

The user text is the state, a blank line, the question, a blank line, `Options:`, then lines `A. ...`, and a final line `Answer with the letter of exactly one option, and nothing else:`.

## Wire

TypeSafe `POST /v1/systemone` with `{"questions":{"decision": ...}}`. No API key. Cost basis is a self-hosted GPU, the same method Cygnet uses.

A developer run of the public 231 items scored 200/231, with hard 82/111 and hard ECE 0.0634. That run is not a rank.
