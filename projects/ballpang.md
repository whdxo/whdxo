[← 프로필로 돌아가기](https://github.com/whdxo)

# BallPang — 야구장 매장, 경기 일정 통합 앱

`팀장, 풀스택`

> 야구장 관람객이 경기 일정, 구장 안 매장, 매장 위치를 한곳에서 확인하는 모바일 앱

[![저장소 바로가기](https://img.shields.io/badge/%EC%A0%80%EC%9E%A5%EC%86%8C_%EB%B0%94%EB%A1%9C%EA%B0%80%EA%B8%B0-whdxo/ballpang__app-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/whdxo/ballpang_app)

![Flutter](https://img.shields.io/badge/Flutter-02569B?style=flat-square&logo=flutter&logoColor=white)
![Dart](https://img.shields.io/badge/Dart-0175C2?style=flat-square&logo=dart&logoColor=white)
![Firebase](https://img.shields.io/badge/Firebase-FFCA28?style=flat-square&logo=firebase&logoColor=black)
![Kakao](https://img.shields.io/badge/Kakao_Map-FFCD00?style=flat-square&logo=kakao&logoColor=black)

| 기간 | 역할 | 맡은 일 |
| --- | --- | --- |
| 2025.03 — 2025.11 (핵심 개발 3~6월) | 팀장, 풀스택 (4명 중 커밋 최다) | Firebase 구조 설계, 지도와 매장 마커, 경기 일정, 앱 구조 리팩토링 |

**한눈에 보기**

- 전국 **9개 KBO 구장**의 식음료 매장 정보를 구장별 JSON 스키마로 정리
- Kakao Map 위에 매장 위치를 핀으로 표시하고, **층, 방향, 구역, 음식 종류**로 바로 필터
- 좌표를 **비율 값**으로 저장해 화면 크기가 달라도 마커 위치가 어긋나지 않게
- 시즌 경기 일정을 크롤링해 Firestore에 저장하고, 구장별 날씨(OpenWeather)와 함께 표시

## 목차

- [문제](#문제)
- [제가 한 일](#제가-한-일)
- [주요 코드](#주요-코드)

<table align="center">
<tr>
<td align="center" width="50%">
  <img src="../assets/ballpang-1.jpg" width="260" alt="홈"><br>
  <b>① 홈</b><br>
  <sub>오늘의 경기, 지금 많이 찾는 매장 TOP 3, 오늘 경기장 추천 매장을 한눈에 봅니다</sub>
</td>
<td align="center" width="50%">
  <img src="../assets/ballpang-2.jpg" width="260" alt="경기 일정"><br>
  <b>② 경기 일정</b><br>
  <sub>달력에서 날짜를 누르면 그날 경기가 나오고, 전체, 내 팀, 홈 경기로 거를 수 있습니다</sub>
</td>
</tr>
<tr>
<td align="center" colspan="2">
  <img src="../assets/ballpang-3.jpg" width="260" alt="매장 탐색"><br>
  <b>③ 매장 탐색</b><br>
  <sub>구장, 층, 구역, 음식 종류로 매장을 거르고 리스트나 지도로 볼 수 있습니다</sub>
</td>
</tr>
</table>

## 문제

KBO 천만 관중 시대에 관람객은 늘었지만, 경기 일정, 구장 매장, 위치 정보가 여러 곳에 흩어져 있었고 넓은 구장에서 원하는 매장을 찾기 어려웠습니다.

## 제가 한 일

| 영역 | 내용 |
| --- | --- |
| 구조 설계 | Flutter Provider 상태 관리, Firebase(Auth, Firestore, Functions) 백엔드 전반 |
| 지도 | Kakao Map 위 매장 마커, 바텀시트 상세 정보, 구역, 카테고리 필터 |
| 데이터 | 9개 구장 매장 데이터를 같은 JSON 스키마로 통일 ([`assets/data/stores`](https://github.com/whdxo/ballpang_app/tree/main/assets/data/stores)) |
| 일정 | 크롤링한 시즌 일정을 Firestore에 저장, 팀별 필터와 캘린더 화면 |
| 사용성 | 비동기 로딩 중 Skeleton UI, 공통 위젯 재사용으로 화면 구조 정리 |

## 주요 코드

| 위치 | 하는 일 |
| --- | --- |
| [`lib/features/map`](https://github.com/whdxo/ballpang_app/tree/main/lib/features/map) | 지도, 매장 마커, 필터, 바텀시트 |
| [`lib/features/schedule`](https://github.com/whdxo/ballpang_app/tree/main/lib/features/schedule) | 경기 일정 캘린더 |
| [`lib/features/store`](https://github.com/whdxo/ballpang_app/tree/main/lib/features/store) | 구장별 매장 목록, 검색, 구역 필터 |

---

[← 프로필로 돌아가기](https://github.com/whdxo)
