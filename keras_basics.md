
# Keras basics

## Layers: the building blocks of deep learning

- The fundamental data structure in neural networks is the **layer**.
- A layer is a data-processing module that takes as input one or more tensors and that outputs one or more tensors.
- Some layers are stateless, but more frequently layers have a state: the layers **weights**, one or several tensors learned with stochastic gradient descent, which together contain the network's **knowledge**.

---

- Different layers are appropriate for different tensor formats and different types of data processing.

- Simple vector data, stored in 2D tensors of shape ```(samples, features)``` is often processed by **densely connected layers**, 
also called **fully connected layers** (the ```Dense``` class in Keras).

- Sequence data, stored in 3D tensors of shape ```(samples, timesteps, features)```, is typically processed by **recurrent** layers 
such as **long-short term memory (LSTM)** layer.

- Image data, stored in 4D tensors, is usually processed by 2D **convolutional** layers ```Con2D```. 

Links to Keras documentation:

- [Core layers](https://keras.io/layers/core/)
- [Convolutional layers](https://keras.io/layers/convolutional/)
- [Recurrent layers](https://keras.io/layers/recurrent/)

---

- You can think of layers as LEGO bricks of deep learning.
- Building deep-learning models in Keras is done by combining compatible layers to form useful data-processing pipelines.
- Layer compatibility means that every layer will only accept input tensors of a certain shape and will return output tensors of a certain shape.
- When using Keras, you don't have to worry about compatibility, because the layers you add to your model are dynamically built to match the shape of the incoming layer.

---

```
%tensorflow_version 2.x
import tensorflow as tf

# define a sequential model
network = tf.keras.models.Sequential()

# add a dense layer with relu activation function
# it takes as input a batch of vectors of length 28*28 
network.add(tf.keras.layers.Dense(512,
                                  activation='relu',
                                  input_shape=(28 * 28,)))

# no need to specify input_shape for second dense layer 
# it is a dense layer with softmax activation function
network.add(tf.keras.layers.Dense(10,
                                  activation='softmax')) 
```

The second layer didn't receive an input shape argument - instead, it automatically inferred its input shape as being the output shape of the first layer.

## Models: networks of layers

- A deep-learning model is a directed, acyclic graph of layers. 
- The most common topology is a sequential (linear) stack of layers, mapping a single input to a single output. These can be implemented using ```tf.keras.models.Sequential()```.  See [Sequential model](https://keras.io/getting-started/sequential-model-guide/).
- Initially, we will only work with linear stacks of layers. 
- Later, we will also look at other network topologies such as two-branch networks, multi-head networks, and inception blocks.
- Such topologies are implemented using [the functional API](https://keras.io/getting-started/functional-api-guide/).

---

- The topology of a network defines a **hypothesis space**. 
- By choosing a network topology, you constrain your **space of possibilities** (hypothesis space) to a specific series of tensor operations, mapping input data to output data.
- You'll be then searching for a good set of values for the weight tensor involved in these tensor operations using stochastic a variant of gradient descent.
- Picking the right network architecture is more art than a science. We will study explicit principles for building neural networks and develop intuition as to what works or doesn't for specific problems.

## Anatomy of a sequential network with dense layers

```
                              Input X
                                |
                                V
+-----------+       +-------------------------+
|  Weights  |  -->  |          Layer          |
+-----------+       |  (data transformation)  |
                    +-------------------------+
                                |
                                .
                                |
                                V
+-----------+       +-------------------------+
|  Weights  |  -->  |          Layer          |
+-----------+       |  (data transformation)  |
      ^             +-------------------------+                         
      | update                  |
      | weight                  V
      |                 +---------------+              +----------------+
      |                 |  Predictions  |              |  True targets  |
      |                 |      ^y       |              |       y        |
      |                 +---------------+              +----------------+
      |                         |                             |
      |                         |     +-----------------+     |
      |                         +---->|  Loss function  |<----+
+-------------+                       +-----------------+
|  Optimizer  |                                |
+-------------+                                |
      ^                                        V
      |                                +--------------+      
      +--------------------------------|  Loss score  |
                                       +--------------+
```

## Loss functions & optimizers

- Once the network architecture is defined, you choose the loss function and the optimizer to configure the learning process:

  - **Loss function (objective function)** 
  - **Optimizer**

- The loss function is the function that will be minimized during training. For supervised learning problems, it measures the deviation between the predicted value and the target for the training examples. See [loss functions](https://keras.io/losses/)

- The optimizer determines how the network will be updated based on the loss function. It implements a specific variant of stochastic gradient descent (SGD). See [Optimizers](https://keras.io/optimizers/).

- A metric function is used to judge the performance of the model.  See [metrics](https://keras.io/metrics/)

---

```
network.compile(optimizer='rmsprop',
                loss='categorical_crossentropy',
                metrics=['accuracy'])
                
history = network.fit(train_images, 
                      train_labels, 
                      epochs=epochs, 
                      batch_size=128, 
                      validation_data=(test_images, test_labels))
```

---

- Choosing the right objective function for the right problem is extremely important: your network will take any shortcut it can, to minimize the loss.

- Fortunately, there are simple guidelines you can use to choose the correct loss for common problems such as classification, regression, and sequence prediction.

| Problem type              | Last layer activation  | Loss function              | 
|:-:                        |:-:                     |:-:                         |
| Binary classification     | sigmoid                | binary_crossentropy        |
| Multiclass, single-label  | softmax                | categorical_crossentropy   |
| Mutlticlass, multi-label  | sigmoid                | binary_crossentropy        |
| Regression to real values | none                   | mse                        |
| Regression to \[0,1\]     | sigmoid                | mse or binary_crossentropy |
