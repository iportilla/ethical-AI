# Lab 3: Gated models and the collapsing patch window

Students weigh who should get frontier cyber-capable models first (security researchers, governments or the public) in a threat landscape where AI turns published patches into exploits within hours. **The technical work is strictly defensive:** it helps overloaded security teams triage.

**Microsoft principles:** Privacy & Security (primary), Reliability & Safety, Accountability, Transparency, Fairness
**IBM pillars:** Robustness, Explainability, Privacy
**IBM tools:** AI Explainability 360, Adversarial Robustness Toolbox, IBM zero trust guidance

> **Ground rule:** no lab activity involves writing, running or requesting exploit code. Part B uses only public vulnerability metadata.

## Case summary

- **Researchers-first release.** In April 2026 Anthropic announced Claude Mythos Preview and withheld it from the public. Access went to a limited group of cybersecurity vendors and technology companies so defenders could find and fix vulnerabilities first.
- **Government gating.** A June 2026 U.S. executive order set up a pre-release review framework, described as voluntary, that gives the government access for up to 30 days before release. At the administration's request, OpenAI limited GPT-5.6's initial release to government-approved partners. The Commerce Department briefly restricted Anthropic's Mythos and Fable models in June before approving a limited release.
- **The patch gap.** Attackers "patch diff" a security update to locate the bug it fixes. In June 2026 Anthropic's red team reported that Mythos Preview built 8 working exploits from 18 recent Firefox patches, and that public models with safeguards turned off could also build some. The Zero Day Clock estimates 2026's average time-to-exploit at under 24 hours, down from about 53 days in 2024. Verizon's 2026 DBIR puts the median time to fix a known-exploited flaw at 43 days.
- **Open weights and cheap hardware** (claims for students to evaluate). Open-weight models run locally on consumer GPUs, with no usage monitoring and with safeguards that can be removed. That limits how much access controls on closed models can achieve.

## Learning objectives

1. Analyze the dual-use tradeoffs of staged, gated and open model releases.
2. Build a defensive model that prioritizes vulnerabilities, and make its decisions explainable and robust.
3. Evaluate who bears the costs when access is restricted, including small defenders.

## Part A: Release-policy stakeholder map (individual, 25 min)

Compare four release regimes: (1) public release, (2) researchers first, (3) government-approved partners first, (4) open weights. For each one, identify who benefits, who is put at risk and who decides. Rate each regime against Microsoft's six principles and IBM's five pillars (+, 0, –), with one line of justification for each rating.

**Fairness prompt:** a hospital IT team of three, or a rural utility, is unlikely to be on an approved-partner list. What do they get under regime 3?

## Part B: Defensive triage model (pairs, 60 min)

Security teams can't patch everything on day zero, so they need to know which vulnerabilities attackers will actually use.

1. Download the public [NVD](https://nvd.nist.gov/vuln/data-feeds) CVE feed for 2024–2026 and the [CISA Known Exploited Vulnerabilities (KEV)](https://www.cisa.gov/known-exploited-vulnerabilities-catalog) catalog.
2. Label each CVE `1` if it appears in KEV and `0` otherwise. Note the heavy class imbalance.
3. Fine-tune `distilbert-base-uncased` (or the security-specific `jackaduma/SecBERT`) on CVE description text, using class weights or focal loss.
4. Report PR-AUC and recall in the top 5% of predictions; ROC-AUC alone is misleading here. Compare against a CVSS-score-only baseline and against [FIRST EPSS](https://www.first.org/epss/) scores.
5. **Fairness audit.** Which vendors or product categories does the model under-prioritize? Who is hurt if a triage model systematically ranks one ecosystem's vulnerabilities too low?

### Part B2: Explain the ranking (AIX360, 15 min)

An analyst won't act on a black-box score at 2 a.m. Use AIX360's LIME wrapper (`aix360.algorithms.lime.LimeTextExplainer`) to explain the model's top 10 predictions and its 5 worst misses (KEV entries it ranked low).

- Which tokens drive "patch this first"? Are they real signals (`remote code execution`, `unauthenticated`) or artifacts (vendor names, CVE-year strings)?
- Rewrite one explanation as a single sentence an on-call analyst could use. This is IBM's Explainability pillar in practice.

### Part B3: Is the triage model robust? (15 min)

Advisory wording varies by vendor, and a vendor could soften language to look less urgent. Paraphrase 30 high-risk descriptions (synonym swaps, passive voice, removing severity adjectives) and measure how far each drops in the ranking. Report the share that fall out of the top 5%.

**Grad:** wrap your model as an ART `PyTorchClassifier` operating on the sentence embedding (input = pooled embedding, output = KEV logit). Run `FastGradientMethod` at increasing `eps` and plot recall in the top 5% against `eps`. Then apply ART's adversarial-training defense (`AdversarialTrainer`) and report the robustness-versus-accuracy tradeoff. Discuss what an embedding-space perturbation does and doesn't tell you about real-world wording changes.

## Part C: Debate (whole class, 25 min)

Motion: *"Frontier labs, not governments, should decide which security researchers get early access to cyber-capable models."*

Assign teams. Each side must address:

- open-weight models on cheap hardware, which neither labs nor governments control;
- what defenders should do architecturally when patching can't keep up. Bring one idea from IBM's [zero trust guidance](https://www.ibm.com/solutions/zero-trust), such as least privilege, micro-segmentation or assume-breach.

## Deliverable: Policy brief (individual, 1 page)

Recommend a release policy for the next frontier model with strong cyber capabilities. State who gets access, when, who approves it and how small defenders are protected. Use your Part B results to argue what defenders actually need in the patch window: a better ranking, an explanation, or an architecture that tolerates an unpatched system.

## Sources

- [Anthropic Frontier Red Team: Measuring LLMs' impact on N-day exploits](https://anthropic.com/research/n-days)
- [The Hacker News: N-day is becoming N-hour](https://thehackernews.com/2026/07/n-day-is-becoming-n-hour-patching.html?m=0)
- [Herbert Smith Freehills Kramer: License to model](https://www.hsfkramer.com/insights/2026-07/license-to-model-emerging-us-rules-impact-global-access-to-frontier-ai)
- [R&D World: Washington's 30-day look at frontier AI](https://www.rdworldonline.com/why-washington-wants-a-30-day-look-at-frontier-ai-before-it-ships-and-is-backing-a-voluntary-approach/)
- [AP via ADN: OpenAI and Anthropic limit new models to approved customers](https://www.adn.com/nation-world/2026/06/26/openai-and-anthropic-limit-new-ai-models-to-trump-approved-customers-during-cybersecurity-review/)
- [Cloud Security Alliance: Government-gated AI and GPT-5.6](https://labs.cloudsecurityalliance.org/research/csa-research-note-frontier-ai-dual-use-cybersecurity-gpt56-2/)
- [AI Explainability 360](https://github.com/Trusted-AI/AIX360) · [Adversarial Robustness Toolbox](https://github.com/Trusted-AI/adversarial-robustness-toolbox) · [IBM zero trust](https://www.ibm.com/solutions/zero-trust)
