# Toast — Round 1: Bakery Profit (Max Profit) + Follow-ups
**Role:** Software Engineer — Backend (Java)
**Round:** R1 — Algorithmic / DSA (live coding, ~45 min)
**Source:** Live interview question with progressive follow-ups

---

## How to Use This Guide

This was a single problem the interviewer **mutated three times** to test how fast you adapt.
Each stage follows the **STAR coding framework**:

- **S — Situation:** What is the problem really asking?
- **T — Task:** The core constraint / challenge.
- **A — Action:** Brute force → optimal, thinking out loud.
- **R — Result:** Final code + complexity stated before moving on.

> **The key twist vs. classic LC 121:** there are **two separate arrays** —
> `costs[i]` (prep/buy cost on day `i`) and `sales[j]` (revenue/sell price on day `j`).
> Profit for a trade is `sales[j] - costs[i]`, **not** `prices[j] - prices[i]`.

---

## Stage 1 — Bakery Profit with a 5-Day Gap

### S — Situation
Buy on day `i`, sell on day `j` where `j - i >= 5`.
Maximize `sales[j] - costs[i]`. Return `0` if no profitable sale exists.

### T — Task
This is the "min so far" pattern (LC 121 family) with **two arrays** and a **gap constraint**.
The challenge: don't let a cost that is too *recent* (within 5 days of `j`) be used as a buy.

### A — Action

**Clarifying / counter questions to ask the interviewer first:**
- What if `n < 5`? → no valid pair → return `0`
- What if all profits are negative? → return `0` (no trade beats a bad trade)
- Are indices 0-based? → yes; `j - i >= 5` means at least 5 days apart

**Brute force O(n²):** try every valid pair `(i, j)` with `j >= i + 5`, track max — state it, then improve.

**Optimal O(n) — roll a minimum cost forward.**
As `j` advances, the **latest valid buy day is `j - 5`**. Maintain
`minCost = min(costs[0 .. j-5])` and at each `j` compute `sales[j] - minCost`.

```java
public static long maxProfit(List<Long> costs, List<Long> sales) {
    int n = costs.size();
    if (n < 6) return 0L;  // need a 5-day gap: index 0 -> index 5

    long maxProfit = 0L;
    long minCost = costs.get(0);

    for (int j = 5; j < n; j++) {
        // costs[j-5] just became a valid buy day for selling on j
        minCost = Math.min(minCost, costs.get(j - 5));
        long profit = sales.get(j) - minCost;
        maxProfit = Math.max(maxProfit, profit);
    }
    return maxProfit;
}
```

**Why `costs.get(j - 5)`?** Each iteration "unlocks" exactly one new legal buy candidate
(the one now 5 days behind `j`) and folds it into the running minimum — no look-back needed.

### R — Result
| | Brute Force | Optimised |
|---|---|---|
| Time | O(n²) | O(n) |
| Space | O(1) | O(1) |

**Edge cases handled:**
- `n < 6` → return `0` immediately
- All profits negative → `maxProfit` stays `0` ✅

---

## Stage 2 — Follow-up: Must Sell on a *Different* Day (`j > i`)

> *Interviewer:* "Assume I can't sell the bread on the same day — I can only sell the next day or later. How to solve that?"

### S — Situation
Same bakery profit, but the gap shrinks from 5 days to **1 day**: valid pairs are `j >= i + 1`
(same-day sale is **not** allowed).

### T — Task
Maximize `sales[j] - costs[i]` where `j > i`. Identical pattern — only the offset changes.

### A — Action
Iterate `j` from index `1`. Track `minCost = min(costs[0 .. j-1])`, then compute profit against
the best historical cost.

```java
public static long maxProfit(List<Long> costs, List<Long> sales) {
    int n = costs.size();
    if (n < 2) return 0L;  // need at least 2 days

    long maxProfit = 0L;
    long minCost = costs.get(0);  // day 0 is the earliest valid buy

    for (int j = 1; j < n; j++) {
        // costs[j-1] just became a valid buy day for selling on j
        minCost = Math.min(minCost, costs.get(j - 1));
        long profit = sales.get(j) - minCost;
        maxProfit = Math.max(maxProfit, profit);
    }
    return maxProfit;
}
```

### Diff vs. Stage 1
| | Stage 1 | Stage 2 |
|---|---|---|
| Min gap | 5 days | 1 day |
| Loop starts at | `j = 5` | `j = 1` |
| `minCost` tracks | `costs[0 .. j-5]` | `costs[0 .. j-1]` |
| Base check | `n < 6` | `n < 2` |

### R — Result
Time: O(n) | Space: O(1)

> **Interviewer trap:** Don't update `minCost` with `costs.get(j)` — that would allow buying
> and selling on the **same** day. Always use `costs.get(j - 1)` to enforce strict `j > i`. ✅

---

## Stage 3 — Follow-up: Multiple Buys Allowed

> *Interviewer:* "There can be multiple buys."

This is the LC 122 (Best Time to Buy and Sell Stock II) variant — but with the **separate
`costs`/`sales` arrays** twist that changes the greedy.

### S — Situation
Buy multiple times, hold **at most one** unit at a time (must sell before buying again),
still `j > i` (no same-day sale). `costs[i]` = prep cost, `sales[i]` = revenue, and they
can differ on the same day.

### T — Task
Sum the profit of every profitable, non-overlapping transaction.

### A — Action

**Counter questions:**
- Hold multiple stocks simultaneously? → No — must sell before buying again
- Buy and sell on the same day? → No (`j > i`)
- Can `costs` and `sales` differ on the same day? → Yes

**Naive greedy (the LC 122 instinct — note its limitation):**
Take every positive day-to-day gain `sales[i] - costs[i-1]`.

```java
public static long maxProfit(List<Long> costs, List<Long> sales) {
    int n = costs.size();
    if (n < 2) return 0L;

    long totalProfit = 0L;
    for (int i = 1; i < n; i++) {
        // buy on day i-1, sell on day i; take only if positive
        long profit = sales.get(i) - costs.get(i - 1);
        if (profit > 0) totalProfit += profit;
    }
    return totalProfit;
}
```

**Why that isn't the whole story:** because `costs` and `sales` are **separate** arrays,
"buy day `i-1`, sell day `i`" pairs aren't telescoping the way single-array LC 122 does.
To capture a multi-day rising run with a single buy, walk in **buy → peak → next buy** pairs:

```java
public static long maxProfit(List<Long> costs, List<Long> sales) {
    int n = costs.size();
    if (n < 2) return 0L;

    long totalProfit = 0L;
    int i = 0;

    while (i < n - 1) {
        // skip days where buying isn't worth it
        while (i < n - 1 && sales.get(i + 1) <= costs.get(i)) {
            i++;
        }
        if (i == n - 1) break;

        long buyCost = costs.get(i);

        // climb to the local peak in sales
        int j = i + 1;
        while (j < n - 1 && sales.get(j + 1) >= sales.get(j)) {
            j++;
        }

        totalProfit += sales.get(j) - buyCost;
        i = j + 1;  // next buy must come after this sell
    }
    return totalProfit;
}
```

### R — Result
Time: O(n) | Space: O(1)

**Say these out loud:**
| Scenario | Answer |
|---|---|
| Overlapping transactions? | Not allowed — sell before next buy |
| Same-day buy + sell? | No — `j > i` enforced |
| Why does greedy work? | Every positive segment is independent — no future dependency |
| What if everything is negative? | Returns `0` — no trade beats a bad trade |

---

## Pattern Summary

| Stage | Constraint | Technique | Loop offset |
|---|---|---|---|
| 1 | `j - i >= 5` | Rolling min cost | `j` from 5, fold `costs[j-5]` |
| 2 | `j > i` | Rolling min cost | `j` from 1, fold `costs[j-1]` |
| 3 | multiple buys, `j > i` | Greedy buy→peak | scan in transaction pairs |

> **Senior-level signal:** call out the two-array twist explicitly — most candidates pattern-match
> to single-array LC 121/122 and silently use `prices[j] - prices[i]`. Naming `sales[j] - costs[i]`
> early, and protecting the gap with `costs[j-gap]`, is what the interviewer is watching for.

---

## Related
- See [toast-sde-prep.md](toast-sde-prep.md) — Problem 1 (single-array stock profit w/ 5s gap),
  LC 121, LC 122 for the classic single-price-array versions of this same pattern.
