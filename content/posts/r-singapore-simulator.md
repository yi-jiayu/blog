---
title: "r/singapore Simulator"
slug: r-singapore-simulator
subtitle: "Generating Reddit comments with Markov chains"
date: 2018-04-28T13:45:03+08:00
---

[Previously](https://medium.com/@jiayu./a-primer-on-markov-chains-2668d94032b1), we generated some new sentences from a small pool of existing sentences. Now, we'll generate some new comments based on all the existing comments in a Reddit thread!

This post is a continuation of "A Primer on Markov Chains", which introduces Markov chains and how they can be used to generate text:

After reading the daily r/singapore random discussion and small questions thread last week, I wondered if it would be possible to generate comments which looked like they came from there. Now, we'll be doing exactly that with today's thread:

We'll use the Python Reddit API Wrapper, or PRAW, to download all the comments from a Reddit thread. While the Reddit API is quite interesting and working with the Reddit comment tree could be an interesting exercise on its own, for now we'll stick to PRAW for simplicity.

We'll also use the Markovify library for building our Markov model instead of our own implementation, once again just out of convenience. Markovify is already widely used for a variety of text generation applications, while our own implementation doesn't even handle casing or punctuation at the moment.

## Downloading Reddit comments

Did you know that you can append .json (or .xml) behind a subreddit or thread URL to get the JSON (or XML) representation of its content? For example:

```
https://www.reddit.com/r/singapore.json
https://www.reddit.com/r/singapore/comments/8eoesx/rsingapore_random_discussion_and_small_questions.json
```

This is similar to what you'll get through the Reddit API itself, although probably with stricter rate limits than through the API.

In order to use the Reddit API, we'll have to create a new Reddit application and obtain some OAuth2 credentials, which the Reddit API uses for authentication.

### Creating a Reddit application

Log in to your Reddit account and head to the bottom of https://www.reddit.com/prefs/apps to create a new Reddit application. You should see the following form:

Take a look at the API usage guidelines, but don't worry about registering for production usage right now---we won't be doing anything that could be considered production usage.

Make sure you've selected "script". Give your new application a name, description and put in an about url (perhaps this blog post? haha). The redirect url shouldn't matter for a script application, so put anything you want or just point it at localhost:

After you've created your application, you'll be able to obtain your OAuth2 client ID and secret. Your client ID is at the top right underneath the app name and type, and you can see your client secret in the "secret" field.

### Iterating over comments with PRAW

I want to write about working directly with the Reddit API next time, but right now this is all we need to start using PRAW. Comment extracting and parsing seems to be a pretty common use case for PRAW, so there's a comprehensive tutorial in the PRAW documentation already:

The following code will iterate over all the comments in a thread and print the author and comment body. The Reddit API does not necessarily return every comment at once; sometimes the response contains placeholders. `submission.comments.replace_more(limit=None)` ensures that we replace all the placeholders with the actual comments they represent. For large threads, this step will take up most of the time spent.
