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

Author : Evix

## Context

Chinese student who had been living in France for a year to study French recently returned to her home country for a short one-week vacation. A few days later, her body was found lifeless in a park in China. Local authorities are struggling to understand what happened to her. Before her disappearance, the young woman was very active on social media. Your mission is to retrace her steps and discover which park her body was found in.

The victim’s name was Lin Xiayu, and she used the username```LinXiayu35170``` on social media.

Flag format: MCTF{Xxxx_Xxxx} 

{{< figure
    src="/image/montagne.png"
 >}}

## Social Media search

Our first mission is to find the socials account of miss Lin Xiayu. We can proceed to input the ```LinXiayu35170``` username on [WhatsMyName](https://whatsmyname.app/).

Nothing relevent so I decide to check on mainstreams socials like Facebook, Instagram, Twitter...

We found 2 accounts on [Facebook](https://www.facebook.com/people/Lin-Xiayu/pfbid02hJB4m3ebYNpbp1erRUedrV8q4X7ZV2GEL63f5mSn3yPhjwJwrJqJ19zcWY5SbkGpl/?locale=fr_FR) and [Twitter](https://x.com/LinXiayu35170)

## Facebook

On the Facebok account we can find an Image 

{{< figure
	src="/image/ville.jpg" 
>}}

## Twitter

On the Twitter account we can find some tweets saying that she loves croissant ! I can understand I love them too vive la France !

{{< figure 
	src="/image/xiatwitter.png" 
>}}

More interesting, if we check on the followers we can find a strange account @heaprunner named Lucas Juillet that says weird things like "We think that we are sometimes alone, but we never really are"

{{< figure 
	src="/image/heaprunner.png" 
>}}

He even replied to one of Lin Xiayu tweets saying that "Her accent is the most intersting thing about you"

{{< figure src="/image/heaprunner_replies.png" >}}

That's some weird behaviour, this person can likely be the killer, let's find some information on this heaprunner

## Lucas Juillet

Since this guy have a Mr. Robot profile picture (an hacker series), I decide to check ```heaprunner``` username on github

That's a nice hit

{{< figure src="/image/heaprunner_github.png" >}}

Plus, there is an interesting repo named ```pattern analysis```

Basically, it's supposed to analyse runner pattern's to get their location

After checking the .py files there is on useful line on the ```pattern_model.py```

```
class PatternModel:
    def __init__(self):
        self.history = []
        self.threshold = 0.87
        self.runner = "lulu kicourt"
        self.active = True
```

On the self.runner attribute, we find "lulu kicourt", in french : lulu isrunning

This is possibly a Strava account

## Strava account

When checking the username, we actually find someone who runs in China

{{< figure src="/image/strava.png" >}}

Zomming on the picture, her run end in a park, Zhao Park...

{{< figure src="/image/park.png" >}}

I tried to find the exact location on Google Map

{{< figure src="/image/gmap.png" >}}

Looking on the Zhao Park photos wich is in reality Zhaoyuan Park we find the exact image that were in the beginning

{{< figure src="/image/gmapfoto.png" >}}

MCTF{Zhaoyuan_Park}








