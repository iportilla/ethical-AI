# Lab 2: "Research or Scale?" Responsible AI when research is folded into product

Students analyze what happens to responsible AI work when a research group moves into a product organization focused on scaling. They then test experimentally whether scale alone closes subgroup performance gaps, or whether research methods do.

**Microsoft principles:** Reliability & Safety (primary), Accountability, Transparency, Inclusiveness
**IBM pillars:** Fairness, Explainability, Robustness
**IBM tools:** AI Fairness 360, AI Explainability 360

## Case summary

- **Early Sept 2026:** Microsoft moves its roughly 50-person AI Frontiers group out of Microsoft Research and into Microsoft AI.
- **Nov 6, 2026:** Co-founder John Langford writes on his blog [hunch.net](https://hunch.net) that the group is over and that he has left. He describes Microsoft AI's philosophy as "scaling, and aggressively not research."
- **AutoGen**, the group's widely used agent framework, entered maintenance mode in October 2025 and was archived in October 2026, with no further security fixes.

> **Note for students:** AI Frontiers was a frontier research group, not an AI ethics team. Part of this lab is working out where responsible AI work actually lived and what the reorganization put at risk. For a direct ethics-team comparison, see Microsoft's 2023 layoff of its Ethics & Society team (optional reading).

**Counter-model: IBM.** At IBM, each Pillar of Trust is its own research initiative inside IBM Research, connected to the company-wide AI Ethics Board ([CEDA](https://ceda.com.au/newsandresources/opinion/economy/operationalising-responsible-ai)). The toolkits used in this lab (AIF360, AIX360, ART) came out of that research and were later donated to the Linux Foundation. Keep that structure in mind as the contrast throughout the lab.

## Learning objectives

1. Identify who in an organization carries out each responsible AI principle.
2. Assess the safety risks of depending on abandoned open-source AI software.
3. Show empirically that model scale and subgroup fairness are separate problems, using IBM's fairness and explainability toolkits.

## Part A: Who owns the principles? (individual, 20 min)

For each of Microsoft's six principles and IBM's five pillars, name the role that usually carries it out (research, product, governance office, security team). Mark which roles the September reorganization weakened or removed. Then answer: could Microsoft AI have kept the work going the way IBM does, with research initiatives that report into a governance board?

## Part B: Orphaned code (pairs, 30 min)

Inspect the archived AutoGen repository.

1. List the security-relevant issues and unmerged pull requests that were open when it was archived.
2. Run `pip-audit` on a minimal project that depends on AutoGen and record any flagged dependencies.
3. Compare with how IBM handled its toolkits: AIF360, AIX360 and ART moved to the LF AI & Data Foundation with open governance instead of being archived. Write a five-step responsible-deprecation policy for an AI framework (notice period, migration path, security handoff, foundation transfer and so on).

## Part C: What does "scaling only" miss? (pairs, 45 min)

Use the CelebA dataset to predict the `Smiling` attribute, with `Male` and `Young` as subgroup attributes.

1. Train three CNNs of increasing size (ResNet-18, ResNet-34, ResNet-50) for the same number of epochs.
2. For each model, log overall accuracy, accuracy for each of the four subgroups, the worst-group gap and estimated energy use with `codecarbon`.
3. Compute AIF360 group metrics for each model on the test set:

   ```python
   from aif360.datasets import BinaryLabelDataset
   from aif360.metrics import ClassificationMetric
   metric = ClassificationMetric(gold, pred,
                                 unprivileged_groups=[{'Young': 0}],
                                 privileged_groups=[{'Young': 1}])
   metric.equal_opportunity_difference(), metric.average_odds_difference()
   ```

4. Plot worst-group accuracy and equal-opportunity difference against parameter count and against kWh.
5. **Research intervention (AIF360 Reweighing).** Run AIF360's `Reweighing` preprocessor on a table of each image's label and protected attributes. Its output `instance_weights` then drive a PyTorch `WeightedRandomSampler` for the *smallest* model:

   ```python
   from aif360.algorithms.preprocessing import Reweighing
   rw = Reweighing(unprivileged_groups=[{'Young': 0}], privileged_groups=[{'Young': 1}])
   weights = rw.fit_transform(attr_dataset).instance_weights
   ```

   Compare its worst-group gap and energy use with the largest model's.

**Guiding question:** did adding parameters close the gap, or did a research choice do more for less energy?

### Part C2: Why does the worst group fail? (AIX360, 15 min)

Pull penultimate-layer embeddings from your best model. Use AIX360's `ProtodashExplainer` (`from aix360.algorithms.protodash import ProtodashExplainer`) to find the five most representative training images for the worst-performing subgroup, and five for the best. What do the prototypes suggest about where the gap comes from: missing data, label noise or the model?

**Grad:** add Group DRO ([Sagawa et al., 2020](https://arxiv.org/abs/1911.08731)) as a third intervention, and compare it with AIF360's in-processing `ExponentiatedGradientReduction` (equalized-odds constraint), fit on the frozen embeddings with a logistic-regression head. Discuss when each method is appropriate.

## Deliverable: Memo to a CTO (individual, 1 page)

Recommend where responsible AI review should sit after a reorganization like this one: in research, embedded in product, or as an independent function like IBM's AI Ethics Board. Use evidence from the case and from your own Part C results.

## Sources

- [FourWeekMBA: Langford exits; AI Frontiers is over](https://fourweekmba.com/ai-langford-exits-microsoft-its-50-person-ai-frontiers-is-over/)
- [TechBytes: The end of Microsoft Research's AI Frontiers group](https://techbytes.app/posts/the-end-of-microsoft-researchs-ai-frontiers-group/)
- [Winzheng: AI Frontiers and AutoGen archiving](https://www.winzheng.com/en/article/microsoft-ai-frontiers-disbanded-research-vs-scaling)
- [CEDA: Operationalising responsible AI (IBM governance)](https://ceda.com.au/newsandresources/opinion/economy/operationalising-responsible-ai)
- [AI Fairness 360](https://github.com/Trusted-AI/AIF360) · [AI Explainability 360](https://github.com/Trusted-AI/AIX360)
