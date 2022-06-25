<p align="center">
    <img width="200px;" src="https://raw.githubusercontent.com/woowacourse/atdd-subway-admin-frontend/master/images/main_logo.png"/>
</p>
<p align="center">
  <img alt="npm" src="https://img.shields.io/badge/npm-%3E%3D%205.5.0-blue">
  <img alt="node" src="https://img.shields.io/badge/node-%3E%3D%209.3.0-blue">
  <a href="https://edu.nextstep.camp/c/R89PYi5H" alt="nextstep atdd">
    <img alt="Website" src="https://img.shields.io/website?url=https%3A%2F%2Fedu.nextstep.camp%2Fc%2FR89PYi5H">
  </a>
  <img alt="GitHub" src="https://img.shields.io/github/license/next-step/atdd-subway-service">
</p>

<br>

# 인프라공방 샘플 서비스 - 지하철 노선도

<br>

## 🚀 Getting Started

### Install
#### npm 설치
```
cd frontend
npm install
```
> `frontend` 디렉토리에서 수행해야 합니다.

### Usage
#### webpack server 구동
```
npm run dev
```
#### application 구동
```
./gradlew clean build
```
<br>


### 1단계 - 웹 성능 테스트
#### 1. 웹 성능예산은 어느정도가 적당하다고 생각하시나요

**1-1. 내 사이트 분석**   
1-1-1. [내 사이트](https://ttungga.r-e.kr/)

| Device | FCP   | TTI   | SI    | TBT   | LCP   | CLS   | Score |
|--------|-------|-------|-------|-------|-------|-------|-------|
| Mobile | 14.6s | 15.2s | 14.6s | 590ms | 15.2s | 0.041 | 31    |
| PC     | 2.7s  | 2.8s  | 2.7s  | 50ms  | 2.8s  | 0.004 | 68    |

**1-2. 경쟁사 분석**   
1-2-1. [서울교통공사](http://www.seoulmetro.co.kr/kr/cyberStation.do)

| Device | FCP  | TTI  | SI   | TBT   | LCP  | CLS   | Score |
|--------|------|------|------|-------|------|-------|-------|
| Mobile | 7.2s | 8.7s | 9.1s | 400ms | 7.7s | 0     | 41    |
| PC     | 1.6s | 2.0s | 2.5s | 150ms | 3.6s | 0.014 | 66    |

1-2-2. [네이버 지도](https://map.naver.com/v5/subway)

| Device | FCP  | TTI  | SI   | TBT    | LCP  | CLS   | Score |
|--------|------|------|------|--------|------|-------|-------|
| Mobile | 2.1s | 2.4s | 2.8s | 40ms   | 3.8s | 0     | 86    |
| PC     | 0.5s | 4.7s | 4.1s | 1040ms | 2.4s | 0.019 | 44    |

1-2-3. [카카오맵](https://map.kakao.com/)

| Device | FCP  | TTI  | SI   | TBT   | LCP  | CLS   | Score |
|--------|------|------|------|-------|------|-------|-------|
| Mobile | 1.7s | 4.4s | 6.8s | 60ms  | 4.6s | 0.005 | 75    |
| PC     | 0.6s | 2.5s | 3.1s | 430ms | 0.7s | 0.018 | 72    |

**1-3. 성능예산**
* FCP
  * Mobile: 2.0s 이하
  * PC: 1.0s 이하
* TTI
  * Mobile: 2.0s 이하 (네이버 지도 대비 20% 향상)
  * PC: 1.8s 이하 (서울교통공사 대비 20% 향상)

#### 2. 웹 성능예산을 바탕으로 현재 지하철 노선도 서비스는 어떤 부분을 개선하면 좋을까요
* 텍스트 기반 리소스 압축 ex) gzip, minify
  * /js/vendors.js (2,125.0 KiB)
  * /js/main.js (172.0 KiB)
* 캐싱 정책 설정하기
  * [변경이 적은 정적 리소스에 대한 캐시 컨트롤 설정](https://web.dev/http-cache/)
  * 백엔드 서버에서 변경이 적은 데이터 조회 API에 대한 캐시 설정 ex) 노선 목록 조회 등

---

### 2단계 - 부하 테스트 
#### 1. 부하테스트 전제조건은 어느정도로 설정하셨나요
* 예상 1일 사용자 수(DAU): 44만
  * 1일 지하철 승차인원의 약 50% = 220만 / 우리 앱의 목표 점유율 20%
* 1명당 1일 평균 요청 수: 6회
  * 메인 페이지 접속 + 로그인 + 경로 조회 페이지 접속 + 경로 조회: 4회
  * 메인 페이지 접속 + 경로 조회 페이지 접속 + 경로 조회: 2회
  * 지하철을 타는 사람은 대부분 집에 돌아와야 함: 하루에 2번 서비스 접속
  * 갈 때는 로그인 후 경로를 조회하고 올 때는 로그인 안 한다고 가정
* 피크 시간대의 집중률: 4
  * 출퇴근 시간에 4배의 사용자가 몰린다고 가정
* Throughput
  * 1일 총 요청 수: 264만
  * 1일 평균 rps: 30
  * 1일 최대 rps: 120
  * Latency: 100ms 이하
* VUser
  * T(VU iteration) = 6(요청 수) * 0.1(http_req_duration) (+1s 지연시간) = 1.6
  * min VUser = (30 * 1.6) / 5 = 10
  * max VUser = (120 * 1.6) / 5 = 38
* 부하 테스트 시 저장될 데이터 건수 및 크기
  * Station : 616
  * Section: 340
  * Line: 23

#### 2. Smoke, Load, Stress 테스트 스크립트와 결과를 공유해주세요
loadtest 디렉토리에 파일 정리하였습니다.

---

### 3단계 - 로깅, 모니터링
1. 각 서버내 로깅 경로를 알려주세요

2. Cloudwatch 대시보드 URL을 알려주세요
