## Background

This is personal project I wanted to undergo after completing my Google
Data Analytics Certificate. I am a recent FromSoftware fan and was
curious about what percentage of Elden Ring players complete certain
bosses. Then, if there was any pattern found between completed bosses
and their stats.

## Introduction

Elden Ring is an open-world game by FromSoftware and has achievements
tied to specific bosses. In this project, I found the achievement name,
the corresponding bosses and then pulled those bosses’ stats. Then, I
took that information and looked for patterns.

I used .csv files for most of the exported files so that I can more
easily move the files between applications for analysis.

### External Sources and References

[Steam API](https://steamcommunity.com/dev)

[SteamDB](https://steamdb.info/app/1245620/stats/)

[Fextralife Elden Ring Wiki](https://eldenring.wiki.fextralife.com)

[Todd
Birchard](https://hackersandslackers.com/extract-data-from-complex-json-python/)
(copied json_extract from his article)

### Limitations

Since Elden Ring is an open-world game, it can not be determined where a
player may have “given up” in the story-line. There are a few
achievements tied to required events, but those are not explored in this
particular project.

Furthermore, a player may stumble upon a boss too early and never return
to a boss after repeated defeats. I would need in-game map information
to delve into the possibility of those scenarios, which is not within
the scope of this project.

Finally, there are differences in bosses’ move sets and relative
difficulty. This would require frame data, movement speeds, and other
information I do not have access to.

### Questions

1.  Which bosses are completed most and least by Elden Ring owners on
    Steam?

2.  Are there patterns between the most/least completed bosses and their
    absorption or resistance values?

3.  How could one use this data to inform a specific build or strategy
    in game?

### Conclusions

1.  Margit, Godrick, Red Wolf of Radagon, and Leonine Misbegotten are
    the most defeated bosses. The Regal Ancestor Spirit, Mohg, Malenia,
    Lichdragon Fortissax are the least defeated bosses.

2.  There is an increase in the average absorption and resistance values
    for bosses with \<43.10% completion.

3.  If doing a specific run or build it can be important to avoid
    depending too much on status effects and elemental damage weapons or
    spells.

## Data Collection with Python

Full disclosure, I am not a data scientist and I learned basic Python in
high school and only picked it up again recently. Perhaps, in the
future, I can come back and improve upon these data collection codes as
I continue to learn more.

### 1. Achievement Completion % from Steam API

[Steam API](https://steamcommunity.com/dev) allows Steam users to access
public user data. json_extract was taken from [Todd
Birchard](https://hackersandslackers.com/extract-data-from-complex-json-python/).
The following code returns the Elden Ring achievement names and the
percentage of players who own Elden Ring that have that achievement:

``` python
import pandas as pd
import requests
import numpy as np

#Elden Ring Steam ID
game_id = '1245620'
responses = requests.get("http://api.steampowered.com/ISteamUserStats/GetGlobalAchievementPercentagesForApp/v0002/?gameid="+game_id+"format=json")

#json_extract is from Todd Birchard on hackersandslackers.com
def json_extract(obj, key):
    """Recursively fetch values from nested JSON."""
    arr = []
    def extract(obj, arr, key):
        """Recursively search for values of key in JSON tree."""
        if isinstance(obj, dict):
            for k, v in obj.items():
                if isinstance(v, (dict, list)):
                    extract(v, arr, key)
                elif k == key:
                    arr.append(v)
        elif isinstance(obj, list):
            for item in obj:
                extract(item, arr, key)
        return arr

    values = extract(obj, arr, key)
    return values

#Combine name with percentages into one dataframe
df_name = json_extract(responses.json(), 'name')
df_percent = json_extract(responses.json(), 'percent')
test = np.column_stack((df_name,df_percent))
df = pd.DataFrame(test, columns=['ID','Percent'])

#comment following line to stop file from being overwritten
#csv = df.to_csv('./data/download/steam_api_data.csv')

df.head()
```

    ##       ID             Percent
    ## 0  ACH39   79.80000305175781
    ## 1  ACH25    74.9000015258789
    ## 2  ACH04    68.9000015258789
    ## 3  ACH40  63.900001525878906
    ## 4  ACH26                62.5

### 2. Official Achievement Names and Descriptions

I pulled the display names and descriptions for the achievements from
[SteamDB](https://steamdb.info/app/1245620/stats/). I then connected
these names and descriptions to the Steam API IDs:

``` python
for row in a_names:
    #rows in list of achievement names
    names.append(row)
for a_row in a_percent:
    #rows in list of achievement percents
    percents.append(a_row)
#Achievement descriptions
for rows in names:
    for divclass in soup.find_all('tr', {'id' : 'achievement-'+rows}):
        ad.append(divclass.text)
    dfs = pd.DataFrame(ad)
    ads = dfs.replace(r'\n',' ',regex=True)
#Split columns and delete blank columns
ads= pd.DataFrame(ads[0].str.split('  ',expand=True),columns=[0, 1, 2])
stack = np.column_stack((ads,percents))
merge = pd.DataFrame(stack, columns=['ID','Boss', 'Description', 'Percent'])
final = merge[['Boss', 'ID', 'Description', 'Percent']]

file_address = 'achievement_descriptions.csv'
#comment following line to stop file from being overwritten
#ads.to_csv(file_address) 

final.head()
```

    ##                    Boss      ID Description    Percent
    ## 0       Roundtable Hold   ACH39     Hidden.  79.800003
    ## 1  Margit the Fell Omen   ACH25     Hidden.  74.900002
    ## 2   Shardbearer Godrick   ACH04     Hidden.  68.900002
    ## 3            Great Rune   ACH40     Hidden.  63.900002
    ## 4   Red Wolf of Radagon   ACH26     Hidden.       62.5

### 3. Fextralife URLs

I took the URLs from the [Fextralife Elden Ring Bosses
page](https://eldenring.wiki.fextralife.com/Bosses):

``` python
import numpy as np
import pandas as pd
import requests
from bs4 import BeautifulSoup

url = 'https://eldenring.wiki.fextralife.com/Bosses'
reqs = requests.get(url)
soup = BeautifulSoup(reqs.text, "html.parser")

#Empty frames for Titles and corresponding URLs
titles = []
urls = []

links = soup.find_all('a', class_='wiki_link')
for link in links:
    Title = (link.get('title'))
    HRef = (link.get('href'))
    titles.append(Title)
    urls.append(HRef)

test = np.column_stack((titles,urls))

df = pd.DataFrame(test, columns=['Title','URL'])

#comment following line to stop file from being overwritten
#csv = df.to_csv('boss_urls.csv')
df.head()
```

    ##                      Title               URL
    ## 0          Elden Ring Wiki  /Elden+Ring+Wiki
    ## 1         Elden Ring Runes            /Runes
    ## 2       Elden Ring Weapons          /Weapons
    ## 3         Elden Ring Armor            /Armor
    ## 4  Elden Ring Magic Spells     /Magic+Spells

### 4. Individual Boss Absorption and Resistance Stats

Using a VLOOKUP, I connected the boss name with the website’s labels and
returned the URL extension. Using the VLOOKUP file, I pulled specific
paragraphs from Fextralife boss pages. Each loop is output into a
separate worksheet in an Excel workbook. This raw file needs the most
cleaning; there are extra rows, special characters and differing stat
terms.

``` python
import pandas as pd
import requests
from bs4 import BeautifulSoup

df=pd.read_csv("steam_boss_info-Vlookup.csv")
new_url = (df["URL"])

#Empty frame for URL endings
address = []

#create Excel sheet
writer = pd.ExcelWriter('boss_stats.xlsx', engine='xlsxwriter')

for row in new_url:
    #print(row)
    address.append(row)
for rows in address:
    url = 'https://eldenring.wiki.fextralife.com'+rows
    reqs = requests.get(url)
    soup = BeautifulSoup(reqs.text, "html.parser")
    #Empty frame for boss stat information
    stats = []
    for divclass in soup.find_all('div', {'class' : 'col-sm-6'}):
        for ultag in divclass.find_all('ul'):
            for litag in ultag.find_all('li'):
                #minimize the long strings not needed from this loop
                if len(litag.text) < 43:
                    stats.append(litag.text.split(":"))
    dfs = pd.DataFrame(stats)
    unique = dfs.drop_duplicates()
    #checks that the sheet name doesn't exceed Excel worksheet maximum
    sheet_names = rows[:30]
    unique.to_excel(writer, sheet_names.strip("/"))
#comment following line to stop file from being overwritten
#writer.save()
unique.head()
```

    ##                 0    1
    ## 0  Phy (Standard)   35
    ## 1     Phy (Slash)   35
    ## 2    Phy (Strike)   35
    ## 3    Phy (Pierce)   10
    ## 4           Magic   40

## Data Cleaning with Excel

![](Table3.png)

## Preparing for Analysis with Tableau

There is a range from 27.9% to 74.90% for the % of Elden Ring players
that completed certain bosses. From my own personal knowledge of the
game, it was unsurprising that bosses that are optional and/or hard to
find like the Lichdragon Fortissax, Shardbearer Malenia, and Mohg the
Fell Omen were the least completed.

![](.\Dark%20Background%20Data%20Viz\Boss%20Completion%20of%20Elden%20Ring%20Owners%20on%20Steam.png)

The following are the overviews of the absorption and resistance numbers
side by side with the completion percentages to search for initial
patterns. Astel, Godrick, Valiant Gargoyles, and the Royal Knight
Loretta were omitted from the elemental and status effect
visualizations. Those bosses had several immune values that I translated
to “9999” in my data, so I removed them to avoid skewing the averages
Their data was included in the Physical Damage absorption analysis:

![](.\Dark%20Background%20Data%20Viz\Elemental%20Damage%20&%20Boss%20Completion.png)

![](.\Dark%20Background%20Data%20Viz\Status%20Effects%20&%20Boss%20Completion.png)
*Madness and Sleep were omitted as the value was either immune or blank;
This made it an uncommon binary effect. Death was included as it was the
most common binary status effect.*

![](.\Dark%20Background%20Data%20Viz\Physical%20Attacks%20&%20Boss%20Completion.png)

A difference could be seen in the amount of high absoprtion and
resistancce values on the lower completion % group, but I needed to make
it more visually clear to reach a confident conclusion

## Data Analysis

I looked at the stat averages of the above and below 43.10% completion
groups and found compelling differences. I made sure the scales were the
same across the absorption categories, to prevent visual distortion of
the results.

![](https://github.com/Maria-Catherine/Elden-Ring-Boss-Analysis/blob/main/Elemental_Damage.png?raw=true) The biggest
difference between the groups is Lightening and Holy, differing by 14
and 15 respectively. The bosses that were completed least had a higher
average absorption in all elemental damage types.

![](.\Dark%20Background%20Data%20Viz\Status%20Effect.png) The biggest
difference between the groups is Poison and Scarlet Rot, differing by
187 and 189 respectively. The bosses that were completed the least had a
higher average resistance to all status effects

![](.\Dark%20Background%20Data%20Viz\Physical%20Damage.png) Physical
damage types showed the smallest relative differences. The biggest
difference is Strike and Standard, differing by 15 and 6 respectively.
The bosses that were completed the least had increased physical damage
absorption, but not as drastically as status effects and elemental
damage.

## Conclusion

There is a negative relationship between the absorption and resistance
numbers and the % of Steam Elden Ring players who defeat a boss.

This project stemmed from my interest in data, video games, and how
people are known to drop FromSoftware games due to difficulty. So, after
looking at all this data, I have some general insights and applications
for my conclusions:

**Build Choice:** Individuals will often pick a certaub toye if damage
and weapon they will stick with for a playthrough of a game. One popular
one is a Bleed Build, which increases the infliction of what is labeled
Hemorrhage in this project. This can lead people to get frustrated with
late-game bosses that are bleed resistant and must be fought based on
physical damage. This results give an overview of what bosses may be
more difficult if one is doing a certain build, a nice alternative to
Googling “what’s a good build” or “is Bleed the best build.” I
personally plan to use this data for my next playthrough of Elden Ring.

**Run/Ending Planning:** Similar to choosing a build, some will know
they want to pursue a certain ending or “completion run” where they
fight everything. If someone is wanting to pursue a certain characters
ending or story line, they may need to fight Malenia, fir example, and a
Scarlet Rot focused build may increase the difficulty.

## Closing

This was a fun project to practice and strengthen my skills, new and
old, and answer questions I had about Elden Ring. The results of
searching for builds feels less personal and I enjoy seeing a story
unfold for myself. I plan on doing a similar project with Dark Souls 3,
a more linear FromSoftware game.

#### Thank you for reading!

