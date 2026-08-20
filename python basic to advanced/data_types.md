# Python Data Types — Complete Notes (Basic to Advanced)

---

## 1. Introduction

Every value in Python is an **object**, and every object has a **type**. The type determines:
- What operations can be performed on the value
- Whether the value is mutable or immutable
- How much memory it occupies
- How it behaves in comparisons, hashing, and iteration

```python
x = 10
print(type(x))   # <class 'int'>
print(id(x))      # memory address (identity)
```

Python is **dynamically typed** (type is checked at runtime) but **strongly typed** (no implicit unsafe conversions, e.g. `"5" + 5` raises `TypeError`).

---

## 2. Categories of Data Types

| Category | Types |
|---|---|
| Numeric | `int`, `float`, `complex`, `bool` |
| Sequence | `str`, `list`, `tuple`, `range` |
| Mapping | `dict` |
| Set | `set`, `frozenset` |
| Binary | `bytes`, `bytearray`, `memoryview` |
| None | `NoneType` |

**Mutable vs Immutable** (fundamental concept, revisit constantly):

| Immutable | Mutable |
|---|---|
| int, float, complex, bool, str, tuple, frozenset, bytes | list, dict, set, bytearray |

---

## 3. Numeric Types

### 3.1 `int`
- Arbitrary precision (no overflow, unlike C/Java `int`).
```python
a = 10
big = 10**100          # works fine, Python handles big ints natively
print(a.bit_length())  # 4
```
- Underscore for readability: `1_000_000`
- Base conversions: `bin(10)`, `oct(10)`, `hex(10)`, `int("1010", 2)`

### 3.2 `float`
- IEEE 754 double precision (64-bit).
- Beware of precision errors:
```python
0.1 + 0.2  # 0.30000000000000004
```
- Fix with `decimal.Decimal` or `round()`:
```python
from decimal import Decimal
Decimal("0.1") + Decimal("0.2")   # Decimal('0.3')
```
- Special values: `float('inf')`, `float('-inf')`, `float('nan')`
- `math.isnan()`, `math.isinf()` for checks (`nan != nan` is `True`!)

### 3.3 `complex`
```python
z = 3 + 4j
z.real   # 3.0
z.imag   # 4.0
abs(z)   # 5.0 (magnitude)
```

### 3.4 `bool`
- Subclass of `int` (`True == 1`, `False == 0`).
```python
isinstance(True, int)  # True
True + True             # 2
```
- **Falsy values**: `0`, `0.0`, `""`, `[]`, `{}`, `()`, `set()`, `None`, `False`
- Everything else is truthy.

---

## 4. Sequence Types

### 4.1 `str` (immutable)
```python
s = "Hello"
s[0]          # 'H'
s[::-1]       # 'olleH' (reversed via slicing)
s.encode()    # bytes
```
- **f-strings** (preferred formatting):
```python
name = "Mohit"
f"{name=}"          # "name='Mohit'" (debug specifier, 3.8+)
f"{3.14159:.2f}"    # '3.14'
```
- Strings are immutable → `s += "x"` creates a **new** string object.
- Interning: small strings/identifiers are cached (`sys.intern`) for identity comparison speed.

### 4.2 `list` (mutable)
```python
lst = [1, 2, 3]
lst.append(4)
lst[1:2] = [20, 21]   # slice assignment
```
- Dynamic array internally (over-allocates capacity to amortize `append` cost → O(1) amortized).
- List comprehension:
```python
squares = [x**2 for x in range(10) if x % 2 == 0]
```
- Shallow vs deep copy:
```python
import copy
a = [[1,2],[3,4]]
b = a.copy()          # shallow — inner lists shared
c = copy.deepcopy(a)  # fully independent
```

### 4.3 `tuple` (immutable)
```python
t = (1, 2, 3)
single = (5,)          # comma required for single-element tuple
```
- Immutable but can contain mutable elements:
```python
t = ([1,2], 3)
t[0].append(3)   # allowed! t[0] itself can't be reassigned, but its contents can mutate
```
- Used for: fixed records, dict keys (if all elements hashable), function multi-returns.
- **Named tuples** (readable, lightweight records):
```python
from collections import namedtuple
Point = namedtuple("Point", ["x", "y"])
p = Point(1, 2)
p.x, p.y   # 1, 2
```

### 4.4 `range`
- Lazy, memory-efficient sequence of integers (doesn't store all values).
```python
r = range(1_000_000)   # constant memory
```

---

## 5. Mapping Type — `dict`

```python
d = {"a": 1, "b": 2}
d["c"] = 3
d.get("z", "default")
```
- **Since Python 3.7**: dicts preserve insertion order (guaranteed, not incidental).
- Keys must be **hashable** (immutable types, or objects implementing `__hash__` + `__eq__`).
- Dict comprehension:
```python
squares = {x: x**2 for x in range(5)}
```
- Merging (3.9+): `d1 | d2`, `d1 |= d2`
- `defaultdict`, `Counter`, `OrderedDict` — see Section 8.

**Under the hood**: dict is a hash table. Average O(1) lookup/insert/delete; O(n) worst case on hash collisions.

---

## 6. Set Types

### 6.1 `set` (mutable, unordered, unique elements)
```python
s = {1, 2, 3}
s.add(4)
s1 | s2   # union
s1 & s2   # intersection
s1 - s2   # difference
s1 ^ s2   # symmetric difference
```

### 6.2 `frozenset` (immutable version)
```python
fs = frozenset([1, 2, 3])   # hashable → can be a dict key or set element
```

Elements of a `set` must be hashable (so a `set` can't contain a `list`, but can contain a `tuple`).

---

## 7. Binary Types

| Type | Mutable | Use case |
|---|---|---|
| `bytes` | No | Immutable binary data, e.g. `b"hello"` |
| `bytearray` | Yes | Mutable binary buffer |
| `memoryview` | — | Zero-copy view over binary data (avoids duplication) |

```python
b = b"hello"
ba = bytearray(b)
ba[0] = 72          # mutate in place
mv = memoryview(ba) # view without copying
```

---

## 8. `None` and `NoneType`

```python
x = None
x is None      # ALWAYS use `is`, never `==`, for None checks
```
- Singleton — only one `None` object exists in a running interpreter.

---

## 9. Type Checking & Conversion

```python
type(x) == int        # works but discouraged
isinstance(x, int)     # preferred (respects subclassing)
isinstance(x, (int, float))   # multiple types
```

**Explicit conversion (casting):**
```python
int("42")        # 42
float("3.14")     # 3.14
str(42)           # "42"
list("abc")       # ['a','b','c']
tuple([1,2,3])
set([1,1,2,3])    # {1,2,3}
```

---

## 10. Identity vs Equality (`is` vs `==`)

```python
a = [1, 2, 3]
b = [1, 2, 3]
a == b   # True  (same value)
a is b   # False (different objects in memory)

x = 256
y = 256
x is y   # True — small ints (-5 to 256) are cached/interned by CPython
```

`is` compares **identity** (`id()`); `==` compares **value** (calls `__eq__`).

---

## 11. Mutability Deep Dive — Why It Matters

```python
def add_item(item, lst=[]):   # DANGEROUS: default mutable argument
    lst.append(item)
    return lst

add_item(1)   # [1]
add_item(2)   # [1, 2]  <- BUG: default list persists across calls!
```
**Fix:**
```python
def add_item(item, lst=None):
    if lst is None:
        lst = []
    lst.append(item)
    return lst
```

This is one of the most common real-world bugs stemming from misunderstanding mutability.

---

## 12. Advanced: Iterators & Generators

Generators aren't a "data type" in the strict sense (there's no `generator` literal), but they're a core Python object type built on the **iterator protocol** — essential once you move past basic types.

### 12.1 Iterator Protocol
Any object implementing `__iter__` and `__next__` is an iterator.
```python
class Counter:
    def __init__(self, limit):
        self.limit = limit
        self.n = 0
    def __iter__(self):
        return self
    def __next__(self):
        if self.n >= self.limit:
            raise StopIteration
        self.n += 1
        return self.n

for x in Counter(3):   # 1, 2, 3
    print(x)
```
- `list`, `dict`, `set`, `str` are **iterable** (have `__iter__`) but are NOT themselves iterators — calling `iter()` on them produces a separate iterator object.
```python
lst = [1, 2, 3]
it = iter(lst)
next(it)   # 1
next(it)   # 2
```

### 12.2 Generator Functions (`yield`)
A function with `yield` returns a **generator object** — lazy, computed on demand, one value at a time.
```python
def count_up_to(n):
    i = 1
    while i <= n:
        yield i
        i += 1

gen = count_up_to(5)
type(gen)     # <class 'generator'>
next(gen)     # 1
next(gen)     # 2
for x in gen: # continues from where it left off: 3, 4, 5
    print(x)
```
- Execution **pauses** at `yield` and resumes on the next `next()` call — state is preserved between calls.
- Once exhausted, raises `StopIteration` (loops handle this silently).
- Generators are **single-use** — once consumed, you can't restart without calling the function again.

### 12.3 Generator Expressions (lazy comprehension)
```python
squares = (x**2 for x in range(1_000_000))   # generator object, not a list
sum(squares)   # computed lazily, minimal memory
```
Compare:
```python
[x**2 for x in range(10**8)]   # list  -> builds ALL values in memory, huge RAM
(x**2 for x in range(10**8))   # gen   -> constant memory, one value at a time
```

### 12.4 Why Generators Matter (Memory & Performance)
- **Lazy evaluation**: values produced on demand, not stored all at once.
- Ideal for large/streamed data — e.g., reading a huge billing CSV line by line, processing large query result sets, infinite sequences.
```python
def read_large_file(path):
    with open(path) as f:
        for line in f:
            yield line.strip()

for line in read_large_file("huge_transactions.csv"):
    process(line)   # never loads the whole file into memory
```
- Composable via **pipelines**:
```python
lines = read_large_file("data.csv")
non_empty = (l for l in lines if l)
parsed = (l.split(",") for l in non_empty)
```

### 12.5 `yield from` — Delegating to Sub-generators
```python
def inner():
    yield 1
    yield 2

def outer():
    yield from inner()
    yield 3

list(outer())   # [1, 2, 3]
```

### 12.6 Sending Values Into a Generator (`.send()`)
Generators are technically **coroutine-like** — they can receive values, not just produce them.
```python
def echo():
    while True:
        received = yield
        print(f"Got: {received}")

g = echo()
next(g)          # prime the generator (advance to first yield)
g.send("hello")  # Got: hello
```

### 12.7 Generators vs Lists — Quick Comparison

| Aspect | List | Generator |
|---|---|---|
| Memory | Stores all elements | One element at a time |
| Speed (creation) | Slower (builds all upfront) | Faster (nothing computed yet) |
| Reusable | Yes (iterate multiple times) | No (single-use, exhausts) |
| Indexing (`lst[3]`) | Yes | No |
| `len()` support | Yes | No |
| Use case | Small/medium data, need random access | Large/streamed/infinite data, one-pass |

### 12.8 `itertools` — Generator Toolbox
```python
import itertools

itertools.count(1)            # infinite counter: 1, 2, 3, ...
itertools.cycle([1,2,3])      # repeats forever: 1,2,3,1,2,3,...
itertools.chain([1,2],[3,4])  # 1,2,3,4 (lazy concatenation)
itertools.islice(gen, 5)      # take first 5 from any iterable/generator
```

---

## 13. Advanced: `collections` Module (Specialized Data Types)

```python
from collections import defaultdict, Counter, OrderedDict, deque, ChainMap

# defaultdict — auto-initializes missing keys
dd = defaultdict(list)
dd["x"].append(1)   # no KeyError

# Counter — frequency counting
Counter("mississippi")   # Counter({'i':4, 's':4, 'p':2, 'm':1})

# deque — O(1) append/pop from both ends (great for queues, sliding windows)
dq = deque([1,2,3])
dq.appendleft(0)
dq.pop()

# ChainMap — merges multiple dicts into a single view without copying
cm = ChainMap({"a":1}, {"b":2})
```

---

## 14. Advanced: `typing` Module & Type Hints

```python
from typing import List, Dict, Tuple, Optional, Union, Callable

def process(items: List[int], mapping: Dict[str, int]) -> Optional[float]:
    ...

# Python 3.10+ simplified syntax
def process(items: list[int], mapping: dict[str, int]) -> float | None:
    ...
```
- Type hints are **not enforced at runtime** — they're for tooling (mypy, IDEs, readability).
- `TypedDict`, `Protocol`, `Generic` for advanced static typing.

---

## 15. Advanced: `dataclasses` (structured custom types)

```python
from dataclasses import dataclass, field

@dataclass
class Patient:
    name: str
    age: int
    diagnoses: list[str] = field(default_factory=list)

p = Patient("Amit", 40)
```
- Auto-generates `__init__`, `__repr__`, `__eq__`.
- `@dataclass(frozen=True)` → makes instances immutable (like a custom `tuple`/`namedtuple` hybrid).

---

## 16. Advanced: Custom Types & Data Model (Dunder Methods)

Any class can define custom behavior for built-in operations:

```python
class Money:
    def __init__(self, amount):
        self.amount = amount
    def __add__(self, other):
        return Money(self.amount + other.amount)
    def __eq__(self, other):
        return self.amount == other.amount
    def __hash__(self):
        return hash(self.amount)
    def __repr__(self):
        return f"Money({self.amount})"

m1 = Money(100) + Money(50)   # Money(150)
```

Relevant to your HMS billing work: you could model `Money`/`Currency` as an immutable custom type with `__add__`, `__sub__`, and rounding rules baked in, to avoid float precision bugs in financial calculations (use `Decimal` internally).

---

## 17. Memory & Performance Notes

- **Interning**: small ints (-5 to 256) and some strings are cached — `is` comparisons can be misleading for larger values.
- **`sys.getsizeof()`** — check memory footprint of an object.
- Lists over-allocate memory (growth factor) to make `append` O(1) amortized; tuples don't, since they're fixed-size, so they're more memory-efficient for static data.
- Use `__slots__` in classes to avoid per-instance `__dict__` overhead when you have many instances (e.g., thousands of `Patient` objects):
```python
class Patient:
    __slots__ = ("name", "age")
```

---

## 18. Quick Reference Cheatsheet

| Operation | int/float | str | list | tuple | dict | set |
|---|---|---|---|---|---|---|
| Mutable | ❌ | ❌ | ✅ | ❌ | ✅ | ✅ |
| Ordered | — | ✅ | ✅ | ✅ | ✅ (3.7+) | ❌ |
| Indexable | ❌ | ✅ | ✅ | ✅ | via key | ❌ |
| Hashable | ✅ | ✅ | ❌ | ✅* | ❌ | ❌ |
| Duplicates allowed | — | ✅ | ✅ | ✅ | keys: ❌ | ❌ |

*tuple is hashable only if all its elements are hashable.

---

## 19. Common Interview / Practical Gotchas

1. `a = b = []` → both names point to the **same** list object.
2. `+=` on a list mutates in place; `+` creates a new list.
3. `NaN != NaN` — never use `==` to check for NaN, use `math.isnan()`.
4. Dict/set lookups are O(1) average but require hashable, immutable-ish keys.
5. String concatenation in a loop (`s += x`) is O(n²) — use `"".join(list_of_parts)` instead.
6. Comparing floats for equality directly is unsafe — use `math.isclose(a, b)`.
7. A generator is exhausted after one full iteration — `sum(gen)` then `list(gen)` gives `[]` the second time. Convert to `list()` first if you need to reuse it.