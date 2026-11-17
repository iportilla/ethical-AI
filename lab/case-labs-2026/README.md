# Ethical AI Case Labs (Fall 2026)

**Course:** Applied Deep Learning (APPM 4350/5350), CU Boulder
**Unit:** Ethical and Responsible AI

Three labs that apply **Microsoft's Responsible AI principles** and **IBM's Pillars of Trust** to cases from fall 2026. Each lab pairs a case analysis with a hands-on deep learning exercise that uses IBM's open-source Trusted AI toolkits. Each lab ends with a short memo in which students take a position and defend it.

| Lab | Case | Primary principle | IBM toolkit | Time |
| --- | --- | --- | --- | --- |
| [Lab 1](lab1-nvidia-accountability.md) | Jensen Huang's resignation from Nvidia | Accountability / Transparency | AI FactSheets, AIF360 | ~2 hrs |
| [Lab 2](lab2-research-vs-scale.md) | Microsoft's AI Frontiers group folded into Microsoft AI | Reliability & Safety / Fairness | AIF360, AIX360 | ~2 hrs |
| [Lab 3](lab3-gated-models-patch-window.md) | Government-gated frontier models and the collapsing patch window | Privacy & Security / Robustness | AIX360, ART, zero trust | ~2.5 hrs |

## Two frameworks, one crosswalk

Microsoft has six principles. IBM's AI Ethics Board defines five Pillars of Trust and backs most of them with an open-source toolkit, now hosted by the [LF AI & Data Foundation](https://lfaidata.foundation/). Students should use both vocabularies. Where the frameworks disagree is itself a discussion point. Microsoft names **Inclusiveness** and **Accountability** as principles; IBM handles accountability through governance (its AI Ethics Board and Focal Points) rather than as a pillar.

| Microsoft principle | Closest IBM pillar | IBM tool students use |
| --- | --- | --- |
| Fairness | Fairness | [AI Fairness 360 (AIF360)](https://github.com/Trusted-AI/AIF360) |
| Reliability & Safety | Robustness | [Adversarial Robustness Toolbox (ART)](https://github.com/Trusted-AI/adversarial-robustness-toolbox) |
| Privacy & Security | Privacy, Robustness | ART (membership inference), [IBM zero trust](https://www.ibm.com/solutions/zero-trust) |
| Inclusiveness | Fairness (partly) | AIF360 subgroup metrics |
| Transparency | Transparency, Explainability | [AI Explainability 360 (AIX360)](https://github.com/Trusted-AI/AIX360), [AI FactSheets](https://www.ibm.com/docs/en/software-hub/5.2.x?topic=services-ai-factsheets) |
| Accountability | (governance, not a pillar) | AI FactSheets lineage; IBM AI Ethics Board model |

## Logistics

- **Environment:** Google Colab with a T4 GPU, Python 3.11+. Run `pip install -r requirements.txt`, then `pip install --no-deps xport`. The second step is needed for AIX360's ProtoDash; [requirements.txt](requirements.txt) explains why.
- **Teaming:** pairs for the technical parts, individual for memos.
- **Level split:** APPM 4350 completes the core tasks. APPM 5350 also completes the extension marked **Grad** in each lab.

### Grading (each lab, 100 pts)

| Component | Points |
| --- | --- |
| Principle mapping (Microsoft and IBM) | 20 |
| Technical work and audit | 35 |
| Discussion participation | 10 |
| Position memo | 35 |

Memos are graded on reasoning and use of evidence, never on the position taken.

## Instructor prep

- [ ] Build `data/petition_posts.csv` for Lab 1 (~600 public posts, hand-labeled stance). It is **not** in the repo yet.
- [ ] Pre-download CelebA for Lab 2; the Google Drive mirror is often rate-limited.
- [ ] Snapshot the NVD and CISA KEV feeds for Lab 3 the week before class so every pair uses the same version.
- [ ] Recheck the Lab 1 and Lab 3 case facts the morning of class. Both stories are still developing.

## Facts to keep straight

- **Lab 1:** reporting confirms two affairs, not three. Present any third as unverified unless a source turns up.
- **Lab 2:** AI Frontiers was a research group, not an ethics team. Microsoft's 2023 layoff of its Ethics & Society team is the cleaner ethics-team comparison.
- **Lab 3:** the U.S. pre-release review is officially voluntary, but commentators argue the threat of export controls makes it effectively mandatory.

## IBM references

- [IBM: What is responsible AI? (Pillars of Trust)](https://www.ibm.com/think/topics/responsible-ai)
- [IBM Policy: AI Pillars](https://www.ibm.com/policy/blog/ibm-artificial-intelligence-pillars)
- [IBM: Foundations of trustworthy AI](https://www.ibm.com/think/insights/trustworthy-ai-foundations)
- [CEDA: Operationalising responsible AI (IBM's governance structure)](https://ceda.com.au/newsandresources/opinion/economy/operationalising-responsible-ai)
