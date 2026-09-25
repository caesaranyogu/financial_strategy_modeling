# Mathematical Modeling of an XRP Trading Strategy

> A software-development study guide for translating a crypto trading idea into variables, equations, algorithms, optimization problems, and validation tests.

## Core idea

Mathematical modeling means taking a real-world rule written in ordinary language and translating it into variables, equations, functions, constraints, and state transitions that a computer can evaluate.

For the XRP strategy:

```text
Market Data → State → Signal → Position → Exit → Return
                                      ↓
                              Optimization → Validation
```

The important distinction is:

1. **Data** — what actually happened.
2. **Model** — mathematical representation of how the strategy behaves.
3. **Optimization** — searching parameter values according to an objective.
4. **Validation** — testing frozen parameters on unseen data.

The uploaded XRP dataset used in the surrounding analysis contains 1,827 daily observations from September 24, 2021 through September 24, 2026.

---

# 1. Learn the notation first

## Variables

A variable is a quantity that can change.

```math
P
```

can represent XRP price.

## Parameters

A parameter is a number chosen to define the strategy.

```math
t = 0.08
```

means an 8% profit target.

## Subscripts

A subscript is an index/label. It is not a mathematical operation.

```math
P_1 = price\ observation\ 1
```

```math
P_2 = price\ observation\ 2
```

```math
P_t = price\ at\ time\ t
```

```math
R_i = return\ of\ trade\ i
```

Think of `t` as a **time index** and `i` as a **trade index**.

---

# 2. Time and market data

For daily observations:

```math
t = 1,2,3,\ldots,T
```

- `t` = current day/time index
- `T` = final observation

For each day:

```math
O_t,H_t,L_t,C_t,V_t
```

where:

- `O_t` = open
- `H_t` = high
- `L_t` = low
- `C_t` = close
- `V_t` = volume

A compact representation is:

```math
X = \{O_t,H_t,L_t,C_t,V_t\}_{t=1}^{T}
```

Read this as: dataset `X` contains OHLCV for every day from 1 through `T`.

---

# 3. Reference high and drawdown

Your strategy asks: **How far has XRP fallen from a reference high?**

Let:

```math
H_t^*
```

be the relevant reference/local high at time `t`.

The superscript `*` is just a label here. It is **not multiplication**.

Define drawdown:

```math
D_t = 1 - \frac{P_t}{H_t^*}
```

If the reference high is $2.00 and current price is $1.70:

```math
D_t = 1 - 1.70/2.00 = 0.15 = 15%
```

Your verbal rule:

> Buy after XRP falls 15% from the reference high.

becomes:

```math
D_t \ge 0.15
```

---

# 4. Parameterization: turn rules into adjustable numbers

Instead of hard-coding 15%, define:

```math
e = entry\ drawdown\ threshold
```

Then the entry rule is:

```math
D_t \ge e
```

Now `e` can be tested at 5%, 10%, 15%, 20%, etc.

This is **parameterization**: turning a fixed rule into a model controlled by parameters.

---

# 5. Entry as an indicator function

An indicator function is a mathematical if/else that returns 1 when a condition is true and 0 otherwise.

```math
I_t = \begin{cases}
1 & \text{if entry condition is satisfied}\\
0 & \text{otherwise}
\end{cases}
```

More compactly:

```math
I_t = \mathbf{1}(D_t \ge e)
```

The bold `1` means an indicator function. In Python this is essentially:

```python
entry_signal = int(drawdown >= entry_threshold)
```

---

# 6. Profit target and stop

Let:

```math
t = profit\ target
```

```math
s = stop\ loss\ percentage
```

If entry price is `P_entry`:

```math
P_target = P_entry(1+t)
```

```math
P_stop = P_entry(1-s)
```

For +8% / -2%:

```math
t = 0.08, \qquad s = 0.02
```

so:

```math
P_target = 1.08P_entry
```

```math
P_stop = 0.98P_entry
```

The basic exit system is therefore controlled by `(t,s)`.

---

# 7. Piecewise functions: the mathematical if/else

Define trade `i`'s return:

```math
R_i = \begin{cases}
+t & \text{if target is hit first}\\
-s & \text{if stop is hit first}
\end{cases}
```

The large curly bracket is **piecewise notation**. It means: use one expression under one condition and another expression under another condition.

Python:

```python
if target_hit_first:
    R_i = t
else:
    R_i = -s
```

For +8% / -2%, `R_i` can be `0.08` or `-0.02`.

---

# 8. Random variable and probability

Before trade `i` resolves, you don't know whether `R_i` will be +8% or -2%. That uncertainty lets us model `R_i` as a random variable.

Define:

```math
p = P(\text{win})
```

Here `P(...)` means **probability**, not price. Context matters.

If the only outcomes are win/loss:

```math
P(\text{loss}) = 1-p
```

If `p = 0.30`, then win probability is 30% and loss probability is 70%.

---

# 9. Expected return

`E[R]` means the **expected value** of return `R`. It is a long-run weighted average, not a promise about the next trade.

```math
E[R] = p(t) + (1-p)(-s)
```

which simplifies to:

```math
\boxed{E[R] = pt - (1-p)s}
```

Interpretation:

> Expected return = probability of winning × size of win − probability of losing × size of loss.

Example, +8% / -2% with a 30% win rate:

```math
E[R] = (0.30)(0.08) - (0.70)(0.02)
     = 0.024 - 0.014
     = 0.010
```

So expected return is 1% per trade **under those assumptions and before costs**.

---

# 10. Break-even win rate

Set expected return to zero:

```math
pt-(1-p)s=0
```

Distribute:

```math
pt-s+ps=0
```

Rearrange:

```math
p(t+s)=s
```

Divide:

```math
\boxed{p^* = \frac{s}{t+s}}
```

The star in `p*` means a special solution value here: the break-even probability. It is not multiplication.

For +8% / -2%:

```math
p^* = \frac{0.02}{0.08+0.02}=0.20
```

So the theoretical break-even win rate is 20% before costs.

---

# 11. Is the 2% stop actually optimal?

Not from the earlier analysis. The earlier analysis held `s = 0.02` fixed while changing other parameters. To test the stop itself, make `s` a parameter and test it.

For example:

```math
s \in \{0.005,0.01,0.015,0.02,0.025,0.03,0.04,0.05\}
```

Then evaluate the strategy for each value.

But **optimal depends on the objective**. Possible objectives include:

```math
J = final\ wealth
```

```math
J = CAGR
```

```math
J = \frac{return}{maximum\ drawdown}
```

or a risk-adjusted objective such as:

```math
J = \frac{E[R]}{\sigma(R)}
```

where `σ` (sigma) means standard deviation/dispersion of returns.

Therefore there is no universally optimal stop. There is an optimal stop **relative to a dataset, model, and objective**.

---

# 12. Two-dimensional optimization

If target and stop are both variable:

```math
t \in \{1%,2%,3%,4%,5%,6%,8%,10%,12%,15%\}
```

and:

```math
s \in \{0.5%,1%,1.5%,2%,2.5%,3%,4%,5%,7.5%,10%\}
```

then there are:

```math
10 \times 10 = 100
```

parameter combinations.

For each combination calculate metrics such as:

- number of trades
- win rate
- expected return
- final capital
- maximum drawdown
- profit factor
- risk-adjusted return

Then:

```math
\boxed{(t^*,s^*) = \arg\max_{t,s} J(t,s)}
```

This is a **two-dimensional parameter search**.

---

# 13. `theta`: the strategy parameter vector

The Greek letter theta is:

```math
\theta
```

Think of lowercase theta as **one complete configuration of your strategy**.

For example:

```math
\theta=(e,t,s,w)
```

could mean:

- `e` = entry threshold
- `t` = target
- `s` = stop
- `w` = allocation

For example:

```math
\theta=(0.15,0.08,0.02,0.50)
```

is one strategy configuration.

In Python this is similar to:

```python
theta = (0.15, 0.08, 0.02, 0.50)
```

The parentheses are just a container. They are not multiplication.

---

# 14. `Theta`: the parameter space

Uppercase theta is:

```math
\Theta
```

Think of uppercase Theta as **all configurations we are willing to test**.

For example, if we allow:

```text
entry: 10%, 15%, 20%
target: 3%, 5%, 8%
stop: 1%, 2%, 3%
```

then `Theta` represents all allowed combinations.

The notation:

```math
\theta \in \Theta
```

means:

> theta is one member of the allowed parameter space.

The symbol `∈` means **belongs to / is an element of**.

---

# 15. `argmax`: the other intimidating symbol

```math
\arg\max
```

Break it apart:

- `max` = maximum
- `arg` = argument/input that produced the result

So `argmax` means:

> **Give me the input that produces the maximum output.**

Example:

```text
Strategy A → $5,500
Strategy B → $6,200
Strategy C → $5,900
```

`max` asks: what is the biggest result? → `$6,200`

`argmax` asks: which input produced the biggest result? → `Strategy B`

---

# 16. Objective function `J(theta)`

Let:

```math
J(\theta)
```

mean the performance of the strategy described by `theta`.

The `J` is an **objective function**.

For example:

```math
J(\theta) = ending\ account\ value
```

Then:

```math
\boxed{\theta^* = \arg\max_{\theta\in\Theta}J(\theta)}
```

means:

> Search through every allowed strategy configuration and find the configuration that produces the largest objective value.

Breakdown:

```math
\theta^* = best\ configuration\ found
```

```math
\arg\max = find\ the\ input\ producing\ the\ maximum
```

```math
\theta\in\Theta = search\ only\ allowed\ configurations
```

```math
J(\theta) = measure\ performance
```

---

# 17. Strategy as a computational function

The backtest can be thought of as:

```math
W_T = f(\theta,X)
```

where:

- `X` = XRP market dataset
- `theta` = strategy parameters
- `f` = backtesting/model function
- `W_T` = ending wealth

This is a very important software concept.

You're turning a strategy into a function:

```python
def backtest(data, theta):
    ...
    return ending_wealth
```

Now the computer can call that function hundreds or thousands of times with different `theta` values.

---

# 18. Full parameter vector

A richer XRP strategy might be:

```math
\theta=(e_1,e_2,w_1,w_2,t,s,f,\delta,\tau,L_{max},r_{max})
```

where:

- `e1` = first entry drawdown
- `e2` = second entry drawdown
- `w1,w2` = capital allocation weights
- `t` = target
- `s` = stop
- `f` = transaction cost
- `δ` = slippage
- `τ` = tax assumption
- `Lmax` = maximum daily losses
- `rmax` = maximum portfolio risk

This is where a simple trading idea becomes a genuine mathematical model.

---

# 19. Optimization over the whole strategy

The full optimization problem is:

```math
\boxed{\theta^* = \arg\max_{\theta\in\Theta} J(\theta)}
```

Or, if the objective is explicitly the backtest ending wealth:

```math
\boxed{\theta^* = \arg\max_{\theta\in\Theta} f(\theta,X)}
```

Read it as:

> Find the strategy configuration in the allowed parameter space that produces the best objective when applied to the XRP dataset.

---

# 20. Overfitting

Suppose you search enough parameters and eventually find:

```math
e=17.3%,\qquad t=6.84%,\qquad s=1.73%
```

and that combination turned $5,000 into $19,000 historically.

That does not prove those values represent a genuine market edge.

You may have simply found the combination that happened to fit the historical sample.

That is **overfitting**.

The danger increases as the number of parameters and tested combinations grows.

---

# 21. Train/test separation

Let:

```math
X = X_{train} \cup X_{test}
```

The symbol `∪` means **union**: combine the two sets.

Use:

```math
X_{train}
```

to select:

```math
\theta^*
```

Then freeze `theta*`.

Evaluate the frozen strategy on:

```math
X_{test}
```

without changing it.

This asks the important question:

> Does the model work on data it did not use to choose its parameters?

---

# 22. Monte Carlo simulation

Historical data gives one realized sequence of trades:

```math
R_1,R_2,\ldots,R_n
```

Monte Carlo asks what might happen if trade outcomes with similar characteristics occurred in different sequences.

We can resample the observed trade returns many times and generate thousands of possible account paths.

This doesn't predict the future. It explores the distribution of outcomes under the assumptions of the simulation.

---

# 23. The `prod` symbol: `∏`

This is the **product operator**, analogous to `Σ` for summation.

```math
\sum_{i=1}^{n}R_i
```

means:

```text
R1 + R2 + R3 + ... + Rn
```

while:

```math
\prod_{i=1}^{n}(1+R_i)
```

means:

```text
(1+R1)(1+R2)(1+R3)...(1+Rn)
```

The bottom:

```math
i=1
```

means start at trade 1.

The top:

```math
n
```

means continue through trade `n`.

---

# 24. Compound account evolution

Suppose:

```math
W_0 = $5,000
```

The subscript 0 means starting wealth, before trade 1.

If trade 1 earns 8%:

```math
W_1=W_0(1.08)
```

If trade 2 loses 2%:

```math
W_2=W_1(0.98)
```

If trade 3 earns 8%:

```math
W_3=W_2(1.08)
```

Combined:

```math
W_3=W_0(1.08)(0.98)(1.08)
```

For `n` trades:

```math
\boxed{W_n=W_0\prod_{i=1}^{n}(1+R_i)}
```

This is why account growth is multiplicative rather than additive when the entire account is repeatedly exposed.

---

# 25. The complete mathematical architecture

```text
REAL-WORLD STRATEGY
        │
        ▼
     DATA X
        │
        ▼
  STATE VARIABLES
  Pₜ, Hₜ*, Dₜ
        │
        ▼
      SIGNAL
     Dₜ ≥ e
        │
        ▼
     POSITION
      qᵢ, wᵢ
        │
        ▼
       EXIT
   target / stop
        │
        ▼
   TRADE RETURN
       Rᵢ
        │
        ▼
   ACCOUNT Wₙ
       ∏
        │
        ▼
    OBJECTIVE
      J(θ)
        │
        ▼
   OPTIMIZATION
      argmax
        │
        ▼
    VALIDATION
  unseen X_test
```

The translation skill to practice:

| English question | Mathematical concept |
|---|---|
| What changes? | Variable |
| What am I choosing? | Parameter |
| What describes the current situation? | State variable |
| What condition causes an action? | Boolean/indicator |
| What happens under different conditions? | Piecewise function |
| What is uncertain? | Random variable / probability |
| What is the long-run average? | Expected value `E[X]` |
| What parameters am I searching? | Parameter space `Θ` |
| What am I optimizing? | Objective `J(θ)` |
| Which input gives the best result? | `argmax` |
| What compounds across trades? | Product `∏` |
| Does the model generalize? | Out-of-sample validation |
| Did I fit history too closely? | Overfitting |

---

# 26. Symbol dictionary

| Symbol | Meaning in this model |
|---|---|
| `t` | time/day index |
| `T` | final day index |
| `i` | trade index |
| `n` / `N` | number of trades / final trade index |
| `P_t` | price at time `t` |
| `O_t` | open at time `t` |
| `H_t` | high at time `t` |
| `L_t` | low at time `t` |
| `C_t` | close at time `t` |
| `V_t` | volume at time `t` |
| `H_t*` | reference high |
| `D_t` | drawdown at time `t` |
| `e` | entry threshold |
| `t` | profit-target parameter (context-dependent; in code prefer `target`) |
| `s` | stop-loss parameter |
| `R_i` | return from trade `i` |
| `p` | win probability |
| `E[R]` | expected return |
| `p*` | break-even probability in this model |
| `w_i` | allocation weight for entry `i` |
| `q_i` | quantity purchased in entry `i` |
| `f` | transaction-cost parameter |
| `δ` | slippage parameter |
| `τ` | tax parameter |
| `L_max` | maximum daily losses |
| `r_max` | maximum portfolio risk |
| `W_0` | starting wealth |
| `W_n` | wealth after `n` trades |
| `X` | market dataset |
| `X_train` | training dataset |
| `X_test` | test dataset |
| `θ` | one strategy parameter vector |
| `Θ` | allowed parameter space |
| `J(θ)` | objective/performance function |
| `f(θ,X)` | backtest/model function |
| `θ*` | selected/best parameter vector under the objective |
| `argmax` | input producing the maximum output |
| `Σ` | sum operator |
| `∏` | product/multiplication operator |
| `∈` | belongs to |
| `∪` | union |
| `σ` | standard deviation |
| `*` | context-dependent label; here not multiplication |

> **Notation warning:** `t` is overloaded in this study: it means time index in `P_t`, but also profit target in the trading parameter notation. In production Python, use clearer names such as `time_index` and `target` to avoid ambiguity.

---

# 27. Python: notation made concrete

The following code intentionally mirrors the mathematics.

## 27.1 Basic variables and subscripts

```python
prices = [1.20, 1.25, 1.22]

# Mathematical P_1, P_2, P_3
P_1 = prices[0]
P_2 = prices[1]
P_3 = prices[2]

print(P_1, P_2, P_3)
```

In Python, list indexing starts at 0, while mathematical examples often start at 1. This is a useful example of why mathematical notation and programming notation are related but not identical.

---

## 27.2 Drawdown function

Mathematics:

```math
D_t = 1 - P_t/H_t^*
```

Python:

```python
def drawdown(price: float, reference_high: float) -> float:
    """Return drawdown as a decimal."""
    return 1.0 - price / reference_high

print(drawdown(1.70, 2.00))  # 0.15
```

---

## 27.3 Entry condition

Mathematics:

```math
D_t \ge e
```

Python:

```python
def entry_signal(price: float, reference_high: float, entry_threshold: float) -> bool:
    d = drawdown(price, reference_high)
    return d >= entry_threshold

print(entry_signal(1.70, 2.00, 0.15))  # True
print(entry_signal(1.80, 2.00, 0.15))  # False
```

---

## 27.4 Indicator function

Mathematics:

```math
I_t = 1(D_t \ge e)
```

Python:

```python
def indicator(condition: bool) -> int:
    return int(condition)

I_t = indicator(entry_signal(1.70, 2.00, 0.15))
print(I_t)  # 1
```

---

## 27.5 Target and stop prices

Mathematics:

```math
P_target=P_entry(1+target)
```

```math
P_stop=P_entry(1-stop)
```

Python:

```python
def target_price(entry_price: float, target: float) -> float:
    return entry_price * (1.0 + target)


def stop_price(entry_price: float, stop: float) -> float:
    return entry_price * (1.0 - stop)

entry = 1.50
target = 0.08
stop = 0.02

print(target_price(entry, target))  # 1.62
print(stop_price(entry, stop))      # 1.47
```

---

## 27.6 Piecewise trade return

Mathematics:

```math
R_i = \begin{cases}
t & target\ first\\
-s & stop\ first
\end{cases}
```

Python:

```python
def trade_return(target: float, stop: float, target_hit_first: bool) -> float:
    if target_hit_first:
        return target
    return -stop

print(trade_return(0.08, 0.02, True))   # 0.08
print(trade_return(0.08, 0.02, False))  # -0.02
```

---

## 27.7 Expected return

Mathematics:

```math
E[R] = pt - (1-p)s
```

Python:

```python
def expected_return(win_probability: float,
                     target: float,
                     stop: float) -> float:
    p = win_probability
    return p * target - (1.0 - p) * stop

print(expected_return(0.30, 0.08, 0.02))  # 0.01
```

---

## 27.8 Break-even probability

Mathematics:

```math
p* = s/(t+s)
```

Python:

```python
def breakeven_win_rate(target: float, stop: float) -> float:
    return stop / (target + stop)

print(breakeven_win_rate(0.08, 0.02))  # 0.20
```

---

## 27.9 Compound wealth

Mathematics:

```math
W_n=W_0\prod_{i=1}^{n}(1+R_i)
```

Python:

```python
def compound_wealth(starting_wealth: float, returns: list[float]) -> float:
    wealth = starting_wealth

    for r in returns:
        wealth *= (1.0 + r)

    return wealth

returns = [0.08, -0.02, 0.08]
print(compound_wealth(5000.0, returns))
```

The loop is the programming equivalent of the product operator.

---

## 27.10 Summation versus product

Mathematics:

```math
\sum_{i=1}^{n}R_i
```

Python:

```python
returns = [0.08, -0.02, 0.08]

sum_of_returns = sum(returns)
print(sum_of_returns)
```

Mathematics:

```math
\prod_{i=1}^{n}(1+R_i)
```

Python:

```python
import math

returns = [0.08, -0.02, 0.08]

growth_factors = [1.0 + r for r in returns]
compound_factor = math.prod(growth_factors)

print(growth_factors)
print(compound_factor)
```

---

## 27.11 Parameter vector theta

Mathematics:

```math
θ=(e,t,s,w)
```

Python:

```python
theta = {
    "entry": 0.15,
    "target": 0.08,
    "stop": 0.02,
    "allocation": 0.50,
}

print(theta)
```

A dictionary is often better than a tuple in production because each value has an explicit name.

---

## 27.12 A strategy function

Mathematics:

```math
W_T=f(θ,X)
```

Python:

```python
def strategy_score(data, theta):
    """Placeholder model: return an objective score for theta."""
    # A real implementation would simulate the complete strategy here.
    # The point is that theta and data are inputs and the performance is output.
    return backtest(data, theta)
```

This is the central software abstraction:

```text
inputs → model → output
```

---

# 28. Optimization in Python

A simple grid search is the direct programming equivalent of testing a finite parameter space.

```python
def grid_search(data, entries, targets, stops, objective):
    best_theta = None
    best_score = float("-inf")
    results = []

    for entry in entries:
        for target in targets:
            for stop in stops:
                theta = {
                    "entry": entry,
                    "target": target,
                    "stop": stop,
                }

                score = objective(data, theta)

                results.append({
                    "theta": theta,
                    "score": score,
                })

                if score > best_score:
                    best_score = score
                    best_theta = theta

    return best_theta, best_score, results
```

The mathematical idea is:

```math
θ* = argmax_{θ∈Θ} J(θ)
```

The Python `if score > best_score` is implementing the logic of `argmax`.

---

# 29. Stop-loss optimization

If you want to test whether 2% is actually supported by the data, first hold everything else constant and vary only `stop`.

```python
stops = [
    0.005,  # 0.5%
    0.010,  # 1%
    0.015,  # 1.5%
    0.020,  # 2%
    0.025,  # 2.5%
    0.030,  # 3%
    0.040,  # 4%
    0.050,  # 5%
]

results = []

for stop in stops:
    theta = {
        "entry": 0.10,
        "target": 0.08,
        "stop": stop,
    }

    score = backtest(data, theta)

    results.append({
        "stop": stop,
        "score": score,
    })

best = max(results, key=lambda row: row["score"])
print(best)
```

This is conceptually:

```math
s^* = argmax_{s∈S}J(s)
```

---

# 30. Two-dimensional target/stop search

```python
targets = [0.01, 0.02, 0.03, 0.04, 0.05, 0.06, 0.08, 0.10, 0.12, 0.15]
stops = [0.005, 0.01, 0.015, 0.02, 0.025, 0.03, 0.04, 0.05, 0.075, 0.10]

results = []

for target in targets:
    for stop in stops:
        theta = {
            "entry": 0.10,
            "target": target,
            "stop": stop,
        }

        score = backtest(data, theta)

        results.append({
            "target": target,
            "stop": stop,
            "score": score,
        })

best = max(results, key=lambda row: row["score"])
print(best)
```

There are 100 combinations because:

```python
len(targets) * len(stops)
```

returns:

```text
100
```

---

# 31. Train/test validation in Python

```python
def train_test_split_by_date(data, split_date):
    train = data[data["date"] <= split_date].copy()
    test = data[data["date"] > split_date].copy()
    return train, test
```

Conceptually:

```math
X = X_train ∪ X_test
```

Then:

```python
best_theta, train_score, _ = grid_search(
    train,
    entries=[0.10, 0.15, 0.20],
    targets=[0.03, 0.05, 0.08],
    stops=[0.01, 0.02, 0.03],
    objective=backtest,
)

# Freeze best_theta.

test_score = backtest(test, best_theta)

print("Chosen parameters:", best_theta)
print("Training score:", train_score)
print("Test score:", test_score)
```

The critical point is that `best_theta` is **not changed after looking at the test result**.

---

# 32. Monte Carlo in Python

A simple bootstrap-style simulation of trade returns:

```python
import random


def monte_carlo_paths(historical_returns, starting_wealth, n_trades, n_simulations):
    final_wealth = []

    for _ in range(n_simulations):
        wealth = starting_wealth

        for _ in range(n_trades):
            r = random.choice(historical_returns)
            wealth *= (1.0 + r)

        final_wealth.append(wealth)

    return final_wealth
```

Usage:

```python
historical_returns = [
    0.08, -0.02, 0.08, -0.02, -0.02,
    0.08, -0.02, 0.08
]

simulated = monte_carlo_paths(
    historical_returns,
    starting_wealth=5000,
    n_trades=50,
    n_simulations=10000,
)

print(min(simulated))
print(max(simulated))
print(sum(simulated) / len(simulated))
```

This is a model of possible sequences under the assumption that the historical trade-return distribution is representative.

---

# 33. Maximum drawdown as a model metric

A strategy can make money while suffering very large temporary losses. So final wealth alone is not enough.

```python
def max_drawdown(wealth_path):
    peak = wealth_path[0]
    worst = 0.0

    for wealth in wealth_path:
        peak = max(peak, wealth)
        drawdown = (peak - wealth) / peak
        worst = max(worst, drawdown)

    return worst
```

Mathematically, if `W_t` is wealth at time `t`, a running peak can be represented as:

```math
M_t = max_{1\le j\le t} W_j
```

and drawdown as:

```math
DD_t = 1 - \frac{W_t}{M_t}
```

Maximum drawdown is:

```math
MDD = max_t DD_t
```

Notice the difference between:

```math
D_t = price\ drawdown
```

and:

```math
DD_t = account\ drawdown
```

Same mathematical idea, different object.

---

# 34. Profit factor

If winning trade returns and losing trade returns are available separately:

```python
def profit_factor(returns):
    gross_profit = sum(r for r in returns if r > 0)
    gross_loss = -sum(r for r in returns if r < 0)

    if gross_loss == 0:
        return float("inf")

    return gross_profit / gross_loss
```

Conceptually:

```math
Profit\ Factor = \frac{Gross\ Profit}{Gross\ Loss}
```

A value above 1 means gross winning returns exceed gross losing returns in the sample.

---

# 35. A complete toy model

This small example puts several concepts together without pretending to be a production backtester.

```python
from dataclasses import dataclass
from math import prod


@dataclass(frozen=True)
class StrategyParams:
    entry: float
    target: float
    stop: float


def drawdown(price: float, reference_high: float) -> float:
    return 1.0 - price / reference_high


def trade_return(target: float, stop: float, win: bool) -> float:
    return target if win else -stop


def expected_return(p: float, target: float, stop: float) -> float:
    return p * target - (1.0 - p) * stop


def breakeven_probability(target: float, stop: float) -> float:
    return stop / (target + stop)


def compound(start: float, returns: list[float]) -> float:
    return start * prod(1.0 + r for r in returns)


params = StrategyParams(
    entry=0.15,
    target=0.08,
    stop=0.02,
)

print("Drawdown:", drawdown(1.70, 2.00))
print("Target price:", 1.50 * (1 + params.target))
print("Stop price:", 1.50 * (1 - params.stop))
print("Expected return at p=.30:", expected_return(0.30, params.target, params.stop))
print("Break-even win rate:", breakeven_probability(params.target, params.stop))
print("Compounded wealth:", compound(5000, [0.08, -0.02, 0.08]))
```

---

# 36. The deeper software-development lesson

The trading strategy is just the example. The same modeling process applies to software systems, simulations, finance, engineering, logistics, games, and operations research.

When given a messy real-world problem, ask:

1. **What are the entities?**
2. **What changes over time?**
3. **What variables describe the system?**
4. **What values are fixed/selected parameters?**
5. **What is the current state?**
6. **What conditions trigger transitions?**
7. **What outputs should the system produce?**
8. **What uncertainty exists?**
9. **What objective are we optimizing?**
10. **How do we validate the model?**

That is mathematical modeling translated into software thinking.

---

# 37. Spaced-repetition review

## Day 1 — Recognition

Explain without looking:

- What is a variable?
- What is a parameter?
- What does a subscript mean?
- What do `t` and `i` represent?
- What is `D_t`?
- What are `t` and `s` in the trade model?
- What does `R_i` represent?
- What does `E[R]` mean?
- What does `∏` mean?
- What does `argmax` mean?
- What are `θ` and `Θ`?

## Day 3 — Reconstruction

Without looking, reconstruct:

```math
D_t = 1 - P_t/H_t^*
```

```math
P_target = P_entry(1+t)
```

```math
P_stop = P_entry(1-s)
```

```math
E[R] = pt-(1-p)s
```

```math
p^* = s/(t+s)
```

```math
W_n=W_0\prod_{i=1}^{n}(1+R_i)
```

## Day 7 — Explain

Explain these in plain English:

```math
\theta\in\Theta
```

```math
\arg\max_{\theta\in\Theta}J(\theta)
```

```math
X=X_train\cup X_test
```

Then explain why train/test separation reduces the risk of overfitting.

## Day 14 — Implement

Write Python from memory for:

- drawdown
- entry signal
- target/stop prices
- expected return
- break-even win rate
- compound wealth
- simple grid search

## Day 30 — Generalize

Take a completely different problem, such as:

> “A delivery company wants to minimize late deliveries while controlling cost.”

Define:

- variables
- parameters
- state
- objective function
- constraints
- parameter space
- optimization problem
- train/test or validation method if historical data is involved

If you can do that, you are no longer merely memorizing the XRP example. You understand the modeling framework.

---

# 38. Final mental model

Memorize this progression:

```text
REAL WORLD
    ↓
What changes?
    ↓
VARIABLES
    ↓
What do I choose?
    ↓
PARAMETERS θ
    ↓
What is the current situation?
    ↓
STATE
    ↓
What condition causes action?
    ↓
SIGNAL / FUNCTION
    ↓
What happens next?
    ↓
STATE TRANSITION
    ↓
What is the result?
    ↓
OUTPUT / RETURN
    ↓
What am I trying to improve?
    ↓
OBJECTIVE J(θ)
    ↓
Which parameters do best?
    ↓
ARGMAX
    ↓
Does it work on unseen data?
    ↓
VALIDATION
```

The key symbols to know first are:

```math
\boxed{P_t = value\ at\ time\ t}
```

```math
\boxed{R_i = result\ of\ trade\ i}
```

```math
\boxed{\theta = one\ strategy\ configuration}
```

```math
\boxed{\Theta = all\ allowed\ configurations}
```

```math
\boxed{E[R] = expected\ return}
```

```math
\boxed{\sum = add\ a\ sequence}
```

```math
\boxed{\prod = multiply\ a\ sequence}
```

```math
\boxed{\arg\max = find\ the\ input\ producing\ the\ maximum}
```

```math
\boxed{\theta^*=\arg\max_{\theta\in\Theta}J(\theta)}
```

That last equation is the heart of the optimization problem:

> **Search the allowed strategy configurations and find the one that maximizes the objective.**

And the most important caveat is equally simple:

> **The best historical parameter set is not automatically the best future parameter set.**

That is why mathematical modeling must be followed by validation.
