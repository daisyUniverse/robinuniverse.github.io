---
title: The long and Winding Road to creating TwitFix
subtitle: It always seems like it should be so simple
---

## Welcome to my Blog!

First off, I just want to say, hi! Welcome to my shiny new blog! I'm using [Beautiful Jekyll](https://beautifuljekyll.com/) hosted on my github. and I gotta say it's a pretty painless experience so far for something that looks Pretty Good, while letting you just throw new md files in a directory to make new posts without worrying about databases and all that. Neat.

Anyways, to the topic at hand, and the type of thing I wanted to make this blog for - Documenting my dumbass projects to show that even the dumbest bodge can result in useful tools and resources

and the first project under that spotlight will be TwitFix, I'll be going over a brief history of it's development, a breakdown of how it works, and it's adoption by a decent number of users

But first, let me briefly explain what it is exactly that TwitFix is meant to do

## What is TwitFix?

Discord, the popular chatting app, is my primary method of communication with most people these days, it has lots of really nice features, one specifically being that if any user sends a link to any site, the discord client will create an embed for that link, this allows for things like watching Youtube videos without opening the link in your browser, or, more relevant to this story, Twitter Video links could be posted, and the linked video could be played in the client. Operative word being *could*. At some point, this specific funtion was broken, and it was broken at the back-end level, This was terribly frustrating, tweets were flying, but for months they went largely ignored. TwitFix simply Fixes Twitter video embeds.

All the user experience of TwitFix is pasting a Twitter video link into your chat box, changing the `twitter.com` to `fxtwitter.com` and posting the new link, this points the request to my server, which instead of giving out a broken embed, returns a fixed embed that properly displays the video

Discord eventually did end up fixing embeds, and I briefly considered shutting the project down, but many people have come to agree that my implementation of the embed is more reliable than discords own implementation, so while traffic is slowed, many people still use TwitFix for embeds, and some of it's auxiliary features

---

## How did this come about, anyways?

I have been developing Discord Bots in python for a good long while, me and one of my best friends basically learned Python together this way, and she eventually got much better than me, and we eventually created a function of the bot that could do some fun stuff with videos, specifically running Youtube links through Youtube-DL and then uploading the resulting video download, this allowed for some fun stuff like downloading specific timestamps of a video and quickly generating a discord link to post in other servers. And the great thing about this command was that it was not particularly picky about what kind of link you fed into it, as YTDL can download from a ton of different sources, so for a long time we used this command to get around Twitter's broken embeds

One day, I realized, there has to be a better way to do this, so I started searching. I realized that if Youtube-DL can grab direct video links, what's stopping me? So I opened the Twitter video page in dev mode, and started poking around.

### Implementation 0

My first attempt actually was going along a very different path from what TwitFix eventually ended up being, which is to say it was a slightly modified version of that video function with the rest of the bot stripped away, The idea being that it would be a microbot that anyone could run on their PC, even now this idea has some minor advantages to the centralized method I ended up using ( You could very practically download every video and reupload them! Discord treats uploaded videos much nicer than web embeded ones on All platforms ) , but I suspect that adoption for something like this would not have gotten anywhere near where the final TwitFix has reached, for exactly one reason: People would have to do something to set it up first for each server, meaning it would only appeal to a very small set of people ( those who know how to set up a bot, and own a server to add it to )

I saw this early on, and realized that it may be possible to move this functionality to a server. After working with Codie on MineOnline, I got a very small amount of time interacting with Flask, and it intrigued me, because it's the only time I've ever been able to use my Python scripting brain for Web Development, so taking that, I started researching Flask

### Implementation 1

The second attempt was simply to do some web scraping to pull the direct MP4 URL from a given url, but as anyone who has ever done any web scraping knows, it is quite a fiddly business. I got this implementation working, but just barely. it was slow, unreliable, and didn't work for many videos. I very quickly got frustrated with this approach, and decided to step back and take another look

### Implementation 2

This time around I was thinking about how YTDL worked so well. Going in, I knew that downloading each video wholesale wasn't going to be practical, even at the scale at which I was working on at the time, which was purely among my moderately sized friend group, I realized that by the time the video was downloaded, discords built in embed caching system would have passed over the link long ago. So when researching, I found that ytdl has a function to return an info json, containing almost all the data that the Twitter API can pull about a twitter video, and it had everything I needed to grab the direct MP4 link, all I had to do was take all that info, slap it into a page, and post that as fast as possible. And this worked! This was the first time I got TwitFix working reliably, though, I quickly noticed that I was desperately in need of some kind of cache system, as every single time the link was viewed by anyone on discord, it was hitting my server, and my server was spinning up YTDL and hitting twitters servers, even with a few people, that quickly added up to hitting Twitter over a dozen times for One link. No good.

### Implementation 2.1

I quickly slapped together a json object to store video links, descriptions, titles, and anything else I could think of, and saved those to a local json file. at this point, I considered TwitFix feature complete. I had a GitHub repo, and I was using in front of friends, and even before I started tweeting about it, it started picking up traction, and I quickly realized many problems

1. Youtube-DL uses a Guest Token to use the Twitter API - This started to give me some issues as it started to get hit more than 100 time per day
2. I was running this as a single script, still using the built in debug flask server software. I knew this would not scale well
3. The embeds at this point were very bare. The basic embed method using OpenGraph tags, as it turns out, is implemented pretty poorly in the Discord client, meaning that really, the only text you can include is a single field, which I tried to use for the tweets text, but as it had a very small character limit, it wasn't great
4. I realized that running this script in threads, would lead to issues with the json caching system. Files don't really like being read and written from multiple places at the same time

I started off with what looked like the easiest problem, the cache. In the end I decided to use mongoDB, this allowed me to keep the cache offsite, and synced between many servers, with the only downside to this being that the mongoDB atlas free account I was using only allowed for 512mb, but once I started running, I realized it wouldn't be too much of a concern, as that equates to about 1 Million unique cached urls. I still use this system, and I have had to delete the data twice now, meaning we are up to about 2.5 Million unique cached links

Up next: solve the embed problem. This was a Huge Pain In The Ass. I will probably write an entire blog post about the hours of experimentation I did with discord embeds and their structure, but for now, lets just accept that I fixed it

to make a VERY long story short, I had to implement an oEmbed endpoint in order to serve those kinds of embeds, which allowed me a bit more wiggle room with video embeds

### Implementation 3

At this point, code was starting to get messy, things were all over the place from experimentation, but importantly, it was working, it was at this point that I created an actual tweet advertising the service, and it started to get a response that I could not have expected, if it followed this trend, it would reach over 1000 requests a day, and with the code in the state it was in, I doubted that it would be able to handle the traffic. At this point, I decided that I had two options, either continue to refine the script, or completely start over and refactor the majority of the code before the morning hit and more traffic started coming in. 

And so I began to rewrite TwitFix. I don't remember much of it as it was mostly a blur of energy drinks and loud music, but in the end I had something Much Better, I made everything follow a clear line of functions, I did more work defining the "VNF" ( Video iNFo ) object, and started to work on implementing a different source for video links - The actual Twitter API

After a lot experimentation, I got it working. It was pulling tweets from my Twitter API key, and I wasn't hitting any rate limits. It was finally ready to be hit much harder than I ever hoped it would be, and almost as if on cue, my initial tweet got a ton more traction, and it was now a race to get my refactored code live ready in time, which I did!

---

From this point on, much of the development that happened with TwitFix were relatively minor changes, adding more quality of life features, getting the server settings just right, getting an even better domain name sorted out, and at one point, moving it over to a much higher end VPS, but those stories can wait for another blog post

Hope y'all enjoyed! I hope to keep writing entries about some of my other projects, and I'll also release news about TwitFix's progress from this point on here, so stick around if you're into that sort of thing! I also plan on releasing a more technical document showing exactly how TwitFix operates currently
