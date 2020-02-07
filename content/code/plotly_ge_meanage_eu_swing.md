---
title: "Plotly_ge_meanage_eu_swing"
date: 2020-02-03T14:24:10Z
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
winning_party_list = ["LAB", "CON"]

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

# get required age range
df_age = df_age.query("Age_year >= " + str(age_start))

df_age_grp = df_age.groupby("PCON11CD").apply(lambda df, a, b: sum(df[a] * df[b]) / sum(df[b]), 'Age_year', 'Age_pop')

# convert series to dataframe with same name
df_age_grp = df_age_grp.to_frame().reset_index()

# give column a name
df_age_grp = df_age_grp.rename(columns= {0: 'mean_age'})

# join EU data with election headlines
df_eu_hd = df_con_eu.join(df_con_head.set_index(['ons']), on=['ons_code'], how='inner')

# join the result with age data
df_age_eu = df_age_grp.join(df_eu_hd.set_index(['ons_code']), on=['PCON11CD'], how='inner')

# join that with the vote share change data for selected party.
df_age_eu_vs = df_age_eu.join(df_swing.set_index(['ons']), on=['PCON11CD'], how='inner')


# replace WIN with HOLD (not clear how it differs)
df_age_eu_vs['headline'] = df_age_eu_vs['flash'].apply(lambda x: ' '.join(x.split()[:2])).str.replace('WIN', 'HOLD')

# round percentages
decimals = 1   
df_age_eu_vs['mean_age'] = df_age_eu['mean_age'].apply(lambda x: round(x, decimals))

df_age_eu_vs.rename(columns = {'wpp':'Last election'}, inplace = True) 

# set log_x = 'true' to make x axis wider
fig = px.scatter_3d(
   df_age_eu_vs, x = 'mean_age', y = 'leave_to_use', z = 'swing',
   color = 'headline', hover_name = 'conName', hover_data = ['Last election'],
   labels = {'mean_age':'Mean age of electorate', 'leave_to_use':'Leave %', 'swing':'Swing from Labour to Conservative %', 'headline':'Result'}, 
   title = 'Mean age of electorate by constituency leave vote and swing from Labour to Conservative', 
   color_discrete_map = color_dict, log_x='False') 

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

with open('plotly_ge_meanage_eu_swing.html', 'w') as f:
   f.write(fig.to_html(include_plotlyjs='cdn'))


```