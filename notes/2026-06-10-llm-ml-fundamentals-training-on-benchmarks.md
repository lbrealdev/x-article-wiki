# Source

- Author: [@TheAhmadOsman](https://x.com/TheAhmadOsman)
- Article/Post: https://x.com/TheAhmadOsman/status/2064724789952958663
- Date: 2026-06-10
- Title: LLM/ML Fundamentals: Why Training on Benchmarks, Evals, or Test Sets Is a Cardinal Sin

## Long summary

Machine learning is meant to predict performance on the next unseen case, not prove that a model can repeat examples it has already seen. That is why benchmarks, evals, and test sets exist: they are independent measuring instruments for generalization. Once test items, scoring logic, examples, or answer patterns become public optimization targets, they stop being clean measurement and start to become a benchmaxxing utility. Training on that instrument breaks it and turns the score into a record of exposure, not capability.

The basic rule: train on training data; tune on validation data; measure once, honestly, on test data. Once you train, tune, prompt-optimize, scaffold-optimize, filter, or select against the test set, it is not a test set anymore. Classical ML already forbade fitting preprocessing on the full dataset before splitting, choosing features with test labels, or picking checkpoints from final test performance. The same logic applies to LLMs, with a much larger surface area. Training on benchmarks, evals, or test sets is a cardinal sin because it corrupts a trustworthy estimate of generalization.

The piece opens from a related exchange where the author refuses to publish the evals or data that make private model rankings useful: publishing them can destroy what makes them work. Screenshots illustrate a confidently wrong take about benchmark training, polished public recommendations (including a Nex-N2 hardware-tier example), and field reports where benchmark-shaped success fails on fresh, messy tasks. What to trust: clean protocol, fresh tasks, private holdouts, disclosed limits, and real behavior under conditions the model did not get to study and train on—not the prettiest ranking or a score turned into a target.

This post is framed as the measurement layer after the author’s five-part local AI series (GPU memory math, memory bandwidth, inference engines, LLMs 101, and step-by-step engineering projects via thelocalaibook.com). Benchmark names will age; the core point will not.

Modern ML assigns jobs to data: training fits parameters; validation/dev chooses hyperparameters, prompts, architectures, checkpoints, thresholds, and scaffolds (and is overfittable); test/eval/benchmark estimates generalization after decisions are frozen; private/hidden tests need protected examples, labels, and feedback; deployment monitoring tracks drift but is not a clean pre-deployment test. Low training loss is not the target—low expected loss on the deployment distribution is.

Vocabulary: fitting includes any information path that changes the system (weights, normalization, features, thresholds, prompts, tool routing, retrieval indexes, agent loops, rejection sampling, self-consistency, benchmark-specific postprocessing, model selection). “We did not update the weights” is not a defense if test results changed the system. Generalization requires an independent holdout. Leakage patterns include preprocessing, feature, duplicate, group, temporal, target, leaderboard, and benchmark contamination. Contamination is leakage at benchmark scale and, for LLMs, often a system problem—not only a dataset problem.

LLMs make contamination easier: massive mixed corpora; tokenization (BPE, WordPiece, Unigram) and Transformers at scale; Chinchilla-style compute-optimal scaling of tokens; and many post-pretraining stages (continued pretraining, SFT, RLHF/DPO, safety tuning, tools/RAG, prompts/scaffolds, distillation, synthetic data). Each stage can contaminate. Semantic contamination matters: rephrases, translations, and soft/semantic duplicates can inflate scores while evading exact string matching.

Benchmarks answer a narrow question under stated conditions; they do not alone prove general intelligence, safety, usefulness, or deployability. Static suites (MMLU across 57 subjects, BIG-bench, GPQA, ARC-style, HELM’s multi-metric push) age, saturate, and get optimized against—the 2026 AI Index highlighted reliability and gaming concerns, and a 2025 study tied saturation to age and scale. Harder refreshes (MMLU-Pro, GPQA, Humanity's Last Exam, ARC-AGI-style) help but keep the incentive problem. Dynamic evals (LiveBench, LiveCodeBench, LiveMedBench) and human preference arenas (Chatbot Arena) and LLM-as-judge setups each help in some settings and bring their own biases and failure modes.

Why training on test sets is a cardinal sin: it destroys the test’s reason for existing; turns science into self-deception (memorization/mimicry mistaken for capability); fuels a benchmark-hacking arms race; breaks comparability across teams and systems; hides real-world failure in high-stakes domains; weakens safety evals (e.g. refusing only public jailbreaks while MLCommons AILuminate points toward standardized safety testing); and remains invalidating even when accidental.

LLM-specific pathways include pretraining mirrors and Q&A scrapes; instruction-tuning and preference-data contamination; synthetic-data laundering; retrieval/RAG of benchmark answers in closed-book evals; prompt and scaffold overfitting; and code-eval risks around SWE-bench, SWE-bench Verified, and SWE-bench-Live (weak validation, repository-history exploits, future-state leakage).

Practical taxonomy: official training/dev splits, one frozen test evaluation, retired benchmarks no longer reported as clean, disclosed browsing when allowed—are fine when handled correctly. Looking at test examples for prompts, using test labels for checkpoints, including test Q&A/paraphrases/synthetic variants, preference training on test items, repeated hidden-leaderboard feedback, closed-book RAG over answer corpora, and continuing to claim a consumed test is fresh—are contamination risks or violations. Principle: any information path from test examples, labels, answer keys, scoring rules, hidden tests, or repeated feedback into model or system decisions is a contamination risk.

Old evals also fail via saturation, dataset aging, invalid/ambiguous items, and shortcut artifacts. Proper classical design: split before preprocessing; use time/group/repository-aware splits where needed; resample only inside training; use validation for decisions and test for measurement; nest validation when selection is heavy; report uncertainty; audit for leakage. LLM extras: define the evaluated object (base vs chat vs RAG vs agent vs product); freeze protocol before testing; maintain exclusion lists across all data stages; use layered decontamination (exact, n-gram, MinHash/SimHash, embeddings, translation-aware, code clone, canaries, black-box probes); prefer fresh/private/live tests for serious claims; separate development vs audit evals; do not rely on one benchmark family; evaluate robustness, not just peak score.

Clean practice reports enough protocol detail for interpretation and can honestly say the model and eval were frozen before the test, only permitted training data was used, validation drove development, contamination was checked across stages, hidden labels were not inspected, uncertainty and limitations are reported, and the team will not keep optimizing on that test while calling it fresh. Common anti-patterns (“everyone trains on them,” “questions only,” “benchmark-style synthetic,” “prompt-only,” “leaderboard feedback only,” “RAG found it online,” “public = fair game”) are rejected with the corresponding caveats. Training on official train splits, restrained use of development splits, inspecting a consumed final test, training on retired benchmarks without clean claims, and genuinely new format-inspired data are not sins—ordinary capability transfer is not contamination.

Benchmarks are public goods with a lifecycle (launch → adoption → optimization → saturation → contamination → retirement/refresh). A 2026-oriented hygiene checklist covers before/during/after evaluation controls, and ten habits: classical split discipline; LLM contamination control; freshness; breadth; depth; robustness; human grounding; safety/reliability; system transparency; lifecycle management. Treat benchmarks like scientific instruments: calibrated, protected, versioned, audited, and retired when worn out.

Final principle: training on benchmarks replaces “Can this system handle new cases?” with “Has this system absorbed the measuring instrument?” Respect the ruler—do not bend it to fit the model.

## Key claims

- Benchmarks, evals, and test sets exist to estimate generalization on unseen cases; training, tuning, prompt/scaffold optimization, filtering, or selection against them destroys that measurement.
- Core rule: train on training data, tune on validation data, measure once on untouched test data; once the test influences decisions, it is no longer a test set.
- Fitting is broader than weight updates: any path from test information into the system (including prompts, tools, retrieval, scaffolds, and model selection) counts as optimization against the test.
- Leakage and contamination (exact, near-duplicate, semantic, translation, distributional, procedural, feedback) bias scores upward and can be accidental yet still invalidating.
- LLM pipelines widen contamination paths across pretraining, continued pretraining, SFT, RLHF/DPO, synthetic data, RAG, prompts/scaffolds, distillation, and code-agent evals (including SWE-bench / SWE-bench Verified / SWE-bench-Live failure modes).
- Static public benchmarks (e.g. MMLU, BIG-bench, GPQA, ARC-style, HELM) age, saturate, and get gamed; harder refreshes and dynamic suites (LiveBench, LiveCodeBench, LiveMedBench) help but do not remove optimization incentives.
- Contaminated scores break comparability, mislead deployment/investment/research, hide real-world and safety failures, and reward benchmaxxing over capability.
- Clean evaluation freezes the protocol before testing, uses layered decontamination and exclusion lists, prefers fresh/private/live holdouts for serious claims, separates development from audit evals, reports uncertainty/limitations, and retires or refreshes consumed benchmarks.
- Official training/dev splits, one frozen test evaluation, retired benchmarks no longer claimed as clean, and disclosed retrieval-allowed evals are not sins when handled correctly.
- Benchmarks are public goods with a lifecycle; treat them as scientific instruments—calibrated, protected, versioned, audited, and retired when worn out.

## Actionables

- Keep training, validation, and test roles strict: never train, tune, prompt-optimize, scaffold-optimize, filter, or select using test/eval items you still want to report as clean measurement.
- Before serious claims, freeze model version, prompts, tools, retrieval corpora, decoding, sampling, extraction, and scoring; run layered contamination checks across pretraining, fine-tuning, preference, synthetic, and retrieval data.
- Prefer fresh, private, live, or post-cutoff tests (and private holdouts) for frontier or high-stakes claims; treat public static benchmarks as aging instruments that need refresh or retirement.
- Separate development smoke/canary evals from audit/release holdouts; after inspecting a test, mark it consumed and stop calling later scores on it clean.
- Report full protocol, uncertainty, limitations, and contamination risk; disclose retrieval/browsing when used; do not compare contaminated and clean systems as if the numbers mean the same thing.
- Use multiple benchmark families and robustness checks (prompt sensitivity, distribution shift, adversarial variants, variance) rather than a single peak score.

## References

### X source

- https://x.com/TheAhmadOsman/status/2064724789952958663
- https://x.com/TheAhmadOsman/status/2063834271564145071
- https://x.com/TheAhmadOsman/status/2040103488714068245
- https://x.com/TheAhmadOsman/status/2041331757329285589
- https://x.com/TheAhmadOsman/status/2057183854444843202
- https://x.com/TheAhmadOsman/status/2057590224729911346
- https://x.com/TheAhmadOsman/status/2058745340895870985

### Other

- http://thelocalaibook.com/
