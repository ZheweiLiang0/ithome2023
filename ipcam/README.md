# 用 IP 攝影機辨識車牌

瀏覽器沒辦法直接讀 RTSP，所以用 [go2rtc](https://github.com/AlexxIT/go2rtc) 轉成 MJPEG / JPEG，
再由 `plate.html` 的「IP 攝影機」模式讀取畫面辨識。

```
IP 攝影機 ──RTSP──> go2rtc ──MJPEG / JPEG──> plate.html（手機瀏覽器）
```

## 1. 選一台機器跑 go2rtc

- **Android 手機本身**（用網路線轉 Type-C 接 PoE 主機時）：安裝 Termux
- **樹莓派 / 電腦**：和攝影機在同一個網路即可

go2rtc 轉 MJPEG / JPEG 需要 ffmpeg。

### Android（Termux）

```sh
pkg install ffmpeg wget
wget https://github.com/AlexxIT/go2rtc/releases/latest/download/go2rtc_linux_arm64
chmod +x go2rtc_linux_arm64
mkdir -p www
cp plate.html www/          # 從這個 repo 複製
./go2rtc_linux_arm64 -config go2rtc.yaml
```

手機瀏覽器打開 `http://127.0.0.1:1984/plate.html`。
`127.0.0.1` 算安全來源，所以「手機鏡頭」和「IP 攝影機」兩種模式都能用。

### 樹莓派 / Linux 電腦

步驟同上，下載對應平台的 go2rtc（例如 `go2rtc_linux_arm64`、`go2rtc_linux_amd64`），
手機打開 `http://<那台的 IP>:1984/plate.html`。
這時網頁不是 https，「手機鏡頭」模式會被瀏覽器擋，只能用「IP 攝影機」模式。

## 2. 改 go2rtc.yaml

把 `cam1` 的 RTSP 網址換成你的攝影機。先用電腦打開 `http://<IP>:1984/`，確認看得到畫面。

## 3. 在 plate.html 連線

切到「IP 攝影機」，網址填：

| 類型 | 網址 |
| --- | --- |
| MJPEG 串流（較流暢） | `http://<IP>:1984/api/stream.mjpeg?src=cam1` |
| JPEG 截圖（較省電） | `http://<IP>:1984/api/frame.jpeg?src=cam1` |

按「連線」，對準車牌後按「辨識」，或勾選「連續辨識」每 1.5 秒自動辨識一次。

## 網路線直連攝影機時的 IP 設定

PoE **交換器**不會發 IP，手機要手動設在攝影機的網段，例如攝影機是 `192.168.1.64`，
手機就設 `192.168.1.100`、遮罩 `255.255.255.0`。接 **NVR** 時看 NVR 給攝影機的網段。

## 常見問題

- **「連不到攝影機」**：網址錯、不同網段，或網頁是從 https（如 GitHub Pages）打開的。請改從 go2rtc 的網址打開。
- **「不允許讀取攝影機畫面」**：go2rtc 沒設 `origin: "*"`，或直接連攝影機而攝影機不支援 CORS。
