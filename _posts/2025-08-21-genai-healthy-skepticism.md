---
layout: post
classes: wide
title:  Generative AI and healthy skepticism
date:   2025-08-21 00:00:29 +0530
categories: programming
---

Generative AI ecosystem is a constantly evolving technology with a heavy push across the society and especially in software engineering to adapt it and empower themselves with increased productivity. It's a constant source of news with quotes and headlines from executives, engineers, vibe coding enthusiasts etc. about how people who don't adapt it won't survive the cycle. In this post it will be about taking a simple

### Migrating airflow.cfg from Airflow 2 to Airflow 3

I use Airflow at work and also a committer on the Apache Airflow team. Airflow 2 was released on 2020 and the major version Airflow 3 was released on April 2025. Given that Airflow is written in Python and the general experience towards Python 2 to 3 migration it was planned to make the migration as smooth as possible with a tool similar to 2to3. ruff has become a standard tool in the Python ecosystem and was enhanced to automate Airflow 2 to 3 changes to DAGs. One major aspect of the migration was towards cleanup of deprecated configurations from Airflow 2 that had to be removed/renamed in Airflow 3. This felt like a good use case for Generative AI since the Airflow configuration is an ini file where options were moved or deleted across sections. It's a case of taking Airflow 2 cfg and translating it to Airflow 3 with objective rules for the transformation. I tried doing it with prompts and the experience is as below.

I started with the initial prompt of "Please migrate airflow.cfg from Airflow 2 to Airflow 3". The models suggested using "airflow config generate" and "airflow config export" to take a backup of the airflow config. On trying `airflow config generate` and `airflow config export` I was greeted with the message that these commands don't even exist. I was under the impression that maybe these commands themselves were removed in Airflow 3. On a search of git history with `git log -G'airflow config export'` and `git log -G'airflow config generate'` I couldn't find any mentions of these commands.

```shell
airflow config generate
Usage: airflow config [-h] COMMAND ...

View configuration

Positional Arguments:
  COMMAND
    get-value
              Print the value of the configuration
    lint      lint options for the configuration changes while migrating from Airflow 2.x to Airflow 3.0
    list      List options for the configuration
    update    update options for the configuration changes while migrating from Airflow 2.x to Airflow 3.0

Options:
  -h, --help  show this help message and exit

airflow config command error: argument COMMAND: invalid choice: 'generate' (choose from 'get-value', 'lint', 'list', 'update'), see help above.
```

Then the models started writing an extensive 500 line script to parse the cfg file using configparser module and then to rename the options/sections. The script was wrapped with cli options using rich/click and other bells and whistles of customization though this was a one time task. There were close to 50 options that had to be updated but the script only had 5 keys. I tried to prompt it to add additional keys but it was still not complete. I was asked to check UPDATING.md in GitHub but the file never exists.

```
$ find . -iname '*upgrade*md'
$ find . -iname '*updat*md'
```

What caught my eye was that the above output had `airflow config update` with a description to migrate the configuration from Airflow 2 to 3. I tried enriching the chat with giving it a hint over `airflow config update` which is a much more robust solution from Airflow community to make the changes that is also updated with Airflow 3.x versions in future. The changes to configuration in this case are deterministic and finite with clear source and target but the model started arguing over the fact such an option never exists in `airflow config` subcommand never exists in Airflow 3. It again started to market the script it wrote as the defacto migration tool and to create a new command `airflow config update` that acts as a bash wrapper to call this python script to update. It again started to respond with commands `airflow config validate` and `airflow config diff` that clearly don't even exist.

As much as it has been my experience towards a single task that doesn't generalize it to all situations there have been many demonstrable positive improvements using AI. One of the use cases was about leveraging AI to provide translations for UI components in Apache Airflow project slated for Airflow 3.1 release that makes it more accessible to wider audiences. It has been a positive collaboration with people from different communities and languages in this effort. You can read more about it in the below link

[AI and Open Source: Expanding Apache Airflow's Global Impact Through Collaboration](https://news.apache.org/foundation/entry/ai-and-open-source-expanding-apache-airflows-global-impact-through-collaboration)


### Conflict of interest

> During the gold rush its a good time to be in the pick and shovel business - Mark Twain

Advertisements these days are all over the web about how it's a waste of time to learn programming or a beginner friendly programming language like Python. Some of these are quotes taken out of context from executives to grab headlines to grab clicks. Some even act as advertisements to courses to deter people from actually learning a language and to take their paid course in prompting with which you can start to earn in lakhs quickly. Just like when someone recommends a stock they have financial interest in these statements are often present to keep the technology buzz and hot since they have their personal interest in the company and technology.

The interests don't just end financially and also gives access to a lot of data. These days with a lot of the tools like MCP to be connected internal documents, source code, personal files etc. there is a treasure trove of data that people trust with the providers which earlier used to be very hard to obtain. There were a lot of recent posts about how companies have access to data in the name of Generative AI. These have also generated a lot of discussion about the widespread use of AI. Generative AI is a form of generative revenue for companies that sell it but are currently an expense/investment with returns left as an exercise to the reader.

* [Copilot Broke Your Audit Log, but Microsoft Won't Tell You](https://pistachioapp.com/blog/copilot-broke-your-audit-log) - [HN discussion](https://news.ycombinator.com/item?id=44957454)
* [Perplexity is using stealth, undeclared crawlers to evade website no-crawl directives](blog.cloudflare.com/perplexity-is-using-stealth-undeclared-crawlers-to-evade-website-no-crawl-directives/) - [HN discussion](https://news.ycombinator.com/item?id=44957454)
* [How We Exploited CodeRabbit: From a Simple PR to RCE and Write Access on 1M Repositories](https://research.kudelskisecurity.com/2025/08/19/how-we-exploited-coderabbit-from-a-simple-pr-to-rce-and-write-access-on-1m-repositories/) - [HN discussion](https://news.ycombinator.com/item?id=44953032)


### News cycles and healthy skepticism

As much as industry leaders and visionaries have lot of insights into the future with their experience, access to usage data etc. it's a good practice to always step back and put up a hat of healthy skepticism to understand that there is also a conflict of interest in these statements both positive and negative. For the last few months there have been posts about how LLM and agents will replace software engineers but this week it has been all about the opposite. The narratives keep changing and suddenly anything with AI optimistic or pessimistic attracts clicks which leads to lot of polar opinions leaning towards extreme ends of the spectrum. As much as people want to hear something that substantiates their position it will be helpful to discern through the news and noise cycle so that people learn to accept the capabilities and disadvantages. To see where it fits and where it doesn't to accept it instead of alienating the other camp that has a different voice will be healthy. The voices on both ends are constant throughout the tech cycles like Internet, mobile, big data, cloud computing, cryptocurrency and so on.

* [How 95% of AI companies fail](https://www.financialexpress.com/life/technology/generative-ai-pilots-reporting-95-failure-finds-mit-study-author-explains-the-learning-gap/3951657/)
* [AWS CEO says using AI to replace junior staff is 'Dumbest thing I've ever heard'](https://www.theregister.com/2025/08/21/aws_ceo_entry_level_jobs_opinion/)


### Some interesting reads

* [LLM inevitabilism](https://tomrenner.com/posts/llm-inevitabilism/) and [HN discussion](https://news.ycombinator.com/item?id=44567857)
* [Writing Code Was Never The Bottleneck](https://ordep.dev/posts/writing-code-was-never-the-bottleneck) and [HN discussion](https://news.ycombinator.com/item?id=44429789)

*Pushes to git and rushes to order dinner*
