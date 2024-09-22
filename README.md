## QuantWalk: A Quantum Walk Down Wall Street

### Background

In [decentralized finance](https://ethereum.org/en/defi/), a perpetual is a type of futures contract with no expiration date (meaning that positions can be held indefinitely). These contracts use a funding rate mechanism to facilitate regular payments to keep the contract price close to the price of the underlying asset spot price. Decentralized platforms, such as [dydx](https://dydx.exchange/), facilitate these trades using smart contracts, providing a transparent and permissionless trading environment. With this project *QuantWalk: A Quantum Walk Down Wall Street*, we seek to take advantage of the properties of the discrete-time quantum walk to price perpertuals more effectively than classical pricing strategies.

*<u>Reference</u>:* *On The Potential Of Quantum Walks For Modelling Financial Return Distributions.* Backer et Al. Available at: https://arxiv.org/html/2403.19502v1. 

### The Classical Approach

In a classical setting, pricing relies on the assumption that asset prices follow a stochastic process known as [Geometric Brownian Motion](https://en.wikipedia.org/wiki/Geometric_Brownian_motion) (GBM):

$$
dS_t = \mu S_t \, dt + \sigma S_t \, dW_t
$$

Here, the drift term $\mu$ indicates the average rate of return while the volatility term $\sigma$ indicates the extent to which the price fluctuates.

However, there are numerous limitations to this model: GBM assumes that returns follow a normal distribution, underestimating the probability of extreme price changes which occur far more frequently than predicted by the normal distribution. This can be described as a 'leptokurtic' feature of the return distribution. Furthermore, this model produces a symmetric return distribution whereas real data rarely displays full symmetry. These limitations lead us to consider a quantum approach.


### A Quantum Approach

In a quantum setting, we propose a more sophisticated approach by replacing the [Wiener process](https://en.wikipedia.org/wiki/Wiener_process) $W_t$ with a quantum walk $Q_t$:

$$
dS_t = \mu S_t \, dt + \sigma S_t \, dQ_t
$$

### Implementation

A quantum walk is the quantum analogue of a classical random walk, where the walker can exist in a superposition of states, leading to faster spreading and unique interference patterns. It is used in quantum computing and algorithms for tasks like search and optimization. Here, we implement a discrete-time quantum walk for pricing perpetuals. 

To do this, we use the following registers: a coin register $\mathcal{H}_C$ which remains in superposition and a position register $\mathcal{H}_P$ which holds price information. Thus, the quantum space for the entire walker is given by $\mathcal{H}_C \otimes \mathcal{H}_C$.

Now, to take a step in the quantum walk, we apply a coin operator to $\mathcal{H}_C$ to put the walker in superposition. In this implementation, we use a Hadamard:

$$
C |0\rangle = H |0\rangle = \frac{1}{\sqrt{2}} \left( |0\rangle + |1\rangle \right)
$$

$$
C |1\rangle = H |1\rangle = \frac{1}{\sqrt{2}} \left( |0\rangle - |1\rangle \right)
$$

From here, We apply a controlled NOT (CNOT) gate from the coin register to a shift gate, enabling phase kickback and creating an entangled system that can explore multiple paths simultaneously.

These shift operators are defined as follows:

$$
S |0\rangle |j\rangle = |0\rangle |j+1\rangle
$$

$$
S |1\rangle |j\rangle = |1\rangle |j-1\rangle
$$

```python
def shift_right(circuit, position_register):
    for i in range(len(position_register)):
        circuit.cx(coin_register[0], position_register[i])
        circuit.x(position_register[i])

def shift_left(circuit, position_register):
    circuit.x(coin_register[0])
    for i in range(len(position_register)):
        circuit.cx(coin_register[0], position_register[i])
    circuit.x(coin_register[0]) 
```

Now, for each step in our walk, we apply these gates simultaneously:

$$
U = SC
$$

```python
def step(circuit, position_register, coin_register):
    circuit.h(coin_register[0])
    shift_right(circuit, position_register)
    shift_left(circuit, position_register)
```

To simulate the quantum walk over $n$ steps, we repeat the step function $n$ times:

$$
U_n = (SC)^n
$$

```python
for _ in range(n_steps):
    step(circuit, position_register, coin_register)
```


### Conclusion

Overall, we observe that the classical model faces several limitations that are addressed by the quantum model. The quantum walk model offers greater flexibility in the shape of the return distribution, accomodating biasas, bipolar behaviours, and extreme price changes (which the classical model fails to simulate). Additionally, unlike the classical model which produces symmetric return distributions, the quantum walk model allows for biases and introduces non-markovian properties due to interference effects. 

By leveraging quantum mechanics, this pricing model addresses the inherent limitations of classical approaches, offering a more comprehensive understanding of asset price dynamics. As the financial landscape evolves, the quantum approach holds promise for enhancing the accuracy and reliability of pricing models in an increasingly complex market environment.



