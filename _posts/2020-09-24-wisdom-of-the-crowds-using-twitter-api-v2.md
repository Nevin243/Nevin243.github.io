---
layout:     post
title:      Wisdom of the Crowds, Using Twitter API v2
date:       2020-09-24
summary:    Investigating psychology with Twitter's new API
categories: python postman 
---
![A blue bird scolling twitter on a branch](https://dev-to-uploads.s3.amazonaws.com/i/tuot7jca1jlicdi7oy6w.jpg)

The [Wisdom of the Crowds](https://en.wikipedia.org/wiki/Wisdom_of_the_crowd) is a concept that states that averaging a large group's opinions, or guesses, smoothes out their bias and noise to give a measured opinion or the right answer. Using this concept and the new Twitter API - can we start winning some competitions?

---

# The Idea
It's a pretty interesting concept and reading about it again recently in [Thinking, Fast and Slow](https://amzn.to/3i3rRLW) got me thinking about a short project to try out the new Twitter API. The author points specifically at challenges to guess the number of 'things' in a jar, individuals are often wrong but in theory, averaging the values should give you the winning answer!

So, there's the odd competition like this on Twitter, let's see if we can weaponise the new Twitter API to win some prizes?

# Twitter API
The new version of Twitter's API is in early access with a limited number of endpoints but it already looks interesting; it can't be worse than the restricted original right?

To test out this idea, we'll need a developer account to get started, you can [apply for access here](https://developer.twitter.com/en/apply-for-access). Once approved there are a few steps for setting up a [new project](https://developer.twitter.com/en/docs/twitter-api/tweets/lookup/quick-start) that are covered well in their new guides and far quicker than the original. I already had an account and more or less right away had a project set-up with keys generated. No more long waiting just to test out some ideas in a throwaway project.

Before building something out, we'll need to see if we can access the tweets for the competitions and prototype the calls we'll need to get the guesses. The docs [link off](https://documenter.getpostman.com/view/9956214/T1LMiT5U) to a prebuilt collection for Postman that lets us get started right away! (+1 for API V2)

# Postman Prototyping
We'll pick [this competition](https://twitter.com/ifixit/status/1293279536294105088) as our target since it's already over and we have the answer.

Then rushing in without reading the [docs](https://developer.twitter.com/en/docs/twitter-api/tweets/filtered-stream/introduction), we can use the Tweet's ID (from the URL) and the [Tweet Lookup Request](https://developer.twitter.com/en/docs/twitter-api/tweets/lookup/quick-start) we can get the [conversationID](https://developer.twitter.com/en/docs/twitter-api/conversation-id), which all replies and retweets will include, so we can use to filter tweets for it. We need to add in our Bearer Token before we can get the response, we can generate it through the Auth page for the project on the Twitter Developer Portal. When that's in we can send the request and get something like this;

![Postman window with Twitter API Call response](https://dev-to-uploads.s3.amazonaws.com/i/gxeyp6pynzwfghzuxgs3.png)

Great - now we have the conversation ID, we can use the [Filtered Stream](https://developer.twitter.com/en/docs/twitter-api/tweets/filtered-stream/introduction) endpoint to get the guesses!

Adding a rule to the stream to filter for the conversation ID, starting to stream in... nothing? Wait, what?

Back to the docs... ohhhh, ohhhh, oh.... oh. 

So that's not how it works.

![That's not how the force works](https://media.giphy.com/media/l3fZOxBdAIOiyt8CQ/giphy.gif)

So, streaming - very cool but; 
1) doesn't work on Postman, 
2) It's a stream... it's new, not historic... 

The docs quite clearly state that it's for monitoring an event in real-time, streaming tweets in when the competition starts we'd be able to calculate the average as it changes, but for proving out our idea it's not much use. I'll be revisiting the streaming endpoint soon though - looks interesting.

# Recent Search
The correct endpoint is [Recent Search](https://developer.twitter.com/en/docs/twitter-api/tweets/search/introduction), here we can search for tweets that meet our filters up to one week old. 
Why one week? Good question - seems like it's because the API is in early access, historic tweets are on their [platform roadmap](https://trello.com/b/myf7rKwV/twitter-developer-platform-roadmap) but not something we can use today.

**Note**: Due to the time restriction, we can't recreate the response for running this on the competition above. It was well over a week when I initially drafted this. Picking some trending topic today will show it working. Luckily we have the data from the initial run after the competition ended.

Running Recent Search with some parameters;
- `query=conversationID:<id>` - the competition tweet to filter for,
- `max_results=100` - the maximum tweets per response,
- `tweet.fields=text` - just giving us the text of the tweets

![PostMan request with for recent search](https://dev-to-uploads.s3.amazonaws.com/i/po78pr1cqtll04ncppk0.png)

We get a response with a data section that has the tweets with that conversation ID and a `next_token` parameter in the meta section. We can use this as a parameter in the next request, to get the next 'page' of tweets. As long as there's a `next_token` parameter in the response, we know there are more pages to parse!

![Twitter response in Postman](https://dev-to-uploads.s3.amazonaws.com/i/5rqlh0ykbsvwasf84936.png)

But we can't do this paging or the logic we'll need to find our answer in Postman; it does allow us to convert the request to a bunch of popular tools and languages though. So, we can output a Python 3 (and requests as we don't hate ourselves) snippet and copy directly to a separate script.

```python
import requests

url = "https://api.twitter.com/2/tweets/search/recent?max_results=100&tweet.fields=text&query=conversation_id:<id>"

payload = {}
headers = {
  'Authorization': '<Bearer Token>'
}

response = requests.request("GET", url, headers=headers, data = payload)

print(response.text.encode('utf8'))
```

# Paging and Parsing
Using the sample as a basis we need to add a little logic to handle the paging and parsing the tweets for the guesses. With a recursive call to handle the paging and a little regex to filter the numbers out we end left with something like this;

```python
url = ENDPOINT + str(conv_id) + POSTFIX

    if next_token:
        url = ENDPOINT + str(conv_id) + POSTFIX + TOKEN_PREFIX + next_token

    response = requests.request("GET", url, headers=headers, data = payload)

    json = response.json()

    for tweetObj in json['data']:
        tweet = tweetObj["text"]
        # Regex - maybe not the most pythonic
        guess = re.findall(r'\b\d+\b', tweet)

        if guess:
            guesses += guess

    if 'next_token' in json['meta']:
        token = json['meta']['next_token']
        build_data(token)
    else:
        token = []
```

# The Wisdom
This leaves us with the list of 206 guesses, which some python builtins make short work of and give us an average of 820 beans in the jar.

Which is exactly 100 out, the correct answer was 720 beans.

Giving the theory the benefit of the doubt and a little visualisation later, I discovered there's a lot of outlier guesses. Some people entered things like '1' or values far too big to be a real guess; well that or they think the jar is massive.

We can use [SciPy's](https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.trim_mean.html) trimmed mean to take out around 10% of the values out from the ends gives us an answer of 761 beans - A much closer guess!

With this relatively small sample of 206 guesses, I thought it was pretty impressive; unfortunately for us, someone guessed exactly right. But this theory was still considerably closer than my initial guess of 460. It'll be interesting to try it with a larger sample on the next competition I see!

---

Overall, a quick intro to the Twitter API makes it look really interesting and I'll have to revisit it in future. You can read about more of my Python-related posts [here](https://dev.to/m_nevin)!

<span>Cover photo by <a href="https://unsplash.com/@morningbrew?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Morning Brew</a> on <a href="https://unsplash.com/?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span> - They actually write a good newsletter - sign up [here](https://www.morningbrew.com/daily/r/?kid=0af228c8)