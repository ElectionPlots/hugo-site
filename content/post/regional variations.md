---
title: "Regional Variations"
date: 2020-02-03T11:13:47Z
draft: false
tags: ["UK", "GE", "2019", "age", "Brexit", "swing", "regions"]
libs: ["Python", "plotly.graph_objects", "scatter", "annotations", "colorscale", "buttons"]
summary: How do the graphs look for different regions of the UK?![Filter by region](/plotly_go_eu_swing_buttons_small.jpg#r)
---

The Office of National Statistics (ONS) have identified twelve electoral regions in the UK:

| ONS Region ID | Region name              | Constituencies |
|---------------|--------------------------|----------------|
| E12000001     | North East               | 29             |
| E12000002     | North West               | 75             |
| E12000003     | Yorkshire and The Humber | 54             |
| E12000004     | East Midlands            | 46             |
| E12000005     | West Midlands            | 59             |
| E12000006     | East                     | 58             |
| E12000007     | London                   | 73             |
| E12000008     | South East               | 84             |
| E12000009     | South West               | 55             |
| N92000002     | Northern Ireland         | 18             |
| S92000003     | Scotland                 | 59             |
| W92000004     | Wales                    | 40             |

Here are a couple of the graphs from previous posts with added buttons allowing the data to be filtered by region (best viewed via full page view):

{{< plotly_go_meanage_eu_buttons >}}
{{< graphbtns3 "plotly_go_meanage_eu_buttons" >}}

{{< plotly_go_eu_swing_buttons >}}
{{< graphbtns3 "plotly_go_eu_swing_buttons" >}}