---
title: Python Lessons Learned During AIND
excerpt: "Python lessons learned during Udacity Artificial Intelligence Nanodegree."
layout: post
---

Python is an interesting language. It's one of the most popular languages for machine learning, and for that reason it's an interesting language. But it's also interesting because it seems to command -- and even demand -- a very particular way of writing code. It demands _Pythonic_ code.

Before I started the Udacity Artificial Intelligence Nanodegree, I had relatively little experience with Python. Throughout the course, I analyzed or wrote a large amount of Python code. This post describes some functions and code snippets that I found to be  useful in learning Python in general, and also writing more _Pythonic_ code.

### Variables are scoped to functions

Variables are scoped to functions, not to blocks within functions. A function introduced in a `with` or `for` block remains in scope for the remainder of the function. This is different than many languages such as Java or Swift.

{% highlight python %}
def my_function():
	for i in range(10):
		if i == 0:
			flag = True
	print(flag)
{% endhighlight %}