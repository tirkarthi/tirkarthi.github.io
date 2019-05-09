---
layout: post
title:  f-string debugging in Python 3.8
date:   2019-05-09 00:00:29 +0530
categories: programming
---

print style debugging is a form of debugging where print statements are inserted to print values of expressions or variables that we need to track. loggers are common if we want to use the log statements in production. But there are many times where quick print statements will do the trick in debugging and understanding the control flow.

### f-strings

f-strings were introduced in Python 3.6 with [PEP 498](https://www.python.org/dev/peps/pep-0498/). With f-strings you can evaluate an expression as part of the string along with inserting result of function calls and so on. This is not a very novel idea and is present in many languages like Perl, Ruby, JavaScript etc. A basic usage of this is as below

```pycon
>>> name = "karthikeyan"
>>> print(f"Hello, {name}")
Hello, karthikeyan
>>> print(f"Hello, {name.capitalize()}")
Hello, Karthikeyan
```

### f-strings debugging

f-strings simplified a lot of places where str.format and % style formatting. There was still a place where you want to print a value of the variable or expression and also add some context with the string like variable name or some arbitrary name so that when you have many statements you can differentiate between the printed values. So using variable name followed by value is more common format of print style debugging.

This caused users to write `f"name = {name}"` and can get unwieldy when variable names are long like `filtered_data_from_third_party` would be written as `f"filtered_data_from_third_party = {filtered_data_from_third_party}"`. In those cases we resort to shorter names we understand easily at the context like `f"filtered data {filtered_data_from_third_pary}"`. f-strings also support format specifiers so you can write `f"{name!r}"` which is same as `f"{repr(name)}"`.

Given the above boilerplate [an idea](https://mail.python.org/pipermail/python-ideas/2018-October/053956.html) was posted in python-ideas around a format specifier where you can use it and the f-string would expand like a macro into `<variable_name> = <value_of_variable>`. Initially `!d` was chosen so `f"{name!d}"` would expand to `f"name={repr(name)}"`. This was initially implemented in [bpo36774](https://bugs.python.org/issue36774). After discussion `!d` was changed to use `=` with input from Guido and other core devs since there could be usecases served by `!d` in the future and to reserve alphabetical names in format-string for other uses. So this was changed to use `=` as the notation for this feature in [bpo36817](https://bugs.python.org/issue36817). An overview of it's usage is as below with various features available.

```pycon
./python.exe
Python 3.8.0a4+ (heads/master:88db8bd064, May  9 2019, 15:54:59)
[Clang 7.0.2 (clang-700.1.81)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> name = "karthikeyan"
>>> f"name={name}"
'name=karthikeyan'
>>> f"{name=}"
"name='karthikeyan'"
```

You can see an extra set of single quotes because the repr of the variable is used for printing unlike the value. Using `=` helps here but it would be nice if there is some space around the operator so that it's more readable. You can use space around `=` as in `f"{name =}"` that will expand to `f"name ={name}`

```pycon
>>> f"{name =}"
"name ='karthikeyan'"
>>> f"{name = }"
"name = 'karthikeyan'"
```

You can also any expression in similar manner where the expression is printed and the repr of return value is printed.

```pycon
>>> f"{name.upper()=}"
"name.upper()='KARTHIKEYAN'"
>>> f"{name.upper() = }"
"name.upper() = 'KARTHIKEYAN'"
```

You can also insert symbols before and after the repr value too.

```pycon
>>> f"{name.upper() = :-^20}"
'name.upper() = ----KARTHIKEYAN-----'
>>> f"{name.upper() = :->20}"
'name.upper() = ---------KARTHIKEYAN'
>>> f"{name.upper() = :>20}"
'name.upper() =          KARTHIKEYAN'
```

Normal comparisons using `==` should also work fine being backwards compatible.

```pycon
>>> count = 3
>>> f"{count==3}"
'True'
>>> f"{count<=3}"
'True'
>>> f"{count>=3}"
'True'
```

### Changing output format

I also used Rust's `dbg!` macro in the past which is similar to f-string but it also included the filename and line number. The macro is [implemented](https://doc.rust-lang.org/beta/src/std/macros.rs.html#322-337) like a user-defined one since Rust supports macros. So I can print the variable and also get additional context over the filename and line number. This is helpful in cases where there are multiple places where I track the value of the same variable to see change in state

```rust
dbg!(name) // Prints [src/main.rs:4] name = "karthikeyan"
```

```python
# prog.py

name = "karthikeyan"
age = 25

print(f"{name = }")
print(f"{age = }")

age += 1
name = name.upper()

print(f"{name = }")
print(f"{age = }")

```

```
$ ./python.exe prog.py
name = 'karthikeyan'
age = 25
name = 'KARTHIKEYAN'
age = 26
```

This gives me name and age but could be less useful when there are 10 statements sprinkled around a larger program. Some statements could be inside if statement that is conditionally executed and so on. With respect to Python we need to change the compiler internals to add support for this since there is no macro system like Rust. But the change seemed to be simple as I looked into the [implementation](https://github.com/python/cpython/pull/13123/files#diff-4d35cf8992b795c5e97e9c8b6167cb34R5232) . The expression is formed as text with below statement.

```c
Py_ssize_t len = expr_text_end-expr_start;
expr_text = PyUnicode_FromStringAndSize(expr_start, len);
```

I checked out the C API utilities present and around the file. After a couple of segfaults I got this hack to work

1. It seems filename is present from a compiling object that I can obtain from `c->c_filename`.
2. The node object can be used with a macro `LINENO` to obtain the line number of the node in this case the f-string.
3. C API provides `PyUnicode_FromFormat` like a format string for the objects so I can wrap get the format `[filename:lineno]` like Rust's `dbg!` macro
4. I can use `PyUnicode_Concat` to concat `[filename:lineno]` and `expr_text` as `expr_text = PyUnicode_Concat(location, expr_text);`

```c
PyObject *location = NULL;
location = PyUnicode_FromFormat("[%S:%d] ", c->c_filename, LINENO(n)); // [filename:lineno]
expr_text = PyUnicode_Concat(location, expr_text); // "[filename:lineno] [expr_text]"
```

I applied three lines of change and compiled the build. Voila! we have filename and line number support.

```
./python.exe prog.py
[prog.py:6] name = 'karthikeyan'
[prog.py:7] age = 25
[prog.py:12] name = 'KARTHIKEYAN'
[prog.py:13] age = 26
```

Since `PyUnicode_FromFormat` can be changed I can change it fit my personal preference like printing `filename:lineno |> expr_text`

```console
./python.exe prog.py
prog.py:6 |> name = 'karthikeyan'
prog.py:7 |> age = 25
prog.py:12 |> name = 'KARTHIKEYAN'
prog.py:13 |> age = 26
```

Links :

* [python-ideas thread](https://mail.python.org/pipermail/python-ideas/2018-October/053956.html)
* [Using !d in f-strings](https://bugs.python.org/issue36774)
* [Using = in f-strings](https://bugs.python.org/issue36817)
* [Implementation PR](https://github.com/python/cpython/pull/13123/)
* [Rust dbg! macro](https://doc.rust-lang.org/beta/std/macro.dbg.html)
* [filename:lineno branch](https://github.com/tirkarthi/cpython/tree/fstring-filename)

There are many more ways in which you can customise, it's a matter of personal preference and tradeoff over using print statement or resorting to a logger that is more customizable without changing C internals of the interpreter. But I loved playing around with this and hope it will be used by more people in the future. It just missed the merge window for 3.8 alpha 4 and will be available as part of first beta 3.8 beta 1. Thanks to Eric V. Smith and Larry Hastings for this feature.

Feel free to leave a comment about the feature. Suggestions on writing style are also welcome.

_Switches tmux to see PGO build results_
