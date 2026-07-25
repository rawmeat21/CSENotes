It is simply a method to train neural networks

![[Pasted image 20260722022906.png]]


![[Pasted image 20260722023132.png]]

Here, initially at step 0, all weights are 1 and biases are 0.
Then, we pick the 1st row and do forward propagation to make a predict. It will likely be wrong because of the random weights.

**We use a linear activation function for this example and a mse loss function

To change the output we need to change the prediction.

![[Pasted image 20260722023722.png]]

The final prediction depends on 5 things.

![[Pasted image 20260722023835.png]]

Changing weights is easy, but what about the outputs??

But changing the outputs would lead to other things being changed.

Backpropagation is basically propagrating the errors to previous layers.

![[Pasted image 20260722024112.png]]

This update happens for every single node!!

We go back and update the weights and biases using gradient descent.

 ![[Pasted image 20260722024333.png]]

How to calculate the derivatives?

![[Pasted image 20260722024630.png]]

lets try to find the first one:

![[Pasted image 20260722024614.png]]

![[Pasted image 20260722024820.png]]
![[Pasted image 20260722025158.png]]

This is the overall algo:

![[Pasted image 20260722025831.png]]

We usually run this algo a few times (epochs) until the loss is minimised.


### Backpropagation with classification

![[Pasted image 20260722183528.png]]

Here we use sigmoid as activation function and binary cross entropy loss.

For more: https://www.youtube.com/watch?v=ma6hWrU-LaI&list=PLKnIA16_RmvYuZauWaPlRTC54KxSNLtNn&index=17


### Why does backpropagation work?

![[Pasted image 20260722183857.png]]

The calculation of weights is where the magic happens.

Loss function is a function of all the parameters.

Backpropagation works because it finds the weights and biases such that the loss is minimum.

![[Pasted image 20260722185513.png]]

Why do we subtract slope?? - to find the minimum pt.

If slope is +ve, we want to move left (decrease)
if slope is -ve, we want to move right (increase)

Why learning rate? - To smoothen out the steps to reach minimum (this is just from gradient descent)


Additionally, backpropagation uses memoization to calculate the derivatives. It stores intermediate derivatives so that they don't have to be recalculated.

![[Pasted image 20260722192135.png]]



### Does backprop happen only after a full prediction is made?

**Yes.** Backpropagation happens **only after a complete forward pass is finished** and a prediction (y^​) has been generated at the output node.

It does **not** execute layer-by-layer during the forward pass.

Why is it called backpropagation though? - Its because it calculates derivatives, and derivatives current layers depend on previous layers' derivatives.





