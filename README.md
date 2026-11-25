# AI 활용 개발 · 서비스 기획 | jimnnieo-svg

2027년 2월 졸업 예정입니다. AI 코딩 도구로 웹·Android 프로토타입을 만들고, 요구사항 설계부터 기능 통합과 검증까지 맡아 왔습니다. **AI 활용 개발(바이브코딩), AI 서비스 기획, AX·업무 자동화** 직무에 지원합니다.

개발을 시작할 때는 사용자가 어디에서 막히는지와 어떤 결과를 성공으로 볼지 먼저 정합니다. AI가 제안한 구현은 그 기준에 맞춰 확인합니다. 같은 입력으로 오류를 재현하고, 모의 환경과 실기기에서 각각 무엇을 확인했는지 기록합니다.

## 대표 프로젝트

### 1. Fashion AI · 체형 분석·3D 아바타·AI 피팅 개발 | 팀 캡스톤

- **사진 기반 체형 분석** · 기존 카메라 화면에 전신 촬영 가이드를 추가했습니다. ML Kit 관절점과 MediaPipe 인체 마스크를 이용해 입력 키를 기준으로 어깨·허리·골반의 정면 폭을 추정했습니다. [측정 코드](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/ui/viewmodel/BodyMeasurementCalculator.kt#L71-L75) · [초기 구현 · PR #3](https://github.com/JEC0387/capston-second/pull/3)
- **3D 아바타** · 사용자 치수로 메시를 생성하고, 치수를 편집하면서 아바타를 회전·확대하거나 옷장 의류를 배치할 수 있도록 OpenGL ES 뷰어를 개발했습니다. [메시 생성 코드](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/data/PremiumAvatarMesh.kt#L191-L245) · [뷰어 도입 커밋](https://github.com/JEC0387/capston-second/commit/832755b87183e0a14b0c6d2b3845e7e017b32a12)
- **AI 가상 피팅** · Android에서 사람·의류 입력을 준비하고, FastAPI에서 외부 생성 모델 Genlook을 호출해 진행 상태와 결과를 앱에 돌려주는 흐름을 개발했습니다. 원본과 결과를 비교하고 PNG를 저장·복원할 수 있습니다. [공급자 호출 코드](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/backend/genlook_outfit_provider.py#L718-L729) · [연동 커밋](https://github.com/JEC0387/capston-second/commit/b981d273898c68f6df5f3b15b4c00466f9939c79)

팀 기능을 통합하면서 서버 전송 사진이 축소되는 오류를 재현해 수정했습니다. Galaxy S25+에서는 실제 생성과 저장·재열기를 확인했고, 상품의 단추·무늬 재현은 품질 기준에 미달했습니다. 이후 제 계정으로 APK 18633을 발행했습니다. [통합 PR #19](https://github.com/JEC0387/capston-second/pull/19) · [인증 복원 PR #23](https://github.com/JEC0387/capston-second/pull/23) · [실기기 검증](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/docs/P54_S25_GENLOOK_E2E_20261106.md#L3) · [18633 릴리스](https://github.com/JEC0387/capston-second/releases/tag/fashionfit-1.1.86.33-p70-20261115)

**발표 영상** · [FashionFit 캡스톤 최종 발표 (YouTube)](https://www.youtube.com/watch?v=6TrEPlsQ6sA)

**[Fashion AI 상세 포트폴리오 · 26개 기능의 코드, 검증 결과와 면접 문답](https://github.com/jimnnieo-svg/jimnnieo-svg/blob/main/docs/fashion-ai.md)**

### 2. Campus Flow · 과제 마감 관리 웹앱

과제 등록부터 마감일 확인, 완료 처리, 주간 통계 반영까지 한 화면에서 이어지도록 설계했습니다. Firebase 연동, 검색, 통계, JSON 백업·복원의 구현과 검증을 진행했습니다. V3의 Chromium 테스트는 모의 Firebase에서 기본 회귀 43건, 달력 16건, 완료 차트 16건을 통과했습니다. 운영 Firebase와 다중 기기 동기화는 별도 검증 대상입니다.

[V3 구현 브랜치](https://github.com/jimnnieo-svg/campus-flow/tree/feat/campus-flow-bento-v3) · [설계 변경 · PR #1](https://github.com/jimnnieo-svg/campus-flow/pull/1) · [실제 테스트 화면과 결과 JSON](https://github.com/jimnnieo-svg/campus-flow/tree/3790cac3335894ceeb2d62b89bfa244117d06977/screenshots)

**실제 구현 화면 · Campus Flow V3**

[![Campus Flow V3 달력 날짜 선택과 과제 필터링 — 모의 Firebase·Chromium 캡처](https://github.com/jimnnieo-svg/campus-flow/blob/3790cac3335894ceeb2d62b89bfa244117d06977/screenshots/taskly-calendar-selection.jpg?raw=true)](https://github.com/jimnnieo-svg/campus-flow/blob/3790cac3335894ceeb2d62b89bfa244117d06977/screenshots/taskly-calendar-selection.jpg)

*날짜 선택 시 해당 날짜의 과제만 표시되는 실제 테스트 화면입니다. 합성 과제와 예시 계정으로 검증했습니다.*

### 3. CoupleReels · 커플 취향 공유 Android 앱

릴스를 저장할 때 원본을 상대에게 보여줄지 직접 정하도록 서비스와 UI/UX를 기획했습니다. 브랜드 콘셉트와 추천 방식을 정하고, AI 코딩 도구로 프로토타입과 APK를 제작했습니다.

- **P0-3**에서 취향에 따른 교차 추천과 공유 흐름을 구현했습니다. 저장된 JUnit 결과 8개에는 44건, 실패·오류 0건이 기록돼 있습니다. 실제 두 휴대폰의 Firebase 동기화는 이 결과의 검증 범위 밖입니다. [P0-3 검증 결과](https://github.com/jimnnieo-svg/couplereels-android/tree/82ee4e8f24f7696f681bce355e8aff35a274b24f/reports/p0-3)
- **P0-8**에서는 로컬 두 프로필의 보관함을 분리하고, 직접 공유한 원본만 상대 화면에 전달하도록 했습니다. 캡션 분류는 키워드 규칙 기반이며 유사 콘텐츠 공급자는 미연결 상태입니다. [P0-8 구현 브랜치](https://github.com/jimnnieo-svg/couplereels-android/tree/feature/p0-8-library-20261119) · [공유 정책과 테스트 범위](https://github.com/jimnnieo-svg/couplereels-android/blob/4e465863ff08556467dc090af72d28d9980a1d4f/docs/P0-8-IMPLEMENTATION.txt)

**실제 구현 화면 · CoupleReels P0-8**

<a href="https://github.com/jimnnieo-svg/couplereels-android/blob/df2da0d32784c87da61e61dbb93e827a7431598b/public/screens/library.webp"><img src="https://github.com/jimnnieo-svg/couplereels-android/blob/df2da0d32784c87da61e61dbb93e827a7431598b/public/screens/library.webp?raw=true" alt="P0-8 개인 보관함" width="210"></a> <a href="https://github.com/jimnnieo-svg/couplereels-android/blob/df2da0d32784c87da61e61dbb93e827a7431598b/public/screens/profiles.webp"><img src="https://github.com/jimnnieo-svg/couplereels-android/blob/df2da0d32784c87da61e61dbb93e827a7431598b/public/screens/profiles.webp?raw=true" alt="P0-8 로컬 체험 프로필 및 공유 설정" width="210"></a>

*보관함과 로컬 두 프로필의 공유 설정. 기존 소개 사이트에서 사용한 식별정보 제거 공개용 사본이며, 실제 계정·커플 연결 화면이 아닙니다.*

### 4. Object Detection · Python 이미지 추론

카메라로 찍거나 업로드한 사진을 Streamlit에서 받아 사전학습 YOLOv8n으로 CPU 추론하는 예제입니다. 입력 전처리부터 탐지 상자, 신뢰도, 객체별 개수 표시까지 한 파일에서 확인할 수 있습니다. [추론 코드](https://github.com/jimnnieo-svg/object_detect/blob/0729f3cc5d1c646805ec7332e82b6f8bc13709d6/app.py) · [실행 방법](https://github.com/jimnnieo-svg/object_detect)

## 프로젝트에서 사용한 기술

| 분야 | 기술과 적용 경험 |
|---|---|
| Android | Kotlin · Jetpack Compose로 앱 화면과 공유 흐름 구성, Gradle로 APK 빌드 |
| 웹·데이터 | HTML · CSS · JavaScript로 일정 관리 화면 구현, Firebase Authentication · Firestore 연동 |
| AI 서비스 | Python · FastAPI 기반 피팅 서비스 통합, Streamlit · Ultralytics YOLO · OpenCV로 이미지 추론 및 시각화 |
| 검증 | Playwright · Chromium으로 웹 회귀 검증, Android 프로젝트의 JUnit 결과와 GitHub PR로 변경 근거 관리 |

<!-- 자료 제공 후 검토하여 추가: Fashion AI 캡스톤 최종 발표 영상 / Fashion AI 실제 앱 시연 영상 / CoupleReels 바이브코딩 사례 발표 영상. 개인정보·제3자 저작권·팀원 공개 동의 확인 후 제목과 링크를 기재. -->
<!-- 본인이 지정한 자료만 추가: 개인 포트폴리오 웹사이트 / PDF 포트폴리오 / 공개용 연락처. -->
