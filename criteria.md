# Acceptance criteria — Provenance Guard

Five criteria that say what "working" means for this service, written in unit 7
**before** you built it.

**All five are yours.** None are given.

An acceptance criterion names a number or something plainly observable. _"The
detector is accurate"_ is an opinion — and perfect AI detection isn't possible
anyway, so it's an opinion about something nobody can deliver. _"Of 10 pieces I
wrote myself, at most 1 is labelled high-confidence AI"_ is a criterion.

Under each, write a sentence or two on **why that target**. _"I allow at most 1
false positive in 10 because on a writing site a wrong accusation costs more
than a missed detection"_ is a real answer.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## Pick numbers you can defend

**Cover at least three of these five areas.** They're prompts, not a form.

| Area            | A question it could answer                                    |
| --------------- | ------------------------------------------------------------- |
| False positives | How often may a human's writing get called AI?                |
| Score spread    | How far apart should clearly-AI and clearly-human text score? |
| Label coverage  | Is every one of your three labels actually reachable?         |
| The appeal path | Does an appeal always change status and always get recorded?  |
| Bad input       | What happens when `text` is missing, empty, or enormous?      |

Two things worth knowing before you pick numbers:

- **The expensive mistake is one-sided.** Missing some AI text costs the site a
  little. Accusing a real writer costs that person a lot. A criterion that
  treats both errors as equally bad is describing a system nobody would ship.
- **Unit 8 tests across three trials, and the target has to hold across all
  three.** Your first signal runs a model, so identical input can score
  differently twice. A target of "at most 1 in 10" against trials of 2, 1, 2 is
  a **miss**.

---

## Criteria 1: False positive on human writing

Out of 30 human-written texts at most 3 can be labeled as“High-confidence AI” in each of 3 trials.

<!-- Your criterion. It must name a number. -->

**Why this target:**
In writing, a wrong accusation for writer costs more than a missed AI detection. So 10% of error is allowed because some human writing especially from second language writers can look machine like and 0 would be unrealistic.

---

## Criteria 2: score spread

On a 0 to 1 scale the median score of 30 AI texts is at least 0.4 higher than the median score of 30 human texts in each of 3 trials.

**Why this target:**
If the two groups (30 human texts and 30 AI texts) score close together, no threshold can separate them, and the labels would be guesses. A gap of 0.4 is wide enough to set useful cutoffs.

---

## Criteria 3: Label coverage

Each of the three labels is produced at least 3 times across the 30 text label set in each of 3 trials.

**Why this target:**
A label that never appears does nothing useful and does not give any useful information to readers, 3 appearances is minimum for a label to be meaningful.

---

## Criteria 4: Appeal path

In 10 test appeals, 10 of 10 create a log entry with a timestamp and a recorded decision in each of 3 trials and change status to 'under_review'

**Why this target:**
An appeal that is silently dropped or never logged gives the writer no information and appeal decision should not be automatic. So logging yes or no is must with no exceptions.

---

## Criteria 5: Bad Input

For texts under 20 words the system returns a “not enough text to score” message with no score and no crash in 3 of 3 cases, in each of 3 trials.

**Why this target:**
A short input gives the model almost nothing to judge, so the score it produced would be noise. A text which has words fewer than 20 rarely contains enough information to decide.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 8 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath, like this:

         ## 2. Uncertain cases land in the unsure band

         The system handles uncertain cases well.

         **Why this target:** ...

         > **Revised in unit 8:** Text I judged borderline scores between
         > 0.4 and 0.6 in at least 4 of 5 cases.
         >
         > **Why revised:** "well" gave me nothing to check. I couldn't
         > reach a verdict from it at all.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "At most 1 of 10 of my own pieces flagged as AI" → "at most 3
            of 10", because 1 of 10 turned out to be hard.

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->
