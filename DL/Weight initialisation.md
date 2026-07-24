## Why it matters?

If you don't do it properly, you may get:

- vanishing gradient
- exploding gradient
- slow convergence

## What shouldn't you do?

1. Don't initialise all weights to 0.

When you use ReLU, at first, your activation function will be 0. But then your derivative is also 0. So, no change happens and the model doesn't train.

Same problem with tanh. 

**What about sigmoid?**

It will give 0.5. But is can be shown that the weights would become same. This will cause the NN to behave like a single perceptron! and the curve will be a line.


2. No non-zero constant value

ReLU will behave like linear model (single perceptron)


3. Random init

2 cases: small or large random values.

Small random values may give vanishing gradient problem for tanh and sigmoid. ReLU will give slow training.

Large values can cause slow training or vanishing gradient. There will be saturation because of big values for sigmoid and tanh. ReLU will give unstable gradient.



### What to use?

![[Pasted image 20260722223204.png]]

Use xavier/glorat with tanh or sigmoid.
Use He with ReLU




