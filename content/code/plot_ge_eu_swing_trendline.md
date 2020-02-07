---
title: "Plot_ge_eu_swing_trendline"
date: 2020-02-02T23:04:28Z
draft: false
type: "code"
hidden: true
---

```py3 {linenos=table}
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd
import seaborn as sns
from sys import exit

colour_dict = dict({'CON HOLD':'b',
                  'CON GAIN':'aqua',
                  'LAB HOLD': 'red',
                  'LAB GAIN': 'violet',
                  'GRN HOLD': 'g',
                  'DUP HOLD': 'sandybrown',
                  'LD HOLD': 'yellow',
                  'LD GAIN': 'orange',
                  'SNP HOLD': 'black',
                  'SNP GAIN': 'gray',
                  'PC HOLD': 'pink',
                  'SF HOLD': 'peru',
                  'SF GAIN': 'wheat',
                  'SDLP HOLD': 'salmon',
                  'SDLP GAIN': 'lightsalmon',
                  'APNI GAIN': 'lawngreen',
                  'SPE HOLD': 'navy'})

df_con = pd.read_csv('constituencies.csv')
df_can_res = pd.read_csv('candidate_results.csv')
df_con_head = pd.read_csv('constituency_headline.csv')
df_con_eu = pd.read_csv('eureferendum_constituency.csv')

age_start = 18
winning_party_list = ["CON","LAB"]

# add ons code to candidate results
df_res = df_con.join(df_can_res.set_index(['conID']), on=['conID'], how='inner')

# include only the two parties with the most votes in each seat
df_res = df_res.sort_values(['ons', 'votes']).groupby('ons').tail(2)

# exclude parties other than lab and con
df_res = df_res[df_res['partyCode'].isin(winning_party_list)]

# exclude seats with only one row
df_res = df_res.groupby('ons').filter(lambda x: len(x) > 1)

# we want the vote share for both Lab and Con
# get df for Lab vote share change
query_string = 'partyCode =="LAB"'
df_lvsc = df_res.query(query_string)

# rename vote share change column
df_lvsc.rename(columns= {'voteShareChange': 'lvsc'}, inplace=True)

# get df for Con vote share change
query_string = 'partyCode =="CON"'
df_cvsc = df_res.query(query_string)

# rename vote share change column
df_cvsc.rename(columns= {'voteShareChange': 'cvsc'}, inplace=True)

# now join the two into one df

df_swing = pd.merge(df_lvsc,df_cvsc[['ons','cvsc']],on='ons', how='left')

# now calculate the swing to the cons using the Butler method: (cvsc - lvsc) / 2
df_swing['swing'] = (df_swing['cvsc'] - df_swing['lvsc']) / 2

# join EU data with election headlines
df_eu_hd = df_con_eu.join(df_con_head.set_index(['ons']), on=['ons_code'], how='inner')

# join with the vote share change data for selected party.
df_eu_vs = df_eu_hd.join(df_swing.set_index(['ons']), on=['ons_code'], how='inner')

# remove % sign and convert to float
df_eu_vs['leave_to_use'] = df_eu_vs['leave_to_use'].str.rstrip('%').astype('float') 

# replace WIN with HOLD (not clear how it differs)
df_eu_vs['headline'] = df_eu_vs['flash'].apply(lambda x: ' '.join(x.split()[:2])).str.replace('WIN', 'HOLD')

sns.lmplot(y="swing", x="leave_to_use", data=df_eu_vs, hue="headline", height=8.27, aspect=11.7/8.27, fit_reg=False, palette=colour_dict, legend=False)

plt.gca().set_xticklabels(['{:.0f}'.format(x) for x in plt.gca().get_xticks()]) 
plt.gca().set_yticklabels(['{:.0f}%'.format(x) for x in plt.gca().get_yticks()]) 

plt.ylabel('Swing from Labour to Conservative %')
plt.xlabel("Leave %")

plt.title("2019 General Election: Constituency leave vote by swing from Labour to Conservative", fontsize=12)

handles, labels = plt.gca().get_legend_handles_labels()

# uncomment for CON and LAB only filter
order = [0,1,2,3]

plt.legend([handles[idx] for idx in order],[labels[idx] for idx in order])
plt.legend(loc="upper left")

#plot trendline
x = df_eu_vs.leave_to_use
y = df_eu_vs.swing

z = np.polyfit(x, y, 1)
p = np.poly1d(z)
plt.plot(x,p(x),"r:")

plt.tight_layout()
plt.show()


```
