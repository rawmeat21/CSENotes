It is an alogorithm used for supervised ML.

It is also the building block of DL.

![[Pasted image 20260721211604.png]]

Now you should look at my handwritten notes.

Lets discuss the problem with a single perceptron: NON LINEAR DATA

![[Pasted image 20260721223327.png]]

We needed a curve for this, but perceptron will draw a line!

![[Pasted image 20260721223456.png]]

Now, we'll be using the LoR method of perceptron for the rest of this part.

![[Pasted image 20260721224102.png]]

![[Pasted image 20260721224133.png]]

We combine the models.

![[Pasted image 20260721224141.png]]

And do some smoothening.

 ![[Pasted image 20260721224339.png]]

Lets see how we would do this. Consider a single pt, take the 2 probabilites you get by different models (0.8 and 0.7), add them up (= 1.5) and pass them to the sigmoid function to get new probability.



Now lets consider something else, what if we wanted to give more importance to a particular model? What to do?? USE WEIGHTS!

![[Pasted image 20260721224709.png]]


What if we wanted to add some bias??

![[Pasted image 20260721224919.png]]

Just add it and pass it to the sigmoid function.


![[Pasted image 20260721225004.png]]

Did you see something? We basically have another perceptron!!

![[Pasted image 20260721225051.png]]

![[Pasted image 20260721225206.png]]

Complete picture.

![[Pasted image 20260721225242.png]]

voila!

![[Pasted image 20260721225306.png]]

This whole process is called forward propagration.

### Changes we can make to this neural network archtecture

![[Pasted image 20260721225539.png]]

We can increase number of nodes to catch more non linearity.

![[Pasted image 20260721225647.png]]

We can add more nodes / variables / features.

![[Pasted image 20260721225740.png]]

We can add more nodes to output. This is usually used when we have > 2 classes (multiclass classification).

![[Pasted image 20260721225904.png]]

We can increase number of layers to capture complex boundaries.


