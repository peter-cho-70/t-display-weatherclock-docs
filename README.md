# T-Display-S3 WeatherClock — 문서 사이트

LilyGO T-Display-S3(ESP32-S3)용 Wi-Fi 시계 + 날씨 디스플레이 예제 **WeatherClock**의 공개 문서 사이트. 소스코드는 [LilyGO의 T-Display-S3 예제 저장소](https://github.com/Xinyuan-LilyGO/T-Display-S3) 안의 `examples/WeatherClock`에 있다.

## 무엇을 만들었나

- Wi-Fi에 연결해 NTP로 시간을 맞추고, [Open-Meteo](https://open-meteo.com/) API(무료, API 키 불필요)로 현재 날씨를 받아와 320×170 화면에 표시.
- 코드를 고쳐서 재업로드하지 않고도, 기기가 띄우는 웹 설정 화면(캡티브 포털)에서 Wi-Fi와 위치(도시명)를 바꿀 수 있음.
- 기온·체감온도·날씨상태·습도·풍향·강수량을 보여주는 날씨 중심 레이아웃, 시계는 우상단에 작게 표시.

## 문서

문서 전체 목록은 **[doc/index.html](doc/index.html)** 허브 페이지에서 검색·탐색할 수 있다.

- [설치 및 USB 연결 가이드](doc/guide/setup.md) — 빌드 환경, USB 포트 인식 문제 해결, 최초 Wi-Fi/위치 설정, 트러블슈팅.
- [개발 현황 (STATUS)](doc/STATUS.md)
- [개발 기록 (DEVLOG)](doc/DEVLOG.md)
