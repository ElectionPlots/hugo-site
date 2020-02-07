---
title: "Plotly_ge_eu_swing_trendline"
date: 2020-02-02T23:17:13Z
draft: false
type: "code"
hidden: true
---

```py3 {linenos=table}
import pandas as pd
import plotly.express as px

colour_dict = dict({'CON HOLD':'blue',
                  'CON GAIN':'aqua',
                  'LAB HOLD': 'red',
                  'LAB GAIN': 'violet',
                  'GRN HOLD': 'green',
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
winning_party_list = ["LAB", "CON"]

# TO DO: Select only required columns in joins, maybe need merge for this

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
#print('df_eu_hd')
#print(sorted(df_eu_hd))

# join with the vote share change data for selected party.
df_eu_vs = df_eu_hd.join(df_swing.set_index(['ons']), on=['ons_code'], how='inner')

# remove % sign and convert to float
df_eu_vs['leave_to_use'] = df_eu_vs['leave_to_use'].str.rstrip('%').astype('float') 

# replace WIN with HOLD (not clear how it differs)
df_eu_vs['headline'] = df_eu_vs['flash'].apply(lambda x: ' '.join(x.split()[:2])).str.replace('WIN', 'HOLD')

#df_age_eu_vs.rename(columns = {'conName':'Constituency'}, inplace = True) 
df_eu_vs.rename(columns = {'wpp':'Last election'}, inplace = True) 

fig = px.scatter(df_eu_vs, x = 'leave_to_use', y = 'swing',
              color = 'headline',hover_name = 'conName', hover_data = ['Last election'], 
              labels = {'leave_to_use':'Leave %', 'swing':'Swing from Labour to Conservative %', 'headline':'Result'}, 
              title = 'Constituency leave vote by swing from Labour to Conservative', color_discrete_map = colour_dict, trendline='ols')               

fig['layout'].update(scene=dict(aspectmode="data"))

fig.show()  

with open('plotly_ge_eu_swing_trendline.html', 'w') as f:
   f.write(fig.to_html(include_plotlyjs='cdn'))


```