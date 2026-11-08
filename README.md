# Co끼리 — 정민규 개인 기여 기록

전체 프로젝트의 소스 코드와 팀 협업 이력은 [SeD-ShallweData/2nd_Side2026_Project](https://github.com/SeD-ShallweData/2nd_Side2026_Project)에서 확인할 수 있습니다.

이 저장소는 **팀 프로젝트 Co끼리에서 정민규가 맡은 역할과 구현한 기능을 정리한 개인 포트폴리오 저장소**입니다. 개인 작업 범위는 원본 저장소에 제출하고 병합된 PR을 기준으로 설명합니다.

## 저장소 구성

| 경로 | 내용 |
| --- | --- |
| [source/](source/) | 개인이 추가한 API·서비스·저장소·테스트·명세와 현장 제보 migration |
| [patches/](patches/) | DB·운영 설정·대시보드 등 공동 파일에 반영한 PR별 변경 내역 |
| [CONTRIBUTIONS.md](CONTRIBUTIONS.md) | 파일별 원본 커밋·PR 링크와 작업물 선별 기준 |

소스는 개인 작업 당시의 커밋에서 추출했습니다. 실행과 테스트에는 원본 프로젝트의 공통 모듈과 설정이 필요하며, 파일별 출처와 검증 범위는 선별 목록에서 확인할 수 있습니다.

## 프로젝트 소개

**Co끼리**는 구직자와 근로자가 일하기 전과 일하는 중에 노동 관련 정보를 확인하고 도움을 받을 수 있도록 만든 웹 서비스입니다. AI Rookie 및 창의종합설계 경진대회를 위해 팀으로 개발했습니다.

사업장 검색과 임금체불·산업재해 관련 신호 조회, 노동법 자료 검색을 활용한 AI 상담, 근로계약서 진단, 커뮤니티, 현장 위험 제보를 하나의 서비스로 제공합니다. 상담 챗봇의 이름은 **돈워리**입니다.

## 담당 역할

**사용자 기능의 백엔드 API 구현과 실제 데이터베이스 연동**을 담당했습니다. 인증·커뮤니티·현장 제보·즐겨찾기 기능을 구현하고, DB 담당자의 스키마와 프론트엔드 담당자의 화면을 연결했습니다. 관리자 배치 현황과 임금 대시보드의 데이터 조회 경로도 정비했습니다.

주요 사용 기술은 TypeScript, Next.js Route Handlers, PostgreSQL, Node.js, sharp, Vitest입니다. GCP VM 환경에서 기능이 동작하도록 저장 경로와 운영 설정 검증도 보완했습니다.

## 주요 업무

### 1. 회원 인증·세션·권한 처리

로그인·로그아웃·세션 조회를 구현한 뒤, 회원가입과 실제 PostgreSQL 저장으로 확장했습니다.

- 비밀번호를 scrypt로 해시하고, 세션 토큰은 SHA-256 해시로 DB에 저장
- 일반 사용자·관리자·감독관 권한을 검사하고, 회원가입 시 일반 사용자 권한을 서버에서 고정
- 존재하지 않는 계정과 잘못된 비밀번호의 로그인 실패 응답을 통일
- 로그인 5회 연속 실패 시 15분 잠금과 남은 대기 시간 응답 구현
- 동시 로그인 요청에 의한 잠금 우회와 DB 장애의 실패 횟수 집계 방지
- 중복 이메일 회원가입 응답 정책과 API 명세 정리

관련 PR은 [#17](https://github.com/SeD-ShallweData/2nd_Side2026_Project/pull/17), [#40](https://github.com/SeD-ShallweData/2nd_Side2026_Project/pull/40), [#72](https://github.com/SeD-ShallweData/2nd_Side2026_Project/pull/72), [#83](https://github.com/SeD-ShallweData/2nd_Side2026_Project/pull/83)입니다.

### 2. 커뮤니티·게시글 신고

게시글과 신고를 처리하는 API를 구현하고, 메모리 기반 임시 저장소에서 실제 DB로 전환했습니다.

- 게시글 작성·조회·수정·삭제와 작성자 권한 검사 구현
- 신고 접수·중복 신고 차단·관리자 검토 구현
- 신고 승인과 게시글 숨김을 하나의 트랜잭션으로 처리
- 대기 중인 신고의 중복을 차단하고, 기각된 신고의 재접수 허용
- 익명 게시글의 작성자 이름을 저장소와 API 응답 단계에서 제거
- 게시글 삭제 시 삭제 상태를 저장해 기록 유지

프론트엔드에서 사용하던 API 응답 형식을 유지하면서 저장 방식을 전환했습니다. 관련 PR은 [#17](https://github.com/SeD-ShallweData/2nd_Side2026_Project/pull/17), [#40](https://github.com/SeD-ShallweData/2nd_Side2026_Project/pull/40), [#72](https://github.com/SeD-ShallweData/2nd_Side2026_Project/pull/72)입니다.

### 3. 현장 위험 제보·사진 저장

일반 사용자가 글과 사진으로 현장 위험을 제보하고, 감독관이 제보 목록·상세·사진을 조회하는 API를 구현했습니다.

- 제보 내용과 사진 정보를 PostgreSQL에 저장하고, 사진 원본은 VM의 비공개 영구 저장소에 보관
- 사진의 실제 형식·크기·용량·개수와 손상 여부 검증
- 감독관에게 제공하는 사진에서 위치·촬영 시각·기기 메타데이터 제거
- 제보자 이메일·내부 사용자 ID·원본 파일명·저장 경로의 응답 노출 차단
- 감독관 화면과 API에 로그인·권한 검사 적용
- 임금·산재 분류와 접수 상태를 반영하는 DB migration 및 검사 항목 추가
- DB 권한과 사진 저장소 상태를 준비 상태 검사에 반영

접수 상태의 초기값과 목록 정렬을 구현했으며, 제출한 PR 시점의 상태 변경 기능은 후속 정책 확정 과제로 남겼습니다. 관련 PR은 [#45](https://github.com/SeD-ShallweData/2nd_Side2026_Project/pull/45), [#72](https://github.com/SeD-ShallweData/2nd_Side2026_Project/pull/72)입니다.

### 4. 사용자별 관심 사업장·즐겨찾기

로그인한 일반 사용자의 즐겨찾기 추가·목록 조회·삭제 API를 구현했습니다. 프론트엔드 개발을 위한 임시 저장소를 먼저 제공한 뒤, DB 담당자가 만든 테이블과 권한 설정에 맞춰 실제 DB를 연결했습니다.

- 사용자별 즐겨찾기를 분리하고 사업장 존재 여부를 서버에서 검증
- 동일 사업장의 반복 추가·삭제를 멱등하게 처리
- 실제 DB의 중복 데이터 생성 방지
- 즐겨찾기와 인증 기능의 실제 DB 모드 설정을 함께 검증
- GCP 운영 DB에서 추가·조회·중복 방지·삭제를 확인하고, 검증 데이터는 트랜잭션 롤백으로 정리

관련 PR은 [#103](https://github.com/SeD-ShallweData/2nd_Side2026_Project/pull/103), [#130](https://github.com/SeD-ShallweData/2nd_Side2026_Project/pull/130)입니다.

### 5. 배치 현황·임금 대시보드 데이터 연동

관리자 배치 현황의 고정 예시 데이터를 실제 DB 조회로 전환했습니다. 현재 서비스 배치와 전체 적재 이력을 공통 조회 서비스로 제공하고, 관리자 권한 정책과 화면 연결을 맞췄습니다. 관련 PR은 [#100](https://github.com/SeD-ShallweData/2nd_Side2026_Project/pull/100)입니다.

임금 대시보드에서는 API 내부의 지역·업종 중복 집계를 DB 담당자가 만든 집계 뷰 조회로 전환했습니다. 필터 선택지도 같은 뷰를 사용하도록 맞추고 기존 API 응답 형식을 유지했습니다. 관련 PR은 [#104](https://github.com/SeD-ShallweData/2nd_Side2026_Project/pull/104)입니다.

## 공통 설계·검증

- **저장소 분리** — 서비스 로직과 저장소 구현을 분리해 임시 저장소와 실제 DB를 전환할 수 있도록 구성했습니다.
- **최소 권한 연결** — 인증·커뮤니티·현장 제보의 DB 연결을 기능별 계정으로 분리했습니다.
- **설정 오류 처리** — 실제 DB 모드의 설정이 누락되면 오류를 반환해 저장 실패를 확인할 수 있도록 했습니다.
- **운영 준비** — 제보 사진 저장 경로·접근 권한, 환경변수 검사, DB와 사진의 백업·복원 절차를 보완했습니다.
- **검증** — 인증·권한·익명 처리·신고·즐겨찾기 회귀 테스트를 작성하고, PR별 타입 검사·린트·빌드를 수행했습니다. 실제 DB 연동은 PR에 기록한 범위에서 별도로 검증했습니다.

구현과 검증의 근거는 [#40](https://github.com/SeD-ShallweData/2nd_Side2026_Project/pull/40), [#72](https://github.com/SeD-ShallweData/2nd_Side2026_Project/pull/72), [#130](https://github.com/SeD-ShallweData/2nd_Side2026_Project/pull/130)에서 확인할 수 있습니다.

## 협업 범위

DB 담당자가 설계한 사용자 데이터 구조와 권한, 프론트엔드 담당자가 구현한 화면을 기준으로 API와 저장소를 연결했습니다. PR #40의 역할 통합 migration은 DB 담당자의 커밋을 작성자 정보와 함께 반영했고, 즐겨찾기 테이블과 권한은 선행 PR #115를 바탕으로 연동했습니다.

GCP 관련 기여에는 애플리케이션의 DB·파일 저장 연동과 운영 설정 준비가 포함됩니다. 운영 환경변수 반영과 서비스 배포는 운영 담당자와 협업했습니다. 관련 이력은 [#40](https://github.com/SeD-ShallweData/2nd_Side2026_Project/pull/40), [#72](https://github.com/SeD-ShallweData/2nd_Side2026_Project/pull/72), [#130](https://github.com/SeD-ShallweData/2nd_Side2026_Project/pull/130)에서 확인할 수 있습니다.
