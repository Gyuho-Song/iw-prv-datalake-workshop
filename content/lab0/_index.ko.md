---
title: 실습1. HOL 환경 구성
weight: 20
pre: "<b>1. </b>"
---

**실습을 진행할 두개의 VPC(Source와 Target)를 구성합니다. Source VPC는 고객사의 On-premise를 시뮬레이션한 VPC이고 Target VPC는 Private DataLake로 사용할 VPC입니다. 각각의 VPC에는 Public subnet과 Private subnet이 있으며 외부 통신을 위한 NAT Gateway가 구성됩니다. 이 단계는 AWS CloudFormation을 이용하여 진행합니다. 이후에는 실습 데이터를 저장할 Aurora Database를 생성하고 서비스간 통신을 위한 Private endpoint를 구성합니다.** <br/><br/>

## Table of Contents

1. 실습 데이터 셋 정보
2. VPC 생성
3. Aurora 데이터 베이스 생성
4. Aurora 셋업 및 데이터셋 로드
5. Private Endpoint 설정

### 실습 데이터셋 정보
Kaggle - Fannie Mae & Freddie Mac Database 2008-2018
[Original dataset link](https://www.kaggle.com/jeromeblanchet/fannie-mae-freddie-mac-public-use-database)

본 실습에서는 이 중 Single Family Properties Census Tract file 2013 ~ 2018 데이터셋만을 사용했습니다.
![dataset_desc](/prvdlworkshop/images/dataset_desc.png)

### VPC 생성
실습을 진행할 VPC를 생성하고 필요한 환경설정을 진행합니다. 미리 준비된 CloudFormation을 이용하여 배포합니다.
1. 아래의 링크를 클릭하여 CloudFormation Template을 수행합니다.

2. AWS Console에서 CloudFormation 수행 상태를 확인하고 Deploy가 완료되면 다음 단계로 진행합니다.

3. AWS Console에서 VPC 구성을 확인합니다.

### Aurora 데이터 베이스 생성
원천 데이터를 저장해놓을 Aurora 데이터베이스를 생성합니다.
1. Amazon RDS 페이지로 이동합니다.
![rds_console](/prvdlworkshop/images/rds_console.png)

2. 다음 정보를 참조해서 데이터 베이스를 생성합니다.
```
데이터베이스 생성 방식 선택 : 표준 생성
엔진 옵션 : Amazon Aurora
에디션 : MySQL과 호환되는 Amazon Aurora
용량 유형: 프로비저닝됨
버전 : Aurora (MySQL 5.7) 2.07.0 이상
DB 클러스터 식별자 : prv-dl-db-cluster
마스터 사용자 이름 : admin
마스터 암호 : awsiw2020
연결 VPC 정보 : 이전 단계에서 생성한 Source VPC
데이터 베이스 인증 옵션 : 암호 인증
추가 구성 > 초기 데이터베이스 이름: srcdb
```

3. 연결 VPC 정보에 특히 유의합니다. (앞 단계에서 CloudFormation으로 생성한 Source VPC를 선택해야함)

![rds_create_vpc](/prvdlworkshop/images/rds_create_vpc.png)

4. 추가 연결 구성 항목을 펼쳐서 VPC 보안 그룹에 CloudFormation에서 미리 생성한 'iw-aurora-'로 시작하는 항목을 선택합니다.

![rds_create_sg](/prvdlworkshop/images/rds_create_sg.png)

5. Amazon Aurora 클러스터가 생성되면 엔드포인트를 따로 복사합니다.

![rds_aurora_ep](/prvdlworkshop/images/rds_aurora_ep.png)

### Aurora 셋업 및 데이터셋 로드
실습에 사용할 데이터셋을 다운받아서 Aurora 데이터베이스로 적재합니다.
1. Aurora 데이터베이스에 접속하여 작업을 할 EC2 Instance에 ssh로 접속합니다. EC2 정보는 EC2 console에서 확인할 수 있습니다. Source VPC에 있는 EC2에 접속합니다.
![ec2_view](/prvdlworkshop/images/ec2_view.png)
```
ssh -i <ssh private key> ec2-user@<instance ip or dns name>
```

2. AWS cli 설정을 합니다.
```
aws configure
```

3. 실습에서 사용할 S3 버킷을 생성합니다.
```
aws s3 mb s3://prv-dl-<USERID> --region ap-northeast-2
```

4. 실습에서 사용할 데이터를 다운로드 하고 압축을 풉니다.
```
wget "https://private-data-lake-iw2020.s3.ap-northeast-2.amazonaws.com/sfct_dataset/SFCT.tar.gz"
tar xvfz SFCT.tar.gz
```

5. EC2에 mysql을 설치하고 Aurora에 접속합니다. 아까 복사해둔 Aurora database의 endpoint를 이용합니다.
```
sudo yum install mysql
mysql -h <Aurora database endpoint> -P 3306 -u admin -p
```
암호를 묻는 창에는 아까 설정한 'awsiw2020'를 입력합니다.

6. Database schema와 user를 생성합니다.
```
CREATE DATABASE estate;

CREATE USER 'user1'@'%' IDENTIFIED BY 'password1';

GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, RELOAD, PROCESS, REFERENCES, INDEX, ALTER, SHOW DATABASES, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, REPLICATION SLAVE, REPLICATION CLIENT, CREATE VIEW, SHOW VIEW, CREATE ROUTINE, ALTER ROUTINE, CREATE USER, EVENT, TRIGGER ON *.* TO 'user1'@'%' WITH GRANT OPTION;
```

7. Table을 생성하고 데이터를 Table로 load합니다.
```
USE estate;

CREATE TABLE SFCT (
ent_flag int,
record_num int,
us_pos_code int,
msa_code int,
country_code int,
census_tract int,
minority_pct float,
median_income int,
local_med_income int,
income_ratio float,
borrower_income int,
area_med_income int,
borrower_income_ratio float,
upb int,
loan_purpose int,
federal_guarntee int,
num_borrower int,
first_buyer int,
borrower_race1 int,
borrower_race2 int,
borrower_race3 int,
borrower_race4 int,
borrower_race5 int,
borrower_ethnicity int,
co_borrower_race1 int,
co_borrower_race2 int,
co_borrower_race3 int,
co_borrower_race4 int,
co_borrower_race5 int,
co_borrower_ethnicity int,
borrower_gender int,
co_borrower_gender int,
age_borrower int,
age_co_borrower int,
occupancy_code int,
rate_spread float,
hoepa_status int,
property_type int,
lien_status int
);

LOAD DATA LOCAL INFILE '/home/ec2-user/SFCT/2013/fhlmc_sf2013c_loans.txt' INTO TABLE SFCT
FIELDS TERMINATED BY ' '
OPTIONALLY ENCLOSED BY '"';

LOAD DATA LOCAL INFILE '/home/ec2-user/SFCT/2013/fnma_sf2013c_loans.txt' INTO TABLE SFCT
FIELDS TERMINATED BY ' '
OPTIONALLY ENCLOSED BY '"';

LOAD DATA LOCAL INFILE '/home/ec2-user/SFCT/2014/fhlmc_sf2014c_loans.txt' INTO TABLE SFCT
FIELDS TERMINATED BY ' '
OPTIONALLY ENCLOSED BY '"';

LOAD DATA LOCAL INFILE '/home/ec2-user/SFCT/2014/fnma_sf2014c_loans.txt' INTO TABLE SFCT
FIELDS TERMINATED BY ' '
OPTIONALLY ENCLOSED BY '"';

LOAD DATA LOCAL INFILE '/home/ec2-user/SFCT/2015/fhlmc_sf2015c_loans.txt' INTO TABLE SFCT
FIELDS TERMINATED BY ' '
OPTIONALLY ENCLOSED BY '"';

LOAD DATA LOCAL INFILE '/home/ec2-user/SFCT/2015/fnma_sf2015c_loans.txt' INTO TABLE SFCT
FIELDS TERMINATED BY ' '
OPTIONALLY ENCLOSED BY '"';

LOAD DATA LOCAL INFILE '/home/ec2-user/SFCT/2016/fhlmc_sf2016c_loans.txt' INTO TABLE SFCT
FIELDS TERMINATED BY ' '
OPTIONALLY ENCLOSED BY '"';

LOAD DATA LOCAL INFILE '/home/ec2-user/SFCT/2016/fnma_sf2016c_loans.txt' INTO TABLE SFCT
FIELDS TERMINATED BY ' '
OPTIONALLY ENCLOSED BY '"';

LOAD DATA LOCAL INFILE '/home/ec2-user/SFCT/2017/fhlmc_sf2017c_loans.txt' INTO TABLE SFCT
FIELDS TERMINATED BY ' '
OPTIONALLY ENCLOSED BY '"';

LOAD DATA LOCAL INFILE '/home/ec2-user/SFCT/2017/fnma_sf2017c_loans.txt' INTO TABLE SFCT
FIELDS TERMINATED BY ' '
OPTIONALLY ENCLOSED BY '"';

LOAD DATA LOCAL INFILE '/home/ec2-user/SFCT/2018/fhlmc_sf2018c_loans.txt' INTO TABLE SFCT
FIELDS TERMINATED BY ' '
OPTIONALLY ENCLOSED BY '"';

LOAD DATA LOCAL INFILE '/home/ec2-user/SFCT/2018/fnma_sf2018c_loans.txt' INTO TABLE SFCT
FIELDS TERMINATED BY ' '
OPTIONALLY ENCLOSED BY '"';

```
8. Table에 저장된 데이터를 살펴봅니다. Dictionary 데이터를 매핑하기 전이기 때문에 모든 값이 숫자형으로 되어 있는 것을 확인할 수 있습니다.
```
select * from SFCT limit 10;
```



### Private Endpoint 설정
AWS의 내부망을 사용하여 데이터를 전송하는데 꼭 필요한 Private Endpoint들을 생성합니다. S3와 Glue의 엔드포인트가 필요합니다.
1. AWS Console에서 VPC Service로 이동합니다.
![mv_vpc_ep](/prvdlworkshop/images/mv_vpc_ep.png)

2. 엔드포인트 항목을 선택하여 S3 endpoint를 만듭니다. **주의!** VPC 선택창에서 실습에서 사용할 **Target VPC**를 꼭 선택하셔야 합니다.
![create_s3_ep](/prvdlworkshop/images/create_s3_ep.png)

3. Glue endpoint도 만듭니다. 마찬가지로 VPC 선택창에서 실습에서 사용할 **Target VPC**를 꼭 선택하셔야 합니다.
![create_glue_ep](/prvdlworkshop/images/create_glue_ep.png)

### 실습2에서 계속

---

<p align="center">
© 2020 Amazon Web Services, Inc. 또는 자회사, All rights reserved.
</p>
