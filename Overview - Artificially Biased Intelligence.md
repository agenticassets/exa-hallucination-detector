## Paper overview for `Artificially Biased Intelligence.md`

**Question.** Do large language models (LLMs) exhibit *human-like cognitive biases* when making structured financial judgments, and can those biases be mitigated at deployment time?

**Approach.** The authors run a **prompt-pair experiment**: for each bias, they create **25 control/treatment prompt pairs** that are economically equivalent but differ by one “bias trigger” (e.g., gain vs. loss framing, irrelevant numerical anchors, authority cues). They field these across **48 LLMs**, enforce **numeric outputs** (mostly 1–10 scales; anchoring uses dollar valuations), and estimate within-pair treatment effects with prompt-pair fixed effects and inference clustered by model.

## Biases tested

They test **11 bias families** grouped into:

* **Information presentation:** framing, anchoring (ambiguous & unambiguous).
* **Social influence & narrative heuristics:** sycophancy, herding, authority, representativeness, availability.
* **Reference/position dependence:** endowment, sunk cost, loss aversion, disposition setting.

## Main findings (magnitudes are economically meaningful)

Across models, LLMs show **systematic, nontrivial biases**:

### 1) Information presentation biases

* **Framing is largest on average:** **+1.62 points** on a 1–10 scale (control mean 3.64 → treatment 5.25).
* **Anchoring distorts valuations even when the “right” calculation is pinned down:**

  * Ambiguous inputs: **+$20.11** (≈16.6% of control mean).
  * Unambiguous inputs: **+$7.34** (≈6.5% of control mean).

### 2) Social influence and narrative salience

Large effects from cues that should be normatively irrelevant:

* **Sycophancy:** **+0.82**
* **Herding:** **+1.17**
* **Authority:** **+1.41**
* **Representativeness:** **+1.47** (base rates get swamped by “startup archetype” narrative)
* **Availability:** **+1.39** (vivid events inflate risk despite identical objective rates)

### 3) Reference/position dependence

* **Endowment:** **+0.59**
* **Sunk cost:** **+1.29** (notably, pushes toward continuing *negative-NPV* projects when prior spend is highlighted)
* **Loss aversion:** statistically significant but **small** on average (**−0.30**) and not the canonical “reflection effect” direction in this implementation.
* **Disposition setting:** **~0 on average** (0.03; not statistically different from zero), though later results show heterogeneity linked to intelligence.

## Heterogeneity: model choice is “governance,” not plumbing

* Bias susceptibility varies widely across models; the paper’s framing-bias plot shows large dispersion in average treatment effects by model.
* Biases do **not** collapse to one “behavioral robustness” factor: some biases correlate (e.g., framing with representativeness), others (anchoring) are more distinct.

## Intelligence has a non-monotone relationship with bias

Using benchmark “intelligence” measures, capability is **not a guarantee of debiasing**:

* Biases that **decline with intelligence:** **framing** and **representativeness** (more capable models are more invariant to some presentation/narrative manipulations).
* Biases that **increase with intelligence:** **sunk cost**, **loss aversion**, and **disposition-style asymmetries** (even though average disposition is near zero, higher-intelligence models tend to show more of it).

## “Investment style” tendencies: momentum vs value signatures

They run additional prompt-pair tests to see whether models overweight:

* **Recent performance (momentum cue):** **+2.42** rating shift.
* **Low valuation multiples (value cue):** **+2.34** shift.

Key pattern:

* **Less capable models look more like momentum investors** (more sensitive to recent returns and social/narrative cues).
* **More capable models look more like value investors** (greater emphasis on fundamentals/valuation).
* Value tendency is positively related to intelligence (reported correlation ≈ **0.51**).

## Mitigation: works, but only bias-by-bias

Two deployable interventions are tested:

1. **Context engineering** (explicit “be rational, ignore X bias” system messages):

* Often helps (e.g., framing reduced ~43%), and can nearly eliminate some (availability and herding in this setup).
* Can also **over-correct** or behave unpredictably (loss aversion flips sign).

2. **Prompt preprocessing** (LLM rewrites the prompt to remove triggers, then answers):

* Extremely effective for some (e.g., **framing reduced ~94%**, large reductions for authority/representativeness/availability; strong reduction for unambiguous anchoring).
* Weak or counterproductive for others (e.g., **herding remains large**; loss aversion becomes more negative).

**Bottom line:** “Debiasing” is an empirical validation problem—generic guardrails are insufficient.

## Contributions and implications

* Provides a **causal measurement framework** for LLM cognitive bias in financially framed tasks.
* Shows that LLMs can introduce **systematic distortions** large enough to flip thresholds, rankings, and valuations in realistic workflows.
* Establishes that **model selection implicitly sets factor/style tilts** (momentum vs value) in AI-assisted pipelines.
* Recommends **bias-specific evaluation and controls** before integrating LLMs into investment decision processes.

