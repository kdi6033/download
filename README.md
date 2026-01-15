<div align="center">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=auto&height=220&section=header&text=Download&fontSize=50&animation=fadeIn&fontAlignY=38&desc=IoT%20·%20피지컬%20AI%20·%20온디바이스%20AI%20기반의%20스마트%20공장%20자동화%20솔루션&descAlignY=55&descAlign=50" />
</div>

<div align="center">
  <a href="https://i2r.link">🌐 공식 홈페이지</a> &nbsp;&nbsp; | &nbsp;&nbsp;
  <a href="https://i2r.link/products">🛒 i2r 제품구매</a> &nbsp;&nbsp; | &nbsp;&nbsp;
  <a href="https://www.youtube.com/@i2r-link">▶️ YouTube</a>
</div>
----

i2r 제품의 최신 펌웨어를 아두이노 프로그램하여 직접 다운로드 합니다.

i2r 보드용 1분 OTA 가이드 (세 가지만 수정)

이 스케치는 i2r 보드(ESP32)가 부팅 후 GitHub에 올라있는 .bin 펌웨어를 HTTPS로 직접 다운로드 → 플래시에 기록 → 자동 재부팅까지 수행합니다.
사용자는 딱 3곳만 바꾸면 됩니다.

## 1) 와이파이정보 입력 (3가지)
사용하는 와이파이 정보와 다운로드 하려는 파일 이름을 기록 한 후에 컴파일 하면 다운로드가 진행 됩니다.

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

스케치 빌드 & 업로드

본 OTA 스케치를 보드에 업로드합니다.

시리얼 모니터(115200bps)를 열면 진행률(%)과 상태 로그가 출력됩니다.

자동 설치 & 재부팅

다운로드/검증/플래시 기록이 완료되면 자동으로 재부팅되고, 새 펌웨어가 실행됩니다.

## 2) 보안 참고

샘플은 편의상 clientSecure.setInsecure();로 서버 인증서 검증을 생략합니다. 운영 환경에서는 루트 CA를 지정해 TLS 검증을 활성화하기를 권장합니다.    
아두이노 tool 은 다음과 같이 선택하세요     
<img width="300" height="821" alt="tool" src="https://github.com/user-attachments/assets/0fdb43de-8309-49db-843f-a70cfa61ecf1" />    
partitions.csv 파일을 같은 디렉토리에 작성하세요     
<img width="500" height="193" alt="file" src="https://github.com/user-attachments/assets/846920c3-1c34-4120-8dd2-94f3720c40cc" />
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

✅  다음은 아두이노 시리얼 모니터에 아무글자나 입력하고 리턴키를 누르면 다운로드 되고 실패시 반복해서 다시 성공 할 때까지 글자를 입력하세요.    

<img width="700" alt="image" src="https://github.com/user-attachments/assets/160186ae-cd9d-4d10-9a08-9651cbc591e2" />

```
/*
 * i2r IoT PLC OTA Firmware Updater (ESP32 v3.3.0 Compatible)
 * 기능:
 *  1️⃣ Wi-Fi 연결
 *  2️⃣ GitHub .bin 파일 OTA 다운로드
 *  3️⃣ 다운로드 진행률(%) 표시
 *  4️⃣ 완료 시 자동 재부팅
 *  5️⃣ 키 입력 시 시작 / 실패 시 재시도 가능
 *
 * 작성자: 김동일 교수 (Doowon Univ.)
 * GitHub: https://github.com/kdi6033/download
 */

#include <WiFi.h>
#include <WiFiClientSecure.h>
#include <HTTPUpdate.h>

const char* ssid     = "i2r";     // 🔹 Wi-Fi SSID
const char* password = "00000000";    // 🔹 Wi-Fi PASSWORD
String fileName = "i2r-03.ino.bin";

// -----------------------------------------------------
// OTA 다운로드 함수
// -----------------------------------------------------
void download_program() {
  if (WiFi.status() == WL_CONNECTED) {
    WiFiClientSecure clientSecure;
    clientSecure.setInsecure();  // 인증서 검증 무시

    // 콜백 등록
    httpUpdate.onStart([]() {
      Serial.println("🔹 Update Started");
    });
    httpUpdate.onEnd([]() {
      Serial.println("✅ Update Finished");
    });
    httpUpdate.onProgress([](int cur, int total) {
      Serial.printf("Progress: %d%%\n", (cur * 100) / total);
    });
    httpUpdate.onError([](int error) {
      Serial.printf("❌ Update Error: %d\n", error);
    });

    httpUpdate.setFollowRedirects(HTTPC_STRICT_FOLLOW_REDIRECTS);
    String url = "https://github.com/kdi6033/download/raw/main/" + fileName;
    Serial.println("📥 Downloading from: " + url);

    // OTA 실행
    t_httpUpdate_return ret = httpUpdate.update(clientSecure, url);

    switch (ret) {
      case HTTP_UPDATE_FAILED:
        Serial.printf("❌ HTTP_UPDATE_FAILED (%d): %s\n",
                      httpUpdate.getLastError(),
                      httpUpdate.getLastErrorString().c_str());
        break;
      case HTTP_UPDATE_NO_UPDATES:
        Serial.println("⚠️  No updates available.");
        break;
      case HTTP_UPDATE_OK:
        Serial.println("✅ Update successful! Rebooting...");
        break;
    }
  } else {
    Serial.println("❌ WiFi not connected. Cannot start OTA.");
  }
}

// -----------------------------------------------------
// Wi-Fi 연결 함수
// -----------------------------------------------------
void connectWiFi() {
  Serial.printf("Connecting to WiFi SSID: %s\n", ssid);
  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);

  unsigned long startAttemptTime = millis();

  while (WiFi.status() != WL_CONNECTED && millis() - startAttemptTime < 10000) {
    delay(500);
    Serial.print(".");
  }
  Serial.println();

  if (WiFi.status() == WL_CONNECTED) {
    Serial.print("✅ WiFi connected! IP: ");
    Serial.println(WiFi.localIP());
  } else {
    Serial.println("❌ WiFi connection failed.");
  }
}

// -----------------------------------------------------
// setup()
// -----------------------------------------------------
void setup() {
  Serial.begin(115200);
  Serial.println("\n[i2r OTA Firmware Updater v2]");
  
  connectWiFi();

  while (true) {
    Serial.println("\n▶ 아무 키나 누르면 OTA 다운로드를 시작합니다...");
    while (!Serial.available()) {
      delay(100);
    }

    Serial.read();  // 입력 버퍼 비우기
    Serial.println("🔹 OTA 다운로드를 시작합니다...");

    // ✅ 실제 다운로드 실행
    download_program();

    Serial.println("\n⚙️  다운로드가 실패했거나 완료되지 않았습니다.");
    Serial.println("👉 다시 시도하려면 아무 키나 누르세요.");
    while (!Serial.available()) {
      delay(100);
    }
    Serial.read();
  }
}

void loop() {
  // OTA 완료 시 자동 재부팅되므로 loop는 비워둡니다.
}
```


