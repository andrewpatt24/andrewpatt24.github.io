---
layout: post
title:  "Using BeautifulSoup4 in Python"
date:   2014-07-08 15:50:01 +0100
categories: Python python BeautifulSoup
---
I have been keen to have a go at webscraping for a while now. I've also wanted to do some visualisations on the vast online libraries of recipes across the web. I have been visiting <a title="Open Source Food" href="http://www.opensourcefood.com" target="_blank">Open Source Food</a> a lot for new recipes, and the fact that most of the recipes are tagged means there is a perfect link between all the recipes on the website. They use a wordcloud on the website for the visualisation but I think there is so much more you can do with this data.

It took me about a week in small stints to learn the basics of Python (using the great tutorials on <a title="Code Academy" href="http://www.codecademy.com/" target="_blank">Code Academy</a>) and about a morning to get Beautiful soup working how I wanted it. I am looking forward to creating some cool visualisations using either this data or an embellished set I may re-scrape, but for now I thought I'd share the code I used:

<script src="https://gist.github.com/andrewpatt24/77254fba1d6e0db66d23.js"></script>

NB: I know web-scraping is somewhat of a touchy subject among some people. The recipes on <a title="Open Source Food" href="http://www.opensourcefood.com" target="_blank">Open Source Food</a> are held under a <a title="Creative Commons" href="http://creativecommons.org/licenses/by-sa/3.0/" target="_blank">creative commons</a> licence, and I have made sure to pull the recipe owners names and web address so they can be fully documented in any visualisations which directly refer to any of the recipes.
