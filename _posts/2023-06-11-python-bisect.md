---
title: 'Internalizing Python `bisect_right()` and `bisect_left()`'
date: 2023-06-11
permalink: /posts/2023/06/python-bisect/
tags:
  - Python
  - Algorithm
categories:
  - CS
---

Binary search is a popular topic in coding interviews. I was struggling with finding the right template for implementing binary search as it's annoying to get the equal sign or less than equal sign correctly in the right place. Eventually I decided to use a template similar to `bisect_right()` and `bisect_left()` in Python `bisect` library.

Here I'm going to share how I internalize `bisect_right()` and `bisect_left()` so that I no longer need to memorize it.

Of course, if your interviewer gives you a green light to use Python's built-in libraries, I'd recommend just using it so that you don't need to reinvent the wheel. And, it also showcases your familiarity with the Python language. Or, you can implement binary search yourself first and then tell your interviewer that there's actaully  Python built-in library that you can use.

# Python's bisect_right and bisect_left
`bisect_right` (also known as `bisect`) and `bisect_left` perform binary search operations on sorted lists, returning the index where an element needs to be inserted to maintain the sorted order.

1. `bisect_right`: Given a sorted list `a`, find the **right** most insertion position (i.e. index) for value `x` to maintain sorted order. If `x` already appears in the list, the insertion point is after (to the right of) any existing entries. In other words, the returned index `i` partitions the list into two halves such that, all elements less than or equal to `x` are in `a[:i]` and all elements greater than `x` are in `a[i:]`.

2. `bisect_left`: Given a sorted list `a`, find the **left** most insertion position (i.e. index) for value `x` to maintain sorted order. This function works similarly to `bisect_right`, but if `x` already appears in the list, the insertion point returned is the first (or leftmost) occurrence of `x`. Therefore, the returned index `i` partitions the list into two halves such that, all elements less than `x` are in `a[:i]` and all elements greater than or equal to `x` are in `a[i:]`.

Here are the links to their implementations: [`bisect_right`](https://github.com/python/cpython/blob/20a56d8becba1a5a958b167fdb43b1a1b9228095/Lib/bisect.py#L21-L54) and [`bisect_left`](https://github.com/python/cpython/blob/20a56d8becba1a5a958b167fdb43b1a1b9228095/Lib/bisect.py#L74-L107)

# Internalizing bisect_right and bisect_left
To truly understand these functions, I'll start by creating simplified versions of them and then gradually refine them to match the Python library implementations.

Let's start with this code for `bisect_right` that does the same thing but is easier to understand in my opinion:
```python
def bisect_right(a: List[int], target: int) -> int:
  # Assuming `a` is a sorted list
  left = 0
  # It's possible to insert a new element at (last index of a) + 1 if the new element is greater than all existing elements
  right = len(a) 

  while left < right:
    middle = (left + right) // 2
    if target > a[middle]: 
      # The insertion position for `target` must be greater than `middle`
      left = middle + 1 
    elif target < a[middle]:
      # The insertion position for `target` must be less than or equal to `middle`
      right = middle
    else:
      # If `target == a[middle]`, the right most insertion index must be at least `middle + 1`
      # since all elements to the left of insertion position should be <= `target`.
      left = middle + 1

  return left
```

If we refactor the above code a little bit, we have what's close to the `bisect_right` in Python's `bisect` library:
```python
def bisect_right(a: List[int], target: int) -> int:
  left = 0
  right = len(a) 

  while left < right:
    middle = (left + right) // 2
    if target < a[middle]:
      right = middle
    else:
      left = middle + 1
  return left
```


Similarly, we can understand `bisect_left` by looking at a simpler version:
```python
# Assuming `a` is a sorted list
def bisect_left(a: List[int], target: int) -> int:
  left, right = 0, len(a) 

  while left < right:
    middle = (left + right) // 2
    if target > a[middle]: 
      # The insertion position for `target` must be greater than `middle`
      left = middle + 1 
    elif target < a[middle]:
      # The insertion position for `target` must be less than or equal to `middle`
      right = middle
    else:
      # If `target == a[middle]`, there are two cases to consider:
      # 1. If there is only one element whose value equal to target in the list. Then the 
      # insertion position is equal to `middle`
      # 2. If there are more than one element whose value equal to target in the list, then 
      # we want to keep searching left till we find the first occurrence of `target`. So 
      # the insertion position must be less than or equal to `middle`.
      right = middle

  return left
```

And again, if we refactor the above code a little bit, we have what's close to `bisect_left` in Python `bisect` library:
```python
def bisect_left(a: List[int], target: int) -> int:
  left, right = 0, len(a) 

  while left < right:
    middle = (left + right) // 2
    if target > a[middle]:
      left = middle + 1 
    else:
      right = middle
  return left
```

So now I alway start with writing the simpler version first because it's easy to just write it out without memorizing it. And then I'll merge conditions that do the same thing and finally get to the implementation in Python's `bisect` library. 

# Finding the Leftmost or Rightmost Occurrence of an Element
In many cases, what we want is not the leftmost or rightmost insertion position for a target element in a sorted list. Instead, we want to find the leftmost or rightmost occurrence of that element. We can modify the return value of the above code to achieve this. E.g. if we want to find the rightmost element that is less than or equal to target, we will return `left - 1` instead in `bisect_right`. If `left - 1` < 0, this means all elements in the list are greater than target.


# A note on integer overflow
In other programming languages (e.g. C++) where there's a limit on the size of integer, you might want to calculate `middle` by `middle = left + (right - left) // 2` to avoid integer overflow.

[Python doesn't have a limit on integer value](https://stackoverflow.com/questions/13795758/what-is-sys-maxint-in-python-3) so it's ok to do `middle = (left + right) // 2`