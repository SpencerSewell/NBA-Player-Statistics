# NBA-Player-Statistics
``` python
import time
from string import ascii_lowercase
import pandas as pd
import numpy as np
import requests
from bs4 import BeautifulSoup
import lxml

# Variables and Lists setup
base_url = "https://www.basketball-reference.com"
master_list = []
master_data = []
progress = 0

def scrape_letter(letter):
    url = f"{base_url}/players/{letter}/"
    response = requests.get(url)
    player_urls = []  # List to store player URLs

    if response.status_code == 200:
        soup = BeautifulSoup(response.content, 'html.parser')
        for th in soup.find_all('th', {'scope': 'row', 'class': 'left'}):
            link = th.find('a')
            if link:
                player_url = base_url + link.get('href')
                player_urls.append(player_url)

    else:
        print("Failed to retrieve the webpage.")
        exit()

    return player_urls

for letter in ascii_lowercase:
    player_urls = scrape_letter(letter)
    print(f"Gathered links for players who's name start with {letter}")
    master_list.append(player_urls)  # Append the list of URLs to the master_list
    time.sleep(60 / 20)  # Rate Limit

total_length = sum(len(urls) for urls in master_list)

#Test Case
#testurl = master_list[0][0]
#response = requests.get(testurl)
#soup = BeautifulSoup(response.content, 'html.parser')
#name_tag = soup.find('h1')
#player_name = name_tag.get_text(strip=True) if name_tag else "Unknown Player"
#dfs = pd.read_html(testurl)

#Gathering Data
for i in range(0, len(master_list)):
  for j in range(0, len(master_list[i])):
    print(f'Total Progress: {progress}/{total_length}')
    url2 = master_list[i][j]
    response2 = requests.get(url2)
    soup = BeautifulSoup(response2.content, 'html.parser')
    name_tag = soup.find('h1')
    player_name = name_tag.get_text(strip=True) if name_tag else "Unknown Player"
    dfs = pd.read_html(url2)
    if dfs:
      player_df = dfs[0]
      player_df['Player'] = player_name
      master_data.append(player_df)
    progress += 1
    time.sleep(60 / 20)

final_dataframe = pd.concat(master_data, ignore_index=True)
final_dataframe.to_csv('basketball_data.csv', index=False)
```
