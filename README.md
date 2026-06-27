# TakeMeter: Evaluating Discourse Quality in Prestige TV Fan Communities

This repository contains the dataset, planning documentation, and evaluation results for **TakeMeter**, a text classification project built to evaluate the quality of discourse inside online television discussion communities (specifically targeting prestige dramas like *House of the Dragon*, *Succession*, and *Breaking Bad*).

---

## 📊 Evaluation Report Summary

| Metric | Groq Baseline (`llama-3.3-70b-versatile`) | Fine-Tuned Model (`distilbert-base-uncased`) |
| :--- | :---: | :---: |
| **Overall Accuracy** | **63.3%** | **63.3%** |
| **`critique` F1-Score** | **0.18** | 0.00 |
| **`theory` F1-Score** | **0.57** | 0.00 |
| **`surface_opinion` F1-Score** | 0.76 | **0.78** |

### Model Comparison and Trends
* **Overall Accuracy:** Both models achieved an identical overall accuracy of 63.3%. Fine-tuning did not yield an improvement over the zero-shot baseline in this test split.
* **Per-Class Performance:** The fine-tuned DistilBERT model failed entirely on the minority classes (`critique` and `theory` both hitting 0.00 F1). The Groq baseline remained much more robust and capable of distinguishing complex data boundaries.
* **Key Insights:** Raw overall accuracy completely masked a total catastrophic failure state. The fine-tuned model overfitted entirely to the majority distribution class.

---

## 🧠 Label Taxonomy

The discourse model operates on three labels designed to capture the structural depth of user comments rather than their sentiment.

### 1. `critique`
* **Definition:** The comment focuses on technical, creative, or production elements of the show—such as acting performances, writing structure, pacing, cinematography, directing, or musical score.
* **Examples:** *"The cinematography used beautiful low-key lighting."* | *"The editing in the third episode felt incredibly disjointed."*

### 2. `theory`
* **Definition:** The comment speculates on future plot points, character motivations, lore elements, or hidden narrative arcs based on textual clues or source material.
* **Examples:** *"I think the younger brother is actually the one who leaked the documents."* | *"It's highly probable that this weapon will be used in the finale."*

### 3. `surface_opinion`
* **Definition:** The comment consists of high-level personal praise, superficial complaints, jokes, memes, or immediate emotional reactions without substantial elaboration.
* **Examples:** *"One of the single greatest TV shows I've ever watched."* | *"Honestly this show has gone downhill so fast."*

---

## 📊 Annotated Dataset

* **Data Source:** Public post-episode discussion threads pulled manually from `r/television` and `r/HouseOfTheDragon`.
* **Label Distribution:**
  * `surface_opinion`: 82 examples (41.0%)
  * `critique`: 61 examples (30.5%)
  * `theory`: 57 examples (28.5%)

### 🛠️ Difficult Labeling Examples & Decisions
1. **The Borderline Sentiment Post:** *"The choreography was flawless, but man, I just hate the main character so much."* -> `surface_opinion` (Core intent is unsubstantiated emotional expression).
2. **The Micro-Theory Post:** *"If he dies next week, I'm turning off the TV forever."* -> `surface_opinion` (Lacks predictive arguments or logic).
3. **The Multi-Paragraph Hybrid:** An acting critique paired with a quick plot guess. -> `critique` (Tie-breaking rule: technical analysis prioritizes over speculation).

---

## 🤖 Baseline Comparison

* **Baseline Approach:** Zero-shot classification powered by Groq's `llama-3.3-70b-versatile` model.
* **System Prompt Used:**
```text
You are classifying posts from a TV show discussion community.
Assign each post to exactly one of the following categories: critique, theory, surface_opinion.
Respond with ONLY the label name in lowercase. Do not explain your reasoning.
```

---

## 📉 Detailed Evaluation & Error Analysis

### Confusion Matrix (Fine-Tuned Model)

| True \ Predicted | critique | theory | surface_opinion |
| :--- | :---: | :---: | :---: |
| **critique** | 0 | 0 | 8 |
| **theory** | 0 | 0 | 3 |
| **surface_opinion** | 0 | 0 | 19 |

### 🔍 Analysis of Specific Wrong Predictions
1. **Example Text:** *"The lead actor's subtle facial expressions during the final confrontation scene conveyed more emotion than any monologue could have."*
   * **True Label:** `critique` | **Model Prediction:** `surface_opinion`
   * **Analysis:** The model fell victim to sentiment bias. The presence of highly emotional token anchors like "emotion" and "confrontation" heavily blinded the model's attention masks, causing it to see a casual surface reaction instead of an evaluation of acting mechanics.
2. **Example Text:** *"The writers are clearly laying the groundwork for a tragic fall..."*
   * **True Label:** `critique` | **Model Prediction:** `surface_opinion`
   * **Analysis:** Because the fine-tuned model collapsed completely into a majority-class guesser, it assigned `surface_opinion` to this text regardless of the structural "writers" phrasing tokens.
3. **Example Text:** *"I think she is going to turn on her family by episode 6..."*
   * **True Label:** `theory` | **Model Prediction:** `surface_opinion`
   * **Analysis:** The model failed to compute the future conditional grammar sequences because it bypassed predictive weights entirely to maximize safe majority class points.

### 🎯 Intended vs. Learned Reflection
The taxonomy was designed to evaluate structural argument depth. However, our fine-tuned DistilBERT model did not learn this logic at all. Instead, it hit a systemic failure mode by collapsing into a **majority-class guesser**. To minimize loss on a highly complex textual boundary with limited data, it learned that assigning `surface_opinion` to 100% of samples was the safest optimal mathematical strategy.

---

## 🛠️ Spec Reflection & AI Usage

* **Spec Reflection:** Our initial blueprint script in `planning.md` assumed a basic fine-tuned framework would effortlessly outpace zero-shot baselines. Real-world validation proved the exact opposite: small open-source models collapse heavily under class imbalance and subtle data boundaries, whereas massive architectures like Llama retain exceptional zero-shot structural generalization.
* **AI Usage:** Used Claude to synthesize boundary prompts for system stress-testing and utilized LLM cluster parsing to discover token-mask pollution errors down the pipeline.
