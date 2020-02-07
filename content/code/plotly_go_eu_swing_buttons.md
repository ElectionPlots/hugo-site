---
title: "Plotly_go_eu_swing_buttons"
date: 2020-02-03T11:55:46Z
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

df_con = pd.read_csv('constituencies.csv')
df_can_res = pd.read_csv('candidate_results.csv')
df_con_head = pd.read_csv('constituency_headline.csv')
df_con_eu = pd.read_csv('eureferendum_constituency.csv')
df_regions = pd.read_csv('HoC-2019GE-results-by-constituency.csv')

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

# join EU data with election headlines
df_eu_hd = df_con_eu.join(df_con_head.set_index(['ons']), on=['ons_code'], how='inner')
#print('df_eu_hd')
#print(sorted(df_eu_hd))

# join the result with age data
#df_eu_vs = df_age_grp.join(df_eu_hd.set_index(['ons_code']), on=['PCON11CD'], how='inner')

# join with the vote share change data for selected party.
df_eu_vs = df_eu_hd.join(df_swing.set_index(['ons']), on=['ons_code'], how='inner')

# remove % sign and convert to float
df_eu_vs['leave_to_use'] = df_eu_vs['leave_to_use'].str.rstrip('%').astype('float') 

# replace WIN with HOLD (not clear how it differs)
df_eu_vs['headline'] = df_eu_vs['flash'].apply(lambda x: ' '.join(x.split()[:2])).str.replace('WIN', 'HOLD')

# round percentages
decimals = 1    
df_eu_vs['leave_to_use'] = df_eu_vs['leave_to_use'].apply(lambda x: round(x*100, decimals))

df_eu_vs.rename(columns = {'constituency_name':'Constituency'}, inplace = True) 

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

df_eu_vs['colour'] =  df_eu_vs.apply(get_color, axis=1)

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
df_eu_vs = df_eu_vs.join(df_regions.set_index(['ons_id']), on=['ons_code'], how='inner')
# print(list(df_eu_vs.columns))
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
        x = df_eu_vs['leave_to_use'], 
        y = df_eu_vs['swing'],
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
            color = df_eu_vs['colour'],
            showscale=True), 
        text=df_eu_vs['Constituency'], 
        hovertemplate=hvt, 
        customdata = df_eu_vs['wpp'],
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
regions = list(df_eu_vs.sort_values('ons_region_id').region_name.unique())

# Create a data frame dictionary for regional dfs
regiondict = {region : pd.DataFrame() for region in regions}

for key in regiondict.keys():
    regiondict[key] = df_eu_vs[:][df_eu_vs.region_name == key]

# Add Traces and buttons for each region
for region, df in regiondict.items():

    fig.add_trace(
        go.Scatter(
            x = df['leave_to_use'], 
            y = df['swing'],
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
    title='<b>Constituency leave vote by swing from Labour to Conservative</b>', 
    title_x=0.5,
    xaxis={'title':'Leave %'}, 
    yaxis={'title':'Swing from Labour to Conservative %'},)

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

with open('plotly_go_eu_swing_buttons.html', 'w') as f:
    f.write(fig.to_html(include_plotlyjs='cdn'))

```
