# Chap 6. 다양한 이미지 수집 크롤러 만들기

## 1. 이번    장에서    배울   내용    소개

이미지를 수집하기 위해서는 웹 페이지에서 원하는 이미지의 URL 주소를 추출한 후 웹서버에서 이미지를 다운로드 받는데 이 때 두 가지 경우가 있습니다.
1. 이미지가 저장된 서버에서 이미지를 가져올 때 인증 정보 없이 그냥 가져올 수 있는 경우
2. 인증 정보를 보낸 후 이미지를 가져올 수 있는 경우

여기에 적절한 예제인 휴넷 홈페이지와 pixabay 홈페이지에서 이미지를 크롤링하는 코드를 보면서 공부해보도록 하겠습니다.

## 2. 코드 리뷰 - Case 1. 인증 정보 없이 가져올 수 있는 경우 (휴넷)

### Step 1. 필요한 모듈과 라이브러리를 로딩하고 검색어를 입력 받습니다


```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.chrome.service import Service
from bs4 import BeautifulSoup
import urllib.request
import urllib.parse
import time
import math
import os
import random
```

### Step 2. 사용자에게 검색 관련 정보들을 입력 받습니다.


```python
print("=" *100)
print(" 이 크롤러는 휴넷 사이트의 강의 자료 수집용 웹크롤러입니다.")
print("=" *100)
query_txt = input('1.수집할 자료의 키워드는 무엇입니까?(예: 파이썬): ')

try :
    cnt = int( input('2.수집할 건수는 총 몇건입니까?(기본값:10): '))
except ValueError :
    cnt = 10
    print('기본값인 10 건으로 수집을 진행합니다.')
page_cnt = math.ceil( cnt / 12)

f_dir=input('3.파일이 저장될 경로만 쓰세요(예: c:\\py_temp\\ ) : ')
if f_dir =='' :
    f_dir = "c:\\py_temp\\"
```

### Step 3. 크롬 드라이버 설정 및 웹 페이지 열기


```python
s = Service("c:/py_temp/chromedriver.exe")
driver = webdriver.Chrome(service=s)

url = 'https://www.hunet.co.kr/'
driver.get(url)
driver.maximize_window()
time.sleep(3)
```

### Step 4. 자동으로 검색어 입력 후 조회하기


```python
element = driver.find_element(By.ID,'txtKeyword')
driver.find_element(By.ID,'txtKeyword').click( )
element.send_keys(query_txt)
element.send_keys("\n")
```

### Step 5. 사용자가 요청한 건수만큼 더보기 클릭하기


```python
for a in range(0,page_cnt) :
    try :
        driver.find_element(By.XPATH,'//*[@id="divEducationList"]/div/div[2]/div[5]/a[1]').click() #펼쳐보기 클릭
        time.sleep(2)
        try :
            result = driver.switch_to_alert() #alert창이 떳을 경우 확인 누르기
            result.accept( )
        except :
            continue
    except :
        print('페이지 이동이 끝났습니다. 이제 데이터를 수집하겠습니다')
        break
```

휴넷 사이트의 경우 아래 그림과 같이 한 페이지에 12개의 그림이 있고 다음 페이지로 가고 싶을 경우 아래의 **펼쳐보기** 버튼을 클릭해야 합니다.

![image.png](attachment:image.png)

또한 크롤링의 정보가 사용자가 원하는 정보 수 보다 작을 경우 아래와 같이 alert창이 뜨는데 driver.switch_to_alert()로 창을 변환 후 확인을 누르는 작업입니다.

![image.png](attachment:image.png)

### Step 6. 이미지 추출하여 저장하기 


```python
file_no = 0                                
count = 1
img_src2=[]  # 이미지 원본 URL 주소 저장할 리스트

n = time.localtime()
s = '%04d-%02d-%02d-%02d-%02d-%02d' % (n.tm_year, n.tm_mon, n.tm_mday, n.tm_hour, n.tm_min, n.tm_sec)
img_dir = f_dir+s+'-'+query_txt
os.makedirs(img_dir)
os.chdir(img_dir)

html = driver.page_source
soup = BeautifulSoup(html, 'html.parser')
img_src = soup.find('ul','vod_list').find_all('img')

for i in img_src :
    img_src1=i['src']
    img_src2.append(img_src1)
    print(img_src1)
    count += 1
    if count > cnt :
        break
```

폴더도 잘 만들고 src로 img url도 잘 긁어왔어도 에러가 생길 수 있습니다. 아래의 사진을 보겠습니다.

![image.png](attachment:image.png)

위 그림의 첫번째 줄과 마지막 줄처럼 이미지 주소에서 한글이 있는 부분 보이죠?
이럴 경우 urllib.request.urlretrieve( ) 함수가 한글을 제대로 인식을 못해서 오류가 발생합니다. 그래서 예외처리를 사용하여 건너뛰거나 인코딩을 지정하는 방법으로 이미지 이름을 인식시켜서 다운로드를 받아야 하는데 이 부분의 코드가 아래와 같습니다.


```python
for i in range(0,len(img_src2)) :
    file_no += 1 
    #이미지 파일에 한글 이름이 들어갈 경우 오류가 발생하므로 인코딩 지정해 주어야 함
    urllib.request.urlretrieve(urllib.parse.quote(img_src2[i].encode('utf8'), '/:'),str(file_no)+'.jpg')
    time.sleep(0.5)      
    print("%s 번째 이미지 저장중입니다=======" %file_no)
```

### Step 7. 요약 정보를 출력합니다    


```python
print("=" *70)
print("총 저장 건수는 %s 건 입니다 " %file_no)
print("파일 저장 경로: %s 입니다" %img_dir)
print("=" *70)

driver.close( )
```

### 전체 코드 실행(Case1)


```python
#17. 이미지 다운로드용 웹크롤러 만들기
# Step 1. 필요한 모듈과 라이브러리를 로딩하고 검색어를 입력 받습니다
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.chrome.service import Service
from bs4 import BeautifulSoup
import urllib.request
import urllib.parse
import time
import math
import os
import random

#Step 2. 사용자에게 검색 관련 정보들을 입력 받습니다.
print("=" *100)
print(" 이 크롤러는 휴넷 사이트의 강의 자료 수집용 웹크롤러입니다.")
print("=" *100)
query_txt = input('1.수집할 자료의 키워드는 무엇입니까?(예: 파이썬): ')

try :
    cnt = int( input('2.수집할 건수는 총 몇건입니까?(기본값:10): '))
except ValueError :
    cnt = 10
    print('기본값인 10 건으로 수집을 진행합니다.')
page_cnt = math.ceil( cnt / 12)

f_dir=input('3.파일이 저장될 경로만 쓰세요(예: c:\\py_temp\\ ) : ')
if f_dir =='' :
    f_dir = "c:\\py_temp\\"

#Step 3. 크롬 드라이버 설정 및 웹 페이지 열기
s = Service("c:/py_temp/chromedriver.exe")
driver = webdriver.Chrome(service=s)

url = 'https://www.hunet.co.kr/'
driver.get(url)
driver.maximize_window()
time.sleep(3)

#Step 4. 자동으로 검색어 입력 후 조회하기
element = driver.find_element(By.ID,'txtKeyword')
driver.find_element(By.ID,'txtKeyword').click( )
element.send_keys(query_txt)
element.send_keys("\n")

#Step 5. 사용자가 요청한 건수만큼 더보기 클릭하기
for a in range(0,page_cnt) :
    try :
        driver.find_element(By.XPATH,'//*[@id="divEducationList"]/div/div[2]/div[5]/a[1]').click() #펼쳐보기 클릭
        time.sleep(2)
        try :
            result = driver.switch_to_alert() #alert창이 떳을 경우 확인 누르기
            result.accept( )
        except :
            continue
    except :
        print('페이지 이동이 끝났습니다. 이제 데이터를 수집하겠습니다')
        break
    
# Step 6. 이미지 추출하여 저장하기 
file_no = 0                                
count = 1
img_src2=[]  # 이미지 원본 URL 주소 저장할 리스트

n = time.localtime()
s = '%04d-%02d-%02d-%02d-%02d-%02d' % (n.tm_year, n.tm_mon, n.tm_mday, n.tm_hour, n.tm_min, n.tm_sec)
img_dir = f_dir+s+'-'+query_txt
os.makedirs(img_dir)
os.chdir(img_dir)

html = driver.page_source
soup = BeautifulSoup(html, 'html.parser')
img_src = soup.find('ul','vod_list').find_all('img')

for i in img_src :
    img_src1=i['src']
    img_src2.append(img_src1)
    print(img_src1)
    count += 1
    if count > cnt :
        break

for i in range(0,len(img_src2)) :
    file_no += 1 
    #이미지 파일에 한글 이름이 들어갈 경우 오류가 발생하므로 인코딩 지정해 주어야 함
    urllib.request.urlretrieve(urllib.parse.quote(img_src2[i].encode('utf8'), '/:'),str(file_no)+'.jpg')

    time.sleep(0.5)      
    print("%s 번째 이미지 저장중입니다=======" %file_no)

# Step 7. 요약 정보를 출력합니다                
print("=" *70)
print("총 저장 건수는 %s 건 입니다 " %file_no)
print("파일 저장 경로: %s 입니다" %img_dir)
print("=" *70)

driver.close( )
```

    ====================================================================================================
     이 크롤러는 휴넷 사이트의 강의 자료 수집용 웹크롤러입니다.
    ====================================================================================================
    1.수집할 자료의 키워드는 무엇입니까?(예: 파이썬): 파이썬
    2.수집할 건수는 총 몇건입니까?(기본값:10): 30
    3.파일이 저장될 경로만 쓰세요(예: c:\py_temp\ ) : c:\py_temp\
    페이지 이동이 끝났습니다. 이제 데이터를 수집하겠습니다
    https://img.hunet.co.kr/files/Process/Course/Sample/[썸네일]341x187_파이썬을-활용한-빅데이터-분석.jpg
    https://img.hunet.co.kr/files/Process/Course/Sample/HLSP36564.png
    https://img.hunet.co.kr/files/Process/Course/Sample/HLSP23999.jpg
    https://img.hunet.co.kr/files/Process/Course/Sample/HLSP24803.JPG
    https://img.hunet.co.kr/files/Process/Course/Sample/HLSP24046.JPG
    https://img.hunet.co.kr/files/Process/Course/Sample/HLSP24803.JPG
    https://img.hunet.co.kr/files/Process/Course/Sample/HLSP36000.jpg
    https://img.hunet.co.kr/files/Process/Course/Sample/HLSP51995.jpg
    https://img.hunet.co.kr/files/Process/Course/Sample/HLSP40170.jpg
    https://img.hunet.co.kr/files/Process/Course/Sample/HLSP52750.JPG
    https://img.hunet.co.kr/files/Process/Course/Sample/HLSP52750.JPG
    https://img.hunet.co.kr/files/Process/Course/Sample/썸네일_01(2).jpg
    1 번째 이미지 저장중입니다=======
    2 번째 이미지 저장중입니다=======
    3 번째 이미지 저장중입니다=======
    4 번째 이미지 저장중입니다=======
    5 번째 이미지 저장중입니다=======
    6 번째 이미지 저장중입니다=======
    7 번째 이미지 저장중입니다=======
    8 번째 이미지 저장중입니다=======
    9 번째 이미지 저장중입니다=======
    10 번째 이미지 저장중입니다=======
    11 번째 이미지 저장중입니다=======
    12 번째 이미지 저장중입니다=======
    ======================================================================
    총 저장 건수는 12 건 입니다 
    파일 저장 경로: c:\py_temp\2022-07-09-15-34-59-파이썬 입니다
    ======================================================================
    

### 실행 결과 확인

![image.png](attachment:image.png)

실행이 잘 동작하는 걸 확인 할 수 있습니다.

## 3. 코드 리뷰 - Case 2. 인증 정보를 보낸 후 이미지를 가져올 수 있는 경우 (pixabay)

Case 1 과 중복되는 부분은 설명을 생략하도록 하겠습니다.

### Step 1. 필요한 모듈과 라이브러리를 로딩합니다.


```python
from bs4 import BeautifulSoup
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.chrome.service import Service
import urllib.request
import urllib
import time
import math
import os
import random
```

### Step 2. 필요한 정보를 입력 받습니다.


```python
print("=" *80)
print(" pixabay 사이트에서 이미지를 검색하여 수집하는 크롤러 입니다 ")
print("=" *80)

query_txt = input('1.크롤링할 이미지의 키워드는 무엇입니까?: ')
cnt = int(input('2.크롤링 할 건수는 몇건입니까?: '))
real_cnt = math.ceil(cnt / 100) # 실제 크롤링 할 페이지 수
f_dir=input('3.파일이 저장될 경로만 쓰세요(예: c:\\py_temp\\ ) : ')
if f_dir =='' :
    f_dir = "c:\\py_temp\\"
    
print("\n")
print("요청하신 데이터를 수집 중이오니 잠시만 기다려 주세요~~^^")
```

### Step 3. 파일을 저장할 폴더를 생성합니다


```python
n = time.localtime()
s = '%04d-%02d-%02d-%02d-%02d-%02d' % (n.tm_year, n.tm_mon, n.tm_mday, n.tm_hour, n.tm_min, n.tm_sec)

img_dir = f_dir+s+'-'+query_txt
os.makedirs(img_dir)
os.chdir(img_dir)   
```

### Step 4. 크롬 드라이버를 사용해서 웹 브라우저를 실행한 후 검색합니다


```python
s_time = time.time( )

s = Service("c:/py_temp/chromedriver.exe")
driver = webdriver.Chrome(service=s)

driver.get('https://pixabay.com/ko/')
time.sleep(3)

# 검색어 입력 창에 검색어 입력 후 검색 수행
element = driver.find_element(By.NAME,'q')
element.send_keys(query_txt)
element.submit()
```

### Step 5. 이미지 추출하여 저장하기 

먼저 pixabay 사이트의 경우 많은 이미지를 보려면 페이지 버튼을 클릭하는 대신 화면을 아래로 스크롤 다운을 해야 추가 이미지들이 보여집니다.  
그래서 아래와 같이 화면을 아래로 내리는 사용자 정의 함수를 생성한 후 실행해서 화면을 아래 로 이동한 후 소스코드를 가져와야 합니다.


```python
file_no = 1
count = 1
img_src2=[]    #이미지 파일의 url 주소 저장용 리스트

# 스크롤 다운 함수 만들기
def scroll_down(driver):
    #driver.execute_script("window.scrollTo(0,document.body.scrollHeight);")
    driver.execute_script("window.scrollBy(0,1000)")
    time.sleep(1)
```

그 다음 원본 이미지 url 주소를 수집합니다.
이미지는 div의 container--HcTw2 클래스에 저장되어있습니다.


```python
for a in range(1 , real_cnt+1):  
    
    for b in range(0,5):
        scroll_down(driver)
        time.sleep(1)
        
    # 원본 이미지 url 주소 수집 
    html = driver.page_source
    soup = BeautifulSoup(html, 'html.parser')    
    img_src = soup.find('div','container--HcTw2').find_all('img') #이미지 저장되어있는 div class
    
    for c in img_src :
        img_src1=c['src']
        
        if 'http' in img_src1 :
            img_src2.append(img_src1)
            print(img_src1)
            
        count += 1  
        
        if count > cnt :
            break
```

이대로면 이미지를 가져와서 저장할 수 있겠지만 이미지를 가지고 있는 pixabay 서버 가 클라이언트의 정보를 요청하기 때문에 다음과 같은 코드를 작성합니다.


```python
    #수집된 url 주소로 이미지 파일 가져와서 저장하기
    for e in range(0,len(img_src2)) :

        class AppURLopener(urllib.request.FancyURLopener):
            version = "Mozilla/5.0 (Windows NT 6.3; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) \
                       Chrome/47.0.2526.69 Safari/537.36"

        urllib.urlopener = AppURLopener()
        urllib.urlopener.retrieve(img_src2[e],str(file_no)+'.jpg')

        print("%s 페이지에서 %s 번째 이미지 저장중입니다=======" %(a,file_no))

        time.sleep(0.5) 

        if file_no >= cnt :
                break

        file_no += 1  
                
    if a > real_cnt :
        break
```

class AppURLopener() 로 브라우저의 정보를 서버쪽에 알려주도록 클래스를 만들고 urllib.urlopner로 서버에 접속 한 후 이미지를 다운로드 받아서 내컴퓨터에 저장합니다.

### Step 6. 요약 정보를 출력합니다     


```python
e_time = time.time( )
t_time = e_time - s_time

print("=" *70)
print("총 소요시간은 %s 초 입니다 " %round(t_time,1))
print("총 저장 건수는 %s 건 입니다 " %file_no)
print("파일 저장 경로: %s 입니다" %img_dir)
print("=" *70)

driver.close( )
```

### 전체 코드 실행(Case2)


```python
# Case 2. pixapay 사이트에서 그림 수집하기
# Step 1. 필요한 모듈과 라이브러리를 로딩합니다.

from bs4 import BeautifulSoup
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.chrome.service import Service
import urllib.request
import urllib
import time
import math
import os
import random

#Step 2. 필요한 정보를 입력 받습니다.
print("=" *80)
print(" pixabay 사이트에서 이미지를 검색하여 수집하는 크롤러 입니다 ")
print("=" *80)

query_txt = input('1.크롤링할 이미지의 키워드는 무엇입니까?: ')
cnt = int(input('2.크롤링 할 건수는 몇건입니까?: '))
real_cnt = math.ceil(cnt / 100) # 실제 크롤링 할 페이지 수
f_dir=input('3.파일이 저장될 경로만 쓰세요(예: c:\\py_temp\\ ) : ')
if f_dir =='' :
    f_dir = "c:\\py_temp\\"
    
print("\n")
print("요청하신 데이터를 수집 중이오니 잠시만 기다려 주세요~~^^")

#Step 3. 파일을 저장할 폴더를 생성합니다
n = time.localtime()
s = '%04d-%02d-%02d-%02d-%02d-%02d' % (n.tm_year, n.tm_mon, n.tm_mday, n.tm_hour, n.tm_min, n.tm_sec)

img_dir = f_dir+s+'-'+query_txt
os.makedirs(img_dir)
os.chdir(img_dir)      

#Step 4. 크롬 드라이버를 사용해서 웹 브라우저를 실행한 후 검색합니다
s_time = time.time( )

s = Service("c:/py_temp/chromedriver.exe")
driver = webdriver.Chrome(service=s)

driver.get('https://pixabay.com/ko/')
time.sleep(3)

# 검색어 입력 창에 검색어 입력 후 검색 수행
element = driver.find_element(By.NAME,'q')
element.send_keys(query_txt)
element.submit()

# Step 5. 이미지 추출하여 저장하기 
file_no = 1
count = 1
img_src2=[]    #이미지 파일의 url 주소 저장용 리스트

# 스크롤 다운 함수 만들기
def scroll_down(driver):
    #driver.execute_script("window.scrollTo(0,document.body.scrollHeight);")
    driver.execute_script("window.scrollBy(0,1000)")
    time.sleep(1)
    
for a in range(1 , real_cnt+1):  
    
    for b in range(0,5):
        scroll_down(driver)
        time.sleep(1)
        
    # 원본 이미지 url 주소 수집 
    html = driver.page_source
    soup = BeautifulSoup(html, 'html.parser')    
    img_src = soup.find('div','container--HcTw2').find_all('img')
    
    for c in img_src :
        img_src1=c['src']
        
        if 'http' in img_src1 :
            img_src2.append(img_src1)
            print(img_src1)
            
        count += 1  
        
        if count > cnt :
            break
            
    #수집된 url 주소로 이미지 파일 가져와서 저장하기
    for e in range(0,len(img_src2)) :

        class AppURLopener(urllib.request.FancyURLopener):
            version = "Mozilla/5.0 (Windows NT 6.3; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) \
                       Chrome/47.0.2526.69 Safari/537.36"

        urllib.urlopener = AppURLopener()
        urllib.urlopener.retrieve(img_src2[e],str(file_no)+'.jpg')

        print("%s 페이지에서 %s 번째 이미지 저장중입니다=======" %(a,file_no))

        time.sleep(0.5) 

        if file_no >= cnt :
                break

        file_no += 1  
                
    if a > real_cnt :
        break

# Step 6. 요약 정보를 출력합니다                
e_time = time.time( )
t_time = e_time - s_time

print("=" *70)
print("총 소요시간은 %s 초 입니다 " %round(t_time,1))
print("총 저장 건수는 %s 건 입니다 " %file_no)
print("파일 저장 경로: %s 입니다" %img_dir)
print("=" *70)

driver.close( )
```

    ================================================================================
     pixabay 사이트에서 이미지를 검색하여 수집하는 크롤러 입니다 
    ================================================================================
    1.크롤링할 이미지의 키워드는 무엇입니까?: 고양이
    2.크롤링 할 건수는 몇건입니까?: 30
    3.파일이 저장될 경로만 쓰세요(예: c:\py_temp\ ) : c:\py_temp\
    
    
    요청하신 데이터를 수집 중이오니 잠시만 기다려 주세요~~^^
    https://cdn.pixabay.com/photo/2015/04/23/21/59/tree-736877__480.jpg
    https://cdn.pixabay.com/photo/2017/02/20/18/03/cat-2083492__340.jpg
    https://cdn.pixabay.com/photo/2014/11/30/14/11/cat-551554__340.jpg
    https://cdn.pixabay.com/photo/2012/05/03/23/13/cat-46676__340.png
    https://cdn.pixabay.com/photo/2016/03/28/12/35/cat-1285634__340.png
    https://cdn.pixabay.com/photo/2014/03/29/09/17/cat-300572__340.jpg
    https://cdn.pixabay.com/photo/2015/11/16/14/43/cat-1045782__340.jpg
    https://cdn.pixabay.com/photo/2015/03/27/13/16/maine-coon-694730__340.jpg
    https://cdn.pixabay.com/photo/2015/06/07/19/42/animal-800760__340.jpg
    https://cdn.pixabay.com/photo/2014/04/13/20/49/cat-323262__340.jpg
    https://cdn.pixabay.com/photo/2018/03/27/17/25/cat-3266673__340.jpg
    https://cdn.pixabay.com/photo/2017/07/25/01/22/cat-2536662__340.jpg
    https://cdn.pixabay.com/photo/2018/01/28/12/37/cat-3113513__340.jpg
    https://cdn.pixabay.com/photo/2013/05/30/18/21/cat-114782__340.jpg
    https://cdn.pixabay.com/photo/2017/11/09/21/41/cat-2934720__340.jpg
    https://cdn.pixabay.com/photo/2017/12/11/15/34/lion-3012515__340.jpg
    https://cdn.pixabay.com/photo/2015/05/22/05/52/cat-778315__340.jpg
    https://cdn.pixabay.com/photo/2017/11/14/13/06/kitty-2948404__340.jpg
    https://cdn.pixabay.com/photo/2017/07/24/19/57/tiger-2535888__340.jpg
    https://cdn.pixabay.com/photo/2016/07/10/21/47/cat-1508613__340.jpg
    https://cdn.pixabay.com/photo/2014/12/22/10/04/lions-577104__340.jpg
    https://cdn.pixabay.com/photo/2018/03/26/20/49/tiger-3264048__340.jpg
    https://cdn.pixabay.com/photo/2017/10/25/16/54/african-lion-2888519__340.jpg
    https://cdn.pixabay.com/photo/2016/02/10/16/37/cat-1192026__340.jpg
    https://cdn.pixabay.com/photo/2019/11/08/11/56/kitten-4611189__340.jpg
    https://cdn.pixabay.com/photo/2018/10/01/09/21/pets-3715733__340.jpg
    https://cdn.pixabay.com/photo/2016/09/05/21/37/cat-1647775__340.jpg
    https://cdn.pixabay.com/photo/2018/01/11/23/16/woman-3077180__340.jpg
    https://cdn.pixabay.com/photo/2016/06/14/00/14/cat-1455468__340.jpg
    https://cdn.pixabay.com/photo/2017/11/06/09/53/tiger-2923186__340.jpg
    1 페이지에서 1 번째 이미지 저장중입니다=======
    

    C:\Users\HONGSU~1\AppData\Local\Temp/ipykernel_19248/3631728124.py:94: DeprecationWarning: AppURLopener style of invoking requests is deprecated. Use newer urlopen functions/methods
      urllib.urlopener = AppURLopener()
    

    1 페이지에서 2 번째 이미지 저장중입니다=======
    1 페이지에서 3 번째 이미지 저장중입니다=======
    1 페이지에서 4 번째 이미지 저장중입니다=======
    1 페이지에서 5 번째 이미지 저장중입니다=======
    1 페이지에서 6 번째 이미지 저장중입니다=======
    1 페이지에서 7 번째 이미지 저장중입니다=======
    1 페이지에서 8 번째 이미지 저장중입니다=======
    1 페이지에서 9 번째 이미지 저장중입니다=======
    1 페이지에서 10 번째 이미지 저장중입니다=======
    1 페이지에서 11 번째 이미지 저장중입니다=======
    1 페이지에서 12 번째 이미지 저장중입니다=======
    1 페이지에서 13 번째 이미지 저장중입니다=======
    1 페이지에서 14 번째 이미지 저장중입니다=======
    1 페이지에서 15 번째 이미지 저장중입니다=======
    1 페이지에서 16 번째 이미지 저장중입니다=======
    1 페이지에서 17 번째 이미지 저장중입니다=======
    1 페이지에서 18 번째 이미지 저장중입니다=======
    1 페이지에서 19 번째 이미지 저장중입니다=======
    1 페이지에서 20 번째 이미지 저장중입니다=======
    1 페이지에서 21 번째 이미지 저장중입니다=======
    1 페이지에서 22 번째 이미지 저장중입니다=======
    1 페이지에서 23 번째 이미지 저장중입니다=======
    1 페이지에서 24 번째 이미지 저장중입니다=======
    1 페이지에서 25 번째 이미지 저장중입니다=======
    1 페이지에서 26 번째 이미지 저장중입니다=======
    1 페이지에서 27 번째 이미지 저장중입니다=======
    1 페이지에서 28 번째 이미지 저장중입니다=======
    1 페이지에서 29 번째 이미지 저장중입니다=======
    1 페이지에서 30 번째 이미지 저장중입니다=======
    ======================================================================
    총 소요시간은 43.0 초 입니다 
    총 저장 건수는 30 건 입니다 
    파일 저장 경로: c:\py_temp\2022-07-09-16-30-07-고양이 입니다
    ======================================================================
    

### 실행 결과 확인

![image.png](attachment:image.png)

코드는 잘 실행 되었지만 호랑이가 있는 경우도 볼 수 있습니다.  
아마 호랑이가 고양이과라서 같이 검색이 된게 아닐까 하는 생각을 해 봅니다.
