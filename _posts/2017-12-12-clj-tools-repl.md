---
layout: post
title:  Clojure Deps Python Guide
date:   2017-12-12 05:14:29 +0530
categories: clojure
---

With Clojure 1.9 being one of the biggest releases with the inclusion of spec it also includes better tools for Clojure installation and to run standalone programs. Before this inclusion one needs to manually download the clojure JAR and then do `java -cp <path-to-clojure.jar> -clojure.main` to start the REPL and though the Clojure ecosystem highly recommends REPL based workflows sometimes you need to just run a script with some file as parameter instead of waiting for Lein to startup. Lumo has done a great job at this part but the current tools are also a great start in the direction. This tutorial thus compares the new tooling with Python's REPL options.

## Installation of Clojure

You can refer to [installation](https://clojure.org/guides/getting_started) and on MacOS it's as simple as `brew install clojure`. This downloads Clojure JAR and installs two helper bash scripts called `clj` and `clojure` on $PATH. You can view the file as below :

```shell
➜  hello-world $ which clj
/usr/local/bin/clj
➜  hello-world $ cat /usr/local/bin/clj
#!/usr/bin/env bash

if type -p rlwrap >/dev/null 2>&1; then
  rlwrap -r -q '\"' -b "(){}[],^%3@\";:'" clojure "$@"
else
  echo "Please install rlwrap for command editing or use \"clojure\" instead."
fi
```

It's basically a wrapper with rlwrap around Clojure. Similarly it installs a shell script of the name `clojure` which is a little more complicated to handle all the options. You can access it as below :

```shell
➜  hello-world which clojure
/usr/local/bin/clojure
```

## deps.edn

The `clj` uses a file called `deps.edn` to take care of the path for dependencies, code, loading older dependencies, etc. We can set the path to be used to look for code with the key `:path` and similar to lein we can dependencies under `:deps`. A sample `deps.edn` to look for code in `src` and in current working directory is as below :

```
{
 ;; Paths in project
 :paths ["src", "."]

}
 ```

## Start a REPL

* Python

```shell
➜  hello-world $ python3
Python 3.5.2 (default, Oct 11 2016, 05:00:16)
[GCC 4.2.1 Compatible Apple LLVM 7.0.2 (clang-700.1.81)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>>
```

* Clojure

```shell
➜  hello-world $ clj
Clojure 1.9.0
user=>
```

## Execute a one liner

* Python

```shell
➜  hello-world $ python3 -c  'print("Hello, World!")'
Hello, World!
```

* Clojure

```shell
➜  hello-world $ clj -e  'println("Hello, World!")'
Hello, World!
```

## Run a hello world script

* Python

You can have a file called `hello.py` and run it with `python3 hello.py`

```
➜  hello-world $ cat hello.py
print("Hello, World!")
➜  hello-world $ python3 hello.py
Hello, World!
```

* Clojure

In clojure we can create a new file called `hello.clj` and then declare the namespace hello and print the same string

```shell
➜  hello-world $ cat hello.clj
(ns hello)

(println "Hello, World!")
➜  hello-world $ clj hello.clj
Hello, World!
```

## main function entry point

* Python

In Python we can define main function with the common boilerplate `if __name__ == "__main__"` . When the entry point is defined for a module we can execute it as `python -m hello` which executes the main function of the module.

```shell
➜  hello-world $ cat hello.py
if __name__ == "__main__":
    print("Hello, World!")
➜  hello-world $ python3 hello.py
Hello, World!
➜  hello-world $ touch  __init__.py
➜  hello-world $ python3 -m hello
Hello, World!
```

* Clojure

In clojure we can define the same main function except that we pass the `-m` flag to clj so that `-main` function is executed. In addition to that we pass the name of the namespace instead of the file itself. In this case we call `clj -m hello`

```shell
➜  hello-world $ cat hello.clj
(ns hello)

(defn -main
  []
  (println "Hello, World!"))
➜  hello-world $ clj -m hello
Hello, World!
```

## Use dependencies

* Python

In Python for the most part we install a library with pip inside a virtual environment and then use it with normal import calls inside a REPL.

```shell
➜  hello-world python3
Python 3.5.2 (default, Oct 11 2016, 05:00:16)
[GCC 4.2.1 Compatible Apple LLVM 7.0.2 (clang-700.1.81)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> import json
>>> json.dumps({"a": 1})
'{"a": 1}'
```

* Clojure

In Clojure we need to declare the dependency in the deps.edn so that it can be included in the classpath similar to python `sys.path`. The library is downloaded for the first time and then reused for the subsequent calls

```shell
➜  hello-world $ cat deps.edn
{
 ;; Paths in project
 :paths ["src", "."]

 ;; Project dependencies, a map from lib to coordinate
 :deps {
        cheshire/cheshire {:mvn/version "5.8.0"}
        }
}
➜  hello-world $ clj
Clojure 1.9.0
user=> (require '[cheshire.core :refer :all])
nil
user=> (encode {:a 1})
"{\"a\":1}"
```

## Pain points

* Clojure startup time is much more heavy and since Clojure 1.9 loads spec as it hits main the REPL startup is slow. As you can see it takes 8x more time compared to Python 3.

```
➜  hello-world echo | time clj
Clojure 1.9.0
user=> user=>
clj  3.16s user 0.18s system 199% cpu 1.677 total
➜  hello-world echo | time python3
python3  0.07s user 0.02s system 39% cpu 0.226 total
```

* No auto-completion like lein since it's not a lightweight version of Lein but more like a plain REPL. Python 3 provides good auto-completion compared to Clojure.
* The tooling around the new set of tools is also low that you can't start a new clj REPL and then send the code from an emacs buffer to the REPL. Of course it can be written with elisp it's just
that there are no packages built around it.

## Much more

This tutorial is more of a comparison between Python and Clojure with respect to the new tools. The [guide](https://clojure.org/guides/deps_and_cli) and [reference](https://clojure.org/reference/deps_and_cli) can serve as good resource to explore more options like alias, dependency overriding, etc.

## Thanks :)

This release is pretty cool with the new tools and inclusion of spec. I am very much happy towards the efforts undertaken by the community for almost 2 years for this release and much more curious to see how they shape up in the next version 1.10 and years to come. With Clojure released just before Christmas I hope you have more fun and wish you all a merry Christmas in advance :)
