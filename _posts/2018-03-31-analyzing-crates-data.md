---
layout: post
title:  Analysing crates.io data
date:   2018-03-31 00:00:29 +0530
categories: rust
---

> I do a lot of work on open source, but my most valuable contributions haven’t been code. Writing a patch is the easiest part of open source. The truly hard stuff is all of the rest: bug trackers, mailing lists, documentation, and other management tasks - Steve Klabnik, How to be an open source gardener.

The above [quote](http://words.steveklabnik.com/how-to-be-an-open-source-gardener) by Steve Klabnik who is a prolific open source contributor and currently involved with Rust emphasises the importance of bug filing, docs, etc. I am learning Clojure for the past one year and I thought making open source contributions is a great way to interact with the community. I made a [post](https://tirkarthi.github.io/clojure/2018/03/18/data-mining-clojars.html) previously on using Clojars metadata to analyse JDK 9 and Clojure 1.9 issues that helped me file issues to ensure compatibility. I used the same method here to find the modules that were broken on a nightly version of a rustc due to a recent stabilisation.

## TryFrom / TryInto Stabilisation

I was looking around for easy issues that I can fix in the Rust ecosystem and I looked into some of the highly downloaded modules and I came across rand which had a failure against using the nightly version of rustc. When I looked into the travis logs there seemed to be an issue with one of the dependencies that caused the whole test suite to fail. The issue was with stdweb and I looked into stdweb which also had a failure on nightly but the tests had passed a few days back. So I thought to look around the changes merged to nightly version and I searched for the PR and issues with `try_from`. I came across the [PR](https://github.com/rust-lang/rust/pull/49305) regarding the stabilisation of the feature that seemed to be the source of the breakage and a lot of modules failed to compile.

## Dependency calculation using crates.io-index

I thought this is a problem similar to Clojure 1.9 where they made a stricter version of a part of the syntax that caused a lot of core modules to fail and subsequently caused a lot of modules in the ecosystem to fix it in their codebase or upgrading to the fixed upstream dependency to make sure it needs to work. I looked around for data and fortunately I was pointed to the [crates.io-index](https://github.com/rust-lang/crates.io-index) with help from IRC. It had JSON data of all the modules. So I inserted the data into MongoDB since it was easy to query around JSON and I need to worry less about schema for this hack.

## Finding top breakages

I first downloaded the data using `wget https://github.com/rust-lang/crates.io-index/archive/master.zip`. After unzipping the repo I can see the file structure. For example rand was present at `ra/nd/rand` . I used `find . -type f > files.txt` which returned me all the JSON file paths. You can see that every file has a line with a JSON dump of the metadata for the version. Hence the last line is always the latest version which we need to insert. I created a database called `crates` and collection named `crates_data` I wrote a simple Python script that read all the file paths from files.txt and inserted the last line of the file into MongoDB.


**Python file**

https://gist.github.com/tirkarthi/846f9d2fd87c90463c5ca9cf99f3d44d#file-scraper-py

**Sample record**

```Shell
> db.crates_data.findOne({})
{
	"_id" : ObjectId("5abe2ab5e638270f08d60e8a"),
	"name" : "my-password",
	"vers" : "0.1.0",
	"deps" : [
		{
			"name" : "tiny-keccak",
			"req" : "^1.3",
			"features" : [ ],
			"optional" : false,
			"default_features" : true,
			"target" : null,
			"kind" : "normal"
		},
		{
			"name" : "bs58",
			"req" : "^0.2",
			"features" : [ ],
			"optional" : false,
			"default_features" : true,
			"target" : null,
			"kind" : "normal"
		}
	],
	"cksum" : "1ca2105cea234fdf7ca682f66e474abb26bd2bbbae7b5faa58ef2e73b7e4612e",
	"features" : {

	},
	"yanked" : false
}
```

## Interesting queries

This enabled me to perform different queries on the data.

I can find the modules that have `stdweb` as direct dependency as below:

```Shell
> db.crates_data.distinct("name", {deps: {$elemMatch : {name: "stdweb", "kind": "normal"}}}, {"name": 1})
[
        "rand",
        "hobofan_stdweb_logger",
        "bosh_compiler",
        "webgl",
        "webcomponent",
        "screeps-game-api",
        "virtual_view_dom",
        "genact",
        "livesplit-hotkey",
        "cpal",
        "dominator",
        "papito_dom",
        "papito",
        "squark-stdweb",
        "yew"
]
```

Since we see rand being a dependency we can then query with rand in a similar manner to find the dependants of rand that got me around 944 results. So we can get a sense of the breakages that can happen when one module fails to compile that can have an effect on around 10% of the ecosystem. With this I tried to find the top 20 direct dependencies. For this I read all the module names and then queried for the count of the items that had the module name as a dependency. Since we can benefit from indices I created an index on `deps.name` using `db.crates_data.createIndex({"deps.name": 1})`. I then stored the result as a dictionary of module name and the count which got me the result as below :

**Top 20 direct dependencies in Rust**

```JSON
{
 "serde": 2081,
 "libc": 1768,
 "serde_derive": 1542,
 "serde_json": 1488,
 "log": 1431,
 "clap": 1063,
 "hyper": 1025,
 "rand": 944,
 "lazy_static": 892,
 "regex": 753,
 "byteorder": 738,
 "futures": 705,
 "winapi": 680,
 "error-chain": 621,
 "rustc-serialize": 620,
 "url": 614,
 "chrono": 578,
 "time": 533,
 "tokio-core": 446,
 "bitflags": 406,
}
```

We can serde used in a lot of places as a direct dependency which can lead to higher numbers if we take into account transitive dependencies. With around 14000+ modules serde accounts for around 15% of the ecosystem's modules as a direct dependency.

**Python file**

https://gist.github.com/tirkarthi/846f9d2fd87c90463c5ca9cf99f3d44d#file-build-top-py

## Takeaways

It was an interesting exercise that helped me get to know more about the Rust ecosystem. It also helped me estimate how much of the ecosystem can break due to breaking changes with rustc being stable and introducing stabilisation of different features. The team also took swift action in resolving this issue with the [PR](https://github.com/rust-lang/rust/pull/49518). Hope they find a better way to introduce these changes with respect to this feature and other important changes to come in 2018. Thanks to everyone involved in the PR and helpful folks at IRC.

_Closes emacs for dinner_
