---
title: "Lessons Learned During Udacity AIND, Part 4: Interesting Deep Learning Applications"
excerpt: "Part 4 of lessons learned during Udacity Artificial Intelligence Nanodegree covers the interesting applications of deep neural networks implemented in Term 2"
layout: post
---

We continue the series on lessons learned during the Udacity Artificial Intelligence Nanodegree with applications of deep neural networks. The first unit of AIND Term 2 covered deep learning and included projects to introduce applications of deep neural networks. Here I've selected a few of the projects and provided lessons learned for each.

## Image Classification

For this project, I built a dog breed classifier. It takes an image of a dog as input and outputs a guess as to which breed of dog is pictured.

### Best practice is to use convolutional neural network

Specifically, I built a convolution neural network (CNN) that takes as input an image of a dog and outputs a prediction of the breed of the dog, e.g. Alaskan Malamute, Parson Russel Terrier, Belgian Malinois.

### Reuse a pre-trained deep neural network via transfer learning

To build a good image classifier for a small dataset project, use transfer learning to leverage a pre-trained deep neural network.

The CNN architecture employed a technique known as transfer learning. With transfer learning, you reuse layers and weights of a pre-trained network to create a new network trained specifically on your project's dataset. The pre-trained network must have been trained for a similar task, in this case image classification.

Transfer learning is ideal for small dataset projects, as it allows you to leverage the results of large dataset projects without incurring the time and cost of building a large dataset and training a deep network yourself.

The transfer learning process is:
  * Select an appropriate deep CNN architecture and obtain a pre-trained model
  * Slice off the top -- the fully connected layers used for classification) -- of the pre-trained neural network
  * Add a new fully connected layer where the number of neurons matches the number of classes in the new data set (133 dog breeds in this case)
  * Randomize the weights of the new fully connected layer, while freezing all the weights from the pre-trained network
  * Train the network to optimize the weights of the new fully connected layer

### Optimize transfer learning by pre-computing the _bottleneck features_

In practice, transfer learning is implemented a bit differently. A "shallow" network is built that includes only an input layer and a classification layer. This shallow network is then trained with the so-called bottleneck features of the pre-trained network instead of the actual images. Here's how it works:

  * For each image in the training set, input that image into the pre-trained network and compute the output of the topmost CNN layer (not the output of the network's softmax classification layer, but an internal, intermediate output). This gives the bottleneck features for each image in the training set.
  * Train the shallow network using these bottleneck features as input. Each training example is a multi-dimensional vector, or tensor, rather than an image. As an example, the tensor could have dimensions, aka shape, of (7, 7, 512).

Specifically, for each image, the corresponding bottleneck feature vector is input to the neural network instead of the image itself. The bottleneck features can be thought of as an encoding of the image that's richer than the raw pixel values, allowing more performance from a single fully connected layer than would otherwise be possible.

### Libraries like Keras contain implementations of successful CNN architectures

The deep learning library Keras used for the project supports the CNN architectures that have proven to be successful in image classification tasks. The specific pre-trained networks supported by Keras are VGG-16, VGG-19, ResNet50, InceptionV3 and Xception.

### Tune the "shallow" network to the training set

To optimize the transfer learning architecture, some tweaks were made to the basic recipe to tune the number of parameters in the "shallow" neural network to the size of the dataset.

There were 6,680 images in the training set.

A global average pooling, or GAP, layer was placed immediately on top of the output of the pre-trained deep network. This had the effect of dramatically reducing the complexity (number of weights) of the network. In addition, a droput "layer" was added between the GAP layer and the fully connected 133-neuron softmax classifier layer. 20% dropout was used during training.

### Transfer learning can yield good results without a large dataset

The shallow model was trained for 20 epochs using the InceptionV3 bottleneck features precomputed for us. VGG-16 failed to achieve the target performance goal of 60% class prediction accuracy. I next tried InceptionV3 as it's one of the more recent ones in the set and achieved 80% accuracy on the test set.

### There's no intelligence in a neural network; you need to build that

A CNN, like any neural network, is a giant mathematical computation, not artificial intelligence. It doesn't truly "recognize" anything. The CNN knows only about image pixel values and performs a lengthly computation on those pixel values no matter what they represent.

If you build a dog breed classifier and you pass in an image of something other than a dog, say a human face, the CNN will still make a guess as to dog breed. This feature of CNNs was used to determine the "spirit dog" of a human in the image.

## Language Modeling and Text Generation

https://github.com/udacity/aind2-rnn


