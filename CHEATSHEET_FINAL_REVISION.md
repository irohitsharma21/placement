# Final Revision Cheat Sheet

> **Read this in the last 30 minutes before the exam. Learn nothing new.** This page exists to reload what you already know, not to teach.

---

## The exam in one box

```
9:00 AM sharp — 1 minute late = not allowed to sit. ID card pinned and visible.
Your allotted seat ONLY. No phone, no smartwatch, no earphones.

CODING  120 min  5 questions  (OOPs / Arrays / Stacks & Queues / Linked List / Trees)
TCR      60 min  ~60 MCQs     (SQL + OOPs + FSD/DevOps) — reasoning required per question

>> Completed coding questions LOCK. Choose your order carefully.
>> Usable runs = Total runs - 1. Dry-run on paper BEFORE you press Run.
>> Auto-saved at every step. A crash loses nothing. Wait 10-20s, retry ONCE, then raise your hand.
```

## The plan

```
0-6 min      Read all 5 questions. Mark each E / M / H.
6-120 min    Solve EASY -> MEDIUM -> HARD. Hard cap 25 min per question.
             Stuck at 15 min? Write the brute force. Partial marks are real marks.
Then TCR     Pass 1 (0-45): answer what you know, Flag the rest.
             Pass 2 (45-58): the flagged ones.
             58-60: check the grid shows nothing unanswered. Submit.
```

---

## C++ template — type this first

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);

    return 0;
}
```

**Before every Run:** every variable initialised? loop bounds `<` vs `<=`? every `->` null-guarded? `long long` for sums? base case in every recursion? output format matches the sample **exactly**? class definition ends with `};`?

---

## The six pieces of code you must be able to write cold

```cpp
// 1. REVERSE A LINKED LIST
Node* reverse(Node* head) {
    Node *prev = nullptr, *curr = head, *nxt = nullptr;
    while (curr) { nxt = curr->next; curr->next = prev; prev = curr; curr = nxt; }
    return prev;
}

// 2. SLOW / FAST  (middle, cycle, nth-from-end, palindrome all use this)
while (fast && fast->next) { slow = slow->next; fast = fast->next->next; }

// 3. BINARY SEARCH
int lo = 0, hi = n - 1;
while (lo <= hi) {
    int mid = lo + (hi - lo) / 2;
    if (a[mid] == t) return mid;
    else if (a[mid] < t) lo = mid + 1;
    else hi = mid - 1;
}

// 4. KADANE
int best = a[0], cur = a[0];
for (int i = 1; i < n; i++) { cur = max(a[i], cur + a[i]); best = max(best, cur); }

// 5. LEVEL ORDER (BFS)
queue<TreeNode*> q; q.push(root);
while (!q.empty()) {
    int sz = q.size();                      // FREEZE the level boundary
    for (int i = 0; i < sz; i++) {
        TreeNode* n = q.front(); q.pop();
        if (n->left) q.push(n->left);
        if (n->right) q.push(n->right);
    }
}

// 6. MONOTONIC STACK (next greater element)
for (int i = 0; i < n; i++) {
    while (!st.empty() && a[st.top()] < a[i]) { res[st.top()] = a[i]; st.pop(); }
    st.push(i);
}
```

---

## Recursion contracts — say these before you write

| Problem | Base case | Combine |
|---|---|---|
| height | `!r -> 0` | `1 + max(L, R)` |
| count nodes | `!r -> 0` | `1 + L + R` |
| diameter | `!r -> 0` | record `L+R`, return `1+max(L,R)` |
| balanced | `!r -> 0` | `-1` propagates failure upward |
| LCA | `!r or r==p or r==q -> r` | both sides non-null -> this node |
| validate BST | `!r -> true` | carry `(min, max)` bounds downward |

**BST deletion, three cases:** leaf -> remove; one child -> splice it in; two children -> copy the inorder successor's value here, then delete the successor from the right subtree.
**Inorder of a BST is sorted.** Half of all BST questions collapse to this one fact.

---

## Complexity — quote these without hesitating

| | Access | Search | Insert | Delete |
|---|---|---|---|---|
| Array | O(1) | O(n) | O(n) | O(n) |
| Linked list | O(n) | O(n) | **O(1)** at a known position | **O(1)** at a known position |
| Stack / Queue | — | O(n) | O(1) | O(1) |
| Hash table | — | O(1) avg | O(1) avg | O(1) avg |
| Balanced BST | O(log n) | O(log n) | O(log n) | O(log n) |
| Heap | — | O(n) | O(log n) | O(log n) |

Merge sort **O(n log n) always**, O(n) space, stable. Quick sort **O(n^2) worst** (sorted input plus a bad pivot), O(log n) space, in-place, not stable.
**Monotonic stack and deque problems are O(n)** even with the inner `while` — each element is pushed once and popped once. Expect this as a trap.

---

## TCR — THE FORMULA

> **1. DEFINE what the concept is.**
> **2. CONNECT it to why this option is right in this scenario.**
> **3. ELIMINATE the best wrong option in one clause.**
>
> Two to three sentences, roughly 200-250 characters, under 25 seconds.
> **Never leave a reasoning box empty** — a well-argued wrong option beats a blank.

**Elimination ladder when unsure:** absolutes (always / never / only) are usually wrong -> in debugging questions prefer *investigate* over *destroy* -> match the scope of the question -> two options meaning the same thing are both wrong -> pick what a senior engineer would say.

---

## SQL — the ten highest-yield facts

1. Execution order: **FROM -> WHERE -> GROUP BY -> HAVING -> SELECT -> ORDER BY**. Hence no SELECT alias in WHERE, but aliases are fine in ORDER BY.
2. **WHERE filters rows before grouping; HAVING filters groups after.** Aggregates only in HAVING.
3. LEFT JOIN keeps all left rows with NULLs; **a WHERE condition on the right table turns it into an inner join** — put that condition in ON instead.
4. `COUNT(*)` counts rows including NULLs; `COUNT(col)` counts non-NULLs. All other aggregates ignore NULLs.
5. `NULL = NULL` is unknown — use `IS NULL`. `NOT IN (..., NULL)` returns nothing at all.
6. **TRUNCATE** = DDL, no WHERE, no rollback, fast. **DELETE** = DML, has WHERE, rollback-able, logged. **DROP** removes the table itself.
7. **1NF** atomic values, **2NF** no partial dependency, **3NF** no transitive dependency.
8. **ACID**: Atomicity (all-or-nothing), Consistency (valid states), Isolation (concurrency), Durability (survives a crash).
9. Index = B+ tree: faster reads, slower writes. **Unusable** when the column is wrapped in a function or the LIKE pattern has a leading `%`.
10. **RANK** skips numbers after ties (1,1,3); **DENSE_RANK** does not (1,1,2). Use DENSE_RANK for "Nth highest".

---

## OOPs — the ten highest-yield facts

1. **Encapsulation hides data** (access modifiers); **abstraction hides complexity** (interfaces).
2. **Overloading** = same name, different parameters, resolved at compile time. **Overriding** = same signature, base vs derived, resolved at run time via the vtable.
3. **Base destructors must be virtual**, or `delete basePtr` leaks the derived part.
4. **Constructors cannot be virtual** — the vptr is not established yet.
5. Construction order: **base then derived**. Destruction: the exact reverse.
6. **Rule of Three:** needing a destructor implies needing a copy constructor and copy assignment operator.
7. **Object slicing:** assigning a derived object to a base **by value** discards derived data. Use references or pointers.
8. An abstract class has at least one pure virtual (`= 0`) and cannot be instantiated.
9. **Diamond problem** -> `virtual` inheritance makes the base sub-object shared.
10. **SOLID:** Single responsibility, Open/closed, Liskov substitution, Interface segregation, Dependency inversion.

---

## FSD & DevOps — the highest-yield facts

1. **GET** safe and idempotent; **PUT** idempotent but not safe; **POST** neither.
2. **401** = not authenticated. **403** = authenticated but not permitted. **404** not found, **500** server error.
3. **HTTP is stateless** — which is exactly what enables horizontal scaling behind a load balancer.
4. **A JWT is signed, not encrypted** — never put secrets in the payload.
5. **Authentication** = who you are; **authorization** = what you may do. Authentication comes first.
6. **Passwords: hashed and salted with bcrypt or argon2.** Never encrypted, never plain text.
7. **SQL injection -> parameterised queries. XSS -> escape output plus CSP. CSRF -> tokens plus SameSite.**
8. **CORS errors are fixed on the server**, by sending the right Access-Control-Allow-Origin header.
9. **Event loop:** microtasks (promises) drain completely before macrotasks (setTimeout). The answer is `1 4 3 2`.
10. **`const`** prevents rebinding, not mutation.
11. **Props** come from the parent and are read-only; **state** is internal and triggers a re-render.
12. **Never use the array index as a React key** when the list can be reordered or filtered.
13. **Node is single-threaded and non-blocking** — excellent for I/O, poor for CPU-bound work.
14. **Container** shares the host kernel (seconds, MB); **VM** carries a full guest OS (minutes, GB). **Image** = blueprint, **container** = running instance.
15. **Kubernetes:** the Pod is the smallest unit and pods are ephemeral, so a **Service** provides a stable endpoint and **Ingress** routes external traffic. Liveness failure **restarts** the container; readiness failure only **removes it from the endpoints**. Healthy pods but no external access -> **check Service and Ingress**.
16. **A Jenkins Pipeline is a CI/CD workflow** defined as stages in a Jenkinsfile.
17. **Continuous Delivery** keeps a manual approval before production; **Continuous Deployment** removes it.

---

## The final word

Three things to hold onto when the clock starts:

1. **You already cleared Assessment 1.** This material is within reach — you have prepared for it.
2. **Partial marks are everywhere.** A brute force that runs beats an elegant solution that does not compile. A well-argued wrong option beats an empty box. Always put *something* down.
3. **Speed comes from deciding, not typing.** Read, choose, commit, move on. Do not reopen settled decisions.

Breathe. Read each question twice. Dry-run before you Run.

**You've got this.**
