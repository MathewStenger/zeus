# -*- coding: utf-8 -*-
"""
Created on Wed Jan 17 10:35:50 2018

@author: Mathew Stenger
"""

import fitbit
import gather_keys_oauth2 as Oauth2
import pandas as pd 
import datetime

#Update based on Fitbit values https://dev.fitbit.com/apps/new
CLIENT_ID = '22CQ5S'
CLIENT_SECRET = '36d06a53aa5c889e4068d20b6974cb51'

""" 
Callback URL http://127.0.0.1:8080/

OAuth 2.0: Authorization URI
https://www.fitbit.com/oauth2/authorize 

OAuth 2.0: Access/Refresh Token Request URI
https://api.fitbit.com/oauth2/token 
"""
#Autorization Tokens derived from ID/Secret
server = Oauth2.OAuth2Server(CLIENT_ID, CLIENT_SECRET)
server.browser_authorize()

ACCESS_TOKEN = str(server.fitbit.client.session.token['access_token'])
REFRESH_TOKEN = str(server.fitbit.client.session.token['refresh_token'])

auth2_client = fitbit.Fitbit(CLIENT_ID, CLIENT_SECRET, oauth2=True, access_token=ACCESS_TOKEN, refresh_token=REFRESH_TOKEN)


#Date conversion
yesterday = str((datetime.datetime.now() - datetime.timedelta(days=1)).strftime("%Y%m%d"))
yesterday2 = str((datetime.datetime.now() - datetime.timedelta(days=1)).strftime("%Y-%m-%d"))
today = str(datetime.datetime.now().strftime("%Y%m%d"))


#HR Data
fit_statsHR = auth2_client.intraday_time_series('activities/heart', base_date=yesterday2, detail_level='1sec')

time_list = []
val_list = []

for i in fit_statsHR['activities-heart-intraday']['dataset']:
    val_list.append(i['value'])
    time_list.append(i['time'])

heartdf = pd.DataFrame({'Heart Rate':val_list,'Time':time_list})

print(heartdf.head())


heartdf.to_csv('/Users/shsu/Downloads/python-fitbit-master/Heart/heart'+ \
               yesterday+'.csv', \
               columns=['Time','Heart Rate'], header=True, \
               index = False)

# Sleep Data
fit_statsSl = auth2_client.sleep(date='today')
stime_list = []
sval_list = []

for i in fit_statsSl['sleep'][0]['minuteData']:
    stime_list.append(i['dateTime'])
    sval_list.append(i['value'])

sleepdf = pd.DataFrame({'State':sval_list,
                     'Time':stime_list})

sleepdf['Interpreted'] = sleepdf['State'].map({'2':'Awake','3':'Very Awake','1':'Asleep'})
sleepdf.to_csv('/Users/shsu/Downloads/python-fitbit-master/Sleep/sleep' + \
               today+'.csv', \
               columns = ['Time','State','Interpreted'],header=True, 
               index = False)