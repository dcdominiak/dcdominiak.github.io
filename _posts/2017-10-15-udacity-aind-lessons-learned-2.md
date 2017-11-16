---
title: "Lessons Learned During Udacity AIND, Part 2: Planning, Natural Language Recognition"
excerpt: "Part 2 of lessons learned during Udacity Artificial Intelligence Nanodegree covers the remainder of Term 1."
layout: post
---

In this section, I'll cover two well-established domains, planning and natural language recognition.

## Planning

In this project, I designed a system to plan the movement of air cargo from airport to airport. The system combines several well-established components that date as far back as the 1960s.

### What computer scientists mean by planning

Everyone has an intuitive understanding of the act of planning and the resulting plan. A plan is a list of actions to perform to achieve a goal. Maybe the actions are performed one after the other, maybe in any order. Planning is the act of creating the list of actions.

Computer scientists define a graph where nodes represent states of the world (well, typically a very small subset of it) and edges represent actions to perform when in that state. One node in the graph (usually the root) is the initial state and one is the goal state. Planning is running an algorithm that searches the graph to find the node that represents the goal state.

In the air cargo example, a state describes the location of all the planes and all the packages. The initial state is the location of the planes and packages after all the day's deliveries have been collected from customers, and the goal state has each package at its destination city.

### How to frame the general planning problem

Planning is searching the graph to find a path from the initial state (root node) to the goal state. The goal state is not a node, but a set of criteria for evaluating nodes. Any node that meets the criteria can be selected as the final node in the path. The plan is the path found by the search algorithm. Clearly the search strategy matters a great deal.

### A* search algorithm is best practice for planning

The A* search algorithm is the go-to algorithm for search in planning problems. It finds the lowest-cost path from among all possible paths, and does it in an efficient way. At each node, it searches the possible paths from that node to the goal, starting from the most promising one, the one that is estimated to have the least cost. All possible paths are searched in order from least estimated cost to greatest. The search proceeds recursively down each possible path, building up a tree of paths.

The cost estimate used by the A* algorithm is generated using a heuristic. The heuristic is a measure of the path cost that's relatively inexpensive to calculate and that doesn't require the search to continue in order to generate. An example heuristic is using the straight line distance between two cities in a route planning system. While no roads actually traverse that straight line exactly, it's a good estimate of the distance to that city. (An more importantly, it's an _admissible_ estimate: it never overestimates the true cost.)

### How to frame the logistics planning problem

Unlike with vehicular route planning, the state space for a logistics planning problem is large, complicated and much more abstract.

## American Sign Language Recognizer

In this project, I designed an American Sign Language (ASL) recognizer using hidden Markov models (HMMs). One of the leading researchers in applying machine learning to ASL recognition created the project for Udactiy and taught the lessons.

### Hidden Markov models are a good choice for building language recognizers

ASL is a visual language of hand gestures and facial expressions. Each hand gesture follows a specific pattern, such as moving your right hand to the right for a distance, then moving it in the opposite direction for a similar distance. Different speakers will execute this gesture is subtlely different ways, maybe with a slight upward or downward angle, maybe with a larger or smaller distance moved, but they all follow the basic pattern. The challenge in designing a recognizer is to detect the underlying pattern without getting thrown off by the variations from speaker to speaker.

A good way to handle variations on a basic pattern is to build a probabilistic model for each word, and a hidden Markov model (HMM) is such a model. 

An HMM is a mathematical model of a sequence with a specific pattern, but with a degree of randomness thrown in. It defines a set of states, transitions between states, and an output value in each state. Its defining characteristic is that the transitions between states happen randomly and the output values in each state are random numbers. The HMM is like a probabilistic state machine. The states are not observed directly (they're hidden), but are an inherent property of the underlying process generating the sequence, in this case the mind of the ASL speaker.

How do hidden Markov models help in spoken language recognition? The key insight in using HMMs in a language recognizer is that it's possible to build a model for each spoken word as a time-based sequence of values. For ASL, those values are the positions of the hands of the speaker. HMMs allow us to introduce the right amount of randomness in the model to account for the variations observed across speakers without losing the underlying pattern.

An HMM is a sequence generator? How does that help? A recognizer doesn't need to generate sequences, it needs to analyze them.

There's another way to use an HMM. An HMM can be used to evaluate the similarity of a given sequence to the type of sequences produced by the model. The similarity is measured in terms of the probability that the given sequence was produced by the model. It's these similarity measures that the ASL recognizer computes and uses to recognize words.

### How to frame the sign language recognition problem

The HMM-based sign language recognizer is a system that takes a time-based sequence of feature vectors as input and outputs a guess as to which word in the recognizer's vocubulary that sequence most resembles. (I'll discuss those feature vectors in the next lesson learned.)

The sign language recognizer has one trained model (one HMM) for each word in the vocabulary. The input to a model is a sequence of feature vectors and the output is the (log) probability that that model generated that input sequence. Each model is trained with examples of the sequence so that it learns the statistics of how it unfolds over time. Ideally a model will output a value of 1.0 for a sequence representing the word it was trained on and 0.0 for all other words. In practice, things are not so precise.

When the recognizer receives an input word (sequence of vectors), it runs each vocabulary word model to get a probability for that word. The result is a probability distribution over the vocabulary words. It then selects the word with the highest probability and declares that as the recognized word.

For this project, several simplifications were made to keep the scope manageable:

  * Discrete word recognition instead of continuous (sentence) recognition,
  * A limited vocabulary (only 112 words vs. thousands), and
  * No statistical language modeling to assist in word recognition.

### The processing pipeline for visual language recognition is complex

ASL is a visual language of hand gestures and facial expressions. So first you need video of the speaker, or at least a sequence of images taken at some constant rate. The video frames or still images capture the gestures and expressions as they unfold over time. In this project, the facial expressions were ignored and only hand gestures were used.

Next, you need a way to detect and measure the positions of the hands, both left and right, in each of the images. The sign for a word is determined by the movement of the hands, both in relation to the center of the person signing and to each other. The dataset for the project uses pixel distances for hand position coordinates. (Hand position tracking is a challenging problem in its own right and was not covered in the AIND material. The hand position coordinates were provided for us.)

Finally, the raw hand position data is converted into a time-based sequence of feature vectors. The conversion of raw coordinates to features can be done in a number of ways. There is no right answer, so you need to develop some intuition about "good" features and try different ones to see which give the best results.

### Good features for an ASL recognizer have some defining characteristics

In my estimation, there are two defining characteristics of features for sign language recognition:

  1. The feature set must account for differences in speaker height and arm length.
  2. The feature set must account for differences in positioning of the speaker within the video frame.

These characteristics are necessary for the system to recognize similar gestures as similar. I selected normalized-by-speaker polar coordinates for the final feature set, as that meets both these criteria. It references all hand positions to a central location on the speaker (the nose), and normalizes by average distance of the hand from that location. The average here is taken across all training examples associated with that speaker. The former accounts for the speaker's position within the video frame, while the latter accounts for the speaker's height and arm length.

This analysis doesn't generalize to other problems such as audio speech recognition, but it was an interesting exercise in machine learning feature engineering.

