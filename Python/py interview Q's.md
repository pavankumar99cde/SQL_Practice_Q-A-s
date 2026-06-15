# Python Coding Interview Questions - Multiple Solutions

---

## 1. Reverse a String

### Solution 1: Using Slicing

```python
def reverse_string(s):
    return s[::-1]

print(reverse_string("hello"))
```

### Solution 2: Using reversed()

```python
def reverse_string(s):
    return ''.join(reversed(s))

print(reverse_string("hello"))
```

### Solution 3: Using Loop

```python
def reverse_string(s):
    result = ""
    for ch in s:
        result = ch + result
    return result

print(reverse_string("hello"))
```

---

## 2. Check Whether a String is Palindrome

### Solution 1: Using Slicing

```python
def is_palindrome(s):
    return s == s[::-1]

print(is_palindrome("madam"))
```

### Solution 2: Using Two Pointers

```python
def is_palindrome(s):
    left = 0
    right = len(s) - 1

    while left < right:
        if s[left] != s[right]:
            return False

        left += 1
        right -= 1

    return True

print(is_palindrome("madam"))
```

### Solution 3: Using reversed()

```python
def is_palindrome(s):
    return s == ''.join(reversed(s))

print(is_palindrome("madam"))
```

---

## 3. Find Factorial of a Number

### Solution 1: Using Recursion

```python
def factorial(n):
    if n == 0:
        return 1

    return n * factorial(n - 1)

print(factorial(5))
```

### Solution 2: Using Iteration

```python
def factorial(n):
    result = 1

    for i in range(1, n + 1):
        result *= i

    return result

print(factorial(5))
```

### Solution 3: Using math.factorial()

```python
import math

print(math.factorial(5))
```

---

## 4. Fibonacci Series

### Solution 1: Using Loop

```python
def fibonacci(n):
    a, b = 0, 1

    for _ in range(n):
        print(a, end=" ")
        a, b = b, a + b
```

### Solution 2: Using Recursion

```python
def fibonacci(n):
    if n <= 1:
        return n

    return fibonacci(n - 1) + fibonacci(n - 2)
```

### Solution 3: Using Generator

```python
def fibonacci():
    a, b = 0, 1

    while True:
        yield a
        a, b = b, a + b
```

---

## 5. Find Largest and Second Largest Number

### Solution 1: Using Sorting

```python
def second_largest(nums):
    nums = sorted(set(nums))
    return nums[-2]
```

### Solution 2: Using Two Variables

```python
def second_largest(nums):
    first = second = float('-inf')

    for num in nums:
        if num > first:
            second = first
            first = num
        elif first > num > second:
            second = num

    return second
```

### Solution 3: Using heapq

```python
import heapq

def second_largest(nums):
    return heapq.nlargest(2, nums)[1]
```

---

## 6. Remove Duplicates from List

### Solution 1: Using set()

```python
def remove_duplicates(nums):
    return list(set(nums))
```

### Solution 2: Preserve Order

```python
def remove_duplicates(nums):
    seen = set()
    result = []

    for num in nums:
        if num not in seen:
            seen.add(num)
            result.append(num)

    return result
```

### Solution 3: Using dict.fromkeys()

```python
def remove_duplicates(nums):
    return list(dict.fromkeys(nums))
```

---
