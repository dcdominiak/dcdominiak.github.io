---
title: "Lessons Learned During Udacity AIND, Part 3: Deep Learning with Convolutional Neural Networks and Recurrent Neural Networks"
excerpt: "Part 3 of lessons learned during Udacity Artificial Intelligence Nanodegree covers deep learning with convolutional neural networks (CNNs) and recurrent neural networks (RNNs)."
layout: post
---

## Convolutional Neural Networks

### How to think about the filtering done by the layers of a CNN

The first layer of a convolution neural network performs a set of convolutional filtering operations on the input image, one for each feature map. The exact type of filter in each case is not dictated as in a traditional image processing system but is learned by the network. What about the higher layers?

Suppose the input to a CNN is a 228x228 color image. A filter in the first layer with a 3x3 kernel filters a 9x9 grid of pixels in the input image across three color channels:

neuron = w_r * red_channel_filter() + w_g * greeen_channel_filter() + w_b * blue_channel_filter()

Suppose further the second to last convolutional layer is 7x7x512. A filter in the final layer then takes as input a 7x7 downsampled “image” with 512 channels. Unlike the first convolutional layer where the channels were color channels in the input image, these channels are the outputs of filters in the previous layer and it's quite difficult to put nice labels on them. But the computation is similar. Each neuron outputs a single value:

neuron = channel_1_filter(3x3) + channel_2_filter(3x3) + … + channel_512_filter(3x3)

A filter’s output “pixels” are called its activation map, and each CNN layer contains many such maps.

### Strategy for dropout during training of a CNN

An effective strategy for applying dropout during training of a CNN is to gradually increase the dropout percentage from lower layers to higher ones. Start with a low dropout rate in the lower layers (near the input layer) of say 0.2, and gradually increase dropout to about 0.5.

Why does this strategy work well? As dropout zeros out some neurons, it limits the amount of information that flows through a layer during training. In the lower layers of a CNN there are typically fewer feature maps (e.g. 16), and therefore fewer paths for the signals to flow through. Low dropout rates here ensures that enough information is getting through to the higher layers. By constrast, in the higher layers there are more feature maps (e.g. 512) and more paths for the signals to travel. Higher dropout rates can be used here.

### Leverage pretrained model using transfer learning

Transfer learning is a technique common in image classification systems whereby a pretrained deep neural network model is used as the lower layers of the new, custom network. The top layer of the pretrained model, typically a fully-connected layer with a softmax output, is replaced with one or more fully-connected layers. During training, only the weights of the new top layers are learned. The weights of the pretrained model stay fixed.

Why does this architecture work well? The pretrained network has learned a set of features that are generally useful in image classification tasks. The new top layers are trained to combine those features to classify images into the new set of categories.

### Precompute bottleneck features to use during training of the new model

To recap, best practice is transfer learning to arrive at the architecture for an image classifier:

  * Take a freely available, pretrained image classification deep neural network,
  * Remove its classification layer, then
  * Add a new classification layer suited to your specific classification task.

We now need to train this network. We collect our training images, but pause here because we notice an optimization.

Since we're not going to be training the layers borrowed from the pretrained network (we're going to take all those weights as-is), we observe that every time we run a training example through those layers, we'll get the same result. We can therefore compute the output vector from the pretrained network for each training example and store it. During training we'll use those stored vectors. If we're planning on running dozens of epochs, that's a lot of computational savings.

The precomputed vector outputs of the pretrained network on the new training dataset are called _bottleneck features_.

## Recurrent Neural Networks

A recurrent neural network is a neural network specialized for processing a sequence of values.

### An RNN maps an input sequence to a fixed-size vector

An RNN learns a mapping of an input sequence to a fixed-size vector. It encodes the input sequence into a hidden state vector. The size of the vector controls the learning capacity of the RNN.

The size of the RNN hidden state vector is a hyperparameter of an RNN layer. Any size can be used, but for good performance it should be matched to the amount of training data.

The hidden state vector is computed via a weight matrix multiplication, and each weight represents a parameter that must be learned during training. The larger the state vector, the more model parameters that must be learned, and the more model parameters that must be learned, the more training data is needed, same as with a feedforward neural network layer.

### An RNN learns a single function that gets applied at each time step

Rather than learning a specific function for each time step, the RNN learns a general function that gets applied at all time steps.

### An RNN is equivalent to an entire hidden layer in a neural network

The elements of the hidden state vector of an RNN are analogous to the neurons of a fully-connected feedforward hidden layer.

An RNN cell seems like it would be analogous to a single neuron in a feedforward layer, but that's not the right way to look at it.

The hidden state of an RNN cell is a vector of real numbers, and it's this vector that is output to the next layer in the neural network architecture. The RNN cell can therefore be thought of as taking the place of all of the neurons in the feedforward hidden layer, plus introducing the extra complexity of the recurrence part.

### An RNN learns by applying backpropagation in the _unrolled graph_

Even though an RNN cell is constructed to compute a single output vector from a single input vector, in practice it isn't typically used that way. We're dealing with sequences after all, not individual samples.

Instead, many elements of the input sequence are captured and processed one after the other, and only the output of the final step is taken as the RNN output. The intermediate outputs are discarded. To do this, we repeatedly operate the RNN cell, once for each input sequence element. This repeated computation of the RNN cell's output is called _unrolling the graph_. The RNN cell is trained by applying backpropagation to the unrolled graph, called backpropagation-through-time (BPTT).

The longer the time dependencies that we need to capture, the more we'll need to unroll the graph. As an example, it's not uncommon for a language model RNN to process 100 or more sequence elements at once. This allows the neural network to learn dependencies between elements separated by up to 100 words (or characters).

### LSTMs were developed because RNNs were too hard to train

LSTM RNNs were developed (twenty years ago, in 1997) to find a way to effectively train deep RNNs that suffer from vanishing/exploding gradients. During backpropagation-through-time, the local gradient computation for cells early in the unrolled graph is a long series of multiplications that drive the output to zero (most of the time) or infinity (rarely). This became known as the vanishing/exploding gradient problem.

LSTM RNNs introduce paths where the gradient can flow for a long number of time steps through the unrolled graph without vanishing. This change in RNN cell architecture allowed RNNs to be trained effectively.

Bonus: The vanishing/exploding gradient problem wasn't the only issue. Even if the vanishing gradients were mitigated, simple RNNs were unable to learn dependencies across any more than 10-20 time steps. The problem of training a simple (somtimes called "vanilla") RNN to learn long-term dependencies is described well in section 10.7 of the book [Deep Learning](http://www.deeplearningbook.org/contents/rnn.html):
“...whenever the model is able to represent long-term dependencies, the gradient of a long-term interaction has exponentially smaller magnitude than the gradient of a short-term interaction. This means not that it is impossible to learn, but that it might take a very long time to learn long-term dependencies, because the signal about these dependencies will tend to be hidden by the smallest fluctuations arising from short-term dependencies.”
