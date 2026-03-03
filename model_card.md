---
license: apache-2.0
base_model: Qwen/Qwen3-14B
tags:
  - revit
  - bim
  - code-generation
  - architecture
  - engineering
  - construction
  - aec
  - ifc
  - fine-tuned
  - qlora
  - unsloth
datasets:
  - custom
language:
  - en
pipeline_tag: text-generation
model-index:
  - name: revit-coder-14b
    results:
      - task:
          type: text-generation
          name: Revit API Code Generation
        metrics:
          - type: custom-composite
            value: 0.800
            name: Composite Score (40 questions)
---

# revit-coder-14b

![Hero Banner](images/01-hero-banner.png)

A fine-tuned Qwen3-14B specialized in **Revit API code generation**, **IFC reasoning**, and **BIM development patterns**.

Trained on 177,127 domain-tagged examples for $48 on a single GPU. Achieved competitive performance with Claude Opus 4.6 (zero-shot) on a 40-question Revit C# benchmark (0.800 avg vs 0.793).

**GitHub:** [schauh11/revit-coder-14b](https://github.com/schauh11/revit-coder-14b) - Benchmark suite, training scripts, and full results.

## Benchmark Results

**40-question Revit C# benchmark**, pure code generation, no IFC or pattern questions:

| Model | Avg Score | Wins | Parameters | Inference |
|-------|-----------|------|------------|-----------|
| **revit-coder-14b** | **0.800** | **25** | 14B | Local (Ollama) |
| Claude Opus 4.6 | 0.793 | 15 | ~100B+ | API |

**Note:** This comparison shows revit-coder-14b (fine-tuned on 177K Revit examples) vs Claude Opus 4.6 (zero-shot, no domain training). The fine-tuned model naturally has domain advantages on this specific benchmark.

### By Difficulty

| Difficulty | Count | revit-coder-14b | Claude Opus 4.6 | Winner |
|------------|-------|-----------------|-----------------|--------|
| Easy | 9 | 0.800 | 0.796 | 14B |
| Medium | 19 | **0.839** | 0.801 | **14B** |
| Hard | 12 | 0.736 | **0.779** | Claude |

![Average Scores by Difficulty](images/04-scores-by-difficulty.png)

![Scoring Components Breakdown](images/05-scoring-components.png)

All 40 questions and both models' full responses are published in [BENCHMARK_FULL.md](https://github.com/schauh11/revit-coder-14b/blob/main/benchmark/BENCHMARK_FULL.md).

## Intended Use

**Primary:** Domain-specialized Revit API code generation. An experiment in fine-tuning a small model on domain-specific data to see how it compares to general-purpose frontier models.

**Capabilities:**
- Generate correct Revit C# code (FilteredElementCollector, Transaction patterns, BuiltInParameter)
- Validate Revit API usage (catch missing Transactions, null checks, type filter issues)
- Reason about IFC spatial hierarchies and property sets
- Produce Revit development patterns (IExternalEventHandler, IUpdater, ISelectionFilter)

**Limitations:**
- Optimized for Revit 2025/2026 (.NET 8) API, may not cover older API versions
- Strongest on revit_csharp domain; weaker on IFC STEP format generation
- Best results under 800 tokens; quality may degrade on very long outputs
- Not a general-purpose coding model, use a frontier model for non-Revit tasks
- The benchmark comparison is inherently asymmetric: this model received domain-specific training while Claude did not. In production use with proper system prompts and examples, Claude may outperform this model on complex tasks

## Training

![Training Pipeline](images/02-training-pipeline.png)

| Spec | Value |
|------|-------|
| **Base Model** | Qwen3-14B-Instruct |
| **Method** | QLoRA (rank 64, alpha 128) |
| **Framework** | Unsloth + HuggingFace TRL |
| **Training Data** | 159,414 examples (90% of 177K total) |
| **Validation Data** | 8,856 examples (5%) |
| **Epochs** | 3 |
| **Sequence Length** | 4096 tokens |
| **Batch Size** | 16 effective (4 x 4 gradient accumulation) |
| **Learning Rate** | 2e-4 (cosine schedule) |
| **Warmup Steps** | 200 |
| **Optimizer** | AdamW 8-bit |
| **Weight Decay** | 0.01 |
| **GPU** | NVIDIA B200 192GB |
| **Training Time** | ~8 hours |
| **Training Cost** | ~$48 |
| **LoRA Dropout** | 0 (required for Unsloth) |
| **Early Stopping Patience** | 3 epochs |
| **Random Seed** | 42 |
| **Packing** | Enabled |

### Train/Val/Test Split

| Split | Examples | Percentage |
|-------|----------|------------|
| Train | 159,414 | 90% |
| Validation | 8,856 | 5% |
| Test | 8,857 | 5% |

Stratified sampling by domain ensures proportional representation across all splits. The 40-question benchmark is separate from these splits—it tests zero-shot generalization on new questions.

### Training Data Distribution

![Data Distribution](images/03-data-distribution.png)

| Domain | Records | % | Description |
|--------|---------|---|-------------|
| revit_csharp | 143,060 | 72.7% | Revit API C# code from docs, examples, references |
| ifc_reasoning | 44,571 | 22.6% | IFC topology, spatial hierarchies, BIM reasoning |
| aps_schema | 4,980 | 2.5% | APS/Forge cloud API patterns |
| revit_patterns | 3,758 | 1.9% | Development patterns (IUpdater, events, filters) |
| revit_python | 285 | 0.1% | pyRevit Python automation |
| mcp_tools | 149 | 0.1% | MCP tool definitions for AI-BIM integration |

**Data format:** ChatML with domain-specific system prompts. Each record includes `<|im_start|>system`, `<|im_start|>user`, `<|im_start|>assistant` sections.

**Sources:** Revit API Docs 2025/2026, Revit SDK code examples, IFC/BIM specifications, Autodesk forums, Building Coder blog, APS SDK documentation.

## Usage

### Ollama (Recommended)

```bash
# Pull or create the model
ollama run revit-coder-14b-f16

# Query
ollama run revit-coder-14b-f16 "Write C# code to collect all walls and group by type name"
```

### Python (transformers)

```python
from transformers import AutoModelForCausalLM, AutoTokenizer

model_name = "schauh11/revit-coder-14b"  # HuggingFace repo
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForCausalLM.from_pretrained(model_name, device_map="auto")

messages = [
    {"role": "system", "content": "You are a Revit API expert specialized in C# and .NET 8."},
    {"role": "user", "content": "Write code to get all rooms and their areas."},
]
text = tokenizer.apply_chat_template(messages, tokenize=False, add_generation_prompt=True)
inputs = tokenizer(text, return_tensors="pt").to(model.device)
outputs = model.generate(**inputs, max_new_tokens=1024, temperature=0.1)
print(tokenizer.decode(outputs[0], skip_special_tokens=True))
```

### Unsloth (for inference with LoRA adapter)

```python
from unsloth import FastLanguageModel

model, tokenizer = FastLanguageModel.from_pretrained(
    model_name="path/to/revit-coder-14b-lora",
    max_seq_length=4096,
    load_in_4bit=True,
)
FastLanguageModel.for_inference(model)
```

## Scoring Methodology

Each benchmark response is scored on three axes:

- **Signal Presence (40%):** Fraction of expected domain keywords found (e.g., `FilteredElementCollector`, `Transaction`, `IfcRelAggregates`)
- **Code Quality (30%):** Domain-specific structural checks (namespaces, class structure, API patterns)
- **Completeness (30%):** Response length, code block formatting, error-free output

**Composite = 0.4 x signal + 0.3 x quality + 0.3 x completeness**

No reference answers are used. Scoring is quality-based, checking that correct API patterns and domain concepts appear in the output.

## Environmental Impact

| Metric | Value |
|--------|-------|
| Hardware | 1x NVIDIA B200 192GB |
| Training time | ~8 hours |
| Cloud cost | ~$48 (RunPod) |
| CO2 estimate | ~2.4 kg (based on US grid average) |

## Citation

```bibtex
@misc{revit-coder-14b-2026,
  title={revit-coder-14b: Domain-Specialized Code Generation for Revit API},
  author={Sanjay Chauhan},
  year={2026},
  url={https://huggingface.co/schauh11/revit-coder-14b}
}
```

## License

Apache 2.0, same as the base Qwen3-14B model.
