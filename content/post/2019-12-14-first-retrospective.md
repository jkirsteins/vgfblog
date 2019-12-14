---
title: "First Retrospective"
date: 2019-12-14T16:44:32+01:00
draft: false
ogimage: /1st-retro-dash-ui.png
categories: [retrospectives]
tags: []
---

I started working on Very Good Finances 6 weeks ago. Since then, my velocity has slowed down considerably, so I think it is time for some reflection.

Let's see what happened, what went right, and set some goals for the next month.

<!--more-->

## The Plan

### Sustainability

It is hard (for me) to sustain work on side projects unless I do it full time. More often than not, I would start something on a rainy weekend, and then not find time to return to it during the workdays. When the next weekend arrives, I'd lost considerable velocity, and find it harder to return to it.

On top of that, I knew that I am bad at managing my energy expenditure. I could work on a technical task very intently for days, only to be completely drained when I returned to the world of the living. The price for two days of intense, efficient coding, would be five days of exhaustion, and impaired day-to-day functioning.

So an important objective was for me to approach this project differently.  Instead of hunkering down to code every odd weekend, I would change how I work:

1. I decided to take two weeks off work to lay down the technical foundations. If I could work full-time, I hoped to sustain the momentum for long enough to get past the boring parts. I envisioned this to be a 2-week hackathon.
2. I would also work in the open. Participating in communities like Product Hunt and Indie Hackers, posting about goals and progress, would keep me accountable. It would also help break up my daily rhythm, and prevent me from exhausting myself in single-minded coding.

### Minimum Discussable Product

I have a love-hate relationship with soliciting feedback on my work. I am often incredibly thin-skinned when someone offers anything but praise for my creative output. On the other hand, actionable feedback can be incredibly motivating. It gives a sense of purpose and reduces uncertainty.

I decided early on that I want to get to something functional as soon as possible, so I can start shopping it around and solicit feedback.

To make it explicit, I wrote down the requirements for the MDP. By 18 November I wanted to have:

- a web application where anybody could sign up, and log in;
- the ability to edit budget categories, but not necessarily the budget itself;
- the ability to link bank accounts (for Revolut and N26);
- the ability to add transactions to these accounts manually.

My stretch goals were to have:

- the ability to create unlinked accounts;
- automatic transaction sync with Revolut;
- automatic transaction sync with N26.

### Non-Goals

I also had a single, explicit non-goal.

I did not want to spend time thinking and planning. I was afraid I could spend weeks thinking about what I wanted to build, so better to plow ahead.

I knew that I wanted to build something to help me manage my finances better. I knew that the starting point would be budgeting because I did not enjoy budgeting with the tools I could find.

The rest I would discover on the way, as I start talking to people about their budgeting needs.

## Outcome

![Screenshot as of 14 Dec 2019](/1st-retro-dash-ui.png)

<center>
*A screenshot of the current version as of 14-Dec-2019. It is going to be publicly available [here](https://app.dev.verygoodfinances.com) throughout its development.*
</center>

### Development

By 18 November, I had set up a simple application, where users could log in through Auth0. It was running only on my computer and would log out Safari users on every refresh.

Once logged in, it would allow users to link bank accounts through a third-party data aggregation API. Users could also see their accounts and their respective transactions.

I extracted the code for invoking the API and published it as an open-sourced library.

Since then, I also published it online. Unfortunately, it is not yet featured enough for soliciting feedback.

#### Development Scorecard

The scorecard against my original goals:

- ✅ ability to sign up and login;
- ✅ ability to link bank accounts;
- ❌ ability to edit budget categories;
- ❌ ability to edit transactions;
- ❌ automatic transaction sync;
- ❌ manual account creation.

Overall, I achieved half of the primary goals and none of the optional goals.

### Working Publicly

While I knew I wanted to be public about the work, I hadn't set out any specific goals. Shortly after starting, I created a daily reminder to post something on Indie Hackers and Product Hunt Makers.

⚠️ I kept this up nicely while I was on my vacation, but as soon as I returned to work, this interaction dropped to near zero.

✅ I set up this development diary, where I plan to post weekly, and I already have a content backlog for a few weeks.

### Starting a Newsletter

I set up [a newsletter about personal finance](https://thatfullpocket.com) almost immediately. I rationalized this with the idea that I need to do research anyway, and this would let me interact with people who are interested in the topic.

To compound this failure at discipline, I jumped into the deep end straight off the bat. I failed to consult people who might know what it takes or to ask for advice. Instead, I set up a landing page, a mailing list, and then spent a third of my vacation googling about newsletters.

Working on the newsletter derailed the plan for no good reason.

The only silver lining is that it is surprisingly enjoyable to have a weekly deadline, and a reason to write.

Score: ❌

### Customer Insights

I spent some time bothering my wife, friends, and family about their financial management. "Do you budget?" "Why not?" "What is missing from your current approach?"

Some comments I received in no particular order:

- some people wanted reminders about upcoming recurring payments (but not all of them);
- many budgeting apps offer only a 4-week planning period, but many people would prefer a 2-week  interval;
- not having automatic transaction sync is a pain point in current tools (in the EU);
- it would be useful to integrate with stockbroker APIs;
- don't assume that people work with a single currency.

It is good feedback, but I did not come up with a coherent vision of what I want to build.

I started out thinking I want to build a better budgeting tool, and arrived at a vague, "I want to help people form better financial habits."

Score: ❌

### What I Feel Good About

I (mostly) managed to avoid a recurring pitfall of mine. Too often, I decide to work in technologies that interest me and not technologies that make me productive.

I considered starting this project using serverside Swift. Halfway through, I had the urge to rewrite the frontend using Blazor.

I'm happy I went with Auth0, instead of writing a user management system.

I did succumb to some technical choices and tasks, which were not essential. The upside, however, is that I can turn those missteps into future content for this dev diary.

## Going Forward

Taking all of this into account, I have some ideas on how I want to proceed in the future.

Here is how I want to change my approach:

1. Do more planning. For fear of inaction, I completely neglected planning. Failure to set out clear goals makes it hard to judge, how well I am progressing, and if what I'm doing matters.
2. Spend less time on things that do not move me closer to my stated goals. Did I have to spend time wrestling with Auth0 for Safari users? Did I have to fix all the bugs I encountered, and write regression tests? No, all of that could have waited, and I could have moved faster on the things that matter.

And to hold myself accountable, here are my specific goals for the next progress update post (in order of descending importance):

- Implement a way to edit a monthly budget (categories and balances);
- Post weekly about the progress here;
- Post daily on Indie Hackers, and Product Hunt.

I will post the next update by 1 February 2020, because I think it's odd to have status updates in the middle of a month.