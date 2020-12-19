# Case4
Google Play Comment Crawler

1.  calling package and objects
```
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
import time
import json
from bs4 import BeautifulSoup
import pandas as pd

 
# 避免彈出視窗
options = Options()
options.add_argument("--disable-notifications")

# 創立物件chrome
chrome_options = webdriver.ChromeOptions()
chrome_options.add_argument('--headless')
chrome_options.add_argument('--no-sandbox')
chrome_options.add_argument('--disable-dev-shm-usage')
chrome = webdriver.Chrome('chromedriver',chrome_options=chrome_options)
chrome.get("https://play.google.com/store/apps/details?id=com.PChome.Shopping&showAllReviews=true")
time.sleep(1)
chrome.find_element_by_xpath('//div[@class="MocG8c UFSXYb LMgvRb KKjvXb"]').click()
time.sleep(1)
try:
  chrome.find_element_by_xpath('//div[@jsname="V68bde"]//span[@class="vRMGwf oJeWuf"]').click()
except:
  pass
```

2. acquire html codes
```
for i in range(100): # 動作總共執行次數
  # 每往下滑動五次須點擊一次按鈕
  for i in range(5):
    time.sleep(1)
    # 往下滑動網頁
    chrome.execute_script("window.scrollTo(0, document.body.scrollHeight);")
  try:
    # 點擊「載入更多」按鈕
    drive.find_element_by_xpath('//div[@class="U26fgb O0WRkf oG5sRB c0oVfc n9lfJ"]').click()
  except:
    pass
```

3. transfer to structured data
```
# 轉換為結構式資料
soup = BeautifulSoup(chrome.page_source)
chrome.close()

list_of_comments = []
for i,element in enumerate(soup.find_all('div',{'jscontroller':'H6eOGe'})):
  if i % 10 == 0:
    print(i)
  try:
    likecounts = element.find('div', {'class':'jUL89d y92BAb'}).text
  except:
    likecounts = str(0)

  Content = element.find('span', {'jsname':'bN97Pc'}).text

  # 若有「完整評論」則抓取裡面內容
  if len(element.find('span', {'jsname':'fbQN7e'}).text) >= 20:
    Content = element.find('span', {'jsname':'fbQN7e'}).text
  
  # 建立dataframe
  df = pd.DataFrame([{'Name':element.find('span', {'class':'X43Kjb'}).text,
                        'Sentimen':element.find('div', {'class':'pf5lIe'}).find('div')['aria-label'],
                        'likecounts':likecounts,
                        'Timestamp':element.find('span',{'class':'p2TkOb'}).text,
                        'Content':Content}],
                      columns = ['Name', 'Timestamp', 'Content', 'Sentimen', 'likecounts'])
  list_of_comments.append(df)
comments = pd.concat(list_of_comments, ignore_index=True) # dataframe疊合
```
