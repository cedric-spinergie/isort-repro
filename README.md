Reproduction repository for https://github.com/PyCQA/isort/issues/2200.



# Original issue description

## Context

I have a particular setup with a repo in which there are two folders containing python projects.  
Those project each have their own `pyproject.toml` with their own config for `isort`.

If I run `isort project1`, everything works fine. Same with `isort project2`. However, if I do `isort .` from the root directory, `isort` doesn't seem to find the configuration and considers all firstparty imports to be thirdparty instead.

## Reproduction

I have a reproduction repo for people to see what goes wrong.  

<details>

<summary>To be future proof</summary>

### Structure

```
 > tree
.
├── django
│   ├── example.py
│   ├── pyproject.toml
│   └── setting.py
└── scripts
    ├── example2.py
    ├── pyproject.toml
    └── value.py

2 directories, 6 files
```

### Files in `django`

**django/pyproject.toml**
```toml
[tool.isort]
profile = "black"
line_length = 120
sections = "FUTURE,STDLIB,THIRDPARTY,FIRSTPARTY,LOCALFOLDER"
```

**django/example.py**
```py
import sys

import pandas

from setting import test


def my_example(request):
    print(test)
```

**django/setting.py**
```py
test = "hello"
```

### Files in `scripts`

**scripts/pyproject.toml**
```toml
[tool.isort]
profile = "black"
line_length = 120
sections = "FUTURE,STDLIB,THIRDPARTY,FIRSTPARTY,LOCALFOLDER"
```

**scripts/example2.py**
```py
import sys

import pandas

from value import test2


def second_test(request):
    print(test2)
```

**scripts/value.py**
```py
test2 = "hola"
```

</details>

## The problem

As you can see, the main files `django/example.py` and `scripts/example2.py` each have 3 imports:
- 1 stdlib import (`sys`)
- 1 third party import (`pandas`)
- 1 first party import (`setting` or `value`)

By default, those imports are properly separated ; showing that `isort` properly understands the category they each belong to.

Now, if we do `isort django` or `isort scripts`, nothing changes, as expected.  
But if we do `isort .`, we'll see the main files change to put `setting` / `value` into the third party exports category.

**I expect `isort django`, `isort scripts` and `isort .` to behave the exact same way and always result in the exact same code.**

Fun facts:
- `isort django scripts` will properly format `django` but mess up `scripts`.
- `isort scripts django` will properly format `scripts` but mess up `django`.

> The option `--resolve-all-configs` does not change anything. It's as if it were entirely ignored or maybe I misunderstood its purpose.

## Version

- `isort 5.12.0`
- `Python 3.9.18`

