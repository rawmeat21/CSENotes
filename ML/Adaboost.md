Adaboost is a stage wise additive method.

![[Pasted image 20260720122306.png]]

Step 1: Make the split (choose the one with most information gain)

We see that there are + pts in blue region when they should actually be in red. What happens next is the importance of these pts increases.

We assign a weight to the 1st decision stump. 

![[Pasted image 20260720122721.png]]

Step 2: we again make a split, this time its on a different region because importance of those + pts increased. But again, we end up with 2 - pts in blue region. Hence, we increase their importance. 

![[Pasted image 20260720122939.png]]

Step 3: we again do a split and stop here. The model still made some mistakes with 3 + pts.

![[Pasted image 20260720123050.png]]

h_i(x) are the prediction model functions of the 3 decision stumps.

Then we use this formula and find the sign (+ or -). This is our final classifier model.

![[Pasted image 20260720123214.png]]

Consider a single data point. We use all 3 models to get the prediction, multiply by weights and add them all. Check the sign, and we have our prediction.


### How weights are assigned

![[Pasted image 20260720124249.png]]

Initially we assign weights to each row = (1 / number of data rows)

![[Pasted image 20260720124427.png]]

Next we develop a decision stump based on this data. It makes mistakes at some places.

Also, note the formula for alpha: When error = 0.5, alpha = 0, when error -> 1, alpha -> -inf, when error -> 0, alpha -> inf

![[Pasted image 20260720124630.png]]

How to find error? error = sum of weights of all the rows where model was wrong.

![[Pasted image 20260720124734.png]]


But, we also need to increase the weights of those rows which were misclassified. We also need to decrease weights for the correct rows. 

![[Pasted image 20260720124904.png]]

We also need to ensure the sum of weights = 1. For that we normalise by dividing each row with the sum of all weights.

![[Pasted image 20260720125127.png]]


Now we do upsampling.

Upsampling - Boosting the misclassified rows.

![[Pasted image 20260720125343.png]]

We convert the rows into ranges.
Then we pick 5 random values, then pick the row which contains a particular random value. Misclassfied rows will have higher range and hence more chance of getting selected.

Finally we obtain a new dataset with just those rows. 
Now we can redo step 1.