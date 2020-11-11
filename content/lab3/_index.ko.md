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

5. QuickSight 메인화면으로 나온 뒤 왼쪽에 있는 메뉴에서 데이터 세트 버튼을 클릭한 뒤 오른쪽 상단에 있는 새 데이터 세트 버튼을 누릅니다.
![mv_qs_dataset](/prvdlworkshop/images/mv_qs_dataset.png)

6. 다음 화면에서 Redshift(수동 연결)를 누르고 다음의 정보를 참조하여 데이터 소스를 등록합니다.
![add_rs_in_qs1](/prvdlworkshop/images/add_rs_in_qs1.png)
```
데이터 원본 이름: prv-dl-data-mart
연결 유형: prv-dl-quicksight-connection
데이터 베이스 서버: <Redshift cluster endpoint - schema 제외>
포트: 5439
데이터 베이스 이름: prv-dl-mart
사용자 이름: awsuser
암호: awsiw2020
```

7. 위의 정보를 채우고 데이터 원본 생성 버튼을 누릅니다.
![create_origin_dataset](/prvdlworkshop/images/create_origin_dataset.png)


8. 테이블 선택 화면이 나오면 *first_buyer_age_mart*를 선택하고 다음 화면에서 *더 빠른 분석을 위해 SPICE로 가져오기*를 선택한 뒤 Visualize 버튼을 누릅니다.
![first_dataset_complete](/prvdlworkshop/images/first_dataset_complete.png)

9. 다음 화면에서 왼쪽에 있는 필드 목록 하단에 있는 두개의 필드를 모두 선택한 뒤 드래그 하여 오른쪽에 있는 캔버스로 끌어놓습니다.
시각적 객체 유형에서는 누적 막대 콤보 차트를 선택합니다.
![first_visual_chart1](/prvdlworkshop/images/first_visual_chart1.png)

10. 화면 상단의 필드 모음에서 막대 부분을 클릭하여 펼칩니다. 막대에서 age_borrower(합계)를 제거합니다.
![first_visual_chart2](/prvdlworkshop/images/first_visual_chart2.png)

11. 화면 상단의 필드 모음에서 X축 부분을 클릭하여 펼칩니다. X축에서 정렬 기준을 오름차순으로 선택하고 age_borrower를 선택합니다.
첫번째 차트가 완성되었습니다.
![first_visual_chart3](/prvdlworkshop/images/first_visual_chart3.png)

12. 다시 새 데이터 세트 버튼을 클릭하고 하단에 있는 prv-dl-data-mart를 선택합니다.
![create_second_dataset](/prvdlworkshop/images/create_second_dataset.png)

13. 위에서의 절차를 반복하여 이번에는 *loan_gurantee_mart*를 선택하고 다음 화면에서 *더 빠른 분석을 위해 SPICE로 가져오기*를 선택한 뒤 Visualize 버튼을 누릅니다. 다음 화면에서 왼쪽에 있는 필드 목록 하단에 있는 세개의 필드를 모두 선택합니다.
![second_visual_chart1](/prvdlworkshop/images/second_visual_chart1.png)

14. 시각적 객체 유형에서는 가로 막대 차트를 선택하고 위쪽에 있는 필드 모음에서는 Y축에 federal_guarntee, 값에는 borrower_income_avg, 그룹/색상에는 lien_status를 선택합니다.
![second_visual_chart2](/prvdlworkshop/images/second_visual_chart2.png)

15. 위와 같은 방식으로 나머지 loan_gurantee_mart와 speculation_area_mart를 시각화 해보시기 바랍니다.

### 실습 5에서 계속


<p align="center">
© 2020 Amazon Web Services, Inc. 또는 자회사, All rights reserved.
</p>
