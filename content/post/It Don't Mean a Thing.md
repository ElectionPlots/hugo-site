---
title: "It Don't Mean a Thing"
date: 2020-01-23T12:09:19Z
draft: false
tags: ["UK", "GE", "2019", "age", "Brexit", "vote share", "Merseyside"]
libs: ["Python", "matplotlib", "seaborn", "plotly.express", "scatter", "trendline"]
summary: Looking at the change in the Labour vote share in relation to the age demographics of constituencies and the Leave vote percentage.![Vote by leave vote and Labour vote share change](/plot_ge_eu_vs_lc_trendline_small.png#r)
---

I mentioned in a [previous post]( {{< ref "mean age daydream.md" >}} ) that there was an element missing from the graphs presented so far, and that missing element is the change in vote share for the parties compared with the last election. My interest here is what happened to the Labour vote in relation to both age demographics and the EU referendum vote in each constituency. This graph shows the change in the Labour vote in relation to mean (average) age in each seat. To make it meaningful, it only includes seats won by either Labour or Conservative. 

![Vote by mean age and Labour vote share change](/plot_ge_meanage_vs_lc.png)
{{< graphbtns2 "plot_ge_meanage_vs_lc" >}}

Plotting a trendline, we can see that there is a trend towards a drop in Labour vote share as the average age rises, but for no obvious reason, this doesn't seem to apply once the average age gets above 52.

![Vote by mean age and Labour vote share change](/plot_ge_meanage_vs_lc_trendline.png)
{{< graphbtns2 "plot_ge_meanage_vs_lc_trendline" >}}

Plotting leave vote by Labour vote share change:

![Vote by leave vote and Labour vote share change](/plot_ge_eu_vs_lc_trendline.png)
{{< graphbtns2 "plot_ge_eu_vs_lc_trendline" >}}

Here we can see that the decrease in Labour vote is more marked; this is indicated by the steeper trendline. Looking at the distribution of the seats, irrespective of who won, we can see that the correlation is consistent for the entire leave vote spectrum. So, contrary to what was suggested by previous graphs, does this mean that Brexit was actually a more significant factor than age?

But what of our [three outliers from Merseyside]( {{< ref "the plot thickens.md" >}} )? We can drill down to the constituency level and locate them:

{{< plotly_ge_meanage_vs_lc >}}
{{< graphbtns3 "plotly_ge_meanage_vs_lc" >}}

The three seats, Wirral South, Wirral West and Sefton Central, are the three red dots furthest to the right on the graph. They all had a drop in Labour vote share but not enough to lose the seat. Threre were only a handful of other seats with similar majorities in 2017 that Labour actually lost this time around. The fact that they bucked the trend in terms of age demographics remains to be explained.