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
빈도: 온디맨드 실행
데이터 베이스: 데이터베이스 추가 버튼을 누르고 prv-dl-db 입력
```
5. 방금 생성한 크롤러를 선택하고 크롤러 실행 버튼을 클릭
![run_glue_aurora_crawler](/prvdlworkshop/images/run_glue_aurora_crawler.png)

6. 등록된 Aurora 데이터 베이스 테이블을 확인
![table_crawl](/prvdlworkshop/images/table_crawl.png)

### Glue를 이용한 데이터 loading 작업
등록되어 있는 Aurora Table에 있는 데이터를 S3로 가져오는 작업을 수행합니다.
1. AWS Glue 콘솔에서 ETL 카테고리 아래에 있는 작업 페이지로 들어갑니다.
![glue_console_job](/prvdlworkshop/images/glue_console_job.png)

2. 작업 추가 버튼을 눌러서 작업을 등록합니다. 아래의 정보를 참조합니다.
```
이름: prv-dl-glue-job-sfct-load-to-s3
IAM 역할: prv-dl-workshop-role
Type: Spark
Glue Version: Spark 2.4, Python 3 with improved job startup times (Glue Version 2.0)
작업 실행 대상: AWS Glue가 생성하여 제안하는 스크립트
스크립트 파일 이름: prv-dl-glue-job-sfct-load-job
데이터 원본: estate_sfct
변환 유형 선택: 스키마 변경
데이터 스토어: Amazon S3
형식: parquet
연결: 연결 추가 버튼을 클릭해서 새로운 S3 연결 생성
연결 이름: prv-dl-connection-to-s3
연결 VPC: Target VPC
연결 서브넷: Target VPC의 private subnet 중 하나
연결 보안 그룹: prv-dl-glue-sg
대상 경로: <이전 Step에서 생성한 S3 bucket의 하위 폴더를 지정>: s3://prv-dl-<USERID>/rawdata
Output Schema Definition: 작업 저장 및 스크립트 편집 버튼 클릭
스키마 편집 화면에서 우측 상단에 있는 X 버튼을 눌러서 창을 닫음
```
3. 등록된 작업을 선택하고 작업 실행 버튼을 클릭합니다.
![run_glue_job](/prvdlworkshop/images/run_glue_job.png)

4. 작업이 완료되면 대상 S3 버킷에서 데이터를 확인합니다.
![s3_raw_data_check](/prvdlworkshop/images/s3_raw_data_check.png)

5. S3로 로딩된 데이터를 Crawler를 이용하여 Glue Catalog에 다시 추가합니다. 크롤러 페이지에서 크롤러 추가 버튼을 클릭합니다.
![glue_aurora_crawler](/prvdlworkshop/images/glue_aurora_crawler.png)

6. 다음의 정보를 참조하여 크롤러를 생성합니다.
```
크롤러 이름: prv-dl-rawdata-crawler
Crawler source type: Data stores
Repeat crawls of S3 data stores: Crawl all folders
데이터 스토어 선택: S3
연결: prv-dl-connection-to-s3
포함경로: s3://prv-dl-<USERID>/rawdata
다른 데이터 스토어 추가: 아니요
IAM 역할: prv-dl-workshop-role
빈도: 온디맨드 실행
데이터 베이스: prv-dl-db
```

7. 방금 생성한 크롤러를 선택하고 크롤러 실행 버튼을 클릭
![run_glue_aurora_crawler](/prvdlworkshop/images/run_glue_aurora_crawler.png)

8. S3로 로딩된 데이터가 Glue Catalog Table에 등록되어 있는 것을 확인
![table_crawl](/prvdlworkshop/images/table_crawl.png)


### Glue Dev endpoint Notebook을 활용한 Dictionary 데이터 매핑
Raw 데이터에 SFCT의 Dictionary 정보를 매핑 시켜서 분석 결과를 직관적으로 이해할 수 있게 변환합니다.
Dictionary 정보는 Kaggle의 Fannie Mae & Freddie Mac Database 2008-2018 페이지에서 pdf 파일로 제공합니다.
작업을 간편하게 하기 위해서 Glue의 DevEndpoint 노트북을 사용했습니다.

1. AWS Glue 콘솔에서 개발 엔드포인트 페이지로 들어간 후 엔드포인트 추가 버튼을 클릭합니다.
![add_glue_dev_ep](/prvdlworkshop/images/add_glue_dev_ep.png)
2. 아래 정보를 참조해서 개발 엔드포인트를 생성합니다.
```
개발 엔드포인트 이름: prv-dl-glue-dev-ep
IAM 역할: prv-dl-workshop-role
네트워킹: VPC, 서브넷 및 보안 그룹 선택
VPC: Target VPC
서브넷: Target VPC의 private subnet 중 하나
보안 그룹: prv-dl-glue-sg
```
3. 노트북 페이지에서 SageMaker 노트북 탭을 선택하고 노트북 생성 버튼을 클릭합니다.
![dev_ep_notebook](/prvdlworkshop/images/dev_ep_notebook.png)
4. 아래 정보를 참조해서 노트북을 생성합니다.
```
노트북 이름: aws-glue-prv-dl-notebook
개발 엔드포인트에 연결: prv-dl-glue-dev-ep
IAM 역할: IAM 역할 생성 선택 후 이름은 끝에 prv-dl-role을 추가합니다.
VPC: Target VPC
서브넷: Target VPC의 private subnet 중 하나
보안 그룹: prv-dl-glue-sg
```
5. Glue notebook Role에 S3에 접근 할 수 있는 권한을 부여하기 위해서 AWS 서비스 콘솔에서 IAM service로 이동합니다.
![mv_iam_glue](/prvdlworkshop/images/mv_iam_glue.png)

6. 검색창에 prv-dl으로 검색해서 방금 생성한 AWSGlueServiceSageMakerNotebookRole-prv-dl-role을 선택합니다.
![glue_nb_role_mod](/prvdlworkshop/images/glue_nb_role_mod.png)

7. 정책 연결 버튼을 눌러서 policy를 추가합니다.
![add_pol_glue_nb](/prvdlworkshop/images/add_pol_glue_nb.png)

8. 검색창에 s3full을 입력해서 AmazonS3FullAccess를 선택하고 옆에 있는 체크박스를 클릭한 뒤 정책 연결 버튼을 클릭합니다.
![add_s3_full_pol](/prvdlworkshop/images/add_s3_full_pol.png)

9. 실습을 진행하고 있는 Labtop PC 웹브라우져의 주소창에 다음의 url을 입력해서 노트북을 다운받습니다.
```
https://private-data-lake-iw2020.s3.ap-northeast-2.amazonaws.com/sfct_dataset/prv-dl-etl-transform.ipynb
```

10. 다시 Glue 서비스 콘솔로 돌아온 뒤 생성한 노트북을 엽니다. 노트북 페이지에서 노트북 열기 버튼을 클릭합니다.
![open_nb](/prvdlworkshop/images/open_nb.png)

11. 다운받았던 prv-dl-etl-transform.ipynb 노트북을 업로드 버튼을 눌러서 올립니다.
![upload_notebook](/prvdlworkshop/images/upload_notebook.png)

12. prv-dl-etl-transform.ipynb을 클릭해서 내용을 확인합니다. 셀의 내용을 확인하면서 차례 차례 실행합니다. 한번에 전부 수행하시려면 메뉴에 있는 Cell 버튼을 눌러서 메뉴를 펼치고 Run All을 클릭합니다.
![run_jupyter_cell](/prvdlworkshop/images/run_jupyter_cell.png)

13. 마지막 부분의 Cell에 있는 S3 bucket명을 현재 실습을 진행하고 있는 버킷으로 바꿔줍니다.
![jupyter_nb_bucket](/prvdlworkshop/images/jupyter_nb_bucket.png)

14. 수행이 완료되면 실습을 진행하고 있는 S3 버킷의 dictionaried_data 폴더에 데이터가 정상적으로 처리되었는지 확인합니다.
![s3_raw_data_check](/prvdlworkshop/images/s3_raw_data_check.png)

15. S3로 로딩된 데이터를 Crawler를 이용하여 Glue Catalog에 다시 추가합니다. 크롤러 페이지에서 크롤러 추가 버튼을 클릭합니다.
![glue_aurora_crawler](/prvdlworkshop/images/glue_aurora_crawler.png)

16. 다음의 정보를 참조하여 크롤러를 생성합니다.
```
크롤러 이름: prv-dl-rawdata-crawler
Crawler source type: Data stores
Repeat crawls of S3 data stores: Crawl all folders
데이터 스토어 선택: S3
연결: prv-dl-connection-to-s3
포함경로: s3://prv-dl-<USERID>/dictionaried_data
다른 데이터 스토어 추가: 아니요
IAM 역할: prv-dl-workshop-role
빈도: 온디맨드 실행
데이터 베이스: prv-dl-db
```

17. 방금 생성한 크롤러를 선택하고 크롤러 실행 버튼을 클릭
![run_glue_aurora_crawler](/prvdlworkshop/images/run_glue_aurora_crawler.png)

18. S3로 로딩된 데이터가 Glue Catalog Table에 등록되어 있는 것을 확인
![table_crawl](/prvdlworkshop/images/table_crawl.png)


---
<p align="center">
© 2020 Amazon Web Services, Inc. 또는 자회사, All rights reserved.
</p>
