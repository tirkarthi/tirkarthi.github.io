---
layout: post
title: London CPython core dev sprint
date:   2019-08-18 00:00:29 +0530
categories: life
---

Python core developer sprint for 2019 happened in London this year. Fortunately, there was some resource to accommodate mentees in the sprint. I was initially reluctant to travel since I have never flew outside India and was little anxious about travel planning, visa, etc. I was encouraged and nominated by mentor Andrew Svetlov to attend the core developer sprint. I applied and also got funding for my travel and stay without which it would be way more expensive. The rest of the post is about my experience in the first sprint.

### Getting to London

I was (un)?fortunate to have British Airways strike happening on September 9, 10 and 27. Since the sprint was from September 9-13 I booked for the only direct flight from Chennai to Heathrow. It got cancelled two weeks before the journey and after calling the customer support for 3 days with 2 hours per day and getting declined 2 of my debit cards I finally got rebooked to September 7th morning. It turned out to be good that I had a day extra now to spare to catch up with my jetlag and also to explore the city by Sunday. Also my visa started on September 7th so it was pretty much last minute.

Given that I was travelling first time I slightly overpacked that I never wore some of the clothes I packed. I took the flight which started by Saturday morning 6 AM and just had breakfast and lunch along with two movies along the way with some sleep here and there. Finally I got to the airport and I have never been to a large airport that needed trains to connect to terminals. I got to the border control.

Officer : Purpose of visit? <br>
Me : Attending a conference <br>
Officer : When are you returning? <br>
Me : By September 14. A week of stay. <br>
Officer : What is the conference about? <br>
Me : It's about the Python programming language <br>
Officer : Python? What is it? <br>
Me : It's an open source programming language. <br>
Officer : Oh, computers. Little brainy stuff there. Good luck! <br>

I got an oyster card and loaded 20 pounds. The journey was great and I managed to connect correctly to get to the hotel. I never knew how it would be to at 15 degrees since in Chennai it's scorching 38-40 degrees. I got to the hotel and along the way I mistakenly bought sparkling water which turned out to be horrible and just got to know the difference between sparkling water and still water. I went to the nearby bridge and castle on Saturday. I enjoyed a kebab for the night.

For Sunday pretty much everyone I checked with were in jetlag mode. So I went alone to St. James park, Buckingham palace and Hyde park. I was little amazed by such large parks open for free throughout the year along with being well maintained. The public transport was also great that I can just swipe and go to different places along with the fact that it's capped and as you know I hit the cap. By evening I went to London eye and enjoyed the evening despite my partial jetlag and it would be sprints next day.

### Sprints

On Monday I got to the Bloomberg office by morning. The office had amazing interiors but little did I know it's too large that I could be lost if I miss following someone at the sprints :) I met Pablo who was organising the sprint along with Mario and also got to be with everyone in the same room. It was my first sprint and being surrounded by experienced contributors was little intimidating. I was primarily planning to work on mock and my other backlog PRs so got to know the mock team.

I met Chris and Mario who were very much helpful in reviewing my mock PRs. Getting to know them in person was great since I have only interacted with them by email. We discussed around mock backport, merging AsyncMock that would mean dropping Python 2 support. There was also Python 2 death march posted just in time during the week about the last release of Python 2 and it's support after 2020. I also got to meet Lisa who wrote AsyncMock. Michael was also at the sprint giving thoughtful guidance around the mock codebase and design. So it was pretty special to have everyone at the same place as I could call the mock team and get to talk in person with them given that we are all separated by at least 3 different timezones. It also led to some design discussions around await and call count that wouldn't have happened if there were no meeting in person and we got to know about working with some real world code.

Unfortunately, Andrew couldn't be at the sprints and Yury was kind enough to mentor me at the sprints. I refined my documentation PR on `asyncio.Stream` to get it merged. I also wrote some initial docs and changelog for `unittest.IsolatedAsyncioTestCase` . During the documentation I also got to know more about the new API which helped me in triaging 4-5 asyncio tickets that could be closed. This also helped me to get to know little more about asyncio and async constructs like async generators etc. Andrew was available online sprinted from home. Both Andrew and Yury were very helpful in providing feedback to PRs. I also got to know about a release blocker bug about early return from async for loop that also got merged in the 3.7/3.8 window.

Some of my other time were spent on getting my old PRs reviewed and some of the minor changes were also merged along with some test case related PRs. Everyone were very helpful and guiding me all along that I am thankful for. I also got to review mock.ANY comparison and kudos to Elizabeth for patiently getting along with the PR in a detailed manner and the initial report got to fix several other cases regarding equality and comparison. The sprint I would say was pretty good in terms of contributions. We also got to review and merge a lot of PRs from other contributors along with discussing the real problem around how to tackle backlog PRs and increasing the review bandwidth given the number of open PRs.

### Faces to voices

One of the difficult parts in getting to know people in real life that you only on the internet is getting to know the real life personality despite online interactions that could paint a different picture. In my case though I write long form posts but I am generally shy to go and initiate conversation let alone in a different country with people from different parts of the world. So trying to maintain your internet persona that is different from real world persona is a little difficult task. 

I was also reminded by mentor that as much as sprints are about coding it's also about talking to people. Despite various accents, countries, timezones etc. I found the whole sprint to be a very welcoming group of diverse people. The general theme around the sprints in my opinion was around to be helpful to each other and to get to know in person. So it was more about putting faces to voices and voices to faces than to be a coding event given that contributions happen before and after the week too. The sprint also helps in breaking the asynchronous nature of the internet for long form discussions like PEPs, designs, ideas etc. There was also an open discussion panel where people got to ask questions and discuss around various things like future of Python, community outlook etc.

I could go on about lot of the events but I will guess I will stop here :) Looking back I am glad I took the opportunity to attend the sprint. I hope to attend the sprint next year though it might be in US which is a 22 hour flight to me :) It also keeps me motivated to ensure I keep contributing back something useful too.

### Getting to Chennai

I could also list down the cultural shifts to me as I got to London. I got to flight early and was just shopping around given the frugal part of me. I also did the worst possible mistake of not sleeping at the plane and went on to watch 5 seasons of Chernobyl consecutively which lead to a Chernobyl in my own sleep cycle for few days.

* I overpacked since it's firs time and will *try* to be more organised for future events which means I bought less chocolate back home :(
* Eating with spoon given that in India I always used to eat by hand. I sometimes ordered very different things. Sometimes I didn't know the difference and ate out of order ;)
* Explaining timezone to my parents and also given the jetlag my dislike towards timezone just increased.
* I ate more chicken and bread ever in one week. So glad to be back to India to get dosa.
* Earning in India and spending in London means everything looks more costly from Indian perspective.
* Sparkling water and still water.
* Forgetting to refund oyster card which I hope to use in a future trip to UK :)

### Thanks

Thanks to Pablo and Mario for organising the sprint and Bloomberg for providing such a great venue along with catering. Thank you very much to PSF for the funding provided without which the sprint would be way more expensive for me in terms of travel and stay. The cost comes to something along the lines of INR 150-200k for travel and stay of one week in London which would equate to 10 months of living expense in India. Thank you Ewa for patiently helping me along the way even at the last minute for my accommodation due to the British Airways strike. 

It would be a special event for me given the amount of things I got to do first time on my own. So I am very much thankful and grateful to meet you all in person.

_Switches tab to see PGO build results_