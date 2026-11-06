# Decision Models as Retrieval Gates Against Injection and Pollution

**Authors:** Mahdi Ghadamyari, Reza Habibi
**Institution:** Not stated on the arXiv record
**Published:** November 5, 2026
**Link:** https://arxiv.org/abs/2611.08325
**Code:** not linked on the abstract page at time of writing

---

## Problem Statement

RAG pipelines retrieve text from the open web and hand it to a generator. That retrieved text can carry two distinct threats, usually conflated:

1. **Injection** — retrieved passages contain hidden instructions aimed at the model.
2. **Pollution** — pages are synthetic, written to be retrieved and to mislead the answer, with no instruction payload at all.

Most defenses address the first with sanitization and citation prompting, and address the second not at all. The paper asks a cheaper question: can a small **decision model** — one that answers typed questions about a passage instead of generating tokens — act as a gate that screens retrieved content before the generator ever sees it?

## Methodology

**The gate.** Released open-weight decision models (Kev-4B) that answer typed questions about a text rather than producing free-form output. Screening is a classification decision, not a generation — which is where the latency comes from.

**Two testbeds:**

1. **Injection benchmark:** 1,000 questions over forum-style web pages with hidden injected instructions. Compared: no defense, published mitigations (sanitization + normalization + citation prompting), and published mitigations *plus* the gate.
2. **Pollution testbed:** 200 queries where **two thirds** of candidate pages are fake. Compares filtering the whole candidate pool *before* top-10 selection against filtering only the top 10 after retrieval.

**Two ablations worth noting:** gate input modality (raw HTML vs extracted text) and large-LLM judges as a cost/quality baseline.

## Key Results

**Injection:**
- No defense: **27.9%** attack success
- Published mitigations: **13.6%**
- Mitigations + gate: **5.7%**
- Cost: **76 ms per passage**
- False positives: **4 of 186** benign pages (pages with instructions *for human readers* — a genuinely hard negative class)
- Clean-query responsiveness essentially unchanged: **85.6% → 86.2%**

**Pollution:**
- Filter the pool before top-10 selection: **0.5%** fake-page exposure, accuracy held at **83%**
- No filter: **28%** exposure
- Filter only the top 10: the generator is left with too few pages to answer — the filter must run *before* ranking, not after

**Judge comparison:**
- Large LLMs as judges rank attacks better (**AUROC 0.99** vs **0.94**) but cost **10x** more
- Gates catch more attacks when they read the page's **HTML** than when they read extracted text

## What's Novel

1. **Decision models as a security control.** Using a non-generative classifier as the gate is the architectural point: it cannot be instructed, because it does not follow instructions — it answers a typed question. That structurally removes the "the defense model got injected too" failure mode that plagues LLM-judge defenses.
2. **Separating injection from pollution, then showing one gate handles both.** Pollution is under-studied and, in practice, more common than hand-crafted injection on the open web.
3. **The placement result.** Filtering before ranking versus after ranking is a pipeline-architecture decision that most RAG stacks get wrong, and the paper quantifies the cost of getting it wrong (0.5% vs unfilterable).

## My Connection

This is directly implementable in any retrieval-backed agent Manny builds or tests.

Three concrete takeaways:
- **Gate the pool, not the top-k.** If you filter after ranking, you either lose recall (too few pages) or you inherit the ranker's blind spots. Filtering before selection is cheaper and strictly better here.
- **76 ms per passage** is the number to quote when someone argues defense latency is prohibitive. For a 20-passage retrieval that's ~1.5 s — real, but not disqualifying for most agent workloads.
- **Read the HTML, not the text.** The gate performs better on HTML than on extracted text, which suggests hidden-instruction signals (CSS-hidden divs, comments, attribute text) survive in markup and are destroyed by extraction. If you sanitize by extracting text first, you have already thrown away the evidence.

The 4/186 false-positive rate is also worth noting as a realistic floor: pages that legitimately contain instructions for humans (forum posts, docs) are the adversarial-negative class that will drive operator complaints.

## What I Learned

The cheapest effective prompt-injection defense may be a model that doesn't write. A 4B decision model answering typed yes/no questions about a passage cut attack success from 27.9% to 5.7% in 76 milliseconds, for a tenth the cost of a frontier judge — and because it can't follow instructions, it can't be talked out of its verdict. The second lesson is about pipeline order: filter candidates before you rank them, because a filter applied to the top ten has already lost. And read the raw markup, since that's where the hidden instructions live.
