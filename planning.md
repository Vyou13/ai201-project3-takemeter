# Project Planning Specification: TakeMeter

This document outlines the core architecture, taxonomy constraints, and data collection strategies designed prior to the execution of the TakeMeter classification pipeline.

---

## 📺 1. Community Selection & Framing
* **Chosen Community:** Online television fan communities (specifically targeting subreddits like `r/television`).
* **Task Viability:** Prestige television discourse is uniquely suited for multi-class classification because user engagement spans an expansive qualitative spectrum. In a single thread, high-effort production critiques sit directly alongside reactive memes and narrative forecasting. This intense linguistic variance provides distinct semantic boundaries for text classification.

---

## 🧠 2. Label Taxonomy & Operational Definitions

### `critique`
* **Definition:** A post or comment evaluating the technical, aesthetic, structural, or performance attributes of a television production (e.g., acting choices, screenplay pacing, musical score, lighting, directing).
* **Anchor Example 1:** *"The use of low-key tracking shots in the corridor sequences perfectly isolates the character's mounting paranoia."*
* **Anchor Example 2:** *"The dialogue this season feels incredibly clunky, reducing complex political operators to shouting caricatures."*

### `theory_prediction`
* **Definition:** A post or comment explicitly forecasting future plot developments, character deaths, or canonical reveals using textual clues, foreshadowing, or comparisons to literary source material.
* **Anchor Example 1:** *"Because the director focused on the sigil ring twice in this episode, I predict he is planning to poison the king in the finale."*
* **Anchor Example 2:** *"The show is setting up an alliance between the two minor houses based on that obscure line from season 1."*

### `surface_opinion`
* **Definition:** A post or comment summarizing immediate emotional responses, personal preferences, unelaborated praise or complaints, memes, or casual humor.
* **Anchor Example 1:** *"One of the single greatest TV shows I've ever watched. I've only seen it once but I find myself thinking about it at least once a week."*
* **Anchor Example 2:** *"That episode was total trash. Worst hour of television ever created."*

---

## ⚠️ 3. Hard Edge Cases & Deterministic Boundaries
* **The Anticipated Boundary Blur:** Hybrid inputs where a user couples technical vocabulary with highly speculative future predictions (e.g., *"The pacing is so fast they are obviously setting up a massive war in episode 8"*).
* **Resolution Strategy:** A priority-tier hierarchy will be enforced. If a post mentions technical production attributes, it is up-voted to `critique` *unless* the technical vocabulary is purely decorative framing for a plot forecast. If the core linguistic intent is guessing a future narrative outcome, it maps strictly to `theory_prediction`. Pure emotional sentiment with casual terminology automatically cascades to `surface_opinion`.

---

## 📥 4. Data Collection & Imbalance Mitigation Plan
* **Collection Strategy:** Manual text sampling from public post-episode review mega-threads on Reddit.
* **Volume Targets:** 200 distinct text items, targeting approximately 65 examples per structural label.
* **Imbalance Fail-safe:** If after pulling 200 random comments a single category (such as `surface_opinion`) dominates more than 70% of the dataset rows, sampling methods will shift from random pulling to keyword-targeted thread hunting (e.g., searching for terms like *"cinematography"*, *"pacing"*, or *"predict"*) until all three labels capture at least 20% to 30% of the distribution.

---

## 📊 5. Evaluation Metrics Justification
* **Primary Metrics:** Per-Class Precision, Per-Class Recall, and Macro F1-Score.
* **Rationale:** Raw accuracy is a dangerous metric because it masks asymmetric model failures. If the model identifies `surface_opinion` effortlessly but completely fails to draw the line between a `critique` and a `theory_prediction`, global accuracy might still look acceptable while the core classifier utility remains broken. Tracking per-class F1-scores forces accountability across the softer semantic boundaries.

---

## 🎯 6. Definition of Success
* **Deployment Threshold:** To consider this text classifier ready for deployment as an automated community filtering tool, it must hit:
  * An overall accuracy threshold of **$\ge 80\%$** on the test set.
  * A minimum F1-score of **$0.75$** for the complex `critique` class.
* **Justification:** Human moderation of online discourse is highly subjective; a model operating at a consistent $80\%$ baseline provides reliable scale for sorting vast community archives without introducing massive systemic error.

---

## 🤖 7. AI Tool Plan

### Label Stress-Testing
Prior to annotating rows manually, the proposed taxonomy definitions will be fed into Claude to synthetically generate 10 adversarial boundary comments. If these synthetic samples cannot be cleanly mapped using our deterministic boundary rules, the label definitions will be iteratively adjusted before human labeling begins.

### Annotation Assistance
No automated LLM bulk pre-labeling will be deployed for this project. To maximize data integrity and keep human intuition closest to the training distribution, all 200 data vectors will be purely hand-annotated in the master spreadsheet.

### Failure Analysis
Following evaluation passes, any mismatched indices will be exported to an AI tool to isolate thematic clusters among the misclassifications. The AI's analytical patterns will be manually verified against the raw text to cross-examine whether structural syntax lengths or negative vocabulary terms are polluting DistilBERT's attention masks.
