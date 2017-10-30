---
title: Python Lessons Learned During AIND
excerpt: "Python lessons learned during Udacity Artificial Intelligence Nanodegree."
layout: post
---

Python is an interesting language. It's one of the most popular languages for machine learning, and for that reason it's an interesting language. But it's also interesting because it seems to command -- and even demand -- a very particular way of writing code. It demands _Pythonic_ code.

Before I started the Udacity Artificial Intelligence Nanodegree, I had relatively little experience with Python. Throughout the course, I analyzed or wrote a large amount of Python code. This post describes some functions and code snippets that I found to be  useful in learning Python in general, and also writing more _Pythonic_ code.

### Multiple assignments in a single statement

Let's start with an easy one. Python lets you assign values to two or more variables in a single statement.

I'm not talking about multiple statements on a single line, like this:

```python
x_prev = x; y_prev = y
```

No. Even better. With Python, you can write it like this:

```python
x_prev, y_prev = x, y
```

### Conditional assignment with compact syntax

Python supports a conditional expression which allows you to perform a conditional assignement with a compact syntax. For example, instead of this:

```python
if output == correct_output:
	is_correct_string = 'Yes' 
else:
	is_correct_string = 'No'
```

you can write the assignment with the more compact syntax:

```python
is_correct_string = 'Yes' if output == correct_output else 'No'
```

### Creating a list (array) with a list comprehension

One of the iconic features of Python -- and one of the most _Pythonic_ -- is the list comprehension.

### Creating a dictionary with a list comprehension

A list comprehension can be used to create a dictionary.

```python
x = { x: y for x in range(10) for y in range(10) }
```

### Reading elements of an array with enumerate()

The `enumerate()` built-in function allows you to loop through the elements of an array and get both the index and the element value.

```python
for i, val in enumerate(x):
	x[i] = alpha * val
```

### Variables are scoped to functions

Variables are scoped to functions, not to blocks within functions. A function introduced in a `with` or `for` block remains in scope for the remainder of the function. This is different than many languages such as Java or Swift.

```python
def my_function():
	for i in range(10):
		if i == 0:
			flag = True
	print(flag)
```