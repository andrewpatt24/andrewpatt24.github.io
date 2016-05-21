---
layout: post
title:  "Automating Mongodb databases with Mongolab and Google's app engine"
date:   2014-10-17 12:37:01 +0100
categories: Mongodb Python python BeautifulSoup
---
After <a href="http://www.dinnerwithdata.com/picking-a-fantasy-football-team-using-optimization/" title="Picking a fantasy football team using optimisation" target="_blank">optimising fantasy football teams</a> in one of my previous posts, I wanted to take this idea further, and see if I can find an automated approach for substitutions and team selections through the season.

Before I do that however, I realised I needed to find a way to automate collecting the fantasy points data. While looking at Simon Rapers post on <a href="http://www.coppelia.io/" title="coppelia.io" target="_blank">Coppelia.io</a>, and how he used the <a href="https://cloud.google.com/appengine/" title="Google App Engine" target="_blank">Google app engine platform</a> for his new host of his twitterbot <a href="http://www.coppelia.io/a-new-home-for-pifreak/" title="A new home for PiFreak" target="_blank">PiFreak</a>, I thought this may be a useful tool to automate the data collection.

As I hadn't used Mongodb before, I decided that that would be my prefered storage mechanism. <a href="https://mongolab.com/" title="MongoLab" target="_blank">Mongolab</a> provide a sandbox account for free, and since the data was quite small, I would never reach its limit during the season. This meant I could use the free tier on both mongolab and the google app engine, which is obviously a bonus!

I won't go through the basics of either Mongodb/Mongolab or the Google App Engine here, as both sites have great introductions as to how they work. If you want to access a mongodb database through python, pymongo is a great resource, its unfortunate I wasn't able to use it here but I will be using it extensively to access the data for analysis.

To get the app running, I have it scheduled to run at 10:00am every morning using the cron.yaml script. The code pulls in a small dataset of gameweek start dates, and checks whether today is the day before that (so I have time to do anything with it before the transfer deadline!). If this is correct it then pulls the data out using <a href="http://www.dinnerwithdata.com/using-beautifulsoup4-for-python/" title="Using Beautifulsoup4 in python" target="_blank">beautiful soup</a>, and uploads the set of player dictionaries to my Mongodb database hosted on Mongolab.

Below is the python script I use to do the check, and if successful, scrape the data and upload it to mongolab using there API:

<script src="https://gist.github.com/andrewpatt24/5ccf0a44a9bb71b21c9f.js"></script>

If you want to see the whole app, you can visit it on my github repository <a href="https://github.com/andrewpatt24/getFootyData" title="getFootyData" target="_blank">here</a>.

Doing this process, here are the things I learnt:

<ol>
	<li>Unfortunately the Google app engine doesn't support mongodb so you cant use things like pymongo to interact with your database. This is where Mongolab comes to the rescue with its REST API. Without this it would not have been possible to host the app on google app engine.</li>
	<li>You have to be careful with building your app on a windows machine. Pulling in external data from a text file in python has the usual <code>\n</code> and <code>\r\n</code> issues. For a while my app was failing just because of this difference since (I can assume) the google app engine is linux based. I have run into this a number of times as I use a mac at home, but a PC at work. This handy <a href="http://stackoverflow.com/questions/4599936/handling-r-n-vs-n-newlines-in-python-on-mac-vs-windows" target="_blank">stackoverflow answer</a> sums up nicely the problem and how to deal with it</li>
	<li>I'm still properly understanding the best structure to use my on my mongo database so things may change slightly once I grab more and more data. However it will all still be relatively small so I don't think I'm going to need to worry.</li>
</ol>

The next steps I think for this project is to start developing the optimizer in python. The code in the previous post is obviously in R so I would struggle to automate it effectively. Once that is done I may create another app (its all free!) to automate this procedure. If only the Premier league site had an API for controlling my team too......

