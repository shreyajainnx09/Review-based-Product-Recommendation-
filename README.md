# Review-Based Product Recommendation (Sephora)

> Tell it what you want — *"hydrating, lightweight and good value"* — and it recommends a skincare product and **explains why, using real sentences quoted from customer reviews.**

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1cJCMZcd2kLDey3cYP-rkb9SYmx41js-2)

🔴 **Live notebook:** [Run it on Google Colab](https://colab.research.google.com/drive/1cJCMZcd2kLDey3cYP-rkb9SYmx41js-2): no setup needed.

A classical-NLP pipeline (no LLMs, no generated text) built as the final project for **Natural Language and Responsive AI (TEE7022)**. Every explanation is extracted word-for-word from real Sephora reviews.

---

## ✨ Example

```
Request     : hydrating and lightweight and good value   [category: Moisturizers]
Aspects     : hydration, texture, value
Recommended : Vitamin Enriched Face Base Priming Moisturizer (Bobbi Brown), $66, avg rating 4.6

Why:
  - hydration: "...it moisturize and primes my face perfectly, leaves skin nourished and plump..."
      90 mentions, 97% positive
  - texture:   "This priming moisturizer feels so amazing on my skin, is lightweight..."
      83 mentions, 96% positive
  - value:     "...it's worth it!"
      24 mentions, 92% positive

Runners-up  : GinZing Ultra-Hydrating Energy-Boosting Cream | The True Cream Moisturizing Bomb
```

---

## 🧠 How it works

```
Request ──► tokenise + lemmatise ──► detect aspects ──► rank products ──► quote evidence
                                          ▲                   ▲
Reviews ──► clean ──► split sentences ──► aspects + sentiment per sentence
```

The notebook is built phase by phase, one NLP concept per phase:

| Phase | Step | Technique |
|---|---|---|
| 0 | Load & join products + reviews | pandas |
| 1 | Clean raw text (HTML, links, `soooo`, `!!!!`) | Regular expressions |
| 2 | Sentence segmentation & tokenisation (with contraction expansion) | Regex |
| 3 | Stop-word removal — **keeps negations** (*not, no, never*) | sklearn stop list |
| 4 | Stemming vs lemmatisation | Snowball, simplemma |
| 5 | POS tagging → extract opinion adjectives | TextBlob |
| 6 | Uni/bi/trigrams + add-one (Laplace) smoothing | NLTK |
| 7 | Bag of words → choose lexicon keywords | CountVectorizer |
| 8 | Aspect lexicon (10 aspects) | Lexical semantics |
| 9 | Sentence sentiment (VADER + 25% star rating + negated-problem rule) | VADER |
| 10 | Rank products by requested aspects (smoothed, min-evidence filter) | — |
| 11 | Evidence-backed explanation + honest caveat | — |
| 12 | Evaluation | Spearman |
| 13 | Live interactive demo | — |

**Supported aspects:** hydration · texture · scent · longevity · value · packaging · irritation · effectiveness · absorption · coverage

### Notable design choices
- **Negation-aware:** *"no residue"*, *"did not irritate"* are scored as praise, not complaints.
- **Word-sense caution:** *dry* is excluded from the lexicon (skin type vs. complaint is ambiguous).
- **Evidence threshold:** a product is only ranked if it has ≥ 3 review sentences for *every* requested aspect.
- **Smoothed scores:** `aspect score = Σ(score × weight) / (Σweight + K)` so one lucky review can't win.

---

## 📊 Evaluation

| Check | Result |
|---|---|
| **Faithfulness** — quotes found verbatim in source review | **9 / 9** |
| **Sentiment sanity** — mean sentence score by stars | rises monotonically: −0.24 (1★) → +0.50 (5★) |
| **Stability** — split-half rank correlation | scent 0.75, texture 0.59, absorption 0.59, hydration 0.38 … packaging 0.14 |
| **Quality** — pick's avg rating vs category avg | above average in all 4 test requests (e.g. 4.61 vs 4.26) |

---

## 🚀 Getting started

### Quickest: run on Colab
1. Open the **[live Colab notebook](https://colab.research.google.com/drive/1cJCMZcd2kLDey3cYP-rkb9SYmx41js-2)**
2. **Runtime → Run all**
3. Upload `product_info.csv` and `reviews_0-250.csv` when prompted (see dataset link below)
4. Type your request in the Phase 13 live demo cell

### Run locally

#### 1. Get the data
Download **[Sephora Products and Skincare Reviews](https://www.kaggle.com/datasets/nadyinky/sephora-products-and-skincare-reviews)** from Kaggle and place these next to the notebook:
- `product_info.csv`
- `reviews_0-250.csv`

(In Colab the notebook will prompt you to upload them, or you can uncomment the `kagglehub` cell.)

#### 2. Install dependencies
```bash
pip install vaderSentiment simplemma textblob scikit-learn pandas nltk scipy
```

#### 3. Run
Open `ShreyaJain_NLP_finalproject.ipynb` in Jupyter or Google Colab and run all cells. Use the Phase 13 cell to try your own request:

```python
recommend("good scent and long lasting", category="Face Wash & Cleansers")
```

### Config (Phase 0)
| Variable | Default | Meaning |
|---|---|---|
| `TOP_PRODUCTS` | 150 | Keep the most-reviewed products |
| `MAX_PER_PRODUCT` | 150 | Max reviews sampled per product |
| `MIN_CHARS` | 25 | Drop very short reviews |
| `K` | 3 | Score smoothing strength |
| `MIN_MENTIONS` | 3 | Evidence needed per aspect |

With defaults: **10,201 reviews → 43,590 sentences across 69 products.**

---

## ⚠️ Limitations
- Keyword lexicon can't disambiguate word senses beyond the manual exclusions.
- The lemmatiser is context-free (e.g. *worse* → *wrong*).
- Only requests that mention one of the 10 aspects are understood.
- Stability is weak for low-mention aspects like packaging and coverage.

---

## 👩‍💻 Author
**Shreya Jain** — Natural Language and Responsive AI (TEE7022), Continuous Assessment-IV
