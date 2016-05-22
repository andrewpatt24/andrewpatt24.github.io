---
layout: post
title:  "Picking a fantasy football team using optimization"
date:   2014-08-12 09:41:01 +0100
categories: [R]
---
It's that time again in the year when all of my friends are putting together fantasy premier league competitions, and after failing numerous years with my distinct lack of football knowledge, I thought I would employ a strategy I am actually good at.

So below is documenting how I've shaped my team in Week 1. It is using a basic linear optimisation approach which you can read <a title="Wikipedia: Linear Programming" href="http://en.wikipedia.org/wiki/Linear_programming" target="_blank">here</a>. In short, the algorithm is maximising the amount of points I can get from a group of players given specified constraints.

These specified constraints to the current system, which have been built into my program, are:
<ul>
	<li>Squad Size: To join the game select a fantasy football squad of 15 players, consisting of 2 Goalkeepers, 5 Defenders, 5 Midfielders and 3 Forwards</li>
	<li>Total Spend: You have £100 million to construct your squad</li>
	<li>Team Quota: You can select up to 3 players from a single Barclays Premier League team.</li>
</ul>
&nbsp;

I have therefore used these constraints to build the function in R below:

<script src="https://gist.github.com/andrewpatt24/1b9c2a94ab1496be82b2.js"></script>


This has been built around the <a title="Player and Points list" href="http://fantasy.premierleague.com/player-list/" target="_blank">fantasy points data</a> which you can easily extract from the main website. There is a little bit of  editing you need to do so it is in the right format, but the code is quite simple and is below:

<script src="https://gist.github.com/andrewpatt24/926f33cabb62e216c067.js"></script>

The above code has said that I want to optimize my whole squad (All 15 players) with the full £100m. The output we gain from the above code is:

&nbsp;
<table style="border-collapse: collapse;width: 144pt" border="0" width="192" cellspacing="0" cellpadding="0"> <col style="width: 48pt" span="3" width="64" /> 
<tbody>
<tr style="height: 12.75pt">
<td style="height: 12.75pt;width: 48pt" width="64" height="17">Player</td>
<td style="width: 48pt" width="64">Points</td>
<td style="width: 48pt" width="64">Cost</td>
</tr>
<tr style="height: 12.75pt">
<td style="height: 12.75pt" height="17">Howard</td>
<td align="right">160</td>
<td align="right">5.5</td>
</tr>
<tr style="height: 12.75pt">
<td style="height: 12.75pt" height="17">Speroni</td>
<td align="right">144</td>
<td align="right">5</td>
</tr>
<tr style="height: 12.75pt">
<td style="height: 12.75pt" height="17">Coleman</td>
<td align="right">180</td>
<td align="right">7</td>
</tr>
<tr style="height: 12.75pt">
<td style="height: 12.75pt" height="17">Terry</td>
<td align="right">172</td>
<td align="right">6.5</td>
</tr>
<tr style="height: 12.75pt">
<td style="height: 12.75pt" height="17">Koscielny</td>
<td align="right">155</td>
<td align="right">6</td>
</tr>
<tr style="height: 12.75pt">
<td style="height: 12.75pt" height="17">Mertesacker</td>
<td align="right">157</td>
<td align="right">6</td>
</tr>
<tr style="height: 12.75pt">
<td style="height: 12.75pt" height="17">Fonte</td>
<td align="right">149</td>
<td align="right">5.5</td>
</tr>
<tr style="height: 12.75pt">
<td style="height: 12.75pt" height="17">Yaya Toure</td>
<td align="right">241</td>
<td align="right">11</td>
</tr>
<tr style="height: 12.75pt">
<td style="height: 12.75pt" height="17">Gerrard</td>
<td align="right">205</td>
<td align="right">9</td>
</tr>
<tr style="height: 12.75pt">
<td style="height: 12.75pt" height="17">Puncheon</td>
<td align="right">131</td>
<td align="right">6</td>
</tr>
<tr style="height: 12.75pt">
<td style="height: 12.75pt" height="17">Sidwell</td>
<td align="right">126</td>
<td align="right">5.5</td>
</tr>
<tr style="height: 12.75pt">
<td style="height: 12.75pt" height="17">Noble</td>
<td align="right">125</td>
<td align="right">5.5</td>
</tr>
<tr style="height: 12.75pt">
<td style="height: 12.75pt" height="17">Giroud</td>
<td align="right">187</td>
<td align="right">8.5</td>
</tr>
<tr style="height: 12.75pt">
<td style="height: 12.75pt" height="17">Lambert</td>
<td align="right">179</td>
<td align="right">7.5</td>
</tr>
<tr style="height: 12.75pt">
<td style="height: 12.75pt" height="17">Campbell</td>
<td align="right">108</td>
<td align="right">5.5</td>
</tr>
</tbody>
</table>
&nbsp;

This looks relatively correct, and tells us that last year if we were to keep this squad we could potentially have had ~1929 points (quickly calculating from the top 11 players in the squad). While that isn't great vs the best teams from last year, remember not all players would be playing well all season, so you may have to re-adjust teams based on form.

&nbsp;

Interestingly, that may still not be the best option to pursue. I have built the optimisation function so that it wont just pick the best 15, but will potentially pick the best 11 for you with a smaller pot of money. You can then pick smaller players to fill the bench afterwards.

&nbsp;

To do this correctly, we need to work out what is the maximum amount of money we can optimise over, while still having enough to fill the bench. To do this it is best to actually work backwards. What is the least amount of money we can spend on the bench players?

&nbsp;

I have done this with the following selections:

Pick 1 Goalkeeper, 1 Defender, 1 Midfield, 1 Attacker with £17m:

<code>Opt.squad(footy.data,cost=17,n.gk=1,n.df=1,n.md=1,n.fd=1)</code>

&nbsp;
<table style="border-collapse: collapse;width: 144pt" border="0" width="192" cellspacing="0" cellpadding="0"> <col style="width: 48pt" span="3" width="64" /> 
<tbody>
<tr style="height: 12.75pt">
<td style="height: 12.75pt;width: 48pt;border-left-width: initial;border-left-color: initial" width="64" height="17">Player</td>
<td id="origin" style="width: 48pt" width="64">Points</td>
<td style="width: 48pt" width="64">Cost</td>
</tr>
<tr style="height: 12.75pt">
<td style="height: 12.75pt;border-left-width: initial;border-left-color: initial" height="17">Myhill</td>
<td align="right">43</td>
<td align="right">4</td>
</tr>
<tr style="height: 12.75pt">
<td style="height: 12.75pt;border-left-width: initial;border-left-color: initial" height="17">Bruce</td>
<td align="right">38</td>
<td align="right">4</td>
</tr>
<tr style="height: 12.75pt">
<td style="height: 12.75pt;border-left-width: initial;border-left-color: initial" height="17">El Ahmadi</td>
<td align="right">69</td>
<td align="right">4.5</td>
</tr>
<tr style="height: 12.75pt">
<td style="height: 12.75pt;border-left-width: initial;border-left-color: initial" height="17">Vaz Te</td>
<td align="right">19</td>
<td align="right">4.5</td>
</tr>
</tbody>
</table>
&nbsp;

We can then just use the remaining money to pick our most optimal 4-4-2 with the remaining £83m:

<code>Opt.squad(footy.data,cost=83,n.gk=1,n.df=4,n.md=4,n.fd=2)</code>

&nbsp;
<table style="border-collapse: collapse;width: 144pt" border="0" width="192" cellspacing="0" cellpadding="0"> <col style="width: 48pt" span="3" width="64" /> 
<tbody>
<tr style="height: 23.25pt">
<td style="height: 23.25pt;width: 48pt;border-left-width: initial;border-left-color: initial" width="64" height="31">Player</td>
<td id="origin" style="width: 48pt" width="64">Points</td>
<td style="width: 48pt" width="64">Cost</td>
</tr>
<tr style="height: 13.5pt">
<td style="height: 13.5pt;border-left-width: initial;border-left-color: initial" height="18">Howard</td>
<td align="right">160</td>
<td align="right">5.5</td>
</tr>
<tr style="height: 13.5pt">
<td style="height: 13.5pt;border-left-width: initial;border-left-color: initial" height="18">Baines</td>
<td align="right">169</td>
<td align="right">7</td>
</tr>
<tr style="height: 13.5pt">
<td style="height: 13.5pt;border-left-width: initial;border-left-color: initial" height="18">Coleman</td>
<td align="right">180</td>
<td align="right">7</td>
</tr>
<tr style="height: 13.5pt">
<td style="height: 13.5pt;border-left-width: initial;border-left-color: initial" height="18">Terry</td>
<td align="right">172</td>
<td align="right">6.5</td>
</tr>
<tr style="height: 13.5pt">
<td style="height: 13.5pt;border-left-width: initial;border-left-color: initial" height="18">Fonte</td>
<td align="right">149</td>
<td align="right">5.5</td>
</tr>
<tr style="height: 13.5pt">
<td style="height: 13.5pt;border-left-width: initial;border-left-color: initial" height="18">Yaya Toure</td>
<td align="right">241</td>
<td align="right">11</td>
</tr>
<tr style="height: 23.25pt">
<td style="height: 23.25pt;border-left-width: initial;border-left-color: initial" height="31">Hazard</td>
<td align="right">202</td>
<td align="right">10</td>
</tr>
<tr style="height: 13.5pt">
<td style="height: 13.5pt;border-left-width: initial;border-left-color: initial" height="18">Gerrard</td>
<td align="right">205</td>
<td align="right">9</td>
</tr>
<tr style="height: 13.5pt">
<td style="height: 13.5pt;border-left-width: initial;border-left-color: initial" height="18">Sidwell</td>
<td align="right">126</td>
<td align="right">5.5</td>
</tr>
<tr style="height: 13.5pt">
<td style="height: 13.5pt;border-left-width: initial;border-left-color: initial" height="18">Giroud</td>
<td align="right">187</td>
<td align="right">8.5</td>
</tr>
<tr style="height: 13.5pt">
<td style="height: 13.5pt;border-left-width: initial;border-left-color: initial" height="18">Lambert</td>
<td align="right">179</td>
<td align="right">7.5</td>
</tr>
</tbody>
</table>
&nbsp;

Here we can see that actually using this the top eleven players calculated separately we could have got 1970 points last season. We can do this in many different ways, for example if we only wanted 1 good attacker and have more midfielders, or we may want 12/13 good players in case of injury.

&nbsp;

There are some drawbacks to the current method:
<ul>
	<li>No new transfers into existing clubs are counted</li>
	<li>No players from promoted clubs are counted</li>
</ul>
&nbsp;

It is therefore not a perfect way to pick a fantasy team, but it's good to analyse last years and see what you can get with what you have got.

&nbsp;

I think for next steps I will be building something that can calculate expected points next season rather than just looking at last seasons points, based on what club they play for, age, and how long they have at the club. I will also try and build an algorithm to correctly swap the best players throughout the season to maximise the points in each gameweek. For these two ventures I will obviously need a lot more data which needs to be sourced.

&nbsp;

As I've already said, it isn't perfect but it is a useful tool to help ascertain my starting squad for the season.

<strong>Edit:</strong> I've recently found a very similar article written about a month before I posted this one. You can find this at <a href="http://pena.lt/y/2014/07/24/mathematically-optimising-fantasy-football-teams/">pena.lt/y</a>. There are many more articles where he has taken similar ideas to deeper levels. I'll definitely be keeping a track of his further work! Great stuff!