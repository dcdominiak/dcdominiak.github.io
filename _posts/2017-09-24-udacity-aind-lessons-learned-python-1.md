---
title: "Python Lessons Learned During Udacity AIND"
excerpt: "Python lessons learned during Udacity Artificial Intelligence Nanodegree."
layout: post
---

Python is an interesting language. It's one of the most popular languages for machine learning, and for that reason it's an interesting language. But it's also interesting because it seems to command -- and even demand -- a very particular way of writing code. It demands _Pythonic_ code.

Before I started the Udacity Artificial Intelligence Nanodegree, I had relatively little experience with Python. Throughout the course, I analyzed or wrote a fair amount of Python code. This post describes some language features, functions and code snippets that I found to be  useful in learning Python in general, and also writing more _Pythonic_ code.

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

Python supports a conditional expression which allows you to perform a conditional assignment with a compact syntax. For example, instead of this:

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

### Reading elements of an array with enumerate()

The `enumerate()` built-in function allows you to loop through the elements of an array and get both the index and the element value.

```python
for i, val in enumerate(x):
	x[i] = alpha * val
```

### List comprehension instead of a for loop to create a list (array)

One of the iconic features of Python -- and one of the most _Pythonic_ -- is the list comprehension. A list comprehension is typically used to create a list instead of a for loop, as it's more compact.

For example, it can be used to compute the error vector when training a neural network:

```python
output_error = [ y[i] - y_hat[i] for i in range(len(y)) ]
```

Or it can be used to encode a character string using a tokenizer:

```python
encoded_input = [ vocab_to_int[i] for i in text ]
```

### Creating a dictionary with a list comprehension

A list comprehension can be used to create a dictionary.

```python
int_to_vocab = { index: word for index, word in enumerate(list(vocab_words)) }
```

### List comprehension instead of nested for loops

A list comprehension can be used in place of nested for loops.

```python
# Generate a list of all possible triplets
bases = [ 'U', 'C', 'A', 'G' ]
codons = [ a + b + c for a in bases for b in bases for c in bases ]
```

### Variables are scoped to functions

Variables are scoped to functions, not to blocks within functions. A function introduced in a `with` or `for` block remains in scope for the remainder of the function. This is different than many languages such as C, Java or Swift.

```python
def my_function():
	for item in items:
		if item == target_value:
			found_item = True
	print(found_item)
```

### Return multiple values from a function

It's trivial to have a function return multiple values: use a tuple.

```python
x, y, z = my_function()
```

If you don't need all the return values, ignore one or more with `_`:

```python
x, y, _ = my_function()
```

### Walk through list elements in reverse order

Python's array indexing syntax can be used to access the elements of a list in reverse order:

```python
a = [1, 2, 3]
for item in a[::-1]:
	print(item)
```

Technically, this creates a view into the original list where the elements are reversed. Changing values in the original list will also change the view. If you want to permanently reverse the order, use `a.reverse()`.

Don't try to parse the syntax `a[::-1]` in the form start:stop:step, though. What does a step of -1 mean? This is one of those idiomatic expressions in the language.

### Don't forget the built-in functions

The built-in function `all()` returns `True` if all elements of a list are true; `any()` returns `True` if any elements are true.

`min()` and `max()` can be used in cases where NumPy isn't being imported, otherwise the NumPy equivalents are available.

### Build or split a file path

The `os.path` module can be used to either build a file path from a directory path and a filename, or extract the filename from a path.

```python
full_path = 'logs/log-1.log'
path, filename = os.path.split(full_path)
full_path = os.path.join(path, filename)
```

### Python supports object-oriented programming

Python supports a mechanism for defining classes of objects, creating instances of those classes and calling methods on those instances, otherwise known as object-oriented programming.

```python
class MyClass(object):
	def __init__(self):
		self._instance_var_1 = []
		self._instance_var_2 = 0

	def compute_something_useful(self, var2):
		self._instance_var_2 = var2
		return self._instance_var_2 * self._instance_var_2
```
