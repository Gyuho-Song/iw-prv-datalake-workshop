---
title: 실습3. LakeFormation을 활용한 권한 설정
weight: 40
pre: "<b>3. </b>"
---

**이전 단계에서 처리한 데이터 중 민감한 데이터 컬럼에 대한 권한을 LakeFormation을 이용하여 제어합니다. 본 실습에서는 인종 정보가 포함된 borrower_race컬럼들과 co_borrower_race 컬럼들을 민감한 데이터라고 가정했습니다.** <br/><br/>

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
추가 구성: 기본 값 사용을 해제해서 수정이 가능하게 변경
VPC: Target VPC
VPC 보안 그룹: prv-dl-redshift-sg
클러스터 서브넷 그룹: prv-dl-private-subnet-group
향상된 VPC 라우팅: 활성화
```

### RedShift 접속 환경 설정

---
<p align="center">
© 2020 Amazon Web Services, Inc. 또는 자회사, All rights reserved.
</p>
