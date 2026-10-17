[← 프로필로 돌아가기](https://github.com/whdxo)

# ManalGak — 모임 중간 지점 + 맛집 추천

`팀장, 백엔드`

> 모두의 이동 시간이 비슷한 중간 지점과 장소를 추천하는 모바일 웹

[![저장소 바로가기](https://img.shields.io/badge/%EC%A0%80%EC%9E%A5%EC%86%8C_%EB%B0%94%EB%A1%9C%EA%B0%80%EA%B8%B0-CHOSSEANG/ManalGak--magicDev-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/CHOSSEANG/ManalGak-magicDev)

![Spring Boot](https://img.shields.io/badge/Spring_Boot-6DB33F?style=flat-square&logo=springboot&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-DC382D?style=flat-square&logo=redis&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-4479A1?style=flat-square&logo=mysql&logoColor=white)
![Kakao](https://img.shields.io/badge/Kakao_API-FFCD00?style=flat-square&logo=kakao&logoColor=black)
![ODsay](https://img.shields.io/badge/ODsay_API-0A66C2?style=flat-square)
![Next.js](https://img.shields.io/badge/Next.js-000000?style=flat-square&logo=nextdotjs&logoColor=white)

| 기간 | 역할 | 맡은 일 | AI |
| --- | --- | --- | --- |
| 2026.01 — 2026.02 (4주, 구름톤 딥다이브 팀 프로젝트) | 팀장, 백엔드 (5명 중 BE 3, FE 2) | 외부 API 연동, Redis 캐시, 지도, 이동시간 화면 | Claude Code (설계, 구현 보고서를 남기며 개발) |

**한눈에 보기**

- 경로 조회 **약 400ms → 약 50ms** (캐시 히트)
- 참여자 4명 경로 **1.2초 → 0.4초** (병렬 조회)
- 데이터마다 다른 캐시 수명: 장소 **1시간**, 경로 **30분**, 막차 **10분**
- 출발지가 바뀌면 **그 모임의 캐시만** 골라 지우고 화면에 실시간 알림

## 목차

- [1) 문제: 같은 경로를 매번 외부 API에 다시 묻고 있었음](#1-문제-같은-경로를-매번-외부-api에-다시-묻고-있었음)
- [2) 구조: 있으면 꺼내 쓰고, 없을 때만 외부 API를 부름](#2-구조-있으면-꺼내-쓰고-없을-때만-외부-api를-부름)
- [3) 수명: 바뀌는 속도가 다른 데이터라 유지 시간도 다르게](#3-수명-바뀌는-속도가-다른-데이터라-유지-시간도-다르게)
- [4) 무효화: 빠르게 만드는 것보다, 언제 지울지가 더 어려웠음](#4-무효화-빠르게-만드는-것보다-언제-지울지가-더-어려웠음)
- [5) 병렬 조회: 참여자 경로를 한 명씩이 아니라 동시에](#5-병렬-조회-참여자-경로를-한-명씩이-아니라-동시에)
- [6) 직접 작성한 코드](#6-직접-작성한-코드)
- [7) AI 활용과 회고](#7-ai-활용과-회고)

<table align="center">
<tr>
<td align="center" width="50%">
  <img src="../assets/manalgak-1.jpg" width="260" alt="모임 만들기"><br>
  <b>① 모임 만들기</b><br>
  <sub>모임 이름, 목적(음식점, 카페 등), 날짜와 시간을 고르면 그날 날씨도 함께 보여 줍니다</sub>
</td>
<td align="center" width="50%">
  <img src="../assets/manalgak-2.jpg" width="260" alt="중간 지점과 추천 장소"><br>
  <b>② 중간 지점, 추천 장소</b><br>
  <sub>참여자별 출발지와 경로를 지도에 그리고, 모두에게 공평한 추천 장소를 표시합니다</sub>
</td>
</tr>
<tr>
<td align="center" colspan="2">
  <img src="../assets/manalgak-3.jpg" width="260" alt="참여자별 이동시간"><br>
  <b>③ 참여자별 이동시간</b><br>
  <sub>고른 장소까지 각자 몇 분 걸리는지 대중교통, 자동차로 나눠 비교합니다</sub>
</td>
</tr>
</table>

## 1) 문제: 같은 경로를 매번 외부 API에 다시 묻고 있었음

추천 장소, 경로, 막차를 볼 때마다 카카오, ODsay API를 새로 호출했습니다.
화면이 느렸고, 참여자가 늘수록 호출 수가 곱으로 늘어 **호출 한도에 가까워졌습니다.**

## 2) 구조: 있으면 꺼내 쓰고, 없을 때만 외부 API를 부름

```
요청 ─▶ Redis 캐시 ──있음──▶ 바로 응답 (약 50ms)
           │ 없음
           ▼
         DB ──있음──▶ 응답 + Redis에 저장
           │ 없음
           ▼
     외부 API (카카오, ODsay) ─▶ 응답 + DB, Redis에 저장 (약 400ms)
```

## 3) 수명: 바뀌는 속도가 다른 데이터라 유지 시간도 다르게

| 데이터 | 수명 | 이유 |
| --- | --- | --- |
| 추천 장소 | 1시간 | 장소 정보는 자주 바뀌지 않고, 카카오 호출을 줄여야 함 |
| 경로 | 30분 | 교통 상황이 시간대별로 바뀌어 너무 오래된 경로는 부정확 |
| 막차 | 10분 | 막차 시간 확인용이라 실시간성이 가장 중요 |
| AI 요약 | 24시간 | 같은 입력이면 결과가 같고, OpenAI 호출 비용이 큼 |

캐시 키는 `places:meeting:{모임}:candidate:{후보지}:{목적}:{개수}`처럼 **모임 단위로 묶이게** 규칙을 정했습니다.
그래서 나중에 "이 모임 것만" 골라 지울 수 있습니다.

## 4) 무효화: 빠르게 만드는 것보다, 언제 지울지가 더 어려웠음

- 참여자가 출발지를 바꾸면 **그 모임, 그 참여자와 관련된 캐시만** 찾아서 삭제
- 키를 찾을 때 `KEYS` 대신 `SCAN`을 사용: 운영 중인 Redis를 멈추지 않기 위해
- 지운 뒤에는 같은 모임 참여자 화면에 실시간으로 알려 오래된 경로를 보지 않게 함

## 5) 병렬 조회: 참여자 경로를 한 명씩이 아니라 동시에

참여자마다 순서대로 경로를 묻던 것을 `CompletableFuture`로 동시에 요청해, 4명 기준 **1.2초 → 0.4초**로 줄였습니다.

## 6) 직접 작성한 코드

| 파일 | 하는 일 |
| --- | --- |
| [`CacheTTL.java`](https://github.com/CHOSSEANG/ManalGak-magicDev/blob/develop/backend/src/main/java/com/magicdev/manalgak/common/cache/CacheTTL.java) | 데이터별 캐시 수명과 그 이유 |
| [`CacheKeys.java`](https://github.com/CHOSSEANG/ManalGak-magicDev/blob/develop/backend/src/main/java/com/magicdev/manalgak/common/cache/CacheKeys.java) | 모임 단위로 묶이는 캐시 키 규칙 |
| [`CacheInvalidationService.java`](https://github.com/CHOSSEANG/ManalGak-magicDev/blob/develop/backend/src/main/java/com/magicdev/manalgak/common/cache/CacheInvalidationService.java) | 모임, 후보지, 참여자 단위 캐시 삭제 (SCAN) |
| [`OdsayApiService.java`](https://github.com/CHOSSEANG/ManalGak-magicDev/blob/develop/backend/src/main/java/com/magicdev/manalgak/domain/external/odsay/service/OdsayApiService.java) | 참여자 경로 병렬 조회 |
| [`LastTrainService.java`](https://github.com/CHOSSEANG/ManalGak-magicDev/blob/develop/backend/src/main/java/com/magicdev/manalgak/domain/odsay/service/LastTrainService.java) | 막차 조회와 캐시 |
| [`PlaceCacheService.java`](https://github.com/CHOSSEANG/ManalGak-magicDev/blob/develop/backend/src/main/java/com/magicdev/manalgak/domain/place/service/PlaceCacheService.java) | 추천 장소 캐시 |

## 7) AI 활용과 회고

- Claude Code와 **캐시 키, 수명 설계를 문서로 먼저 정리한 뒤** 구현하고, 결과를 보고서로 남겼습니다.
- 팀장으로서 백엔드 3명이 같은 규칙으로 캐시를 쓰도록 키와 수명을 공통 코드로 묶어 두었습니다.
- 빠르게 만드는 것보다, **언제 지워야 하는지 정하는 게** 더 어려웠습니다.

---

[← 프로필로 돌아가기](https://github.com/whdxo)
