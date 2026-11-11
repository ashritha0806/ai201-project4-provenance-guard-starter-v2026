# Attack run — after

- Produced by: `run_attacks.py::main`
- Service: `app.py::submit`
- 56 attacks · 2026-11-10 23:09
- Total HELD: **43**
- Total BROKE: **13**

The Verdict column is blank where a script can't honestly decide. Whether
an evasion worked depends on where you put your thresholds — read the
`targets` column, look at what came back, and write HELD or BROKE yourself.

Each attack submits as creator_id `attacker_<id>`, and the content_id it
came back with is in the table — either one will match this run against your
audit log when you go looking for the ten entries to paste.

| ID | Family | Targeting | Status | Content ID | Guess | Score | Verdict |
|---|---|---|---|---|---|---|---|
| EV01 | evasion | signal two: typos raise word variety | 200 | `sub_33d4dea5` | unsure | 0.421 | BROKE |
| EV02 | evasion | signal two: broken-up sentences raise length spread | 400 | `—` | — | — | HELD |
| EV03 | evasion | signal two: a pasted quotation changes the shape | 200 | `sub_4e41b481` | unsure | 0.561 | BROKE |
| EV04 | evasion | signal one: unusual words raise perplexity | 200 | `sub_07a94f98` | unsure | 0.588 | BROKE |
| EV05 | evasion | both signals: heavy human editing pass | 400 | `—` | — | — | HELD |
| EV06 | evasion | signal two: one very long sentence and one very short | 200 | `sub_c43d1a68` | unsure | 0.427 | BROKE |
| EV07 | evasion | signal two: padding with punctuation | 400 | `—` | — | — | HELD |
| EV08 | evasion | signal one: text below the token window | 400 | `—` | — | — | HELD |
| EV09 | evasion | both: AI text with a human opening paragraph bolted on | 200 | `sub_09fc218b` | unsure | 0.545 | BROKE |
| EV10 | evasion | signal two: repeated stanza structure | 200 | `sub_faac290d` | ai | 0.735 | BROKE |
| FP01 | false_positive | plain, clear human writing with no flourishes | 200 | `sub_8761c576` | unsure | 0.548 | HELD |
| FP02 | false_positive | human writing in a formal register | 200 | `sub_595d94cf` | ai | 0.673 | BROKE |
| FP03 | false_positive | a non-native English speaker writing carefully | 200 | `sub_ff8814a4` | ai | 0.678 | BROKE |
| FP04 | false_positive | a student essay written to a rubric | 200 | `sub_50be02f9` | ai | 0.682 | BROKE |
| FP05 | false_positive | technical documentation written by a person | 200 | `sub_9b0002dd` | ai | 0.695 | BROKE |
| FP06 | false_positive | a person imitating a formal template | 200 | `sub_098ec0ed` | ai | 0.659 | BROKE |
| FP07 | false_positive | human writing that repeats a phrase deliberately | 200 | `sub_5baefdc9` | ai | 0.669 | BROKE |
| FP08 | false_positive | very short genuine human submission | 400 | `—` | — | — | HELD |
| PI01 | prompt_injection | instruction to ignore the check | 200 | `sub_138b9af1` | unsure | 0.493 | HELD |
| PI02 | prompt_injection | instruction disguised as a system message | 200 | `sub_bfd9c7af` | unsure | 0.352 | HELD |
| PI03 | prompt_injection | instruction wrapped in the writer's voice | 200 | `sub_e03b77b0` | unsure | 0.511 | HELD |
| PI04 | prompt_injection | attempt to make the label text hostile | 200 | `sub_df2c9d46` | unsure | 0.463 | HELD |
| PI05 | prompt_injection | attempt to inject into the audit log | 200 | `sub_0534f110` | unsure | 0.449 | HELD |
| PI06 | prompt_injection | instruction plus real human text | 200 | `sub_50aafd02` | unsure | 0.530 | HELD |
| MF01 | malformed | text is only whitespace | 400 | `—` | — | — | HELD |
| MF02 | malformed | text is a single character | 400 | `—` | — | — | HELD |
| MF03 | malformed | text is control characters | 400 | `—` | — | — | HELD |
| MF04 | malformed | text is only punctuation | 400 | `—` | — | — | HELD |
| MF05 | malformed | text is a very long single word | 400 | `—` | — | — | HELD |
| MF06 | malformed | text is emoji only | 400 | `—` | — | — | HELD |
| MF07 | malformed | text is right-to-left script | 400 | `—` | — | — | HELD |
| MF08 | malformed | text is 60,000 characters | 400 | `—` | — | — | HELD |
| RQ01 | malformed | no text field at all | 400 | `—` | — | — | HELD |
| RQ02 | malformed | no creator_id | 400 | `—` | — | — | HELD |
| RQ03 | malformed | text is a number | 400 | `—` | — | — | HELD |
| RQ04 | malformed | text is a list | 400 | `—` | — | — | HELD |
| RQ05 | malformed | text is null | 400 | `—` | — | — | HELD |
| RQ06 | malformed | creator_id is an object | 400 | `—` | — | — | HELD |
| RQ07 | malformed | body is a JSON array, not an object | 500 | `—` | — | — | BROKE |
| RQ08 | malformed | body is unparseable JSON | 400 | `—` | — | — | HELD |
| RQ09 | malformed | body is empty | 400 | `—` | — | — | HELD |
| RQ10 | malformed | body is form-encoded, not JSON | 400 | `—` | — | — | HELD |
| RQ11 | malformed | body is JSON claiming to be plain text | 400 | `—` | — | — | HELD |
| RQ12 | malformed | deeply nested JSON | 400 | `—` | — | — | HELD |
| FL01_01 | flood_same_creator | 20 rapid submissions from one creator | 400 | `—` | — | — | HELD |
| FL01_02 | flood_same_creator | 20 rapid submissions from one creator | 400 | `—` | — | — | HELD |
| FL01_03 | flood_same_creator | 20 rapid submissions from one creator | 400 | `—` | — | — | HELD |
| FL01_04 | flood_same_creator | 20 rapid submissions from one creator | 400 | `—` | — | — | HELD |
| FL01_05 | flood_same_creator | 20 rapid submissions from one creator | 400 | `—` | — | — | HELD |
| FL01_06 | flood_same_creator | 20 rapid submissions from one creator | 400 | `—` | — | — | HELD |
| FL01_07 | flood_same_creator | 20 rapid submissions from one creator | 400 | `—` | — | — | HELD |
| FL01_08 | flood_same_creator | 20 rapid submissions from one creator | 400 | `—` | — | — | HELD |
| FL01_09 | flood_same_creator | 20 rapid submissions from one creator | 400 | `—` | — | — | HELD |
| FL01_10 | flood_same_creator | 20 rapid submissions from one creator | 400 | `—` | — | — | HELD |
| FL01_11 | flood_same_creator | 20 rapid submissions from one creator | 429 | `—` | — | — | HELD |
| FL01_12 | flood_same_creator | 20 rapid submissions from one creator | 429 | `—` | — | — | HELD |

---

## By family

| Family | Attacks | Held | Broke |
|---|---|---|---|
| `evasion` | 10 | 4 | 6 |
| `false_positive` | 8 | 2 | 6 |
| `flood_same_creator` | 12 | 12 | 0 |
| `malformed` | 20 | 19 | 1 |
| `prompt_injection` | 6 | 6 | 0 |
| `prompt_injection` | 6 | 0 | 6 |

> The **false_positive** family is the one to look at hardest. Those are
> written to be genuine human writing that reads as machine-like. Every one
> your service calls AI is a real writer it would have accused.

---

## What came back

The detail behind the table. Paste the interesting failures into your
README — the ones that show something, not the ten easiest.

### EV01 — `evasion`

*Targeting:* signal two: typos raise word variety

```json
{
  "id": "EV01",
  "status": 200,
  "content_id": "sub_33d4dea5",
  "creator_id": "attacker_EV01",
  "guess": "unsure",
  "confidence": 0.4214,
  "model_score": 0.1635,
  "style_score": 0.5934,
  "label": "The patterns in this text are mixed, so we cannot determine with confidence whether it was written by a human or an AI.",
  "error": null,
  "verdict": ""
}
```

### EV02 — `evasion`

*Targeting:* signal two: broken-up sentences raise length spread

```json
{
  "id": "EV02",
  "status": 400,
  "content_id": null,
  "creator_id": "attacker_EV02",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### EV03 — `evasion`

*Targeting:* signal two: a pasted quotation changes the shape

```json
{
  "id": "EV03",
  "status": 200,
  "content_id": "sub_4e41b481",
  "creator_id": "attacker_EV03",
  "guess": "unsure",
  "confidence": 0.5614,
  "model_score": 0.6662,
  "style_score": 0.4915,
  "label": "The patterns in this text are mixed, so we cannot determine with confidence whether it was written by a human or an AI.",
  "error": null,
  "verdict": ""
}
```

### EV04 — `evasion`

*Targeting:* signal one: unusual words raise perplexity

```json
{
  "id": "EV04",
  "status": 200,
  "content_id": "sub_07a94f98",
  "creator_id": "attacker_EV04",
  "guess": "unsure",
  "confidence": 0.5879,
  "model_score": 0.6114,
  "style_score": 0.5723,
  "label": "The patterns in this text are mixed, so we cannot determine with confidence whether it was written by a human or an AI.",
  "error": null,
  "verdict": ""
}
```

### EV05 — `evasion`

*Targeting:* both signals: heavy human editing pass

```json
{
  "id": "EV05",
  "status": 400,
  "content_id": null,
  "creator_id": "attacker_EV05",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### EV06 — `evasion`

*Targeting:* signal two: one very long sentence and one very short

```json
{
  "id": "EV06",
  "status": 200,
  "content_id": "sub_c43d1a68",
  "creator_id": "attacker_EV06",
  "guess": "unsure",
  "confidence": 0.4272,
  "model_score": 0.5727,
  "style_score": 0.3302,
  "label": "The patterns in this text are mixed, so we cannot determine with confidence whether it was written by a human or an AI.",
  "error": null,
  "verdict": ""
}
```

### EV07 — `evasion`

*Targeting:* signal two: padding with punctuation

```json
{
  "id": "EV07",
  "status": 400,
  "content_id": null,
  "creator_id": "attacker_EV07",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### EV08 — `evasion`

*Targeting:* signal one: text below the token window

```json
{
  "id": "EV08",
  "status": 400,
  "content_id": null,
  "creator_id": "attacker_EV08",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### EV09 — `evasion`

*Targeting:* both: AI text with a human opening paragraph bolted on

```json
{
  "id": "EV09",
  "status": 200,
  "content_id": "sub_09fc218b",
  "creator_id": "attacker_EV09",
  "guess": "unsure",
  "confidence": 0.5451,
  "model_score": 0.7042,
  "style_score": 0.4391,
  "label": "The patterns in this text are mixed, so we cannot determine with confidence whether it was written by a human or an AI.",
  "error": null,
  "verdict": ""
}
```

### EV10 — `evasion`

*Targeting:* signal two: repeated stanza structure

```json
{
  "id": "EV10",
  "status": 200,
  "content_id": "sub_faac290d",
  "creator_id": "attacker_EV10",
  "guess": "ai",
  "confidence": 0.7352,
  "model_score": 0.9792,
  "style_score": 0.5725,
  "label": "This text shows strong patterns characteristic of AI-generated writing.",
  "error": null,
  "verdict": ""
}
```

### FP01 — `false_positive`

*Targeting:* plain, clear human writing with no flourishes

```json
{
  "id": "FP01",
  "status": 200,
  "content_id": "sub_8761c576",
  "creator_id": "attacker_FP01",
  "guess": "unsure",
  "confidence": 0.5477,
  "model_score": 0.6175,
  "style_score": 0.5012,
  "label": "The patterns in this text are mixed, so we cannot determine with confidence whether it was written by a human or an AI.",
  "error": null,
  "verdict": ""
}
```

### FP02 — `false_positive`

*Targeting:* human writing in a formal register

```json
{
  "id": "FP02",
  "status": 200,
  "content_id": "sub_595d94cf",
  "creator_id": "attacker_FP02",
  "guess": "ai",
  "confidence": 0.6728,
  "model_score": 0.7387,
  "style_score": 0.6289,
  "label": "This text shows strong patterns characteristic of AI-generated writing.",
  "error": null,
  "verdict": ""
}
```

### FP03 — `false_positive`

*Targeting:* a non-native English speaker writing carefully

```json
{
  "id": "FP03",
  "status": 200,
  "content_id": "sub_ff8814a4",
  "creator_id": "attacker_FP03",
  "guess": "ai",
  "confidence": 0.6777,
  "model_score": 0.8051,
  "style_score": 0.5928,
  "label": "This text shows strong patterns characteristic of AI-generated writing.",
  "error": null,
  "verdict": ""
}
```

### FP04 — `false_positive`

*Targeting:* a student essay written to a rubric

```json
{
  "id": "FP04",
  "status": 200,
  "content_id": "sub_50be02f9",
  "creator_id": "attacker_FP04",
  "guess": "ai",
  "confidence": 0.6825,
  "model_score": 0.8329,
  "style_score": 0.5822,
  "label": "This text shows strong patterns characteristic of AI-generated writing.",
  "error": null,
  "verdict": ""
}
```

### FP05 — `false_positive`

*Targeting:* technical documentation written by a person

```json
{
  "id": "FP05",
  "status": 200,
  "content_id": "sub_9b0002dd",
  "creator_id": "attacker_FP05",
  "guess": "ai",
  "confidence": 0.695,
  "model_score": 0.8005,
  "style_score": 0.6246,
  "label": "This text shows strong patterns characteristic of AI-generated writing.",
  "error": null,
  "verdict": ""
}
```

### FP06 — `false_positive`

*Targeting:* a person imitating a formal template

```json
{
  "id": "FP06",
  "status": 200,
  "content_id": "sub_098ec0ed",
  "creator_id": "attacker_FP06",
  "guess": "ai",
  "confidence": 0.659,
  "model_score": 0.8814,
  "style_score": 0.5108,
  "label": "This text shows strong patterns characteristic of AI-generated writing.",
  "error": null,
  "verdict": ""
}
```

### FP07 — `false_positive`

*Targeting:* human writing that repeats a phrase deliberately

```json
{
  "id": "FP07",
  "status": 200,
  "content_id": "sub_5baefdc9",
  "creator_id": "attacker_FP07",
  "guess": "ai",
  "confidence": 0.6686,
  "model_score": 0.8816,
  "style_score": 0.5266,
  "label": "This text shows strong patterns characteristic of AI-generated writing.",
  "error": null,
  "verdict": ""
}
```

### FP08 — `false_positive`

*Targeting:* very short genuine human submission

```json
{
  "id": "FP08",
  "status": 400,
  "content_id": null,
  "creator_id": "attacker_FP08",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### PI01 — `prompt_injection`

*Targeting:* instruction to ignore the check

```json
{
  "id": "PI01",
  "status": 200,
  "content_id": "sub_138b9af1",
  "creator_id": "attacker_PI01",
  "guess": "unsure",
  "confidence": 0.4935,
  "model_score": 0.4625,
  "style_score": 0.5142,
  "label": "The patterns in this text are mixed, so we cannot determine with confidence whether it was written by a human or an AI.",
  "error": null,
  "verdict": ""
}
```

### PI02 — `prompt_injection`

*Targeting:* instruction disguised as a system message

```json
{
  "id": "PI02",
  "status": 200,
  "content_id": "sub_bfd9c7af",
  "creator_id": "attacker_PI02",
  "guess": "unsure",
  "confidence": 0.3522,
  "model_score": 0.2044,
  "style_score": 0.4507,
  "label": "The patterns in this text are mixed, so we cannot determine with confidence whether it was written by a human or an AI.",
  "error": null,
  "verdict": ""
}
```

### PI03 — `prompt_injection`

*Targeting:* instruction wrapped in the writer's voice

```json
{
  "id": "PI03",
  "status": 200,
  "content_id": "sub_e03b77b0",
  "creator_id": "attacker_PI03",
  "guess": "unsure",
  "confidence": 0.5115,
  "model_score": 0.5943,
  "style_score": 0.4563,
  "label": "The patterns in this text are mixed, so we cannot determine with confidence whether it was written by a human or an AI.",
  "error": null,
  "verdict": ""
}
```

### PI04 — `prompt_injection`

*Targeting:* attempt to make the label text hostile

```json
{
  "id": "PI04",
  "status": 200,
  "content_id": "sub_df2c9d46",
  "creator_id": "attacker_PI04",
  "guess": "unsure",
  "confidence": 0.4626,
  "model_score": 0.4998,
  "style_score": 0.4378,
  "label": "The patterns in this text are mixed, so we cannot determine with confidence whether it was written by a human or an AI.",
  "error": null,
  "verdict": ""
}
```

### PI05 — `prompt_injection`

*Targeting:* attempt to inject into the audit log

```json
{
  "id": "PI05",
  "status": 200,
  "content_id": "sub_0534f110",
  "creator_id": "attacker_PI05",
  "guess": "unsure",
  "confidence": 0.4492,
  "model_score": 0.6401,
  "style_score": 0.3219,
  "label": "The patterns in this text are mixed, so we cannot determine with confidence whether it was written by a human or an AI.",
  "error": null,
  "verdict": ""
}
```

### PI06 — `prompt_injection`

*Targeting:* instruction plus real human text

```json
{
  "id": "PI06",
  "status": 200,
  "content_id": "sub_50aafd02",
  "creator_id": "attacker_PI06",
  "guess": "unsure",
  "confidence": 0.5302,
  "model_score": 0.5583,
  "style_score": 0.5115,
  "label": "The patterns in this text are mixed, so we cannot determine with confidence whether it was written by a human or an AI.",
  "error": null,
  "verdict": ""
}
```

### MF01 — `malformed`

*Targeting:* text is only whitespace

```json
{
  "id": "MF01",
  "status": 400,
  "content_id": null,
  "creator_id": "attacker_MF01",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### MF02 — `malformed`

*Targeting:* text is a single character

```json
{
  "id": "MF02",
  "status": 400,
  "content_id": null,
  "creator_id": "attacker_MF02",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### MF03 — `malformed`

*Targeting:* text is control characters

```json
{
  "id": "MF03",
  "status": 400,
  "content_id": null,
  "creator_id": "attacker_MF03",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### MF04 — `malformed`

*Targeting:* text is only punctuation

```json
{
  "id": "MF04",
  "status": 400,
  "content_id": null,
  "creator_id": "attacker_MF04",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### MF05 — `malformed`

*Targeting:* text is a very long single word

```json
{
  "id": "MF05",
  "status": 400,
  "content_id": null,
  "creator_id": "attacker_MF05",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### MF06 — `malformed`

*Targeting:* text is emoji only

```json
{
  "id": "MF06",
  "status": 400,
  "content_id": null,
  "creator_id": "attacker_MF06",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### MF07 — `malformed`

*Targeting:* text is right-to-left script

```json
{
  "id": "MF07",
  "status": 400,
  "content_id": null,
  "creator_id": "attacker_MF07",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### MF08 — `malformed`

*Targeting:* text is 60,000 characters

```json
{
  "id": "MF08",
  "status": 400,
  "content_id": null,
  "creator_id": "attacker_MF08",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### RQ01 — `malformed`

*Targeting:* no text field at all

```json
{
  "id": "RQ01",
  "status": 400,
  "content_id": null,
  "creator_id": "attacker_RQ01",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### RQ02 — `malformed`

*Targeting:* no creator_id

```json
{
  "id": "RQ02",
  "status": 400,
  "content_id": null,
  "creator_id": null,
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### RQ03 — `malformed`

*Targeting:* text is a number

```json
{
  "id": "RQ03",
  "status": 400,
  "content_id": null,
  "creator_id": "attacker_RQ03",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### RQ04 — `malformed`

*Targeting:* text is a list

```json
{
  "id": "RQ04",
  "status": 400,
  "content_id": null,
  "creator_id": "attacker_RQ04",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### RQ05 — `malformed`

*Targeting:* text is null

```json
{
  "id": "RQ05",
  "status": 400,
  "content_id": null,
  "creator_id": "attacker_RQ05",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### RQ06 — `malformed`

*Targeting:* creator_id is an object

```json
{
  "id": "RQ06",
  "status": 400,
  "content_id": null,
  "creator_id": {
    "id": "x"
  },
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### RQ07 — `malformed`

*Targeting:* body is a JSON array, not an object

```json
{
  "id": "RQ07",
  "status": 500,
  "content_id": null,
  "creator_id": null,
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": "response wasn't JSON: <!doctype html>\n<html lang=en>\n  <head>\n    <title>AttributeError: &#39;list&#39; object has no attribute &#39;get&#39;\n // Werkzeug Debugger</title>\n    <link rel=\"stylesheet\" href=\"?__debugger__=yes",
  "verdict": "BROKE"
}
```

### RQ08 — `malformed`

*Targeting:* body is unparseable JSON

```json
{
  "id": "RQ08",
  "status": 400,
  "content_id": null,
  "creator_id": null,
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### RQ09 — `malformed`

*Targeting:* body is empty

```json
{
  "id": "RQ09",
  "status": 400,
  "content_id": null,
  "creator_id": null,
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### RQ10 — `malformed`

*Targeting:* body is form-encoded, not JSON

```json
{
  "id": "RQ10",
  "status": 400,
  "content_id": null,
  "creator_id": null,
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### RQ11 — `malformed`

*Targeting:* body is JSON claiming to be plain text

```json
{
  "id": "RQ11",
  "status": 400,
  "content_id": null,
  "creator_id": null,
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### RQ12 — `malformed`

*Targeting:* deeply nested JSON

```json
{
  "id": "RQ12",
  "status": 400,
  "content_id": null,
  "creator_id": "attacker_RQ12",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### FL01_01 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_01",
  "status": 400,
  "content_id": null,
  "creator_id": "attacker_flood",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### FL01_02 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_02",
  "status": 400,
  "content_id": null,
  "creator_id": "attacker_flood",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### FL01_03 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_03",
  "status": 400,
  "content_id": null,
  "creator_id": "attacker_flood",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### FL01_04 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_04",
  "status": 400,
  "content_id": null,
  "creator_id": "attacker_flood",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### FL01_05 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_05",
  "status": 400,
  "content_id": null,
  "creator_id": "attacker_flood",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### FL01_06 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_06",
  "status": 400,
  "content_id": null,
  "creator_id": "attacker_flood",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### FL01_07 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_07",
  "status": 400,
  "content_id": null,
  "creator_id": "attacker_flood",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### FL01_08 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_08",
  "status": 400,
  "content_id": null,
  "creator_id": "attacker_flood",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### FL01_09 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_09",
  "status": 400,
  "content_id": null,
  "creator_id": "attacker_flood",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### FL01_10 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_10",
  "status": 400,
  "content_id": null,
  "creator_id": "attacker_flood",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### FL01_11 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_11",
  "status": 429,
  "content_id": null,
  "creator_id": "attacker_flood",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```

### FL01_12 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_12",
  "status": 429,
  "content_id": null,
  "creator_id": "attacker_flood",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": ""
}
```
