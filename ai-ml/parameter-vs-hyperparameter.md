# Parameter vs Hyperparameter

From [Machine Learning Mastery](https://machinelearningmastery.com/difference-between-a-parameter-and-a-hyperparameter/):

* **Hyperparameter**: A configuration that is external to the model or optimizer, and whose value cannot be estimated from data.
* **Parameter**: A configuration that is internal to the model or optimizer, and whose value can be estimated from data.

Examples:

* k-nearest neighbors (KNN) algorithm looks at the `K` closest points in the dataset. `K` is predetermined before running the algorithm. `K` is a hyperparameter.
* Learning rate `alpha` in gradient descent external to the solver. `alpha` is a hyperparameter.
* Coefficients in linear or logistic regression are learned from training. Coefficients are model parameters.
* Weights in a neural network are learned from training and are not determined beforehand. Weights are NN parameters.
