# ✈️ AirplaneService
>+ 항공편 간편 예매 서비스 프로그램

## 📋 프로그램 소개
>- 사용자가 항공편을 쉽게 예약할 수 있게 지원하는 프로그램
<br>

### 🕐 개발 기간
>- 2024/11/28 ~ 2024/12/01

### 🏘️ 개발 환경
>- Java 21
>- Oracle IDE 23
>- Eclipse

### 🧰핵심 기능
>- 로그인 시 __관리자__ 및 __일반회원__ 구분 및 그에 따른 기능 구분
>
><br>
>
>   - __관리자__ 테이블
>       - __CUSTOMER__ 목록, 생성, 수정, 삭제, 검색 기능 제공
>       - __PLANE__목록, 생성, 수정, 삭제, 검색 기능 제공
>       - __COUNTRY__목록, 생성, 수정, 삭제, 검색 기능 제공
>       - __FLIGHT__목록, 생성, 수정, 삭제, 검색 기능 제공
>   - __고객__ 테이블
>       - __예매__   : 사용자가 항공편 예약을 쉽게 할 수 있는 시스템 구현
>       - __내정보__ : 수정 및 삭제 기능 제공
>       - __항공편__ : 권한을 부여받지 못한 객체에게 예매 가능 한 항공편 목록을 제공
>
><br>
>
>- MVC 아키텍처를 통한 코드 구조화

## 💠 ERD
>![](https://github.com/munjungbae/airPlaneService/blob/main/Relational.png)

### 📝 주요 기능
- 예약 시 고객 예약 합계 증가 및 삭제 시 감소
```
CREATE OR REPLACE TRIGGER COUNT_UP_TRG
AFTER INSERT ON BOOKING
FOR EACH ROW
BEGIN
    UPDATE CUSTOMER SET C_COUNT=C_COUNT+1 WHERE NO = :NEW.CUSTOMER_NO;
END;
/
```
```
CREATE OR REPLACE TRIGGER COUNT_DOWN_TRG
AFTER DELETE ON BOOKING
FOR EACH ROW
BEGIN
    UPDATE CUSTOMER SET C_COUNT=C_COUNT-1 WHERE NO = :OLD.CUSTOMER_NO;
END;
/
```
- 도착지 거리대비 도착시간 자동 계산
```
CREATE OR REPLACE TRIGGER FLIGHT_ARRIVAL_TIRRGER
BEFORE INSERT OR UPDATE ON FLIGHT
FOR EACH ROW
DECLARE
    SHOUR NUMBER(4.2);
BEGIN
    SELECT HOUR INTO SHOUR FROM COUNTRY C WHERE C.NO = :NEW.ARRIVAL_COUNTRY_NO;
    :NEW.ARRIVAL_HOUR := :NEW.DEPARTURE_HOUR + (SHOUR/24);
END;
/
```
- 항공기 추가 시 좌석 자동 생성
```
CREATE OR REPLACE TRIGGER PLANE_INSERT_TRG
AFTER INSERT ON PLANE
FOR EACH ROW
BEGIN
    FOR i IN 1 .. :NEW.ROWX LOOP
        FOR j IN 1 .. :NEW.COLY LOOP
            INSERT INTO SEATS VALUES (LPAD(PLANE_SEQ.NEXTVAL, 6, '0'), :NEW.NO, LPAD(i, 2, '0'), LPAD(j, 2, '0'));
        END LOOP;
    END LOOP;
END;
/
```
- 비행거리별 비행 소요 시간 계산
```
CREATE OR REPLACE TRIGGER COUNTRY_HOUR_TRG
BEFORE INSERT OR UPDATE ON COUNTRY
FOR EACH ROW
BEGIN
    -- 평균 속도 900 km/h
    :NEW.HOUR := :NEW.DISTANCE / 900;
END;
/
```
- PLANE 추가시에 SEATS TABLE 자동으로 추가
```
CREATE OR REPLACE TRIGGER PLANE_INSERT_TRG
    AFTER INSERT 
    ON PLANE
    FOR EACH ROW
DECLARE
    X1 NUMBER;
    X2 NUMBER;
BEGIN
    FOR i IN 1 .. :NEW.ROWX LOOP--FOR IN 구문으로 SEATS에 값 추가 ASCCII 코드를 사용해서 A...Z 반환
    FOR j IN 1 .. :NEW.COLY LOOP
    X1 :=ASCII('A')+ i/26; 
    X2 :=ASCII('A')+ MOD(i, 26);
    INSERT INTO SEATS VALUES (TO_CHAR((SELECT NVL(MAX(NO),0)+1 FROM SEATS),'FM000000'),:NEW.NO, CHR(X1)||CHR(X2), TO_CHAR(j,'FM00'));
    END LOOP;
    END LOOP;
END;
/
```
- 예약 시 좌석에 맞춰서 좌석번호를 업데이트
```
CREATE OR REPLACE TRIGGER BOOKING_SEATS_TRG
BEFORE INSERT OR UPDATE
ON BOOKING
FOR EACH ROW
DECLARE
SNO CHAR(6);
BEGIN
    SELECT NO INTO SNO FROM SEATS WHERE ROWX=SUBSTR(:NEW.SEAT,1,2) AND COLY=SUBSTR(:NEW.SEAT,3) AND PLANE_NO = 
    (SELECT PLANE_NO 
     FROM FLIGHT
     WHERE NO = :NEW.FLIGHT_NO);
     :NEW.SEATS_NO:=SNO;
END;
/
```
- 예약정보 입력 시 코드 자동 생성
```
CREATE OR REPLACE TRIGGER BOOKING_INSERT_TRG
BEFORE INSERT
ON BOOKING
FOR EACH ROW
BEGIN
:NEW.CODE:=:NEW.GROUP_NO||'-'||:NEW.CUSTOMER_NO||'-'||:NEW.FLIGHT_NO;
END;
/
```

### 📝실행화면

- 로그인 화면

 ![login](https://github.com/user-attachments/assets/9cfdd855-2c08-496a-9478-eb34d884700a)
 
- __관리자__ 메뉴 고객 정보 및 기체 정보

![customer, plane](https://github.com/user-attachments/assets/918322b8-d32d-43e2-aba0-91f51851ee51)

- __관리자__ 메뉴 각 나라 및 항공편 정보

![country, flight](https://github.com/user-attachments/assets/8a9ca208-922b-4365-8c5e-086eff18df05)

- __고객__ 메뉴 예매 및 예매 과정

![CUSTOMER BOOKING](https://github.com/user-attachments/assets/bb50e043-2c54-4b11-a2f9-abfe7ea3a214)

- __고객__ 메뉴 예매 정보 확인 및 예매 가능 목록

![CUSTOMER BOOKING LIST, LIST](https://github.com/user-attachments/assets/f89280a7-2adf-44b4-b1d9-7f3f89c4e188)

