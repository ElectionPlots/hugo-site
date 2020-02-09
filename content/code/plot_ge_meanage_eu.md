---
title: "plot_ge_meanage_eu_csv"
date: 2020-01-26T11:30:44Z
draft: false
type: "code"
hidden: true
---

```py3 {linenos=table}
import matplotlib.pyplot as plt
import pandas as pd
import seaborn as sns

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

df_age = pd.read_csv('Age_by_year_data.csv')
df_con_head = pd.read_csv('constituency_headline.csv')
df_con_eu = pd.read_csv('eureferendum_constituency.csv')

age_start = 18

# first get required age range
df_age = df_age.query("Age_year >= " + str(age_start))

# group by constituency and calculate mean ages
df_age_grp = df_age.groupby("PCON11CD").apply(lambda df, a, b: sum(df[a] * df[b]) / sum(df[b]), 'Age_year', 'Age_pop')

# convert series to dataframe with same name
df_age_grp = df_age_grp.to_frame().reset_index()

# give new mean column a name
df_age_grp = df_age_grp.rename(columns= {0: 'mean_age'})

# name index
df_age_grp.index.name = 'ons_code'

# filter on party if required. Comment out to show all parties. If filtering, adjust legend order list accordingly
# df_con_head.query('wp =="CON" or wp =="LAB"', inplace = True)
# join EU data with election headlines
df_eu_hd = df_con_eu.join(df_con_head.set_index(['ons']), on=['ons_code'], how='inner')

# join the result with age data
df_age_eu = df_age_grp.join(df_eu_hd.set_index(['ons_code']),on=['PCON11CD'], how='inner')

# replace WIN with HOLD (not clear how it differs)
df_age_eu['headline'] = df_age_eu['flash'].apply(lambda x: ' '.join(x.split()[:2])).str.replace('WIN', 'HOLD')

# remove % sign and convert to float
df_age_eu['leave_to_use'] = df_age_eu['leave_to_use'].str.rstrip('%').astype('float') / 100.0

sns.lmplot(y="leave_to_use", x="mean_age", data=df_age_eu, hue="headline", height=8.27, aspect=11.7/8.27, fit_reg=False, palette=colour_dict, legend=False)

plt.gca().set_xticklabels(['{:.0f}'.format(x) for x in plt.gca().get_xticks()]) 
plt.gca().set_yticklabels(['{:.0f}%'.format(x*100) for x in plt.gca().get_yticks()]) 

plt.ylabel("Leave %")
plt.xlabel("Mean age of electorate")

plt.title("2019 General Election: Mean age of electorate by leave vote", fontsize=12)

handles, labels = plt.gca().get_legend_handles_labels()

# order legend using the default positions - comment out if filtering on party
order = [0,1,2,6,3,7,4,13,14,15,8,11,9,10,12,5]

# uncomment for CON and LAB only filter
# order = [0,1,2,3]
plt.legend([handles[idx] for idx in order],[labels[idx] for idx in order])

plt.tight_layout()
plt.show()
```

