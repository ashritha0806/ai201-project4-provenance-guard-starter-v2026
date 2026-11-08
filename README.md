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

A writer submits an essay that they wrote themselves, in plain even sentences. Signal one reads the plain vocabulary as predictable, so it leans AI. Signal two reads the even sentence length as machine like, so it leans AI too. The combined confidence lands in the AI range, so the response label says the work is likely AI generated. The writer sees only the label and has no access to the underlying scores. Their only option is POST /appeal with their content_id and reasoning. That moves the item to under_review.

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
  -d '{"text": "The implementation of the proposed framework requires careful consideration of several key factors.", "creator_id": "student_1"}'
```

```bash
$ curl -X POST http://127.0.0.1:5000/submit \
  -H "Content-Type: application/json" \
  -d '{"text": "Choosing a laptop comes down to your priorities. If you travel often, portability and battery life matter most, while heavy gaming or video editing calls for stronger processing power and a better graphics card.", "creator_id": "student_1"}'
```

```json
{
  "confidence": 0.5563,
  "content_id": "sub_f257ae1c",
  "guess": "ai",
  "label": "unsure",
  "model_score": 0.5563,
  "style_score": 0.0
}

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
  "count": 2,
  "entries": [
    {
      "combined_score": 0.5563,
      "content_id": "sub_e25f5d26",
      "creator_id": "student_1",
      "guess": "ai",
      "label": "unsure",
      "model_score": 0.5563,
      "status": "decided",
      "style_score": 0.0,
      "timestamp": "2026-11-06T03:16:55+00:00"
    },
    {
      "combined_score": 0.5563,
      "content_id": "sub_f257ae1c",
      "creator_id": "student_1",
      "guess": "ai",
      "label": "unsure",
      "model_score": 0.5563,
      "status": "decided",
      "style_score": 0.0,
      "timestamp": "2026-11-06T18:27:26+00:00"
    }
  ]
}
```

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

<!-- ═══════════════════════ UNIT 8 — THE TEST ═══════════════════════ -->

---

## Rate Limiting

**My limits:** **_ per minute, _** per day

**Why those numbers and not others:**

<!-- Think about two people: a real writer submitting their own work a few
     times an hour, and a script sending a thousand variations to map your
     thresholds. Your numbers have to be liveable for the first and hostile to
     the second. -->

**What a caller counts as:** <!-- per address, or per creator_id? They fail
differently — one script can look like a thousand callers, and one household
can look like one. -->

**The run of status codes when I pushed past it:**

<!-- python run_attacks.py --flood 15 -->

```

```

---

## Attack Run — Before

<!-- Every attack with its outcome and a held-or-broke mark.
     `run_attacks.py --label before` produces the table. -->

**Ten audit log entries from the run**

<!-- Choose entries that show the interesting failures, not the ten easiest. -->

```json

```

**The single worst outcome** — the one I'd least want to explain to a writer
whose work got caught by it:

---

## Run Log and Verdicts

<!-- Your five criteria across three trials. `run_eval.py --label before`. -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
| --------- | ------ | ----- | ----- | ----- | ------- |
| 1.        |        |       |       |       |         |
| 2.        |        |       |       |       |         |
| 3.        |        |       |       |       |         |
| 4.        |        |       |       |       |         |
| 5.        |        |       |       |       |         |

**Real output from one trial**, pasted as text, naming the file and function
that produced it:

```json

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

---

## The Improvement

**What I changed:**

**Which diagnosis pointed at it:**

### Attack Run — After

<!-- Same format. `run_attacks.py --label after` -->

**Did it help, and how do I know:**

<!-- Widening the unsure band usually fixes one problem and creates another.
     Reporting that trade honestly is worth full credit. -->

---

## What's Still Broken

<!-- For each attack still getting through and each criterion still missed:
     what you'd do, and why you stopped. -->

**The trade I'd make if this ran for real:**

<!-- Two sentences. Fewer wrong accusations means more AI text getting through.
     Say which way you'd go and WHO PAYS FOR IT. -->

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
