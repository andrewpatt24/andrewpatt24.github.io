---
layout: post
title:  "Shiny applications hosting on shinyapps.io"
date:   2014-08-12 09:41:01 +0100
categories: R Shiny
---
I've thought shiny apps were a good idea for a while now. For anyone who doesn't know, Shiny is a package in Rstudio to create Web applications powered by R (you can find out more <a title="Shiny for rstudio" href="http://shiny.rstudio.com/" target="_blank">here</a>). Locally, these can be good ways to graphically explore data. To share them though, they need a dedicated shiny server. Given this fact, I've been torn as to whether they are actually worth it, as it may just be easier to create aweb application from scratch using html/css/javascript and some of the popular scripts like d3.js.

However, a few weeks ago I came across the new platform (as a service) from rstudio to easily implement shiny servers at <a title="shinyapps.io" href="https://www.shinyapps.io" target="_blank">shinyapps.io</a>.

This means that sharing any work developed in R has been made a lot easier.

I'm not going to go through how to make a shiny application here, but there is a fantastic <a title="Shiny tutorial" href="http://shiny.rstudio.com/tutorial/" target="_blank">walkthrough</a> on the rstudio site. Below I will just go over a shiny app I created and how I implemented it on the shinyapps.io server.

However you can find all of my code in my Shiny application <a title="Shiny Profiling app" href="https://github.com/andrewpatt24/simple-shiny-app" target="_blank">repository</a>.

I originally created a web application to graphically represent categorical variables against a Flag, so as to understand the differences across an explanatory variable. For the showcase here, I have included a subset of the variables from the Titanic survival data.
<h2>Notes on building a shiny app and using shinyapps.io</h2>
You can find a step by step guide of how to create the shiny app on the server <a title="Getting started with shinyapps.io" href="http://shiny.rstudio.com/articles/shinyapps.html" target="_blank">here</a>. However below a few tips I found on my way:
<ol>
	<li>The basic packages you will need are "Shiny", along with "shinyapps" and "scrypt" which you will need to install from github using "devtools":<code>
install.packages("Shiny")
install.packages("ggplot2")
install.packages("reshape2")
install.packages("devtools")
install_github('rCharts','ramnathv')
install_github('rstudio/shinyapps')
install_github('rstudio/rscrypt')
</code>Note I have also included ggplot2, reshape2 and rCharts here, the latter two used in the shiny app. These are pretty standard at the top of any of my scripts, and if you are not using them then check them out. In particular, The Shiny application has used rCharts, which is my go-to graphical package for R if I'm going to be showing anyone else my work, leveraging d3 - it's really easy to use and looks very slick.</li>
	<li>Put a meaningful name to your account. This sounds silly but I ended up putting my name there which was little silly, as it is now on every webpage I now create.If you are using it for a specific project, the name is included in the web page for each shiny app so best to keep it meaningful.</li>
	<li>Make sure you set your working directory straight away using <code>setwd("")</code>. I don't normally use this as I think its bad practice if you are working with lots of different files, but here it makes alot of sense.</li>
	<li>The compilation of the app on the server works differently if you have built your app on a windows system. It took me a while but I found that if you are referencing any of the libraries, then they need to be in lower-case (Apart from when you actually load it in using <code>library()</code>). There may be a few more similar issues but that was the only one I ran into.</li>
</ol>
Finally, check out my shiny application at <a href="https://apatterson.shinyapps.io/profile-app/" target="_blank">https://apatterson.shinyapps.io/profile-app/</a>. You can also easily add in a username/password for your app, although I'm not overly sure how secure it would be so probably using it for sensitive data isn't the best idea. Also, at least in the alpha version, This isn't the smoothest of operations and has caused some errors when loading the app.

Below is a small iframe of the app, however use the link above to see the full size page.

<iframe src="https://apatterson.shinyapps.io/profile-app/" height="800px" width="800px"></iframe>

Also, if you like the app and want to use it, fork it on github <a title="simple-shiny-app" href="https://github.com/andrewpatt24/simple-shiny-app" target="_blank">here</a>.
<h2>Final Verdict</h2>
While shinyapps.io has made it alot easier to share your shiny application, I still can't help feeling it may be easier, and more versatile, to produce it in pure html/js. That way it can be shared with people who do not have access to R, and you are not limited only to what shiny has to offer. If however you are firmly entrenched is using R, and want to keep it that way, Rstudio's new "platform as a service" has defiantely made that alot easier. It's still in alpha at the moment but I expect it to get better, and easier, as it develops.