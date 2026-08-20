# TCR 05 — The Reasoning Playbook

> **This file is the difference between a good score and a great one.**
>
> The official instructions say it plainly: *"Your score is based on both the correctness of the option you select **and** the quality of your reasoning. A correct answer with strong, logical reasoning scores highest. Simply guessing the right option without sound reasoning will not earn full marks."*
>
> Almost everyone revises SQL and OOPs. Almost nobody practises **writing the justification under time pressure.** Spend 30 minutes on this file and you gain marks that content revision alone cannot buy.

---

## 1. The constraints you are writing under

From the exam screenshots:
- The reasoning field is **mandatory** and has a **minimum of 10 characters**.
- The box holds up to **500 characters**. The sample model answer was about **215 characters** — roughly **two sentences**.
- Questions carry different weights (the samples showed 1-mark and 2-mark items) and difficulty tags (EASY / MEDIUM).
- There is a **"Skip — I'm unsure"** option. Treat it as a last resort, not a convenience.
- **~60 questions in 60 minutes = 60 seconds each**, including reading, deciding, and typing.

**Therefore your reasoning must be:** two to three sentences, written in under 25 seconds, and structurally identical every time so you never have to invent a format mid-exam.

---

## 2. THE FORMULA — memorise this

> **Sentence 1 — DEFINE.** State what the key concept *is*.
> **Sentence 2 — CONNECT.** Explain why that makes *this option* correct *in this scenario*.
> **Sentence 3 — ELIMINATE.** Dismiss the strongest wrong option in one clause.

Sentence 3 is optional when time is tight — but it is what separates a good answer from a full-marks answer, because it proves you considered the alternatives rather than pattern-matching.

### The formula applied to the PDF's own sample

**Q: What is a Jenkins Pipeline?** (Container / Repository / **CI/CD workflow** / VM)

> **[DEFINE]** A Jenkins Pipeline is a set of automated stages defined in a Jenkinsfile that build, test and deploy software.
> **[CONNECT]** It therefore represents the CI/CD workflow, automating the delivery process from a code change through to deployment.
> **[ELIMINATE]** A container or VM is merely where a stage might execute, and a repository only stores the source.

That is 3 sentences, ~250 characters, and it earns full marks.

### The formula applied to the second sample

**Q: A Kubernetes application is healthy internally but users cannot access it externally. Pods are running and passing readiness checks. What should be investigated first?**
(**Service and Ingress configuration** / Delete and recreate all pods / Upgrade the cluster / Increase node CPU)

> **[DEFINE]** Readiness probes passing confirms the pods are healthy and are receiving internal traffic, so the failure lies in how the application is exposed outside the cluster.
> **[CONNECT]** That exposure is controlled by the Service and Ingress — a default ClusterIP Service is internal-only, and a broken Ingress rule blocks external routing.
> **[ELIMINATE]** Recreating pods, upgrading the cluster or adding CPU do not touch the networking layer the symptom points to.

---

## 3. Weak vs strong reasoning — see the difference

**Q: Why use `HAVING` instead of `WHERE` when filtering on `COUNT(*) > 5`?**

| Quality | What it looks like | Why it scores that way |
|---|---|---|
| **Zero marks** | "Because HAVING is correct here." | Restates the answer. No concept. |
| **Weak** | "WHERE cannot be used with COUNT." | True, but states a rule without the reason. |
| **Strong** | "WHERE is evaluated before rows are grouped, so aggregate values do not exist yet; HAVING is evaluated after GROUP BY and can therefore filter on COUNT(*). Filtering with WHERE first is still preferred for non-aggregate conditions, since fewer rows reach the grouping stage." | Explains the *mechanism* (execution order), applies it, and adds a correct nuance. |

**The test to apply to your own sentence:** *does it explain the mechanism, or does it just assert the rule?* Mechanism scores; assertion does not.

---

## 4. Domain-specific templates

Fill in the blanks. These cover the vast majority of what you will be asked.

### SQL
- *"[Clause A] is evaluated before [Clause B] in SQL's execution order, so ____. This is why ____ is correct."*
- *"A [join type] preserves ____, which means unmatched rows appear as NULL; the option chosen accounts for that, while ____ would silently discard them."*
- *"An index converts a full scan into a B+ tree lookup, so ____ improves read performance at the cost of slower writes."*
- *"This violates [normal form] because ____ depends on ____ rather than on the whole primary key, causing update anomalies."*
- *"[ACID property] guarantees ____, which is exactly what this scenario requires."*

### OOPs
- *"[Concept] means ____, which is achieved through ____. In this scenario that produces ____."*
- *"This is resolved at [compile/run] time, because ____, making it [static/dynamic] polymorphism."*
- *"Without ____, the derived part of the object would ____, which is why ____ is required."*
- *"This satisfies the [SOLID principle] principle: ____ can be extended without modifying existing tested code."*

### FSD / Web
- *"HTTP is stateless, so ____; this option preserves that property while ____ would require server-side state."*
- *"[Status code] indicates ____, whereas ____ indicates ____ — the scenario describes the former."*
- *"React re-renders when ____, so ____; using ____ instead would break ____."*
- *"Since the failure appears at the [layer] layer while [other layer] is healthy, the investigation should begin at ____."*

### DevOps
- *"[Tool] is responsible for ____ in the delivery pipeline, so ____ is the correct description."*
- *"The symptom indicates the ____ layer is functioning, which narrows the cause to ____."*
- *"Containers share the host kernel, so ____, unlike virtual machines which ____."*

---

## 5. When you do NOT know the answer

**Never leave the box empty. Never click Skip and forget it.** Reasoning is marked partly on its own merit, so a well-argued wrong option beats a blank.

**Use elimination reasoning, and say so honestly:**
> *"Options A and C both describe caching behaviour, so they cannot both be right and are likely distractors. D contradicts the premise that the pods are healthy. B is the only option that addresses the external routing path described in the question, so I select B."*

That is a genuinely strong paragraph. It shows structured thinking even without certainty, and examiners reward it.

**The elimination ladder, in priority order:**
1. Options containing **always / never / only / all** are usually wrong — real systems have trade-offs.
2. In debugging questions, prefer the option that **investigates before it destroys.** "Check the configuration" beats "delete and recreate everything."
3. Match the **scope** of the question. External problem -> external component. Read-performance question -> read-path answer.
4. If **two options say the same thing**, both are usually wrong.
5. In conceptual questions, the **longest, most qualified option** is often right, because accurate technical statements need caveats.
6. Choose what a **senior engineer** would say, not the dramatic option.

---

## 6. Words that make reasoning sound rigorous

Sprinkle these — they signal causal thinking rather than recall.

| Function | Phrases |
|---|---|
| Cause | "because", "since", "as a result of", "which is why" |
| Consequence | "therefore", "consequently", "this means that", "so" |
| Contrast | "whereas", "unlike", "in contrast to", "while X does Y, Z does W" |
| Elimination | "the alternative would", "this rules out", "cannot apply here because" |
| Trade-off | "at the cost of", "in exchange for", "the trade-off is" |
| Precision | "specifically", "in this scenario", "given that the question states" |

**Phrases to avoid:** "I think", "maybe", "it seems", "obviously", "everyone knows". They add no information and weaken the answer.

**Always anchor to the question's own words.** If the question says *"users report they cannot access it externally"*, use the word **externally** in your reasoning. It proves you read the scenario rather than recognising a keyword.

---

## 7. The 60-second execution routine

Practise until it is automatic:

```
0-10 sec   Read the question. Underline mentally the ONE word that decides it
           (externally / first / before / not / always / average).
10-25 sec  Eliminate two options. Pick one.
25-50 sec  Type the reasoning: DEFINE -> CONNECT -> ELIMINATE.
50-60 sec  Click Next. Do not re-read. Do not second-guess.
```

**Time discipline rules:**
- If you are at **90 seconds** on a question, **Flag it and move on immediately.** One question is never worth two others.
- **Pass 1 (0–45 min):** everything you are confident on. **Pass 2 (45–58 min):** the flagged ones. **58–60 min:** confirm the navigation grid shows nothing unanswered, then submit.
- Watch the answered counter shown at the bottom of the screen. At the 30-minute mark you should be near question 30. If you are at 20, speed up — write two sentences instead of three.

---

## 8. Do not over-write

The single most common way to lose marks here is **running out of time because early answers were too long.**

A 500-character essay on question 5 that costs you three unanswered questions at the end is a net loss. **Two precise sentences on all 60 questions beats five beautiful sentences on 40.**

Rule of thumb: **if your reasoning would take more than 25 seconds to type, cut it.**

---

## 9. Ten fully-worked examples

Cover the reasoning, write your own, then compare.

**1. Q: Which SQL clause filters groups after aggregation?** (WHERE / HAVING / GROUP BY / ORDER BY)
> **HAVING.** SQL evaluates WHERE before rows are grouped, so aggregate values do not yet exist at that point. HAVING runs after GROUP BY and can therefore filter on aggregates such as COUNT(*) or AVG(salary). WHERE remains the right choice for row-level conditions.

**2. Q: A base class pointer holding a derived object calls a non-virtual method. Which version runs?** (Base / Derived / Compile error / Undefined)
> **Base.** Without `virtual`, the call is resolved by static binding using the pointer's declared type, so the compiler selects the base implementation. Only virtual functions are dispatched at run time via the vtable using the object's actual type.

**3. Q: Why is PUT idempotent but POST is not?**
> **PUT** replaces a resource with the complete state supplied at a known URI, so sending it repeatedly leaves the resource in the same final state. POST creates a new subordinate resource on each call, so repeating it produces duplicates. Idempotency describes the resulting state, not the response body.

**4. Q: Pods are Running and readiness probes pass, but external users cannot reach the app. Investigate first?**
> **Service and Ingress configuration.** Passing readiness checks confirms the pods are healthy and receiving internal traffic, so the fault lies in the external exposure path. A default ClusterIP Service is internal-only and a misconfigured Ingress rule blocks routing from outside, whereas restarting pods or adding CPU does not touch that networking layer.

**5. Q: Complexity of the sliding-window-maximum deque solution?** (O(nk) / O(n log n) / **O(n)** / O(n^2))
> **O(n).** Although there is an inner while loop, each index is pushed onto the deque exactly once and popped at most once across the entire run. The total work is therefore proportional to n, not to n times k, which is what the nested loop superficially suggests.

**6. Q: Why must a base class destructor be virtual?**
> Deleting a derived object through a base-class pointer with a non-virtual destructor invokes only the base destructor, leaving the derived part's resources unreleased. Declaring it virtual makes the destructor call dynamically dispatched, so the derived destructor runs first and then the base's, releasing everything.

**7. Q: A LEFT JOIN with a WHERE condition on the right table returns fewer rows than expected. Why?**
> The LEFT JOIN preserves unmatched left rows by filling the right table's columns with NULL. A WHERE predicate on one of those columns evaluates to UNKNOWN for those rows and discards them, so the query behaves as an inner join. Moving the condition into the ON clause preserves the outer-join semantics.

**8. Q: Why is using an array index as a React key an anti-pattern?**
> Keys give React a stable identity for each list item during reconciliation. An index is positional rather than identity-based, so when the list is reordered, filtered or has items inserted, the same key maps to a different item — causing React to reuse the wrong DOM nodes and attach component state to the wrong row.

**9. Q: Why do containers start faster than virtual machines?**
> A container shares the host kernel and virtualises only the operating system layer, so starting one is essentially starting a process. A virtual machine virtualises hardware and must boot an entire guest operating system, which is why it takes minutes and gigabytes rather than seconds and megabytes.

**10. Q: Which index would speed up `WHERE dept_id = 5 AND salary > 50000`?** (index on salary / **composite on (dept_id, salary)** / index on name / no index)
> A composite index on (dept_id, salary) matches the query's access pattern: the equality predicate on dept_id narrows the B+ tree to a contiguous range, within which the salary range is scanned directly. By the leftmost-prefix rule an index on salary alone could not serve the dept_id filter efficiently.

---

## 10. Your 20-minute practice drill

1. Open [06_PRACTICE_BANK.md](06_PRACTICE_BANK.md).
2. Cover the answers.
3. Set a timer for 45 seconds per question.
4. **Write the reasoning out** — on paper or by typing. Do not just think it. The bottleneck tomorrow is *producing* sentences under pressure, and only writing trains that.
5. Compare with the model answer. Score yourself on whether you explained the **mechanism**, not just the rule.

Do 20 questions this way and the format will be automatic by morning.

---

## 11. The one-line summary to carry into the exam

> **Define the concept. Connect it to this scenario. Dismiss the best distractor. Two to three sentences. Move on.**

Next: [06_PRACTICE_BANK.md](06_PRACTICE_BANK.md)
