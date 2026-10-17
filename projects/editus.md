[← 프로필로 돌아가기](https://github.com/whdxo)

# EditUs — 브라우저 Web IDE

`통합 담당, 풀스택`

> 설치 없이 브라우저에서 함께 코딩하고, 실행하고, AI 리뷰까지 받는 Web IDE

[![저장소 바로가기](https://img.shields.io/badge/%EC%A0%80%EC%9E%A5%EC%86%8C_%EB%B0%94%EB%A1%9C%EA%B0%80%EA%B8%B0-whdxo/web__ide__project-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/whdxo/web_ide_project)

![Spring Boot](https://img.shields.io/badge/Spring_Boot-6DB33F?style=flat-square&logo=springboot&logoColor=white)
![WebSocket](https://img.shields.io/badge/WebSocket_(STOMP)-010101?style=flat-square&logo=socketdotio&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-DC382D?style=flat-square&logo=redis&logoColor=white)
![React](https://img.shields.io/badge/React-20232A?style=flat-square&logo=react&logoColor=61DAFB)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white)
![OpenAI](https://img.shields.io/badge/GPT--4o--mini-412991?style=flat-square&logo=openai&logoColor=white)
![AWS S3](https://img.shields.io/badge/AWS_S3-569A31?style=flat-square&logo=amazons3&logoColor=white)

| 기간 | 역할 | 맡은 일 | AI |
| --- | --- | --- | --- |
| 2025.12 (4주, 구름톤 딥다이브 팀 프로젝트) | 통합 담당, 풀스택 (6명 중 커밋 최다) | AI 코드 리뷰, 코드 실행, 채팅 배포 안정화 | 서비스: GPT-4o-mini<br>개발: Claude Code |

**한눈에 보기**

- 코드 작성 → 실행 → **AI 리뷰(점수, 요약, 개선 제안)** 가 한 화면에서
- Judge0 API로 **10개 언어** 실행 (Python, Java, JavaScript, TypeScript, C, C++, Go, Rust, Ruby, PHP)
- 배포 직후 **채팅 접속자 0명** 장애를 연결 경로 점검으로 찾아 해결
- 팀 채팅은 WebSocket(STOMP) + Redis Pub/Sub, 파일은 AWS S3에 저장

## 목차

- [1) 문제: 환경 설정에 첫날을 쓰고 있었음](#1-문제-환경-설정에-첫날을-쓰고-있었음)
- [2) 제가 만든 것](#2-제가-만든-것)
- [3) 배포 장애: 배포하자 채팅 접속자가 0명, 로컬에서는 멀쩡했음](#3-배포-장애-배포하자-채팅-접속자가-0명-로컬에서는-멀쩡했음)
- [4) 다시 한다면](#4-다시-한다면)
- [5) 주요 코드](#5-주요-코드)

<p align="center">
  <img src="../assets/editus-ide.jpg" width="640" alt="EditUs 코드 작성, 실행, AI 리뷰 화면">
  <br>
  <sub>코드 작성 → 실행 → AI 리뷰(점수, 요약, 개선 제안)가 한 화면에서</sub>
</p>

## 1) 문제: 환경 설정에 첫날을 쓰고 있었음

OS, 버전이 달라 로컬 환경 구성에 시간이 걸리고, 에디터, 리뷰, 채팅 도구가 흩어져 있었습니다.

## 2) 제가 만든 것

| 기능 | 내용 |
| --- | --- |
| AI 코드 리뷰 | 파일을 보내면 점수, 한 줄 요약, 개선점 3~5개를 JSON으로 받아 이력과 함께 저장 |
| 코드 실행 | Judge0 API로 10개 언어 실행, 입력값(stdin)과 결과, 오류를 화면에 표시 |
| 통합 | 프론트, 백엔드 응답 형식 불일치 4곳 정리 (파일을 열면 검은 화면이 뜨던 문제) |
| 채팅 | 한글 입력 시 메시지가 두 번 전송되던 문제 수정 |

## 3) 배포 장애: 배포하자 채팅 접속자가 0명, 로컬에서는 멀쩡했음

```
브라우저 (useChat.ts) ─▶ 접속 주소 만들기 ─✕─▶ 서버 STOMP (/ws-chat) ─▶ Redis Pub/Sub
                          VITE_API_URL → undefined
```

| | |
| --- | --- |
| **원인** | 프론트가 존재하지 않는 환경변수를 읽어 기본 주소로 접속 → 서버가 여는 `/ws-chat`이 아닌 `/ws`로 연결 시도 |
| **해결** | WebSocket 주소 변수를 먼저 쓰고, 없으면 API 주소에서 `/ws-chat` 경로를 만들도록 수정. 접속자 수 메시지는 DB에 저장하지 않게 분리 |
| **AI** | 연결 경로(브라우저 → Nginx → 서버 → Redis)를 문서로 점검해 원인을 좁히고, 수정 커밋은 Claude Code와 공동 작성 |

## 4) 다시 한다면

지금의 AI 리뷰는 서버가 OpenAI 응답을 끝까지 기다리는 **동기 호출**입니다.
응답이 늦으면 요청이 묶이고, 타임아웃이나 재시도가 없어 실패하면 사용자는 오류만 보게 됩니다.
다시 만든다면 리뷰 요청을 큐에 넣어 비동기로 처리하고, 끝나면 이미 쓰고 있는 WebSocket으로 알려 주겠습니다.
여기에 타임아웃, 재시도, 응답 형식 검증을 먼저 넣겠습니다.

## 5) 주요 코드

| 파일 | 하는 일 |
| --- | --- |
| [`AIReviewService.java`](https://github.com/whdxo/web_ide_project/blob/main/backend/src/main/java/com/editus/backend/domain/ai/service/AIReviewService.java) | OpenAI 호출, 리뷰 결과 저장 |
| [`AIReviewPanel.tsx`](https://github.com/whdxo/web_ide_project/blob/main/frontend/src/features/ai/components/AIReviewPanel.tsx) | 리뷰 요청, 결과, 이력 화면 |
| [`CodeExecutionService.java`](https://github.com/whdxo/web_ide_project/blob/main/backend/src/main/java/com/editus/backend/domain/code/service/CodeExecutionService.java) | Judge0 연동, 언어별 실행 |
| [`useChat.ts`](https://github.com/whdxo/web_ide_project/blob/main/frontend/src/features/chat/hooks/useChat.ts) | 채팅 연결 (배포 장애를 고친 곳) |

---

[← 프로필로 돌아가기](https://github.com/whdxo)
