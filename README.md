# The-Draft-Gap-Unstructured-Data-Analysis-
The project aims to understand the relationship between poverty rate and the beer industry in Colorado.


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
