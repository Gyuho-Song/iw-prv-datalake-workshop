---
title: 실습1. HOL 환경 구성
weight: 20
pre: "<b>1. </b>"
---

**실습을 진행할 두개의 VPC를 구성합니다. 각각의 VPC에는 Public subnet과 Private subnet이 있으며 외부 통신을 위한 NAT Gateway가 구성됩니다. 이 단계는 AWS CloudFormation을 이용하여 진행합니다. 이후에는 실습 데이터를 저장할 Aurora Database를 생성하고 서비스간 통신을 위한 Private endpoint를 구성합니다.** <br/><br/>

## Table of Contents

1. VPC 생성
2. Aurora 데이터 베이스 생성
3. Aurora 셋업 및 데이터셋 로드
4. Private Endpoint 설정

### VPC 생성
1. 아래의 링크를 클릭하여 CloudFormation Template을 수행합니다.

2. AWS Console에서 CloudFormation 수행 상태를 확인하고 Deploy가 완료되면 다음 단계로 진행합니다.

3. AWS Console에서 VPC 구성을 확인합니다.

### Aurora 데이터 베이스 생성
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
```

3. 연결 VPC 정보에 특히 유의합니다. (앞 단계에서 CloudFormation으로 생성한 Source VPC를 선택해야함)

![rds_create_vpc](/prvdlworkshop/images/rds_create_vpc.png)

4. 추가 연결 구성 항목을 펼쳐서 VPC 보안 그룹에 CloudFormation에서 미리 생성한 'iw-aurora-'로 시작하는 항목을 선택합니다.

![rds_create_sg](/prvdlworkshop/images/rds_create_sg.png)

5. Amazon Aurora 클러스터가 생성되면 엔드포인트를 따로 복사합니다.

![rds_aurora_ep](/prvdlworkshop/images/rds_aurora_ep.png)

### Aurora 셋업 및 데이터셋 로드
1. Aurora 데이터베이스에 접속하여 작업을 할 EC2 Instance에 ssh로 접속합니다. EC2 정보는 EC2 console에서 확인할 수 있습니다.
 ![ec2_view](/prvdlworkshop/images/ec2_view.png)
```
ssh -i <ssh private key> ec2-user@<instance ip or dns name>
```

2. EC2에 mysql을 설치하고 Aurora에 접속합니다. 아까 복사해둔 Aurora database의 endpoint를 이용합니다.
```
mysql -h <Aurora database endpoint> -P 3306 -u admin -p
```
암호를 묻는 창에는 아까 설정한 'awsiw2020'를 입력합니다.


---

<p align="center">
© 2020 Amazon Web Services, Inc. 또는 자회사, All rights reserved.
</p>
