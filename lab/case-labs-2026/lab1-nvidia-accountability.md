# Lab 1: "Bring Back Jensen?" Leadership accountability at an AI infrastructure company

Students decide whether they would sign a petition to reinstate Jensen Huang as Nvidia's CEO, and justify that decision with the Responsible AI principles. **Signing is never required.** The graded work is the reasoning.

**Microsoft principles:** Accountability (primary), Transparency, Privacy & Security, Reliability & Safety
**IBM pillars:** Transparency, Fairness
**IBM tools:** AI FactSheets, AI Fairness 360

## Case summary

- **Late Oct 2026:** The New York Times reports that Alice Huang, a former Megaspeed executive, cultivated ties with Huang on behalf of Chinese intelligence. Huang initially calls the relationship ordinary business.
- **Nov 8:** Huang admits an affair in a memo to employees. The board opens an independent investigation, run by independent directors with outside counsel, and says it has confidence in him.
- **Nov 9–11:** Nvidia shares fall. The Wall Street Journal reports two affairs and that the board is preparing to remove him.
- **Nov 12:** Nvidia announces Huang's resignation as President, CEO and director. CFO Colette Kress becomes interim CEO and Harvey C. Jones becomes the board's first chair.

**Case packet:** the board's Nov 8 statement, Huang's memo, the Nov 12 Form 8-K, and three articles with different framings (see Sources).

## Learning objectives

1. Map a real governance failure to both Microsoft's principles and IBM's Pillars of Trust.
2. Run, audit and document a pretrained NLP model on a contested, opinion-heavy dataset.
3. Write a position argued from principles and evidence that answers the strongest counterargument.

## Part A: Map the timeline to the principles (individual, 20 min)

Build a timeline of the case from the packet. Tag each event with one or more Microsoft principles **and** the closest IBM pillar, and write one sentence explaining the tag.

- Huang's initial public denial → Transparency (both frameworks).
- An investigation run by independent directors with outside counsel → Accountability (Microsoft); IBM has no matching pillar. What does IBM use instead?
- Possible exposure of Nvidia technology to a foreign intelligence service → Privacy & Security (Microsoft), Privacy and Robustness (IBM).

**Governance comparison.** IBM runs AI ethics through a central structure: a Privacy Advisory Committee, an AI Ethics Board, Focal Points in each business unit and an Advocacy Network ([CEDA](https://ceda.com.au/newsandresources/opinion/economy/operationalising-responsible-ai)). Sketch Nvidia's equivalent, based on the packet. Where does accountability run through a single person, and what would an IBM-style structure have changed?

## Part B: Stance detection on the petition debate (pairs, 45 min)

Data: `data/petition_posts.csv`. About 600 public posts arguing for or against reinstatement, with columns `text`, `gold_stance`, `framing` (`financial` / `ethical` / `security`) and `lang` (`en` / `other`).

1. Load `facebook/bart-large-mnli` with the Hugging Face `zero-shot-classification` pipeline.
2. Classify each post as `supports reinstatement`, `opposes reinstatement` or `neutral`.
3. Report accuracy, macro-F1 and a confusion matrix against `gold_stance`.
4. **Error audit.** Pull 20 misclassified posts and categorize the failures: sarcasm, quoted opposing views, moral vs. financial framing, non-English or code-switched text.
5. **Prompt sensitivity.** Re-run with two alternative label phrasings (for example `wants Huang back as CEO`). How much do the results move? What does that say about using zero-shot labels as evidence of public opinion?

### Part B2: Is the classifier fair across framings? (AIF360, 15 min)

Treat `framing` as the group attribute (privileged = `financial`) and "classified correctly" as the favorable outcome.

```python
from aif360.datasets import BinaryLabelDataset
from aif360.metrics import ClassificationMetric
# Build gold and predicted BinaryLabelDatasets (supports = 1, otherwise 0),
# then compare groups:
#   metric.equal_opportunity_difference()
#   metric.disparate_impact()
```

If the model systematically misreads ethically framed posts, then any "public opinion" summary built on it is biased. Who would that help or hurt in a reinstatement debate?

### Part B3: Write an AI FactSheet (15 min)

Document your classifier using IBM's [AI FactSheets](https://www.ibm.com/docs/en/software-hub/5.2.x?topic=services-ai-factsheets) fields (a markdown template is fine if you don't have watsonx.governance access):

- **Purpose and criticality:** what decisions should and should *not* rest on this model?
- **Measured characteristics:** your Part B and B2 metrics, including the worst-performing group.
- **Lineage:** model version, label phrasing, data snapshot date, who ran it.

**Grad:** fine-tune `distilroberta-base` on 70% of the gold labels and compare it to zero-shot on the held-out 30%. Re-run the AIF360 metrics. Does fine-tuning shrink or widen the framing gap?

## Part C: Key-person risk (group discussion, 20 min)

- Should a company that supplies most of the world's AI compute hold its CEO to a different standard than an ordinary company?
- The board first expressed confidence, then accepted a resignation four days later. Was that accountability working or failing?
- If a sentiment model shows most online posts support reinstatement, should the board care? What would your FactSheet tell them about trusting that number?

## Deliverable: Position memo (individual, 1 page)

Would you sign the petition? Argue from at least three principles (Microsoft or IBM), use evidence from the packet and from your Part B results, and answer the strongest counterargument.

## Sources

- [NVIDIA Form 8-K: CEO transition (Nov 12, 2026)](https://www.sec.gov/Archives/edgar/data/0001045810/000104581026000082/nvidiaannouncesceotransiti.htm)
- [Cointelegraph: Nvidia preparing to oust Huang after affairs, per WSJ](https://cointelegraph.com/news/nvidia-preparing-to-oust-ceo-jensen-huang-reports?rand=50490)
- [Fortune: Why Nvidia's board may rein in Huang rather than fire him](https://fortune.com/2026/11/11/nvidia-board-directors-jensen-huang-captive-board/)
- [KeenGamer: board statement and Huang's apology email](https://www.keengamer.com/articles/news/nvidia-board-opens-investigation-into-ceo-jensen-huang-after-affair-admission/)
- [Axios: Should Jensen Huang be fired?](https://www.axios.com/2026/11/09/jensen-huang-nvidia-ceo-chinese-spy)
- [CEDA: Operationalising responsible AI (IBM governance)](https://ceda.com.au/newsandresources/opinion/economy/operationalising-responsible-ai)
