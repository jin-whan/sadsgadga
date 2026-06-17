#마이크로프로세서응용설계 기말 실기평가

## 1. 기본 정보
- 학번:2019732056  
- 이름:최진환  
- 작품명: 웹 서버를 통한 주식 인기순 출력 프로그램

## 2. 선택 기능
- 선택 기능 1: F3 Python 데이터 처리
- 선택 기능 2: F4 웹 서버 또는 웹 API
- 선택 기능 3: F5 외부 연동 또는 저장 기능

## 3. 작품 목적
주식 거래대금을 분석하여 현재 인기있는 주식을 웹 서버를 통해 확인할 수 있도록 함.

## 4. 사용한 하드웨어
RasberryPi4
## 5. 사용한 소프트웨어 및 라이브러리
|구분|내용|
|---|---|
|실행 환경|Juptyter Lab/Jupyter Notebook|
|Python version|Python 3.13.5|
|사용 라이브러리|BeautifulSoup, requests, bottle|
|Web framework|Bottle|

## 6. 전체 시스템 동작 설명
먼저 관심있는 종목을 codes 배열에 종목코드의 형태로 넣어줍니다. 다음 Web scrapping을 이용해 Naver-pay 증권에 접속해 주식의 이름, 현재가, 거래량, 거래대금을 scrapping 하는 함수를 작성합니다. 이때 거래대금 순으로 정렬하기 위해 문자열로 반환된 거래대금을 python의 replace함수를 통해 ,를 지워주고 정수로 변환합니다.  
이후 웹 서버를 구동하고 웹 서버에 접속 시 직전의 함수가 실행되고 딕셔너리 형태의 주식 정보가 수집됩니다. 수집된 정보를 python의 sort 함수를 통해 거래대금이 높은 순으로 정렬합니다.
이후 웹 서버에 python의 join 함수를 통해 lines에 저장된 출력문을 html 줄바꿈 명령어 사이사이에 배치해 출력합니다.

## 7. 선택 기능별 구현 증거표
|선택 기능|기능명|구현 여부|코드 위치|영상 시간|증거 설명|
|---|---|---|---|---|---|
|F3|Python 데이터 처리|O|main.ipynb line 21 33 38|00:00~00:17|정수 변환과 정렬이 잘 이루어집니다.|
|F4|웹 서버 또는 웹 API|O|main.ipynb line 25~40|00:17~00:24|웹 서버에 잘 출력됩니다.|
|F5|외부 연동 또는 저장 기능|O|main.ipynb line 7~23|00:24~|현재가가 잘 반영되고 있습니다.|

## 8. 실행 방법
main.ipynb 파일을 열고 셀을 실행한다.
이후 RasberryPi의 IP로 http://IP:8080/ 접속한다.

## 9 .주요 코드 설명
```
def get_stock_data(code):
    
    url = 'https://finance.naver.com/item/main.nhn?code=' + code
    
    response = requests.get(url)
    response.raise_for_status()
    html = response.text
    soup = BeautifulSoup(html, 'html.parser')
    
    name = soup.select_one('div.rate_info dl.blind strong').get_text()
    price = soup.select_one('div.rate_info div.today span.blind').get_text()
    trade_amount = soup.select_one('table.no_info span.sp_txt9+em span.blind').get_text()
    trade_value_s = soup.select_one('table.no_info span.sp_txt10+em span.blind').get_text()

    trade_value = int(trade_value_s.replace(',',''))

    return {'name':name, 'price':price, 'trade_amount':trade_amount, 'trade_value_s':trade_value_s, 'trade_value':trade_value}
```
주식의 종목코드를 입력받으면 NaverPay 증권에 접속해 주식 이름, 현재가, 거래량, 거래대금을 받아와 딕셔너리 형태로 반환하는 함수입니다. 거래대금 같은 경우 정렬에 사용하기 위해 정수로 변환해줍니다.

```
@route('/')
def index():
    stock_data = []
    lines = []

    for code in codes:
        stock_data.append(get_stock_data(code))

    stock_data.sort(key = lambda x:x['trade_value'] , reverse=True)

    for i,s in enumerate(stock_data):
        lines.append(f'인기 {i+1}위 : {s['name']} / 현재가 : {s['price']} / 거래량 : {s['trade_amount']} / 거래대금 : {s['trade_value_s']} 백만 원')
    
    return '<br>'.join(lines)

run(host='0.0.0.0', port=8080)
```
웹 서버 구동 후 주식 데이터 수집 함수가 돌아간 뒤 정수로 변환한 거래대금을 기준으로 정렬합니다. 이후 웹 서버에 출력하기 위해 lines로 정리한 뒤 줄바꿈과 함께 출력합니다.

##10. 구현 결과
인기 1위 : SK하이닉스 / 현재가 : 2,408,000 / 거래량 : 1,492,519 / 거래대금 : 3,571,728 백만 원  
인기 2위 : 삼성전자 / 현재가 : 336,500 / 거래량 : 8,648,192 / 거래대금 : 2,893,557 백만 원  
인기 3위 : 한화오션 / 현재가 : 138,900 / 거래량 : 4,334,298 / 거래대금 : 596,463 백만 원  
인기 4위 : 현대차 / 현재가 : 621,000 / 거래량 : 453,748 / 거래대금 : 281,306 백만 원  
인기 5위 : LG전자 / 현재가 : 234,500 / 거래량 : 746,465 / 거래대금 : 172,213 백만 원  

거래대금 순으로 잘 정렬된 결과가 출력되었습니다.

##11. 한계점 및 개선 가능성
route를 통해 웹 서버에 접속할 시 주식 데이터를 수집하지만 이후 다시 접속하지 않으면 실시간으로 주식 정보가 갱신되지 않는 한계점이 있습니다. 이는 Python의 time을 이용한 반복으로 개선 가능성이 있습니다. 또한 html의 table을 활용한다면 표 형식으로 정리할 수도 있는 개선 가능성이 있습니다.
