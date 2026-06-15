
---

## 26. Parse a Log File and Extract Error Messages

### Solution 1: Using String Search

```python
def extract_errors(file_path):

    errors = []

    with open(file_path, "r") as file:

        for line in file:

            if "ERROR" in line:
                errors.append(line.strip())

    return errors

print(extract_errors("app.log"))
```

### Solution 2: Using Regex

```python
import re

def extract_errors(file_path):

    errors = []

    pattern = re.compile(r"ERROR.*")

    with open(file_path, "r") as file:

        for line in file:

            if pattern.search(line):
                errors.append(line.strip())

    return errors

print(extract_errors("app.log"))
```

### Solution 3: Using List Comprehension

```python
with open("app.log") as file:

    errors = [
        line.strip()
        for line in file
        if "ERROR" in line
    ]

print(errors)
```

---

## 27. Write a Function to Batch Records into Chunks of Size N

### Solution 1: Using Loop

```python
def chunks(data, size):

    result = []

    for i in range(
        0,
        len(data),
        size
    ):
        result.append(
            data[i:i + size]
        )

    return result

print(chunks(
    [1,2,3,4,5,6,7],
    3
))
```

### Solution 2: Generator

```python
def chunks(data, size):

    for i in range(
        0,
        len(data),
        size
    ):
        yield data[i:i + size]

for batch in chunks(
        [1,2,3,4,5,6,7],
        3):

    print(batch)
```

### Solution 3: itertools

```python
from itertools import islice

def chunks(data, size):

    iterator = iter(data)

    while True:

        batch = list(
            islice(
                iterator,
                size
            )
        )

        if not batch:
            break

        yield batch

print(
    list(
        chunks(
            [1,2,3,4,5,6,7],
            3
        )
    )
)
```

---

## 28. Find Top 10 Most Frequent Words

### Solution 1: Counter

```python
from collections import Counter

with open("data.txt") as file:

    words = file.read().split()

    result = (
        Counter(words)
        .most_common(10)
    )

print(result)
```

### Solution 2: Dictionary

```python
freq = {}

with open("data.txt") as file:

    for word in file.read().split():

        freq[word] = (
            freq.get(word, 0) + 1
        )

top_words = sorted(
    freq.items(),
    key=lambda x: x[1],
    reverse=True
)[:10]

print(top_words)
```

### Solution 3: Pandas

```python
import pandas as pd

with open("data.txt") as file:

    words = file.read().split()

df = pd.DataFrame(
    words,
    columns=["word"]
)

print(
    df["word"]
    .value_counts()
    .head(10)
)
```

---

## 29. Implement Producer Consumer Pattern

### Solution 1: Queue Module

```python
from queue import Queue
from threading import Thread

queue = Queue()

def producer():

    for i in range(5):
        queue.put(i)

def consumer():

    while not queue.empty():
        print(
            queue.get()
        )

Thread(
    target=producer
).start()

Thread(
    target=consumer
).start()
```

### Solution 2: multiprocessing.Queue

```python
from multiprocessing import (
    Process,
    Queue
)

queue = Queue()

def producer():

    queue.put("data")

def consumer():

    print(
        queue.get()
    )

Process(
    target=producer
).start()

Process(
    target=consumer
).start()
```

### Solution 3: asyncio Queue

```python
import asyncio

queue = asyncio.Queue()

async def producer():

    await queue.put(
        "record"
    )

async def consumer():

    item = await queue.get()

    print(item)
```

---

## 30. Remove Duplicates from Million Records Efficiently

### Solution 1: Set

```python
records = [
    1,2,3,3,4,4
]

unique_records = list(
    set(records)
)

print(unique_records)
```

### Solution 2: Preserve Order

```python
seen = set()

result = []

for record in records:

    if record not in seen:

        seen.add(record)

        result.append(record)

print(result)
```

### Solution 3: Generator Approach

```python
def unique_stream(data):

    seen = set()

    for item in data:

        if item not in seen:

            seen.add(item)

            yield item

for item in unique_stream(
        records):

    print(item)
```

---

## 31. Convert List of Dictionaries to DataFrame

### Solution 1: DataFrame Constructor

```python
import pandas as pd

data = [
    {
        "id": 1,
        "name": "John"
    },
    {
        "id": 2,
        "name": "Mike"
    }
]

df = pd.DataFrame(data)

print(df)
```

### Solution 2: from_records

```python
import pandas as pd

df = pd.DataFrame.from_records(
    data
)

print(df)
```

### Solution 3: json_normalize

```python
import pandas as pd

df = pd.json_normalize(
    data
)

print(df)
```

---

## 32. Find Average Salary Department Wise

### Solution 1: GroupBy Mean

```python
import pandas as pd

result = (
    df.groupby(
        "department"
    )["salary"]
    .mean()
)

print(result)
```

### Solution 2: agg()

```python
result = (
    df.groupby(
        "department"
    )
    .agg({
        "salary":"mean"
    })
)

print(result)
```

### Solution 3: Pivot Table

```python
result = pd.pivot_table(
    df,
    index="department",
    values="salary",
    aggfunc="mean"
)

print(result)
```

---

## 33. Group Records by Key and Calculate Aggregates

### Solution 1: GroupBy

```python
result = (
    df.groupby(
        "department"
    )["salary"]
    .sum()
)

print(result)
```

### Solution 2: Multiple Aggregations

```python
result = (
    df.groupby(
        "department"
    )
    .agg(
        {
            "salary":
            [
                "sum",
                "mean",
                "max"
            ]
        }
    )
)

print(result)
```

### Solution 3: Pivot Table

```python
result = pd.pivot_table(
    df,
    index="department",
    values="salary",
    aggfunc=[
        "sum",
        "mean",
        "max"
    ]
)

print(result)
```

---

## 34. Retry an API Call Three Times

### Solution 1: Loop

```python
import requests

for attempt in range(3):

    try:

        response = requests.get(
            "https://api.com"
        )

        print(
            response.json()
        )

        break

    except Exception:

        print(
            "Retrying..."
        )
```

### Solution 2: Function Wrapper

```python
import requests
import time

def retry_api():

    for _ in range(3):

        try:

            return requests.get(
                "https://api.com"
            )

        except Exception:

            time.sleep(2)

response = retry_api()

print(response)
```

### Solution 3: tenacity Library

```python
from tenacity import retry

@retry(stop=3)
def call_api():

    return requests.get(
        "https://api.com"
    )

print(call_api())
```

---

## 35. Read Multiple CSV Files and Combine

### Solution 1: glob + concat

```python
import glob
import pandas as pd

files = glob.glob(
    "*.csv"
)

df = pd.concat(
    [
        pd.read_csv(file)
        for file in files
    ]
)

print(df)
```

### Solution 2: os.listdir

```python
import os
import pandas as pd

dfs = []

for file in os.listdir():

    if file.endswith(".csv"):

        dfs.append(
            pd.read_csv(file)
        )

df = pd.concat(dfs)

print(df)
```

### Solution 3: pathlib

```python
from pathlib import Path
import pandas as pd

files = Path(
    "."
).glob("*.csv")

df = pd.concat(
    [
        pd.read_csv(file)
        for file in files
    ]
)

print(df)
```


---

## 36. Implement LRU (Least Recently Used) Cache

### Solution 1: Using OrderedDict

```python
from collections import OrderedDict

class LRUCache:

    def __init__(self, capacity):

        self.capacity = capacity

        self.cache = OrderedDict()

    def get(self, key):

        if key not in self.cache:
            return -1

        self.cache.move_to_end(key)

        return self.cache[key]

    def put(self, key, value):

        if key in self.cache:

            self.cache.move_to_end(key)

        self.cache[key] = value

        if len(self.cache) > self.capacity:

            self.cache.popitem(
                last=False
            )

cache = LRUCache(2)

cache.put(1, 100)
cache.put(2, 200)

print(cache.get(1))
```

### Solution 2: Using Dictionary + List

```python
class LRUCache:

    def __init__(self, capacity):

        self.capacity = capacity

        self.cache = {}

        self.order = []

    def get(self, key):

        if key not in self.cache:
            return -1

        self.order.remove(key)

        self.order.append(key)

        return self.cache[key]

    def put(self, key, value):

        if key in self.cache:

            self.order.remove(key)

        elif len(self.cache) >= self.capacity:

            oldest = self.order.pop(0)

            del self.cache[oldest]

        self.cache[key] = value

        self.order.append(key)
```

### Solution 3: Using functools.lru_cache

```python
from functools import lru_cache

@lru_cache(maxsize=3)
def square(num):

    print("Calculating...")

    return num * num

print(square(5))
print(square(5))
```

---

## 37. Find Longest Substring Without Repeating Characters

### Solution 1: Sliding Window (Optimal)

```python
def longest_substring(s):

    seen = {}

    left = 0

    max_length = 0

    for right in range(len(s)):

        if (
            s[right] in seen and
            seen[s[right]] >= left
        ):

            left = (
                seen[s[right]] + 1
            )

        seen[s[right]] = right

        max_length = max(
            max_length,
            right - left + 1
        )

    return max_length

print(
    longest_substring(
        "abcabcbb"
    )
)
```

### Solution 2: Brute Force

```python
def longest_substring(s):

    maximum = 0

    for i in range(len(s)):

        visited = set()

        for j in range(i, len(s)):

            if s[j] in visited:
                break

            visited.add(s[j])

            maximum = max(
                maximum,
                len(visited)
            )

    return maximum

print(
    longest_substring(
        "abcabcbb"
    )
)
```

### Solution 3: Using Set Window

```python
def longest_substring(s):

    left = 0

    right = 0

    seen = set()

    maximum = 0

    while right < len(s):

        while s[right] in seen:

            seen.remove(
                s[left]
            )

            left += 1

        seen.add(
            s[right]
        )

        maximum = max(
            maximum,
            right - left + 1
        )

        right += 1

    return maximum

print(
    longest_substring(
        "abcabcbb"
    )
)
```

---

## 38. Find All Pairs with Given Target Sum

### Solution 1: Hash Set (Optimal)

```python
def find_pairs(nums, target):

    seen = set()

    result = []

    for num in nums:

        diff = (
            target - num
        )

        if diff in seen:

            result.append(
                (diff, num)
            )

        seen.add(num)

    return result

print(
    find_pairs(
        [1,2,3,4,5],
        5
    )
)
```

### Solution 2: Nested Loops

```python
def find_pairs(nums, target):

    result = []

    for i in range(len(nums)):

        for j in range(
            i + 1,
            len(nums)
        ):

            if (
                nums[i] +
                nums[j]
            ) == target:

                result.append(
                    (
                        nums[i],
                        nums[j]
                    )
                )

    return result

print(
    find_pairs(
        [1,2,3,4,5],
        5
    )
)
```

### Solution 3: Two Pointers

```python
def find_pairs(nums, target):

    nums.sort()

    left = 0

    right = len(nums) - 1

    result = []

    while left < right:

        current_sum = (
            nums[left] +
            nums[right]
        )

        if current_sum == target:

            result.append(
                (
                    nums[left],
                    nums[right]
                )
            )

            left += 1
            right -= 1

        elif current_sum < target:

            left += 1

        else:

            right -= 1

    return result

print(
    find_pairs(
        [1,2,3,4,5],
        5
    )
)
```

---

## 39. Detect Cycle in Linked List

### Solution 1: Floyd Cycle Detection (Optimal)

```python
class Node:

    def __init__(self, value):

        self.value = value

        self.next = None

def has_cycle(head):

    slow = head

    fast = head

    while (
        fast and
        fast.next
    ):

        slow = slow.next

        fast = fast.next.next

        if slow == fast:

            return True

    return False
```

### Solution 2: Using Set

```python
def has_cycle(head):

    visited = set()

    current = head

    while current:

        if current in visited:

            return True

        visited.add(current)

        current = current.next

    return False
```

### Solution 3: Mark Nodes

```python
def has_cycle(head):

    current = head

    while current:

        if hasattr(
            current,
            "visited"
        ):

            return True

        current.visited = True

        current = current.next

    return False
```

---

## 40. Implement Binary Search

### Solution 1: Iterative (Most Asked)

```python
def binary_search(
        nums,
        target):

    left = 0

    right = len(nums) - 1

    while left <= right:

        mid = (
            left + right
        ) // 2

        if nums[mid] == target:

            return mid

        elif nums[mid] < target:

            left = mid + 1

        else:

            right = mid - 1

    return -1

print(
    binary_search(
        [1,2,3,4,5,6,7],
        5
    )
)
```

### Solution 2: Recursive

```python
def binary_search(
        nums,
        left,
        right,
        target):

    if left > right:

        return -1

    mid = (
        left + right
    ) // 2

    if nums[mid] == target:

        return mid

    if nums[mid] < target:

        return binary_search(
            nums,
            mid + 1,
            right,
            target
        )

    return binary_search(
        nums,
        left,
        mid - 1,
        target
    )

print(
    binary_search(
        [1,2,3,4,5,6,7],
        0,
        6,
        5
    )
)
```

### Solution 3: Using bisect

```python
import bisect

nums = [
    1,2,3,4,5,6,7
]

target = 5

index = bisect.bisect_left(
    nums,
    target
)

if (
    index < len(nums)
    and
    nums[index] == target
):

    print(index)

else:

    print(-1)
```
---
