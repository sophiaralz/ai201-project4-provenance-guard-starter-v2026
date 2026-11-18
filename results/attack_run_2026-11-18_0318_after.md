# Attack run — after

- Produced by: `run_attacks.py::main`
- Service: `app.py::submit`
- 56 attacks · 2026-11-18 03:18
- Auto-marked BROKE (5xx, crash, no response): **1**
- Judged by hand: **55** — HELD **36**, BROKE **20** (incl. the auto-BROKE)

The Verdict column is blank where a script can't honestly decide. Whether
an evasion worked depends on where you put your thresholds — read the
`targets` column, look at what came back, and write HELD or BROKE yourself.

Each attack submits as creator_id `attacker_<id>`, and the content_id it
came back with is in the table — either one will match this run against your
audit log when you go looking for the ten entries to paste.

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

---

## By family

| Family | Attacks | Auto-BROKE | HELD | BROKE |
|---|---|---|---|---|
| `evasion` | 10 | 0 | 1 | 9 |
| `false_positive` | 8 | 0 | 4 | 4 |
| `flood_same_creator` | 12 | 0 | 12 | 0 |
| `malformed` | 20 | 1 | 13 | 7 |
| `prompt_injection` | 6 | 0 | 6 | 0 |
| **Total** | **56** | **1** | **36** | **20** |

Judged with the same rules as the before run. Changed from before: FP06 BROKE → HELD, EV04 HELD → BROKE.

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
  "content_id": "c_57ca0c8e4292",
  "creator_id": "attacker_EV01",
  "guess": "unsure",
  "confidence": 0.5533,
  "model_score": 0.1635,
  "style_score": 0.9431,
  "label": "We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it.",
  "error": null,
  "verdict": "BROKE"
}
```

### EV02 — `evasion`

*Targeting:* signal two: broken-up sentences raise length spread

```json
{
  "id": "EV02",
  "status": 200,
  "content_id": "c_e721dc2362d3",
  "creator_id": "attacker_EV02",
  "guess": "unsure",
  "confidence": 0.4739,
  "model_score": 0.5943,
  "style_score": 0.3535,
  "label": "We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it.",
  "error": null,
  "verdict": "BROKE"
}
```

### EV03 — `evasion`

*Targeting:* signal two: a pasted quotation changes the shape

```json
{
  "id": "EV03",
  "status": 200,
  "content_id": "c_95585ea55802",
  "creator_id": "attacker_EV03",
  "guess": "unsure",
  "confidence": 0.6357,
  "model_score": 0.6662,
  "style_score": 0.6051,
  "label": "We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it.",
  "error": null,
  "verdict": "BROKE"
}
```

### EV04 — `evasion`

*Targeting:* signal one: unusual words raise perplexity

```json
{
  "id": "EV04",
  "status": 200,
  "content_id": "c_5b924586432a",
  "creator_id": "attacker_EV04",
  "guess": "unsure",
  "confidence": 0.7588,
  "model_score": 0.6114,
  "style_score": 0.9062,
  "label": "We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it.",
  "error": null,
  "verdict": "BROKE"
}
```

### EV05 — `evasion`

*Targeting:* both signals: heavy human editing pass

```json
{
  "id": "EV05",
  "status": 200,
  "content_id": "c_d6a552179cc2",
  "creator_id": "attacker_EV05",
  "guess": "human",
  "confidence": 0.3307,
  "model_score": 0.3212,
  "style_score": 0.3402,
  "label": "This reads like it was written by a real person.",
  "error": null,
  "verdict": "BROKE"
}
```

### EV06 — `evasion`

*Targeting:* signal two: one very long sentence and one very short

```json
{
  "id": "EV06",
  "status": 200,
  "content_id": "c_359f80a64392",
  "creator_id": "attacker_EV06",
  "guess": "unsure",
  "confidence": 0.482,
  "model_score": 0.5727,
  "style_score": 0.3913,
  "label": "We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it.",
  "error": null,
  "verdict": "BROKE"
}
```

### EV07 — `evasion`

*Targeting:* signal two: padding with punctuation

```json
{
  "id": "EV07",
  "status": 200,
  "content_id": "c_919720f7075a",
  "creator_id": "attacker_EV07",
  "guess": "unsure",
  "confidence": 0.4431,
  "model_score": 0.5862,
  "style_score": 0.3,
  "label": "We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it.",
  "error": null,
  "verdict": "BROKE"
}
```

### EV08 — `evasion`

*Targeting:* signal one: text below the token window

```json
{
  "id": "EV08",
  "status": 200,
  "content_id": "c_ebba9ce7d11a",
  "creator_id": "attacker_EV08",
  "guess": "unsure",
  "confidence": 0.5812,
  "model_score": 0.2452,
  "style_score": 0.9172,
  "label": "We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it.",
  "error": null,
  "verdict": "BROKE"
}
```

### EV09 — `evasion`

*Targeting:* both: AI text with a human opening paragraph bolted on

```json
{
  "id": "EV09",
  "status": 200,
  "content_id": "c_eab9f9619c32",
  "creator_id": "attacker_EV09",
  "guess": "unsure",
  "confidence": 0.6783,
  "model_score": 0.7042,
  "style_score": 0.6523,
  "label": "We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it.",
  "error": null,
  "verdict": "BROKE"
}
```

### EV10 — `evasion`

*Targeting:* signal two: repeated stanza structure

```json
{
  "id": "EV10",
  "status": 200,
  "content_id": "c_c4b8e725cde4",
  "creator_id": "attacker_EV10",
  "guess": "ai",
  "confidence": 0.8926,
  "model_score": 0.9792,
  "style_score": 0.8061,
  "label": "This shows strong signs of being written by AI. If you wrote it yourself, you can appeal and a person will review it.",
  "error": null,
  "verdict": "HELD"
}
```

### FP01 — `false_positive`

*Targeting:* plain, clear human writing with no flourishes

```json
{
  "id": "FP01",
  "status": 200,
  "content_id": "c_c55178ff1727",
  "creator_id": "attacker_FP01",
  "guess": "unsure",
  "confidence": 0.68,
  "model_score": 0.6175,
  "style_score": 0.7425,
  "label": "We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it.",
  "error": null,
  "verdict": "HELD"
}
```

### FP02 — `false_positive`

*Targeting:* human writing in a formal register

```json
{
  "id": "FP02",
  "status": 200,
  "content_id": "c_f331bcd21f30",
  "creator_id": "attacker_FP02",
  "guess": "ai",
  "confidence": 0.8492,
  "model_score": 0.7387,
  "style_score": 0.9597,
  "label": "This shows strong signs of being written by AI. If you wrote it yourself, you can appeal and a person will review it.",
  "error": null,
  "verdict": "BROKE"
}
```

### FP03 — `false_positive`

*Targeting:* a non-native English speaker writing carefully

```json
{
  "id": "FP03",
  "status": 200,
  "content_id": "c_0f367fc985a9",
  "creator_id": "attacker_FP03",
  "guess": "ai",
  "confidence": 0.8385,
  "model_score": 0.8051,
  "style_score": 0.8718,
  "label": "This shows strong signs of being written by AI. If you wrote it yourself, you can appeal and a person will review it.",
  "error": null,
  "verdict": "BROKE"
}
```

### FP04 — `false_positive`

*Targeting:* a student essay written to a rubric

```json
{
  "id": "FP04",
  "status": 200,
  "content_id": "c_b00ba11126a4",
  "creator_id": "attacker_FP04",
  "guess": "ai",
  "confidence": 0.8331,
  "model_score": 0.8329,
  "style_score": 0.8333,
  "label": "This shows strong signs of being written by AI. If you wrote it yourself, you can appeal and a person will review it.",
  "error": null,
  "verdict": "BROKE"
}
```

### FP05 — `false_positive`

*Targeting:* technical documentation written by a person

```json
{
  "id": "FP05",
  "status": 200,
  "content_id": "c_5419a7667753",
  "creator_id": "attacker_FP05",
  "guess": "ai",
  "confidence": 0.8618,
  "model_score": 0.8005,
  "style_score": 0.9231,
  "label": "This shows strong signs of being written by AI. If you wrote it yourself, you can appeal and a person will review it.",
  "error": null,
  "verdict": "BROKE"
}
```

### FP06 — `false_positive`

*Targeting:* a person imitating a formal template

```json
{
  "id": "FP06",
  "status": 200,
  "content_id": "c_97253baf98e1",
  "creator_id": "attacker_FP06",
  "guess": "unsure",
  "confidence": 0.7811,
  "model_score": 0.8814,
  "style_score": 0.6808,
  "label": "We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it.",
  "error": null,
  "verdict": "HELD"
}
```

### FP07 — `false_positive`

*Targeting:* human writing that repeats a phrase deliberately

```json
{
  "id": "FP07",
  "status": 200,
  "content_id": "c_0a83dcc2edf0",
  "creator_id": "attacker_FP07",
  "guess": "unsure",
  "confidence": 0.6,
  "model_score": 0.8816,
  "style_score": 0.4561,
  "label": "We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it.",
  "error": null,
  "verdict": "HELD"
}
```

### FP08 — `false_positive`

*Targeting:* very short genuine human submission

```json
{
  "id": "FP08",
  "status": 200,
  "content_id": "c_d4424105cb3f",
  "creator_id": "attacker_FP08",
  "guess": "unsure",
  "confidence": 0.5446,
  "model_score": 0.3223,
  "style_score": 0.767,
  "label": "We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it.",
  "error": null,
  "verdict": "HELD"
}
```

### PI01 — `prompt_injection`

*Targeting:* instruction to ignore the check

```json
{
  "id": "PI01",
  "status": 200,
  "content_id": "c_5332e72663c3",
  "creator_id": "attacker_PI01",
  "guess": "unsure",
  "confidence": 0.4999,
  "model_score": 0.4625,
  "style_score": 0.5373,
  "label": "We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it.",
  "error": null,
  "verdict": "HELD"
}
```

### PI02 — `prompt_injection`

*Targeting:* instruction disguised as a system message

```json
{
  "id": "PI02",
  "status": 200,
  "content_id": "c_cdae40032243",
  "creator_id": "attacker_PI02",
  "guess": "unsure",
  "confidence": 0.3522,
  "model_score": 0.2044,
  "style_score": 0.5,
  "label": "We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it.",
  "error": null,
  "verdict": "HELD"
}
```

### PI03 — `prompt_injection`

*Targeting:* instruction wrapped in the writer's voice

```json
{
  "id": "PI03",
  "status": 200,
  "content_id": "c_63ec18cfc867",
  "creator_id": "attacker_PI03",
  "guess": "unsure",
  "confidence": 0.6847,
  "model_score": 0.5943,
  "style_score": 0.7751,
  "label": "We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it.",
  "error": null,
  "verdict": "HELD"
}
```

### PI04 — `prompt_injection`

*Targeting:* attempt to make the label text hostile

```json
{
  "id": "PI04",
  "status": 200,
  "content_id": "c_67c1890ea690",
  "creator_id": "attacker_PI04",
  "guess": "unsure",
  "confidence": 0.5709,
  "model_score": 0.4998,
  "style_score": 0.6419,
  "label": "We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it.",
  "error": null,
  "verdict": "HELD"
}
```

### PI05 — `prompt_injection`

*Targeting:* attempt to inject into the audit log

```json
{
  "id": "PI05",
  "status": 200,
  "content_id": "c_b39e0324ce29",
  "creator_id": "attacker_PI05",
  "guess": "unsure",
  "confidence": 0.4844,
  "model_score": 0.6401,
  "style_score": 0.3286,
  "label": "We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it.",
  "error": null,
  "verdict": "HELD"
}
```

### PI06 — `prompt_injection`

*Targeting:* instruction plus real human text

```json
{
  "id": "PI06",
  "status": 200,
  "content_id": "c_ca6f3346e5be",
  "creator_id": "attacker_PI06",
  "guess": "unsure",
  "confidence": 0.6564,
  "model_score": 0.5583,
  "style_score": 0.7546,
  "label": "We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it.",
  "error": null,
  "verdict": "HELD"
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
  "verdict": "HELD"
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
  "verdict": "HELD"
}
```

### MF03 — `malformed`

*Targeting:* text is control characters

```json
{
  "id": "MF03",
  "status": 200,
  "content_id": "c_becfc2bd0d05",
  "creator_id": "attacker_MF03",
  "guess": "human",
  "confidence": 0.35,
  "model_score": 0.0,
  "style_score": 0.7,
  "label": "This reads like it was written by a real person.",
  "error": null,
  "verdict": "BROKE"
}
```

### MF04 — `malformed`

*Targeting:* text is only punctuation

```json
{
  "id": "MF04",
  "status": 200,
  "content_id": "c_381025c66c45",
  "creator_id": "attacker_MF04",
  "guess": "unsure",
  "confidence": 0.3556,
  "model_score": 0.0112,
  "style_score": 0.7,
  "label": "We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it.",
  "error": null,
  "verdict": "HELD"
}
```

### MF05 — `malformed`

*Targeting:* text is a very long single word

```json
{
  "id": "MF05",
  "status": 200,
  "content_id": "c_2d049b1ba74a",
  "creator_id": "attacker_MF05",
  "guess": "ai",
  "confidence": 0.8488,
  "model_score": 0.9976,
  "style_score": 0.7,
  "label": "This shows strong signs of being written by AI. If you wrote it yourself, you can appeal and a person will review it.",
  "error": null,
  "verdict": "BROKE"
}
```

### MF06 — `malformed`

*Targeting:* text is emoji only

```json
{
  "id": "MF06",
  "status": 200,
  "content_id": "c_67034a38cb21",
  "creator_id": "attacker_MF06",
  "guess": "ai",
  "confidence": 0.8484,
  "model_score": 0.9969,
  "style_score": 0.7,
  "label": "This shows strong signs of being written by AI. If you wrote it yourself, you can appeal and a person will review it.",
  "error": null,
  "verdict": "BROKE"
}
```

### MF07 — `malformed`

*Targeting:* text is right-to-left script

```json
{
  "id": "MF07",
  "status": 200,
  "content_id": "c_213c5ff7edbe",
  "creator_id": "attacker_MF07",
  "guess": "ai",
  "confidence": 0.8032,
  "model_score": 0.9064,
  "style_score": 0.7,
  "label": "This shows strong signs of being written by AI. If you wrote it yourself, you can appeal and a person will review it.",
  "error": null,
  "verdict": "BROKE"
}
```

### MF08 — `malformed`

*Targeting:* text is 60,000 characters

```json
{
  "id": "MF08",
  "status": 200,
  "content_id": "c_d34b4a134cd8",
  "creator_id": "attacker_MF08",
  "guess": "ai",
  "confidence": 0.8797,
  "model_score": 0.9876,
  "style_score": 0.7718,
  "label": "This shows strong signs of being written by AI. If you wrote it yourself, you can appeal and a person will review it.",
  "error": null,
  "verdict": "BROKE"
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
  "verdict": "HELD"
}
```

### RQ02 — `malformed`

*Targeting:* no creator_id

```json
{
  "id": "RQ02",
  "status": 200,
  "content_id": "c_8451478270c0",
  "creator_id": null,
  "guess": "unsure",
  "confidence": 0.4698,
  "model_score": 0.2395,
  "style_score": 0.7,
  "label": "We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it.",
  "error": null,
  "verdict": "HELD"
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
  "verdict": "HELD"
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
  "verdict": "HELD"
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
  "verdict": "HELD"
}
```

### RQ06 — `malformed`

*Targeting:* creator_id is an object

```json
{
  "id": "RQ06",
  "status": 200,
  "content_id": "c_e00c2c615c17",
  "creator_id": {
    "id": "x"
  },
  "guess": "unsure",
  "confidence": 0.3746,
  "model_score": 0.0492,
  "style_score": 0.7,
  "label": "We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it.",
  "error": null,
  "verdict": "BROKE"
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
  "verdict": "HELD"
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
  "verdict": "HELD"
}
```

### RQ10 — `malformed`

*Targeting:* body is form-encoded, not JSON

```json
{
  "id": "RQ10",
  "status": 429,
  "content_id": null,
  "creator_id": null,
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": "HELD"
}
```

### RQ11 — `malformed`

*Targeting:* body is JSON claiming to be plain text

```json
{
  "id": "RQ11",
  "status": 429,
  "content_id": null,
  "creator_id": null,
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": "HELD"
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
  "verdict": "HELD"
}
```

### FL01_01 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_01",
  "status": 200,
  "content_id": "c_97011883a949",
  "creator_id": "attacker_flood",
  "guess": "unsure",
  "confidence": 0.3643,
  "model_score": 0.0287,
  "style_score": 0.7,
  "label": "We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it.",
  "error": null,
  "verdict": "HELD"
}
```

### FL01_02 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_02",
  "status": 200,
  "content_id": "c_bb6f57008d27",
  "creator_id": "attacker_flood",
  "guess": "unsure",
  "confidence": 0.3643,
  "model_score": 0.0287,
  "style_score": 0.7,
  "label": "We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it.",
  "error": null,
  "verdict": "HELD"
}
```

### FL01_03 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_03",
  "status": 200,
  "content_id": "c_11359fd49cc7",
  "creator_id": "attacker_flood",
  "guess": "unsure",
  "confidence": 0.3643,
  "model_score": 0.0287,
  "style_score": 0.7,
  "label": "We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it.",
  "error": null,
  "verdict": "HELD"
}
```

### FL01_04 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_04",
  "status": 200,
  "content_id": "c_5c80533f9acf",
  "creator_id": "attacker_flood",
  "guess": "unsure",
  "confidence": 0.3643,
  "model_score": 0.0287,
  "style_score": 0.7,
  "label": "We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it.",
  "error": null,
  "verdict": "HELD"
}
```

### FL01_05 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_05",
  "status": 200,
  "content_id": "c_0c5e235d7adb",
  "creator_id": "attacker_flood",
  "guess": "unsure",
  "confidence": 0.3643,
  "model_score": 0.0287,
  "style_score": 0.7,
  "label": "We can't tell whether a person or AI wrote this. That happens a lot with plain, everyday writing, so it is not an accusation. If you wrote it yourself, you can ask a person to review it.",
  "error": null,
  "verdict": "HELD"
}
```

### FL01_06 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_06",
  "status": 429,
  "content_id": null,
  "creator_id": "attacker_flood",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": "HELD"
}
```

### FL01_07 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_07",
  "status": 429,
  "content_id": null,
  "creator_id": "attacker_flood",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": "HELD"
}
```

### FL01_08 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_08",
  "status": 429,
  "content_id": null,
  "creator_id": "attacker_flood",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": "HELD"
}
```

### FL01_09 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_09",
  "status": 429,
  "content_id": null,
  "creator_id": "attacker_flood",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": "HELD"
}
```

### FL01_10 — `flood_same_creator`

*Targeting:* 20 rapid submissions from one creator

```json
{
  "id": "FL01_10",
  "status": 429,
  "content_id": null,
  "creator_id": "attacker_flood",
  "guess": null,
  "confidence": null,
  "model_score": null,
  "style_score": null,
  "label": null,
  "error": null,
  "verdict": "HELD"
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
  "verdict": "HELD"
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
  "verdict": "HELD"
}
```
