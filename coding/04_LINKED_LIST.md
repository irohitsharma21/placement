# 04 — Linked Lists

> This is the most **predictable** topic in the paper. There are maybe 15 canonical questions and they recycle endlessly. Two techniques cover almost all of them: **slow/fast pointers** and **three-pointer reversal**. Master those two and you are done.

---

## 0. The node and the setup

```cpp
struct Node {
    int data;
    Node* next;
    Node(int d) : data(d), next(nullptr) {}
};
```

```cpp
// Build a list from n values
Node* buildList(vector<int>& v) {
    Node *head = nullptr, *tail = nullptr;
    for (int x : v) {
        Node* n = new Node(x);
        if (!head) head = tail = n;
        else { tail->next = n; tail = n; }
    }
    return head;
}

// Print
void printList(Node* head) {
    for (Node* c = head; c; c = c->next) cout << c->data << " ";
    cout << "\n";
}

// Length
int length(Node* head) {
    int n = 0;
    for (Node* c = head; c; c = c->next) n++;
    return n;
}
```

### The two rules that prevent 90% of linked list bugs
1. **Never dereference without checking.** Every `p->next` needs `p != nullptr` established first. In a loop condition, order matters: `while (p && p->next)` — the short-circuit means `p->next` is only evaluated when `p` is non-null. `while (p->next && p)` would crash.
2. **Save before you overwrite.** The moment you write `curr->next = something`, the old `curr->next` is gone. Store it in a temp first. This is why reversal needs three pointers.

### The dummy node trick — use it constantly
```cpp
Node dummy(0);
dummy.next = head;
Node* prev = &dummy;
// ... work ...
return dummy.next;      // correct even if the original head was deleted or changed
```
A dummy head removes every "what if it is the first node?" special case. Reach for it in: delete-a-node, merge, remove-duplicates, partition, remove-nth-from-end.

---

## 1. Insertion & deletion

```cpp
Node* insertAtHead(Node* head, int x) {
    Node* n = new Node(x);
    n->next = head;
    return n;                                  // new head
}

Node* insertAtTail(Node* head, int x) {
    Node* n = new Node(x);
    if (!head) return n;
    Node* c = head;
    while (c->next) c = c->next;
    c->next = n;
    return head;
}

Node* insertAtPos(Node* head, int pos, int x) {     // 0-indexed
    if (pos == 0) return insertAtHead(head, x);
    Node* c = head;
    for (int i = 0; i < pos - 1 && c; i++) c = c->next;
    if (!c) return head;                        // position out of range
    Node* n = new Node(x);
    n->next = c->next;                          // 1. link new node forward FIRST
    c->next = n;                                // 2. then link previous to new
    return head;
}
```
**Order matters in insertion:** point the new node forward before you re-point the previous node. Do it the other way and you lose the rest of the list.

```cpp
Node* deleteValue(Node* head, int x) {
    Node dummy(0); dummy.next = head;
    Node* prev = &dummy;
    while (prev->next) {
        if (prev->next->data == x) {
            Node* del = prev->next;
            prev->next = del->next;
            delete del;
            break;                              // remove `break` to delete ALL matches
        }
        prev = prev->next;
    }
    return dummy.next;
}
```

### Delete a node given only a pointer to it (no head)
```cpp
void deleteGivenNode(Node* node) {
    if (!node || !node->next) return;      // impossible for the last node
    node->data = node->next->data;         // copy the next node's value in
    Node* t = node->next;
    node->next = t->next;
    delete t;                              // delete the NEXT node instead
}
```
You cannot unlink a node without its predecessor, so you impersonate the successor and delete that instead. Nice reasoning question.

---

## 2. REVERSE A LINKED LIST — learn this cold

### Iterative (three pointers) — the one to memorise
```cpp
Node* reverse(Node* head) {
    Node *prev = nullptr, *curr = head, *nxt = nullptr;
    while (curr) {
        nxt = curr->next;      // 1. SAVE the rest of the list
        curr->next = prev;     // 2. REVERSE this link
        prev = curr;           // 3. ADVANCE prev
        curr = nxt;            // 4. ADVANCE curr
    }
    return prev;               // prev ends on the last node = the new head
}
```
**Dry run** `1->2->3->NULL`:
```
start: prev=N  curr=1
it1:   nxt=2, 1->N,      prev=1, curr=2      list so far: 1->N
it2:   nxt=3, 2->1,      prev=2, curr=3                   2->1->N
it3:   nxt=N, 3->2,      prev=3, curr=N                   3->2->1->N
curr == NULL -> return prev = 3
```
**O(n) time, O(1) space.** Write this out on paper three times tonight. It is the single most reused piece of code in the entire topic.

### Recursive
```cpp
Node* reverseRec(Node* head) {
    if (!head || !head->next) return head;     // base: empty or single node
    Node* newHead = reverseRec(head->next);    // reverse the rest
    head->next->next = head;                   // make the next node point back to me
    head->next = nullptr;                      // and I point to null
    return newHead;                            // the new head never changes on the way up
}
```
O(n) time, **O(n) space** for the call stack — mention this trade-off if asked.

### Reverse the first k nodes / in groups of k
```cpp
Node* reverseKGroup(Node* head, int k) {
    // 1. check that k nodes remain
    Node* c = head;
    for (int i = 0; i < k; i++) { if (!c) return head; c = c->next; }

    // 2. reverse this block of k
    Node *prev = nullptr, *curr = head, *nxt = nullptr;
    for (int i = 0; i < k; i++) {
        nxt = curr->next;
        curr->next = prev;
        prev = curr;
        curr = nxt;
    }

    // 3. head is now the tail of this block; attach the recursively-processed rest
    head->next = reverseKGroup(curr, k);
    return prev;                               // new head of this block
}
```
The check in step 1 is what distinguishes "reverse in groups of k, leaving a short remainder as-is" from "reverse everything". Read the statement carefully to see which is wanted.

---

## 3. SLOW & FAST POINTERS (Floyd's tortoise and hare) — the second master pattern

`slow` moves 1 step, `fast` moves 2. When `fast` reaches the end, `slow` is at the middle. That single fact solves middle-finding, cycle detection, palindrome checking, and nth-from-end.

### 3.1 Middle of the list
```cpp
Node* middle(Node* head) {
    Node *slow = head, *fast = head;
    while (fast && fast->next) {        // note the order — short-circuit protects fast->next
        slow = slow->next;
        fast = fast->next->next;
    }
    return slow;
}
```
For an even-length list this returns the **second** middle (in `1->2->3->4` it returns 3). To get the **first** middle, use `while (fast->next && fast->next->next)`.
Always confirm which one the question wants — it is a common silent failure.

### 3.2 Detect a cycle
```cpp
bool hasCycle(Node* head) {
    Node *slow = head, *fast = head;
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
        if (slow == fast) return true;      // they can only meet inside a loop
    }
    return false;
}
```
**Why they must meet:** once both pointers are inside the cycle, `fast` gains exactly one position on `slow` per iteration, so the gap shrinks by 1 each step and must eventually hit 0. It can never "jump over" `slow`.

### 3.3 Find where the cycle starts
```cpp
Node* cycleStart(Node* head) {
    Node *slow = head, *fast = head;
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
        if (slow == fast) {                     // meeting point found
            slow = head;                         // reset ONE pointer to head
            while (slow != fast) {               // now both move 1 step
                slow = slow->next;
                fast = fast->next;
            }
            return slow;                         // this is the cycle entry
        }
    }
    return nullptr;
}
```
**The proof (worth being able to state):** let `L` = distance from head to the cycle start, `C` = cycle length, and `k` = distance from the cycle start to the meeting point. When they meet, slow has travelled `L+k` and fast `2(L+k)`; the difference `L+k` must be a whole number of cycles. So `L+k = mC`, hence `L = mC - k` — exactly the distance from the meeting point forward to the cycle start. Therefore two pointers moving one step each, one from the head and one from the meeting point, meet at the entry.

### 3.4 Length of the cycle
```cpp
int cycleLength(Node* head) {
    Node *slow = head, *fast = head;
    while (fast && fast->next) {
        slow = slow->next; fast = fast->next->next;
        if (slow == fast) {
            int len = 1;
            Node* c = slow->next;
            while (c != slow) { c = c->next; len++; }
            return len;
        }
    }
    return 0;
}
```

### 3.5 Remove the Nth node from the end (one pass)
```cpp
Node* removeNthFromEnd(Node* head, int n) {
    Node dummy(0); dummy.next = head;
    Node *fast = &dummy, *slow = &dummy;
    for (int i = 0; i < n; i++) {              // fast gets an n-node head start
        if (!fast->next) return head;          // n larger than the list
        fast = fast->next;
    }
    while (fast->next) { fast = fast->next; slow = slow->next; }
    Node* del = slow->next;
    slow->next = del->next;
    delete del;
    return dummy.next;
}
```
**The gap trick:** keep the two pointers exactly `n` apart. When the front hits the end, the back is at the node *before* the one to delete. The dummy node is what makes "delete the head" work without a special case.

### 3.6 Palindrome linked list — O(n) time, O(1) space
```cpp
bool isPalindrome(Node* head) {
    if (!head || !head->next) return true;

    Node *slow = head, *fast = head;              // 1. find the middle
    while (fast->next && fast->next->next) {
        slow = slow->next;
        fast = fast->next->next;
    }

    Node* second = reverse(slow->next);           // 2. reverse the second half
    Node* first = head;
    Node* copy = second;

    bool ok = true;
    while (second) {                               // 3. compare
        if (first->data != second->data) { ok = false; break; }
        first = first->next;
        second = second->next;
    }

    slow->next = reverse(copy);                    // 4. restore (good practice)
    return ok;
}
```
**This is the best single question in the topic** — it uses find-the-middle, reverse, and two-pointer comparison in one problem. If you can write this from scratch you understand linked lists.

---

## 4. Merging and sorting

### 4.1 Merge two sorted lists
```cpp
Node* mergeTwo(Node* a, Node* b) {
    Node dummy(0);
    Node* tail = &dummy;
    while (a && b) {
        if (a->data <= b->data) { tail->next = a; a = a->next; }
        else                    { tail->next = b; b = b->next; }
        tail = tail->next;
    }
    tail->next = a ? a : b;              // attach whatever remains
    return dummy.next;
}
```
O(n+m) time, O(1) extra space. `<=` (not `<`) keeps the merge **stable**.

### 4.2 Merge sort on a linked list — O(n log n) time, O(log n) stack
```cpp
Node* sortList(Node* head) {
    if (!head || !head->next) return head;

    Node *slow = head, *fast = head->next;     // fast starts ONE ahead ->
    while (fast && fast->next) {                // slow lands on the FIRST middle
        slow = slow->next;
        fast = fast->next->next;
    }
    Node* mid = slow->next;
    slow->next = nullptr;                       // split into two lists

    return mergeTwo(sortList(head), sortList(mid));
}
```
**Why merge sort and not quick sort for linked lists?** Merge sort needs only sequential access, which is all a linked list offers, and the merge step needs no extra array (unlike arrays, where merge sort costs O(n) extra space). Quick sort needs random access for good pivoting and degrades badly. **This is a very common TCR question — remember the answer.**
**The trap:** if `fast` starts at `head` instead of `head->next`, a 2-node list never splits and you recurse forever.

### 4.3 Merge k sorted lists (min-heap)
```cpp
Node* mergeKLists(vector<Node*>& lists) {
    auto cmp = [](Node* a, Node* b) { return a->data > b->data; };   // min-heap
    priority_queue<Node*, vector<Node*>, decltype(cmp)> pq(cmp);

    for (Node* l : lists) if (l) pq.push(l);

    Node dummy(0); Node* tail = &dummy;
    while (!pq.empty()) {
        Node* n = pq.top(); pq.pop();
        tail->next = n; tail = n;
        if (n->next) pq.push(n->next);
    }
    tail->next = nullptr;
    return dummy.next;
}
```
O(N log k) where N is the total node count and k the number of lists.

---

## 5. More canonical problems

### 5.1 Intersection of two linked lists — the elegant O(1)-space trick
```cpp
Node* getIntersection(Node* a, Node* b) {
    if (!a || !b) return nullptr;
    Node *p = a, *q = b;
    while (p != q) {
        p = p ? p->next : b;        // when p finishes list A, restart it on B
        q = q ? q->next : a;        // when q finishes list B, restart it on A
    }
    return p;                        // meeting point, or nullptr if no intersection
}
```
**Why it works:** each pointer travels `lenA + lenB` before the second lap ends, so they arrive at the intersection simultaneously regardless of the different prefix lengths. If there is no intersection, both become `nullptr` at the same moment and the loop exits.

### 5.2 Remove duplicates from a SORTED list
```cpp
Node* removeDuplicatesSorted(Node* head) {
    Node* c = head;
    while (c && c->next) {
        if (c->data == c->next->data) {
            Node* d = c->next;
            c->next = d->next;
            delete d;               // do NOT advance c — the new next may also be a duplicate
        } else c = c->next;
    }
    return head;
}
```

### 5.3 Remove duplicates from an UNSORTED list
```cpp
Node* removeDuplicatesUnsorted(Node* head) {
    unordered_set<int> seen;
    Node dummy(0); dummy.next = head;
    Node* prev = &dummy;
    while (prev->next) {
        if (seen.count(prev->next->data)) {
            Node* d = prev->next;
            prev->next = d->next;
            delete d;
        } else {
            seen.insert(prev->next->data);
            prev = prev->next;
        }
    }
    return dummy.next;
}
```
O(n) time, O(n) space. Without extra space it becomes O(n^2) with two nested pointers — mention the trade-off.

### 5.4 Add two numbers represented as linked lists (digits reversed)
```cpp
Node* addTwoNumbers(Node* a, Node* b) {
    Node dummy(0); Node* tail = &dummy;
    int carry = 0;
    while (a || b || carry) {                  // the `|| carry` handles a final 9+1 case
        int sum = carry;
        if (a) { sum += a->data; a = a->next; }
        if (b) { sum += b->data; b = b->next; }
        carry = sum / 10;
        tail->next = new Node(sum % 10);
        tail = tail->next;
    }
    return dummy.next;
}
```
If the digits are stored **most-significant first**, either reverse both lists first, or push them onto two stacks and build the result backwards.

### 5.5 Rotate a list right by k
```cpp
Node* rotateRight(Node* head, int k) {
    if (!head || !head->next || k == 0) return head;

    int n = 1;
    Node* tail = head;
    while (tail->next) { tail = tail->next; n++; }   // find length and tail

    k %= n;
    if (k == 0) return head;

    tail->next = head;                                // make it circular

    Node* newTail = head;
    for (int i = 0; i < n - k - 1; i++) newTail = newTail->next;

    Node* newHead = newTail->next;
    newTail->next = nullptr;                          // break the circle
    return newHead;
}
```
"Make it circular, walk to the new tail, cut" — a much cleaner mental model than juggling separate pointers.

### 5.6 Reorder list: L0 -> Ln -> L1 -> Ln-1 -> ...
```cpp
void reorderList(Node* head) {
    if (!head || !head->next) return;

    Node *slow = head, *fast = head;                     // 1. split at the middle
    while (fast->next && fast->next->next) { slow = slow->next; fast = fast->next->next; }
    Node* second = reverse(slow->next);                   // 2. reverse the second half
    slow->next = nullptr;

    Node* first = head;                                   // 3. interleave
    while (second) {
        Node* t1 = first->next;
        Node* t2 = second->next;
        first->next = second;
        second->next = t1;
        first = t1;
        second = t2;
    }
}
```
Another three-in-one: middle + reverse + merge.

### 5.7 Segregate even and odd nodes (keeping relative order)
```cpp
Node* oddEvenList(Node* head) {
    if (!head || !head->next) return head;
    Node *odd = head, *even = head->next, *evenHead = even;
    while (even && even->next) {
        odd->next  = even->next;  odd  = odd->next;
        even->next = odd->next;   even = even->next;
    }
    odd->next = evenHead;
    return head;
}
```

### 5.8 Clone a linked list with random pointers — O(1) extra space
```cpp
struct RNode { int data; RNode *next, *random; RNode(int d):data(d),next(nullptr),random(nullptr){} };

RNode* cloneList(RNode* head) {
    if (!head) return nullptr;

    for (RNode* c = head; c; ) {                  // 1. weave copies in: A->A'->B->B'
        RNode* copy = new RNode(c->data);
        copy->next = c->next;
        c->next = copy;
        c = copy->next;
    }
    for (RNode* c = head; c; c = c->next->next)   // 2. set random on the copies
        if (c->random) c->next->random = c->random->next;

    RNode* newHead = head->next;                   // 3. unweave the two lists
    for (RNode* c = head; c; ) {
        RNode* copy = c->next;
        c->next = copy->next;
        if (copy->next) copy->next = copy->next->next;
        c = c->next;
    }
    return newHead;
}
```
**The trick:** interleaving means every original node's copy is at `original->next`, so `random` can be resolved in O(1) without a hash map. The hash-map version is simpler (`map<RNode*,RNode*>`) but costs O(n) space — say which you chose and why.

---

## 6. Doubly Linked List

```cpp
struct DNode {
    int data;
    DNode *prev, *next;
    DNode(int d) : data(d), prev(nullptr), next(nullptr) {}
};

DNode* insertHead(DNode* head, int x) {
    DNode* n = new DNode(x);
    n->next = head;
    if (head) head->prev = n;
    return n;
}

DNode* deleteNode(DNode* head, DNode* del) {
    if (!head || !del) return head;
    if (head == del) head = del->next;
    if (del->next) del->next->prev = del->prev;
    if (del->prev) del->prev->next = del->next;
    delete del;
    return head;
}

DNode* reverseDLL(DNode* head) {
    DNode *curr = head, *temp = nullptr;
    while (curr) {
        temp = curr->prev;                // swap prev and next on every node
        curr->prev = curr->next;
        curr->next = temp;
        curr = curr->prev;                // which is the OLD next
    }
    return temp ? temp->prev : head;      // temp->prev is the last node visited
}
```

**Singly vs doubly (TCR-ready comparison):**

| | Singly | Doubly |
|---|---|---|
| Memory per node | 1 pointer | 2 pointers |
| Traverse backwards | No | Yes |
| Delete a node given only its pointer | O(n) (need the predecessor) | **O(1)** |
| Insert before a given node | O(n) | O(1) |
| Used in | simple lists, stacks, adjacency lists | LRU cache, browser history, undo/redo, deque |

---

## 7. Circular Linked List

```cpp
// Detect circular
bool isCircular(Node* head) {
    if (!head) return true;
    Node* c = head->next;
    while (c && c != head) c = c->next;
    return c == head;
}

// Traverse safely (do-while, or the loop never starts)
void printCircular(Node* head) {
    if (!head) return;
    Node* c = head;
    do { cout << c->data << " "; c = c->next; } while (c != head);
}

// Insert at the end of a circular list
Node* insertEndCircular(Node* head, int x) {
    Node* n = new Node(x);
    if (!head) { n->next = n; return n; }
    Node* c = head;
    while (c->next != head) c = c->next;
    c->next = n;
    n->next = head;
    return head;
}
```
Used for: round-robin CPU scheduling, circular buffers, multiplayer turn order.

---

## 8. Array vs Linked List — expect this in TCR

| | Array | Linked List |
|---|---|---|
| Memory | contiguous | scattered, one node at a time |
| Access element i | **O(1)** | O(n) |
| Insert/delete at head | O(n) | **O(1)** |
| Insert/delete at a known position | O(n) | **O(1)** given the previous node |
| Search (unsorted) | O(n) | O(n) |
| Extra memory | none | one pointer per node |
| Cache performance | **excellent** (locality) | poor (pointer chasing) |
| Size | fixed (or amortised resize) | grows freely |
| Binary search possible | Yes | No (no random access) |

**The nuance that scores marks:** "insertion in a linked list is O(1)" is only true **if you already hold a pointer to the position**. If you must search for it, finding it is O(n) and the insertion itself is O(1). Say it precisely and you get the full mark.

---

## 9. The bug checklist — run through this before submitting any LL question

- [ ] `head == nullptr` (empty list) handled?
- [ ] Single node handled?
- [ ] Two nodes handled? (this is where slow/fast splits go wrong)
- [ ] Every `->next` guarded by a null check, in the right short-circuit order?
- [ ] After deleting, did you set the predecessor's `next` **before** freeing?
- [ ] Did you return the **new** head where the head may change? (Use a dummy node.)
- [ ] In reversal, are you returning `prev` (not `curr`, which is null)?
- [ ] Did you `nullptr`-terminate the tail after splitting a list?
- [ ] No infinite loop — does every path advance a pointer?

---

## 10. The 20-minute drill — do this tonight

Close the file. Write these from memory, on paper:

1. `reverse(head)` — iterative, 6 lines. **Twice.**
2. `middle(head)` — slow/fast, 5 lines.
3. `hasCycle(head)` — 6 lines.
4. `mergeTwo(a, b)` — with the dummy node, 8 lines.
5. `removeNthFromEnd(head, n)` — the gap trick, 8 lines.
6. `isPalindrome(head)` — combines 1, 2 and a comparison.

If #6 comes out correct without help, you are ready for anything they can ask on linked lists.

---

## 11. Self test (cover the answers)

1. Why does the slow/fast loop condition read `while (fast && fast->next)`? -> *`fast->next->next` needs both to exist; `&&` short-circuits so `fast->next` is only evaluated once `fast` is known non-null.*
2. After Floyd's cycle detection, why does resetting one pointer to the head find the cycle entry? -> *Because L = mC - k, the distance from head to the entry equals the distance from the meeting point to the entry.*
3. Why prefer merge sort over quick sort for linked lists? -> *Merge sort needs only sequential access and its merge requires no extra array for lists; quick sort needs random access for pivoting and degrades to O(n^2).*
4. What does a dummy node buy you? -> *It removes every special case where the head itself is inserted, deleted, or changed.*
5. Can you binary search a sorted linked list? -> *Not usefully — finding the middle is O(n), so the total cost stays O(n).*
6. Time and space of iterative vs recursive reversal? -> *Both O(n) time; iterative is O(1) space, recursive is O(n) stack space.*

Next: [05_BINARY_TREES.md](05_BINARY_TREES.md)
