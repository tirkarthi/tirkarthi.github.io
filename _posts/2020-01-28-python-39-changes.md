---
layout: post
title:  python 3.9 compatibility changes
date:   2020-01-28 00:00:29 +0530
categories: programming
---

With the EoL of Python 2 being in line with development of Python 3.9 there were changes made to Python 3.9 that broke a lot of packages since many deprecation warnings became errors. This sparked [a thread](https://mail.python.org/archives/list/python-dev@python.org/thread/EYLXCGGJOUMZSE5X35ILW3UNTJM3MCRE/) on python-dev that the changes should be postponed to 3.10 and later as Fedora is doing a rebuild of its packages with Python 3.9. Below is an image of the in progress build of the broken packages only in Fedora and things across PyPI could be more. The bug is at https://bugzilla.redhat.com/show_bug.cgi?id=1785415

![](/images/python-39-issues.png)

The tracker issue when fedora did a rebuild on Python 3.8 is as below at https://bugzilla.redhat.com/show_bug.cgi?id=1686977

![](/images/python-38-issues.png)

I have filed issues around 85 issues in the upstream repositories for along with 50 PRs for the projects related to Python 3.9 and below are my opinions on the changes as I worked on them for this month.

### Ignored deprecations

The changes that were made in Python 3.9 that broke a lot of packages were stuff that were deprecated from Python 3.4 (March 2014) and before. Sometimes they were deprecated in Python 2 like using `Threading.is_alive` in favour of `Threading.isAlive` to be removed in Python 3. Many such deprecations are still around like unittest assertion helper aliases that are still not made into errors. Though the deprecations were put in place along with the version in which they will be removed they were ignored. Sometimes the deprecations were ignored in large projects like removal of importing ABC from collections module directly that CPython was using for documentation builds. This made the change to be postponed to 3.9 though the plan was for Python 3.8. This has become a chicken and egg scenario where something is deprecated but a popular library ignored it and the upstream delays the removal giving more time that the project might or might not fix based on its priorities.

From a maintenance perspective many packages that were popular but not maintained also get affected like nose with 4 million downloads not maintained and not useable on Python 3.9 due to importing ABC directly from collections. pycrypto not maintained has 4 million downloads and is incompatible with Python 3.8 due to `time.clock` removal. These projects are not going to get fixed and make a release and might need a long term maintenance even if they do. There are many more packages yet to be discovered as more and more packages add Python 3.9 or get reported by users who try them on latest Python.

Fundamentally the libraries ignore them unless they turn all the warnings into errors with -Werror and sometimes the warning might be from a third party package or more users complain about it until someone makes a PR to get it fixed. Getting the fix is in one part and making a release so that it's available to the users is other problem like html5lib affected by collections import yet to have a release. Along with fix the library might need to use shims to maintain compatibility guarantees in case of python 2 support by the library increasing maintenance.

### Language change and deprecation budget

Every programming language goes through a development cycle where new features are introduced and in some cases the changes might be syntactical that their usage will make the code to be syntactically incompatible with previous versions. Depending on the language's compatibility guarantees there might be a very low gap to introduce changes. So the changes are introduced in mind with maximum benefit to the users for the tradeoff. This can be termed as language budget for a given version as I heard it to be used in Java world.

f-strings are one example where using f-strings would mean that versions of Python below 3.6 would produce a syntax error. This means popular libraries or even the ones that get started might need to wait until EoL of Python 3.5 so that they can use it. Python 3.5.0 was released in September 2015 with EoL to be around mid 2020 since each version of Python. Not everybody keeps updating Python and there are large companies that might have transitioned from Python 2 to a version below Python 3.6 that also might have support from vendors and are generally happy to keep using it. So upgrade for these companies come at a cost and a move like 3.5 to 3.6 needs to justify the features that helps development, performance, etc.

Given that Python is getting more and more popular the ecosystem also tends to grow the community also expects a long term support. So changes like positional argument syntax, walrus operator introduced in Python 3.8 needs to wait for drop of Python 3.7 support. Syntax changes that are added in future would also need to take this into consideration. The complexity budget also means that the changes have to be taught to beginners in a way that it will not be overwhelming to them.

### 3.10 and above

3.10 and above might also have more changes like

* Raising `SyntaxWarning` for invalid sequences was reverted to `DeprecationWarning` and might be introduced again.
* `@coroutine` being deprecated that might become an error.
* Passing `loop` to many async functions were deprecated.

Having the changes early in the cycle gathers more feedback about the implications of the change on the ecosystem. Having a CI with -Werror for core language deprecation only would be good but many packages use different build and test systems to coordinate this. There could be something like cpantesters to gather more metric about this.

As said above the changes also come at a cost and the API might sometime be used widely that it just needs to be accepted as a fact of life. As Python is being used more and more the changes the budget shrinks. The core language has to be in line with the philosophy of being simple to learn and easy to use. Each change being made will be supported for 5 years and even more by the ecosystem with upstream and downstream consumers requiring upgrade of packages too. Users of wider spectrum from kids learning Python for fun to large enterprise systems that value stability, low maintenance, etc have to be catered to foster the development and adoption.

_Pushes to git to eat curd rice_