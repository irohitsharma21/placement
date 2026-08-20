# 03 — Stacks & Queues

> **The one idea that unlocks 70% of this topic: the monotonic stack.** If a problem says "next greater", "previous smaller", "span", or "largest rectangle", it is a monotonic stack. Learn that pattern and the rest is bookkeeping.

---

## 1. Fundamentals

| | Stack | Queue |
|---|---|---|
| Order | **LIFO** (Last In First Out) | **FIFO** (First In First Out) |
| Insert | `push` (at top) | `enqueue` / `push` (at rear) |
| Remove | `pop` (from top) | `dequeue` / `pop` (from front) |
| Real use | function call stack, undo, expression eval, DFS, backtracking | scheduling, BFS, printer/CPU queues, buffering |
| STL | `stack<T>` | `queue<T>` |

**All operations are O(1)** for both.

```cpp
stack<int> st;
st.push(5); st.top(); st.pop();       // pop() returns VOID
queue<int> q;
q.push(5); q.front(); q.back(); q.pop();
```
WARNING: `.top()` / `.front()` on an **empty** container is undefined behaviour and usually a crash. Always `if (!st.empty())`.

---

## 2. Implement a stack yourself (array and linked list)

### Array-based
```cpp
class ArrayStack {
    vector<int> a;
    int topIdx;
    int capacity;
public:
    ArrayStack(int cap) : capacity(cap), topIdx(-1) { a.resize(cap); }

    void push(int x) {
        if (topIdx == capacity - 1) { cout << "Stack Overflow\n"; return; }
        a[++topIdx] = x;
    }
    int pop() {
        if (isEmpty()) { cout << "Stack Underflow\n"; return -1; }
        return a[topIdx--];
    }
    int peek() const { return isEmpty() ? -1 : a[topIdx]; }
    bool isEmpty() const { return topIdx == -1; }
    bool isFull()  const { return topIdx == capacity - 1; }
    int size() const { return topIdx + 1; }
};
```

### Linked-list-based (no fixed capacity)
```cpp
class LinkedStack {
    struct Node { int data; Node* next; Node(int d) : data(d), next(nullptr) {} };
    Node* head = nullptr;      // head IS the top -> O(1) push and pop
    int cnt = 0;
public:
    void push(int x) { Node* n = new Node(x); n->next = head; head = n; cnt++; }
    int pop() {
        if (!head) { cout << "Underflow\n"; return -1; }
        Node* t = head; int v = t->data; head = head->next; delete t; cnt--; return v;
    }
    int peek() const { return head ? head->data : -1; }
    bool isEmpty() const { return head == nullptr; }
    int size() const { return cnt; }
    ~LinkedStack() { while (head) { Node* t = head; head = head->next; delete t; } }
};
```
**Array vs linked list trade-off (a TCR favourite):** the array version has better cache locality and no per-node allocation, but a fixed size (unless you resize). The linked version grows freely but costs a pointer per element plus allocation time.

---

## 3. Implement a queue

### Circular queue with an array (the classic — do not use a naive linear queue)
```cpp
class CircularQueue {
    vector<int> a;
    int front, rear, count, capacity;
public:
    CircularQueue(int cap) : a(cap), front(0), rear(-1), count(0), capacity(cap) {}

    bool enqueue(int x) {
        if (isFull()) { cout << "Queue Overflow\n"; return false; }
        rear = (rear + 1) % capacity;          // wrap around
        a[rear] = x;
        count++;
        return true;
    }
    int dequeue() {
        if (isEmpty()) { cout << "Queue Underflow\n"; return -1; }
        int v = a[front];
        front = (front + 1) % capacity;
        count--;
        return v;
    }
    int peek() const { return isEmpty() ? -1 : a[front]; }
    bool isEmpty() const { return count == 0; }
    bool isFull()  const { return count == capacity; }
};
```
**Why circular?** In a linear array queue, after several dequeues the front slots are wasted and the queue reports "full" while empty space exists. The modulo wraps `rear` back around and reuses them.
**Why keep a `count`?** Because `front == rear+1` is ambiguous between full and empty. A counter removes the ambiguity. (The alternative is to sacrifice one slot.)

---

## 4. Stack from Queues, and Queue from Stacks (guaranteed-favourite questions)

### 4.1 Queue using two stacks
```cpp
class QueueViaStacks {
    stack<int> in, out;
public:
    void push(int x) { in.push(x); }

    int pop() {
        if (out.empty())                       // only transfer when out is empty
            while (!in.empty()) { out.push(in.top()); in.pop(); }
        if (out.empty()) return -1;
        int v = out.top(); out.pop(); return v;
    }
    int front() {
        if (out.empty())
            while (!in.empty()) { out.push(in.top()); in.pop(); }
        return out.empty() ? -1 : out.top();
    }
    bool empty() const { return in.empty() && out.empty(); }
};
```
**The reasoning you should be able to state:** transferring reverses the order, converting LIFO into FIFO. Each element is moved between stacks at most once, so although a single `pop` can cost O(n), the **amortised** cost is **O(1)**.
**The bug to avoid:** transferring when `out` is non-empty destroys the ordering. Only transfer when `out` is empty.

### 4.2 Stack using two queues (push-costly version)
```cpp
class StackViaQueues {
    queue<int> q1, q2;
public:
    void push(int x) {
        q2.push(x);                               // new element goes in alone
        while (!q1.empty()) { q2.push(q1.front()); q1.pop(); }  // everything else behind it
        swap(q1, q2);
    }
    int pop()  { if (q1.empty()) return -1; int v = q1.front(); q1.pop(); return v; }
    int top()  { return q1.empty() ? -1 : q1.front(); }
    bool empty() const { return q1.empty(); }
};
```
Push O(n), pop O(1). The alternative design makes push O(1) and pop O(n) by rotating n-1 elements at pop time.

---

## 5. Balanced Parentheses (the single most common stack question)

```cpp
bool isValid(string s) {
    stack<char> st;
    for (char c : s) {
        if (c == '(' || c == '[' || c == '{') st.push(c);
        else {
            if (st.empty()) return false;               // closing with nothing open
            char t = st.top(); st.pop();
            if ((c == ')' && t != '(') ||
                (c == ']' && t != '[') ||
                (c == '}' && t != '{')) return false;   // mismatched type
        }
    }
    return st.empty();                                   // leftovers = unclosed
}
```
**Three failure modes, three checks:** closing when empty, wrong type of match, and leftovers at the end. Miss any one and you fail test cases.

**Variant — minimum insertions/removals to balance:**
```cpp
int minRemovals(string s) {
    int open = 0, removals = 0;
    for (char c : s) {
        if (c == '(') open++;
        else if (open > 0) open--;
        else removals++;              // unmatched ')'
    }
    return removals + open;           // + unmatched '('
}
```

**Variant — longest valid parentheses substring (stack of indices):**
```cpp
int longestValidParentheses(string s) {
    stack<int> st;
    st.push(-1);                                  // sentinel base index
    int best = 0;
    for (int i = 0; i < (int)s.size(); i++) {
        if (s[i] == '(') st.push(i);
        else {
            st.pop();
            if (st.empty()) st.push(i);           // new base
            else best = max(best, i - st.top());
        }
    }
    return best;
}
```

---

## 6. MONOTONIC STACK — the master pattern

**The template.** A stack that stays sorted (increasing or decreasing). While the incoming element violates the order, pop — and *the moment you pop, you have learned something* about the popped element.

```cpp
for (int i = 0; i < n; i++) {
    while (!st.empty() && CONDITION(st.top(), a[i])) {
        // the answer for st.top() is a[i] (or is computed here)
        st.pop();
    }
    st.push(i);
}
```

| You want | Stack keeps | Pop while |
|---|---|---|
| Next **greater** element | decreasing | `a[st.top()] < a[i]` |
| Next **smaller** element | increasing | `a[st.top()] > a[i]` |
| Previous **greater** | decreasing (traverse same direction, answer = new top) | `a[st.top()] <= a[i]` |
| Previous **smaller** | increasing | `a[st.top()] >= a[i]` |

### 6.1 Next Greater Element
```cpp
vector<int> nextGreater(vector<int>& a) {
    int n = a.size();
    vector<int> res(n, -1);
    stack<int> st;                                 // stores INDICES
    for (int i = 0; i < n; i++) {
        while (!st.empty() && a[st.top()] < a[i]) {
            res[st.top()] = a[i];                  // a[i] is the next greater for st.top()
            st.pop();
        }
        st.push(i);
    }
    return res;                                     // anything left in st has no next greater
}
```
Dry run on `[4, 5, 2, 25]`: push 0. i=1: a[0]=4 < 5 -> res[0]=5, pop; push 1. i=2: 5 > 2, push 2. i=3: a[2]=2<25 -> res[2]=25; a[1]=5<25 -> res[1]=25; push 3. Result `[5, 25, 25, -1]`. Correct.
**O(n) time** — each index is pushed once and popped at most once.

### 6.2 Previous Smaller Element
```cpp
vector<int> prevSmaller(vector<int>& a) {
    vector<int> res(a.size(), -1);
    stack<int> st;
    for (int i = 0; i < (int)a.size(); i++) {
        while (!st.empty() && a[st.top()] >= a[i]) st.pop();
        res[i] = st.empty() ? -1 : a[st.top()];    // whatever survives IS the previous smaller
        st.push(i);
    }
    return res;
}
```

### 6.3 Next Greater in a CIRCULAR array
```cpp
vector<int> nextGreaterCircular(vector<int>& a) {
    int n = a.size();
    vector<int> res(n, -1);
    stack<int> st;
    for (int i = 0; i < 2 * n; i++) {              // loop twice
        int idx = i % n;
        while (!st.empty() && a[st.top()] < a[idx]) { res[st.top()] = a[idx]; st.pop(); }
        if (i < n) st.push(idx);                    // only push during the first pass
    }
    return res;
}
```

### 6.4 Stock Span problem
"For each day, how many consecutive days up to today had a price <= today's price?"
```cpp
vector<int> stockSpan(vector<int>& price) {
    int n = price.size();
    vector<int> span(n);
    stack<int> st;
    for (int i = 0; i < n; i++) {
        while (!st.empty() && price[st.top()] <= price[i]) st.pop();
        span[i] = st.empty() ? (i + 1) : (i - st.top());
        st.push(i);
    }
    return span;
}
```
This is literally "previous greater element" reframed as a distance.

### 6.5 Largest Rectangle in a Histogram  (the hardest classic — worth knowing cold)
```cpp
int largestRectangleArea(vector<int>& h) {
    int n = h.size(), best = 0;
    stack<int> st;                                 // increasing stack of indices
    for (int i = 0; i <= n; i++) {
        int cur = (i == n) ? 0 : h[i];             // sentinel 0 flushes the stack at the end
        while (!st.empty() && h[st.top()] >= cur) {
            int height = h[st.top()]; st.pop();
            int width = st.empty() ? i : (i - st.top() - 1);
            best = max(best, height * width);
        }
        st.push(i);
    }
    return best;
}
```
**The idea:** for each bar, the largest rectangle *with that bar as the shortest one* extends left to the previous smaller bar and right to the next smaller bar. The monotonic stack finds both boundaries in one pass.
**The width formula is the whole question.** When you pop, `i` is the next-smaller boundary on the right, and the new `st.top()` is the previous-smaller boundary on the left, so the width is the gap strictly between them: `i - st.top() - 1`. If the stack empties, the bar extended all the way to index 0, so width = `i`.

### 6.6 Trapping Rain Water (two-pointer version — simpler than the stack version)
```cpp
int trap(vector<int>& h) {
    int l = 0, r = h.size() - 1, leftMax = 0, rightMax = 0, water = 0;
    while (l < r) {
        if (h[l] < h[r]) {
            leftMax = max(leftMax, h[l]);
            water += leftMax - h[l];               // safe: rightMax >= h[r] > h[l]
            l++;
        } else {
            rightMax = max(rightMax, h[r]);
            water += rightMax - h[r];
            r--;
        }
    }
    return water;
}
```
**Why it is correct:** water above a bar = `min(maxLeft, maxRight) - height`. When `h[l] < h[r]`, we know the right side has something at least as tall, so `leftMax` is definitely the binding constraint — we can commit without knowing the true `rightMax`.

---

## 7. Min Stack — O(1) getMin()

```cpp
class MinStack {
    stack<int> st;
    stack<int> mins;                     // parallel stack of running minima
public:
    void push(int x) {
        st.push(x);
        if (mins.empty() || x <= mins.top()) mins.push(x);
        else mins.push(mins.top());      // repeat the current min
    }
    void pop() {
        if (st.empty()) return;
        st.pop(); mins.pop();            // always in lockstep
    }
    int top()    { return st.top(); }
    int getMin() { return mins.top(); }
};
```
**Note `x <= mins.top()`, not `<`.** With duplicate minima, a strict `<` loses a copy and `getMin` becomes wrong after a pop. This version pushes on every push so it is safe either way, but the habit matters in the space-optimised variant.

---

## 8. Expression evaluation

### 8.1 Evaluate postfix (Reverse Polish Notation)
```cpp
int evalPostfix(vector<string>& tokens) {
    stack<int> st;
    for (string& t : tokens) {
        if (t == "+" || t == "-" || t == "*" || t == "/") {
            int b = st.top(); st.pop();
            int a = st.top(); st.pop();        // ORDER MATTERS: a is the LEFT operand
            if (t == "+") st.push(a + b);
            if (t == "-") st.push(a - b);
            if (t == "*") st.push(a * b);
            if (t == "/") st.push(a / b);
        } else st.push(stoi(t));
    }
    return st.top();
}
```
The classic bug: assigning the first pop to `a`. The **second** pop is the left operand, because it was pushed earlier.

### 8.2 Infix to Postfix (Shunting-yard)
```cpp
int prec(char c) {
    if (c == '^') return 3;
    if (c == '*' || c == '/') return 2;
    if (c == '+' || c == '-') return 1;
    return 0;
}

string infixToPostfix(string s) {
    stack<char> st;
    string out;
    for (char c : s) {
        if (isalnum(c)) out += c;                       // operand -> straight to output
        else if (c == '(') st.push(c);
        else if (c == ')') {
            while (!st.empty() && st.top() != '(') { out += st.top(); st.pop(); }
            if (!st.empty()) st.pop();                  // discard the '('
        } else {                                         // operator
            while (!st.empty() && prec(st.top()) >= prec(c) && c != '^') {
                out += st.top(); st.pop();
            }
            st.push(c);
        }
    }
    while (!st.empty()) { out += st.top(); st.pop(); }
    return out;
}
// a+b*c        -> abc*+
// (a+b)*c      -> ab+c*
// a+b*c-d/e    -> abc*+de/-
```
`^` is right-associative, which is why it is excluded from the `>=` pop condition.

---

## 9. Deque problems

### 9.1 Sliding Window Maximum (monotonic deque — O(n))
```cpp
vector<int> maxSlidingWindow(vector<int>& a, int k) {
    deque<int> dq;                       // holds INDICES, values decreasing
    vector<int> res;
    for (int i = 0; i < (int)a.size(); i++) {
        if (!dq.empty() && dq.front() <= i - k) dq.pop_front();    // 1. drop out-of-window
        while (!dq.empty() && a[dq.back()] <= a[i]) dq.pop_back(); // 2. drop dominated
        dq.push_back(i);                                            // 3. add current
        if (i >= k - 1) res.push_back(a[dq.front()]);               // 4. front is the max
    }
    return res;
}
```
**Why pop the smaller ones from the back?** If `a[i]` is bigger and arrives later, every earlier smaller element can never be the maximum of any future window — it is dominated in both value and freshness. Discarding them keeps the deque decreasing, so the front is always the answer.
**O(n)** — each index enters and leaves the deque exactly once.

### 9.2 First negative number in every window of size k
Same structure, but the deque holds the indices of negative numbers only, in order.

---

## 10. LRU Cache (hash map + doubly linked list) — favourite "combined" question

```cpp
class LRUCache {
    int cap;
    list<pair<int,int>> dll;                                  // front = most recently used
    unordered_map<int, list<pair<int,int>>::iterator> mp;     // key -> node position
public:
    LRUCache(int c) : cap(c) {}

    int get(int key) {
        if (!mp.count(key)) return -1;
        dll.splice(dll.begin(), dll, mp[key]);                // move node to front, O(1)
        return mp[key]->second;
    }

    void put(int key, int val) {
        if (mp.count(key)) {
            mp[key]->second = val;
            dll.splice(dll.begin(), dll, mp[key]);
            return;
        }
        if ((int)dll.size() == cap) {                          // evict least recently used
            mp.erase(dll.back().first);
            dll.pop_back();
        }
        dll.push_front({key, val});
        mp[key] = dll.begin();
    }
};
```
**Why both structures?** The hash map gives O(1) lookup; the doubly linked list gives O(1) reordering and eviction. Neither alone can do both. "Combine two structures to get both properties" is exactly what this question tests.

---

## 11. Other high-probability questions

### Sort a stack using only recursion
```cpp
void insertSorted(stack<int>& st, int x) {
    if (st.empty() || st.top() <= x) { st.push(x); return; }
    int t = st.top(); st.pop();
    insertSorted(st, x);
    st.push(t);
}
void sortStack(stack<int>& st) {
    if (st.empty()) return;
    int t = st.top(); st.pop();
    sortStack(st);
    insertSorted(st, t);
}
```

### Reverse a stack using recursion (same shape, but insert at the bottom)
```cpp
void insertBottom(stack<int>& st, int x) {
    if (st.empty()) { st.push(x); return; }
    int t = st.top(); st.pop();
    insertBottom(st, x);
    st.push(t);
}
void reverseStack(stack<int>& st) {
    if (st.empty()) return;
    int t = st.top(); st.pop();
    reverseStack(st);
    insertBottom(st, t);
}
```

### Check a palindrome using a stack
```cpp
bool isPalindromeStack(string s) {
    stack<char> st;
    for (char c : s) st.push(c);
    for (char c : s) { if (c != st.top()) return false; st.pop(); }
    return true;
}
```

### First non-repeating character in a stream (queue + frequency array)
```cpp
void firstNonRepeating(string stream) {
    queue<char> q;
    vector<int> freq(256, 0);
    for (char c : stream) {
        freq[(unsigned char)c]++;
        q.push(c);
        while (!q.empty() && freq[(unsigned char)q.front()] > 1) q.pop();
        cout << (q.empty() ? '#' : q.front()) << " ";
    }
}
```

### Celebrity problem (stack elimination, O(n))
```cpp
// knows(a,b) == true if a knows b. A celebrity is known by everyone and knows nobody.
int findCelebrity(int n, vector<vector<int>>& knows) {
    stack<int> st;
    for (int i = 0; i < n; i++) st.push(i);
    while (st.size() > 1) {
        int a = st.top(); st.pop();
        int b = st.top(); st.pop();
        if (knows[a][b]) st.push(b);      // a knows b -> a cannot be the celebrity
        else st.push(a);                   // a does not know b -> b cannot be
    }
    int c = st.top();
    for (int i = 0; i < n; i++)            // verify the candidate
        if (i != c && (knows[c][i] || !knows[i][c])) return -1;
    return c;
}
```
**Elimination insight:** every comparison rules out exactly one person, so n-1 comparisons leave one candidate — which still must be verified.

---

## 12. Complexity summary

| Operation | Stack | Queue | Deque | Priority queue |
|---|---|---|---|---|
| Insert | O(1) | O(1) | O(1) both ends | O(log n) |
| Delete | O(1) | O(1) | O(1) both ends | O(log n) |
| Peek min/max | O(n), or O(1) with a MinStack | O(n) | O(n) | **O(1)** |
| Search | O(n) | O(n) | O(n) | O(n) |

**Monotonic stack / deque problems are O(n)** despite the inner `while` loop — every element is pushed once and popped at most once. Be ready to say exactly that in a TCR reasoning box; "what is the complexity?" questions on this pattern are a classic trap where the nested loop tempts you into answering O(n^2).

---

## 13. Self test (cover the answers)

1. Why is a circular queue better than a linear array queue? -> *A linear queue wastes the slots freed at the front and reports full while space exists; the modulo wraps around and reuses them.*
2. Amortised complexity of pop in a two-stack queue, and why? -> *O(1). Each element is transferred from `in` to `out` at most once over its lifetime.*
3. In the histogram problem, what does `i - st.top() - 1` compute? -> *The width strictly between the previous smaller bar and the next smaller bar.*
4. Why does the sliding-window-maximum deque store indices, not values? -> *So you can test whether the front has fallen outside the window.*
5. Which structures make an LRU cache, and why each? -> *Hash map for O(1) lookup, doubly linked list for O(1) reorder and eviction.*
6. Postfix form of `(a+b)*(c-d)`? -> *ab+cd-* followed by `*`.
7. Where is a stack used implicitly by the language itself? -> *The call stack for function calls; also DFS, recursion, undo, and backtracking.*
8. Stack overflow vs heap overflow? -> *Stack overflow = too-deep recursion or huge local arrays exhausting the call stack. Heap overflow = exhausting dynamic memory via `new`/`malloc`.*

Next: [04_LINKED_LIST.md](04_LINKED_LIST.md)
