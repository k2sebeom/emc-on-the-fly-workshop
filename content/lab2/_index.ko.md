---
title: Lambda 함수 설정하기
weight: 40
pre: "<b>3. </b>"
---

![Diagram](/static/diagram.png)

Lambda 함수를 이용해서 사용자가 동영상을 업로드 했을 때에 MediaConvert 작업을 실행하고, HLS 을 위한 manifest 파일을 작성해줍니다.

### MediaConvert 작업 실행 함수
1. [Lambda 콘솔](https://console.aws.amazon.com/lambda) 에 접속합니다.
1. "함수 생성" 을 선택합니다.
1. "새로 작성"을 선택하고, 함수이름으로 "ConvertHLS" 를 입력합니다.
1. 런타임으로 Python 3.9 를 선택합니다.
1. "함수 생성"을 눌러 함수를 생성합니다.
1. [코드](https://github.com/k2sebeom/mediaconvert-on-the-fly/blob/main/LambdaFunctions/ConvertHLS/lambda_function.py) 를 복사해서 함수의 코드 소스에 붙여넣기 합니다.
1. 코드의 8~11 줄에 있는 변수들을 설정해 주고, Deploy 를 눌러 함수를 배포합니다.
```
JOB_TEMPLATE_NAME = "{MediaConvert Job Template}" # 생성한 Job Template 의 이름 (워크샵에서는 HLSOntheFly)
ROLE_ARN = "{MediaConvert Role Arn}" # IAM Role 의 Arn (워크샵에서는 생성한 MediaConvertRole 의 arn)
SOURCE_BUCKET = "{Source S3 Bucket Name}" # 소스 버킷 이름 (워크샵에서는 emc-on-the-fly-source)
DEST_BUCKET = "{Destination S3 Bucket Name}" # 결과 버킷 이름 (워크샵에서는 emc-on-the-fly-destination)
```
### MediaConvert 작업 실행 함수 트리거 설정하기
1. Lambda 함수의 "트리거 추가" 를 선택합니다. ![LAMBDA1](/static/ko/lambda1.png)
1. S3 를 선택 후, 소스 버킷을 선택합니다. 이하 값들은 기본값으로 설정하고, 트리거를 생성합니다. ![LAMBDA](/static/ko/lambda2.png)

### Manifest 준비 함수
1. [Lambda 콘솔](https://console.aws.amazon.com/lambda) 에 접속합니다.
1. "함수 생성" 을 선택합니다.
1. "새로 작성"을 선택하고, 함수이름으로 "PrepareOntheFly" 를 입력합니다.
1. 런타임으로 Python 3.9 를 선택합니다.
1. "함수 생성"을 눌러 함수를 생성합니다.
1. [코드](https://github.com/k2sebeom/mediaconvert-on-the-fly/blob/main/LambdaFunctions/PrepareOntheFlyHLS/lambda_function.py) 를 복사해서 함수의 코드 소스에 붙여넣기 합니다.
1. Deploy 를 눌러 함수를 배포합니다.

### Manifest 준비 함수 트리거 설정하기
1. Lambda 함수의 "트리거 추가" 를 선택합니다.
1. S3 를 선택 후, 결과 버킷을 선택합니다.
1. 접미사 항목에 "p.m3u8" 을 입력합니다. ![LAMBDA](/static/ko/lambda3.png)
1. 트리거를 생성합니다.

### Lambda 함수 권한 설정하기
1. 두 함수 모두, 구성-권한 을 선택합니다.
1. 역할 이름 밑의 링크를 클릭합니다.
1. 정책 연결을 누른 후, AmazonS3FullAccess 와 AWSElementalMediaConvertFullAccess 정책을 연결합니다. ![LAMBDA](/static/ko/lambda4.png)

---
<p align="center">
© 2020 Amazon Web Services, Inc. 또는 자회사, All rights reserved.
</p>
