# <카카오페이 서버 개발과제>

## contents
1. 분석설계
2. 개발SW 리스트 상세
3. 테스트 시나리오
* 참고) 테이블 DDL
* 참고) 카카오과제 원 내용

## 1. 분석설계
##### 1) 요구사항 분석 : 통합 바코드 발급, 포인트 적립/사용, 기간별 내역 조회에 대한 각 API개발
#####	- 요구사항 정리
	    ㄱ) 사용자와 멤버십 정책 -> 사용자:멤버십바코드 = 1:1, 사용자 탈퇴는 없음
	    ㄴ) 가맹점과 업종소속 -> 업종분류(A/B/C)에 각 속한 상점(가맹점)이 존재하며, 하나의 상점은 한업종만 속함
	    ㄷ) 포인트 사용 정책 -> 포인트는 각 상점에서 적립 후 동종업종의 상점에서만 사용가능, 
	       가족/친구와 한 멤버십바코드 동시사용 가능
##### 2) 요구기능 설계
#####	- API 기능 정의
	    ㄱ) 통합 바코드 발급 -> key-column : 사용자id-int(9) 또는 바코드Varchar(10), 
	       기등록 유저 발급시도시 기존 바코드 리턴
	    ㄴ) 포인트 적립 -> 상점id/바코드/적립금액 요청으로 해당상점에 속한 업종에 적립하며,
	       미등록 상점과 바코드로 요청시 각 해당 오류정보로 처리 
	    ㄷ) 포인트 사용 -> 상점id/바코드/사용금액 요청으로 해당상점에 속한 업종 잔액을 확인 후 사용처리하며, 
	        포인트부족, 미등록 상점id와 바코드는 각 해당 오류정보로 처리
	    ㄹ) 내역 조회 -> 시작기간/종료기간/멤버십바코드 요청으로 사용시기, 구분(적립,사용), 상점명, 업종정보를 
	       필수로 리스트 리턴하며, 조회조건의 바코드가 미등록일 경우 해당 메시지를 포함하여 처리
	    ㅁ) 오류 처리 -> exception처리를 공통 처리하며, 각 예외 상황에 맞는 메세지 반환
##### 3) 테이블 리스트 및 설계 : 대상 테이블 3개 (통합바코드마스터, 상점마스터, 포인트적립사용이력)
##### 4) 개발 프레임웍/환경 결정: Java1.8, Spring boot, Eclipse
##### 5) 단위/통합 테스트 방법 설정 : Swagger-ui

## 2. 개발 SW 리스트 상세
	* IDE : Eclipse IDE Version: 2022-09 (4.25.0)
	* Language : Java (openjdk version "1.8.0_332")
	* Framework : spring-boot 2.3.9.RELEASE
	* WAS : Embeded Tomcat
	* DB : h2 Database
	* Test : Springfox Swagger ui 3.0
	* 기타 : gradle-7.5.1, Mybatis 2.1.4, lombok

## 3. 테스트 시나리오
##### 1) git 소스 clone 및 gladle refresh, source build
##### 2) rest api URI테스트 boot starter 후 http://localhost:8085/swagger-ui/ 접속
##### 3) Membership서비스 확장하여, example 값 참고하여, 통합바코드발급->포인트적립->포인트사용->포인트내역조회 순서로 테스트해본다.
   
■    ㄱ) POST /v1/memship/new     -> 통합바코드생성 (신규회원)

```
		정상 Input : x-user-id(header)에 숫자 9자로 넣고 Request body는 그대로 비워두고 실행한다. 
		            Responses의 아이디 입력값과 바코드번호를 확인
		오류 exception : x-user-id(header)에 9자리 이상 숫자를 넣어 본다 (공통사항)
		
		Header
		{"userId": "123456789"}
		Request body
		{}
		Responses
		{
		  "success": "true",
		  "data": {
		    "userId": 123456789,
		    "barcodeNo": "7714462582"
		  }
		}
```


■    ㄴ) POST /v1/memship/earn    -> 포인트적립 (상점이 소속한 업종별)

```
		정상 Input : x-user-id(header)에 숫자 9자로 넣고 Request body에
		            barcodeNo, partner, pointAmt를 채운후 실행한다.
		오류 exception : 멤버십바코드 또는 상점id에 가짜 입력값을 넣어 실행해본다.            
		            
		Header
		{"userId": "123456789"}
		Request body
		{
		  "barcodeNo": "7714462582",
		  "partner": "F",
		  "pointAmt": 1000
		}
		Request body
		{
		  "success": "true",
		  "data": {
		    "partner": "F",
		    "barcodeNo": "7714462582",
		    "pointAmt": 1000,
		    "typeCd": "EARN"
		  }
		}
```

■    ㄷ) POST /v1/memship/use     -> 포인트사용 (업종별 잔액내)

```
		정상 Input : x-user-id(header)에 숫자 9자로 넣고 Request body에
		            barcodeNo, partner, pointAmt를 채운후 실행한다.
		Header
		{"userId": "123456789"}
		Request body
		{
		  "barcodeNo": "7714462582",
		  "partner": "F",
		  "pointAmt": 1000
		}
		Request body
		{
		  "success": "true",
		  "data": {
		    "partner": "F",
		    "barcodeNo": "7714462582",
		    "pointAmt": 1000,
		    "typeCd": "USE"
		  }
		}
```

■    ㄹ) POST /v1/memship/hist    -> 포인트내역 (기간내 포인트 적립사용 내역)

```
		정상 Input : x-user-id(header)에 숫자 9자로 넣고 Request body에
		            barcodeNo, endAt, startAT를 채운후 실행한다.
		Header
		{"userId": "123456789"}
		Request body
		{
		  "barcodeNo": "7714462582",
		  "endAt": "20221231",
		  "startAt": "20221201"
		}
		Responses
		{
		  "success": "true",
		  "data": {
		    "pointCnt": 25,
		    "memshipPointList": [
		      {
		        "approvedAt": "2022-12-12 01:03:55.159",
		        "typeCd": "EARN",
		        "partnerNm": "F마트",
		        "categoryNm": "식품",
		        "pointAmt": 1000,
		        "barcodeNo": "7714462582"
		      },
		   ]
		}
 ```

##### 3) DB확인 - H2 접속 후 http://localhost:8085/h2 
##### (Server모드, jdbc url -> jdbc:h2:./data/memshipdb, ID/PW : happypapa/happypapa)




#### 참고) 테이블 DDL
##### 1) 통합바코드
```
         CREATE TABLE "PUBLIC"."MEMSHIP_BARCODE_TM" (
            "BARCODE" VARCHAR(10) NOT NULL COMMENT '바코드번호',
            "JOINED_AT" DATETIME COMMENT '바코드등록일시',
            "APPROVED_ID" INT(9) COMMENT '사용자',
            "FIRST_CRET_AT" DATETIME COMMENT '최초등록일시',
            "FIRST_CRET_ID" INT(9) COMMENT '최초등록자',
            "LAST_CHNG_AT" DATETIME COMMENT '마지막변경일시',
            "LAST_CHNG_ID" INT(9) COMMENT '마지막변경자'
         ) 
```

##### 2) 상점마스터
```
         CREATE TABLE "PUBLIC"."MEMSHIP_PARTNER_TM" (
            "PARTNER" VARCHAR(20) NOT NULL COMMENT '상점코드',
            "PARTNER_NM" VARCHAR(40) NOT NULL COMMENT '상점명',
            "CATEGORY" VARCHAR(20) NOT NULL COMMENT '업종유형코드',
            "CATEGORY_NM" VARCHAR(40) NOT NULL COMMENT '업종유형명',
            "FIRST_CRET_AT" DATETIME COMMENT '최초등록일시',
            "FIRST_CRET_ID" INT(9) COMMENT '최초등록자',
            "LAST_CHNG_AT" DATETIME COMMENT '마지막변경일시',
            "LAST_CHNG_ID" INT(9) COMMENT '마지막변경자'
         ) 
```
##### 3) 포인트 적립/사용 이력
```
         CREATE TABLE "PUBLIC"."MEMSHIP_POINT_TH" (
            "BARCODE" VARCHAR(10) NOT NULL COMMENT '바코드번호',
            "TYPE_CD" VARCHAR(1) NOT NULL COMMENT 'use:1,earn:0 구분',
            "POINT_AMT" INT(9) NOT NULL COMMENT '포인트금액',
            "APPROVED_AT" DATETIME COMMENT '승인일시',
            "USER_ID" INT(9) COMMENT '사용자',
            "FIRST_CRET_AT" DATETIME COMMENT '최초사용일시',
            "FIRST_CRET_ID" INT(9) COMMENT '최초등록 사용자',
            "LAST_CHNG_AT" DATETIME COMMENT '마지막변경일시',
            "LAST_CHNG_ID" INT(9) COMMENT '마지막변경자'
         ) 
```


***
## 참고) 카카오과제 원 내용
***

```
##멤버십 서비스
        1. 서비스 설명
- 카카오페이에는 멤버십 서비스가 있습니다. 해당 서비스를 만들어봅시다.
- 멤버십은 사용자 별로 하나의 멤버십 바코드를 발급하고 있습니다.
- 상점에서 포인트 적립 또는 사용을 할 수 있습니다.
- 상점은 상점명과 하나의 업종 정보를 가지고 있습니다.
- 같은 업종의 가맹점들은 적립된 포인트를 같이 쓸 수 있습니다.
- 업종정보는 다음과 같이 3가지가 존재합니다. (A: 식품, B : 화장품, C : 식당)
- 하나의 상점은 하나의 업종 정보만 가지고 있고 변경 될수 없습니다.
- 사용자 탈퇴는 없으며, 발급된 바코드는 계속 유효합니다.
- 발급된 멤버십바코드는 가족이나 친구끼리 공유가 가능합니다.
        2. 요구사항
- 아래 API들을 구현 합니다.
1. 통합 바코드 발급 API
2. 포인트 적립 API
3. 포인트 사용 API
4. 기간별 내역 조회 API
- 작성하신 어플리케이션이 다수의 서버에 다수의 인스턴스로 동작하더라도 기능에 문제가 없도록
설계되어야 합니다.
- 멤버십바코드는 공유가 가능해서 동일 바코드에 의한 사용, 적립 요청이 동시간에 들어올 경우도
고려해야 합니다.
- 각 기능 및 제약사항에 대한 단위테스트를 반드시 작성합니다.

        3. API 설명
1) 통합 바코드 발급 API
- 요청값은 사용자 id가 포함됩니다.
- 사용자 id는 9자리 숫자, 멤버십 바코드는 10자리 숫자형 스트링을 사용합니다.
- 발급된 멤버십 바코드는 다른 사람과 중복될수 없습니다.
- 다음번 발급될 멤버십 바코드가 예측 가능해서도 안됩니다.
- 이미 발급된 id의 발급 요청이 올 경우 기존 멤버십 바코드를 반환합니다.
2) 포인트 적립 API
- 요청값은 상점 id, (1)번 API로 발급한 바코드, 적립금이 포함됩니다.
- 포인트 적립은 상점의 업종별로 통합하여 적립됩니다.
- 등록되지 않은 상점 id 경우 등록되지 않은 상점 오류를 돌려줍니다.
- 등록되지 않은 멤버십 바코드의 경우 등록되지 않은 멤버십 바코드 오류를 돌려줍니다.
3) 포인트 사용 API
- 요청값은 상점 id, (1)번 API로 발급한 바코드, 사용금액이 포함됩니다.
- 포인트 사용은 요청한 상점의 업종별 통합 적립금에서 사용됩니다.(B업종 정립금이 있더라도 요청
상점이 A업종이라고 사용할수 없습니다.)
- 포인트 사용요청 시 적립 금액을 초과하는 사용의 경우 포인트 부족으로 사용할수 없다는 오류를
돌려줍니다.
- 등록되지 않은 상점 id 경우 등록되지 않은 상점 오류를 돌려줍니다.
- 등록되지 않은 멤버십 바코드의 경우 등록되지 않은 멤버십 바코드 오류를 돌려줍니다.
4) 내역 조회 API
- 요청값은 시작기간, 종료기간, 멤버십 바코드가 포함되어야 합니다.
- 응답값에는 사용시기, 구분(적립, 사용), 상점명, 업종정보등이 포함되어야 합니다.
- 등록되지 않은 바코드의 경우 등록되지 않은 멤버십 바코드 오류를 돌려줍니다.
Ex)
{"history":[{"approved_at":"2022-04-12
10:10:10","type":"use","category":"A","partner_name":"F마트"},{"approved_at":"2022-04-12
20:10:10","type":"earn","category":"A","partner_name":"Z마트"}]}
기술 제약사항
- 개발 언어는 Java, Kotlin 중 익숙한 개발 언어를 선택하여 과제를 진행해주시면 됩니다.
- 설계 내용과 설계의 이유, 핵심 문제해결 전략 및 분석한 내용을 작성하여 "readme.md" 파일에
첨부 해주세요.
- 데이터베이스 사용에는 제약이 없습니다.
- API 의 HTTP Method들 (GET | POST | PUT | DEL) 은 자유롭게 선택하세요.
- 에러응답, 에러코드는 자유롭게 정의해주세요.
평가항목
- 프로젝트 구성 방법 및 관련된 시스템 아키텍쳐 설계 방법이 적절한가?
- 요구사항을 잘 이해하고 구현하였는가?
- 작성한 어플리케이션 코드의 가독성 좋고 의도가 명확한가?
- 작성한 테스트 코드는 적절한 범위의 테스트를 수행하고 있는가? (예. 유닛/통합 테스트 등)
- 어플리케이션은 다량의 트래픽에도 무리가 없도록 효율적으로 작성되었는가
```