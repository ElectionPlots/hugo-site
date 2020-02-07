---
title: "Plotly_ge_meanage_vs_lc"
date: 2020-02-02T14:30:53Z
draft: false
type: "code"
hidden: true
---

```py3 {linenos=table}
import matplotlib.pyplot as plt
import pandas as pd
import seaborn as sns
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
df_age = pd.read_csv('Age_by_year_data.csv')
df_con_head = pd.read_csv('constituency_headline.csv')

age_start = 18
party_vsc = 'LAB'
winning_party_list = ["LAB", "CON"]

# filter candidate results by party. We only want the vote share for one party. This is for the vote share change axis.
query_string = 'partyCode =="' + party_vsc + '"'
df_can_res.query(query_string, inplace = True)

# add ons code to candidate results
df_res = df_con.join(df_can_res.set_index(['conID']), on=['conID'], how='inner')

# first get required age range
df_age = df_age.query("Age_year >= " + str(age_start))

df_age_grp = df_age.groupby("PCON11CD").apply(lambda df, a, b: sum(df[a] * df[b]) / sum(df[b]), 'Age_year', 'Age_pop')

# convert series to dataframe with same name
df_age_grp = df_age_grp.to_frame().reset_index()

# give column a name
df_age_grp = df_age_grp.rename(columns= {0: 'mean_age'})

# filter on party if required. Comment out to show all parties. If filtering, adjust legend order list accordingly.
# This filter is for the result represented by the dots and can be for multiple parties.
df_con_head.query('wp in @winning_party_list', inplace = True)

# join the result with age data
df_age_hd = df_age_grp.join(df_con_head.set_index(['ons']), on=['PCON11CD'], how='inner')
#print(sorted(df_age_hd))

# join that with the vote share change data for selected party.
df_age_hd_vs = df_age_hd.join(df_res.set_index(['ons']), on=['PCON11CD'], how='inner')

# replace WIN with HOLD (not clear how it differs)
df_age_hd_vs['headline'] = df_age_hd_vs['flash'].apply(lambda x: ' '.join(x.split()[:2])).str.replace('WIN', 'HOLD')

# round percentages
decimals = 3    
df_age_hd_vs['mean_age'] = df_age_hd_vs['mean_age'].apply(lambda x: round(x, decimals))

df_age_hd_vs.rename(columns = {'wpp':'Last election'}, inplace = True) 

fig = px.scatter(df_age_hd_vs, x = 'mean_age', y = 'voteShareChange',
              color = 'headline',hover_name = 'conName', hover_data = ['Last election'],
              labels = {'mean_age':'Mean age of electorate', 'voteShareChange':party_vsc + ' vote share change %', 'headline':'Result'}, 
              title = 'Mean age of electorate by change in Labour vote share', color_discrete_map = colour_dict) 
fig.show()

with open('plotly_ge_meanage_vs.html', 'w') as f:
    f.write(fig.to_html(include_plotlyjs='cdn'))



```