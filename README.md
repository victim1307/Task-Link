# Task-Link
Amazon Web scrapper
```python
from amazoncaptcha import AmazonCaptcha
import time
import pandas
import pyuser_agent
from bs4 import BeautifulSoup
import requests
import json


def solve_captcha():
    # this helps in solving captacha of the amazon site, we can send a post request of solution of the captcha to the link and solve it.
    url = "https://images-na.ssl-images-amazon.com/captcha/lqbiackd/Captcha_iykgxrlkyh.jpg"
    captcha = AmazonCaptcha.fromlink(url)
    solution = captcha.solve()
    print(solution)


def get_data_from_google():
    count = 0
    # I have used pandas to read the data from the google sheet, so that i dont have to download the fiel everytime local
    df = pandas.read_csv(f"https://docs.google.com/spreadsheets/d/1BZSPhk1LDrx8ytywMHWVpCqbm8URTxTJrIRkD7PnGTM/export?format=csv")
    # Convert the dataframe to a list
    lst = (df.values.tolist())
    # Used start_time to get the time when the program started
    start = time.time()
    # Loop through the list of valuse extracted from the google sheet
    for val in lst:
        #if loop is used to print after every 100 links and check the progess
        if(count%100 == 0 and count != 0):
            end = time.time()
            print("\n 100 URLs finished and the time take is: "+str(end-start)+" seconds\n")
            start = time.time()
            #count = 1
        country_code = val[3]
        asin = val[2]
        # URL of the product page
        url = 'https://www.amazon.{coutry_code}/dp/{asin}'.format(coutry_code=country_code, asin=asin)
        count += 1
        # Call the function get_data to get the data from the product page
        get_data(url)

def get_data(url):
    # This function is used to get the data from the product page
    # Used pyuser_agent to get the user agent
    # since the user agent is getting blocked frequently, we can use the pyuser_agent to get the user agent in random manner
    ua = pyuser_agent.UA()
    headers = {
          "User-Agent" : ua.random
    }
    # Send a get request to the url and get the response
    r = requests.get(url,headers=headers)
    if(r.status_code == 200):
        # If the response is 200, then we can get the data from the page
        try:
            # Get the data from the page
            print(r.status_code)
            print(url)
            content = r.content
            # Create a soup object to parse the html content
            soup = BeautifulSoup(content, 'html.parser')
            # Get the title of the product
            pro_name = soup.find('span', id='productTitle').text
            # Get the product image
            poduct_img = soup.find('img', id='imgBlkFront')
            if poduct_img is None:
                poduct_img = soup.find('img', id='landingImage')
            pro_det = ""
            # Get the product description
            for i in soup.find_all('div', attrs={'class':'a-section feature detail-bullets-wrapper bucket'}):
                for j in i.find_all('ul', attrs={'class':'a-unordered-list a-nostyle a-vertical a-spacing-none detail-bullet-list'}):
                    for k in j.find_all('span',attrs={'class':'a-list-item'}):
                        kk = " ".join(k.text.split())
                 ```       pro_det += kk+"\n"
            # Get the product price i have got too many non printable characters in the price, so i have used the encode and decode function to remove the non printable characters
            pro_det = pro_det.encode('ascii', 'ignore').decode()
            pro_price = ""
            # Get the product price
            for i in soup.find_all('span', attrs={'class':'a-button a-button-selected a-spacing-mini a-button-toggle format'}):
                pro_price = ' '.join(i.text.split())
            pro_price = pro_price.encode('ascii', 'ignore').decode()
            # Converting the extracted data to a JSON object
            json_data = {
                "Product Name" : pro_name,
                "Product Image Url":poduct_img['src'],
                "Product Description":pro_det,
                "Product Price":pro_price
                }
            # Open the file in append mode and wrote the Json Objects to the file    
            with open("data1.json", "a") as f:
                json.dump(json_data, f)
                f.write(",\n")
            print(y)
        except:
            # If the response is not 200, then we can not get the data from the page
            print("Error in getting data please run the code again after some")
            pass
    else:
        print("Error "+str(r.status_code)+" "+url+" not available")

get_data_from_google()
```
