---
title: "Plotly_ge_meanage_eu_lc_arrows"
date: 2020-02-01T21:54:02Z
draft: false
type: "code"
hidden: true
---

```py3 {linenos=table}
import matplotlib.pyplot as plt
import pandas as pd
import plotly.express as px
import plotly.graph_objects as go

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

df_age = pd.read_csv('Age_by_year_data.csv')
df_con_head = pd.read_csv('constituency_headline.csv')
df_con_eu = pd.read_csv('eureferendum_constituency.csv')

age_start = 18
winning_party_list = ["LAB", "CON"]

# filter on party if required. Comment out to show all parties. If filtering, adjust legend order list accordingly.
# This filter is for the result represented by the dots and can be for multiple parties.
df_con_head.query('wp in @winning_party_list', inplace = True)

# first get required age range
df_age = df_age.query("Age_year >= " + str(age_start))

# group by constituency and calculate mean ages
df_age_grp = df_age.groupby("PCON11CD").apply(lambda df, a, b: sum(df[a] * df[b]) / sum(df[b]), 'Age_year', 'Age_pop')

# convert series to dataframe with same name
df_age_grp = df_age_grp.to_frame().reset_index()

# give column a name
df_age_grp = df_age_grp.rename(columns= {0: 'mean_age'})

# join EU data with election headlines
df_eu_hd = df_con_eu.join(df_con_head.set_index(['ons']), on=['ons_code'], how='inner')

# join the result with age data
df_age_eu = df_age_grp.join(df_eu_hd.set_index(['ons_code']), on=['PCON11CD'], how='inner')

# replace WIN with HOLD (not clear how it differs)
df_age_eu['headline'] = df_age_eu['flash'].apply(lambda x: ' '.join(x.split()[:2])).str.replace('WIN', 'HOLD')

# remove % sign and convert to float
df_age_eu['leave_to_use'] = df_age_eu['leave_to_use'].str.rstrip('%').astype('float') / 100.0

# round decimals
decimals = 1    
df_age_eu['leave_to_use'] = df_age_eu['leave_to_use'].apply(lambda x: round(x*100, decimals))
df_age_eu['mean_age'] = df_age_eu['mean_age'].apply(lambda x: round(x, decimals))

df_age_eu.rename(columns = {'constituency_name':'Constituency'}, inplace = True) 
df_age_eu.rename(columns = {'wpp':'Last election'}, inplace = True) 

fig = px.scatter(df_age_eu, x = 'mean_age', y = 'leave_to_use',
              color = 'headline',hover_name = 'Constituency', hover_data = ['Last election'],
              labels = {'mean_age':'Mean age of electorate', 'leave_to_use':'Leave %', 'headline':'Result'}, 
              title = 'Mean age of electorate by constituency leave vote', color_discrete_map = colour_dict) 

fig.add_annotation(
    go.layout.Annotation(
        x=54.6,
        y=44.1,
        text="Outliers",
        ax=-34,
        ay=155)
)
fig.add_annotation(
    go.layout.Annotation(
        x=54,
        y=44.7,
        text="",
        ax=-10,
        ay=154)
)
fig.add_annotation(
    go.layout.Annotation(
        x=53,
        y=46.5,
        text="",
        ax=30,
        ay=174)
)
fig.update_annotations(dict(
    xref="x",
    yref="y",
    showarrow=True,
    arrowhead=1,
    standoff=4,
))

fig.show()

with open('plotly_ge_meanage_eu_lc_arrows.html', 'w') as f:
    f.write(fig.to_html(include_plotlyjs='cdn'))
```
