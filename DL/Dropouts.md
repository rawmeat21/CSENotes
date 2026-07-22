### How to prevent overfitting in NN

- Add more data
- reduce complexity
- early stopping
- regularization
- dropouts

![[Pasted image 20260722202814.png]]

Dropout is basically dropping some nodes from input / hidden layers randomly. 

At each epoch, we randomly switch off some of the nodes. So, the number of nodes keep decreasing with each epoch.

Technically, our **NN changes every epoch**.

#### Why does it work?

First, a large number of nodes will try to capture every kind of variation in the data. 

Second, a node may favour another node (higher weight). If we delete that node, then we can prevent that kind of bias towards some pattern.

It's technically like ensemble learning over NN. Its similar to Random forest in this manner. 


#### So, in the final model, do the nodes go away?

No, they stay. They are modified though, according to the following rule: 

W := W*(1 - p), where p = fraction of rows that get removed randomly

So, basically, W -> W * (probability of not getting removed)


