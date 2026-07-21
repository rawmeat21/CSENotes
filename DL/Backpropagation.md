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




