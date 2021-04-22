# 로그프레소 팝업 관련 쿼리

## 팝업

- 20210422 prod -> dev 이전
- prod 디지파츠, 카플랫, 카플랫 비즈 dev로 이동
- CLIENT_ID 리스트

|리스트   |
|---|
| ce45a83c8c58458c9ca0252b6ac480dd  |
| 796b1d446b054d0b9e95191720d87528  |
| 2cfd111262b04a329e0c5366f46c4a4e  |




### 공용-dev
- 확인사항(20210422) 
   - 그린카 제외해서 팝업을 보낼지?
   - de팀에서 알아서 해줄듯?

eval url = "https://www.airdeep.co.kr/otoair/openapi/notify.do?"

| eval header = dict("client-id", "otoairadmin")

| eval DETC_CD = concat("DETC_CD=", DETC_CD)

| eval DETC_TIME = concat("&DETC_TIME=", replace(DETC_TIME, " ", "-"))

| eval TERMINAL_ID = concat("&TERMINAL_ID=", TERMINAL_ID)

| eval CLIENT_ID = concat("&CLIENT_ID=", CLIENT_ID)

| eval url = concat(url, DETC_CD, DETC_TIME, TERMINAL_ID, CLIENT_ID)

| wget header=header method=get

| eval send_time = now()

| rename _wget_code as result, line as message, DETC_CD as detc_cd, DETC_TIME as detc_time, TERMINAL_ID as terminal_id, CLIENT_ID as client_id, REGION_ID as region_id

| fields detc_cd, detc_time, terminal_id, client_id, region_id, send_time, message, url, result, client_id, region_id

| import create=t otoair_dev_send_log


### 그린카-dev

- 그린카에서 차번호로 보여주기 요청
- de팀에서 팝업을 보내기 위해 url encoding 요청
- logpresso에서 방법 두가지
  - json 형태 변환
  - 추후에 해야함(컨플루언스 참조)
    - https://docs.logpresso.com/manual/query_manual_ko#25215a8a5b3ef81c#SEARCH
  - urlencode 함수
    - https://docs.logpresso.com/manual/query_manual_ko#556e5091663b9cc9#SEARCH

eval url = "https://www.airdeep.co.kr/otoair/openapi/notify.do?"

| serach CLIENT_ID == "b1446038ac4a46cc8718a25f14ab4a46	"

| eval header = dict("client-id", "otoairadmin")

| eval DETC_CD = concat("DETC_CD=", DETC_CD)

| eval DETC_TIME = concat("&DETC_TIME=", replace(DETC_TIME, " ", "-"))

| join type=left TERMINAL_ID
[memlookup name=greencar_mapping]

| eval TERMINAL_ID = concat("&CAR_NO=", CAR_NO)

| eval CLIENT_ID = concat("&CLIENT_ID=", CLIENT_ID)

| eval url = concat(url, urlencode( concat(DETC_CD, DETC_TIME, TERMINAL_ID, CLIENT_ID), "UTF-8"))

| wget header=header method=get

| eval send_time = now()

| rename _wget_code as result, line as message, DETC_CD as detc_cd, DETC_TIME as detc_time, CAR_NO as car_no, CLIENT_ID as client_id, REGION_ID as region_id

| fields detc_cd, detc_time, car_no, client_id, region_id, send_time, message, url, result, client_id, region_id

| import create=t otoair_dev_send_log

### test 쿼리

table otoair_dev_detect_result

| eval url = "https://www.otoair.co.kr/otoair/openapi/notify.do?"

| eval header = dict("client-id", "otoairadmin")

| eval DETC_CD = concat("DETC_CD=", DETC_CD)

| eval DETC_TIME = concat("&DETC_TIME=", replace(DETC_TIME, " ", "-"))

| join type=left TERMINAL_ID
[memlookup name=greencar_mapping]

| eval TERMINAL_ID = concat("&CAR_NO=", CAR_NO)

| eval CLIENT_ID = concat("&CLIENT_ID=", CLIENT_ID)

| eval uri = concat(DETC_CD, DETC_TIME, TERMINAL_ID, CLIENT_ID)

| eval url = concat(url, urlencode(uri, "UTF-8"))

| wget header=header method=get

