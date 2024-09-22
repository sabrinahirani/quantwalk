## Perpetual Pricing Using Quantum Walks

### Background: What is a Perpetual?

In [decentralized finance](https://ethereum.org/en/defi/), a [perpetual](https://dydx.exchange/crypto-learning/perpetuals-crypto) is a type of futures contract with no expiration date (meaning that positions can be held indefinitely). These contracts use a funding rate mechanism to facilitate periodic payments to keep the contract price close to the price of the underlying asset spot price. Decentralized platforms, such as [dydx](https://dydx.exchange/), facilitate these trades using smart contracts, providing a transparent and permissionless trading environment.

### Pricing: The Classical Approach

In a classical setting, pricing relies on the assumption that asset prices follow Geometric Brownian Motion (GBM): a stochastic process that models the behaviour of asset prices over time:

$$
dS_t = \mu S_t \, dt + \sigma S_t \, dW_t
$$

Here, the drift term $\mu$ represents the average rate of return while the volatility term $\sigma$ measures the extent to which the price fluctuates.

**Common Approaches:**

1) Black-Scholes Model: One of the most widely used models in finance, it derives a closed-form solution for the pricing of options and derivatives, including perpetual options. It assumes constant volatility and allows for the calculation of fair value based on inputs like strike price, time to expiration, and risk-free interest rate.

2) Monte Carlo Simulations: This numerical method simulates a large number of potential future price paths based on the GBM model. By aggregating the results, traders can estimate the expected price and associated risks, making it useful for complex derivatives where analytical solutions may not be available.

**Limitations:**

* Assumptions of Normality: GBM assumes that returns are normally distributed, which can underestimate the probability of extreme events or "fat tails" in financial markets. The inability of the GBM model to capture large price changes which occur much more frequently than the Gaussian distribution predicts. When examining return distributions on small time scales, one finds that the distribution is more peaked and the tails are heavier than the Gaussian assumption would permit. This can be described as a “leptokurtic” feature of the return distributions. Furthermore, GBM results in symmetric return distributions. For real data, however, the tails rarely display full symmetry. 

* Constant Volatility: Many classical models assume constant volatility, which fails to capture the changing nature of market conditions over time, leading to potential mispricing.

* Market Dynamics: Classical approaches often do not account for behavioral factors or other market anomalies that can influence prices, limiting their effectiveness in certain scenarios.

These limitations lead us to a quantum approach.


### Pricing: A Quantum Approach

In a quantum setting, we propose a more sophisticated approach by replacing the wiener process with a quantum walk:

$$
dS_t = \mu S_t \, dt + \sigma S_t \, dQ_t
$$

Here, $dQ_t$ represents a quantum walk.

Quantum walks are the quantum equivalent of the classical Markov chain.

### Implementation

In this section, we implement a discrete-time quantum walk for pricing perpetual contracts.

#### Quantum Walk Setup

We utilize two quantum registers: a coin register \( H_C \) and a position register \( H_P \). The coin register facilitates superposition, while the position register holds the price information.

1. **Coin Register**: We employ a Hadamard gate on the coin register to create superposition, referred to as the coin operator.

   ```python
   # Apply Hadamard on the coin register
   circuit.h(coin_register[0])
    ```

2. **Position Register**: The position register is used to calculate the price. We apply a controlled NOT (CNOT) gate from the coin register to a shift gate, enabling phase kickback and creating an entangled system that can explore multiple paths simultaneously.

**Shift Operations:**

The `shift_right` and `shift_left` functions manage the entangled states between the coin and position registers:

```
def shift_right(circuit, position_register):
    """Shift |j> to |j+1>."""
    for i in range(len(position_register)):
        circuit.cx(coin_register[0], position_register[i])  # Controlled shift
        circuit.x(position_register[i])  # Flip position register

def shift_left(circuit, position_register):
    """Shift |j> to |j-1>."""
    circuit.x(coin_register[0])  # Flip for opposite shift
    for i in range(len(position_register)):
        circuit.cx(coin_register[0], position_register[i])  # Controlled shift
    circuit.x(coin_register[0])  # Flip back
```

**Explanation**:

    shift_right shifts the position state to the right (increasing price) based on the state of the coin.
    shift_left shifts the position state to the left (decreasing price) similarly.

**Step Function:**

The step function applies the Hadamard gate and the shift operations:

```
def step(circuit, position_register, coin_register):
    """U = SC."""
    # Apply Hadamard to the coin
    circuit.h(coin_register[0])
    # Apply controlled shift operators
    shift_right(circuit, position_register)
    shift_left(circuit, position_register)
```

This function prepares the system for the next step of the quantum walk by putting the coin in superposition and then performing the shift operations, enabling the exploration of multiple price paths.

### Conclusion

The classical model, particularly the Geometric Brownian Motion (GBM), faces several limitations that the quantum walk model effectively addresses. GBM struggles to capture large price changes, whereas the quantum walk model offers greater flexibility in the shape of return distributions, accommodating bipolar behaviors, biases towards increasing or decreasing prices, extreme changes, and decoherence effects. Additionally, while GBM produces symmetric return distributions, the quantum walk model allows for biases by tuning parameters such as (η, θ) and |ψ(0)⟩. Unlike GBM, which assumes a Markovian process characterized by independent and identically distributed (i.i.d.) randomness as per the Efficient Market Hypothesis, the quantum walk model introduces non-Markovian properties due to interference effects. Moreover, while classical models like the Lognormal and Poisson processes exhibit divergent higher moments, the quantum walk model ensures that these moments do not diverge. Lastly, where classical models provide limited insight into price evolution dynamics and focus solely on return distributions, the quantum walk model offers a more comprehensive dynamical time evolution that ultimately leads to richer return distributions.

By leveraging quantum mechanics, this pricing model addresses the inherent limitations of classical approaches, offering a more comprehensive understanding of asset price dynamics. As the financial landscape evolves, the quantum approach holds promise for enhancing the accuracy and reliability of pricing models in an increasingly complex market environment.



