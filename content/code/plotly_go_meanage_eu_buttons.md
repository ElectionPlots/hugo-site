---
title: "Plotly_go_meanage_eu_buttons"
date: 2020-02-03T11:39:46Z
draft: false
type: "code"
hidden: true
---

```py3 {linenos=table}
import pandas as pd
import plotly.graph_objects as go
from sys import exit

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
df_regions = pd.read_csv('HoC-2019GE-results-by-constituency.csv')

age_start = 18
winning_party_list = ["LAB", "CON"]

# filter on party if required. Comment out to show all parties. If filtering, adjust legend order list accordingly.
# This filter is for the result represented by the dots and can be for multiple parties.
df_con_head.query('wp in @winning_party_list', inplace = True)

# first get required age range
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

# replace WIN with HOLD (not clear how it differs)
df_age_eu['headline'] = df_age_eu['flash'].apply(lambda x: ' '.join(x.split()[:2])).str.replace('WIN', 'HOLD')

# remove % sign and convert to float
df_age_eu['leave_to_use'] = df_age_eu['leave_to_use'].str.rstrip('%').astype('float') / 100.0

# round percentages
decimals = 1    
df_age_eu['leave_to_use'] = df_age_eu['leave_to_use'].apply(lambda x: round(x*100, decimals))
df_age_eu['mean_age'] = df_age_eu['mean_age'].apply(lambda x: round(x, decimals))

df_age_eu.rename(columns = {'constituency_name':'Constituency'}, inplace = True) 

def get_color(row):
    if row['headline'] == 'LAB HOLD':
        col = 'red'
    elif row['headline'] == 'LAB GAIN':
        col = 'violet'
    elif row['headline'] == 'CON HOLD':
        col = 'blue'
    else:
        col = 'aqua'
    return col    

df_age_eu['colour'] =  df_age_eu.apply(get_color, axis=1)

# variables for colorscale
color_names = ['CON GAIN','CON HOLD','LAB GAIN','LAB HOLD']
color_vals = list(range(len(color_names)))
num_colors = len(color_vals)

# map the interval [0, max_val] to [0,1] by t--> t/max_val
# see https://community.plot.ly/t/manually-customize-colorbar-scatter-python/18520
# and https://community.plot.ly/t/colors-for-discrete-ranges-in-heatmaps/7780

# displayed with highest number at top
colorscale = [
    [0, 'aqua'],
    [0.25, 'aqua'],
    [0.25, 'blue'],
    [0.5, 'blue'],
    [0.5, 'violet'],
    [0.75, 'violet'],
    [0.75, 'red'],
    [1, 'red']]
cmin = -0.5
cmax = num_colors - 0.5

# ONS regions
"""
ons_region_id	region_name	CountOfons_id
E12000001	North East	29
E12000002	North West	75
E12000003	Yorkshire and The Humber	54
E12000004	East Midlands	46
E12000005	West Midlands	59
E12000006	East	58
E12000007	London	73
E12000008	South East	84
E12000009	South West	55
N92000002	Northern Ireland	18
S92000003	Scotland	59
W92000004	Wales	40
"""

# join with regions data
df_age_eu = df_age_eu.join(df_regions.set_index(['ons_id']), on=['PCON11CD'], how='inner')
# print(list(df_age_eu.columns))
# exit(0)

# Initialize figure
fig = go.Figure()

# html for hover text
hvt = '<b>%{text}</b><br><b>Mean age</b>: %{x}<br><b>Leave vote</b>: %{y:.3n}%<br><b>Last election</b>: %{customdata}<extra>%</extra>'

# variables for button position
button_layer_1_height = 1.12
button_layer_2_height = 1.065

# list of buttons
btnlist = []
i=0

# list of booleans specifying which trace is visible
vislist = [False] * 12

# make trace for all regions visible initially
vislist[i] = True    

# first add trace for all regions
fig.add_trace(
    go.Scatter(
        x = df_age_eu['mean_age'], 
        y = df_age_eu['leave_to_use'],
        mode = 'markers', 
        marker = dict(
            colorbar = dict(
                title='Result',
                tickvals= color_vals,
                ticktext= color_names,
                tickfont=dict(size=8),
                len=0.2
            ),
            colorscale = colorscale,
            cmin=cmin,
            cmax=cmax,
            color = df_age_eu['colour'],
            showscale=True), 
        text=df_age_eu['Constituency'], 
        hovertemplate=hvt, 
        customdata = df_age_eu['wpp'],
        showlegend=False
    )
)

# add new button to list
btnlist.append(
    dict(
        args=[{"visible": vislist}],
        label="All",
        method="update"
        )
)

# create buttons in list (only one so far)
fig.update_layout(
    updatemenus=[
        go.layout.Updatemenu(
            type = "buttons",
            direction="right",
            pad={"r": 10, "t": 10},
            showactive=True,
            x=0,
            xanchor="left",
            y=button_layer_2_height,
            yanchor="top",
            buttons=btnlist,
        ),
    ]
)

i=1

# get list of regions sorted by ons region id
regions = list(df_age_eu.sort_values('ons_region_id').region_name.unique())

# Create a data frame dictionary for regional dfs
regiondict = {region : pd.DataFrame() for region in regions}

for key in regiondict.keys():
    regiondict[key] = df_age_eu[:][df_age_eu.region_name == key]

# Add Traces and buttons for each region
for region, df in regiondict.items():

    fig.add_trace(
        go.Scatter(
            x = df['mean_age'], 
            y = df['leave_to_use'],
            mode='markers', 
            marker = dict(
                colorbar = dict(
                title='Result',
                tickvals= color_vals,
                ticktext= color_names,
                tickfont=dict(size=8),
                len=0.2
            ),
            colorscale = colorscale,
            cmin=cmin,
            cmax=cmax,
            color = df['colour'],
            showscale=True), 
            text=df['Constituency'], 
            hovertemplate=hvt, 
            customdata = df['wpp'],
            showlegend=False
        )
    )

    # re-initialise for this button
    vislist = [False] * 12
    vislist[i] = True    

    # add new button to list
    btnlist.append(
        dict(
            args=[{"visible": vislist}],
            label=region,
            method="update"
            )
    )

    # for each iteration we recreate all the buttons. Not ideal but there 
    # doesn't seem to be any alternative apart from hard coding the region 
    # names and creating the buttons individually without looping.
    fig.update_layout(
        updatemenus=[
            go.layout.Updatemenu(
                type = "buttons",
                direction="right",
                pad={"r": 10, "t": 10},
                showactive=True,
                x=0,
                xanchor="left",
                y=button_layer_2_height,
                yanchor="top",
                buttons=btnlist, 
                font=dict(size=10)
            ),
        ]
    )  

    i = i + 1  

# common to all traces    
fig.update_layout(
    title='<b>Mean age of electorate by constituency leave vote</b>', 
    title_x=0.5,
    xaxis={'title':'Mean age of electorate'}, 
    yaxis={'title':'Leave %'},)

# add extra explanatory text
fig.update_layout(
    annotations=[
        go.layout.Annotation(
            text='Seats won by<br>Conservative<br>or Labour',
            align='left',
            showarrow=False,
            xref='paper',
            yref='paper',
            x=1.0,
            y=1.0,
            # bordercolor='black',
            # borderwidth=1,
            font=dict(size=10)
        )
    ]
)

fig.show()

with open('plotly_go_meanage_eu_buttons.html', 'w') as f:
    f.write(fig.to_html(include_plotlyjs='cdn'))
```
