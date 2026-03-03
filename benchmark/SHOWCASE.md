# Benchmark Deep Dive: revit-coder-14b vs Claude Opus 4.6

A 14B model fine-tuned on 177K Revit-specific examples for $48. Benchmark results: 0.800 avg score vs Claude Opus 4.6's 0.793 across 40 Revit C# code generation questions (25 wins, 15 losses).

**Comparison Context:** This is an asymmetric comparison. revit-coder-14b was fine-tuned on 177K domain examples; Claude Opus 4.6 was queried zero-shot with no system prompt or domain context. The results demonstrate the value of domain-specific fine-tuning but should not be interpreted as the 14B model being "better" than Claude—they serve different purposes and have different training contexts.

> **Results:** revit-coder-14b **0.800** avg vs Claude Opus 4.6 **0.793** avg. Re-run `python benchmark/score.py && python benchmark/generate_charts.py` to refresh.

---

## The Hero Chart

Which model wins each question , and by how much?

![Per-Question Score Delta](charts/01_hero_delta.png)

Bars going right = revit-coder-14b wins. Bars going left = Claude wins. Sorted by magnitude. **25 questions teal, 15 coral.**

---

## Win/Loss Summary

![Win Loss Summary](charts/05_win_loss_summary.png)

---

## Win/Loss by Difficulty

![Difficulty Win/Loss](charts/02_difficulty_winloss.png)

The 14B model performs strongest on **medium-complexity** questions (14 of 19 wins), which cover typical daily Revit plugin work. Claude has a slight edge on hard questions (6 of 12), where multi-class compositional answers benefit from a larger model.

| Difficulty | Count | 14B Wins | Claude Wins | 14B Avg | Claude Avg |
|------------|-------|----------|-------------|---------|------------|
| Easy | 9 | 5 | 4 | 0.800 | 0.796 |
| Medium | 19 | **14** | 5 | **0.839** | 0.801 |
| Hard | 12 | 6 | **6** | 0.736 | **0.779** |

---

## Difficulty Analysis

![Difficulty Breakdown](charts/03_difficulty_breakdown.png)

**Key insight:** The 14B model's biggest advantage is on medium-complexity tasks , parameter filters, family placement, view creation, extensible storage, grid/level creation. These are the exact patterns that make up daily Revit plugin work and map directly to the 143K training examples.

On hard questions, Claude's advantage comes from multi-class implementations (FamilyManager editing, TransactionGroup rollback, DWG export) where its larger context handling produces more complete code. The 14B model still wins 6 of 12 hard questions though , showing domain training transfers beyond simple pattern recall.

---

## Score Components

![Component Radar](charts/04_component_radar.png)

Three scoring axes:
- **Signal Presence (40%):** Does the response contain the right API keywords? (e.g., `FilteredElementCollector`, `Transaction`, `ParameterFilterRuleFactory`)
- **Code Quality (30%):** Does it have proper structure? (namespaces, class declarations, API patterns)
- **Completeness (30%):** Is it thorough? (response length, code blocks, error-free)

The 14B model excels at **signal presence** , it knows the exact Revit API terms from training. Claude leads on **completeness** with longer, more structured responses.

---

## Per-Question Scores

| ID | Difficulty | 14B | Claude | Delta | Winner |
|----|-----------|-----|--------|-------|--------|
| rc01 | easy | 0.477 | 0.602 | -0.125 | Claude |
| rc02 | medium | 0.922 | 0.757 | +0.165 | **14B** |
| rc03 | medium | 0.881 | 0.831 | +0.050 | **14B** |
| rc04 | hard | 0.616 | 0.764 | -0.148 | Claude |
| rc05 | medium | 0.757 | 0.557 | +0.200 | **14B** |
| rc06 | medium | 0.868 | 0.818 | +0.050 | **14B** |
| rc07 | hard | 0.802 | 0.677 | +0.125 | **14B** |
| rc08 | medium | 0.948 | 0.897 | +0.051 | **14B** |
| rc09 | easy | 0.725 | 0.757 | -0.032 | Claude |
| rc10 | hard | 0.487 | 0.786 | -0.299 | Claude |
| rc11 | medium | 0.807 | 0.708 | +0.099 | **14B** |
| rc12 | medium | 0.853 | 0.802 | +0.051 | **14B** |
| rc13 | hard | 0.682 | 0.897 | -0.215 | Claude |
| rc14 | medium | 0.922 | 0.772 | +0.150 | **14B** |
| rc15 | medium | 0.725 | 0.669 | +0.056 | **14B** |
| rc16 | hard | 0.762 | 0.743 | +0.019 | **14B** |
| rc17 | medium | 0.487 | 0.772 | -0.285 | Claude |
| rc18 | easy | 0.772 | 0.708 | +0.064 | **14B** |
| rc19 | easy | 0.814 | 0.809 | +0.005 | **14B** |
| rc20 | medium | 0.878 | 0.897 | -0.019 | Claude |
| rc21 | medium | 0.814 | 0.853 | -0.039 | Claude |
| rc22 | easy | 0.932 | 0.860 | +0.072 | **14B** |
| rc23 | hard | 0.708 | 0.897 | -0.189 | Claude |
| rc24 | medium | 0.907 | 0.897 | +0.010 | **14B** |
| rc25 | easy | 0.859 | 0.882 | -0.023 | Claude |
| rc26 | easy | 0.907 | 0.833 | +0.074 | **14B** |
| rc27 | easy | 0.932 | 0.882 | +0.050 | **14B** |
| rc28 | easy | 0.782 | 0.833 | -0.051 | Claude |
| rc29 | medium | 0.859 | 0.897 | -0.038 | Claude |
| rc30 | medium | 0.823 | 0.657 | +0.166 | **14B** |
| rc31 | medium | 0.888 | 0.882 | +0.006 | **14B** |
| rc32 | medium | 0.890 | 0.825 | +0.065 | **14B** |
| rc33 | medium | 0.850 | 0.825 | +0.025 | **14B** |
| rc34 | medium | 0.868 | 0.897 | -0.029 | Claude |
| rc35 | hard | 0.745 | 0.580 | +0.165 | **14B** |
| rc36 | hard | 0.868 | 0.777 | +0.091 | **14B** |
| rc37 | hard | 0.814 | 0.764 | +0.050 | **14B** |
| rc38 | hard | 0.745 | 0.794 | -0.049 | Claude |
| rc39 | hard | 0.888 | 0.838 | +0.050 | **14B** |
| rc40 | hard | 0.713 | 0.825 | -0.112 | Claude |

Full per-question breakdown also available in `benchmark/results/report.md`.

---

## Notable Results

### Biggest 14B Win: rc10 reversal → rc30 (ParameterFilterRuleFactory)

rc30 asks for `ParameterFilterRuleFactory` with `ElementParameterFilter` , a pattern the 14B model nails from training data. It produces the exact `CreateEqualsRule` + `WherePasses` pipeline. Claude misses the `ParameterFilterRuleFactory` factory method, using a less idiomatic approach.

### Biggest 14B Win: rc05 (Floor Type Grouping)

The 14B model produces a clean `GroupBy` with `GetTypeId` + `doc.GetElement` pattern scoring 0.757 vs Claude's 0.557. The model knows the exact Revit API pattern for type-based grouping.

### Biggest Claude Win: rc10 (IUpdater Implementation)

Claude produces a complete `IUpdater` with registration in `IExternalApplication.OnStartup`, proper `UpdaterId`, and `AddTrigger` , a complex multi-class pattern where Claude's 0.786 beats the 14B's 0.487. The 14B model struggles with the full registration lifecycle.

### The Bug Validation Question (rc04)

Both models identify the missing Transaction, but the 14B model also catches the missing `WhereElementIsNotElementType()` filter and the need for `IsReadOnly` check , demonstrating deep Revit API knowledge from its 143K training examples.

---

## Failure Analysis: Where the 14B Model Breaks Down

### 1. Complex Multi-Class Implementations

When a question requires multiple cooperating classes (IUpdater + registration, FamilyManager + IFamilyLoadOptions, TransactionGroup + error handling), the 14B model sometimes produces incomplete implementations. This is where Claude's larger capacity gives it an edge.

### 2. Token Ceiling (Fixed in v2)

In the initial run, `num_predict=512` caused truncated responses. The fix: `num_predict=1024`. Most answers now finish naturally under 500 tokens.

### 3. Double ChatML Encoding (Fixed in v2)

The v1 script manually wrapped prompts in `<|im_start|>` tags, but the Modelfile's TEMPLATE already adds them. Fix: switched from `/api/generate` to `/api/chat` endpoint.

### 4. Very Long Compositional Answers

When a question requires >600 tokens of output, the 14B model occasionally loses coherence. This is a known small-model limitation, most visible on hard questions.

---

## Scoring Methodology

### No Reference Answers

We don't use gold-standard reference answers. Instead, scoring checks for:

1. **Expected signals** , Does the API keyword appear? Each question has a list of expected terms (e.g., `FilteredElementCollector`, `Transaction`, `ParameterFilterRuleFactory`). Fraction found = signal score.

2. **Domain-specific quality** , Structural checks:
   - Code blocks, `Autodesk.Revit` namespace, class declarations, Transaction usage

3. **Completeness** , Response length, code block count, explanation presence, no errors.

### Composite Formula

```
Composite = 0.40 * signal + 0.30 * quality + 0.30 * completeness
```

### Limitations

- Signal scoring rewards keyword presence, not correctness of usage
- Quality checks are structural, not semantic (doesn't verify code compiles)
- Completeness rewards longer responses, which may favor Claude's verbose style
- No human evaluation , fully automated

### Comparison Fairness

This benchmark compares a domain-specialized model (trained on 177K Revit examples) against a general-purpose frontier model in zero-shot mode. The results demonstrate the value of fine-tuning for specialized domains but should not be interpreted as the 14B model being "better" than Claude. With proper system prompts, few-shot examples, and domain context, Claude would likely outperform the 14B model on many tasks—particularly complex multi-class implementations. The experiment shows that domain fine-tuning can achieve competitive performance at lower cost and with local inference, not that small models are superior to frontier models.

---

## Reproduce

### Prerequisites

- Python 3.10+
- Ollama with `revit-coder-14b-f16` model loaded
- `pip install matplotlib numpy requests`

### Run the Benchmark

```bash
# Step 1: Run fine-tuned model (streams tokens live, ~80 min for 40 questions)
python benchmark/run_ollama.py

# Step 2: Get Claude responses
# Run capture_claude.py to generate the prompt, paste into Claude Opus 4.6
# Save Claude's response, then parse it

# Step 3: Parse Claude responses
python benchmark/parse_claude_response.py

# Step 4: Score both models
python benchmark/score.py --verbose

# Step 5: Generate charts
python benchmark/generate_charts.py

# Results in:
#   benchmark/results/scores.json      , raw scores
#   benchmark/results/report.md        , markdown report
#   benchmark/charts/*.png             , 5 chart images
```

---

## Model Details

### revit-coder-14b

| Spec | Value |
|------|-------|
| Base Model | Qwen3-14B-Instruct |
| Fine-tuning | QLoRA (rank 64, alpha 128, Unsloth) |
| Training Data | 177,127 domain-tagged examples |
| Training Time | ~8 hours on NVIDIA B200 |
| Training Cost | ~$48 |
| Quantization | F16 (27 GB via Ollama) |
| Inference | Local, Ollama |

### Claude Opus 4.6

| Spec | Value |
|------|-------|
| Parameters | Not disclosed (~100B+) |
| Context | Zero-shot, no system prompt |
| Access | Direct chat (manual) |

---

*Benchmark suite: `benchmark/` | Charts: `benchmark/charts/` | Raw data: `benchmark/results/`*
