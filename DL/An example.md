### Step 1: Establish the Dataset and the Network Setup

We will track six specific candidates (P1 through P6) with three features (IQ, CGPA, Age) to predict Placement (the Ground Truth).

This image visualizes our raw data on a 2D feature space (IQ vs CGPA), showing how complex the boundary might be. We also see the _empty_ ANN structure (3 Inputs, 2 Hidden nodes, 1 Output node).

![[Pasted image 20260724232256.png]]

Let’s give our network some illustrative _random initial weights_ for this node:

Given weights $w_{IQ} = 0.2$, $w_{CGPA} = 1.1$, $w_{Age} = -0.3$, bias $b = 0.1$ $$ z = (110 \times 0.2) + (8.5 \times 1.1) + (22 \times -0.3) + 0.1 $$ $$ z = 22 + 9.35 - 6.6 + 0.1 = 24.85 $$

The linear sum is $z = 24.85$. If we used only this, the network could only draw a straight flat line, which fails on complex data (as shown in the comparison visual in Image 19).

We need to break that linearity.

We apply the **ReLU Activation Function** to the sum ($z$):

**ReLU Formula:**
$$
f(z) = \max(0, z)
$$

**Calculation:**
$$
f(24.85) = \max(0, 24.85) = 24.85
$$


![[Pasted image 20260724232904.png]]

### Step 3: The Output Layer – Probability (Activation #2) & Error (Loss Function)

Now we zoom into the final step: the **Output Node**. We will use a _different_ activation function and then our loss function.

#### Activation Function #2: The Probability Force

The task of the final node is to combine the signals from Hidden A and Hidden B to predict placement.

#### The Job:

To force _any_ output number into a probability range exactly between `0.0` (0% chance of placement) and `1.0` (100% chance).

**VERY IMPORTANT: The activations of a hidden layer act as new, transformed feature representations of the input data.**

**Calculation for P1 (Output Probability):**

- Input signal from Node A (from Image 19): $a_A = 24.85$
- Input signal from Node B (illustrative): $a_B = 5.2$
- Final Output Weights: $w_A = 0.1$, $w_B = 0.3$, bias $= -3.0$

**Final linear sum ($z_{final}$):**
$$
z_{final} = (24.85 \times 0.1) + (5.2 \times 0.3) - 3.0
$$
$$
z_{final} = 2.485 + 1.56 - 3.0 = 1.045
$$

We cannot output `1.045`. What does that mean? Is P1 placed or not? We need a percentage chance.

We apply **Sigmoid Activation**:

**Sigmoid Formula:**
$$
\sigma(z) = \frac{1}{1 + e^{-z}}
$$

**Calculation:**
$$
\sigma(1.045) = \frac{1}{1 + e^{-1.045}} \approx 0.74
$$

The Network's Activated Prediction ($\hat{y}$) is **0.74** (74% Probability of Placement).

#### The Loss Function: The Reality Check

The network has made its non-linear guess (ŷ=0.74). We are done propagating features.

_Now the learning begins._ We are looking at the **Binary Cross-Entropy Loss Function**.

#### The Job:

This function exists only during training. It compares the _Prediction_ (what the network thinks) against the _Ground Truth_ (what is actually true) for a data point. It returns a single number (the Loss or Error), which tells the network exactly how incorrect its guess was.

If a student was _Placed_ (Ground Truth = 1), a prediction of ŷ=0.98 gives very low loss. A prediction of ŷ=0.15 gives _catastrophically high_ loss. If the answer is 1, Binary Cross-Entropy gives high penalty for low probability. If the answer is 0, it gives high penalty for high probability.

**Calculation for P1 (Ground Truth y=1):**

- Network Prediction ($\hat{y}$): 0.74 (74%)
- Ground Truth ($y$): 1 (Placed, from Image 18)
- The Job: We compare these. Binary Cross-Entropy is a logarithmic distance. The formula looks like this:

$$
L = -\left[y \log(\hat{y}) + (1-y)\log(1-\hat{y})\right]
$$

**Simplified Calculation (BCE Loss for $y=1$):**
$$
L = -\log(0.74) \approx 0.301
$$

The network made an error (Loss) of **0.301**.

- This error value (0.301) is the entire goal. During training (Image 3 in the previous turn), this specific error signal is passed to the optimizer (like Backpropagation) to update every weight ($w_1$, $w_A$, $w_{CGPA}$, etc.) ever so slightly so that next time Candidate P1 (or similar candidates) is calculated, the output probability might be 0.82 instead of 0.74, and the resulting loss (0.198) will be lower.

![[Pasted image 20260724233533.png]]


### Is the loss function calculated only once?

It depends on whether you mean **per training step** or **across the whole training process**:

1. **Per Training Iteration (Forward Pass):** **Yes, exactly once.** For a single input (or single mini-batch), the loss function L(y^​,y) is evaluated **once** at the very end of the forward pass to produce a scalar error value.
    
2. **Across Full Model Training:** **No.** The loss function is evaluated repeatedly, once for every forward pass performed across every batch and epoch in training.

```
[ Step 1: Forward Pass ]
  Input (X) ---> Layer 1 ---> Layer 2 ---> Output Layer ---> Prediction (ŷ)
                                                                 |
[ Step 2: Loss Evaluation ]                                      v
  Loss Function L(ŷ, y) <--------------------------------  Target (y)
          |
          v
[ Step 3: Backpropagation ]
  Input Layer <--- Layer 1 <--- Layer 2 <--- Output Layer (Compute ∂L/∂W)
          |
          v
[ Step 4: Weight Update ]
  W_new = W_old - η * (∂L/∂W)
```


