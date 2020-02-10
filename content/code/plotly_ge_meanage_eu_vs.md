---
title: "Plotly_ge_meanage_eu_vs"
date: 2020-02-03T14:24:03Z
draft: false
type: "code"
hidden: true
---

```py3 {linenos=table}
import pandas as pd
import plotly.express as px

color_dict = dict({'CON HOLD':'blue',
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
df_con_eu = pd.read_csv('eureferendum_constituency.csv')

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

# join EU data with election headlines
df_eu_hd = df_con_eu.join(df_con_head.set_index(['ons']), on=['ons_code'], how='inner')

# join the result with age data
df_age_eu = df_age_grp.join(df_eu_hd.set_index(['ons_code']), on=['PCON11CD'], how='inner')

# join that with the vote share change data for selected party.
df_age_eu_vs = df_age_eu.join(df_res.set_index(['ons']), on=['PCON11CD'], how='inner')

# replace WIN with HOLD (not clear how it differs)
df_age_eu_vs['headline'] = df_age_eu_vs['flash'].apply(lambda x: ' '.join(x.split()[:2])).str.replace('WIN', 'HOLD')

# round percentages
decimals = 1   
df_age_eu_vs['mean_age'] = df_age_eu['mean_age'].apply(lambda x: round(x, decimals))

#df_age_eu_vs.rename(columns = {'conName':'Constituency'}, inplace = True) 
df_age_eu_vs.rename(columns = {'wpp':'Last election'}, inplace = True) 

# set log_x = 'true' to make x axis wider
fig = px.scatter_3d(df_age_eu_vs, x = 'mean_age', y = 'leave_to_use', z = 'voteShareChange',
              color = 'headline',hover_name = 'conName', hover_data = ['Last election'],
              labels = {'mean_age':'Mean age of electorate', 'leave_to_use':'Leave %', 'voteShareChange':party_vsc + ' vote share change %', 'headline':'Result'}, 
              title = 'Mean age of electorate by constituency leave vote and change in Labour vote share', color_discrete_map = color_dict, log_x='false') 

fig['layout'].update(showlegend=False)

fig.update_traces(
   marker=dict(
      size=4, line = dict(
                  width=2, color='DarkSlateGrey'
                  )
      ),
   selector=dict(mode='markers')
   )

fig.show()  

with open('plotly_ge_meanage_eu_vs.html', 'w') as f:
   f.write(fig.to_html(include_plotlyjs='cdn'))


```