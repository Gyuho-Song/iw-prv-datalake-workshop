---
title: 실습3. LakeFormation을 활용한 권한 설정
weight: 40
pre: "<b>3. </b>"
---

**이전 단계에서 처리한 데이터 중 민감한 데이터 컬럼에 대한 권한을 LakeFormation을 이용하여 제어합니다. 본 실습에서는 인종 정보가 포함된 borrower_race컬럼들과 co_borrower_race 컬럼들을 민감한 데이터라고 가정했습니다.** <br/><br/>

![phase2_archi](/prvdlworkshop/images/phase2_archi.png)

## Table of Contents

1. RedShift Spectrum이 사용할 Role 생성
2. LakeFormation으로 Redshift Spectrum에서 접근 가능한 컬럼의 권한 설정
3. Redshift 클러스터 생성
4. RedShift 접속 환경 설정
5. RedShift Spectrum을 이용한 데이터 loading

### RedShift Spectrum이 사용할 Role 생성
IAM 서비스에서 RedShift Spectrum이 사용할 Role을 생성합니다.

1. IAM 서비스 콘솔로 이동합니다.  
![mv_iam_glue](/prvdlworkshop/images/mv_iam_glue.png)

2. 역할만들기 유형에서 AWS 서비스가 선택되어 있는지 확인합니다.
![iam_glue_service](/prvdlworkshop/images/iam_glue_service.png)

3. 화면 하단에서 Redshift를 선택합니다. 사용 사례는 Redshift - Customizable 선택합니다.
![redshift_iam_role](/prvdlworkshop/images/redshift_iam_role.png)

4. 권한 정책 연결에서 다음의 정책들을 선택합니다.
```
AmazonRedshiftFullAccess
AmazonS3FullAccess
```

5. 역할 이름에 *prv-dl-redshift-role*을 입력하고 역할을 생성합니다.

6. 생성한 role의 역할 ARN값을 복사해둡니다.

### LakeFormation으로 Redshift Spectrum에서 접근 가능한 컬럼의 권한 설정
이전 단계에서 생성한 rawdata table에 있는 컬럼중에 Redshift Spectrum이 접근할 수 있는 컬럼들에 대한 권한 설정을 합니다.
인종 정보가 포함된 borrower_race컬럼들과 co_borrower_race 컬럼들을 제외한 컬럼들을 선택합니다.

1. AWS Lake Formation 서비스 콘솔로 이동합니다.
![aws_lf_console](/prvdlworkshop/images/aws_lf_console.png)

2. Permissions 항목 하단에 있는 Data permissions 페이지로 이동하여 Grant 버튼을 클릭합니다.
![lf_grant](/prvdlworkshop/images/lf_grant.png)

3. Grant permissions 페이지에서 다음의 정보를 참조하여 입력하고 Grant 버튼을 클릭합니다..
```
IAM users and roles: prv-dl-redshift-role
Database: prv-dl-db
Table: dictionaried_data
Columns: include Columns
include columns: borrower_race1~5, co_borrower_race1~5를 제외한 모든 컬
Table permissions: Select만 선택
```

### Redshift 클러스터 생성
1. RedShift 서비스 콘솔로 이동합니다.
![mv_rs_console](/prvdlworkshop/images/mv_rs_console.png)

2. 클러스터 생성 전에 Private subnet에 Redshift를 배포하기 위한 서브넷 그룹을 생성합니다.
구성 항목 하단에 있는 서브넷 그룹을 선택하여 클러스터 서브넷 그룹 페이지로 이동한 뒤 클러스터 서브넷 그룹 생성 버튼을 클릭합니다.
![cluster_subnet_group](/prvdlworkshop/images/cluster_subnet_group.png)

3. 클러스터 서브넷 그룹 생성 페이지가 나오면 다음의 정보를 참조하여 서브넷 그룹을 생성합니다.
```
이름: prv-dl-private-subnet-group
설명: RedShift private subnet group for prv dl workshop
VPC: Target VPC
가용영역 a에 있는 서브넷 중 Private subnet을 선택
가용영역 c에 있는 서브넷 중 Private subnet을 선택
```

4. 이제 Redshift 클러스터를 만듭니다. 클러스터 생성 버튼을 누르고 다음의 정보를 참조하여 Redshift 클러스터를 생성합니다.
```
클러스터 식별자: prv-dl-dw-cluster
마스터 사용자 이름: awsuser
마스터 사용자 암호: awsiw2020
데이터 베이스 이름: prv-dl-mart
데이터 베이스 포트: 5439
추가 구성: 기본 값 사용을 해제해서 수정이 가능하게 변경
VPC: Target VPC
VPC 보안 그룹: prv-dl-redshift-sg
클러스터 서브넷 그룹: prv-dl-private-subnet-group
향상된 VPC 라우팅: 활성화
```

### RedShift 접속 환경 설정
실습 환경에서 VPC의 Private 서브넷에 있는 Redshift 클러스터에 접속하기 위해서는 public subnet에 있는 bastion host VM을 통해서 접속해야합니다. 하지만 이 환경을 on-premise 데이터 센터로 구성하실 경우에는 Private IP 주소를 이용하여 바로 접속하시면 되기 때문에 bastion host 세팅이 필요하지 않습니다.

1. AWS EC2 콘솔로 이동하여 *prv-dl-target-vpc-bastion* 이라는 이름의 인스턴스를 살펴봅니다.
![mv_ec2_rs_jb](/prvdlworkshop/images/mv_ec2_rs_jb.png)

2. 아래쪽에 있는 속성 창에 있는 보안 탭을 클릭한 뒤 보안 그룹에 있는 해당 보안 그룹을 클릭합니다.
![mv_rs_jb_sg](/prvdlworkshop/images/mv_rs_jb_sg.png)

3. Redshift 접속이 가능하려면 Redshift의 보안 그룹에서 bastion host에서 오는 트래픽을 허용하도록 설정이 되어있어야하고 bastion host의 보안그룹에서는 bastion host와 redshift 위치한 VPC의 CIDR block에서 들어오는 트래픽이 허용되도록 설정이 되어 있는 것이 좋습니다.
정상적으로 설정이 되어있는지 확인하고 창을 닫습니다.
![review_rs_sg](/prvdlworkshop/images/review_rs_sg.png)

4. SQL Workbench/J를 다운받고 설치합니다.
다운로드 링크: [SQL-Workbench/J](https://www.sql-workbench.eu/downloads.html)

5. AWS 공식 홈페이지에서 Redshift jdbc 드라이버를 다운받습니다.

    안내 페이지 링크: [Amazon Redshift jdbc guide](https://docs.aws.amazon.com/ko_kr/redshift/latest/mgmt/configure-jdbc-connection.html)

    다운로드 링크: [Amazon Redshift jdbc driver download](https://s3.amazonaws.com/redshift-downloads/drivers/jdbc/1.2.47.1071/RedshiftJDBC42-no-awssdk-1.2.47.1071.jar)

6. 다운받은 Jdbc 드라이버를 SQL workbench tool에 등록합니다.
![redshift_jdbc_driver](/prvdlworkshop/images/redshift_jdbc_driver.png)

7. 이제 Target VPC의 Bastion host를 이용해서 SSH 터널링을 합니다.
```
ssh -i .ssh/<your private ssh key> -L 5439:<Redshift cluster endpoint - schema 제외>:5439 ec2-user@<bastion host public ip>

예: ssh -i .ssh/prv-dl.pem -L 5439:prv-dl-dw-cluster.cfrje0yghpil.ap-northeast-2.redshift.amazonaws.com:5439 ec2-user@3.34.126.248
```

8. 만들어진 터널을 통해서 SQL Workbench/J를 Redshift cluster에 연결합니다. Tool의 왼쪽 위에 있는 Create new connection profile 버튼을 누르고 새 연결 프로파일을 생성합니다. 생성 시 다음의 정보를 참조합니다.
```
Profile Name: prv-dl-redshift
Dirver: 앞에서 등록한 AWS Redshift JDBC 드라이버(com.amazon.redshift.jdbc.Driver)
url: jdbc:redshift://127.0.0.1:5439/prv-dl-mart
Username: awsuser
Password: awsiw2020
```
화면의 ssh 버튼을 눌러서 나오는 ssh 연결 설정은 다음의 정보를 참조합니다.
```
SSH hostname: <Bastion host ip>
username: ec2-user
Private key file: <your private ssh key 경로>
Local port: 5439
DB hostname: <Redshift cluster endpoint - schema 제외>
DB port: 5439
```

Test 버튼을 눌러서 연결을 테스트 해봅니다. 연결이 정상적일 때에도 처음에 Test를 해보면 연결이 안될 때가 있으니 두번정도 해보시는 것을 추천드립니다.

9. Ok 버튼을 눌러서 Workbench에 접속합니다.
![ok_sqlworkbench](/prvdlworkshop/images/ok_sqlworkbench.png)

### RedShift Spectrum을 이용한 데이터 loading
연결이 완료되면 Redshift spectrum을 이용해서 S3에 저장된 데이터를 대상으로 쿼리합니다. 마트 테이블 생성은 다음 단계에서 진행하고 여기서는 쿼리가 정상적으로 되는지만 확인합니다.

1. 현재 RedShift Schema에 있는 테이블 목록을 확인합니다. 쿼리 결과에 아무것도 나오지 않는 것을 확인합니다.
```
select distinct(tablename) from pg_table_def where schemaname = 'public';
```

2. RedShift Spectrum을 통해서 S3에 저장되어 있는 데이터를 쿼리할 수 있게 Glue Data Catalog에 있는 테이블을 등록합니다.
```
create external schema prvdl
from data catalog database 'prv-dl-db'
iam_role '<이전 단계에서 생성했던 prv-dl-redshift-role의 역할 ARN값>'
```

3. Redshfit Spectrum으로 쿼리해서 Lake Formation에서 권한을 허용해준 컬럼들에 대한 결과들이 정상적으로 출력되는지 확인합니다.
```
select * from prvdl.dictionaried_data limit 20;
```
![rs_spec_initial](/prvdlworkshop/images/rs_spec_initial.png)

### 실습4에서 계속

---
<p align="center">
© 2020 Amazon Web Services, Inc. 또는 자회사, All rights reserved.
</p>
