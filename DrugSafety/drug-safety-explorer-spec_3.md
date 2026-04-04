# 🔬 Drug Safety Explorer
### Product Specification · April 2026

---

## What It Is

Drug Safety Explorer is an interactive web tool that lets anyone explore real FDA drug safety data. It pulls live information directly from the U.S. Food and Drug Administration's public database — giving users a clear, visual window into adverse event reports, recall history, and safety outcomes for hundreds of medications.

It was built for educational use: students, researchers, healthcare students, and curious members of the public who want to understand drug safety without needing a medical or technical background.

> ⚠️ **This tool is for educational purposes only.** It is not medical advice. Do not use it to make health or treatment decisions. Always consult a qualified healthcare professional.

---

## Purpose

Most FDA safety data is publicly available but difficult to access — buried in raw database exports or dense regulatory filings. Drug Safety Explorer makes that data approachable by:

- Translating raw report counts into readable visualizations
- Letting users compare drugs within the same class side-by-side
- Surfacing patterns in how safety events have been reported over time
- Providing plain-language explanations for every data point shown

---

## Who It's For

| Audience | How they use it |
|---|---|
| Students & educators | Explore real-world drug safety data for coursework or teaching |
| Healthcare students | Compare adverse event profiles across drug classes |
| Researchers | Get a quick overview of FAERS report volumes before deeper study |
| General public | Understand what FDA safety data exists for a medication they take |

---

## What It Shows

For any drug or pair of drugs, the tool surfaces the following safety information — all pulled live from the FDA:

- **Total adverse event reports** — how many times this drug has been mentioned in an FDA safety report
- **Serious outcomes** — reports involving hospitalization, disability, life-threatening events, or death
- **Most common reactions** — the top 10 symptoms or events reported alongside the drug, ranked by frequency
- **Reporting timeline** — how report volume has changed year over year since 2004
- **Severity breakdown** — a visual breakdown across five outcome categories (serious, hospitalization, life-threatening, disabling, death)
- **Recall history** — any FDA enforcement actions or recalls affecting products containing the drug, with classification and reason

---

## How It Works

The tool has three ways to explore:

### 🧬 Drug Class Explorer
Select a category of medications — like SSRIs (antidepressants) or Statins (cholesterol drugs) — and instantly see a live safety snapshot of every drug in that class. A chart compares total reports and serious outcomes across all five drugs at a glance. From there, pick one or two drugs to explore in depth.

### ⚖️ Compare Two Drugs
Enter any two drug names and view their full safety profiles side-by-side. An auto-generated insight highlights the difference in report volumes and serious outcome rates between the two.

### 🔍 Custom Search
Search any single drug by name to pull up its complete safety profile.

---

## Drug Classes Available

| Class | What they treat | Example drugs |
|---|---|---|
| SSRIs | Depression and anxiety | sertraline, fluoxetine, escitalopram |
| Statins | High cholesterol | atorvastatin, rosuvastatin, simvastatin |
| ACE Inhibitors | High blood pressure | lisinopril, enalapril, ramipril |
| NSAIDs | Pain and inflammation | ibuprofen, naproxen, celecoxib |
| PPIs | Acid reflux / stomach acid | omeprazole, pantoprazole, esomeprazole |
| ARBs | High blood pressure | losartan, valsartan, olmesartan |

---

## Interpreting the Data

The tool includes contextual help buttons (`?`) throughout the interface. Clicking any one opens a plain-language explanation of what that piece of data means and — importantly — how not to misread it.

Key interpretive notes surfaced to users:

- **More reports ≠ more dangerous.** A widely prescribed drug will naturally accumulate more adverse event reports simply because more people take it. Report counts are raw totals, not rates.
- **Reports are not proof of causation.** A patient who died while taking a medication may have died from their underlying disease. Adverse event reports describe what happened alongside a drug, not necessarily because of it.
- **Seriousness is self-reported.** The FDA does not independently verify every adverse event submission.
- **Recall classification matters.** A Class I recall is the most serious (potential for serious harm or death). A Class III recall typically involves a regulatory violation that is unlikely to cause harm.

---

## Data Source & Attribution

All data is sourced live from the OpenFDA public API — a free, openly accessible database maintained by the U.S. Food and Drug Administration.

Endpoints used:
- **Drug Adverse Events (FAERS)** — the FDA's Adverse Event Reporting System, containing voluntary and mandatory safety reports submitted by patients, healthcare professionals, and manufacturers
- **Drug Enforcement** — FDA recall and enforcement action records

> This product uses publicly available data from the U.S. Food and Drug Administration (FDA). FDA is not responsible for the product and does not endorse or recommend this or any other product.

---

## Limitations

| Area | What to know |
|---|---|
| Report volume | Counts are not normalized by how widely a drug is prescribed, so direct comparisons can be misleading without context |
| Drug name matching | Results depend on how the drug name appears in FDA records — generic names work best |
| Causation | Nothing in this tool establishes that a drug caused a reported event |
| Recall coverage | Recall search may occasionally miss entries or return partial results for drugs with short or common names |
| Data freshness | Data reflects what is currently in the OpenFDA database; there may be a lag between FDA records and real-time events |

---

*Drug Safety Explorer · April 2026 · For educational use only*
