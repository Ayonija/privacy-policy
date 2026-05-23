# Day 58 — Graphs: BFS with State Grouping, Clone & Jump Game IV
**Week 09 | Phase 1: DSA Mastery | Month 2**

## Focus
Minimum Genetic Mutation mirrors Word Ladder with a fixed bank. Clone Graph handles cyclic graphs by tracking clones in a HashMap. Jump Game IV's key insight: process all same-value indices in one BFS level, then clear the bucket to avoid quadratic revisits.

---

## DSA (2 hours)
### Pattern: BFS with Word/State Substitution + Graph Deep Copy + BFS with Value Index Grouping

**Minimum Genetic Mutation (LC 433):**
Exactly like Word Ladder but with 8-character gene strings using alphabet {A, C, G, T}. BFS from startGene to endGene; a mutation is valid only if the intermediate gene is in the bank. Generate candidates by substituting each position with {A, C, G, T}.

**Clone Graph (LC 133):**
BFS/DFS with a `{original: clone}` HashMap. Before processing a node's neighbours, check if a clone already exists — reuse it to prevent infinite loops in cyclic graphs. New nodes are created only on first encounter.

**Jump Game IV (LC 1345):**
From index i, you can jump to i-1, i+1, or any index j where `arr[j] == arr[i]`. BFS for minimum jumps. Key optimisation: group indices by value (`val_to_indices` map). When visiting a node via a same-value jump, process ALL same-value indices in one BFS step, then **clear the bucket** — if you don't clear it, future BFS levels will re-enqueue already-visited same-value indices, leading to O(n²) time.

**Trigger condition:**
- "minimum mutations/transformations with a valid-intermediate set" → BFS with bank-validated substitutions
- "deep copy a graph including cycles" → BFS/DFS with `{original: clone}` HashMap to handle cycles
- "minimum jumps where same-value positions can be jumped to directly" → BFS + value→index map; clear after use

**Time complexity:** LC 433: O(N×8×4) = O(N) | LC 133: O(V+E) | LC 1345: O(n)
**Space complexity:** O(N) / O(V+E) / O(n)

---

### Problems
| # | Problem | LC # | Difficulty | Pattern | Key Insight |
|---|---------|------|------------|---------|-------------|
| 1 | Minimum Genetic Mutation | 433 | Medium | BFS with bank-validated mutations | Try {A,C,G,T} substitutions per position; valid if in bank; BFS levels = mutation count |
| 2 | Clone Graph | 133 | Medium | BFS/DFS with old→new HashMap | HashMap tracks created clones; prevents infinite loops in cycles; clone before recursing neighbours |
| 3 | Jump Game IV | 1345 | Hard | BFS + value-grouped index map (clear after use) | Process all same-value indices at once; clear the bucket after first use to prevent re-enqueuing |

---

### Code Skeleton
```python
from collections import deque, defaultdict

# Minimum Genetic Mutation (LC 433)
def minMutation(startGene, endGene, bank):
    bank_set = set(bank)
    if endGene not in bank_set: return -1
    queue = deque([(startGene, 0)])
    visited = {startGene}
    while queue:
        gene, mutations = queue.popleft()
        if gene == endGene: return mutations
        for i in range(8):
            for ch in 'ACGT':
                candidate = gene[:i] + ch + gene[i+1:]
                if candidate in bank_set and candidate not in visited:
                    visited.add(candidate)
                    queue.append((candidate, mutations + 1))
    return -1

# Clone Graph (LC 133)
class Node:
    def __init__(self, val=0, neighbors=None):
        self.val = val
        self.neighbors = neighbors or []

def cloneGraph(node):
    if not node: return None
    clones = {}   # original → clone

    def dfs(orig):
        if orig in clones: return clones[orig]
        clone = Node(orig.val)
        clones[orig] = clone         # register before recursing (handles cycles)
        for nb in orig.neighbors:
            clone.neighbors.append(dfs(nb))
        return clone

    return dfs(node)

# Jump Game IV (LC 1345)
def minJumps(arr):
    n = len(arr)
    if n == 1: return 0
    val_to_idx = defaultdict(list)
    for i, v in enumerate(arr):
        val_to_idx[v].append(i)

    visited = {0}
    queue = deque([0])
    steps = 0

    while queue:
        steps += 1
        for _ in range(len(queue)):
            i = queue.popleft()
            # Same-value jumps
            for j in val_to_idx[arr[i]]:
                if j not in visited:
                    if j == n - 1: return steps
                    visited.add(j)
                    queue.append(j)
            val_to_idx[arr[i]].clear()   # clear to prevent re-processing this group
            # Adjacent jumps
            for j in (i - 1, i + 1):
                if 0 <= j < n and j not in visited:
                    if j == n - 1: return steps
                    visited.add(j)
                    queue.append(j)
    return -1
```

---

### Edge Cases to Trace Before Coding
- LC 433: startGene == endGene → return 0 immediately; endGene not in bank → return -1 immediately
- LC 133: null node → return None; disconnected graph — problem guarantees connected graph; single node with no neighbours → clone with empty neighbours
- LC 1345: single-element array → return 0; last element is same value as first → same-value jump to answer in 1 step; large arrays with many duplicate values → clearing val_to_idx after first use is critical for O(n)

---

### Interview Pattern Drill
| Problem | BFS queue content | Neighbour generation | Key optimisation |
|---------|-----------------|---------------------|-----------------|
| Genetic Mutation | `(gene_string, count)` | 8 positions × 4 chars | Bank set for O(1) validation |
| Clone Graph | BFS over original nodes | `node.neighbors` | HashMap prevents cycle-loop |
| Jump Game IV | index | `i-1, i+1, val_to_idx[arr[i]]` | Clear same-value bucket after first visit |

---

### STAR Interview Framework

> **BFS + Value-Grouped Index Map (Jump Game IV):** brute-force O(n²) re-processing same-value groups → this approach O(n) time, O(n) space

**S:** "Given array of n values where same-value indices can be jumped to directly. Naive BFS re-enqueues same-value indices every time any member is visited — O(n²) for arrays with many duplicates."
**T:** "Need O(n) by processing each same-value group at most once across all BFS levels."
**A (60%):**
1. *Classify:* "'Minimum jumps with same-value teleportation' → BFS with value→index map; one-time bucket clearing."
2. *Init:* "Build `val_to_idx` map grouping all indices by value. Enqueue index 0."
3. *Loop/Step:* "BFS level by level. For each index i dequeued: process ALL `val_to_idx[arr[i]]` same-value jumps; then `val_to_idx[arr[i]].clear()` to prevent re-processing. Then enqueue `i-1` and `i+1` if unvisited."
4. *Termination:* "Return step count when index `n-1` is first reached."
5. *Gotcha:* "Clear the bucket AFTER processing all its members — not before, or you lose the jump targets."
**R:** "O(n) time, O(n) space. For n=50,000 with all equal values: 50K ops vs 2.5B ops without clearing."

**Alternatives & why not:**
| Alternative | Use when | Why not here |
|------------|----------|-------------|
| Standard BFS | No same-value teleportation | Correct but O(n²) for large duplicate groups |
| Dijkstra | Weighted jump costs | All jumps cost 1 — BFS is sufficient and faster |

---

## System Design (1 hour)
### Topic: DNS Security — DNSSEC, DoH, and DoT

**The DNS security problem:**
Classic DNS sends queries in plaintext over UDP port 53. Three threats:
1. **DNS cache poisoning (Kaminsky attack):** an attacker floods a recursive resolver with forged DNS responses; if the forged response's transaction ID matches before the real response arrives, the resolver caches the malicious IP.
2. **Eavesdropping:** anyone on the network path can read which domains you query — exposes browsing behaviour.
3. **Man-in-the-Middle:** ISP or attacker can intercept and modify DNS responses to redirect users to malicious servers.

**DNSSEC — DNS Security Extensions:**
Adds cryptographic signatures to DNS records. The authoritative nameserver signs each record set with its private key; resolvers verify the signature chain from the DNS root down to the domain.

```
Root zone key (KSK) → signs TLD keys
TLD key → signs domain keys
Domain key → signs individual DNS records (A, MX, etc.)
```

- **Prevents:** cache poisoning (forged records fail signature verification)
- **Does NOT prevent:** eavesdropping (records are signed but not encrypted)
- **Limitation:** complex to deploy and manage; ~30 % of domains use DNSSEC (2025)

**DNS-over-HTTPS (DoH):**
DNS queries are sent as HTTPS POST/GET requests to a DoH resolver (e.g., `https://1.1.1.1/dns-query`). The DNS query is encrypted inside HTTPS — ISPs and on-path attackers cannot see what you're querying.

- **Browser support:** Firefox (built-in, Cloudflare 1.1.1.1), Chrome (configurable)
- **Controversy:** bypasses enterprise DNS monitoring and parental controls; shifts DNS query visibility from ISP to DoH provider (privacy improvement but centralisation trade-off)
- **Port:** 443 (same as HTTPS — hard to block)

**DNS-over-TLS (DoT):**
DNS queries sent over a TLS-encrypted TCP connection to the resolver on port 853. Same encryption benefit as DoH but distinguishable from HTTPS traffic (port 853 can be firewall-blocked — less evasive than DoH).

**Comparison:**
| Feature | Classic DNS | DNSSEC | DoH | DoT |
|---------|-------------|--------|-----|-----|
| Encrypts queries | No | No | Yes (HTTPS) | Yes (TLS) |
| Prevents cache poisoning | No | Yes | No | No |
| Prevents eavesdropping | No | No | Yes | Yes |
| Firewall-resistant | — | — | Yes (port 443) | No (port 853) |
| Deployment complexity | Low | High | Medium | Medium |

**Interview talking point:** "If asked how to prevent DNS spoofing for your service, answer: deploy DNSSEC on your authoritative zone — clients with DNSSEC-validating resolvers can't be redirected to a forged IP. For client-side privacy (protecting your users' DNS queries from eavesdropping), encourage DoH or DoT in your client apps. Note that DNSSEC and DoH/DoT solve different problems — DNSSEC verifies record authenticity; DoH/DoT encrypts the query channel."

---

## Assessment / Mock (—)
### Activity: —

---

## Behavioral (30 min)
- Leadership principle: Earn Trust

**Full STAR Story — "Prioritising Security Vulnerability Fixes":**
**S (20%):** "At a healthcare data platform, a security audit revealed three DNS-layer vulnerabilities simultaneously: no DNSSEC on our authoritative zone (cache poisoning risk), plaintext DNS queries from our mobile clients (eavesdropping risk), and a misconfigured TTL that would leave old records alive for 24 hours during an incident."
**T:** "I had to triage and sequence fixes across three layers — analogous to choosing between DNSSEC, DoH, and DoT — with one week before a SOC 2 audit."
**A (60% — 'I' not 'we'):** "(1) I mapped each vulnerability to its threat model: DNSSEC prevents record forgery (integrity), DoT/DoH prevents eavesdropping (privacy), TTL misconfiguration affects incident response speed. (2) I ranked by blast radius: DNSSEC first because forged DNS records could redirect all users to a malicious server — a HIPAA breach scenario. (3) I deployed DNSSEC on the authoritative zone in 2 days and verified the signature chain with `dig +dnssec`. (4) I then configured DoT on our mobile client DNS resolver and lowered our operational records' TTL from 86400s to 300s for safer incident response."
**R (20%):** "All three fixes shipped before the audit. SOC 2 auditors cited our DNS security posture as an example of defence-in-depth. Zero DNS-related security incidents in the 18 months following."
*Works for: Earn Trust, Insist on the Highest Standards, Dive Deep.*

---

## Flashcards

| Q | A |
|---|---|
| How does Minimum Genetic Mutation generate valid BFS neighbours? | For each of 8 positions, try substituting {A, C, G, T}; add the candidate to the BFS queue only if it is in the bank set and not yet visited |
| How does Clone Graph handle cyclic graphs without infinite recursion? | The `{original: clone}` HashMap is populated BEFORE recursing into neighbours — if a cycle brings DFS back to the same node, the HashMap lookup returns the existing clone immediately |
| What happens if you don't clear `val_to_idx` after processing same-value jumps in Jump Game IV? | Re-enqueuing indices already visited causes O(n²) redundant work — clearing the list after the first visit ensures each same-value group is processed at most once across all BFS levels |
| What two threats does DNSSEC address, and what does it NOT address? | Prevents cache poisoning (forged records fail signature verification) and integrity violations; does NOT encrypt queries — eavesdropping on which domains are queried is still possible |
| What is the difference between DoH and DoT? | Both encrypt DNS queries. DoH wraps DNS in HTTPS on port 443 (hard to block, bypasses enterprise filters). DoT uses TLS on dedicated port 853 (distinguishable, can be firewall-blocked, but more transparent to network administrators) |

---

## Checklist
- [ ] Solved all problems (timed — 20 min each, no hints)
- [ ] Rewrote pattern skeleton from memory
- [ ] Stated time + space complexity aloud for each solution
- [ ] Traced at least 2 edge cases per problem
- [ ] Completed system design section
- [ ] Reviewed all 5 flashcards
- [ ] Logged unsolved problems to `knowledge-base/revision-log.md`
