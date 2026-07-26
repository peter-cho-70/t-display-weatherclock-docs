# 개발 기록 (DEVLOG)

> 날짜별로 새 섹션을 계속 append한다(과거 항목은 지우거나 고치지 않음 — 현재 상태 요약은 STATUS.md 몫). 각 항목은 무엇을 했는지뿐 아니라 **수정/버그/이슈**를 구분해 적는다.

## 2026-07-26 — WeatherClock 최초 구현 및 실기 검증

- **추가/변경**:
  - Wi-Fi 연결 + NTP 시각 동기화 + Open-Meteo 날씨 조회를 하나로 묶은 `examples/WeatherClock` 예제 신규 작성. TFT_eSPI 320×170 가로 레이아웃 사용.
  - WiFiManager 캡티브 포털 기반 설정 UI 추가: 저장된 Wi-Fi 정보가 없으면 ESP32가 "T-Display-S3-Setup" AP를 열고, 화면에 접속 안내(SSID, IP)를 표시. 폰/PC로 접속한 브라우저 폼에서 Wi-Fi SSID/PW + 도시명(City) + UTC 오프셋을 입력받아 저장.
  - 도시명은 Open-Meteo 지오코딩 API(`geocoding-api.open-meteo.com`)로 위도/경도를 자동 조회, NVS(Preferences, 네임스페이스 `wclock`)에 캐싱해 도시가 바뀔 때만 재조회.
  - BOOT 버튼(GPIO0, `PIN_BUTTON_1`) 3초 홀드 시 저장된 Wi-Fi/설정 초기화 후 재부팅 → 설정 화면 재진입.
  - BUTTON_2(GPIO14, `PIN_BUTTON_2`) 짧게 누르면 날씨 즉시 재조회하도록 추가.
  - 레이아웃을 날씨 중심으로 재설계: 기온(큰 숫자)·체감온도·날씨상태(WMO 코드 → 텍스트)·습도·풍속/풍향(8방위)·강수량을 표시. 시계는 우상단 "HH:MM"만 작게(분 단위 갱신) 축소, 초 단위 표시는 제거.
- **버그 수정**:
  - **HTTPS 호출이 조용히 실패**: ESP32 Arduino의 기본 `loop()` 태스크 스택이 8KB인데, `WiFiClientSecure`의 TLS 처리 + `WiFiManager` + `TFT_eSPI` + `ArduinoJson`이 겹치면서 스택이 부족해 HTTPS 요청이 실패하는 것으로 판단. `SET_LOOP_TASK_STACK_SIZE(16 * 1024)`로 스택을 16KB로 확장해 해결.
  - **`JSON error: Invalid input`**: Open-Meteo API가 `Transfer-Encoding: chunked`로 응답(`curl -D -`로 헤더 직접 확인)하는데, 기존 코드가 `HTTPClient::getStream()`을 `ArduinoJson::deserializeJson()`에 그대로 넘겨서 HTTP 청크 프레이밍 바이트가 JSON 파서에 섞여 들어간 것이 원인. `http.getString()`으로 청크를 완전히 해제한 문자열을 받은 뒤 파싱하도록 수정 → 실기 업로드 후 날씨 정상 표시 확인.
  - 실패 시 원인을 알 수 없던 문제 → 화면에 실제 HTTP 상태코드/에러 메시지(`weather.errorMsg`)를 표시하도록 개선, 부팅 직후 첫 조회는 최대 3회 자동 재시도 추가.
- **이슈/막힌 점**:
  - PlatformIO 업로드 시 **USB 포트가 인식되지 않는 문제**가 있었음. T-Display-S3는 별도 USB-UART 브릿지 칩 없이 ESP32-S3 **네이티브 USB(CDC)**를 사용하는데, 칩에 `ARDUINO_USB_CDC_ON_BOOT=1`로 빌드된 펌웨어가 한 번도 올라간 적이 없으면(빈 칩이거나 CDC 미지원 펌웨어 상태) 업로드 툴의 자동 리셋(부트로더 진입) 신호에 응답할 펌웨어 자체가 없어 포트가 안 잡힘. Arduino IDE로 먼저 성공적으로 업로드(또는 BOOT 버튼 누른 채 RST 눌렀다 떼는 수동 다운로드 모드 진입, README FAQ #4와 동일)하면 그 이후부터는 PlatformIO 업로드도 자동으로 정상 작동함 — Arduino IDE 자체의 특별한 기능이 아니라, "CDC 지원 펌웨어가 한 번은 칩에 심어져야 자동 리셋이 동작한다"는 하드웨어 특성 때문.
