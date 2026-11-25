# Fashion AI | 코드로 보는 개인 기여

Android 팀 캡스톤에서 전신 사진 분석, 치수로 조작하는 3D 아바타, 외부 AI 가상 피팅을 개발했습니다. 카메라·분석 화면, 추천, DB, 백업, 배포에는 팀의 선행 구현이 있습니다. 아래에는 제가 추가하거나 확장한 호출 경로와 팀 기능에 연결한 범위를 표시했습니다.

사진 한 장에서 추정하는 값은 신체의 정면 폭입니다. 3D 아바타는 치수 기반 메시이고, 생성형 피팅 결과는 외부 Genlook 모델에서 받습니다. Galaxy S25+에서 결과 생성·비교·저장에 성공했지만 상품 디테일 재현은 품질 기준에 미달했습니다.

각 코드 링크는 APK 18633 릴리스 시점의 파일과 줄을 가리킵니다. 변경 링크는 해당 기능을 도입하거나 개선한 커밋입니다. 테스트 소스와 실제 실행 기록은 아래에서 별도로 표시했습니다. [초기 기능 PR #3](https://github.com/JEC0387/capston-second/pull/3) · [통합 PR #19](https://github.com/JEC0387/capston-second/pull/19) · [인증 복원 PR #23](https://github.com/JEC0387/capston-second/pull/23)

## 촬영과 체형 분석

### 1. 전신 촬영·자동 셔터

기존 CameraX 화면에 관절 위치로 정면 자세와 전신 포함 여부를 판단하는 가이드를 추가했습니다. 화면이 활성 상태이고 촬영 잠금을 얻었을 때만 사진을 찍어 중복 저장을 막습니다.

핵심 구현 · `CameraPreviewScreen · CaptureGuide.evaluate · AutoCaptureGate.tryLock`

[`if (disposed || !resumed || !gate.tryLock()) return`](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/ui/screens/CameraPreviewScreen.kt#L135)

[코드 CameraPreviewScreen.kt L135-L142](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/ui/screens/CameraPreviewScreen.kt#L135-L142) · [코드 CaptureGuide.kt L24-L50](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/ui/camera/CaptureGuide.kt#L24-L50) · [변경 812b355](https://github.com/JEC0387/capston-second/commit/812b3555937eea44e5bf89c8b820529bca22059e)

가이드 조건은 단위 테스트에서 확인할 수 있습니다. 기기별 촬영 성공률은 측정하지 않았습니다. [테스트 CaptureGuideTest](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/test/java/com/jeiu/fashionproject/CaptureGuideTest.kt#L19) · [실행·판정 기록 P17_CAMERA_PERSONAL_VTON_HANDOFF_20261104](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/docs/P17_CAMERA_PERSONAL_VTON_HANDOFF_20261104.md#L5)

### 2. 사진 분석과 부분 결과

팀의 사진 분석 화면을 확장해 ML Kit 관절점과 MediaPipe 인체 마스크를 치수 계산에 연결했습니다. 자세·분할 품질이 부족하면 판단 가능한 항목만 화면에 돌려줍니다.

핵심 구현 · `BodyAnalysisViewModel.analyzeImage · detectPose`

[`val pose = detectPose(image)`](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/ui/viewmodel/BodyAnalysisViewModel.kt#L139)

[코드 BodyAnalysisViewModel.kt L139-L158](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/ui/viewmodel/BodyAnalysisViewModel.kt#L139-L158) · [코드 BodyAnalysisScreen.kt L103-L170](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/ui/screens/BodyAnalysisScreen.kt#L103-L170) · [변경 812b355](https://github.com/JEC0387/capston-second/commit/812b3555937eea44e5bf89c8b820529bca22059e)

합성 입력을 사용하는 분석 경로 테스트가 있습니다. 실제 사용자 집단의 분류 정확도는 미측정입니다. [테스트 P31SyntheticBodyPipelineTest](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/test/java/com/jeiu/fashionproject/P31SyntheticBodyPipelineTest.kt#L29) · [실행·판정 기록 P17_CAMERA_PERSONAL_VTON_HANDOFF_20261104](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/docs/P17_CAMERA_PERSONAL_VTON_HANDOFF_20261104.md#L5)

### 3. 실루엣·픽셀을 cm로 환산

입력한 키를 사진 속 전신 높이로 나눠 픽셀당 길이를 구하고, 마스크에서 얻은 어깨·허리·골반의 정면 폭에 적용했습니다.

핵심 구현 · `BodyMeasurementCalculator.calculate · SilhouetteMeasurer.inspect`

[`val scale = heightCm / heightPx`](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/ui/viewmodel/BodyMeasurementCalculator.kt#L71)

[코드 BodyMeasurementCalculator.kt L71-L75](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/ui/viewmodel/BodyMeasurementCalculator.kt#L71-L75) · [코드 BodyAnalysisViewModel.kt L157-L158](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/ui/viewmodel/BodyAnalysisViewModel.kt#L157-L158) · [변경 812b355](https://github.com/JEC0387/capston-second/commit/812b3555937eea44e5bf89c8b820529bca22059e)

환산식과 실루엣 측정기의 단위 테스트가 있습니다. 줄자로 잰 둘레와의 오차를 입증하는 자료는 없습니다. [테스트 BodyMeasurementCalculatorTest](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/test/java/com/jeiu/fashionproject/BodyMeasurementCalculatorTest.kt#L12) · [테스트 SilhouetteMeasurerTest](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/test/java/com/jeiu/fashionproject/SilhouetteMeasurerTest.kt#L11)

### 4. 수기 치수와 사진 추정값의 출처

수기 치수와 사진 추정값에 출처와 측정 시점을 보존했습니다. 같은 부위라도 정면 폭과 둘레가 섞이지 않도록 입력 경로를 구분했습니다.

핵심 구현 · `BodyDimensions.resolve · ManualBodyStore.update`

[`put(key,cm,MeasurementOrigin.PHOTO_ESTIMATE,estimated.measuredAt)`](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/data/BodyDimensions.kt#L49)

[코드 BodyDimensions.kt L49-L65](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/data/BodyDimensions.kt#L49-L65) · [코드 ManualBodyMeasurements.kt L86-L109](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/data/ManualBodyMeasurements.kt#L86-L109) · [변경 d0484d4](https://github.com/JEC0387/capston-second/commit/d0484d49e89c9898caa9061cf8be2c0b2a099824)

치수 출처와 수기 값 저장을 다루는 테스트가 있습니다. [테스트 BodyDimensionsTest](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/test/java/com/jeiu/fashionproject/BodyDimensionsTest.kt#L13) · [테스트 P34ManualBodyPersistenceTest](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/test/java/com/jeiu/fashionproject/P34ManualBodyPersistenceTest.kt#L37)

## 3D 아바타와 착장 편집

### 5. 파라미터 기반 3D 아바타

사용자 치수로 몸통과 팔다리의 단면을 정해 메시를 만들었습니다. Compose 화면에서 Android GL 뷰에 형상과 카메라 회전·줌 값을 전달합니다.

핵심 구현 · `BodyAvatarMesh.build · AvatarSurface.update`

[`parameters.heightCm/100f`](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/data/BodyAvatar.kt#L68)

[코드 BodyAvatar.kt L64-L73](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/data/BodyAvatar.kt#L64-L73) · [코드 BodyAvatarDialog.kt L63-L69](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/ui/components/BodyAvatarDialog.kt#L63-L69) · [변경 832755b](https://github.com/JEC0387/capston-second/commit/832755b87183e0a14b0c6d2b3845e7e017b32a12)

아바타 파라미터 및 프로필 테스트가 있습니다. 사진에서 3D 형상을 복원하는 모델 학습은 이 구현의 범위 밖입니다. [테스트 BodyAvatarTest](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/test/java/com/jeiu/fashionproject/BodyAvatarTest.kt#L10) · [테스트 AvatarProfileTest](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/test/java/com/jeiu/fashionproject/AvatarProfileTest.kt#L25)

### 6. 3D 메시 고도화·실시간 편집

인체 형태 함수를 격자에서 샘플링한 뒤 표면을 삼각형으로 변환했습니다. 치수를 바꾸면서 메시의 변화를 확인할 수 있도록 편집 화면을 연결했습니다.

핵심 구현 · `PremiumAvatarMesh.build/extract · AvatarAnalysisForm`

[`shape.field(minX+x*step,ys[y],minZ+z*step)`](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/data/PremiumAvatarMesh.kt#L191)

[코드 PremiumAvatarMesh.kt L191-L245](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/data/PremiumAvatarMesh.kt#L191-L245) · [코드 AvatarAnalysisForm.kt L87-L133](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/ui/components/AvatarAnalysisForm.kt#L87-L133) · [변경 5ce064e](https://github.com/JEC0387/capston-second/commit/5ce064ec60ebca7d0be4f7daa1f3060bf9350fc6)

메시 형상과 편집 화면을 확인하는 회귀 테스트가 있습니다. [테스트 P70LiveAvatarGeometryTest](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/test/java/com/jeiu/fashionproject/P70LiveAvatarGeometryTest.kt#L15) · [테스트 P70BodyFlowUiTest](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/test/java/com/jeiu/fashionproject/P70BodyFlowUiTest.kt#L51)

### 7. 저장 프로필 기반 체형 추정

저장한 가슴·허리·엉덩이 치수의 차와 비율로 체형을 추정합니다. 필수 입력이 비었을 때는 빠진 부위를 반환해 화면에서 보완할 수 있게 했습니다.

핵심 구현 · `AvatarBodyAnalysis.analyze · AvatarAnalysisInput.build`

[`if (missing.isNotEmpty()) return AvatarBodyResult(measurements, null, missing, null, null)`](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/recommend/AvatarBodyAnalysis.kt#L41)

[코드 AvatarBodyAnalysis.kt L41-L46](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/recommend/AvatarBodyAnalysis.kt#L41-L46) · [코드 AvatarAnalysisInput.kt L38-L40](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/data/AvatarAnalysisInput.kt#L38-L40) · [변경 a2e6aa9](https://github.com/JEC0387/capston-second/commit/a2e6aa9f10b115877669060d29e1cb60725b4665)

치수 조합과 미입력 분기를 다루는 테스트가 있습니다. 체형 추정은 규칙 기반입니다. [테스트 AvatarBodyAnalysisTest](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/test/java/com/jeiu/fashionproject/AvatarBodyAnalysisTest.kt#L29) · [테스트 P70BodyFlowUiTest](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/test/java/com/jeiu/fashionproject/P70BodyFlowUiTest.kt#L51)

### 8. 의류 오버레이·사진과 3D 조작

팀의 피팅 화면에 아바타용 의류 배치를 추가했습니다. 사진 모드는 자세를, 아바타 모드는 신체 파라미터를 기준으로 옷을 배치하고 드래그한 결과를 보존합니다.

핵심 구현 · `VirtualFittingScreen · fitAvatar · photoAutoPlacement`

[`fitAvatar(slot,g.item,g.aspect*region.height/region.width,canvasSize,avatarProfile!!)`](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/ui/screens/VirtualFittingScreen.kt#L514)

[코드 VirtualFittingScreen.kt L514-L547](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/ui/screens/VirtualFittingScreen.kt#L514-L547) · [코드 AvatarFittingStore.kt L25-L44](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/data/AvatarFittingStore.kt#L25-L44) · [변경 ab7514e](https://github.com/JEC0387/capston-second/commit/ab7514eec1299da08333b355d5e63c0733e2bf6f)

배치 계산과 화면 조작 테스트가 있습니다. 이 단계의 미리보기는 로컬 오버레이입니다. [테스트 AvatarFittingTest](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/test/java/com/jeiu/fashionproject/AvatarFittingTest.kt#L34) · [테스트 PresentationFittingUiTest](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/test/java/com/jeiu/fashionproject/PresentationFittingUiTest.kt#L35)

### 9. 캔버스 크기 변경 시 정렬 보존

화면 크기가 바뀔 때 이전 축척과 이미지 여백을 계산해 의류 배치를 새 캔버스 좌표로 옮깁니다.

핵심 구현 · `resizePhotoPlacement · photoAutoPlacement`

[`val ratio = newScale / oldScale`](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/ui/screens/PhotoFittingCoordinates.kt#L19)

[코드 PhotoFittingCoordinates.kt L13-L20](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/ui/screens/PhotoFittingCoordinates.kt#L13-L20) · [코드 VirtualFittingScreen.kt L141-L145](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/ui/screens/VirtualFittingScreen.kt#L141-L145) · [변경 c14fca0](https://github.com/JEC0387/capston-second/commit/c14fca05ce60a148be08e5e47499c91b005f3485)

리사이즈 전후의 의류 위치를 비교하는 회귀 테스트가 있습니다. [테스트 P22PhotoResizeTest](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/test/java/com/jeiu/fashionproject/P22PhotoResizeTest.kt) · [테스트 P16FittingUiTest](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/test/java/com/jeiu/fashionproject/P16FittingUiTest.kt#L31)

## 이미지 입력부터 AI 결과 저장까지

### 10. 이미지 입력 정규화·해시 계약

사람·의류 이미지의 방향과 종횡비를 보존하면서 PNG 입력을 준비했습니다. 기기 처리용과 서버 전송용 크기를 분리하고 정지 WebP 입력까지 확장했습니다.

핵심 구현 · `OnDeviceVtonImages.prepare · ServerOutfitVtonViewModel.start`

[`Bitmap.createBitmap(decoded, 0, 0, decoded.width, decoded.height, matrix, true)`](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/data/OnDeviceVtonImages.kt#L242)

[코드 OnDeviceVtonImages.kt L241-L249](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/data/OnDeviceVtonImages.kt#L241-L249) · [코드 ServerOutfitVtonViewModel.kt L269-L274](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/ui/viewmodel/ServerOutfitVtonViewModel.kt#L269-L274) · [변경 e99da04](https://github.com/JEC0387/capston-second/commit/e99da0440bfdcf3f97761749990dd1a8a78c5f50)

통합 PR의 합성 이미지 루프백에서 서버 전송 규격을 확인했습니다. P68의 후속 빌드 기록은 별도 버전입니다. [테스트 P57ServerImageContractTest](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/test/java/com/jeiu/fashionproject/P57ServerImageContractTest.kt#L152) · [실행·판정 기록 P68_RELEASE_18625_VERIFICATION](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/docs/P68_RELEASE_18625_VERIFICATION.md#L9)

### 11. 서버형 피팅 요청·상태 복원

앱에서 요청 시점의 세션·입력·티켓을 묶어 FastAPI 작업을 제출하고, 받은 상태를 화면에 반영하며 완료를 조회합니다. 재개 시에는 계정과 기존 티켓을 확인합니다.

핵심 구현 · `ServerOutfitVtonViewModel.start/resume · OutfitService · submit`

[`val first = gateway.submit(session, ticket, body)`](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/ui/viewmodel/ServerOutfitVtonViewModel.kt#L291)

[코드 ServerOutfitVtonViewModel.kt L291-L294](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/ui/viewmodel/ServerOutfitVtonViewModel.kt#L291-L294) · [코드 outfit_vton_api.py L650-L690](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/backend/outfit_vton_api.py#L650-L690) · [변경 1e2abd1](https://github.com/JEC0387/capston-second/commit/1e2abd1525679dedd17a8bc9d95580bcb2778737)

S25+ 기록에서 대기 시간 초과 후 기존 작업을 조회해 결과를 복원했습니다. 모든 중단·취소 시나리오를 실기기로 검증한 것은 아닙니다. [테스트 P48ServerOutfitControllerTest](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/test/java/com/jeiu/fashionproject/P48ServerOutfitControllerTest.kt#L92) · [실행·판정 기록 P54_S25_GENLOOK_E2E_20261106](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/docs/P54_S25_GENLOOK_E2E_20261106.md#L3)

### 12. Genlook 외부 생성 API

서버 어댑터가 사람·의류 입력을 업로드하고 Genlook에 생성을 요청한 뒤 작업 상태와 결과를 조회합니다. 상·하의 중간 결과를 검토한 다음 신발 단계로 진행합니다. 이미지 생성은 외부 모델이 수행합니다.

핵심 구현 · `GenlookProvider._stage/generate · build_stage_body`

[`created = await self._request("POST", "/try-on", accepted=(201,),`](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/backend/genlook_outfit_provider.py#L718)

[코드 genlook_outfit_provider.py L718-L729](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/backend/genlook_outfit_provider.py#L718-L729) · [코드 genlook_request_contract.py L81-L102](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/backend/genlook_request_contract.py#L81-L102) · [변경 b981d27](https://github.com/JEC0387/capston-second/commit/b981d273898c68f6df5f3b15b4c00466f9939c79)

S25+에서 실제 생성 경로는 통과했습니다. 원래 상품의 단추·포켓·무늬 재현은 품질 기준에 미달했습니다. [테스트 test_genlook_p51_pair](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/backend/tests/test_genlook_p51_pair.py#L108) · [실행·판정 기록 P54_S25_GENLOOK_E2E_20261106](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/docs/P54_S25_GENLOOK_E2E_20261106.md#L3)

### 13. AI 결과와 원본 비교·저장·재열기

결과를 요청 당시 원본과 연결해 비교할 수 있게 하고, 받은 PNG 바이트를 선택한 위치에 저장해 다시 엽니다.

핵심 구현 · `ServerOutfitVtonScreen · openSavedPhoto · TryOnComparisonDialog`

[`OnDeviceVtonImages.saveExact(context, bytes, uri)`](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/ui/screens/ServerOutfitVtonScreen.kt#L109)

[코드 ServerOutfitVtonScreen.kt L109-L124](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/ui/screens/ServerOutfitVtonScreen.kt#L109-L124) · [코드 TryOnComparisonDialog.kt L20-L43](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/ui/components/TryOnComparisonDialog.kt#L20-L43) · [변경 1d4dad8](https://github.com/JEC0387/capston-second/commit/1d4dad82802e24b8c3f1cb9533b4e9a1c886dc5a)

S25+ 검증 기록에서 저장·재열기한 이미지의 해시가 일치했습니다. [테스트 P48ServerOutfitUiTest](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/test/java/com/jeiu/fashionproject/P48ServerOutfitUiTest.kt#L56) · [실행·판정 기록 P54_S25_GENLOOK_E2E_20261106](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/docs/P54_S25_GENLOOK_E2E_20261106.md#L3)

### 14. 아바타 착장 프레임 PNG 내보내기

현재 OpenGL 프레임을 캡처한 뒤 화면의 의류 배치와 투명도로 합성해 착장 PNG를 만듭니다.

핵심 구현 · `AvatarOutfitExport.create · AvatarFrameCapture`

[`val frame=capture.captureBitmap()`](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/ui/screens/AvatarOutfitExport.kt#L25)

[코드 AvatarOutfitExport.kt L25-L40](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/ui/screens/AvatarOutfitExport.kt#L25-L40) · [코드 AvatarOutfitExport.kt L21-L34](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/ui/screens/AvatarOutfitExport.kt#L21-L34) · [변경 5f1302a](https://github.com/JEC0387/capston-second/commit/5f1302a7145bcafc5dd97c5a6caf2ae5701efb03)

프레임 캡처와 내보내기를 다루는 테스트가 있습니다. 서버 AI 결과 저장과 별개의 경로입니다. [테스트 P63AvatarCaptureTest](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/test/java/com/jeiu/fashionproject/P63AvatarCaptureTest.kt#L29) · [테스트 P63AvatarExportUiTest](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/test/java/com/jeiu/fashionproject/P63AvatarExportUiTest.kt#L38)

## 실험 경로와 팀 서비스 연동

### 15. 로컬 VTON 실행 실험

Python 작업자가 기존 가중치·런타임을 확인하고 사람·의류 입력을 로컬 생성 파이프라인에 전달하게 했습니다.

핵심 구현 · `local_vton_worker.main · LocalVtonRepository.submit/result`

[`check_weights(Path(options["weights_dir"]))`](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/backend/local_vton_worker.py#L81)

[코드 local_vton_worker.py L81-L87](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/backend/local_vton_worker.py#L81-L87) · [코드 LocalVtonRepository.kt L71-L86](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/data/LocalVtonRepository.kt#L71-L86) · [변경 eafcbcf](https://github.com/JEC0387/capston-second/commit/eafcbcfba507b4aaba9b0b292698ad66bab13bd7)

S25+ 앱에서 로컬 작업자와 연결한 공식 예제의 성공 기록이 있습니다. 임의 사진이나 휴대폰 단독 추론 성공으로 일반화할 수 없습니다. [실행·판정 기록 P15_APP_VTON_HANDOFF_20261104](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/docs/P15_APP_VTON_HANDOFF_20261104.md#L3) · [실행·판정 기록 P56_FINAL_ACCEPTANCE_20261106](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/docs/P56_FINAL_ACCEPTANCE_20261106.md#L3)

### 16. 팀 DB 통합·체형 기록 동기화

팀 DB와 체형 분석을 연결했습니다. 분석 성공 및 저장 동의를 확인해 이력을 전달하고, 연결을 회복하면 대기열을 처리합니다.

핵심 구현 · `DbSync.flush · BodyAnalysisCompletion.complete`

[`val health = BackendApi.getJson("/api/health/database") ?: return@withLock`](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/data/DbSync.kt#L86)

[코드 DbSync.kt L86-L98](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/data/DbSync.kt#L86-L98) · [코드 BodyAnalysisViewModel.kt L166-L167](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/ui/viewmodel/BodyAnalysisViewModel.kt#L166-L167) · [변경 9731363](https://github.com/JEC0387/capston-second/commit/9731363037a7a30b6ebd0c48cfdc7eb70854de51)

DB API와 체형 데이터 저장 조건 테스트가 있습니다. 팀 DB 스키마는 공동 작업의 기반입니다. [테스트 test_db_api](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/backend/tests/test_db_api.py#L17) · [테스트 BodyDataPrivacyTest](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/test/java/com/jeiu/fashionproject/BodyDataPrivacyTest.kt#L16)

### 17. 계정 전환·수기 측정 복원

서버가 확인한 계정 식별자에 로컬 측정값을 연결해 인증이 갱신돼도 같은 사용자의 값이 복원되도록 했습니다.

핵심 구현 · `BodyMeasurementScope.current/prepare · ManualBodyStore.load/update`

[`DataAuth.bodyScope(AuthStore.mode.value)`](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/data/BodyMeasurementScope.kt#L15)

[코드 BodyMeasurementScope.kt L14-L23](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/data/BodyMeasurementScope.kt#L14-L23) · [코드 ManualBodyMeasurements.kt L51-L86](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/data/ManualBodyMeasurements.kt#L51-L86) · [변경 11400fa](https://github.com/JEC0387/capston-second/commit/11400fad2fd54efb5379140e382f8ecbbd4ae6e6)

계정 소유권·인증 보존 테스트와 병합된 PR #23에서 변경 경계를 확인할 수 있습니다. [테스트 BodyAccountOwnershipTest](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/test/java/com/jeiu/fashionproject/BodyAccountOwnershipTest.kt#L99) · [테스트 P57AuthPreservationTest](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/test/java/com/jeiu/fashionproject/P57AuthPreservationTest.kt#L71)

### 18. 신체 기반 사이즈·코디 보정

상품 어깨 단면과 사용자 어깨 폭의 차이를 비교하고, 출처가 있는 신체 데이터를 코디 점수에 반영했습니다. 팀 추천 흐름에 연결한 보정 범위입니다.

핵심 구현 · `BodySizeRecommendation.shoulderSuggestions · BodyAwareOutfitScorer.evaluate`

[`cm - shoulder >= MIN_SHOULDER_EASE_CM`](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/recommend/BodySizeRecommendation.kt#L63)

[코드 BodySizeRecommendation.kt L59-L66](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/recommend/BodySizeRecommendation.kt#L59-L66) · [코드 BodyAwareOutfitScorer.kt L10-L13](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/recommend/BodyAwareOutfitScorer.kt#L10-L13) · [변경 d0484d4](https://github.com/JEC0387/capston-second/commit/d0484d49e89c9898caa9061cf8be2c0b2a099824)

사이즈 제안과 신체 기반 점수 테스트가 있습니다. 실제 구매 후 맞음새는 측정하지 않았습니다. [테스트 BodySizeRecommendationTest](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/test/java/com/jeiu/fashionproject/BodySizeRecommendationTest.kt#L16) · [테스트 BodyAwareOutfitRecommendationTest](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/test/java/com/jeiu/fashionproject/BodyAwareOutfitRecommendationTest.kt#L27)

### 19. 상품 식별·스타일 가이드

기존 상품 화면에 체형 조건별 스타일 제안과 상품 식별자 기반 상세 이동을 연결했습니다.

핵심 구현 · `BodyStyleSuggestions.create · AppNavigation`

[`(state as? BodyAnalysisUiState.Success)?.result`](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/recommend/BodyStyleSuggestions.kt#L29)

[코드 BodyStyleSuggestions.kt L24-L83](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/recommend/BodyStyleSuggestions.kt#L24-L83) · [코드 AppNavigation.kt L307](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/navigation/AppNavigation.kt#L307) · [변경 1e0584b](https://github.com/JEC0387/capston-second/commit/1e0584b000328812487c32f4fd56002edc33e9d8)

상품 선택 후 이동 대상과 조건별 제안을 검사하는 테스트가 있습니다. [테스트 P19ProductNavigationTest](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/test/java/com/jeiu/fashionproject/P19ProductNavigationTest.kt#L32) · [테스트 P19BodyStyleSuggestionsTest](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/test/java/com/jeiu/fashionproject/P19BodyStyleSuggestionsTest.kt#L36)

### 20. 옷장 데모 복원·사용자 데이터 보존

기존 옷장 데이터를 먼저 읽고 비어 있을 때만 사용자가 데모 의류를 복원할 수 있게 했습니다.

핵심 구현 · `WardrobeStore.addDemoSamplesIfEmpty/writeForRestore`

[`if (!loaded || _items.value.isNotEmpty()) return false`](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/data/WardrobeStore.kt#L163)

[코드 WardrobeStore.kt L161](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/data/WardrobeStore.kt#L161) · [코드 WardrobeStore.kt L255](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/data/WardrobeStore.kt#L255) · [변경 ebd2fc8](https://github.com/JEC0387/capston-second/commit/ebd2fc874acb010ee4b89dbe0c01bc4430e24b60)

복원 동작과 샘플 미리보기 테스트가 있습니다. [테스트 WardrobeSampleRestoreUiTest](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/test/java/com/jeiu/fashionproject/WardrobeSampleRestoreUiTest.kt#L51) · [테스트 SampleWardrobePreviewTest](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/test/java/com/jeiu/fashionproject/SampleWardrobePreviewTest.kt#L12)

## 대체 피팅 경로 · 데이터 보존 · 배포

### 21. 초기 외부 피팅 프록시

초기 피팅에서 앱이 서버를 경유해 Fashn 작업을 요청하고 상태를 조회하게 했습니다. 동기식 공급자 호출은 작업 스레드로 넘깁니다.

핵심 구현 · `FashnProvider.run/status`

[`value = await asyncio.to_thread(self._request, "run", body)`](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/backend/tryon_proxy.py#L95)

[코드 tryon_proxy.py L95-L261](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/backend/tryon_proxy.py#L95-L261) · [변경 d9ec445](https://github.com/JEC0387/capston-second/commit/d9ec44506a56fe6edeb708c66a059b4cf1d63617)

프록시 테스트가 있습니다. 후속 Genlook 실기기 검증은 다른 공급자 경로의 결과입니다. [테스트 test_tryon_proxy](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/backend/tests/test_tryon_proxy.py)

### 22. 로컬 드레이핑 합성

기기의 비트맵과 마스크로 의류 레이어를 사람 사진 위에 그리는 합성 경로를 추가했습니다.

핵심 구현 · `LocalDrapeCompositor.render`

[`abs(photoAspect/poseAspect-1)>aspectTolerance`](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/ui/screens/P36LocalDrapeCompositor.kt#L588)

[코드 P36LocalDrapeCompositor.kt L582-L592](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/ui/screens/P36LocalDrapeCompositor.kt#L582-L592) · [변경 232ccbf](https://github.com/JEC0387/capston-second/commit/232ccbf632e87b5c9c4a7f0f10d31698bc9c9f06)

합성기 테스트가 있습니다. 해당 경로는 외부 생성 API를 호출하지 않습니다. [테스트 P36DrapeCompositorTest](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/test/java/com/jeiu/fashionproject/P36DrapeCompositorTest.kt)

### 23. 온디바이스 VTON의 실행 한계

기기용 생성 엔진의 입력·호출 인터페이스를 구성하고 엔진이 준비되지 않으면 명시적인 실패를 반환합니다.

핵심 구현 · `DeviceEngineAvailability · DeviceVtonFailure`

[`throw DeviceVtonFailure("온디바이스 생성형 엔진이 준비되지 않았어요.")`](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/data/OnDeviceVtonEngine.kt#L94)

[코드 OnDeviceVtonEngine.kt L94-L94](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/data/OnDeviceVtonEngine.kt#L94-L94) · [변경 68ab317](https://github.com/JEC0387/capston-second/commit/68ab31778127415d95ea5e8dffa74dcb21152000)

부분 구현입니다. 진단 연산까지 기록됐고 휴대폰 단독 전체 생성은 검증되지 않았습니다. [실행·판정 기록 P44_MODEL_DECISION](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/docs/P44_MODEL_DECISION.md)

### 24. 옷장 스냅샷·계정 복원

팀의 사진 백업을 확장해 인증된 스냅샷을 검증하고 옷장·착용 기록·선호 항목을 임시 위치에 먼저 복원하도록 했습니다.

핵심 구현 · `BackupRestore.restoreAuthenticated`

[`WardrobeStore.writeForRestore(staging, items)`](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/data/BackupRestore.kt#L251)

[코드 BackupRestore.kt L251-L260](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/data/BackupRestore.kt#L251-L260) · [변경 2b01552](https://github.com/JEC0387/capston-second/commit/2b01552c0d8f5f72b18f9ca71d12b9f67f2d40c1)

기존 데이터 보존 테스트가 있습니다. [테스트 P23DataPreservationTest](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/test/java/com/jeiu/fashionproject/P23DataPreservationTest.kt)

### 25. DB 마이그레이션 사전 검증

마이그레이션 실행 전에 DB 이력과 검토된 변경 목록을 대조한 뒤 트랜잭션을 진행하도록 했습니다.

핵심 구현 · `run_migrations · _check_history`

[`applied = _check_history(cursor.fetchall(), migrations, review)`](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/backend/migration_runner.py#L247)

[코드 migration_runner.py L247-L254](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/backend/migration_runner.py#L247-L254) · [변경 1eccb83](https://github.com/JEC0387/capston-second/commit/1eccb83999697a0dac3c7fe9b50b10c1424d2f9e)

P57은 로컬 모의 드라이버에 대한 검증입니다. 운영 DB에 실제 적용했다는 증거로 사용하지 않습니다. [실행·판정 기록 P57_TEAM_INTEGRATION_20261107](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/docs/P57_TEAM_INTEGRATION_20261107.md)

### 26. 빌드 검증·APK 발행

팀 빌드 워크플로에서 프리뷰 빌드를 식별하고 회귀 결과를 남기도록 개선했습니다. 제 계정으로 APK 18633 릴리스를 발행했습니다.

핵심 구현 · `Gradle 테스트·assembleDebug · upload-artifact`

[`:app:assembleDebug :app:testDebugUnitTest`](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/.github/workflows/android-debug-apk.yml#L168)

[코드 android-debug-apk.yml L168-L168](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/.github/workflows/android-debug-apk.yml#L168-L168) · [변경 13b378f](https://github.com/JEC0387/capston-second/commit/13b378f7ea82619d5381252e4865f9efa49a0073)

P68의 빌드·테스트 기록은 18625입니다. 18633 발행은 별도의 릴리스에서 확인할 수 있습니다. 후속 팀 데모 배포는 팀의 작업입니다. [실행·판정 기록 P68_RELEASE_18625_VERIFICATION](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/docs/P68_RELEASE_18625_VERIFICATION.md) · [APK 18633](https://github.com/JEC0387/capston-second/releases/tag/fashionfit-1.1.86.33-p70-20261115)

## 기술 면접 문답

**어디부터 본인이 만든 기능인가요?** 팀에 카메라와 사진 분석 화면이 있었습니다. 제가 제출한 초기 브랜치에서 전신 가이드·실루엣 측정·cm 계산과 각 테스트를 추가하고 기존 ViewModel에 연결했습니다. 같은 커밋이 PR #3으로 병합됐습니다. 이후 치수 기반 아바타와 서버 피팅 경로를 개발했습니다. [초기 변경](https://github.com/JEC0387/capston-second/commit/812b3555937eea44e5bf89c8b820529bca22059e) · [팀 병합](https://github.com/JEC0387/capston-second/pull/3)

**사진 한 장으로 무엇을 알 수 있나요?** 촬영 조건을 만족할 때 사진의 전신 픽셀 높이를 입력 키로 보정해 부위의 정면 폭을 추정합니다. 둘레 입력은 출처가 다르므로 따로 저장합니다. 전신 높이나 분할 품질이 부족하면 부분 결과로 돌려보냅니다. 실제 치수 대비 오차율은 측정 자료가 없어 수치로 주장하지 않습니다. [환산식](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/ui/viewmodel/BodyMeasurementCalculator.kt#L66-L95) · [치수 출처](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/data/BodyDimensions.kt#L30-L65)

**3D 아바타가 만들어지는 원리는 무엇인가요?** 입력 치수와 편집값을 프로필에 모아 몸통·팔다리의 형상 파라미터를 만들고, 연속 형태 함수를 샘플링해 삼각형 표면을 생성합니다. Android GL 뷰가 메시와 카메라 회전·줌을 처리합니다. 아바타 착장에서는 이 아바타와 배치된 의류를 합성합니다. [메시 생성](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/data/PremiumAvatarMesh.kt#L165-L245) · [GL 뷰 연결](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/ui/components/BodyAvatarDialog.kt#L60-L71)

**AI 피팅에서 개발한 것과 외부 서비스가 한 일은 무엇인가요?** 사람·의류 이미지 준비, 요청 계약, 인증된 서버 작업, 앱 폴링·결과 표시를 개발했습니다. Genlook은 전달받은 입력으로 이미지를 생성합니다. 상·하의 중간 결과를 검토한 뒤 신발 단계로 넘어갑니다. 외부 모델 학습은 제 기여로 기재하지 않습니다. [서버 작업](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/backend/outfit_vton_api.py#L650-L690) · [공급자 호출](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/backend/genlook_outfit_provider.py#L693-L735)

**AI 코딩 도구의 출력을 어떻게 검증했나요?** 정상 화면 외에 입력이 축소되는 조건, 계정 전환, 실패 후 재시작을 정했습니다. 예를 들어 설치된 앱에 합성 이미지를 넣어 서버 수신 바이트와 입력 크기를 확인했습니다. 저장소로 확인되는 것은 이 판단과 테스트, 수정 결과이며 AI가 작성한 줄 수는 측정하지 않았습니다. [실기기 루프백 검증](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/docs/P57_TEAM_INTEGRATION_20261107.md)

**원본 비교는 왜 요청 시점에 묶었나요?** 생성 도중 사용자가 다른 옷을 선택하거나 계정을 바꾸면 현재 화면의 사진이 요청 당시 사진과 달라집니다. 요청에 사용한 선택과 티켓을 유지하고 상태 복원 때 세션을 확인합니다. 유료 생성을 다시 요청하는 대신 기존 상태를 조회합니다. [컨트롤러](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/fashion_app/app/src/main/java/com/jeiu/fashionproject/ui/viewmodel/ServerOutfitVtonViewModel.kt#L243-L317)

**실기기 성공과 제품 품질 사이에는 어떤 차이가 있었나요?** S25+로 생성·비교·PNG 저장과 읽기를 확인했습니다. 하지만 재킷의 단추와 포켓처럼 구매 판단에 필요한 세부 요소는 변형돼 정식 품질 기준을 충족하지 못했습니다. 그래서 보고서에 작동 결과와 상품 충실도 판정을 함께 남겼습니다. [P54](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/docs/P54_S25_GENLOOK_E2E_20261106.md) · [P56](https://github.com/JEC0387/capston-second/blob/b0c72f355629d8bd4e09fbdfb912aed93a3b0429/docs/P56_FINAL_ACCEPTANCE_20261106.md)

**팀원과 배포 책임은 어떻게 나뉘었나요?** 저는 핵심 기능 개발과 통합·QA를 맡았고 18633 릴리스를 발행했습니다. 추천·DB·인증·쇼핑·배포에는 팀의 선행 작업이 있습니다. 11월 23일 데모 APK는 팀 워크플로의 후속 배포입니다. 행사 단말에 설치된 정확한 파일까지는 GitHub에서 확인할 수 없습니다. [18633](https://github.com/JEC0387/capston-second/releases/tag/fashionfit-1.1.86.33-p70-20261115) · [팀 데모](https://github.com/JEC0387/capston-second/releases/tag/apk-demo-20261123-cefb781)
