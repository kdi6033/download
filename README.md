# download
i2r 제품의 펌웨어를 다운로드 합니다.

i2r 보드용 1분 OTA 가이드 (세 가지만 수정)

이 스케치는 i2r 보드(ESP32)가 부팅 후 GitHub에 올라있는 .bin 펌웨어를 HTTPS로 직접 다운로드 → 플래시에 기록 → 자동 재부팅까지 수행합니다.
사용자는 딱 3곳만 바꾸면 됩니다.

## 1) 수정할 곳 (3가지)
```
// 1) Wi-Fi SSID
const char* ssid     = "i2r";

// 2) Wi-Fi PASSWORD
const char* password = "00000000";

// 3) GitHub에 올려둔 펌웨어 파일명 (예: i2r-03.ino.bin)
String fileName = "i2r-03.ino.bin";
```

## 2) 사용 방법

펌웨어(.bin) 업로드

자신의 펌웨어 바이너리를 GitHub 저장소의 download/main 경로에 업로드합니다.
(이 레포 예시: https://github.com/kdi6033/download)

스케치 빌드 & 업로드

본 OTA 스케치를 보드에 업로드합니다.

시리얼 모니터(115200bps)를 열면 진행률(%)과 상태 로그가 출력됩니다.

자동 설치 & 재부팅

다운로드/검증/플래시 기록이 완료되면 자동으로 재부팅되고, 새 펌웨어가 실행됩니다.

## 3) 정상 동작 로그 예시

```
[i2r OTA Firmware Updater]
Connecting to WiFi SSID: i2r
✅ WiFi connected! IP: 192.168.x.x
Downloading from: https://github.com/.../i2r-03.ino.bin
Update Started
Progress: 5%
...
Progress: 100%
Update Finished
HTTP_UPDATE_OK
```

## 4) 자주 묻는 질문(FAQ)

파일을 못 찾는 경우
fileName 철자와 업로드 위치(브랜치/폴더)를 다시 확인하세요.

중간에 끊김(연결 손실) 발생
Wi-Fi 신호 품질과 전원(케이블/어댑터)을 점검하세요.
(팁: 대용량 파일은 라우터 근처에서 테스트하면 성공률이 높습니다.)

리다이렉트가 원인 같아요
필요 시 URL을 다음처럼 바꿔 리다이렉트를 피할 수 있습니다.
https://raw.githubusercontent.com/kdi6033/download/main/<fileName>

## 5) 보안 참고

샘플은 편의상 clientSecure.setInsecure();로 서버 인증서 검증을 생략합니다. 운영 환경에서는 루트 CA를 지정해 TLS 검증을 활성화하기를 권장합니다.
<img width="507" height="821" alt="tool" src="https://github.com/user-attachments/assets/0fdb43de-8309-49db-843f-a70cfa61ecf1" />
<img width="806" height="193" alt="file" src="https://github.com/user-attachments/assets/846920c3-1c34-4120-8dd2-94f3720c40cc" />


아두이노 소스프로그램
```
/*
 * i2r IoT PLC OTA Firmware Updater (ESP32 v3.3.0 Compatible)
 * 기능:
 *  1️⃣ Wi-Fi 연결
 *  2️⃣ GitHub .bin 파일 OTA 다운로드
 *  3️⃣ 다운로드 진행률(%) 표시
 *  4️⃣ 완료 시 자동 재부팅
 *
 * 작성자: 김동일 교수 (Doowon Univ.)
 * GitHub: https://github.com/kdi6033/download
 */

#include <WiFi.h>
#include <WiFiClientSecure.h>
#include <HTTPUpdate.h>

const char* ssid     = "i2r";        // 🔹 Wi-Fi SSID
const char* password = "00000000";   // 🔹 Wi-Fi PASSWORD
String fileName = "i2r-03.ino.bin";

// 다운로드 함수 
void download_program() {
  if (WiFi.status() == WL_CONNECTED) {
    WiFiClientSecure clientSecure;
    clientSecure.setInsecure();  // 인증서 검증 무시

    // Add optional callback notifiers
    httpUpdate.onStart([]() {
      Serial.println("Update Started");
    });
    httpUpdate.onEnd([]() {
      Serial.println("Update Finished");
    });
    httpUpdate.onProgress([](int cur, int total) {
      Serial.printf("Progress: %d%%\n", (cur * 100) / total);
    });
    httpUpdate.onError([](int error) {
      Serial.printf("Update Error: %d\n", error);
    });

    httpUpdate.setFollowRedirects(HTTPC_STRICT_FOLLOW_REDIRECTS);
    String url = "https://github.com/kdi6033/download/raw/main/" + fileName;
    Serial.println("Downloading from: " + url);
    
    // 서버에서 HTTP 응답 코드 확인 추가
    t_httpUpdate_return ret = httpUpdate.update(clientSecure, url);
    Serial.printf("HTTP Code: %d\n", clientSecure.connected() ? clientSecure.available() : -1);
  
    switch (ret) {
      case HTTP_UPDATE_FAILED:
        Serial.printf("HTTP_UPDATE_FAILD Error (%d): %s\n", httpUpdate.getLastError(), httpUpdate.getLastErrorString().c_str());
        break;

      case HTTP_UPDATE_NO_UPDATES:
        Serial.println("HTTP_UPDATE_NO_UPDATES");
        break;

      case HTTP_UPDATE_OK:
        Serial.println("HTTP_UPDATE_OK");
        break;
    }
  }
}


void setup() {
  Serial.begin(115200);

  Serial.println("\n[i2r OTA Firmware Updater]");
  Serial.printf("Connecting to WiFi SSID: %s\n", ssid);

  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);

  // WiFi 연결 대기
  while (WiFi.status() != WL_CONNECTED ) {
    delay(500);
    Serial.print(".");
  }
  Serial.println();

  if (WiFi.status() != WL_CONNECTED) {
    Serial.println("❌ WiFi connection failed! Rebooting in 5 seconds...");
    delay(5000);
    ESP.restart();
  }

  Serial.print("✅ WiFi connected! IP: ");
  Serial.println(WiFi.localIP());

  download_program();

}

void loop() {
  // OTA 실행 후 자동 재부팅하므로 loop는 비워둡니다.
  delay(1000);
}

```
partitions.csv
```
# Name,   Type, SubType, Offset,   Size,      Flags
nvs,      data, nvs,     0x9000,   0x5000,
otadata,  data, ota,     0xe000,   0x2000,
app0,     app,  ota_0,   0x10000,  0x280000,
app1,     app,  ota_1,   0x290000, 0x280000,
coredump, data, coredump,0x510000, 0x10000,
spiffs,   data, spiffs,  0x520000, 0x1E0000 
```

