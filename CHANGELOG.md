# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [Unreleased]

---

## [1.0.1] - 2026-06-28

### Fixed
- README의 GitHub 릴리즈 배지 링크 포맷 오류 수정 (`bc8b6f1`)

### Added
- README에 GitHub 릴리즈 배지 추가 (`489cb35`)

---

## [1.0.0] - 2026-06-26

### Added
- PRD(제품 요구사항 정의서) 문서 추가 (`PRD.md`) (`ef3fcf0`)
- 프로젝트 스크린샷 추가 (`static/screenshot.png`) (`ef3fcf0`)
- 프롬프트 참고 문서 추가 (`prompt.md`) (`ef3fcf0`)
- README 업데이트: 실행 화면, 주요 기능, 기술 스택, 작업 내용 요약 보강 (`ef3fcf0`)

---

## [0.3.0] - 2026-06-24

### Fixed
- Svelte 5 이벤트 핸들러 일관성 오류로 인한 빌드 에러 수정 (`8282f0d`)
  - 기존 `on:event` 디렉티브를 Svelte 5 표준인 `onevent` 형태로 통일
  - `svelte-dnd-action`의 `onconsider` / `onfinalize` 커스텀 이벤트 처리 방식 적용

### Changed
- 웹사이트 타이틀을 `Image2PDF`로 변경 (`6f4acfc`)

---

## [0.2.0] - 2026-06-24

### Added
- 이미지 썸네일 클릭 시 원본 크기 미리보기 팝업(라이트박스) 기능 추가 (`2d85f4a`)
  - 배경 클릭 또는 닫기 버튼으로 팝업 닫기 지원
  - `backdrop-blur` 배경 오버레이 적용

---

## [0.1.0] - 2026-06-24

### Added
- GitHub Pages 자동 배포 파이프라인 구성 (`64c19e8`)
  - `@sveltejs/adapter-static`을 사용한 정적 사이트 빌드 설정
  - `BASE_PATH` 환경변수를 통한 GitHub Pages 서브패스 자동 처리
  - GitHub Actions 워크플로우 (`deploy.yml`) 추가: `main` 브랜치 푸시 시 자동 빌드 및 배포

---

## [0.0.1] - 2026-06-24

### Added
- SvelteKit + Svelte 5(룬 문법) + Tailwind CSS v4 기반 프로젝트 초기 구성 (`114c7bd`)
- 드래그 & 드롭 파일 업로드 영역 구현 (데스크톱 드래그 및 모바일 터치 파일 선택 지원)
- `svelte-dnd-action`을 활용한 썸네일 순서 변경 기능 (모바일 터치 / 데스크톱 드래그 모두 지원)
- `pdf-lib` 기반 클라이언트 사이드 PDF 변환 로직 구현
  - JPG / PNG 이미지 임베드 지원
  - 이미지 원본 비율 유지하며 페이지 중앙 배치 (`scaleToFit`)
  - 변환 완료 후 `Blob` URL을 통한 다운로드 링크 생성
- 반응형 썸네일 그리드 (모바일 2열 → 데스크톱 5열)
- 이미지 변환 중 로딩 스피너 및 UI 비활성화 처리
- `lucide-svelte` 아이콘 적용
- 다크 테마 기반 UI (`bg-neutral-950`)
