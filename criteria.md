# Acceptance criteria — Provenance Guard

Five criteria that say what "working" means for this service, written in unit 7
**before** you built it.

**All five are yours.** None are given.

An acceptance criterion names a number or something plainly observable. *"The
detector is accurate"* is an opinion — and perfect AI detection isn't possible
anyway, so it's an opinion about something nobody can deliver. *"Of 10 pieces I
wrote myself, at most 1 is labelled high-confidence AI"* is a criterion.

Under each, write a sentence or two on **why that target**. *"I allow at most 1
false positive in 10 because on a writing site a wrong accusation costs more
than a missed detection"* is a real answer.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## Pick numbers you can defend

**Cover at least three of these five areas.** They're prompts, not a form.

| Area | A question it could answer |
|---|---|
| False positives | How often may a human's writing get called AI? |
| Score spread | How far apart should clearly-AI and clearly-human text score? |
| Label coverage | Is every one of your three labels actually reachable? |
| The appeal path | Does an appeal always change status and always get recorded? |
| Bad input | What happens when `text` is missing, empty, or enormous? |

Two things worth knowing before you pick numbers:

- **The expensive mistake is one-sided.** Missing some AI text costs the site a
  little. Accusing a real writer costs that person a lot. A criterion that
  treats both errors as equally bad is describing a system nobody would ship.
- **Unit 8 tests across three trials, and the target has to hold across all
  three.** Your first signal runs a model, so identical input can score
  differently twice. A target of "at most 1 in 10" against trials of 2, 1, 2 is
  a **miss**.

---
## 1. False positives

<!-- Your criterion. It must name a number. -->

Of 10 pieces I wrote myself,  including at least 3 written plainly and casually, at most 1 is labeled high-confidence AI.

**Why this target:**

Calling a real writer an AI is an expensive mistake. My plain Christmas paragraph scored 0.89 on the model signal, so this target is hard, and I chose it on purpose. 


---

## 2. Score spread

<!-- Your criterion. -->
Clearly-AI samples score at least 0.30 higher on average than clearly-human samples. 


**Why this target:**
If the two kinds of text score close together, the labels are basically guesses. Right now my clearly-human sample scored 0.47, which is almost a coin flip. 


---

## 3. Label coverage

<!-- Your criterion. -->
Each of the three labels (human, unsure, AI) is produced by at least 2 different real inputs. 


**Why this target:**
A label that can never appear is a gap in the ranges, and Milestone 5 requires all three to be reachable. 


---

## 4. The appeal path

<!-- Your criterion. -->
10 out of 10 appeals with a valid content_id change the status to under_review and appear in /log. 10 out of 10 appeals with a fake ID get a 404.


**Why this target:**
The appeal is the only thing between a wrong accusation and nothing at all, so it can't fail even once. 


---

## 5. Bad input

<!-- Your criterion. -->
Missing, empty, white-space only and very long (over 50,000 characters) text all return 4xx error with a message. None of them returns a 500 or crashes the server. 


**Why this target:**
A crash on bad input is one of the most common real failures. 


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
