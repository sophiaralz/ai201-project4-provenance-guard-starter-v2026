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

A piece of text goes through seven steps between arriving and getting a label:

1. **A writer sends it in.** A POST to `/submit` with JSON `{"text": "...", "creator_id": "..."}`. This adds the raw input and who sent it.
2. **The `/submit` handler checks it (`app.py::submit`).** If `text` is missing, empty, or not a string, it stops here with a 400 error. This adds a guarantee that everything after this point has real text to work with. 
3. **The model signal reads it (`detector.py::model_signal`).** gpt2 runs locally and measures how predictable the text is (perplexity), then squashes that onto 0–1. This adds `model_score`, where higher means more likely AI.
4. **The style signal measures it (`stylometry.py::style_signal`).** It looks at how much sentence length varies, how much vocabulary repeats, and how punctuation is used, without reading the meaning. This adds `style_score`, also 0–1 with a higher number meaning more likely AI. 
5. **The two scores are combined (`scoring.py::combine_signals`).** This adds one `confidence` number from 0 (confidently human) to 1 (confidently AI).
6. **The number becomes words (`scoring.py::score_to_label`).** The confidence falls into one of three bands. This adds `guess` (`"human"` / `"unsure"` / `"ai"`) and the `label` sentence a reader actually sees. 
7. **The decision is logged and returned (`audit.py::log_decision`).** The system gives the submission a unique `content_id`, writes it to the audit log with both signal scores, and sends everything back in the response. The `content_id` is what lets the writer appeal later through `/appeal`.

---

## Signals and Scoring

<!-- For each signal: what property it measures, why that might differ between
     human and AI writing, and WHAT IT CAN'T SEE.

     The blind spot is not optional. If you can't name it you don't understand
     the signal yet — and unit 8's attack set is built out of blind spots. -->

### Signal one — the model signal

**What it measures:** How predictable the text is to a language model (gpt2). The model reads the text one word at a time and we measure how surprised it is by each next word. This is called perplexity. Low perplexity means the text is easy to predict, which maps to a higher AI score. 

**Why that might differ between human and AI writing:** An AI writes by repeatedly choosing a likely next word, so its output is text a model finds easy to predict. People's writing is harder to predict: they make odd word choices, have "aha" moments, or suddenly change topic. That makes human text more surprising to the model. 

**What it can't see:** Predictable doesn't mean machine-written. Some people write very predictably just because they use common words in short sentences. This includes people writing in a second language who stick to words they're sure of, younger writers, and anyone writing plainly on purpose. It also fails the other way: AI text full of rare or fancy words surprises the model, so it scores as human.

### Signal two — the style signal

**What it measures:** The shape of the text, not its meaning. Three measures: how much sentence length varies, how much vocabulary repeats, and how much varied punctuation there is. They are combined into a 0-1 score, where a higher number means more likely AI. 

**Why that might differ between human and AI writing:** AI sentences tend to be even and medium-length, with safe vocabulary. Human writing is less regular: sentence length can vary drastically, and an uncommon word can show up suddenly.

**What it can't see:** It can't read, so it never sees the meaning. It has no idea what the text says, so anything that changes the shape without changing the substance moves the score. Someone could add typos or make some sentences shorter, and AI text would look more human.

### The combining rule

<!-- Write the rule in words, then give the numbers. What happens when the two
     signals disagree? -->

**The rule:** Average the two signals equally. But if they disagree by more than 0.30, cap the combined score at 0.60 so it can never reach "high-confidence AI". When two methods disagree that much, the honest answer is "unsure". The cost of false accusation is higher than the cost of letting some AI text through. 

**The numbers:**
- Model signal weight: 0.5
- Style signal weight: 0.5
- Disagreement gap: 0.30 (signals further apart than this count as disagreeing)
- Disagreement cap: 0.60 (the highest a disagreeing score can go)
- Below 0.35 = high-confidence human, 0.35-0.70 = unsure, above 0.70 = high-confidence AI

**Where it lives:** `scoring.py::combine_signals` (numbers in `config.py`)

<!-- ⚠️ The grader checks your code against that line. If your rule lives
     somewhere else, say where. -->

**A case where my two signals split, and what my rule does with it:**
My own Christmas paragraph ("I am so excited for Christmas! I can't wait to see my family...") got a model_score of 0.89, because the words are very common and easy to predict. Its style_score was only 0.56, because the sentence lengths vary a lot (6, 18, 8 and 5 words). A plain average gives 0.72, which would be labeled AI. The signals are 0.33 apart, more than the 0.30 gap, so the rule caps it at 0.60. That lands in "unsure" instead of "AI". I first planned a gap of 0.35, but testing showed this paragraph sits 0.33 apart and would have slipped through as AI, so I lowered the gap to 0.30. Without the cap, a small change in wording could push a real writer's casual note over the AI line. When I resubmitted it with "cannot" instead of "can't", the signals were 0.297 apart, just under the 0.30 gap, so the cap didn't apply and it scored 0.7035, which is labeled AI (see Sample Run). One word was the difference between "unsure" and an accusation. My rule moves the edge, but it doesn't remove it.

**Testing the rule on four inputs:**

| Input | model_score | style_score | confidence | Label |
|---|---|---|---|---|
| Clearly AI (`samples/clearly_ai.json`) | 0.82 | 0.70 | 0.76 | AI |
| Clearly human (`samples/clearly_human.json`) | 0.47 | 0.12 | 0.30 | human |
| Human writing, plain and conventional (my Christmas paragraph) | 0.89 | 0.56 | 0.60 (capped) | unsure |
| Lightly edited AI (the clearly-AI sample with simpler words and one sentence split in two) | 0.69 | 0.42 | 0.56 | unsure |

**What surprised me:** Two things surprised me. First, my real Christmas paragraph scored higher on the model signal (0.89) than the actual AI sample (0.82), and only the disagreement cap keeps it out of the AI label. Second, light editing was enough to dodge the AI label: swapping a few fancy words and splitting one sentence dropped the AI sample from 0.76 (AI) to 0.56 (unsure). My system is better at not accusing real writers than at catching edited AI, which is the trade-off I chose, but I didn't expect either gap to be that big.

---

## The False Positive

<!-- A real writer submits real work and the system calls it AI.
     What score do they get, what does the label say, what can they do? -->

**The writer:** someone writing a short, casual note about their own life, in plain sentences with common words. I used a paragraph I wrote myself:

> "I am so excited for Christmas! I can't wait to see my family and to drink hot chocolate and watch a movie with them. I also can't wait to see my cat. I miss her so much."

**The score they got:** `model_score` **0.89**, with a perplexity of 12.2. gpt2 found the text very easy to predict because phrases like "can't wait to see my family" and "hot chocolate" are some of the most common in English. For comparison, the repo's vivid clearly-human sample scored only 0.47 (perplexity 48.2), barely on the human side. Plain writing scored almost twice as "AI" as fancy writing, and both are human. (The style signal and the combining rule aren't built yet, so this is the model signal alone.)

**What the label says to them:** with a simple cutoff, this writer would read "This was written by AI." about their own Christmas note. That is an accusation, and it's wrong.

**What they can do:** appeal. They send their `content_id` and their reasoning to `/appeal`. The item moves to `under_review`, and a person looks at it instead of the detector. If the response didn't give them their `content_id`, they couldn't appeal at all, so the ID always has to be in the response.

**What this means for my thresholds:** a single strong signal can't be enough to call something AI. The high-confidence AI threshold has to sit high, and the "unsure" band has to be wide, so that ordinary writing like this lands in "unsure" rather than "AI". The style signal also needs to be able to pull a score like this down. The wording of the AI and unsure labels has to leave room for "you may well have written this yourself".

---

## Endpoint Contracts

<!-- What goes in and what comes out, field by field. Could someone else
     build against this without asking you a question? -->



### POST /submit

**Request** (JSON body):

| Field | Type | Required | Meaning |
|---|---|---|---|
| `text` | string | yes | The writing to check. Must not be empty or only spaces. |
| `creator_id` | string | no | Who submitted it. Defaults to `"unknown"`. |

**Response (200):**

| Field | Type | Meaning |
|---|---|---|
| `content_id` | string | Unique ID for this submission, like `"c_3f9a1b2c4d5e"`. Needed to appeal. |
| `creator_id` | string | Echoes who submitted it. |
| `guess` | string | One of `"human"`, `"unsure"`, `"ai"`. |
| `confidence` | number, 0–1 | Combined score. 0 = confidently human, 1 = confidently AI. |
| `label` | string | The sentence a reader actually sees. |
| `model_score` | number, 0–1 | Signal one (predictability). Higher = more AI. |
| `style_score` | number, 0–1 | Signal two (shape of the text). Higher = more AI. |

**Errors:**

| Status | When | Body |
|---|---|---|
| 400 | `text` missing, empty, or not a string | `{"error": "bad_request", "message": "..."}` |
| 400 | `text` too short to score (fewer than ~3 tokens) | `{"error": "too_short", "message": "..."}` |
| 413 | `text` too long (limit decided in Milestone 4) | `{"error": "too_long", "message": "..."}` |

### POST /appeal

**Request** (JSON body):

| Field | Type | Required | Meaning |
|---|---|---|---|
| `content_id` | string | yes | The ID returned by `/submit`. |
| `reasoning` | string | yes | Why the writer thinks the label is wrong. Must not be empty. |
| `creator_id` | string | no | Who is appealing. Defaults to `"unknown"`. |

**Response (200):**

| Field | Type | Meaning |
|---|---|---|
| `content_id` | string | Echoes the appealed submission. |
| `status` | string | Always `"under_review"`. |
| `message` | string | Confirmation for the writer, e.g. "Your appeal was received. A person will review it." |

**Errors:**

| Status | When | Body |
|---|---|---|
| 400 | `content_id` or `reasoning` missing or empty | `{"error": "bad_request", "message": "..."}` |
| 404 | No submission with that `content_id` | `{"error": "not_found", "message": "..."}` |

---

## Label Variants

<!-- The exact text of all three labels, as a reader would see them, and the
     score range each covers.

     Write them for someone who has never heard the word "threshold". -->

| Label | Score range | The exact text a reader sees |
|---|---|---|
| high-confidence human | 0.00 - 0.35 |This reads like it was written by a real person.  |
| unsure | 0.35 - 0.70 | We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it. |
| high-confidence AI | 0.70 - 1.00 | This shows strong signs of being written by AI. If you wrote it yourself, you can appeal and a person will review it. |

**Why I worded the "unsure" one this way:**
<!-- It's the hardest of the three. It has to admit uncertainty without
     sounding like an accusation, because the person reading it may well have
     written every word themselves. -->

I wrote "unsure" this way because I wanted to make sure that the person reading it doesn't think that I am accusing them of using AI. I let them know that this happens often because AI can't detect plain, everyday writing well from AI-written or human-written. 

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
  -d '{"text": "I am so excited for Christmas! I cannot wait to see my family and to drink hot chocolate and watch a movie with them. I also cannot wait to see my cat. I miss her so much.", "creator_id": "sophia"}'
```

```json
{
  "confidence": 0.7035,
  "content_id": "c_e6a6d3c121f4",
  "creator_id": "sophia",
  "guess": "ai",
  "label": "This shows strong signs of being written by AI. If you wrote it yourself, you can appeal and a person will review it.",
  "model_score": 0.8519,
  "style_score": 0.5551
}
```

This is my own writing, and it was labeled AI. This is the false positive from Milestone 1 happening for real, so it's the one I appealed.

**An appeal**

```bash
$ curl -X POST http://127.0.0.1:5000/appeal \
  -H "Content-Type: application/json" \
  -d '{"content_id": "c_e6a6d3c121f4", "reasoning": "I wrote this myself about my own Christmas plans."}'
```

```json
{
  "content_id": "c_e6a6d3c121f4",
  "message": "Your appeal was received. A person will review this decision.",
  "status": "under_review"
}
```

**What the log shows afterwards**

```bash
$ curl "http://127.0.0.1:5000/log?limit=2"
```

```json
{
  "count": 2,
  "entries": [
    {
      "combined_score": 0.7035,
      "content_id": "c_e6a6d3c121f4",
      "creator_id": "sophia",
      "guess": "ai",
      "label": "This shows strong signs of being written by AI. If you wrote it yourself, you can appeal and a person will review it.",
      "model_score": 0.8519,
      "status": "decided",
      "style_score": 0.5551,
      "timestamp": "2026-11-08T01:11:08+00:00"
    },
    {
      "content_id": "c_e6a6d3c121f4",
      "creator_id": "sophia",
      "event": "appeal",
      "reasoning": "I wrote this myself about my own Christmas plans.",
      "status": "under_review",
      "timestamp": "2026-11-08T01:13:29+00:00"
    }
  ]
}
```

The original decision stays in the log unchanged, and the appeal is added right after it with the status `under_review`.

---

## How I Used AI

**Moment 1**

- *What I asked for:* I asked Claude to give me an example text for the clearly_ai.json
- *What came back:* I got a text that is very obviously AI
- *What I changed:* When I ran the text with detector.py, the number was very close to 1 (meaning the detector determined it was AI written)

**Moment 2**

- *What I asked for:* I asked for help writing the three labels
- *What came back:* Draft sentences, including "our tools can be wrong" in the AI label, and "this is not an accusation" in the unsure label.
- *What I changed:* I compared against my fianl text in scoring.py. I dropped "Our tools can be wrong", changed "a person" to "a real person" in the human label to be more specific, and reworded the unsure label to make more sense for the average reader. 

<!-- ═══════════════════════ UNIT 8 — THE TEST ═══════════════════════ -->

---

## Rate Limiting

**My limits:** ___ per minute, ___ per day

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
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

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
