# Chap 5. 다양한   댓글    및   리뷰    정보   수집하기

- 이번 시간에 예제로 사용할 내용은 네이버 영화 리뷰에 작성된 댓글을 수집하는 것입니다.

![image-2.png](attachment:image-2.png)

- 우리가 수집할 내용을 정리하면 아래와 같습니다.
    1. 별점
    2. 작성자ID
    3. 리뷰 내용
    4. 작성 일자
    5. 공감 횟수
    6. 비공감 횟수

## Step 1. 필요한 모듈과 라이브러리를 로딩합니다.


```python
from bs4 import BeautifulSoup
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.chrome.service import Service
import time
import sys
import math
import numpy  
import pandas as pd 
import os
```

## Step 2. 크롤링할 영화 제목과 리뷰 건수를 변수로 저장합니다.


```python
print("=" *80)
print("연습문제 :네이버 영화 리뷰 정보 수집하기 ")
print("=" *80)

query_txt = input('1.크롤링 할 영화의 제목을 입력하세요: ')
query_url = 'https://movie.naver.com'

cnt = int(input('2.크롤링 할 리뷰건수는 몇건입니까?: '))
page_cnt = math.ceil(cnt / 10)
    
print("-" *80)
print("요청하신 데이터를 수집 중이오니 잠시만 기다려 주세요~~~~")
```

## Step 3. 크롬 드라이버를 사용해서 웹 브라우저를 실행합니다.


```python
s = Service("c:/py_temp/chromedriver.exe")
driver = webdriver.Chrome(service=s)

driver.get(query_url)
driver.maximize_window()
time.sleep(2)

# 영화 제목으로 검색
element = driver.find_element(By.ID,"ipt_tx_srch")
element.send_keys(query_txt)
driver.find_element(By.XPATH,'//*[@id="jSearchArea"]/div/button').click()
driver.find_element(By.XPATH,'//*[@id="old_content"]/ul[2]/li/dl/dt/a').click()
```

## Step 4 평점 옵션 버튼 클릭


```python
driver.find_element(By.LINK_TEXT,"평점").click()
```

## Step 5. 현재 페이지 리뷰와 점수 등 내용 수집


```python
# IFRAME 변경
driver.switch_to.frame('pointAfterListIframe')

html = driver.page_source
soup = BeautifulSoup(html, 'html.parser')

#현재 총 리뷰 건수를 확인하여 사용자의 요청건수와 비교 후 동기화
result= soup.find('div', class_='score_total').find('strong','total').find('em').get_text()

print("=" *80)
search_cnt = int(result.replace(",",""))

if cnt > search_cnt :
    cnt = search_cnt

print("전체 검색 결과 건수 :",search_cnt,"건")
print("실제 최종 출력 건수",cnt)

print("\n")
page_cnt = math.ceil(cnt / 10)
print("크롤링 할 총 페이지 번호: ",page_cnt)
print("=" *80)

score2=[]
review2=[]
writer2=[]
wdate2=[]
#gogam2=[]
g_gogam2=[]
b_gogam2=[]
#dwlist2=[]

count = 0         # 크롤링 건수 카운트 변수
click_count = 1   # 페이지 번호

while ( click_count <= page_cnt )  :
            
    html = driver.page_source
    soup = BeautifulSoup(html, 'html.parser')

    score_result = soup.find('div', class_='score_result').find('ul')
    slist = score_result.find_all('li')

    for li in slist:

        count += 1

        print("\n")
        print("총 %s 건 중 %s 번째 리뷰 데이터를 수집합니다==============" %(cnt,count))
        score = li.find('div', class_='star_score').find('em').get_text()
        print("1.별점:","*"*int(score),": ", score)
        score2.append(score)

        review = li.find('div', class_='score_reple').find('p').get_text().replace('\n','').strip()
        print("2.리뷰내용:",review)
        review2.append(review)

        dwlist = li.find('div',class_='score_reple').find_all('em')
        writer = dwlist[0].find('span').get_text()
        print("3.작성자:",writer)
        writer2.append(writer)

        wdate = dwlist[1].text
        print('4.작성일자:',wdate)
        wdate2.append(wdate)

        gogam = li.find('div', class_='btn_area').find_all('strong')
        g_gogam = gogam[0].text
        print('5.공감:',g_gogam)
        g_gogam2.append(g_gogam)

        b_gogam = gogam[1].text
        print('6.비공감:', b_gogam)
        b_gogam2.append(b_gogam)

        if count == cnt :
            break

    time.sleep(1)  

    click_count += 1

    if click_count > page_cnt :
        break
    else :
        driver.find_element(By.LINK_TEXT,"%s" %click_count).click()

    time.sleep(1)
```

여기서 가장 중요한건 가장 윗줄의 iframe 변경입니다. default iframe이 리뷰없는 곳에 설정이 되어있다면 내가 크롤링하고자 하는 부분으로 iframe을 변경해줘야 합니다. iframe 변경시 함수인자로는 iframe의 id값을 사용합니다.

## Step 6. xls 형태와 csv 형태로 저장하기


```python
naver_movie = pd.DataFrame()
naver_movie['별점(평점)']=score2
naver_movie['리뷰내용']=review2
naver_movie['작성자']=writer2
naver_movie['작성일자']=wdate2
naver_movie['공감횟수']=g_gogam2
naver_movie['비공감횟수']=b_gogam2

# csv 형태로 저장하기
naver_movie.to_csv(f'{query_txt}평점리뷰.csv',encoding="utf-8-sig",index=True)

# 엑셀 형태로 저장하기
naver_movie.to_excel(f'{query_txt}평점리뷰.xls' ,index=True,engine='openpyxl')
```

# 실제로 적용해보기

## 1. 코드작성


```python
from bs4 import BeautifulSoup
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.chrome.service import Service

import time
import sys
import math
import numpy  
import pandas as pd 
import os

print("=" *80)
print("연습문제 :네이버 영화 리뷰 정보 수집하기 ")
print("=" *80)

query_txt = input('1.크롤링 할 영화의 제목을 입력하세요: ')
query_url = 'https://movie.naver.com'

cnt = int(input('2.크롤링 할 리뷰건수는 몇건입니까?: '))
page_cnt = math.ceil(cnt / 10)
    
print("-" *80)
print("요청하신 데이터를 수집 중이오니 잠시만 기다려 주세요~~~~")

s = Service("c:/py_temp/chromedriver.exe")
driver = webdriver.Chrome(service=s)

driver.get(query_url)
driver.maximize_window()
time.sleep(2)

# 영화 제목으로 검색
element = driver.find_element(By.ID,"ipt_tx_srch")
element.send_keys(query_txt)
driver.find_element(By.XPATH,'//*[@id="jSearchArea"]/div/button').click()
driver.find_element(By.XPATH,'//*[@id="old_content"]/ul[2]/li/dl/dt/a').click()

driver.find_element(By.LINK_TEXT,"평점").click()

# IFRAME 변경
driver.switch_to.frame('pointAfterListIframe')

html = driver.page_source
soup = BeautifulSoup(html, 'html.parser')

#현재 총 리뷰 건수를 확인하여 사용자의 요청건수와 비교 후 동기화
result= soup.find('div', class_='score_total').find('strong','total').find('em').get_text()

print("=" *80)
search_cnt = int(result.replace(",",""))

if cnt > search_cnt :
    cnt = search_cnt

print("전체 검색 결과 건수 :",search_cnt,"건")
print("실제 최종 출력 건수",cnt)

print("\n")
page_cnt = math.ceil(cnt / 10)
print("크롤링 할 총 페이지 번호: ",page_cnt)
print("=" *80)

score2=[]
review2=[]
writer2=[]
wdate2=[]
#gogam2=[]
g_gogam2=[]
b_gogam2=[]
#dwlist2=[]

count = 0         # 크롤링 건수 카운트 변수
click_count = 1   # 페이지 번호

while ( click_count <= page_cnt )  :
            
    html = driver.page_source
    soup = BeautifulSoup(html, 'html.parser')

    score_result = soup.find('div', class_='score_result').find('ul')
    slist = score_result.find_all('li')

    for li in slist:

        count += 1

        print("\n")
        print("총 %s 건 중 %s 번째 리뷰 데이터를 수집합니다==============" %(cnt,count))
        score = li.find('div', class_='star_score').find('em').get_text()
        print("1.별점:","*"*int(score),": ", score)
        score2.append(score)

        review = li.find('div', class_='score_reple').find('p').get_text().replace('\n','').strip()
        print("2.리뷰내용:",review)
        if '관람객' in review:    # 관람객 평점일 시 관람객을 제거해주는 용도
            review = review.replace('\t',"")
            review = review.replace('관람객',"")
        review2.append(review)

        dwlist = li.find('div',class_='score_reple').find_all('em')
        writer = dwlist[0].find('span').get_text()
        print("3.작성자:",writer)
        writer2.append(writer)

        wdate = dwlist[1].text
        print('4.작성일자:',wdate)
        wdate2.append(wdate)

        gogam = li.find('div', class_='btn_area').find_all('strong')
        g_gogam = gogam[0].text
        print('5.공감:',g_gogam)
        g_gogam2.append(g_gogam)

        b_gogam = gogam[1].text
        print('6.비공감:', b_gogam)
        b_gogam2.append(b_gogam)

        if count == cnt :
            break

    time.sleep(1)  

    click_count += 1

    if click_count > page_cnt :
        break
    else :
        driver.find_element(By.LINK_TEXT,"%s" %click_count).click()

    time.sleep(1)

naver_movie = pd.DataFrame()
naver_movie['별점(평점)']=score2
naver_movie['리뷰내용']=review2
naver_movie['작성자']=writer2
naver_movie['작성일자']=wdate2
naver_movie['공감횟수']=g_gogam2
naver_movie['비공감횟수']=b_gogam2

# csv 형태로 저장하기
naver_movie.to_csv(f'{query_txt}평점리뷰.csv',encoding="utf-8-sig",index=True)

# 엑셀 형태로 저장하기
naver_movie.to_excel(f'{query_txt}평점리뷰.xls' ,index=True,engine='openpyxl')
```

    ================================================================================
    연습문제 :네이버 영화 리뷰 정보 수집하기 
    ================================================================================
    1.크롤링 할 영화의 제목을 입력하세요: 범죄도시2
    2.크롤링 할 리뷰건수는 몇건입니까?: 20
    --------------------------------------------------------------------------------
    요청하신 데이터를 수집 중이오니 잠시만 기다려 주세요~~~~
    ================================================================================
    전체 검색 결과 건수 : 25609 건
    실제 최종 출력 건수 20
    
    
    크롤링 할 총 페이지 번호:  2
    ================================================================================
    
    
    총 20 건 중 1 번째 리뷰 데이터를 수집합니다==============
    1.별점: ********** :  10
    2.리뷰내용: 관람객																		이 영화의 속편은 100% 성공이다. 원래 한국영화들 속편은 잘 안되고 속편 나오면 망하는 케이스던데 이번 범죄도시2는 속편 성공함 오랜만에 극장에서 다같이 웃으면서 봄 장이수 개웃기고 ㅋㅋㅋ 이번에도 명장면 + 명...
    3.작성자: Abc123(fowe****)
    4.작성일자: 2022.05.18 09:14
    5.공감: 5280
    6.비공감: 258
    
    
    총 20 건 중 2 번째 리뷰 데이터를 수집합니다==============
    1.별점: ********** :  10
    2.리뷰내용: 관람객																																																												시리즈로 계속 나왔으면 좋겠다. 마동석한테 최적화된 작품이다.
    3.작성자: 돼지고싶냐(viry****)
    4.작성일자: 2022.05.18 10:31
    5.공감: 4167
    6.비공감: 123
    
    
    총 20 건 중 3 번째 리뷰 데이터를 수집합니다==============
    1.별점: ******** :  8
    2.리뷰내용: 관람객																																																												니가 강해상이냐? 아뇨. 구씬데요?
    3.작성자: lllllllllllllllllll(kimh****)
    4.작성일자: 2022.05.18 10:06
    5.공감: 4255
    6.비공감: 459
    
    
    총 20 건 중 4 번째 리뷰 데이터를 수집합니다==============
    1.별점: ********** :  10
    2.리뷰내용: 관람객																																																												전편을 보고 가야 장이수 얼굴만 봐도 웃음이 나오는 이유를 알 수 있습니다:)
    3.작성자: 코코몽(jinw****)
    4.작성일자: 2022.05.18 10:17
    5.공감: 3329
    6.비공감: 97
    
    
    총 20 건 중 5 번째 리뷰 데이터를 수집합니다==============
    1.별점: ********* :  9
    2.리뷰내용: 관람객																																																												와…. 손석구 연기 진짜 미쳣다…오바하는 범죄자연기가 아니라 ㄹㅇ 범죄자같음
    3.작성자: 뿌이잉뽀이잉(dldk****)
    4.작성일자: 2022.05.18 23:51
    5.공감: 3135
    6.비공감: 91
    
    
    총 20 건 중 6 번째 리뷰 데이터를 수집합니다==============
    1.별점: ********** :  10
    2.리뷰내용: 관람객																																																												액션과 유머 적절한 5:5였다. 꼭 사운드 좋은 곳에서 보길
    3.작성자: ㅇㅅㅇ2(ansi****)
    4.작성일자: 2022.05.18 10:04
    5.공감: 2818
    6.비공감: 103
    
    
    총 20 건 중 7 번째 리뷰 데이터를 수집합니다==============
    1.별점: ********** :  10
    2.리뷰내용: 관람객																																																												장첸따위는 잊어라 구씨가 온다!!
    3.작성자: dlkl****
    4.작성일자: 2022.05.18 13:56
    5.공감: 2601
    6.비공감: 445
    
    
    총 20 건 중 8 번째 리뷰 데이터를 수집합니다==============
    1.별점: ********** :  10
    2.리뷰내용: 관람객																																																												잼써잼써ㅠ 마블리 펀치 사운드 장난 아니니까 꼭 극장에서 보세요ㅋㅋ
    3.작성자: shiizn16(ksbb****)
    4.작성일자: 2022.05.18 09:49
    5.공감: 1941
    6.비공감: 67
    
    
    총 20 건 중 9 번째 리뷰 데이터를 수집합니다==============
    1.별점: ********** :  10
    2.리뷰내용: 관람객																																																												통쾌함을 마블에서 못보고 여기서 보네요. 스토리도 깔끔해서 몰입 하나도 안끊깁니다웃긴 요소가 기대벽이 있었음에도 꽤 있어요 개인적으로 조금 더 줘패고 싶었네요
    3.작성자: 프로필(leew****)
    4.작성일자: 2022.05.18 11:59
    5.공감: 1691
    6.비공감: 82
    
    
    총 20 건 중 10 번째 리뷰 데이터를 수집합니다==============
    1.별점: ******** :  8
    2.리뷰내용: 관람객																																																												이건 크게 잘될만하다.
    3.작성자: 점심시간(pigp****)
    4.작성일자: 2022.05.18 09:12
    5.공감: 1436
    6.비공감: 88
    
    
    총 20 건 중 11 번째 리뷰 데이터를 수집합니다==============
    1.별점: ********** :  10
    2.리뷰내용: 관람객																																																												진짜 쩐다. 특히 손석구 분노 연기 부들부들 떠는 게 디지고. 오랜만에 한국 영화 제대로 봤다. 천만 거뜬할 듯
    3.작성자: char****
    4.작성일자: 2022.05.18 19:47
    5.공감: 1136
    6.비공감: 20
    
    
    총 20 건 중 12 번째 리뷰 데이터를 수집합니다==============
    1.별점: ********* :  9
    2.리뷰내용: 관람객																																																												1편 윤계상 악역 하는거 보고 역대급 이라 생각했는데 손석구가 그걸 넘어선 느낌;;ㄷㄷ
    3.작성자: DYNAMODE(dw50****)
    4.작성일자: 2022.05.18 15:23
    5.공감: 1155
    6.비공감: 112
    
    
    총 20 건 중 13 번째 리뷰 데이터를 수집합니다==============
    1.별점: ********** :  10
    2.리뷰내용: 관람객																																																												1편이 꽤나 수작이라 내심 걱정스러운 마음으로 봤는데 너무 재미져요 2!.3편도 기대!
    3.작성자: 임(disc****)
    4.작성일자: 2022.05.18 09:44
    5.공감: 992
    6.비공감: 52
    
    
    총 20 건 중 14 번째 리뷰 데이터를 수집합니다==============
    1.별점: ********** :  10
    2.리뷰내용: 관람객																																																												니 내 누군지 아늬? 나 하얼빈에 장첸이야
    3.작성자: 니드(nico****)
    4.작성일자: 2022.05.18 17:29
    5.공감: 972
    6.비공감: 36
    
    
    총 20 건 중 15 번째 리뷰 데이터를 수집합니다==============
    1.별점: ********** :  10
    2.리뷰내용: 관람객																																																												솔직하게 장첸보다 강해상이 더 강해보인다
    3.작성자: 르에(naka****)
    4.작성일자: 2022.05.18 19:19
    5.공감: 1043
    6.비공감: 129
    
    
    총 20 건 중 16 번째 리뷰 데이터를 수집합니다==============
    1.별점: ********** :  10
    2.리뷰내용: 관람객																																																												아.. 범죄자가 그렇게 섹시하면 어떡해요. 반해버렸네
    3.작성자: white americano(tlgh****)
    4.작성일자: 2022.05.18 21:03
    5.공감: 906
    6.비공감: 59
    
    
    총 20 건 중 17 번째 리뷰 데이터를 수집합니다==============
    1.별점: ********** :  10
    2.리뷰내용: 관람객																																																												와 한국영화의 속편 중에 처음으로 실망 안했다..
    3.작성자: heav****
    4.작성일자: 2022.05.18 12:35
    5.공감: 825
    6.비공감: 23
    
    
    총 20 건 중 18 번째 리뷰 데이터를 수집합니다==============
    1.별점: ********** :  10
    2.리뷰내용: 관람객																																																												누가 5야 ? 저도 순간 고민했습니다
    3.작성자: 막내꼬마(yulr****)
    4.작성일자: 2022.05.19 00:35
    5.공감: 795
    6.비공감: 32
    
    
    총 20 건 중 19 번째 리뷰 데이터를 수집합니다==============
    1.별점: ******* :  7
    2.리뷰내용: 관람객																																																												구씨가 산포시로 도망친 이유
    3.작성자: 동동(ehdt****)
    4.작성일자: 2022.05.18 18:19
    5.공감: 796
    6.비공감: 37
    
    
    총 20 건 중 20 번째 리뷰 데이터를 수집합니다==============
    1.별점: ********** :  10
    2.리뷰내용: 관람객																		 1보다 재밌는게 말이되나여??진심 ㅋㅋㅋㅋ명장면 많고 웃긴장면도 많곸ㅋㅋㅋ감탄해씀 누구의 머릿속에서 나온아이디어인짘ㅋㅋ특히 에스컬레이터 씬ㅋㅋㅋㅋ그냥 보세여!!밑에 누가 킬링타임용이라고 적었던데 장난하나요..긴장감...
    3.작성자: 청청(dudd****)
    4.작성일자: 2022.05.19 00:16
    5.공감: 715
    6.비공감: 43
    

최근에 나온 범죄도시2의 리뷰로 크롤링 하였습니다.

## 2. 저장된 파일 불러오기


```python
df_csv = pd.read_csv(f'{query_txt}평점리뷰.csv',encoding='utf-8')
df_csv.head()
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
      <th>Unnamed: 0</th>
      <th>별점(평점)</th>
      <th>리뷰내용</th>
      <th>작성자</th>
      <th>작성일자</th>
      <th>공감횟수</th>
      <th>비공감횟수</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>10</td>
      <td>이 영화의 속편은 100% 성공이다. 원래 한국영화들 속편은 잘 안되고 속편 나오면...</td>
      <td>Abc123(fowe****)</td>
      <td>2022.05.18 09:14</td>
      <td>5280</td>
      <td>258</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>10</td>
      <td>시리즈로 계속 나왔으면 좋겠다. 마동석한테 최적화된 작품이다.</td>
      <td>돼지고싶냐(viry****)</td>
      <td>2022.05.18 10:31</td>
      <td>4167</td>
      <td>123</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>8</td>
      <td>니가 강해상이냐? 아뇨. 구씬데요?</td>
      <td>lllllllllllllllllll(kimh****)</td>
      <td>2022.05.18 10:06</td>
      <td>4255</td>
      <td>459</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>10</td>
      <td>전편을 보고 가야 장이수 얼굴만 봐도 웃음이 나오는 이유를 알 수 있습니다:)</td>
      <td>코코몽(jinw****)</td>
      <td>2022.05.18 10:17</td>
      <td>3329</td>
      <td>97</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>9</td>
      <td>와…. 손석구 연기 진짜 미쳣다…오바하는 범죄자연기가 아니라 ㄹㅇ 범죄자같음</td>
      <td>뿌이잉뽀이잉(dldk****)</td>
      <td>2022.05.18 23:51</td>
      <td>3135</td>
      <td>91</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_xls = pd.read_excel(f'{query_txt}평점리뷰.xls')
df_xls.head()
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
      <th>Unnamed: 0</th>
      <th>별점(평점)</th>
      <th>리뷰내용</th>
      <th>작성자</th>
      <th>작성일자</th>
      <th>공감횟수</th>
      <th>비공감횟수</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>10</td>
      <td>이 영화의 속편은 100% 성공이다. 원래 한국영화들 속편은 잘 안되고 속편 나오면...</td>
      <td>Abc123(fowe****)</td>
      <td>2022.05.18 09:14</td>
      <td>5280</td>
      <td>258</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>10</td>
      <td>시리즈로 계속 나왔으면 좋겠다. 마동석한테 최적화된 작품이다.</td>
      <td>돼지고싶냐(viry****)</td>
      <td>2022.05.18 10:31</td>
      <td>4167</td>
      <td>123</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>8</td>
      <td>니가 강해상이냐? 아뇨. 구씬데요?</td>
      <td>lllllllllllllllllll(kimh****)</td>
      <td>2022.05.18 10:06</td>
      <td>4255</td>
      <td>459</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>10</td>
      <td>전편을 보고 가야 장이수 얼굴만 봐도 웃음이 나오는 이유를 알 수 있습니다:)</td>
      <td>코코몽(jinw****)</td>
      <td>2022.05.18 10:17</td>
      <td>3329</td>
      <td>97</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>9</td>
      <td>와…. 손석구 연기 진짜 미쳣다…오바하는 범죄자연기가 아니라 ㄹㅇ 범죄자같음</td>
      <td>뿌이잉뽀이잉(dldk****)</td>
      <td>2022.05.18 23:51</td>
      <td>3135</td>
      <td>91</td>
    </tr>
  </tbody>
</table>
</div>



다양한 리뷰를 성공적으로 크롤링하였습니다!
