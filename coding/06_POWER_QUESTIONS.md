# 06 — Power Questions

> **15 famous problems, each of which teaches several concepts at once.** If you are short on time, this is the file to drill. Each entry states *what it teaches* — because the point is transferable technique, not the individual answer.

For each one: read the problem, look away, and try to state the **approach in one sentence** before you look at the code. That one sentence is what you will need tomorrow, in both sections.

---

## 1. Palindrome Linked List
**Teaches:** slow/fast pointers + in-place reversal + two-pointer comparison — three techniques in one problem.

**One-sentence approach:** Find the middle with slow/fast, reverse the second half in place, then walk both halves together comparing values.

```cpp
bool isPalindrome(Node* head) {
    if (!head || !head->next) return true;
    Node *slow = head, *fast = head;
    while (fast->next && fast->next->next) { slow = slow->next; fast = fast->next->next; }
    Node* second = reverse(slow->next);
    Node* first = head;
    while (second) {
        if (first->data != second->data) return false;
        first = first->next; second = second->next;
    }
    return true;
}
```
**Why it is the best linked-list question:** if you can write this cold, you own the topic. **O(n) time, O(1) space.**
**Follow-up they may ask:** "Could you do it with a stack?" — yes, push the first half and compare, but that costs O(n) space. Being able to compare the two approaches is exactly the kind of reasoning TCR rewards.

---

## 2. Largest Rectangle in Histogram
**Teaches:** the monotonic stack, and the "previous smaller / next smaller boundary" idea that many problems reduce to.

**One-sentence approach:** For each bar, the widest rectangle where that bar is the *shortest* runs from the previous smaller bar to the next smaller bar — a monotonic increasing stack finds both boundaries in a single pass.

```cpp
int largestRectangleArea(vector<int>& h) {
    int n = h.size(), best = 0;
    stack<int> st;
    for (int i = 0; i <= n; i++) {
        int cur = (i == n) ? 0 : h[i];                 // sentinel flushes the stack
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
**Extends to:** "Maximal rectangle in a binary matrix" — run this once per row over a running histogram of consecutive 1s. If you understand this, you understand the extension for free.

---

## 3. LRU Cache
**Teaches:** combining two data structures to get both of their guarantees; hash map + doubly linked list; the design-question mindset.

**One-sentence approach:** A hash map gives O(1) lookup and a doubly linked list gives O(1) reordering/eviction, so store the map's values as *iterators into the list*.

```cpp
class LRUCache {
    int cap;
    list<pair<int,int>> dll;                                  // front = most recent
    unordered_map<int, list<pair<int,int>>::iterator> mp;
public:
    LRUCache(int c) : cap(c) {}
    int get(int key) {
        if (!mp.count(key)) return -1;
        dll.splice(dll.begin(), dll, mp[key]);
        return mp[key]->second;
    }
    void put(int key, int val) {
        if (mp.count(key)) { mp[key]->second = val; dll.splice(dll.begin(), dll, mp[key]); return; }
        if ((int)dll.size() == cap) { mp.erase(dll.back().first); dll.pop_back(); }
        dll.push_front({key, val});
        mp[key] = dll.begin();
    }
};
```
**Why it matters beyond coding:** LRU is also an FSD/system-design answer (cache eviction policies), so this one question spans both sections. Alternatives worth naming: LFU (least frequently used), FIFO, and TTL-based expiry.

---

## 4. Validate a Binary Search Tree
**Teaches:** why local checks are not global checks — the most transferable *reasoning* lesson in the whole tree topic.

**One-sentence approach:** Pass a valid `(min, max)` range down the recursion; every node must lie strictly inside the range its ancestors imply, not merely satisfy its parent.

```cpp
bool validate(TreeNode* r, long minV, long maxV) {
    if (!r) return true;
    if (r->data <= minV || r->data >= maxV) return false;
    return validate(r->left, minV, r->data) && validate(r->right, r->data, maxV);
}
```
**The counterexample to remember:**
```
      5
     / \
    3   7
       / \
      2   8      <- 2 satisfies its parent 7, but violates the root 5
```
**Say this in a TCR box:** *"The BST property is a constraint over entire subtrees, not adjacent nodes, so validation must carry ancestor bounds downward."*

---

## 5. Lowest Common Ancestor (general binary tree)
**Teaches:** thinking of recursive return values as *messages travelling up the tree* — the mental model that unlocks hard tree problems.

**One-sentence approach:** Each call reports "did I find p or q below me?"; the first node that receives a positive report from *both* sides is the LCA.

```cpp
TreeNode* lca(TreeNode* root, TreeNode* p, TreeNode* q) {
    if (!root || root == p || root == q) return root;
    TreeNode* L = lca(root->left, p, q);
    TreeNode* R = lca(root->right, p, q);
    if (L && R) return root;
    return L ? L : R;
}
```
**Once you have LCA, you get for free:** distance between two nodes = `d(p) + d(q) - 2*d(lca)`.

---

## 6. Merge Sort on a Linked List
**Teaches:** divide and conquer, why algorithm choice depends on the data structure, and the split-at-the-middle idiom.

**One-sentence approach:** Split at the middle using slow/fast, sort each half recursively, and merge with a dummy node.

```cpp
Node* sortList(Node* head) {
    if (!head || !head->next) return head;
    Node *slow = head, *fast = head->next;      // fast starts ONE ahead
    while (fast && fast->next) { slow = slow->next; fast = fast->next->next; }
    Node* mid = slow->next;
    slow->next = nullptr;
    return mergeTwo(sortList(head), sortList(mid));
}
```
**The TCR-ready insight:** *"Merge sort suits linked lists because it needs only sequential access and its merge requires no auxiliary array, whereas quick sort depends on random access for pivoting."* This exact question appears in written rounds constantly.

---

## 7. Trapping Rain Water
**Teaches:** the two-pointer method, and how to justify a greedy commitment — genuinely excellent reasoning practice.

**One-sentence approach:** Water above a bar is `min(maxLeft, maxRight) - height`; move whichever pointer is at the shorter wall, because that side's max is provably the binding constraint.

```cpp
int trap(vector<int>& h) {
    int l = 0, r = h.size()-1, leftMax = 0, rightMax = 0, water = 0;
    while (l < r) {
        if (h[l] < h[r]) { leftMax = max(leftMax, h[l]); water += leftMax - h[l]; l++; }
        else             { rightMax = max(rightMax, h[r]); water += rightMax - h[r]; r--; }
    }
    return water;
}
```
**Why the greedy step is safe:** when `h[l] < h[r]`, we know *some* bar on the right is at least `h[r] > h[l]`, so the right max cannot be the limiting factor for position `l`. We may commit using `leftMax` alone without ever computing the true right max.

---

## 8. Subarray Sum Equals K
**Teaches:** prefix sums + hash map, and why a sliding window fails when negatives are allowed.

**One-sentence approach:** `sum(l..r) = pre[r] - pre[l-1]`, so for each `r` count how many earlier prefixes equal `pre[r] - k`.

```cpp
int subarraySum(vector<int>& a, int k) {
    unordered_map<long long,int> freq;
    freq[0] = 1;                       // the empty prefix
    long long sum = 0; int count = 0;
    for (int x : a) {
        sum += x;
        if (freq.count(sum - k)) count += freq[sum - k];
        freq[sum]++;
    }
    return count;
}
```
**The reasoning that earns marks:** *"A sliding window requires the running sum to grow monotonically as the window widens, which fails once negative numbers can shrink it. Prefix sums assume no monotonicity, so they handle negatives."*
**The `freq[0] = 1` line** is what lets a prefix that *itself* equals `k` be counted. Forgetting it is the classic bug.

---

## 9. Kadane Algorithm (Maximum Subarray)
**Teaches:** the essence of dynamic programming in four lines — define the state, find the recurrence.

**One-sentence approach:** Let `cur` be the best subarray sum *ending exactly at i*; either extend the previous one or restart at `a[i]`.

```cpp
int maxSubArray(vector<int>& a) {
    int best = a[0], cur = a[0];
    for (int i = 1; i < (int)a.size(); i++) {
        cur = max(a[i], cur + a[i]);
        best = max(best, cur);
    }
    return best;
}
```
**The DP framing to say out loud:** *"dp[i] = max(a[i], dp[i-1] + a[i]). Since dp[i] depends only on dp[i-1], one variable replaces the array — O(1) space."*
**Edge case:** initialising `best = 0` silently breaks on all-negative input. Initialise from `a[0]`.

---

## 10. Queue using two Stacks
**Teaches:** amortised analysis — the concept most people cannot explain, and therefore a great TCR differentiator.

**One-sentence approach:** Push onto an input stack; when popping, if the output stack is empty, pour everything across (which reverses the order) and pop from there.

```cpp
class QueueViaStacks {
    stack<int> in, out;
public:
    void push(int x) { in.push(x); }
    int pop() {
        if (out.empty()) while (!in.empty()) { out.push(in.top()); in.pop(); }
        if (out.empty()) return -1;
        int v = out.top(); out.pop(); return v;
    }
};
```
**The reasoning:** *"A single pop can cost O(n), but each element is transferred from `in` to `out` at most once in its lifetime, so n operations cost O(n) in total — O(1) amortised."*
**The bug to avoid:** transferring while `out` is non-empty scrambles the order. Only transfer when `out` is empty.

---

## 11. Sliding Window Maximum
**Teaches:** the monotonic deque, and the notion of *dominated* candidates.

**One-sentence approach:** Keep a deque of indices with decreasing values; the front is always the window maximum, because any smaller element that arrived earlier can never win again.

```cpp
vector<int> maxSlidingWindow(vector<int>& a, int k) {
    deque<int> dq; vector<int> res;
    for (int i = 0; i < (int)a.size(); i++) {
        if (!dq.empty() && dq.front() <= i - k) dq.pop_front();
        while (!dq.empty() && a[dq.back()] <= a[i]) dq.pop_back();
        dq.push_back(i);
        if (i >= k - 1) res.push_back(a[dq.front()]);
    }
    return res;
}
```
**The complexity trap:** the nested `while` makes it *look* O(nk), but each index is pushed once and popped once, so it is **O(n)**. Expect exactly this trap as a TCR multiple-choice question.

---

## 12. Longest Substring Without Repeating Characters
**Teaches:** the variable-size sliding window with a hash map, and why stale state causes subtle bugs.

**One-sentence approach:** Expand the right edge; when a repeat appears *inside the current window*, jump the left edge past the previous occurrence.

```cpp
int lengthOfLongestSubstring(string s) {
    unordered_map<char,int> last;
    int l = 0, best = 0;
    for (int r = 0; r < (int)s.size(); r++) {
        if (last.count(s[r]) && last[s[r]] >= l) l = last[s[r]] + 1;
        last[s[r]] = r;
        best = max(best, r - l + 1);
    }
    return best;
}
```
**The subtle part:** the `last[s[r]] >= l` guard. Without it, an occurrence from *before* the window drags `l` backwards and the window becomes invalid. Test with `"abba"` — that input exposes the bug.

---

## 13. Search in a Rotated Sorted Array
**Teaches:** adapting binary search when the invariant is partially broken — the mark of real understanding versus memorisation.

**One-sentence approach:** At least one half around `mid` is always properly sorted; work out which, check whether the target lies inside it, and discard the other half.

```cpp
int searchRotated(vector<int>& a, int t) {
    int lo = 0, hi = a.size() - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (a[mid] == t) return mid;
        if (a[lo] <= a[mid]) {                        // left half sorted
            if (a[lo] <= t && t < a[mid]) hi = mid - 1; else lo = mid + 1;
        } else {                                       // right half sorted
            if (a[mid] < t && t <= a[hi]) lo = mid + 1; else hi = mid - 1;
        }
    }
    return -1;
}
```
**Generalise the lesson:** binary search does not require a sorted array — it requires a **predicate that is monotone** over the search space. That reframing also solves "minimum capacity to ship in D days", "minimum eating speed", and "integer square root".

---

## 14. Diameter of a Binary Tree
**Teaches:** the "return one value to the parent, record another globally" pattern that turns many O(n^2) tree solutions into O(n).

**One-sentence approach:** During the height computation, record `leftHeight + rightHeight` at each node as a candidate diameter.

```cpp
int diameter = 0;
int heightForDiameter(TreeNode* r) {
    if (!r) return 0;
    int lh = heightForDiameter(r->left);
    int rh = heightForDiameter(r->right);
    diameter = max(diameter, lh + rh);      // the best path THROUGH me
    return 1 + max(lh, rh);                  // what my PARENT can use
}
```
**The same pattern also solves:** maximum path sum, balanced-tree checking, largest BST subtree. Recognising the pattern is worth more than the four separate solutions.

---

## 15. Design a Bank Account / Shape Hierarchy (OOPs)
**Teaches:** every OOP pillar at once — encapsulation through validation, inheritance, run-time polymorphism, virtual destructors.

**One-sentence approach:** Keep the data private behind a validating mutator, make the behaviour that varies by subtype `virtual`, and hold everything through base-class pointers.

```cpp
class Account {
protected:
    double balance;
public:
    Account(double b) : balance(b) {}
    virtual bool withdraw(double amt) {                       // the varying behaviour
        if (amt <= 0 || amt > balance) return false;
        balance -= amt; return true;
    }
    virtual ~Account() {}                                     // ESSENTIAL
};

class CurrentAccount : public Account {
    double overdraft;
public:
    CurrentAccount(double b, double od) : Account(b), overdraft(od) {}
    bool withdraw(double amt) override {
        if (amt <= 0 || amt > balance + overdraft) return false;
        balance -= amt; return true;
    }
};
```
**Say this:** *"Client code holds `Account*` and never changes when a new account type is added — that is the Open/Closed Principle, delivered by run-time polymorphism."* One sentence covering SOLID, polymorphism and design, and it works verbatim in a TCR reasoning box.

---

## How to use this file in the last hour before the exam

For each of the 15, spend 60 seconds:
1. Read the title.
2. Say the **one-sentence approach** aloud without looking.
3. Say the **complexity**.
4. Say the **one bug or edge case** that breaks it.

If you can do that for all 15, you have covered the technique behind essentially every question they can reasonably ask.

| # | Problem | Technique | Complexity |
|---|---|---|---|
| 1 | Palindrome linked list | slow/fast + reverse | O(n) time, O(1) space |
| 2 | Largest rectangle in histogram | monotonic stack | O(n) / O(n) |
| 3 | LRU cache | hash map + doubly linked list | O(1) per operation |
| 4 | Validate BST | range recursion | O(n) / O(h) |
| 5 | LCA | recursive messages | O(n) / O(h) |
| 6 | Sort a linked list | merge sort | O(n log n) / O(log n) |
| 7 | Trapping rain water | two pointers | O(n) / O(1) |
| 8 | Subarray sum = K | prefix sum + hash map | O(n) / O(n) |
| 9 | Maximum subarray | Kadane / DP | O(n) / O(1) |
| 10 | Queue from two stacks | amortised analysis | O(1) amortised |
| 11 | Sliding window maximum | monotonic deque | O(n) / O(k) |
| 12 | Longest substring, no repeats | sliding window + map | O(n) / O(k) |
| 13 | Search rotated array | modified binary search | O(log n) / O(1) |
| 14 | Tree diameter | return-vs-record | O(n) / O(h) |
| 15 | Bank account / shapes | OOP pillars | — |

Next: [../tcr/01_SQL.md](../tcr/01_SQL.md)
