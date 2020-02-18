---
layout:     post
title:      A beginners guide to Cron
date:       2019-05-06
summary:    A quick starter to automating your scripts
categories: beginners shell scripting productivity
---

Need to back-up on a Linux server monthly? Automate some scripting on a Mac every few days? or run some hacky code on your Raspberry Pi every hour?

Cron jobs are here to make it easier.

---

Recently in work, I've been automating some regular data capture using a really bloated approach until I remembered about Cron. 

The reminder of how useful it can be has led to using it on a variety of smaller projects I've been working on such as some data collection across a few Raspberry Pis.

Using Cron has reminded me of how much utility it has and how sometimes how it is overlooked. So, this post is aimed at helping beginners, those new to UNIX-like environments or someone a bit rusty, start using Cron. 

So first;

# What is Cron?
Cron's name comes from the Greek word for time; Chronus, And it's all about time with Cron, it's a time-based job scheduler for Unix-like OS's. Meaning it allows us to schedule jobs to run at fixed times, dates, or intervals continuously, these jobs can be anything as long as it's one line; from a simple command to a complex script. 

Often these jobs are used for system maintenance, automation or admin tasks, used for taking the onus off the user to remember and enabling it to run with no intervention.

Sounds useful, how do we start?


# Creating a Cron job
It all happens on the command line, so starting there and open the Cron table, or crontab. It's where we can see, edit and create new Cron jobs, this can be accessed by typing this;
``` bash
crontab -e
```
If you've never used Cron before it'll open an empty file for you to enter your commands.

Note: It may ask you to set a default editor, if you haven't already, to edit the file containing the table, this can any command line editor such as Vim or nano.

## Cron Format
Cron follows a standard format, 5 values then your command, minutes, hours, day, month and day of the week. For example;
``` bash
5 7 * * * /home/test.sh
```
This would run a bash script, test.sh, at 5 past 7, every day.

A good visual used a lot for explaining what each of these 5 values covers, is this one;

``` bash
┌───────────── minute (0 - 59)
│ ┌───────────── hour (0 - 23)
│ │ ┌───────────── day of the month (1 - 31)
│ │ │ ┌───────────── month (1 - 12)
│ │ │ │ ┌───────────── day of the week (0 - 6) (Sunday to Saturday, 7 is also Sunday on some systems)
│ │ │ │ │                                   
│ │ │ │ │
* * * * * command to execute
```

Some examples would be ` 0 * * * *` would be every minute, ` * * * * *` would be every second or `0 0 * * *` the start of every month. An asterisk means the same as it does in most applications, first to the last inclusive, so everything. 

But there's more you can do though than picking one value;

#### Slash 
You can use '/' to indicate steps, so `*/10 * * * *` would indicate every 10 minutes. 

#### Hyphen
Another is '-' can be used for inclusive ranges so `0 0 1-6 * *` would be the start of every month for the first 6 months. 

#### Comma
The last one I'll talk about for now is ',' which can be used to create lists, for example, `0 * * * MON,WED,FRI` which would run every minuite on Mondays, Wednesdays and Fridays.

There are more options such as using some characters but these aren't standard across most platforms.

#### Keywords
Above we also just used some keywords, Cron has some functionality to understand keyphrases like shorthand for days and months. So for the day of week field, you can use MON-SUN. And for the month field, you can use JAN-DEC.

On a lot of systems there some keywords that can be used for some heavily used times, here's a list of some of the most common ones;

``` bash
@reboot        Run once at startup
@yearly        Run once a year (0 0 1 1 *)
@annually      same as @yearly
@monthly       Run once a month (0 0 1 * *)
@weekly        Run once a week (0 0 * * 0)
@daily         Run once a day (0 0 * * *)
@midnight      (same as @daily)
@hourly        Run once an hour (0 * * * *)
```

To make this easier there's a number of fairly good online [crontab generators](https://crontab-generator.org/), to help build your job in a more visual way.

## You've got mail?
At this point, it's worth mentioning that you're going to get some mail. Every-time a Cron job runs, so maybe once a minute, it uses SMTP to send you an email notification to your terminal. You'll see a prompt next time you open your terminal saying you've got mail, running `mail` will allow you to see these and from there you can `del *` to delete all of the mail... 

Alternatively, if you add this to the start of the crontab
``` bash
MAILTO=""
```
It'll prevent it from sending your terminal mail.

## Checking your Job
Now if you've created a job, you can see our Cron table, which will contain it by running this;

``` bash
crontab -l
```

Which will allow you to see your job and any others you've set, `-e` will enable you to go back and edit your job if you want to.

## Removing a job
To remove your crontab to stop all your jobs you can simply run this;
``` bash
crontab -r
```
And they'll all be gone, or you can just edit the crontab and remove individual ones that you don't want.


# Pitfalls
Although Cron seems fairly straightforward, it's also full of gotchas, speaking to anyone who's worked with Cron jobs usually leads to some story about a misconfigured job that's been a pain to resolve or lead to some catastrophe. So when using them you can regularly find yourself getting tripped up over simple things or some of the odder aspects of the tool.

Some of the most common traps come from:
- Wrong Crontab notation
- Permission problems
- Environment variables

This Stack Exchange [thread](https://askubuntu.com/questions/23009/why-crontab-scripts-are-not-working) covers a lot of the most common ones and is a really valuable resource for troubleshooting your issues.

---

Overall, Cron is a great tool when used in the right place but it's not a one-stop shop for your scheduling needs. Remember to pick the right tool for the job but if its Cron, hopefully, this guide will set you off in the right path!
