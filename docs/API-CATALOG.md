# 전체 API 목록과 실효 권한

기준: `master` / `53a3c80e4e0c0d36e4cf84e97047fd5676628376`

총 15개 Controller, 59개 endpoint를 확인했습니다. 아래의 "실효 접근 조건"은 [`SecurityConfig`](https://github.com/team-incube/Flooding-Server-V2/blob/53a3c80e4e0c0d36e4cf84e97047fd5676628376/src/main/kotlin/team/incube/flooding/global/security/config/SecurityConfig.kt)의 filter-chain 규칙과 Service 내부 검사를 함께 반영한 결과입니다.

- `공개`: JWT 없이 호출 가능
- `인증`: 유효한 Bearer access token 필요
- 역할명: Spring Security URL 규칙 또는 Service 내부 검사까지 통과해야 하는 실효 조건
- `리더`: 대상 동아리의 `leader.id`가 현재 사용자 ID와 같아야 함
- 별도 표기가 없는 endpoint는 `anyRequest().authenticated()`에 의해 인증이 필요함

## 인증

| Method | Path | 처리 | 실효 접근 조건 |
| --- | --- | --- | --- |
| POST | `/auth/signin` | DataGSM OAuth 로그인, 최초 사용자 생성, JWT 발급 | 공개 |
| POST | `/auth/reissue` | refresh token 검증 및 access/refresh 회전 | 공개 + Redis 저장값 일치 |

## 사용자

| Method | Path | 처리 | 실효 접근 조건 |
| --- | --- | --- | --- |
| GET | `/users/me` | 현재 사용자 정보 조회 | 인증 |
| GET | `/users` | 이름·학번 검색 및 pagination | 인증 |
| POST | `/users/me/profile-image` | 프로필 이미지 검증·R2 업로드 | 인증 |
| PATCH | `/users/{userId}/role` | 사용자 역할 변경 | `ADMIN` 또는 `DORMITORY_MANAGER` |

## AI

| Method | Path | 처리 | 실효 접근 조건 |
| --- | --- | --- | --- |
| POST | `/ai/chat` | 인증 헤더를 전달해 AI 챗봇 호출 | 인증 |
| POST | `/ai/song` | 최근 기상음악 최대 5곡으로 추천 요청 | 인증 + 신청 이력 존재 |

## 홈베이스

| Method | Path | 처리 | 실효 접근 조건 |
| --- | --- | --- | --- |
| POST | `/homebase/{homebaseId}` | 정원·시간·학생 중복 검사 후 예약 생성 | 인증 |
| GET | `/homebase?date=yyyy-MM-dd` | 날짜별 예약 목록 조회 | 인증 |
| DELETE | `/homebase/{reservationId}` | 예약 및 구성원 정보 삭제 | 인증 |

## NEIS

| Method | Path | 처리 | 실효 접근 조건 |
| --- | --- | --- | --- |
| GET | `/v2/neis/meals?date=yyyy-MM-dd` | 급식 조회 | 인증 |
| GET | `/v2/neis/timetables?grade=&classNumber=&date=` | 학년·반·날짜별 시간표 조회 | 인증 |

## 자습

| Method | Path | 처리 | 실효 접근 조건 |
| --- | --- | --- | --- |
| GET | `/dormitory/studies` | 당일 신청자와 신청 가능 상태 조회 | 인증 |
| POST | `/dormitory/studies` | 시간·금지·정원 검사 후 신청 | 인증 |
| DELETE | `/dormitory/studies` | 본인 신청 취소 | 인증 |
| GET | `/dormitory/studies/attendance` | 출석 SSE 구독 | `ADMIN` 또는 `DORMITORY_MANAGER` |
| POST | `/dormitory/studies/attendance/{userId}` | 출석 체크 및 이벤트 방송 | `ADMIN` 또는 `DORMITORY_MANAGER` |
| DELETE | `/dormitory/studies/attendance/{userId}` | 출석 체크 해제 및 이벤트 방송 | `ADMIN` 또는 `DORMITORY_MANAGER` |
| POST | `/dormitory/studies/ban/{userId}` | 1주일 자습 금지 | `ADMIN` 또는 `DORMITORY_MANAGER` |
| DELETE | `/dormitory/studies/ban/{userId}` | 자습 금지 해제 | `ADMIN` 또는 `DORMITORY_MANAGER` |
| GET | `/public/study/attendances` | 공개 출석 목록 조회 | 공개 |

## 기상음악

| Method | Path | 처리 | 실효 접근 조건 |
| --- | --- | --- | --- |
| GET | `/dormitory/music/subscribe` | 기상음악 SSE 구독 | 인증 |
| GET | `/dormitory/music?date=&sort=` | 날짜별 목록 및 TIME/LIKE 정렬 | 인증 |
| POST | `/dormitory/music` | YouTube 정보 조회 후 당일 신청 저장 | 인증 |
| DELETE | `/dormitory/music/{musicId}` | 본인 신청 취소 | 인증 + 신청자 본인 |
| POST | `/dormitory/music/{musicId}/like` | 좋아요 생성 | 인증 |
| DELETE | `/dormitory/music/{musicId}/like` | 본인 좋아요 삭제 | 인증 |

## 마사지

| Method | Path | 처리 | 실효 접근 조건 |
| --- | --- | --- | --- |
| GET | `/dormitory/massages` | 대기열과 본인 상태 조회 | 인증 |
| POST | `/dormitory/massages` | 시간·정원 검사 후 선착순 신청 | 인증 |
| DELETE | `/dormitory/massages` | 본인 신청 취소, 당일 재신청 차단 | 인증 |

## 기숙사 벌점

| Method | Path | 처리 | 실효 접근 조건 |
| --- | --- | --- | --- |
| GET | `/dormitory/penalties/me` | 본인 벌점 조회 | 인증 |
| GET | `/dormitory/penalties` | 전체 벌점 pagination 조회 | `ADMIN` 또는 `DORMITORY_MANAGER` |
| PUT | `/dormitory/penalties/{userId}` | 벌점 절대값 설정 및 이력 기록 | `ADMIN` 또는 `DORMITORY_MANAGER` |

## 기숙사 청소 구역

| Method | Path | 처리 | 실효 접근 조건 |
| --- | --- | --- | --- |
| POST | `/dormitory/cleaning-zones` | 청소 구역 생성 | `ADMIN` 또는 `DORMITORY_MANAGER` |
| GET | `/dormitory/cleaning-zones` | 전체 구역과 배정 인원 수 조회 | 인증 |
| GET | `/dormitory/cleaning-zones/{zoneId}` | 구역 상세와 배정 학생 조회 | 인증 |
| PATCH | `/dormitory/cleaning-zones/{zoneId}/members` | 구성원을 전체 교체 방식으로 배정 | `ADMIN` 또는 `DORMITORY_MANAGER` |

## 동아리 기본 기능

| Method | Path | 처리 | 실효 접근 조건 |
| --- | --- | --- | --- |
| POST | `/clubs` | 동아리 개설 신청 생성 | 인증 |
| PATCH | `/clubs/{clubId}/approval` | 개설 신청 승인·거부 | `ADMIN` 또는 `STUDENT_COUNCIL` |
| DELETE | `/clubs/{clubId}` | 동아리와 관련 폼·신청·참가자 삭제 | 리더, `ADMIN` 또는 `STUDENT_COUNCIL` |
| PUT | `/clubs/{clubId}` | 동아리 정보 전체 수정 | 리더, `ADMIN` 또는 `STUDENT_COUNCIL` |
| POST | `/clubs/{clubId}/autonomous/applications` | 자율 동아리 선착순 신청 | `GENERAL_STUDENT`, `STUDENT_COUNCIL`, `ADMIN` |
| GET | `/clubs?type=&name=` | 유형별 목록·검색 | 인증 |
| GET | `/clubs/{clubId}` | 상세·멤버·DataGSM 프로젝트 조회 | 인증 |
| POST | `/clubs/representative-image` | 대표 이미지 검증·R2 업로드 | `GENERAL_STUDENT`, `STUDENT_COUNCIL`, `ADMIN` |
| GET | `/clubs/export` | 전공 동아리 명단 XLSX 생성 | `ADMIN` |
| GET | `/clubs/opening-status` | 개설 신청 기간 정보 조회 | 인증 |
| PATCH | `/clubs/opening-period` | 개설 신청 기간 설정 | `ADMIN` |

## 동아리 신청

| Method | Path | 처리 | 실효 접근 조건 |
| --- | --- | --- | --- |
| POST | `/clubs/{clubId}/applications` | 활성 폼에 답변 제출 | `GENERAL_STUDENT`, `STUDENT_COUNCIL`, `ADMIN` |
| PATCH | `/clubs/{clubId}/applications/{userId}` | 신청자를 참가자로 승인 | 리더 |
| GET | `/clubs/opening/requests` | 개설 신청 목록 조회 | `ADMIN` (filter는 학생회도 허용하지만 Service가 ADMIN만 허용) |
| GET | `/clubs/{clubId}/applications` | 폼 신청자와 답변 조회 | 리더, `ADMIN` 또는 `STUDENT_COUNCIL` |

## 동아리 신청 폼

| Method | Path | 처리 | 실효 접근 조건 |
| --- | --- | --- | --- |
| POST | `/clubs/{clubId}/forms` | 새 활성 폼·필드·선택지 생성 | 리더 |
| GET | `/clubs/{clubId}/forms` | 활성 폼 조회 | 인증 |
| PUT | `/clubs/{clubId}/forms` | 폼 전체 수정 | 리더 + 기존 신청자 없음 |

## 동아리 멤버

| Method | Path | 처리 | 실효 접근 조건 |
| --- | --- | --- | --- |
| POST | `/clubs/{clubId}/member/{userId}` | 멤버 직접 초대 | 리더 또는 `ADMIN` |
| PATCH | `/clubs/{clubId}/transfer/{targetUserId}` | 리더 소유권 위임 | 현재 리더 |
| DELETE | `/clubs/{clubId}/member/exile/{userId}` | 멤버 추방 | `ADMIN` |

## 응답 형식 예외

- `/auth/signin`, `/auth/reissue`: token response DTO를 직접 반환
- `POST /dormitory/music`: `WakeUpMusicResponse`를 직접 반환
- 두 SSE endpoint: `text/event-stream`, 공통 JSON wrapper 제외
- `GET /clubs/export`: XLSX binary stream
- 나머지 대부분: the-moment SDK의 `CommonApiResponse`
