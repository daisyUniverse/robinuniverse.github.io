---
title: Goodbye, TwitFix!
subtitle: So long, and thanks, for all the fish!
---

# Goodbye, TwitFix!

Hello! You're probably here because you heard there was some kind of nefarious going's on's with TwitFix, there is quite a lot of misinformation being spread about this, so let me take this time to quickly catch us up with the what's happened

Before we move on, let me just get this TL;DR out of the way:
**No, TwitFix never took users' data**

I kept a list of all tweets ever passed through the service, and I could roughly see how much it was embedded (hits)
This data was always publicly available, and has been for months via the /latest/ endpoint, and it was necessary to keep this list
in order for TwitFix to function effectively at the scale it was at ( By the time of shut down, around 100k uses per day, and 20k new links cached per day )

This controversy is over the fact that I said that I would scroll through /latest/ from time to time, and I would report posts that I saw on there that clearly violated Twitter's TOS to Twitter ( Specifically, clearly lolicon art, and text / image posts that contained clear transphobia )
I probably did this to about half a dozen posts in total, which makes what is coming up all the more depressing.

## What Happned?

### STAGE 00 - MOTDs

About a week or so ago, I started adding a message of the day to TwitFix, it wasn't much, it was usually just some little quote or some joke I thought of that day, and it would link to random funny videos or websites that I thought related to the message - Most people seemed to not mind, it's my service, and I can do what I like with it after all, and it didn't detract any usage from embeds, so I figured there was no harm in it

I started getting the first bits of hate messages when I changed it to say "Trans Rights are Human Rights!", which if you know me, you know that means quite a lot to me! But this seemed to be the line for a lot of people, for some reason. I fear this is where I started to appear on the radar on a few people.
I have to admit, after this point I did start to take the piss a little bit, partially because it made weirdos mad, and partially because I hoped someone would finally fork TwitFix so I could finally move on to something else in my life

### STAGE 01 - BetterTwitFix

When I woke up and saw that someone had finally fulfilled my wish and forked a live instance of TwitFix, I thought that was a cause for celebration, I tweeted out an announcement, I added a link to the repo as my MOTD, I even had an extra cup of coffeee. There were a few changes to the code that I thought were a little strange, namely the fact that they removed the code responsible for all TwitFix API endpoints, and strangely enough, the stat tracking functionality

(The stat tracking data was basically a daily count of how many times embeds were served and how many new links were cached, an example of that data follows:)

```json
{
    "_id":{"$oid":"628042804486395c6f4b26de"},
    "date":"2022-05-15",
    "embeds":{"$numberInt":"251380"},
    "linksCached":{"$numberInt":"39554"},
    "api":{"$numberInt":"7547"},
    "downloads":{"$numberInt":"73"}
}
```

This came off as kinda weird to me, so I started to ask the creator about this on Twitter, which led to the tweet that started this whole thing:

### STAGE 02 - PRIVACY CONCERNS

![tweet](https://pbs.twimg.com/media/FS02P_zX0AIdXQh?format=jpg&name=medium)

► "The only thing I can think of is that I scroll through /latest/ and report transphobes and L0L1 posters, and you want to prevent something like that"

This was screenshotted and reposted by a user named @Kyonko802, whose [post](https://twitter.com/Kyonko802/status/1525904328552157184) then gained traction, causing it to be seen by every single person who did not like the fact that I would dare to scroll through the public list of tweets, that I have made available to everyone for 5+ months, and have regularly reported progress in the development of, where no one has ever sent me any kind of complaint about before now

This was now being spun into a privacy concern, at first, the misunderstandings were pretty normal, to be fair, I did not specify how often I did this, so many people were assuming that I literally scrolled through every single tweet that, of the 100k+ that pass through my server daily, and I report anything I don't like
Let me just do a lightning round of the misconceptions I've seen, roughly in the order that I saw them appear

MISCONCEPTIONS/RUMORS

► *"I look at every single tweet that is used, and I report hundreds of tweets including all porn, because I'm a prude"*

Literally a large chunk of my audience I had was from adult audiences, which I specifically catered features to ( a huge amount of work went into making fxtwt work with posts other than videos specifically to allow for adult audiences to bypass the broken embeds specific to them )

► *"I somehow know who posts what, and when I see things I don't like posted, I report that user"*

No, just, no. something like that isn't even possible ( for reasons I will get into later in the technical section of this blog post )

► *"I am making FBI or some other kind of legal report"*

Nope! Just to Twitter! I don't have nearly enough useful data to do something like that even if I thought it was warranted

► *"TwitFix was logging the messages and DMS of the servers of everyone that used it"*

This is just ridiculous and unhinged but I did see it more than once, TwitFix isn't a Discord bot that sits in every server and listens to every word you say, that's just, not how the internet works. When TwitFix is used, the only thing I can see is the IP of the Discord Embed Scraper Robot that grabs the post content, these robots are extremely generic and cannot even be used to identify a region much less a server

Since this went live, I have received a basically non stop stream of hate messages, some violent, to the point where I have had to take my personal Twitter account private, and when they couldn't get to me, they started attacking my girlfriend as well.
The last few days have been complete hell for me. I'm a fairly non confrontational person, I'm just a nerd who likes to make stuff. I didn't sign up for all of this

So I made the decision to finally drop the TwitFix project - this was a long time coming, to be fair, if you have read my previous post, you'd know I was already on the fence about it, but as I've been telling people, this was the boulder that broke the camel's back. I made the embed show a goodbye message, and I stepped away

## The Technical Part

Ok, I'm now going to explain, in enough detail that I think is followable, just how TwitFix worked, start to finish, just so I can possibly ease some fears that people may still have. I will represent the function of TwitFix on a pipeline, from Discord, to my server, back to Discord, showing each major step of data processing necessary.

### DISCORD

The request starts here, when you change the link from Twitter.com to fxTwitter.com, it tells your client to ask Discord's server for an embed that represents that link, if one is already on file for that particular link, it will pull that internally, and never reach out to my server, the exact mechanics of this process are not clear to me as they are part of Discord internally, but basically it's simple enough to think of it as Discord grabbing that embed data from a bucket if it happens to have one nearby. If the data is not available to your client, it will ask Discord to make a web request on your behalf to grab all the data for you, at which point it will add that data to the 'bucket'
This is the part where my server comes into play

### MY SERVER

**Request**

I will get the request from Discord, specifically, I receive the request from the IP of one of many different Discord Embed Scraper Bots, which is the component of Discord that is responsible for gathering all of the data that goes into the embed, and also assures that there is no direct contact between my server, and the user.
All of the data I need from the request is contained within the text of the url, which I can use to determine which tweet it is this request is looking for

**Twitter API**

This is the part where though the process of regex magic, I find out which specific tweet the user wants, I will then ask my database if I've already gotten data for this tweet before, and if I haven't, I'll proceed to ask Twitter for all of the data for that particular tweet

**The Tweet Cache**

At this point, I will do some processing to find out some basic info about the tweet, as each type of tweet requires they be stored in their own way, as a **VNF** ( This stood for Video iNFo, but after I added support for non video tweets, it just stuck )
Here is a full example of a typical VNF, this is literally all of the data, besides the stats, that I retain

```json
{
    "_id":{"$oid":"627aa773b88f36e89d112595"},       "This is an internal mongoDB id that allows me to index the database"
    "tweet":"https://twitter.com/i/status/1523651859046866946","This is a link to the original tweet"
    "url":"https://video.twimg.com/ext_tw_video/1523651602355298311/pu/vid/1280x720/YwpKtfTk_zsMv8yf.mp4?tag=12","This is the direct MP4 URL to the contained video"
    "description":"this is one of the funniest things that I guess you can just do in a kids anime https://t.co/sULkndVZpJ","This is the tweet text"
    "thumbnail":"http://pbs.twimg.com/ext_tw_video_thumb/1523651602355298311/pu/img/SPmRG7TmmRUI-ED1.jpg","This is the video thumbnail"
    "uploader":"Jes",                              "This is the uploader's display name"
    "screen_name":"typeofsun",                     "This is the uploader's screen name"
    "pfp":"http://pbs.twimg.com/profile_images/1503165618861735939/LKNSF1dd_normal.jpg","This is the uploader's profile picture"
    "type":"Video",                                "This is an internal value used to track what type of VNF it is for embed processing"
    "images":["","","","",""],                     "If any images are contained in a post, they are kept in this array"
    "hits":{"$numberInt":"2"},                     "This is an extremely rough counter for how many times a given tweet is pulled from the cache. Only very vaguely representative of how many servers the tweet is seen in"
    "likes":{"$numberInt":"1409"},                 "Likes count at the time of API call"
    "rts":{"$numberInt":"571"},                    "Ditto for rts"
    "time":"Mon May 09 13:11:25 +0000 2022",       "Date the tweet was created"
    "qrt":{},                                      "If a post is a QRT, it will contain a QRT object here, which is like a stripped down VNF"
    "nsfw":false                                   "This is the results from the possibly_sensitive"
}
```

This is the full and complete data that is added to a big dumb pile in my database, which I then call from, this ensures that no matter what, no matter how many users I have, I only ever hit Twitters API once per tweet, this is the only thing that makes TwitFix as a service even halfway viable for someone like me working at the scale I was working at.

**Building the Embed**

Depending on the type of tweet, embeds are pretty drastically different, so it is necessary for me to dynamically interpret the tweet content to build the embed from a number of render templates that I use
This process is fairly complex, and I don't intend for this blog post to be a full deep dive into the technical aspects of the process, as much as it is just to teach people how the parts that relate to this issue work, and embed building is fairly inconsequential in this context, so for now, we'll call it a black box, you put a tweet in, and a fresh html file comes out that represents all the data that Discord needs to build the embed for the user, which is then returned to Discord

And then the process is done, Discord shows the user the data, and then they are free to use the improved embed however they choose, and this process happened on my server about 100,000 times a day, 20% of which were new links that had to be grabbed, processed, and added to the database

I know this post is a bit of a chonker, but I'm hoping that giving as clear as an explaination that I can will squash some of the wild speculation going around. Thank you for reading this, and I'm sorry that I had to let you all down by giving up on the project
I hope you all know that all I ever wanted to do was make the best service I could, and never at any point did I want to break the trust of any of my users - I was clearly announcing every part of the /latest/ part project, and I always made sure to keep the readme updated with its functions

I’d like to very sincerely thank everyone who has shown me support - y’all have made this whole thing a lot more bearable, and my DM requests on twitter are now more support than they are hate - I’m sorry for not responding, as I’ve been trying to avoid social media for my own brains sake, but I will thank every single one of you personally

Also - I’m going to be removing everyone from the montlhy contributions side of my Ko-Fi, as I will be shutting down the vast majority of my servers, server costs will no longer be an issue

I hope I see you all in the future, for me? who knows? an IRC server that has a channel for every site on the internet? More art? 3D and 2D? Maybe even a game, the story of a goat who restores an abandoned park, Whatever happens, I’ll see you space cowboy

<3