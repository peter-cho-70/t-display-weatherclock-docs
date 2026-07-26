# 개발 현황 (STATUS)

> 기준일: 2026-07-26 — 이 문서는 append하지 않고 매번 "현재 기준"으로 덮어써서 갱신한다. 히스토리는 DEVLOG.md에 남긴다.

## ✅ 완료

- LilyGO T-Display-S3용 Wi-Fi 시계 + 날씨 디스플레이 예제(`examples/WeatherClock`) 최초 구현.
- TFT_eSPI 기반 320×170 가로 레이아웃, [Open-Meteo](https://open-meteo.com/) API(API 키 불필요, 무료)로 위치·날씨 조회.
- WiFiManager 캡티브 포털 기반 설정 UI: 최초 부팅 시 AP 모드로 전환되어 폰/PC 브라우저로 Wi-Fi SSID/비밀번호 + 도시명 + UTC 오프셋을 입력받음. 도시명은 Open-Meteo 지오코딩 API로 위경도 자동 변환.
- 설정값(Wi-Fi, 도시, 좌표)은 NVS(Preferences)에 영구 저장 — 재부팅해도 유지, BOOT 버튼 3초 홀드로 초기화 가능.
- 날씨 중심 레이아웃: 기온·체감온도·날씨상태·습도·풍향(8방위)·강수량 표시. 시계는 우상단에 "HH:MM"만 작게 축소(분 단위 갱신).
- BUTTON_2(GPIO14) 짧게 눌러 날씨 즉시 수동 새로고침.
- 실제 보드에 업로드하여 정상 동작 확인 완료 (2026-07-26).

## 🚧 진행 중

- 없음 — 현재 기능 단위는 완결된 상태.

## 🐞 알려진 이슈

- City 입력란은 **영문 도시명**을 권장. 한글 등 비영문 입력 시 Open-Meteo 지오코딩 정확도가 떨어질 수 있음(URL 인코딩 자체는 처리되어 있음).
- 온도 단위 "°C"의 도(°) 기호와 한글 텍스트는 TFT_eSPI 내장 비트맵 폰트(Font2/4/6/7/8)에 글리프가 없어 표시하지 않음("27C" 형태로만 표시). 필요하면 커스텀 폰트(`factory` 예제의 `font_Alibaba.c` 방식)를 추가해야 함.
- HTTPS 통신 시 `WiFiClientSecure::setInsecure()`로 인증서 검증을 생략함 — 공개 날씨 데이터를 읽기만 하는 개인 프로젝트라 실용적으로 단순화한 것으로, 민감정보 송수신이 없어 문제되지 않지만 참고할 것.
- Open-Meteo API가 `Transfer-Encoding: chunked`로 응답하므로, `HTTPClient`에서 JSON을 파싱할 때는 반드시 `getString()`으로 청크를 해제한 뒤 `deserializeJson()`에 넘겨야 함(`getStream()` 직결 시 "Invalid input" 오류 발생) — 코드에 이미 반영됨, 향후 다른 API 연동 시 동일한 패턴 주의.

## 🗺️ 다음 우선순위

- (미정) 필요 시: 한글 표시를 위한 커스텀 폰트 추가, 날씨 아이콘 표시, 시간별/일별 예보 확장 등을 검토.
