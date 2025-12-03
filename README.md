# Draft Gap

# Project Overview
Colorado ranks fourth in the United States for the total number of craft breweries, reflecting a strong local craft beer culture. This project examines how the distribution and what types of breweries intersect with neighborhood economic conditions, and how these factors influence award recognition. By considering both economic and social dimensions, we explore how craft breweries not only reflect but also shape access, opportunity, and community engagement across different regions.

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
print(df)
```

                               NAME B17001_002E B17001_001E state county  \
    0        Adams County, Colorado       48959      510494    08    001   
    1      Alamosa County, Colorado        2466       15402    08    003   
    2     Arapahoe County, Colorado       49673      645068    08    005   
    3    Archuleta County, Colorado        1283       13155    08    007   
    4         Baca County, Colorado         702        3432    08    009   
    ..                          ...         ...         ...   ...    ...   
    59      Summit County, Colorado        2097       30854    08    117   
    60      Teller County, Colorado        1851       24508    08    119   
    61  Washington County, Colorado         520        4560    08    121   
    62        Weld County, Colorado       30560      315304    08    123   
    63        Yuma County, Colorado        1396        9628    08    125   

        poverty_rate  
    0       9.590514  
    1      16.010908  
    2       7.700428  
    3       9.752946  
    4      20.454545  
    ..           ...  
    59      6.796526  
    60      7.552636  
    61     11.403509  
    62      9.692234  
    63     14.499377  

    [64 rows x 6 columns]

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
```

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
#df.to_csv('colorado_beer_awards.csv', index=False)
```

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
```

# Join GABF and Colorado Breweries API results

``` python
import pandas as pd
#winners = pd.read_csv("C:/Users/court/Downloads/Unstructered_final_project/Data_Dump/gabf_fixed.csv")
winners = GABF
#breweries = pd.read_csv("C:/Users/court/Downloads/Unstructered_final_project/Data_Dump/breweries_co_or_zip08.csv")
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
#colorado_awards = pd.read_csv("C:/Users/court/Downloads/Unstructered_final_project/Data_Dump/colorado_beer_awards.csv")
colorado_awards = CBC
#breweries = pd.read_csv("C:/Users/court/Downloads/Unstructered_final_project/Data_Dump/breweries_co_or_zip08.csv")
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
#colorado_awards = pd.read_csv("C:/Users/court/Downloads/Unstructered_final_project/cbcxcoloradobrews.csv")
colorado_awards = CBC
#gabf_winners = pd.read_csv("C:/Users/court/Downloads/Unstructered_final_project/gabfxcoloradobrews.csv")
winners = GABF
gabf_winners = winners
#breweries = pd.read_csv("C:/Users/court/Downloads/Unstructered_final_project/Data_Dump/breweries_co_or_zip08.csv")
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
#master_breweries = pd.read_csv("C:/Users/court/Downloads/Unstructered_final_project/coloradobrewsxawards.csv")
master_breweries = master_brewery_list_final
#poverty_data = pd.read_csv("C:/Users/court/Downloads/poverty.csv")
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

    C:\Users\court\AppData\Local\Temp\ipykernel_11536\1619414278.py:6: SettingWithCopyWarning:


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

# Analysis of Brewery Distribution and Socioeconomic Factors

## Quick Stats

### Data Preparation
```python
# Drop duplicate brewery names for basic calculations
Final_Join_unique = Final_Join.drop_duplicates(subset=['name'])