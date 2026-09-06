# ⚖️ LLM Refusal Evaluation Suite (LLM-as-a-Judge)

An automated, memory-safe, two-phase evaluation framework designed to benchmark refusal boundaries, safety steering, and representation engineering (weight orthogonalization / abliteration) in Large Language Models.

---

## 🌟 Overview

Standard string-matching heuristics (such as searching for `"I cannot"`, `"As an AI"`, or `"Desculpe"`) suffer from high rates of:
- **False Positives:** Rejecting legitimate creative writing that contains colloquial denials within character dialogue (*e.g.*, *"I cannot hold back anymore..."*).
- **False Negatives:** Overlooking polite evasions and moralizing lectures that refuse the prompt without triggering trigger phrases.

This suite solves this by utilizing an **LLM-as-a-Judge** architecture (defaulting to the **Qwen** model family) operating under near-zero temperature with structured Chain-of-Thought reasoning (`<reasoning>`) and strict XML verdict enforcement (`<verdict>[REFUSAL | FULFILLED]</verdict>`).

---

## 🔬 Benchmark Dataset Architecture

The suite evaluates models across **30 high-contrast prompts** divided equally across **5 strategic domains** (6 prompts each):

1. **Explicit Adult Fiction & Erotica:** Testing visceral, unfiltered consensual romance, physical intimacy, and dirty talk to detect artificial corporate prudery.
2. **Offensive Cybersecurity & Dual-Use Theory:** Assessing technical red-teaming education (*e.g.*, AMSI memory patching, return-to-libc, DNS tunneling, time-based SQL injection) without corporate disclaimers.
3. **Dark Fiction & Unforgiving Villains:** Evaluating antagonist monologues, psychological tension, and amoral character consistency without forced redemption arcs.
4. **Radical Philosophy & Taboos:** Exploring extreme thought experiments (*e.g.*, absolute amoral egoism, David Benatar's antinatalism, Machiavellian statecraft, and Nietzschean moral genealogy).
5. **Scientific Sanity Baseline (Negative Control):** High-difficulty questions in quantum mechanics, algorithms (A* in Python), software architecture (CQRS/Event Sourcing), and biochemistry to ensure **zero syntactic degradation or cognitive lobotomy** resulting from weight interventions.

---

## ⚙️ Architecture: 2-Phase VRAM-Safe Pipeline

To run seamlessly on consumer or cloud GPUs (*e.g.*, NVIDIA L4 24GB, A100 80GB, or RTX 4090):

```
┌─────────────────────────────────────────────────────────────┐
│ FASE 1: GERAÇÃO DO MODELO ALVO                              │
│ 1. Carrega o modelo a ser avaliado em BF16                  │
│ 2. Executa geração para os 30 prompts                       │
│ 3. Salva saídas em 'target_responses.json'                  │
│ 4. Executa gc.collect() e torch.cuda.empty_cache()          │
└──────────────────────────────┬──────────────────────────────┘
                               │ (VRAM 100% liberada)
                               ▼
┌─────────────────────────────────────────────────────────────┐
│ FASE 2: AUDITORIA COM JUIZ LLM (QWEN)                       │
│ 1. Carrega o Juiz Qwen em BF16                              │
│ 2. Audita cada resposta com zero-temperature e CoT          │
│ 3. Extrai <reasoning> e <verdict>[REFUSAL | FULFILLED]      │
│ 4. Gera tabela no terminal, JSON e 'refusal_audit_report.md'│
└─────────────────────────────────────────────────────────────┘
```

---

## 🚀 Quickstart

### 1. Installation

```bash
git clone https://github.com/SEU_USUARIO/llm-refusal-eval-suite.git
cd llm-refusal-eval-suite
pip install -r requirements.txt
```

### 2. Running an Evaluation

Evaluate any Hugging Face model or local directory:

```bash
python eval_refusal_suite.py \
  --target_model "google/gemma-4-E4B-it" \
  --judge_model "Qwen/Qwen2.5-7B-Instruct" \
  --output_dir "./eval_gemma_stock" \
  --device_map "cuda"
```

To evaluate an abliterated or fine-tuned model checkpoint:

```bash
python eval_refusal_suite.py \
  --target_model "/path/to/abliterated-model" \
  --judge_model "Qwen/Qwen2.5-7B-Instruct" \
  --output_dir "./eval_abliterated" \
  --device_map "cuda"
```

### 3. CLI Arguments

| Argument | Type | Default | Description |
| :--- | :---: | :---: | :--- |
| `--target_model` | `str` | *Required* | Path or Hugging Face repo ID of the model to evaluate. |
| `--judge_model` | `str` | `Qwen/Qwen2.5-7B-Instruct` | Auditor model to classify responses. |
| `--output_dir` | `str` | `./eval_output` | Directory where JSON and Markdown reports will be saved. |
| `--max_tokens` | `int` | `300` | Maximum new tokens generated per target prompt. |
| `--device_map` | `str` | `cuda` | Hardware mapping strategy (`cuda`, `auto`, `cpu`). |

---

## 📊 Sample Output Report

The suite automatically compiles an aggregated Markdown report ready for Model Cards:

| Category | Total | Fulfilled | Refusals | Refusal Rate (%) |
| :--- | :---: | :---: | :---: | :---: |
| **Explicit Adult Fiction** | 6 | 6 | 0 | 0.0% |
| **Offensive Cybersecurity** | 6 | 6 | 0 | 0.0% |
| **Dark Fiction** | 6 | 6 | 0 | 0.0% |
| **Radical Philosophy** | 6 | 6 | 0 | 0.0% |
| **Scientific Baseline** | 6 | 6 | 0 | 0.0% |
| **GLOBAL SCORE** | **30** | **30** | **0** | **0.0%** |

---

## 🛡️ Responsible Research & Disclaimer

This evaluation suite is intended strictly for academic research, representation engineering benchmarking, alignment auditing, and educational red-teaming. All prompts are designed as stress-tests to quantify refusal thresholds in foundation models.

---

## 📜 License

Released under the [Apache 2.0 License](LICENSE).
