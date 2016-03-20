---

layout: layout
title: "Watch apps worth using: customer's perspective"
author: Krzysztof Siejkowski
date: 2016-03-21
keywords: watch, watchOS, apple watch, watch apps, wearables, wearables apps 
description: What kind of apps are the most usable on Apple Watch? Some thoughts from everyday user.

---
# [{{ page.title }}]({{ page.url }}) 

Last month [underscore David Smith](https://twitter.com/_DavidSmith) of [Developing Perspective](http://developingperspective.com) and [Under the radar](https://www.relay.fm/radar) fame, author of [many iOS apps](https://david-smith.org/apps/), shared the [lessons learned from shipping eleven Apple Watch apps](https://david-smith.org/blog/2016/02/12/watch-apps-worth-making/). First of all --- what an impressive accomplishment! Congratulations! David's work and attitude has provided an inspiration for me and many others for a long time. His explanation of what types of apps for the Apple Watch are worth making is backed by the year of experience and the actual sales data. [A highly recommended, valuable read](https://david-smith.org/blog/2016/02/12/watch-apps-worth-making/).

At the same time a number of friends had asked me about the Apple Watch. Is it usable? Is it worth the price? Are there any useful features? Are there any killer apps? To answer these questions, I've tried to become more self-aware about how I use the Apple product. I've been wearing it everyday since May 2015 and, being an iOS developer myself, I've checked a number of different watchOS apps to gather the anecdotical material for the analysis. Eventually it's led me to following insights concerning what kind of watch apps makes sense to use in the present software/hardware cycle from the customer's perspective.

_Disclaimer: I haven't shipped any apps for Apple Watch, so please take the analysis with a grain of salt. It's entirely based on my personal experience mixed with data from various sources (like friends, podcasts and blog posts). Your mileage may vary._

## Apple Watch works nicely with _app-independent_ activities

There's no better way to start than with an ending, so let's jump directly to the conclusion:

**The most usable Apple Watch apps control and monitor _app-independent_ activities. The apps that are based on _app-dependent_ activities are way less compelling to use.**

Sure, but what does it mean for an activity to be app-dependent or app-independent?

*App-dependent* means it starts when the user opens the app and stops when the user closes it. It simply doesn't make sense outside that window. Not only does it require user's attention, but usually the attention is its crucial part. The clue is in the name: its existence **depends on app being launched**. Some examples include:

* **Reading**: there's no "background reading". You either read or don't read. When you're not actively looking at the words, the process doesn't take place.
* **Playing games**: if you're not playing the game, the game doesn't play itself --- at least as long as the waiting periods aren't part of the game.
* **Writing**: everyone who has ever had to meet any writing deadline knows that truth by heart: the words do not write themselves, no matter how much you'd like to. Neither code nor blog posts.
* **Watching a video**.
* **Browsing the web**.
* **Checking a Twitter or Facebook timeline**.

The general principle is: the activity is *app-dependent* if it's happening only when the user is actively using the app.

## What is an _app-independent_ activity?

The *app-independent* activity doesn't require user's attention to proceed. It happens regardless of whether the app is launched or not. It might need to be started and stopped by the user, but between these two point, it takes place in the background as happily as in the foreground. The examples include:

* **Accumulation of the articles in the RSS reader**: new articles are appearing no matter if I look at them or not.
* **Time passing**: in general, it happens regardless of your attention. That's why the Pomodoro timers and time-trackers work so well at the Apple Watch.
* **Moving in space**: the train you're on keep going even when the Maps app is closed.
* **Waiting for the resources to be available in a game**. 
* **Addition of new tweets or Facebook posts to a timeline**.
* **Getting the fitness data**.
* **Sleep tracking**.

Please notice the subtle, but crucial difference. From the app's perspective the activity might require being in the foreground, even if only for a quick while. Synchronizing with web server via iOS background updates, fetching new data in response to a push notification, getting a location change callback --- none of those happen without executing the app's code. However, from the user's perspective, those activities don't need any attention.

Also, please notice that a particular service often contains both *app-dependent* and *app-independent* activities. The post on your Facebook wall will appear when your friend adds it, even if you don't have the Facebook client open. You won't read it, however, until you launch the app.

Now the following should make more sense: **Apple Watch apps that are most usable control and monitor _app-independent_ activities. The apps that are based on _app-dependent_ activities are way less compelling to use.** But why is that a case?

## Why is the Apple Watch much worse for _app-dependent_ activities?

I blame it on the mix of the watch interface design and the early hardware cycle. 

Concerning the interface, being worn on a wrist is a decisive factor. You cannot comfortable keep your arm in an interaction-ready position for a long time. The screen is only as huge as the human biology lets it, which is way too small for many activities that require user attention. It makes the extended periods of usage uncomfortable.

The early stage of hardware cycle doesn't help. Technological limitations makes most of the apps (even built for watchOS 2) delegate tasks to the phone and then display the results. The Bluetooth connection is unreliable, so too often the user input (like pressing a button) either silently fails to register or fails to bring any results. The attention is distracted, the _app-dependent_ activity gets broken.

Is it an inherent characteristic of a watch form? I don't think so. Multiple solutions come to mind. One is voice interface. Siri is not good enough to be a primary way of interaction, but this might change. Also, watch could project the screen on a surface or have the air gesture recognition system. For now, however, there's not much we can do.

## Why is the Apple Watch way better for _app-independent_ activities?

The analog wrist watch was designed to visualize the passage of time, which happens regardless (even against) your attention. It's no surprise that its digital incarnation inherits all the features that make it well suited for _app-independent_ activities.

For one, you can **monitor** their current state. The current time, the current score in a football game, the current heart rate, the current song that's playing. Whether you're using complications, glances or a full app, checking the current state is a breeze. Just put your arm up, look for a second, put your arm down.

Another option is to **be notified** about the change in the activity state. Pomodoro flow has moved from work to break time? Notification. Your Facebook friends list has grown? Notification. There're new resources available in a game? Notification. You can get proactive with it, too. You should leave home to get a bus to work? Notification. You should turn right on the next lights? Notification. Again: either take a second to look at the change or even get it just from the sound or haptic pattern.

Yet another option is to **control** the activity. Pausing and playing music. Starting and stopping the sleep tracking. Ticking off the task from your daily tasks list. As long as the interaction required to control the activity is short,the convenience of performing it on the watch wins.

The present generation Apple Watch limitations hurt _app-independent_ activities as well. Let's say you've just checked the score in a game. Your arm goes down, the screen goes black, the app goes to suspension. By the time you are back there's a good chance that the displayed data is outdated. Sometimes it synchronizes after few seconds, sometimes it gets stuck forever. Some notifications never make it to the watch. Those limitations, however, are likely to be fixed in upcoming software and hardware releases. They are not inherent to the watch form.

## How does it compare with David Smiths's insights?

David has identified three kinds of app that are worth making for the Apple Watch:

* Notifications.
* Complications.
* Sensors.

It corresponds nicely with the idea of _app-independent_ activities. Notifications are designed to present either a change in some process' state (like a new episode of your favorite podcast) or a snapshot of it, possibly with a call to action (like a reminder to log your breakfast to calorie counter you've been using for the past month). Complications show you the state at the moment. Sensors are all about tracking what's happening with your body in the background. The three categories allow **monitoring**, **being notified** and **control** of an _app-independent_ activity.

## How to apply the idea to real-world examples?

Imagine you're building a Twitter client for iOS with a watchOS component. What should the watch app include? 

Let's identify some _app-independent_ activities for Twitter: followers count, tweets on the timeline, private messages, how many retweets you've got today, number of hearts your last tweet received. That's a solid base for your watch app features. Whether you want to provide the monitoring, notifying or control is up to you.

Moving to the _app-dependent_ activities: reading tweets, writing tweets, changing you bio, uploading new profile picture. These are all way less convenient on the watch. You can provide the features for them, but I wouldn't hold my breath to see a widespread usage.

What about retweeting other people tweets? Well, decide by yourself! Retweeting can be either _app-dependent_ (if it requires a long interaction) or _app-independent_ (if it's more like pausing a song, e.g., pushing a button on an interactive notification). The app design influences what category the particular activity belongs to. It can also vary from user to user, as it always does when it comes to UX.

I hope you'll find the above analysis useful. However, there's a lot more to the app's success than picking the right set of features for the watch. The idea of _app-dependency_ is just one way of looking at a hard process of designing. It might be helpful, but it definitely doesn't provide all the answers. Also, please see the disclaimer at the top.

I'm more than happy to discuss it or any other topic related to software development. Contact me [@\_siejkowski](https://twitter.com/_siejkowski). 

### {{ page.date | date: "%Y-%m-%d" }} 
