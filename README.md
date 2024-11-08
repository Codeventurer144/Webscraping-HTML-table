# Webscraping and Analyzing Data on Notable Firms in Nigeria.
## Introduction
Assuming I was given a job by a Nigerian Government Parastatal to fetch some data on a Federal government website containing the names of, and other information about several notable firms in Nigeria since the 1890s with the aim of creating a visual representation of their age and how many of them are still active, amongst other purposes. I am to use webscraping, HTML manipulation, as well as python analytic and visualization skills—all in Jupyter notebooks—to get the requested analysis.

## Problem Statement
-	What is the comparative age distribution of the notable firms?
-	What percentage of them are inactive?
-	What is the number of company (by sector) established each year?


## Skills/Concepts Demonstrated
- Webscraping 
- Html Manipulation
- Python analytics and Visualization

## Data Sourcing and Description
### Sourcing
The data was scraped from a HTML Table in this website https://en.wikipedia.org/wiki/List_of_companies_of_Nigeria ,into a Jupyter Notebook, and converted into a Pandas DataFrame.
The HTML table consists of a real-world dataset which contains 51 rows and 7 columns as tabularized with their descriptions below: 

### Description
Columns | Description
:-----------:|:----------------------:
Name | The Name of the Firm
Industry	| The industry each firm belongs to
Sector	 | The sector each firm belongs to
Headquarters | The headquarter location in Nigeria
Founded	| The year each firm was founded
Note | Further description of the firm
Private/State| ‘P’/’S’ identifies whether it is currently private or state owned
Active/Defunct | ‘A’/’D’ identifies whether it is currently active or inactive
## Methodology and Findings
### Methodology
#### Library and Modules Used
Using the code below, the following were imported into Jupyter Notebook
```python
from bs4 import BeautifulSoup
import requests
import pandas as pd
import lxml
print ('all libraries were installed')
```
-	requests (for getting the HTML content of the webpage)
-	BeautifulSoup (for parsing the content using lxml) 
-	lxml (was discovered by me to be the most suitable parser)
-	pandas (to create data frame(s) for convenient analysis)
#### Scraping the Raw HTML Table content
I used the [inspect] option to view the raw HTML structure of the website and the HTML table I desired to scrape; which I discovered was classified as “wikitable sortable”. This was then included as code parameter in finding the table.
```python
url = 'https://en.wikipedia.org/wiki/List_of_companies_of_Nigeria'
response = requests.get(url)
soup = BeautifulSoup(response.content, 'lxml')
Table1 = soup.find_all('table', class_="wikitable sortable")
Table1
```
#### Getting Table Headers and body
The following code was used to get the table headers. 
```python
ths = soup.find_all('th') #the '==$0' in the table structure would not allow me get all 'th' tags in the table
header_texts = [th.text.strip() for th in ths] #so I first extract all the 'th' tags in the page
first_seven_ths = ths[:7] #from the output above, the first seven (from Name to Status) are the text that I want
table_head = [th.text.strip() for th in first_seven_ths]
print(table_head)
```
Because of the ‘==$0’ here and there in the HTML structure of the page, it made extracting the table body impossible with the conventional methods. So I copied the table from the raw html into a variable ‘tbody_html_content’. Using BeautifulSoup and a ‘for loop’ I was able to parse the content and iterate through it; extracting only the text into variable ‘cells’. ‘cells’ was fitted into a dataframe and the table was ready for analysis, all using the code below.
####Data Transformation
In order to make the data suitable for the first visualization, I first sorted all the rows in the table according to the years (in ascending order), used the datetime module to fetch the current date, converted the type in the year column from string to integer, then created a new column called ‘Age’ which is the subtraction of current year with that of the founded years.
```python
df = df.sort_values(by='Founded', ascending=True)
# to visualize age of each company
from datetime import datetime
import matplotlib.pyplot as plt
current_year = datetime.now().year
df['Founded'] = df['Founded'].astype(int)
df['Age'] = current_year - df['Founded'] #current year minus yearfounded
```
For the third question, I had to create a new data frame which contained only three column using…
```python
df.groupby(['Founded', 'Sector']).size().reset_index(name='Count')
```
to fetch the columns that contain the year, the secotr and to create one that contains the count of each sector per year.
#### Visualization
##### Bar Chart
To answer the first question in the problem statement above (What is the comparative age distribution of the notable firms?). The following code was used to generate a bar chart:
```python
# PLOTTING A BAR CHART
plt.figure(figsize=(14, 8)) # the number of firms require this size for readability
plt.bar(df['Name'], df['Age'], color='skyblue', edgecolor='black')

plt.title('Age of Notable Firms in Nigeria')
plt.xlabel('Firm Name')
plt.ylabel('Age (Years)')
plt.xticks(rotation=45, ha='right')
plt.grid(axis='y', alpha=0.75) #75% opacity for a clearer visualization
plt.tight_layout()

plt.show()
```
##### Pie Chart
To answer the second question in the problem statement above (What percentage of them are inactive?). The following code was used to generate a pie chart:
```python 
# The number of active and defunct companies
status_counts = df['Active/Defunct'].value_counts()
labels = ['Active', 'Defunct']
sizes = [status_counts.get('A', 0), status_counts.get('D', 0)]

plt.figure(figsize=(6, 4))  # Size of the pie chart
plt.pie(sizes, labels=labels, autopct='%1.1f%%', startangle=90, colors=['skyblue', 'lightcoral'])
plt.title('Percentage of Active vs Defunct Companies')
plt.axis('equal') 

plt.show()
```
##### Scatter Plot
To answer the third question in the problem statement above (What is the number of company (by sector) established each year?). I figured that a scatter plot would do justice to the visualization, and I was right! The following code was used to generate a scatter plot:
```python
#To visualize the count of each sector by year
import seaborn as sns
# to group the df by Year and Sector to get counts
sector_year_counts = df.groupby(['Founded', 'Sector']).size().reset_index(name='Count')
sector_year_counts
sns.scatterplot(data=sector_year_counts, x='Sector', y='Founded', size='Count', 
                sizes=(50, 1000), hue='Sector', palette='Set2', legend=False, marker='o')

plt.title('Count of Companies by Sector and Year Founded', fontsize=16)
plt.xlabel('Sector', fontsize=12)
plt.ylabel('Year Founded', fontsize=12)
plt.xticks(rotation=45, ha='right')  # Rotate x-axis labels for better readability
plt.tight_layout()

plt.show()
```
### Findings 
{![](image.png)}. From the first visualization, the follow are noticeable:
-	First Bank is the oldest of the notable firms.
-	Dotts Media House and Jiji.ng (an online shop) are the youngest of them.
-	The history of Nigeria in one visualization.
-	First Bank marked the beginning of a more advance relationship between Nigeria and England.
-	Economic growth through international trade called for fast means of the movement of goods, Thus the Nigerian Railway company.
-	Discovery of Crude Oil in Nigeria attracted Shell, a foreign company.
-	The limitations of railways led to the need of road networks and related infrastructure which the Julius Berger Company took advantage of to start such projects.
-	The presence of several banks, as well as the failure of many, led to the building of the Central Bank of Nigeria to oversee all banks and regulate the National Currency.

{![](image.png)}.
From the second Visualization, the following is also clear:
-	Only 2% of the firms are currently defunct. This is remarkable as it reveals the ability of these firms to stand through the various seasons of Nigeria’s economic fluctuations

{![](image.png)}.
From the last visualization, one can observe the following:
-	Of all the notable firms, more Banks have been built from the late 1800s up until the last decade.
-	Airlines started appearing from after WW2 in 1945 (as is general Nigerian History knowledge).
-	Internet and Digital marketing firms emerged during the early 21st century (this is historically known as the early phase of the Internet/Digital age in Nigeria)
##Conslusion
This project has utilized python code to perform Webscraping, HTML manipulation, Data Analysis and Visualization to reveal the deep insight a seemingly basic dataset, showing and strengthening concepts and knowledge surrounding the Historical landscape of Nigeria as an economy.
