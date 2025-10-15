# **Lab 1 — Simulated Annealing for the Multi-Dimensional Knapsack Problem**

This implementation addresses the **multi-dimensional, multi-knapsack optimization problem**, a generalization of the classic **0–1 Knapsack Problem**.  
In this variant, there are multiple knapsacks, each with its own capacity constraints across several dimensions (e.g., weight, volume, cost).  
Each item has a **value** and a **multi-dimensional weight vector**, and must be assigned to at most one knapsack (or left unassigned).  
The goal is to **maximize the total value** of items placed in the knapsacks while satisfying all capacity constraints.

Because this problem is **NP-hard**, exact methods quickly become infeasible as the number of items and knapsacks grows.  
For this reason, we adopt a **metaheuristic approach** — specifically, **Simulated Annealing (SA)** — which searches for near-optimal solutions efficiently by exploring and refining the solution space over time.

---

## **Problem and Solution Space**

### **Problem Space**
The **problem space** represents all possible configurations of how items can be distributed among the knapsacks (including the option of leaving items unassigned).  
If there are $n$ items and $k$ knapsacks, each item can be in one of $k + 1$ states (placed in one of the knapsacks, or not placed at all).  
Thus, the total number of possible configurations is roughly:

$$
(k + 1)^n
$$

This enormous combinatorial space makes exhaustive search impossible for large $n$, since even small increases in problem size cause an exponential explosion in possibilities.

---

### **Solution Space**
The **solution space** is the subset of the problem space containing **valid configurations** — that is, those that satisfy all capacity constraints in every dimension for every knapsack.

Each solution can be visualized as a point in this multi-dimensional landscape, where:
- The **coordinates** represent item placements (binary assignment variables).
- The **height** of the point represents the **total value** (objective function).

In this landscape:
- **Feasible regions** (within constraints) are allowed solutions.  
- **Infeasible regions** (where constraints are violated) are excluded.  
- The **global maximum** corresponds to the optimal configuration (maximum total value).  
- Numerous **local maxima** represent suboptimal but valid configurations.

Simulated Annealing can be understood as a **stochastic walk** through this landscape — a process of **moving from one valid solution to another** by making small, random or greedy edits.  
Initially, at high temperature, the algorithm explores the landscape freely, even accepting downward moves (to escape local maxima).  
As the temperature cools, it settles into increasingly stable, high-value regions.

---

## **Overview of the Simulated Annealing Method**

At each iteration:
1. A neighboring (slightly modified) solution is generated via the `tweak()` function.
2. The new solution’s **value** is computed.
3. The algorithm decides whether to **accept** it based on the **Metropolis criterion**:

$$
P(\text{accept}) =
\begin{cases}
1, & \text{if } \Delta > 0 \\
e^{\Delta / T}, & \text{if } \Delta \leq 0
\end{cases}
$$

where:
- $\Delta$ = change in total value (new − current),
- $T$ = current temperature.

Initially, the algorithm is exploratory (high $T$), allowing worse solutions to be accepted.  
As the temperature decreases, the system becomes more selective, converging toward a stable, high-value configuration.

The **temperature schedule** follows an exponential cooling law:

$$
T_i = T_0 \left( \frac{T_f}{T_0} \right)^{i / N}
$$

where:
- $T_0$: initial temperature,  
- $T_f$: final temperature,  
- $N$: total number of iterations.

---

## **Neighborhood Generation: The `tweak()` Function**

The **`tweak()`** function determines how new candidate solutions are generated, depending on the current temperature:

- **High Temperature (Exploration):**  
  The algorithm performs more random modifications via `random_edit()`.
  
- **Low Temperature (Exploitation):**  
  The algorithm applies value-based greedy edits via `greedy_edit()`, focusing on maximizing total value locally.

If a valid configuration is found, it’s accepted; otherwise, the algorithm retries a few times before reverting to the previous valid state.

---

## **Local Editing Operators**

### 1. `random_edit()`
Performs random modifications to diversify the search:
- **Add:** randomly inserts an unassigned item into a knapsack (if it fits),
- **Remove:** randomly deletes an item from a knapsack,
- **Transfer:** moves an item between knapsacks if feasible.

These random changes help escape local optima and maintain diversity.

---

### 2. `greedy_edit()`
Implements a **value-density–based heuristic**:
1. Computes item densities  $d_i = \frac{v_i}{\|w_i\|_2 + \varepsilon}$
2. Attempts to insert high-density items into the knapsack where they fit best.
3. Optimizes transfers between knapsacks to maximize global value.

This operator is most effective when the temperature is low — guiding the search toward refinement rather than exploration.

---

## **Acceptance and Cooling**

The **acceptance probability** allows occasional downhill moves early on, providing flexibility to jump out of local maxima.  
As the temperature cools, these probabilities decrease, and the system “freezes” near an optimal configuration.

Temperature decreases according to:

$$
T_i = T_0 \left( \frac{T_f}{T_0} \right)^{i / N}
$$

---

## **Convergence and Early Stopping**

To avoid unnecessary computation:
- If no improvement occurs for a given number of iterations (`early_stop`), optimization halts early.  
- If stagnation occurs too soon, the algorithm may restart from the best solution found so far.

---

## **Summary**

| Phase | Behavior | Method | Purpose |
|-------|-----------|---------|----------|
| **Exploration** | Random modifications | `random_edit()` | Escape local optima |
| **Exploitation** | Greedy improvements | `greedy_edit()` | Refine best solutions |
| **Cooling** | Gradual temperature reduction | `temperature_at()` | Balance exploration/exploitation |
| **Acceptance** | Probabilistic worsening acceptance | `acceptance()` | Allow exploration early on |
| **Early Stop** | Terminate when stagnation occurs | `optimize()` | Prevent excessive computation |



