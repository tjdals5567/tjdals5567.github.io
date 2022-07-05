---
layout: single
title:  "포스트 규칙을 생활화"
categories : NLP
---


# 빈도분석 연습문제

- 뉴스데이터로 빈도분석 연습문제를 풀어봅시다.

## 문제

1.데이터를 pd.read_csv() 함수로 읽으시오

2.데이터에서 결측치가 있는 행을 제거하시오

3.데이터에서 [분류2]는 제거한 복사본 data1을 만드시오

4.데이터에서 [분류1]은 제거한 복사본 data2를 만드시오

5.data1에서 [분류1]을 기준으로 텍스트의 어휘 빈도 특징을 조사하시오
* 단, [분류1] 기준은 ‘상위기준,하위기준’ 으로 묶여 있으므로 상위기준별 빈도 특징과 하위기준별 빈도 특징을 각각 구하시오 (통계적 특징 및 그래프)
6.data2에서 [분류1]를 기준으로 텍스트의 어휘 빈도 특징을 조사하시오
* 단, [분류2] 기준은 ‘상위기준,하위기준‘ 으로 묶여 있으므로 상위기준별 빈도 특징과 하위기준별 빈도 특징을 각각 구하시오 (통계적 특징 및 그래프)
7.2018년 자료와 2019년 자료의 어휘의 빈도별 특징을 구하시오
* 분류별 특징까지는 분석하지 않아도 됨

-------------------------------

# 필요 라이브러리 호출


```python
import pandas as pd
import numpy as np
from collections import Counter
import rhinoMorph
import matplotlib.pyplot as plt
from matplotlib import font_manager,rc
%matplotlib inline
```

아래는 plot을 그릴때 한글출력을 위해 사용합니다.


```python
font_path = 'C:\\Windows\\Fonts\\HANBatang.ttf'
font = font_manager.FontProperties(fname=font_path).get_name()
rc('font',family=font)
```

- - -

# 문제1 : CSV파일 호출


```python
df = pd.read_excel('뉴스데이터.csv',sheet_name = '번역')
```

아래에서 df파일을 확인합니다.


```python
df.head()
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
      <th>id</th>
      <th>날짜</th>
      <th>분류1</th>
      <th>분류2</th>
      <th>텍스트</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>77369</td>
      <td>20180619</td>
      <td>지역,광주</td>
      <td>지역,전남</td>
      <td>하늘길 운항노선도 광주-제주 광주-양양 광주-김해 등 국내는 물론 일본과 중국 등 ...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>77370</td>
      <td>20180619</td>
      <td>지역,광주</td>
      <td>지역,전남</td>
      <td>저가항공보다는 프레미엄 비즈니스 전문 항공사를 추구하는 에어필립은 지방 도시간 항공...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>77371</td>
      <td>20180619</td>
      <td>지역,광주</td>
      <td>지역,전남</td>
      <td>울릉도와 흑산도 등의 소형공항이 수년 내 문을 열 경우 광주․전남 지역민의 관광 접...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>77372</td>
      <td>20180619</td>
      <td>지역,광주</td>
      <td>지역,전남</td>
      <td>지역민들이 해외여행을 위해 인천공항에 갈 때마다 고속버스에 ‘수화물’을 싣고 내리던...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>77378</td>
      <td>20180619</td>
      <td>지역,광주</td>
      <td>지역,전남</td>
      <td>포항출신인 필립에셋 엄일석 회장은 “광주에 18년 동안 살면서 강원도나 인천공항에 ...</td>
    </tr>
  </tbody>
</table>
</div>




```python
df.dtypes
```




    id      int64
    날짜      int64
    분류1    object
    분류2    object
    텍스트    object
    dtype: object




```python
df.shape
```




    (9057, 5)



- - -

# 문제2 :결측치 제거


```python
df.isnull().sum()
```




    id        0
    날짜        0
    분류1       0
    분류2    2734
    텍스트       0
    dtype: int64




```python
df = df.dropna()
```


```python
print(df.shape)
print(df.isnull().sum())
```

    (6323, 5)
    id     0
    날짜     0
    분류1    0
    분류2    0
    텍스트    0
    dtype: int64
    

- - - 

# 문제 3-4 : data1,data2 만들기


```python
data1 = df.drop('분류2',axis = 1)
data2 = df.drop('분류1',axis = 1)
```


```python
data1.head()
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
      <th>id</th>
      <th>날짜</th>
      <th>분류1</th>
      <th>텍스트</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>77369</td>
      <td>20180619</td>
      <td>지역,광주</td>
      <td>하늘길 운항노선도 광주-제주 광주-양양 광주-김해 등 국내는 물론 일본과 중국 등 ...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>77370</td>
      <td>20180619</td>
      <td>지역,광주</td>
      <td>저가항공보다는 프레미엄 비즈니스 전문 항공사를 추구하는 에어필립은 지방 도시간 항공...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>77371</td>
      <td>20180619</td>
      <td>지역,광주</td>
      <td>울릉도와 흑산도 등의 소형공항이 수년 내 문을 열 경우 광주․전남 지역민의 관광 접...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>77372</td>
      <td>20180619</td>
      <td>지역,광주</td>
      <td>지역민들이 해외여행을 위해 인천공항에 갈 때마다 고속버스에 ‘수화물’을 싣고 내리던...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>77378</td>
      <td>20180619</td>
      <td>지역,광주</td>
      <td>포항출신인 필립에셋 엄일석 회장은 “광주에 18년 동안 살면서 강원도나 인천공항에 ...</td>
    </tr>
  </tbody>
</table>
</div>




```python
data2.head()
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
      <th>id</th>
      <th>날짜</th>
      <th>분류2</th>
      <th>텍스트</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>77369</td>
      <td>20180619</td>
      <td>지역,전남</td>
      <td>하늘길 운항노선도 광주-제주 광주-양양 광주-김해 등 국내는 물론 일본과 중국 등 ...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>77370</td>
      <td>20180619</td>
      <td>지역,전남</td>
      <td>저가항공보다는 프레미엄 비즈니스 전문 항공사를 추구하는 에어필립은 지방 도시간 항공...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>77371</td>
      <td>20180619</td>
      <td>지역,전남</td>
      <td>울릉도와 흑산도 등의 소형공항이 수년 내 문을 열 경우 광주․전남 지역민의 관광 접...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>77372</td>
      <td>20180619</td>
      <td>지역,전남</td>
      <td>지역민들이 해외여행을 위해 인천공항에 갈 때마다 고속버스에 ‘수화물’을 싣고 내리던...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>77378</td>
      <td>20180619</td>
      <td>지역,전남</td>
      <td>포항출신인 필립에셋 엄일석 회장은 “광주에 18년 동안 살면서 강원도나 인천공항에 ...</td>
    </tr>
  </tbody>
</table>
</div>



- - -

# 문제5 : data1 어휘 빈도 조사

### 상위기준, 하위기준 분리


```python
def split_word_cnt(df):
    high_word = [] #상위기준 리스트 생성
    low_word = [] #하위기준 리스트 생성
    target = df.iloc[:,2] 
    
    for i in target:
        
        """
        분류1을 기준으로 ,로 split 후 첫번째는 상위기준으로 리스트에 추가,
        두번째 이하는 하위기준으로 리스트에 추가합니다
        """ 
        
        word = i.split(',')
        for i in range(len(word)):
            if i == 0:
                high_word.append(word[0])
            else:
                low_word.append(word[i])
                
    high_word = list(set(high_word)) #list(set()) 형태로 중복 문자 제거
    low_word = list(set(low_word))
    return high_word, low_word
```


```python
data1_high, data1_low = split_word_cnt(data1)
```


```python
#상위기준 확인
data1_high
```




    ['경제', '문화', 'IT_과학', '사회', '국제', '지역', '스포츠']




```python
#하위기준 확인
data1_low
```




    ['중동_아프리카',
     '골프',
     '산업_기업',
     '모바일',
     '요리_여행',
     '미디어',
     '부산',
     '스포츠일반',
     '지역일반',
     '콘텐츠',
     '경제일반',
     '충북',
     '전북',
     '출판',
     '울산',
     '반도체',
     '농구_배구',
     '한국프로축구',
     '취업_창업',
     '전시_공연',
     '날씨',
     '제주',
     '국가대표팀',
     '중남미',
     '인터넷_SNS',
     '일본프로야구',
     '무역',
     '과학',
     '야구',
     '전남',
     '대전',
     '자원',
     '충남',
     '노동_복지',
     '광주',
     '아시아',
     '올림픽_아시안게임',
     '유통',
     '장애인',
     '중국',
     '보안',
     '종교',
     '미술_건축',
     '경북',
     '증권_증시',
     '강원',
     '문화일반',
     'IT_과학일반',
     '축구',
     '서비스_쇼핑',
     '대구',
     '생활',
     '메이저리그',
     '외환',
     '러시아',
     '자동차',
     '환경']



### 형태소 분리


```python
def make_morph(df,df_high): #make_morph(데이터프레임, 상위/하위 기준)
    #형태소 분석기로 rhinoMorph를 사용
    rn = rhinoMorph.startRhino()
    
    #stopword 설정
    stopword_ko = ['하다','있다','되다','그','않다','없다','나','말','사람','이'
                   ,'보다','한','때','년','같다','대하다','일','이','생각','위하다',
                  '때문','그것','그러나','가다','받다','그렇다','알다','사회','더',
                  '그녀','문제','오다','그리고','크다','속']
    
    morphed_dict = {}
    for i in df_high:
        
        """
        기준의 문자가 포함된 데이터프레임을 따로 만들어 텍스트를 형태소 분리와 불용어 처리를 합니다.
        그 후 리스트에 저장한 뒤에, {기준:리스트} 형식으로 사전형으로 저장합니다.
        """
        
        df_i = df[df.iloc[:,2].str.contains(i)]
        morphed_data = []
        for j in df_i['텍스트']:
            # 형태소 분리
            morphed_data_j = rhinoMorph.onlyMorph_list(rn,j,pos=['NNG','NNP','VV'
                                                               ,'VA','XR','IC',
                                                              'MM','MAG','MAJ'],
                                                      eomi = True)
            morphed_data = morphed_data + morphed_data_j
        
        # 불용어 처리
        morphed_data_stop = [word for word in morphed_data if not word in stopword_ko] 
        morphed_dict[i] = Counter(morphed_data_stop)
        
    return morphed_dict
```


```python
data1_high_morph = make_morph(data1,data1_high)
```

    filepath:  C:\Users\HongSungMin\anaconda3\Lib\site-packages
    classpath:  C:\Users\HongSungMin\anaconda3\Lib\site-packages\rhinoMorph/lib/rhino.jar
    RHINO started!
    


```python
data1_low_morph = make_morph(data1,data1_low)
```

    filepath:  C:\Users\HongSungMin\anaconda3\Lib\site-packages
    classpath:  C:\Users\HongSungMin\anaconda3\Lib\site-packages\rhinoMorph/lib/rhino.jar
    JVM is already started~
    RHINO started!
    

### 빈도별 막대그래프 그리기


```python
def make_barplot(df_morph):
    """
    상위기준, 하위기준 수에 맞추어 그래프를 그리고 막대그래프는 빈도 상위 20개를 출력하도록 하였습니다.
    """
    
    plt.figure(figsize=(15,len(df_morph)*1.5))
    row_len = round(len(df_morph) / 2)+1
    col_len = 2
    
    for idx,i in enumerate(df_morph.keys()):
        plt.subplot(row_len,col_len,idx+1)
        plt.title(i,fontsize=15)
        
        #빈도순으로 내림차순으로 정렬
        sorted_keys = sorted(df_morph[i], key = df_morph[i].get, reverse = True)
        sorted_values = sorted(df_morph[i].values(),reverse = True)
        
        #출력되는 막대그래프를 최대 20개로 설정
        if len(sorted_values) >= 20:
            plt.bar(range(20),sorted_values[:20])
            plt.xticks(range(20),sorted_keys[:20],rotation=45)
        else:
            plt.bar(range(len(sorted_values)),sorted_values)
            plt.xticks(range(len(sorted_keys)),sorted_keys,rotation=45)
    plt.tight_layout()
    plt.show()
```

---

**출력되는 그래프가 많습니다.**


```python
make_barplot(data1_high_morph)
```


    
![png](output_41_0.png)
    



```python
make_barplot(data1_low_morph)
```


    
![png](output_42_0.png)
    


# 문제6 : data2 어휘 빈도 조사


```python
# 함수를 정의해 놓았기 때문에 data2는 쉽습니다.
data2_high, data2_low = split_word_cnt(data2)
data2_high_morph = make_morph(data2,data2_high)
data2_low_morph = make_morph(data2,data2_low)
```

    filepath:  C:\Users\HongSungMin\anaconda3\Lib\site-packages
    classpath:  C:\Users\HongSungMin\anaconda3\Lib\site-packages\rhinoMorph/lib/rhino.jar
    JVM is already started~
    RHINO started!
    filepath:  C:\Users\HongSungMin\anaconda3\Lib\site-packages
    classpath:  C:\Users\HongSungMin\anaconda3\Lib\site-packages\rhinoMorph/lib/rhino.jar
    JVM is already started~
    RHINO started!
    


```python
make_barplot(data2_high_morph)
```


    
![png](output_45_0.png)
    



```python
make_barplot(data2_low_morph)
```


    
![png](output_46_0.png)
    


# 문제7 : 연도별 어휘 빈도 조사


```python
#날짜가 정수형이므로 문자형으로 변환합니다
df_date = df.astype({"날짜": str})
```


```python
df_2018 = df_date[df_date['날짜'].str.contains('2018')] # 2018이 포함되어있는 df_date
df_2019 = df_date[df_date['날짜'].str.contains('2019')] # 2019이 포함되어있는 df_date
```


```python
#다시 정수형으로 변환합니다.
df_2018 = df_2018.astype({"날짜": int})
df_2019 = df_2019.astype({"날짜": int})
```


```python
df_2018.head()
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
      <th>id</th>
      <th>날짜</th>
      <th>분류1</th>
      <th>분류2</th>
      <th>텍스트</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>77369</td>
      <td>20180619</td>
      <td>지역,광주</td>
      <td>지역,전남</td>
      <td>하늘길 운항노선도 광주-제주 광주-양양 광주-김해 등 국내는 물론 일본과 중국 등 ...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>77370</td>
      <td>20180619</td>
      <td>지역,광주</td>
      <td>지역,전남</td>
      <td>저가항공보다는 프레미엄 비즈니스 전문 항공사를 추구하는 에어필립은 지방 도시간 항공...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>77371</td>
      <td>20180619</td>
      <td>지역,광주</td>
      <td>지역,전남</td>
      <td>울릉도와 흑산도 등의 소형공항이 수년 내 문을 열 경우 광주․전남 지역민의 관광 접...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>77372</td>
      <td>20180619</td>
      <td>지역,광주</td>
      <td>지역,전남</td>
      <td>지역민들이 해외여행을 위해 인천공항에 갈 때마다 고속버스에 ‘수화물’을 싣고 내리던...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>77378</td>
      <td>20180619</td>
      <td>지역,광주</td>
      <td>지역,전남</td>
      <td>포항출신인 필립에셋 엄일석 회장은 “광주에 18년 동안 살면서 강원도나 인천공항에 ...</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_2019.head()
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
      <th>id</th>
      <th>날짜</th>
      <th>분류1</th>
      <th>분류2</th>
      <th>텍스트</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>571</th>
      <td>876590</td>
      <td>20190101</td>
      <td>지역,울산</td>
      <td>지역,전북</td>
      <td>예를 들어 SK텔레콤 T1, 아프리카 프릭스, 젠지, 이번에 LCK로 올라온 담원 ...</td>
    </tr>
    <tr>
      <th>572</th>
      <td>876839</td>
      <td>20190101</td>
      <td>국제,러시아</td>
      <td>국제,유럽_EU</td>
      <td>스포츠경향은 31일 슈 측근의 말을 인용해 슈와 임요성 부부가 오래 전부터 이미 별...</td>
    </tr>
    <tr>
      <th>573</th>
      <td>876849</td>
      <td>20190101</td>
      <td>국제,러시아</td>
      <td>국제,유럽_EU</td>
      <td>슈는 지난 6월 서울의 한 호텔 카지노에서 6억원을 빌린 뒤 갚지 않은 혐의로 박모...</td>
    </tr>
    <tr>
      <th>574</th>
      <td>876888</td>
      <td>20190101</td>
      <td>스포츠,축구,국가대표팀</td>
      <td>스포츠,축구,해외축구</td>
      <td>여기서 언급된 이영애와 뜻을 같이한 몇몇은 이기원 서울대 교수와 바이오·병원 운영 ...</td>
    </tr>
    <tr>
      <th>575</th>
      <td>876894</td>
      <td>20190101</td>
      <td>스포츠,축구,국가대표팀</td>
      <td>스포츠,축구,해외축구</td>
      <td>제일의료재단 측은 병원을 인수할 주체가 마땅치 않자 여러 투자자가 컨소시엄을 구성하...</td>
    </tr>
  </tbody>
</table>
</div>



#### 다시 위의 함수를 실행합니다.


```python
df_2018_high, df_2018_low = split_word_cnt(df_2018)
df_2018_high_morph = make_morph(df_2018,df_2018_high)
df_2018_low_morph = make_morph(df_2018,df_2018_low)
```

    filepath:  C:\Users\HongSungMin\anaconda3\Lib\site-packages
    classpath:  C:\Users\HongSungMin\anaconda3\Lib\site-packages\rhinoMorph/lib/rhino.jar
    JVM is already started~
    RHINO started!
    filepath:  C:\Users\HongSungMin\anaconda3\Lib\site-packages
    classpath:  C:\Users\HongSungMin\anaconda3\Lib\site-packages\rhinoMorph/lib/rhino.jar
    JVM is already started~
    RHINO started!
    


```python
df_2019_high, df_2019_low = split_word_cnt(df_2019)
df_2019_high_morph = make_morph(df_2019,df_2019_high)
df_2019_low_morph = make_morph(df_2019,df_2019_low)
```

    filepath:  C:\Users\HongSungMin\anaconda3\Lib\site-packages
    classpath:  C:\Users\HongSungMin\anaconda3\Lib\site-packages\rhinoMorph/lib/rhino.jar
    JVM is already started~
    RHINO started!
    filepath:  C:\Users\HongSungMin\anaconda3\Lib\site-packages
    classpath:  C:\Users\HongSungMin\anaconda3\Lib\site-packages\rhinoMorph/lib/rhino.jar
    JVM is already started~
    RHINO started!
    


```python
make_barplot(df_2018_high_morph)
```


    
![png](output_56_0.png)
    



```python
make_barplot(df_2019_high_morph)
```


    
![png](output_57_0.png)
    


출력이 성공적으로 되었음을 확인할 수 있습니다.
