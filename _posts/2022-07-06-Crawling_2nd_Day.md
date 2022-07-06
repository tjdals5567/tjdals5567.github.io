---
layout: single
title:  "웹크롤링 part2 - 빅리더 10일차"
categories : crawling
tags:
  - BigLeader2022
  - Crawling
  - day10
toc: true
toc_sticky: true
---

# Chap 3. 항목별 내용 추출 후 다양한 형식의 파일로 저장하기

## 1. 작업 개요

이번 시간에는 제목과 내용과 저자 등의 다양한 정보를 각각 따로 추출해서 다양한 형식의 파일 로 저장하는 방법을 살펴보겠습니다. 그리고 많은 양의 데이터를 수집하기 위해 페이지를 변경하 면서 조회하는 방법도 함께 살펴보겠습니다.

그리고 추출된 데이터들을 xls 형식과 csv 형식과 txt 형식의 파일로 저장하는 방법을 전해 드리 겠습니다. 

- 학습목표
    1. 검색된 결과에서 항목별로 데이터를 추출할 수 있다.
    2. 항목별로 추출된 데이터들을 표형태로 만들어서 다양한 형식의 파일로 저장할    수 있다.
    3. 수집할 데이터의 양이 많을 경우 페이지를 바꾸면서 작업할 수 있다.

## 2. 예제코드

- riss.kr에서 전염병을 검색 후 학위 논문을 크롤링 해보겠습니다.

- 순서
    1. riss.kr에 접속합니다.
    2. '전염병'을 검색합니다.
    3. 지정한 건수 만큼 학위논문을 크롤링합니다.
    4. 지정한 형식의 파일로 저장합니다.

### Step 1. 필요한 모듈을 로딩합니다.


```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.chrome.service import Service
import time     
```

### Step 2. 사용자에게 검색 관련 정보들을 입력 받습니다.


```python
print("=" *100)
print(" 이 크롤러는 RISS 사이트의 논문 및 학술자료 수집용 웹크롤러입니다.")
print("=" *100)
query_txt = input('1.수집할 자료의 키워드는 무엇입니까? : ')
```

### Step 3. 수집된 데이터를 저장할 파일 이름 입력받기 


```python
ft_name = input('2.결과를 저장할 txt형식의 파일명을 쓰세요(예: c:\\py_temp\\riss.txt): ')
fc_name = input('3.결과를 저장할 csv형식의 파일명을 쓰세요(예: c:\\py_temp\\riss.csv): ')
fx_name = input('4.결과를 저장할 xls형식의 파일명을 쓰세요(예: c:\\py_temp\\riss.xls): ')
```

### Step 4. 크롬 드라이버 설정 및 웹 페이지 열기


```python
s = Service("c:/py_temp/chromedriver.exe")
driver = webdriver.Chrome(service=s)
url = 'https://www.riss.kr/'
driver.get(url)
time.sleep(5)
driver.maximize_window()
```

### Step 5. 자동으로 검색어 입력 후 조회하기


```python
element = driver.find_element(By.ID,'query')
driver.find_element(By.ID,'query').click( )
element.send_keys(query_txt)
element.send_keys("\n")
```

### Step 6.학위 논문 선택하기


```python
driver.find_element(By.LINK_TEXT,'학위논문').click()
time.sleep(2)
```

### Step 7.Beautiful Soup 로 본문 내용만 추출하기


```python
from bs4 import BeautifulSoup
html_1 = driver.page_source
soup_1 = BeautifulSoup(html_1, 'html.parser')

content_1 = soup_1.find('div','srchResultListW').find_all('li')
for i in content_1 :
    print(i.get_text().replace("\n",""))
```

### Step 8. 총 검색 건수를 보여주고 수집할 건수 입력받기


```python
import math
total_cnt = soup_1.find('div','searchBox pd').find('span','num').get_text()
print('검색하신 키워드 %s (으)로 총 %s 건의 학위논문이 검색되었습니다' %(query_txt,total_cnt))
collect_cnt = int(input('이 중에서 몇 건을 수집하시겠습니까?: '))
collect_page_cnt = math.ceil(collect_cnt / 10)
print('%s 건의 데이터를 수집하기 위해 %s 페이지의 게시물을 조회합니다.' %(collect_cnt,collect_page_cnt))
print('=' *80)
```

### Step 9. 각 항목별로 데이터를 추출하여 리스트에 저장하기


```python
no2 = [ ]        #번호 저장
title2 = [ ]     #논문제목 저장
writer2 = [ ]    #논문저자 저장
org2 = [ ]       #소속기관 저장
no = 1

for a in range(1, collect_page_cnt + 1) :
    
    html_2 = driver.page_source
    soup_2 = BeautifulSoup(html_2, 'html.parser')

    content_2 = soup_2.find('div','srchResultListW').find_all('li')
    
    for b in content_2 :    
        #1. 논문제목 있을 경우만
        try :
            title = b.find('div','cont').find('p','title').get_text()
        except :
            continue
        else :
            f = open(ft_name, 'a' , encoding="UTF-8")
            print('1.번호:',no)
            no2.append(no)
            f.write('\n'+'1.번호:' + str(no))

            print('2.논문제목:',title)
            title2.append(title)
            f.write('\n' + '2.논문제목:' + title)
            
            writer = b.find('span','writer').get_text()
            print('3.저자:',writer)
            writer2.append(writer)
            f.write('\n' + '3.저자:' + writer)

            org = b.find('span','assigned').get_text()
            print('4.소속기관:' , org)
            org2.append(org)
            f.write('\n' + '4.소속기관:' + org + '\n')
            
            f.close( )
            
            no += 1
            print("\n")
            
            if no > collect_cnt :
                break

            time.sleep(1)        # 페이지 변경 전 1초 대기 

    a += 1 
    b = str(a)

    try :
        driver.find_element(By.LINK_TEXT ,'%s' %b).click() 
    except :
        driver.find_element(By.LINK_TEXT('다음 페이지로')).click()
        
print("요청하신 작업이 모두 완료되었습니다")
```

### Step 10. 수집된 데이터를 xls와 csv 형태로 저장하기


```python
import pandas as pd 

df = pd.DataFrame()
df['번호']=no2
df['제목']=pd.Series(title2) #리스트에 결측치가 있을 시 pd.Series로 선언하면 만들어짐
df['저자']=pd.Series(writer2)
df['소속(발행)기관']=pd.Series(org2)

# xls 형태로 저장하기
df.to_excel(fx_name,index=False, encoding="utf-8" , engine='openpyxl')

# csv 형태로 저장하기
df.to_csv(fc_name,index=False, encoding="utf-8-sig")

print('요청하신 데이터 수집 작업이 정상적으로 완료되었습니다')
```

     - 리스트에 결측치가 있을 시를 대비하여 pd.Series형식으로 변환하여 저장합니다. 데이터프레임으로 변환시 결측치를 포함하여 반환시켜줍니다.

## 3. 실행


```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.chrome.service import Service
import time     

print("=" *100)
print(" 이 크롤러는 RISS 사이트의 논문 및 학술자료 수집용 웹크롤러입니다.")
print("=" *100)
query_txt = input('1.수집할 자료의 키워드는 무엇입니까? : ')

ft_name = input('2.결과를 저장할 txt형식의 파일명을 쓰세요(예: c:\\py_temp\\riss.txt): ')
fc_name = input('3.결과를 저장할 csv형식의 파일명을 쓰세요(예: c:\\py_temp\\riss.csv): ')
fx_name = input('4.결과를 저장할 xls형식의 파일명을 쓰세요(예: c:\\py_temp\\riss.xls): ')

s = Service("c:/py_temp/chromedriver.exe")
driver = webdriver.Chrome(service=s)
url = 'https://www.riss.kr/'
driver.get(url)
time.sleep(5)
driver.maximize_window()

element = driver.find_element(By.ID,'query')
driver.find_element(By.ID,'query').click( )
element.send_keys(query_txt)
element.send_keys("\n")

driver.find_element(By.LINK_TEXT,'학위논문').click()
time.sleep(2)

from bs4 import BeautifulSoup
html_1 = driver.page_source
soup_1 = BeautifulSoup(html_1, 'html.parser')

content_1 = soup_1.find('div','srchResultListW').find_all('li')
for i in content_1 :
    print(i.get_text().replace("\n",""))
    
import math
total_cnt = soup_1.find('div','searchBox pd').find('span','num').get_text()
print('검색하신 키워드 %s (으)로 총 %s 건의 학위논문이 검색되었습니다' %(query_txt,total_cnt))
collect_cnt = int(input('이 중에서 몇 건을 수집하시겠습니까?: '))
collect_page_cnt = math.ceil(collect_cnt / 10)
print('%s 건의 데이터를 수집하기 위해 %s 페이지의 게시물을 조회합니다.' %(collect_cnt,collect_page_cnt))
print('=' *80)

no2 = [ ]        #번호 저장
title2 = [ ]     #논문제목 저장
writer2 = [ ]    #논문저자 저장
org2 = [ ]       #소속기관 저장
no = 1

for a in range(1, collect_page_cnt + 1) :
    
    html_2 = driver.page_source
    soup_2 = BeautifulSoup(html_2, 'html.parser')

    content_2 = soup_2.find('div','srchResultListW').find_all('li')
    
    for b in content_2 :    
        #1. 논문제목 있을 경우만
        try :
            title = b.find('div','cont').find('p','title').get_text()
        except :
            continue
        else :
            f = open(ft_name, 'a' , encoding="UTF-8")
            #print('1.번호:',no) #잘 실행되고 있는지 확인하고 싶으면 사용하세요
            no2.append(no)
            f.write('\n'+'1.번호:' + str(no))

            #print('2.논문제목:',title)
            title2.append(title)
            f.write('\n' + '2.논문제목:' + title)
            
            writer = b.find('span','writer').get_text()
            #print('3.저자:',writer)
            writer2.append(writer)
            f.write('\n' + '3.저자:' + writer)

            org = b.find('span','assigned').get_text()
            #print('4.소속기관:' , org)
            org2.append(org)
            f.write('\n' + '4.소속기관:' + org + '\n')
            
            f.close( )
            
            no += 1
            print("\n")
            
            if no > collect_cnt :
                break

            time.sleep(1)        # 페이지 변경 전 1초 대기 

    a += 1 
    b = str(a)

    try :
        driver.find_element(By.LINK_TEXT ,'%s' %b).click() 
    except :
        driver.find_element(By.LINK_TEXT('다음 페이지로')).click()
        
print("요청하신 작업이 모두 완료되었습니다")

import pandas as pd 

df = pd.DataFrame()
df['번호']=no2
df['제목']=pd.Series(title2) #리스트에 결측치가 있을 시 pd.Series로 선언하면 만들어짐
df['저자']=pd.Series(writer2)
df['소속(발행)기관']=pd.Series(org2)

# xls 형태로 저장하기
df.to_excel(fx_name,index=False, encoding="utf-8" , engine='openpyxl')

# csv 형태로 저장하기
df.to_csv(fc_name,index=False, encoding="utf-8-sig")

print('요청하신 데이터 수집 작업이 정상적으로 완료되었습니다')
```

    ====================================================================================================
     이 크롤러는 RISS 사이트의 논문 및 학술자료 수집용 웹크롤러입니다.
    ====================================================================================================
    1.수집할 자료의 키워드는 무엇입니까? : 전염병
    2.결과를 저장할 txt형식의 파일명을 쓰세요(예: c:\py_temp\riss.txt): Epidemic.txt
    3.결과를 저장할 csv형식의 파일명을 쓰세요(예: c:\py_temp\riss.csv): Epidemic.csv
    4.결과를 저장할 xls형식의 파일명을 쓰세요(예: c:\py_temp\riss.xls): Epidemic.xls
    1전염병 모델의 연대기적 분석과 후속모델에 대한 고찰이상원高麗大學校 大學院2012국내석사RANK : 27772927원문보기목차검색조회음성듣기전염병은 고대 시대부터 줄곧 인간을 위협해왔으며, 기원전 430년부터 428년에 발생한 아테네지역의 전염병이 Thucydides에 의해 보고된 이후, 전 세계적으로 많은 사망자를 발생하게 한 전염병들이 다수의 생물학자들에 의해 분석되어 졌다. 전염병에 대한 연구가 2차 세계대전 이전에는 단순히 전염병을 관찰하는데 그친 반면, 2차 대전이후에는 전염병을 무기화 하고 그것을 어떻게 통제할 것인지에 초점이 맞춰지기 시작하였다. 또한 페니실린에 발견으로 이러한 질병을 통제하는 방법이 다변화되었으며, Bernoulli가 천연두의 확산에 대해 우두접종의 긍정적인 효과를 보고 한 후, Kermack-McKendrick에 의해 수학에 기초를 둔 모델링이 시작되었다. 이는 생물학적인 분석에 수학적인 사고를 접목하여, 전염병을 좀 더 체계적으로 분석하고자 하는 시도로 볼 수 있다.  Anderson-May는 좀 더 이러한 연구들을 발전시켰으며, 현재 많은 연구자들에 의해 새로운 발생하는 전염병들에 대해 연구가 이루어지고 있다. 특히, 상미분방정식과 편미분방정식을 도구로 하여 여러 가지 형태의 전염병 모델을 발전시킴으로써 전염병을 효과적으로 통제할 수 있는 연구가 활발히 진행되어 지고 있다.  그동안 많은 연구에서 전염병을 통제하는데 가장 중요한 요소는 전염병 전파에 의해 영향 받지 않는 점(Disease free equilibrium : DFE)에 대한 의 값으로서, 그 값이  보다 크거나 혹은 작은 경우에 따라 전염병의 통제 여부를 판단하여 왔으나, 최근에는 통제 재생산률(Control reproduction rate : 모델링에서 각종 통제 변수들에 의해 변동되어지는 질병이 전파되는 비율)라는 값이 전염병을 통제하는 새로운 요소로 연구되어 지고 있다.   본 논문에서는 지금까지 발전되어온 수많은 논문들에서 큰 틀을 이루고 있는 전염병에 대한 모델들을 연대기적으로 나열하고 분석함으로써, 기존의 모델에서 중요한 변수를 설정하는 방법을 고찰하고, 향후 고려할 수 있는 발전된 모델들에 대해 예측해 본다.
    원문보기
    목차검색조회
    음성듣기
    I portrayed these aspects of daily life in digital drawings, acrylic paintings, and oil paintings. I also did two videos: one is to showcase the process of making drawings (esquisse) with digital media; and the other is to display each story in the form of a diary along with photographs of my works. Characterized by a mixed use of traditional and digital media, my work is bound up with the circumstances of the times in which offline space is more indistinguishably jumbled with online space. I have documented altered aspects of daily life attributed to an infectious disease in writings and paintings and studied them in association with social situations. This work is the product of an individual’s effort to survived, adapt, and overcome the infectious disease, and it is hoped it will provide solace and arouse sympathy from contemporary people. In addition to this, I hope this dissertation could serve as an opportunity to predict the future by looking back on quotidian aspects from the outbreak and spread of COVID-19 and this study on present aspects of daily life would be a milestone to prepare future aspects of daily life.rtaining to current events. Those who are usually indifferent to current affairs more frequently watch news programs to identify confirmed cases at home and abroad and their traffic. This led to a growing interest in all current affairs. I portrayed aspects of my family who perceive realities through media coverage and considered the side effects that might be brought on in the course of embracing media reports. The fourth is the change in pursuit of leisure or hobbies. A home café culture has been created and leisure activities including growing plants at home have increased. Aspects of traveling and outing have also changed. I represented this situation, adapting quickly to an altered hobby life. 
    원문보기
    목차검색조회
    음성듣기
    3전염병과 펜데믹 상황에서 메디컬 처치의 운영에 관한 연구이재훈칼빈대학교 신학대학원2021국내석사RANK : 27772927원문보기목차검색조회
    원문보기
    목차검색조회
    마지막으로 SNS 사용 정도에 따라 전염병 위험 지각과 관광행동에 간의 영향 관계에 있어서 차이가 있는지 보았다. 이는 저사용 집단의 경우, 전염병에 대한 정보를 찾아보고 체계적으로 생각하는 것이 관광행동에 영향을 미치고 있었다. 한편 고사용 집단의 경우, 전염병에 대한 정보를 찾아본다거나 체계적으로 생각해서 그 판단에 의해 관광행동을 결정하는 것이 아니라 위험에 대한 지각이 직접적으로 관광을 하고자 하는 행동에 영향을 미치고 있는 것으로 확인되었다. 따라서 전염병에 대한 정보를 찾아보는 SNS 저사용 집단을 위해서는 TV나 문자, 신문, 방송을 통해 전염병에 대한 정보를 전달하는 것이 중요하다고 볼 수 있으며, 반면 SNS 고사용자를 위해서는 SNS를 통해서 전염병에 대한 최신 정보 및 정확한 정보를 SNS를 통해서 전달하는 것이 중요할 것으로 확인되었다.해외여행 목적의 출국자가 전년도 동월보다 93.9% 감소하였으며, 국내 입국자 역시 95%로 급감한 것으로 확인하였다.
    원문보기
    목차검색조회
    음성듣기
    5국제보건규칙 개정에 따른 우리나라 검역전염병 지정방안김윤호경북대학교 보건대학원2007국내석사RANK : 27772927원문보기목차검색조회음성듣기국제보건규칙 개정에 따라 우리나라 검역전염병의 수정이 필요한 시점에 우리나라의 검역전염병의 지정근거를 제시하기 위하여 우리나라와 세계주요국의 검역전염병, 해외유입전염병 중 일부, 세계보건규칙에 예시된 전염병을 중심으로 검토하였다. 치명률, 유행가능성, 예방접종효과, 격리효과, 국내해외유입전염병 발생 유무, 검역적용여부 등 6가지 항목을 고려한 검역전염병 지정타당도를 개발 적용하였다본 연구결과를 요약하면 다음과 같다.  첫째 우리나라 검역전염병은 국제보건규칙에서 정한 검역전염병과 똑같은 변화과정을 거쳤다. 다른나라의 경우 세계보건규칙에서 정한 검역전염병 이외에 주변국의 전염병발생현황 및 자국에 맞는 검역전염병을 지정운영하여 왔다.  둘째 우리나라와 중국을 제외한 미국, 일본, 호주, 캐나다에서는 에볼라, 콩고크리미아 출혈열등 바이러스성 출혈열을 검역대상 전염병으로 관리하고 있었다.  셋째 우리나라에서는 검역전염병으로 지정하고 있지 않으나 다른나라에서 검역법에 규정하고 있는 질병은 바이러스출혈, 세계적으로 박멸을 선언했던 천연두, 디프테리아, 홍역, 유행성이하선염, 신증후성출혈열, 리프트계곡열등 28종이다.  넷째. 일본와 중국은 검역전염병이외의  질병발생을 감시하기 위해 검역법에 그 대상을 명시하고 있었다.  다섯째 검토대상질병을 대상으로 지정타당도를 적용한 결과 결핵, 보툴리누스증, 야토병은 각1점을 유행가능성은 높지만 예방접종이나 격리가 이 병의 전파방지에 도움이 되지 않기 때문에 지정타당도 점수가 낮은 콜레라를 비롯하여 천연두, 라싸열, 웨스트나일열, 재귀열, 인플루엔자, 탄저는 2점 이다. 3점은 황열, 마르부르그병, 콩코크리미아, 에볼라, 사스, 뎅기열, 말라리아, 일본뇌염, 한타바이러스폐증후군, 폴리오, 홍역, 공수병이며 디프테리아, 조류인플루엔자 인체감염, 신증후성출혈열, 장티푸스, 수막구균성수막염, 리프트계곡열이 4점을 받고 페스트, 유행선이하선염이 5점을 받았다. A형유행성독감은 치명률을 알 수 없어 지정타당도 점수를 구하지 못하였다.  본 연구를 토대로 지정타당도 점수에서 2점을 받은 전염병 중 전염병예방법 제1군전염병을 검역전염병으로 3점이상을 받은 전염병 중 전염병예방법의 제2군전염병은 감시전염병으로 지정하고 그 이외 3점이상을 받은 전염병을 검역전염병으로 추가지정 운영을 제안해 본다.
    원문보기
    목차검색조회
    음성듣기
    In this paper, we will conduct in-depth research on the Prevention of hospital infection of respiratory infectious diseases, and in the future, we will conduct in-depth research on more hospitals to avoid and reduce hospital infection. It is hoped that the study will contribute to the avoidance and reduction of hospital infections of respiratory infectious diseases.xample analysis. According to the evaluation results and the actual specific situation of the hospital, preventive measures are put forward according to the hospital infection prevention standard level.Before the risk of nosocomial infection is serious, one pedestrian line and one motor vehicle line are set in the hospital. The underground access line to the treatment area is closed, which can reserve an entrance to the treatment area.When the risk of nosocomial infection is serious, the space is transformed. This can integrate the treatment area into a treatment area. A ground entrance is provided which can close the underground parking lot. In the common area of the diagnosis and treatment area, a spatial transformation was carried out to allow patients to enter the diagnosis and treatment unit by the side of the corridor. When the risk of infection is particularly serious, the emergency department, outpatient department, examination room and ward of key diagnosis and treatment units can be transformed through the space, so that patients can enter the diagnosis and treatment room from one side of the corridor. At the same time, the spatial analysis parameters reach the ideal value of prevention.s and treatment unit, select the highest connection value, the highest control value, the average depth, the average degree of integration, comprehensibility, and plotted into a comprehensive data map.
    원문보기
    목차검색조회
    음성듣기
    7전염병에 대한 기독교 견해와 고찰이성훈칼빈대학교 신학대학원2021국내석사RANK : 27772927원문보기목차검색조회
    원문보기
    목차검색조회
    8우리나라 법정전염병 관리실태 및 예방대책에 관한 연구심상숙경희대학교 대학원1994국내석사RANK : 27772926복사/대출신청 
    복사/대출신청 
    9학령전 아동을 위한 호흡기전염병 예방 프로그램의 개발 및 효과에 관한 연구김일옥梨花女子大學校1999국내박사RANK : 27772926원문보기There are many critical developmental periods among preschool children and especially their health in this period can be the base of their health throughout their lives. Whereas, many preschool children today are not being given enough caring than in the past because of the increased number of women who have jobs, and increased opportunities for higher education for women. So it can be said that the burden of kindergartens or nursery schools on caring and health education for preschoolers has been increased. There are many problems in health education for preschoolers because  their cognitive ability is still developing. Therefore, the health education programs in this period of time must fit for the development of their cognitive ability and their health care needs. Especially, the concept of their cause-result must be considered. So I attempted to develop a health education program which fits preschoolers cognitive level and their health care needs. To enhance the ability of self care for preschoolers and to accomplish this goal, the topic of the program was decided as respiratory communicable disease prevention. The respiratory communicable disease prevention program consisted of texts, cartoons, photographs, discussions, demonstrations, puzzle games, die games, compensation/reinforcement, and token economy. This study was a quasi experimental study under the nonequivalent control group with pretest-posttest design. The subjects of this study were 45 preschool children who are attending 3 different district nursery schools and they were matched by the age, pretest knowledge, and pretest behavior.  The instrument used in this study was criterion referenced test items that were developed by a researcher for evaluating the subjects knowledge, attitude, and behavior about respiratory communicable disease prevention. A pretest was administered a week before treatment. Experimental group Ⅰ was administered by the treatment of respiratory communicable disease prevention program. Experimental group Ⅱ was administered by above program with token economy program. The posttest was conducted on the eighth day. The third test for behavior was completed 15th day. To determine the effect of the program, the data were analyzed by the SAS 6.12 program with Kruskal Wallis test, ANCOVA, ANOVA, Duncans test and paired t-test.  The results of this study were as follows:1. There was a significant difference in knowledge between the experimental groups and control group(F=5.89, P=0.0197).2. There was a significant difference in attitude between the experimental groups and control group(F=3.29, P=0.0469).3. There was a non-significant difference in behavior between the experimental groups and control group(F=0.00, P=0.9512).4. In the experimental groupⅡ, there was highly significant increase in behavior after token economy(t=4.5252, P=0.0005). In conclusion, it was found that the respiratory communicable disease prevention program for preschool children was effective in changing the preschoolers knowledge and attitude on the respiratory communicable disease prevention, but not enough for changing the preschoolers behavior.  Token economy was improved as an effective and strong method for inducing desirable changes of  preschoolers behavior.  Therefore, to develop health education programs for young subjects, various instruction techniques and advanced instructional media like animation, CD-ROM titles and video games etc. must be included, and to induce the change of childrens behavior, a well planned token economy program is suggested. This respiratory communicable disease prevention program is recommended to be widely used not only in kindergartens and nursery school,  but in clinical settings as well for young clients health promotion and disease prevention. 학령전기는 발달단계상 많은 결정적 시기를 내포하여 이 시기의 건강이 일생을 통해 누릴 건강의 토대가 된다. 아동은 자신의 건강 문제에 대하여 확인하고 이에 대해 스스로 대처하는데 어려움이 있으며, 면역계의 불완전한 발달과 외부와의 접촉 증가로 인하여 감염성 질환에 이환되기 쉽다.  학령전기 아동의 인지발달은 아직 미숙한 단계에 있지만 교육이 가능한 시기여서 일상생활과 연계된 교육을 통하여 건강의 증진과 질병의 예방이 가능하다. 학령전 아동은 자라면서 다양한 환경과 접하게 되지만 특히 눈에 보이는 구체적인 사실을 중심으로 추리하며 주의 집중 시간이 성인에 비하여 매우 짧기 때문에 교육내용에 대한 아동들의 이해를 돕기 위해서는 효과적인 교육 매체가  요구된다.  이에 본 연구자는 교육을 통한 학령전 아동의 자기간호능력을 증진시키기 위하여 이 시기에 가장 흔히 발생되는 호흡기 전염병에 대한 예방을 주제로 아동의 흥미를 자극할 수 있는 시각적 매체인 만화와 토의, 시범, 놀이, 보상, 강화와 조건강화로 이루어진 건강교육 프로그램을 개발하고 그 효과를 측정하고자 보육 시설의 아동을 대상으로 연구를 수행하였다. 본 연구의 설계는 비동등성 대조군 전후설계로 수행된 유사실험연구이다. 연구의 대상자는 서울 시내에 위치한 3개의 구립 어린이집에 다니는 5세와 6세의 아동으로 연령과 사전의 호흡기 전염병 예방에 대한 지식 점수 그리고 사전의 행위 점수로 짝짓기 한 45명이다. 실험군Ⅰ에게 제공된 프로그램은 인과관계의 개념이 부족한 학령전 아동을 위하여 호흡기 전염병의 원인인 병인요인, 과정인 환경요인 그리고 결과인 숙주요인에 대한 내용으로 구성된 텍스트와 텍스트에 대한 이해를 돕기 위한 만화, 교육 내용을 확장하고 응용하기 위한 토의, 행위의 정확성과 강도를 높이기 위한 시범, 교육 내용을 복습하기 위한 퍼즐 놀이와 주사위 놀이, 아동들의 성취행동을 보상하기 위한 강화로 구성되었으며 실험군Ⅱ에게는 토오컨 강화 프로그램이 추가로 제공되었다. 프로그램의 효과를 측정하기 위한 도구는 본 프로그램에서 개발된 준거지향검사문항(Criterion-referenced test items)으로 호흡기 전염병 예방에 대한 지식과 태도와 행위를 측정하도록 개발되었다. 자료수집은 실험군Ⅰ과 실험군Ⅱ를 대상으로는 교육 7일 전에 사전 측정을, 2일간의 교육 실시 후, 제 8일에 사후 측정을 통해 이루어졌으며 실험군 Ⅱ에게는 제 8일에 토오컨 강화가 적용되었고 제 15일에 실험군Ⅰ과 실험군Ⅱ를 대상으로 행위에 대한 측정이 이루어졌다. 한편 대조군에게는 사전 측정을 실시하고 제 8일에 사후 측정을 하여 수행되었다. 수집된 자료의 분석은 SAS 6.12를 이용하여 Kruskal Wallis test, ANCOVA, ANOVA, paired t-test로 실시되었다.  본 연구의 결과를 요약하면 다음과 같다.1. 호흡기 전염병 예방에 대한 지식과 태도에서 실험군Ⅰ과 실험군Ⅱ는 사전에 비해 사후에 유의한 변화를 보였으며(F=5.89, P=0.0197, F=3.29, P=0.0469)를 보였으나 행위에는 유의한 변화가 없었다(F=0.00, P=0.9512).2. 실험군Ⅱ는 실험군Ⅰ 보다 토오컨 강화 후에 매우 유의하게 행위가 변화되었다(t=4.5252, P=0.0005). 이상의 결과를 통하여 아동을 대상으로 한 건강 교육 프로그램은 아동의 인지발달 수준으로 이해 가능한 것이어야 하며 이해를 돕기 위하여 단순한 설명이상의 강력한 흥미 유발 도구가 요구되며, 아동의 사고의 확장과 표현 능력을 기르기 위하여 이야기 나누기가 효과적이었으며 구체적인 목표 행동이 있을 경우에는 반드시 시범을 포함시켜야 하며, 아동들을 위한 교육에서 놀이가 중요한 비중을 차지한다는 것도 확인되었다. 그러나 학령전 아동의 예방 행위를 변화시키는 데는 위와 같은 방법만으로는 부족했으며 토오컨 강화와 같은 보다 강력한 동기유발 도구가 필요하다는 것이 밝혀졌다.  그러므로 아동을 위한 건강 교육 프로그램을 개발할 때에는 건강교육에 대한 요구사정과 더불어 아동의 발달 수준을 고려한 적절한 교육 내용과 교수 매체 그리고 교육 방법을 선정하고 반드시 형성 평가를 실시하여 그 결과를 프로그램에 반영함으로써 보다 질 좋은 프로그램이 되도록 하여야겠다.  또한 학령전 아동을 위한 호흡기 전염병 예방을 프로그램은 대상자의 건강증진의 목적을 달성하기 위한 간호중재 방안의 일환으로 임상에서뿐만 아니라 일선 교육 현장에서 널리 유용하게 활용되도록 하여야겠다.
    원문보기
     따라서, 학생들이 과학(생물) 수업을 통해 전염병에 대한 올바른 과학적 소양을 함양할 수 있도록 실제 수업에서 질병 탐구활동을 포함하여 전염병 예방 교육과 연계된 지도가 이루어지고, 교육과정에서 해당 단원의 교육 내용과 활동이 개선될 필요가 있다. ‘독감(40.6%)’ 다음으로 ‘질병 관련 탐구활동이 없었다.’는 응답이 38.4%로 2번째로 높았다. 또한, 과학(생물) 수업에서 전염병과 관련된 내용을 배워야 할 필요성에 대해서는 72.4%의 학생들이 배울 필요성이 있다고 응답했다.언급되고 있는 중동호흡기증후군(Middle East Respiratory Syndrome,MERS), 인플루엔자(독감), 결핵, 식중독, 쯔쯔가무시증, 결막염의 6가지 전염병에 대하여 문항을 제작하였다. 문항은 지식과 실천 영역으로 나누어 지식 영역에는 원인 및 전파경로와 증상을 묻는 문항을, 실천 영역에는 예방 및 대처법을 묻는 문항으로 구성하였다. 그 외 전염병관련정보의 출처와 과학(생물)수업에서 알게 된 정보, 전염병 교육의 필요성에 대한 인식을 묻는 문항들을 추가하여 총 49문항을 제작하고 경남 지역의 일부 고등학생을 대상으로 설문 조사를 실시하였다. 
    원문보기
    목차검색조회
    음성듣기
    검색하신 키워드 전염병 (으)로 총 1,480 건의 학위논문이 검색되었습니다
    이 중에서 몇 건을 수집하시겠습니까?: 5
    5 건의 데이터를 수집하기 위해 1 페이지의 게시물을 조회합니다.
    ================================================================================
    1.번호: 1
    2.논문제목: 전염병 모델의 연대기적 분석과 후속모델에 대한 고찰
    3.저자: 이상원
    4.소속기관: 高麗大學校 大學院
    
    
    1.번호: 2
    2.논문제목: 전염병으로 인한 사회적 변화로서 제한적 일상 표현 연구 : COVID-19 팬데믹을 소재로 한 연구작을 중심으로
    3.저자: 박연경
    4.소속기관: 부산대학교
    
    
    1.번호: 3
    2.논문제목: 전염병과 펜데믹 상황에서 메디컬 처치의 운영에 관한 연구
    3.저자: 이재훈
    4.소속기관: 칼빈대학교 신학대학원
    
    
    1.번호: 4
    2.논문제목: 전염병 위험지각이 관광행동의도에 미치는 영향
    3.저자: 김혜진
    4.소속기관: 세종대학교 대학원
    
    
    1.번호: 5
    2.논문제목: 국제보건규칙 개정에 따른 우리나라 검역전염병 지정방안
    3.저자: 김윤호
    4.소속기관: 경북대학교 보건대학원
    
    
    요청하신 작업이 모두 완료되었습니다
    요청하신 데이터 수집 작업이 정상적으로 완료되었습니다
    

- 출력이 성공적으로 되었음을 확인할 수 있습니다.

## 4. 파일 확인

- 파일도 정상적으로 저장되었는지 확인해 보겠습니다.

### txt파일


```python
with open('Epidemic.txt','r',encoding='utf-8') as f:
    lines = f.readlines()
    for line in lines:
        print(line)
```

    
    
    1.번호:1
    
    2.논문제목:전염병 모델의 연대기적 분석과 후속모델에 대한 고찰
    
    3.저자:이상원
    
    4.소속기관:高麗大學校 大學院
    
    
    
    1.번호:2
    
    2.논문제목:전염병으로 인한 사회적 변화로서 제한적 일상 표현 연구 : COVID-19 팬데믹을 소재로 한 연구작을 중심으로
    
    3.저자:박연경
    
    4.소속기관:부산대학교
    
    
    
    1.번호:3
    
    2.논문제목:전염병과 펜데믹 상황에서 메디컬 처치의 운영에 관한 연구
    
    3.저자:이재훈
    
    4.소속기관:칼빈대학교 신학대학원
    
    
    
    1.번호:4
    
    2.논문제목:전염병 위험지각이 관광행동의도에 미치는 영향
    
    3.저자:김혜진
    
    4.소속기관:세종대학교 대학원
    
    
    
    1.번호:5
    
    2.논문제목:국제보건규칙 개정에 따른 우리나라 검역전염병 지정방안
    
    3.저자:김윤호
    
    4.소속기관:경북대학교 보건대학원
    
    

### csv 파일


```python
Epidemic = pd.read_csv('Epidemic.csv',encoding='utf-8')
Epidemic
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>번호</th>
      <th>제목</th>
      <th>저자</th>
      <th>소속(발행)기관</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>전염병 모델의 연대기적 분석과 후속모델에 대한 고찰</td>
      <td>이상원</td>
      <td>高麗大學校 大學院</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>전염병으로 인한 사회적 변화로서 제한적 일상 표현 연구 : COVID-19 팬데믹을...</td>
      <td>박연경</td>
      <td>부산대학교</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>전염병과 펜데믹 상황에서 메디컬 처치의 운영에 관한 연구</td>
      <td>이재훈</td>
      <td>칼빈대학교 신학대학원</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>전염병 위험지각이 관광행동의도에 미치는 영향</td>
      <td>김혜진</td>
      <td>세종대학교 대학원</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>국제보건규칙 개정에 따른 우리나라 검역전염병 지정방안</td>
      <td>김윤호</td>
      <td>경북대학교 보건대학원</td>
    </tr>
  </tbody>
</table>
</div>



### xls파일


```python
Epidemic = pd.read_excel('Epidemic.xls')
Epidemic
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>번호</th>
      <th>제목</th>
      <th>저자</th>
      <th>소속(발행)기관</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>전염병 모델의 연대기적 분석과 후속모델에 대한 고찰</td>
      <td>이상원</td>
      <td>高麗大學校 大學院</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>전염병으로 인한 사회적 변화로서 제한적 일상 표현 연구 : COVID-19 팬데믹을...</td>
      <td>박연경</td>
      <td>부산대학교</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>전염병과 펜데믹 상황에서 메디컬 처치의 운영에 관한 연구</td>
      <td>이재훈</td>
      <td>칼빈대학교 신학대학원</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>전염병 위험지각이 관광행동의도에 미치는 영향</td>
      <td>김혜진</td>
      <td>세종대학교 대학원</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>국제보건규칙 개정에 따른 우리나라 검역전염병 지정방안</td>
      <td>김윤호</td>
      <td>경북대학교 보건대학원</td>
    </tr>
  </tbody>
</table>
</div>



- 이상으로 모두 출력이 정상적으로 되어있음을 확인할 수 있습니다.

## 5. 실전에 적용해보기

- 앞서 실습을 활용하여 naver.com 에서 빅데이터 청년 캠퍼스를 검색한 결과를 크롤링해 보겠습니다.

- 순서
    1. 네이버에 빅데이터 청년 캠퍼스를 검색합니다.
    2. VIEW 항목으로 들어갑니다.
    3. 상위 15건의 블로그의 날짜, 작성자, 포스팅 제목, 요약 순으로 크롤링합니다.
    4. csv, xls 파일로 저장합니다.

![2022-07-06-0](https://user-images.githubusercontent.com/84655119/177511504-84b0a829-a16b-42f9-8412-8bcf57d0b8b8.png)

### 실습


```python
#Step 1. 필요한 모듈을 로딩합니다
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.chrome.service import Service
import time          

#Step 2. 빅데이터 청년 캠퍼스를 변수로 저장합니다.
print("=" *100)
print("빅데이터 청년 캠퍼스를 검색합니다.")
print("=" *100)
query_txt = '빅데이터 청년 캠퍼스'
print("\n")

fc_name = input('3.결과를 저장할 csv형식의 파일명을 쓰세요(예: c:\\py_temp\\riss.csv): ')
fx_name = input('4.결과를 저장할 xls형식의 파일명을 쓰세요(예: c:\\py_temp\\riss.xls): ')
print("\n")

#Step 3. 크롬 드라이버 설정 및 웹 페이지 열기
s = Service("c:/py_temp/chromedriver.exe")
driver = webdriver.Chrome(service=s)

url = 'https://www.naver.com/'
driver.get(url)
time.sleep(2)
driver.maximize_window()

#Step 4. 자동으로 검색어 입력 후 조회하기
element = driver.find_element(By.ID,'query') #naver의 검색칸의 ID
driver.find_element(By.ID,'query').click( )
element.send_keys(query_txt)
element.send_keys("\n")

time.sleep(2)
element = driver.find_element_by_xpath('//*[@id="lnb"]/div[1]/div/ul/li[2]/a').click() #VIEW의 xpath로 클릭

from bs4 import BeautifulSoup
html_1 = driver.page_source #현재 페이지의 전체 소스코드를 다 가져오기
soup_1 = BeautifulSoup(html_1, 'html.parser')
date = []
author = []
title = []
summary = []
first_div = soup_1.find_all("div","total_area")[:15] #상위 15건
for j in first_div:
    date.append(j.find("span","sub_time sub_txt").get_text()) #날짜 크롤링
    author.append(j.find("span","elss etc_dsc_inner").get_text()) #작성자 크롤링
    title.append(j.find("a","api_txt_lines total_tit _cross_trigger").get_text()) #제목 크롤링
    summary.append(j.find("div","api_txt_lines dsc_txt").get_text()) #요약 크롤링
    
df = pd.DataFrame()
df['date'] = pd.Series(date)
df['author'] = pd.Series(author)
df['title'] = pd.Series(title)
df['summary'] = pd.Series(summary)

# xls 형태로 저장하기
df.to_excel(fx_name,index=False, encoding="utf-8" , engine='openpyxl')

# csv 형태로 저장하기
df.to_csv(fc_name,index=False, encoding="utf-8-sig")

print('요청하신 데이터 수집 작업이 정상적으로 완료되었습니다')
```

    ====================================================================================================
    빅데이터 청년 캠퍼스를 검색합니다.
    ====================================================================================================
    
    
    3.결과를 저장할 csv형식의 파일명을 쓰세요(예: c:\py_temp\riss.csv): bigleader.csv
    4.결과를 저장할 xls형식의 파일명을 쓰세요(예: c:\py_temp\riss.xls): bigleader.xls
    
    
    

    C:\Users\HONGSU~1\AppData\Local\Temp/ipykernel_16452/794259087.py:35: DeprecationWarning: find_element_by_xpath is deprecated. Please use find_element(by=By.XPATH, value=xpath) instead
      element = driver.find_element_by_xpath('//*[@id="lnb"]/div[1]/div/ul/li[2]/a').click() #VIEW의 xpath로 클릭
    

    요청하신 데이터 수집 작업이 정상적으로 완료되었습니다
    

## 6. 저장한 실습 파일 확인


```python
bigleader = pd.read_csv('bigleader.csv',encoding='utf-8')
bigleader
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>author</th>
      <th>title</th>
      <th>summary</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2020.12.24.</td>
      <td>고려대학교 세종캠퍼스</td>
      <td>고려대학교 세종캠퍼스 ‘혁신성장 청년인재 집중양성사업’ 교육생 2020 뉴스 빅데이...</td>
      <td>뉴스 빅데이터 해커톤 대회 시상식이 열렸다. 정보통신기획평가원(IITP)의 예산지원...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2022.06.28.</td>
      <td>선정적인블로그</td>
      <td>데이터 청년 캠퍼스 연세대학교 빅데이터 분석처리 과정 서류합격 / 예비 5번 / 최...</td>
      <td>예에에에 데이터 청년 캠퍼스란?? 데이터 청년인재 양성 사업은 빅데이터 기술을 선도...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2020.06.18.</td>
      <td>요거트 아보카도</td>
      <td>2020 데이터 청년 캠퍼스 면접 후기 , 지원 꿀팁(빅데이터 청년 인재)</td>
      <td>+) 최종결과 최종합격이 되었답니다 ! http://bigjob.dbguide.ne...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2022.05.13.</td>
      <td>세종시</td>
      <td>세종시 데이터 청년 캠퍼스 빅데이터 청년인재 교육생 모집</td>
      <td>세종시와 홍익대학교 산학협력단이 협력하여 추진하는 데이터 청년 캠퍼스에 참여할 청년...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2020.09.07.</td>
      <td>박성준 강사(생애자원관리연구소)</td>
      <td>[박성준 강사] 삼성멀티캠퍼스_청년취업아카데미_인공지능 기반의 빅데이터 분석 시스템...</td>
      <td>[박성준 강사 - 생애자원관리연구소 교육] 8월 28일 삼성멀티캠퍼스에서 진행한 청...</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2022.05.31.</td>
      <td>소소늘보 지식복리 공간 :)</td>
      <td>청년취업사관학교 빅데이터 분석가 되어볼까</td>
      <td>느껴졌었는데 청년취업사관학교 파이썬 교육가ㅗ정과 후기들을 읽으니 나도 해볼 수 있을...</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2019.05.16.</td>
      <td>청년정책</td>
      <td>빅데이터 전문가로 가는 지름길(feat.공공 빅데이터 청년인턴십, 데이터 청년 캠퍼스)</td>
      <td>2019 데이터 청년 캠퍼스 빅데이터는 과학기술입니다. 그리고 우리나라에서 과학기술...</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2021.12.14.</td>
      <td>용인예술과학대학교 홍보단</td>
      <td>용인예술과학대학교 빅데이터경영과X경희대학교 빅데이터 연구센터 '데이터 청년 캠퍼스' 진행</td>
      <td>2021년 빅데이터 청년인재 양성사업에 선정되어 '데이터청년캠퍼스' 사업을 진행하였...</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2022.05.12.</td>
      <td>coq</td>
      <td>데이터 청년 캠퍼스로 스마트 인력 양성 한걸음</td>
      <td>맞잡고 '데이터 청년 캠퍼스'를 운영한다. 데이터 청년 캠퍼스는 한국데이터산업진흥원...</td>
    </tr>
    <tr>
      <th>9</th>
      <td>6시간 전</td>
      <td>아라공 Cafe - 무료교육 정보 / IT자격증 / 취업정보</td>
      <td>2022년 대학일자리플러스센터 사업 지역청년 특화 프로그램’디지털 미래인재 양성 프로그램</td>
      <td>지역청년 특화 프로그램’디지털 미래인재 양성 프로그램 전국 최고 수준의 빅데이터, ...</td>
    </tr>
    <tr>
      <th>10</th>
      <td>2021.06.28.</td>
      <td>Happily Ever After</td>
      <td>[데이터청년캠퍼스] 01 빅데이터개념 &amp; 판다스 시각화 &amp; SQL</td>
      <td>2021-06-28 9:30~17:30 &lt;빅데이터 개념 - 이원석 교수님&gt; ★ Di...</td>
    </tr>
    <tr>
      <th>11</th>
      <td>어제</td>
      <td>독취사-취업,대학생,대기업,공기업,GSAT,NCS,인턴,공모전,토익</td>
      <td>2022 대학 일자리플러스센터 사업 지역 청년 특화 프로그램 디지털 미래인재 양성 ...</td>
      <td>지역청년 특화 프로그램’디지털 미래인재 양성 프로그램 전국 최고 수준의 빅데이터, ...</td>
    </tr>
    <tr>
      <th>12</th>
      <td>6시간 전</td>
      <td>유니티 허브 (Unity Hub) - 개발자 커뮤니티</td>
      <td>2022년 대학일자리플러스센터 사업 지역청년 특화 프로그램’디지털 미래인재 양성 프로그램</td>
      <td>지역청년 특화 프로그램’디지털 미래인재 양성 프로그램 전국 최고 수준의 빅데이터, ...</td>
    </tr>
    <tr>
      <th>13</th>
      <td>2021.12.10.</td>
      <td>스펙업｜취업,대학생,인턴,대기업,공기업,GSAT,대외활동,공모전</td>
      <td>마감임박 [삼성 HR 전문 교육기관 멀티캠퍼스] K-Digital Training ...</td>
      <td>빅데이터 기반의 지능형 서비스 개발을 경험해보고 싶은 분 3. 데이터 분석 기초역량...</td>
    </tr>
    <tr>
      <th>14</th>
      <td>2021.07.18.</td>
      <td>김치야, 대학원 가자</td>
      <td>빅데이터 청년 캠퍼스 - 경기대!!</td>
      <td>작년 이 맘 때 광교 테크노 밸리에서 진행했던 데청캠 교육! 한 달 넘게 비오는 데...</td>
    </tr>
  </tbody>
</table>
</div>




```python
bigleader = pd.read_excel('bigleader.xls')
bigleader
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>author</th>
      <th>title</th>
      <th>summary</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2020.12.24.</td>
      <td>고려대학교 세종캠퍼스</td>
      <td>고려대학교 세종캠퍼스 ‘혁신성장 청년인재 집중양성사업’ 교육생 2020 뉴스 빅데이...</td>
      <td>뉴스 빅데이터 해커톤 대회 시상식이 열렸다. 정보통신기획평가원(IITP)의 예산지원...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2022.06.28.</td>
      <td>선정적인블로그</td>
      <td>데이터 청년 캠퍼스 연세대학교 빅데이터 분석처리 과정 서류합격 / 예비 5번 / 최...</td>
      <td>예에에에 데이터 청년 캠퍼스란?? 데이터 청년인재 양성 사업은 빅데이터 기술을 선도...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2020.06.18.</td>
      <td>요거트 아보카도</td>
      <td>2020 데이터 청년 캠퍼스 면접 후기 , 지원 꿀팁(빅데이터 청년 인재)</td>
      <td>+) 최종결과 최종합격이 되었답니다 ! http://bigjob.dbguide.ne...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2022.05.13.</td>
      <td>세종시</td>
      <td>세종시 데이터 청년 캠퍼스 빅데이터 청년인재 교육생 모집</td>
      <td>세종시와 홍익대학교 산학협력단이 협력하여 추진하는 데이터 청년 캠퍼스에 참여할 청년...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2020.09.07.</td>
      <td>박성준 강사(생애자원관리연구소)</td>
      <td>[박성준 강사] 삼성멀티캠퍼스_청년취업아카데미_인공지능 기반의 빅데이터 분석 시스템...</td>
      <td>[박성준 강사 - 생애자원관리연구소 교육] 8월 28일 삼성멀티캠퍼스에서 진행한 청...</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2022.05.31.</td>
      <td>소소늘보 지식복리 공간 :)</td>
      <td>청년취업사관학교 빅데이터 분석가 되어볼까</td>
      <td>느껴졌었는데 청년취업사관학교 파이썬 교육가ㅗ정과 후기들을 읽으니 나도 해볼 수 있을...</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2019.05.16.</td>
      <td>청년정책</td>
      <td>빅데이터 전문가로 가는 지름길(feat.공공 빅데이터 청년인턴십, 데이터 청년 캠퍼스)</td>
      <td>2019 데이터 청년 캠퍼스 빅데이터는 과학기술입니다. 그리고 우리나라에서 과학기술...</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2021.12.14.</td>
      <td>용인예술과학대학교 홍보단</td>
      <td>용인예술과학대학교 빅데이터경영과X경희대학교 빅데이터 연구센터 '데이터 청년 캠퍼스' 진행</td>
      <td>2021년 빅데이터 청년인재 양성사업에 선정되어 '데이터청년캠퍼스' 사업을 진행하였...</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2022.05.12.</td>
      <td>coq</td>
      <td>데이터 청년 캠퍼스로 스마트 인력 양성 한걸음</td>
      <td>맞잡고 '데이터 청년 캠퍼스'를 운영한다. 데이터 청년 캠퍼스는 한국데이</td>
    </tr>
    <tr>
      <th>9</th>
      <td>6시간 전</td>
      <td>아라공 Cafe - 무료교육 정보 / IT자격증 / 취업정보</td>
      <td>2022년 대학일자리플러스센터 사업 지역청년 특화 프로그램’디지털 미래인재 양성 프로그램</td>
      <td>지역청년 특화 프로그램’디지털 미래인재 양성 프로그램 전국 최고 수준의 빅데이터, ...</td>
    </tr>
    <tr>
      <th>10</th>
      <td>2021.06.28.</td>
      <td>Happily Ever After</td>
      <td>[데이터청년캠퍼스] 01 빅데이터개념 &amp; 판다스 시각화 &amp; SQL</td>
      <td>2021-06-28 9:30~17:30 &lt;빅데이터 개념 - 이원석 교수님&gt; ★ Di...</td>
    </tr>
    <tr>
      <th>11</th>
      <td>어제</td>
      <td>독취사-취업,대학생,대기업,공기업,GSAT,NCS,인턴,공모전,토익</td>
      <td>2022 대학 일자리플러스센터 사업 지역 청년 특화 프로그램 디지털 미래인재 양성 ...</td>
      <td>지역청년 특화 프로그램’디지털 미래인재 양성 프로그램 전국 최고 수준의 빅데이터, ...</td>
    </tr>
    <tr>
      <th>12</th>
      <td>6시간 전</td>
      <td>유니티 허브 (Unity Hub) - 개발자 커뮤니티</td>
      <td>2022년 대학일자리플러스센터 사업 지역청년 특화 프로그램’디지털 미래인재 양성 프로그램</td>
      <td>지역청년 특화 프로그램’디지털 미래인재 양성 프로그램 전국 최고 수준의 빅데이터, ...</td>
    </tr>
    <tr>
      <th>13</th>
      <td>2021.12.10.</td>
      <td>스펙업｜취업,대학생,인턴,대기업,공기업,GSAT,대외활동,공모전</td>
      <td>마감임박 [삼성 HR 전문 교육기관 멀티캠퍼스] K-Digital Training ...</td>
      <td>빅데이터 기반의 지능형 서비스 개발을 경험해보고 싶은 분 3. 데이터 분석 기초역량...</td>
    </tr>
    <tr>
      <th>14</th>
      <td>2021.07.18.</td>
      <td>김치야, 대학원 가자</td>
      <td>빅데이터 청년 캠퍼스 - 경기대!!</td>
      <td>작년 이 맘 때 광교 테크노 밸리에서 진행했던 데청캠 교육! 한 달 넘게 비오는 데...</td>
    </tr>
  </tbody>
</table>
</div>



성공적으로 저장이 되었음을 확인할 수 있습니다!
