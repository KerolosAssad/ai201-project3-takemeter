# TakeMeter — r/gamedesign Post Classifier

A 4-label text classifier that sorts r/gamedesign posts by communicative purpose: broad design theory, specific mechanic questions, player-experience observations, or development-process discussion. Built as a comparison between a fine-tuned DistilBERT classifier and a zero-shot Groq (`llama-3.3-70b-versatile`) baseline.

## Project Summary

r/gamedesign hosts several genuinely different kinds of posts: people arguing for a general design principle, people asking how to build or balance a specific system in their own game, people trying to understand player behavior, and people talking about the process of researching, prototyping, or testing a project. A regular in the community recognizes these as different conversations; this project tries to teach a model the same distinction.

The four labels are:

- **`design_theory`** — a broad, reusable design principle, generally argued across multiple games rather than tied to one system the poster is building.
- **`mechanic_design`** — a question about creating, balancing, or evaluating a specific gameplay system.
- **`player_experience`** — a post centered on how players think, feel, choose, or behave while playing.
- **`dev_process`** — a post about the process of making, researching, testing, documenting, or revising a game project.

Full definitions, worked examples, and edge-case decision rules are in [`planning.md`](./planning.md).

## Dataset

- **200 posts** collected from r/gamedesign via PRAW, sampled from a mix of `new` and `top` listings, deduplicated by post ID and by normalized-text similarity.
- Saved as a single labeled CSV (`dataset.csv`) with `text`, `label`, and `notes` columns. The `notes` column documents which examples were genuinely difficult to label and why.
- Stratified 70/15/15 split, handled automatically by the training notebook.

**Label distribution:**

| Label | Count | % |
|---|---:|---:|
| `mechanic_design` | 76 | 38.0% |
| `design_theory` | 61 | 30.5% |
| `dev_process` | 38 | 19.0% |
| `player_experience` | 25 | 12.5% |

No label exceeds 70% of the dataset. The distribution was rebalanced once during collection by replacing several weak/narrow `mechanic_design` examples with stronger `dev_process` and `player_experience` examples, since those two labels were initially underrepresented.

**Resulting test set:** 30 examples (`design_theory`: 9, `mechanic_design`: 12, `player_experience`: 4, `dev_process`: 5). Note that `player_experience` landed at only 4 test examples — below the 10-per-class minimum I'd originally targeted for stable per-class metrics. Per-class precision/recall on this label should be read as directionally suggestive, not statistically robust; a single misclassification shifts its recall by 25 points.

### Three genuinely difficult labeling decisions

1. **"Earliest balance patch in history?"** (chess queen / 7×7-to-8×8 board change) — explicitly flagged by the poster as "not a serious post," which initially suggested `player_experience` (a casual observation) rather than an argued claim. I labeled it `design_theory` because it does use specific historical evidence (the rule change, the resulting shift toward aggressive play) to argue an actual claim about what counts as a successful balance patch — the joking tone doesn't change that the post's content is an argument, not an observation of behavior.
2. **The Syntaris automation post** ("Automation without micromanagement, is it possible to keep depth while reducing control?") — the poster describes their own prototype in significant mechanical detail (graph-based modules, worker pathing, clustering tradeoffs), which on a surface read looks like `mechanic_design`. I labeled it `design_theory` because the post's center of gravity is a genre-wide question ("can automation games stay deep without micromanagement?"), and the prototype is presented as evidence for that broader claim rather than as an open, unresolved system question the poster is asking for help with — the same "removal test" logic I use for the `design_theory`/`player_experience` boundary.
3. **"Should classes be balanced to similar win rates, or is intentional difficulty variation a feature?"** — presents real win-rate data (34%–89% spread) and a reframing ("what if the spread is the feature?"), which could read as `design_theory` since it poses a generalizable question. I labeled it `mechanic_design` because the question is fundamentally about what to do with the poster's own specific class roster in their own dungeon-solitaire game, not a claim meant to generalize beyond that project.

## Model & Training

- **Base model:** `distilbert-base-uncased` (HuggingFace), fine-tuned with a randomly-initialized classification head for 4 labels.
- **Training data:** 140 examples (the 70% train split).
- **Hardware:** Google Colab, T4 GPU.

I ran three training configurations rather than one, because the first run's behavior (validation accuracy *decreasing* across epochs) was itself informative and worth diagnosing rather than accepting as a single result.

| Run | Epochs | Learning rate | Val. accuracy by epoch | Final val. accuracy |
|---|---:|---:|---|---:|
| 1 (default) | 3 | 2e-5 | 0.40 → 0.367 → 0.367 | 0.367 |
| 2 | 3 | 1e-5 | 0.30 → 0.367 → 0.40 | 0.40 |
| 3 | 6 | 1e-5 | 0.30 → 0.367 → 0.40 → 0.40 → 0.367 → 0.367 | 0.367 (best checkpoint: epoch 3/4 at 0.40) |

**Run 1 (notebook defaults: 3 epochs, lr 2e-5):** validation accuracy peaked immediately at epoch 1 (0.40) and degraded every epoch after, while validation loss kept slowly decreasing — the model became more confident in its predictions while those predictions got *less* correct, a clear early-overfitting signature on a training set this small.

**Run 2 (lr lowered to 1e-5, 3 epochs):** *Changed from default because* run 1's overfitting trajectory suggested the optimizer was taking steps too large for a training set with only 27 total optimization steps (140 examples / batch size 16, over 3 epochs). At a lower learning rate, the trajectory reversed: accuracy climbed every epoch and had not yet plateaued at epoch 3.

**Run 3 (lr 1e-5, epochs raised to 6):** *Changed because* run 2 was still improving when training stopped, so I extended training to see whether the upward trend continued, plateaued, or reversed (delayed overfitting). It plateaued at 0.40 around epochs 3-4, then reversed back to 0.367 by epoch 6 — the same overfitting pattern as run 1, just delayed by the lower learning rate. Validation loss decreased every single epoch through all 6, even as accuracy declined in the back half, confirming the model was gaining confidence without gaining correctness.

**Final model used for evaluation:** Run 3's training, with `load_best_model_at_end=True` (`metric_for_best_model="accuracy"`) — meaning the actual saved/evaluated model is the epoch-3 checkpoint (0.40 validation accuracy), not the degraded epoch-6 weights, despite training running the full 6 epochs.

Across all three configurations, peak validation accuracy never exceeded 0.40 — well below the Groq baseline's 0.833 test accuracy. This ceiling appears to be a function of training-set size and task difficulty, not hyperparameter choice; see Reflection below.

## Baseline

Zero-shot baseline using `llama-3.3-70b-versatile` via Groq, given a system prompt containing the full label taxonomy, one worked example per label, and explicit decision rules for every edge case documented in `planning.md` (the `design_theory`/`player_experience` "removal test," the `mechanic_design`/`dev_process` "journey vs. final system" distinction, and routing rules for resource requests and opinion polls).

**Baseline results (test set, n=30, 0 unparseable responses):**

| Label | Precision | Recall | F1 | Support |
|---|---:|---:|---:|---:|
| `design_theory` | 0.70 | 0.78 | 0.74 | 9 |
| `mechanic_design` | 0.92 | 0.92 | 0.92 | 12 |
| `player_experience` | 1.00 | 0.75 | 0.86 | 4 |
| `dev_process` | 0.80 | 0.80 | 0.80 | 5 |
| **Macro avg** | **0.85** | **0.81** | **0.83** | 30 |

**Overall accuracy: 0.833**

## Evaluation Report

### Fine-tuned model results (test set, n=30)

| Label | Precision | Recall | F1 | Support |
|---|---:|---:|---:|---:|
| `design_theory` | 0.33 | 0.11 | 0.17 | 9 |
| `mechanic_design` | 0.44 | 1.00 | 0.62 | 12 |
| `player_experience` | 0.00 | 0.00 | 0.00 | 4 |
| `dev_process` | 0.00 | 0.00 | 0.00 | 5 |
| **Macro avg** | **0.19** | **0.28** | **0.20** | 30 |

**Overall accuracy: 0.433**

### Baseline vs. fine-tuned comparison

| Metric | Baseline (Groq) | Fine-tuned (DistilBERT) |
|---|---:|---:|
| Overall accuracy | 0.833 | 0.433 |
| Macro F1 | 0.83 | 0.20 |
| `design_theory` F1 | 0.74 | 0.17 |
| `mechanic_design` F1 | 0.92 | 0.62 |
| `player_experience` F1 | 0.86 | 0.00 |
| `dev_process` F1 | 0.80 | 0.00 |

The fine-tuned model's overall accuracy (0.433) is numerically higher than its own training-time validation accuracy (0.40) and higher than the majority-class baseline (~0.38, always predicting `mechanic_design`) — but this is misleading on its own. The model achieved that accuracy by predicting `mechanic_design` for roughly 27 of the 30 test examples (90%), not by learning the four categories. `mechanic_design` recall is a perfect 1.00 only because the model predicted that label almost everywhere; `player_experience` and `dev_process` both score exactly 0.00 across precision, recall, and F1 — the model never predicts either label at all, for any test example.

This is a more severe failure than the training-time validation metrics alone suggested. Validation accuracy degradation (Model & Training, above) showed the model was *losing* the ability to distinguish classes over training; the test set confirms what it converged to: a near-total collapse onto the single largest training class.

### Confusion matrix (fine-tuned model)

| | Pred: `design_theory` | Pred: `mechanic_design` | Pred: `player_experience` | Pred: `dev_process` |
|---|---:|---:|---:|---:|
| **True: `design_theory`** | 1 | 8 | 0 | 0 |
| **True: `mechanic_design`** | 0 | 12 | 0 | 0 |
| **True: `player_experience`** | 0 | 4 | 0 | 0 |
| **True: `dev_process`** | 2 | 3 | 0 | 0 |

The `mechanic_design` column has nonzero entries in every row — it is where almost all predictions land regardless of true label. The `player_experience` and `dev_process` columns are entirely empty: the model never predicts either label, correct or otherwise, for any of the 30 test examples.

### Three wrong predictions, analyzed

**1. "What is it with Halo that makes me not underutilize grenades?"**
- True label: `player_experience` / Predicted: `mechanic_design` (confidence: 0.29)
- Why this boundary is hard: this post is one of my own worked examples in `planning.md`, specifically chosen to demonstrate the `design_theory`/`player_experience` removal test (the same game, Halo, appears under `design_theory` in a different post arguing a general principle about optimal vs. interesting play). The fine-tuned model has no access to that removal-test logic — it can only see surface text, and a post discussing a specific in-game mechanic (grenades, throwing animation, destructible cover) reads lexically similar to a `mechanic_design` post even though its actual subject is the poster's own behavior and motivation.
- Labeling problem or model/data problem: model/data problem. I labeled this consistently with my own decision rule, and a second pass over the post confirms the label is correct under that rule. The model failed to learn the rule, not because the rule was misapplied during annotation.
- What would fix it: this is a strong piece of evidence for the broader hypothesis below — the fix isn't more examples of this exact post type, it's giving the model some equivalent of the reasoning rule itself, which a fine-tuned classification head structurally cannot receive the way a prompted LLM can.

**2. "How do people design book adaptations?"**
- True label: `dev_process` / Predicted: `design_theory` (confidence: 0.28)
- Why this boundary is hard: this is the only one of the 15 wrong predictions shown that *didn't* default to `mechanic_design`, meaning the model did register some signal in this post beyond "guess the majority class" — it just registered the wrong signal. The post asks "how do designers usually go about this," a phrasing that's structurally similar to the genre-spanning questions used in several `design_theory` training examples, even though this specific post is actually about the poster's own process for adapting one specific webnovel into one specific game.
- Labeling problem or model/data problem: borderline. The post genuinely sits close to the `design_theory`/`dev_process` boundary I documented for resource requests — "how do people typically do X" can read as either a request for general design wisdom or a request for process guidance on the poster's own project. I labeled it `dev_process` because the deciding context (an indie game adaptation, the poster's own project) is in the post, but the phrasing alone, stripped of that context, would more plausibly read as `design_theory`.
- What would fix it: more training examples explicitly contrasting "general question phrased like theory" against "process question phrased like theory" might help the model learn this particular lexical trap, since this error isn't explained by majority-class collapse the way most of the others are.

**3. "Demo Design Question: Separate Curated Map or Part of the Full Open World?"**
- True label: `dev_process` / Predicted: `mechanic_design` (confidence: 0.30)
- Why this boundary is hard: the post title contains "Design Question," and the body discusses building paths and gathering resources — vocabulary that overlaps heavily with `mechanic_design` training examples. But the actual question is a production-scoping decision (how much of the full game world to expose in a demo build), which is exactly the `mechanic_design`/`dev_process` boundary I documented: the final system is not the subject here, a packaging/scoping decision about an existing system is.
- Labeling problem or model/data problem: model/data problem, and a textbook case for it. This is precisely the ambiguity my own `mechanic_design` vs. `dev_process` edge-case rule was written to resolve, and the rule resolves it cleanly by hand. The model has no access to "is the final system the subject, or is the process of releasing/scoping it the subject" as a question it can ask — it only has lexical overlap to go on, and the lexical overlap here points the wrong way.
- What would fix it: same root cause as #1. This isn't a labeling inconsistency or a data-quality issue; it's a category of distinction (system vs. process, when the same vocabulary describes both) that requires the kind of explicit reasoning step a prompted LLM gets for free and a fine-tuned classifier does not.

### Sample classifications

| Post excerpt | True label | Predicted label | Confidence |
|---|---|---|---:|
| "What is it with Halo that makes me not underutilize grenades?" | `player_experience` | `mechanic_design` | 0.29 |
| "How do people design book adaptations?" | `dev_process` | `design_theory` | 0.28 |
| "Demo Design Question: Separate Curated Map or Part of the Full Open World?" | `dev_process` | `mechanic_design` | 0.30 |
| "Picking a resource type for a 4X" | `mechanic_design` | `mechanic_design` | ~0.30* |
| "Mina the Hollower had an 800+ Page Design Doc" | `dev_process` | `mechanic_design` | 0.29 |

\* Exact confidence not pulled from a specific run output; included to show a correct prediction for contrast.

The one correct prediction shown ("Picking a resource type for a 4X") is reasonable on its face — it genuinely is a `mechanic_design` post, asking a specific resource-system question for the poster's own 4X game — but given that confidence scores are essentially flat (0.28-0.30) across every prediction shown here, correct or incorrect, this "correct" classification is much more likely an artifact of the model's collapse onto `mechanic_design` as a default guess than evidence the model identified anything specific about this post's content.

**A note on confidence:** every wrong prediction listed above carries a confidence between 0.28 and 0.30 — barely above the 0.25 a uniform random guess across 4 classes would produce. The model isn't confidently wrong; it's barely committing to any prediction at all. This is consistent with a model that never found a strong signal for any class and is outputting something close to its prior over the label distribution, rather than a model that learned sharp (even if sometimes incorrect) decision boundaries.

### Reflection: what the model learned vs. what I intended

The baseline already exceeded every target I set in `planning.md` (macro F1 0.83 against a 0.70 target, all per-class recalls ≥0.75 against a 0.70/0.50 floor) **before any fine-tuning happened**, and the fine-tuned model came nowhere close to closing that gap across three separate training configurations (final macro F1: 0.20).

I intended fine-tuning to produce a small, fast, deployable classifier that approximated the taxonomy I designed. What actually happened is closer to a collapse: the model predicted `mechanic_design` for 90% of the test set, never predicted `player_experience` or `dev_process` at all, and every prediction — right or wrong — carried confidence barely above random chance (0.28-0.30 out of a 0.25 baseline). This isn't "the model learned an imperfect version of my categories." It's closer to "the model found that guessing the largest class minimizes loss on a training set this small, and never developed a meaningfully different strategy."

The more precise conclusion isn't just "140 examples is too little data" — it's that **this specific taxonomy's hardest boundaries are defined by explicit reasoning procedures rather than lexical patterns**, and that distinction matters for which kind of model can succeed at it. The `design_theory`/`player_experience` removal test ("does the argument survive without the behavioral example?") and the `mechanic_design`/`dev_process` "system vs. process" test are not vocabulary distinctions — two posts using nearly identical words ("design," "system," "players") can land on opposite sides of either boundary depending on what the post's central question actually is. The Groq baseline succeeds because it receives these rules directly, as natural-language instructions, and applies them. DistilBERT receives only `(text, label)` pairs and has to *induce* an equivalent decision procedure purely from statistical patterns in 140 examples, with no access to the rules themselves. More training data might eventually allow that induction to succeed, but based on this result, the bottleneck isn't best described as "needs more examples of the same kind" — it's that the examples alone don't carry the reasoning the labels actually depend on. The wrong-prediction analysis above (especially #1 and #3) makes this concrete: in both cases, I can resolve the ambiguity instantly by applying my own written rule, and the model has no equivalent rule available to it at all.

For this specific classification task, **grounded prompting outperformed fine-tuning, and the gap is plausibly structural rather than a matter of data volume** — a finding I did not expect going into this project, where my original hypothesis was simply that fine-tuning would underperform the baseline by some moderate margin, not collapse to near-majority-class guessing.

## Spec Reflection

**One way the spec helped:** Writing the hard edge cases in `planning.md` *before* annotating forced me to commit to a removal-test decision rule for the `design_theory`/`player_experience` boundary up front. When I hit genuinely ambiguous posts during annotation (like the chess balance-patch post), I had an actual rule to apply rather than making an ad hoc call — and I could later reuse that exact rule, word for word, in the Groq baseline prompt, which is plausibly part of why the baseline performs as well as it does.

**One way my implementation diverged from the spec:** The spec's Dataset Splitting section targeted a minimum of 10 test examples per label (40+ total). At 200 total examples and a 70/15/15 split, my actual test set landed at 30 examples, with `player_experience` at only 4. I chose not to inflate the dataset size purely to hit that target, since `planning.md`'s Data Collection Plan explicitly prioritized example quality over forcing numerical equality across labels — but this is a real tradeoff, and it means my `player_experience` metrics are less statistically reliable than the spec originally intended.

## AI Usage

1. **Dataset construction and dedup:** I used Claude to process and clean batches of scraped r/gamedesign posts — classifying each post against my written label definitions, flagging duplicate/near-duplicate posts by normalized title matching, and tracking class-distribution counts as I added batches. I did not accept any label without reviewing the full post text myself; in several cases I overrode the suggested label (e.g., routing genre-pattern posts to `mechanic_design` rather than `design_theory` per the resource-request rule). All deduplication was verified programmatically (exact and near-duplicate checks) rather than taken on the AI's word alone.
2. **Classification prompt design for the Groq baseline:** I drafted an initial system prompt myself, then asked Claude to check it against `planning.md` for internal consistency. This caught a real contradiction in an earlier draft (the prompt's opinion-poll rule didn't match the routing logic I'd actually used during annotation), and the prompt's edge-case rules were rewritten to match the document exactly before running the baseline.
3. **Failure pattern analysis on the wrong predictions:** After Section 4 produced the fine-tuned model's confusion matrix and wrong-prediction list, I discussed the misclassified examples (true label, predicted label, confidence, text excerpt) with Claude and asked it to identify patterns rather than just list individual errors. It surfaced two patterns I then verified directly against the raw output rather than accepting at face value: (1) the model had predicted `mechanic_design` for roughly 27 of 30 test examples, meaning most "correct" `mechanic_design` predictions were a side effect of majority-class collapse rather than learned signal — I confirmed this by back-calculating the implied prediction count from the reported precision/recall, not just trusting the claim; (2) every prediction's confidence score, right or wrong, fell in a narrow 0.28–0.30 band, barely above the 0.25 a uniform 4-class guess would produce — I verified this by checking the actual range across all 15 listed wrong predictions rather than a sampled subset. What I corrected: my original hypothesis going into fine-tuning (stated in `planning.md`'s Baseline Results section) was that DistilBERT would underperform the Groq baseline by "some moderate margin" due to limited training data. The verified patterns above led me to revise that conclusion in the README's Reflection section to something more specific — that the model didn't learn a weaker version of the taxonomy, it collapsed to near-constant majority-class guessing, and the most likely reason is structural (the labels depend on explicit reasoning rules the baseline received and the classifier never could), not simply a matter of needing more examples of the same kind.

## Repository Structure

```
.
├── README.md
├── planning.md
├── dataset.csv
├── evaluation_results.json
└── confusion_matrix.png
```