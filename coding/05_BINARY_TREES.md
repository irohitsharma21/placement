# 05 — Binary Trees & BSTs

> **The whole topic is one idea: recursion with a clear "what do I ask my children, and what do I return to my parent?" contract.** Get that contract right and every tree problem is 6 lines.

---

## 0. Setup

```cpp
struct TreeNode {
    int data;
    TreeNode *left, *right;
    TreeNode(int d) : data(d), left(nullptr), right(nullptr) {}
};
```

### Building a tree from level-order input (with -1 for null)
```cpp
TreeNode* buildTree() {
    int rootVal;
    cin >> rootVal;
    if (rootVal == -1) return nullptr;

    TreeNode* root = new TreeNode(rootVal);
    queue<TreeNode*> q;
    q.push(root);

    while (!q.empty()) {
        TreeNode* cur = q.front(); q.pop();
        int l, r;
        cin >> l >> r;
        if (l != -1) { cur->left  = new TreeNode(l); q.push(cur->left);  }
        if (r != -1) { cur->right = new TreeNode(r); q.push(cur->right); }
    }
    return root;
}
```

### The recursion contract — use this template for EVERY tree problem
```
TYPE solve(node):
    if (node == nullptr) return BASE_VALUE;      // what does an empty tree mean here?
    L = solve(node->left);                        // trust the recursion
    R = solve(node->right);
    return COMBINE(L, R, node->data);             // how do I use my children's answers?
```
Ask yourself only two questions and you never get stuck:
1. **What is the answer for an empty tree?** (that is your base case)
2. **Given my children's answers, how do I compute mine?** (that is your combine step)

### Terminology (TCR asks these)
- **Height of a node** = edges on the longest path down to a leaf. Height of a leaf = 0. (Some texts count nodes, making it 1 — say which convention you are using.)
- **Depth of a node** = edges from the root down to it. Root depth = 0.
- **Full binary tree**: every node has 0 or 2 children.
- **Complete binary tree**: all levels filled except possibly the last, which fills left to right. (This is what a heap is.)
- **Perfect binary tree**: all internal nodes have 2 children and all leaves are at the same level. Has exactly `2^h+1 - 1` nodes.
- **Balanced binary tree**: the height difference between the left and right subtree of every node is at most 1.
- **Degenerate/skewed tree**: every node has one child — effectively a linked list, so operations degrade to O(n).
- A binary tree with n nodes has **n-1 edges** and a minimum height of `floor(log2 n)`.

---

## 1. TRAVERSALS — the absolute foundation

```
        1
       / \
      2   3
     / \
    4   5

Preorder   (Root, Left, Right):  1 2 4 5 3
Inorder    (Left, Root, Right):  4 2 5 1 3
Postorder  (Left, Right, Root):  4 5 2 3 1
Level order (BFS):               1 2 3 4 5
```
**Memory hook:** the name tells you where the **Root** goes. **Pre**order = root first. **In**order = root in the middle. **Post**order = root last. Left always precedes right.

### Recursive (write these in 30 seconds)
```cpp
void preorder(TreeNode* r, vector<int>& out) {
    if (!r) return;
    out.push_back(r->data);
    preorder(r->left, out);
    preorder(r->right, out);
}
void inorder(TreeNode* r, vector<int>& out) {
    if (!r) return;
    inorder(r->left, out);
    out.push_back(r->data);
    inorder(r->right, out);
}
void postorder(TreeNode* r, vector<int>& out) {
    if (!r) return;
    postorder(r->left, out);
    postorder(r->right, out);
    out.push_back(r->data);
}
```

### Iterative preorder (stack)
```cpp
vector<int> preorderIter(TreeNode* root) {
    vector<int> out;
    if (!root) return out;
    stack<TreeNode*> st;
    st.push(root);
    while (!st.empty()) {
        TreeNode* n = st.top(); st.pop();
        out.push_back(n->data);
        if (n->right) st.push(n->right);      // push RIGHT first...
        if (n->left)  st.push(n->left);       // ...so LEFT pops first
    }
    return out;
}
```

### Iterative inorder (stack) — the one worth memorising
```cpp
vector<int> inorderIter(TreeNode* root) {
    vector<int> out;
    stack<TreeNode*> st;
    TreeNode* cur = root;
    while (cur || !st.empty()) {
        while (cur) { st.push(cur); cur = cur->left; }   // go as far left as possible
        cur = st.top(); st.pop();
        out.push_back(cur->data);                         // visit
        cur = cur->right;                                 // then go right
    }
    return out;
}
```

### Iterative postorder (two stacks — easiest to remember)
```cpp
vector<int> postorderIter(TreeNode* root) {
    vector<int> out;
    if (!root) return out;
    stack<TreeNode*> s1, s2;
    s1.push(root);
    while (!s1.empty()) {
        TreeNode* n = s1.top(); s1.pop();
        s2.push(n);
        if (n->left)  s1.push(n->left);
        if (n->right) s1.push(n->right);
    }
    while (!s2.empty()) { out.push_back(s2.top()->data); s2.pop(); }
    return out;
}
```
This produces Root-Right-Left in `s2`, which reversed is Left-Right-Root. Nice reasoning to state.

### Level order (BFS with a queue) — level by level
```cpp
vector<vector<int>> levelOrder(TreeNode* root) {
    vector<vector<int>> res;
    if (!root) return res;
    queue<TreeNode*> q;
    q.push(root);
    while (!q.empty()) {
        int sz = q.size();                    // CAPTURE the size = this level's node count
        vector<int> level;
        for (int i = 0; i < sz; i++) {
            TreeNode* n = q.front(); q.pop();
            level.push_back(n->data);
            if (n->left)  q.push(n->left);
            if (n->right) q.push(n->right);
        }
        res.push_back(level);
    }
    return res;
}
```
**`int sz = q.size()` before the inner loop is the entire trick** — it freezes the level boundary before you start adding the next level.

### Zigzag / spiral level order
```cpp
vector<vector<int>> zigzag(TreeNode* root) {
    vector<vector<int>> res;
    if (!root) return res;
    queue<TreeNode*> q; q.push(root);
    bool leftToRight = true;
    while (!q.empty()) {
        int sz = q.size();
        vector<int> level(sz);
        for (int i = 0; i < sz; i++) {
            TreeNode* n = q.front(); q.pop();
            int idx = leftToRight ? i : sz - 1 - i;    // fill from the correct end
            level[idx] = n->data;
            if (n->left)  q.push(n->left);
            if (n->right) q.push(n->right);
        }
        leftToRight = !leftToRight;
        res.push_back(level);
    }
    return res;
}
```

---

## 2. Basic properties — all the same 4-line recursion

```cpp
int height(TreeNode* r) {                       // in edges; use 0 for null
    if (!r) return 0;
    return 1 + max(height(r->left), height(r->right));
}

int countNodes(TreeNode* r) {
    if (!r) return 0;
    return 1 + countNodes(r->left) + countNodes(r->right);
}

int countLeaves(TreeNode* r) {
    if (!r) return 0;
    if (!r->left && !r->right) return 1;
    return countLeaves(r->left) + countLeaves(r->right);
}

int sumNodes(TreeNode* r) {
    if (!r) return 0;
    return r->data + sumNodes(r->left) + sumNodes(r->right);
}

int maxValue(TreeNode* r) {
    if (!r) return INT_MIN;
    return max(r->data, max(maxValue(r->left), maxValue(r->right)));
}

bool identical(TreeNode* a, TreeNode* b) {
    if (!a && !b) return true;
    if (!a || !b) return false;                  // one null, one not
    return a->data == b->data && identical(a->left, b->left)
                              && identical(a->right, b->right);
}

TreeNode* mirror(TreeNode* r) {                  // invert the tree
    if (!r) return nullptr;
    swap(r->left, r->right);
    mirror(r->left);
    mirror(r->right);
    return r;
}

bool isSymmetric(TreeNode* a, TreeNode* b) {     // call with (root->left, root->right)
    if (!a && !b) return true;
    if (!a || !b) return false;
    return a->data == b->data && isSymmetric(a->left, b->right)
                              && isSymmetric(a->right, b->left);   // note the CROSS
}
```
Notice how every one of these is the same shape. **That is the point.** Learn the shape, not the eight functions.

---

## 3. Diameter — and the O(n) trick that generalises

The diameter is the longest path between any two nodes (it need not pass through the root).

**Naive O(n^2):**
```cpp
int diameterSlow(TreeNode* r) {
    if (!r) return 0;
    int through = height(r->left) + height(r->right);
    return max({through, diameterSlow(r->left), diameterSlow(r->right)});
}
```

**Optimal O(n) — compute the height and update a global answer in the same pass:**
```cpp
int diameter = 0;

int heightForDiameter(TreeNode* r) {
    if (!r) return 0;
    int lh = heightForDiameter(r->left);
    int rh = heightForDiameter(r->right);
    diameter = max(diameter, lh + rh);          // best path THROUGH this node
    return 1 + max(lh, rh);                      // what my PARENT needs from me
}
```
**This "return one thing, update another" pattern is the single most useful trick in tree problems.** The same shape solves: balanced check, maximum path sum, largest BST subtree.

### Check if balanced — same trick, using -1 as a failure signal
```cpp
int checkBalance(TreeNode* r) {
    if (!r) return 0;
    int lh = checkBalance(r->left);   if (lh == -1) return -1;
    int rh = checkBalance(r->right);  if (rh == -1) return -1;
    if (abs(lh - rh) > 1) return -1;             // -1 means "unbalanced somewhere below"
    return 1 + max(lh, rh);
}
bool isBalanced(TreeNode* r) { return checkBalance(r) != -1; }
```

### Maximum path sum (path may start and end anywhere)
```cpp
int maxSum = INT_MIN;

int maxGain(TreeNode* r) {
    if (!r) return 0;
    int l = max(0, maxGain(r->left));            // clamp at 0: a negative branch is skipped
    int rr = max(0, maxGain(r->right));
    maxSum = max(maxSum, r->data + l + rr);      // path THROUGH this node
    return r->data + max(l, rr);                  // a path going UP can use only ONE side
}
```
**The key reasoning:** a path that continues to the parent can only descend through one child, so you *return* `max(l, r)` but *record* `l + r`. Be ready to say that sentence — it is the whole question.

---

## 4. Views of a binary tree

```cpp
// LEFT view: first node of each level
void leftView(TreeNode* root) {
    if (!root) return;
    queue<TreeNode*> q; q.push(root);
    while (!q.empty()) {
        int sz = q.size();
        for (int i = 0; i < sz; i++) {
            TreeNode* n = q.front(); q.pop();
            if (i == 0) cout << n->data << " ";          // i == sz-1 gives the RIGHT view
            if (n->left)  q.push(n->left);
            if (n->right) q.push(n->right);
        }
    }
}
```

```cpp
// TOP view: first node seen at each horizontal distance
void topView(TreeNode* root) {
    if (!root) return;
    map<int,int> m;                                    // hd -> value (map keeps hd sorted)
    queue<pair<TreeNode*,int>> q;
    q.push({root, 0});
    while (!q.empty()) {
        auto [n, hd] = q.front(); q.pop();
        if (!m.count(hd)) m[hd] = n->data;             // only the FIRST at this hd
        if (n->left)  q.push({n->left,  hd - 1});
        if (n->right) q.push({n->right, hd + 1});
    }
    for (auto& p : m) cout << p.second << " ";
}

// BOTTOM view: identical, but overwrite every time -> m[hd] = n->data;  (last wins)
```
**Why BFS and not DFS for views?** BFS guarantees you meet nodes in increasing depth order, so "first seen at this horizontal distance" really is the topmost. A DFS could reach a deeper node first and give a wrong top view.

```cpp
// VERTICAL order traversal: group by horizontal distance, ties broken by level then value
vector<vector<int>> verticalOrder(TreeNode* root) {
    map<int, map<int, multiset<int>>> m;                // hd -> level -> values
    queue<pair<TreeNode*, pair<int,int>>> q;            // node, (hd, level)
    if (root) q.push({root, {0, 0}});
    while (!q.empty()) {
        auto [n, p] = q.front(); q.pop();
        m[p.first][p.second].insert(n->data);
        if (n->left)  q.push({n->left,  {p.first - 1, p.second + 1}});
        if (n->right) q.push({n->right, {p.first + 1, p.second + 1}});
    }
    vector<vector<int>> res;
    for (auto& col : m) {
        vector<int> v;
        for (auto& lvl : col.second) for (int x : lvl.second) v.push_back(x);
        res.push_back(v);
    }
    return res;
}
```

```cpp
// BOUNDARY traversal = left boundary (top-down) + leaves (left to right) + right boundary (bottom-up)
void leftBoundary(TreeNode* r, vector<int>& out) {
    if (!r || (!r->left && !r->right)) return;
    out.push_back(r->data);
    if (r->left) leftBoundary(r->left, out);
    else leftBoundary(r->right, out);
}
void leaves(TreeNode* r, vector<int>& out) {
    if (!r) return;
    if (!r->left && !r->right) { out.push_back(r->data); return; }
    leaves(r->left, out);
    leaves(r->right, out);
}
void rightBoundary(TreeNode* r, vector<int>& out) {
    if (!r || (!r->left && !r->right)) return;
    if (r->right) rightBoundary(r->right, out);
    else rightBoundary(r->left, out);
    out.push_back(r->data);                     // push AFTER recursing -> bottom-up
}
vector<int> boundary(TreeNode* root) {
    vector<int> out;
    if (!root) return out;
    out.push_back(root->data);
    leftBoundary(root->left, out);
    leaves(root, out);
    rightBoundary(root->right, out);
    return out;
}
```

---

## 5. Lowest Common Ancestor (LCA)

### In a general binary tree — O(n)
```cpp
TreeNode* lca(TreeNode* root, TreeNode* p, TreeNode* q) {
    if (!root || root == p || root == q) return root;
    TreeNode* L = lca(root->left,  p, q);
    TreeNode* R = lca(root->right, p, q);
    if (L && R) return root;          // p and q found on OPPOSITE sides -> this is the LCA
    return L ? L : R;                  // both on one side -> pass that side's answer up
}
```
**Read the return values as messages:** `nullptr` = "neither node is below me", a node = "I found this one (or the LCA) below me". Only when both children report a find is the current node the meeting point.

### In a BST — O(h), simpler
```cpp
TreeNode* lcaBST(TreeNode* root, TreeNode* p, TreeNode* q) {
    while (root) {
        if (p->data < root->data && q->data < root->data)      root = root->left;
        else if (p->data > root->data && q->data > root->data) root = root->right;
        else return root;              // the split point IS the LCA
    }
    return nullptr;
}
```

### Path from root to a node
```cpp
bool findPath(TreeNode* r, int target, vector<int>& path) {
    if (!r) return false;
    path.push_back(r->data);
    if (r->data == target) return true;
    if (findPath(r->left, target, path) || findPath(r->right, target, path)) return true;
    path.pop_back();                   // BACKTRACK — this node is not on the path
    return false;
}
```
That `path.pop_back()` is the essence of backtracking. Distance between two nodes = `depth(p) + depth(q) - 2 * depth(lca)`.

---

## 6. Root-to-leaf path problems

```cpp
bool hasPathSum(TreeNode* r, int target) {
    if (!r) return false;
    if (!r->left && !r->right) return target == r->data;      // leaf check
    return hasPathSum(r->left,  target - r->data) ||
           hasPathSum(r->right, target - r->data);
}

void allPaths(TreeNode* r, vector<int>& cur, vector<vector<int>>& res) {
    if (!r) return;
    cur.push_back(r->data);
    if (!r->left && !r->right) res.push_back(cur);
    else { allPaths(r->left, cur, res); allPaths(r->right, cur, res); }
    cur.pop_back();                                            // backtrack
}
```
WARNING: `if (!r) return target == 0;` is **wrong** for hasPathSum — a single-child node would let a null branch falsely succeed. Always test at the **leaf**, not at null.

---

## 7. BINARY SEARCH TREES

**The BST property:** for every node, all values in the left subtree are **smaller** and all values in the right subtree are **larger**. This holds for the *entire* subtree, not just the immediate children — that is the source of the most common bug.

**The single most important consequence: the inorder traversal of a BST is sorted (ascending).** Half of all BST questions are solved by remembering this one line.

### Search, insert, delete
```cpp
TreeNode* searchBST(TreeNode* r, int key) {
    while (r && r->data != key)
        r = (key < r->data) ? r->left : r->right;
    return r;                                     // O(h): O(log n) balanced, O(n) skewed
}

TreeNode* insertBST(TreeNode* r, int key) {
    if (!r) return new TreeNode(key);
    if (key < r->data)      r->left  = insertBST(r->left,  key);
    else if (key > r->data) r->right = insertBST(r->right, key);
    return r;                                     // duplicates ignored
}

TreeNode* minNode(TreeNode* r) { while (r->left) r = r->left; return r; }

TreeNode* deleteBST(TreeNode* r, int key) {
    if (!r) return nullptr;
    if (key < r->data)      r->left  = deleteBST(r->left,  key);
    else if (key > r->data) r->right = deleteBST(r->right, key);
    else {
        // CASE 1: no child, or CASE 2: one child
        if (!r->left)  { TreeNode* t = r->right; delete r; return t; }
        if (!r->right) { TreeNode* t = r->left;  delete r; return t; }
        // CASE 3: two children -> replace with the INORDER SUCCESSOR
        TreeNode* succ = minNode(r->right);       // smallest in the right subtree
        r->data = succ->data;
        r->right = deleteBST(r->right, succ->data);
    }
    return r;
}
```
**Deletion has exactly three cases — say them out loud:** leaf (just remove), one child (splice the child in), two children (copy the inorder successor's value here, then delete the successor from the right subtree). The successor works because it is the smallest value still larger than everything on the left — exactly what preserves the BST property. (The inorder *predecessor*, the largest in the left subtree, works equally well.)

### Validate a BST — the classic trap
```cpp
// WRONG: only checks immediate children
bool wrongValidate(TreeNode* r) {
    if (!r) return true;
    if (r->left  && r->left->data  >= r->data) return false;
    if (r->right && r->right->data <= r->data) return false;
    return wrongValidate(r->left) && wrongValidate(r->right);
}
//        5
//       / \
//      3   7
//         / \
//        2   8      <- 2 < 5 but sits in the RIGHT subtree. The check above passes it.

// CORRECT: carry down a valid range
bool validate(TreeNode* r, long minV, long maxV) {
    if (!r) return true;
    if (r->data <= minV || r->data >= maxV) return false;
    return validate(r->left,  minV, r->data) &&
           validate(r->right, r->data, maxV);
}
bool isValidBST(TreeNode* r) { return validate(r, LONG_MIN, LONG_MAX); }

// ALTERNATIVE: inorder must be strictly increasing
TreeNode* prevNode = nullptr;
bool isValidInorder(TreeNode* r) {
    if (!r) return true;
    if (!isValidInorder(r->left)) return false;
    if (prevNode && prevNode->data >= r->data) return false;
    prevNode = r;
    return isValidInorder(r->right);
}
```
**Use `long` for the bounds**, or a node holding `INT_MAX` fails against an `INT_MAX` bound.

### Kth smallest / largest in a BST
```cpp
int cnt = 0, ans = -1;
void kthSmallest(TreeNode* r, int k) {
    if (!r || cnt >= k) return;
    kthSmallest(r->left, k);
    if (++cnt == k) { ans = r->data; return; }
    kthSmallest(r->right, k);
}
// kth LARGEST: do a reverse inorder (right, node, left) with the same counter.
```
Straight from "inorder of a BST is sorted".

### Inorder successor
```cpp
TreeNode* inorderSuccessor(TreeNode* root, TreeNode* p) {
    TreeNode* succ = nullptr;
    while (root) {
        if (p->data < root->data) { succ = root; root = root->left; }   // candidate found
        else root = root->right;
    }
    return succ;
}
```

### Build a balanced BST from a sorted array
```cpp
TreeNode* sortedArrayToBST(vector<int>& a, int lo, int hi) {
    if (lo > hi) return nullptr;
    int mid = lo + (hi - lo) / 2;
    TreeNode* root = new TreeNode(a[mid]);        // the middle becomes the root
    root->left  = sortedArrayToBST(a, lo, mid - 1);
    root->right = sortedArrayToBST(a, mid + 1, hi);
    return root;
}
```
Picking the middle as the root is what guarantees balance — same principle as binary search.

---

## 8. Construction & serialisation

### Build a tree from inorder + preorder
```cpp
unordered_map<int,int> idx;                    // value -> index in inorder, for O(1) lookup
int preIdx = 0;

TreeNode* build(vector<int>& pre, int inLo, int inHi) {
    if (inLo > inHi) return nullptr;
    int rootVal = pre[preIdx++];                // preorder gives the root first
    TreeNode* root = new TreeNode(rootVal);
    int mid = idx[rootVal];                     // its position splits inorder
    root->left  = build(pre, inLo, mid - 1);    // LEFT first — preIdx order matters
    root->right = build(pre, mid + 1, inHi);
    return root;
}
```
**Why you need two traversals:** preorder (or postorder) identifies the root; inorder tells you how the remaining nodes split into left and right subtrees. **Preorder + postorder alone is not enough** to reconstruct a general binary tree — a common TCR question. (For postorder + inorder, read postorder backwards and build the **right** subtree first.)

### Serialise and deserialise
```cpp
void serialize(TreeNode* r, string& s) {
    if (!r) { s += "# "; return; }              // marker for null
    s += to_string(r->data) + " ";
    serialize(r->left, s);
    serialize(r->right, s);
}

TreeNode* deserialize(stringstream& ss) {
    string tok;
    if (!(ss >> tok) || tok == "#") return nullptr;
    TreeNode* r = new TreeNode(stoi(tok));
    r->left  = deserialize(ss);
    r->right = deserialize(ss);
    return r;
}
```
The null markers are what make the preorder string unambiguous — without them the structure cannot be recovered.

---

## 9. Other high-probability problems

### Flatten a binary tree into a linked list (in preorder, using right pointers)
```cpp
void flatten(TreeNode* root) {
    TreeNode* cur = root;
    while (cur) {
        if (cur->left) {
            TreeNode* pred = cur->left;              // rightmost node of the left subtree
            while (pred->right) pred = pred->right;
            pred->right = cur->right;                 // graft the right subtree onto it
            cur->right = cur->left;
            cur->left = nullptr;
        }
        cur = cur->right;
    }
}
```
O(1) space — this is the **Morris traversal** idea (rewire threads instead of using a stack).

### Convert a BST to a sorted doubly linked list (in place)
```cpp
TreeNode* prevN = nullptr, *headN = nullptr;
void bstToDLL(TreeNode* r) {
    if (!r) return;
    bstToDLL(r->left);
    if (!prevN) headN = r;
    else { prevN->right = r; r->left = prevN; }      // left = prev, right = next
    prevN = r;
    bstToDLL(r->right);
}
```
Again: inorder of a BST is sorted, so an inorder walk that links each node to the previous one produces a sorted list.

### Children Sum Property
```cpp
bool childrenSum(TreeNode* r) {
    if (!r || (!r->left && !r->right)) return true;
    int sum = (r->left ? r->left->data : 0) + (r->right ? r->right->data : 0);
    return r->data == sum && childrenSum(r->left) && childrenSum(r->right);
}
```

### Print all ancestors of a node
```cpp
bool printAncestors(TreeNode* r, int target) {
    if (!r) return false;
    if (r->data == target) return true;
    if (printAncestors(r->left, target) || printAncestors(r->right, target)) {
        cout << r->data << " ";                       // printed on the way back UP
        return true;
    }
    return false;
}
```

### Maximum width of a binary tree
```cpp
int maxWidth(TreeNode* root) {
    if (!root) return 0;
    int best = 0;
    queue<pair<TreeNode*, long long>> q;              // node, index in a virtual full tree
    q.push({root, 0});
    while (!q.empty()) {
        int sz = q.size();
        long long first = q.front().second, last = first;
        for (int i = 0; i < sz; i++) {
            auto [n, id] = q.front(); q.pop();
            long long norm = id - first;              // normalise to prevent overflow
            last = id;
            if (n->left)  q.push({n->left,  2*norm + 1});
            if (n->right) q.push({n->right, 2*norm + 2});
        }
        best = max(best, (int)(last - first + 1));
    }
    return best;
}
```

### Nodes at distance K from a target (parent map + BFS)
```cpp
void markParents(TreeNode* r, unordered_map<TreeNode*, TreeNode*>& par) {
    queue<TreeNode*> q; q.push(r);
    while (!q.empty()) {
        TreeNode* n = q.front(); q.pop();
        if (n->left)  { par[n->left]  = n; q.push(n->left);  }
        if (n->right) { par[n->right] = n; q.push(n->right); }
    }
}
```
Once every node knows its parent, the tree behaves like an undirected graph and a plain BFS from the target for K levels gives the answer. **"Add parent pointers to turn a tree into a graph"** is a pattern worth remembering.

---

## 10. Complexity table

| Operation | Balanced BST | Skewed BST | General binary tree |
|---|---|---|---|
| Search | O(log n) | O(n) | O(n) |
| Insert | O(log n) | O(n) | O(n) |
| Delete | O(log n) | O(n) | O(n) |
| Traversal | O(n) | O(n) | O(n) |
| Space (recursion) | O(log n) | O(n) | O(h) |

**Why do AVL and Red-Black trees exist?** A plain BST degenerates to a linked list when data arrives sorted, destroying the O(log n) guarantee. Self-balancing trees rotate on insert/delete to keep the height O(log n). AVL is more strictly balanced (faster lookups); Red-Black rebalances less aggressively (faster insert/delete) and is what `std::map` and most language libraries use.

---

## 11. Self test (cover the answers)

1. Which traversal of a BST gives sorted output? -> *Inorder.*
2. Why is preorder + postorder insufficient to rebuild a binary tree? -> *They do not reveal the left/right split; a single-child node is ambiguous. You need inorder.*
3. In the O(n) diameter solution, why return `1+max(lh,rh)` but record `lh+rh`? -> *The parent can only use a path descending through one side; the full diameter through this node uses both.*
4. What are the three cases of BST deletion? -> *Leaf; one child; two children (replace with the inorder successor).*
5. Why does the naive isValidBST fail? -> *It only compares with immediate children; a node can violate an ancestor's bound while satisfying its parent.*
6. Why use BFS, not DFS, for a top view? -> *BFS visits in non-decreasing depth, so the first node seen at each horizontal distance is genuinely the topmost.*
7. Height of a perfect binary tree with n nodes? -> *log2(n+1) - 1 in edges.*
8. Space complexity of recursive traversal? -> *O(h) — O(log n) if balanced, O(n) if skewed.*
9. How do you find the LCA in a BST quickly? -> *Walk down from the root; the first node whose value lies between the two targets is the LCA.*

---

## 12. Tonight's drill

Write from memory, on paper:
1. All three recursive traversals (they should take 30 seconds).
2. Iterative inorder with a stack.
3. Level order with the `int sz = q.size()` trick.
4. `height`, `diameter` (O(n) version), `isBalanced`.
5. `lca` for a general binary tree — and be able to explain the return values as messages.
6. BST `insert` and `delete` (all three cases).
7. `isValidBST` with the min/max range.

That set covers virtually every binary tree question that appears in a placement assessment.

Next: [06_POWER_QUESTIONS.md](06_POWER_QUESTIONS.md)
