# Facts-Only MF Assistant

**Product chosen for all milestones: Groww.** Independent NextLeap course prototype, not affiliated with Groww, HDFC, SEBI or AMFI.

## Scope
AMC: HDFC Mutual Fund. Schemes: HDFC Large Cap Fund, HDFC Flexi Cap Fund, HDFC ELSS – Tax Saver Fund and HDFC Mid Cap Fund. The collection contains 22 unique public official sources: four Direct-plan scheme pages, eight KIM/SID PDFs, eight AMC guides/directories and two AMFI/SEBI educational pages. See `sources.csv` and `sources.json` for source IDs, retrieval dates and hashes.

The checked-in corpus has 42 short reviewed, paraphrased evidence chunks; 27 are answer-eligible and 15 are supplementary document/disclosure references. Source pages were collected on 2026-10-02. This is the ingestion date, not necessarily the effective date of scheme terms. The SID/KIM documents are dated 2025-11-21. Official webpages can change after collection.

## Two executable modes — important for submission
1. **Hosted / no-install retrieval demo:** Uses rule-based intent and policy routing, scheme/field filters, sparse BM25-style lexical retrieval over reviewed chunks, and fixed evidence-backed phrasing. This is an extractive FAQ prototype, **not a running LLM**, and must not be described as a live generative RAG service.
2. **Local LLM-backed RAG mode:** `server.py` sends the allowed factual query and retrieved evidence to a locally running Ollama model, using `prompt.md`. It validates the generated JSON answer and citation ID against approved evidence before showing it. The model uses an intentionally constrained copy-generation prompt; its pretrained financial knowledge cannot supply new facts. Unavailable, invalid or ungrounded outputs fall back to the retrieval answer and are labelled as such. Model weights are not included. This mode needs the setup below; a real model inference was not run in the build environment.

To demonstrate W2/W3 with a live LLM, enable local LLM mode and confirm at least one response is labelled **Local LLM response validated against retrieved evidence**. A fallback does not count as a successful LLM run.

## Setup: retrieval demo
Install Python 3.10+ and Node.js 18+. Unzip the complete package, open its root folder in a terminal, then run:

```sh
python server.py
```

Open `http://127.0.0.1:8000`. On Windows, use `py server.py` if appropriate. No Python packages, API key or model are needed for this mode. Do not open `index.html` as a file URL because the JSON corpus loads with `fetch`.

## Setup: local LLM RAG
Install [Ollama](https://ollama.com/download) and run its service. Pull the model:

```sh
ollama pull qwen2.5:3b
```

Start the FAQ server with the LLM enabled:

macOS/Linux:
```sh
MF_LLM=1 python server.py
```

Windows PowerShell:
```powershell
$env:MF_LLM = "1"
py server.py
```

Open `http://127.0.0.1:8000`. For the notebook, open `MF-Assistant.ipynb` in the same folder. Optional environment variable `MF_MODEL` selects an already downloaded Ollama model. All model requests go to localhost; no cloud account data or hosted API keys are involved. Runtime endpoint reference: [Ollama chat API](https://docs.ollama.com/api/chat). This runtime documentation is not part of the mutual fund evidence corpus.

## How it works
`User input → PII/advice/performance gate → scheme and fact identification → reviewed corpus filter → BM25-style ranking → grounded answer / optional LLM → validation → one citation + source date.`

The policy gate refuses advice, projections and account processing before a model request. Out-of-scope, ambiguous and unsupported queries produce a refusal, clarification or an explicit cannot-verify message with an official link. One scheme and at most two facts are supported per query. No conversation history is persisted.

The runtime source allowlist comprises HDFC AMC, AMFI and SEBI. Final citations come from chunk metadata, never from model-generated URLs. Source text is paraphrased and curated to exclude return figures and promotional claims. KIM/SID references are supplementary because their dates are older than the scheme-page snapshot.

## Supported factual questions
- Scheme minimum SIP, benchmark, riskometer snapshot, category, exit load and lock-in.
- Account/consolidated account and capital gains statement guide links.
- Factual difference between Direct and Regular plan cost structures.
- Expense-ratio questions route to the official TER disclosure and explain why a dated numeric answer cannot be verified in this snapshot.

## Privacy
No PAN, Aadhaar, account/folio numbers, OTPs, emails or phone numbers. The browser blocks identifiable pasted/input patterns and clears matching content before a submission; the server independently checks before inference. Questions are never written to files, localStorage, cookies or analytics. Question text is transient in memory while processing; synthetic PII tests are redacted in the test report. Regex blocking is conservative, not an exhaustive PII detector; four-or-more-digit numbers can be blocked even when harmless. Statement authentication occurs only on the official external service.

## Refreshing the source collection
`ingest.py` fetches only the 22 allowlisted URLs and preserves raw snapshots and extraction locally; do not redistribute full snapshots as submission content. It requires `pypdf` for PDF extraction:

```sh
python -m pip install pypdf
python ingest.py
```

Before refreshing answers, manually review dated disclosures and addenda, confirm plan/scheme/field mappings, and update reviewed paraphrases in `build_corpus.py`. Then run `python build_corpus.py` and `node test.js`. The builder verifies selected source assertions but is not a fully automated financial-fact extraction service. Do not overwrite the source date without updating the engine metadata and re-reviewing facts.

## Checks and limits
`node test.js` passed 35 grounding, policy, scope, clarification and citation cases. JavaScript syntax and Python compilation were checked. The local HTTP/API path was checked and four constrained LLM validation tests passed using a simulated model; the real Ollama model has not been benchmarked. Browser visual QA was unavailable in this environment.

Known limits: no live data; no verified dated numeric TER; no account access; no return figures; English queries with curated aliases rather than broad semantic embeddings; no open-ended financial questions; older supporting documents may be superseded by addenda; no automatic refresh. The hosted link starts private and may require owner sign-in, so use the portable package or a short local demo for evaluators without access.

## Included deliverables
- `dist/index.html`: working UI, welcome line, exactly three example questions and disclaimer.
- `dist/sources.csv`: 22 official URLs and provenance.
- `dist/sample-qa.md`: 10 generated sample answers with links and dates.
- `dist/disclaimer.md`: UI disclaimer.
- `prompt.md`: system prompt and grounding instructions.
- `server.py`, `MF-Assistant.ipynb`: local LLM RAG path.
- `dist/system-design.md`: design rationale and decision rules.
- `dist/demo-script.md`: under-three-minute demonstration outline.
- `dist/test-results.json`: behavioural verification.


## Working prototype

[Open MF Facts](https://mf-facts-prem.premkumarnj2424.chatgpt.site) — private access; hosted retrieval demo, not live LLM inference.

## Official source list — 22 URLs

| ID | Official page/document | Source URL |
|---|---|---|
| large | HDFC Large Cap Fund — Direct | [Official source](https://www.hdfcfund.com/explore/mutual-funds/hdfc-large-cap-fund/direct) |
| large-sid | HDFC Large Cap Fund SID | [Official source](https://files.hdfcfund.com/s3fs-public/SID/2025-11/SID%20-%20HDFC%20Large%20Cap%20Fund%20dated%20November%2021%2C%202025_0.pdf) |
| large-kim | HDFC Large Cap Fund KIM | [Official source](https://files.hdfcfund.com/s3fs-public/KIM/2025-11/KIM%20-%20HDFC%20Large%20Cap%20Fund%20dated%20November%2021%2C%202025_0.pdf) |
| flexi | HDFC Flexi Cap Fund — Direct | [Official source](https://www.hdfcfund.com/explore/mutual-funds/hdfc-flexi-cap-fund/direct) |
| flexi-sid | HDFC Flexi Cap Fund SID | [Official source](https://files.hdfcfund.com/s3fs-public/SID/2025-11/SID%20-%20HDFC%20Flexi%20Cap%20Fund%20dated%20November%2021%2C%202025_0.pdf) |
| flexi-kim | HDFC Flexi Cap Fund KIM | [Official source](https://files.hdfcfund.com/s3fs-public/KIM/2025-11/KIM%20-%20HDFC%20Flexi%20Cap%20Fund%20dated%20November%2021%2C%202025_1.pdf) |
| elss | HDFC ELSS Tax Saver Fund — Direct | [Official source](https://www.hdfcfund.com/explore/mutual-funds/hdfc-elss-tax-saver-fund/direct) |
| elss-sid | HDFC ELSS Tax Saver SID | [Official source](https://files.hdfcfund.com/s3fs-public/SID/2025-11/SID%20-%20HDFC%20ELSS%20Tax%20Saver%20dated%20November%2021%2C%202025.pdf) |
| elss-kim | HDFC ELSS Tax Saver KIM | [Official source](https://files.hdfcfund.com/s3fs-public/KIM/2025-11/KIM%20-%20HDFC%20ELSS%20Tax%20Saver%20dated%20November%2021%2C%202025_0.pdf) |
| mid | HDFC Mid Cap Fund — Direct | [Official source](https://www.hdfcfund.com/explore/mutual-funds/hdfc-mid-cap-fund/direct) |
| mid-sid | HDFC Mid Cap Fund SID | [Official source](https://files.hdfcfund.com/s3fs-public/SID/2025-11/SID%20-%20HDFC%20Mid%20Cap%20Fund%20dated%20November%2021%2C%202025_1.pdf) |
| mid-kim | HDFC Mid Cap Fund KIM | [Official source](https://files.hdfcfund.com/s3fs-public/KIM/2025-11/KIM%20-%20HDFC%20Mid%20Cap%20Fund%20dated%20November%2021%2C%202025_1.pdf) |
| factsheets | Monthly factsheets | [Official source](https://www.hdfcfund.com/mutual-funds/factsheets) |
| ter | Total expense ratio disclosures | [Official source](https://www.hdfcfund.com/statutory-disclosure/total-expense-ratio-of-mutual-fund-schemes/reports) |
| risk | Annual riskometer disclosures | [Official source](https://www.hdfcfund.com/statutory-disclosure/riskometers) |
| statements | Request account and capital gains statements | [Official source](https://www.hdfcfund.com/services/additional-info/request-statement) |
| cas | Consolidated account statement | [Official source](https://www.hdfcfund.com/services/consolidated-account-statement) |
| sip | SIP FAQs | [Official source](https://www.hdfcfund.com/services/faqs/systematic-investment-plan) |
| kim | KIM directory | [Official source](https://www.hdfcfund.com/mutual-funds/fund-documents/kim) |
| sid | SID directory | [Official source](https://www.hdfcfund.com/mutual-funds/fund-documents/sid) |
| amfi-risk | AMFI: risks in mutual funds | [Official source](https://www.amfiindia.com/investor/knowledge-center-info?zoneName=riskInMutualFunds) |
| sebi-plans | SEBI: regular and direct mutual funds | [Official source](https://investor.sebi.gov.in/regular_and_direct_mutual_funds.html) |

## Sample Q&A

### 1. Minimum SIP of HDFC Flexi Cap?

HDFC Flexi Cap Fund lists a minimum SIP of ₹100 on its official Direct-plan page.

Source: https://www.hdfcfund.com/explore/mutual-funds/hdfc-flexi-cap-fund/direct

Last updated from sources: 2026-10-02

### 2. HDFC ELSS minimum SIP

HDFC ELSS - Tax Saver Fund lists a minimum SIP of ₹500 on its official Direct-plan page.

Source: https://www.hdfcfund.com/explore/mutual-funds/hdfc-elss-tax-saver-fund/direct

Last updated from sources: 2026-10-02

### 3. HDFC Large Cap exit load

HDFC Large Cap Fund lists a 1% exit load for redemption or switch-out within one year of allotment, and none after one year.

Source: https://www.hdfcfund.com/explore/mutual-funds/hdfc-large-cap-fund/direct

Last updated from sources: 2026-10-02

### 4. HDFC Mid Cap benchmark

HDFC Mid Cap Fund uses NIFTY Midcap 150 Index (Total Returns Index) as its benchmark.

Source: https://www.hdfcfund.com/explore/mutual-funds/hdfc-mid-cap-fund/direct

Last updated from sources: 2026-10-02

### 5. HDFC ELSS lock-in

HDFC ELSS - Tax Saver Fund has a three-year statutory lock-in from allotment of the respective units.

Source: https://www.hdfcfund.com/explore/mutual-funds/hdfc-elss-tax-saver-fund/direct

Last updated from sources: 2026-10-02

### 6. HDFC ELSS exit load

HDFC ELSS - Tax Saver Fund lists no exit load; its three-year statutory lock-in still applies.

Source: https://www.hdfcfund.com/explore/mutual-funds/hdfc-elss-tax-saver-fund/direct

Last updated from sources: 2026-10-02

### 7. HDFC Large Cap riskometer

The collected official page labels HDFC Large Cap Fund as Very High risk; this is a source snapshot, not a live risk assessment.

Source: https://www.hdfcfund.com/explore/mutual-funds/hdfc-large-cap-fund/direct

Last updated from sources: 2026-10-02

### 8. How can I download a capital gains statement?

Open HDFC’s official Request Statement page and choose “Capital Gains Statement”; complete any authentication only on the official service.

Source: https://www.hdfcfund.com/services/additional-info/request-statement

Last updated from sources: 2026-10-02

### 9. Download my CAS

Use HDFC’s official Consolidated Account Statement page to request or download your statement, completing any authentication only on the official service.

Source: https://www.hdfcfund.com/services/consolidated-account-statement

Last updated from sources: 2026-10-02

### 10. Should I buy HDFC Flexi Cap?

I can explain official scheme facts, but I cannot recommend funds or advise on buying, selling or portfolio choices. The linked AMFI page explains mutual fund risks.

Source: https://www.amfiindia.com/investor/knowledge-center-info?zoneName=riskInMutualFunds

Last updated from sources: 2026-10-02

## UI disclaimer

Facts-only. No investment advice.

This independent course prototype answers public factual questions about four HDFC Mutual Fund schemes using official sources. It does not recommend investments, calculate or compare returns, or access accounts. Never enter PAN, Aadhaar, account or folio numbers, OTPs, email addresses or phone numbers. Verify dated disclosures on the linked official page before relying on a fact.

Last updated from sources: 2026-10-02
