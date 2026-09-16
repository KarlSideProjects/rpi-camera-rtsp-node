# rpi-camera-rtsp-node

<img src="./docs/assets/rpi-camera-rtsp-node.svg" width="96" alt="Raspberry Pi 相機串流節點圖示">

**把 Raspberry Pi 與 CSI 相機變成需要帳密的網路影像來源，供既有 NVR、瀏覽器或電腦視覺主機接收。**

適合想自行部署相機串流的使用者與系統整合者：Pi 負責擷取與硬體 H.264 編碼，下游主機負責錄影或分析。本庫提供**安裝文件、疑難排解與二進位發行套件**，不包含 Node Agent 原始碼。

現在可從 [Releases](https://github.com/KarlSideProjects/rpi-camera-rtsp-node/releases) 取得套件，在接有 CSI 相機的 Pi 上安裝後，以 VLC、ffplay 或瀏覽器觀看即時串流。沒有硬體時，可先閱讀 [相機偵測故障案例](./docs/question-and-answer.zh-TW.md)；本庫沒有可直接操作的線上展示。

**繁體中文** | [English](./README.en.md)

## 解決什麼問題

相機擷取、串流服務、帳密、開機啟動與下游接入若各自手動組裝，換一張 SD 卡或一台 Pi 就要重新排查。本專案把這些步驟收斂成可重複安裝的節點，讓整合者先取得標準 RTSP 入口，再選擇 NVR 或 AI/CV 工具。

| 可用能力 | 使用者得到的結果 |
| --- | --- |
| 硬體 H.264 串流 | 一次編碼，提供 RTSP、WebRTC 與 HLS 入口 |
| 讀取帳密與 IP allowlist | 可限制哪些使用者與網路端點讀取畫面 |
| 安裝器與 systemd service | 部署 Node binary、MediaMTX、設定與服務 |
| 分級硬體設定 | Zero 2 W 參考設定與原始 Zero 的低資源設定 |
| 公開 Q&A 與採購檢查文件 | 可依症狀排查硬體／OS，也可建立自己的 BOM 證據 |

錄影、物件辨識、YOLO 推論與中央管理由下游系統承擔；本節點不提供這些功能。與特定 NVR／AI 工具的相容性仍需在實際環境驗證。

## 如何運作

```text
CSI 相機 → Raspberry Pi／MediaMTX（硬體 H.264）
                           ├─ RTSP → go2rtc／NVR／AI 主機
                           ├─ WebRTC → 瀏覽器
                           └─ HLS → 相容播放器
```

MediaMTX 負責影像串流，Node Agent 負責設定、服務控制、mDNS 與狀態檢查，不處理影像 frame。把影像處理留在較有算力的主機，可減少 Pi 的工作，但也意味著需要另建錄影或分析系統。

## 快速開始

準備有 CSI 相機的 Raspberry Pi、電源、網路及可辨識相機的 OS。先確認 `rpicam-hello --list-cameras`（舊工具為 `libcamera-hello --list-cameras`）能列出相機。

在 Pi 上執行：

```bash
curl -fsSL https://github.com/KarlSideProjects/rpi-camera-rtsp-node/releases/latest/download/install.sh |   bash -s --     --read-username viewer     --read-password '<your-password>'
```

密碼由安裝者設定，installer 不會替你產生或公開列印。需要限制來源 IP 時，可加 `--read-ip-allowlist '<client-ip-or-cidr>'`。完整帳密、架構判斷與相機排查請見 [Q&A](./docs/question-and-answer.zh-TW.md)。

從同一 LAN 或 VPN 的電腦，以安裝時設定的密碼測試；若設有 allowlist，測試電腦也需在清單內：

```bash
ffplay "rtsp://viewer:<read-password>@<node-host>:8554/cam"
```

| 協議 | 預設入口 |
| --- | --- |
| RTSP | `rtsp://<user>:<pass>@<node-host>:8554/cam` |
| WebRTC | `http://<node-host>:8889/cam/` |
| HLS | `http://<node-host>:8888/cam/index.m3u8` |

WebRTC／HLS 使用相同讀取帳密。手動組 RTSP URL 時，帳密中的特殊字元須 percent-encode。go2rtc 下游設定範例：

```yaml
streams:
  rpi_camera:
    - rtsp://viewer:<read-password>@<node-host>:8554/cam
```

常用診斷：

```bash
rpicam-hello --list-cameras
sudo systemctl status rpi-camera-mediamtx.service
```

## 硬體與驗證邊界

| 類型 | 硬體／設定 | 如何解讀 |
| --- | --- | --- |
| Reference | Zero 2 W、64-bit OS；1280×720 約 30 fps | 發行驗收參考目標，實際結果仍受 OS、相機與網路影響 |
| Constrained | 原始 Pi Zero／Zero W、ARMv6；640×480 約 15 fps | 降低資源需求的設定，不等於 Reference 驗收 |
| Integration proxy | Pi 3B | 可作整合測試，不能代替 Zero 2 W 實機結果 |

2026-09-16 查核時，公開最新發行為 [v0.1.17](https://github.com/KarlSideProjects/rpi-camera-rtsp-node/releases/tag/v0.1.17)，包含安裝器、ARM64／ARMHF／ARMv6／ARMv6 legacy 套件與 `release-manifest.json`。發行檔存在可證明可取得交付物，不代表本次已重跑實機串流或效能驗收。

[Q&A 的 Pi 3B／OV5647 案例](./docs/question-and-answer.zh-TW.md) 保留了「燈亮但相機未列舉」的訊息、device-tree 與 I2C 排查。它呈現診斷過程與尚待硬體排除的原因，不應當成該相機已修復的證明。

## 可延伸的工程探究

可用同一相機比較不同解析度、幀率及網路條件下的播放穩定性，記錄控制變因與觀測結果；也可從 Q&A 練習區分「有供電」「驅動啟用」與「感測器真的回應」三種證據。這是可設計的探究活動，尚無本庫提供的教學成效研究。

## 供應鏈、授權與來源

硬體產地須依實際料號、收貨標籤與採購文件確認；本專案不保證完整 BOM 產地，也不提供採購合規背書。導入者可使用 [供應鏈自我驗證清單](./docs/supply-chain-verification.zh-TW.md)。本專案不含完整商用 IP camera 的外殼、保固、PoE 或 SLA。

依 [LICENSE](./LICENSE)，本產品為專有、非商業用途二進位軟體，沒有授予原始碼或任意再散布權利。商業使用、企業營運與 SI／政府標案導入請先聯絡權利人確認授權。隨附的 MediaMTX 依其 MIT 授權處理，不能把該元件授權套用到整個產品。

- 一般商用詢問：[Commercial inquiry](https://github.com/KarlSideProjects/rpi-camera-rtsp-node/issues/new?template=commercial-license.yml)。
- 報價、採購、NDA 與敏感部署資訊：<jhihweijhan@gmail.com>。
- 英文疑難排解：[Question and Answer](./docs/question-and-answer.md)。

請勿在公開 issue 貼出憑證、內部 RTSP URL、網段、攝影機位置、客戶資訊或 NDA 內容。
