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

An effective strategy for applying dropout during training of a CNN is to gradually increase the dropout percentage from lower layers to higher ones. Start with a low dropout rate in the lower layers (near the input layer) of say 0.2, and gradually increasing dropout to about 0.5.

As dropout zeros out some neurons, it limits the amount of information that flows through a layer during training. In the lower layers of a CNN there are typically fewer feature maps (e.g. 16), and therefore fewer paths for the signals to flow through. Low dropout rates here ensures that enough information is getting through to the higher layers. By constrast, in the higher layers there are more feature maps (e.g. 512) and more paths for the signals to travel. Higher dropout rates can be used here.

### Leverage pretrained model using transfer learning

Transfer learning is a technique common in image classification systems whereby a pretrained deep neural network model is used as the lower layers of the new network. The top layer of the pretrained model, typically a fully-connected layer with a softmax output, is replaced with one or more fully-connected layers. During training, only the weights of the new top layers are learned. The weights of the pretrained model stay fixed.

Why does this work? The pretrained network has learned a set of features that are generally useful in image classification tasks. The new top layers are trained to combine those features to classify images into the new set of categories.

### Precompute bottleneck features to use during training of the new model

We've applied transfer learning to arrive at the architecture for our image classifier. We've taken a freely available, pretrained image classification neural network, removed its classification layer then added our own. We now need to train this network. We collect our training images, and pause here because we notice an optimization.

Since we're not going to be training the layers borrowed from the pretrained network (we're going to take all those weights as-is), we observe that every time we run a training example through those layers, we'll get the same result. We can then compute the output vector from the pretrained network for each training example and store it. During training we'll use those stored vectors. If we're planning on running dozens of epochs, that's a lot of computational savings.

The precomputed vector outputs of the pretrained network on the new training dataset are called _bottleneck features_.

