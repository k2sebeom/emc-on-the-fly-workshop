---
title: 실습4. AWS 기반 데이터 웨어하우징 - Amazon Redshift
weight: 60
pre: "<b>6. </b>"
---

#### Redshift 을 위한 IAM Role 설정
1. AWS Management Console에 로그인 한 뒤 **IAM** 서비스에 접속합니다.
2. **Roles**를 클릭하고 **Create new role**을 클릭합니다.
3. **Select type of trusted entity** 는 **AWS service** 를 선택하고 **Choose the service that will use this role** 은 **Redshift**를 선택합니다. 마지막으로 **Select your use case** 는 **Redshiftcustomizable**을 선택한 후 **Next: Permissions** 버튼을 클릭합니다.
![image01](images/01.png)
4. 생성할 Role에는 두 개의 Policy를 Attach 합니다. **Filter**에 s3 를 입력하고 **AmazonS3FullAccess** 정책을 체크한 후, _다시 Filter에 Glue를 입력하고 **AWSGlueConsoleFullAccess** 정책을 체크한 후_ **Next: Tags**를 클릭 후, 다음 **Next: Review** 버튼을 클릭합니다.
![image02](images/02.png)
5. **Role name**에 **redshift_role**을 입력하고 **Create role**을 클릭하여 IAM Role 생성을 완료합니다. 차후 실습에 사용을 위해 Role ARN을 기록합니다. (arn:aws:iam:: 으로 시작)
![image02](images/03.png)

#### Amazon Redshift 클러스터 생성
실습을 위하여 Redshift 클러스터를 생성합니다. 클러스터는 샘플 데이터가 있는 **us-west-2 (Oregon)** 와 동일한 리전에 생성해야 합니다. 또한 Redshift 의 접속을 위하여 보안 그룹 설정을 주의하여 생성 하시기 바랍니다. <br/>

1. AWS Management Console에서 **Redshift** 서비스에 접속 후 좌측 **Clusters** 탭을 선택 합니다.
2. **Launch cluster** 버튼을 클릭하여 클러스터 생성을 시작합니다.
3. **Cluster identifier**, **Database name**, **Master user name**, **Master user password**, **Confirm password**를 임의대로 차례로 입력한 뒤 **Continue**를 클릭합니다. (Database Port는 Default Port인 5439 사용합니다.)
    * Cluster identifier : redshift
    * Database name : redshift
    * Master user name : admin
    * Password: **** (임의의 Redshift Master용 password 생성)
![image04](images/04.png)
4. Node Configuration 부분은 default 옵션으로 진행합니다. **Continue**를 클릭합니다.
![image05](images/05.png)
5. Additional Configuration에서 **VPC security groups**에는 **default**를 **선택**합니다. **Available roles**에는 이전 단계에 생성한 **redshift_role**을 선택합니다. **Continue**를 클릭합니다.
![image06](images/06.png)
6. Review한 뒤 **Launch cluster**를 선택하여 클러스터를 생성합니다. **Cluster Status**가 **available**이 되면 다음으로 진행합니다.

#### Redshift 접속
이 과정에서는 Amazon Redshift 클러스터에 연결합니다. 저희 실습에서는 Redshift에서 기본으로 제공하는 쿼리 작성기인 Query editor를 이용합니다. **본 도구는 임시적으로 사용하는 도구**이며, 실제로는 JDBC 연결을 통해 **&nbsp; _적절한 데이터베이스 관리도구를 이용_ &nbsp;**하게 됩니다.<br/>

1. Redshift 관리 콘솔에서 **Query editor**를 실행 합니다.
![image07](images/07.png)
2. 다음 설정을 구성합니다.
    * Cluster: redshift
    * Database: redshift
    * Username: admin
    * Password: (이전 단계에서 만든 password 입력)
    * Connect 를 클릭

#### 외부 테이블 생성
이 실습에서는 **외부 테이블**을 생성합니다. 일반 Redshift 테이블과는 달리 외부 테이블은 Amazon S3에 저장된 데이터를 참조합니다. <br/>
먼저 **외부 스키마**를 정의합니다. 외부 스키마는 외부 데이터 카탈로그에 있는 데이터베이스를 참조하고, 클러스터가 사용자 대신 Amazon S3에 액세스할 수 있도록 권한을 부여하는 IAM 역할 식별자(ARN)를 제공합니다. <br/>

1. **&nbsp; _INSERT-YOUR-REDSHIFT-ROLE_ &nbsp;**을 실습 2에서 생성한 redshift_role의 ARN값으로 대체하고 **Query editor**에서 이 명령을 실행합니다. 
``` markup
CREATE EXTERNAL SCHEMA spectrum
FROM DATA CATALOG
DATABASE 'spectrumdb'
IAM_ROLE 'INSERT-YOUR-REDSHIFT-ROLE'
CREATE EXTERNAL DATABASE IF NOT EXISTS
```
**Query editor**의 결과는 별도 정보가 표시되지 않고 "No records found"라는 메시지를 수신합니다. <br/>
Schema "spectrum" already exists라는 메시지를 수신하면, 다음 단계로 진행하십시오. <br/>
이제 spectrum 스키마에 저장될 외부 테이블을 생성합니다.
![image08](images/08.png)

2. **Query editor**에서 이 명령을 실행하여 외부 테이블을 생성합니다. 
``` markup
CREATE EXTERNAL TABLE spectrum.sales(
    salesid INTEGER,
    listid INTEGER,
    sellerid INTEGER,
    buyerid INTEGER,
    eventid INTEGER,
    dateid SMALLINT,
    qtysold SMALLINT,
    pricepaid DECIMAL(8,2),
    commission DECIMAL(8,2),
    saletime TIMESTAMP
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE
LOCATION 's3://id-redshift-apnortheast2/tickit/spectrum/sales/'
TABLE PROPERTIES ('numRows'='172000')
```
**Query editor**에는 아무런 정보가 표시되지 않습니다. 외부 테이블은 테이블의 목록에 표시되지 않기 때문입니다. 
이 문이 Amazon S3에 있는 디렉터리를 가리키는 테이블 정의를 생성했습니다. 디렉터리에는 172,456개의 행이 있는 11MB 텍스트 파일 1개가 포함되어 있습니다. 다음은 파일 콘텐츠 샘플입니다. 
``` markup
2 4 8117 11498 4337 1983 2 76.00 11.40 2008-06-06 05:00:16
6 10 24858 24888 3375 2023 2 394.00 59.10 2008-07-16 11:59:24
7 10 24858 7952 3375 2003 4 788.00 118.20 2008-06-26 00:56:06
8 10 24858 19715 3375 2017 1 197.00 29.55 2008-07-10 02:12:36
```
각 줄에는 수량, 가격 및 판매 날짜와 같은 판매 정보가 있습니다.

#### Amazon S3에 저장된 데이터를 쿼리
이 실습에서는 외부 테이블에 대해 쿼리를 실행합니다. 이 쿼리는 Redshift Spectrum을 사용하여 Amazon S3에서 바로 데이터를 처리합니다.<br/>

1. 다음 명령을 실행하여 S3에 저장된 행의 수를 쿼리합니다. 
``` markup
SELECT COUNT(*) FROM spectrum.sales
```
출력값은 파일에 172,456 개의 레코드가 있음을 보여줍니다.

2. 다음 명령을 실행하여 외부 테이블에 저장된 데이터 샘플을 확인합니다.
``` markup
SELECT * FROM spectrum.sales LIMIT 10
```
S3에 저장된 탭으로 분리된 데이터가 일반 Redshift 테이블과 정확히 동일하게 표시되는 것을 확인할 수 있습니다. Spectrum은 S3에서 데이터를 읽지만 마치 Redshift가 직접 읽는 것처럼 표시합니다.<br/>
또한, 쿼리는 합계 계산과 같은 일반 SQL 문을 포함할 수 있습니다. <br/>
다음 명령을 실행하여 일의 매출을 계산합니다.
``` markup
SELECT SUM(pricepaid)
FROM spectrum.sales
WHERE saletime::date = '2008-06-26'
```

Amazon Redshift Spectrum은 임시 Amazon Redshift 테이블로 데이터를 로드할 필요 없이 Amazon S3에 저장된 데이터에 직접 쿼리를 실행합니다.<br/>
또한, S3에 저장된 데이터와 Amazon Redshift에 저장된 데이터를 조인할 수 있습니다. 이를 보여주기 위해 event라는 일반 Redshift 테이블을 생성하고 이 테이블로 데이터를 로드합니다.

1. 다음 명령을 실행하여 일반 Redshift 테이블을 생성합니다.<br/>
event 테이블이 페이지 왼쪽의 테이블 목록에 표시됩니다.

``` markup
CREATE TABLE event(
eventid INTEGER NOT NULL DISTKEY,
venueid SMALLINT NOT NULL,
catid SMALLINT NOT NULL,
dateid SMALLINT NOT NULL SORTKEY,
eventname VARCHAR(200),
starttime TIMESTAMP
)
```

2. **INSERT-YOUR-REDSHIFT-ROLE**을 이전 단계에서 생성한 redshift_role의 ARN 값을 대체하고 pgweb에서 이 명령을 실행하여 데이터를 events 테이블로 로드합니다. <br/>
약 30초 가량의 로딩 시간이 소요됩니다.

```
COPY event
FROM 's3://id-redshift-apnortheast2/tickit/allevents_pipe.txt'
IAM_ROLE 'INSERT-YOUR-REDSHIFT-ROLE'
DELIMITER '|'
TIMEFORMAT 'YYYY-MM-DD HH:MI:SS'
REGION 'us-west-2'
```

3. 다음 명령을 실행하여 event 데이터의 샘플을 확인합니다.
```
SELECT * FROM event LIMIT 10
```
이제 이 새로운 event 테이블의 데이터 (Redshift 저장 데이터)와 외부 sales 테이블의 데이터 (S3 저장데이터)를 조인하는 쿼리를 실행할 수 있습니다.

4. 다음의 명령을 통해 로컬 event 테이블과 외부 sales 테이블을 조인하여 상위 10개 이벤트의 총 매출을 확인합니다.
```
SELECT TOP 10
    spectrum.sales.eventid,
    SUM(spectrum.sales.pricepaid)
FROM spectrum.sales, event
WHERE spectrum.sales.eventid = event.eventid
    AND spectrum.sales.pricepaid > 30
GROUP BY spectrum.sales.eventid
ORDER BY 2 DESC
```
이 쿼리는 가격이 30 USD 이상의 이벤트 별 (Redshift 저장 데이터) 로 그룹화된 총 매출 (S3 저장 데이터) 을 나열합니다.

5. 다음의 명령을 실행하여 위의 쿼리에 대한 쿼리 플랜을 확인합니다.<br/>
이 쿼리 플랜은 Redshift가 해당 쿼리를 어떻게 실행할 지 보여줍니다. Amazon S3에 있는 데이터에 대해 S3 Seq Scan, S3 HashAggregate 및 S3 Query Scan 단계가 실행됩니다.
```
EXPLAIN
SELECT TOP 10
    spectrum.sales.eventid,
    SUM(spectrum.sales.pricepaid)
FROM spectrum.sales, event
WHERE spectrum.sales.eventid = event.eventid
    AND spectrum.sales.pricepaid > 30
GROUP BY spectrum.sales.eventid
ORDER BY 2 DESC
```

#### 파티션된 데이터 사용
외부 테이블은 디렉터리로 사전에 파티션 될 수 있으며, 각 디렉터리는 데이터 하위집합을 포함합니다.<br/>
데이터를 파티션할 때 파티션 키를 필터링하여 Redshift Spectrum이 스캔하는 데이터의 양을 제한할 수 있습니다.<br/>
시간에 따라 데이터를 파티션하는 것이 일반적입니다. 예를 들어, 년, 월, 일 및 시간에 따라 파티션할 수 있습니다. 데이터가 여러 소스에서 수신되는 경우, 데이터 소스 식별자와 날짜로 파티션할 수 있습니다.<br/>
다음은 분할된 데이터를 보여주는 디렉터리 목록으로, 디렉터리에 월별로 파티션된 S3 파일 집합을 표시합니다. (참고: AWS Cli가 설치된 로컬 머신에서 확인 가능 합니다.)

```
$ aws s3 ls s3://id-redshift-apnortheast2/tickit/spectrum/sales_partition/
```

```
PRE saledate=2008-01/
PRE saledate=2008-02/
PRE saledate=2008-03/
PRE saledate=2008-04/
PRE saledate=2008-05/
PRE saledate=2008-06/
PRE saledate=2008-07/
PRE saledate=2008-08/
PRE saledate=2008-09/
PRE saledate=2008-10/
PRE saledate=2008-11/
```

이제 이 데이터를 사용하는 외부 테이블을 정의 합니다.<br/>
다음 명령을 실행하여 파티션된 데이터에 따라 새로운 sales_partitioned 테이블을 정의합니다.

```
CREATE EXTERNAL TABLE spectrum.sales_partitioned(
    salesid INTEGER,
    listid INTEGER,
    sellerid INTEGER,
    buyerid INTEGER,
    eventid INTEGER,
    dateid SMALLINT,
    qtysold SMALLINT,
    pricepaid DECIMAL(8,2),
    commission DECIMAL(8,2),
    saletime TIMESTAMP
)
PARTITIONED BY (saledate DATE)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '|'
STORED AS TEXTFILE
LOCATION 's3://id-redshift-apnortheast2/tickit/spectrum/sales_partition/'
TABLE PROPERTIES ('numRows'='172000')
```

(이 쿼리를 실행하면 화면에 응답이 표시되지 않지만, 테이블 정의가 생성됩니다.)<br/>
salesdate 필드에 따라 테이블이 파티션됨을 Redshift Spectrum에 알려주는 문이 추가되었습니다.<br/>
그런 다음 Redshift Spectrum은 기존 파티션에 대한 정보를 받아야 어떤 디렉터리를 사용할 지 알 수 있습니다.<br/>
다음 명령을 실행하여 파티션을 추가합니다.

```
ALTER TABLE spectrum.sales_partitioned
ADD PARTITION (saledate='2008-01-01')
LOCATION 's3://id-redshift-apnortheast2/tickit/spectrum/sales_partition/saledate=2008-01/';
ALTER TABLE spectrum.sales_partitioned
ADD PARTITION (saledate='2008-02-01')
LOCATION 's3://id-redshift-apnortheast2/tickit/spectrum/sales_partition/saledate=2008-02/';
ALTER TABLE spectrum.sales_partitioned
ADD PARTITION (saledate='2008-03-01')
LOCATION 's3://id-redshift-apnortheast2/tickit/spectrum/sales_partition/saledate=2008-03/';
ALTER TABLE spectrum.sales_partitioned
ADD PARTITION (saledate='2008-04-01')
LOCATION 's3://id-redshift-apnortheast2/tickit/spectrum/sales_partition/saledate=2008-04/';
ALTER TABLE spectrum.sales_partitioned
ADD PARTITION (saledate='2008-05-01')
LOCATION 's3://id-redshift-apnortheast2/tickit/spectrum/sales_partition/saledate=2008-05/';
ALTER TABLE spectrum.sales_partitioned
ADD PARTITION (saledate='2008-06-01')
LOCATION 's3://id-redshift-apnortheast2/tickit/spectrum/sales_partition/saledate=2008-06/';
ALTER TABLE spectrum.sales_partitioned
ADD PARTITION (saledate='2008-07-01')
LOCATION 's3://id-redshift-apnortheast2/tickit/spectrum/sales_partition/saledate=2008-07/';
ALTER TABLE spectrum.sales_partitioned
ADD PARTITION (saledate='2008-08-01')
LOCATION 's3://id-redshift-apnortheast2/tickit/spectrum/sales_partition/saledate=2008-08/';
ALTER TABLE spectrum.sales_partitioned
ADD PARTITION (saledate='2008-09-01')
LOCATION 's3://id-redshift-apnortheast2/tickit/spectrum/sales_partition/saledate=2008-09/';
ALTER TABLE spectrum.sales_partitioned
ADD PARTITION (saledate='2008-10-01')
LOCATION 's3://id-redshift-apnortheast2/tickit/spectrum/sales_partition/saledate=2008-10/';
ALTER TABLE spectrum.sales_partitioned
ADD PARTITION (saledate='2008-11-01')
LOCATION 's3://id-redshift-apnortheast2/tickit/spectrum/sales_partition/saledate=2008-11/';
ALTER TABLE spectrum.sales_partitioned
ADD PARTITION (saledate='2008-12-01')
LOCATION 's3://id-redshift-apnortheast2/tickit/spectrum/sales_partition/saledate=2008-12/';
```

이제 특정 salesdate를 사용하는 모든 쿼리에서 해당 날짜와 관련된 디렉터리만 스캔합니다.<br/>
비교를 위해 2개의 서로 다른 데이터 소스에 쿼리를 실행합니다.<br/>

1. 원래 sales 테이블에 다음 명령을 실행하고 실행에 걸리는 시간을 기록합니다.
```
    SELECT TOP 10
    spectrum.sales.eventid,
    SUM(pricepaid)
    FROM spectrum.sales, event
    WHERE spectrum.sales.eventid = event.eventid
    AND pricepaid > 30
    AND date_trunc('month', saletime) = '2008-12-01'
    GROUP BY spectrum.sales.eventid
    ORDER BY 2 DESC
```

2. 파티션된 데이터에 다음의 명령을 실행하고 실행에 걸리는 시간을 기록합니다.
```
    SELECT TOP 10
    spectrum.sales_partitioned.eventid,
    SUM(pricepaid)
    FROM spectrum.sales_partitioned, event
    WHERE spectrum.sales_partitioned.eventid = event.eventid
    AND pricepaid > 30
    AND saledate = '2008-12-01'
    GROUP BY spectrum.sales_partitioned.eventid
    ORDER BY 2 DESC
```
두번째 쿼리가 더 빠르게 실행되는 것을 확인합니다. 이는 Amazon S3에서 읽는 데이터가 더 적기 때문입니다. 데이터 볼륨이 클수록 실행 속도의 차이가 더 분명해 집니다. (다만 본 예제와 같이 데이터량이 작은 경우 그 차이는 미비합니다.) 또한, Amazon S3에서 읽는 데이터량에 따라 Redshift Spectrum에 대한 요금이 부과되므로, 쿼리 실행 비용도 줄어듭니다.<br/>

파티션에 대한 정보는 SVV_EXTERNAL_PARTITIONS 시스템 뷰에서 확인할 수 있습니다.<br/>
1. 다음의 명령을 실행하여 sales_partitioned 테이블에 대한 파티션을 확인합니다.
```
    SELECT *
    FROM SVV_EXTERNAL_PARTITIONS
    WHERE tablename = 'sales_partitioned'
```
<br/>
2. External tables에 대한 정보는 _SVV_EXTERNAL_COLOUMS 시스템 뷰에서 확인할 수 있습니다.<br/>
3. 다음의 명령을 실행하여 sales_partitioned 테이블에 정의된 열을 확인합니다.

#### 실습 완료
이번 실습을 완료하였습니다. 비용 발생을 최소화 하기 위하여 실습환경을 정리하십시오.

---
<p align="center">
© 2019 Amazon Web Services, Inc. 또는 자회사, All rights reserved.
</p>
