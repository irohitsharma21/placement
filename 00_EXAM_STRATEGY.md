# 00 — Exam Day Strategy

> Read tonight. Read again at 8:45 AM. This file is worth more marks than any single algorithm.

---

## 1. The three rules that make this exam different

### Rule A — Completed questions LOCK
Once you finish a coding question, it is sealed. No going back, no improving it later.

**Consequence:** never "finish" a question you're unhappy with just to move on — but also never sit on a question forever hoping to perfect it. You need a *decision point*.

### Rule B — Usable runs = Total runs − 1
The last run is reserved for final submission. If the header says 5 runs, you have **4 real attempts** to test, and the 5th is your submit.

**Consequence:** the browser is not your compiler. **Your brain is the compiler.** You dry-run on paper first.

### Rule C — TCR requires written reasoning per question
~60 questions in 60 minutes. That's **60 seconds each**, including writing 2–3 sentences of justification.

**Consequence:** you cannot afford to think slowly. Reasoning must come out as a reflex, from memorised templates. See [tcr/05_REASONING_PLAYBOOK.md](tcr/05_REASONING_PLAYBOOK.md).

---

## 2. Coding section — the 120-minute plan

### Step 1: The 6-minute survey (minutes 0–6)
Before writing a single line, **open and read all 5 questions.** Reading is not "starting" — you only commit when you begin coding a question. (If the portal forces a choice on open, read the visible titles/statements from the list view first.)

For each question note one letter on your rough sheet:
- **E** — I know this cold, I can code it in 10 minutes.
- **M** — I know the approach, will take 20–25 minutes.
- **H** — I'm not sure of the approach.

### Step 2: Order — Easy → Medium → Hard
**Always solve in increasing difficulty.** Reasons:
1. Locked questions mean an early disaster on a hard question costs you the whole paper.
2. Early wins build momentum and calm your hands.
3. If time runs out, it runs out on the question worth the least to you.

**Never** start with the question that "looks most interesting."

### Step 3: Per-question time budget

| Phase | Time | What you do |
|---|---|---|
| Understand | 2 min | Re-read statement. Write down: input format, output format, constraints, 2 sample cases. |
| Design | 3–5 min | Write the approach in 3 bullet points **on paper**. Name the pattern. State the complexity. |
| Code | 10–15 min | Type it. No experimenting. |
| **Dry run** | **4 min** | **On paper, with the sample input, line by line.** This is the step everyone skips and it is the step that saves your runs. |
| Run | 1 min | Only after dry run passes. |
| Fix + submit | 3 min | |

**Hard cap: 25 minutes per question.** 5 × 25 = 125. You have 120. So one question must come in under budget — that's why you do easy ones first.

### Step 4: The stuck protocol
If you are at **15 minutes with no working approach**:
1. Write the **brute force**. A correct O(n²) beats an incorrect O(n log n) every single time. Partial marks are real marks.
2. Handle the sample input correctly at minimum.
3. Then submit and move on.

**A brute force that runs is always better than an elegant solution that doesn't compile.**

### Step 5: The dry-run discipline (this is the core skill)
Before you press Run, on paper:

```
Input:  [2, 1, 5, 6, 2, 3]
i=0: stack=[], push 0        -> stack=[0]
i=1: h[1]=1 < h[0]=2, pop 0  -> area = 2*1 = 2, ans=2
...
```

You are checking specifically for:
- [ ] **Off-by-one**: is the loop `< n` or `<= n`? Is it `i-1` or `i`?
- [ ] **Empty input**: n = 0. Does it crash?
- [ ] **Single element**: n = 1.
- [ ] **Uninitialised variable** — the #1 silent killer in C++.
- [ ] **Integer overflow** — use `long long` when summing or multiplying.
- [ ] **Null pointer deref** — every `node->next` needs `node != NULL` first.
- [ ] Did you actually **print** the answer, in exactly the format asked?

---

## 3. TCR section — the 60-minute plan

**60 questions, 60 minutes.** Budget:

| Question type | Time | Action |
|---|---|---|
| I know it instantly | 30–40 sec | Select + write 2 sentences. Move. |
| I can work it out | 60–75 sec | Reason it, select, write 2–3 sentences. |
| No idea | 40 sec | **Eliminate 2 options**, pick the best remaining, and write reasoning for *why the others are wrong.* Move. |

### Never leave a question blank
The reasoning box earns marks independently of the option. Even a wrong option with sharp reasoning earns partial credit. **Blank earns zero, guaranteed.** The interface has a "Skip — I'm unsure" button: **do not use it** unless you're returning later.

### The two-pass method
- **Pass 1 (0–45 min):** answer everything you're confident on. Flag the hard ones with the Flag button and move immediately. Do not burn 3 minutes on question 7.
- **Pass 2 (45–58 min):** return to flagged ones. Now you have leftover time and a warmed-up brain.
- **58–60 min:** verify every question shows "answered" in the navigation panel, then submit.

The navigation grid colour-codes Answered / Flagged / Unanswered — use it as your checklist.

### Reasoning length
The sample screenshot showed a **min-10-character** requirement and a 500-character box. The sample answer was ~215 characters — roughly **2 sentences**. That's your target. Don't write essays; you don't have the time and it isn't rewarded.

---

## 4. Answering an MCQ you don't know — the elimination ladder

Apply in this order:

1. **Absolutes are usually wrong.** Options containing *always*, *never*, *only*, *all* are more often distractors — real systems have trade-offs.
2. **The option that "does the least" is often right** in debugging questions. ("What should be investigated *first*?" → check config before you delete/recreate/upgrade anything. The Kubernetes sample question in the PDF is exactly this pattern: *Service and Ingress configuration*, not "delete all pods".)
3. **Match the question's scope.** If it asks about *external* access, the answer is about the *external-facing* component (Service/Ingress), not internal ones (pods, CPU).
4. **Two options that mean the same thing** are both usually wrong — the answer is one of the other two.
5. **The longest, most qualified option** is often correct in conceptual questions, because correct technical statements need caveats.
6. If truly stuck, **pick the option that a senior engineer would say**, not the dramatic one.

---

## 5. Reasoning quality — the 3-sentence formula

Memorise this. It works for SQL, OOPs, FSD and DevOps alike.

> **(1) State what the concept IS.**
> **(2) State WHY that makes this option correct — connect it to the specific scenario in the question.**
> **(3) Rule out the strongest wrong option in one clause.**

**Example (from the PDF's own sample):**
> *"A Jenkins Pipeline is a set of automated stages defined in a Jenkinsfile that build, test and deploy code. It represents a CI/CD workflow rather than a runtime unit, which is why 'CI/CD workflow' is correct — a container or VM is where a job may execute, and a repository only stores the code."*

Notice: defines it → connects it → eliminates the distractors. Three sentences, ~250 characters. That's the model.

---

## 6. The morning checklist

**The night before**
- [ ] University ID card located and ready to pin on.
- [ ] Alarm set for a wake-up that gets you there by **8:45, not 9:00**.
- [ ] Bag packed. Route and room number confirmed.
- [ ] Phone/smartwatch/earphones stay outside the exam hall.
- [ ] Sleep 6+ hours.

**Morning of**
- [ ] Eat something. 4 hours is long and low blood sugar destroys reasoning.
- [ ] Arrive 8:45. **Sit only on your allotted seat number** — wrong seat = malpractice.
- [ ] Skim [CHEATSHEET_FINAL_REVISION.md](CHEATSHEET_FINAL_REVISION.md). Learn nothing new.
- [ ] Take a rough sheet if allowed, and a pen.

**First 60 seconds in the exam**
- [ ] Note the total run count in the header. Compute your usable runs = total − 1. Write it down.
- [ ] Note the clock time you started. Write down the wall-clock time at which you must switch to TCR.

---

## 7. If something goes wrong

The instructions are explicit and reassuring — **your work is auto-saved at every step.** Nothing is lost by a crash.

| Symptom | Action |
|---|---|
| "Failed to fetch" / frozen / blank | Wait **10–20 seconds, retry ONCE.** Do not click Run/Submit repeatedly. |
| Still broken | **Raise your hand.** Tell the invigilator exactly what you were doing and the exact error text. |
| Want to exit SEB | **Never force-quit.** Only the invigilator can, with a secure password. You resume exactly where you left off. |
| Lost time to a technical fault | It is tracked and accounted for. Don't panic-rush afterwards. |

**Do not leave your seat before submitting both parts and exiting SEB** — that's treated as a violation.

---

## 8. Mindset

Three things to hold in your head at 9:00 AM:

1. **You already qualified Assessment 1.** You're in the group that can do this. The material is not beyond you.
2. **Partial marks are everywhere** — brute force code, decent reasoning on a wrong option. The worst strategy is perfectionism; the second worst is giving up on a question. Always put *something* down.
3. **Speed comes from decisions, not typing.** The people who finish are the ones who decide fast and commit, not the ones who type fast.

Now go read [coding/00_CPP_TEMPLATE_AND_STL.md](coding/00_CPP_TEMPLATE_AND_STL.md).
