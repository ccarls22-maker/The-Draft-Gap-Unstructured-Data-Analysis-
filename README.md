# Draft Gap


# Project Overview

Colorado ranks fourth in the United States for the total number of craft
breweries, reflecting a strong local craft beer culture. This project
examines how the distribution and what types of breweries intersect with
neighborhood economic conditions, and how these factors influence award
recognition. By considering both economic and social dimensions, we
explore how craft breweries not only reflect but also shape access,
opportunity, and community engagement across different regions.

# Refrence Links:

Beer API: https://www.openbrewerydb.org/ Poverty API:
https://www.census.gov/data/developers/data-sets/Poverty-Statistics.html
Colorado Brewers Cup:https://coloradobeer.org/brewers-cup/ Great
American Beer Fest:
https://www.greatamericanbeerfestival.com/the-competition/winners/

### County poverty rate Colorado

``` python
import requests
import pandas as pd
url = "https://api.census.gov/data/2021/acs/acs5"

params = {
    'get': 'NAME,B17001_002E,B17001_001E',
    'for': 'county:*', 
    'in': 'state:08'  #### REMEMBER TO CHANGE THIS IF YOU WANT TO CHANGE THE STATE!!!
}
response = requests.get(url, params=params)
data = response.json()
df = pd.DataFrame(data[1:], columns=data[0])
df['poverty_rate'] = (df['B17001_002E'].astype(int) / df['B17001_001E'].astype(int)) * 100

df
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }
&#10;    .dataframe tbody tr th {
        vertical-align: top;
    }
&#10;    .dataframe thead th {
        text-align: right;
    }
</style>

|  | NAME | B17001_002E | B17001_001E | state | county | poverty_rate |
|----|----|----|----|----|----|----|
| 0 | Adams County, Colorado | 48959 | 510494 | 08 | 001 | 9.590514 |
| 1 | Alamosa County, Colorado | 2466 | 15402 | 08 | 003 | 16.010908 |
| 2 | Arapahoe County, Colorado | 49673 | 645068 | 08 | 005 | 7.700428 |
| 3 | Archuleta County, Colorado | 1283 | 13155 | 08 | 007 | 9.752946 |
| 4 | Baca County, Colorado | 702 | 3432 | 08 | 009 | 20.454545 |
| ... | ... | ... | ... | ... | ... | ... |
| 59 | Summit County, Colorado | 2097 | 30854 | 08 | 117 | 6.796526 |
| 60 | Teller County, Colorado | 1851 | 24508 | 08 | 119 | 7.552636 |
| 61 | Washington County, Colorado | 520 | 4560 | 08 | 121 | 11.403509 |
| 62 | Weld County, Colorado | 30560 | 315304 | 08 | 123 | 9.692234 |
| 63 | Yuma County, Colorado | 1396 | 9628 | 08 | 125 | 14.499377 |

<p>64 rows × 6 columns</p>
</div>

### Poverty Rate by Zipcode

``` python
import requests
import pandas as pd
url = "https://api.census.gov/data/2021/acs/acs5"

params = {
    'get': 'NAME,B17001_002E,B17001_001E',
    'for': 'zip code tabulation area:*'
}
response = requests.get(url, params=params)
data = response.json()
df = pd.DataFrame(data[1:], columns=data[0])
df['poverty_rate'] = (df['B17001_002E'].astype(int) / df['B17001_001E'].astype(int)) * 100
df['zip_state'] = df['zip code tabulation area'].str[:2]
co_df = df[df['zip_state'].isin(['80', '81'])]

co_df
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }
&#10;    .dataframe tbody tr th {
        vertical-align: top;
    }
&#10;    .dataframe thead th {
        text-align: right;
    }
</style>

|  | NAME | B17001_002E | B17001_001E | zip code tabulation area | poverty_rate | zip_state |
|----|----|----|----|----|----|----|
| 28345 | ZCTA5 80002 | 2230 | 19709 | 80002 | 11.314628 | 80 |
| 28346 | ZCTA5 80003 | 3587 | 36676 | 80003 | 9.780238 | 80 |
| 28347 | ZCTA5 80004 | 2106 | 35174 | 80004 | 5.987377 | 80 |
| 28348 | ZCTA5 80005 | 900 | 28875 | 80005 | 3.116883 | 80 |
| 28349 | ZCTA5 80007 | 364 | 17132 | 80007 | 2.124679 | 80 |
| ... | ... | ... | ... | ... | ... | ... |
| 28867 | ZCTA5 81653 | 0 | 0 | 81653 | NaN | 81 |
| 28868 | ZCTA5 81654 | 314 | 1441 | 81654 | 21.790423 | 81 |
| 28869 | ZCTA5 81655 | 76 | 307 | 81655 | 24.755700 | 81 |
| 28870 | ZCTA5 81656 | 0 | 299 | 81656 | 0.000000 | 81 |
| 28871 | ZCTA5 81657 | 302 | 5164 | 81657 | 5.848180 | 81 |

<p>527 rows × 6 columns</p>
</div>

– Saved results to a CSV

# Brewery API Pull for Colorado

``` python
import requests
import pandas as pd
BASE_URL = "https://api.openbrewerydb.org/v1/breweries"
all_breweries = []
try:
    page = 1
    while True:
        co = requests.get(BASE_URL, params={"by_state": "Colorado", "per_page": 200, "page": page}) ### Change state here!!!
        items = co.json()
        if not items:
            break
        all_breweries.extend(items)
        page += 1
except requests.RequestException:
    pass
columns = ["name", "brewery_type", "city", "state", "postal_code", "website_url"]
df = pd.DataFrame(all_breweries)[columns]
df['postal_code'] = df['postal_code'].astype(str).str[:5]
BAPI = df
BAPI
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }
&#10;    .dataframe tbody tr th {
        vertical-align: top;
    }
&#10;    .dataframe thead th {
        text-align: right;
    }
</style>

|  | name | brewery_type | city | state | postal_code | website_url |
|----|----|----|----|----|----|----|
| 0 | 10 Barrel Brewing Co - Denver | large | Denver | Colorado | 80205 | None |
| 1 | 105 West Brewing Co | micro | Castle Rock | Colorado | 80109 | http://www.105westbrewing.com |
| 2 | 12Degree Brewing | brewpub | Louisville | Colorado | 80027 | http://www.12degree.com |
| 3 | 14er Brewing Company | proprietor | Denver | Colorado | 80205 | http://www.14erBrewing.com |
| 4 | 3 Freaks Brewing Co | micro | Highlands Ranch | Colorado | 80126 | http://www.3freaksbrewery.com |
| ... | ... | ... | ... | ... | ... | ... |
| 443 | Yetters Brewing Company | brewpub | Greeley | Colorado | 80631 | https://yettersbrewingcompany.com/ |
| 444 | Zephyr Brewing Co | micro | Denver | Colorado | 80216 | http://www.zephyrbrewingco.com |
| 445 | Zuni Street Brewing Company | micro | Denver | Colorado | 80211 | http://www.zunistreetbrewing.com |
| 446 | Zwei Brewing Co | micro | Fort Collins | Colorado | 80525 | http://www.zweibruderbrewing.com |
| 447 | Zymos Brewing | micro | Littleton | Colorado | 80123 | http://zymosbrewing.com |

<p>448 rows × 6 columns</p>
</div>

– Saved results to a CSV

# Colorado Brewers Cup awards

``` python
import requests
from bs4 import BeautifulSoup
import pandas as pd
def cbc():
    url = "https://coloradobeer.org/brewers-cup/"
    
    headers = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36'
    }
    
    try:
        response = requests.get(url, headers=headers)
        response.raise_for_status()
        
        soup = BeautifulSoup(response.content, 'html.parser')
        
        beer_data = []
        tables = soup.find_all('table')
        
        for table in tables:
            for row in table.find_all('tr'):
                cells = row.find_all('td')
                if len(cells) >= 6:
                    entry = {
                        'category': cells[0].get_text(strip=True),
                        'beer_name': cells[2].get_text(strip=True),  
                        'brewery': cells[3].get_text(strip=True),    
                        'city': cells[4].get_text(strip=True),       
                        'subcategory': cells[5].get_text(strip=True) 
                    }
                    beer_data.append(entry)
        
        return beer_data
        
    except Exception as e:
        return []
data = cbc()
CBC = pd.DataFrame(data)

CBC
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }
&#10;    .dataframe tbody tr th {
        vertical-align: top;
    }
&#10;    .dataframe thead th {
        text-align: right;
    }
</style>

|  | category | beer_name | brewery | city | subcategory |
|----|----|----|----|----|----|
| 0 | American & International Pale Lagers | Venga | Cerveceria Colorado | Denver | Australasian, Latin American or Tropical-Style... |
| 1 | American & International Pale Lagers | Landing Gear Pils | Westbound & Down Brewing Co. | Idaho Springs | International-Style Pilsener |
| 2 | American & International Pale Lagers | Cityscapes | Ratio Beerworks | Denver | International Pale Lager |
| 3 | Pale Bitter European Lagers | Classic Pilsner | New Image Brewing Co. | Wheat Ridge | German Pils |
| 4 | Pale Bitter European Lagers | Dortmunder | Hideaway Park Brewery | Winter Park | German Helles Exportbier |
| ... | ... | ... | ... | ... | ... |
| 73 | Wood & Aged Beers | Here Be Monsters | Cerebral Brewing - Aurora Arts | Aurora | Wood- and Barrel-Aged Strong Stout |
| 74 | Wood & Aged Beers | Black Pearl 4x3 | Locavore Beer Works | Littleton | Wood- and Barrel-Aged Strong Beer |
| 75 | Experimental & Specialty Beers | East County Fine Malt Liquor | The Post Brewing Co. | Lafayette | American-Style Malt Liquor |
| 76 | Experimental & Specialty Beers | Cinnamon Roll Blonde Ale | Bearded Brewer Artisan Ales | Longmont | Experimental Beer |
| 77 | Experimental & Specialty Beers | Oily Oaf | Wonderland Brewing Co. | Broomfield | Kentucky Common Beer |

<p>78 rows × 5 columns</p>
</div>

— Saved Results to a csv

# Get GABF results

``` python
import requests
from bs4 import BeautifulSoup
import pandas as pd
url = "https://www.greatamericanbeerfestival.com/2025-winners/"
response = requests.get(url)
soup = BeautifulSoup(response.content, 'html.parser')
data = []
for table in soup.find_all('table'):
    for row in table.find_all('tr'):
        cells = row.find_all('td')
        if len(cells) < 4:
            continue
            
        medal_text = cells[0].get_text().lower()
        if 'gold' in medal_text:
            medal = 'Gold'
        elif 'silver' in medal_text:
            medal = 'Silver'
        elif 'bronze' in medal_text:
            medal = 'Bronze'
        else:
            continue
        beer_name = cells[1].get_text(strip=True)
        brewery = cells[2].get_text(strip=True)
        city = cells[3].get_text(strip=True)
        
        state = ""
        category = ""
        for i, cell in enumerate(cells):
            if '_state' in str(cell.get('class', [])):
                state = cell.get_text(strip=True)
                if i + 1 < len(cells):
                    category = cells[i + 1].get_text(strip=True)
                break
        
        data.append({
            'medal': medal,
            'beer_name': beer_name,
            'brewery': brewery, 
            'city': city,
            'state': state,
            'category': category
        })
GABF = pd.DataFrame(data)
#df.to_csv("gabf_2025_winners_fixed.csv", index=False)

GABF
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }
&#10;    .dataframe tbody tr th {
        vertical-align: top;
    }
&#10;    .dataframe thead th {
        text-align: right;
    }
</style>

|  | medal | beer_name | brewery | city | state | category |
|----|----|----|----|----|----|----|
| 0 | Gold | Mr. Oktoberfest | The Mitten Brewing Co. | Grand Rapids | MI | American Amber Lager |
| 1 | Silver | Oktoberfest | Transmission Brewing Co. | Ventura | CA | American Amber Lager |
| 2 | Bronze | Full Steam Ahead | 1849 Brewing Co. | Grass Valley | CA | American Amber Lager |
| 3 | Gold | Ash Cloud | Barrel Mountain Brewing | Battle Ground | WA | American Amber/Red Ale |
| 4 | Silver | Summon Ifrit | BattleMage Brewing Co. | Vista | CA | American Amber/Red Ale |
| ... | ... | ... | ... | ... | ... | ... |
| 342 | Silver | Perpetual Peace | No Label Brewing Co. | Katy | TX | Wood- and Barrel-Aged Strong Beer |
| 343 | Bronze | Moroccan Christmas | Wild Blue Yonder Brewing Co. | Castle Rock | CO | Wood- and Barrel-Aged Strong Beer |
| 344 | Gold | Vladislav | Diebolt Brewing | Denver | CO | Wood- and Barrel-Aged Strong Stout |
| 345 | Silver | Darkstar November | Bottle Logic Brewing | Anaheim | CA | Wood- and Barrel-Aged Strong Stout |
| 346 | Bronze | Breakside La Maison du Bang! | Breakside Brewery & Taproom | Milwaukie | OR | Wood- and Barrel-Aged Strong Stout |

<p>347 rows × 6 columns</p>
</div>

# Join GABF and Colorado Breweries API results

``` python
import pandas as pd

winners = GABF

breweries = BAPI
winners['brewery_clean'] = winners['brewery'].str.lower().str.strip().str.replace(' ', '').str.replace(r'[^\w\s]', '', regex=True)
winners['brewery_norm'] = winners['brewery'].str.replace(r'(?i)\b(?:company|co\.?)\b', 'Co', regex=True).str.split('-', n=1).str[0].str.strip()
breweries['name_norm'] = breweries['name'].str.replace(r'(?i)\b(?:company|co\.?)\b', 'Co', regex=True).str.split('-', n=1).str[0].str.strip()
winners['brewery_clean'] = winners['brewery_norm'].str.lower().str.strip().str.replace(r'[^\w\s]', '', regex=True).str.replace(' ', '')
breweries['name_clean'] = breweries['name_norm'].str.lower().str.strip().str.replace(r'[^\w\s]', '', regex=True).str.replace(' ', '')
winners.drop(columns=['brewery_norm'], inplace=True)
breweries.drop(columns=['name_norm'], inplace=True)
result = winners.merge(breweries, left_on='brewery_clean', right_on='name_clean', how='left')
result = result.drop(['brewery_clean', 'name_clean'], axis=1)

result
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }
&#10;    .dataframe tbody tr th {
        vertical-align: top;
    }
&#10;    .dataframe thead th {
        text-align: right;
    }
</style>

|  | medal | beer_name | brewery | city_x | state_x | category | name | brewery_type | city_y | state_y | postal_code | website_url |
|----|----|----|----|----|----|----|----|----|----|----|----|----|
| 0 | Gold | Mr. Oktoberfest | The Mitten Brewing Co. | Grand Rapids | MI | American Amber Lager | NaN | NaN | NaN | NaN | NaN | NaN |
| 1 | Silver | Oktoberfest | Transmission Brewing Co. | Ventura | CA | American Amber Lager | NaN | NaN | NaN | NaN | NaN | NaN |
| 2 | Bronze | Full Steam Ahead | 1849 Brewing Co. | Grass Valley | CA | American Amber Lager | NaN | NaN | NaN | NaN | NaN | NaN |
| 3 | Gold | Ash Cloud | Barrel Mountain Brewing | Battle Ground | WA | American Amber/Red Ale | NaN | NaN | NaN | NaN | NaN | NaN |
| 4 | Silver | Summon Ifrit | BattleMage Brewing Co. | Vista | CA | American Amber/Red Ale | NaN | NaN | NaN | NaN | NaN | NaN |
| ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |
| 345 | Silver | Perpetual Peace | No Label Brewing Co. | Katy | TX | Wood- and Barrel-Aged Strong Beer | NaN | NaN | NaN | NaN | NaN | NaN |
| 346 | Bronze | Moroccan Christmas | Wild Blue Yonder Brewing Co. | Castle Rock | CO | Wood- and Barrel-Aged Strong Beer | NaN | NaN | NaN | NaN | NaN | NaN |
| 347 | Gold | Vladislav | Diebolt Brewing | Denver | CO | Wood- and Barrel-Aged Strong Stout | Diebolt Brewing | micro | Denver | Colorado | 80211 | http://www.dieboltbrewing.com |
| 348 | Silver | Darkstar November | Bottle Logic Brewing | Anaheim | CA | Wood- and Barrel-Aged Strong Stout | NaN | NaN | NaN | NaN | NaN | NaN |
| 349 | Bronze | Breakside La Maison du Bang! | Breakside Brewery & Taproom | Milwaukie | OR | Wood- and Barrel-Aged Strong Stout | NaN | NaN | NaN | NaN | NaN | NaN |

<p>350 rows × 12 columns</p>
</div>

# Join Colorado Breweries and Colorado Brewers Cup Winners

``` python
import pandas as pd

colorado_awards = CBC

breweries = BAPI
colorado_awards['brewery_clean'] = colorado_awards['brewery'].str.lower().str.strip().str.replace(r'[^\w\s]', '', regex=True).str.replace(' ', '')
breweries['name_clean'] = breweries['name'].str.lower().str.strip().str.replace(r'[^\w\s]', '', regex=True).str.replace(' ', '')
colorado_awards['brewery_norm'] = colorado_awards['brewery'].str.replace(r'(?i)\b(?:company|co\.?)\b', 'Co', regex=True).str.split('-', n=1).str[0].str.strip()
breweries['name_norm'] = breweries['name'].str.replace(r'(?i)\b(?:company|co\.?)\b', 'Co', regex=True).str.split('-', n=1).str[0].str.strip()
colorado_awards['brewery_clean'] = colorado_awards['brewery_norm'].str.lower().str.strip().str.replace(r'[^\w\s]', '', regex=True).str.replace(' ', '')
breweries['name_clean'] = breweries['name_norm'].str.lower().str.strip().str.replace(r'[^\w\s]', '', regex=True).str.replace(' ', '')
colorado_awards.drop(columns=['brewery_norm'], inplace=True)
breweries.drop(columns=['name_norm'], inplace=True)
colorado_result = colorado_awards.merge(breweries, left_on='brewery_clean', right_on='name_clean', how='left')
colorado_result = colorado_result.drop(['brewery_clean', 'name_clean'], axis=1)
colorado_result
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }
&#10;    .dataframe tbody tr th {
        vertical-align: top;
    }
&#10;    .dataframe thead th {
        text-align: right;
    }
</style>

|  | category | beer_name | brewery | city_x | subcategory | name | brewery_type | city_y | state | postal_code | website_url |
|----|----|----|----|----|----|----|----|----|----|----|----|
| 0 | American & International Pale Lagers | Venga | Cerveceria Colorado | Denver | Australasian, Latin American or Tropical-Style... | NaN | NaN | NaN | NaN | NaN | NaN |
| 1 | American & International Pale Lagers | Landing Gear Pils | Westbound & Down Brewing Co. | Idaho Springs | International-Style Pilsener | Westbound & Down Brewing Company | brewpub | Idaho Springs | Colorado | 80452 | http://www.westboundanddown.com |
| 2 | American & International Pale Lagers | Cityscapes | Ratio Beerworks | Denver | International Pale Lager | Ratio Beerworks | micro | Denver | Colorado | 80205 | http://ratiobeerworks.com |
| 3 | Pale Bitter European Lagers | Classic Pilsner | New Image Brewing Co. | Wheat Ridge | German Pils | New Image Brewing Co | brewpub | Arvada | Colorado | 80002 | http://www.nibrewing.com |
| 4 | Pale Bitter European Lagers | Dortmunder | Hideaway Park Brewery | Winter Park | German Helles Exportbier | Hideaway Park Brewery | micro | Winter Park | Colorado | 80482 | http://www.hideawayparkbrewery.com |
| ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |
| 78 | Wood & Aged Beers | Black Pearl 4x3 | Locavore Beer Works | Littleton | Wood- and Barrel-Aged Strong Beer | Locavore Beer Works | micro | Littleton | Colorado | 80123 | http://www.locavorebeerworks.com |
| 79 | Experimental & Specialty Beers | East County Fine Malt Liquor | The Post Brewing Co. | Lafayette | American-Style Malt Liquor | The Post Brewing Co | brewpub | Boulder | Colorado | 80302 | http://www.postbrewing.com |
| 80 | Experimental & Specialty Beers | East County Fine Malt Liquor | The Post Brewing Co. | Lafayette | American-Style Malt Liquor | The Post Brewing Co | brewpub | Lafayette | Colorado | 80026 | http://www.postbrewing.com |
| 81 | Experimental & Specialty Beers | Cinnamon Roll Blonde Ale | Bearded Brewer Artisan Ales | Longmont | Experimental Beer | NaN | NaN | NaN | NaN | NaN | NaN |
| 82 | Experimental & Specialty Beers | Oily Oaf | Wonderland Brewing Co. | Broomfield | Kentucky Common Beer | Wonderland Brewing Co. | micro | Broomfield | Colorado | 80020 | http://www.wonderlandbrewing.com |

<p>83 rows × 11 columns</p>
</div>

# Join Colorado Breweries x GAMF x Colrado Cup

``` python
import pandas as pd
import re

colorado_awards = CBC

winners = GABF
gabf_winners = winners

breweries = BAPI
pattern = r'(?i)\b(?:company|co\.?)\b'
def clean_name(name):
    if pd.isna(name):
        return ''
    return re.sub(r'[^\w\s]', '', str(name).lower()).strip().replace(' ', '')
def clean_beer(beer_name):
    if pd.isna(beer_name):
        return ''
    return re.sub(r'[^\w\s]', '', str(beer_name).lower()).strip().replace(' ', '')
breweries['name_clean'] = breweries['name'].apply(clean_name)
colorado_awards['brewery_clean'] = colorado_awards['brewery'].apply(clean_name)
gabf_winners['brewery_clean'] = gabf_winners['brewery'].apply(clean_name)
colorado_awards['beer_norm'] = colorado_awards['beer_name'].apply(clean_beer)
gabf_winners['beer_norm'] = gabf_winners['beer_name'].apply(clean_beer)
colorado_awards['beer_id'] = colorado_awards['brewery_clean'] + '_' + colorado_awards['beer_norm']
gabf_winners['beer_id'] = gabf_winners['brewery_clean'] + '_' + gabf_winners['beer_norm']
colorado_clean = colorado_awards.copy()
colorado_clean['colorado_cup_medal'] = 'Colorado Cup Winner'
colorado_clean = colorado_clean.rename(columns={
    'category': 'colorado_cup_category', 
    'beer_name': 'colorado_cup_beer',
    'subcategory': 'colorado_cup_subcategory'
})
gabf_clean = gabf_winners.copy()
gabf_clean = gabf_clean.rename(columns={
    'medal': 'gabf_medal',
    'category': 'gabf_category',
    'beer_name': 'gabf_beer'
})
master_brewery_list = breweries.copy()
master_brewery_list = pd.merge(
    master_brewery_list,
    colorado_clean[['brewery_clean', 'beer_id', 'colorado_cup_medal', 'colorado_cup_category', 'colorado_cup_beer', 'colorado_cup_subcategory']],
    left_on='name_clean',
    right_on='brewery_clean',
    how='left'
)
gabf_temp = gabf_clean[['beer_id', 'gabf_medal', 'gabf_category', 'gabf_beer']].copy()
master_brewery_list = pd.merge(
    master_brewery_list,
    gabf_temp,
    on='beer_id',
    how='left'
)
master_brewery_list = master_brewery_list.drop(columns=['name_clean', 'brewery_clean', 'beer_id']).reset_index(drop=True)
master_brewery_list_final = master_brewery_list.drop_duplicates(subset=['name', 'colorado_cup_beer', 'gabf_beer'])
master_brewery_list_final
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }
&#10;    .dataframe tbody tr th {
        vertical-align: top;
    }
&#10;    .dataframe thead th {
        text-align: right;
    }
</style>

|  | name | brewery_type | city | state | postal_code | website_url | colorado_cup_medal | colorado_cup_category | colorado_cup_beer | colorado_cup_subcategory | gabf_medal | gabf_category | gabf_beer |
|----|----|----|----|----|----|----|----|----|----|----|----|----|----|
| 0 | 10 Barrel Brewing Co - Denver | large | Denver | Colorado | 80205 | None | NaN | NaN | NaN | NaN | NaN | NaN | NaN |
| 1 | 105 West Brewing Co | micro | Castle Rock | Colorado | 80109 | http://www.105westbrewing.com | NaN | NaN | NaN | NaN | NaN | NaN | NaN |
| 2 | 12Degree Brewing | brewpub | Louisville | Colorado | 80027 | http://www.12degree.com | NaN | NaN | NaN | NaN | NaN | NaN | NaN |
| 3 | 14er Brewing Company | proprietor | Denver | Colorado | 80205 | http://www.14erBrewing.com | NaN | NaN | NaN | NaN | NaN | NaN | NaN |
| 4 | 3 Freaks Brewing Co | micro | Highlands Ranch | Colorado | 80126 | http://www.3freaksbrewery.com | NaN | NaN | NaN | NaN | NaN | NaN | NaN |
| ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |
| 452 | Yetters Brewing Company | brewpub | Greeley | Colorado | 80631 | https://yettersbrewingcompany.com/ | NaN | NaN | NaN | NaN | NaN | NaN | NaN |
| 453 | Zephyr Brewing Co | micro | Denver | Colorado | 80216 | http://www.zephyrbrewingco.com | NaN | NaN | NaN | NaN | NaN | NaN | NaN |
| 454 | Zuni Street Brewing Company | micro | Denver | Colorado | 80211 | http://www.zunistreetbrewing.com | NaN | NaN | NaN | NaN | NaN | NaN | NaN |
| 455 | Zwei Brewing Co | micro | Fort Collins | Colorado | 80525 | http://www.zweibruderbrewing.com | NaN | NaN | NaN | NaN | NaN | NaN | NaN |
| 456 | Zymos Brewing | micro | Littleton | Colorado | 80123 | http://zymosbrewing.com | NaN | NaN | NaN | NaN | NaN | NaN | NaN |

<p>455 rows × 13 columns</p>
</div>

# Complete Join By Adding Poverty Data

``` python
import pandas as pd

master_breweries = master_brewery_list_final

poverty_data = co_df
master_breweries['postal_code'] = master_breweries['postal_code'].astype(str)
poverty_data['zip code tabulation area'] = poverty_data['zip code tabulation area'].astype(str)
Final_Join = master_breweries.merge(
    poverty_data,
    left_on='postal_code',
    right_on='zip code tabulation area',
    how='left'
)
Final_Join = Final_Join.drop(columns=['zip_state', 'B17001_001E', 'B17001_002E', 'NAME'], errors='ignore')
Final_Join = Final_Join.drop('zip code tabulation area', axis=1)
Final_Join 
```

    C:\Users\court\AppData\Local\Temp\ipykernel_22328\2793759248.py:6: SettingWithCopyWarning:


    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead

    See the caveats in the documentation: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }
&#10;    .dataframe tbody tr th {
        vertical-align: top;
    }
&#10;    .dataframe thead th {
        text-align: right;
    }
</style>

|  | name | brewery_type | city | state | postal_code | website_url | colorado_cup_medal | colorado_cup_category | colorado_cup_beer | colorado_cup_subcategory | gabf_medal | gabf_category | gabf_beer | poverty_rate |
|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|
| 0 | 10 Barrel Brewing Co - Denver | large | Denver | Colorado | 80205 | None | NaN | NaN | NaN | NaN | NaN | NaN | NaN | 16.865957 |
| 1 | 105 West Brewing Co | micro | Castle Rock | Colorado | 80109 | http://www.105westbrewing.com | NaN | NaN | NaN | NaN | NaN | NaN | NaN | 2.381577 |
| 2 | 12Degree Brewing | brewpub | Louisville | Colorado | 80027 | http://www.12degree.com | NaN | NaN | NaN | NaN | NaN | NaN | NaN | 4.457223 |
| 3 | 14er Brewing Company | proprietor | Denver | Colorado | 80205 | http://www.14erBrewing.com | NaN | NaN | NaN | NaN | NaN | NaN | NaN | 16.865957 |
| 4 | 3 Freaks Brewing Co | micro | Highlands Ranch | Colorado | 80126 | http://www.3freaksbrewery.com | NaN | NaN | NaN | NaN | NaN | NaN | NaN | 2.037298 |
| ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |
| 450 | Yetters Brewing Company | brewpub | Greeley | Colorado | 80631 | https://yettersbrewingcompany.com/ | NaN | NaN | NaN | NaN | NaN | NaN | NaN | 23.486929 |
| 451 | Zephyr Brewing Co | micro | Denver | Colorado | 80216 | http://www.zephyrbrewingco.com | NaN | NaN | NaN | NaN | NaN | NaN | NaN | 18.488724 |
| 452 | Zuni Street Brewing Company | micro | Denver | Colorado | 80211 | http://www.zunistreetbrewing.com | NaN | NaN | NaN | NaN | NaN | NaN | NaN | 10.581230 |
| 453 | Zwei Brewing Co | micro | Fort Collins | Colorado | 80525 | http://www.zweibruderbrewing.com | NaN | NaN | NaN | NaN | NaN | NaN | NaN | 8.864950 |
| 454 | Zymos Brewing | micro | Littleton | Colorado | 80123 | http://zymosbrewing.com | NaN | NaN | NaN | NaN | NaN | NaN | NaN | 4.504465 |

<p>455 rows × 14 columns</p>
</div>

# Quick Stats

``` python
#Drop duplicate brewery names for basic calcs
Final_Join_unique = Final_Join.drop_duplicates(subset=['name'])
```

``` python
Final_Join_unique.describe()
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }
&#10;    .dataframe tbody tr th {
        vertical-align: top;
    }
&#10;    .dataframe thead th {
        text-align: right;
    }
</style>

|       | poverty_rate |
|-------|--------------|
| count | 443.000000   |
| mean  | 10.656961    |
| std   | 6.163887     |
| min   | 1.889559     |
| 25%   | 5.523160     |
| 50%   | 9.516031     |
| 75%   | 13.704166    |
| max   | 32.217558    |

</div>

``` python
#State Average Poverty Rate
Avg_poverty = co_df['poverty_rate'].mean()
Avg_poverty
```

    np.float64(10.988587014808395)

``` python
# Average poverty rate for zipcodes with at least one craft brewery
zip_means = Final_Join.groupby('postal_code')['poverty_rate'].mean()
zip_means.mean()
```

    np.float64(9.77252825788744)

``` python
# Create Flag for award winners
Final_Join['medal_flag'] = (
    (Final_Join['colorado_cup_medal'].notnull())|
    (Final_Join['gabf_medal'].notnull())
).astype(int)

# Create Flag for Breweries over of Under State Avg Poverty

Final_Join['Above_Avg_poverty'] = (Final_Join['poverty_rate'] > Avg_poverty).astype(int)

Final_Join
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }
&#10;    .dataframe tbody tr th {
        vertical-align: top;
    }
&#10;    .dataframe thead th {
        text-align: right;
    }
</style>

|  | name | brewery_type | city | state | postal_code | website_url | colorado_cup_medal | colorado_cup_category | colorado_cup_beer | colorado_cup_subcategory | gabf_medal | gabf_category | gabf_beer | poverty_rate | medal_flag | Above_Avg_poverty |
|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|----|
| 0 | 10 Barrel Brewing Co - Denver | large | Denver | Colorado | 80205 | None | NaN | NaN | NaN | NaN | NaN | NaN | NaN | 16.865957 | 0 | 1 |
| 1 | 105 West Brewing Co | micro | Castle Rock | Colorado | 80109 | http://www.105westbrewing.com | NaN | NaN | NaN | NaN | NaN | NaN | NaN | 2.381577 | 0 | 0 |
| 2 | 12Degree Brewing | brewpub | Louisville | Colorado | 80027 | http://www.12degree.com | NaN | NaN | NaN | NaN | NaN | NaN | NaN | 4.457223 | 0 | 0 |
| 3 | 14er Brewing Company | proprietor | Denver | Colorado | 80205 | http://www.14erBrewing.com | NaN | NaN | NaN | NaN | NaN | NaN | NaN | 16.865957 | 0 | 1 |
| 4 | 3 Freaks Brewing Co | micro | Highlands Ranch | Colorado | 80126 | http://www.3freaksbrewery.com | NaN | NaN | NaN | NaN | NaN | NaN | NaN | 2.037298 | 0 | 0 |
| ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |
| 450 | Yetters Brewing Company | brewpub | Greeley | Colorado | 80631 | https://yettersbrewingcompany.com/ | NaN | NaN | NaN | NaN | NaN | NaN | NaN | 23.486929 | 0 | 1 |
| 451 | Zephyr Brewing Co | micro | Denver | Colorado | 80216 | http://www.zephyrbrewingco.com | NaN | NaN | NaN | NaN | NaN | NaN | NaN | 18.488724 | 0 | 1 |
| 452 | Zuni Street Brewing Company | micro | Denver | Colorado | 80211 | http://www.zunistreetbrewing.com | NaN | NaN | NaN | NaN | NaN | NaN | NaN | 10.581230 | 0 | 0 |
| 453 | Zwei Brewing Co | micro | Fort Collins | Colorado | 80525 | http://www.zweibruderbrewing.com | NaN | NaN | NaN | NaN | NaN | NaN | NaN | 8.864950 | 0 | 0 |
| 454 | Zymos Brewing | micro | Littleton | Colorado | 80123 | http://zymosbrewing.com | NaN | NaN | NaN | NaN | NaN | NaN | NaN | 4.504465 | 0 | 0 |

<p>455 rows × 16 columns</p>
</div>

``` python
#Break down of brewery types
Final_Join_unique['brewery_type'].value_counts()
```

    brewery_type
    micro         227
    brewpub       140
    planning       39
    regional       11
    contract        9
    closed          9
    large           7
    proprietor      4
    Name: count, dtype: int64

``` python
# Brewery Type split on Pverty Flag
Final_Join_unique['Above_Avg_poverty'] = (Final_Join_unique['poverty_rate'] > Avg_poverty).astype(int)


Final_Join_unique.groupby(['Above_Avg_poverty', 'brewery_type']).size()
```

    C:\Users\court\AppData\Local\Temp\ipykernel_22328\4261389395.py:2: SettingWithCopyWarning:


    A value is trying to be set on a copy of a slice from a DataFrame.
    Try using .loc[row_indexer,col_indexer] = value instead

    See the caveats in the documentation: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy

    Above_Avg_poverty  brewery_type
    0                  brewpub          80
                       closed            9
                       contract          8
                       large             3
                       micro           143
                       planning         24
                       proprietor        3
                       regional          4
    1                  brewpub          60
                       contract          1
                       large             4
                       micro            84
                       planning         15
                       proprietor        1
                       regional          7
    dtype: int64

``` python
# Breweries above or below poverty flag 
Final_Join_unique['Above_Avg_poverty'].value_counts()
```

    Above_Avg_poverty
    0    274
    1    172
    Name: count, dtype: int64

# Early Insights

Early observations reveal a clear divide in access to craft beer.
Experimental microbreweries, which often push the boundaries with
innovative brewing techniques, are predominantly located in more
affluent areas. In contrast brewpubs which frequently serve as community
gathering spaces are more common in regions with higher poverty rates.
This suggests that both economic factors and neighborhood demographics
may influence the type of breweries present in an area. Now that we have
a lead in the data, I turned to Tableau to examine the Draft Gap in
depth. The visualizations above provide a clearer view of these
patterns.

# Conclusion

The Draft Gap, examines how Colorado’s economic divide shapes one of its
signature industries: craft beer. A comparison of ZIP codes above and
below the state’s average poverty rate highlights distinct economic
sectors. And when the focus narrows to ZIP codes with at least one
brewery, disparities become even more visible, only 172/448 breweries
operate in areas with a below-average poverty rate.

Two themes emerge from this pattern: the dual engines of the industry
and the inequity of recognition. Colorado’s brewery landscape is
dominated by two types of producers, microbreweries and brewpubs, each
serving a different role. Surprisingly, only 37% of microbreweries are
located in higher-poverty regions, despite having lower start-up costs
than brewpubs. Brewpubs, which combine brewing with a restaurant
component, show a slightly stronger presence in these communities at
42%.

Brewpubs often function as local gathering spaces, reinforcing social
cohesion in economically insecure areas, while microbreweries tend to
specialize in experimental products that advance brewing techniques and
draw industry attention.

Recognition in the industry is driven largely by two key awards (1) the
Colorado Brewers Cup and (2) the highly sought-after Great American Beer
Festival medals. Because microbreweries cluster in more affluent ZIP
codes, breweries in those areas receive a disproportionate share of
awards. As a result, many brewpubs (despite producing high-quality beer)
are held back by limited funding, reduced visibility, and fewer
resources to compete for those recognitions.

This imbalance matters in a state where the average brewery lifespan is
estimated at 6.5 years and the national industry is contracting.
Although earlier research suggested award outcomes did not strongly
affect long-term survival, it also found brewpubs exited the market more
frequently, hinting at deeper socio-economic dynamics. When these
dynamics are viewed together (brewery type, neighborhood conditions, and
award distribution) a pattern emerges in which breweries in less
affluent areas struggle to gain recognition and market footholds. In an
increasingly crowded industry, such recognition may be more critical
than ever for survival and community impact.

While Colorado represents only one case within a broader industry, the
patterns here highlight how supporting access, visibility, and long-term
stability for breweries across all communities can reinforce the
strength of the craft beer sector and the connections people build
around it.
