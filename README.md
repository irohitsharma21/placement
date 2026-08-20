# JSS Talent Check — Assessment 2 : Complete Prep Pack

> **Exam:** Tomorrow, 9:00 AM – 1:00 PM. Report **9:00 AM sharp** (1 minute late = not allowed to sit).
> **Bring:** University ID card, pinned/worn and visible. **Nothing electronic.**
> **Language chosen:** C++

---

## 1. What the exam actually is

| Part | Contents | Time | Format |
|---|---|---|---|
| **Coding** | 5 questions — OOPs, Arrays, Stacks & Queues, Linked List, Binary Trees | 120 min | Write & run code in browser (SEB) |
| **TCR** | ~60 questions — SQL + OOPs + FSD (incl. DevOps) | 60 min | MCQ **+ written reasoning for each** |

**Scoring twist on TCR:** marks = correctness of option **+ quality of your written reasoning.** A right answer with no reasoning ≠ full marks. This is the section most people underestimate. See [tcr/05_REASONING_PLAYBOOK.md](tcr/05_REASONING_PLAYBOOK.md).

**Two hard mechanics you must respect:**
1. **A completed coding question LOCKS.** You cannot return to it. Order matters enormously.
2. **Runs are limited.** Usable runs = Total runs − 1 (last one is your submission). You must debug in your head, not by spamming Run.

Full tactical breakdown → **[00_EXAM_STRATEGY.md](00_EXAM_STRATEGY.md)** — read this first, and again in the morning.

---

## 2. File map

### Exam-day
| File | What it is |
|---|---|
| [00_EXAM_STRATEGY.md](00_EXAM_STRATEGY.md) | Question order, run budgeting, time splits, panic protocol |
| [CHEATSHEET_FINAL_REVISION.md](CHEATSHEET_FINAL_REVISION.md) | The 30-minutes-before-exam single sheet |

### Coding (120 min section)
| File | Topic |
|---|---|
| [coding/00_CPP_TEMPLATE_AND_STL.md](coding/00_CPP_TEMPLATE_AND_STL.md) | Boilerplate, I/O patterns, full STL cheat sheet, compile-error fixes |
| [coding/01_OOPS_CODING.md](coding/01_OOPS_CODING.md) | C++ OOP syntax + the classic "design a class" questions |
| [coding/02_ARRAYS.md](coding/02_ARRAYS.md) | Two pointers, sliding window, prefix sum, Kadane, binary search, matrix |
| [coding/03_STACKS_QUEUES.md](coding/03_STACKS_QUEUES.md) | Monotonic stack, parentheses, min stack, histogram, deque, LRU |
| [coding/04_LINKED_LIST.md](coding/04_LINKED_LIST.md) | Reverse, cycle, merge, k-groups, palindrome, clone, sort |
| [coding/05_BINARY_TREES.md](coding/05_BINARY_TREES.md) | Traversals, views, LCA, diameter, BST ops, serialize |
| [coding/06_POWER_QUESTIONS.md](coding/06_POWER_QUESTIONS.md) | 15 famous questions that each teach several concepts at once |

### TCR (60 min section)
| File | Topic |
|---|---|
| [tcr/01_SQL.md](tcr/01_SQL.md) | Joins, GROUP BY, subqueries, window fns, normalization, ACID, indexes |
| [tcr/02_OOPS_THEORY.md](tcr/02_OOPS_THEORY.md) | 4 pillars, overloading vs overriding, vtables, SOLID, patterns |
| [tcr/03_FSD.md](tcr/03_FSD.md) | HTTP/REST, JS, React, Node/Express, DB choice, architecture |
| [tcr/04_DEVOPS.md](tcr/04_DEVOPS.md) | Git, Docker, Kubernetes, Jenkins/CI-CD — **confirmed in the exam sample** |
| [tcr/05_REASONING_PLAYBOOK.md](tcr/05_REASONING_PLAYBOOK.md) | How to write reasoning that scores full marks, with templates |
| [tcr/06_PRACTICE_BANK.md](tcr/06_PRACTICE_BANK.md) | 70 practice MCQs with answers **and model reasoning** |

---

## 3. Tonight's study plan (8+ hours available)

Do it in this order. The order is deliberate — it front-loads what is both **high-probability** and **fast to absorb**.

| Block | Time | Do this |
|---|---|---|
| **1** | 0:00 – 0:20 | [00_EXAM_STRATEGY.md](00_EXAM_STRATEGY.md) + [coding/00_CPP_TEMPLATE_AND_STL.md](coding/00_CPP_TEMPLATE_AND_STL.md). Memorise the template until you can type it blind. |
| **2** | 0:20 – 1:20 | [coding/04_LINKED_LIST.md](coding/04_LINKED_LIST.md) — highest-density topic, most predictable questions. **Hand-write** reverse + cycle + merge without looking. |
| **3** | 1:20 – 2:20 | [coding/05_BINARY_TREES.md](coding/05_BINARY_TREES.md) — traversals, height, LCA, views, BST validate. |
| **4** | 2:20 – 3:10 | [coding/03_STACKS_QUEUES.md](coding/03_STACKS_QUEUES.md) — monotonic stack pattern is the whole topic. |
| **5** | 3:10 – 4:00 | [coding/02_ARRAYS.md](coding/02_ARRAYS.md) — patterns, not problems. |
| **6** | 4:00 – 4:45 | [coding/01_OOPS_CODING.md](coding/01_OOPS_CODING.md) — this also doubles as TCR OOPs revision. |
| **7** | 4:45 – 5:00 | **Break. Eat. Walk.** Non-negotiable. |
| **8** | 5:00 – 6:00 | [tcr/01_SQL.md](tcr/01_SQL.md) — biggest single TCR chunk and the easiest marks. |
| **9** | 6:00 – 6:40 | [tcr/02_OOPS_THEORY.md](tcr/02_OOPS_THEORY.md) — mostly revision by now. |
| **10** | 6:40 – 7:30 | [tcr/03_FSD.md](tcr/03_FSD.md) + [tcr/04_DEVOPS.md](tcr/04_DEVOPS.md) |
| **11** | 7:30 – 8:00 | [tcr/05_REASONING_PLAYBOOK.md](tcr/05_REASONING_PLAYBOOK.md) — memorise the 3-sentence formula. |
| **12** | 8:00 – 9:00 | [tcr/06_PRACTICE_BANK.md](tcr/06_PRACTICE_BANK.md) — cover the answers, do them timed at 45 sec each. |
| **13** | Morning | [CHEATSHEET_FINAL_REVISION.md](CHEATSHEET_FINAL_REVISION.md) only. Nothing new. |

**If you fall behind, cut in this order:** cut block 5 (arrays — you likely already know these), then block 12 (do half), then block 6. **Never cut blocks 1, 2, 8, or 11.**

### How to actually study, not just read
- Every code block: **cover it, write it from memory on paper, then compare.** Reading code creates recognition, not recall. Tomorrow you need recall.
- For each topic, you must be able to answer without looking: *"What is the pattern? What is the time complexity? What is the edge case that breaks it?"*
- **Sleep at least 6 hours.** A tired brain at 9 AM loses more marks than one extra hour of notes gains. If it's already late, cut the plan, don't cut the sleep.

---

## 4. The single most important idea in this pack

> **This exam does not reward knowing things. It rewards *explaining* things (TCR) and *getting code right the first time* (limited runs).**

Both come from the same skill: being able to say out loud **why** a solution works before you write it. Every file here is built to train that. When you read a solution, don't ask "what is the code?" — ask **"what is the invariant that makes this correct?"**

Good luck. You've got a full night — use it deliberately.
