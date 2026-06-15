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
---

## 7. Count Frequency of Each Character in a String

### Solution 1: Using Dictionary

```python
def char_frequency(s):
    freq = {}

    for ch in s:
        freq[ch] = freq.get(ch, 0) + 1

    return freq

print(char_frequency("hello"))
```

### Solution 2: Using collections.Counter

```python
from collections import Counter

def char_frequency(s):
    return Counter(s)

print(char_frequency("hello"))
```

### Solution 3: Using Dictionary Comprehension

```python
def char_frequency(s):
    return {ch: s.count(ch) for ch in set(s)}

print(char_frequency("hello"))
```

---

## 8. Find the First Non-Repeating Character

### Solution 1: Using Dictionary

```python
def first_non_repeating(s):
    freq = {}

    for ch in s:
        freq[ch] = freq.get(ch, 0) + 1

    for ch in s:
        if freq[ch] == 1:
            return ch

    return None

print(first_non_repeating("swiss"))
```

### Solution 2: Using Counter

```python
from collections import Counter

def first_non_repeating(s):
    freq = Counter(s)

    for ch in s:
        if freq[ch] == 1:
            return ch

    return None

print(first_non_repeating("swiss"))
```

### Solution 3: Using count()

```python
def first_non_repeating(s):
    for ch in s:
        if s.count(ch) == 1:
            return ch

    return None

print(first_non_repeating("swiss"))
```

---

## 9. Check Whether Two Strings are Anagrams

### Solution 1: Using Sorting

```python
def is_anagram(s1, s2):
    return sorted(s1) == sorted(s2)

print(is_anagram("listen", "silent"))
```

### Solution 2: Using Counter

```python
from collections import Counter

def is_anagram(s1, s2):
    return Counter(s1) == Counter(s2)

print(is_anagram("listen", "silent"))
```

### Solution 3: Using Character Frequency

```python
def is_anagram(s1, s2):
    if len(s1) != len(s2):
        return False

    freq = {}

    for ch in s1:
        freq[ch] = freq.get(ch, 0) + 1

    for ch in s2:
        freq[ch] = freq.get(ch, 0) - 1

    return all(value == 0 for value in freq.values())

print(is_anagram("listen", "silent"))
```

---

## 10. Find Intersection of Two Lists

### Solution 1: Using Sets

```python
def intersection(list1, list2):
    return list(set(list1) & set(list2))

print(intersection([1,2,3], [2,3,4]))
```

### Solution 2: Using Loop

```python
def intersection(list1, list2):
    result = []

    for item in list1:
        if item in list2 and item not in result:
            result.append(item)

    return result

print(intersection([1,2,3], [2,3,4]))
```

### Solution 3: Using List Comprehension

```python
def intersection(list1, list2):
    return [x for x in list1 if x in list2]

print(intersection([1,2,3], [2,3,4]))
```

---

## 11. Sort a List Without Using sort()

### Solution 1: Bubble Sort

```python
def bubble_sort(nums):
    n = len(nums)

    for i in range(n):
        for j in range(0, n - i - 1):
            if nums[j] > nums[j + 1]:
                nums[j], nums[j + 1] = nums[j + 1], nums[j]

    return nums

print(bubble_sort([5,3,8,1]))
```

### Solution 2: Selection Sort

```python
def selection_sort(nums):
    n = len(nums)

    for i in range(n):
        min_index = i

        for j in range(i + 1, n):
            if nums[j] < nums[min_index]:
                min_index = j

        nums[i], nums[min_index] = nums[min_index], nums[i]

    return nums

print(selection_sort([5,3,8,1]))
```

### Solution 3: Using sorted()

```python
nums = [5,3,8,1]

print(sorted(nums))
```

---

## 12. Find All Duplicate Elements in a List

### Solution 1: Using Set

```python
def find_duplicates(nums):
    seen = set()
    duplicates = set()

    for num in nums:
        if num in seen:
            duplicates.add(num)
        else:
            seen.add(num)

    return list(duplicates)

print(find_duplicates([1,2,2,3,4,4]))
```

### Solution 2: Using Counter

```python
from collections import Counter

def find_duplicates(nums):
    freq = Counter(nums)

    return [key for key, value in freq.items() if value > 1]

print(find_duplicates([1,2,2,3,4,4]))
```

### Solution 3: Using List Comprehension

```python
def find_duplicates(nums):
    return list(set([x for x in nums if nums.count(x) > 1]))

print(find_duplicates([1,2,2,3,4,4]))
```

---

## 13. Move All Zeros to the End

### Solution 1: Two Lists

```python
def move_zeros(nums):
    non_zeros = [x for x in nums if x != 0]
    zeros = [x for x in nums if x == 0]

    return non_zeros + zeros

print(move_zeros([0,1,0,3,12]))
```

### Solution 2: Two Pointers

```python
def move_zeros(nums):
    index = 0

    for i in range(len(nums)):
        if nums[i] != 0:
            nums[index], nums[i] = nums[i], nums[index]
            index += 1

    return nums

print(move_zeros([0,1,0,3,12]))
```

### Solution 3: Using sort Key

```python
def move_zeros(nums):
    return sorted(nums, key=lambda x: x == 0)

print(move_zeros([0,1,0,3,12]))
```

---

## 14. Find Missing Number in Array

### Solution 1: Sum Formula

```python
def missing_number(nums, n):
    expected = n * (n + 1) // 2
    actual = sum(nums)

    return expected - actual

print(missing_number([1,2,4,5], 5))
```

### Solution 2: Using Set

```python
def missing_number(nums, n):
    numbers = set(nums)

    for i in range(1, n + 1):
        if i not in numbers:
            return i

print(missing_number([1,2,4,5], 5))
```

### Solution 3: Using XOR

```python
def missing_number(nums, n):
    xor = 0

    for i in range(1, n + 1):
        xor ^= i

    for num in nums:
        xor ^= num

    return xor

print(missing_number([1,2,4,5], 5))
```

---

## 15. Find Maximum Occurring Element

### Solution 1: Counter

```python
from collections import Counter

def max_occurring(nums):
    return Counter(nums).most_common(1)[0][0]

print(max_occurring([1,2,2,3,3,3]))
```

### Solution 2: Dictionary

```python
def max_occurring(nums):
    freq = {}

    for num in nums:
        freq[num] = freq.get(num, 0) + 1

    return max(freq, key=freq.get)

print(max_occurring([1,2,2,3,3,3]))
```

### Solution 3: Using max() and count()

```python
def max_occurring(nums):
    return max(set(nums), key=nums.count)

print(max_occurring([1,2,2,3,3,3]))
```

---
---

## 16. Implement a Stack Using a Python List

### Solution 1: Using List

```python
stack = []

stack.append(10)
stack.append(20)
stack.append(30)

print(stack.pop())
```

### Solution 2: Using Class

```python
class Stack:

    def __init__(self):
        self.items = []

    def push(self, item):
        self.items.append(item)

    def pop(self):
        return self.items.pop()

stack = Stack()

stack.push(10)
stack.push(20)

print(stack.pop())
```

### Solution 3: Using deque

```python
from collections import deque

stack = deque()

stack.append(10)
stack.append(20)

print(stack.pop())
```

---

## 17. Implement a Queue Using Python Collections

### Solution 1: Using deque

```python
from collections import deque

queue = deque()

queue.append(10)
queue.append(20)

print(queue.popleft())
```

### Solution 2: Using List

```python
queue = []

queue.append(10)
queue.append(20)

print(queue.pop(0))
```

### Solution 3: Using Queue Module

```python
from queue import Queue

queue = Queue()

queue.put(10)
queue.put(20)

print(queue.get())
```

---

## 18. Check Whether Parentheses are Balanced

### Solution 1: Using Stack

```python
def is_balanced(expr):

    stack = []

    for ch in expr:

        if ch == "(":
            stack.append(ch)

        elif ch == ")":

            if not stack:
                return False

            stack.pop()

    return len(stack) == 0

print(is_balanced("(())"))
```

### Solution 2: Using Counter

```python
def is_balanced(expr):

    count = 0

    for ch in expr:

        if ch == "(":
            count += 1

        elif ch == ")":
            count -= 1

        if count < 0:
            return False

    return count == 0

print(is_balanced("(())"))
```

### Solution 3: Multiple Brackets

```python
def is_balanced(expr):

    stack = []

    mapping = {
        ")": "(",
        "]": "[",
        "}": "{"
    }

    for ch in expr:

        if ch in "([{":
            stack.append(ch)

        elif ch in ")]}":

            if not stack:
                return False

            if stack.pop() != mapping[ch]:
                return False

    return len(stack) == 0

print(is_balanced("{[()]}"))
```

---

## 19. Find the Kth Largest Element

### Solution 1: Sorting

```python
def kth_largest(nums, k):

    nums.sort(reverse=True)

    return nums[k - 1]

print(kth_largest([3,2,1,5,6,4], 2))
```

### Solution 2: heapq

```python
import heapq

def kth_largest(nums, k):

    return heapq.nlargest(k, nums)[-1]

print(kth_largest([3,2,1,5,6,4], 2))
```

### Solution 3: Min Heap

```python
import heapq

def kth_largest(nums, k):

    heap = nums[:k]

    heapq.heapify(heap)

    for num in nums[k:]:

        if num > heap[0]:
            heapq.heapreplace(heap, num)

    return heap[0]

print(kth_largest([3,2,1,5,6,4], 2))
```

---

## 20. Merge Two Sorted Lists

### Solution 1: Sort After Merge

```python
def merge_lists(a, b):

    return sorted(a + b)

print(merge_lists([1,3,5], [2,4,6]))
```

### Solution 2: Two Pointers

```python
def merge_lists(a, b):

    i = 0
    j = 0

    result = []

    while i < len(a) and j < len(b):

        if a[i] < b[j]:
            result.append(a[i])
            i += 1
        else:
            result.append(b[j])
            j += 1

    result.extend(a[i:])
    result.extend(b[j:])

    return result

print(merge_lists([1,3,5], [2,4,6]))
```

### Solution 3: heapq.merge

```python
import heapq

result = list(heapq.merge(
    [1,3,5],
    [2,4,6]
))

print(result)
```

---

## 21. Read a Large CSV File Efficiently

### Solution 1: Pandas Chunks

```python
import pandas as pd

for chunk in pd.read_csv(
        "data.csv",
        chunksize=10000):

    print(chunk.shape)
```

### Solution 2: CSV Module

```python
import csv

with open("data.csv") as file:

    reader = csv.reader(file)

    for row in reader:
        print(row)
```

### Solution 3: Generator

```python
def read_file(path):

    with open(path) as file:

        for line in file:
            yield line

for row in read_file("data.csv"):
    print(row)
```

---

## 22. Count Records for Each Category

### Solution 1: Pandas value_counts

```python
import pandas as pd

df = pd.read_csv("data.csv")

print(
    df["category"].value_counts()
)
```

### Solution 2: Group By

```python
import pandas as pd

df = pd.read_csv("data.csv")

result = (
    df.groupby("category")
      .size()
)

print(result)
```

### Solution 3: Dictionary

```python
counts = {}

for category in categories:

    counts[category] = (
        counts.get(category, 0) + 1
    )

print(counts)
```

---

## 23. Find Duplicate Rows in Dataset

### Solution 1: duplicated()

```python
import pandas as pd

df = pd.read_csv("data.csv")

duplicates = df[
    df.duplicated()
]

print(duplicates)
```

### Solution 2: duplicated Keep False

```python
duplicates = df[
    df.duplicated(
        keep=False
    )
]

print(duplicates)
```

### Solution 3: Group By Count

```python
duplicates = (
    df.groupby(
        df.columns.tolist()
    )
    .size()
    .reset_index(name="count")
)

duplicates = duplicates[
    duplicates["count"] > 1
]

print(duplicates)
```

---

## 24. Compare Two Files

### Solution 1: Set Difference

```python
with open("file1.txt") as f1:
    file1 = set(f1.readlines())

with open("file2.txt") as f2:
    file2 = set(f2.readlines())

print(file1 - file2)
print(file2 - file1)
```

### Solution 2: difflib

```python
import difflib

with open("file1.txt") as f1:
    lines1 = f1.readlines()

with open("file2.txt") as f2:
    lines2 = f2.readlines()

diff = difflib.unified_diff(
    lines1,
    lines2
)

print("".join(diff))
```

### Solution 3: Pandas

```python
import pandas as pd

df1 = pd.read_csv("file1.csv")
df2 = pd.read_csv("file2.csv")

difference = pd.concat(
    [df1, df2]
).drop_duplicates(
    keep=False
)

print(difference)
```

---

## 25. Flatten Nested JSON

### Solution 1: pandas.json_normalize

```python
import pandas as pd

data = {
    "id": 1,
    "user": {
        "name": "John",
        "city": "NY"
    }
}

df = pd.json_normalize(data)

print(df)
```

### Solution 2: Recursive Function

```python
def flatten_json(
        data,
        parent=""):

    result = {}

    for key, value in data.items():

        new_key = (
            f"{parent}_{key}"
            if parent else key
        )

        if isinstance(value, dict):

            result.update(
                flatten_json(
                    value,
                    new_key
                )
            )

        else:
            result[new_key] = value

    return result

print(flatten_json(data))
```

### Solution 3: Custom Stack

```python
def flatten_json(data):

    stack = [(data, "")]

    result = {}

    while stack:

        current, prefix = stack.pop()

        for key, value in current.items():

            new_key = (
                f"{prefix}.{key}"
                if prefix else key
            )

            if isinstance(value, dict):
                stack.append(
                    (value, new_key)
                )
            else:
                result[new_key] = value

    return result
```

---
