# Python fundamentals for an AI backend engineer

## Learning outcomes

By the end, you should be able to read ordinary Python, write small data-processing programs, explain Python's object model at a practical level, use a virtual environment, and debug common errors. You do not need to abandon Java: keep Java for the service skills you already own and learn Python where the AI/data ecosystem makes it useful.

## 1. What Python is and how a program runs

Python is a high-level, dynamically typed language. A variable name refers to an object; a name does not permanently have a declared Java-like type. The interpreter executes a module top to bottom, creating objects and binding names as it goes.

~~~python
samples = [2, 4, 6]
average = sum(samples) / len(samples)
print(average)
~~~

The expression on the right is evaluated first, then its result is bound to the name on the left. Python integers have arbitrary precision in ordinary use. Floating point values use finite binary precision, so values such as 0.1 may not have an exact binary representation.

### Java comparison

| Java | Python |
|---|---|
| Static type declared or inferred at compile time | Runtime types; optional annotations help humans and tools |
| Braces define blocks | Indentation defines blocks |
| Class-based application entry point is common | A module or script can be the entry point |
| Maven/Gradle commonly manage dependencies | pip plus a virtual environment is common |
| Checked exceptions are part of the language model | Exceptions are generally unchecked |

A type annotation documents a contract and enables static analysis; Python generally does not enforce it at runtime:

~~~python
def normalize(text: str) -> str:
    return text.strip().lower()
~~~

## 2. Values, names, identity and mutability

A name is a reference to an object. Assignment binds another name; it does not automatically copy the object.

~~~python
left = [1, 2]
right = left
right.append(3)
assert left == [1, 2, 3]
~~~

Lists and dictionaries are mutable. Integers, floats, strings and tuples are immutable (a tuple can contain a mutable object, so immutability is shallow). **==** checks value equality. **is** checks whether two references identify the same object; use it primarily for singleton checks such as **value is None**.

This distinction matters when functions receive mutable inputs, when caching data, and when sharing objects across tasks. If a function should not mutate caller-owned data, copy deliberately, for example **copy = items.copy()** for a shallow list copy.

### Core containers

- **list**: ordered, mutable sequence; use for collected examples or batches.
- **tuple**: ordered, immutable sequence; use for fixed records or multiple return values.
- **dict**: key-to-value mapping; keys must be hashable.
- **set**: unique hashable values; useful for membership and deduplication.
- **str**: immutable Unicode text; indexes count Unicode code points, not user-perceived grapheme clusters.
- **bytes**: immutable raw byte sequence; keep distinct from decoded text.

Common mistakes: relying on dictionary ordering as a business contract without understanding your Python version, mutating a list while iterating over it, and assuming a shallow copy duplicates nested objects.

## 3. Expressions, control flow and iteration

Python uses **if, elif, else, for, while, break,** and **continue**. Truthiness is convenient but can hide intent: empty containers, zero, **None**, and **False** are falsey. For an optional value, write **if result is None** instead of **if not result** if an empty but valid result has different meaning.

~~~python
for index, item in enumerate(["a", "b"]):
    print(index, item)

for name, score in {"Mina": 8, "Raj": 9}.items():
    print(name, score)
~~~

An iterable produces values one at a time. A list comprehension constructs a list; a generator expression can produce values lazily:

~~~python
squares = [n * n for n in range(10) if n % 2 == 0]
square_stream = (n * n for n in range(10) if n % 2 == 0)
~~~

Use comprehensions for simple transformations. Use a named loop when the logic has multiple conditions, side effects, error handling, or deserves a helpful name. Lazy iteration can reduce peak memory for large files, but the source and downstream operations still determine whether the whole dataset is eventually materialized.

## 4. Functions and scope

A function is a named unit with inputs, outputs and a focused responsibility. Python passes object references by assignment: a function may mutate a mutable argument, but rebinding its parameter does not rebind the caller's name.

~~~python
def clamp(value: float, low: float = 0.0, high: float = 1.0) -> float:
    if low > high:
        raise ValueError("low must not exceed high")
    return max(low, min(value, high))
~~~

Avoid a mutable default such as **def add(item, items=[])**: that list is created once and reused across calls. Prefer **None**, then create a fresh list inside the function. Keyword-only arguments (after the asterisk separator) can make safety-critical options explicit. Return values are preferable to hidden global state because they make behavior easier to test.

A function should validate inputs at the boundary where the contract is known. Do not add redundant validation to every internal helper; place invariants where they are easiest to explain and enforce.

## 5. Errors, files and data formats

Exceptions represent exceptional failure of an operation. Catch a specific exception when you can recover or add context. Avoid **except Exception: pass**, which hides programming errors and corrupt input.

~~~python
from pathlib import Path
import json

path = Path("settings.json")
try:
    settings = json.loads(path.read_text(encoding="utf-8"))
except FileNotFoundError:
    settings = {}
except json.JSONDecodeError as error:
    raise ValueError(f"Invalid JSON in {path}") from error
~~~

Use **with** for resources such as files: it closes the handle even if an exception occurs. Specify encoding for text. JSON supports objects, arrays, strings, numbers, booleans and null; it does not preserve arbitrary Python objects or guarantee your application's schema. Validate decoded data before trusting it.

For CSV, specify delimiter/encoding and handle missing or malformed fields explicitly. Do not load a huge file into memory just because a tutorial uses **read_text**; stream rows when scale requires it.

## 6. Modules, packages and environments

A module is a Python file. A package groups modules. Keep imports explicit and avoid naming your files **json.py, typing.py** or **statistics.py**, because a local file can shadow a standard library module.

A virtual environment isolates a project's installed packages from other Python projects. Typical Windows setup:

~~~powershell
py -m venv .venv
.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install numpy
python -m pip freeze
~~~

Use a dependency manifest and lock strategy appropriate to the repository; a freeze output records versions but is not automatically a thoughtful dependency policy. Never commit environment folders, secrets, customer data, or large model weights unless the project intentionally manages them.

## 7. Classes, dataclasses and protocols

Use a class when an object has a meaningful identity, lifecycle, or behavior that belongs together. Do not translate every Java POJO into a Python class by habit. A dataclass is convenient for data with named fields:

~~~python
from dataclasses import dataclass

@dataclass(frozen=True)
class Document:
    document_id: str
    text: str
    source: str
~~~

**frozen=True** prevents field reassignment but does not deep-freeze referenced mutable values. Type protocols and structural typing can express that code needs an object with certain operations, without requiring a specific inheritance tree.

## 8. A first data-cleaning program

~~~python
import csv
from pathlib import Path

def load_examples(path: Path) -> list[dict[str, str]]:
    rows: list[dict[str, str]] = []
    with path.open(newline="", encoding="utf-8") as handle:
        for row_number, row in enumerate(csv.DictReader(handle), start=2):
            text = (row.get("text") or "").strip()
            label = (row.get("label") or "").strip()
            if not text or not label:
                raise ValueError(f"Missing text or label at row {row_number}")
            rows.append({"text": text, "label": label})
    return rows
~~~

This is not yet a production ingestion pipeline: it keeps all rows in memory, has no schema version, no privacy controls, no quality report and no idempotent storage. It demonstrates a narrow contract and useful error location.

## 9. Testing and debugging

A unit test checks one behavior with controlled inputs. Test normal cases, boundaries, invalid inputs, and invariants. For **clamp**, include in-range, below-range, above-range, and invalid bounds. Tests should assert visible behavior rather than private implementation details. Use a debugger or print/logging at the boundary where state changes; avoid dumping personal or secret data into logs.

Formatting, linting, type checking and tests answer different questions:
- formatter: consistent layout;
- linter: suspicious or disallowed patterns;
- type checker: whether annotations and inferred types agree;
- tests: whether chosen behaviors hold for cases you wrote.

## 10. Python concepts used constantly in AI

- **NumPy array**: an n-dimensional typed array optimized for numerical operations; unlike a Python list, arithmetic is vectorized and normally applies elementwise.
- **pandas DataFrame**: labeled tabular data abstraction; helpful for exploration, but not a substitute for database/query design.
- **iterator/generator**: incremental production of items; useful for streaming records and avoiding unnecessary copies.
- **context manager**: manages setup and cleanup around a block, commonly written with **with**.
- **exception chaining**: preserves the original cause when adding domain context using **raise ... from error**.
- **serialization**: conversion to a transport or storage representation. Serialization does not itself validate trust, privacy, or schema.

## Questions and answers

**Q: Do I need to become a Python language expert before learning AI?**  
A: No. Reach the point where you can read functions, collections, modules, exceptions and tests, then learn numerical libraries. Return to advanced language details when a real project needs them.

**Q: Why learn Python if my production backend is Java?**  
A: Python is a strong language for data exploration, model training and many AI libraries. An engineer can keep Spring Boot as the primary service and use a Python model worker behind a versioned API where that boundary is useful.

**Q: Does dynamic typing mean there are no types?**  
A: Every runtime value has a type. Dynamic typing means names are not normally restricted to one declared type by the language compiler; annotations and tools recover some early feedback.

**Q: When should I use a list versus a generator?**  
A: Use a list when you need repeated indexing or a small materialized collection. Use an iterator/generator when sequential processing and reduced intermediate memory matter. Neither makes an operation automatically fast.

## Practice

1. Write a function that accepts a list of numeric scores and returns the mean, rejecting an empty list with a useful error.
2. Stream a CSV one row at a time and count missing labels without printing row contents.
3. Write tests for a normalization function: whitespace, empty string, Unicode, and invalid input.
4. Explain, with a small experiment, the difference between **==** and **is**.
5. Create a small package with a data model module, parsing module and tests.

## Completion check

You can explain name binding versus copying, identify mutable objects, use a virtual environment, parse a text file safely, write a focused function, and test its observable behavior. Continue to the math chapters, then practice NumPy in a notebook or script.