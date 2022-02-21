---
title: On the Fly 로 HLS 시청하기 (기본)
weight: 60
pre: "<b>5-1. </b>"
---

![Diagram](/static/diagram.png)


### MediaConvert 작업 실행 함수
1. [S3 콘솔](https://console.aws.amazon.com/s3) 에 접속합니다.
1. 소스 버킷을 선택합니다.
1. "업로드"를 눌러 동영상 파일을 업로드합니다.
1. [다음 코드](https://github.com/k2sebeom/mediaconvert-on-the-fly/tree/main/demo/player.html) 을 다운받은 후, player.html 을 엽니다.
1. [MediaConvert 콘솔](https://console.aws.amazon.com/mediaconvert) 에 접속하면, 작업이 생성되었음을 확인할 수 있습니다.
1. 작업이 InQueue 에서 Transcoding 으로 변환될 때까지 기다립니다.
1. player.html 에서 "{cloudfront 배포 도메인}/{확장자를 제외한 동영상 파일 이름}/master.m3u8" 을 입력하고, play 를 누르면, 동영상이 재생됩니다. 변환이 진행되면서, 동영상의 총 길이가 점점 늘어남을 확인할 수 있습니다. (영상이 재생되지 않을 경우, 재생 가능할 만큼 충분한 양이 변환되지 않은 것이므로, 조금 더 기다렸다가 play 를 누르는 것을 반복합니다.)


---
<p align="left">
© 2020 Amazon Web Services, Inc. 또는 자회사, All rights reserved.
</p>
