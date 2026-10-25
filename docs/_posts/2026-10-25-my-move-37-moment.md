---
layout: post
title:  "My Move 37 moment"
date:   2026-10-25 12:00:00 -0500
tags: [ai,vibe-coding]
---

In my [previous blog post]({% post_url 2026-03-01-fitbit-dashboard %}), describing work I did over the Christmas 
break 2025, I wrote about my first attempt to engineer a serious application using Claude Code. I had downloaded my 
personal data from Fitbit, and used the agent to engineer a client-server application to import, host, transform and 
export that data, and to serve as a display to replace the Fitbit dashboard that Google had taken off the internet. 
In that project, I took care to make the AI stick to the architecture, watched code quality, and hand reviewed every line 
before it was committed. In the conclusion of that post, I wrote the following. It will soon become clear why I
highlighted one phrase in this quote.

> It's perfectly capable of implementing a design and architecture philosophy *when
asked to do so*, and it is capable of weighing pros and cons of technical choices. This used to be the entire *how*
part of the job, so our work has changed significantly and permanently, there is no way around it. The *what* and *why*
parts are different, because the choices made there depend on priorities. The agent doesn't need a Fitbit dashboard.
**The agent has no reason to care why the total number of steps per day is interesting, but the average number of
heartbeats per day is not.**
>
> Maybe next year the *what* and *why* steps will be automated too, but as of early 2026, Claude Code is not the tool for
that yet.

Well here we are almost a year later. Besides Fitbit, I've been using a [calorie counting app](https://www.fatsecret.com) 
for about a decade. Apart from the embarrassing name, the app is perfectly fine. It has a feature to sync its data with 
Apple Health, but only going forward, not with historical data. That's ok, I'll vibe code it. I have been using 
[T3Code](https://t3.codes) for this, which is an orchestrator of harnesses. In simpler terms, it allows you to control 
Claude Code, Codex, Cursor, Grok etc. from one single application, which is really useful if you're on the $20 tier of 
several of them. When you hit your token limit with Claude, you just seamlessly switch to Codex and continue working 
without interruption. It also has a few other neat features, such as a mini browser window that the agent can click 
while testing and developing the application it is building for you. 

I downloaded my 10 years of diet data from the website, and gave Claude Opus 5.5 the following prompt: 

> look in this directory. fatsecret-apple-health-2017-10-12-2026-10-18 contains diet data that should be in the format 
> of apple health. make a plan to create an app to import that data into my apple health

I won't copy the development plan Claude presented because I believe in sharing the prompt, not the slop. The plan was 
solid. It would make a simple python web application to import the data, store it in a SQLite database, display the data 
in a simple web UI, and export it in Apple Health import format. It also created a plan to build an iOS app to import 
the data on my phone. Apple only allows write access to HealthKit from an iOS device. 

Looking at the data, Claude asked some clarifying questions about timestamps, because the diary entries were not all in 
the same time zone. Here's my second prompt of the session to answer that question:

> ive travelled between continental europe and the US a lot over the years. was mostly based in amsterdam, am now 
> based in north carolina

It replied that this makes sense, there was no problem with the timestamps after all and entries had really been 
logged from Amsterdam (CET/CEST), North Carolina (ET) and US West Coast (PT). 

It next asked me if I had Xcode installed and if I had a developer account. After I answered those questions, it built 
both the web application and the iOS app, imported the data, exported it to Apple Health format XML, and deployed 
the **single-use** iOS app on my phone. I had to run it manually and click the button to import the XML file, and it 
was done. I can now get a bar chart of the average daily amount of Polyunsaturated Fat I consumed in July 2018. 
Awesome! 

At no moment did I ever look at the code, because it wasn't relevant. The entire application was single-use. 
No one would ever have written a single-use web application and iOS app by hand before. It would have taken weeks, 
not counting research.

## My Move 37 moment

The vibe coding experience above didn't shock me. A period of 10 months is not a long time, but it felt like a long 
time. I've made an effort to stay on top of developments in Agentic Engineering and maintain some level of understanding 
of the underlying theory and limitations of the technology, so this didn't come as a surprise. I fully expected this to 
work exactly as I described above. 

For my next project, I decided to import the roughly 10 years of records of runs, rides and hikes from the 
[MapMyFitness](https://www.mapmyfitness.com) app into Apple Health as well. 

I'll share the prompt, not the slop:

> this project contains csv with workouts from mapmyfitness.com we want to make a local application that does roughly 
> the same as the official website but only for my data 

That's the whole prompt. About an hour later I had the app to import and view the data. A few more prompts to add local 
storage in SQLite and an export function to Apple Health format, and it was done. It made a nice dashboard with bar 
charts per activity. I hadn't told it to do that, it just understood how I would want this data to be displayed. 

![Fitness Dashboard](/images/2026-10-25/fitness-dashboard-1.png)

In the chart above, I noticed something weird. I ran 32 km in a week in August? I haven't been capable of such feats 
since I injured my knee a couple of years ago. Maybe the bot made a mistake? This was exciting, not because I want the 
bot to fail; I went through that stage a few years ago and am fully in the acceptance stage now. It was exciting because 
there was something to investigate. Or so I thought. 

![Workouts](/images/2026-10-25/workouts-error-1.png)

Oh... I clicked through to the detail page. It had made a very nice detail page without being asked. 

![Detail](/images/2026-10-25/workouts-error-detail.png)

I looked at the CSV, and saw no warning there. I asked the agent:

> where does this warning come from:
> 
> ⚠ The average pace is far off your usual run pace (GPS glitch, tracker left running, or logged as the wrong activity). 
It is excluded from records and pace trends.

I'll include the screenshot of the full response: 

![Move 37](/images/2026-10-25/move-37-moment.png)

Now in the past, with older, slower phones, the tracker would sometimes fail to keep up. The tracker uses a combination 
of GPS coordinates and maps of roads and trails to make an accurate estimate of where you are. 

It would sometimes lose connection with GPS, especially going on trails far off road, so sometimes instead of tracking 
me on the running trail, it had me teleporting back and forth between two adjacent roads, so it would show me as having 
run for example 12 km instead of 10, giving an implausibly high average speed. I'd add that in the comments afterwards. 
Don't blame the app for that; the problem never recurred after finally upgrading to a newer phone. I haven't had to 
add comments to records in years. 

From my comments from several years ago, the agent deduced that the data might not always be 100% reliable, and 
completely on its own - again, I shared all the relevant prompts here - decided to design an algorithm to detect faulty 
data and excluded it from the records page. The offending record on August 25th probably reflects me going on a 
morning run, then forgetting to turn the tracker off, sitting at my desk working all day, driving to the store and 
perhaps taking a nap, covering 28.37 km in 18 hours and 19 minutes at an average pace of 38 minutes and 45 seconds per 
kilometer. That is not the work of a stochastic parrot, or even a junior developer. 

In 2016, [AlphaGo played against professional Go player Lee Sedol](https://en.wikipedia.org/wiki/AlphaGo_versus_Lee_Sedol).
During this match, it made the now famous 'Move 37'. This was a surprising and unconventional move played by Google 
DeepMind's artificial intelligence.

Since then, the term 'Move 37' has become an expression for situations where an AI shocks and surprises people by 
exceeding expectations of its capabilities. This was my Move 37 moment. I don't mean Claude made an alien or inhuman 
move, quite the opposite. It unexpectedly acted like a human engineer. 

A year ago, Claude made a Fitbit dashboard for me where it displayed the day's average heart rate, because why not? For 
the bot, heart rate is just a number, right? I wrote: 

> **The agent has no reason to care why the total number of steps per day is interesting, but the average number of
heartbeats per day is not.**

At the time I thought we had reached the limitations of AI coding agents here. Without lived experience of the world, 
it would need a human to decide what's interesting and what isn't, right? Well here I am, less than a year later, 
reconsidering this statement. 

## Senior Software Engineer

In these exchanges, Claude didn't act like a bot; it acted like a senior software engineer. It asked clarifying 
questions, it noticed discrepancies in the data and asked me about those, then it came up with a design and implemented 
and tested it. It implemented useful features I didn't explicitly ask for, without also adding useless or irrelevant 
features. 

_If you are still dismissing coding agents as fancy autocomplete, a stochastic parrot, or only suitable for
dumb routine work, I am not going to debate this topic with you any more._ 

Last year, leaders of the frontier AI labs predicted that in a year to 18 months, we would be at the point that we are
now. I think Google demonstrated that a drawing on a paper napkin was enough of a design for Gemini to build an entire
website. When their model was released to the public, it fell short of the expectations. It became tempting to dismiss
the grandiose claims as marketing. Well, we are there now.

At the Rails World 2026 Opening Keynote, [DHH made the following statement](https://x.com/dhh/status/2102936073642869121):

> It's pencils down, people. Writing code by hand is no longer an economically viable skill for most programmers at
> most companies. But the future of making software has never been brighter. Don't you dare black pill this beautiful
> moment!

I am ambivalent about his optimism about the future, but I'm not in denial. If I want to do the best I can do, in 2026
that is not writing code by hand. I will adapt, but I'm not yet at DHH's stage of enthusiasm about the situation. 

I wrote [my first line of code]({% post_url 2024-12-31-bbc-master %}) in the late 1980s. The ability to code, create software, research, troubleshooting and debugging have been part of my identity as well 
as my source of income throughout my adult life. We have to either admit that this is coming to an end, or keep 
retreating into ever smaller corners of the profession where bots can't or aren't allowed to go yet. 

I'll leave you with this [Eulogy for the Software Engineer](https://www.youtube.com/watch?v=uuKOXGVIP8I).


