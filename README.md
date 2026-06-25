# README.md

This file provides guidance when working with code in this repository.

## 저장소 성격

SKT 복지온(Welfare) 모바일 앱의 **사내 배포(QA/검증)용 다운로드 페이지**입니다. App Store / Play Store 외부에서 dev·qa·prod 빌드를 설치할 수 있도록 버전 목록과 다운로드 링크를 제공합니다.

- 빌드된 정적 사이트(Vue.js, GitHub Pages 추정 — base path `/qa-dev/`)이며, **이 저장소에는 Vue 소스코드가 없습니다.** `js/`, `css/`는 다른 곳에서 빌드된 산출물(minified)입니다.
- 실제 앱 바이너리(`.apk`, `.ipa`)는 이 저장소가 아니라 **Dropbox**에 호스팅되며, 여기서는 그 링크만 관리합니다.
- 따라서 일상적인 작업은 거의 전부 **`app_versions.json`에 새 버전 항목을 추가**하는 것입니다. (git 로그가 전부 "버전 추가"인 이유)

## 핵심 파일

- `app_versions.json` — **단일 진실 공급원(SSOT).** 페이지가 런타임에 fetch하여 렌더링. `android`/`ios` × `dev`/`qa`/`prod`로 구분된 버전 배열. **git에 커밋됨.**
- `manifests/manifest_<버전>[_dev|_qa].plist` — iOS OTA 설치용 Apple manifest. IPA URL·bundle 정보 포함. **`.gitignore`에 포함되어 커밋되지 않음** (로컬 작업/업로드용).
  - **파일명 규칙(환경별 상이)**: dev → `manifest_<버전>_dev.plist`, qa → `manifest_<버전>_qa.plist`, prod → `manifest_<버전>.plist`(**접미사 없음**).
- `docs/VERSION_ADD_GUIDE.md` — iOS 버전 추가 상세 절차서. 버전 추가 전 반드시 참고.
- `index.html`, `js/`, `css/`, `img/`, `favicon.ico` — 빌드 산출물. 손으로 수정하지 않음.

## 동작 방식 (페이지 → 설치)

`app_versions.json`의 각 항목은 플랫폼별로 형태가 다릅니다:

- **Android**: `{ version, link }` — `link`는 `.apk` Dropbox URL. 클릭 시 바로 다운로드.
- **iOS**: `{ version, link, file }` — `link`는 manifest `.plist` Dropbox URL, `file`은 `.ipa` Dropbox URL.

설치 로직(빌드된 `js/app.*.js`의 `install()`)은 Dropbox 공유 URL을 직접 다운로드 URL로 변환합니다:
- `&dl=0` 제거 → `www.dropbox.com`을 `dl.dropboxusercontent.com`으로 치환
- URL에 `manifest`가 포함되면 `itms-services://?action=download-manifest&url=...`로 감싸 iOS OTA 설치를 트리거

이 때문에 **URL 형식 규칙이 엄격**합니다 (아래 참조).

## 버전 추가 작업 (가장 흔한 작업)

세부는 `docs/VERSION_ADD_GUIDE.md`를 따르되 핵심 규칙:

1. **URL 형식이 위치별로 다름 — 섞지 말 것:**
   - `app_versions.json`의 `link`/`file`: `https://www.dropbox.com/scl/fi/.../...?rlkey=...&st=...&dl=0` (공유 URL 형식)
   - manifest `.plist` **내부**의 IPA url: `https://dl.dropboxusercontent.com/s/.../...?rlkey=...` (직접 다운로드 형식)
2. **버전은 시간순 정렬** — 새 항목은 해당 배열 **맨 끝**에 추가.
3. **JSON 형식 주의** — 이전 마지막 항목 뒤에 쉼표 추가, 새 마지막 항목엔 쉼표 없음. **들여쓰기는 탭** (기존 항목과 동일하게).
4. 버전명 규칙: Android `welfare-<버전>-<dev|qa|prd>`, iOS `Welfare_<버전>_<dev|qa|prod>`.
   - 주의: Android prod 접미사는 `-prd`, iOS prod 접미사는 `_prod`로 **표기가 다름**.
5. iOS는 manifest plist 안 `bundle-version`을 새 버전으로 맞췄는지 확인.

## 커밋

- 커밋 메시지는 기존 관례를 따름 (예: `iOS QA: Welfare_2.6.1_qa 버전 추가`, `android 2.0.0 버전 추가`). 한글, 버전·플랫폼·환경 명시.
- `manifests/`와 `docs/`는 `.gitignore` 대상이므로 보통 `app_versions.json` 한 파일만 커밋됨.
- 원격: `git@github.com:welfare-skt/qa-dev.git`, 기본 브랜치 `main`.

## 빌드 산출물 수정이 필요하다면

`js/`·`css/`·`index.html`은 외부 Vue 프로젝트의 빌드 결과입니다. UI/동작 변경이 필요하면 이 저장소가 아니라 **원본 Vue 소스 저장소를 찾아 거기서 빌드**해야 합니다 (이 저장소에는 빌드 도구·`package.json`이 없음).
