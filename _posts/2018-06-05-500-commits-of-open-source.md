---
layout: post
title:  500 commits of Open Source
date:   2018-06-05 00:00:29 +0530
categories: programming
---

_Note : 500 commits of Open Source is a pun on 500 days of summer. This is a post on my past 3 months of open source work, the right and most importantly wrong things. I wrote this as an introspective post and my experience in the experiment._

I have always been a big fan of open source and I am highly indebted to the stuff that I get to work on which are fully open source along with being a significant part of my career. So I thought to contribute back and enrich my experience with the community and the past three months have been excellent since I have done around 500 contributions in the three months which is more than my whole four years of career combined. I also learned more about interacting with the community. The post\* is split into three parts : 

* [Backstory](#backstory)
* [The good parts](#good)
* [The not so good parts](#bad)

\* If you find my writing boring do skip to the [resources](#resources) where I have some really good picks of some great open source maintainers about open source.

First, I will start of the positive things (Hey! I am sticking to New Year resolutions :)

## [Backstory](#backstory)

### First bug

2018 started out to be slow with respect to open source. Most of the fixes I made were fixes to CI configurations which were outdated, typo fixes and so on. I have been learning Clojure these days and I thought to look into some of the GitHub issues of popular libraries to see if I can contribute anything. I use [Leiningen](https://github.com/technomancy/leiningen) a lot and hence I looked into the [issues](https://github.com/technomancy/leiningen/issues). I stumbled upon a [NullPointerException](https://github.com/technomancy/leiningen/issues/2380) when you try to autocomplete on a particular case. The first thing was to reproduce the issue which I couldn't and hence commented on the issue. David Jarvis who created the issue replied back that it might be a specific issue related to Java 9 since I was on JDK 8 meanwhile the original issue was on JDK 9.

I tried it with JDK 9 and the issue was reproducible. It gave me a stack trace which told me it's an upstream issue. I looked into the auto-completion library code and most importantly I was looking into test to write a reproducible test for this. I wrote a test and it was caused due to some of the classloading changes made. The [fix](https://github.com/ninjudd/clojure-complete/pull/25) was a simple filter function to check for `nil` among the results. I wrote a test case and also added JDK 9 to Travis to make sure it's tested.

Unfortunately, Justin Balthrop, the author of the library was not maintaining the library and hence asked either of us to merge this as he adds us a contributor. I was still a newbie and I declined it. David was kind enough to merge this but I still couldn't get it released since we lost contact with the author. Fortunately, the author had added Colin Jones, author of reply which was also used in Leiningen added as a contributor who was also kind enough to release. This was then bumped in reply and then bumped in Leiningen to [fix](https://github.com/technomancy/leiningen/pull/2398) the issue.

### First PR

I was always looking into writing a small module to learn more about release process, docs, issues and so on. I wrote my first module this year [numcompress](https://github.com/tirkarthi/numcompress) which is a Clojure port of a Python library with same name. For sometime I was also involved in security work in my company and hence I was looking into SSRF attacks which led me to write a simple [middleware](https://github.com/tirkarthi/clj-http-ssrf) for clj-http which I released back as a second module. 

I received my [first PR](https://github.com/tirkarthi/clj-http-ssrf/pull/4) from [Ackerley Tng](https://github.com/ackerleytng) for the module along with a couple of issues. It was an awesome feeling like somebody uses that? He also added as my module as a dependency to his project which felt good too since being a dependent on a project is indeed a proud moment. This gave me a good sense of happiness along with interacting with him in merging it and making releases for the same.

## [good](#good)

### How to contribute

> I do a lot of work on open source, but my most valuable contributions haven’t been code. Writing a patch is the easiest part of open source. The truly hard stuff is all of the rest: bug trackers, mailing lists, documentation, and other management tasks. Here’s some things I’ve learned along the way. - Steve Klabnik, [How to be an open source gardener](http://words.steveklabnik.com/how-to-be-an-open-source-gardener)

Contributions come in all sizes and as said above there are always boring but important parts like testing, documentation, proof-reading, triaging etc where you can work on to make the project better. Every form of improvement is an improvement and most projects welcome contributions of all sorts.

A simple way I have found to contribute is to contribute to the tools you use daily. It gives some motivation to work on stuff that you use daily where you can see the improvement and also find your time more valuable like Leiningen in this case. The other thing I have found to be a good way to find issues is that if there are any large changes involved then there will be other places to fix which you can figure out using GitHub search. In my case I fixed a couple of issues involving JDK 9 and I used GitHub search to look for similar issues and fix them back.

Another good way is to do some data mining project depending upon the ecosystem. I had Clojure ecosystem metadata which I used to find incompatible dependencies and then make relevant fixes. Once you get the approach you can use the same in other areas where I did the same thing with Rust ecosystem to file back issues and PRs for the same.

### Be kind

One of the important things the Leiningen issue has taught me is to respect each others time and effort even more. We need to take others efforts into account instead of the adding the burden of maintenance and entitlement for releasing something. Sometimes I am more eager to get my code released but I think I handled it well and I am happy to learn more about each others time.

This also taught me a significant part about software that it's more about people than code that helps in contribution and the general outlook of the community. Along with people factor comes the [bus factor](https://en.wikipedia.org/wiki/Bus_factor) where a bug might not be fixed due to the author being not available. There can be a fork which has the fix but there are also other aspects like technical debt apart from code itself which could be avoided.

A big part of a healthy community is in being kind since we interact with people of varied culture, timezone, language etc. Do say thanks more and be open to accepting your mistakes and moving on with your day enriching yourself.

### Impact and intrinsic motivation

After the fix was done it gave me a good sense of motivation in seeing my code being used by thousands of developers though it's small it gives me confidence over how I can make an impact with code. On a similar note it also motivates me more to work on stuff that matters since with code there is always stuff to improve. I think it's a big factor that keeps people going and hence if you use some big software do take sometime to write a note of thanks to the developer and it makes each others day a lot better in terms the value added in day-to-day life with code.

Thought impact plays a large part don't be afraid of writing something for your own fun time to time. People may find the ideas silly but as long as you are happy and makes your life better it doesn't really matter whether the code is impactful. It has helped me in writing very silly reddit bots, crawlers, data visualisations etc.

### Interact more

Well this might seem to be an odd advice from a person like me but I find interaction with other people to be really helpful. I write which I should be doing even more. I try to answer forums and at least I google the error message to point them to some source. I also interact a lot on Reddit and other places where I feel expressing my thoughts on writing helps me prioritise more on what I clearly want to express.

## [NotSoGood](#bad)

As with any good activity there are also stuff to improve on like code and these are some of the bad things I did

### Freaking out on GitHub streaks

A lot of times I might get a week or so of streaking going only to find some other activity where I need to prioritise more like travelling home, going out with friends or reading a book or so on. Initially I was laying too much emphasis on GitHub streaks that led me to do stuff like intentionally opening up an issue or making some small commit or spreading my commits over a series of days or so on. TBH, just cheating to myself over an insignificant metric that no one sees at all. I am not against streaks but it's just that I had a bad approach towards it.

This also means that unless you have a day job on open source or involved with a big project there will be days where you can't really commit back anything useful which is **absolutely fine**. We can't generate ideas everyday and it only leads to unwanted stress and squeezes out the fun part of things since I need to remind myself I am doing it out of interest and there is nothing entitled upon me to do this at all.

### Communication through the globe

When you are involved with a big project people will always prioritise your PR over some other feature. The only medium of communication you really have is email and issue tracker. Hence being kind really helps everybody. People generally argue over issue tracker and trolling the author though they are not willing to fork and fix it themselves but in turn resort to these stuff to get unwanted attention.

We also need to keep in mind we are interacting with people of various cultures, timezones, languages, ideologies and so on which we should have empathy for but also be ready to hit the unsubscribe button when you don't really find the tone of the conversation helpful enough. It's there for a reason.

### Take recreational breaks

One of the things that people generally tend to view people who do a lot of free time programming is that they are workaholics. I find this odd that since people are happy doing whatever they want why do you bother them with these opinions. Well, it's both a curse and a blessing that you are paid for stuff you do on free time but don't let others opinions really creep you out. As long as you are happy doing whatever you want it doesn't really matter.

In the same way if you are working for a lot of days and then feel like taking a recreational break to do stuff like reading a book or so on to rejuvenate your creativity that's fine too. Don't let others opinion that you are slacking off responsibility and entitlement since you have released something or your contributions are not so green. To be clear **stop giving more fucks about everything.**

### Compare with yourself not with others

There will always be a feeling that people are contributing a lot and are working on stuff like compilers, OS and block-chain (BWAHAHAA). The key is to take inspiration and follow a plan with discipline comparing your own code with your past self instead of looking into others who also started out on the same way as a beginner. I always look at my own code and I find if I am not hating the code I wrote six months (which I do) to refactor it better than I haven't really improved with respect to that aspect. As an example here is [some code]((https://github.com/tirkarthi/Wordzilla/blob/master/src/com/Yennaachi/Wordzilla/MainActivity.java#L141)) I wrote 4 years back that you have to scroll horizontaly three times to view the end : 

```java
if((enter_length!=0) && (enter_str.charAt(0)!=' ') && (!Character.isDigit(enter_str.charAt(0))))
		{	
			char you_test= enter_str.charAt(0);
			enter_str = enter_str.toLowerCase();
			if(i==1||you_test==wordzilla_last)
			{
				try
				{
					Cursor c = words.rawQuery("select * from "+you_test+" where word = '"+enter_str+"';",null);
					try
					{
						if(c.moveToNext())
						{
							c.moveToFirst();
							String enter_txt = c.getString(c.getColumnIndex("word"));
							int flag_int = c.getInt(c.getColumnIndex("id"));
							if(flag_int==0)
							{
								//you
								int word_zilla = enter_str.length();
								final char wordzilla_test = enter_txt.charAt(--word_zilla);
								words.execSQL("UPDATE "+you_test+" SET id='"+1+"'WHERE word='"+enter_str+"'");
								c.close();
								words.close();
								new LongOperation().execute("");
								if(enter_length>9)
								{
									if(enter_length>=14)
									{
										Toast.makeText(getApplicationContext(), "Congrats! Mega Bonus : " + enter_length*100 , Toast.LENGTH_LONG).show();
										score_bonus+=enter_length*100;
									}
									else
									{	
										Toast.makeText(getApplicationContext(), "Bonus : " + enter_length*10 , Toast.LENGTH_LONG).show();
										score_bonus+=enter_length*10;
									}
								}
								//score
								present_score += score_in + score_bonus;
								Runnable runnable = new Runnable() {
									@Override
									public void run() {
										for(int i=present_score-score_in;i<=present_score;i++)
										{
											final int value = i;
											try
{

```

**Code that I wrote [2 weeks](https://github.com/tirkarthi/clj-foundationdb/blob/8a32136282d3c57ef0798fde2b8fedd98ee5c64c/src/clj_foundationdb/core.clj#L159) back**

```clojure
(defn get-range-startswith
  "Get a range of key values as a vector that starts with prefix
  (let [fd     (select-api-version 510)
        prefix \"f\"]
  (with-open [db (open fd)]
     (tr! db (get-range-startswith tr key prefix))))
  "
  [tr prefix]
  (let [prefix      (key->packed-tuple prefix)
        range-query (Range/startsWith prefix)]
    (->> (.getRange tr range-query)
         range->kv)))

(spec/fdef get-range-startswith
:args (spec/cat :tr tr? :prefix serializable?))

```

## Conclusion

Well that brings me to the end of the rant. As I have said there are a lot of low-hanging fruits to improve upon but I found this experiment to be interesting and I hope it was useful to you as well. I always loved contributing back to open source and I feel I have done a good job along with establishing it as a habit. So here's to the next 500 commits and more to life.

*closes the GitHub issue opened as a reminder to write this post*


### [Resources](#resources)

* [Getting Started in Open Source](https://www.kennethreitz.org/essays/getting-started-in-open-source) - Kenneth Reitz
* [Growing Open Source Seeds](https://www.kennethreitz.org/essays/growing-open-source-seeds) - Kenneth Reitz
* [How to be an open source gardener](http://words.steveklabnik.com/how-to-be-an-open-source-gardener) - Steve Klabnik
* [The Minimally-nice Open Source Software Maintainer](http://brson.github.io/2017/04/05/minimally-nice-maintainer) - Brian Anderson
* [Sindre Sorhus AMA](https://blog.sindresorhus.com/answering-anything-678ce5623798) - Sindre Sorhus
* [Sindre Sorhus Interview](https://medium.freecodecamp.org/sindre-sorhus-8426c0ed785d) - Sinder Sorhus

PS : For some reason Jekyll and linking to headings seem to be a tedious process or I am missing something

