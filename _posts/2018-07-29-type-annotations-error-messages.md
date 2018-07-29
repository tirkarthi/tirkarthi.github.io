---
layout: post
title:  Improving error messages with type annotations
date:   2018-07-29 00:00:29 +0530
categories: programming
---

For the past one month I have been working on CPython contributing both PRs and issues in case of test failures. Python 3 has type annotations which is different from type hints that you can use in Python 2 with a special format of comments to specify types. Through type annotations you can optionally annotate the arguments and return type. They don't do any compile time or runtime checks for types but can be used with tools like [Mypy](https://github.com/python/mypy) for type checking. This post is about an approach to improve the current error messages in CPython by returning type annotations as part of the error messages.

* [GitHub branch](https://github.com/tirkarthi/cpython/tree/error-message-annotations)
* [Python-ideas thread](https://mail.python.org/pipermail/python-ideas/2018-July/052463.html)
* [Bug tracker](https://bugs.python.org/issue34254)

**Current error message format**

```shell
cpython git:(error-message-annotations) $ ./python signatures.py
Traceback (most recent call last):
  File "signatures.py", line 7, in <module>
    get_profile_a(1)
TypeError: get_profile_a() missing 1 required positional argument: 'likes'
```

**Proposed format with type annotations**

```shell
cpython git:(error-message-annotations) $ ./python signatures.py
Traceback (most recent call last):
  File "signatures.py", line 7, in <module>
    get_profile_a(1)
TypeError: get_profile_a() missing 1 required positional argument: 'likes'
Annotation: (user_id: <class 'int'>, likes: typing.Dict[str, int], return: typing.Dict[str, int])
```

### First C contribution to CPython

I came across [an issue](https://bugs.python.org/issue34127) in the bug tracker regarding pluralization in error messages. Some of the error messages were returning "expected 1 arguments" instead of "expected 1 argument" (Note s in argument). This was not really the case across the whole language. Some of the parts were working fine and some where not. The error messages in the issue were generated from `getargs.c` which had these cases. The fix was quite simple to modify the format string from `arguments` to `argument%s` in the message and to supply "s" based on whether the count is 1 or not. I don't check for the count to be less than 1 because it should read "0 arguments" and the error message won't receive negative integer. You can find the PR [here](https://github.com/python/cpython/pull/8395) and there is a [follow up PR](https://github.com/python/cpython/pull/8438) for other instances. A sample code of the logic is as below : 

**Before patch**

`printf("expected %d arguments", argcount);`

```shell
$ python3.6
Python 3.6.4 (default, Mar 12 2018, 13:42:53)
[GCC 4.2.1 Compatible Apple LLVM 7.0.2 (clang-700.1.81)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> foo = {}
>>> foo.get()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: get expected at least 1 arguments, got 0
```

**After patch**

`printf("expected %d argument%s", argcount, argcount == 1 ? "s" : "");`

```shell
$ ./python
Python 3.8.0a0 (heads/error-message-annotations:9f28f9a, Jul 28 2018, 14:28:34)
[GCC 5.4.0 20160609] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> foo = {}
>>> foo.get()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: get expected at least 1 argument, got 0
```

### Type annotations in functions

Python3 supports type annotations so you can add annotations to the functions and you can run Mypy during your CI step or so to catch calls that are invalid. 


```python
# signatures.py

from typing import Dict

def get_profile_a(user_id: int, likes: Dict[str, int]) -> Dict[str, int]:
    return {'user_id': user_id, 'likes': sum(likes.values())}

if __name__ == "__main__":
    get_profile_a(1, {'1': '1'})
```

You can see that we take the values of likes dictionary and sum them which expect a list of integers but we give it a string '1'. When we run it we get a runtime error like below

```shell
$ python3.6 signatures.py
Traceback (most recent call last):
  File "signatures.py", line 7, in <module>
    get_profile_a(1, {'1': '1'})
  File "signatures.py", line 4, in get_profile_a
    return {'user_id': user_id, 'likes': sum(likes.values())}
TypeError: unsupported operand type(s) for +: 'int' and 'str'
```

When we run this against Mypy it generates the below error message and as you can see it takes around half a second.

```shell
$ time mypy signatures.py
signatures.py:7: error: Dict entry 0 has incompatible type "str": "str"; expected "str": "int"
mypy signatures.py  0.61s user 0.03s system 99% cpu 0.649 total
```

This is a simple application of Mypy and about runtime error messages in Python.

### Type annotations in error messages

We can see that the error messages while performing doing runtime computations with incompatible types. There is another common case where we miss some of the arguments while playing around with the API. They also generate error messages as below at runtime as below

```python
# signatures.py

from typing import Dict

def get_profile_a(user_id: int, likes: Dict[str, int]) -> Dict[str, int]:
    return {'user_id': user_id, 'likes': sum(likes.values())}

if __name__ == "__main__":
    get_profile_a(1)
```

```shell
$ python3.6 signatures.py
Traceback (most recent call last):
  File "signatures.py", line 7, in <module>
    get_profile_a(1)
TypeError: get_profile_a() missing 1 required positional argument: 'likes'
```

This generates an error message about the argument being absent. But since we have type annotations in the function wouldn't it be better if it included the type annotation in the error as well. The results of my approach is as below : 

```shell
cpython git:(error-message-annotations) $ ./python signatures.py
Traceback (most recent call last):
  File "signatures.py", line 7, in <module>
    get_profile_a(1)
TypeError: get_profile_a() missing 1 required positional argument: 'likes'
Annotation: (user_id: <class 'int'>, likes: typing.Dict[str, int], return: typing.Dict[str, int])
```

This approach along with the previous case also gives us the annotation of the parameter likes so we know the type of the argument along with more context about the function itself.

### Code behind errors

When I was contributing my initial PR I came across this idea and I thought to give this a shot so my first search was to see from which function the error occurs. I gave rg a try with `rg 'required positional argument' *` but this doesn't return me anything useful. So I tweaked it a little bit as `rg 'missing.*required.*argument' *` and gave a try. This gave me a bunch of test files and harcoded error messages and gave a result in Python/eval.c where `format_missing` method was generating the error message. I changed `argument` to `argument123` and recompiled python. This was the function that was generating the error messages.

```shell
$ python3.6 signatures.py
Traceback (most recent call last):
  File "signatures.py", line 7, in <module>
    get_profile_a(1)
TypeError: get_profile_a() missing 1 required positional argument123: 'likes'
```

Now we know the function and the next step was to know how we can get the type annotations for a function object. In Python there is an attribute `__annotations__` through which you can get the annotations of a function as a dictionary.

```shell
$ ./python
Python 3.8.0a0 (heads/error-message-annotations:9f28f9a, Jul 28 2018, 14:28:34)
[GCC 5.4.0 20160609] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import signatures
>>> signatures.get_profile_a
<function get_profile_a at 0x7f34f8e45d60>
>>> signatures.get_profile_a.__annotations__
{'user_id': <class 'int'>, 'likes': typing.Dict[str, int], 'return': typing.Dict[str, int]}
```

The problem was that we only have the code object inside the function and we don't have access to the function object. One way to obtain this is to get the value of `globals()` and then do a lookup with `code_object->co_name` so this gives us the function object. We can get all the globals with respect to current frame with the C function [PyEval_GetGlobals](https://docs.python.org/3/c-api/reflection.html#c.PyEval_GetGlobals). This returns a dictionary and through with co_name as key we can get the function object. Python also exposes C API to return annotations for a function object called [PyFunction_GetAnnotations](https://docs.python.org/3/c-api/function.html#c.PyFunction_GetAnnotations) . Now that we have a dictionary of annotations we can iterate through the dictionary to construct the annotation string we need to add to the error message. The sample code is as below : 


```c
    // get all globals
    int set = 0;
    globals = PyEval_GetGlobals();
    func_object = PyDict_GetItem(globals, co->co_name);
    if (func_object != NULL) {
        annotations = PyFunction_GetAnnotations(func_object);
    }

    // TODO: Handle cases in get_type_hints
    // TODO: Honor __no_type_check__ ?
    // TOOD: Handle forward references

    // Annotations are present then we can form a annotation string
    if (annotations != NULL) {
        while (PyDict_Next(annotations, &pos, &key, &value)) {
            // Take the string representation of the type. int -> <class 'int'>
            // Dict[str, int] -> typing.Dict[str, int]
            type_name = PyObject_Str(value);
            if (set == 0) {
                // Testing function_annotations to be NULL is not reliable so use this for initialization
                function_annotations = PyUnicode_Concat(PyUnicode_FromFormat("%U: ", key), type_name);
                // Set value to indicate annotations are present
                set += 1;
            } else {
                // Make a temporary string for current item and then append to existing string
                temp = PyUnicode_Concat(PyUnicode_FromFormat(", %U: ", key), type_name);
                function_annotations = PyUnicode_Concat(function_annotations, temp);
            }
        }
    }

    if (set == 0) {
        PyErr_Format(PyExc_TypeError,
                     "%U() missing %i required %s argument%s: %U",
                     co->co_name,
                     len,
                     kind,
                     len == 1 ? "" : "s",
                     name_str);
    } else {
        PyErr_Format(PyExc_TypeError,
                     "%U() missing %i required %s argument%s: %U \nAnnotation: (%U)",
                     co->co_name,
                     len,
                     kind,
                     len == 1 ? "" : "s",
                     name_str,
                     function_annotations);
     }
```

**After patch**

```shell
cpython git:(error-message-annotations) $ ./python signatures.py
Traceback (most recent call last):
  File "signatures.py", line 7, in <module>
    get_profile_a(1)
TypeError: get_profile_a() missing 1 required positional argument: 'likes'
Annotation: (user_id: <class 'int'>, likes: typing.Dict[str, int], return: typing.Dict[str, int])
```

### Not so fast

Things seemed to work fine but when I wrote test cases it was still generating the old error messages. Something was wrong and after some debugging it dawned to me that the test functions are defined as a function inside a class or as a function inside another function so when it hits the code to generate the error message I take `code_object->co_name` to look it up in globals but it's not a global variable.

```shell
cpython git:(error-message-annotations) $ rlwrap ./python
Python 3.8.0a0 (heads/error-message-annotations:9f28f9a, Jul 28 2018, 14:28:34)
[GCC 5.4.0 20160609] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> class Foo:
...     def square(number: int) -> int:
...         return number * number
...
>>> Foo.square.__code__.co_name
'square'
>>> global_vars = globals()
>>> global_vars.get(Foo.square.__code__.co_name, "Not found")
'Not found'
>>> Foo.square.__annotations__
{'number': <class 'int'>, 'return': <class 'int'>}
```

So we have hit a wall where this doesn't work for methods of a class and for functions defined inside a function. This is a real blocker since we can't test this now and also it's only useful for top level functions. So the only was to pass the function object inside `format_missing` function to get the annotations instead of trying to get the function object from globals. So this led me to modify the function API and since this was not documented I went through the call stack and passed along the function object. Now we can see that it works for these cases too

```shell
cpython git:(error-message-annotations) $ rlwrap ./python
Python 3.8.0a0 (heads/error-message-annotations:9f28f9a, Jul 28 2018, 14:28:34)
[GCC 5.4.0 20160609] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> class Foo:
...     def square(number: int) -> int:
...         return number * number
...
>>> Foo.square()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: square() missing 1 required positional argument: 'number'
Annotation: (number: <class 'int'>, return: <class 'int'>)
```

### Areas of improvement

Now I have fixed this for `format_missing` which is used for errors with less arguments there is a function called `too_many_positional` and other places where we need to do similar logic and should have access to the function object for this. As you can see this needs to be refactored out as a common function or something. I couldn't see any performance impacts of this since mostly annotations is a small dictionary and most of them are hash calls that are already optimised. Also in most cases when an argument is missing for a function it's good to have little cost to display some context where this more helpful.

More general areas and problems are as below : 

* Stick to a format. Will it make more sense to say the below error which is more compact and also makes `<class 'int'>` as `int` and make `return` annotation to be after `->`

```shell
>>> Foo.square()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: square() missing 1 required positional argument: 'number'
Annotation: square(number: int) -> int:
```
* Is there additional way to make this more general so that code duplication can be avoided and it's implemented across other places like unknown keyword argument, too many arguments and so on.
* Most importantly how do you feel about the new error messages in general. Python error messages are good but there are languages like Elm and Rust that have significant effort towards error messages that help beginners.
* Also this happens only when the functions are annotated and it doesn't do automatic type inferencing. This might motivate developers to write more type annotations since the ROI is good now.
* Will this be accepted by the core team given the code complexity and the benefit it adds?

### Conclusion

Given my beginner level C skills there are more areas in the code to improve. You can take a look at the code and compile it. Give this a shot and maybe report cases where this segaults the compiler :) Though I am okay with this not being accepted or not feasible across the language I am happy I am able to hack through things to learn more about the language and perhaps suggest more feasible improvements. This wouldn't have possible without Python being open source and the community that drives innovation across the language and the ecosystem. So thanks Guido and everyone.

_Closes emacs and checks his CPython test build :)_

If you like this post then you might like my other posts on similar subjects : 

* [My joys and mistakes with open source](https://tirkarthi.github.io/programming/2018/06/04/500-commits-of-open-source.html)
* [Analysing 30k issues CPython bug tracker](https://tirkarthi.github.io/python/2018/06/26/analyzing-python-bug-tracker.html)
* [Analysing 10k modules in Clojure ecosystem for JDK 9 problems](https://tirkarthi.github.io/clojure/2018/03/17/data-mining-clojars.html)
* [Analysing Rust crates for top dependencies and security issues](https://tirkarthi.github.io/rust/2018/03/30/analyzing-crates-data.html)