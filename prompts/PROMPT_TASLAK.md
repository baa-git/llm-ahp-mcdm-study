# Prompt drafts v2 (awaiting approval)

Three conditions, two tasks each (main criteria, Delivery sub-criteria). Temperature 0.6, reasoning off or minimal.
Main run: one response per matrix, no revision; every matrix is kept and aggregated regardless of CR,
exactly as for the experts. Revision is a separate, later step (Phase 2).

---

## Condition 1 — GUIDED

The original thesis prompt, byte-for-byte (`prompt_exports/eda_ahp_system_prompt.txt`,
`eda_main_criteria_user_prompt.json`, `eda_delivery_subcriteria_user_prompt.json`).
It contains a priority order, hint-bearing criterion descriptions, the supplier KPI report,
and the "never use all 1s" rule.

---

## Condition 2 — LEAN

### System message

```
You are a procurement professional with experience in supplier evaluation.
Answer the following questions. Respond only in the requested JSON format.
```

### User message — main criteria

```
Five criteria used in supplier evaluation are compared in pairs.

For each question: "Which of the following two criteria is more important to you?"

Answer scale (left criterion = A, right criterion = B):
9   : A is extremely more important than B.
8   : A is between very strongly and extremely more important than B. (Intermediate value)
7   : A is very strongly more important than B.
6   : A is between strongly and very strongly more important than B. (Intermediate value)
5   : A is strongly more important than B.
4   : A is between moderately and strongly more important than B. (Intermediate value)
3   : A is moderately more important than B.
2   : A is between equally and moderately more important than B. (Intermediate value)
1   : A and B are equally important.
1/2 : B is between equally and moderately more important than A. (Intermediate value)
1/3 : B is moderately more important than A.
1/4 : B is between moderately and strongly more important than A. (Intermediate value)
1/5 : B is strongly more important than A.
1/6 : B is between strongly and very strongly more important than A. (Intermediate value)
1/7 : B is very strongly more important than A.
1/8 : B is between very strongly and extremely more important than A. (Intermediate value)
1/9 : B is extremely more important than A.

Questions:
1.  Left: Delivery and Operational Performance | Right: Financial Performance
2.  Left: Delivery and Operational Performance | Right: Quality Performance
3.  Left: Delivery and Operational Performance | Right: Volume and Logistics
4.  Left: Delivery and Operational Performance | Right: Compliance and Reliability
5.  Left: Financial Performance | Right: Quality Performance
6.  Left: Financial Performance | Right: Volume and Logistics
7.  Left: Financial Performance | Right: Compliance and Reliability
8.  Left: Quality Performance | Right: Volume and Logistics
9.  Left: Quality Performance | Right: Compliance and Reliability
10. Left: Volume and Logistics | Right: Compliance and Reliability

Response format (only this JSON):
{"answers": [{"question": 1, "value": "3"}, {"question": 2, "value": "1/5"}, ...]}
```

### User message — Delivery sub-criteria

Same text; first lines and questions change:

```
Four sub-criteria of "Delivery and Operational Performance" are compared in pairs.

For each question: "Which of the following two sub-criteria is more important to you?"

[same scale]

Questions:
1. Left: Delivered Orders | Right: Pending Orders
2. Left: Delivered Orders | Right: Cancelled Orders
3. Left: Delivered Orders | Right: Partially Delivered Orders
4. Left: Pending Orders | Right: Cancelled Orders
5. Left: Pending Orders | Right: Partially Delivered Orders
6. Left: Cancelled Orders | Right: Partially Delivered Orders
```

---

## Condition 3 — COT (chain of thought)

System message, scale and questions identical to LEAN. Before the response format, add:

```
Before answering, think step by step:
1. State in one sentence what each criterion represents in supplier evaluation.
2. Rank the criteria from most to least important in your view.
3. Choose each value so that it is consistent with this ranking
   (for example, if A is more important than B and B than C, A must be even more important than C).

Response format (only this JSON):
{"reasoning": "your brief reasoning for steps 1-3",
 "ranking": ["most important criterion", "...", "least important criterion"],
 "answers": [{"question": 1, "value": "3"}, ...]}
```

No priority hint, description or data is given for any criterion.

---

## Phase 2 — revision prompt (later; only for matrices with CR > 0.10)

```
The comparison matrix built from your answers has a consistency ratio CR = {cr}.
The accepted limit is 0.10; your answers contradict each other.

Most contradictory comparisons:
{contradictions}

Your previous answers:
{previous_answers}

Review your answers. You do not have to change your view; make only the changes needed
to remove the contradiction.
Use only these values: 1/9, 1/8, 1/7, 1/6, 1/5, 1/4, 1/3, 1/2, 1, 2, 3, 4, 5, 6, 7, 8, 9.
You may change the direction of an answer if that is what resolves a contradiction.
Answer all questions again in the same JSON format.
```

It never says which criterion should be more important. Revision limit is set explicitly and logged.

Protocol v2 (prompt ids `revision_..._v2`):
- Consistent means CR rounded to 4 decimals is at most 0.10.
- `{contradictions}` lists the three most inconsistent triads and, for every circular (ordinal) triad
  (max 3), a line "Questions a, b and c form a circular preference: X over Y, Y over Z, Z over X."
- Each revision is built from the best valid answers so far (lowest CR among the original and all
  parsed revisions): original messages + one assistant turn (the original reply, or the best
  answers in the condition's JSON format when they come from a revision) + the revision prompt.
  The chain's final answers are the best ones.

---

## Design notes

- Per-question answers, not a matrix; the code builds the reciprocal matrix.
- The word "survey" is not used. Role line mirrors the expert panel (procurement practitioners).
- LEAN asks for no rationale. Qualitative analysis uses COT "reasoning" and the original GUIDED rationales.
- Question order follows the questions the experts were asked. Order-effect test is optional and separate.
