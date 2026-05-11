# Code Complexity

## Overview

In this project we **avoid writing complex code that is difficult to understand and maintain**. In order to do so, we rely on cyclomatic complexity analysis and a few basic identation rules.

We rely on xenon and radon for cyclomatic complexity analysis, and B is the maximum acceptable grade for any function or method. 
For indentation, deep nesting is frowned upon but situation dependent: use code smells (for loop inside a for loop, if statement inside a for loop, etc.) as a guide to refactor code into smaller, more focused functions.


---

## ✅ Accepted conventions

### 1️⃣ Keep cyclomatic complexity low

We aim to keep the cyclomatic complexity of our functions and methods below 10. This means that we should avoid having too many branches (if statements, loops, etc.) in a single function. If a function exceeds this threshold, we should consider refactoring it into smaller, more focused functions.

### 2️⃣ Avoid deep nesting

We should avoid nesting code more than 3 levels deep. Deeply nested code can be difficult to read and understand. If we find ourselves nesting code too deeply, we should consider refactoring by modularizing it to reduce the nesting.

Prefer code that is long but thin (i.e., with low nesting) over code that is short but deeply nested. This often leads to clearer and more maintainable code.


---

## ❌ Anti-patterns (never do these)

- Writing functions with high cyclomatic complexity
```python
def process_data(data: list[Data]) -> list[Result]:
    for x in data:
      if condition1:
          if condition2:
              # do something
              if condition3:
                  # do something else
              else:
                  # do something else
          else:
              # do something else
      elif condition3:
          # do something else
      else:
          # do something else
```

- Writing deeply nested code
```python
def process_data(data: list[Data]) -> list[Result]:
    for x in data:
        if condition1:
            # do something
            for x in range(10):
                if condition2:
                    ...
```