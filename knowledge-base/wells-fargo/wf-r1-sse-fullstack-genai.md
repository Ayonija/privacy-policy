# Wells Fargo — Round 1 Online Assessment
**Role:** R-SWIFT Senior Software Engineer — Full Stack Java + AI/GenAI/Agentic AI  
**Platform:** HackerRank  
**Topics:** ML/AI Theory, Angular, React, Java, Data Structures, Python Concurrency, SQL

---

## Q1 — Transformer Architecture (FALSE Statement)

**Question:** Which of the following statements about the Transformer architecture is FALSE?

1. The self-attention mechanism allows each position to attend to all positions in the previous layer
2. Positional encodings are added to the input embeddings to retain information about token position
3. The feedforward networks in each layer apply the same transformation to each position separately
4. Causal masking in the decoder prevents attending to future tokens but isn't needed in the encoder

**Selected Answer:** Option 4

> ⚠️ **INCORRECT** — Option 4 is a TRUE statement. The FALSE statement is **Option 1**.

---

### Why Option 4 is TRUE (and therefore wrong to pick as the FALSE answer)

Causal masking IS used in the decoder to prevent attending to future tokens during autoregressive generation. The encoder uses full bidirectional attention — no causal masking needed since the entire input sequence is available. This accurately describes the standard Transformer (Vaswani et al., 2017).

### Why Option 1 is FALSE ✓

The statement claims self-attention allows "each position to attend to **all** positions." This is FALSE because:

1. **In the decoder**, self-attention uses **causal masking** — a position can only attend to itself and *preceding* positions. Future positions are masked out (score set to −∞ before softmax). So "all positions" is factually wrong for decoder self-attention.
2. The phrase "**in the previous layer**" is also misleading. Self-attention operates *within the same sequence representation* (Q, K, V all come from the same source). It doesn't "attend to a different layer" — it attends across all positions of the current representation.

---

### Interview Explanation

The Transformer (encoder-decoder) has two distinct attention patterns:

**Encoder — Bidirectional Self-Attention:**
- Every token attends to every other token (past AND future context)
- No masking (except padding masking)
- Used in: BERT, RoBERTa, sentence embeddings
- Best for: tasks needing full context — NER, classification, translation encoding

**Decoder — Causal (Masked) Self-Attention:**
- Each token can only attend to itself and tokens that came before it
- Causal mask: upper-triangular matrix of −∞ applied before softmax
- Prevents information leakage from future tokens during training
- Enables autoregressive generation (one token at a time at inference)
- Used in: GPT, LLaMA, Claude

**Decoder also has Cross-Attention:**
- Decoder queries attend to ALL encoder outputs (no masking here)
- This is how the decoder "reads" the encoded input during translation

**Feedforward Network (FFN):**
- Applied identically and independently to each position (position-wise)
- `FFN(x) = max(0, xW₁ + b₁)W₂ + b₂` — two linear layers with ReLU
- Same weights shared across all positions within a layer

**Positional Encodings:**
- Transformer has no inherent token-order awareness
- Sinusoidal fixed encodings (original paper) or learned encodings added to input embeddings
- Enable the model to distinguish "cat sat" from "sat cat"

---

### Cross-Questions

- *What is the time complexity of self-attention?* → O(n²·d) where n = sequence length, d = model dim. Quadratic scaling is the key bottleneck for long contexts.
- *Why does the decoder need causal masking during training?* → Without it, the model could "cheat" by attending to the correct next token during teacher-forcing, making training meaningless.
- *What's the difference between BERT (encoder-only), GPT (decoder-only), and T5 (encoder-decoder)?* → BERT sees full context (bidirectional), GPT generates autoregressively (causal), T5 uses both for seq2seq tasks.
- *How does multi-head attention differ from single-head?* → Multi-head runs h parallel attention heads on different linear projections, each learning different relationship patterns, then concatenates and projects.

---

## Q2 — Use Case of Generative AI (NOT a use case)

**Question:** Which of the following is *not* a use case of Generative AI?

Options: Image synthesis, Fraud detection, Text summarization, Code generation

**Selected Answer:** Fraud detection ✅

> **CORRECT**

---

### Why Fraud Detection is NOT Generative AI

Fraud detection is a **discriminative** task — it learns the boundary between fraudulent and legitimate transactions: P(fraud | transaction features). It classifies existing data, it does not generate new data.

Generative AI learns P(data) or P(data | condition) to *create* new content.

| Task | Type | Model Examples |
|------|------|---------------|
| Fraud detection | Discriminative | XGBoost, Random Forest, Neural Classifier |
| Image synthesis | Generative | DALL-E, Stable Diffusion, GANs |
| Text summarization | Generative | GPT-4, Claude, T5 |
| Code generation | Generative | Copilot, CodeLlama, Claude |

---

### Interview Explanation

**Generative AI** refers to models that can produce new artifacts — text, images, audio, code, video — by learning the underlying data distribution.

**Discriminative AI** learns to distinguish between classes. Given input X, output Y. No new data is created.

Fraud detection uses:
- Supervised learning on labeled transaction history (fraud vs. legitimate)
- Models: Logistic Regression, Gradient Boosting, Isolation Forest (anomaly detection)
- Features: transaction amount, merchant category, time-of-day, geo-velocity
- Class imbalance handled via SMOTE, class weights, or anomaly detection framing

**Where GenAI CAN help fraud detection (indirectly):**
- Generating synthetic fraud samples to address severe class imbalance
- LLMs for rule generation or summarizing fraud patterns for analysts
- But the core detection model remains discriminative

---

### STAR Answer (GenAI use at a financial institution)

**Situation:** Wells Fargo processes millions of transactions daily and needs to flag fraud in near-real-time.  
**Task:** Distinguish between using GenAI vs. discriminative AI for the fraud pipeline.  
**Action:** Built a two-stage pipeline — a discriminative classifier (XGBoost on transaction features) for real-time scoring, and a separate GenAI-based synthetic data generator (CTGAN) to augment underrepresented fraud patterns in training data.  
**Result:** Clear architectural separation: GenAI for data augmentation, discriminative AI for the actual fraud scoring. Reduced false negative rate by 15% after retraining on augmented data.

---

### Cross-Questions

- *Can GenAI be used in fraud detection?* → Yes, indirectly — for synthetic data generation, explanation generation, or analyst tooling. Not for the core classification.
- *What is the class imbalance problem in fraud detection?* → Fraud events are rare (0.1–2% of transactions). Models trained naively predict "not fraud" always. Solutions: SMOTE oversampling, undersampling, anomaly detection, cost-sensitive learning.
- *How would you deploy a fraud model at scale?* → Feature store for real-time features, model serving with low-latency (<50ms) inference, A/B testing new models, continuous retraining on new fraud patterns.

---

## Q3 — Decision Tree Algorithm Metric

**Question:** In a decision tree algorithm, which metric is commonly used to measure the impurity of a node?

Options: Standard deviation, Euclidean distance, Gini impurity, Cosine similarity

**Selected Answer:** Gini impurity ✅

> **CORRECT**

---

### Explanation

**Gini Impurity** measures the probability that a randomly chosen element from a node would be incorrectly classified if labeled randomly according to the node's class distribution:

```
Gini(D) = 1 - Σ pᵢ²
```

- pᵢ = proportion of class i in node D
- **Gini = 0**: perfectly pure node (all samples same class)
- **Gini = 0.5** (binary case): maximum impurity (50/50 split)

The CART (Classification and Regression Trees) algorithm uses Gini impurity to select the best split at each node — choosing the feature and threshold that minimizes the weighted average Gini of the two child nodes.

**Other metrics and their domains:**
| Metric | Used For | Why NOT for decision trees |
|--------|----------|--------------------------|
| Standard deviation | Regression trees (continuous targets) | Not for classification |
| Euclidean distance | K-Means clustering, KNN | Not an impurity measure |
| Cosine similarity | NLP, text similarity | Not an impurity measure |
| Entropy | Also used in decision trees (ID3, C4.5) | `−Σ pᵢ log₂(pᵢ)` |

**Gini vs. Entropy:**
- Gini: computationally cheaper (no log operations), range [0, 0.5] for binary
- Entropy: slightly higher sensitivity to class distribution changes, range [0, 1]
- In practice, results are nearly identical — both produce similar trees

---

### STAR Answer

**Situation:** Building a customer churn prediction model for a banking product, needing interpretable results for the product team.  
**Task:** Choose a model and splitting criterion that's both accurate and explainable.  
**Action:** Used a Decision Tree with Gini impurity as the splitting criterion. At each node, evaluated all feature-threshold combinations and selected the split minimizing `weighted_gini = (n_left/n) * gini_left + (n_right/n) * gini_right`. Applied cost-complexity pruning (ccp_alpha) to avoid overfitting.  
**Result:** Produced an interpretable tree where the product team could trace why a customer was predicted as churning — e.g., "balance < $500 AND no mobile app activity > 60 days."

---

### Cross-Questions

- *What is information gain?* → Information Gain = Entropy(parent) − weighted Entropy(children). Used by ID3 and C4.5 algorithms. CART uses Gini instead.
- *How do you prevent overfitting in a decision tree?* → Pruning (pre- or post-), max_depth, min_samples_split, min_samples_leaf, max_leaf_nodes, or cost-complexity pruning.
- *Why would you choose Random Forest over a single Decision Tree?* → Random Forest ensembles many decorrelated trees (via bagging + feature subsampling), dramatically reducing variance while maintaining low bias.

---

## Q4 — Optimal Trading Strategies

**Question:** Which AI technique is most suitable for training an AI agent to learn and execute optimal trading strategies?

Options: Supervised Learning, Generative AI, Reinforcement Learning, K-Means Clustering

**Selected Answer:** Reinforcement Learning ✅

> **CORRECT**

---

### Explanation

Reinforcement Learning is the natural fit because trading is a **sequential decision-making problem under uncertainty** — exactly what RL is designed for. There are no pre-labeled "correct actions," the agent learns by interacting with the market environment and receiving reward signals.

**RL Framework applied to trading:**

| RL Component | Trading Equivalent |
|---|---|
| **Agent** | The trading algorithm |
| **Environment** | Financial market |
| **State** | Price data, indicators, portfolio holdings, volatility |
| **Action** | Buy N shares, Sell N shares, Hold |
| **Reward** | Profit/Loss, risk-adjusted return (Sharpe ratio) |
| **Policy** | Learned mapping: state → optimal action |
| **Episode** | A trading period (day, week, quarter) |

The agent maximizes **cumulative discounted reward** over time — finding strategies that maximize long-term return, not just immediate gains.

**Why others don't fit:**
- **Supervised Learning:** Requires labeled ground-truth actions. What is the "correct" trade for every market state? This is unknowable a priori.
- **Generative AI:** Creates content (text, images). Doesn't make sequential decisions in an environment.
- **K-Means Clustering:** Groups similar data points. Doesn't learn to act or optimize a policy.

**Real-world RL algorithms used in trading:**
- **DQN (Deep Q-Network):** Discrete action spaces (buy/sell/hold)
- **PPO (Proximal Policy Optimization):** Stable policy gradient for continuous actions
- **SAC (Soft Actor-Critic):** Off-policy, handles continuous action spaces (order sizing)

---

### STAR Answer

**Situation:** A fintech team needed to automate options trading for a specific asset class, replacing manual rule-based strategies.  
**Task:** Design an AI system that adapts to changing market conditions and learns optimal entry/exit strategies.  
**Action:** Implemented a PPO-based RL agent. State space: 60-day price window + technical indicators (RSI, MACD, Bollinger Bands) + current portfolio state. Action space: buy/sell/hold with position sizing. Reward: risk-adjusted Sharpe ratio. Trained on 10 years of historical data with random episode sampling to avoid look-ahead bias.  
**Result:** Agent outperformed the rule-based baseline by 12% annualized return on out-of-sample test data, while maintaining similar drawdown characteristics.

---

### Cross-Questions

- *What are the challenges of applying RL to financial markets?* → Non-stationarity (market regimes change), sparse/delayed rewards, transaction costs as negative rewards, overfitting to historical periods, exploration risk (losing real money during learning).
- *What is the exploration-exploitation tradeoff?* → Agent must balance exploring new strategies (risk) vs. exploiting known profitable strategies. Handled via ε-greedy, entropy regularization (in PPO/SAC), or UCB.
- *How do you prevent an RL trading agent from overfitting to historical data?* → Walk-forward validation, out-of-sample testing on unseen market periods, domain randomization, transaction cost penalization, regularization in the neural network policy.

---

## Q5 — Angular: Back-end Communication

**Question:** An Angular app adds employee data to a database via a back-end service using JWT authentication. Which option should replace `/*INSERT CODE HERE*/` in `employee.service.ts`?

**Selected Answer:** Option with `http.post(...)` returning `Observable<any>` with `headers: { 'authorization': '${auth_token}' }` ✅

> **CORRECT**

---

### Why This Option is Correct

```typescript
constructor(private http: HttpClient) {}

addEmployee(employee: { name: number, jobTitle: string, experience: number }): Observable<any> {
  return this.http.post('api.demo.com/addEmployee', employee, {
    'headers': { 'authorization': '${auth_token}' }
  });
}
```

**Four key correctness criteria:**

1. **`HttpClient`** (not `Http`) — `HttpClient` is the modern Angular module (`@angular/common/http`, Angular 4.3+). The older `Http` module is deprecated and removed in Angular 13+.

2. **`http.post()`** (not `http.get()`) — The operation is sending/storing data. HTTP semantics:
   - GET: retrieve, no side effects, idempotent
   - POST: create/submit data, has side effects (writes to DB)

3. **Headers structure** — JWT must be inside `{ headers: { ... } }`. Placing `authorization` at the root options object level is incorrect Angular HttpClient syntax.

4. **`Observable<any>` return type** — Angular's `HttpClient` returns RxJS Observables, not Promises. The component subscribes to this Observable.

**Why other options fail:**
- Option 1: `authorization` placed at root of options object (missing `headers:` wrapper) → Angular ignores it, request goes without auth header
- Option 3: Uses `http.get()` → wrong HTTP verb for data creation
- Option 4: Uses legacy `Http` class instead of `HttpClient`

---

### Production-Grade Pattern

```typescript
import { HttpClient, HttpHeaders } from '@angular/common/http';
import { Injectable } from '@angular/core';
import { Observable, throwError } from 'rxjs';
import { catchError } from 'rxjs/operators';

@Injectable({ providedIn: 'root' })
export class EmployeeService {
  private apiUrl = 'https://api.demo.com';

  constructor(private http: HttpClient) {}

  addEmployee(employee: Employee): Observable<Employee> {
    const headers = new HttpHeaders({
      'Authorization': `Bearer ${this.getToken()}`,
      'Content-Type': 'application/json'
    });
    return this.http.post<Employee>(`${this.apiUrl}/addEmployee`, employee, { headers })
      .pipe(catchError(this.handleError));
  }
}
```

---

### STAR Answer

**Situation:** Building an employee management Angular app where the back-end requires JWT authentication for all write operations.  
**Task:** Implement the `addEmployee` service method that posts employee data with proper auth headers.  
**Action:** Used `HttpClient.post()` with the JWT token wrapped in the `headers` option. Added an HTTP Interceptor to automatically attach the token to all outgoing requests rather than manually adding headers in every service method.  
**Result:** Clean, DRY implementation — the interceptor handles token injection globally, and individual service methods focus on business logic. Token expiry handled in the interceptor with a refresh flow.

---

### Cross-Questions

- *What are Angular HTTP Interceptors?* → Classes implementing `HttpInterceptor` that can inspect/modify every outgoing request and incoming response. Used for: auth token injection, logging, error handling, retry logic.
- *How would you handle 401 Unauthorized responses?* → In the interceptor, catch 401s, attempt token refresh, retry the failed request with the new token.
- *What is the difference between `Observable` and `Promise` in Angular HTTP?* → Observables are lazy (don't execute until subscribed), cancellable, and support operators (map, filter, retry). Promises are eager and non-cancellable.
- *How do you handle loading states and errors in a component using this service?* → `async` pipe in template, or manual subscribe with loading/error flags, or NgRx for state management.

---

## Q6 — Angular and Styles (ViewEncapsulation)

**Question:** An Angular component has styles applied with a class. The developer wants to apply the same styling to multiple elements of the same class across the app. Which `encapsulation` option achieves this?

Options: ViewEncapsulation.None, Emulated, Native, ShadowDom

**Selected Answer:** `encapsulation: ViewEncapsulation.None` ✅

> **CORRECT**

---

### Explanation

Angular's `ViewEncapsulation` determines how component CSS is scoped:

| Mode | Mechanism | Styles Affect |
|------|-----------|---------------|
| `Emulated` (default) | Angular adds unique attribute selectors (`_ngcontent-xxx`) to CSS rules | Component's own DOM only |
| `None` | No scoping — styles added to `<head>` as global CSS | **All matching elements in entire app** |
| `ShadowDom` | Native browser Shadow DOM | Strict isolation — styles cannot leak in or out |

**Why `ViewEncapsulation.None` is correct:**

With `None`, Angular does NOT add attribute-based scoping. The style `.view { background: green; }` becomes a global CSS rule, applying to every element with class `.view` across the entire application — exactly what the developer wants.

```typescript
import { ViewEncapsulation } from '@angular/core';

@Component({
  selector: 'app-view',
  templateUrl: './view.component.html',
  styles: [`.view { background: green; }`],
  encapsulation: ViewEncapsulation.None  // styles become global
})
class ViewComponent {
  @Input() title: string;
}
```

**Default `Emulated` behavior for comparison:**
Angular transforms `.view { ... }` to `.view[_ngcontent-abc-c123] { ... }` — scoped only to this component's DOM, even if other components have elements with class `.view`.

**Trade-off to mention in interviews:**
`ViewEncapsulation.None` can cause unintended style pollution across unrelated components. Prefer targeted CSS custom properties (CSS variables) or a global stylesheet for truly shared styles, reserving `None` for UI library components with intentional global styles.

---

### Cross-Questions

- *When would you use ShadowDom vs None?* → ShadowDom for web components / component libraries where strict isolation is needed (styles from the host app should not break the component). None for legacy apps or components explicitly designed to inject global styles.
- *What is the Angular default and why?* → `Emulated` — gives the appearance of encapsulation without requiring native Shadow DOM browser support; works everywhere including IE11 (historically).
- *How do you share styles across components without ViewEncapsulation.None?* → CSS custom properties (variables) in `:root`, global `styles.css`, Angular Material theming, or a shared SCSS mixin file.

---

## Q7 — React: Welcome (Output)

**Question:** What is the output of this React code?

```jsx
import React from "react";

function App() {
  return (
    <div>
      <p>Hi</p>
      <p>Welcome to Hackerrank</p>
    </div>
    <div>
      <p>This is a react problem!</p>
    </div>
  );
}

export default App;
```

Options: "Hi / Welcome to Hackerrank", all three lines, "This is a react problem!", or generates an error

**Selected Answer:** "This code will generate an error" ✅

> **CORRECT**

---

### Explanation

React components must return a **single root JSX element**. This function attempts to return **two sibling `<div>` elements** with no common parent. This violates JSX syntax rules and throws:

```
SyntaxError: Adjacent JSX elements must be wrapped in an enclosing tag.
Did you want a JSX fragment <>...</>?
```

The error occurs at **compile time** (Babel/TypeScript transpilation), before the code even reaches the browser.

**How to fix:**

Option 1 — React Fragment (no extra DOM node):
```jsx
function App() {
  return (
    <>
      <div><p>Hi</p><p>Welcome to Hackerrank</p></div>
      <div><p>This is a react problem!</p></div>
    </>
  );
}
// Output: Hi / Welcome to Hackerrank / This is a react problem!
```

Option 2 — Wrapper `<div>`:
```jsx
function App() {
  return (
    <div>
      <div>...</div>
      <div>...</div>
    </div>
  );
}
```

**Why Fragment is preferred over wrapper div:**
- Doesn't add an extra DOM node (cleaner HTML)
- Avoids breaking CSS flexbox/grid layouts where an extra wrapper would alter structure
- `<React.Fragment key={id}>` supports the `key` prop for list items

---

### Cross-Questions

- *What is a React Fragment and when do you use it?* → `<React.Fragment>` or `<>` groups multiple elements without adding a DOM node. Essential in table rows (`<tr>` can't have a `<div>` wrapper) and list items needing keys.
- *What is the Virtual DOM?* → React's in-memory representation of the real DOM. On state change, React re-renders to a new VDOM, diffs it against the previous (reconciliation), and only applies minimal real DOM changes (batched updates).
- *What's the difference between React.createElement and JSX?* → JSX is syntactic sugar. `<div className="x">` compiles to `React.createElement('div', { className: 'x' })`.
- *What triggers a React re-render?* → `setState`, `useState` setter, `useReducer` dispatch, new props from parent, `forceUpdate` (class components), or context value changes.

---

## Q8 — Empty Collection (Java Stack + Queue)

**Question:** Which code displays an empty (`{}`) collection when executed?

**Selected Answer:** Option with `i = i - 1` in the first (transfer) loop but NOT in the second (drain) loop

> ⚠️ **INCORRECT** — The selected option does NOT produce an empty collection.

**Correct Answer:** The option with `i = i - 1` in **both** loops.

---

### Analysis of Selected (Incorrect) Option

```java
Stack<Integer> stack = new Stack<>();
Queue queue = new LinkedList();

for (int i = 0; i < 10; i++)
    stack.add(i);
// stack = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

// Loop 1: i = i-1 trick — drains stack completely into queue
for (int i = 0; i < stack.size(); i++) {
    int n = stack.remove(0);  // removes from index 0 (bottom of stack)
    queue.add(n);
    i = i - 1;                // i → -1, then i++ → i = 0 again
    // Runs until stack.size() = 0 (condition: 0 < 0 is false)
}
// queue = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

// Loop 2: NO i = i-1 — only partially drains queue
for (int i = 0; i < queue.size(); i++) {
    int n = (int) queue.remove();  // i increases AND size decreases
}
// Trace: i=0,size=10 → i=1,size=9 → i=2,size=8 → ... → i=5,size=5 → STOPS
// queue = [5, 6, 7, 8, 9]  ← NOT EMPTY
```

**The drain loop stops halfway** because `i` increases at the same rate that `queue.size()` decreases. They meet in the middle.

---

### Correct Option (Both loops use `i = i - 1`)

```java
Stack<Integer> stack = new Stack<>();
Queue queue = new LinkedList();

for (int i = 0; i < 10; i++)
    stack.add(i);

// Transfer: same as above — drains stack completely
for (int i = 0; i < stack.size(); i++) {
    int n = stack.remove(0);
    queue.add(n);
    i = i - 1;
}
// queue = [0 .. 9]

// Drain: i = i - 1 keeps i at 0; runs until queue.size() = 0
for (int i = 0; i < queue.size(); i++) {
    int n = (int) queue.remove();
    i = i - 1;  // i → -1, then i++ → i = 0 again
    // Loop condition: 0 < queue.size(); terminates when queue is empty
}
// queue = []  ← EMPTY ✓

System.out.println(queue);  // prints []
```

---

### The `i = i - 1` Trick Explained

This is an "infinite loop until empty" pattern disguised as a for loop:
- `i = i - 1` followed by the for loop's `i++` results in `i` always returning to 0
- The loop termination condition checks `queue.size()` which decreases each iteration
- Loop exits ONLY when `queue.size() == 0` (condition `0 < 0` is false)
- Effectively equivalent to: `while (!queue.isEmpty()) { queue.remove(); }`

---

### Subtlety: `stack.remove(0)` vs `stack.pop()`

`Stack<T>` in Java extends `Vector<T>`. `remove(0)` removes the element at **index 0** (the bottom of the stack), not the top. For proper LIFO stack behavior, use `stack.pop()` or `stack.peek()`. This matters for correctness in real code — the question uses `remove(0)` intentionally to drain from the bottom.

---

### Cross-Questions

- *What is the time complexity of this code?* → O(n) total — each element is touched a constant number of times.
- *Why is `Stack<T>` in Java considered a legacy class?* → It extends `Vector` (synchronized, thread-safe), which adds unnecessary overhead. Prefer `ArrayDeque<T>` as a stack (`push`, `pop`, `peek`) for better performance.
- *What's the difference between `Queue.remove()` and `Queue.poll()`?* → `remove()` throws `NoSuchElementException` on empty queue; `poll()` returns `null`. Prefer `poll()` for safe usage.
- *How would you rewrite loop 2 idiomatically?* → `while (!queue.isEmpty()) { queue.poll(); }` — clear intent, no off-by-one risk.

---

## Q9 — Java Generics

**Question:** What is the result of compiling/running this code?

```java
public class Generic<T> {
    private T value;
    public Generic(T value) { this.value = value; }
    public T getValue() { return value; }
}

import java.util.ArrayList;
public class Main {
    public static void main(String args[]) {
        ArrayList<Generic> g = new ArrayList<>();  // ← raw type
        Generic<?> g1 = new Generic<>(10);         // Generic<Integer>
        Generic<?> g2 = new Generic<>("Hello");    // Generic<String>
        g.add(g1);
        g.add(g2);
        int i    = g.get(0).getValue();   // ← compile error
        String s = g.get(1).getValue();   // ← compile error
        System.out.println(s);
        System.out.println(i);
    }
}
```

Options: compile-time error on retrieval, compile-time error creating with int, prints "Hello 10", run-time error from mixed types in ArrayList

**Selected Answer:** "Compile-time error while retrieving the values from array list and assigning them to their respective datatypes" ✅

> **CORRECT**

---

### Explanation

The root cause is **Java type erasure** combined with a **raw type**.

`ArrayList<Generic>` uses the raw type `Generic` (no type parameter). Due to type erasure:
- At runtime, all generic type information is removed
- `Generic<T>.getValue()` becomes `getValue()` returning `Object`
- `g.get(0)` returns a raw `Generic` instance
- `g.get(0).getValue()` returns `Object`

Assigning `Object` to `int` → **compile-time error**: *"incompatible types: Object cannot be converted to int"*  
Assigning `Object` to `String` → **compile-time error**: *"incompatible types: Object cannot be converted to String"*

**Why the other options are wrong:**

- *"Compile-time error creating Generic with int value"* — WRONG. Java **autoboxing** converts `int 10` to `Integer` automatically. `new Generic<>(10)` creates `Generic<Integer>` without issue. Generics don't support primitives directly, but autoboxing handles this transparently.

- *"Hello 10"* — WRONG. Code doesn't compile, so it never runs.

- *"Run-time error from mixed types in ArrayList"* — WRONG. `ArrayList<Generic>` (raw) erases to `ArrayList`, which accepts any `Generic<?>` object. No runtime check prevents adding `Generic<Integer>` and `Generic<String>` to the same list. The error is at compile time, not runtime.

---

### Type Erasure Deep Dive

```java
// At compile time (what you write):
ArrayList<Generic<Integer>> typed = new ArrayList<>();

// At runtime (after type erasure):
ArrayList typed = new ArrayList();  // same bytecode as raw type
```

Erasure means:
- No generic type info is retained in `.class` files (except in metadata)
- `instanceof Generic<Integer>` is illegal — you can only check `instanceof Generic`
- Arrays of generic types are problematic: `new Generic<String>[10]` is illegal

**Fix for the original code:**
```java
// Option 1: Use proper generic type
ArrayList<Generic<?>> g = new ArrayList<>();
int i = (int)(Integer) g.get(0).getValue();  // safe cast with instanceof check

// Option 2: Use typed ArrayList
ArrayList<Generic<Object>> g = new ArrayList<>();
```

---

### Cross-Questions

- *What is type erasure in Java?* → Generic type parameters are replaced with their bounds (or Object if unbounded) at compile time. The JVM has no knowledge of generics — they exist only at the source level for compile-time type safety.
- *What is the difference between `Generic<?>` and `Generic<Object>`?* → `Generic<?>` is a wildcard — it accepts any `Generic<X>` (read-only, you can't add to it). `Generic<Object>` only accepts `Generic<Object>` specifically.
- *What is PECS (Producer Extends, Consumer Super)?* → `<? extends T>` for reading (covariant), `<? super T>` for writing (contravariant). E.g., `List<? extends Number>` lets you read Numbers but not add.
- *Can you create an array of a generic type?* → No — `new T[n]` is illegal. Use `(T[]) new Object[n]` with an unchecked cast, or use `ArrayList<T>` instead.

---

## Q10 — Time Complexity of Linked List Insertion

**Question:** Consider a linked list of n elements. What is the time taken to insert an element at a position that is pointed to by some pointer?

Options: O(1), O(log₂n), O(n), O(nlog₂n)

**Selected Answer:** O(1) ✅

> **CORRECT**

---

### Explanation

The critical qualifier is **"pointed to by some pointer"** — you already have a direct reference to the node at the insertion point. No traversal is needed.

Linked list insertion at a known position (given a node reference `prev`):

```java
Node newNode = new Node(value);   // O(1)
newNode.next = prev.next;         // O(1)
prev.next = newNode;              // O(1)
```

Three pointer operations → **O(1) total**, regardless of list size.

**Contrast: finding the position first costs O(n)**

```java
// Traverse to find the kth node: O(n)
Node curr = head;
for (int i = 0; i < k; i++) curr = curr.next;

// Insert at curr: O(1)
// Total: O(n) + O(1) = O(n)
```

---

### Complexity Comparison Table

| Data Structure | Insert at known position | Insert at beginning | Insert at end | Search |
|---|---|---|---|---|
| **Linked List (with ptr)** | **O(1)** | O(1) | O(1) with tail ptr | O(n) |
| Array | O(n) — shift elements | O(n) | O(1) amortized | O(n) or O(log n) sorted |
| BST | O(log n) average | N/A | N/A | O(log n) average |
| Hash Table | O(1) average | N/A | N/A | O(1) average |

---

### STAR Answer

**Situation:** Designing a real-time order book system for a trading platform where orders need to be inserted at specific price-sorted positions.  
**Task:** Achieve O(1) insertion once the correct price level is identified.  
**Action:** Used a doubly linked list to represent price levels. A separate HashMap mapped price → list node (pointer). When a new order arrived at a known price, insertion was O(1) using the stored node reference, bypassing O(n) traversal.  
**Result:** Order insertion throughput of 500K ops/sec achieved with constant-time insertion, compared to O(n) baseline using sorted arrays.

---

### Cross-Questions

- *What's the time complexity of deletion at a known position in a doubly linked list?* → O(1) — just update prev/next pointers. (In singly linked list, you need the *previous* node; if given a pointer only to the node itself, it's O(n) to find prev, unless using the "copy successor" trick.)
- *Why is array insertion O(n)?* → All elements after the insertion index must be shifted by one position to make room.
- *What is a skip list and how does it achieve O(log n) insertion?* → A probabilistic data structure with multiple linked list layers. Each higher layer skips over more elements, enabling binary-search-like traversal to the insertion point in O(log n).

---

## Q11 — Values in a Hash Table

**Question:** Keys and values are integers. Iterating over key-value pairs and printing Key=a, Value=b — which could be a valid output?

**Selected Answer:** Key=1 Value=10; Key=1 Value=11; Key=2 Value=10; Key=3 Value=100

> ⚠️ **INCORRECT** — Hash tables have UNIQUE keys. Key=1 appears TWICE in the selected output, which is impossible.

**Correct Answer:** Key=1 Value=10; Key=2 Value=10; Key=3 Value=10; Key=4 Value=10

---

### Explanation

**Fundamental hash table contract: keys are unique.**

If you insert `put(1, 10)` then `put(1, 11)`, the second call **overwrites** the first. The hash table will have Key=1 → Value=11 (only one entry for key 1). You will never see Key=1 printed twice when iterating.

**Evaluating each option:**

| Option | Validity | Reason |
|--------|----------|--------|
| Key=1×2, Key=2, Key=3 | ❌ Invalid | Key=1 appears twice |
| Key=1, Key=2×2, Key=3 | ❌ Invalid | Key=2 appears twice |
| **Key=1, Key=2, Key=3, Key=4** | ✅ **Valid** | All unique keys; values can all be 10 |
| Key=1, Key=2, Key=3×2 | ❌ Invalid | Key=3 appears twice with different values |

The valid output has **four distinct keys** (1, 2, 3, 4) each mapping to value 10. Values can repeat freely — only key uniqueness is enforced.

---

### Hash Table Internals

**Hash Collision ≠ Duplicate Key:**
- **Hash collision:** Two DIFFERENT keys hash to the same bucket (e.g., `key=1` and `key=11` both land in bucket 1 for a small table)
- These are still separate entries with separate keys — iteration shows both
- Handled by chaining (LinkedList/BST per bucket) or open addressing (probing)
- **Duplicate key:** Same key inserted twice — second value OVERWRITES the first. This is NOT a collision.

**Java HashMap iteration (no order guarantee):**
```java
HashMap<Integer, Integer> map = new HashMap<>();
map.put(1, 10);
map.put(2, 10);
map.put(3, 10);
map.put(4, 10);
for (Map.Entry<Integer, Integer> e : map.entrySet()) {
    System.out.println("Key=" + e.getKey() + ", Value=" + e.getValue());
}
// Valid output — order may vary, but each key appears EXACTLY once
```

---

### Cross-Questions

- *How does Java's HashMap handle hash collisions?* → Separate chaining: each bucket is a singly linked list. In Java 8+, if a chain exceeds 8 entries, it converts to a **balanced Red-Black Tree** (O(log n) worst case instead of O(n)).
- *What's the time complexity of HashMap get/put in Java?* → O(1) average, O(log n) worst case (Java 8+, when using tree bins). O(n) worst case in Java 7 (all keys collide, linear chain).
- *What makes a good hash function for use as a HashMap key?* → Deterministic, uniform distribution across buckets, fast to compute, uses all bits of the key. Java's `Object.hashCode()` can be overridden; always override `equals()` when you override `hashCode()`.
- *What is the difference between HashMap, LinkedHashMap, and TreeMap?* → HashMap: unordered, O(1); LinkedHashMap: insertion-order or access-order, O(1); TreeMap: sorted by key, O(log n).

---

## Q12 — Synchronization Puzzle (Python Threading with Lock)

**Question:** A `SharedCounter` uses `threading.Lock()` in its `increment` method. 5 workers each do 10,000 increments. What does it print?

Options: guaranteed 50000, may print less, may print more, may raise exception

**Selected Answer:** "It is guaranteed to print 50000" ✅

> **CORRECT**

---

### Explanation

The `SharedCounter` uses Python's `threading.Lock()` correctly via the `with` context manager:

```python
def increment(self, delta=1):
    with self._value_lock:      # acquire lock
        self.value += delta     # critical section
    # lock released automatically (even on exception)
```

**Why this guarantees 50000:**
- Only one thread can execute `self.value += delta` at a time
- The `with` statement: acquires the lock, executes the block, releases the lock
- All 5 workers serialize through the lock → no lost updates
- 5 workers × 10,000 increments = **50,000 exactly**

**What would go wrong WITHOUT the lock:**

```python
def increment(self):
    self.value += delta  # NOT atomic!
```

`self.value += delta` compiles to three bytecode instructions:
1. `LOAD_ATTR` (read `self.value`)
2. `BINARY_ADD` (add delta)
3. `STORE_ATTR` (write back to `self.value`)

The GIL can be released between any of these instructions, allowing another thread to read the stale value before the first thread writes back → **race condition → lost updates → final count < 50000**.

**Why the GIL alone does NOT protect this:**
The GIL prevents two Python threads from executing Python bytecode simultaneously, but it can be released between bytecode instructions. The three-instruction `+=` sequence is NOT atomic at the bytecode level.

---

### Python Threading Primitives

| Primitive | Use Case |
|---|---|
| `Lock()` | Mutual exclusion — only one thread at a time |
| `RLock()` | Reentrant lock — same thread can acquire multiple times |
| `Semaphore(n)` | Allow up to n threads concurrently |
| `Event()` | One thread signals others (e.g., "data ready") |
| `Condition()` | Wait/notify on a condition variable (producer-consumer) |
| `Queue` | Thread-safe FIFO — preferred for inter-thread communication |

---

### Cross-Questions

- *Would using `threading.RLock()` change the behavior here?* → No difference for this single-level acquisition. RLock is needed when `increment` might call another method that also acquires the same lock (reentrant scenario).
- *What is a deadlock and how would one occur here?* → Thread A holds lock 1, waits for lock 2; Thread B holds lock 2, waits for lock 1. Prevented by always acquiring locks in a consistent order, or using `threading.Condition`.
- *What is thread starvation?* → A thread continuously fails to acquire a lock because other threads keep acquiring it first. Python's `threading.Lock()` does not guarantee fairness.
- *How would you make `get_value()` + `increment()` a compound atomic operation?* → Expose a combined method that acquires the lock once for both operations, or use a `Condition` variable.

---

## Q13 — GIL Conundrum (Python Multiprocessing for CPU-bound)

**Question:** `count_up_to(n)` loops n times incrementing a counter (CPU-bound). With GIL in effect, which `main()` version DECREASES total execution time?

Options: threading (2 threads), multiprocessing (2 processes), sequential calls, threading + sleep(1)

**Selected Answer:** `import multiprocessing` with two `Process` instances ✅

> **CORRECT**

---

### Explanation

`count_up_to` is **CPU-bound** (tight Python loop, no I/O, no blocking). This is the exact scenario where the GIL is most harmful for threads and where multiprocessing shines.

**Why `threading` DOESN'T help for CPU-bound tasks:**
- The GIL allows only one Python thread to run Python bytecode at a time
- Two threads alternate on a single CPU core — net effect is sequential
- Context switching overhead can even make it SLOWER than sequential
- GIL is only released for: I/O operations, C extensions that explicitly release it (NumPy), every ~100 bytecodes (cooperative switching)

**Why `multiprocessing` DOES help:**
- Each `Process` spawns a separate OS process with its **own Python interpreter** and its **own GIL**
- Two processes run on two different CPU cores **simultaneously** (true parallelism)
- Total time ≈ time for one `count_up_to(n)` call (both run in parallel)

**Threading + sleep(1):** Adding sleep makes it WORSE — introduces a 1-second forced wait for no reason.

---

### Threading vs Multiprocessing Decision Guide

| Scenario | Best Choice | Reason |
|---|---|---|
| CPU-bound (pure Python) | **multiprocessing** | Bypasses GIL, true parallelism |
| CPU-bound (NumPy/C ext) | threading or multiprocessing | NumPy releases GIL |
| I/O-bound (network, disk) | **threading** or asyncio | GIL released during I/O wait |
| Many concurrent I/O tasks | **asyncio** | Single-threaded event loop, no GIL contention |
| Inter-process shared state | threading or Manager | multiprocessing requires IPC overhead |

---

### STAR Answer

**Situation:** A data pipeline processing 1M records required parallel computation in Python — normalizing and transforming each record was CPU-intensive.  
**Task:** Reduce processing time from 40 seconds on a single core.  
**Action:** Replaced `threading.Thread` with `multiprocessing.Pool.map()`. Divided the 1M records into 8 chunks (matching CPU core count). Each worker process ran independently without GIL contention.  
**Result:** Processing time reduced from 40s to 7s (5.7× speedup on 8 cores). Added `chunksize` tuning to reduce IPC overhead — smaller chunks meant more communication, larger chunks reduced parallelism granularity.

---

### Cross-Questions

- *What is the GIL and why does CPython have it?* → Global Interpreter Lock ensures only one thread executes Python bytecode at a time. It simplifies CPython's memory management (reference counting is not thread-safe without a lock). PyPy and other implementations have different approaches.
- *What Python libraries bypass the GIL?* → NumPy (C extensions release GIL during computation), Cython (`nogil` blocks), `concurrent.futures.ProcessPoolExecutor`, `multiprocessing`.
- *When would you choose `asyncio` over `multiprocessing`?* → Asyncio for I/O-bound workloads with high concurrency (thousands of simultaneous network requests). Multiprocessing for CPU-bound parallel computation. Threading sits in between.
- *What is the overhead of `multiprocessing` vs `threading`?* → Process creation is slower (fork/spawn), each process has separate memory (no sharing without IPC), serialization cost for data passed between processes (pickling).

---

## Q14 — SQL: Downloaded File Types Report

**Question:** Write a MySQL query returning file download analytics for June 2022. Columns: `mime`, `extension` (distinct, sorted, comma-separated), `files` (count), `total` (MiB or GiB). Sorted by total size descending.

**Schema:** `events(dt VARCHAR(19), mime VARCHAR(255), filename VARCHAR(255), filesize INT)`

**Written Answer:**

```sql
SELECT
    mime,
    GROUP_CONCAT(
        DISTINCT SUBSTRING_INDEX(filename, '.', -1)
        ORDER BY SUBSTRING_INDEX(filename, '.', -1) ASC
        SEPARATOR ','
    ) AS extension,
    COUNT(*) AS files,
    CASE
        WHEN SUM(filesize) >= 1024 * 1024 * 1024
            THEN CONCAT(ROUND(SUM(filesize) / (1024 * 1024 * 1024), 2), ' GiB')
        ELSE
            CONCAT(ROUND(SUM(filesize) / (1024 * 1024), 2), ' MiB')
    END AS total
FROM events
WHERE dt >= '2022-06-01'
  AND dt < '2022-07-01'
GROUP BY mime
ORDER BY SUM(filesize) DESC;
```

**Verification:** ✅ CORRECT — matches expected output

---

### Query Component Breakdown

**`SUBSTRING_INDEX(filename, '.', -1)`**
Extracts file extension:
- `SUBSTRING_INDEX(str, delim, count)` — if count is negative, returns everything after the last occurrence of `delim`
- `'report.final.pdf'` → `'pdf'`
- `'archive.tar.gz'` → `'gz'`

**`GROUP_CONCAT(DISTINCT ... ORDER BY ... SEPARATOR ',')`**
Aggregates multiple extension values per group:
- `DISTINCT` removes duplicate extensions within a MIME group
- `ORDER BY ... ASC` sorts them alphabetically before concatenation
- `SEPARATOR ','` joins with comma (default is `,` but explicit is clearer)
- Result for `video/mpeg`: `'mp3,mpeg'` (alphabetically sorted)

**`CASE WHEN SUM(filesize) >= 1024³ THEN ... GiB ELSE ... MiB`**
Human-readable size formatting:
- 1 GiB = 1024³ = 1,073,741,824 bytes
- 1 MiB = 1024² = 1,048,576 bytes
- `ROUND(..., 2)` → 2 decimal places
- `CONCAT(..., ' GiB')` → appends unit string

**Date filter: `dt >= '2022-06-01' AND dt < '2022-07-01'`**
- Covers all of June 2022 (including times on June 30)
- Using `<` for the upper bound handles the `dt` column being a datetime string
- Alternative: `YEAR(dt) = 2022 AND MONTH(dt) = 6` (less index-friendly)

**`GROUP BY mime` + `ORDER BY SUM(filesize) DESC`**
- Groups rows by MIME type (aggregation scope)
- Orders final result by total byte size, largest first
- Note: MySQL requires re-computing `SUM(filesize)` in `ORDER BY` since you can't alias aggregates in the same `SELECT`

---

### Expected Output (from test)

| mime | extension | files | total |
|------|-----------|-------|-------|
| text/plain | txt | 2 | 1.67 GiB |
| image/jpeg | jpeg | 3 | 911.15 MiB |
| audio/mpeg3 | mp3 | 1 | 858.38 MiB |
| video/mpeg | mp3,mpeg | 2 | 486.59 MiB |

---

### STAR Answer

**Situation:** HackerAd advertising network needed monthly storage analytics to identify which MIME types were consuming the most bandwidth for capacity planning.  
**Task:** Write a SQL report query returning MIME-level aggregates with human-readable sizes.  
**Action:** Used `GROUP BY mime` with `GROUP_CONCAT(DISTINCT ...)` to aggregate extensions per type. Applied a `CASE/WHEN` expression to convert bytes to MiB/GiB dynamically. Filtered June 2022 events using range comparison on the datetime string column.  
**Result:** Query returned 14 MIME-type rows in 1 result set, revealing `text/plain` as the largest category at 1.67 GiB, and `video/x-msvideo` as smallest at 47.77 MiB — actionable data for storage tier decisions.

---

### Cross-Questions

- *What's the difference between WHERE and HAVING?* → `WHERE` filters individual rows BEFORE aggregation. `HAVING` filters groups AFTER aggregation. Example: `HAVING COUNT(*) > 5` filters groups with fewer than 5 files.
- *How would you add a HAVING clause to filter MIME types with more than 2 files?* → Add `HAVING COUNT(*) > 2` after `GROUP BY mime`.
- *How would you optimize this query for a table with 100M rows?* → Add composite index on `(dt, mime, filesize)` for the WHERE + GROUP BY + SUM. Consider partitioning the table by month. Use `EXPLAIN` to verify index usage.
- *How would you write this in PostgreSQL?* → Replace `GROUP_CONCAT` with `STRING_AGG(DISTINCT SPLIT_PART(filename, '.', -1) ORDER BY ... , ',')` — syntax differs but concept is the same.
- *What if a filename has no extension?* → `SUBSTRING_INDEX('noextension', '.', -1)` returns the whole string `'noextension'`. Add `NULLIF` or `CASE` to handle filenames without dots gracefully.

---

## Summary: Answer Correctness

| # | Topic | Selected | Correct? |
|---|-------|----------|----------|
| Q1 | Transformer Architecture (FALSE stmt) | Option 4 | ⚠️ Option 1 is the FALSE one |
| Q2 | Gen AI Use Case | Fraud detection | ✅ |
| Q3 | Decision Tree Metric | Gini impurity | ✅ |
| Q4 | Optimal Trading Strategies | Reinforcement Learning | ✅ |
| Q5 | Angular HTTP | http.post with headers | ✅ |
| Q6 | Angular ViewEncapsulation | ViewEncapsulation.None | ✅ |
| Q7 | React Output | Generates an error | ✅ |
| Q8 | Empty Collection (Java) | Loop without dual `i=i-1` | ⚠️ Correct option needs `i=i-1` in BOTH loops |
| Q9 | Java Generics | Compile error on retrieval | ✅ |
| Q10 | Linked List Insert Time | O(1) | ✅ |
| Q11 | Hash Table Valid Output | Duplicate key=1 | ⚠️ Correct is unique-key option (1,2,3,4 → 10) |
| Q12 | Sync Puzzle | Guaranteed 50000 | ✅ |
| Q13 | GIL Conundrum | multiprocessing | ✅ |
| Q14 | SQL File Report | Written query | ✅ |

**Questions to review before next round:** Q1 (Transformer masking), Q8 (loop trace), Q11 (hash table uniqueness)
