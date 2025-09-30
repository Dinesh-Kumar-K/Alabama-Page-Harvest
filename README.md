```markdown
# Alabama Page Harvest

This notebook demonstrates a web scraping process to harvest data from the Alabama Secretary of State's corporate detail pages. It utilizes Python libraries like `pandas`, `requests`, `beautifulsoup4`, and `asyncio` for efficient data extraction and processing.

## Setup

First, we install the necessary library for Excel file handling.

```python
pip install openpyxl
```

## Import Modules

We import all the essential libraries for our web scraping task.

```python
# Importing Important libraries
import pandas as pd
from tqdm import tqdm
import re
import time
import urllib
import warnings
from tqdm import trange
import sys
import requests
from bs4 import BeautifulSoup
from concurrent.futures import ThreadPoolExecutor
```

## Initial Testing

Let's start by testing the requests and BeautifulSoup parsing with a sample URL.

```python
url="https://opencorporates.com/companies/us_ak?action=search_companies&commit=Go&controller=searches&order=&q=google&type=companies&utf8=%E2%9C%93"
```

We define a user-agent header to mimic a browser.

```python
headers={'user-agent':'Mozilla/5.0 (Windows NT 10.0; apple64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/116.0.0.0 Safari/537.36'}
```

Making the request to the URL.

```python
res=requests.get(url,headers=headers)
```

Printing the response status.

```python
res
```

Parsing the HTML content with BeautifulSoup.

```python
sou1=BeautifulSoup(res.content,'lxml')
```

Displaying the parsed soup object.

```python
sou1
```

Printing the text content of the parsed soup.

```python
sou.text.strip()
```

Extracting a company name as a test.

```python
cmp_name=sou.find('thead').find('td',class_="aiSosDetailHead").text.strip()
```

Displaying the extracted company name.

```python
cmp_name
```

## Data Harvesting Function

This section defines a function `harvest_page` to scrape data from a single Alabama corporate detail page.

```python
def harvest_page(num):
  loo_df=pd.DataFrame()
  url=f'https://arc-sos.state.al.us/cgi/corpdetail.mbr/detail?corp={num}'

  headers={ "Accept":"text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7",
            "Accept-Encoding":"gzip,deflate,br",
            "Accept-Language":"en-US,en;q=0.9,ta;q=0.8",
            "Connection":"keep-alive",
            "Cookie":"_ga=GA1.1.2085200574.1702026901;_ga_HEFYBZ2QN2=GS1.1.1702285595.5.1.1702285603.0.0.0",
            "Host":"arc-sos.state.al.us",
            "Sec-Fetch-Dest":"document",
            "Sec-Fetch-Mode":"navigate",
            "Sec-Fetch-Site":"none",
            "Sec-Fetch-User":"?1",
            "Upgrade-Insecure-Requests":"1",
            "User-Agent":"Mozilla/5.0(WindowsNT10.0;Win64;x64)AppleWebKit/537.36(KHTML,likeGecko)Chrome/120.0.0.0Safari/537.36",
            "sec-ch-ua":"'Not_ABrand';v='8','Chromium';v='120','GoogleChrome';v='120'",
            "sec-ch-ua-mobile":"?0",
            "sec-ch-ua-platform":"Windows"
          }

  try:
    res=requests.get(url,headers=headers,timeout=40)
    sou=BeautifulSoup(res.content,'lxml')
    data_find=sou.text.strip()
    try:
        cmp_name=sou.find('thead').find('td',class_="aiSosDetailHead").text.strip()
        stus_code=res.status_code
        datas0=[]
        datas1=[]

        for i in sou.find('div',attrs={'align':'center'}).find_all('tr'):
          try:
            attri_name=i.find('td',class_="aiSosDetailDesc").text.strip()
            detail_value=i.find('td',class_="aiSosDetailValue").text.strip()

            datas1.append(detail_value)
            datas0.append(attri_name)

          except:
            pass

        for i in range(len(datas0)):
          loo_df[datas0[i]]=[str(datas1[i])]

        loo_df['company']=[cmp_name]
        loo_df['Url_number']=[num]
        loo_df['Status']=[stus_code]

        loo_df['Test']=["Allow"]
    except:
       if data_find in "You have exceeded the number accesses allowed.":
        return harvest_page(num)
       else:
           loo_df['Test']=["Not Have"]

    return loo_df
  except:
    return loo_df
```

Running the `harvest_page` function with a sample number.

```python
#find=harvest_page('000000007')
#harvest_page('000000002')
```

Displaying the result of the test run.

```python
#find
```

Saving the collected data to a CSV file.

```python
# dtas_oyut.to_csv('ayeoirtqie.csv')
```

### Generating a Range of Numbers

We create a list of numerical strings to be used as corporate numbers.

```python
str_n="000000000"
range_v=[]
for i in range(1,1000):
  st_v=f"{len(str(i))}"
  num_f=str_n[int(st_v):]+str(i)
  range_v.append(num_f)
```

Checking the length of the generated range.

```python
len(range_v)
```

## Thread Compile

This section utilizes `ThreadPoolExecutor` to concurrently fetch data from multiple pages.

```python
collect_frame=[]
def compile_run():
    with tqdm(total=len(range_v),desc="harvest") as progress:
        with ThreadPoolExecutor(max_workers=5) as executor:
            for i in executor.map(harvest_page,range_v):
               collect_frame.append(i)
               progress.update()
               time.sleep(.1)
               if len(collect_frame)%5==0:
                time.sleep(.3)
compile_run()
```

Concatenating the results from all threads into a single DataFrame.

```python
final_df=pd.DataFrame()
for i in collect_frame:
    final_df=pd.concat([final_df,i],axis=0,ignore_index=True)
```

Printing the number of columns for each collected frame.

```python
for ciu,i in enumerate(collect_frame):
    print(f'Length {len(i.columns)} --- {ciu}')
```

Saving the final DataFrame to a CSV file.

```python
#final_df.to_csv("arc-sos.state Sample 1k out.csv")
```

Displaying the final DataFrame.

```python
final_df
```

Creating a sample DataFrame for demonstration.

```python
datas=pd.DataFrame()
```

```python
datas['work']=[10,10,10,10,10]
datas['Do']=[4,5,6,2,8]
```

```python
datas
```

## Proxies and User Agents

This section loads proxy and user-agent lists from CSV files.

```python
procxi_df=pd.read_csv('/content/Free_Proxy_List.csv')
user_agent_df=pd.read_csv("/content/100 User Agent Pass Frame.csv")
```

Extracting lists of proxies and user agents.

```python
procxi_list=procxi_df['ip'].to_list()
user_agent_list=user_agent_df['User_Agents'].to_list()
```

This is an empty code cell.

```python

```

## Async Function

This section introduces asynchronous programming for more efficient web scraping.

```python
### Async Code

import aiohttp
import asyncio
import nest_asyncio
```

Importing the `random` module.

```python
import random
```

Selecting a random user agent.

```python
user_agent_list_len=range((len(user_agent_list)))
user_agent_random_num=random.choice(user_agent_list_len)
print(user_agent_list[user_agent_random_num])
```

Defining the asynchronous harvesting function.

```python
async def harvest_page_asy(session,num):
        loo_df=pd.DataFrame()
        try:
          headers={ "Accept":"text/html,application/xhtml+xml,application/xml;q=0.9,application/signed-exchange;v=b3;q=0.7",
                        "Accept-Encoding":"gzip,deflate,br",
                        "Accept-Language":"en-US,en;q=0.9,ta;q=0.8",
                        "Connection":"keep-alive",
                        "Cookie":"_ga=GA1.1.2085200574.1702026901;_ga_HEFYBZ2QN2=GS1.1.1702285595.5.1.1702285603.0.0.0",
                        "Host":"arc-sos.state.al.us",
                        "Sec-Fetch-Dest":"document",
                        "Sec-Fetch-Mode":"navigate",
                        "Sec-Fetch-Site":"none",
                        "Sec-Fetch-User":"?1",
                        "Upgrade-Insecure-Requests":"1",
                        "User-Agent":"Mozilla/5.0(WindowsNT10.0;Win64;x64)AppleWebKit/537.36(KHTML,likeGecko)Chrome/120.0.0.0Safari/537.36",
                        "sec-ch-ua":"'Not_ABrand';v='8','Chromium';v='120','GoogleChrome';v='120'",
                        "sec-ch-ua-mobile":"?0",
                        "sec-ch-ua-platform":"Windows"
                      }
          url=f'https://arc-sos.state.al.us/cgi/corpdetail.mbr/detail?corp={num}'
          async with session.get(url, headers=headers,timeout=10) as response:
              status_code = response.status
              page_content = await response.text()
              #res=requests.get(url,headers=headers,timeout=40)
              sou=BeautifulSoup(page_content,'lxml')
              data_find=sou.text.strip()
              try:
                  cmp_name=sou.find('thead').find('td',class_="aiSosDetailHead").text.strip()
                  stus_code=res.status_code
                  datas0=[]
                  datas1=[]

                  for i in sou.find('div',attrs={'align':'center'}).find_all('tr'):
                    try:
                      attri_name=i.find('td',class_="aiSosDetailDesc").text.strip()
                      detail_value=i.find('td',class_="aiSosDetailValue").text.strip()

                      datas1.append(detail_value)
                      datas0.append(attri_name)

                    except:
                      pass

                  for i in range(len(datas0)):
                    loo_df[datas0[i]]=[str(datas1[i])]

                  loo_df['company']=[cmp_name]
                  loo_df['Url_number']=[num]
                  loo_df['Status']=[stus_code]

                  loo_df['Test']=["Allow"]
              except:
                if data_find in "You have exceeded the number accesses allowed.":
                  time.sleep(1.5)
                  return harvest_page(num)
                else:
                  loo_df['Url_number']=[num]
                  loo_df['Test']=["Not Have"]

              return loo_df
        except:
          loo_df['Url_number']=[num]
          loo_df['Test']=["Error"]
          time.sleep(.3)
          return loo_df
```

#### Async Compile

This section orchestrates the asynchronous harvesting process.

```python
collect_frame=[]
async def main(list_num):
    tasks = []
    async with aiohttp.ClientSession() as session:
        for perma in list_num:
            tasks.append(harvest_page(session, perma))
        results = []
        with tqdm(total=len(list_num),desc="harvest") as progress:
          for task in asyncio.as_completed(tasks):
              data=await task
              collect_frame.append(data)
              time.sleep(.2)
              progress.update(1)
              if len(collect_frame)%6==0:
                time.sleep(1)


if __name__ == "__main__":
    nest_asyncio.apply()
    asyncio.run(main(range_v[:1000]))
```

Consolidating the asynchronous results.

```python
final_df=pd.DataFrame()
for i in collect_frame:
    final_df=pd.concat([final_df,i],axis=0,ignore_index=True)
```

Displaying the final DataFrame from asynchronous operations.

```python
final_df
```

Counting occurrences by the 'Test' status.

```python
final_df.groupby('Test').count()
```

Displaying the final DataFrame again.

```python
final_df
```

Sorting the DataFrame by 'Url_number'.

```python
final_df.sort_values(['Url_number'], ascending=[True])
```

Getting a list of 'Url_number' values that resulted in an 'Error'.

```python
final_df[final_df['Test']=="Error"]['Url_number'].to_list()
```

### Loop Complies

This section implements a loop to re-process any entries that resulted in an 'Error' during the previous runs.

```python
tests_range=range_v[:10]
```

Initializing a DataFrame to store the final collected data.

```python
find_final_data=pd.DataFrame()
```

The `while` loop continues until all entries are successfully processed.

```python
while True:
    collect_frame=[]
    async def main(list_num):
        tasks = []
        async with aiohttp.ClientSession() as session:
            for perma in list_num:
                tasks.append(harvest_page_asy(session, perma))
            results = []
            with tqdm(total=len(list_num),desc="harvest") as progress:
                for task in asyncio.as_completed(tasks):
                    data=await task
                    time.sleep(.3)
                    collect_frame.append(data)
                    progress.update(1)
                    if len(collect_frame)%10==0:
                        time.sleep(2)


    if __name__ == "__main__":
        nest_asyncio.apply()
        asyncio.run(main(tests_range))

    final_df=pd.DataFrame()
    for i in collect_frame:
        final_df=pd.concat([final_df,i],axis=0,ignore_index=True)

    find_df=final_df[final_df['Test']!="Error"]
    tests_range=final_df[final_df['Test']=="Error"]['Url_number'].to_list()

    find_final_data=pd.concat([find_df,find_final_data],axis=0)

    print(f"Total Count : {len(collect_frame)}")
    print(f"Find Count : {len(find_df)}")
    print(f"Not Fount : {len(tests_range)}")

    if len(tests_range)==0:
        print('Break')
        break

```

Extracting 'Url_number' and 'Test' columns from the final data.

```python
tests=find_final_data[['Url_number','Test']]
```

Displaying the `find_final_data` DataFrame.

```python
find_final_data
```

This is an empty code cell.

```python

```
```

---

Here's the Jupyter notebook content converted into a GitHub-ready Markdown file.

```md
# Alabama Corporate Records Scraper

This notebook demonstrates how to scrape corporate information from the Alabama Secretary of State website. It utilizes libraries like `pandas`, `requests`, `BeautifulSoup`, and `tqdm` for data manipulation, web scraping, and progress tracking. Both synchronous and asynchronous approaches are explored.

## Setup and Imports

First, we import all the necessary libraries.

```python
# Importing Important libraries
import pandas as pd
from tqdm import tqdm
import re
import time
import urllib
import warnings
from tqdm import trange
import sys
import requests
from bs4 import BeautifulSoup
from concurrent.futures import ThreadPoolExecutor
```

We also disable a common warning that might appear during SSL certificate verification when using `requests`.

```python
# Disable warnings related to SSL certificate verification
warnings.filterwarnings('ignore')
```

## Initial Data Fetching and Inspection

Let's start by fetching data for a single corporate record and inspecting the content.

```python
# Example URL for a corporate detail page
url="https://arc-sos.state.al.us/cgi/corpdetail.mbr/detail?corp=000000008"
```

We define a user-agent header to mimic a web browser.

```python
# Define user-agent for requests
headers={'user-agent':'Mozilla/5.0 (Windows NT 10.0; apple64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/116.0.0.0 Safari/537.36'}
```

Make the HTTP GET request to the URL. `verify=False` is used here for simplicity but in production, it's better to handle SSL verification properly.

```python
# Make the request to the URL
res=requests.get(url,headers=headers,verify=False)
```

Parse the HTML content using BeautifulSoup with the `lxml` parser.

```python
# Parse the HTML content
sou=BeautifulSoup(res.content,'lxml')
```

Display the stripped text content of the page.

```python
# Get and display the stripped text content
sou.text.strip()
```

Extract the company name from the page. It's located in a `thead` tag within a `td` with the class `aiSosDetailHead`.

```python
# Extract the company name
cmp_name=sou.find('thead').find('td',class_="aiSosDetailHead").text.strip()
```

Print the extracted company name.

```python
# Display the company name
cmp_name
```

## Function to Harvest Page Data (Synchronous)

This function is designed to fetch and parse data for a single corporate record given its number. It handles potential errors and rate limiting.

```python
def harvest_page(num):
  """
  Fetches and parses corporate detail information for a given corporate number.

  Args:
    num (str): The corporate number to fetch data for.

  Returns:
    pandas.DataFrame: A DataFrame containing the parsed company details,
                      or an empty DataFrame if an error occurs or data is not found.
  """
  loo_df=pd.DataFrame()
  url=f'https://arc-sos.state.al.us/cgi/corpdetail.mbr/detail?corp={num}'
  headers={ "Accept":"text/html,application/xhtml+xml,application/xml;q=0.9,application/signed-exchange;v=b3;q=0.7",
            "Accept-Encoding":"gzip,deflate,br",
            "Accept-Language":"en-US,en;q=0.9,ta;q=0.8",
            "Connection":"keep-alive",
            "Cookie":"_ga=GA1.1.2085200574.1702026901;_ga_HEFYBZ2QN2=GS1.1.1702285595.5.1.1702285603.0.0.0",
            "Host":"arc-sos.state.al.us",
            "Sec-Fetch-Dest":"document",
            "Sec-Fetch-Mode":"navigate",
            "Sec-Fetch-Site":"none",
            "Sec-Fetch-User":"?1",
            "Upgrade-Insecure-Requests":"1",
            "User-Agent":"Mozilla/5.0(WindowsNT10.0;Win64;x64)AppleWebKit/537.36(KHTML,likeGecko)Chrome/120.0.0.0Safari/537.36",
            "sec-ch-ua":"'Not_ABrand';v='8','Chromium';v='120','GoogleChrome';v='120'",
            "sec-ch-ua-mobile":"?0",
            "sec-ch-ua-platform":"Windows"
          }
  try:
    # Make the request with a timeout
    res=requests.get(url,headers=headers,timeout=40)
    sou=BeautifulSoup(res.content,'lxml')
    data_find=sou.text.strip()

    try:
        # Extract company name
        cmp_name=sou.find('thead').find('td',class_="aiSosDetailHead").text.strip()
        stus_code=res.status_code
        datas0=[]
        datas1=[]

        # Iterate through table rows to extract attribute names and values
        for i in sou.find('div',attrs={'align':'center'}).find_all('tr'):
          try:
            attri_name=i.find('td',class_="aiSosDetailDesc").text.strip()
            detail_value=i.find('td',class_="aiSosDetailValue").text.strip()

            datas1.append(detail_value)
            datas0.append(attri_name)

          except:
            # Skip if attribute or value not found in a row
            pass

        # Populate the DataFrame with extracted data
        for i in range(len(datas0)):
          loo_df[datas0[i]]=[str(datas1[i])]

        loo_df['company']=[cmp_name]
        loo_df['Url_number']=[num]
        loo_df['Status']=[stus_code]
        loo_df['Test']=["Allow"] # Mark as successfully parsed

    except Exception as e:
       # Handle specific rate limiting or other parsing errors
       if "You have exceeded the number accesses allowed." in data_find:
        print(f"Rate limited for {num}. Retrying after a short delay.")
        time.sleep(1)
        return harvest_page(num) # Retry the request
       else:
        print(f"Error parsing data for {num}: {e}")
        loo_df['Url_number']=[num]
        loo_df['Test']=["Not Have"] # Mark as not found or unparseable

    return loo_df
  except requests.exceptions.RequestException as e:
    print(f"Request failed for {num}: {e}")
    return pd.DataFrame({'Url_number': [num], 'Test': ["Error"]}) # Mark as request error

```

### Testing the `harvest_page` Function

Let's test the function with a specific corporate number.

```python
# Test harvest_page with a specific number
# dtas_oyut=harvest_page('000000002') # This line is commented out
harvest_page('000000007')
```

The following line, if uncommented, would save the output of a single harvest to a CSV file.

```python
# dtas_oyut.to_csv('ayeoirtqie.csv') # This line is commented out
```

## Generating a Range of Corporate Numbers

We need to create a list of corporate numbers to scrape. The website uses a fixed-width format for these numbers (e.g., `000000000`).

```python
# Define the base string for corporate numbers
str_n="000000000"
range_v=[]
# Generate numbers from 1 to 999, formatted correctly
for i in range(1,1000):
  st_v=f"{len(str(i))}" # Get the number of digits in the current number
  num_f=str_n[int(st_v):]+str(i) # Format the number with leading zeros
  range_v.append(num_f)
```

Check the total number of generated corporate numbers.

```python
# Print the number of generated IDs
len(range_v)
```

## Batch Scraping with ThreadPoolExecutor (Synchronous)

This section demonstrates how to use `ThreadPoolExecutor` to speed up the scraping process by running multiple requests concurrently.

```python
collect_frame=[] # List to store DataFrames from each successful harvest
def compile_run():
    """
    Uses ThreadPoolExecutor to concurrently harvest data for a list of corporate numbers.
    """
    # Set up a progress bar for the harvesting process
    with tqdm(total=len(range_v),desc="harvest") as progress:
        # Use ThreadPoolExecutor for parallel execution
        with ThreadPoolExecutor(max_workers=10) as executor:
            # Map the harvest_page function to each number in range_v
            for i in executor.map(harvest_page,range_v):
               collect_frame.append(i) # Append the resulting DataFrame
               progress.update() # Update the progress bar
               time.sleep(.1) # Small delay to avoid overwhelming the server
               if len(collect_frame)%5==0:
                time.sleep(.2) # Slightly longer delay after a few successful harvests

# Execute the batch scraping process
compile_run()
```

## Google Colab Integration (Optional)

If running in Google Colab, you can mount your Google Drive to save the final output.

```python
# Mount Google Drive
# from google.colab import drive
# drive.mount('/content/drive') # This line is commented out
```

Combine all the collected DataFrames into a single `final_df`.

```python
final_df=pd.DataFrame()
for i in collect_frame:
    final_df=pd.concat([final_df,i],axis=0,ignore_index=True)
```

Analyze the results by counting how many entries were successfully parsed (`Allow`) versus those that were not.

```python
# Group by 'Test' column to count successful vs. unsuccessful harvests
final_df.groupby('Test').count()
```

The following line, if uncommented, would save the `final_df` to a CSV file in your Google Drive.

```python
#final_df.to_csv('/content/drive/MyDrive/Company Projects/Small task/Sample_Output_file.csv') # This line is commented out
```

Empty cells, often found at the end of notebooks.

```python

```

## Async Code Implementation

This section refactors the scraping logic to use `asyncio` and `aiohttp` for asynchronous web requests, which can be more efficient for I/O-bound tasks like web scraping.

```python
import aiohttp
import asyncio
import nest_asyncio

# Apply nest_asyncio to allow running asyncio in environments like Jupyter notebooks
nest_asyncio.apply()
```

The `harvest_page` function is now asynchronous.

```python
async def harvest_page(session,num):
        """
        Asynchronously fetches and parses corporate detail information.

        Args:
          session (aiohttp.ClientSession): The aiohttp client session.
          num (str): The corporate number to fetch data for.

        Returns:
          pandas.DataFrame: A DataFrame with parsed details or error indicators.
        """
        loo_df=pd.DataFrame()
        try:
          # Headers remain the same for the asynchronous request
          headers={ "Accept":"text/html,application/xhtml+xml,application/xml;q=0.9,application/signed-exchange;v=b3;q=0.7",
                        "Accept-Encoding":"gzip,deflate,br",
                        "Accept-Language":"en-US,en;q=0.9,ta;q=0.8",
                        "Connection":"keep-alive",
                        "Cookie":"_ga=GA1.1.2085200574.1702026901;_ga_HEFYBZ2QN2=GS1.1.1702285595.5.1.1702285603.0.0.0",
                        "Host":"arc-sos.state.al.us",
                        "Sec-Fetch-Dest":"document",
                        "Sec-Fetch-Mode":"navigate",
                        "Sec-Fetch-Site":"none",
                        "Sec-Fetch-User":"?1",
                        "Upgrade-Insecure-Requests":"1",
                        "User-Agent":"Mozilla/5.0(WindowsNT10.0;Win64;x64)AppleWebKit/537.36(KHTML,likeGecko)Chrome/120.0.0.0Safari/537.36",
                        "sec-ch-ua":"'Not_ABrand';v='8','Chromium';v='120','GoogleChrome';v='120'",
                        "sec-ch-ua-mobile":"?0",
                        "sec-ch-ua-platform":"Windows"
                      }
          url=f'https://arc-sos.state.al.us/cgi/corpdetail.mbr/detail?corp={num}'
          # Use the provided session to make an asynchronous GET request
          async with session.get(url, headers=headers) as response:
              status_code = response.status
              page_content = await response.text() # Asynchronously get response text
              sou=BeautifulSoup(page_content,'lxml')
              data_find=sou.text.strip()
              try:
                  cmp_name=sou.find('thead').find('td',class_="aiSosDetailHead").text.strip()
                  # Note: `res.status_code` is not available in the async context here.
                  # It should be `status_code` from the response object.
                  stus_code=status_code # Corrected: use status_code from response
                  datas0=[]
                  datas1=[]

                  for i in sou.find('div',attrs={'align':'center'}).find_all('tr'):
                    try:
                      attri_name=i.find('td',class_="aiSosDetailDesc").text.strip()
                      detail_value=i.find('td',class_="aiSosDetailValue").text.strip()
                      datas1.append(detail_value)
                      datas0.append(attri_name)
                    except:
                      pass

                  for i in range(len(datas0)):
                    loo_df[datas0[i]]=[str(datas1[i])]

                  loo_df['company']=[cmp_name]
                  loo_df['Url_number']=[num]
                  loo_df['Status']=[stus_code]
                  loo_df['Test']=["Allow"]
              except Exception as e:
                if "You have exceeded the number accesses allowed." in data_find:
                  print(f"Rate limited for {num}. Retrying after a short delay.")
                  # Asynchronous sleep
                  await asyncio.sleep(2)
                  return await harvest_page(session, num) # Retry asynchronously
                else:
                  print(f"Error parsing data for {num}: {e}")
                  loo_df['Url_number']=[num]
                  loo_df['Test']=["Not Have"]

              return loo_df
        except Exception as e: # Catching aiohttp.ClientError or other exceptions
          print(f"Request failed for {num}: {e}")
          loo_df['Url_number']=[num]
          loo_df['Test']=["Error"]
          return loo_df
```

The `main` function orchestrates the asynchronous scraping.

```python
collect_frame=[] # Re-initialize for async results
async def main(list_num):
    """
    Main asynchronous function to run the scraping tasks.
    """
    tasks = []
    # Create a single aiohttp ClientSession for efficiency
    async with aiohttp.ClientSession() as session:
        for perma in list_num:
            # Create a task for each corporate number
            tasks.append(harvest_page(session, perma))

        # Use asyncio.as_completed to process tasks as they finish
        with tqdm(total=len(list_num),desc="harvest") as progress:
          for task in asyncio.as_completed(tasks):
              data=await task # Await the result of each completed task
              collect_frame.append(data) # Append the DataFrame
              progress.update(1) # Update progress bar
              await asyncio.sleep(.1) # Small asynchronous sleep
              if len(collect_frame)%5==0:
                await asyncio.sleep(.2) # Slightly longer delay

# Run the main asynchronous function
if __name__ == "__main__":
    asyncio.run(main(range_v))
```

Another empty cell.

```python

```

Combine the results from the asynchronous scraping.

```python
final_df=pd.DataFrame()
for i in collect_frame:
    final_df=pd.concat([final_df,i],axis=0,ignore_index=True)
```

Extracting only the `Url_number` and `Test` columns for analysis.

```python
tests=final_df[['Url_number', 'Test']]
```

Displaying any entries that did not have the status `Allow`.

```python
# Show entries that were not successfully parsed
tests[tests['Test']!="Allow"]
```

Display the total number of items collected.

```python
# Total number of collected records
len(collect_frame)
```

Final empty cell.

```python

```
```

---

```markdown
# Alabama State Harvest

This notebook outlines a process for harvesting data from the Alabama Secretary of State's corporate detail pages. It explores different approaches, including sequential requests, multithreading, and asynchronous programming, to efficiently retrieve information.

## Importing Libraries

First, we import all the necessary libraries for data manipulation, web scraping, and asynchronous operations.

```python
# Importing Important libraries
import pandas as pd
from tqdm import tqdm
import re
import time
import urllib
import warnings
from tqdm import trange
import sys
import requests
from bs4 import BeautifulSoup
from concurrent.futures import ThreadPoolExecutor
```

## Initial Testing

Before implementing a full-scale harvesting strategy, we test the process on a single URL to ensure the scraping logic works correctly.

```python
url="https://arc-sos.state.al.us/cgi/corpdetail.mbr/detail?corp=000000008"
```

We define the headers to mimic a web browser for the request.

```python
headers={'user-agent':'Mozilla/5.0 (Windows NT 10.0; apple64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/116.0.0.0 Safari/537.36'}
```

We then make a GET request to the specified URL. `verify=False` is used to disable SSL certificate verification, which might be necessary if there are certificate issues with the target website. A timeout is also set to prevent indefinite waiting.

```python
res=requests.get(url,headers=headers,verify=False,timeout=10)
```

The HTML content is parsed using `BeautifulSoup` with the `lxml` parser.

```python
sou=BeautifulSoup(res.content,'lxml')
```

We extract and print the stripped text content of the page to get a general overview.

```python
sou.text.strip()
```

The company name is extracted from the table header (`thead`) and a specific table data cell (`td`) with the class `aiSosDetailHead`.

```python
cmp_name=sou.find('thead').find('td',class_="aiSosDetailHead").text.strip()
```

The extracted company name is then displayed.

```python
cmp_name
```

## Data Harvesting Function (Synchronous)

This section defines a function `harvest_page` that attempts to scrape data for a given corporate number. It handles potential errors and implements a retry mechanism for rate limiting.

```python
def harvest_page(num):
  loo_df=pd.DataFrame()
  url=f'https://arc-sos.state.al.us/cgi/corpdetail.mbr/detail?corp={num}'
  headers={ "Accept":"text/html,application/xhtml+xml,application/xml;q=0.9,application/signed-exchange;v=b3;q=0.7",
            "Accept-Encoding":"gzip,deflate,br",
            "Accept-Language":"en-US,en;q=0.9,ta;q=0.8",
            "Connection":"keep-alive",
            "Cookie":"_ga=GA1.1.2085200574.1702026901;_ga_HEFYBZ2QN2=GS1.1.1702285595.5.1.1702285603.0.0.0",
            "Host":"arc-sos.state.al.us",
            "Sec-Fetch-Dest":"document",
            "Sec-Fetch-Mode":"navigate",
            "Sec-Fetch-Site":"none",
            "Sec-Fetch-User":"?1",
            "Upgrade-Insecure-Requests":"1",
            "User-Agent":"Mozilla/5.0(WindowsNT10.0;Win64;x64)AppleWebKit/537.36(KHTML,likeGecko)Chrome/120.0.0.0Safari/537.36",
            "sec-ch-ua":"'Not_ABrand';v='8','Chromium';v='120','GoogleChrome';v='120'",
            "sec-ch-ua-mobile":"?0",
            "sec-ch-ua-platform":"Windows"
          }
  try:
    res=requests.get(url,headers=headers,timeout=40)
    sou=BeautifulSoup(res.content,'lxml')
    data_find=sou.text.strip()
    try:
        cmp_name=sou.find('thead').find('td',class_="aiSosDetailHead").text.strip()
        stus_code=res.status_code
        datas0=[]
        datas1=[]

        # Iterate through table rows to extract attribute names and detail values
        for i in sou.find('div',attrs={'align':'center'}).find_all('tr'):
          try:
            attri_name=i.find('td',class_="aiSosDetailDesc").text.strip()
            detail_value=i.find('td',class_="aiSosDetailValue").text.strip()

            datas1.append(detail_value)
            datas0.append(attri_name)

          except:
            pass

        # Populate DataFrame with extracted data
        for i in range(len(datas0)):
          loo_df[datas0[i]]=[str(datas1[i])]

        loo_df['company']=[cmp_name]
        loo_df['Url_number']=[num]
        loo_df['Status']=[stus_code]

        loo_df['Test']=["Allow"]
    except:
       # Handle rate limiting or other parsing errors
       if data_find in "You have exceeded the number accesses allowed.":
        time.sleep(1) # Wait before retrying
        return harvest_page(num) # Recursive call to retry
       else:
        loo_df['Test']=["Not Have"] # Mark as not found or parsed

    return loo_df
  except:
    # Handle general request errors
    return loo_df
```

This is an example of calling the `harvest_page` function with a specific number.

```python
# dtas_oyut=harvest_page('000000002') # Example usage, commented out
harvest_page('000000114')
```

The following commented-out line shows how to save the harvested data to a CSV file.

```python
# dtas_oyut.to_csv('ayeoirtqie.csv')
```

### Generating a Range of Numbers

This section generates a list of corporate numbers in a specific format, padded with leading zeros.

```python
str_n="000000000"
range_v=[]
for i in range(1,1000):
  st_v=f"{len(str(i))}"
  num_f=str_n[int(st_v):]+str(i)
  range_v.append(num_f)
```

We then check the length of the generated list.

```python
len(range_v)
```

## Multithreaded Harvesting

To speed up the harvesting process, we use `ThreadPoolExecutor` to make requests concurrently.

```python
collect_frame=[]
def compile_run():
    with tqdm(total=len(range_v),desc="harvest") as progress:
        with ThreadPoolExecutor(max_workers=10) as executor:
            for i in executor.map(harvest_page,range_v):
               collect_frame.append(i)
               progress.update()
               time.sleep(.1)
               if len(collect_frame)%5==0:
                time.sleep(.2)
compile_run()
```

If you are running this in Google Colab, you might need to mount your Google Drive to save the results.

```python
from google.colab import drive
drive.mount('/content/drive')
```

After the multithreaded harvesting is complete, we combine the results into a single DataFrame.

```python
final_df=pd.DataFrame()
for i in collect_frame:
    final_df=pd.concat([final_df,i],axis=0,ignore_index=True)
```

We then group the results by HTTP status code to see the distribution of responses.

```python
final_df.groupby('Status').count()
```

The final DataFrame containing all harvested data is displayed.

```python
final_df
```

## Asynchronous Harvesting

This section introduces asynchronous programming using `aiohttp` and `asyncio` for potentially faster and more efficient web scraping, especially when dealing with many concurrent requests.

```python
import aiohttp
import asyncio
import nest_asyncio
```

This is the asynchronous version of the `harvest_page` function. It uses `aiohttp` to make non-blocking HTTP requests.

```python
async def harvest_page(session,num):
        loo_df=pd.DataFrame()
        try:
          headers={ "Accept":"text/html,application/xhtml+xml,application/xml;q=0.9,application/signed-exchange;v=b3;q=0.7",
                        "Accept-Encoding":"gzip,deflate,br",
                        "Accept-Language":"en-US,en;q=0.9,ta;q=0.8",
                        "Connection":"keep-alive",
                        "Cookie":"_ga=GA1.1.2085200574.1702026901;_ga_HEFYBZ2QN2=GS1.1.1702285595.5.1.1702285603.0.0.0",
                        "Host":"arc-sos.state.al.us",
                        "Sec-Fetch-Dest":"document",
                        "Sec-Fetch-Mode":"navigate",
                        "Sec-Fetch-Site":"none",
                        "Sec-Fetch-User":"?1",
                        "Upgrade-Insecure-Requests":"1",
                        "User-Agent":"Mozilla/5.0(WindowsNT10.0;Win64;x64)AppleWebKit/537.36(KHTML,likeGecko)Chrome/120.0.0.0Safari/537.36",
                        "sec-ch-ua":"'Not_ABrand';v='8','Chromium';v='120','GoogleChrome';v='120'",
                        "sec-ch-ua-mobile":"?0",
                        "sec-ch-ua-platform":"Windows"
                      }
          url=f'https://arc-sos.state.al.us/cgi/corpdetail.mbr/detail?corp={num}'
          async with session.get(url, headers=headers,timeout=15) as response:
              status_code = response.status
              page_content = await response.text()
              sou=BeautifulSoup(page_content,'lxml')
              data_find=sou.text.strip()
              try:
                  cmp_name=sou.find('thead').find('td',class_="aiSosDetailHead").text.strip()
                  stus_code=res.status_code # Note: res is not defined here, this might be a bug. Should likely be 'response.status'
                  datas0=[]
                  datas1=[]

                  for i in sou.find('div',attrs={'align':'center'}).find_all('tr'):
                    try:
                      attri_name=i.find('td',class_="aiSosDetailDesc").text.strip()
                      detail_value=i.find('td',class_="aiSosDetailValue").text.strip()

                      datas1.append(detail_value)
                      datas0.append(attri_name)

                    except:
                      pass

                  for i in range(len(datas0)):
                    loo_df[datas0[i]]=[str(datas1[i])]

                  loo_df['company']=[cmp_name]
                  loo_df['Url_number']=[num]
                  loo_df['Status']=[stus_code]

                  loo_df['Test']=["Allow"]
              except:
                if data_find in "You have exceeded the number accesses allowed.":
                  time.sleep(1)
                  return harvest_page(session, num) # Corrected to pass session
                else:
                  loo_df['Url_number']=[num]
                  loo_df['Test']=["Not Have"]

              return loo_df
        except Exception as e: # Added a general exception catch
          print(f"Error harvesting {num}: {e}") # Log the error
          loo_df['Url_number']=[num]
          loo_df['Test']=["Error"]
          return loo_df
```

### Asynchronous Execution

This block sets up and runs the asynchronous harvesting process. `nest_asyncio.apply()` is used to allow nested asyncio loops, which can be helpful in environments like Jupyter notebooks.

```python
collect_datas=[]
async def main(links):
    tasks = []
    async with aiohttp.ClientSession() as session:
        # Limiting to first 500 for demonstration purposes
        for perma in range_v[:500]:
            tasks.append(harvest_page(session, perma))
        results = []

        # Using asyncio.as_completed to process results as they become available
        # Note: The progress bar and sleep logic here are outside the direct task completion loop,
        # and might not accurately reflect individual task progress in this specific structure.
        # It's also important to note that collect_frame is a global variable, which can be
        # problematic in complex async scenarios.
        for task in asyncio.as_completed(tasks):
            data=await task
            collect_frame.append(data)
            # progress.update() # 'progress' is not defined in this scope
            time.sleep(.2)
            if len(collect_frame)%5==0:
            time.sleep(.2)

if __name__ == "__main__":
    nest_asyncio.apply()
    # Running the main asynchronous function
    # Note: The 'progress' object is used in compile_run but not defined or initialized here.
    # This section likely needs refinement to integrate the progress bar correctly.
    asyncio.run(main(range_v))
```

## Iterative Loop for Error Handling and Retries

This advanced section implements a loop that continuously attempts to harvest data, specifically retrying any requests that resulted in an "Error" status.

```python
tests_range=range_v[:100] # Initial range to test
find_final_data=pd.DataFrame() # DataFrame to store successfully harvested data

while True:
    collect_frame=[] # Reset collect_frame for each iteration
    async def main(list_num):
        tasks = []
        async with aiohttp.ClientSession() as session:
            for perma in list_num:
                tasks.append(harvest_page(session, perma))
            results = []
            with tqdm(total=len(list_num),desc="harvest") as progress:
                for task in asyncio.as_completed(tasks):
                    data=await task
                    collect_frame.append(data)
                    time.sleep(.2)
                    progress.update(1)
                    if len(collect_frame)%10==0:
                        time.sleep(2)

    if __name__ == "__main__":
        nest_asyncio.apply()
        asyncio.run(main(tests_range)) # Run the async harvesting for the current 'tests_range'

    final_df=pd.DataFrame()
    # Combine all results from the current iteration
    for i in collect_frame:
        final_df=pd.concat([final_df,i],axis=0,ignore_index=True)

    # Separate successful results from errors
    find_df=final_df[final_df['Test']!="Error"]
    # Update 'tests_range' to contain only the Url_numbers that resulted in an 'Error'
    tests_range=final_df[final_df['Test']=="Error"]['Url_number'].to_list()

    # Append successful data to the overall 'find_final_data'
    find_final_data=pd.concat([find_df,find_final_data],axis=0)

    # Print progress statistics
    print(f"Total Count : {len(collect_frame)}")
    print(f"Find Count : {len(find_df)}")
    print(f"Not Fount : {len(tests_range)}")

    # Break the loop if there are no more errors to process
    if len(tests_range)==0:
        print('Break')
        break

```

Finally, we analyze the results to identify any URLs that did not allow harvesting or encountered errors.

```python
tests=find_final_data[['Url_number','Test']]
```

This displays all entries where the 'Test' column is not 'Allow', indicating either 'Not Have' or 'Error'.

```python
tests[tests['Test']!="Allow"]
```
```