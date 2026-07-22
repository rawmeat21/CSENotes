1. Fine tune hyperparameters:

- **number of epochs** - You can use early stopping. Early stopping detects the point of overfitting due to increasing number of epochs and stops the training process.
- **number of hidden layers** - often its better to use more layers and less neurons per layer rather than less layers and more neurons. This is because of NN's property of Representational learning.
- **neurons per layer** - generally the more the better, decrease if you see overfitting
- learning rate
- **optimizer**
- **batch size** - usually smaller batch sizes (8 - 32) gives better results, while larger gives faster results. A different approach is we can vary the batch size as number of epochs increase (called warming up learning rate)
- **activation function**
- **normalisation** - normalising inputs, batch normalisation, normalise activations
1. Solving NN problems:

- Vanishing or exploding gradient
- not enough data - Transfer learning, unsupervised pretraining
- slow training - Use optimizers, learning rate scheduler
- overfitting - regularization and dropouts

