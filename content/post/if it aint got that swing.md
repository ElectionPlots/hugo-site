---
title: "...if it ain't got that swing"
date: 2020-02-02T19:59:32Z
draft: false
summary: Using swing from Labour to Conservative rather than change in Labour vote share.![Constituency leave vote by swing from Labour to Conservative (all)](/plotly_ge_eu_swing_trendline_small.jpg#r)
tags: ["UK", "GE", "2019", "age", "Brexit", "swing"]
libs: ["Python", "matplotlib", "seaborn", "plotly.express", "scatter", "trendline"]
---

I wrote the previous post in the mistaken belief that the term "swing" was synonymous with the change in vote share for a particular party, but I have since discovered that the term refers to a calculation that is based on the change in vote share of *two* parties. The Steed method calculates swing in terms of only the votes cast for the two parties, while the more widely used Butler method calculates it on the basis of the votes cast for all parties. To calculate the swing from Labour to Conservative in any given seat using the Butler method, we apply the following formula:

Let Lcvs be the change in Labour vote share and Ccvs be the change in Conservative vote share.

$$ Swing = \dfrac{(Ccvs - Lcvs)}{2} $$

For example, in Bassetlaw, there was a 11.9% increase in the Conservative vote and a 24.9% decrease in the Labour vote. The swing from Labour to Conservative is calculated as follows:

$$ Swing = \dfrac{(11.9\% - (-24.9\%))}{2} = 18.4\% $$

Using swing from Labour to Conservative rather than change in Labour vote results in similar but not identical graphs. For these graphs to be meaningful, we have to only include those seats where Labour and Conservative are the top two parties.

![Vote by mean age and swing from Labour to Conservative (all)](/plot_ge_meanage_swing_trendline.png)
{{< graphbtns2 "plot_ge_meanage_swing_trendline" >}}

The trendline shows how the swing to the Tories increases with age and in the graph below an even steeper increase with leave vote.

![Constituency leave vote by swing from Labour to Conservative (all)](/plot_ge_eu_swing_trendline.png)
{{< graphbtns2 "plot_ge_eu_swing_trendline" >}}

Just for the hell of it here's a Plotly version of the above graph showing separate trendlines for Labour and Conservative holds and Conservative gains.
{{< plotly_ge_eu_swing_trendline >}}
{{< graphbtns3 "plotly_ge_eu_swing_trendline" >}}