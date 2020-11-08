---
title: 실습4. 데이터 분석 마트 생성 및 시각화
weight: 50
pre: "<b>4. </b>"
---

**RedShift Spectrum을 이용해서 쿼리한 결과를 CTAS 구문을 이용하여 마트 테이블 형태로 Redshift에 저장합니다. 분석 마트 테이블은 생애 첫 주택 구입 대출자의 마트 분포, 대출 기관별 통계, 대출 목적, 투자 목적 주택 구입의 지역별 통계의 총 네개 입니다. 생성한 데이터 마트를 Amazon QuickSight를 이용하여 시각화합니다.** <br/><br/>

![phase3_archi](/prvdlworkshop/images/phase3_archi.png)

## Table of Contents

1. RedShift Spectrum을 이용하여 분석 마트 테이블 생성
2. QuickSight 환경 설정
3. QuickSight를 이용한 시각화 분석
4. QuickSight 대시 보드 생성

### RedShift Spectrum을 이용하여 분석 마트 테이블 생성
SQL Workbench를 통하여 분석 마트 테이블 생성 쿼리를 수행하여 마트 테이블을 생성합니다.

1. first_buyer_age_mart라는 이름으로 생애 첫 주택 구입 대출자의 마트 분포에 대한 분석 마트 테이블을 생성하고 결과를 확인합니다.
```
create table first_buyer_age_mart as
SELECT age_borrower, count(first_buyer) as first_buyer_age_cnt
from prvdl.dictionaried_data
where first_buyer = 'Yes'
group by age_borrower
order by age_borrower;

select * from first_buyer_age_mart
order by age_borrower;
```

2. loan_gurantee_mart라는 이름으로 대출 기관별 통계에 대한 분석 마트 테이블을 생성하고 결과를 확인합니다.
```
create table loan_gurantee_mart as
select federal_guarntee,lien_status,avg(borrower_income) as borrower_income_avg
from prvdl.dictionaried_data
group by federal_guarntee,lien_status;

select * from loan_gurantee_mart;
```

3. loan_mart라는 이름으로 대출 목적에 대한 분석 마트 테이블을 생성하고 결과를 확인합니다.
```
create table loan_mart as
select loan_purpose, count(*) as loan_purpose_cnt
from prvdl.dictionaried_data
group by loan_purpose;

select * from loan_mart;
```

4. speculation_area_mart라는 이름으로 투자 목적 주택 구입의 지역별 통계에 대한 분석 마트 테이블을 생성하고 결과를 확인합니다.
```
create table speculation_area_mart as
select country_code,count(*) as speculation_area_cnt ,avg(area_med_income) as area_med_income
from prvdl.dictionaried_data
where occupancy_code = 'Investment property'
group by country_code
order by speculation_area_cnt desc;

select * from speculation_area_mart;
```

### QuickSight 환경 설정
Redshift 클러스터에 저장되어 있는 데이터 마트 테이블을 private 통신 환경으로 로딩하기 위한 네트워크 설정을 하고 데이터 시각화를 용이하게 하기 위해서 데이터를 spice 영역으로 가져오는 설정을 합니다.

1. Amazon QuickSight 서비스 콘솔로 이동합니다.
![mv_qs_console](/prvdlworkshop/images/mv_qs_console.png)

2. 오른쪽 상단에 있는 프로필 아이콘을 누르고 QuickSight관리 버튼을 클릭합니다.
![mv_qs_manager](/prvdlworkshop/images/mv_qs_manager.png)

3. 메뉴에서 VPC 연결 관리 버튼을 눌러서 VPC 연결 관리 페이지로 이동한 뒤 VPC 연결 추가 버튼을 클릭합니다.
![qs_add_vpc_connection](/prvdlworkshop/images/qs_add_vpc_connection.png)

4. VPC 연결 추가 페이지에서 다음의 정보를 참조하여 VPC 연결을 생성합니다.
```
VPC 연결 이름: prv-dl-quicksight-connection
VPC ID: <Target VPC>
보안 그룹 ID: <EC2 콘솔에서 CloudFormation으로 미리 생성된 prv-dl-quicksight-sg를 찾아서 ID를 복사>
```

5. QuickSight 메인화면으로 나온 뒤 왼쪽에 있는 메뉴에서 데이터 세트 버튼을 클릭
![mv_qs_dataset](/prvdlworkshop/images/mv_qs_dataset.png)



<p align="center">
© 2020 Amazon Web Services, Inc. 또는 자회사, All rights reserved.
</p>
