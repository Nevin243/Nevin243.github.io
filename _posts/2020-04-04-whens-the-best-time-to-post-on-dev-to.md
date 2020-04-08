---
layout:     post
title:      When's the Best Time to Post on Dev.to?
date:       2020-04-04
summary:    Using Python to work out visualise the best times and topics on Dev.to
categories: beginners meta python jupyter
---
![A Group of 4 clocks with different times](https://res.cloudinary.com/practicaldev/image/fetch/s--dr5NBGNo--/c_imagga_scale,f_auto,fl_progressive,h_420,q_auto,w_1000/https://res.cloudinary.com/practicaldev/image/fetch/s--nh33pZbn--/c_imagga_scale%2Cf_auto%2Cfl_progressive%2Ch_420%2Cq_auto%2Cw_1000/https://dev-to-uploads.s3.amazonaws.com/i/mhcvw3k74wn61eej8ppy.jpg)

Having an international community on Dev.to means that there are users active all around the clock, that makes it hard to understand when the community is the most active. Knowing this helps understand your audience more and helps you get the most engagement with the content you're posting.

With some simple Python, we can work out as much as we can about the users of Dev.to and their behaviour, like when we should be posting to get it in front of as many people as we can. Not to mention, its a good chance to learn something about data prep and manipulation.

---

Last year, I worked on a project to understand what users were posting on my company's Yammer, think company Facebook; we were using python to track engagement on topics my team were interested in, how they were being perceived and when they were being interacted with. While looking around for some visualisation ideas, I saw [Pierre's](https://dev.to/daolf) post about the best time to post [here](https://dev.to/daolf/-what-is-the-best-time-to-post-on-devto-a-data-backed-answer--1kob)

Now that I've started posting here more regularly, I thought I'd try something similar, so I decided to try a short project to understand more about the users here when they are reading, are there any trends and what topics they interact with the most. 

---

To try to understand the reader, we need to work out what determines a successful post - normally that's read numbers, comments and reactions. 

Since we can't get reads numbers for every post, we'll look at reactions as the two are strongly linked; more reacts, more reads and more reads, more reacts. So, reactions will be the metric we use to determine the users behaviour or response to a post!

# Getting Data

First, we need data.

We need to know how the previous posts have performed, luckily the team here have a great [API](https://docs.dev.to/api/) for us to use! 

Using Python and the [requests](https://requests.readthedocs.io/en/master/) library, we can call the API and build up a JSON lines file of every post:

```python
# To get the first page on articles on Dev.to
URL = "https://dev.to/api/articles"
payload = {"page":1}
r = requests.get(URL, params=payload)
r.raise_for_status()
f.write(r.text)
```

Splitting out the payload lets us iterate up the page value and get all posts up to a set page number, using this we can grab every post or aim for a rough date.

Running this we can build up a dataset, but it needs some cleaning up, to get it into a more parsable format for the next library we're going to use - Pandas, alongside [Numpy](https://numpy.org/) it's the backbone of data manipulation in Python.

Using Pandas and a simple [generator](http://book.pythontips.com/en/latest/generators.html) we can load the data into a DataFrame;

```python
# Generator for data
def json_line_gen(file_name):
    for row in open(file_name, "r"):
        yield row

json_response = json_line_gen('./data.json')

for json in json_response:
    df = df.append(pd.read_json(json), sort=False)
```
[Data frames](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.html) are really useful, basically, it's a table with labelled rows and columns, that are highly flexible with a lot of functionality built-in. They're easily scaled and manipulated enabling a whole world of data manipulation with one library.


# Prepping our Data
Getting the data frame set-up, we can do some initial exploration and see we need to do the same thing Pierre did; split the date and time into their own columns to make it easier to manipulate. While we're here we can also drop the columns we're not interested in, like the cover image or canonical URL;

```python
df = df.drop(columns=unwanted_columns)

# Splitting Timestamp into hour and day of week
df['hour'] = pd.to_datetime(df['published_at']).dt.hour
df['day_of_week'] = pd.to_datetime(df['published_at']).dt.strftime('%A')
```

# When's the Best Time to Post?

One of the first things we'll want to know about users is when they read posts and we can see this from when they are reacting to the posts. Part of the reason we broke it down into hours and not minutes is so we can make a generalisation, trying to estimate the time down to the minutes is too granular and won't really aid us any more than knowing the hour will.

The best way to visualise this then, will be as a heatmap - we start by grouping the data we need, reaction count and timing, before pivoting the table so that its columns will be the days of the week:

```python
# Get the average reactions per post at a given timeslot
reaction_df = df.groupby(["day_of_week", "hour"]) ["positive_reactions_count"].mean()

# Pivot the dataframe & reorganise the columns
reaction_df = reaction_df.reset_index().pivot('hour', 'day_of_week', 'positive_reactions_count')
reaction_df = reaction_df[["Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"]]
```

Then we use [Seaborn](https://seaborn.pydata.org/), a data visualisation library, to generate a heatmap:

```python
plt.figure(figsize=(16, 16))
sns.heatmap(reaction_df , cmap="coolwarm")
```
![Heatmap with outliers](https://dev-to-uploads.s3.amazonaws.com/i/a6l8rb3wk0dzzh1jkkhd.png)

And we hit a problem, there's no clear trend, sometimes that's just the case but this one hour on Sunday seems like an outlier. That time is a lot darker than the others - let's check, we can use a boxplot as a simple way of checking, again from Seaborn:

```python
sns.boxplot(reaction_df)
```
![Boxplot showing some outliers](https://dev-to-uploads.s3.amazonaws.com/i/nrvp45ugtwh2vmx3f4kr.png)

Definitely an outlier, since we're making a generalisation, let's filter out the outliers and see how it affects our visualisation. Using [z score](https://en.wikipedia.org/wiki/Standard_score) we can remove some of the outlier posts from our original data frame;

``` python
z = np.abs(stats.zscore(reaction_df))
reaction_df = reaction_df[(z < 3).all(axis=1)]
```

This filtering removes some of the top-performing posts of all time, but enables a far more general view of how most posts perform when we generate our heatmap again!

![Heatmap of the best time to post](https://dev-to-uploads.s3.amazonaws.com/i/sgd2jn1t1h8aebggds3j.png)

So that leaves us with a clear band; around midday UTC during the week!

This is similar to Pierre's findings on time, but on a tighter band, could be down to the site's presence growing and having an even broader user base or just having another year worth of data!


# Writing for a Specific Tag
The heatmap was a very general, what if you only are interested in producing content around some specific tags, let's try out one of my favourites - Discuss;

```python
# Tag to find map for
tag = 'discuss'

tag_df = tag_df.loc[tag_df['tags'].str.contains(tag, case=False, na=False)]

tag_df = tag_df.groupby(["day_of_week", "hour"]) ["positive_reactions_count"].mean()
tag_df = tag_df.reset_index().pivot('hour', 'day_of_week', 'positive_reactions_count')
tag_df = tag_df[["Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"]]
```
![Discuss heatmap](https://dev-to-uploads.s3.amazonaws.com/i/hn2qnz644xzzr7jm94r8.png)

Less clear, the problem is we're cutting out so much of our dataset, that we'll just start missing data that will begin skewing results. That said some of the most popular tags might have enough data to look into.

# Engagement and Comments
The discuss tag made me try something else, can we check for the link between how many people read and react to a post and how many people comment on it? This would be especially relevant for tags like discuss but generally if you want readers to interact with your post beyond just a react.

We can use a [regression plot](https://en.wikipedia.org/wiki/Partial_regression_plot) to compare the reactions with comments and see if there's a correlation:

```
sns.regplot(comment_df["comments_count"], comment_df["positive_reactions_count"])
```

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/n71ly5icg5hm7ee19hd8.png)

That's showing a moderate correlation, now that doesn't imply causation, but it's worth investing and trying out to investigate the link in the future.

If we replace the reaction count in our previous heatmaps, with the comment count column and generate a new heatmap:

```
comment_df = comment_df.groupby(["day_of_week", "hour"]) ["comments_count"].mean()
comment_df = comment_df.reset_index().pivot('hour', 'day_of_week', 'comments_count')
comment_df = comment_df[["Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"]]

plt.figure(figsize=(16, 16))
sns.heatmap(comment_df , cmap="coolwarm")
```
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/a71eot47azzmps09z840.png)

We can see there are two big clusters we can test in the future:
- Very early mornings UTC on Saturday and Sunday
- And early evening 5/6 PM UTC Tuesday - Sunday

Using these trends could lead to a more engagement on your posts through comments and discussion, worth a shot to see if there is causation or if you want to engage more with your users.

# What topics should we write about?
Let's jump back to tags then, what if someone wants to go full SEO? What're the most popular topics and does this tell us anything about the community?

To start we need to split the tag list column out into individual items, in Pandas, this used to be a massive pain but now its just a method call:

```python
popular_df = popular_df.explode('tag_list')

# Fill in any gaps
popular_df["tag_list"] = popular_df["tag_list"].fillna('None')
```

From there, we could sum by occurrences, but turns out this has already been done for [us](https://dev.to/tags)! Some big ones you'd expect, Javascript etc, so definitely what users like to post about, but is this what the community wants? are these the most interacting with?


If we first get the average reactions for comparison:
```python
popular_df['positive_reactions_count'].mean()
```

We can then use our dataset, with outliers removed, to generate a list of the top tags with the highest average reaction count:

```python
popular_sum_df = popular_df.groupby(['tag_list'])["positive_reactions_count"].mean()

# Get top 50 average posts
popular_sum_df.sort_values(ascending=False).head(50)
```
After giving some of the tags a check most are still fairly skewed by a couple highly reacted posts, compared to the average but some of the top tags aren't one-offs. They are tags that consistently performing well with good interaction, some at a quick look are;

- Career
- Beginners
- Hooks
- SQL
- Learned

(The full list of tags is available in the repo, take a look at them there)

Some I expected (Careers), some I didn't (SQL) but it allows us to look at what content our users are really interested in and what's not. This means will can filter content that would work best on this site, playing off trends or topics that people care about here.

---

# Summary
Understanding your audience through data is only part of the puzzle, just posting at these times won't just instantly increase your read count. You still need to focus on understanding the end reader and producing high-quality content first!

There's scope to take this further; what length or type of content performs best, is there any sentiment or structural aspects of a post that engages better and what topics are trending at a given time.

To do any of this, we'd need to build a richer dataset, the API looks like it could be manipulated to do that but that's content for another post.

If you want to try and start building something bigger, do this yourself or look at how quickly something like this can be done in python, the Jupyter notebook and scripts are available [here](https://github.com/Nevin243/data-driven-posting)

Now go spend some time trying to understand your reader, happy posting!