# 02 — Arrays

> Arrays questions are **pattern recognition**. There are only about 7 patterns. Learn to name the pattern from the problem statement and the code writes itself.

---

## The pattern-recognition table — read this first

| Clue in the problem statement | Pattern to use |
|---|---|
| "sorted array", "pair with sum", "two ends" | **Two pointers** |
| "subarray of size k", "longest/shortest substring", "contiguous" | **Sliding window** |
| "sum of range i..j", many queries | **Prefix sum** |
| "maximum sum subarray" | **Kadane's algorithm** |
| "count of X", "seen before", "duplicate" | **Hash map / set** |
| "sorted array" + "find/search/first-last position" | **Binary search** |
| "0s, 1s, 2s", "sort colours", "partition" | **Dutch National Flag** |
| "in-place", "O(1) space", "rotate" | **Reversal / swapping trick** |
| "intervals", "meeting rooms" | **Sort by start, then merge** |
| "next greater/smaller element" | **Monotonic stack** (see [03_STACKS_QUEUES.md](03_STACKS_QUEUES.md)) |

---

## PATTERN 1: Two Pointers

**Idea:** two indices moving towards each other (or in the same direction), exploiting sortedness to skip work.

### 1.1 Pair with a given sum (sorted array)
```cpp
pair<int,int> twoSum(vector<int>& a, int target) {
    int i = 0, j = a.size() - 1;
    while (i < j) {
        int sum = a[i] + a[j];
        if (sum == target) return {i, j};
        else if (sum < target) i++;      // need bigger -> move left pointer right
        else j--;                         // need smaller -> move right pointer left
    }
    return {-1, -1};
}
```
**Invariant:** every pair we skip is provably not the answer. If `a[i]+a[j] < target`, then `a[i]` paired with anything <= `a[j]` is also too small, so `i` can never be part of an answer with any remaining `j`. Hence `i++` is safe.
**Time O(n), Space O(1).**

### 1.2 Two Sum on an UNSORTED array (hash map)
```cpp
vector<int> twoSumHash(vector<int>& a, int target) {
    unordered_map<int,int> seen;              // value -> index
    for (int i = 0; i < a.size(); i++) {
        int need = target - a[i];
        if (seen.count(need)) return {seen[need], i};
        seen[a[i]] = i;
    }
    return {};
}
```
**O(n) time, O(n) space.** Use this when the array is not sorted and you must return original indices.

### 1.3 Three Sum (find all unique triplets summing to 0)
```cpp
vector<vector<int>> threeSum(vector<int>& a) {
    sort(a.begin(), a.end());
    vector<vector<int>> res;
    int n = a.size();
    for (int i = 0; i < n - 2; i++) {
        if (i > 0 && a[i] == a[i-1]) continue;         // skip duplicate first element
        int l = i + 1, r = n - 1;
        while (l < r) {
            int s = a[i] + a[l] + a[r];
            if (s == 0) {
                res.push_back({a[i], a[l], a[r]});
                while (l < r && a[l] == a[l+1]) l++;    // skip duplicates
                while (l < r && a[r] == a[r-1]) r--;
                l++; r--;
            }
            else if (s < 0) l++;
            else r--;
        }
    }
    return res;
}
```
**O(n^2).** The duplicate-skipping is what most people get wrong — memorise those three `while` guards.

### 1.4 Remove duplicates from a sorted array in-place
```cpp
int removeDuplicates(vector<int>& a) {
    if (a.empty()) return 0;
    int k = 1;                                // k = write position
    for (int i = 1; i < a.size(); i++)
        if (a[i] != a[k-1]) a[k++] = a[i];
    return k;                                 // first k elements are the answer
}
```
**Slow/fast pointer pattern:** `k` is the slow writer, `i` is the fast reader. Same shape solves "move all zeros to the end", "remove element X".

### 1.5 Move all zeros to the end (keep relative order)
```cpp
void moveZeroes(vector<int>& a) {
    int k = 0;
    for (int i = 0; i < a.size(); i++)
        if (a[i] != 0) swap(a[k++], a[i]);
}
```

### 1.6 Container With Most Water
```cpp
int maxArea(vector<int>& h) {
    int l = 0, r = h.size() - 1, best = 0;
    while (l < r) {
        best = max(best, min(h[l], h[r]) * (r - l));
        if (h[l] < h[r]) l++; else r--;        // always move the SHORTER wall
    }
    return best;
}
```
**Why move the shorter one?** Width always shrinks. The only way to possibly gain area is to raise the limiting height, and the limiting height is the shorter wall. Moving the taller wall can never improve the result.

---

## PATTERN 2: Sliding Window

**Idea:** maintain a window `[l, r]` and a running aggregate; expand `r`, shrink `l` when the window becomes invalid. Each element enters and leaves at most once, so it is **O(n)** not O(n*k).

### 2.1 Fixed window — maximum sum of k consecutive elements
```cpp
int maxSumK(vector<int>& a, int k) {
    int sum = 0;
    for (int i = 0; i < k; i++) sum += a[i];   // first window
    int best = sum;
    for (int i = k; i < a.size(); i++) {
        sum += a[i] - a[i-k];                  // add new, remove old — O(1) slide
        best = max(best, sum);
    }
    return best;
}
```

### 2.2 Variable window — smallest subarray with sum >= target
```cpp
int minSubArrayLen(int target, vector<int>& a) {
    int l = 0, sum = 0, best = INT_MAX;
    for (int r = 0; r < a.size(); r++) {
        sum += a[r];
        while (sum >= target) {                // shrink while still valid
            best = min(best, r - l + 1);
            sum -= a[l++];
        }
    }
    return best == INT_MAX ? 0 : best;
}
```
(Assumes positive numbers — that is what makes shrinking monotone.)

### 2.3 Longest substring without repeating characters (the famous one)
```cpp
int lengthOfLongestSubstring(string s) {
    unordered_map<char,int> last;              // char -> last index seen
    int l = 0, best = 0;
    for (int r = 0; r < s.size(); r++) {
        if (last.count(s[r]) && last[s[r]] >= l)
            l = last[s[r]] + 1;                // jump l past the previous occurrence
        last[s[r]] = r;
        best = max(best, r - l + 1);
    }
    return best;
}
```
**The subtlety:** the `last[s[r]] >= l` check. Without it, a stale index from outside the current window drags `l` backwards.

### 2.4 Longest subarray with at most K distinct elements
```cpp
int longestKDistinct(vector<int>& a, int k) {
    unordered_map<int,int> cnt;
    int l = 0, best = 0;
    for (int r = 0; r < a.size(); r++) {
        cnt[a[r]]++;
        while (cnt.size() > k) {
            if (--cnt[a[l]] == 0) cnt.erase(a[l]);
            l++;
        }
        best = max(best, r - l + 1);
    }
    return best;
}
```

**Sliding-window template to memorise:**
```
for r in 0..n-1:
    add a[r] to window
    while (window is INVALID):
        remove a[l]; l++
    update answer with window [l..r]
```

---

## PATTERN 3: Prefix Sum

**Idea:** precompute cumulative sums so any range sum is answered in O(1).

```cpp
vector<long long> pre(n + 1, 0);
for (int i = 0; i < n; i++) pre[i+1] = pre[i] + a[i];

// sum of a[l..r] inclusive, 0-indexed:
long long rangeSum = pre[r+1] - pre[l];
```
**Use `pre[0] = 0` with size n+1** — it removes every off-by-one from the formula. This is worth memorising exactly.

### 3.1 Count subarrays with sum equal to K (prefix sum + hash map)  -- very common
```cpp
int subarraySum(vector<int>& a, int k) {
    unordered_map<long long,int> freq;
    freq[0] = 1;                     // empty prefix: needed so a prefix that itself equals k counts
    long long sum = 0;
    int count = 0;
    for (int x : a) {
        sum += x;
        if (freq.count(sum - k)) count += freq[sum - k];
        freq[sum]++;
    }
    return count;
}
```
**The insight:** `sum(l..r) = pre[r] - pre[l-1]`. So we want `pre[l-1] = pre[r] - k`. Counting how many earlier prefixes equal `pre[r]-k` counts all valid subarrays ending at `r`.
**This works with negative numbers too** — that is why it beats sliding window here.

### 3.2 Longest subarray with sum K
```cpp
int longestSubarraySumK(vector<int>& a, int k) {
    unordered_map<long long,int> firstIdx;      // prefix sum -> earliest index
    long long sum = 0; int best = 0;
    for (int i = 0; i < a.size(); i++) {
        sum += a[i];
        if (sum == k) best = i + 1;
        if (firstIdx.count(sum - k)) best = max(best, i - firstIdx[sum - k]);
        if (!firstIdx.count(sum)) firstIdx[sum] = i;   // keep EARLIEST -> longest span
    }
    return best;
}
```

### 3.3 Equilibrium index (left sum == right sum)
```cpp
int equilibrium(vector<int>& a) {
    long long total = accumulate(a.begin(), a.end(), 0LL), left = 0;
    for (int i = 0; i < a.size(); i++) {
        if (left == total - left - a[i]) return i;
        left += a[i];
    }
    return -1;
}
```

---

## PATTERN 4: Kadane's Algorithm (maximum subarray sum)

```cpp
int maxSubArray(vector<int>& a) {
    int best = a[0], cur = a[0];
    for (int i = 1; i < a.size(); i++) {
        cur = max(a[i], cur + a[i]);     // either extend the previous subarray, or restart here
        best = max(best, cur);
    }
    return best;
}
```
**Why it works:** if the running sum ever becomes worse than starting fresh at `a[i]`, the prefix was a net liability, so discard it. `cur` = best subarray sum *ending exactly at i*.

**Edge case:** initialising `best = 0` breaks on all-negative arrays. Initialise from `a[0]`.

### 4.1 Kadane with the actual subarray indices
```cpp
void maxSubArrayIdx(vector<int>& a) {
    int best = a[0], cur = a[0], s = 0, bl = 0, br = 0;
    for (int i = 1; i < a.size(); i++) {
        if (a[i] > cur + a[i]) { cur = a[i]; s = i; }   // restart
        else cur += a[i];
        if (cur > best) { best = cur; bl = s; br = i; }
    }
    cout << best << " from " << bl << " to " << br << "\n";
}
```

### 4.2 Maximum product subarray (the twist: track min too)
```cpp
int maxProduct(vector<int>& a) {
    int best = a[0], curMax = a[0], curMin = a[0];
    for (int i = 1; i < a.size(); i++) {
        if (a[i] < 0) swap(curMax, curMin);     // a negative flips max and min
        curMax = max(a[i], curMax * a[i]);
        curMin = min(a[i], curMin * a[i]);
        best = max(best, curMax);
    }
    return best;
}
```
**Why track the minimum?** A large negative product becomes the largest positive product when multiplied by another negative.

---

## PATTERN 5: Binary Search

**Template that never has an infinite loop:**
```cpp
int binarySearch(vector<int>& a, int target) {
    int lo = 0, hi = a.size() - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;       // NOT (lo+hi)/2 -> avoids overflow
        if (a[mid] == target) return mid;
        else if (a[mid] < target) lo = mid + 1;
        else hi = mid - 1;
    }
    return -1;
}
```
**Three things to get right:** `lo <= hi` (not `<`), `mid = lo + (hi-lo)/2`, and `mid +/- 1` (never `lo = mid`, which loops forever).

### 5.1 First and last occurrence of a target
```cpp
int firstOcc(vector<int>& a, int t) {
    int lo = 0, hi = a.size()-1, ans = -1;
    while (lo <= hi) {
        int mid = lo + (hi-lo)/2;
        if (a[mid] == t) { ans = mid; hi = mid - 1; }   // record, then keep going LEFT
        else if (a[mid] < t) lo = mid + 1;
        else hi = mid - 1;
    }
    return ans;
}
int lastOcc(vector<int>& a, int t) {
    int lo = 0, hi = a.size()-1, ans = -1;
    while (lo <= hi) {
        int mid = lo + (hi-lo)/2;
        if (a[mid] == t) { ans = mid; lo = mid + 1; }   // record, then keep going RIGHT
        else if (a[mid] < t) lo = mid + 1;
        else hi = mid - 1;
    }
    return ans;
}
// count of t = lastOcc - firstOcc + 1
```
STL shortcut: `lower_bound(a.begin(),a.end(),t) - a.begin()` gives the first index >= t.

### 5.2 Search in a rotated sorted array  (very common)
```cpp
int searchRotated(vector<int>& a, int t) {
    int lo = 0, hi = a.size() - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (a[mid] == t) return mid;
        if (a[lo] <= a[mid]) {                  // LEFT half is sorted
            if (a[lo] <= t && t < a[mid]) hi = mid - 1;
            else lo = mid + 1;
        } else {                                 // RIGHT half is sorted
            if (a[mid] < t && t <= a[hi]) lo = mid + 1;
            else hi = mid - 1;
        }
    }
    return -1;
}
```
**Key insight:** in a rotated array, at least one half around `mid` is always properly sorted. Identify which, test whether the target lies inside it, and discard the other half.

### 5.3 Find the minimum in a rotated sorted array
```cpp
int findMin(vector<int>& a) {
    int lo = 0, hi = a.size() - 1;
    while (lo < hi) {                       // note: < not <=
        int mid = lo + (hi - lo) / 2;
        if (a[mid] > a[hi]) lo = mid + 1;   // min is to the right
        else hi = mid;                       // min is at mid or to the left
    }
    return a[lo];
}
```

### 5.4 Square root by binary search (binary search on the ANSWER)
```cpp
int mySqrt(int x) {
    long long lo = 0, hi = x, ans = 0;
    while (lo <= hi) {
        long long mid = lo + (hi - lo) / 2;
        if (mid * mid <= x) { ans = mid; lo = mid + 1; }
        else hi = mid - 1;
    }
    return (int)ans;
}
```
**Generalise this:** "minimise the maximum" / "find the smallest capacity that works" problems are all binary search on the answer, with a `feasible(x)` predicate.

---

## PATTERN 6: In-place rearrangement tricks

### 6.1 Reverse an array
```cpp
void reverseArr(vector<int>& a) {
    int i = 0, j = a.size() - 1;
    while (i < j) swap(a[i++], a[j--]);
}
```

### 6.2 Rotate an array left by k — the reversal algorithm (O(n) time, O(1) space)
```cpp
void rotateLeft(vector<int>& a, int k) {
    int n = a.size();
    k %= n;                                       // k can exceed n
    reverse(a.begin(), a.begin() + k);            // reverse first k
    reverse(a.begin() + k, a.end());              // reverse the rest
    reverse(a.begin(), a.end());                  // reverse everything
}
// For rotate RIGHT by k: use k = n - (k % n), then the same three reversals.
```
**Dry run** `[1,2,3,4,5], k=2`: `[2,1|3,4,5]` -> `[2,1|5,4,3]` -> `[3,4,5,1,2]`. Correct.

### 6.3 Dutch National Flag — sort an array of 0s, 1s and 2s in one pass
```cpp
void sort012(vector<int>& a) {
    int low = 0, mid = 0, high = a.size() - 1;
    while (mid <= high) {
        if (a[mid] == 0)      swap(a[low++], a[mid++]);
        else if (a[mid] == 1) mid++;
        else                  swap(a[mid], a[high--]);   // do NOT increment mid here
    }
}
```
**Invariant:** `[0, low)` are all 0s, `[low, mid)` are all 1s, `(high, n-1]` are all 2s, and `[mid, high]` is unexplored.
**The trap:** when you swap with `high`, the incoming element is unexamined, so `mid` must not advance.

### 6.4 Moore Voting algorithm — majority element (appears more than n/2 times)
```cpp
int majority(vector<int>& a) {
    int cand = a[0], count = 1;
    for (int i = 1; i < a.size(); i++) {
        if (count == 0) { cand = a[i]; count = 1; }
        else if (a[i] == cand) count++;
        else count--;
    }
    int c = 0;                                  // verification pass
    for (int x : a) if (x == cand) c++;
    return c > (int)a.size()/2 ? cand : -1;
}
```
**Intuition:** each non-majority element cancels one majority element. Since the majority occurs more than n/2 times, it survives all the cancellation.

### 6.5 Missing number in 1..n
```cpp
int missingNumber(vector<int>& a) {
    long long n = a.size() + 1;
    long long expected = n * (n + 1) / 2;
    long long actual = accumulate(a.begin(), a.end(), 0LL);
    return (int)(expected - actual);
}

// XOR version, immune to overflow:
int missingXor(vector<int>& a) {
    int x = 0;
    for (int i = 1; i <= (int)a.size() + 1; i++) x ^= i;
    for (int v : a) x ^= v;
    return x;                        // pairs cancel (a^a=0), the missing one survives
}
```

---

## PATTERN 7: Intervals

### 7.1 Merge overlapping intervals
```cpp
vector<vector<int>> mergeIntervals(vector<vector<int>>& iv) {
    sort(iv.begin(), iv.end());               // sort by start
    vector<vector<int>> res;
    for (auto& cur : iv) {
        if (res.empty() || res.back()[1] < cur[0]) res.push_back(cur);   // no overlap
        else res.back()[1] = max(res.back()[1], cur[1]);                 // extend
    }
    return res;
}
```
**Sorting by start is the whole trick** — after that, an interval can only overlap with the last one added.

### 7.2 Minimum number of meeting rooms
```cpp
int minRooms(vector<pair<int,int>>& meetings) {
    vector<int> start, finish;
    for (auto& m : meetings) { start.push_back(m.first); finish.push_back(m.second); }
    sort(start.begin(), start.end());
    sort(finish.begin(), finish.end());
    int rooms = 0, best = 0, j = 0;
    for (int i = 0; i < (int)start.size(); i++) {
        while (j < (int)finish.size() && finish[j] <= start[i]) { rooms--; j++; }
        rooms++;
        best = max(best, rooms);
    }
    return best;
}
```

---

## Matrix (2D array) essentials

### Spiral traversal
```cpp
vector<int> spiral(vector<vector<int>>& m) {
    vector<int> res;
    if (m.empty()) return res;
    int top = 0, bottom = m.size()-1, left = 0, right = m[0].size()-1;
    while (top <= bottom && left <= right) {
        for (int j = left; j <= right; j++) res.push_back(m[top][j]);
        top++;
        for (int i = top; i <= bottom; i++) res.push_back(m[i][right]);
        right--;
        if (top <= bottom) { for (int j = right; j >= left; j--) res.push_back(m[bottom][j]); bottom--; }
        if (left <= right) { for (int i = bottom; i >= top; i--) res.push_back(m[i][left]); left++; }
    }
    return res;
}
```
**The two `if` guards** prevent re-traversing a single remaining row or column. Skipping them is the classic bug.

### Rotate a matrix 90 degrees clockwise, in place
```cpp
void rotate90(vector<vector<int>>& m) {
    int n = m.size();
    for (int i = 0; i < n; i++)                 // 1. transpose
        for (int j = i + 1; j < n; j++)
            swap(m[i][j], m[j][i]);
    for (int i = 0; i < n; i++)                 // 2. reverse each row
        reverse(m[i].begin(), m[i].end());
}
// Anticlockwise = transpose, then reverse the ORDER OF ROWS instead.
```
Note `j = i + 1` — starting `j` at 0 swaps everything twice and undoes the transpose.

### Search in a row-wise AND column-wise sorted matrix — O(n+m)
```cpp
bool searchMatrix(vector<vector<int>>& m, int t) {
    int i = 0, j = m[0].size() - 1;           // start at TOP-RIGHT
    while (i < (int)m.size() && j >= 0) {
        if (m[i][j] == t) return true;
        else if (m[i][j] > t) j--;            // everything below in this column is bigger
        else i++;                              // everything left in this row is smaller
    }
    return false;
}
```
The top-right corner is the only start where each comparison eliminates a whole row or column.

### Set matrix zeroes (O(1) extra space — use row 0 and column 0 as markers)
```cpp
void setZeroes(vector<vector<int>>& m) {
    int n = m.size(), c = m[0].size();
    bool col0 = false;
    for (int i = 0; i < n; i++) {
        if (m[i][0] == 0) col0 = true;
        for (int j = 1; j < c; j++)
            if (m[i][j] == 0) { m[i][0] = 0; m[0][j] = 0; }
    }
    for (int i = n - 1; i >= 0; i--) {          // BACKWARDS so the markers stay intact
        for (int j = c - 1; j >= 1; j--)
            if (m[i][0] == 0 || m[0][j] == 0) m[i][j] = 0;
        if (col0) m[i][0] = 0;
    }
}
```

---

## Sorting algorithms — needed for TCR as well

| Algorithm | Best | Average | Worst | Space | Stable? | In-place? |
|---|---|---|---|---|---|---|
| Bubble | O(n) | O(n^2) | O(n^2) | O(1) | Yes | Yes |
| Selection | O(n^2) | O(n^2) | O(n^2) | O(1) | No | Yes |
| Insertion | O(n) | O(n^2) | O(n^2) | O(1) | Yes | Yes |
| Merge | O(n log n) | O(n log n) | O(n log n) | **O(n)** | Yes | No |
| Quick | O(n log n) | O(n log n) | **O(n^2)** | O(log n) | No | Yes |
| Heap | O(n log n) | O(n log n) | O(n log n) | O(1) | No | Yes |
| Counting | O(n+k) | O(n+k) | O(n+k) | O(k) | Yes | No |

**Exam-ready facts:**
- **Quick sort worst case O(n^2)** happens on an already-sorted array when the pivot is the first or last element. Randomised or median-of-three pivot selection fixes it.
- **Merge sort is preferred for linked lists** (no random access needed, and the merge needs no extra array) and is the **stable** choice. Quick sort is preferred for arrays (better cache locality, in-place).
- **Stable** = equal elements keep their original relative order. Matters when sorting records by one field after another.
- `std::sort` is **introsort**: quicksort that switches to heapsort at deep recursion and insertion sort on small chunks. It is **not stable** — use `stable_sort` when stability matters.
- Comparison-based sorting cannot beat **O(n log n)**. Counting/radix/bucket sort beat it only by not comparing (they assume bounded integer keys).

```cpp
// Quick sort (Lomuto partition) — be ready to write this from memory
int partitionArr(vector<int>& a, int lo, int hi) {
    int pivot = a[hi], i = lo - 1;
    for (int j = lo; j < hi; j++)
        if (a[j] < pivot) swap(a[++i], a[j]);
    swap(a[i+1], a[hi]);
    return i + 1;
}
void quickSort(vector<int>& a, int lo, int hi) {
    if (lo < hi) {
        int p = partitionArr(a, lo, hi);
        quickSort(a, lo, p - 1);
        quickSort(a, p + 1, hi);
    }
}

// Merge sort
void mergeParts(vector<int>& a, int l, int m, int r) {
    vector<int> tmp;
    int i = l, j = m + 1;
    while (i <= m && j <= r) tmp.push_back(a[i] <= a[j] ? a[i++] : a[j++]);
    while (i <= m) tmp.push_back(a[i++]);
    while (j <= r) tmp.push_back(a[j++]);
    for (int k = 0; k < (int)tmp.size(); k++) a[l + k] = tmp[k];
}
void mergeSort(vector<int>& a, int l, int r) {
    if (l >= r) return;
    int m = l + (r - l) / 2;
    mergeSort(a, l, m);
    mergeSort(a, m + 1, r);
    mergeParts(a, l, m, r);
}
```

---

## Array edge cases — check every one before you submit

- [ ] Empty array (n = 0) — does `a[0]` crash?
- [ ] Single element (n = 1)
- [ ] All elements equal
- [ ] All negative (breaks Kadane if you initialise `best = 0`)
- [ ] Already sorted / reverse sorted
- [ ] Duplicates present (Three Sum, remove-duplicates)
- [ ] Sum or product overflow -> use `long long`
- [ ] k greater than n in rotate/window problems -> use `k % n`
- [ ] Reading `n` elements but the loop runs `<= n`

---

## 10-minute self test

Write these from memory, no reference:
1. Kadane (4 lines)
2. Binary search (7 lines)
3. Dutch National Flag (6 lines)
4. Rotate by k using three reversals (4 lines)
5. Longest substring without repeating characters (7 lines)

If you can produce all five cold, the array question tomorrow is a formality.

Next: [03_STACKS_QUEUES.md](03_STACKS_QUEUES.md)
