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
- Below 0.35 = high-confidence human, 0.35-0.80 = unsure, above 0.80 = high-confidence AI (0.70 before the Milestone 5 change)

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
| unsure | 0.35 - 0.80 (0.70 before the Milestone 5 change) | We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it. |
| high-confidence AI | 0.80 - 1.00 (0.70 before the Milestone 5 change) | This shows strong signs of being written by AI. If you wrote it yourself, you can appeal and a person will review it. |

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

**Moment 3**

- *What I asked for:* I asked for help building scenarios.py for my five criteria.
- *What came back:* Scenarios for four criteria, AI-written paragraphs for the AI side, and an extension to run_eval.py so the appeal path could be tested automatically.
- *What I changed:* I wrote all 10 pieces for criterion 1 myself and chose the Christmas paragraph as my most personal one. 

<!-- ═══════════════════════ UNIT 8 — THE TEST ═══════════════════════ -->

---

## Rate Limiting


**My limits:** _5_ per minute, _50_ per day

**Why those numbers and not others:**
<!-- Think about two people: a real writer submitting their own work a few
     times an hour, and a script sending a thousand variations to map your
     thresholds. Your numbers have to be liveable for the first and hostile to
     the second. -->

A real writer checking their own work rarely sends more than a few pieces a minute or a dozen a day, so 5 a minute and 50 a day never gets in their way. A script trying to find my thresholds needs about 1,000 tries, and at 50 a day that takes 20 days. Going any lower would start blocking honest writers.


**What a caller counts as:** <!-- per address, or per creator_id? They fail
differently — one script can look like a thousand callers, and one household
can look like one. -->

Per creator_id, fall back to IP address if there isn't one. The weakness is that one script can make up a new creator_id for every request and look like a thousand different callers. 

**The run of status codes when I pushed past it:**

<!-- python run_attacks.py --flood 15 -->

```

200 200 200 200 200 429 429 429 429 429 429 429 429 429 429

```

---

## Attack Run — Before

<!-- Every attack with its outcome and a held-or-broke mark.
     `run_attacks.py --label before` produces the table. -->

56 attacks · 2026-11-18 · HELD **36**, BROKE **20**

| ID | Family | Targeting | Status | Content ID | Guess | Score | Verdict |
|---|---|---|---|---|---|---|---|
| EV01 | evasion | signal two: typos raise word variety | 200 | `c_64a6bdeef030` | unsure | 0.553 | BROKE |
| EV02 | evasion | signal two: broken-up sentences raise length spread | 200 | `c_66f173c1160f` | unsure | 0.474 | BROKE |
| EV03 | evasion | signal two: a pasted quotation changes the shape | 200 | `c_1fca7ef10e9e` | unsure | 0.636 | BROKE |
| EV04 | evasion | signal one: unusual words raise perplexity | 200 | `c_19d0cef5b2ec` | ai | 0.759 | HELD |
| EV05 | evasion | both signals: heavy human editing pass | 200 | `c_db9e9e918037` | human | 0.331 | BROKE |
| EV06 | evasion | signal two: one very long sentence and one very short | 200 | `c_b0ee05a697ea` | unsure | 0.482 | BROKE |
| EV07 | evasion | signal two: padding with punctuation | 200 | `c_0de92409b367` | unsure | 0.443 | BROKE |
| EV08 | evasion | signal one: text below the token window | 200 | `c_40d6dc1b7ec6` | unsure | 0.581 | BROKE |
| EV09 | evasion | both: AI text with a human opening paragraph bolted on | 200 | `c_7a1e4a2b2e9c` | unsure | 0.678 | BROKE |
| EV10 | evasion | signal two: repeated stanza structure | 200 | `c_3963073517bb` | ai | 0.893 | HELD |
| FP01 | false_positive | plain, clear human writing with no flourishes | 200 | `c_a108d9426adb` | unsure | 0.680 | HELD |
| FP02 | false_positive | human writing in a formal register | 200 | `c_07ef3d13fa7d` | ai | 0.849 | BROKE |
| FP03 | false_positive | a non-native English speaker writing carefully | 200 | `c_d7eeaab16168` | ai | 0.839 | BROKE |
| FP04 | false_positive | a student essay written to a rubric | 200 | `c_5f142483f3ba` | ai | 0.833 | BROKE |
| FP05 | false_positive | technical documentation written by a person | 200 | `c_ac19450dda50` | ai | 0.862 | BROKE |
| FP06 | false_positive | a person imitating a formal template | 200 | `c_aa90f0f68783` | ai | 0.781 | BROKE |
| FP07 | false_positive | human writing that repeats a phrase deliberately | 200 | `c_3b3f69189978` | unsure | 0.600 | HELD |
| FP08 | false_positive | very short genuine human submission | 200 | `c_b1face385627` | unsure | 0.545 | HELD |
| PI01 | prompt_injection | instruction to ignore the check | 200 | `c_0b01643e414f` | unsure | 0.500 | HELD |
| PI02 | prompt_injection | instruction disguised as a system message | 200 | `c_174446767801` | unsure | 0.352 | HELD |
| PI03 | prompt_injection | instruction wrapped in the writer's voice | 200 | `c_1bbb2538c571` | unsure | 0.685 | HELD |
| PI04 | prompt_injection | attempt to make the label text hostile | 200 | `c_c73fca1a9c3d` | unsure | 0.571 | HELD |
| PI05 | prompt_injection | attempt to inject into the audit log | 200 | `c_ef40e633457e` | unsure | 0.484 | HELD |
| PI06 | prompt_injection | instruction plus real human text | 200 | `c_ffe13ff56d80` | unsure | 0.656 | HELD |
| MF01 | malformed | text is only whitespace | 400 | `—` | — | — | HELD |
| MF02 | malformed | text is a single character | 400 | `—` | — | — | HELD |
| MF03 | malformed | text is control characters | 200 | `c_889475f4850e` | human | 0.350 | BROKE |
| MF04 | malformed | text is only punctuation | 200 | `c_69c77c3a9293` | unsure | 0.356 | HELD |
| MF05 | malformed | text is a very long single word | 200 | `c_79abb8dcc7eb` | ai | 0.849 | BROKE |
| MF06 | malformed | text is emoji only | 200 | `c_9cbd58ff32fd` | ai | 0.848 | BROKE |
| MF07 | malformed | text is right-to-left script | 200 | `c_51656690bd7c` | ai | 0.803 | BROKE |
| MF08 | malformed | text is 60,000 characters | 200 | `c_f98dcd8f6783` | ai | 0.880 | BROKE |
| RQ01 | malformed | no text field at all | 400 | `—` | — | — | HELD |
| RQ02 | malformed | no creator_id | 200 | `c_bb38c5887a72` | unsure | 0.470 | HELD |
| RQ03 | malformed | text is a number | 400 | `—` | — | — | HELD |
| RQ04 | malformed | text is a list | 400 | `—` | — | — | HELD |
| RQ05 | malformed | text is null | 400 | `—` | — | — | HELD |
| RQ06 | malformed | creator_id is an object | 200 | `c_e63ab6e85d88` | unsure | 0.375 | BROKE |
| RQ07 | malformed | body is a JSON array, not an object | 500 | `—` | — | — | BROKE |
| RQ08 | malformed | body is unparseable JSON | 400 | `—` | — | — | HELD |
| RQ09 | malformed | body is empty | 400 | `—` | — | — | HELD |
| RQ10 | malformed | body is form-encoded, not JSON | 429 | `—` | — | — | HELD |
| RQ11 | malformed | body is JSON claiming to be plain text | 429 | `—` | — | — | HELD |
| RQ12 | malformed | deeply nested JSON | 400 | `—` | — | — | HELD |
| FL01_01 | flood_same_creator | 20 rapid submissions from one creator | 200 | `c_39c01b911ae4` | unsure | 0.364 | HELD |
| FL01_02 | flood_same_creator | 20 rapid submissions from one creator | 200 | `c_7014adc4dc7d` | unsure | 0.364 | HELD |
| FL01_03 | flood_same_creator | 20 rapid submissions from one creator | 200 | `c_cd4439734629` | unsure | 0.364 | HELD |
| FL01_04 | flood_same_creator | 20 rapid submissions from one creator | 200 | `c_f9bfd0cefda7` | unsure | 0.364 | HELD |
| FL01_05 | flood_same_creator | 20 rapid submissions from one creator | 200 | `c_8bad19cc723c` | unsure | 0.364 | HELD |
| FL01_06 | flood_same_creator | 20 rapid submissions from one creator | 429 | `—` | — | — | HELD |
| FL01_07 | flood_same_creator | 20 rapid submissions from one creator | 429 | `—` | — | — | HELD |
| FL01_08 | flood_same_creator | 20 rapid submissions from one creator | 429 | `—` | — | — | HELD |
| FL01_09 | flood_same_creator | 20 rapid submissions from one creator | 429 | `—` | — | — | HELD |
| FL01_10 | flood_same_creator | 20 rapid submissions from one creator | 429 | `—` | — | — | HELD |
| FL01_11 | flood_same_creator | 20 rapid submissions from one creator | 429 | `—` | — | — | HELD |
| FL01_12 | flood_same_creator | 20 rapid submissions from one creator | 429 | `—` | — | — | HELD |

**By family**

| Family | Attacks | Auto-BROKE | HELD | BROKE |
|---|---|---|---|---|
| `evasion` | 10 | 0 | 2 | 8 |
| `false_positive` | 8 | 0 | 3 | 5 |
| `flood_same_creator` | 12 | 0 | 12 | 0 |
| `malformed` | 20 | 1 | 13 | 7 |
| `prompt_injection` | 6 | 0 | 6 | 0 |
| **Total** | **56** | **1** | **36** | **20** |

**Ten audit log entries from the run**

<!-- Choose entries that show the interesting failures, not the ten easiest. -->

```json

{"timestamp": "2026-11-18T10:38:50+00:00", "content_id": "c_d7eeaab16168", "creator_id": "attacker_FP03", "guess": "ai", "model_score": 0.8051, "style_score": 0.8718, "combined_score": 0.8385, "label": "This shows strong signs of being written by AI. If you wrote it yourself, you can appeal and a person will review it.", "status": "decided"}
{"timestamp": "2026-11-18T10:38:50+00:00", "content_id": "c_ac19450dda50", "creator_id": "attacker_FP05", "guess": "ai", "model_score": 0.8005, "style_score": 0.9231, "combined_score": 0.8618, "label": "This shows strong signs of being written by AI. If you wrote it yourself, you can appeal and a person will review it.", "status": "decided"}
{"timestamp": "2026-11-18T10:38:52+00:00", "content_id": "c_51656690bd7c", "creator_id": "attacker_MF07", "guess": "ai", "model_score": 0.9064, "style_score": 0.7, "combined_score": 0.8032, "label": "This shows strong signs of being written by AI. If you wrote it yourself, you can appeal and a person will review it.", "status": "decided"}
{"timestamp": "2026-11-18T10:38:52+00:00", "content_id": "c_9cbd58ff32fd", "creator_id": "attacker_MF06", "guess": "ai", "model_score": 0.9969, "style_score": 0.7, "combined_score": 0.8484, "label": "This shows strong signs of being written by AI. If you wrote it yourself, you can appeal and a person will review it.", "status": "decided"}
{"timestamp": "2026-11-18T10:38:52+00:00", "content_id": "c_f98dcd8f6783", "creator_id": "attacker_MF08", "guess": "ai", "model_score": 0.9876, "style_score": 0.7718, "combined_score": 0.8797, "label": "This shows strong signs of being written by AI. If you wrote it yourself, you can appeal and a person will review it.", "status": "decided"}
{"timestamp": "2026-11-18T10:38:50+00:00", "content_id": "c_db9e9e918037", "creator_id": "attacker_EV05", "guess": "human", "model_score": 0.3212, "style_score": 0.3402, "combined_score": 0.3307, "label": "This reads like it was written by a real person.", "status": "decided"}
{"timestamp": "2026-11-18T10:38:51+00:00", "content_id": "c_174446767801", "creator_id": "attacker_PI02", "guess": "unsure", "model_score": 0.2044, "style_score": 0.5, "combined_score": 0.3522, "label": "We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it.", "status": "decided"}
{"timestamp": "2026-11-18T10:38:51+00:00", "content_id": "c_889475f4850e", "creator_id": "attacker_MF03", "guess": "human", "model_score": 0.0, "style_score": 0.7, "combined_score": 0.35, "label": "This reads like it was written by a real person.", "status": "decided"}
{"timestamp": "2026-11-18T10:38:52+00:00", "content_id": "c_e63ab6e85d88", "creator_id": {"id": "x"}, "guess": "unsure", "model_score": 0.0492, "style_score": 0.7, "combined_score": 0.3746, "label": "We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it.", "status": "decided"}
{"timestamp": "2026-11-18T10:38:51+00:00", "content_id": "c_ef40e633457e", "creator_id": "attacker_PI05", "guess": "unsure", "model_score": 0.6401, "style_score": 0.3286, "combined_score": 0.4844, "label": "We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it.", "status": "decided"}
{"timestamp": "2026-11-18T10:38:52+00:00", "creator_id": "attacker_flood", "event": "rejected", "status": "rejected", "reason": "rate_limited", "limit": "5 per 1 minute"}
{"timestamp": "2026-11-18T10:38:52+00:00", "creator_id": "unknown", "event": "rejected", "status": "rejected", "reason": "rate_limited", "limit": "5 per 1 minute"}


```

**The single worst outcome** — the one I'd least want to explain to a writer
whose work got caught by it:

FP03: a non-native English speaker's careful writing was labelled "strong signs of being written by AI" at 0.84. Writing in a second language often means simple words and even sentences, and both of my signals read that as machine-like. Because both signals agreed, nothing pulled the score back to "unsure". The writer can appeal, but they've already been told their own work looks fake. 

---

## Run Log and Verdicts

<!-- Your five criteria across three trials. `run_eval.py --label before`. -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. My own writing labelled high-confidence AI | ≤ 1 of 10 | 3/10 | 3/10 | 3/10 | MISSED |
| 2. Clearly-AI avg minus clearly-human avg | ≥ 0.30 | 0.26 | 0.26 | 0.26 | MISSED |
| 3. Labels produced by at least 2 inputs | 3 of 3 | 1/3 | 1/3 | 1/3 | MISSED |
| 4. Valid appeals → under_review + in /log; fake ids → 404 | 20 of 20 | 20/20 | 20/20 | 20/20 | MET |
| 5. Missing / empty / whitespace / >50k text → 4xx with message | 4 of 4 | 3/4 | 3/4 | 3/4 | MISSED |

**How I decided:**
1. **MISSED:** 3 of my 10 pieces were labelled AI in every trial, and the target allows at most 1.
2. **MISSED:** the AI average minus the human average was 0.26 in all three trials, below the 0.30 target.
3. **MISSED:** only "unsure" came from 2 or more inputs. "ai" and "human" each came from just one, so 1 of 3 in every trial.
4. **MET:** all 10 valid appeals moved to under_review and showed up in /log, and all 10 fake ids got a 404, in all three trials (20/20 each time).
5. **MISSED:** the 60,000-character body got a 200 in every trial, so only 3 of 4 bad inputs got a 4xx.

**Real output from one trial**, pasted as text, naming the file and function
that produced it:

From `results/run_2026-11-18_0307_before.md`, Trial 1 (all three trials were identical).

*Criterion 1 — my own writing* — produced by `run_eval.py::main` → `app.py::submit` (scenario in `scenarios.py`, "my own writing"). Pieces 3, 6 and 8 came back `ai`.

```json
[
  {
    "status": 200,
    "content_id": "c_0ff129e755e4",
    "error": null,
    "message": null,
    "guess": "unsure",
    "confidence": 0.6,
    "model_score": 0.89,
    "style_score": 0.5551,
    "label": "We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it.",
    "sent": "I am so excited for Christmas! I can't wait to see my family and to dr"
  },
  {
    "status": 200,
    "content_id": "c_15a898848022",
    "error": null,
    "message": null,
    "guess": "unsure",
    "confidence": 0.5698,
    "model_score": 0.406,
    "style_score": 0.7337,
    "label": "We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it.",
    "sent": "Today I played tennis with my fromer opponent. We played for so long. "
  },
  {
    "status": 200,
    "content_id": "c_6aee9998ac9f",
    "error": null,
    "message": null,
    "guess": "ai",
    "confidence": 0.7835,
    "model_score": 0.7825,
    "style_score": 0.7846,
    "label": "This shows strong signs of being written by AI. If you wrote it yourself, you can appeal and a person will review it.",
    "sent": "I will gladly take your ice cream. Thank you for offering it to me. I "
  },
  {
    "status": 200,
    "content_id": "c_8caec29f2701",
    "error": null,
    "message": null,
    "guess": "unsure",
    "confidence": 0.4758,
    "model_score": 0.2268,
    "style_score": 0.7247,
    "label": "We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it.",
    "sent": "Ughh I wish winter break could start already! I am so excited to see m"
  },
  {
    "status": 200,
    "content_id": "c_86cf1cba3a05",
    "error": null,
    "message": null,
    "guess": "unsure",
    "confidence": 0.5999,
    "model_score": 0.7922,
    "style_score": 0.4077,
    "label": "We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it.",
    "sent": "My cat's name is Massia. She is almost 10 years old, but she still act"
  },
  {
    "status": 200,
    "content_id": "c_58beda653c5d",
    "error": null,
    "message": null,
    "guess": "ai",
    "confidence": 0.796,
    "model_score": 0.9342,
    "style_score": 0.6578,
    "label": "This shows strong signs of being written by AI. If you wrote it yourself, you can appeal and a person will review it.",
    "sent": "I am getting ready to go to bed, but I still have a lot of work to do."
  },
  {
    "status": 200,
    "content_id": "c_36cbae6d054f",
    "error": null,
    "message": null,
    "guess": "unsure",
    "confidence": 0.4759,
    "model_score": 0.4553,
    "style_score": 0.4965,
    "label": "We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it.",
    "sent": "I am an undergraduate CS student. What about you? Do you like your maj"
  },
  {
    "status": 200,
    "content_id": "c_3567a7f4632f",
    "error": null,
    "message": null,
    "guess": "ai",
    "confidence": 0.7705,
    "model_score": 0.7259,
    "style_score": 0.8151,
    "label": "This shows strong signs of being written by AI. If you wrote it yourself, you can appeal and a person will review it.",
    "sent": "In general, I think that a lot of students in this generation feel a l"
  },
  {
    "status": 200,
    "content_id": "c_5163adac56ad",
    "error": null,
    "message": null,
    "guess": "unsure",
    "confidence": 0.4868,
    "model_score": 0.771,
    "style_score": 0.2026,
    "label": "We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it.",
    "sent": "Can you take me to Disneyland? I really want to ride the rollercoaster"
  },
  {
    "status": 200,
    "content_id": "c_833a44e8d1f9",
    "error": null,
    "message": null,
    "guess": "unsure",
    "confidence": 0.6,
    "model_score": 0.8067,
    "style_score": 0.433,
    "label": "We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it.",
    "sent": "I really like learning about Artificial Intelligence. It is kind of ha"
  }
]
```

*Criterion 2 — score spread* — produced by `run_eval.py::main` → `app.py::submit`. First five are AI (avg confidence 0.790), last three human (avg 0.526); gap 0.264.

```json
[
  {
    "status": 200,
    "content_id": "c_6c66db86c3cd",
    "error": null,
    "message": null,
    "guess": "ai",
    "confidence": 0.7613,
    "model_score": 0.8217,
    "style_score": 0.701,
    "label": "This shows strong signs of being written by AI. If you wrote it yourself, you can appeal and a person will review it.",
    "sent": "Artificial intelligence has emerged as a transformative force across n"
  },
  {
    "status": 200,
    "content_id": "c_8b9798da0ae1",
    "error": null,
    "message": null,
    "guess": "unsure",
    "confidence": 0.6835,
    "model_score": 0.6395,
    "style_score": 0.7276,
    "label": "We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it.",
    "sent": "Remote work has fundamentally reshaped the modern workplace. By elimin"
  },
  {
    "status": 200,
    "content_id": "c_1a8b867ab6dc",
    "error": null,
    "message": null,
    "guess": "ai",
    "confidence": 0.8579,
    "model_score": 0.9312,
    "style_score": 0.7847,
    "label": "This shows strong signs of being written by AI. If you wrote it yourself, you can appeal and a person will review it.",
    "sent": "Climate change represents one of the most pressing challenges of our t"
  },
  {
    "status": 200,
    "content_id": "c_d1ee8e9881db",
    "error": null,
    "message": null,
    "guess": "ai",
    "confidence": 0.8242,
    "model_score": 0.9338,
    "style_score": 0.7147,
    "label": "This shows strong signs of being written by AI. If you wrote it yourself, you can appeal and a person will review it.",
    "sent": "Regular physical activity offers a wide range of benefits for both phy"
  },
  {
    "status": 200,
    "content_id": "c_c6ed0e253c9d",
    "error": null,
    "message": null,
    "guess": "ai",
    "confidence": 0.8245,
    "model_score": 0.7804,
    "style_score": 0.8685,
    "label": "This shows strong signs of being written by AI. If you wrote it yourself, you can appeal and a person will review it.",
    "sent": "Effective time management is an essential skill in both professional a"
  },
  {
    "status": 200,
    "content_id": "c_38c114a810a2",
    "error": null,
    "message": null,
    "guess": "human",
    "confidence": 0.2984,
    "model_score": 0.4725,
    "style_score": 0.1244,
    "label": "This reads like it was written by a real person.",
    "sent": "I didn't sleep. Not properly, anyway - I lay there listening to the bo"
  },
  {
    "status": 200,
    "content_id": "c_b002c0d8111a",
    "error": null,
    "message": null,
    "guess": "unsure",
    "confidence": 0.68,
    "model_score": 0.6175,
    "style_score": 0.7425,
    "label": "We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it.",
    "sent": "We got to the campsite about four. The pitch was fine, a bit sloped, b"
  },
  {
    "status": 200,
    "content_id": "c_2948f7ff22f0",
    "error": null,
    "message": null,
    "guess": "unsure",
    "confidence": 0.6,
    "model_score": 0.89,
    "style_score": 0.5551,
    "label": "We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it.",
    "sent": "I am so excited for Christmas! I can't wait to see my family and to dr"
  }
]
```

*Criterion 3 — label coverage* — produced by `run_eval.py::main` → `app.py::submit`. `ai` ×1, `unsure` ×4, `human` ×1.

```json
[
  {
    "status": 200,
    "content_id": "c_77547969182c",
    "error": null,
    "message": null,
    "guess": "ai",
    "confidence": 0.7613,
    "model_score": 0.8217,
    "style_score": 0.701,
    "label": "This shows strong signs of being written by AI. If you wrote it yourself, you can appeal and a person will review it.",
    "sent": "Artificial intelligence has emerged as a transformative force across n"
  },
  {
    "status": 200,
    "content_id": "c_ccc81299f25d",
    "error": null,
    "message": null,
    "guess": "unsure",
    "confidence": 0.6835,
    "model_score": 0.6395,
    "style_score": 0.7276,
    "label": "We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it.",
    "sent": "Remote work has fundamentally reshaped the modern workplace. By elimin"
  },
  {
    "status": 200,
    "content_id": "c_ab5c07d60018",
    "error": null,
    "message": null,
    "guess": "unsure",
    "confidence": 0.4275,
    "model_score": 0.3109,
    "style_score": 0.5442,
    "label": "We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it.",
    "sent": "Time management matters more than people admit, especially now that wo"
  },
  {
    "status": 200,
    "content_id": "c_f6da2b4b95e7",
    "error": null,
    "message": null,
    "guess": "unsure",
    "confidence": 0.6,
    "model_score": 0.89,
    "style_score": 0.5551,
    "label": "We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it.",
    "sent": "I am so excited for Christmas! I can't wait to see my family and to dr"
  },
  {
    "status": 200,
    "content_id": "c_f36bbb018e60",
    "error": null,
    "message": null,
    "guess": "human",
    "confidence": 0.2984,
    "model_score": 0.4725,
    "style_score": 0.1244,
    "label": "This reads like it was written by a real person.",
    "sent": "I didn't sleep. Not properly, anyway - I lay there listening to the bo"
  },
  {
    "status": 200,
    "content_id": "c_dd228542a36b",
    "error": null,
    "message": null,
    "guess": "unsure",
    "confidence": 0.68,
    "model_score": 0.6175,
    "style_score": 0.7425,
    "label": "We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it.",
    "sent": "We got to the campsite about four. The pitch was fine, a bit sloped, b"
  }
]
```

*Criterion 4 — appeal path* — produced by `run_eval.py::run_appeals` → `app.py::submit`, `app.py::appeal`, then `GET /log`. Excerpt: 3 of the 10 valid appeals and 3 of the 10 fake ids; all 20 passed, full output in `results/run_2026-11-18_0307_before.md`.

```json
[
  {
    "kind": "valid",
    "sent": "Artificial intelligence has emerged as a transformative force across n",
    "submit_status": 200,
    "content_id": "c_a4fa91775dac",
    "appeal_status": 200,
    "status": "under_review",
    "in_log": true,
    "passed": true
  },
  {
    "kind": "valid",
    "sent": "Remote work has fundamentally reshaped the modern workplace. By elimin",
    "submit_status": 200,
    "content_id": "c_8e8c324b89d7",
    "appeal_status": 200,
    "status": "under_review",
    "in_log": true,
    "passed": true
  },
  {
    "kind": "valid",
    "sent": "Climate change represents one of the most pressing challenges of our t",
    "submit_status": 200,
    "content_id": "c_73230e671cca",
    "appeal_status": 200,
    "status": "under_review",
    "in_log": true,
    "passed": true
  },
  {
    "kind": "fake",
    "content_id": "c_fake101notreal",
    "appeal_status": 404,
    "error": "not_found",
    "passed": true
  },
  {
    "kind": "fake",
    "content_id": "c_fake102notreal",
    "appeal_status": 404,
    "error": "not_found",
    "passed": true
  },
  {
    "kind": "fake",
    "content_id": "c_fake103notreal",
    "appeal_status": 404,
    "error": "not_found",
    "passed": true
  }
]
```

*Criterion 5 — bad input* — produced by `run_eval.py::main` → `app.py::submit`. The 60,000-character body came back 200 with a decision instead of a 4xx.

```json
[
  {
    "status": 400,
    "content_id": null,
    "error": "bad_request",
    "message": "Send JSON with a non-empty \"text\".",
    "guess": null,
    "confidence": null,
    "model_score": null,
    "style_score": null,
    "label": null,
    "sent": "{\"creator_id\": \"eval_bad_missing\"}"
  },
  {
    "status": 400,
    "content_id": null,
    "error": "bad_request",
    "message": "Send JSON with a non-empty \"text\".",
    "guess": null,
    "confidence": null,
    "model_score": null,
    "style_score": null,
    "label": null,
    "sent": "(empty text)"
  },
  {
    "status": 400,
    "content_id": null,
    "error": "bad_request",
    "message": "Send JSON with a non-empty \"text\".",
    "guess": null,
    "confidence": null,
    "model_score": null,
    "style_score": null,
    "label": null,
    "sent": "   \n\t  "
  },
  {
    "status": 200,
    "content_id": "c_79ca7d56b5ed",
    "error": null,
    "message": null,
    "guess": "unsure",
    "confidence": 0.6,
    "model_score": 0.985,
    "style_score": 0.4,
    "label": "We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it.",
    "sent": "We got to the campsite about four. The pitch was fine, a bit sloped, b"
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

All three trials were identical, so every miss is consistent. The 4 missed criteria and 20 broken attacks come from four problems.

**1. Both signals reward plain writing (signal one + signal two).** Explains criterion 1, criterion 2, the human side of criterion 3, and FP02–FP06.
Signal one (gpt2 perplexity) scores common, predictable words as AI. Signal two scores even sentence lengths and few commas or dashes as AI. My punctuation measure ignores full stops, which is most of what plain writers use. So in plain human writing the two signals agree, and the combining rule has nothing to soften. Three of my own pieces were labelled AI: ice cream (0.78 / 0.78), all-nighter (0.93 / 0.66) and job market (0.73 / 0.82). FP02–FP05 scored 0.74–0.83 on signal one and 0.83–0.96 on signal two, and FP06 followed the same pattern (0.88 / 0.68). The plain campsite passage (0.68) also raised the human average, so the gap between AI and human was 0.26 instead of 0.30. Only text with very uneven sentences and lots of punctuation could reach "human".

**2. The AI label needs both signals high, so fooling one is enough (combining rule).** Explains 8 evasions and the AI side of criterion 3.
I average the two signals and cap the score at 0.60 when they're more than 0.30 apart, so reaching "ai" (≥ 0.70) needs both signals high. Breaking up sentences or adding punctuation (EV02, EV06, EV07) dropped signal two to 0.30–0.39. Typos and very short text (EV01, EV08) dropped signal one to 0.16–0.25. A pasted quote or a human opening (EV03, EV09) lowered both a little, and a heavy human edit (EV05) lowered both enough to reach "human". My second AI paragraph averaged 0.68, just under 0.70. I chose this rule so real writers aren't accused, and the cost is that AI text is easy to slip through.

**3. Signal two has no "can't measure" answer (signal two).** Explains MF03, MF05, MF06 and MF07.
My word regex only matches `a-z0-9'`, so emoji and Arabic count as zero words, and signal two then defaults to 0.70, which leans AI. Signal one also scores those inputs as very predictable (0.91–0.99), so emoji and Arabic text were labelled AI. With ANSI control codes it went the other way: signal one gave 0.0, the two signals averaged exactly 0.35, and `<=` made it "human".

**4. The route doesn't check the request (route).** Explains criterion 5, MF08, RQ06 and RQ07.
There's no length limit, so 60,000 characters came back 200. The detector quietly truncates to 400 tokens, and repeated human text scored 0.99 (MF08). A JSON array body crashed `payload.get` with a 500 and left no audit log entry (RQ07). A `creator_id` that is an object was logged as an object (RQ06).

**Biggest problem:** #1, because it accuses real writers. It caused my worst outcome (FP03) and 3 of my own 10 pieces.

**Revisions:** none. All five criteria were measurable, and I kept the targets I missed.



---

## The Improvement

**What I changed:** one line in `config.py`: `AI_THRESHOLD` went from **0.70 to 0.80**. Nothing else changed: the signals, weights, cap and route are the same. This widens the unsure band from 0.35–0.70 to 0.35–0.80.

**Which diagnosis pointed at it:** diagnosis #1, plain human writing scored as AI. My three pieces labelled AI scored 0.77–0.80, just over the old line, and FP06 scored 0.78. I tested other thresholds on the "before" scores first. 0.75 fixed nothing. 0.85 fixed 4 of 5 false positives but left almost no AI text labelled AI. 0.80 was the middle.

**Criteria, before and after** (three trials each, all identical; `run_eval.py --label after`, full output in `results/run_2026-11-18_0318_after.md`):

| Criterion | Target | Before (Run 1 / 2 / 3) | After (Run 1 / 2 / 3) | Verdict before → after |
|---|---|---|---|---|
| 1. My own writing labelled high-confidence AI | ≤ 1 of 10 | 3/10 · 3/10 · 3/10 | 0/10 · 0/10 · 0/10 | MISSED → **MET** |
| 2. Clearly-AI avg minus clearly-human avg | ≥ 0.30 | 0.26 · 0.26 · 0.26 | 0.26 · 0.26 · 0.26 | MISSED → MISSED |
| 3. Labels produced by at least 2 inputs | 3 of 3 | 1/3 · 1/3 · 1/3 | 1/3 · 1/3 · 1/3 | MISSED → MISSED |
| 4. Valid appeals → under_review + in /log; fake ids → 404 | 20 of 20 | 20/20 · 20/20 · 20/20 | 20/20 · 20/20 · 20/20 | MET → MET |
| 5. Missing / empty / whitespace / >50k text → 4xx with message | 4 of 4 | 3/4 · 3/4 · 3/4 | 3/4 · 3/4 · 3/4 | MISSED → MISSED |

### Attack Run — After

<!-- Same format. `run_attacks.py --label after` -->

**By family, before and after (HELD / BROKE):**

| Family | Before | After |
|---|---|---|
| `evasion` | 2 / 8 | 1 / 9 |
| `false_positive` | 3 / 5 | 4 / 4 |
| `prompt_injection` | 6 / 0 | 6 / 0 |
| `malformed` | 13 / 7 | 13 / 7 |
| `flood_same_creator` | 12 / 0 | 12 / 0 |
| **Total** | **36 / 20** | **36 / 20** |

Only two attacks changed: **FP06** went from BROKE to HELD (0.781, ai → unsure), and **EV04** went from HELD to BROKE (0.759, ai → unsure). Every status code was the same.

56 attacks · 2026-11-18 · HELD **36**, BROKE **20** (`results/attack_run_2026-11-18_0318_after.md`)

| ID | Family | Targeting | Status | Content ID | Guess | Score | Verdict |
|---|---|---|---|---|---|---|---|
| EV01 | evasion | signal two: typos raise word variety | 200 | `c_57ca0c8e4292` | unsure | 0.553 | BROKE |
| EV02 | evasion | signal two: broken-up sentences raise length spread | 200 | `c_e721dc2362d3` | unsure | 0.474 | BROKE |
| EV03 | evasion | signal two: a pasted quotation changes the shape | 200 | `c_95585ea55802` | unsure | 0.636 | BROKE |
| EV04 | evasion | signal one: unusual words raise perplexity | 200 | `c_5b924586432a` | unsure | 0.759 | BROKE |
| EV05 | evasion | both signals: heavy human editing pass | 200 | `c_d6a552179cc2` | human | 0.331 | BROKE |
| EV06 | evasion | signal two: one very long sentence and one very short | 200 | `c_359f80a64392` | unsure | 0.482 | BROKE |
| EV07 | evasion | signal two: padding with punctuation | 200 | `c_919720f7075a` | unsure | 0.443 | BROKE |
| EV08 | evasion | signal one: text below the token window | 200 | `c_ebba9ce7d11a` | unsure | 0.581 | BROKE |
| EV09 | evasion | both: AI text with a human opening paragraph bolted on | 200 | `c_eab9f9619c32` | unsure | 0.678 | BROKE |
| EV10 | evasion | signal two: repeated stanza structure | 200 | `c_c4b8e725cde4` | ai | 0.893 | HELD |
| FP01 | false_positive | plain, clear human writing with no flourishes | 200 | `c_c55178ff1727` | unsure | 0.680 | HELD |
| FP02 | false_positive | human writing in a formal register | 200 | `c_f331bcd21f30` | ai | 0.849 | BROKE |
| FP03 | false_positive | a non-native English speaker writing carefully | 200 | `c_0f367fc985a9` | ai | 0.839 | BROKE |
| FP04 | false_positive | a student essay written to a rubric | 200 | `c_b00ba11126a4` | ai | 0.833 | BROKE |
| FP05 | false_positive | technical documentation written by a person | 200 | `c_5419a7667753` | ai | 0.862 | BROKE |
| FP06 | false_positive | a person imitating a formal template | 200 | `c_97253baf98e1` | unsure | 0.781 | HELD |
| FP07 | false_positive | human writing that repeats a phrase deliberately | 200 | `c_0a83dcc2edf0` | unsure | 0.600 | HELD |
| FP08 | false_positive | very short genuine human submission | 200 | `c_d4424105cb3f` | unsure | 0.545 | HELD |
| PI01 | prompt_injection | instruction to ignore the check | 200 | `c_5332e72663c3` | unsure | 0.500 | HELD |
| PI02 | prompt_injection | instruction disguised as a system message | 200 | `c_cdae40032243` | unsure | 0.352 | HELD |
| PI03 | prompt_injection | instruction wrapped in the writer's voice | 200 | `c_63ec18cfc867` | unsure | 0.685 | HELD |
| PI04 | prompt_injection | attempt to make the label text hostile | 200 | `c_67c1890ea690` | unsure | 0.571 | HELD |
| PI05 | prompt_injection | attempt to inject into the audit log | 200 | `c_b39e0324ce29` | unsure | 0.484 | HELD |
| PI06 | prompt_injection | instruction plus real human text | 200 | `c_ca6f3346e5be` | unsure | 0.656 | HELD |
| MF01 | malformed | text is only whitespace | 400 | `—` | — | — | HELD |
| MF02 | malformed | text is a single character | 400 | `—` | — | — | HELD |
| MF03 | malformed | text is control characters | 200 | `c_becfc2bd0d05` | human | 0.350 | BROKE |
| MF04 | malformed | text is only punctuation | 200 | `c_381025c66c45` | unsure | 0.356 | HELD |
| MF05 | malformed | text is a very long single word | 200 | `c_2d049b1ba74a` | ai | 0.849 | BROKE |
| MF06 | malformed | text is emoji only | 200 | `c_67034a38cb21` | ai | 0.848 | BROKE |
| MF07 | malformed | text is right-to-left script | 200 | `c_213c5ff7edbe` | ai | 0.803 | BROKE |
| MF08 | malformed | text is 60,000 characters | 200 | `c_d34b4a134cd8` | ai | 0.880 | BROKE |
| RQ01 | malformed | no text field at all | 400 | `—` | — | — | HELD |
| RQ02 | malformed | no creator_id | 200 | `c_8451478270c0` | unsure | 0.470 | HELD |
| RQ03 | malformed | text is a number | 400 | `—` | — | — | HELD |
| RQ04 | malformed | text is a list | 400 | `—` | — | — | HELD |
| RQ05 | malformed | text is null | 400 | `—` | — | — | HELD |
| RQ06 | malformed | creator_id is an object | 200 | `c_e00c2c615c17` | unsure | 0.375 | BROKE |
| RQ07 | malformed | body is a JSON array, not an object | 500 | `—` | — | — | BROKE |
| RQ08 | malformed | body is unparseable JSON | 400 | `—` | — | — | HELD |
| RQ09 | malformed | body is empty | 400 | `—` | — | — | HELD |
| RQ10 | malformed | body is form-encoded, not JSON | 429 | `—` | — | — | HELD |
| RQ11 | malformed | body is JSON claiming to be plain text | 429 | `—` | — | — | HELD |
| RQ12 | malformed | deeply nested JSON | 400 | `—` | — | — | HELD |
| FL01_01 | flood_same_creator | 20 rapid submissions from one creator | 200 | `c_97011883a949` | unsure | 0.364 | HELD |
| FL01_02 | flood_same_creator | 20 rapid submissions from one creator | 200 | `c_bb6f57008d27` | unsure | 0.364 | HELD |
| FL01_03 | flood_same_creator | 20 rapid submissions from one creator | 200 | `c_11359fd49cc7` | unsure | 0.364 | HELD |
| FL01_04 | flood_same_creator | 20 rapid submissions from one creator | 200 | `c_5c80533f9acf` | unsure | 0.364 | HELD |
| FL01_05 | flood_same_creator | 20 rapid submissions from one creator | 200 | `c_0c5e235d7adb` | unsure | 0.364 | HELD |
| FL01_06 | flood_same_creator | 20 rapid submissions from one creator | 429 | `—` | — | — | HELD |
| FL01_07 | flood_same_creator | 20 rapid submissions from one creator | 429 | `—` | — | — | HELD |
| FL01_08 | flood_same_creator | 20 rapid submissions from one creator | 429 | `—` | — | — | HELD |
| FL01_09 | flood_same_creator | 20 rapid submissions from one creator | 429 | `—` | — | — | HELD |
| FL01_10 | flood_same_creator | 20 rapid submissions from one creator | 429 | `—` | — | — | HELD |
| FL01_11 | flood_same_creator | 20 rapid submissions from one creator | 429 | `—` | — | — | HELD |
| FL01_12 | flood_same_creator | 20 rapid submissions from one creator | 429 | `—` | — | — | HELD |

**Did it help, and how do I know:**

<!-- Widening the unsure band usually fixes one problem and creates another.
     Reporting that trade honestly is worth full credit. -->

Partly. It fixed one problem and created another, and the attack totals didn't move (36 / 20 both times).

- **Better:** criterion 1 went from MISSED to MET (3/10 → 0/10 in all three trials). FP06 is no longer an accusation.
- **Worse:** EV04 (0.76) and the repo's clearly-AI sample (0.76) are now "unsure" instead of "ai". In my label-coverage test, no input reaches "ai" anymore (down from 1). One false accusation was traded for one missed AI text.
- **Not fixed:** FP02–FP05, including my worst outcome FP03, still score 0.83–0.86 and are still labelled AI. A threshold only moves the line. Diagnosis #1 says both signals score plain writing high, so the real fix is in signal two (for example, counting full stops in the punctuation measure).
- **Couldn't change:** criterion 2 measures the scores, and a threshold only changes labels, so the gap stayed 0.26. Criterion 5 is a route problem.

I know because the before and after runs used the same inputs. All three trials matched each time, so every difference above comes from the one-line change.

---

## What's Still Broken

<!-- For each attack still getting through and each criterion still missed:
     what you'd do, and why you stopped. -->

After the change, 20 attacks still break and criteria 2, 3 and 5 are still missed. They fall into the same four groups as my diagnoses. I stopped after one change because the milestone asks for one measured change, and I picked the one that was hurting real writers.

**1. Plain human writing still scored as AI: FP02–FP05, criterion 2, and the "human" side of criterion 3.**
What I'd do: fix signal two. My punctuation measure ignores full stops, so plain writing gets the maximum AI score on that part. I'd count full stops, or replace punctuation with a measure that doesn't punish simple writing. Then I'd re-check every threshold, because changing a signal moves every score.
Why I stopped: it's a bigger change than a threshold, and it would need its own before/after run to tell its effect apart from the threshold change.

**2. Evasions still get through: EV01–EV09 and the "ai" side of criterion 3.**
What I'd do: probably nothing quickly. Labelling more of them AI means lowering the threshold or trusting one signal alone, and both bring back the false accusations I just fixed. A real fix needs a third signal that the evasions don't beat.
Why I stopped: this is the trade I chose on purpose (see below), not something I ran out of time for.

**3. Non-prose input gets a confident label: MF03, MF05, MF06 and MF07.**
What I'd do: in the route, reject text with fewer than a few scorable words (400, "not enough text to check"). I'd also make signal two return a neutral 0.5 instead of 0.70 when it finds no words.
Why I stopped: one change per milestone. This is the next one I'd make.

**4. The route doesn't check the request: criterion 5, MF08, RQ06 and RQ07.**
What I'd do: in `app.py::submit`, return 400 if the body isn't a JSON object (stops the RQ07 crash), 400 if `creator_id` isn't a string (RQ06), and 413 if `text` is over 50,000 characters (criterion 5 and MF08). These are a few lines each.
Why I stopped: same reason, one change. These are the easiest fixes left, and the only ones with no trade-off.

**The trade I'd make if this ran for real:**

<!-- Two sentences. Fewer wrong accusations means more AI text getting through.
     Say which way you'd go and WHO PAYS FOR IT. -->

I'd keep choosing fewer wrong accusations, because a false "AI" label lands on one real person who has to prove they wrote their own work, while a missed AI text costs the site a little. The people who pay are the honest writers who compete with AI text that slips through, and readers who can trust an "unsure" label less.



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
