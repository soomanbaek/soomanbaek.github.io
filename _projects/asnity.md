---
title: Asnity
description: 커뮤니티, 채널이 있는 실시간 채팅 서비스
image: asnity.png
date: 2023-04-28
---

- 개발 기간: 6주 (22.11 ~ 22.12)
- 개발 인원: 4명 (FE 2명, BE 2명)
- [깃허브](https://github.com/boostcampwm-2022/web24-Asnity)
- [데모 영상](https://www.youtube.com/watch?v=2gI3OlJXAZQ)
- [배포 사이트](https://asnity.site/sign-in)
- **Architecture**

  ![1](https://user-images.githubusercontent.com/72093196/234808355-68db6aa7-c5e2-42ab-b59a-008915430195.png)

- **기술 스택**

  ![2](https://user-images.githubusercontent.com/72093196/234808351-74d45cf3-00b4-486d-8b65-26af27cff399.png)

---
## 🛠 기능 소개
### 친구 관리
- 닉네임 혹은 아이디를 통한 사용자 검색과 유저간 팔로우가 가능합니다.
- 팔로잉과 팔로워 조회가 가능합니다.
![1](https://user-images.githubusercontent.com/72093196/234809970-07504562-597f-4772-bf72-d8795f26adad.gif)

### 커뮤니티 별 채널을 통한 Live Chatting
- private 채널, public 채널 기능이 있습니다.
- 커뮤니티의 채널에 안 읽은 메세지 알림 기능이 있습니다.
- 관리자와 사용자간의 권한을 차별화 합니다.
![2](https://user-images.githubusercontent.com/72093196/234809964-88624b86-cd6f-4b00-9315-b8e00bd20e40.gif)

### 안 읽은 채팅 실시간 확인
- 오랫동안 채널에 접속하지 않은 사용자를 위해 어디서부터 채팅을 읽지 않았는지 표시해줍니다.
- 다른 채팅방의 새로운 채팅도 실시간으로 알림 표시 해줍니다.
  ![3](https://user-images.githubusercontent.com/72093196/234809957-19e7b5e2-b551-4215-a2ab-16f80ab9db1b.gif)

---
## 🛠 My work

### 1. 안 읽은 메시지 위치 찾기
>

<img width="662" alt="스크린샷 2023-04-27 오후 5 42 14" src="https://user-images.githubusercontent.com/72093196/234808590-10368892-ce84-4efc-b1ff-83e032f84408.png">

안 읽은 메시지 위치를 찾기 위해서 채널의 메시지의 생성 시간과 사용자가 채널에 마지막 방문한 시간을 비교합니다.

이 때 메시지를 하나씩 전부 비교하는 것은 비효율적이어서 다음과 같은 방법으로 개선하였습니다.
(안 읽은 메시지 위치까지 총 M개의 메시지를 탐색해야 한다고 가정하겠습니다.)

1. 메시지를 100개씩 하나의 도큐먼트로 저장하였습니다.

   메시지 100개씩 하나의 도큐먼트로 저장하고 첫번째 메시지의 생성시간과 마지막 메시지의 생성시간을 저장합니다. 사용자의 마지막 방문 시간이 첫번째 메시지 생성시간과 마지막 메시지 생성시간 사이에 존재하면 안 읽은 메시지가 도큐먼트 내에 있다는 것을 알 수 있습니다.

   이렇게 구현하면 탐색 횟수를 `M` 에서 `M/100 + M%100` 으로 줄일 수 있습니다.

2. 이진탐색을 활용하였습니다.

   도큐먼트에 안 읽은 메시지가 있다는 것을 알게되면, 도큐먼트 내의 100개의 메시지를 전부 확인하지 않고 이진탐색을 활용하도록 구현하였습니다.(메시지는 시간순으로 순차적으로 저장되어 있음) 이진탐색을 활용하면 100개의 메시지를 탐색하는데 `log_2(100) ≒ 6` 의 시간이 걸립니다.

   이렇게 구현하면 탐색 횟수를 `M/100 + M%100` 에서 `M/100 + 6` 으로 줄일 수 있습니다.


> **안 읽은 메시지 위치를 찾기 위해 고민한 내용은 [여기서](https://velog.io/@soomanbaek/%EC%95%88-%EC%9D%BD%EC%9D%80-%EB%A9%94%EC%8B%9C%EC%A7%80-%EB%A1%9C%EC%A7%81) 확인할 수 있습니다.**

### 2. JWT 기반의 인증방식을 활용한 회원가입, 로그인 기능

JWT 방식의 인증 강화 방식인 Access Token과 Refresh Token을 사용하였습니다.

보안을 고려하여 Access Token은 클라이언트 in-memory에 저장하고, Refresh Token은 Cookie에 저장하였습니다.

Access Token을 CSRF 및 XSS 공격에 안전하도록 관리하기 위해 in-memory에 저장하였습니다. 하지만, 브라우저 새로고침 시 Access Token이 사라지는 단점이 있습니다. 이 단점은 Silent Token Refresh를 적용하여 Access Token이 사라져도 사용자는 다시 로그인 하지 않고도 Refresh Token을 통해 재발급 받을 수 있도록 구현하였습니다.

Refresh Token은 새로고침을 해도 사라지지 않도록 Cookie에 저장하였습니다. Cookie에 저장하면서 보안을 강화하기 위해 Secure, HttpOnly, Path 옵션을 설정하였습니다. Refresh Token이 탈취 당할 시 보안에 위험이 있기 때문에 유효한 Refresh Token인지 확인하는 로직을 추가하였습니다.

> **로그인을 구현하기 위해 고민한 내용은 [여기서](https://velog.io/@soomanbaek/%EB%A1%9C%EA%B7%B8%EC%9D%B8-%EC%83%81%ED%83%9C-%EC%9C%A0%EC%A7%80) 확인할 수 있습니다.**

### 3. 채널, 채팅 API 및 테스팅

채널, 채팅과 관련된 API를 구현하였습니다.
- [채널 API](https://documenter.getpostman.com/view/24645741/2s8YzZNyKS)   [채팅 API](https://documenter.getpostman.com/view/24645741/2s8YzZNyKV)
- public, private 채널 접속 권한 차별화
  - public 채널일 경우 커뮤니티의 모든 사용자들이 접근할 수 있고 private 채널일 경우 초대받은 사용자만 접근할 수 있습니다.
  - 관리자 권한 차별화
    - 채널의 정보를 수정할 수 있는 권한은 해당 채널의 관리자 또는 채널이 속한 커뮤니티의 관리자만 할 수 있습니다. 
테스팅
- Jest와 supertest를 활용하여 통합 테스팅을 하였습니다.

---
## 🪴 배운점

1. 기획, 개발, 배포까지 전부 하면서 웹 개발의 전체적인 흐름을 알 수 있었습니다.
  - [기획 과정](https://velog.io/@soomanbaek/Asnity-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-%EA%B0%9C%EC%9A%94)
2. Github Actions를 활용한 CI/CD, 에러 발생 시 Slack 알림 등 개발자를 편리하게 하는 기능들을 적극 이용하였습니다.
3. 협업을 위한 Github 사용법을 배울 수 있었습니다.
  - Github branch 전략, PR 보내고 코드리뷰를 받는 등 Github를 적극 활용하여 협업하였습니다.
4. 소통 능력을 기를 수 있었습니다.
  - [데일리 스크럼](https://www.notion.so/4d62a33565de42c28808f25441b2af62), [주간 회고](https://www.figma.com/file/xh06hDpiwmuvBiBSuftn3I/%EC%A3%BC%EC%B0%A8%EB%B3%84-%ED%9A%8C%EA%B3%A0(KPT)?node-id=0-1&t=eE7q31JVWSCkj6QR-0) 를 진행하였습니다.
  - 의견을 제시할 때 근거를 들며 깊이 있는 소통을 하려고 노력하였습니다. ([예시](https://www.notion.so/DB-2-6e36f848caa24effbb4101f0a8ee79a9))
  - 프로젝트가 끝나고 팀원분들에게 받은 피드백
    <img width="635" alt="스크린샷 2023-04-27 오후 5 38 52" src="https://user-images.githubusercontent.com/72093196/234807810-7cccb5d1-5314-4663-81fe-cd01d3839422.png">