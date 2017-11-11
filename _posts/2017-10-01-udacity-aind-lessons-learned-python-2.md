---
title: "Python Lessons Learned During Udacity AIND, Part 2: NumPy and Matplotlib"
excerpt: "Part 2 of Python lessons learned during Udacity Artificial Intelligence Nanodegree covers useful NumPy and Matplotlib code."
layout: post
---

NumPy is a numerical computing library for Python that's so ubiquitous it feels as if it's part of the language. When developing machine learning applications, it's often necessary to rely on NumPy for certain things even if you're using a high-level library like TensorFlow or Keras.

### NumPy functions can be applied to every element in a list or array

If you apply a NumPy function like `np.exp()` to a Python list, it applies the function to every element in the list, automatically. No looping is necessary.

```python
L = [ 0.3, 0.5, 0.2 ]
np.exp(L)
```

Same is true for a NumPy array, of course.

### Extract parts of NumPy arrays by _slicing_ them

NumPy supports slicing of n-dimensional arrays with the syntax X[start:stop:step].

```python
X[:10]  # the first 10 elements of X
X[10:]  # the rest of X starting with element 11 (index 10)
```

### Remove single-dimensional entries from the shape of an array

Use `np.squeeze(X, axis=0)` to remove single-dimensional entries from the shape of an array.

Suppose X = [ [1], [7], [6], [3] ], or an array of single-dimensional arrays. Applying `squeeze()` collapes the array-of-arrays into the array [ 1, 7, 6, 3 ].

`np.squeeze()` is the opposite of `np.expand_dims()`. See next.

### Insert a single-entry dimension into an array

If X = [ 1, 7, 6, 3 ], applying `np.expand_dims()` returns the array [ [1], [7], [6], [3] ].

`np.expand_dims()` is the opposite of `np.squeeze()` in the sense that it undoes that operation.

### Generate a uniformly distributed random number

The NumPy function `np.random.rand()` returns a random number in the range [0, 1).

### Generate a vector or matrix of random numbers

The NumPy function `np.random.rand()` can return a vector or matrix of random numbers, by supplying arguments.

```python
np.random.rand(3)     # row vector (1x3)
np.random.rand(3, 1)  # column vector (3x1)
np.random.rand(3, 3)  # matrix (3x3)
```

### Compare elements of two vectors

```python
predictions = [np.argmax(model.predict(np.expand_dims(feature, axis=0))) for feature in test_InceptionV3]
```
