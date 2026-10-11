<div align="center">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=auto&height=220&section=header&text=Download&fontSize=50&animation=fadeIn&fontAlignY=38&desc=IoT%20·%20피지컬%20AI%20·%20온디바이스%20AI%20기반의%20스마트%20공장%20자동화%20솔루션&descAlignY=55&descAlign=50" />
</div>

<div align="center">
  <a href="https://i2r.link">🌐 공식 홈페이지</a> &nbsp;&nbsp; | &nbsp;&nbsp;
  <a href="https://i2r.link/products">🛒 i2r 제품구매</a> &nbsp;&nbsp; | &nbsp;&nbsp;
  <a href="https://www.youtube.com/@i2r-link">▶️ YouTube</a>
</div>

----

# i2r 보드 펌웨어 설치하기

웹 브라우저에서 **USB 케이블만 연결**하면 i2r 보드와 Otto 로봇에 최신 펌웨어를 설치할 수 있습니다.
프로그램을 따로 설치할 필요가 없습니다.

## 준비물

- 컴퓨터(Windows, Mac) + **크롬(Chrome)** 또는 **엣지(Edge)** 브라우저 (스마트폰, 아이폰은 안 됩니다)
- USB 케이블 (데이터 전송이 되는 케이블)
- i2r 보드 또는 Otto 로봇

## 설치 방법 (5단계)

1. 보드를 USB 케이블로 컴퓨터에 연결합니다. (Otto는 **PROG** 라고 쓰인 포트에 연결합니다)
2. 크롬이나 엣지에서 **[i2r.link/firmware](https://i2r.link/firmware)** 를 엽니다. (Otto 는 기기 화면의 **펌웨어 다운로드** 버튼)
3. **내 보드와 같은 그림**을 누릅니다. 아래 모델 사진을 보고 확인하세요 → [모델 사진 보기](images/README.md)
4. **COM 포트 선택하고 설치 시작** 을 누르고, 목록에서 보드가 연결된 포트(예: `USB-SERIAL CH340`)를 고릅니다.
5. 진행률이 100% 가 되고 **설치 완료** 가 나오면 끝입니다. 보드가 저절로 다시 시작합니다.

## 꼭 읽어 주세요

- 설치하면 보드에 저장된 **Wi-Fi 설정이 지워집니다.** 설치가 끝나면 다시 입력하세요. ([i2r.link](https://i2r.link) 로그인 → 기기연결)
- 설치 중에는 **USB 를 뽑거나 브라우저 창을 닫지 마세요.**
- 아두이노 **시리얼 모니터는 닫아** 두세요. (같은 포트를 같이 쓸 수 없습니다)
- 포트 목록에 보드가 안 보이면 USB 케이블을 바꿔 보거나 다른 USB 구멍에 꽂아 보세요.
- 연결이 안 되면 보드의 **BOOT 버튼을 누른 채** 설치 시작을 다시 눌러 보세요.
- 펌웨어에는 개인 정보(Wi-Fi 비밀번호, API 키)가 들어 있지 않습니다. 그런 값은 설치 후 각자 입력합니다.

## 이 저장소의 구성

| 위치 | 내용 |
|---|---|
| `모델이름.ino.merged.bin` | 모델별 펌웨어 (예: `i2r-02.ino.merged.bin`, `otto.ino.merged.bin`) |
| `images/모델이름.png` | 펌웨어 다운로드 페이지에 나오는 모델 사진 |
| `i2r02`, `i2r03`, `i2r04`, `hmi` | 소스 코드(zip) |

새 모델을 추가할 때는 펌웨어와 사진을 **같은 모델 이름**으로 올리면 다운로드 페이지에 자동으로 나타납니다.
