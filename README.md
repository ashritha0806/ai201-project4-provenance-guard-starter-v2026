# Provenance Guard

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first.**
>
> Once `python test.py` passes, in one terminal:
>
> ```bash
> python app.py
> ```
>
> and in another:
>
> ```bash
> curl -X POST http://127.0.0.1:5000/ping \
>   -H "Content-Type: application/json" \
>   -d '{"message": "hello"}'
> ```
>
> That's a web service. Something sent it text and it sent something back.
> Everything this unit is filling in the middle.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     Unit 7 asks for the first five sections. Unit 8 adds the five below.

     Everything is pasted as TEXT. A pasted curl response gets full credit;
     a picture of your terminal gets none.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 7 — THE BUILD ═══════════════════════ -->

## What This Does

<!-- The path a submission takes, from arriving to the label a reader sees.
     Six or seven steps — this is the trace you wrote in Milestone 1. -->

1. app.py submit() parses the JSON and checks that text and creator_id are present and usable.
2. detector.py model signal returns the model score.
3. stylometry.py style signal returns the style score.
4. scoring.py combine signals merges the two scores into one confidence value.
5. scoring.py score_to_label maps the confidence to ai, human, or unsure.
6. audit.py log decision records the result under a new content_id.
7. The response returns to the caller, who shows the label to the reader. A disputed label can be appealed through POST /appeal.

### False positive trace

### False positive trace

A writer submits an essay that they wrote themselves, in plain, even sentences. Signal one reads the predictable vocabulary as machine-like (`model_score = 0.72`). Signal two reads the uniform sentence length as repetitive (`style_score = 0.68`). The combined score lands at `0.704` (above the `0.65` AI threshold), returning the label: *"This text shows strong patterns characteristic of AI-generated writing."* The writer sees only this accusatory label with no access to the underlying scores. Their only recourse is to submit `POST /appeal` with their `content_id` and reasoning, which transitions the item's status in the audit log to `under_review` for human review.

### Endpoint contract

POST /submit

- Request: {"text": string, "creator_id": string}
- Response: {"content_id": string, "guess": "ai" | "human" | "unsure", "confidence": number between 0 and 1, "label": string, "model_score": number between 0 and 1, "style_score": number between 0 and 1}
- Error (missing, empty, non-ascii characters or oversized text): {"error": "invalid_text", "message": string} with HTTP 400
- Error (missing, empty, non-ascii characters or oversized creator_id): {"error": "invalid_creator_id", "message": string} with HTTP 400

POST /appeal

- Request: {"content_id": string, "reasoning": string}
- Response: {"content_id": string, "status": "under_review", "message": string}
- Error (unknown content_id): {"error": "content_id_not_found", "message": string} with HTTP 404
- Error (missing, empty reasoning): {"error": "reasoning_not_found", "message": string} with HTTP 404

---

## Signals and Scoring

<!-- For each signal: what property it measures, why that might differ between
     human and AI writing, and WHAT IT CAN'T SEE.

     The blind spot is not optional. If you can't name it you don't understand
     the signal yet — and unit 8's attack set is built out of blind spots. -->

### Signal one — the model signal

**What it measures:**
How predictable the text is to a local language model, measured as perplexity. Low perplexity means the model could easily guess the next words. detector.py model_signal turns perplexity into a 0–1 score, where higher means more likely AI.

**Why that might differ between human and AI writing:**
Language models produce text by choosing likely next words, so their output tends to be predictable. Human writing is more irregular, with unusual word choices and hard to guess the writing pattern.

**What it can't see:**

It rewards common words, ordinary sentences, so plain human writing can score as AI written. It can also miss AI text that uses rare or unusual vocabulary, because that text is harder for the model to predict.

### Signal two — the style signal

**What it measures:**
The shape of the text, without reading its meaning. The planned measures are sentence length spread, type-token ratio (unique words divided by total words), and punctuation density. stylometry.py style signal will combine them into a 0–1 score, where higher means more likely AI.

**Why that might differ between human and AI writing:**
Generated text tends to sound uniform. Most sentences are similar length, so pattern feels same. Human writing tends to vary more in rhythm and punctuation.

**What it can't see:**
Split sentences, quotation, meaning can change its score without changing the substance. Even sometimes plain human writing can look AI written.

### The combining rule

<!-- Write the rule in words, then give the numbers. What happens when the two
     signals disagree? -->

**The rule:**
When the two signals disagree that is one leans human and the other leans AI, their disagreement reach the combined score toward the middle around 0.45 - 0.55, resulting in the "unsure" label. This prevents a single signal from producing a false accusation when the signals are in conflict.

**The numbers:**
WEIGHT_MODEL_SIGNAL = 0.6 (60%)
WEIGHT_STYLE_SIGNAL = 0.4 (40%)
In stylometry: 60% sentence length spread, 20% type-token ratio, 20% punctuation density.

**Where it lives:** `scoring.py::combine_signals`

<!-- ⚠️ The grader checks your code against that line. If your rule lives
     somewhere else, say where. -->

**A case where my two signals split, and what my rule does with it:**

Text: The meeting was adjourned. Although several committee members expressed strong reservations about the newly proposed timeline and budget allocation, the chairperson decided to proceed anyway.

Signal one(detector.model_signal): 0.56(vocabulary leans AI)
Signal two(stylometry.style_signal): 0.3351 (sentence length variation leans Human)
Combined score (scoring.combine_signals): 0.4712

What my rule does with it: The two signals disagree (model_score at 0.562 vs style_score at 0.3351). The combining rule averages them into a combined score of 0.4712, pulling it towards "unsure". This protects the writer from a false accusation based solely on vocabulary predictability by letting the sentence length variation temper the score.

---

## Label Variants

<!-- The exact text of all three labels, as a reader would see them, and the
     score range each covers.

     Write them for someone who has never heard the word "threshold". -->

| Label                 | Score range | The exact text a reader sees |
| --------------------- | ----------- | ---------------------------- |
| high-confidence human |   0.00 – 0.35          |       This text shows natural variations consistent with human writing.                       |
| unsure                |    0.35 – 0.65         |    The patterns in this text are mixed, so we cannot determine with confidence whether it was written by a human or an AI.                          |
| high-confidence AI    |      0.65 – 1.00       |    This text shows strong patterns characteristic of AI-generated writing.                          |

**Why I worded the "unsure" one this way:**
The "unsure" label admits uncertainty without sounding like an accusation against the writer. Because human writing can be plain, direct, or structured, an ambiguous score should never accuse someone of wrong doing. It should state clearly that the system's patterns were mixed and that no determination can be made.
<!-- It's the hardest of the three. It has to admit uncertainty without
     sounding like an accusation, because the person reading it may well have
     written every word themselves. -->

---

## Sample Run

<!-- One submission and one appeal, pasted as text: the request, the response,
     and the log entries. -->

**A submission**

<!-- The response needs content_id, guess, confidence, label, AND both
     signal scores - model_score and style_score. Unit 8's tools read those
     two off the response, so leaving them out costs you next unit's
     diagnostics. -->

```bash
$ curl -X POST http://127.0.0.1:5000/submit \
  -H "Content-Type: application/json" \
  -d '{"text": "Choosing a laptop comes down to your priorities. If you travel often, portability and battery life matter most, while heavy gaming or video editing calls for stronger processing power and a better graphics card.", "creator_id": "student_1"}'
```

```json
{
  "confidence": 0.471,
  "content_id": "sub_76effa73",
  "guess": "unsure",
  "label": "The patterns in this text are mixed, so we cannot determine with confidence whether it was written by a human or an AI.",
  "model_score": 0.5804,
  "style_score": 0.3069
}
```

**An appeal**

```bash
$ curl -X POST http://127.0.0.1:5000/appeal \
  -H "Content-Type: application/json" \
  -d '{"content_id": "sub_76effa73", "reasoning": "I wrote this advice myself based on my own experience buying laptops."}'
```

```json
{
  "content_id": "sub_76effa73",
  "message": "Appeal recorded successfully. Submission is now under review.",
  "status": "under_review"
}
```

**What the log shows afterwards**   

```bash
$ curl "http://127.0.0.1:5000/log?limit=3"
```
```json
{
  "count": 3,
  "entries": [
    {
      "combined_score": 0.471,
      "content_id": "sub_0a31163d",
      "creator_id": "student_1",
      "guess": "human",
      "label": "unsure",
      "model_score": 0.5804,
      "status": "decided",
      "style_score": 0.3069,
      "timestamp": "2026-11-07T22:10:27+00:00"
    },
    {
      "combined_score": 0.471,
      "content_id": "sub_76effa73",
      "creator_id": "student_1",
      "guess": "unsure",
      "label": "The patterns in this text are mixed, so we cannot determine with confidence whether it was written by a human or an AI.",
      "model_score": 0.5804,
      "status": "decided",
      "style_score": 0.3069,
      "timestamp": "2026-11-08T00:18:22+00:00"
    },
    {
      "content_id": "sub_76effa73",
      "creator_id": "student_1",
      "event": "appeal",
      "reasoning": "I wrote this advice myself based on my own experience buying laptops.",
      "status": "under_review",
      "timestamp": "2026-11-08T00:20:39+00:00"
    }
  ]
}
```
---

## How I Used AI

**Moment 1**

- _What I asked for: How to implement style_signal(text) to combine sentence length spread, type-token ratio, and punctuation density into a single score.

- _What came back: A suggested combination using 70% sentence spread and 30% type-token ratio, while calculating punctuation density without actually using it in the score.

- _What I changed: I noticed that punctuation density was computed but unused. I adjusted the weighting formula so all three features can contribute while choosing 60% sentence spread, 20% type-token ratio, and 20% punctuation density for a balanced stylometric score.

**Moment 2**

- _What I asked for: Why my API response returned "content_id": "test-001" and "guess": "ai" even though the audit log had a generated UUID and the combined confidence was below 0.50.
- _What came back: An explanation showing that line 190 in app.py was still returning hardcoded placeholder strings from Milestone 3, and line 177 was checking only model_score >= 0.5 instead of the combined confidence.
- _What I changed: I replaced the hardcoded return values with the actual variables (content_id, model_score, style_score), connected guess and label to scoring.score_to_label, and verified that the audit log and curl responses matched.

**Moment 3 (Unit 8 Verification and Audit)**

- _What I asked for:_ To audit my test outputs, verify that my trial numbers and denominators matched criteria.md targets, check that each criterion had an explicit decision sentence, ensure all 14 broken attacks had stage and mechanism diagnoses, and verify that no required sentences or checklist items were missing before final submission.
- _What came back:_ Feedback pointing out that Criterion 1 had accidentally duplicated the label-coverage JSON block, that criteria diagnoses needed to explicitly reference the service's 5 pipeline stages, and that every criterion required a clear sentence explaining the decision.
- _What I changed:_ I corrected the Criterion 1 pasted JSON with the authentic Trial 1 output from `run_2026-11-10_1243_before.md`, mapped all criteria misses directly to the 5 pipeline stages, added the decision rationales, and verified all checklist items.

<!-- ═══════════════════════ UNIT 8 — THE TEST ═══════════════════════ -->

---

## Rate Limiting

**My limits:** **10 per minute, 200 per day**

**Why those numbers and not others:**
Ten submissions per minute gives a real human writer ample headroom to submit essays, revisions, and test paragraphs without ever hitting an obstacle. However, it is strictly hostile to automated scripts trying to reverse-engineer thresholds, flood server resources, or brute-force evasion attacks by running hundreds of automated variations.
<!-- Think about two people: a real writer submitting their own work a few
     times an hour, and a script sending a thousand variations to map your
     thresholds. Your numbers have to be liveable for the first and hostile to
     the second. -->

**What a caller counts as:** <!-- per address, or per creator_id? They fail differently — one script can look like a thousand callers, and one household can look like one. -->
The rate limiter tracks requests per creator_id from the JSON payload (falling back to the remote IP address if creator_id is missing). Tracking per creator_id ensures a script cannot evade limits simply by cycling through rotating proxies or VPN addresses, while allowing multiple writers sharing a home or campus IP to submit independently.

**The run of status codes when I pushed past it:**

<!-- python run_attacks.py --flood 15 -->

```

  [ 1/15] 200
  [ 2/15] 200
  [ 3/15] 200
  [ 4/15] 200
  [ 5/15] 200
  [ 6/15] 200
  [ 7/15] 200
  [ 8/15] 200
  [ 9/15] 200
  [10/15] 200
  [11/15] 429
  [12/15] 429
  [13/15] 429
  [14/15] 429
  [15/15] 429

============================================================
200 200 200 200 200 200 200 200 200 200 429 429 429 429 429
============================================================

5 of 15 were rate limited.
```

---

## Attack Run — Before

<!-- Every attack with its outcome and a held-or-broke mark.
     `run_attacks.py --label before` produces the table. -->
Total Attacks: 56
Total HELD: 42 (75%)
Total BROKE: 14 (25%)

Breakdown by family:
evasion (10 total): 4 held, 6 broke
false_positive (8 total): 2 held, 6 broke
prompt_injection (6 total): 5 held, 1 broke
malformed (20 total): 19 held, 1 broke
flood_same_creator (12 total): 12 held, 0 broke

**Ten audit log entries from the run**

<!-- Choose entries that show the interesting failures, not the ten easiest. -->

```json
[
  {
    "timestamp": "2026-11-10T00:37:46+00:00",
    "content_id": "sub_b1e5bf52",
    "creator_id": "attacker_EV01",
    "guess": "human",
    "model_score": 0.1635,
    "style_score": 0.5934,
    "combined_score": 0.3355,
    "label": "This text shows natural variations consistent with human writing.",
    "status": "decided"
  },
  {
    "timestamp": "2026-11-10T00:37:47+00:00",
    "content_id": "sub_e3fdd0cd",
    "creator_id": "attacker_EV10",
    "guess": "ai",
    "model_score": 0.9792,
    "style_score": 0.5725,
    "combined_score": 0.8165,
    "label": "This text shows strong patterns characteristic of AI-generated writing.",
    "status": "decided"
  },
  {
    "timestamp": "2026-11-10T00:37:47+00:00",
    "content_id": "sub_499fd68d",
    "creator_id": "attacker_FP01",
    "guess": "unsure",
    "model_score": 0.6175,
    "style_score": 0.5012,
    "combined_score": 0.571,
    "label": "The patterns in this text are mixed, so we cannot determine with confidence whether it was written by a human or an AI.",
    "status": "decided"
  },
  {
    "timestamp": "2026-11-10T00:37:47+00:00",
    "content_id": "sub_ae2b9070",
    "creator_id": "attacker_FP03",
    "guess": "ai",
    "model_score": 0.8051,
    "style_score": 0.5928,
    "combined_score": 0.7202,
    "label": "This text shows strong patterns characteristic of AI-generated writing.",
    "status": "decided"
  },
  {
    "timestamp": "2026-11-10T00:37:47+00:00",
    "content_id": "sub_686bff22",
    "creator_id": "attacker_FP04",
    "guess": "ai",
    "model_score": 0.8329,
    "style_score": 0.5822,
    "combined_score": 0.7326,
    "label": "This text shows strong patterns characteristic of AI-generated writing.",
    "status": "decided"
  },
  {
    "timestamp": "2026-11-10T00:37:47+00:00",
    "content_id": "sub_d34467b7",
    "creator_id": "attacker_PI02",
    "guess": "human",
    "model_score": 0.1774,
    "style_score": 0.4907,
    "combined_score": 0.3027,
    "label": "This text shows natural variations consistent with human writing.",
    "status": "decided"
  },
  {
    "timestamp": "2026-11-10T00:37:47+00:00",
    "creator_id": "attacker_MF01",
    "event": "rejected",
    "status": "rejected",
    "reason": "missing_text"
  },
  {
    "timestamp": "2026-11-10T00:37:47+00:00",
    "creator_id": "attacker_MF06",
    "event": "rejected",
    "status": "rejected",
    "reason": "non_ascii"
  },
  {
    "timestamp": "2026-11-10T00:37:47+00:00",
    "creator_id": "attacker_MF08",
    "event": "rejected",
    "status": "rejected",
    "reason": "oversized_text"
  },
  {
    "timestamp": "2026-11-10T00:37:47+00:00",
    "creator_id": "attacker_flood",
    "event": "rejected",
    "status": "rejected",
    "reason": "rate_limited",
    "limit": "10 per 1 minute"
  }
]
```

**The single worst outcome** — the one I'd least want to explain to a writer
whose work got caught by it:

Attack FP03 a non-native English speaker writing carefully, scored 0.7202 and was falsely labeled as high-confidence AI. Because the writer relied on standard vocabulary and straight forward sentences, the language model found the text highly predictable (model_score = 0.8051). The writer is accused of using AI simply for writing clearly and cautiously in an acquired language.
---

## Run Log and Verdicts

<!-- Your five criteria across three trials. `run_eval.py --label before`. -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
| --------- | ------ | ----- | ----- | ----- | ------- |
| 1. False positives on human writing | ≤ 3 of 30 flagged as AI (≤ 10%) | 0/2  | 0/2  | 0/2  | MISSED |
| 2. Score spread (AI vs Human) | Median gap ≥ 0.40 across 30 AI & 30 human texts | 0.552 (1 pair) | 0.552 (1 pair) | 0.552 (1 pair) | MISSED |
| 3. Label coverage | All 3 labels appear ≥ 3 times across 30 texts | 1 of each (3 texts) | 1 of each (3 texts) | 1 of each (3 texts) | MISSED |
| 4. Appeal path changes status | 10 of 10 appeals change status to under_review | 3/3 (by hand) | 3/3 (by hand) | 3/3 (by hand) | MISSED |
| 5. Bad input (< 20 words) | 400 refused with message, no crash | 4/4  | 4/4  | 4/4 | MET |

### How Each Verdict Was Decided
- **Criterion 1 (False positives on human writing) — MISSED:** Decided as MISSED because the evaluation trial only tested 2 human texts instead of the required 30-text sample, and on the broader attack set, formal human writing suffered a 75% false-positive rate (6 of 8 formal human texts flagged as AI).
- **Criterion 2 (Score spread) — MISSED:** Decided as MISSED because the target required a median score gap ≥ 0.40 across 30 AI and 30 human texts, whereas the trial tested only 1 pair, and on the full attack set the median score gap narrowed to 0.11.
- **Criterion 3 (Label coverage) — MISSED:** Decided as MISSED because although all 3 labels were produced, they appeared only 1 time each across a 3-text scenario rather than the required ≥ 3 times across a 30-text set.
- **Criterion 4 (Appeal path changes status) — MISSED:** Decided as MISSED because while 3 of 3 manual curl appeals successfully transitioned to `under_review`, the testing stopped at 3 trials rather than reaching the required 10 of 10 target.
- **Criterion 5 (Bad input handling) — MET:** Decided as MET because 4 of 4 (100%) malformed and short inputs were cleanly rejected with HTTP 400 and an informative error message across all 3 trials without any server crash.

**Real output from one trial**, pasted as text, naming the file and function that produced it: run_eval.py::main, recorded in results/run_2026-11-10_1243_before.md:

Criterion 1 — my own writing (Trial 1):
```json
[
  {
    "status": 200,
    "guess": "unsure",
    "confidence": 0.4474,
    "model_score": 0.5598,
    "style_score": 0.2789,
    "label": "The patterns in this text are mixed, so we cannot determine with confidence whether it was written by a human or an AI.",
    "sent": "Wait, did you check under the couch cushions? I literally looked every"
  },
  {
    "status": 200,
    "guess": "unsure",
    "confidence": 0.5487,
    "model_score": 0.6574,
    "style_score": 0.3856,
    "label": "The patterns in this text are mixed, so we cannot determine with confidence whether it was written by a human or an AI.",
    "sent": "Honestly, the best part of the trip was just sitting on the dock early"
  }
]
```

Criterion 2 — score spread (Trial 1):
```json
[
  {
    "status": 200,
    "guess": "ai",
    "confidence": 0.8067,
    "model_score": 0.9381,
    "style_score": 0.6097,
    "label": "This text shows strong patterns characteristic of AI-generated writing.",
    "sent": "In today's world, technology plays a crucial role in our daily lives. "
  },
  {
    "status": 200,
    "guess": "human",
    "confidence": 0.255,
    "model_score": 0.2126,
    "style_score": 0.3185,
    "label": "This text shows natural variations consistent with human writing.",
    "sent": "Breathtaking. Visiting Mount Rainier during peak August wildflower sea"
  }
]
```
Criterion 3 — label coverage (Trial 1):
```json
[
  {
    "status": 200,
    "guess": "ai",
    "confidence": 0.8067,
    "model_score": 0.9381,
    "style_score": 0.6097,
    "label": "This text shows strong patterns characteristic of AI-generated writing.",
    "sent": "In today's world, technology plays a crucial role in our daily lives. "
  },
  {
    "status": 200,
    "guess": "human",
    "confidence": 0.2324,
    "model_score": 0.1932,
    "style_score": 0.2913,
    "label": "This text shows natural variations consistent with human writing.",
    "sent": "Run. We sprinted through the damp, murky labyrinth beneath the city, o"
  },
  {
    "status": 200,
    "guess": "unsure",
    "confidence": 0.4712,
    "model_score": 0.562,
    "style_score": 0.3351,
    "label": "The patterns in this text are mixed, so we cannot determine with confidence whether it was written by a human or an AI.",
    "sent": "The meeting was adjourned. Although several committee members expresse"
  }
]
```

Criterion 4 — appeal path (Tested by hand via curl against POST /appeal):
```json
{
  "content_id": "sub_76effa73",
  "message": "Appeal recorded successfully. Submission is now under review.",
  "status": "under_review"
}

{
  "timestamp": "2026-11-10T20:58:52+00:00",
  "content_id": "sub_76effa73",
  "creator_id": "student_1",
  "event": "appeal",
  "reasoning": "Trial 1 test appeal",
  "status": "under_review"
}
```

Criterion 5 — bad input (Trial 1):
```json
[
  {
    "status": 400,
    "guess": null,
    "confidence": null,
    "model_score": null,
    "style_score": null,
    "label": null,
    "sent": "This is only five words."
  },
  {
    "status": 400,
    "guess": null,
    "confidence": null,
    "model_score": null,
    "style_score": null,
    "label": null,
    "sent": "{\"creator_id\": \"eval_bad_missing\"}"
  },
  {
    "status": 400,
    "guess": null,
    "confidence": null,
    "model_score": null,
    "style_score": null,
    "label": null,
    "sent": "word word word word word word word word word word word word word word "
  },
  {
    "status": 400,
    "guess": null,
    "confidence": null,
    "model_score": null,
    "style_score": null,
    "label": null,
    "sent": "{\"text\": null, \"creator_id\": \"eval_bad_null\"}"
  }
]
```

**Diagnoses**

<!-- For each miss AND each broken attack: the stage and the mechanism.

     Your service has five stages: the route, signal one, signal two, the
     combining rule, and the label mapping.

     Not a diagnosis:  "The evasion attacks got through."
     A diagnosis:      "Four evasion inputs got through. All four had typos
                        added to AI-generated text. My second signal measures
                        word variety, and typos raise word variety — so it read
                        them as more human. My combining rule weights that
                        signal at 0.6, so it dragged the whole score down."

     Look for a pattern. Five failures on one signal is one problem, not five. -->
#### 1. Criteria Misses:
- **Criterion 1 Miss (False Positives) & Criterion 2 Miss (Score Spread):**
  - **Stage:** Signal 1 (`detector.py::model_signal`) and Combining Rule (`scoring.py::combine_signals`).
  - **Mechanism:** In initial criteria, I assumed human and AI scores would cleanly separate by ≥0.40. In practice, formal human writing such as essays and non-native writing relies on standard vocabulary that GPT-2 finds highly predictable (model_score > 0.70). Because the combining rule weights Signal 1 at 0.60, human scores were pulled upward while AI evasion scores were pulled downward, resulting in a narrow score gap (~0.11 on the attack set) and causing 6 of 8 formal human texts to be falsely accused of AI.

- **Criterion 3 Miss (Label Coverage):**
  - **Stage:** The label mapping (`scoring.py::score_to_label`).
  - **Mechanism:** The label mapping defines 3 score tiers (`human` < 0.35, `unsure` 0.35–0.65, `ai` > 0.65). While all 3 labels were proven reachable in the mapping, producing each label ≥3 times requires a larger evaluated sample; `scenarios.py` only supplied 3 texts total (1 per band), falling short of the 30-text target.

- **Criterion 4 Miss (Appeal Path):**
  - **Stage:** The route (`app.py::appeal`).
  - **Mechanism:** The evaluation harness (`run_eval.py`) only tests the `/submit` route and lacks automated dispatch to the `/appeal` route. While manual `curl` calls confirmed the route transitions records to `under_review`, the manual test only covered 3 trials instead of the required 10.

#### 2. Broken Attacks (14 BROKE from Attack Set):
- **False Positives on Formal & Non-Native Human Writing (FP02–FP07, 6 attacks — BROKE):**
  - **Stage:** Signal 1 (`detector.py::model_signal`) and Combining Rule.
  - **Mechanism:** Text following formal rubrics, academic templates, or non-native writing uses standard, predictable words. Signal 1 produced high AI scores (0.70 – 0.83). Weighted at 0.60, this dragged combined scores past the 0.65 threshold, falsely accusing human writers.

- **Evasion via Lexical and Structural Tampering (EV01, EV03, EV04, EV06, EV09, EV10, 6 attacks — BROKE):**
  - **Stage:** Signal 2 (`stylometry.py::style_signal`) and Combining Rule.
  - **Mechanism:** In `EV01`, adding typos artificially inflated the type token ratio, driving the style score to human and pulling the combined score to `0.336` (human). In `EV03`, `EV04`, `EV06`, and `EV09`, inserting quotations, extreme sentence length variation, or rare words disrupted the uniform rhythm that Signal 2 looks for, pushing AI text down into the `unsure` band (0.47 – 0.59) instead of being flagged as AI. In `EV10`, repetitive stanzas scored high on model signal (0.979), but the style signal remained moderate (0.572), preventing a decisive label.

- **Prompt Injection Evasion (PI02, 1 attack — BROKE):**
  - **Stage:** Signal 1 (`detector.py::model_signal`).
  - **Mechanism:** `PI02` disguised instructions as system messages using meta language. While the system did not execute the instruction, the atypical phrasing was surprising to GPT-2 (high perplexity), yielding a low model score of 0.177 and allowing AI text to evade detection with a combined score of 0.303.

- **Unhandled JSON Array Request Body (RQ07, 1 attack — BROKE):**
  - **Stage:** The route (`app.py::submit`).
  - **Mechanism:** `RQ07` submitted a raw JSON array rather than a dictionary. Calling `payload.get("text")` on a Python list raised an unhandled `AttributeError`, causing an HTTP 500 server crash instead of a clean 400 refusal.

#### 3. Cross-Cutting Patterns Across Misses and Broken Attacks:
- **Pattern 1: The Predictability Trap (Signal 1 + Combining Rule — 7 failures: FP02–FP07, PI02):**
  Signal 1 equates "predictable next words" directly with "AI-generated". Formal human essays and non-native English writers naturally rely on safe, predictable words, so Signal 1 scores them as AI (>0.70). Because the combining rule assigns 60% of the weight to Signal 1, this single flawed assumption dominates the final verdict and triggers repeated false positives.
- **Pattern 2: Stylometric Brittleness (Signal 2 — 6 failures: EV01, EV03, EV04, EV06, EV09, EV10):**
  Signal 2 relies on surface heuristics (Type-Token Ratio and sentence length standard deviation). Attackers easily bypassed or diluted these metrics through trivial perturbations—adding typos (raising TTR), inserting quotations, or interleaving short and long sentences. This dragged AI text out of the AI band into `unsure` or `human`.
- **Pattern 3: Unchecked Request Deserialization (The Route — 1 failure: RQ07):**
  The route assumed `request.get_json()` always produces a Python dict, leaving the service vulnerable to a 500 crash whenever a valid JSON non-dict (like an array `[]`) is posted.

---

## The Improvement

**What I changed:**
Reweighted the two signals in `config.py`: changed `WEIGHT_MODEL_SIGNAL` from 0.6 to 0.4, and `WEIGHT_STYLE_SIGNAL` from 0.4 to 0.6. The threshold bounds remained unchanged at 0.35 and 0.65.

**Which diagnosis pointed at it:**
Pattern 1 from the Diagnoses section ("The Predictability Trap: Signal 1 + Combining Rule"). The before-run showed that weighting Signal 1 (GPT-2 token perplexity) at 0.60 dominated the overall score, causing formal human writing (FP02–FP07) to be accused of being AI, while allowing prompt injection with unusual system syntax (PI02) to score low and evade as human. Shifting weight toward Signal 2 (stylometry) was chosen to reduce reliance on GPT-2's vocabulary predictability.

### Attack Run — After

Produced by `run_attacks.py::main` (`results/attack_run_2026-11-10_2309_after.md`):

| Metric | Before (0.6 / 0.4) | After (0.4 / 0.6) | Delta |
|---|---|---|---|
| **Total HELD** | 42 | **43** | +1 |
| **Total BROKE** | 14 | **13** | -1 |

#### By family comparison:
| Family | Total Attacks | Before HELD | Before BROKE | After HELD | After BROKE | Key Shift |
|---|---|---|---|---|---|---|
| `flood_same_creator` | 12 | 12 | 0 | 12 | 0 | Unchanged (rate limiter held) |
| `malformed` | 20 | 19 | 1 | 19 | 1 | Unchanged (RQ07 still 500) |
| `prompt_injection` | 6 | 5 | 1 | **6** | **0** | **PI02 flipped from BROKE (0.303, human) to HELD (0.352, unsure)** |
| `evasion` | 10 | 4 | 6 | 4 | 6 | EV01 shifted from 0.336 (human) to 0.421 (unsure) |
| `false_positive` | 8 | 2 | 6 | 2 | 6 | Scores lowered ~0.04-0.07, but FP02-FP07 remain > 0.65 |

### Criteria Evaluation — After
Produced by `run_eval.py::main` (`results/run_2026-11-10_2310_after.md`):
- **Criterion 1 (Personal writing):** Confidence dropped from 0.4474 & 0.5487 to 0.4209 & 0.4943 (still safely in `unsure`).
- **Criterion 2 (Score spread):** AI score moved 0.8067 → 0.7411; human score moved 0.2550 → 0.2761. Median spread narrowed from 0.5517 to 0.4650, still exceeding the 0.40 target.
- **Criterion 3 (Label coverage):** All 3 labels remained reachable (`ai`: 0.7411, `human`: 0.2761, `unsure`: 0.4259).
- **Criterion 5 (Bad input):** 4 of 4 malformed inputs still returned 400 with zero crashes.

**Did it help, and how do I know:**
It helped on prompt injection evasion, but failed to fix the false positives. I know this because PI02 moved from 0.303 (classified as human, allowing evasion) to 0.352 (classified as `unsure`), eliminating that failure and bringing total HELD attacks from 42 to 43. However, FP02–FP07 all still broke because their style scores were also moderately high (0.58–0.63); thus, increasing Signal 2's weight could not pull their combined scores below the 0.65 AI threshold (scored 0.659–0.695).

---

## What's Still Broken

- **FP02–FP07 (Formal human writing false positives):** Still falsely flagged as AI (scores 0.659–0.695). Both GPT-2 perplexity and simple stylometrics mistake formal, structured human English for AI generation. To fix this, I would raise the AI decision threshold from 0.65 to 0.75 or incorporate burstiness (inter-sentence perplexity variance). I stopped because Unit 8 restricts improvements to exactly one intervention.
- **EV03, EV04, EV06, EV09 (Evasion attacks landing in `unsure`):** AI text modified with quotations, rare vocabulary, or extreme sentence lengths dilutes the style score, pulling confidence down into `unsure` (0.42–0.58) rather than triggering AI. Fixing this would require semantic cohesion analysis and deeper syntactic parsing. I stopped to keep the single measured change focused on weights.
- **RQ07 (JSON array crash):** An HTTP POST body consisting of a JSON array  causes `payload.get()` in `app.py::submit` to raise an `AttributeError`, resulting in an unhandled 500 error. The fix is a one line type validation (`if not isinstance(payload, dict): return 400`). I stopped because Milestone 5 permits only one change, and signal reweighting addressed detection accuracy directly.
- **Criteria 1–4 Misses (Target sample sizes and eval harness):** Criteria 1, 2, and 3 were missed because `scenarios.py` only contained 2 to 3 test texts rather than the 30-text corpus specified in `criteria.md`. To fix this, I would populate `scenarios.py` with 30 diverse human essays and 30 AI generations. Criterion 4 was missed because `run_eval.py` does not test `/appeal` automatically. To fix this, I would extend the eval script to post 10 appeals. I stopped because manual curl verified the appeal path works, and the unit guidelines emphasize reporting underpowered trials honestly as MISSED rather than making ad-hoc changes.

**The trade I'd make if this ran for real:**
In a production deployment, I would raise the AI decision threshold from 0.65 to 0.75, routing borderline texts into the manual review queue so that no student or job applicant is falsely accused of cheating. The institution pays for this trade through increased human auditing labor, but it prevents the severe ethical cost of false accusations against honest human writers.

<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 7

       [ ] criteria.md has five numbered criteria, each naming a target
       [ ] Each has a reason underneath
       [ ] POST /submit returns content_id, guess, confidence, label,
           model_score and style_score
       [ ] Two signals that measure DIFFERENT things, each with a named blind spot
       [ ] Three label variants, all reachable, written out in full
       [ ] POST /appeal changes status and writes to the log
       [ ] Signals and Scoring names scoring.py::combine_signals
       [ ] Sample Run: a submission AND an appeal, as pasted text
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN

     SUBMISSION CHECKLIST — unit 8

       [ ] Rate limiting on /submit, with your numbers justified
       [ ] The pasted run of status codes showing 429
       [ ] Audit log with all nine fields, recording rejections too
       [ ] The attack set run IN FULL, every input with an outcome
       [ ] Ten audit log entries pasted as text
       [ ] Five criteria across three trials
       [ ] A diagnosis for every miss and every broken attack — stage AND mechanism
       [ ] One improvement, with Attack Run — After
       [ ] What's Still Broken, including the production trade
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
