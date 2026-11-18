# Lab 2: "Research or Scale?" Responsible AI when research is folded into product

Students analyze what happens to responsible AI work when a research group moves into a product organization focused on scaling. They then test experimentally whether scale alone closes subgroup performance gaps, or whether research methods do. A second thread uses the November 2026 Outlook outage to ask whether staff cuts weaken a company's ability to respond when systems fail.

**Microsoft principles:** Reliability & Safety (primary), Accountability, Transparency, Inclusiveness
**IBM pillars:** Fairness, Explainability, Robustness
**IBM tools:** AI Fairness 360, AI Explainability 360

## Case summary

- **Early Sept 2026:** Microsoft moves its roughly 50-person AI Frontiers group out of Microsoft Research and into Microsoft AI.
- **Nov 6, 2026:** Co-founder John Langford writes on his blog [hunch.net](https://hunch.net) that the group is over and that he has left. He describes Microsoft AI's philosophy as "scaling, and aggressively not research."
- **AutoGen**, the group's widely used agent framework, entered maintenance mode in October 2025 and was archived in October 2026, with no further security fixes.
- **Nov 12–14, 2026:** A global Microsoft 365 outage takes down Outlook and other services for more than 27 hours, while Microsoft is in the middle of large layoffs and buyouts. See the case extension below.

> **Note for students:** AI Frontiers was a frontier research group, not an AI ethics team. Part of this lab is working out where responsible AI work actually lived and what the reorganization put at risk. For a direct ethics-team comparison, see Microsoft's 2023 layoff of its Ethics & Society team (optional reading).

**Counter-model: IBM.** At IBM, each Pillar of Trust is its own research initiative inside IBM Research, connected to the company-wide AI Ethics Board ([CEDA](https://ceda.com.au/newsandresources/opinion/economy/operationalising-responsible-ai)). The toolkits used in this lab (AIF360, AIX360, ART) came out of that research and were later donated to the Linux Foundation. Keep that structure in mind as the contrast throughout the lab.

## Case extension: the November 2026 Outlook outage

**What happened.** Starting around midday ET on Thursday, Nov 12, Outlook, Teams and other Microsoft 365 services failed worldwide. Downdetector reports for Outlook peaked at just over 130,000, and reports came from 74 countries within about an hour. Businesses, governments, schools and healthcare providers were affected. Most business customers had email back by Friday morning. Microsoft said on Saturday, Nov 14, that service had been restored across products. Microsoft said there was no sign of a cyberattack.

**Microsoft's explanation (preliminary).** Microsoft has given two descriptions, and students should compare them:

- An issue from an earlier network capacity upgrade surfaced after a recent network change. Servers failed automated health checks and dropped out of rotation, the remaining servers took on their load until they were overwhelmed too, and clients kept retrying. Servers brought back online were knocked over again by the waiting traffic, a pattern known as metastable failure.
- Charles Lamanna, EVP of Copilot, Agents and Platform, described it as a "latent defect" in a software change that had run worldwide for more than three months without problems before it overwhelmed servers under certain load.

At 10:13 p.m. on the first night, Microsoft said its fixes were not working as expected. Recovery finally came by restarting infrastructure region by region and letting traffic back in slowly. Microsoft has promised a full post-incident report, which had not been published when this lab was written.

**The staffing question.** The outage came in the middle of large workforce cuts: about 9,000 jobs in July 2025, about 4,800 in July 2026, buyout offers to about 9,000 US employees in April 2026, and buyout offers to about 60,000 employees (roughly a quarter of the workforce) in late October 2026. Many social media posts and commentators argued that Microsoft no longer had the right people in place to respond, and that this made the outage worse. Fast Company argued that even when layoffs don't cause a failure, the loss of institutional knowledge can make problems harder to solve. A Bloomberg Opinion column made the same point more sharply. A member of Congress asked for a briefing.

> **Note for students: separate evidence from inference.** No source so far shows that missing staff caused or prolonged this outage, and Microsoft has not commented on staffing. The October buyouts reportedly take effect Dec 31, so those employees were still working during the outage. The layoffs were mostly in sales and Xbox. The claim is a plausible hypothesis, not a finding. Part D asks you to say what evidence would confirm or refute it.

**Why it belongs in this lab.** The AI Frontiers story is about what happens when a company drops the people who do slow, careful work in favor of scale. The outage asks the same question about reliability engineering: who knows how an old, complex system behaves under stress, and are they still there when it breaks? Reliability & Safety is a responsible AI principle, and it depends on people as much as on models.

## Learning objectives

1. Identify who in an organization carries out each responsible AI principle.
2. Assess the safety risks of depending on abandoned open-source AI software.
3. Show empirically that model scale and subgroup fairness are separate problems, using IBM's fairness and explainability toolkits.
4. Evaluate claims that staff reductions weaken incident response, separating evidence from inference.

## Part A: Who owns the principles? (individual, 20 min)

For each of Microsoft's six principles and IBM's five pillars, name the role that usually carries it out (research, product, governance office, security team). Mark which roles the September reorganization weakened or removed. Then answer: could Microsoft AI have kept the work going the way IBM does, with research initiatives that report into a governance board?

Add one row for **incident response**: who carries out Reliability & Safety at 1 a.m. when a global service fails (site reliability engineers, on-call service owners, incident commanders, communications)? Which of these roles depend most on institutional knowledge that is hard to replace?

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

## Part D: Did cuts make the outage worse? (pairs, 30 min)

Use the Outlook outage timeline and sources.

1. **Map the failure.** Draw the feedback loop: network change → servers fail health checks → load shifts → survivors overwhelmed → client retries → returning servers fail again. Mark each point where a person had to make a decision, and each point where automation acted without one.
2. **Evidence vs. inference.** Collect three posts or articles claiming that Microsoft lacked the right people. For each, label the claim as *evidence* (a named source with direct knowledge), *inference* (reasoning from layoff numbers), or *speculation*. What would the post-incident report need to show to confirm or refute the staffing claim? Consider time to detect, time to correct diagnosis, failed fix attempts, which teams were paged and whether runbooks existed.
3. **Knowledge risk.** Microsoft attributed part of the problem to a change that ran for three months before failing. Which kinds of knowledge (how an older system behaves, why a health-check threshold is set where it is, how to bring a region back safely) live mainly in people's heads? Propose two controls that protect that knowledge through a reorganization, such as runbooks rehearsed in game days, ownership maps reviewed before layoffs, retention for key on-call roles, or limits on how much capacity automation can remove at once.
4. **Connect to Part C.** Microsoft AI's philosophy is "scaling, and aggressively not research." Is there a parallel between relying on scale to fix fairness gaps and relying on automation to replace experienced operators? Where does the analogy break down?

## Deliverable: Memo to a CTO (individual, 1 page)

Recommend where responsible AI review should sit after a reorganization like this one: in research, embedded in product, or as an independent function like IBM's AI Ethics Board. Use evidence from the case and from your own Part C results. Include one paragraph on how the company should protect incident response capability (people, knowledge and runbooks) during workforce reductions, using the Outlook outage as your example and being clear about what is known versus suspected.

## Sources

- [FourWeekMBA: Langford exits; AI Frontiers is over](https://fourweekmba.com/ai-langford-exits-microsoft-its-50-person-ai-frontiers-is-over/)
- [TechBytes: The end of Microsoft Research's AI Frontiers group](https://techbytes.app/posts/the-end-of-microsoft-researchs-ai-frontiers-group/)
- [Winzheng: AI Frontiers and AutoGen archiving](https://www.winzheng.com/en/article/microsoft-ai-frontiers-disbanded-research-vs-scaling)
- [NPR: Microsoft scrambles to fix email outage](https://www.npr.org/2026/11/12/nx-s1-6039343/is-outlook-down-widespread-outage-hits-microsoft-email-and-other-apps)
- [CNBC: Microsoft apologizes for massive outage](https://www.cnbc.com/2026/11/12/microsoft-outage-outlook-aws-google.html)
- [Al Jazeera: Microsoft hit by global outages](https://www.aljazeera.com/news/2026/11/13/microsoft-hit-by-global-outages-what-happened-and-why)
- [ABC News: Microsoft identifies issues leading to global Outlook outage](https://abcnews.com/US/outlook-outage-cause-microsoft-reason-teams-down/story?id=138174626)
- [Fox Business: Hidden bug behind the outage](https://www.foxbusiness.com/technology/microsoft-reveals-hidden-bug-behind-global-outlook-microsoft-365-outage)
- [HackerNoon: Why the outage lasted more than 27 hours](https://hackernoon.com/why-microsofts-outlook-outage-lasted-more-than-27-hours)
- [Fast Company: Outage collides with layoffs and buyouts](https://www.fastcompany.com/91641977/microsoft-outlook-outage-bad-optics-with-layoffs-buyouts-updates)
- [OpenTools: Outage puts AI-driven layoffs under scrutiny](https://opentools.ai/news/microsoft-365-outage-ai-layoffs-scrutiny)
- [Crimson Umbrella: Root cause, layoffs, AI and lessons](https://crimsonumbrella.com/insights/microsoft-365-outage-november-2026)
- [CEDA: Operationalising responsible AI (IBM governance)](https://ceda.com.au/newsandresources/opinion/economy/operationalising-responsible-ai)
- [AI Fairness 360](https://github.com/Trusted-AI/AIF360) · [AI Explainability 360](https://github.com/Trusted-AI/AIX360)
