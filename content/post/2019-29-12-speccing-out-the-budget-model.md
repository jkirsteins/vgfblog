---
title: "Designing the budget model"
date: 2019-12-29T20:04:04+01:00
draft: false
tags: [csharp,design,2nd period]
categories: [behind the scenes]
ogimage: /og_image__designing_budget_model.png
---

This is going to be the first post in a new category of posts about developing
Very Good Finances.

The category is going to offer a *behind the scenes* type look, where I document the development
process behind VGF:

- what daily problems I face, and
- what tools I use, and
- how I reason about the design decisions, and
- so on.

For the seminal post, I chose to describe the process behind designing the data model
for the budget.

<!--more-->

*Side note: implementing budgeting is the goal for Feb 1, 2020. I set that goal in the [first retrospective]({{< ref "2019-12-14-first-retrospective.md" >}}) 👈*

## The process (for context)

The way I develop VGF is very simple:

- model the data
- create the interfaces that should support the use cases I want
- write tests
- write implementation

I can go a long time like this, before I connect the dashboard to the backend API.

## What's in a budget?

The budget is a central concept to VGF. While VGF aims to be more than just a budgeting application, budgeting is an important and needed tool. Implementing the budgeting functionality is also my main goal for February 1, 2020 (as outlined in the previous retrospective).

So how do we implement a budget? Let's see, from my point of view, a budget comprises:

- budget categories (with a name, tied to a specific month, and with a planned spending cap, and the remaining amount in the month)
- that's it.

So the starting point of implementing the budget could be a budget category model like this:

<script src="https://gist.github.com/kirsis/2ada35f8df4ec3c228db1db7de35933a.js"></script>

To display the budget, I could query the budget categories for a given user. To get the remaining amounts in each category, I could sum the transaction amounts in the given month, and subtract from the category *amount*.

## First draft evaluation

This design imposes some constraints:

- it assumes that users want a budget aligned to monthly boundaries;
- these budget categories must exist in the database for each month we care about.

The first one is an assumption about user behaviour. [These are usually incorrect](https://xkcd.com/1172/).

The second constraint seems brittle. Luckily, it naturally disappears, as the
model is refactored to support flexible boundaries.

On the plus side, there is an interesting benefit from this structure, which my final approach lacks: the budget can change structures midway through. You can add a budget category in February that you did not have in January.

### Budget intervals

First of all, a budget is not necessarily useful only in a monthly interval. Let's see how people get paid.

A very quick Google turns up this link: https://www.paycor.com/resource-center/10-things-to-know-about-pay-periods. So we might want to have *monthly*, *every 2 weeks*, and *twice a month*.

<p style="padding-left:2em;font-style:italic"><strong>Interesting to note</strong>: you might have a 'twice-a-week' pay period, and a 'twice-a-month' pay period. These are similar but subtly different.</p>

Here is an equally quickly googlable article on trying weekly budgets as well: https://www.moneyunder30.com/need-help-with-your-budget-try-a-weekly-budget.

I think it is fine to implement only a monthly budget initially, but the underlying model needs to support different periods. To start with, we need to define something to refer to the period. Let's start with a simple structure for refering to a specific period:

<script src="https://gist.github.com/kirsis/50d10aec1298ff368ae840eb0ebe5760.js"></script>

We will also need a way to refer to different periods, without the need for the user to specify start and end dates. Something like *Next* and *Previous* period. This requires a way to specify what the *step* should be:

- the step unit can not be *months*, because it would prohibit weekly, biweekly, or twice-a-month budgets; but
- the step unit can not be *days* either, because a month does not have a set number of days.

Let's go with explicitly accounting for every conceivable period.

If there are any users down the line that want something highly specific ("I want to budget between the 4th of the month, to the last Sunday until 15:30, and then until the 4th"), then they'll just have to live with it (or we can add it to the enum).

How about:

<script src="https://gist.github.com/kirsis/8c9299ad99a2903056b8b552260ed5fb.js"></script>

Now that we have a period, and an *Interval*, are we still missing anything? Let's see - can we satisfy the use case:

> Given a Budget interval, and today's date, can I know the required budget period?

The answer is no.

If we have a monthly period, then we can know that the period starts in the current month, on the 1st. But if we have an every-two-weeks budget, then we have no reference point for when the budget was started (the period start/end dates do not align neatly on month boundaries).

We need some type of reference date, which we can use in calculating the period.

Additionally, we need a place to store the chosen interval.

Let's:

- introduce a *Budget* class,
- store the start date and interval there, and
- move the *UserId* into *Budget* and out of the budget categories.

<script src="https://gist.github.com/kirsis/7dc5eafeece6c1586c06ff43f8a59faf.js"></script>

Note that I'm not using SQL, and there is no *BudgetId* that the categories reference. Likely fine to store all the categories in a single document (here I go, making assumptions again).

At this point we can have a stable budget structure (categories) that can be used with any time period. But there is a problem - the *Amount* is still on the category.  With these last changes, this field will not work. We need the amount to be flexible, and apply to a specific time period.

### Flexible amounts

Having flexible budget periods makes it a bit more tricky to specify the amounts. Let's deal with this iteratively.
First thing we know is that we absolutely must move the Amount out from the category, and into something that has a start, and an end date, and refers to a category:

<script src="https://gist.github.com/kirsis/2d625a30d1fd4df53ad75d3e1ae78871.js"></script>

We will need to add some type of Id to the categories, but otherwise this is compatible with what we have so far.

Interestingly, this should allow us to switch budget periods *on the fly*, as needed. If a person starts budgeting weekly, and over time, decides they would be better served with monthly, or quarterly budgets, they can switch the interval. All the budget modifiers still apply to the same time periods, and new ones that are added can have different periods, and overlap.

One final thing that we are missing, is a way to express a *view* into the budget, with specific start and end dates. This view should rely on the underlying *canonical* budget values, but only expose the values the user is interested in:

<script src="https://gist.github.com/kirsis/72dc3efd7f29469ebc453c4f900f301a.js"></script>

Notice that the categories are an `IOrderedEnumerable`, not `IEnumerable`. The user will presumably want to order how the budget categories are displayed. To accomplish this, we can add some type of sorting key to the budget categories.

Additionally, the summary contains information the user will presumably want to see - the allocated, and the remaining values for each category in a given period.

None of the data here is going to be persisted, it is all computed from the underlying data.

Let's move on to how we might interact with this data.

## Retrieving the data

Once I have a rough draft of a model laid out like this, I move on to designing the classes for interacting with the data.

The way VGF is structured responsibility-wise is:

- have a thin domain model. I don't add business logic to these, unless it is purely manipulating its own values, or computing new ones, without any dependencies;
- have a persistence model layer. All this means is that storage-specific information is not added to the domain model;
- have a storage layer, which stores, and retrieves data, but does not apply any logic to this
- have a service layer, which contains the bulk of the business logic (everything involving dependencies, or interactions between multiple models).

The services can depend on the storage interfaces, so let's design those first:

<script src="https://gist.github.com/kirsis/e9d54adaf01e6f280f11244e3be28beb.js"></script>

This is the bare minimum, I think. With this we can:

- get a budget;
- get the budget modifiers to determine allocated amounts for different categories; and
- calculate the sum of all transactions in a period (to determine budget category remainders).

This should enable building most of the use cases. It is still missing methods for updating budget modifiers, but I'll start with the reading part first. I can mock the data in tests until then.

## The services

Now, on top of the models and the storage interfaces, we can add some business logic. Let's see what scenarios need to be accomplished (except for updates):

1. The user should be able to go to a *Budget* section, and have the *current* period visible.
2. The user should be able to click on *Previous* or *Next* period.
3. In each period, the user should be able to see all of their categories, with the total, and remaining amounts.

Calculating the budget summaries is needed for the first two, so let's start with the third item.

### Budget category service

To summarize a category, we need to revisit the `BudgetModifier` class, and add a method to calculate its value in a given period (in case the budget modifier falls partially outside of it).

Once we have that, we can create a `BudgetCategoryService`, which depends on two storage interfaces, and exposes a method for summarizing a given category, for a given user, in a given period. Something like this:

<script src="https://gist.github.com/kirsis/df40717ae50c94d38770f097155a99b4.js"></script>

Now, with this in place, we can mock out fetching the budget periods.

### Budget service

Now let's tackle getting the current budget period. We will need to do some budget period math, which is related to the chosen intervals. I opted to add this as an extension method to the `BudgetInterval` enumeration:

<script src="https://gist.github.com/kirsis/9bcab68c8b0d8698b63beacb3c2739a7.js"></script>

With this in place, we have an entry point for calculating the period for any given point in time. Now we can introduce a budget service, which relies on this:

<script src="https://gist.github.com/kirsis/abf4c364b0dcb4df37f220208ab4c819.js"></script>

This service depends on the budget (retrievable from the storage interface), and the budget category service we established previously.

It exposes a way to get a `BudgetPeriod` for a given point in time (this enables the first scenario), and a way to get a relative period (e.g. *next* or *previous* period. This enables the 2nd scenario).

### Implementation details

You might notice that the service classes are more detailed, and some methods even have an implementation.

This is simply to outline how all the pieces fit together nicely. While speccing out the API, I will leave out low-level implementation details, but implement higher-level methods that depend on these low-level implementations. This lets me get a feel for the API, and spot potential pitfalls before I commit to implementing the logic.

At this point, I feel comfortable enough with the design, to commit to writing tests and the missing implementation details.

## Possible pitfalls

There are some potential pitfalls that I am aware of, but that I don't really know what to do about currently:

- in the `BudgetInterval` extensions, I initialize a `DateTimeOffset` using a default calendar. This is going to be a Gregorian calendar. How will this impact [Japanese users](https://docs.microsoft.com/en-us/dotnet/api/system.globalization.japanesecalendar?view=netframework-4.8), for example?
- I need to be wary of introducing rounding errors when calculating budget modifier values for different time periods. If a budget modifier falls within a period only partially, we want to take into account a partial value. In the adjacent period, we want to use the remaining value. This is a perfect opportunity for subtle rounding errors to creep in, and display non-matching allocated amounts in different periods.

## Off-budget transactions

Savvy budgeters might point out that I have overlooked off-budget transactions
and accounts. There are a few use cases for having this feature:

1. Imagine you have three accounts. Two are day-to-day checkings accounts, and one is a savings account. You want money transfers between the checkings accounts to
not affect the budget (it does not matter where the money is parked, just what
you plan to spend it on).
2. You might want to have an investment account. If you own stocks, you probably
do not want their value fluctuations to affect your budget.
3. Liabilities. If you lend money to someone, you would expect your net worth to remain the same, but the amount available for budgeting to decrease.

All of these can be achieved by having accounts with different types, where the
underlying transactions are tracked (to determine your net worth), but not
counted towards the budget.

VGF does not have these account types yet, but the entry point to supporting
this would probably be in the `ITransactionStore.GetSumInPeriod` method. The budget
does not care about the details, only what amount (that counts towards to the budget)
has been spent.

## And that's it folks

This has been a lot longer post than I anticipated. I hope it's an interesting insight into the process of building VGF.

If you have any comments, or questions, feel free to reach out on Twitter (or sign-up to e-mail updates, and reply to the welcome e-mail).
