# 설치 및 USB 연결 가이드

> 작성일: 2026-07-26

LilyGO T-Display-S3에 `WeatherClock` 예제를 처음 굽고, Wi-Fi/위치를 설정해 정상 동작시키기까지의 전체 절차.

## 1. 빌드 환경

- PlatformIO(VS Code 확장 또는 CLI).
- 저장소 루트 `platformio.ini`에서 아래 줄의 주석을 조정해 `WeatherClock`을 활성 환경으로 지정:
  ```ini
  [platformio]
  ; default_envs = T-Display-S3-MIDI
   default_envs = WeatherClock
  ```
- `[env:WeatherClock]`은 `ArduinoJson`, `WiFiManager` 라이브러리를 자동으로 받아온다(최초 빌드 시 인터넷 필요).

## 2. USB 연결 — 포트가 안 잡힐 때

T-Display-S3는 **별도 USB-UART 브릿지 칩(CP2102/CH340 등)이 없고, ESP32-S3 칩 자체의 네이티브 USB**를 그대로 시리얼 포트로 사용한다. 이 방식의 특성 때문에:

- 정상적으로 `ARDUINO_USB_CDC_ON_BOOT=1`로 빌드된 펌웨어가 이미 칩에서 돌고 있으면, 업로드 툴(`esptool`)이 보내는 리셋 신호에 펌웨어가 응답해서 **자동으로** 다운로드 모드로 들어간다.
- 반대로 칩이 **빈 상태이거나, USB CDC를 쓰지 않는 펌웨어**가 올라가 있으면 응답할 소프트웨어가 없어서 자동 진입이 실패하고, 포트 자체가 안 잡히거나 업로드가 멈춘다.

**해결 — 수동으로 다운로드 모드 진입 (최초 1회만 필요한 경우가 많음):**

1. USB 케이블로 보드를 연결한다.
2. **BOOT** 버튼을 누른 상태로 유지한다.
3. BOOT을 누른 채로 **RST(리셋)** 버튼을 눌렀다 뗀다.
4. BOOT 버튼을 뗀다.
5. 이 상태에서 업로드를 실행한다.
6. 업로드가 끝나면 RST를 한 번 눌러 정상 실행 모드로 나온다.

이 절차로 `ARDUINO_USB_CDC_ON_BOOT=1` 펌웨어가 한 번 칩에 올라가면, **그 다음부터는 PlatformIO에서 자동 업로드가 정상 작동**한다(Arduino IDE로 먼저 구웠을 때 이후 PlatformIO가 잘 되는 것도 같은 원리 — 처음으로 CDC 지원 펌웨어가 심어졌기 때문).

## 3. 최초 부팅 — Wi-Fi / 위치 설정

업로드 후 처음 켜지면 저장된 Wi-Fi 정보가 없으므로 기기가 자체 Wi-Fi 접속점(AP)을 연다. 화면에 아래처럼 안내가 뜬다:

```
WiFi Setup Mode
1. Connect phone/PC WiFi to: T-Display-S3-Setup
2. Open browser: http://192.168.4.1
```

1. 폰이나 PC의 Wi-Fi를 **T-Display-S3-Setup**에 연결한다.
2. 대부분 자동으로 설정 페이지가 뜬다(캡티브 포털). 안 뜨면 브라우저로 `http://192.168.4.1` 접속.
3. 페이지에서:
   - 집에서 쓰는 **Wi-Fi SSID/비밀번호** 선택.
   - **City**: 날씨를 조회할 도시의 **영문명** (예: `Seoul`, `Busan`). 한글 입력도 동작은 하지만 지오코딩 정확도가 떨어질 수 있어 영문을 권장.
   - **UTC offset hours**: 한국은 `9`.
4. 저장하면 기기가 입력한 Wi-Fi로 접속을 시도하고, 성공하면 재부팅 없이 바로 시계+날씨 화면으로 넘어간다.

설정한 값은 기기 내부(NVS)에 저장되어 **재부팅해도 유지**된다.

## 4. 버튼 기능

| 버튼 | 동작 |
|---|---|
| BOOT (`PIN_BUTTON_1`, GPIO0) | 전원을 켤 때부터 **3초 이상** 누르고 있으면 저장된 Wi-Fi/설정을 모두 초기화하고 재부팅 → 다시 설정 화면(AP 모드) 진입 |
| BUTTON_2 (`PIN_BUTTON_2`, GPIO14) | 짧게 누르면 날씨를 즉시 다시 조회 (10분 자동 갱신을 기다리지 않고 바로 테스트하고 싶을 때) |

## 5. 업로드 전에 API가 정상인지 브라우저로 먼저 확인하기

매번 굽고 기다리는 대신, 펌웨어가 실제로 보내는 것과 동일한 URL을 브라우저 주소창에 붙여넣어 먼저 확인할 수 있다.

**위치 조회(지오코딩)** — City 이름을 넣어서:
```
https://geocoding-api.open-meteo.com/v1/search?count=1&name=Seoul
```

**날씨 조회** — 위에서 나온 `latitude`/`longitude` 값으로:
```
https://api.open-meteo.com/v1/forecast?latitude=37.5665&longitude=126.9780&current=temperature_2m,relative_humidity_2m,weather_code,wind_speed_10m,wind_direction_10m,apparent_temperature,precipitation
```

브라우저에서 JSON이 바로 뜨면 API/네트워크는 정상이므로 문제는 기기 쪽이다. 뜨지 않으면 파라미터나 네트워크 쪽 문제다.

## 6. 화면에 "Weather fetch failed"가 뜰 때

날씨 패널에 실패 사유가 그대로 표시된다:

| 메시지 | 의미 |
|---|---|
| `WiFi disconnected` | Wi-Fi 연결이 끊긴 상태 |
| `No location set` | 지오코딩 실패로 좌표가 없는 상태 |
| `HTTP error -1` 등 음수 | TLS 연결/핸드셰이크 자체가 실패 (DNS 실패 등) |
| `HTTP error 4xx/5xx` | 서버는 응답했지만 요청이 잘못됨 |
| `JSON error: ...` | 응답은 받았지만 파싱 실패 |

PlatformIO 시리얼 모니터를 열어두면 동일한 내용이 `Serial.printf`로 실시간으로 찍혀 더 편하게 확인할 수 있다.
