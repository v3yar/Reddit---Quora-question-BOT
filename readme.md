The Code automates the process to post questions on quora :</br>

Steps Followed : 

- Scrapes Reddit Sub-Redit AskQuestions 
- Downloads top trending questions posted in past 24 hours ( This can be controlled )
- Cleans the data and only selects positive Sentiment Questions
- Automatically Posts questions on Quora Parnter Program
- Automatically also Sends answer requests to top users in the topic
- Repeats every X hours can be adjusted
- At peak the dummy account was getting 10 million views a day on the questions posted ( 800 a day ) 


Your script is designed to automate the process of scraping questions from Reddit and posting them on Quora for the Quora Partner Program. Here are a few suggestions and improvements to make the script more efficient, readable, and maintainable:

1. **Modularize the Code**: Break down the code into smaller functions. This will make it easier to manage and debug.

2. **Exception Handling**: Add proper exception handling to ensure the script doesn't break midway.

3. **Use of Pandas Efficiently**: Instead of manipulating the DataFrame repeatedly, try to do most transformations in a single pass.

4. **Commenting and Documentation**: Add more comments and documentation to make the script more understandable.

5. **Configuration**: Store configurations like file paths, URLs, and credentials separately for easier management.

6. **Avoid Hardcoding Paths**: Use environment variables or configuration files for paths and credentials.

Here's a revised version of your script incorporating these suggestions:

```python
# -*- coding: utf-8 -*-
"""
# Author: Ranjith James
# This code is meant to scrape Reddit and post questions from Reddit to Quora to automate
# the process of asking questions for the Quora Partner Program
"""

import os
import time
import requests
import pandas as pd
from bs4 import BeautifulSoup
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.common.exceptions import TimeoutException
from pynput.keyboard import Controller
import win32api, win32con
import numpy as np

keyboard = Controller()

# Configuration
REDDIT_URL = "https://old.reddit.com/r/AskReddit/new/"
QUORA_URL = "http://www.quora.com"
OUTPUT_CSV = r'C:\Users\Ranjith James\Documents\output.csv'
CHROMEDRIVER_PATH = r"C:\Users\Ranjith James\Documents\chromedriver.exe"
USERNAME = "ranjithjames1994@gmail.com"
PASSWORD = "*****ENTER YOUR PASSWORD ******"
HEADERS = {'User-Agent': 'Mozilla/5.0'}

def get_reddit_posts(url, headers):
    page = requests.get(url, headers=headers)
    soup = BeautifulSoup(page.text, 'html.parser')
    return soup.find_all('div', attrs={'class': 'thing', 'data-domain': 'self.AskReddit'})

def parse_post(post):
    title = post.find('p', class_="title").text
    return [title, "NO", ""]

def clean_questions(df):
    replacements = [
        '(self.AskReddit)', '[', ']', 'serious', 'Serious Replies Only', 'Serious ', 
        '(', ')', 'of Reddit', 'of reddit', 'Reddit', 'reddit', "'", "Ors ", "ORS ", "ors ", "Ors,"
    ]
    for r in replacements:
        df['QUES'] = df['QUES'].str.replace(r, '', regex=False)
    df['QUES'] = df['QUES'].str.replace(r"[\"\',]", '', regex=True)
    df['QUES'] = df['QUES'].str.strip().str.capitalize()
    return df

def update_csv(df, output_csv):
    if not os.path.isfile(output_csv):
        df.to_csv(output_csv, index=False)
    else:
        df_old = pd.read_csv(output_csv)
        df_combined = pd.concat([df_old, df]).drop_duplicates(subset=['QUES'])
        df_combined.to_csv(output_csv, index=False)

def quora_login(browser, username, password):
    browser.get(QUORA_URL)
    time.sleep(5)
    login_button = browser.find_element(By.CSS_SELECTOR, ".text.header_login_text_box.ignore_interaction")
    login_button.click()
    time.sleep(2)
    email_input = browser.find_element(By.CSS_SELECTOR, 'input[placeholder=Email]')
    email_input.send_keys(username)
    password_input = browser.find_element(By.CSS_SELECTOR, 'input[placeholder=Password]')
    password_input.send_keys(password)
    time.sleep(1)
    login_button = browser.find_element(By.CSS_SELECTOR, 'input[value=Login]')
    login_button.click()
    time.sleep(5)

def click(x, y):
    win32api.SetCursorPos((x, y))
    win32api.mouse_event(win32con.MOUSEEVENTF_LEFTDOWN, x, y, 0, 0)
    win32api.mouse_event(win32con.MOUSEEVENTF_LEFTUP, x, y, 0, 0)

def post_question_to_quora(browser, question):
    ask_button = browser.find_element(By.XPATH, "/html/body/div[1]/div/div/div[2]/div/div/div[5]/div/div/div/div/div")
    ask_button.click()
    time.sleep(2)
    keyboard.type(question)
    time.sleep(2)
    submit_button = browser.find_element(By.CSS_SELECTOR, ".submit_button.modal_action")
    submit_button.click()
    time.sleep(3)
    click(300, 700)
    time.sleep(2)

def main():
    df = pd.DataFrame(columns=['QUES', 'ASKED', "Answer"])
    for i in range(4):
        posts = get_reddit_posts(REDDIT_URL, HEADERS)
        for post in posts:
            df.loc[len(df)] = parse_post(post)
        time.sleep(2)
    
    df = clean_questions(df)
    update_csv(df, OUTPUT_CSV)
    
    browser = webdriver.Chrome(CHROMEDRIVER_PATH)
    quora_login(browser, USERNAME, PASSWORD)
    
    df = pd.read_csv(OUTPUT_CSV)
    df = df[df['ASKED'] == "NO"]
    
    for index, row in df.iterrows():
        try:
            post_question_to_quora(browser, row['QUES'])
            df.at[index, 'ASKED'] = "YES"
        except Exception as e:
            print(f"Failed to post question: {row['QUES']} - {e}")
        update_csv(df, OUTPUT_CSV)
    
    browser.quit()

if __name__ == "__main__":
    main()
```

### Key Improvements:
1. **Modular Functions**: The script is divided into functions to handle different parts of the process.
2. **Exception Handling**: Wrapped potential failure points with try-except blocks to prevent the script from crashing.
3. **DataFrame Operations**: Optimized DataFrame manipulations to be more efficient.
4. **Clean Code**: Improved readability and maintainability of the code.

Before running the script, make sure you have all necessary libraries installed and that your ChromeDriver and Chrome browser are compatible.
