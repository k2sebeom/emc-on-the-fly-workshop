---
title: Elemental MediaConvert 설정하기
weight: 30
pre: "<b>2. </b>"
---


![Diagram](/static/diagram.png)

Elemental MediaConvert 를 이용해서 사용자가 업로드한 영상을 HLS 형식으로 변환할 것입니다. 설정들을 Job Template 으로 구성하여 여러번 사용할 수 있도록 할 것입니다.

### MediaConvert 를 위한 IAM 역할 생성
1. [IAM 콘솔](https://console.aws.amazon.com/iam) 에 접속합니다.
1. 좌측 탭에서 "역할" 을 선택합니다.
1. "역할 만들기"를 선택합니다.
1. AWS 서비스 - MediaConvert 를 선택합니다.
1. 이하 설정은 기본값 그대로 계속 다음으로 이동합니다.
1. 역할 이름으로 "MediaConvertRole" 을 입력한 후 "역할 만들기" 를 선택합니다.


### Job Template 구성하기
1. [MediaConvert 콘솔](https://console.aws.amazon.com/mediaconvert) 에 접속합니다.
1. 좌측 탭에서 "작업 템플릿" 을 선택합니다.
1. "템플릿 생성" 을 선택합니다.
1. 이름으로 "HLSOntheFly" 를 입력합니다.
1. "출력 그룹" 옆의 "추가" 버튼을 누릅니다. ![EMC1](/static/ko/emc1.png)
1. "Apple HLS" 를 선택합니다. ![EMC2](/static/ko/emc2.png)
1. 생성된 Apple HLS 출력그룹을 선택한 후, 스크롤을 내려 "출력" 항목으로 이동합니다.
1. ABR 을 위해서 3가지 해상도로 출력을 할 것입니다. "출력 추가" 를 눌러 출력을 두 개 추가하고, 알맞은 이름 한정자를 설정해 줍니다. 이후에 Lambda 에서 이름 한정자를 이용하기 떄문에 270p, 360p, 720p, 1080p 중에서 이름 한정자를 선택합니다. 워크샵에서는 270p, 360p, 720p 를 선택합니다. ![EMC3](/static/ko/emc3.png)
1. 각각의 출력을 설정해 줍니다. 직접 설정해 줄 수도 있지만, 편의를 위해서 워크샵에서는 프리셋을 사용합니다. 출력을 선택한 후, 알맞은 프리셋을 적용합니다. ![EMC4](/static/ko/emc4.png)
1. "생성"을 눌러 템플릿을 생성합니다.

---
<p align="center">
© 2020 Amazon Web Services, Inc. 또는 자회사, All rights reserved.
</p>
