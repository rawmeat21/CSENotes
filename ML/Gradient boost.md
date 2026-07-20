![[Pasted image 20260720134617.png]]

Gradient boost is a boosting algo, so it works sequentially. 

At step 1: We find model 1 (M1). M1 simply predicts the mean of all output (here, salary) values.

Then, we need a loss function to know how much errors our model made. 
This is simply = (actual - predicted) = pseudo residual

We need to communicate the errors in M1 to M2.

**It is shown through research that GD works best with DT, so in M2 we use DTrees.

Step 2:

In this stage, the input cols are same.
BUT, the output col is the res1 column!
So this DT predicts the errors made by M1.

![[Pasted image 20260720135223.png]]

![[Pasted image 20260720135334.png]]

Now, if we wanted a prediction of salary by using M2, the method is:

![[Pasted image 20260720135418.png]]

Yep. Though, we want to use learning rate to prevent overfitting.

![[Pasted image 20260720135524.png]]


Now, at this stage, we also need to find the errors:

![[Pasted image 20260720135825.png]]

Remember, the loss function is = actual - predicted

prediction = prediction of salary made by M2
actual = actual salary

Notice, that res1 is more closer to zero than res2, ie, |res1| <= |res2|


Step 3:

Same thing, our output column is res2. We use decision tree for prediction.

![[Pasted image 20260720140118.png]]

![[Pasted image 20260720140232.png]]

This is the prediction formula. Learning rate is kept same for all.


### Adaboost vs Gradient boost

1. Adaboost uses decision stumps (usually), so max number of leaf nodes is 2. Gradient boost uses decision trees with number of leaves 8 to 32.
2. Learning rate - Adaboost uses different weights for different models to assign importance of each one's prediction. Gradient boost, however, uses the same learning rate for all models.