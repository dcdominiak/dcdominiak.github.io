---
title: "Lessons Learned During Udacity AIND, Part 4: Interesting Deep Learning Applications"
excerpt: "Part 4 of lessons learned during Udacity Artificial Intelligence Nanodegree covers the interesting applications of deep neural networks implemented in Term 2"
layout: post
---

We continue the series on lessons learned during the Udacity Artificial Intelligence Nanodegree with applications of deep neural networks. The first unit of AIND Term 2 covered deep learning and included projects to introduce applications of deep neural networks. Here I've selected a few of the projects and provided lessons learned for each.

## Image Classification

For this project, I built a dog breed classifier. It takes an image of a dog as input and outputs a guess as to which breed of dog is pictured.

### Best practice is to use a convolutional neural network for image classification

Specifically, I built a convolution neural network (CNN) that takes as input an image of a dog and outputs a prediction of the breed of the dog, e.g. Alaskan Malamute, Parson Russel Terrier, Belgian Malinois.

### Reuse a pre-trained deep neural network via transfer learning

To build a good image classifier for a small dataset project, use transfer learning to leverage a pre-trained deep neural network.

The CNN architecture employed a technique known as transfer learning. With transfer learning, you reuse layers and weights of a pre-trained network to create a new network trained specifically on your project's dataset. The pre-trained network must have been trained for a similar task, in this case image classification.

Transfer learning is ideal for small dataset projects, as it allows you to leverage the results of large dataset projects without incurring the time and cost of building a large dataset and training a deep network yourself.

The transfer learning process is:
  * Select an appropriate deep CNN architecture and obtain a pre-trained model
  * Slice off the top -- the fully connected layers used for classification -- of the pre-trained neural network
  * Add a new fully connected layer where the number of neurons matches the number of classes in the new data set (133 dog breeds in this case)
  * Randomize the weights of the new fully connected layer, while freezing all the weights from the pre-trained network
  * Train the network to optimize the weights of the new fully connected layer

### Optimize transfer learning by pre-computing the _bottleneck features_

In practice, transfer learning is implemented a bit differently. A "shallow" network is built that includes only an input layer and a classification layer. This shallow network is then trained with the so-called _bottleneck features_ of the pre-trained network instead of the training images. Here's how it works:

  * Use the pre-trained network to compute the bottleneck features for the training set.
    * For each image in your training set, input that image into the pre-trained network and compute the output of the topmost CNN layer (not the output of the network's softmax classification layer, but an internal, intermediate output). This gives the bottleneck features for each image in the training set.
  * Train the shallow network using these bottleneck features as input.
    * Each training example is now a multi-dimensional vector, or tensor, rather than an image. As an example, the tensor could have dimensions, aka shape, of (7, 7, 512).
    * Use a categorical crossentropy loss function during training.

The difference from the build-your-own deep neural network approach bears repeating. During training, each image is replaced with its pre-computed bottleneck feature vector. The bottleneck features can be thought of as an encoding of the image that's richer than the raw pixel values, allowing more performance from a single fully connected layer than would otherwise be possible.

The same holds true when the neural network is used to classify images. Each new image must first be run through the pre-trained deep neural network to compute its bottleneck feature vector. (With a deep network of 50 layers or more, this is not a trivial computation.)

### Libraries like Keras have implementations of successful CNN architectures

The deep learning library Keras used in this project supports the CNN architectures that have proven to be successful in image classification tasks. The specific pre-trained networks supported by Keras are VGG-16, VGG-19, ResNet50, InceptionV3 and Xception.

### Tune the "shallow" network to the training set

To optimize the transfer learning architecture, some tweaks were made to the basic recipe to tune the number of parameters in the "shallow" neural network to the size of the dataset. (There were 6,680 images in the training set.)

A global average pooling, or GAP, layer was placed immediately on top of the output of the pre-trained deep network. This had the effect of dramatically reducing the complexity (number of weights) of the network. In addition, a droput "layer" was added between the GAP layer and the fully connected 133-neuron softmax classifier layer. 20% dropout was used during training.

### Transfer learning can yield good results without a large dataset

The shallow model was trained for 20 epochs using the InceptionV3 bottleneck features precomputed for us. VGG-16 failed to achieve the target performance goal of 60% class prediction accuracy. I next tried InceptionV3 as it's one of the more recent ones in the set and achieved 80% accuracy on the test set.

### There's no intelligence in a neural network; you need to build that

A CNN, like any neural network, is a giant mathematical computation, not artificial intelligence. It doesn't truly "recognize" anything. The CNN knows only about image pixel values and always performs the same lengthy computation on those pixel values, no matter what they represent.

If you build a dog breed classifier and you pass in an image of something other than a dog, say a human face, the CNN will perform the same numerical computation and make a guess as to dog breed, same as before. This feature of CNNs was used to determine the "spirit dog" of a human in the image.

## Language Modeling and Text Generation

For this project, I built a text generator that outputs a string of characters of any length in the style of whatever text it was trained on. I trained the text generator using the book _The Adventures of Sherlock Holmes_ by Sir Arthur Conan Doyle.

### How to frame the text generation problem

To use a neural network for text generation, build a network that takes as input a sequence of characters and outputs a prediction for the next character in the sequence. This prediction takes the form of a probability distribution over the possible characters.

To start generating characters using the generator, select an initial of sequence of characters and compute the output distribution for the next character. Take the most likely character and record it as the first output character. Then append it to the input sequence and feed the new sequence into the neural network. Get the output probability distribution and select the most likely character. Repeat the process until you've got the desired number of characters. You can generate a sentence, a paragraph or an entire novel!

For a more "creative" text generator, instead of taking the most likely character each time, select a character by sampling from the probability distribution output by the neural network. For example, 'e' might have probability 0.75, but 'f' and 'h' could be non-zero with probabilities 0.16 and 0.09. Instead of always selecting 'e' in this case, draw a random number uniformly distributed between 0 and 1 and use it to decide on the next character.

NumPy `random.choice` is an easy way to sample from a probability distribution. For our simple 3 character example:

```python
np.random.choice(3, p=[ 0.75, 0.16, 0.09 ])
```

### Generate training examples for text generator neural network using a sliding window

Consider the text, "It was a quarter past six when we left Baker Street". Let's choose a sliding window size of 10 characters. To build the first training example, take the first 10 characters, "It was a q", as our input and create a label "u", the next character according to the training text. Side the window forward one character to get the next training example: "t was a qu" with label "a". Slide one more character to get the third example: " was a qua" with label "r".

### Tuning a neural network's hyperparameters is necessary for good performance

It's essential to understand and tune the neural network's hyperparameters.

First, a clarification. The deep learning community lumps together two disparate things as hyperparameters: architectural parameters and training parameters. A hyperparameter of a neural network model is essentially anything that is configurable that isn't a weight involved in the computation of a neuron's output. For example, the size of the hidden state vector in an RNN and the training batch size are both hyperparameters, even though from an engineering point of view, these are two very different things.

Some key hyperparameters for the text generator are size of the RNN hidden state vector, the choice of optimizer and the learning rate for that optimizer.

### Training a text generator neural network takes a long time and requires a lot of data

In this project I trained the network using 100,000 characters for 30 epochs, with only modestly convincing results. A [famous blog post](http://karpathy.github.io/2015/05/21/rnn-effectiveness/) reveals that generating convincing Shakespeare-esque text required a training set with millions of characters and took hours to train the model.

### Resources

The Jupyter Notebook for the Udacity AIND text generator project is available on GitHub:<br/>
[https://github.com/udacity/aind2-rnn](https://github.com/udacity/aind2-rnn
)

Project Gutenberg: _The Adventures of Sherlock Holmes_<br/>
[http://www.gutenberg.org/ebooks/1661](http://www.gutenberg.org/ebooks/1661)<br/>
The copyright on Doyle's famous work is long expired and the text of the book is freely available from Project Gutenberg.
