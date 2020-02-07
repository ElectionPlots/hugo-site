---
title: "Mean Age Daydream"
date: 2020-01-19T17:21:39Z
draft: false
tags: ["UK", "GE", "2019", "age", "Brexit"]
techs: ["Python", "matplotlib", "seaborn", "scatter"]
summary: What is the relation between the Leave vote in a constituency and the average age of the voters? How do the 2019 election results correlate with these factors?![Vote by mean age and leave vote (Con and Lab)](/plot_ge_meanage_eu_lc_small.png#r) 
---

Post-election polling by YouGov suggests a striking age divide amongst the British electorate, with the likelihood of voting Conservative increasing with age.

![Vote by age](/How-Britain-voted-2019-age-01.jpg)

Another strong correlation was of course, that between how people voted in The EU referendum and the general election, with Leave voters overwhelmingly more likely to vote Conservative and to a lesser extent, Remainers more likely to vote Labour.

![Remain and Leave voters](/How&#32;Britain&#32;voted&#32;2019&#32;EUref&#32;sankey-01.png)

But can this polling data be reproduced from other sources? I attempted to answer this question using the general election results along with data from the House of Commons Library giving a detailed age breakdown of all constituencies and also the EU referendum vote per constituency. This was the result:

![Vote by mean age and leave vote (all)](/plot_ge_meanage_eu.png)
{{< graphbtns2 "plot_ge_meanage_eu" >}}

This is a bit "noisy" so let's filter on the two main parties:

![Vote by mean age and leave vote (Con and Lab)](/plot_ge_meanage_eu_lc.png)
{{< graphbtns2 "plot_ge_meanage_eu_lc" >}}

The x-axis is the mean age of the electorate in each constituency and the y-axis is the Leave vote percentage for the seat. Each coloured dot represents a seat, with the colour indicating the result as specified in the legend. What is striking about this graph is that there are several Labour "holds" in seats with a high leave vote, but relatively few in seats with the oldest electorates. On the face of it, this seems to suggest that age was a bigger factor in Labour's defeat than Brexit. As we shall see, it would be wrong to jump to this conclusion, because there is an important element missing from this picture. 





