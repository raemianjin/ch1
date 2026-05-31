# AI Console — Android (네이티브 WebView)

무료 AI API 멀티 제공자 채팅 콘솔을 **네이티브 Android 앱**으로 감싼 프로젝트입니다.
저장소 루트가 표준 **Android Gradle 프로젝트**라, 기존에 쓰시던 자동 빌드 템플릿
(`actions/setup-java` + Gradle, `cache: gradle`)이 **그대로 빌드**합니다. 템플릿을 바꿀 필요가 없습니다.

## 구조
```
.
├─ settings.gradle / build.gradle / gradle.properties
├─ gradlew / gradlew.bat / gradle/wrapper/...   # Gradle Wrapper (커밋 필수)
└─ app/
   ├─ build.gradle
   └─ src/main/
      ├─ AndroidManifest.xml
      ├─ java/com/aiconsole/app/MainActivity.java   # WebView 호스트
      ├─ assets/www/index.html                      # 앱 본체(웹)
      └─ res/...                                     # 아이콘/문자열
```

## 빌드
- **CI(기존 템플릿)**: 루트에 Gradle 프로젝트가 있으므로 `./gradlew build` 또는 `./gradlew assembleDebug` 가 동작합니다.
  - 산출물: `app/build/outputs/apk/debug/app-debug.apk` (디버그), `.../release/app-release-unsigned.apk` (릴리스, 미서명).
  - 템플릿이 APK를 아티팩트로 안 올리면, 아래 한 줄만 추가하면 됩니다(템플릿 유지하면서 보강):
    ```yaml
    - uses: actions/upload-artifact@v4
      with:
        name: apk
        path: app/build/outputs/apk/**/*.apk
    ```
- **로컬**: `./gradlew assembleDebug` (JDK 17 필요). Android Studio로 열어도 됩니다.

## 동작 메모
- WebView에 `DomStorage`를 켜서 API 키·설정이 `localStorage`에 저장됩니다.
- `AllowUniversalAccessFromFileURLs` 를 켜서 `file://` 페이지에서도 외부 AI API 호출 시 **CORS 제약을 우회**합니다. (브라우저보다 호환성이 좋습니다.)
- `<input type=file>` / 첨부는 `onShowFileChooser` 로 안드로이드 파일 선택기와 연동됩니다.
- 디버그 APK는 서명되지 않아 설치 시 "출처를 알 수 없는 앱" 허용이 필요합니다.

## 앱 본체 수정
화면/기능은 전부 `app/src/main/assets/www/index.html` 한 파일에 있습니다. 이 파일만 고치면 됩니다.

## CI 트러블슈팅
- `./gradlew: Permission denied` 가 나면 템플릿 빌드 단계 앞에 `chmod +x gradlew` 를 추가하거나, 한 번 `git update-index --chmod=+x gradlew` 후 커밋하세요.
- 빌드 직전 단계에서 `cache: gradle` 가 더 이상 "No file matched" 경고를 내지 않습니다(루트에 Gradle 파일 존재).
