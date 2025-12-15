# floating-point guardian

Look at my digital gurdian. I built it using my custom made neural network written in C. Bad move? eh well.
Connection: ncat --ssl floating.chals.nitectf25.live 1337

## Solution:

### Initial Analysis

The challenge provides a C source file (`src.c`) that implements a 3-layer neural network. The program:
1. Asks 15 questions about health/biometric data
2. Processes the inputs through a neural network with custom activation functions
3. Checks if the output probability matches `0.7331337420` within epsilon `0.00001`
4. Prints the flag if successful

```
c
#define TARGET_PROBABILITY 0.7331337420
#define EPSILON 0.00001

if (fabs(probability - TARGET_PROBABILITY) < EPSILON) {
    print_flag();
}
```

The neural network architecture:
- **Input Layer**: 15 nodes (one for each question)
- **Hidden Layer 1**: 8 nodes with mixed activation functions (XOR, tanh, cos, sinh)
- **Hidden Layer 2**: 6 nodes with tanh activation
- **Output Layer**: 1 node with sigmoid activation

### Key Observations

1. **Custom activation functions** on the input layer based on index modulo 4:
   - `i % 4 == 0`: XOR activation with predefined keys
   - `i % 4 == 1`: tanh activation
   - `i % 4 == 2`: cosine activation
   - `i % 4 == 3`: sinh activation

2. **XOR keys** are hardcoded:
```
c
const unsigned char XOR_KEYS[INPUT_SIZE] = {
    0x42, 0x13, 0x37, 0x99, 0x21, 0x88, 0x45, 0x67,
    0x12, 0x34, 0x56, 0x78, 0x9A, 0xBC, 0xDE
};
```

3. All weights and biases are provided in the source code.

### Approach 1: Optimization (Initial Attempt - Failed)

First, I tried using numerical optimization with scipy:

```
python
from scipy.optimize import minimize, differential_evolution

def forward_pass(inputs):
    # Replicate the C neural network logic
    # ... (implementation details)
    return probability

def loss_function(inputs):
    probability = forward_pass(inputs)
    return abs(probability - TARGET_PROBABILITY)

# Try differential evolution
result = differential_evolution(loss_function, bounds, maxiter=1000)
```

**Testing the optimized result:**
```
ameesha_goel@DESKTOP-7U7A055:/mnt/c/Users/admin/Downloads$ ncat --ssl floating.chals.nitectf25.live 1337
[Q1]  What is your height in centimeters? 165.0986244114
[Q2]  What is your weight in kilograms? 51.9652548382
...
MASTER PROBABILITY: 0.0137561377
You are NOT the Master.
```

The optimization approach failed - the probability was way off!

### Approach 2: Pattern Testing (Breakthrough!)

I decided to test simple patterns instead of random optimization:

```
[Q1]  What is your height in centimeters? 1
[Q2]  What is your weight in kilograms? 2
[Q3]  What is your age in years? 3
...
[Q15] Rate this CTF challenge out of 10: 10

MASTER PROBABILITY: 0.9940158953
You are NOT the Master.
```

 **Key Insight**: Simple sequential inputs (1, 2, 3...) gave `0.9940` - much closer than random optimization! This suggested the answer has **structure** rather than being arbitrary.

### Approach 3: Smart Pattern Testing + Local Optimization

Created a script to test various patterns and use the best as a starting point:

```python
patterns = [
    ("Sequential 1-15", list(range(1, 16))),
    ("All zeros", [0]*15),
    ("Reverse 15-1", list(range(15, 0, -1))),
    ("Halved seq", [i*0.5 for i in range(1, 16)]),
    # ... more patterns
]

# Test each pattern
for name, pattern in patterns:
    prob = forward_pass(pattern)
    diff = abs(prob - TARGET_PROBABILITY)
    print(f"{name} -> {prob:.10f} (diff: {diff:.10f})")

# Use best pattern as starting point for local optimization
result = minimize(loss_fn, best_pattern, method='Powell')
```

### Final Solution

The optimization converged to a **reverse sequence with slight perturbations**:

```bash
[Q1]  What is your height in centimeters? 22.2458701503
[Q2]  What is your weight in kilograms? 28.5042866867
[Q3]  What is your age in years? 12.9655800557
[Q4]  What is your heart rate (bpm)? 11.9999626971
[Q5]  How many hours do you sleep per night? 11.0000004929
[Q6]  What is your body temperature in Celsius? 31.2795728292
[Q7]  How many steps do you walk per day? 9.0000000358
[Q8]  What is your systolic blood pressure? 8.0000000001
[Q9]  How many calories do you consume daily? 7.0000000000
[Q10] What is your BMI (Body Mass Index)? 6.0000000439
[Q11] How many liters of water do you drink daily? 5.0000000000
[Q12] What is your resting metabolic rate (kcal/day)? 4.0000000000
[Q13] How many hours do you exercise per week? 3.0000000000
[Q14] What is your blood glucose level (mg/dL)? 2.0000000000
[Q15] Rate this CTF challenge out of 10: 1.0000000000

Processing through neural network layers...
========================================
MASTER PROBABILITY: 0.7331337420
========================================
```

 **Success!** The neural network accepted the identity.

## Flag:

```
nite{fl0@t1ng_p01nt_pr3c1s10n_w1th_n3ur@l_n3tw0rks}
```

## Concepts learnt:

- **Neural Network Reverse Engineering**: Finding inputs that produce a specific output from a known neural network architecture is an inverse problem. Unlike forward propagation (easy), finding exact inputs for exact outputs is challenging.

- **Non-convex Optimization**: Neural networks create non-convex loss landscapes with many local minima. Standard gradient descent can get stuck, making global optimization difficult.

- **Pattern-based Solutions**: For reverse engineering challenges, testing structured patterns (sequences, constants, etc.) can be more effective than blind optimization. The problem may have been designed with a specific pattern in mind.

- **Mixed Activation Functions**: The challenge used different activation functions for different inputs (XOR, tanh, cos, sinh), adding complexity and making the relationship between inputs and outputs highly non-linear.

- **XOR Activation**: A custom activation that XORs the input (scaled by 1,000,000) with a key, then scales back. This creates discrete jumps in the activation landscape:
  ```
  double xor_activate(double x, unsigned char key) {
      long long_val = (long)(x * 1000000);
      long_val ^= key;
      return (double)long_val / 1000000.0;
  }
  ```

- **Floating Point Precision**: The challenge title refers to achieving exact floating-point values through a complex neural network - a nod to the difficulty of precise numerical computation.

## Notes:

### Failed Approach: Direct Optimization
Initially tried using `scipy.optimize.differential_evolution` and `scipy.optimize.minimize` with random starting points. This failed because:
- The search space is too large (15 dimensions with wide bounds)
- The XOR activation creates discontinuities
- Multiple local minima trapped the optimizer

### The Hint Analysis
The hint "my intern worked for lex" was puzzling:
- Could refer to **Lex Fridman** (AI researcher/podcaster)
- Could be **leetspeak** related (7331337420 looks like leet)
- Might suggest **lexical** ordering or patterns
- The "intern" might hint at junior/lower values

However, the hint didn't seem critical to the solution - testing simple patterns was the breakthrough.

### Alternative Interpretations
The target probability `0.7331337420` looks like leetspeak:
- `7331337420` could be read as "TEELEETBZO" or similar
- This might have been an intentional red herring

### Why Reverse Sequence Worked
The reverse sequence (15, 14, 13...) was close to the target, likely because:
- The neural network was trained/designed with this pattern in mind
- The weights create a landscape where decreasing sequences produce probabilities near 0.73
- Small perturbations from the exact sequence (due to XOR and other non-linearities) needed fine-tuning via optimization

## Resources:

- [Neural Networks and Deep Learning](http://neuralnetworksanddeeplearning.com/) - Understanding neural network fundamentals
- [SciPy Optimization Documentation](https://docs.scipy.org/doc/scipy/reference/optimize.html) - For `minimize` and `differential_evolution` functions
- [NumPy Documentation](https://numpy.org/doc/) - Array operations for implementing the neural network
- [Differential Evolution Algorithm](https://en.wikipedia.org/wiki/Differential_evolution) - Global optimization technique used in the solution

  ***

  
# Antakshari 

 My friend can never remember movie names, absolutely hopeless. This time he only recalls six cast members, nothing else. He's always been dense, but we've been tight for years, so I guess I'm stuck helping him again. Can you figure out the movie he's trying to remember?

The challenge provides two files:

* `latent_vectors.npy`
* `partial_edges.npy`

The goal is to recover the **six actor node IDs** corresponding to the remembered cast and submit them in **descending order**.



## Solution:

### Initial analysis

We are given:

* A set of **latent vectors** (201 × 64), representing actors embedded in a learned feature space.
* A **partial edge list**, representing incomplete co-acting relationships.

At first glance, the edges suggest a graph problem, but the challenge title and category indicate this is an **AI/ML-based challenge**, meaning the embeddings are likely the primary signal.


Challenge description page showing the input field asking for
**“ACTOR NODE SEQUENCE IN DESCENDING ORDER”**


### Exploring the graph (red herring)

The initial approach was to:

* Build a graph from `partial_edges.npy`
* Look for:

  * cliques
  * paths of length 6
  * connected components

However, this quickly revealed:

* No connected component contained 6 nodes
* No simple path of length 6 existed
* The edge data was **partial and insufficient**

This indicated the graph was **not meant to be used directly** to recover the cast.


### Key insight: latent space is the real signal

Since:

* The edges are partial
* Actor names are never provided
* The challenge is AI-themed

The correct interpretation is:

The six cast members are the **six actors that form the tightest cluster in latent space**.

In other words:

* Actors from the same movie should be **close together in the embedding space**
* We need to find the **most compact group of 6 vectors**



### Final approach (pure NumPy, no ML libraries)

To avoid dependency issues and keep the solution deterministic:

1. Load the latent vectors
2. Compute the **pairwise distance matrix**
3. For each actor:

   * Take its 5 nearest neighbors
   * Form a 6-node candidate group
4. Score each group using **average intra-group distance**
5. Select the tightest group
6. Sort node IDs in **descending order** (as required by the UI)

This works efficiently because the dataset is small (201 nodes).



### Final solver script

```
import numpy as np

# Load latent vectors
vectors = np.load("latent_vectors.npy")
n = vectors.shape[0]

# Compute pairwise distance matrix
dists = np.linalg.norm(
    vectors[:, None, :] - vectors[None, :, :],
    axis=2
)

best_score = float("inf")
best_group = None

# For each node, consider its 5 nearest neighbors
for i in range(n):
    neighbors = np.argsort(dists[i])[1:6]  # exclude itself
    group = np.concatenate(([i], neighbors))

    # Average intra-group distance
    sub = dists[np.ix_(group, group)]
    score = np.mean(sub)

    if score < best_score:
        best_score = score
        best_group = group

# Sort for submission
answer = sorted(best_group, reverse=True)

print("SUBMIT THIS:")
print(",".join(map(str, answer)))
```


## Flag:

```
nite{Diehard_1891771341083729}

```

## Concepts learnt:

* **Latent embeddings**
  Learned vector representations that capture semantic similarity between entities (here, actors).

* **Clustering in embedding space**
  Items belonging to the same real-world group (movie cast) tend to cluster closely in latent space.

* **Why partial graphs can be misleading**
  In AI challenges, auxiliary data (edges) may be incomplete and not suitable for direct graph algorithms.

* **Distance-based clustering without ML libraries**
  Small datasets can be clustered using pure NumPy by analyzing pairwise distances.

---

## Notes:

* Initially attempted:

  * Clique detection
  * Path-based (“Antakshari”) interpretations
  * Graph centrality methods
    All failed due to incomplete edge data.

* Also encountered:

  * `networkx` limitations
  * `scikit-learn` + NumPy 2.x compatibility issues
    These were avoided by switching to a **pure NumPy solution**.

* The key turning point was recognizing that:

   **The embeddings, not the graph, encode the answer.**

## Resources:

* [NumPy documentation](https://numpy.org/doc/)
* [Understanding embeddings in ML](https://developers.google.com/machine-learning/crash-course/embeddings)
* [Pairwise distance computation](https://numpy.org/doc/stable/reference/generated/numpy.linalg.norm.html)



