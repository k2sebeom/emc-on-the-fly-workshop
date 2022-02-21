---
date: 2020-03-05
title: Elemental MediaConvert 를 이용한 On-the-Fly HLS 배포
weight: 20
chapter: true
---

# Elemental MediaConvert 를 이용한 On-the-Fly HLS 배포

---

![Diagram](/static/diagram.png)

Elemental MediaConvert 는 비디오를 다양한 포맷으로 변환할 수 있는 서비스입니다. MediaConvert 의 주 사용 목적 중 하나는 업로드한 비디오를 HLS 형식으로 변환하여 이후에 VOD로 제공하기 위한 것입니다. 하지만, 길이가 긴 비디오를 업로드 하였을 경우, 변환에 긴 시간이 걸리기 때문에, 변환 작업이 진행되는 중에 HLS 비디오를 시청하고 싶을 수 있습니다. 이 워크샵에서는 On-the-Fly 로 Elemental MediaConvert 를 이용하여 변환이 진행되는 동안 영상을 시청할 수 있는 간단한 아키텍쳐를 구성해 보고, ABR 기능을 이용하여 사용자의 네트워크 환경에 따라 적합한 해상도의 영상을 송출할 수 있게 하는 설정을 구성하게 됩니다.

---
<p align="center">
© 2020 Amazon Web Services, Inc. 또는 자회사, All rights reserved.
</p>
