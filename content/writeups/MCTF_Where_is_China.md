---
date: '2026-03-16T16:34:44+01:00'
draft: false
title: 'Midnight Flag CTF 2026 | Where_is_China'
tags: ["osint", "geoint"]
keywords: ["osint", "geoint"]
showtoc: true
UseHugoToc: true
tocopen: true 
ShowReadingTime: true
ShowWordCount: true
---
Author: Evix

## Context

A Chinese student who had been living in France for a year to study French recently returned to her home country for a short one-week vacation. A few days later, her body was found lifeless in a park in China. Local authorities are struggling to understand what happened to her. Before her disappearance, the young woman was very active on social media. Your mission is to retrace her steps and discover which park her body was found in.

The victim's name was Lin Xiayu, and she used the username `LinXiayu35170` on social media.

Flag format: MCTF{Xxxx_Xxxx}

{{< figure
    src="/image/montagne.png"
>}}

## Social Media Search

Our first mission is to find the social media accounts of Lin Xiayu. We can start by entering the `LinXiayu35170` username on [WhatsMyName](https://whatsmyname.app/).

Nothing relevant came up, so I decided to check mainstream social media platforms like Facebook, Instagram, and Twitter.

We found 2 accounts on [Facebook](https://www.facebook.com/people/Lin-Xiayu/pfbid02hJB4m3ebYNpbp1erRUedrV8q4X7ZV2GEL63f5mSn3yPhjwJwrJqJ19zcWY5SbkGpl/?locale=fr_FR) and [Twitter](https://x.com/LinXiayu35170).

## Facebook

On the Facebook account we can find an image:

{{< figure
    src="/image/ville.jpg"
>}}

## Twitter

On the Twitter account we can find some tweets saying that she loves croissants — I can understand, I love them too, vive la France!

{{< figure
    src="/image/xiatwitter.png"
>}}

More interestingly, if we check her followers we can find a strange account, @heaprunner, belonging to a user named Lucas Juillet, who says weird things like "We think that we are sometimes alone, but we never really are."

{{< figure
    src="/image/heaprunner.png"
>}}

He even replied to one of Lin Xiayu's tweets saying "Her accent is the most interesting thing about you."

{{< figure src="/image/heaprunner_replies.png" >}}

That's some suspicious behaviour — this person could likely be the killer. Let's find some more information on heaprunner.

## Lucas Juillet

Since this user has a Mr. Robot profile picture (a hacker TV series), I decided to look up the `heaprunner` username on GitHub.

That's a good hit:

{{< figure src="/image/heaprunner_github.png" >}}

There is also an interesting repository named `pattern analysis`. It is supposed to analyse runner patterns to determine their location.

After checking the `.py` files, there is one useful line in `pattern_model.py`:

```python
class PatternModel:
    def __init__(self):
        self.history = []
        self.threshold = 0.87
        self.runner = "lulu kicourt"
        self.active = True
```

In the `self.runner` attribute, we find "lulu kicourt" — in French, this roughly means "lulu is running." This could be a Strava username.

## Strava Account

When searching for this username on Strava, we actually find someone who runs in China:

{{< figure src="/image/strava.png" >}}

Zooming in on the picture, her run ends in a park — Zhao Park...

{{< figure src="/image/park.png" >}}

I then tried to find the exact location on Google Maps:

{{< figure src="/image/gmap.png" >}}

Looking at the photos of Zhao Park — which is in fact Zhaoyuan Park — we find the exact image that was shown at the beginning of the challenge:

{{< figure src="/image/gmapfoto.png" >}}

MCTF{Zhaoyuan_Park}