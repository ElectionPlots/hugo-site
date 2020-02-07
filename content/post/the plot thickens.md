---
title: "The Plot Thickens"
date: 2020-01-19T19:22:08Z
draft: false
tags: ["UK", "GE", "2019", "age", "Brexit", "Merseyside"]
libs: ["Python", "matplotlib", "seaborn", "plotly.express", "plotly.graph_objects", "scatter", "annotation"]
summary: Thinking about three apparent Labour outliers on Merseyside&#58; why did Labour manage to buck the trend and hold these seats?![Vote by mean age and leave vote (Con and Lab)](/plot_ge_meanage_eu_lc_arrows_small.png#r)
---

In the final graph in the [previous post]( {{< ref "mean age daydream.md" >}} ), it was noticeable that there were three Labour seats which seemed to buck the trend of Labour losing in seats with the oldest electorates. They are represented by the three red dots furthest to the left on the graph.

![Vote by mean age and leave vote (Con and Lab)](/plot_ge_meanage_eu_lc_arrows.png)
{{< graphbtns2 "plot_ge_meanage_eu_lc_arrows" >}}

It would be interesting to know which seats these three dots represent. To achieve this I reproduced the graph, this time using the [Plotly Python Open Source Graphing Library](https://plot.ly/python/)'s hover text function. This graph is best seen in full page view.
{{< plotly_ge_meanage_eu_lc >}}
{{< graphbtns3 "plotly_ge_meanage_eu_lc_arrows" >}}


By hovering over the dots those three Labour outliers are revealed as Sefton Central, Wirral West and Wirral South, all Merseyside seats. So is there some factor specific to Merseyside that allowed Labour to keep these seats? This [Liverpool Echo article](https://www.liverpoolecho.co.uk/news/liverpool-news/you-know-tories-once-ran-15632298) gives some historical context, David Jeffrey of Liverpool University  is quoted as saying that during the Thatcher years "It became a part of Scouse identity to be opposed to the party - and that has continued." But why should this effect be more pronounced in Liverpool rather than say, The North-East or Yorkshire? Is it that "Scouse identity" is itself stronger than the general sense of having a regional identity in other areas of the country? Or is it a legacy of the large-scale Catholic Irish immigration into Liverpool following the great famine in the 19th Century: the strong Irish connection weakening English nationalism in the area and thereby opening up a space betwixt and between Ireland and England within which a strong regional identity might emerge? Jeffrey gives a more nuanced analysis of the decline of the Conservative Party in Liverpool [here](https://link.springer.com/article/10.1057/s41293-016-0032-6).

There is also the Hillsborough factor: could the fact that most newsagents in the area don't sell the The Sun have an effect?








