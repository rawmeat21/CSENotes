![[Pasted image 20260722190054.png]]

![[Pasted image 20260722190307.png]]

This is generally seen for sigmoid functions, which have small derivatives (0 - 0.5)

![[Pasted image 20260722190349.png]]

A derivative becomes so small that the weight doesn't change at all.

This is seen more so in deep neural networks with lots of layers, because the derivative decreases even more.


### How to know if this is occuring??

1. If the loss is not decreasing much with number of epochs.
2. plot weights vs epochs graph.


### How to solve this?

1. Reduce model complexity (reduce the layers).
2. using ReLU activation function

![[Pasted image 20260722190935.png]]

Its derivative its either 0 or 1. We can multiply a lot of 1s with no problem. But there is a "dying ReLU" problem, where there is no change if derivative becomes 0.

3. Proper weight initialization
4. Batch normalisation
5. Residual network

### Exploding gradient problem

Big derivatives when multiplied turn into a huge final product. This causes the weights can grow explosively.

Fix - gradient clipping