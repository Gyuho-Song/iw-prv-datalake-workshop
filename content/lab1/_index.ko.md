---
title: 실습2. 데이터 탐색 및 ETL
weight: 30
pre: "<b>2. </b>"
---

**Glue를 사용하기 위한 환경 설정을 하고 이를 이용해서 데이터를 Loading합니다. 이후에는 Glue Dev Endpoint 노트북을 이용하여 Dictionary 데이터를 매핑시키는 데이터 ETL 작업을 진행합니다.** <br/><br/>

## Table of Contents

1. Glue 실행을 위한 IAM role 생성
2. Glue Security Group 생성
3. Glue를 이용한 데이터 Crawling
4. Glue를 이용한 데이터 loading 작업
5. Glue Dev endpoint Notebook을 활용한 Dictionary 데이터 매핑

### Glue 실행을 위한 IAM role 생성
Glue를 실행하기 위한 IAM role을 생성합니다.

1. AWS 서비스 콘솔에서 IAM service로 이동합니다.
![mv_iam_glue](/prvdlworkshop/images/mv_iam_glue.png)

2. 역할만들기 유형에서 AWS 서비스가 선택되어 있는지 확인합니다.
![iam_glue_service](/prvdlworkshop/images/iam_glue_service.png)

3. 화면 하단에서 Glue를 선택합니다.
![iam_glue_service2](/prvdlworkshop/images/iam_glue_service2.png)

4. 권한 정책 연결에서 다음의 정책들을 선택합니다.
```
AmazonRDSFullAccess
AmazonS3FullAccess
AWSGlueServiceRole
```

5. 역할 이름에 **prv-dl-workshop-role**을 입력하고 생성합니다.
![iam_glue_role_name](/prvdlworkshop/images/iam_glue_role_name.png)

### Glue Security Group 생성
Glue의 데이터 통신을 위한 Security Group 설정을 합니다.
1. AWS 콘솔에서 VPC서비스로 이동한 뒤 Security Group에서 Glue가 사용할 Security Group을 생성합니다.
![glue_sg_create](/prvdlworkshop/images/glue_sg_create.png)
2. **보안 그룹 생성** 버튼을 눌러서 Security Group을 만듭니다. 하단의 정보를 참고합니다.
```
보안 그룹 이름: prv-dl-glue-sg
설명: Security Group for Glue ETL in prv dl workshop
VPC: Target VPC
```
3. 보안 그룹이 생성되면 인바운드 규칙 편집 버튼을 눌러서 self referencing 룰을 추가해줍니다.
방금 생성한 Glue Security Group자신을 소스로하는 모든 트래픽을 허용하는 룰입니다.
![glue_sg_self_referencing](/prvdlworkshop/images/glue_sg_self_referencing.png)

### Glue를 이용한 데이터 Crawling
Glue를 이용해서 데이터를 처리하려면 데이터 소스에 대한 정보를 등록해주는 작업을 선행해야합니다.
1. AWS Glue 서비스 콘솔로 이동한 뒤 연결 페이지로 이동합니다. 연결 추가를 눌러서 Aurora 데이터베이스의 Connection 정보를 등록합니다.
![glue_aurora_connection](/prvdlworkshop/images/glue_aurora_connection.png)
2. 연결 구성에 다음의 정보를 참조합니다.
```
연결 이름: prv-dl-aurora
연결 유형: Amazon RDS
데이터 베이스 엔진: Amazon Aurora
인스턴스: prv-dl-db-cluster-instance-1
데이터베이스 이름: srcdb
사용자 이름: admin
암호: awsiw2020
```
3. 다음으로는 크롤러를 추가해줍니다. 크롤러 페이지에서 크롤러 추가 버튼을 클릭합니다.
![glue_aurora_crawler](/prvdlworkshop/images/glue_aurora_crawler.png)
4. 다음의 정보를 참조하여 크롤러를 생성합니다.
```
크롤러 이름: prv-dl-aurora-crawler
Crawler source type: Data stores
Repeat crawls of S3 data stores: Crawl all folders
데이터 스토어 선택: JDBC
연결: prv-dl-aurora
포함경로: estate/%
다른 데이터 스토어 추가: 아니요
IAM 역할: prv-dl-workshop-role
빈도: 온디맨드 실
데이터 베이스: 데이터베이스 추가 버튼을 누르고 prv-dl-db 입력
```
5. 방금 생성한 크롤러를 선택하고 크롤러 실행 버튼을 클릭
![run_glue_aurora_crawler](/prvdlworkshop/images/run_glue_aurora_crawler.png)




---
<p align="center">
© 2020 Amazon Web Services, Inc. 또는 자회사, All rights reserved.
</p>
