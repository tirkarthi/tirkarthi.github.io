---
layout: post
title:  Analyzing bugs.python.org
date:   2018-06-27 00:00:29 +0530
categories: python
---

I was looking to contribute to CPython and I made my first PR which consisted of running aspell through the code looking for typos. I was quite amazed at how smooth the process is. CPython moved to GitHub last year and this has opened up to a lot of integrations like GitHub bots, PR being linked to the issue tracker and so on. This also helps people who want to contribute to the project to be up and running with less friction. CPython was using subversion before moving to mercurial in 2011. They again moved to git and GitHub last year. You can read the full history in the [post](https://snarky.ca/the-history-behind-the-decision-to-move-python-to-github/) by Brett Cannon. From time to time there were also easy issues that are triaged by the core developers for beginners to pick along with providing mentorship. This also helps in growing the team of contributors who can help in fixing and triaging the issues.

Similar posts I wrote on Clojure and Rust ecosystem.

* [Analyzing around 10k modules in Clojure ecosystem to find JDK 9 issues](https://tirkarthi.github.io/clojure/2018/03/17/data-mining-clojars.html)
* [Analyzing crates.io to find top rust dependencies and insecure dependencies](https://tirkarthi.github.io/rust/2018/03/30/analyzing-crates-data.html)


### Obtaining bugs.cpython.org data

I wrote a little scraper that downloads each issue and since it's a static page we need to parse the page to obtain relevant information. Since this is a quick hack I thought MongoDB will be a good fit where you can just dump the data as JSON and query easily. Hence a issue along with the title, python version against which it's filed and comments looks as below. The code and JSON files are available on [GitHub](https://github.com/tirkarthi/cpython-bugs)

```JSON
{
	"_id" : ObjectId("5b335855e63827190a6c1f75"),
	"component" : [
		"Tests"
	],
	"version" : [
		"Python 3.8"
	],
	"title" : "Issue 33853: test_multiprocessing_spawn is leaking memory - Python tracker",
	"content" : [
		{
			"author" : "Author: Pablo Galindo Salgado (pablogsal) *",
			"date" : "Date: 2018-06-13 15:10",
			"comment" : "The test `test_multiprocessing_spawn` is leaking memory according to the x86 Gentoo Refleaks 3.x buildbot:\n\n\nx86 Gentoo Refleaks 3.x\nhttp://buildbot.python.org/all/#/builders/1/builds/253\n\ntest_multiprocessing_spawn leaked [1, 2, 1] memory blocks, sum=4\n1 test failed again:\n    test_multiprocessing_spawn\n\n\nx86 Gentoo Refleaks 3.7\nhttp://buildbot.python.org/all/#/builders/114/builds/135"
		},
		{
			"author" : "Author: STINNER Victor (vstinner) *",
			"date" : "Date: 2018-06-13 15:30",
			"comment" : "Duplicate of bpo-33735."
		}
	]
}
```

### Top issues by number of comments

I was interested to study issues that involved a lot of discussions that could provide a context of how a decision is made or a tough issue is fixed. Since we have content which has an array of comments I group by the length of the comments and then obtain the top issues by number of comments. The gives us the top issues by number of comments made along with the state of the issue. Adding a new regex module compatible with re is the most commented issue and it's also still open. The issue was created on 2008-04-15 by timehorse.

| title                                                                                                                                                             | message_count | state  |
|-----------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------|--------|
| [Issue 2636: Adding a new regex module (compatible with re)](https://bugs.python.org/issue2636)                                                                   | 333           | open   |
| [Issue 13703: Hash collision security issue](https://bugs.python.org/issue13703)                                                                                  | 326           | closed |
| [Issue 23496: Steps for Android Native Build of Python 3.4.2](https://bugs.python.org/issue23496)                                                                 | 176           | open   |
| [Issue 2292: Missing *-unpacking generalizations](https://bugs.python.org/issue2292)                                                                              | 172           | closed |
| [Issue 26839: Python 3.5 running on Linux kernel 3.17+ can block at startup or on importing the random module on getrandom()](https://bugs.python.org/issue26839) | 172           | closed |
| [Issue 1602: windows console doesn't print or input Unicode](https://bugs.python.org/issue1602)                                                                   | 148           | closed |
| [Issue 10181: Problems with Py_buffer management in memoryobject.c (and elsewhere?)](https://bugs.python.org/issue10181)                                          | 147           | closed |
| [Issue 12326: Linux 3: code should avoid using sys.platform == 'linux2'](https://bugs.python.org/issue12326)                                                      | 145           | closed |
| [Issue 6715: xz compressor support](https://bugs.python.org/issue6715)                                                                                            | 138           | closed |
| [Issue 16991: Add OrderedDict written in C](https://bugs.python.org/issue16991)                                                                                   | 134           | closed |

### Top authors by number of comments

Since CPython is driven by volunteers along with companies who use Python a lot employing core contributors I was also interested in the authors who made a lot of comments that is proportional to their activity in the bug tracker. This was a case of unwinding the content array that had the comments and then grouping by the author. This gives us the top authors by number of comments made. Victor Stinner commented more on the bug tracker followed by roundup robot which adds comments when a merge is made along with the changeset information and so on.

| author                                        |   count |
|:----------------------------------------------|--------:|
| Author: STINNER Victor (vstinner) *           |   15556 |
| Author: Roundup Robot (python-dev)            |   13359 |
| Author: Serhiy Storchaka (serhiy.storchaka) * |   13147 |
| Author: Antoine Pitrou (pitrou) *             |   12480 |
| Author: R. David Murray (r.david.murray) *    |    8177 |
| Author: Terry J. Reedy (terry.reedy) *        |    6231 |
| Author: Nick Coghlan (ncoghlan) *             |    4939 |
| Author: Éric Araujo (eric.araujo) *           |    4776 |
| Author: Mark Dickinson (mark.dickinson) *     |    4251 |
| Author: Raymond Hettinger (rhettinger) *      |    4183 |

### Top issues by Python version

Bug tracker has a mechanism where you can specify the Python version against which you want to file an issue. Often times there are multiple versions affected where you need to fix in the latest version and then backport them to supported versions. The issue distribution with respect to Python version shows Python 2.7 to be the most popular version with around 9.5k issues filed both open and closed. This is followed by Python 3.4 and other Python 3 versions.

| version    |   count |
|:-----------|--------:|
| Python 2.7 |    9516 |
| Python 3.4 |    6387 |
| Python 3.5 |    6295 |
| Python 3.6 |    6037 |
| Python 3.3 |    5480 |
| Python 3.2 |    5012 |
| Python 3.7 |    4269 |
| Python 2.6 |    2997 |
| Python 3.1 |    2650 |

### Comment distribution

I also made a query to return the distribution of number of comments per issue.

|  msgs |   count |
|------:|--------:|
|     2 |    5197 |
|     3 |    4437 |
|     4 |    3574 |
|     5 |    2843 |
|     6 |    2302 |
|     7 |    1818 |
|     8 |    1498 |
|     1 |    1272 |
|     9 |    1222 |
|    10 |    1060 |

### Future development

Data enables us to visualize interesting points with the bug tracker being a central place with a lot of historical information and going through the bug tracker often gives us an idea of the  efforts and details with respect to an issue providing more context. There was also a [discussion](https://mail.python.org/pipermail/python-committers/2018-May/005428.html) on using GitHub issues for future development which could help in more integrations and smooth workflow. The [devguide](https://github.com/python/devguide/) is also available on GitHub where you can suggest improvements to the workflow. CPython development has come a long way so far with GitHub and looking forward to more improvements on CPython development.

If you like the above post then you might like similar posts below :

* [Analyzing around 10k modules in Clojure ecosystem to find JDK 9 issues](https://tirkarthi.github.io/clojure/2018/03/17/data-mining-clojars.html)
* [Analyzing crates.io to find top rust dependencies and insecure dependencies](https://tirkarthi.github.io/rust/2018/03/30/analyzing-crates-data.html)

__Closes emacs and pushes to git__
