---
layout: post
title: Finding balance in open source
date:   2019-11-10 00:00:29 +0530
categories: life
---

"There is a difference between being obsessed and being motivated." - The Social Network

### Open source project beginnings

I always wanted to contribute to a large open source project both for fun and the experience of having your code running code on lot of machines with guidance from smart people along the way. So I was little sad during 2017 and just wanted to start writing some code as a distraction to the other problems I had. So I started writing Clojure and created some modules on my own along with filing bug reports on other projects to help compatibility towards Clojure 1.9 that had some changes due to spec. It was fun and I would write blogs where there would be spikes in my Google Analytics page as I post to Reddit. It was still not as exciting as contributing to a large project itself as you eventually run out of ideas to start new projects or lose interest in maintaining existing ones.

So I started looking into larger projects and found fixing documentation typos to be an easy way to learn the workflow. I started with git but found the workflow to be little difficult since it involves sending patches by email. Since I was using Python I wanted to give it a try and CPython, the reference implementation was also moving to git and GitHub. I downloaded the source code and ran aspell across the Docs folder. I found some typos and pushed a fix. I got some review comments over some are not false positive to revert them to get the change eventually merged. I learned more about where to create issues, ask questions and the PR guidelines. Later I found easy issues to be a good way and then tackled an easy locale related aliasing that spawned a discussion on its own to be later closed as not a bug.

### Triaging issues

I was always inspired by Steve Klabnik especially with his work on Rust. I came across his [post](https://words.steveklabnik.com/how-to-be-an-open-source-gardener) about triaging rails issues. So I sat down on an August weekend to filter all the issues with no reply and gave them a read trying to reproduce the report on master. Writing a unittest. Asking for more information. As I read through I got familiar with the relevant people. An intuition over if the issue was already reported, a patch in related issue solves this etc. I read a lot of open issues that helped me with new issues being filed and also with learning over how to communicate since I have never been involved with the volume of activity.

Later I found that there is a separate mailing list called python-bugs that sends me an email for every activity on the bug tracker. python-checkins sends me an email for every commit merged to one of the branches. I subscribed to them and setup a filter on gmail. Usually I aimed at inbox zero to read them fully though I don't understand and have any actionable item for the email. I was later nominated for triager status in a month and got by triager bit by September. So it was a happy moment for me as I never really got anything like that appreciating my work which led me to spend even more time.

### Keeping up with issues

As I kept track of all emails they also started moving up and up. On an average the python-bugs list generates around 2000-2500 emails. This will be high when there are events like sprint and conferences with hackathons. As I got bug reports on my inbox I started fixing them or debugging/bisecting them in case it was working fine to add the relevant commit. I found this to be a good way to keep engaged and over a period of time my sadness was gone or I learnt it was not as big I thought it would be. Then triaging became a habit of mine to spend time daily to go through bugs and scroll through PRs. I watched GitHub repo for sometime but the volume was even more that I left it off.

This meant that almost all the time I would be on email if out of work. I go to my hometown twice a month and started carrying my laptop around. Sometimes I would read something during a weekend dinner and can't wait to get on to the laptop to add the comment. This seemed like the honeymoon phase where everything seemed to be very exciting that I didn't even realize my pattern.

### Tackling more issues

I used to find issues fix them but over a period of time I wanted to focus on one module or perhaps get familiar with it solving issues in that. At that time I had some trivial patches to mock. I started looking into the existing issues and subscribed myself to almost all issues. Though I am not an expert in the codebase I had some understanding about the problem. Also since it was a pure Python module using lot of advanced tricks in Python's runtime I got to know more about it and mock seemed to have maximum utilization of dynamic nature of Python to patch and mock calls.

Along with this whenever I got a good bug in a module involving fair amount of debugging I also searched for other related issues in the module to see if I can help there. In that way I fixed a [security issue](https://bugs.python.org/issue35121) over domain name access check. Through the course of time I got to read more implementations of cookiejar in other languages and also fixed [another](https://bugs.python.org/issue35647) security issue. It was the first time I got to fix a security issue and I was more happy that it kept me going further.

### Strange patterns

One of the things with open source is that you work with lot of other people who are from different countries which means different timezones. This asynchronous nature is a blessing and a curse. Since I am from IST communicating with other from Europe or US means I need to wait for their reply and also for them to wait for me. So I started to cut down response times without knowing by replying to email as quick as possible whenever possible. Since I sleep late night this meant there could be some email at midnight that I want to reply to and spend few extra minutes. Those extra minutes add up especially on the weekends and I noticed myself replying very late in the night. Sometimes I wake up for water and there was an urge to check email and there would be an email that I would like to reply.

As I kept myself adding to issues as they kept coming in and I didn't realize the pattern where I am ruining my own sleep schedule. This seemed fun in the short term but is prone to cause problems in the long term given that I can't really sustain myself. But I kept going without realizing that as much as the other person prioritizes their time I need to do mine instead of going with the urge that the person on the other end is waiting on me.

### Job search and open source

I was also on job search which means programming assignments and interview preparations during which I need to find time for open source work. Job interviews are not as interesting as working on a bug since you don't know if you will get the job and some of them had leetcode style questions that I could use for my own open source work. In fact I wish someone gave me an open source bug that I could help in resolving to showcase my abilities as an engineer thought I never got one. But then I need to feed myself and I need to do the assignments even if I don't like them.

In the due course it means my interaction went down and then emails kept being unread that sometimes I mark all as read just to beat down my own feeling of guilt. Luckily I found a job through a friend of mine in a couple of months and also I got nominated by my mentor to attend the core dev sprint to work on CPython for a week. It was a great opportunity to meet everyone in person. It also came during my notice period which my previous employer was kind enough to have me attend the sprint. This also meant that I have to transition to a new job in a new city. So over the time I practically had no time and also noticed that I started caring more about balance of all this.

### Balance and unpaid work

So today I sat down to see where it all stands. I found myself subscribed to 970 issues in CPython. I wrote a little script to scrap all the emails to plot my activity and of course I wrote it in Python :) python-bugs list generates 2000-2500 emails per month and downloading archives I ran grep to see that I generated 2066 emails apart from the pull request reviews. That's almost a month of activity generated by me over the course of 1 year and 5 months in contributing. Even if I take at least 5 minutes to compose and send an email that comes to around 10000 minutes (165 hours) apart from writing code itself. That's almost a month of unpaid time working 8 hours per day.

```
$ zgrep -i "From: report at bugs.python.org (Karthikeyan Singaravelan)" *gz | wc -l
2066
```

I also plotted down the time of emails sent and noticed that as I reach home by 6 PM almost all of the top time slots are from 6 PM to 12 AM doing open source work with sending 170 emails around 6 PM.

![Emails per hour](/images/emails_per_hour.png)

Script : https://gist.github.com/tirkarthi/3a5f968d9442d62f12c0f43667caa314

It is little more depressing to get this quantified to major contributors who are unpaid scaling it across the open source tools I use everyday. Time is also available in ample amount as a 26 year old with being single and no family responsibilities. As much as this is all done for fun as compensation and time as cost I should also keep in mind that about the balance in life. That even if it's not a problem currently I can't sustain myself over longer period of time and with things like family in future. This also poses the question that if I put all my happiness egg in open source and if there is a point where I lose interest what do I get to do?

This brings me to the quote that there is a difference between being obsessed and being motivated which is slightly thin and blurry to me that I didn't realize it. Right now I am still contributing although slightly less active than before and don't get urges as often to respond to email and issues. I get this as a good learning experience to tweak some parameters in my life so that I get to find a strategy and appreciate balance to keep loving what I love to do :)

P.S. This is more of something that I wanted to write as open as possible backed by numbers that I am responsible for so that if there is someone on a similar path then they can learn from me :)

_Orders lunch from Swiggy_