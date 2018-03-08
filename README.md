
# Analysis
* Observed Trend 1: Cities that are in a lower latitude are currently warmer than cities that are in a higher latitude. This correlates with the fact that cities above the equator are in the winter season and cities below it are in the summer season.
* Observed Trend 2 - Currently, the wind speeds in cities above the equator are more likely to be higher than cities in the lower hemisphere.
* Observed Trend 3 - Cloudiness is more scattered across the cities regardless if where they are in latitude.


```python
#Dependencies
import pandas as pd
import matplotlib.pyplot as plt
from citipy import citipy
import random
import gzip
import json
import requests
import time
```


```python
# API Key
from config import (api_key)
```

# Generate Cities List


```python
# Import Json Data of all Cities from api.openweathermap.org 
jsonfilename="data/city.list.json.gz"
with gzip.GzipFile(jsonfilename, 'r') as data:  
    json_bytes = data.read()                      
    json_str = json_bytes.decode('utf-8')            
    city_json_data = json.loads(json_str) 
```


```python
# Use lists for data 
city_id=[city_json_data[city]['id'] for city in range(len(city_json_data))]
city_name=[city_json_data[city]['name'] for city in range(len(city_json_data))]
city_country=[city_json_data[city]['country'] for city in range(len(city_json_data))]
city_cord_lat=[city_json_data[city]['coord']['lat'] for city in range(len(city_json_data))]
city_cord_lon=[city_json_data[city]['coord']['lon'] for city in range(len(city_json_data))]
```


```python
# Make data frame
city_data = pd.DataFrame(
    {'id': city_id,
     'name': city_name,
     'country': city_country,
     'lat': city_cord_lat,
     'lng' : city_cord_lon
    })
city_data.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>country</th>
      <th>id</th>
      <th>lat</th>
      <th>lng</th>
      <th>name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>UA</td>
      <td>707860</td>
      <td>44.549999</td>
      <td>34.283333</td>
      <td>Hurzuf</td>
    </tr>
    <tr>
      <th>1</th>
      <td>RU</td>
      <td>519188</td>
      <td>55.683334</td>
      <td>37.666668</td>
      <td>Novinki</td>
    </tr>
    <tr>
      <th>2</th>
      <td>NP</td>
      <td>1283378</td>
      <td>28.000000</td>
      <td>84.633331</td>
      <td>Gorkhā</td>
    </tr>
    <tr>
      <th>3</th>
      <td>IN</td>
      <td>1270260</td>
      <td>29.000000</td>
      <td>76.000000</td>
      <td>State of Haryāna</td>
    </tr>
    <tr>
      <th>4</th>
      <td>UA</td>
      <td>708546</td>
      <td>44.599998</td>
      <td>33.900002</td>
      <td>Holubynka</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Select 500 random cities
random_cities = city_data.sample(n=500)
random_cities.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>country</th>
      <th>id</th>
      <th>lat</th>
      <th>lng</th>
      <th>name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>147097</th>
      <td>ES</td>
      <td>3125529</td>
      <td>40.437080</td>
      <td>-0.851500</td>
      <td>Cedrillas</td>
    </tr>
    <tr>
      <th>90834</th>
      <td>BR</td>
      <td>3471840</td>
      <td>-15.897500</td>
      <td>-52.250832</td>
      <td>Aragarcas</td>
    </tr>
    <tr>
      <th>40689</th>
      <td>AU</td>
      <td>2069358</td>
      <td>-35.500000</td>
      <td>138.449997</td>
      <td>Inman Valley</td>
    </tr>
    <tr>
      <th>200468</th>
      <td>US</td>
      <td>4710697</td>
      <td>30.917669</td>
      <td>-99.786461</td>
      <td>Menard</td>
    </tr>
    <tr>
      <th>104176</th>
      <td>SY</td>
      <td>166365</td>
      <td>32.890442</td>
      <td>36.039902</td>
      <td>Nawa</td>
    </tr>
  </tbody>
</table>
</div>



# Perform API Calls


```python
# Set-up lists to collect data and make API call

#Counter
city_count = 1

# Create blank columns for fields
random_cities["Temperature"] = ""
random_cities["Humidity"] = ""
random_cities["Cloudiness"] = ""
random_cities["Wind Speed"] = ""

# Loop through and obtain the weather data using the Open Weather API.
for index, row in random_cities.iterrows():
    # Use time function to keep API call within limits 
    time.sleep(2)
    
    # Open url and use Imperial for (F)
    url = "https://api.openweathermap.org/data/2.5/weather?"
    units = "Imperial"   
    query_url = url + "lat="+ str(row["lat"]) + "&lon=" + str(row["lng"]) + "&appid=" + api_key + "&units=" + units
    
    # Print records log
    print("Processing Record #" + str(city_count)) 
    print(query_url)
    city_count += 1
    
    # Requests to obtain JSON data from Open Weather
    city_weather = requests.get(query_url).json()
     
    #Append the weather data to the appropriate columns.
    #Create debug statement.
    try:
        temperature = city_weather["main"]["temp"]
        humidity = city_weather["main"]["humidity"]
        cloudiness = city_weather["clouds"]["all"]
        wind_speed = city_weather["wind"]["speed"]
        
        random_cities.set_value(index, "Temperature", temperature)
        random_cities.set_value(index,"Humidity", humidity)
        random_cities.set_value(index,"Cloudiness", cloudiness)
        random_cities.set_value(index,"Wind Speed", wind_speed)
    except:
        print("Error, skipping city.")
```

    Processing Record #1
    https://api.openweathermap.org/data/2.5/weather?lat=40.43708&lon=-0.8515&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #2
    https://api.openweathermap.org/data/2.5/weather?lat=-15.8975&lon=-52.250832&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #3
    https://api.openweathermap.org/data/2.5/weather?lat=-35.5&lon=138.449997&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #4
    https://api.openweathermap.org/data/2.5/weather?lat=30.917669&lon=-99.786461&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #5
    https://api.openweathermap.org/data/2.5/weather?lat=32.890442&lon=36.039902&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #6
    https://api.openweathermap.org/data/2.5/weather?lat=49.986141&lon=1.55624&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #7
    https://api.openweathermap.org/data/2.5/weather?lat=52.416672&lon=8.71667&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #8
    https://api.openweathermap.org/data/2.5/weather?lat=37.774929&lon=-122.419418&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #9
    https://api.openweathermap.org/data/2.5/weather?lat=48.232159&lon=7.36843&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #10
    https://api.openweathermap.org/data/2.5/weather?lat=49.416672&lon=-1.53333&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #11
    https://api.openweathermap.org/data/2.5/weather?lat=43.48333&lon=5.5&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #12
    https://api.openweathermap.org/data/2.5/weather?lat=38.774479&lon=-92.257133&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #13
    https://api.openweathermap.org/data/2.5/weather?lat=-17.96785&lon=145.965607&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #14
    https://api.openweathermap.org/data/2.5/weather?lat=68.832222&lon=16.48917&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #15
    https://api.openweathermap.org/data/2.5/weather?lat=51.359081&lon=1.43938&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #16
    https://api.openweathermap.org/data/2.5/weather?lat=32.92775&lon=66.63253&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #17
    https://api.openweathermap.org/data/2.5/weather?lat=48.950001&lon=2.31667&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #18
    https://api.openweathermap.org/data/2.5/weather?lat=40.95211&lon=14.55144&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #19
    https://api.openweathermap.org/data/2.5/weather?lat=31.73333&lon=130.666672&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #20
    https://api.openweathermap.org/data/2.5/weather?lat=37.747452&lon=14.39727&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #21
    https://api.openweathermap.org/data/2.5/weather?lat=52.700001&lon=23.866671&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #22
    https://api.openweathermap.org/data/2.5/weather?lat=46.0&lon=6.66667&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #23
    https://api.openweathermap.org/data/2.5/weather?lat=48.694248&lon=15.52008&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #24
    https://api.openweathermap.org/data/2.5/weather?lat=38.353981&lon=16.2883&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #25
    https://api.openweathermap.org/data/2.5/weather?lat=-34.849998&lon=138.783325&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #26
    https://api.openweathermap.org/data/2.5/weather?lat=50.033329&lon=7.53333&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #27
    https://api.openweathermap.org/data/2.5/weather?lat=49.783298&lon=7.96667&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #28
    https://api.openweathermap.org/data/2.5/weather?lat=22.871389&lon=-82.423889&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #29
    https://api.openweathermap.org/data/2.5/weather?lat=52.04417&lon=4.575&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #30
    https://api.openweathermap.org/data/2.5/weather?lat=41.719669&lon=-1.02812&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #31
    https://api.openweathermap.org/data/2.5/weather?lat=51.133331&lon=7.58333&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #32
    https://api.openweathermap.org/data/2.5/weather?lat=14.98333&lon=102.283333&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #33
    https://api.openweathermap.org/data/2.5/weather?lat=-7.4249&lon=108.474197&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #34
    https://api.openweathermap.org/data/2.5/weather?lat=49.201672&lon=0.53675&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #35
    https://api.openweathermap.org/data/2.5/weather?lat=49.483299&lon=8.795&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #36
    https://api.openweathermap.org/data/2.5/weather?lat=52.640461&lon=11.96154&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #37
    https://api.openweathermap.org/data/2.5/weather?lat=25.984541&lon=-80.198936&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #38
    https://api.openweathermap.org/data/2.5/weather?lat=32.327709&lon=105.032341&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #39
    https://api.openweathermap.org/data/2.5/weather?lat=20.75&lon=-97.616669&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #40
    https://api.openweathermap.org/data/2.5/weather?lat=51.57526&lon=-2.81453&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #41
    https://api.openweathermap.org/data/2.5/weather?lat=47.783192&lon=-53.164768&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #42
    https://api.openweathermap.org/data/2.5/weather?lat=42.781479&lon=-7.41431&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #43
    https://api.openweathermap.org/data/2.5/weather?lat=46.0&lon=14.0&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #44
    https://api.openweathermap.org/data/2.5/weather?lat=49.738331&lon=5.34104&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #45
    https://api.openweathermap.org/data/2.5/weather?lat=11.2135&lon=124.392097&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #46
    https://api.openweathermap.org/data/2.5/weather?lat=41.165279&lon=121.366669&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #47
    https://api.openweathermap.org/data/2.5/weather?lat=10.6043&lon=123.744102&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #48
    https://api.openweathermap.org/data/2.5/weather?lat=48.26667&lon=12.76667&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #49
    https://api.openweathermap.org/data/2.5/weather?lat=41.860619&lon=13.03309&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #50
    https://api.openweathermap.org/data/2.5/weather?lat=48.730579&lon=-3.24208&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #51
    https://api.openweathermap.org/data/2.5/weather?lat=50.299999&lon=3.45&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #52
    https://api.openweathermap.org/data/2.5/weather?lat=27.16783&lon=-80.266159&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #53
    https://api.openweathermap.org/data/2.5/weather?lat=-41.48333&lon=147.483337&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #54
    https://api.openweathermap.org/data/2.5/weather?lat=37.933331&lon=21.26667&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #55
    https://api.openweathermap.org/data/2.5/weather?lat=12.77944&lon=45.036671&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #56
    https://api.openweathermap.org/data/2.5/weather?lat=41.683609&lon=21.144171&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #57
    https://api.openweathermap.org/data/2.5/weather?lat=51.187832&lon=7.7465&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #58
    https://api.openweathermap.org/data/2.5/weather?lat=49.472252&lon=3.00939&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #59
    https://api.openweathermap.org/data/2.5/weather?lat=51.049999&lon=7.5&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #60
    https://api.openweathermap.org/data/2.5/weather?lat=52.98333&lon=6.55&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #61
    https://api.openweathermap.org/data/2.5/weather?lat=50.616699&lon=11.85&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #62
    https://api.openweathermap.org/data/2.5/weather?lat=51.671089&lon=-1.28278&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #63
    https://api.openweathermap.org/data/2.5/weather?lat=19.25&lon=-97.783333&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #64
    https://api.openweathermap.org/data/2.5/weather?lat=25.493731&lon=-80.391319&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #65
    https://api.openweathermap.org/data/2.5/weather?lat=49.08083&lon=8.06722&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #66
    https://api.openweathermap.org/data/2.5/weather?lat=53.727798&lon=7.36965&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #67
    https://api.openweathermap.org/data/2.5/weather?lat=46.533329&lon=76.199997&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #68
    https://api.openweathermap.org/data/2.5/weather?lat=26.218889&lon=115.436394&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #69
    https://api.openweathermap.org/data/2.5/weather?lat=48.083302&lon=11.7167&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #70
    https://api.openweathermap.org/data/2.5/weather?lat=56.033329&lon=37.216671&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #71
    https://api.openweathermap.org/data/2.5/weather?lat=38.716671&lon=-9.41667&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #72
    https://api.openweathermap.org/data/2.5/weather?lat=-31.960569&lon=-61.178028&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #73
    https://api.openweathermap.org/data/2.5/weather?lat=40.874439&lon=37.263889&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #74
    https://api.openweathermap.org/data/2.5/weather?lat=47.216671&lon=9.51667&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #75
    https://api.openweathermap.org/data/2.5/weather?lat=41.916672&lon=23.58333&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #76
    https://api.openweathermap.org/data/2.5/weather?lat=39.556789&lon=-0.39124&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #77
    https://api.openweathermap.org/data/2.5/weather?lat=42.55162&lon=-2.27285&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #78
    https://api.openweathermap.org/data/2.5/weather?lat=56.772758&lon=9.33926&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #79
    https://api.openweathermap.org/data/2.5/weather?lat=50.128689&lon=30.24029&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #80
    https://api.openweathermap.org/data/2.5/weather?lat=-21.8675&lon=-51.728611&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #81
    https://api.openweathermap.org/data/2.5/weather?lat=24.816669&lon=87.900002&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #82
    https://api.openweathermap.org/data/2.5/weather?lat=40.680382&lon=-73.455116&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #83
    https://api.openweathermap.org/data/2.5/weather?lat=52.849239&lon=-3.69857&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #84
    https://api.openweathermap.org/data/2.5/weather?lat=6.8293&lon=79.862999&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #85
    https://api.openweathermap.org/data/2.5/weather?lat=45.283211&lon=4.39752&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #86
    https://api.openweathermap.org/data/2.5/weather?lat=53.283329&lon=11.11667&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #87
    https://api.openweathermap.org/data/2.5/weather?lat=53.950001&lon=13.2833&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #88
    https://api.openweathermap.org/data/2.5/weather?lat=38.51667&lon=141.321671&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #89
    https://api.openweathermap.org/data/2.5/weather?lat=49.833328&lon=3.5&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #90
    https://api.openweathermap.org/data/2.5/weather?lat=13.56972&lon=-89.117218&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #91
    https://api.openweathermap.org/data/2.5/weather?lat=43.088581&lon=-1.77946&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #92
    https://api.openweathermap.org/data/2.5/weather?lat=40.055592&lon=-3.71843&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #93
    https://api.openweathermap.org/data/2.5/weather?lat=-25.477409&lon=-64.964111&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #94
    https://api.openweathermap.org/data/2.5/weather?lat=48.335678&lon=0.83454&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #95
    https://api.openweathermap.org/data/2.5/weather?lat=47.716671&lon=-0.91667&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #96
    https://api.openweathermap.org/data/2.5/weather?lat=10.78085&lon=122.389397&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #97
    https://api.openweathermap.org/data/2.5/weather?lat=9.73333&lon=77.300003&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #98
    https://api.openweathermap.org/data/2.5/weather?lat=53.799999&lon=87.279999&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #99
    https://api.openweathermap.org/data/2.5/weather?lat=57.438938&lon=12.11131&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #100
    https://api.openweathermap.org/data/2.5/weather?lat=50.866669&lon=6.75&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #101
    https://api.openweathermap.org/data/2.5/weather?lat=6.17848&lon=-73.589478&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #102
    https://api.openweathermap.org/data/2.5/weather?lat=40.652061&lon=22.97547&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #103
    https://api.openweathermap.org/data/2.5/weather?lat=61.184608&lon=6.85016&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #104
    https://api.openweathermap.org/data/2.5/weather?lat=54.099998&lon=10.35&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #105
    https://api.openweathermap.org/data/2.5/weather?lat=45.116669&lon=0.71667&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #106
    https://api.openweathermap.org/data/2.5/weather?lat=41.966671&lon=44.099998&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #107
    https://api.openweathermap.org/data/2.5/weather?lat=55.371941&lon=21.06472&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #108
    https://api.openweathermap.org/data/2.5/weather?lat=28.55805&lon=-81.851189&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #109
    https://api.openweathermap.org/data/2.5/weather?lat=49.23333&lon=10.83333&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #110
    https://api.openweathermap.org/data/2.5/weather?lat=50.549999&lon=3.03333&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #111
    https://api.openweathermap.org/data/2.5/weather?lat=48.60767&lon=-2.1503&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #112
    https://api.openweathermap.org/data/2.5/weather?lat=46.754509&lon=33.34864&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #113
    https://api.openweathermap.org/data/2.5/weather?lat=-2.46397&lon=29.573891&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #114
    https://api.openweathermap.org/data/2.5/weather?lat=-8.6562&lon=120.594101&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #115
    https://api.openweathermap.org/data/2.5/weather?lat=51.32967&lon=8.0071&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #116
    https://api.openweathermap.org/data/2.5/weather?lat=13.7351&lon=124.1064&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #117
    https://api.openweathermap.org/data/2.5/weather?lat=40.060841&lon=-95.601929&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #118
    https://api.openweathermap.org/data/2.5/weather?lat=-15.3175&lon=-49.1175&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #119
    https://api.openweathermap.org/data/2.5/weather?lat=47.035992&lon=-52.885559&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #120
    https://api.openweathermap.org/data/2.5/weather?lat=47.384209&lon=8.12284&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #121
    https://api.openweathermap.org/data/2.5/weather?lat=51.367699&lon=-1.71869&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #122
    https://api.openweathermap.org/data/2.5/weather?lat=30.587139&lon=-84.583252&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #123
    https://api.openweathermap.org/data/2.5/weather?lat=39.353779&lon=3.12819&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #124
    https://api.openweathermap.org/data/2.5/weather?lat=48.0606&lon=13.75406&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #125
    https://api.openweathermap.org/data/2.5/weather?lat=37.620762&lon=-120.985771&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #126
    https://api.openweathermap.org/data/2.5/weather?lat=52.235001&lon=6.49861&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #127
    https://api.openweathermap.org/data/2.5/weather?lat=46.886391&lon=13.51028&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #128
    https://api.openweathermap.org/data/2.5/weather?lat=18.85&lon=-97.066673&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #129
    https://api.openweathermap.org/data/2.5/weather?lat=40.40028&lon=30.78833&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #130
    https://api.openweathermap.org/data/2.5/weather?lat=40.444321&lon=-7.85916&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #131
    https://api.openweathermap.org/data/2.5/weather?lat=53.48333&lon=7.98333&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #132
    https://api.openweathermap.org/data/2.5/weather?lat=51.576462&lon=-0.39737&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #133
    https://api.openweathermap.org/data/2.5/weather?lat=49.366669&lon=11.68333&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #134
    https://api.openweathermap.org/data/2.5/weather?lat=0.8&lon=-77.716667&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #135
    https://api.openweathermap.org/data/2.5/weather?lat=25.633329&lon=83.75&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #136
    https://api.openweathermap.org/data/2.5/weather?lat=44.492142&lon=8.41435&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #137
    https://api.openweathermap.org/data/2.5/weather?lat=-34.833462&lon=-56.167351&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #138
    https://api.openweathermap.org/data/2.5/weather?lat=46.75&lon=9.1&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #139
    https://api.openweathermap.org/data/2.5/weather?lat=47.816669&lon=-2.13333&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #140
    https://api.openweathermap.org/data/2.5/weather?lat=-28.33333&lon=153.5&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #141
    https://api.openweathermap.org/data/2.5/weather?lat=48.76667&lon=11.53333&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #142
    https://api.openweathermap.org/data/2.5/weather?lat=29.216669&lon=78.583328&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #143
    https://api.openweathermap.org/data/2.5/weather?lat=-3.69528&lon=-42.785&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #144
    https://api.openweathermap.org/data/2.5/weather?lat=51.436192&lon=-0.30933&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #145
    https://api.openweathermap.org/data/2.5/weather?lat=48.71067&lon=9.41949&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #146
    https://api.openweathermap.org/data/2.5/weather?lat=47.826469&lon=7.25235&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #147
    https://api.openweathermap.org/data/2.5/weather?lat=35.313709&lon=-83.176529&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #148
    https://api.openweathermap.org/data/2.5/weather?lat=50.771099&lon=19.6924&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #149
    https://api.openweathermap.org/data/2.5/weather?lat=41.582161&lon=-3.7955&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #150
    https://api.openweathermap.org/data/2.5/weather?lat=-42.450001&lon=147.449997&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #151
    https://api.openweathermap.org/data/2.5/weather?lat=45.416672&lon=2.83333&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #152
    https://api.openweathermap.org/data/2.5/weather?lat=50.48333&lon=2.66667&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #153
    https://api.openweathermap.org/data/2.5/weather?lat=44.599998&lon=25.716669&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #154
    https://api.openweathermap.org/data/2.5/weather?lat=50.833328&lon=14.76667&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #155
    https://api.openweathermap.org/data/2.5/weather?lat=43.672958&lon=12.96866&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #156
    https://api.openweathermap.org/data/2.5/weather?lat=33.072369&lon=119.445&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #157
    https://api.openweathermap.org/data/2.5/weather?lat=39.892029&lon=-2.55786&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #158
    https://api.openweathermap.org/data/2.5/weather?lat=52.774261&lon=-3.31067&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #159
    https://api.openweathermap.org/data/2.5/weather?lat=42.26667&lon=24.16667&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #160
    https://api.openweathermap.org/data/2.5/weather?lat=56.966671&lon=61.466671&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #161
    https://api.openweathermap.org/data/2.5/weather?lat=51.116669&lon=5.25&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #162
    https://api.openweathermap.org/data/2.5/weather?lat=41.08083&lon=-8.59776&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #163
    https://api.openweathermap.org/data/2.5/weather?lat=32.422279&lon=35.384762&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #164
    https://api.openweathermap.org/data/2.5/weather?lat=43.116669&lon=70.349998&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #165
    https://api.openweathermap.org/data/2.5/weather?lat=48.649929&lon=-0.14424&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #166
    https://api.openweathermap.org/data/2.5/weather?lat=58.363049&lon=11.25938&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #167
    https://api.openweathermap.org/data/2.5/weather?lat=45.376667&lon=38.451668&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #168
    https://api.openweathermap.org/data/2.5/weather?lat=45.542469&lon=8.47973&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #169
    https://api.openweathermap.org/data/2.5/weather?lat=49.1222&lon=-0.91925&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #170
    https://api.openweathermap.org/data/2.5/weather?lat=66.383331&lon=23.66667&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #171
    https://api.openweathermap.org/data/2.5/weather?lat=50.0667&lon=6.48333&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #172
    https://api.openweathermap.org/data/2.5/weather?lat=53.80267&lon=9.2368&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #173
    https://api.openweathermap.org/data/2.5/weather?lat=56.012348&lon=47.556149&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #174
    https://api.openweathermap.org/data/2.5/weather?lat=9.8334&lon=123.950897&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #175
    https://api.openweathermap.org/data/2.5/weather?lat=49.076359&lon=7.00487&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #176
    https://api.openweathermap.org/data/2.5/weather?lat=49.183331&lon=12.35&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #177
    https://api.openweathermap.org/data/2.5/weather?lat=50.150002&lon=8.15&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #178
    https://api.openweathermap.org/data/2.5/weather?lat=14.61544&lon=100.72731&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #179
    https://api.openweathermap.org/data/2.5/weather?lat=46.666672&lon=7.16667&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #180
    https://api.openweathermap.org/data/2.5/weather?lat=53.496422&lon=13.94697&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #181
    https://api.openweathermap.org/data/2.5/weather?lat=55.619999&lon=36.57&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #182
    https://api.openweathermap.org/data/2.5/weather?lat=56.10611&lon=42.472221&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #183
    https://api.openweathermap.org/data/2.5/weather?lat=49.68861&lon=5.91472&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #184
    https://api.openweathermap.org/data/2.5/weather?lat=35.577831&lon=-98.964531&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #185
    https://api.openweathermap.org/data/2.5/weather?lat=52.04583&lon=4.95278&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #186
    https://api.openweathermap.org/data/2.5/weather?lat=49.1077&lon=16.060671&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #187
    https://api.openweathermap.org/data/2.5/weather?lat=50.5&lon=10.86667&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #188
    https://api.openweathermap.org/data/2.5/weather?lat=16.01667&lon=75.566673&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #189
    https://api.openweathermap.org/data/2.5/weather?lat=48.666672&lon=10.86667&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #190
    https://api.openweathermap.org/data/2.5/weather?lat=42.557079&lon=-6.7309&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #191
    https://api.openweathermap.org/data/2.5/weather?lat=-33.369499&lon=149.134293&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #192
    https://api.openweathermap.org/data/2.5/weather?lat=54.057831&lon=12.49223&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #193
    https://api.openweathermap.org/data/2.5/weather?lat=51.450001&lon=12.93333&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #194
    https://api.openweathermap.org/data/2.5/weather?lat=45.23333&lon=3.48333&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #195
    https://api.openweathermap.org/data/2.5/weather?lat=29.998199&lon=120.55912&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #196
    https://api.openweathermap.org/data/2.5/weather?lat=-17.83333&lon=25.85&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #197
    https://api.openweathermap.org/data/2.5/weather?lat=16.383329&lon=95.26667&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #198
    https://api.openweathermap.org/data/2.5/weather?lat=-29.909149&lon=-50.953011&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #199
    https://api.openweathermap.org/data/2.5/weather?lat=52.209721&lon=-7.42583&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #200
    https://api.openweathermap.org/data/2.5/weather?lat=23.0681&lon=114.01236&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #201
    https://api.openweathermap.org/data/2.5/weather?lat=43.816341&lon=5.74638&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #202
    https://api.openweathermap.org/data/2.5/weather?lat=54.320381&lon=19.526951&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #203
    https://api.openweathermap.org/data/2.5/weather?lat=48.75061&lon=-3.28121&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #204
    https://api.openweathermap.org/data/2.5/weather?lat=46.650002&lon=39.957199&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #205
    https://api.openweathermap.org/data/2.5/weather?lat=51.133331&lon=13.5&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #206
    https://api.openweathermap.org/data/2.5/weather?lat=47.666698&lon=11.0333&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #207
    https://api.openweathermap.org/data/2.5/weather?lat=-27.183331&lon=31.383329&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #208
    https://api.openweathermap.org/data/2.5/weather?lat=-29.516939&lon=-50.777779&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #209
    https://api.openweathermap.org/data/2.5/weather?lat=41.60046&lon=-1.28007&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #210
    https://api.openweathermap.org/data/2.5/weather?lat=51.370831&lon=4.01389&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #211
    https://api.openweathermap.org/data/2.5/weather?lat=54.566669&lon=9.68333&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #212
    https://api.openweathermap.org/data/2.5/weather?lat=49.958618&lon=19.946541&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #213
    https://api.openweathermap.org/data/2.5/weather?lat=27.9&lon=86.833336&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #214
    https://api.openweathermap.org/data/2.5/weather?lat=14.78333&lon=-86.0&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #215
    https://api.openweathermap.org/data/2.5/weather?lat=52.866119&lon=12.81341&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #216
    https://api.openweathermap.org/data/2.5/weather?lat=-8.6995&lon=118.799103&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #217
    https://api.openweathermap.org/data/2.5/weather?lat=51.616669&lon=7.51667&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #218
    https://api.openweathermap.org/data/2.5/weather?lat=46.463329&lon=12.20056&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #219
    https://api.openweathermap.org/data/2.5/weather?lat=43.321789&lon=129.763412&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #220
    https://api.openweathermap.org/data/2.5/weather?lat=8.61667&lon=-80.599998&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #221
    https://api.openweathermap.org/data/2.5/weather?lat=47.200001&lon=14.68333&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #222
    https://api.openweathermap.org/data/2.5/weather?lat=57.387222&lon=41.3675&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #223
    https://api.openweathermap.org/data/2.5/weather?lat=45.933331&lon=2.58333&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #224
    https://api.openweathermap.org/data/2.5/weather?lat=42.23011&lon=2.82048&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #225
    https://api.openweathermap.org/data/2.5/weather?lat=20.549999&lon=-102.51667&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #226
    https://api.openweathermap.org/data/2.5/weather?lat=17.162609&lon=102.572723&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #227
    https://api.openweathermap.org/data/2.5/weather?lat=51.683331&lon=11.36667&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #228
    https://api.openweathermap.org/data/2.5/weather?lat=11.7666&lon=122.176498&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #229
    https://api.openweathermap.org/data/2.5/weather?lat=51.506672&lon=10.75194&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #230
    https://api.openweathermap.org/data/2.5/weather?lat=-21.429171&lon=-45.94722&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #231
    https://api.openweathermap.org/data/2.5/weather?lat=41.038689&lon=-3.9186&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #232
    https://api.openweathermap.org/data/2.5/weather?lat=50.93388&lon=-0.18007&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #233
    https://api.openweathermap.org/data/2.5/weather?lat=53.683331&lon=9.66667&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #234
    https://api.openweathermap.org/data/2.5/weather?lat=42.75946&lon=127.800598&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #235
    https://api.openweathermap.org/data/2.5/weather?lat=-12.15556&lon=44.437222&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #236
    https://api.openweathermap.org/data/2.5/weather?lat=41.262669&lon=1.1701&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #237
    https://api.openweathermap.org/data/2.5/weather?lat=37.684219&lon=-88.632828&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #238
    https://api.openweathermap.org/data/2.5/weather?lat=-6.13139&lon=106.379723&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #239
    https://api.openweathermap.org/data/2.5/weather?lat=34.83287&lon=-89.175903&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #240
    https://api.openweathermap.org/data/2.5/weather?lat=57.655277&lon=43.58139&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #241
    https://api.openweathermap.org/data/2.5/weather?lat=36.836128&lon=-3.67426&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #242
    https://api.openweathermap.org/data/2.5/weather?lat=48.7327&lon=13.60082&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #243
    https://api.openweathermap.org/data/2.5/weather?lat=46.6311&lon=38.674198&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #244
    https://api.openweathermap.org/data/2.5/weather?lat=56.585556&lon=43.688889&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #245
    https://api.openweathermap.org/data/2.5/weather?lat=50.73333&lon=9.96667&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #246
    https://api.openweathermap.org/data/2.5/weather?lat=38.248291&lon=-3.20762&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #247
    https://api.openweathermap.org/data/2.5/weather?lat=49.003422&lon=11.66388&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #248
    https://api.openweathermap.org/data/2.5/weather?lat=53.877102&lon=9.48079&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #249
    https://api.openweathermap.org/data/2.5/weather?lat=31.67252&lon=38.663738&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #250
    https://api.openweathermap.org/data/2.5/weather?lat=35.383289&lon=-119.109833&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #251
    https://api.openweathermap.org/data/2.5/weather?lat=40.757118&lon=125.427856&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #252
    https://api.openweathermap.org/data/2.5/weather?lat=41.856491&lon=-3.26815&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #253
    https://api.openweathermap.org/data/2.5/weather?lat=46.86203&lon=3.64652&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #254
    https://api.openweathermap.org/data/2.5/weather?lat=47.997879&lon=-3.318&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #255
    https://api.openweathermap.org/data/2.5/weather?lat=55.39996&lon=40.301971&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #256
    https://api.openweathermap.org/data/2.5/weather?lat=-25.85891&lon=28.18577&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #257
    https://api.openweathermap.org/data/2.5/weather?lat=27.61171&lon=118.97142&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #258
    https://api.openweathermap.org/data/2.5/weather?lat=45.69088&lon=9.39925&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #259
    https://api.openweathermap.org/data/2.5/weather?lat=-27.783331&lon=153.316666&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #260
    https://api.openweathermap.org/data/2.5/weather?lat=9.9959&lon=-84.051262&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #261
    https://api.openweathermap.org/data/2.5/weather?lat=44.133411&lon=-79.416313&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #262
    https://api.openweathermap.org/data/2.5/weather?lat=-8.405&lon=120.523308&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #263
    https://api.openweathermap.org/data/2.5/weather?lat=53.75&lon=12.7&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #264
    https://api.openweathermap.org/data/2.5/weather?lat=-35.758369&lon=147.197311&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #265
    https://api.openweathermap.org/data/2.5/weather?lat=48.400002&lon=13.31667&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #266
    https://api.openweathermap.org/data/2.5/weather?lat=46.823299&lon=15.13565&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #267
    https://api.openweathermap.org/data/2.5/weather?lat=54.166672&lon=8.93333&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #268
    https://api.openweathermap.org/data/2.5/weather?lat=51.049999&lon=11.7&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #269
    https://api.openweathermap.org/data/2.5/weather?lat=11.71806&lon=122.378059&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #270
    https://api.openweathermap.org/data/2.5/weather?lat=46.160831&lon=15.87889&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #271
    https://api.openweathermap.org/data/2.5/weather?lat=48.051208&lon=23.48579&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #272
    https://api.openweathermap.org/data/2.5/weather?lat=32.253941&lon=-8.53351&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #273
    https://api.openweathermap.org/data/2.5/weather?lat=52.583328&lon=9.15&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #274
    https://api.openweathermap.org/data/2.5/weather?lat=53.045441&lon=-0.38692&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #275
    https://api.openweathermap.org/data/2.5/weather?lat=-33.049999&lon=135.466675&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #276
    https://api.openweathermap.org/data/2.5/weather?lat=35.613468&lon=110.958542&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #277
    https://api.openweathermap.org/data/2.5/weather?lat=11.41667&lon=-3.41667&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #278
    https://api.openweathermap.org/data/2.5/weather?lat=-7.2174&lon=108.830101&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #279
    https://api.openweathermap.org/data/2.5/weather?lat=-30.35&lon=117.116669&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #280
    https://api.openweathermap.org/data/2.5/weather?lat=33.88306&lon=-4.1862&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #281
    https://api.openweathermap.org/data/2.5/weather?lat=25.73333&lon=85.0&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #282
    https://api.openweathermap.org/data/2.5/weather?lat=44.40823&lon=7.32293&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #283
    https://api.openweathermap.org/data/2.5/weather?lat=50.187431&lon=12.90333&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #284
    https://api.openweathermap.org/data/2.5/weather?lat=44.994041&lon=7.64182&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #285
    https://api.openweathermap.org/data/2.5/weather?lat=46.32159&lon=9.39864&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #286
    https://api.openweathermap.org/data/2.5/weather?lat=54.441799&lon=18.56003&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #287
    https://api.openweathermap.org/data/2.5/weather?lat=44.275002&lon=39.293056&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #288
    https://api.openweathermap.org/data/2.5/weather?lat=42.916672&lon=24.26667&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #289
    https://api.openweathermap.org/data/2.5/weather?lat=49.833302&lon=7.95&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #290
    https://api.openweathermap.org/data/2.5/weather?lat=54.01907&lon=12.84842&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #291
    https://api.openweathermap.org/data/2.5/weather?lat=-2.59833&lon=-43.461109&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #292
    https://api.openweathermap.org/data/2.5/weather?lat=-7.2609&lon=106.727097&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #293
    https://api.openweathermap.org/data/2.5/weather?lat=51.900002&lon=12.9&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #294
    https://api.openweathermap.org/data/2.5/weather?lat=35.299999&lon=10.71667&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #295
    https://api.openweathermap.org/data/2.5/weather?lat=45.717098&lon=2.73432&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #296
    https://api.openweathermap.org/data/2.5/weather?lat=41.10236&lon=-4.0328&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #297
    https://api.openweathermap.org/data/2.5/weather?lat=12.51667&lon=37.433331&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #298
    https://api.openweathermap.org/data/2.5/weather?lat=47.148571&lon=5.76051&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #299
    https://api.openweathermap.org/data/2.5/weather?lat=33.632332&lon=-92.76683&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #300
    https://api.openweathermap.org/data/2.5/weather?lat=47.647881&lon=-121.914009&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #301
    https://api.openweathermap.org/data/2.5/weather?lat=35.058331&lon=33.783329&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #302
    https://api.openweathermap.org/data/2.5/weather?lat=35.379269&lon=105.009178&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #303
    https://api.openweathermap.org/data/2.5/weather?lat=-8.0878&lon=112.670403&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #304
    https://api.openweathermap.org/data/2.5/weather?lat=62.5&lon=24.83333&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #305
    https://api.openweathermap.org/data/2.5/weather?lat=48.200001&lon=16.25&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #306
    https://api.openweathermap.org/data/2.5/weather?lat=41.48333&lon=23.23333&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #307
    https://api.openweathermap.org/data/2.5/weather?lat=43.120651&lon=13.48758&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #308
    https://api.openweathermap.org/data/2.5/weather?lat=1.35806&lon=103.940277&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #309
    https://api.openweathermap.org/data/2.5/weather?lat=62.566669&lon=25.866671&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #310
    https://api.openweathermap.org/data/2.5/weather?lat=52.75&lon=7.2&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #311
    https://api.openweathermap.org/data/2.5/weather?lat=46.90802&lon=16.122549&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #312
    https://api.openweathermap.org/data/2.5/weather?lat=58.15028&lon=24.96417&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #313
    https://api.openweathermap.org/data/2.5/weather?lat=40.613022&lon=-5.13156&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #314
    https://api.openweathermap.org/data/2.5/weather?lat=39.944279&lon=-74.072906&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #315
    https://api.openweathermap.org/data/2.5/weather?lat=49.56319&lon=18.90567&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #316
    https://api.openweathermap.org/data/2.5/weather?lat=14.9056&lon=120.694504&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #317
    https://api.openweathermap.org/data/2.5/weather?lat=40.513538&lon=-7.95818&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #318
    https://api.openweathermap.org/data/2.5/weather?lat=39.061119&lon=-94.819679&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #319
    https://api.openweathermap.org/data/2.5/weather?lat=15.3338&lon=-15.4766&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #320
    https://api.openweathermap.org/data/2.5/weather?lat=50.31522&lon=11.24399&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #321
    https://api.openweathermap.org/data/2.5/weather?lat=43.051159&lon=-2.11038&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #322
    https://api.openweathermap.org/data/2.5/weather?lat=50.349998&lon=9.3&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #323
    https://api.openweathermap.org/data/2.5/weather?lat=53.866669&lon=12.7&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #324
    https://api.openweathermap.org/data/2.5/weather?lat=-22.9&lon=44.533329&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #325
    https://api.openweathermap.org/data/2.5/weather?lat=36.812462&lon=-121.365768&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #326
    https://api.openweathermap.org/data/2.5/weather?lat=42.905849&lon=-88.138977&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #327
    https://api.openweathermap.org/data/2.5/weather?lat=40.29475&lon=-3.29717&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #328
    https://api.openweathermap.org/data/2.5/weather?lat=52.75&lon=8.91667&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #329
    https://api.openweathermap.org/data/2.5/weather?lat=48.450001&lon=11.0&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #330
    https://api.openweathermap.org/data/2.5/weather?lat=34.446499&lon=-82.39151&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #331
    https://api.openweathermap.org/data/2.5/weather?lat=42.50214&lon=13.7828&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #332
    https://api.openweathermap.org/data/2.5/weather?lat=42.095039&lon=-83.189651&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #333
    https://api.openweathermap.org/data/2.5/weather?lat=54.5667&lon=13.4833&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #334
    https://api.openweathermap.org/data/2.5/weather?lat=39.027569&lon=-83.919647&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #335
    https://api.openweathermap.org/data/2.5/weather?lat=49.612782&lon=11.13222&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #336
    https://api.openweathermap.org/data/2.5/weather?lat=26.91111&lon=119.483063&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #337
    https://api.openweathermap.org/data/2.5/weather?lat=-25.82988&lon=28.2012&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #338
    https://api.openweathermap.org/data/2.5/weather?lat=40.537231&lon=-7.35282&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #339
    https://api.openweathermap.org/data/2.5/weather?lat=50.799999&lon=13.16667&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #340
    https://api.openweathermap.org/data/2.5/weather?lat=51.5&lon=13.8&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #341
    https://api.openweathermap.org/data/2.5/weather?lat=37.527439&lon=-122.513313&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #342
    https://api.openweathermap.org/data/2.5/weather?lat=50.216702&lon=6.5&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #343
    https://api.openweathermap.org/data/2.5/weather?lat=61.216671&lon=26.033331&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #344
    https://api.openweathermap.org/data/2.5/weather?lat=44.299042&lon=-0.47791&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #345
    https://api.openweathermap.org/data/2.5/weather?lat=49.216671&lon=-0.68333&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #346
    https://api.openweathermap.org/data/2.5/weather?lat=49.7239&lon=8.20083&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #347
    https://api.openweathermap.org/data/2.5/weather?lat=50.666672&lon=17.950001&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #348
    https://api.openweathermap.org/data/2.5/weather?lat=49.848061&lon=6.03556&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #349
    https://api.openweathermap.org/data/2.5/weather?lat=44.849998&lon=1.33333&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #350
    https://api.openweathermap.org/data/2.5/weather?lat=47.847389&lon=-2.99981&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #351
    https://api.openweathermap.org/data/2.5/weather?lat=49.26667&lon=7.53333&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #352
    https://api.openweathermap.org/data/2.5/weather?lat=49.650829&lon=6.235&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #353
    https://api.openweathermap.org/data/2.5/weather?lat=-35.116669&lon=147.433334&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #354
    https://api.openweathermap.org/data/2.5/weather?lat=51.666809&lon=-114.135292&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #355
    https://api.openweathermap.org/data/2.5/weather?lat=-21.015829&lon=-49.496109&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #356
    https://api.openweathermap.org/data/2.5/weather?lat=34.467869&lon=-84.429092&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #357
    https://api.openweathermap.org/data/2.5/weather?lat=33.93261&lon=-89.330353&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #358
    https://api.openweathermap.org/data/2.5/weather?lat=48.383331&lon=15.51667&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #359
    https://api.openweathermap.org/data/2.5/weather?lat=45.35817&lon=9.22655&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #360
    https://api.openweathermap.org/data/2.5/weather?lat=56.283333&lon=37.866669&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #361
    https://api.openweathermap.org/data/2.5/weather?lat=33.95932&lon=-81.108978&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #362
    https://api.openweathermap.org/data/2.5/weather?lat=49.9426&lon=20.690519&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #363
    https://api.openweathermap.org/data/2.5/weather?lat=29.278669&lon=114.06778&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #364
    https://api.openweathermap.org/data/2.5/weather?lat=42.311298&lon=-7.96745&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #365
    https://api.openweathermap.org/data/2.5/weather?lat=48.746262&lon=-3.26142&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #366
    https://api.openweathermap.org/data/2.5/weather?lat=32.583939&lon=-117.113083&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #367
    https://api.openweathermap.org/data/2.5/weather?lat=-8.0578&lon=113.797203&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #368
    https://api.openweathermap.org/data/2.5/weather?lat=34.589821&lon=-83.560448&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #369
    https://api.openweathermap.org/data/2.5/weather?lat=43.933331&lon=23.216669&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #370
    https://api.openweathermap.org/data/2.5/weather?lat=47.299999&lon=13.3&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #371
    https://api.openweathermap.org/data/2.5/weather?lat=48.74345&lon=2.06154&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #372
    https://api.openweathermap.org/data/2.5/weather?lat=-16.735001&lon=-40.419998&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #373
    https://api.openweathermap.org/data/2.5/weather?lat=31.510731&lon=-96.424698&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #374
    https://api.openweathermap.org/data/2.5/weather?lat=47.35236&lon=8.27877&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #375
    https://api.openweathermap.org/data/2.5/weather?lat=49.466671&lon=5.81667&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #376
    https://api.openweathermap.org/data/2.5/weather?lat=8.33204&lon=-71.751183&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #377
    https://api.openweathermap.org/data/2.5/weather?lat=35.267681&lon=-75.542374&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #378
    https://api.openweathermap.org/data/2.5/weather?lat=46.716671&lon=2.7&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #379
    https://api.openweathermap.org/data/2.5/weather?lat=28.32362&lon=115.612106&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #380
    https://api.openweathermap.org/data/2.5/weather?lat=50.22805&lon=7.75806&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #381
    https://api.openweathermap.org/data/2.5/weather?lat=59.226391&lon=42.502777&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #382
    https://api.openweathermap.org/data/2.5/weather?lat=54.795834&lon=38.59222&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #383
    https://api.openweathermap.org/data/2.5/weather?lat=51.139992&lon=17.17798&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #384
    https://api.openweathermap.org/data/2.5/weather?lat=52.417549&lon=0.52211&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #385
    https://api.openweathermap.org/data/2.5/weather?lat=47.024288&lon=10.66853&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #386
    https://api.openweathermap.org/data/2.5/weather?lat=52.21965&lon=21.680321&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #387
    https://api.openweathermap.org/data/2.5/weather?lat=31.174509&lon=121.11692&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #388
    https://api.openweathermap.org/data/2.5/weather?lat=31.250441&lon=-99.25061&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #389
    https://api.openweathermap.org/data/2.5/weather?lat=-8.8932&lon=121.161598&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #390
    https://api.openweathermap.org/data/2.5/weather?lat=53.6833&lon=9.61667&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #391
    https://api.openweathermap.org/data/2.5/weather?lat=-8.4629&lon=115.299202&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #392
    https://api.openweathermap.org/data/2.5/weather?lat=-7.5107&lon=108.270103&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #393
    https://api.openweathermap.org/data/2.5/weather?lat=46.416801&lon=-62.448639&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #394
    https://api.openweathermap.org/data/2.5/weather?lat=30.996861&lon=-94.827148&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #395
    https://api.openweathermap.org/data/2.5/weather?lat=40.971882&lon=0.09557&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #396
    https://api.openweathermap.org/data/2.5/weather?lat=51.96336&lon=19.291389&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #397
    https://api.openweathermap.org/data/2.5/weather?lat=19.33333&lon=-99.216667&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #398
    https://api.openweathermap.org/data/2.5/weather?lat=-9.65&lon=-36.48333&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #399
    https://api.openweathermap.org/data/2.5/weather?lat=52.829441&lon=-8.21528&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #400
    https://api.openweathermap.org/data/2.5/weather?lat=43.883331&lon=4.6&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #401
    https://api.openweathermap.org/data/2.5/weather?lat=50.84877&lon=4.34664&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #402
    https://api.openweathermap.org/data/2.5/weather?lat=-12.15833&lon=-39.737221&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #403
    https://api.openweathermap.org/data/2.5/weather?lat=34.217659&lon=41.305698&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #404
    https://api.openweathermap.org/data/2.5/weather?lat=47.231918&lon=7.57002&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #405
    https://api.openweathermap.org/data/2.5/weather?lat=41.878929&lon=14.29332&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #406
    https://api.openweathermap.org/data/2.5/weather?lat=54.25&lon=-0.76667&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #407
    https://api.openweathermap.org/data/2.5/weather?lat=28.11644&lon=114.215683&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #408
    https://api.openweathermap.org/data/2.5/weather?lat=53.399712&lon=14.12533&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #409
    https://api.openweathermap.org/data/2.5/weather?lat=39.430119&lon=49.101662&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #410
    https://api.openweathermap.org/data/2.5/weather?lat=-6.21667&lon=149.550003&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #411
    https://api.openweathermap.org/data/2.5/weather?lat=42.571541&lon=-1.57465&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #412
    https://api.openweathermap.org/data/2.5/weather?lat=43.01667&lon=-2.51667&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #413
    https://api.openweathermap.org/data/2.5/weather?lat=29.558729&lon=112.009911&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #414
    https://api.openweathermap.org/data/2.5/weather?lat=46.333328&lon=25.01667&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #415
    https://api.openweathermap.org/data/2.5/weather?lat=-8.1848&lon=113.726997&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #416
    https://api.openweathermap.org/data/2.5/weather?lat=48.906658&lon=9.4685&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #417
    https://api.openweathermap.org/data/2.5/weather?lat=49.900002&lon=6.88333&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #418
    https://api.openweathermap.org/data/2.5/weather?lat=-37.916672&lon=145.350006&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #419
    https://api.openweathermap.org/data/2.5/weather?lat=-43.23333&lon=147.283325&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #420
    https://api.openweathermap.org/data/2.5/weather?lat=50.450001&lon=6.51667&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #421
    https://api.openweathermap.org/data/2.5/weather?lat=8.19556&lon=125.144722&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #422
    https://api.openweathermap.org/data/2.5/weather?lat=27.92141&lon=-82.817047&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #423
    https://api.openweathermap.org/data/2.5/weather?lat=47.583328&lon=5.83333&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #424
    https://api.openweathermap.org/data/2.5/weather?lat=38.478859&lon=-6.17984&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #425
    https://api.openweathermap.org/data/2.5/weather?lat=41.42992&lon=-3.54599&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #426
    https://api.openweathermap.org/data/2.5/weather?lat=37.136719&lon=-85.956917&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #427
    https://api.openweathermap.org/data/2.5/weather?lat=49.086418&lon=16.81443&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #428
    https://api.openweathermap.org/data/2.5/weather?lat=31.783331&lon=34.75&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #429
    https://api.openweathermap.org/data/2.5/weather?lat=-33.299999&lon=150.016663&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #430
    https://api.openweathermap.org/data/2.5/weather?lat=47.566669&lon=26.200001&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #431
    https://api.openweathermap.org/data/2.5/weather?lat=41.23032&lon=-89.645103&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #432
    https://api.openweathermap.org/data/2.5/weather?lat=30.375561&lon=32.291111&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #433
    https://api.openweathermap.org/data/2.5/weather?lat=-6.3731&lon=105.960701&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #434
    https://api.openweathermap.org/data/2.5/weather?lat=50.119148&lon=20.636101&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #435
    https://api.openweathermap.org/data/2.5/weather?lat=-22.219721&lon=-49.819439&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #436
    https://api.openweathermap.org/data/2.5/weather?lat=13.51824&lon=99.954689&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #437
    https://api.openweathermap.org/data/2.5/weather?lat=43.407848&lon=-73.259552&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #438
    https://api.openweathermap.org/data/2.5/weather?lat=47.3237&lon=10.15463&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #439
    https://api.openweathermap.org/data/2.5/weather?lat=59.48333&lon=24.9&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #440
    https://api.openweathermap.org/data/2.5/weather?lat=42.799999&lon=27.450001&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #441
    https://api.openweathermap.org/data/2.5/weather?lat=43.383331&lon=-0.01667&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #442
    https://api.openweathermap.org/data/2.5/weather?lat=42.816368&lon=-1.74786&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #443
    https://api.openweathermap.org/data/2.5/weather?lat=48.01667&lon=11.2&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #444
    https://api.openweathermap.org/data/2.5/weather?lat=-7.70611&lon=-38.154442&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #445
    https://api.openweathermap.org/data/2.5/weather?lat=30.67491&lon=-87.915268&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #446
    https://api.openweathermap.org/data/2.5/weather?lat=-6.9954&lon=112.400398&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #447
    https://api.openweathermap.org/data/2.5/weather?lat=37.349998&lon=139.316666&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #448
    https://api.openweathermap.org/data/2.5/weather?lat=51.54702&lon=-0.10944&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #449
    https://api.openweathermap.org/data/2.5/weather?lat=54.116669&lon=13.38333&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #450
    https://api.openweathermap.org/data/2.5/weather?lat=28.244181&lon=-82.719269&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #451
    https://api.openweathermap.org/data/2.5/weather?lat=6.64889&lon=124.873062&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #452
    https://api.openweathermap.org/data/2.5/weather?lat=44.01667&lon=24.450001&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #453
    https://api.openweathermap.org/data/2.5/weather?lat=-34.656219&lon=-58.75898&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #454
    https://api.openweathermap.org/data/2.5/weather?lat=5.13969&lon=-73.397392&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #455
    https://api.openweathermap.org/data/2.5/weather?lat=29.751841&lon=118.723778&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #456
    https://api.openweathermap.org/data/2.5/weather?lat=18.48333&lon=-88.316673&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #457
    https://api.openweathermap.org/data/2.5/weather?lat=33.98333&lon=134.366669&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #458
    https://api.openweathermap.org/data/2.5/weather?lat=46.033058&lon=13.47472&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #459
    https://api.openweathermap.org/data/2.5/weather?lat=50.068642&lon=34.758621&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #460
    https://api.openweathermap.org/data/2.5/weather?lat=47.834049&lon=13.41533&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #461
    https://api.openweathermap.org/data/2.5/weather?lat=-26.67313&lon=27.926149&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #462
    https://api.openweathermap.org/data/2.5/weather?lat=19.4&lon=-96.349998&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #463
    https://api.openweathermap.org/data/2.5/weather?lat=44.627949&lon=0.2902&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #464
    https://api.openweathermap.org/data/2.5/weather?lat=38.084831&lon=-0.94401&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #465
    https://api.openweathermap.org/data/2.5/weather?lat=50.716702&lon=7.98333&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #466
    https://api.openweathermap.org/data/2.5/weather?lat=43.193699&lon=-70.620888&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #467
    https://api.openweathermap.org/data/2.5/weather?lat=60.200001&lon=11.05&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #468
    https://api.openweathermap.org/data/2.5/weather?lat=64.133331&lon=25.366671&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #469
    https://api.openweathermap.org/data/2.5/weather?lat=48.784451&lon=1.81247&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #470
    https://api.openweathermap.org/data/2.5/weather?lat=45.3283&lon=5.55342&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #471
    https://api.openweathermap.org/data/2.5/weather?lat=36.686039&lon=25.6311&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #472
    https://api.openweathermap.org/data/2.5/weather?lat=41.218311&lon=13.56281&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #473
    https://api.openweathermap.org/data/2.5/weather?lat=49.866669&lon=12.2&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #474
    https://api.openweathermap.org/data/2.5/weather?lat=45.011871&lon=-113.965897&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #475
    https://api.openweathermap.org/data/2.5/weather?lat=43.599998&lon=-0.41667&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #476
    https://api.openweathermap.org/data/2.5/weather?lat=49.19949&lon=14.21717&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #477
    https://api.openweathermap.org/data/2.5/weather?lat=-26.16667&lon=152.883331&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #478
    https://api.openweathermap.org/data/2.5/weather?lat=45.133381&lon=-72.965843&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #479
    https://api.openweathermap.org/data/2.5/weather?lat=44.805408&lon=-76.626907&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #480
    https://api.openweathermap.org/data/2.5/weather?lat=35.146481&lon=-90.18454&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #481
    https://api.openweathermap.org/data/2.5/weather?lat=51.742779&lon=14.51573&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #482
    https://api.openweathermap.org/data/2.5/weather?lat=45.750839&lon=6.41611&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #483
    https://api.openweathermap.org/data/2.5/weather?lat=34.718819&lon=107.064453&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #484
    https://api.openweathermap.org/data/2.5/weather?lat=30.433531&lon=-89.966743&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #485
    https://api.openweathermap.org/data/2.5/weather?lat=43.165321&lon=-4.9136&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #486
    https://api.openweathermap.org/data/2.5/weather?lat=52.376099&lon=7.6059&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #487
    https://api.openweathermap.org/data/2.5/weather?lat=-3.38361&lon=-57.718609&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #488
    https://api.openweathermap.org/data/2.5/weather?lat=39.901562&lon=-8.83551&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #489
    https://api.openweathermap.org/data/2.5/weather?lat=45.000149&lon=-62.99865&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #490
    https://api.openweathermap.org/data/2.5/weather?lat=50.783329&lon=11.73333&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #491
    https://api.openweathermap.org/data/2.5/weather?lat=52.169998&lon=6.14167&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #492
    https://api.openweathermap.org/data/2.5/weather?lat=45.08469&lon=-93.009941&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #493
    https://api.openweathermap.org/data/2.5/weather?lat=44.079521&lon=-69.485046&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #494
    https://api.openweathermap.org/data/2.5/weather?lat=45.666672&lon=-0.91667&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #495
    https://api.openweathermap.org/data/2.5/weather?lat=71.984093&lon=-125.2463&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #496
    https://api.openweathermap.org/data/2.5/weather?lat=54.038761&lon=43.91386&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #497
    https://api.openweathermap.org/data/2.5/weather?lat=9.98333&lon=77.633331&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #498
    https://api.openweathermap.org/data/2.5/weather?lat=44.198971&lon=12.40064&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #499
    https://api.openweathermap.org/data/2.5/weather?lat=53.355122&lon=-6.24922&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    Processing Record #500
    https://api.openweathermap.org/data/2.5/weather?lat=26.49758&lon=-82.07843&appid=bcb6f6f1972448a5eadc67a4307786a6&units=Imperial
    


```python
random_cities.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>country</th>
      <th>id</th>
      <th>lat</th>
      <th>lng</th>
      <th>name</th>
      <th>Temperature</th>
      <th>Humidity</th>
      <th>Cloudiness</th>
      <th>Wind Speed</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>147097</th>
      <td>ES</td>
      <td>3125529</td>
      <td>40.437080</td>
      <td>-0.851500</td>
      <td>Cedrillas</td>
      <td>44.6</td>
      <td>75</td>
      <td>0</td>
      <td>5.82</td>
    </tr>
    <tr>
      <th>90834</th>
      <td>BR</td>
      <td>3471840</td>
      <td>-15.897500</td>
      <td>-52.250832</td>
      <td>Aragarcas</td>
      <td>74.21</td>
      <td>86</td>
      <td>68</td>
      <td>2.55</td>
    </tr>
    <tr>
      <th>40689</th>
      <td>AU</td>
      <td>2069358</td>
      <td>-35.500000</td>
      <td>138.449997</td>
      <td>Inman Valley</td>
      <td>86</td>
      <td>27</td>
      <td>0</td>
      <td>6.93</td>
    </tr>
    <tr>
      <th>200468</th>
      <td>US</td>
      <td>4710697</td>
      <td>30.917669</td>
      <td>-99.786461</td>
      <td>Menard</td>
      <td>50.97</td>
      <td>34</td>
      <td>90</td>
      <td>4.7</td>
    </tr>
    <tr>
      <th>104176</th>
      <td>SY</td>
      <td>166365</td>
      <td>32.890442</td>
      <td>36.039902</td>
      <td>Nawa</td>
      <td>50</td>
      <td>76</td>
      <td>0</td>
      <td>4.7</td>
    </tr>
  </tbody>
</table>
</div>




```python
random_cities.to_csv("random_cities.csv", index=False)
```

# Latitude vs Temperature Plot


```python
# Create scatter plot for Latitude vs Temperature
plt.scatter(random_cities["lat"],random_cities["Temperature"], alpha = 0.75)
plt.title("City Latitude vs. Max Temperature (03/07/18)")
plt.xlabel("Latitude")
plt.ylabel("Temperature (F)")
plt.grid(b=True, which='major', color='1', alpha =1, linestyle='-')
plt.style.use('ggplot')
plt.savefig("Latitude_Temperature_Plot.png")
plt.show()
```


![png](output_13_0.png)


# Latitude vs. Humidity Plot


```python
# Create scatter plot for Latitude vs Humidity
plt.scatter(random_cities["lat"],random_cities["Humidity"], alpha = 0.75)
plt.title("City Latitude vs. Humidity (03/07/18)")
plt.xlabel("Latitude")
plt.ylabel("Humidity")
plt.grid(b=True, which='major', color='1', alpha =1, linestyle='-')
plt.style.use('ggplot')
plt.savefig("Latitude_Humidity_Plot.png")
plt.show()
```


![png](output_15_0.png)


# Latitude vs. Cloudiness Plot


```python
# Create scatter plot for Latitude vs Cloudiness
plt.scatter(random_cities["lat"],random_cities["Cloudiness"], alpha = 0.75)
plt.title("City Latitude vs. Cloudiness (03/07/18)")
plt.xlabel("Latitude")
plt.ylabel("Cloudiness")
plt.grid(b=True, which='major', color='1', alpha =1, linestyle='-')
plt.style.use('ggplot')
plt.savefig("Latitude_Cloudiness_Plot.png")
plt.show()
```


![png](output_17_0.png)


# Latitude vs. Wind Speed Plot


```python
# Create scatter plot for Latitude vs Wind Speed
plt.scatter(random_cities["lat"],random_cities["Wind Speed"], alpha= 0.75)
plt.title("City Latitude vs Wind Speed (03/07/18)")
plt.xlabel("Latitude")
plt.ylabel("Wind Speed")
plt.grid(b=True, which='major', color='1', alpha =1, linestyle='-')
plt.style.use('ggplot')
plt.savefig("Latitude_WindSpeed_Plot.png")
plt.show()
```


![png](output_19_0.png)

