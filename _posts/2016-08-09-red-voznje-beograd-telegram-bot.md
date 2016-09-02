---
layout: post
title:  Red Voznje Telegram Bot
date:   2016-08-08 20:40:33
categories: blog software scripts github
tags: [beograd, bg, red, voznje, public, transport, autobus, bus, tramvaj, tram, trolley, trolejbus, minibus, schedule, telegram, bot, ]
---
**TL;DR** I created a Telegram bot to show buses and trains schedule for Belgrade. It is not perfect in any
way but it gets the job done. Code is open source.

![public transportation](/assets/publictransportation.jpg)

## Background
After company that i work for moved to new offices in [Science Technology Park](http://www.ntpark.rs/) (STP)
daily commute to and from the place became a bit of a problem for me. Only bus that goes to STP leaves every
45 minutes and it is one of the three, yes, three, buses that i have to connect in order to get home most
efficiently. Oh and working with team which is in a 7 hour difference time zone certainly doesn't help
with leaving at 5pm.

Yes, i can buy a car but I don't like to drive and it is always nicer to have someone drive
 you while you read, so there is that :)

### Problem
How do you know your bus schedules? Official Belgrade public transportation company has a completely useless
schedule. Imagine having to open each line's schedule by downloading a PDF file each in A4 size with only 
half of the page filled with data. To make matters even worse if they have 2 schedules (two directions)
they show the other direction on a separate page! Madness.

Luckily there is a third party service that processes these PDFs (creates separate files for separate
directions) and publish them (again) not as text but images. (_I swear i will make a service that will OCR
these into normal data some day_).  

Imagine having to visit that service and search for each line and then open the images. Or to have
bookmarked all the lines that you can potentially use. I am too lazy for that.

### Solution
Since i use [Telegram](https://telegram.org/) on a daily basis i decided to create a bot that would fetch me
this data (images sadly) without having to search for them on a website. 

**TADAAAH!**
[@BGRedVoznjeBot](http://telegram.me/BGRedVoznjeBot) Telegram bot was born, well, created.

Now i can type `/bus 53` and get Bus line 53 schedule.

Or you can search for `/train pancevacki` to get the schedule for BG trains starting on `Pancevacki most`
station.

![bus 53](/assets/telegrambot1.PNG)![minibus E2](/assets/telegrambot2.PNG)![nightbus 37N and train](/assets/telegrambot3.PNG)

For those of you that just want to use it or try it just click on the bot link above, and for those of you
that would like to maybe host this themselves you can easily do so by cloning 
[telegram-bot-redvoznje](https://github.com/asibin/telegram-bot-redvoznje) on GitHub.

Bot is currently hosted on my private machine while i search for a better hosting solution.
So you may expect some trouble from time to time. Please let me know if you run into any trouble using it.

