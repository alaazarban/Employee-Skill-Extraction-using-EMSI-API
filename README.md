# Skills Extraction from Resumes using EMSI API 


So you want to figure out where your skills fit into today’s job market. Maybe you’re just curious to see a comprehensive constellation of job skills, clean and standardized. Or you need a taxonomy of skills for a Resume parsing project. Well, the [EMSI skills API](https://api.emsidata.com/apis/skills) is one possible tool for the job!
In this tutorial, I will walk you through how to extract skills from raw resume texts using this API.

Getting started is as easy as signing up for the API’s free access. You’ll get authentication credentials emailed to you once you complete that process.
Import Statements

### We’ll use a few packages here, so let’s import those first:

```python
import requests
import json
import numpy as np
import pandas as pd
from pandas.io.json import json_normalize
```

All of these are pretty standard. I’m using the json_normalize package which is an easy means of converting JSON to Pandas DataFrames, which will be nicer for readability.

## Authenticating Your Connection

The first part of accessing the API is simply using the credentials in that signup email to establish a connection and get an access token. I ran the following in a cell in a Jupyter Notebook with Python.

```python
auth_endpoint = "https://auth.emsicloud.com/connect/token" # auth endpoint

client_id = "your_client_id" # replace 'your_client_id' with your client id from your api invite email
client_secret = "your_client_secret" # replace 'your_client_secret' with your client secret from your api invite email
scope = "emsi_open" # ok to leave as is, this is the scope we will used

payload = "client_id=" + client_id + "&client_secret=" + client_secret + "&grant_type=client_credentials&scope=" + scope # set credentials and scope
headers = {'content-type': 'application/x-www-form-urlencoded'} # headers for the response
access_token = json.loads((requests.request("POST", auth_endpoint, data=payload, headers=headers)).text)['access_token'] # grabs request's text and loads as JSON, then pulls the access token from that
```

This code results in an authentication JSON object, where one of the keys is the `access_token`. Here I’ve explicitly accessed the value of that key and assigned it to a variable of the same name for later use.

## The “Hello, World!” of EMSI’s Skills API
EMSI has multiple APIs, but we’ll be focused on the Skills API in this tutorial. To get started, we’re just going to use that access token to pull the full list of skills available to us.

### Pull the Global List of Job Skills

I wrote a simple function to pull the skills list and write it to a Pandas DataFrame for nicer formatting and readability.


```python
def extract_skills_list():
  all_skills_endpoint = "https://emsiservices.com/skills/versions/latest/skills" # List of all skills endpoint
  auth = "Authorization: Bearer " + access_token # Auth string including access token from above
  headers = {'authorization': auth} # headers
  response = requests.request("GET", all_skills_endpoint, headers=headers) # response
  response = response.json()['data'] # the data

  all_skills_df = pd.DataFrame(json_normalize(response)); # Where response is a JSON object drilled down to the level of 'data' key
  return all_skills_df

extract_skills_list()

```

I set the url to the skills list endpoint, concatenated the access token in with the necessary syntactical specifications for the API, and used the requests library to get the data. This results in the following global list of skills:

![Image of skills from EMSI API](https://miro.medium.com/max/1236/1*phFqrSRFeVrTnktme3pEQA.png)


You can see here there are both hard and soft skills, each skill has a unique ID, and each skill is standardized and proper cased. Each skill type has a type ID as well. There are nearly 30,000 skills listed here!

## Extract the Skills That Appear in a Given Document

Say instead you have a document (a resume or job description for example), and you want to find relevant skills that the resume holder has or the job poster wants. The following function will prompt you for a text input. Paste the text in there and set a confidence interval between 0 and 1 (I usually do 0.4 to see a longer list of skills), and skills extracted!

```python
url = "https://emsiservices.com/skills/versions/latest/extract"

# This text variable has all the content of my persoanl resume in raw text 
text = "EXPERIENCEHR Data AnalystSaudi Aramco 2019 - PresentDeveloped 12 dashboards on employees certifications, performance, and position holders’ data Worked on the design and development of Employee Productivity Measurement system Developed a weekly highlights text classifier tool based on supervised machine learning methods Built a sentiment analysis program on employees performance feedbackSAP S/4 HANA Technical Lead Saudi Aramco 2018 - PresentDesigned and built infrastructure, high availability and business continuity for SAP S/4 HANA applicationsHandled SAP HANA licensing and vendor communicationManaged the project across IT Infrastructure teamsHandled technical conversion from SAP ERP to S/4 HANASAP System Administrator Saudi Aramco 2014 - PresentInstallation, upgrade and maintenance of different SAP applicationsSupporting applications including SAP ERP, Business Warehouse and CRM systemsInstallation, upgrade and support of Oracle and SAP HANA databasesDesigning and building high availability and business continuity for databases and applicationsBusiness Continuity Drill Project Manager Saudi Aramco 2020Built business continuity plan, scope and scenarios for Saudi Aramco systemsConducted meetings with service owners and proponentsManaged building infrastructure, designing access and restoring data for 16 applicaitons in remote data centerData Analysis and Data Engineering Nanodegrees Instructor Udacity 2019Mentored 32 students from different countriesPrepared and Delivered class materials in different data-related topics                  Recruitment Specialist Recruitment Specialist Udacity 2019Worked as part of Career Placement Program for SaudisMatching Misk-Udacity graduates with job opportunitiesConducted meetings with Recruitment Managers in Saudi companiesUnix to Saudi Aramco Private Cloud Migration Project Manager Saudi Aramco 2017Built optimized SAP systems migration proceduresManaged and worked on the migration of 40TB total size applications to Saudi Aramco virtualized environment on LinuxACHIEVEMENTS14 Approved Ideas in Saudi Aramco Innovation SystemSaudi AramcoIdeas recognized by IT Management. Reduced man-hour, streamlined and increased efficiency of processes that resulted in total cost saving of $1.8MMFiled patentSaudi Aramco 2020-05-01Filed a patent titled “Optimized Database Migration for Large Systems Between Platforms with Different Endianness”CERTIFICATIONUdacity Data Analyst 2019Udacity Data Engieer 2019Udacity Data Scientist, 2019Udacity Predictive Data Analyst 2019 SAP HANA Technology Associate, 2018 AWS Cloud Practioner, 2018SAP HANA Application Associate 2017Redhat System Administrator 2016SAP Netweaver Technology Associate with Oracle Database 2015"
text = str(text.encode("iso-8859-1", 'ignore'))

orig_text = "... Great candidates, /4 also have' Experience with a particular JS MV* framework (we happen to use React) Experience working with databases Experience with AWS Familiarity with microservice architecture Familiarity with modern CSS practices, e.g. LESS, SASS, CSS-in-JS ..."
payload = "{ \"text\": \""+text+"\", \"confidenceThreshold\": 0.4 }"
headers = {
    'authorization': "Bearer "+access_token,
    'content-type': "application/json"
    }

response = requests.request("POST", url, data=payload, headers=headers)

skills_extract = pd.DataFrame(json_normalize(response.json()['data']))

skills_extract
```


![Output Image](https://drive.google.com/uc?id=1zE3JgE-ufg22-jxeDA9D2m23vyx0TDUP)

