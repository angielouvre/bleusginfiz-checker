app.py
import streamlit as st
import requests
import pandas as pd
import json
import time
from datetime import datetime
from concurrent.futures import ThreadPoolExecutor

st.set_page_config(page_title="羅浮宮查票神器", layout="centered")

API_ENDPOINT = "https://www.ticketlouvre.fr/louvre/b2c/RemotingService.cfc?method=doJson"
date_timelist_dict = {}
current_month = datetime.now().month
current_year = datetime.now().year
months = [current_month + i for i in range(4)]
months = [m if m <= 12 else m-12 for m in months]

month = st.selectbox("選擇月份", months)
inGroup = st.selectbox("票種", ["group", "individual"])
TIMESLOT_SET = None

def query_time_list(date_string):
    query_body = {
        'eventAk': 'LVR.EVN21' if inGroup == "group" else 'LVR.EVN15',
        'eventName': 'performance.read.nt',
        'selectedDate': date_string,
        'eventCode': 'GA' if inGroup == "group" else 'MusWeb'
    }
    r = requests.post(API_ENDPOINT, data=query_body)
    response_dict = json.loads(r.text)
    performance_list = response_dict['api']['result']['performanceList']
    time_list = [perf['perfTime'] for perf in performance_list]
    return time_list

def query_timeslot_availability(date, performanceId, performanceAk):
    try:
        query_body = {
            'eventName': 'ticket.list',
            'dateFrom': date,
            'eventCode': 'GA' if inGroup == 'group' else 'MusWeb',
            'performanceId': performanceId,
            'priceTableId': '1',
            'performanceAk': performanceAk
        }
        r = requests.post(API_ENDPOINT, data=query_body)
        response_dict = json.loads(r.text)
        product_list = response_dict['api']['result']['product.list']
        index = 0 if inGroup == 'group' else 1
        return len(product_list) > 2 and product_list[index]['available'] > 0
    except:
        return False

def query_data(month):
    global TIMESLOT_SET
    data = {
        'year': current_year if month >= current_month else current_year + 1,
        'month': month,
        'eventCode': 'GA' if inGroup == "group" else 'MusWeb',
        'eventAk': 'LVR.EVN21' if inGroup == "group" else 'LVR.EVN15',
        'eventName': 'date.list.nt',
    }
    r = requests.post(API_ENDPOINT, data=data)
    response_dict = json.loads(r.text)
    date_list = response_dict['api']['result']['dateList']
    
    st.subheader(f"{month}月票況")
    for dateObj in date_list:
        weekday = pd.Timestamp(dateObj['date']).day_name()
        available_timeslots = []

        for i, timeslot in enumerate(dateObj['performanceRefList']):
            if query_timeslot_availability(dateObj['date'], timeslot['id'], timeslot['ak']):
                if dateObj['date'] not in date_timelist_dict:
                    date_timelist_dict[dateObj['date']] = query_time_list(dateObj['date'])
                available_timeslots.append(date_timelist_dict[dateObj['date']][i])

        if available_timeslots:
            st.success(f"{dateObj['date']} ({weekday}) 有票：{'，'.join(available_timeslots)}")
        else:
            st.error(f"{dateObj['date']} ({weekday}) 無票")

if st.button("查詢票券"):
    query_data(month)
