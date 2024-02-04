# NBA-Player-Statistics
This project is in collaboration with Drew Wolin for his "Name that Player" game on his website https://drewwolin.com/games/ntp/index.html. I have been tasked with writting a webscraper that gathers data from basketball-reference.com on all of the players career stats. This will be brought into a MySQL database and referenced by his website for a daily game his users can play.

``` python
import time
from string import ascii_lowercase
import pandas as pd
import numpy as np
import requests
from bs4 import BeautifulSoup
import lxml
from io import StringIO

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

#Gathering Data
for i in range(0, len(master_list)):
  for j in range(0, len(master_list[i])):
      print(f'Total Progress: {progress}/{total_length}')
      url2 = master_list[i][j]
      response2 = requests.get(url2)
      soup = BeautifulSoup(response2.content, 'html.parser')
      name_tag = soup.find('h1')
      player_name = name_tag.get_text(strip=True) if name_tag else "Unknown Player"

      # Attempt to find the specific table's div by its ID
      table_wrapper = soup.find('div', id='all_per_game-playoffs_per_game')

      # Proceed only if the table wrapper is found
      if table_wrapper:
          table_html = str(table_wrapper)  # Convert the table wrapper (and its contents) to string
          table_html_io = StringIO(table_html)  # Wrap the HTML content in a StringIO object
          dfs = pd.read_html(table_html_io)  # Read the HTML table into a DataFrame

          if dfs:
              player_df = dfs[0]  # Assuming the table you need is the first one found
              player_df['Player'] = player_name
              master_data.append(player_df)
      else:
          print(f"No playoff per game stats table found for {player_name}")

      progress += 1
      time.sleep(60 / 20)

final_dataframe = pd.concat(master_data, ignore_index=True)
final_dataframe.to_csv('basketball_data.csv', index=False)
```
