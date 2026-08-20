# 00 — C++ Template, I/O Patterns & STL Cheat Sheet

> Memorise the template until you can type it without thinking. In an exam with limited runs, every second spent recalling syntax is a second not spent on the algorithm.

---

## 1. The template — type this first, every time

```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long ll;

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);

    // your code

    return 0;
}
```

**Why each line:**
- `bits/stdc++.h` — one header, every STL container and algorithm. (Works on GCC, which is what these portals use. If it fails to compile, fall back to listing headers individually — see §7.)
- `sync_with_stdio(false)` + `cin.tie(NULL)` — makes `cin`/`cout` fast. Without it, reading 10⁵+ numbers can time out.
- ⚠️ **Once you use these, never mix `scanf`/`printf` with `cin`/`cout`.** Pick one style.

**Safe fallback header set** (if `bits/stdc++.h` is rejected):
```cpp
#include <iostream>
#include <vector>
#include <string>
#include <algorithm>
#include <stack>
#include <queue>
#include <map>
#include <set>
#include <unordered_map>
#include <climits>
#include <cmath>
using namespace std;
```

---

## 2. Input patterns — recognise which one the question wants

### (a) n, then n numbers
```
5
3 1 4 1 5
```
```cpp
int n;
cin >> n;
vector<int> a(n);
for (int i = 0; i < n; i++) cin >> a[i];
```

### (b) Multiple test cases
```
2
3
1 2 3
2
9 9
```
```cpp
int t;
cin >> t;
while (t--) {
    int n; cin >> n;
    vector<int> a(n);
    for (int i = 0; i < n; i++) cin >> a[i];
    solve(a);
}
```

### (c) Read until EOF (no count given)
```cpp
int x;
vector<int> a;
while (cin >> x) a.push_back(x);
```

### (d) A whole line with spaces
```cpp
string line;
getline(cin, line);
```
⚠️ **The classic trap:** after `cin >> n`, the newline is still in the buffer, so the next `getline` reads an empty string. Fix:
```cpp
int n; cin >> n;
cin.ignore();          // eat the leftover newline
string line;
getline(cin, line);
```

### (e) Split a line into tokens
```cpp
string line;
getline(cin, line);
stringstream ss(line);
string word;
vector<string> words;
while (ss >> word) words.push_back(word);
```

### (f) Comma-separated values
```cpp
stringstream ss(line);
string tok;
while (getline(ss, tok, ',')) v.push_back(stoi(tok));
```

### (g) 2D grid / matrix
```cpp
int n, m; cin >> n >> m;
vector<vector<int>> g(n, vector<int>(m));
for (int i = 0; i < n; i++)
    for (int j = 0; j < m; j++)
        cin >> g[i][j];
```

### (h) Linked list / tree input
Assessments usually give you the values and expect **you** to build the structure:
```cpp
// Linked list from n values
int n; cin >> n;
Node *head = NULL, *tail = NULL;
for (int i = 0; i < n; i++) {
    int v; cin >> v;
    Node* nd = new Node(v);
    if (!head) head = tail = nd;
    else { tail->next = nd; tail = nd; }
}
```
```cpp
// Binary tree from level-order with -1 (or 'N') for null
TreeNode* buildTree() {
    int rootVal; cin >> rootVal;
    if (rootVal == -1) return NULL;
    TreeNode* root = new TreeNode(rootVal);
    queue<TreeNode*> q; q.push(root);
    while (!q.empty()) {
        TreeNode* cur = q.front(); q.pop();
        int l, r; cin >> l >> r;
        if (l != -1) { cur->left  = new TreeNode(l); q.push(cur->left); }
        if (r != -1) { cur->right = new TreeNode(r); q.push(cur->right); }
    }
    return root;
}
```

---

## 3. Output — get the format exactly right

```cpp
// space-separated on one line
for (int i = 0; i < n; i++) cout << a[i] << " \n"[i == n-1];
// (clever: prints ' ' normally, '\n' after the last — a string indexed by a bool)

// simple version, safer to remember
for (int i = 0; i < n; i++) cout << a[i] << (i + 1 == n ? "\n" : " ");

// boolean answers — read the statement! Is it "Yes/No", "YES/NO", "true/false", "1/0"?
cout << (ok ? "Yes" : "No") << "\n";

// fixed decimals
cout << fixed << setprecision(2) << ans << "\n";
```

⚠️ **Format mistakes silently fail all test cases even when your logic is perfect.** Copy the expected output format character-for-character from the sample.

---

## 4. STL cheat sheet

### vector — dynamic array
```cpp
vector<int> v;                    // empty
vector<int> v(n);                 // n zeros
vector<int> v(n, 5);              // n fives
vector<int> v = {1, 2, 3};
vector<vector<int>> g(n, vector<int>(m, 0));   // 2D

v.push_back(x);   v.pop_back();
v.size();         v.empty();      v.clear();
v.front();        v.back();
v[i];             v.at(i);        // .at() bounds-checks, [] doesn't
sort(v.begin(), v.end());
sort(v.rbegin(), v.rend());       // descending
reverse(v.begin(), v.end());
v.insert(v.begin() + i, x);       // O(n)
v.erase(v.begin() + i);           // O(n)
```

### string
```cpp
string s = "hello";
s.length();  s.size();  s.empty();
s.substr(pos, len);               // s.substr(1,3) -> "ell"
s.find("ll");                     // index, or string::npos if absent
s + "world";  s += 'x';
s.push_back('a');  s.pop_back();
reverse(s.begin(), s.end());
sort(s.begin(), s.end());
stoi(s);  stoll(s);  to_string(42);
isalpha(c); isdigit(c); isalnum(c); tolower(c); toupper(c);
```

### stack — LIFO
```cpp
stack<int> st;
st.push(x);   st.pop();           // pop() returns void!
st.top();     st.size();  st.empty();
```
⚠️ `st.pop()` does **not** return the value. Always `int x = st.top(); st.pop();`
⚠️ Calling `.top()` on an empty stack is undefined behaviour → crash. Always guard: `if (!st.empty())`.

### queue — FIFO
```cpp
queue<int> q;
q.push(x);    q.pop();
q.front();    q.back();   q.size();  q.empty();
```

### deque — double-ended, O(1) both ends
```cpp
deque<int> dq;
dq.push_back(x);   dq.push_front(x);
dq.pop_back();     dq.pop_front();
dq.front();        dq.back();       dq[i];
```

### priority_queue — heap
```cpp
priority_queue<int> maxh;                                   // max-heap (default)
priority_queue<int, vector<int>, greater<int>> minh;        // min-heap
maxh.push(x);  maxh.top();  maxh.pop();
priority_queue<pair<int,int>> pq;   // sorts by .first, then .second
```

### map / unordered_map
```cpp
map<string,int> m;                 // sorted by key,   O(log n), red-black tree
unordered_map<string,int> um;      // unsorted,        O(1) average, hash table

m["a"] = 1;
m["b"]++;                          // auto-creates with 0 then increments
if (m.count("a")) ...              // membership test
if (m.find("a") != m.end()) ...
m.erase("a");
for (auto &p : m) cout << p.first << " " << p.second << "\n";
for (auto &[k, v] : m) ...         // C++17 structured binding
```
⚠️ **Reading `m["x"]` on a missing key inserts it with value 0.** Use `.count()` to test existence without mutating.

### set / unordered_set
```cpp
set<int> s;                        // sorted, unique, O(log n)
s.insert(x);  s.erase(x);  s.count(x);
*s.begin();                        // smallest
*s.rbegin();                       // largest
multiset<int> ms;                  // allows duplicates
```

### pair
```cpp
pair<int,int> p = {1, 2};
p.first;  p.second;
make_pair(1, 2);
vector<pair<int,int>> vp;
sort(vp.begin(), vp.end());        // sorts by first, ties by second
```

### algorithms you will actually use
```cpp
sort(v.begin(), v.end());
sort(v.begin(), v.end(), [](int a, int b){ return a > b; });   // custom
max_element(v.begin(), v.end());          // returns ITERATOR -> use *
min_element(v.begin(), v.end());
accumulate(v.begin(), v.end(), 0LL);      // sum; use 0LL to avoid overflow
count(v.begin(), v.end(), x);
find(v.begin(), v.end(), x);
binary_search(v.begin(), v.end(), x);     // requires sorted, returns bool
lower_bound(v.begin(), v.end(), x);       // first >= x
upper_bound(v.begin(), v.end(), x);       // first  > x
unique(v.begin(), v.end());               // after sort: v.erase(unique(...), v.end())
next_permutation(v.begin(), v.end());
swap(a, b);
__gcd(a, b);
abs(x);  max(a,b);  min(a,b);  pow(a,b);  sqrt(x);
```

### Constants
```cpp
INT_MAX   INT_MIN     // 2147483647 / -2147483648
LLONG_MAX LLONG_MIN
const int INF = 1e9;
const ll  LINF = 1e18;
```

---

## 5. Custom comparators — the syntax people forget

```cpp
// Sort pairs by second value descending
sort(v.begin(), v.end(), [](pair<int,int> a, pair<int,int> b) {
    return a.second > b.second;
});

// Sort structs
struct Emp { string name; int sal; };
sort(v.begin(), v.end(), [](const Emp& a, const Emp& b) {
    if (a.sal != b.sal) return a.sal > b.sal;   // salary desc
    return a.name < b.name;                      // tie -> name asc
});

// Min-heap of pairs by first
priority_queue<pair<int,int>, vector<pair<int,int>>, greater<pair<int,int>>> pq;
```
**Rule for the comparator:** return `true` when `a` must come **before** `b`.

---

## 6. Complexity — what fits in the time limit

Typical limit is 1 second ≈ **10⁸ simple operations**.

| n | Acceptable complexity |
|---|---|
| n ≤ 10 | O(n!) |
| n ≤ 20 | O(2ⁿ) |
| n ≤ 500 | O(n³) |
| n ≤ 5,000 | O(n²) |
| n ≤ 10⁶ | O(n log n) |
| n ≤ 10⁸ | O(n) |

**Read the constraints — they tell you the intended algorithm.** n ≤ 10⁵ means "don't write O(n²)". n ≤ 1000 means "O(n²) is fine, don't over-engineer."

---

## 7. Compile errors — instant fixes

| Error | Cause | Fix |
|---|---|---|
| `'vector' was not declared` | missing include / namespace | add `#include <bits/stdc++.h>` and `using namespace std;` |
| `expected ';' before '}'` | missing semicolon | note: **a class/struct definition needs a `;` after the closing brace** |
| `no matching function for call to ...` | wrong argument types | check you passed `v.begin()` not `v` |
| `invalid use of incomplete type` | using a class before defining it | move the definition above, or forward-declare |
| `reference to 'count' is ambiguous` | your variable clashes with `std::count` | rename your variable (`cnt`) — same for `size`, `distance`, `data`, `next`, `left`, `right`, `time`, `hash` |
| `'x' was not declared in this scope` | declared inside an inner block | move the declaration out |
| `cannot convert 'Node*' to 'int'` | comparing pointer with number | you meant `node->data`, not `node` |
| segfault / no output | null deref or out-of-bounds | check every `->` is guarded and every index is `< size()` |

**Silent wrong-answer causes (worse than compile errors):**
- Integer overflow — `int * int` overflows even when stored into a `long long`. Cast first: `(ll)a * b`.
- Uninitialised variable — `int sum;` in C++ is **garbage**, not 0. Always `int sum = 0;`
- Integer division — `5/2 == 2`. Use `5.0/2` for a real result.
- Modifying a container while iterating over it.
- Comparing floats with `==`. Use `fabs(a-b) < 1e-9`.

---

## 8. Debugging without runs

You have limited runs, so build the habit of **static debugging**:

1. **Read your code out loud** as if explaining to someone. Bugs surface when verbalised.
2. **Check every loop boundary.** `for (i = 0; i < n; i++)` vs `i <= n`.
3. **Check every pointer dereference** has a null guard before it.
4. **Trace the sample input on paper.** Write the variable values at each step in a table.
5. **Check the base case** of every recursion. Does it terminate?
6. **Confirm you print** in the required format.

If you *must* burn a run to debug, add temporary prints that show internal state — get maximum information from that single run:
```cpp
cerr << "after loop: i=" << i << " sum=" << sum << "\n";   // cerr, so it won't pollute stdout
```
Remove them (or leave `cerr` — it doesn't affect judged output) before the final submit.

---

## 9. Your paper checklist, before every Run

```
[ ] Compiles in my head — every brace, semicolon, and class ends with ;
[ ] Every variable initialised
[ ] Every loop bound checked (< vs <=)
[ ] Every pointer null-checked before ->
[ ] long long where sums/products can be large
[ ] Base case in every recursion
[ ] Output format matches sample EXACTLY
[ ] Dry-ran the sample input on paper and got the sample output
```

Next: [01_OOPS_CODING.md](01_OOPS_CODING.md)
