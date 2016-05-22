---
layout: post
title:  "Scaling over a Partition in R"
date:   2014-07-09 09:41:01 +0100
categories: [R]
---
I made this very quickly so is quite a bulky bit of code for what its actually doing. There may be something already built in R to do this, but below is a bit of code I produced to scale a continuous variable across each partition (from another partition variable). This is useful if you are looking at time series patterns across time periods (e.g. comparing days of the week for a whole year). Enjoy!

<script src="https://gist.github.com/andrewpatt24/62f8055b66a11237cf81.js"></script>