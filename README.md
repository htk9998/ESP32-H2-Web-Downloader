# ESP32-H2 Web Downloader

ESP32-H2-MINI-1 모듈용 브라우저 기반 펌웨어 다운로더.
ESP32-WROOM-32E-Web-Downloader 를 참고해 ESP32-H2 버전으로 구성.

## 사용 URL

```
https://htk9998.github.io/ESP32-H2-Web-Downloader/
```

## 사용 방법

1. 보드를 USB-UART 어댑터(데이터 케이블)로 PC 에 연결
2. 위 URL 을 Chrome / Edge 로 접속
3. bin 파일 선택 — `FW_Total_V*.bin`(전체) 또는 `FW_Update_*.bin`/앱
4. 이미지 종류 선택 → offset 자동 (Total `0x0` / App `0x10000` / 사용자지정)
5. Select COM Port → 포트 선택 → Flash 버튼 클릭
6. 완료 시 보드 자동 재부팅 (Auto Reset 해제 시 수동 리셋)

## 대상 / 이미지

| 항목 | 값 |
|:-:|:-:|
| 칩 | ESP32-H2 (RISC-V single-core, BLE 5 / 802.15.4) |
| 모듈 | ESP32-H2-MINI-1 (H2 = 2MB / H4 = 4MB Flash) |
| Total 이미지 (`FW_Total_*.bin`) | offset `0x0` (부트로더+파티션+앱 병합) |
| App 이미지 (`FW_Update_*.bin`) | offset `0x10000` (앱만) |

- ESP32-H2 는 부트로더가 **`0x0`** 에 위치한다 (ESP32 의 `0x1000` 과 다름).
  `FW_Total_V*.bin` 은 `esptool merge_bin` 으로 0x0 부터 병합한 전체 이미지 → offset `0x0`.
- `FW_Update_*.bin` (앱만) → offset `0x10000` (factory 또는 ota_0 파티션).
  OTA 2슬롯 파티션을 쓰는 펌웨어는 otadata 가 ota_1 을 가리키고 있을 수 있으므로
  앱만 굽는 경우 부팅이 안 되면 Total 이미지로 다시 굽는다.
- 파일 선택 시 이름으로 종류를 자동 판별해 offset 을 맞춰주고, 종류↔offset 이 어긋나면
  (0x0 에 앱만, 0x10000 에 전체) **확인창으로 경고**해 부팅 불능 실수를 막는다.
- 감지된 칩이 ESP32-H2 가 아니면 확인창으로 한 번 더 묻는다.

## 다운로드 모드 (수동 부트)

자동 리셋 회로(DTR/RTS → EN/IO9)가 없는 어댑터(CH9102 등)를 쓰는 보드는 수동으로 넣는다.

| 핀 | 리셋 시 상태 |
|:-:|:-:|
| IO9 (Strapping) | **LOW** (GND 연결) → 다운로드 모드 |
| IO8 (Strapping) | **HIGH** 유지 (풀업). IO8 에 연결된 신호를 누른 채 리셋 금지 |

1. IO9 를 GND 에 연결한 채 전원 투입 또는 리셋
2. 페이지에서 **Auto Reset 체크 해제** → Flash
3. 완료 후 IO9 를 GND 에서 떼고 수동 리셋

Serial 열기 버튼으로 부트 로그를 보면 `DOWNLOAD(USB/UART0)` 또는 `waiting for download` 가 다운로드 모드,
`SPI_FAST_FLASH_BOOT` 가 일반 부팅이다.

## 브라우저 요구사항

| 브라우저 | 지원 |
|:-:|:-:|
| Chrome 89+ | 지원 |
| Edge 89+ | 지원 |
| Opera 76+ | 지원 |
| Firefox | 미지원 (WebSerial 없음) |
| Safari | 미지원 (WebSerial 없음) |

## 기술 스택

- HTML5 + Vanilla JS (프레임워크 없음)
- esptool-js v0.4.5 (CDN : jsdelivr, ESP32-H2 지원)
- WebSerial API (chip 통신, 칩 자동 감지)
