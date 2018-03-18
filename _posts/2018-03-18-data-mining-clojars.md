---
layout: post
title:  Data mining clojars.org for issues
date:   2018-03-18 00:00:29 +0530
categories: clojure
---

> I do a lot of work on open source, but my most valuable contributions haven’t been code. Writing a patch is the easiest part of open source. The truly hard stuff is all of the rest: bug trackers, mailing lists, documentation, and other management tasks - Steve Klabnik, How to be an open source gardener.

The above [quote](http://words.steveklabnik.com/how-to-be-an-open-source-gardener) by Steve Klabnik who is a prolific open source contributor and currently involved with Rust emphasises the importance of bug filing, docs, etc. I am learning Clojure for the past one year and I thought making open source contributions is a great way to interact with the community. This is a post on using clojars POM files metadata for the modules to determine modules that won't run on Clojure 1.9 and JDK 9 to file issues.

## Clojure 1.9 and JDK 9 adoption

The Clojure community has greater adoption rate on using the latest Clojure version and JDK version as I can see from the previous [State of Clojure](http://blog.cognitect.com/blog/2017/1/31/state-of-clojure-2016-results) surveys. As of the 2016 survey 83% were using Clojure 1.8 and around a third were using Clojure 1.9 pre-releases. Along with that over 95% of the users were on JDK 8 when JDK 9 was about to be released. That proves stability of Clojure and JVM along with the users eager to try out latest versions instead of sticking to older versions for legacy reasons. With the above in mind there were also some issues in JDK 9 and Clojure 1.9 releases that affected the users due to deprecation and breaking changes. Even Clojure had [problems](https://dev.clojure.org/jira/browse/CLJ-2077) running on JDK 9 but they were fixed immediately.

### sun.boot.class.path and java.ext.dirs removal

The first issue I came across was with using auto-complete on Leiningen on JDK 9. Leiningen uses reply which in turn uses clojure-complete to provide completions as you type along. But there was a strange issue reported that when you type `clojure.co` and press TAB for compeletion it will throw a NullPointerException. The first difficulty was in reproducing the issue but since the user tested both JDK 8 and JDK 9 we were able to track it down to JDK 9. From the stacktrace we can get to the line where NPE is thrown. The issue was due to the removal of `sun.boot.class.path` and `java.ext.dirs` on JDK 9 that returned nil but worked for JDK 8. So the fix was to filter for nil so that they worked on both JDK 8 and JDK 9.

Relevant PR : https://github.com/ninjudd/clojure-complete/pull/25

Since this is a pretty big removal I am sure some other libraries might also suffer from the same problem. So I used GitHub's search and came across [compliment](https://github.com/alexander-yakushev/compliment/blob/0f6d907b6b2c7f0d45d0763f84e676676bb8bed6/src/compliment/utils.clj#L98-L99) which used the same path but there were checks for nil. This is a good thing regarding these type of fixes that when you fix at one place a simple code search will also give you other modules to incorporate the issues. clojure-complete was not maintained but the author was kind enough to give contributor access to merge this and the author of reply library also bumped the version since he had access to the relevant Clojars group thus fixing them in reply and Leiningen.

### Breakage due to add-on modules

In JDK 9 some of the classes were moved out as separate modules that caused an issue and the fix was to use `--add-modules` jvm-options in project.clj . One instance of this was the usage of `java.activation` which caused `ClassNotFoundException` under JDK 9 but worked under JDK 8. It caused an [issue](https://github.com/dotemacs/pdfboxing/issues/41) in pdfboxing library and the reporter was kind enough to check on different JDK versions to pin down the actual issue. Since the project was a wrapper around an Apache project the fix was to remove the incompatible dependency to provide support. I used GitHub code search to determine other places and raised issues on the respective repos.

Relevant PR : https://github.com/dotemacs/pdfboxing/issues/41

Since this was a common problem with JDK 9 I was happy to find [clojure-java-9](https://github.com/tobias/clojure-java-9) where the issues are tracked and filed the above issues there too. But a common trend I realised among these projects was that they were not testing it against JDK 9 which might have reproduced the issues beforehand. So along with the fixes I added JDK 9 as a target in the CI so that there were no regressions.

### Clojure 1.9 refer-clojure

With the introduction of spec Clojure 1.9 enforced a breaking change on the syntax of `refer-clojure` where some of the projects were not able to run on Clojure 1.9 with `core.async` having a high usage breaking a lot of things. Fortunately, this was during Clojure 1.9 alpha releases and the relevant fix was immediately made against `core.async` and other modules also upgraded that helped in resolution. It also indicates the amount of people trying out alpha releases that helped in prompt reporting. A quick google search with [refer-clojure github issues](https://www.google.com/search?q=refer-clojure+github+issues&ie=utf-8&oe=utf-8&client=firefox-b-ab) returned below :

* [lein-igwheel](https://github.com/bhauman/lein-figwheel/issues/540)
* [reagent](https://github.com/reagent-project/reagent/issues/331)
* [pedestal](https://github.com/pedestal/pedestal/issues/505)
* [yada](https://github.com/juxt/yada/issues/156)
* [cursive-ide](https://github.com/cursive-ide/cursive/issues/714)
* [midje](https://github.com/marick/Midje/issues/369)

They were promptly fixed but still I can see a lot of modules that wouldn't run on Clojure 1.9 when users try to run them. In this case GitHub code search will not be helpful and I resorted to using data from Clojars to fix this issue.

### Data mining Clojars

Clojars is one of the primary places for Clojure ecosystem to host modules and they also had a [note](https://github.com/clojars/clojars-web/wiki/Data#rsync-the-whole-repository) on wiki to rsync the whole data of clojars. But downloading the whole data will blow my bandwidth and space since I was trying it on a tiny DigitalOcean droplet. I asked around for help in Clojars slack channel and came with an rsync command to download only the POM file for all the modules which has the direct dependency information. It took around 1 GB of data which was fair enough. The rsync output is as below :

```
$ rsync --stats -zarv --delete clojars.org::clojars my-wonderful-copy-of-clojars --include="*/" --include="*.pom" --exclude="*"
# A lot of output
Number of files: 293,110 (reg: 153,658, dir: 139,452)
Number of created files: 280,422 (reg: 151,933, dir: 128,489)
Number of deleted files: 0
Number of regular files transferred: 151,933
Total file size: 549,866,331 bytes
Total transferred file size: 544,144,707 bytes
Literal data: 544,144,707 bytes
Matched data: 0 bytes
File list size: 9,557,679
File list generation time: 0.785 seconds
File list transfer time: 0.000 seconds
Total bytes sent: 3,478,606
Total bytes received: 160,580,467

sent 3,478,606 bytes  received 160,580,467 bytes  508,710.30 bytes/sec
total size is 549,866,331  speedup is 3.35
```

This lacks the full dependency tree of transitive dependencies. For that we need to run `mvn -f /path/to/pom dependency:tree` which will print the full dependency tree like `lein deps :tree` but it had the unfortunate side-effect of downloading the jars which was not good. So I resorted back to using only the direct dependencies. I wrote a simple POM parser to iterate through the whole directory and print the POM file as a JSON to insert them into MongoDB. The sample output for audiogum/clj-lazy-json POM file is as below :

**POM file**

```
<?xml version="1.0" encoding="UTF-8"?><project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>audiogum</groupId>
  <artifactId>clj-lazy-json</artifactId>
  <packaging>jar</packaging>
  <version>0.0.3</version>
  <name>clj-lazy-json</name>
  <description>Jackson-based lazy JSON parsing library for Clojure.</description>
  <scm>
    <url>https://github.com/audiogum-forks/clj-lazy-json</url>
    <connection>scm:git:git://github.com/audiogum-forks/clj-lazy-json.git</connection>
    <developerConnection>scm:git:ssh://git@github.com/audiogum-forks/clj-lazy-json.git</developerConnection>
    <tag>eb8f945b57b1a16d2601ae7d568582e00f068d19</tag>
  </scm>
  <build>
    <sourceDirectory>src</sourceDirectory>
    <testSourceDirectory>test</testSourceDirectory>
    <resources>
      <resource>
        <directory>resources</directory>
      </resource>
    </resources>
    <testResources>
      <testResource>
        <directory>resources</directory>
      </testResource>
    </testResources>
    <directory>target</directory>
    <outputDirectory>target/classes</outputDirectory>
    <plugins/>
  </build>
  <repositories>
    <repository>
      <id>central</id>
      <url>https://repo1.maven.org/maven2/</url>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
      <releases>
        <enabled>true</enabled>
      </releases>
    </repository>
    <repository>
      <id>clojars</id>
      <url>https://repo.clojars.org/</url>
      <snapshots>
        <enabled>true</enabled>
      </snapshots>
      <releases>
        <enabled>true</enabled>
      </releases>
    </repository>
  </repositories>
  <dependencyManagement>
    <dependencies/>
  </dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.clojure</groupId>
      <artifactId>clojure</artifactId>
      <version>1.9.0-RC2</version>
    </dependency>
    <dependency>
      <groupId>org.codehaus.jackson</groupId>
      <artifactId>jackson-core-asl</artifactId>
      <version>1.8.6</version>
    </dependency>
  </dependencies>
</project>

<!-- This file was autogenerated by Leiningen.
  Please do not edit it directly; instead edit project.clj and regenerate it.
  It should not be considered canonical data. For more information see
  https://github.com/technomancy/leiningen -->
```

**JSON**

```
[
  {
    "modelVersion": "4.0.0",
    "groupId": "audiogum",
    "artifactId": "clj-lazy-json",
    "packaging": "jar",
    "version": "0.0.3",
    "name": "clj-lazy-json",
    "description": "Jackson-based lazy JSON parsing library for Clojure.",
    "scm": {
      "tag": "url",
      "attrs": null,
      "content": [
        "https://github.com/audiogum-forks/clj-lazy-json"
      ]
    },
    "dependencies": [
      {
        "groupId": "org.clojure",
        "artifactId": "clojure",
        "version": "1.9.0-RC2"
      },
      {
        "groupId": "org.codehaus.jackson",
        "artifactId": "jackson-core-asl",
        "version": "1.8.6"
      }
    ]
  }
]
```

This gave us the info about the repo where we can find the code which for the most part is a GitHub repo and the direct dependencies parsed from the file. With the above part it's just a matter of inserting the data into MongoDB to query them for older dependencies. The first query was to get the projects that used older version of  `core.async` i.e. below 0.3.442 to get the list that won't run on Clojure 1.9 which can be done as below :

```
$ mongo
> use lein
switched to db lein
> db.clojars.distinct("url", {dependencies: {$elemMatch : {artifactId: "core.async"}}, "version": {$lt: "0.3.442"}}, {url: 1, '_id': false})
[
	"http://github.com/pleasetrythisathome/tao",
	"http://dsteurer.org",
	"https://github.com/cncommerce/beetlejuice",
	"https://github.com/tgetgood/lemonade",
	"http://example.com/FIXME",
	"https://github.com/luchiniatwork/contentql",
	"http://github.com/unbounce/hystrix-event-stream-clj",
	"https://github.com/ekimber/shiroko",
	"https://github.com/wkf/hawk",
# A lot of data removed
# See gist : https://gist.github.com/tirkarthi/dc4cef81dd7242fa4d3539c854491c5e
]
```

With `core.async` there were also projects like [dynapath](https://github.com/tobias/dynapath/issues/5) which in turn is used by [alembic](https://github.com/pallet/alembic/pull/16) (PR pending) that are very popular in the Clojure ecosystem that won't run on Clojure 1.9 due to direct dependency or as transitive dependency

### Issues in the approach

With the above, some of the common issues with the approach due to which we couldn't automatically raise the issues are as below :

* GitHub bans automatic reporting of issues for some reason. Relevant [HN discussion](https://news.ycombinator.com/item?id=16544361)
* The project URL being invalid or repo being deleted is a common issue in automation.
* There are cases where the issue is fixed in the master or a PR is already made which would make raising issues irrelevant.
* I need to modify the script to make sure only the latest POM file is tracked instead of using all POM files that will lead to false positives of the issue being fixed in latest version.
* Some of the projects are used for learning purposes and unmaintained forks which were published to Clojars that won't have much consideration on Clojure 1.9 or JDK 9 compatibility.
* Sometimes the projects require older version for legacy reasons or the upgrade is not very simple and don't want to support Clojure 1.9 or JDK 9.
* There is no central repo that tracks the modules Clojure 1.9 issues which is sort of a chicken and egg problem. I used `core.async` since I came across projects that had issues thus enabling me to query. But there were some projects that are yet to be run on JDK 9 or Clojure 1.9 to get the problems and they are also not tracked anywhere so that we can fix other places. I have created a [repo](https://github.com/tirkarthi/clojure-jdk-issues) where I encourage you to file issues.
* Sometimes the project uses a lower version where the issue was not present like the problematic syntax being introduced in 0.1.0 and the project uses 0.0.1 which will not cause issues in the first place.
* The obvious one is that I need to file issues after checking the project.clj manually which was little boring.


### Takeaways from the solution

With the above issues in mind I also came across some takeaways to the solution as below :

* Test the project in JDK 9. Since most of the projects use Travis CI adding a single line of `oraclejdk9` (openjdk9 is not present for some reason) is a great way to catch some issues. I raised some issues to repos where adding oraclejdk9 to .travis.yml uncovered some issues.
* Test with latest JDK consistently. With Java moving to six months release cycle and LTS release of three years there are already a lot of deprecations in latest JDK which might break your project so test against latest JDK once it's in to ensure compatibility.
* Some of the projects are not keen on upgrading the dependencies since they might not have the time to upgrade or maintain the project which is totally acceptable. Since it's open source you can always have a fork where you can tackle the issue and contribute.
* Clojure has a rock solid stability which helps projects to directly upgrade to the latest version and even with folks running alpha versions on production. But consistently testing the ecosystem on latest Clojure and JDK will be helpful too. For example the Perl project smoke tests the entire CPAN for each and every commit to Perl's core on different configurations and reports back the breakages through email to the developers. Though this is not a viable solution and given the commit rate to Clojure repo it's a worthwhile option too.
* I would like to build a web UI where these could serve as good first issues to people who are trying to contribute to open source for the first time and constantly syncing with Clojars to stay updated.

### JDK 10 issues

With JDK 10 in the release candidate stage there are some removals in JDK 10 from the [release notes](http://jdk.java.net/10/release-notes) that might  also cause some issues. There is also a change in [release plan](https://mreinhold.org/blog/forward-faster) which means latest JDK is needed in travis and other CI systems. There are some [issues](https://github.com/junit-team/junit5/issues/1169) fixed already with people trying out JDK 10 early access builds like the Kotlin compiler broke on JDK 10 EA which was fixed.

### Final thoughts

So with the above exercise I managed to manually file around 20 issues and PRs regarding upgrading `core.async`, adding JDK 9 to `.travis.yml` and trying simple module upgrades. I could do more and as I said a repo of the info regarding Clojure 1.9 issues, module versions and info will help. The experience was positive and the folks were helpful in PR and took issues on a positive note.

Thanks again to the module maintainers and helpful folks at #clojars slack channel.

* GitHub repo for POM parsing code : https://github.com/tirkarthi/pom-parse-deps
* GitHub repo for Clojure and JDK issues : https://github.com/tirkarthi/clojure-jdk-issues

Please comment if I am missing something on the above.

_Closes emacs for lunch_
