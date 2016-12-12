import requests
import json
import pandas as pd
import csv
from datetime import date

testbed = {}
today = date.today().strftime('%Y%m%d')


# 필요한 알고리즘 리스트 불러오기
with open('/account_list.csv') as f:
    testbed_list = [tuple(line) for line in csv.reader(f)]
    del testbed_list[0]

    
# 테스트베드 홈페이지에서 데이터 가져오기
for (acnutSN, invtTyCd, account, hbrdAssetsAt, algorithmcd, algorithmnm, algorithmcom) in testbed_list:
    print(acnutSN)
    url = 'http://www.ratestbed.kr/portal/pblntf/chartsRatereturn.json?acnutSn=%s&invtTyCd=0%s&hbrdAssetsAt=%s&basicDt=20170124&odrSn=1&targetSe=P&basicDate=%s' %(acnutSN, invtTyCd, hbrdAssetsAt, today)
    doc = requests.post(url)
    companydata = json.loads(doc.text)['chartsRatereturnAcnut']
    testbed[(algorithmnm, algorithmcom, invtTyCd, acnutSN)] = [companydata[i]['standardPrice'] for i in range(len(companydata))]

    
# 테스트베드 홈페이지에서 날짜 열 가지고 오기
index_url = 'http://www.ratestbed.kr/portal/pblntf/chartsRatereturn.json?acnutSn=&invtTyCd=02&hbrdAssetsAt=00&basicDt=20170124&odrSn=1&targetSe=P&basicDate=%s' %today
index_data = json.loads(requests.post(index_url).text)["chartsRatereturnIndex"]
date_list = [index_data[i]["basicDate"] for i in range(len(index_data))]

raw = (pd.DataFrame(testbed, index=date_list)/1000)-1


# 회사별, 위험성향별 평균값 도출
with open('/algorithm_list.csv') as f:
    algorithm_list = [tuple(line) for line in csv.reader(f)]
    del algorithm_list[0]

average = {}
    
for (algorithmcd, algorithmnm, algorithmcom) in algorithm_list:
    for i in range(1,4):
        average[(algorithmnm, i)] = list(raw[algorithmnm][algorithmcom][str(i)].mean(axis=1))
mean = pd.DataFrame(average, index=date_list)

print('saving')
# xlsx로 저장하기
writer = pd.ExcelWriter('/testbed%s.xlsx' %today)
raw.T.to_excel(writer, sheet_name='raw data')
mean.T.to_excel(writer, sheet_name='average')
print('end')
