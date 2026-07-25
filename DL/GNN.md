![[Pasted image 20260724220811.png]]
![[Pasted image 20260724220938.png]]

So, what is a GNN really?

A GNN is an ANN where the dataset is a graph. The question comes how do we even pass a graph to an ANN? Or why we even need a graph? Let's answer the first part.

There are 2 parts:

### 1. Neighbor Aggregation (Message Passing)

For every node in the graph, you look at its immediate connected neighbors, gather their feature vectors, and combine them using an aggregation function (like sum, average, or max). This updates each node's representation to hold context about its local neighborhood.

### 2. Feature Transformation (The ANN Part)

Once each node has its aggregated feature vector, you pass that vector into a standard ANN layer (applying weight matrix W, bias b, and an activation function like ReLU).

During training, backpropagation updates the weights and biases of that ANN based on the prediction loss (e.g., whether a student gets placed).

Here's an example of the flow:

### 1. Input (Feature Vector)
You feed the model a numeric vector $x$ representing student features:

- $x_1$: CGPA
- $x_2$: Number of internships
- $x_3$: Coding assessment score
- $x_4$: Communication rating

### 2. Hidden Layers (Weighted Sum & Activation)
For each node in a hidden layer:

- **Weighted Sum:** It calculates $z = Wx + b$ (combining inputs with learned weights and biases).
- **Activation:** It applies a non-linear activation function like ReLU to $z$ to capture complex, non-linear relationships.
- **Forwarding:** The activated result is sent forward as the input to the next layer.

### 3. Output Layer (Placement Probability)
Since placement is a binary classification problem (Placed vs Not Placed), the single output node uses a Sigmoid activation function:

$$
\hat{y} = \sigma(z) = \frac{1}{1 + e^{-z}}
$$

Because Sigmoid squashes any real value into a range between 0 and 1, the output $\hat{y}$ directly represents the probability of getting placed (e.g., $\hat{y} = 0.85$ means an 85% chance of placement).

### 4. Loss & Backpropagation

- **Loss Calculation:** A loss function like Binary Cross-Entropy compares $\hat{y}$ against the actual ground truth $y \in \{0, 1\}$.
- **Backpropagation:** Gradients are calculated backward through the network using the chain rule, updating $W$ and $b$ via gradient descent so the network makes better predictions next time.