# 🪨 RosettaBench

> **A contamination-free benchmark for measuring learning, not memorization.**

## 🔗 Project Links
 
| | |
|---|---|
| 📄 **Writeup** | [kaggle.com/competitions/kaggle-measuring-agi/writeups/rosettabench](https://kaggle.com/competitions/kaggle-measuring-agi/writeups/rosettabench) |
| 📊 **Benchmark** | [kaggle.com/benchmarks/namanbnsl/rosetta](https://kaggle.com/benchmarks/namanbnsl/rosetta) |
| 🗂️ **Dataset** | [kaggle.com/datasets/namanbnsl/rosettabench-150-stratified-compressed](https://kaggle.com/datasets/namanbnsl/rosettabench-150-stratified-compressed) |

## The Problem

LLMs are increasingly evaluated on coding and reasoning tasks, yet **existing benchmarks primarily measure *memorised* knowledge, not the ability to *learn*.**

Per Google DeepMind's cognitive framework, learning is the ability to acquire new knowledge or skills through experience, study, or instruction. Current evaluations cannot distinguish a model that truly learns from one that merely recalls.

> Without benchmarks that isolate learning from recall, progress toward AGI cannot be meaningfully measured.

---

## What is RosettaBench?

RosettaBench evaluates whether LLMs can **learn**, not just recall, by requiring them to solve problems in unseen synthetic programming languages. Performance drops — the *learning tax* — reveal that most models rely on memorisation, while only a few truly learn and adapt.

Each problem is presented in a **unique synthetic language**: a remapped variant of Python where all keywords, builtins, methods, operators, and delimiters are replaced with invented tokens.

```
# Python                          # Synthetic Language
def solve(n, items):         →    spimgleox solve(n, items)==>
    for i in range(n):       →        zloumfráit i kraum streix(n)=>
        if items[i] == "*":  →            vraith items[i] ~~"*"=>
            return i         →                shesstrim i
    return -1                →    shesstrim -1

result = []                       result = []
result.append(solve(n, a))   →    result.straumjeim(solve(n, a))
print(" ".join(result))      →    kroumspor(" ".skauk(result))
```

The model receives **6 few-shot example pairs** (synthetic ↔ Python), then must solve the problem entirely in the synthetic language. Every problem gets a unique language — making memorisation impossible.

---

## Tasks

### 🔴 RosettaBench Core
Solve AtCoder problems in a novel synthetic language using only the 6 few-shot examples. Tests in-context language acquisition.

### ⚪ RosettaBench Python Baseline (Control)
Same problems presented in standard Python, with no few-shot examples. Establishes a performance ceiling to compute the **learning tax**.

> **Learning Tax** = `baseline_score − core_score`
>
> A low learning tax means the model learns from context. A high one means it relies on memorisation.

---

## Dataset

**150 AtCoder problems** from LiveCodeBench (`release_v5`): 40 easy, 50 medium, 60 hard.
Only STDIN/STDOUT problems with ≥ 3 test cases were included.

| Column | Type | Description |
|---|---|---|
| `question_id` | string | AtCoder problem ID |
| `question_content` | string | Problem statement |
| `difficulty` | string | `easy` / `medium` / `hard` |
| `all_tests` | list[dict] | Public + private test cases |

*Dataset is fully reproducible via the attached notebook (`random_state=42`).*

---

## Technical Details

**1. Language generation** — A unique synthetic language is generated from a per-problem seed using a syllable-based RNG for structured, pronounceable words. The remapping covers all Python vocabulary (35 keywords, 77 builtins, 55 methods, 25 operators, 4 delimiters). No two problems share a language.

**2. Prompt construction** — The model receives 6 few-shot example pairs (synthetic and Python equivalent), followed by the problem and output instructions: respond with a `<solution>` block containing code in the synthetic language.

**3. Solution extraction** — Responses are first checked for `<solution>` tags. If absent, the last fenced code block is used instead.

**4. Validation & execution** — The extracted code is checked for Python token leaks (`PYTHON_LEAK`). If clean, the code is back-translated to Python and run with a 30-second timeout against all test cases.

**5. Scoring** — A problem is correct only if all test cases pass (`PASS`). Final score is `passed / total` as a float in `[0, 1]`. **No LLM judge is used.**

### Outcome Codes

| Code | Meaning |
|---|---|
| `PASS` | All test cases passed |
| `NO_CODE` | No extractable code in response |
| `PYTHON_LEAK` | Python tokens found in synthetic code |
| `SYNTAX_ERROR` | Back-translated Python failed to compile |
| `RUNTIME_ERROR` | Compiled but crashed at runtime |
| `WRONG_ANSWER` | Ran but produced incorrect output |

---

## Results

> Core scores across 16 models range from **88% to 0.7%**. High performance on the Python baseline does not imply a low learning tax. GPT-5.4 Nano scores 70% on the Python baseline, yet collapses on the synthetic task, scoring just **0.7%**.

### Overall Rankings

*Ranked by core score (descending). Learning Tax = baseline − core (lower = better in-context learning)*

| Rank | Model | Core | Baseline | Learning Tax | Core Cost | Baseline Cost |
|------|-------|------|----------|-------------|-----------|---------------|
| 1 | **Gemini 3.1 Pro Preview** | **88.0%** | 95.0% | **7.0%** | $21.26 | $13.05 |
| 2 | **Gemini 3 Flash Preview** | **80.7%** | 91.0% | **10.3%** | $9.20 | $8.11 |
| 3 | Claude Sonnet 4.6 | 44.7% | 85.0% | 40.3% | $4.20 | $3.50 |
| 4 | Claude Opus 4.6 | 40.7% | 91.0% | 50.3% | $6.14 | $4.31 |
| 5 | GPT-5.4 | 29.3% | 75.0% | 45.7% | $1.71 | $0.90 |
| 6 | Gemini 2.5 Flash | 26.0% | 67.0% | 41.0% | $3.96 | $4.35 |
| 7 | Claude Haiku 4.5 | 25.3% | 79.0% | 53.7% | $1.06 | $0.72 |
| 8 | Gemini 2.5 Pro | 24.0% | 78.0% | 54.0% | $17.58 | $23.79 |
| 9 | DeepSeek R1 | 15.3% | 77.0% | 61.7% | $11.64 | $10.57 |
| 10 | DeepSeek V3 | 15.3% | 74.0% | 58.7% | $0.31 | $0.28 |
| 11 | Qwen3 Coder 480B (A35B Instruct) | 13.3% | 69.0% | 55.7% | $0.17 | $0.20 |
| 12 | Gemini 3.1 Flash Lite Preview | 8.7% | 80.0% | 71.3% | $0.28 | $0.15 |
| 13 | Qwen3 235B (A22B Instruct) | 8.0% | 68.0% | 60.0% | $0.13 | $0.23 |
| 14 | GPT-5.4 Mini | 4.7% | 63.0% | 58.3% | $0.50 | $0.32 |
| 15 | Gemma 3 27B | 3.3% | 32.0% | 28.7% | $0.03 | $0.01 |
| 16 | GPT-5.4 Nano | 0.7% | 70.0% | 69.3% | $0.15 | $0.16 |

> 🏆 **Gemini 3.1 Pro Preview and Gemini 3 Flash Preview are the only models with learning tax under 15%** (7% and 10.3% respectively). Every other model loses at least 40 percentage points.

### Per-Difficulty Breakdown (Core)

| Model | Easy (40) | Medium (50) | Hard (60) |
|-------|-----------|-------------|-----------|
| Gemini 3.1 Pro Preview | 98% | 84% | 85% |
| Gemini 3 Flash Preview | 98% | 80% | 70% |
| Claude Sonnet 4.6 | 82% | 42% | 22% |
| Claude Opus 4.6 | 72% | 40% | 20% |
| GPT-5.4 | 60% | 32% | 7% |
| Gemini 2.5 Flash | 62% | 20% | 7% |
| Claude Haiku 4.5 | 57% | 18% | 10% |
| Gemini 2.5 Pro | 55% | 22% | 5% |
| DeepSeek R1 | 38% | 12% | 3% |
| DeepSeek V3 | 42% | 6% | 5% |
| Qwen3 Coder 480B | 40% | 6% | 2% |
| Gemini 3.1 Flash Lite Preview | 25% | 6% | 0% |
| Qwen3 235B | 22% | 4% | 2% |
| GPT-5.4 Mini | 18% | 0% | 0% |
| Gemma 3 27B | 12% | 0% | 0% |
| GPT-5.4 Nano | 2% | 0% | 0% |

---

## Key Insights

### Hard problems expose the real gap
11 of 16 models score under 10% on hard problems. Hard problems require language acquisition and algorithmic ability simultaneously, making this the most discriminating tier.

### Failure modes
`PYTHON_LEAK` is the dominant failure mode across 14 of 16 models. Most models abandon the synthetic language and fall back on Python tokens — the clearest signal of memorisation over learning. `SYNTAX_ERROR` dominates for Gemini 2.5 Pro (68/150) and Claude Haiku 4.5 (33/150): these models attempt the synthetic language but fail to internalize its structure. `RUNTIME_ERROR` and `WRONG_ANSWER` are more evenly distributed and reflect algorithmic failures rather than language-acquisition failures.

### Memorisers assume, Learners adapt
Problem `abc357_b` required uppercase conversion, but the few-shot example pairs only demonstrated `.lower()`. Every model except Gemini 3 Flash Preview and Gemini 3.1 Pro Preview hallucinated a `.upper()` token. The two exceptions used `.lower()` with a character map, staying within the demonstrated vocabulary.

> **The models didn't fail due to weak coding ability. They failed because they relied on memorisation instead of learning.**

---

## Conclusion

RosettaBench demonstrates that **strong existing benchmark performance does not imply the ability to learn**. Measuring this gap is essential for tracking real progress toward AGI.

---

## References

1. Jain, N., Han, K., Gu, A., Li, W., Yan, F., Zhang, T., Wang, S., Solar-Lezama, A., Sen, K., & Stoica, I. (2024). *LiveCodeBench: Holistic and Contamination Free Evaluation of Large Language Models for Code*. arXiv:2403.07974. https://arxiv.org/abs/2403.07974

2. Burnell, R., Kelly, O., et al. (2026). *Measuring Progress Toward AGI: A Cognitive Taxonomy*. Google DeepMind. https://storage.googleapis.com/deepmind-media/DeepMind/Blog/measuring-progress-toward-agi/measuring-progress-toward-agi-a-cognitive-framework.pdf

3. Paech, S. (2024). *livecodebench-code_generation_lite* [Dataset]. Hugging Face. https://huggingface.co/datasets/sam-paech/livecodebench_code_generation_lite

4. AtCoder. https://atcoder.jp

---
