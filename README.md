# jt-live-whisper v2.16.1 - video recording version

100% 全地端 AI 語音工具集：即時轉錄、即時翻譯、雙向字幕、錄影錄音、離線逐字稿、講者辨識、AI 摘要、WebUI 遠端操作、HDMI 分割輸出。

- Author: Jason Cheng (Jason Tools)、貓又 CatAgain
- License: Apache-2.0

---

## 目錄

- [你可以做到什麼](#你可以做到什麼)
- [快速開始](#快速開始)
- [最常用啟動方式](#最常用啟動方式)
- [HDMI 分割輸出](#hdmi-分割輸出)
- [WebUI 使用方式](#webui-使用方式)
- [離線處理與摘要](#離線處理與摘要)
- [命令列參數總表](#命令列參數總表)
- [輸出檔案說明](#輸出檔案說明)
- [系統需求](#系統需求)
- [常見問題](#常見問題)

---

## 你可以做到什麼

### 即時能力

- 即時語音轉錄（英/中/日）
- 即時翻譯（英翻中、中翻英、日翻中、中翻日）
- 雙向字幕（en_zh、ja_zh）
- 同時轉錄麥克風（--mic）
- 即時錄音（--record）
- 即時錄影（--record-video，可疊攝影機）
- 即時 HDMI 分割輸出（最多 4 路來源）
- 懸浮字幕（--subtitle-overlay）

### 離線能力

- 匯入音訊檔離線轉錄（mp3/wav/m4a/flac）
- 講者辨識（--diarize）
- AI 摘要/逐字稿校正（--summarize）
- 匯出 TXT / HTML / SRT / VTT

### 使用介面

- 互動式選單（不帶參數）
- CLI（帶參數直接跑）
- WebUI（瀏覽器設定與監看）

---

## 快速開始

### 1) 安裝

macOS:

```bash
mkdir -p ~/Apps/jt-live-whisper && cd ~/Apps/jt-live-whisper
curl -fsSL https://raw.githubusercontent.com/jasoncheng7115/jt-live-whisper/main/install.sh -o install.sh
bash install.sh
```

Windows (PowerShell):

```powershell
mkdir C:\jt-live-whisper -Force | Out-Null; cd C:\jt-live-whisper
irm https://raw.githubusercontent.com/jasoncheng7115/jt-live-whisper/main/install.ps1 -OutFile install.ps1
powershell -ExecutionPolicy Bypass -File install.ps1
```

### 2) 啟動（互動模式）

macOS:

```bash
./start.sh
```

Windows:

```powershell
.\start.ps1
```

---

## 最常用啟動方式

### A. 互動式選單（新手推薦）

```bash
./start.sh
# Windows: .\start.ps1
```

### B. CLI 直接啟動（跳過選單）

```bash
./start.sh --mode en2zh --engine llm --llm-model qwen2.5:14b
```

### C. 啟動 WebUI（瀏覽器操作）

```bash
./start.sh --webui
# Windows: .\start.ps1 --webui
```

### D. WebUI + HDMI 一啟動就開輸出

```bash
./start.sh --webui --hdmi-output --hdmi-source desktop
# Windows: .\start.ps1 --webui --hdmi-output --hdmi-source desktop
```

說明：
- 在 --webui 路徑下，HDMI 會在 WebUI 進入設定頁前就啟動，並在整個程序期間保持開啟。
- 不用先按「開始」才有 HDMI 畫面。

---

## HDMI 分割輸出

### 1) 基本概念

- 開關：--hdmi-output
- 來源：--hdmi-source（可重複，最多 4 路）
- 版型：--hdmi-layout
- 顯示器：--hdmi-display
- 解析度：--hdmi-resolution
- 幀率：--hdmi-fps

### 2) 來源格式

可用來源（每個來源一個 --hdmi-source）：

- desktop
- camera
- camera:裝置名稱
- file:C:/demo.mp4
- rtsp://...
- http://... / https://...
- udp://...
- testsrc

### 3) 常用範例

單一路桌面：

```bash
./start.sh --hdmi-output --hdmi-source desktop
```

四分割（桌面+攝影機+串流+測試圖）：

```bash
./start.sh --hdmi-output \
  --hdmi-layout grid \
  --hdmi-display 1 \
  --hdmi-resolution 1920x1080 \
  --hdmi-fps 30 \
  --hdmi-source desktop \
  --hdmi-source camera \
  --hdmi-source rtsp://192.168.1.10/live \
  --hdmi-source testsrc
```

主畫面在左：

```bash
./start.sh --hdmi-output --hdmi-layout focus_left --hdmi-source desktop --hdmi-source camera
```

---

## WebUI 使用方式

啟動：

```bash
./start.sh --webui
```

預設網址：

- http://localhost:19781

WebUI 可以做的事：

- 設定即時模式、翻譯引擎、模型、場景
- 開始/停止即時轉錄
- 匯入音訊做離線處理
- 啟用講者辨識、摘要
- 儲存前次設定
- 檢視即時字幕、進度、輸出檔

WebUI 進階功能：

- 關鍵字即時通知（瀏覽器通知/音效/懸浮字幕閃爍）
- 字幕轉發（Telegram/Slack/Discord/Teams/LINE/Nextcloud Talk/通用 API）
- 懸浮字幕設定（字體比例、透明度、樣式）
- 遠端密碼保護（唯讀/管理）
- 即時切換音訊裝置與靜音控制

WebUI 啟動參數：

- --port PORT：WebUI HTTP port（預設 19781）
- --no-browser：不自動開瀏覽器
- --hdmi-output：WebUI 程式一啟動就啟用 HDMI
- --hdmi-layout
- --hdmi-display
- --hdmi-resolution
- --hdmi-fps
- --hdmi-source（可重複）

範例：

```bash
./start.sh --webui --port 18080 --no-browser --hdmi-output --hdmi-source desktop
```

---

## 離線處理與摘要

### 離線轉錄

```bash
./start.sh --input meeting.mp3 --mode en2zh
```

### 講者辨識

```bash
./start.sh --input meeting.mp3 --diarize
./start.sh --input meeting.mp3 --diarize --num-speakers 3
```

### 摘要與校正

```bash
./start.sh --input meeting.mp3 --summarize
./start.sh --summarize logs/英翻中_逐字稿_20260101_120000.txt
```

### 即時模式結束後自動後處理

```bash
./start.sh --mode en2zh --post-summary-mode both --post-summary-model gpt-oss:120b
# correct_only: 只校正逐字稿
# both: 摘要 + 校正
```

---

## 命令列參數總表

以下為 translate_meeting.py 主要參數（即 ./start.sh / .\start.ps1 傳入）。

### 基本流程

- --mode MODE
- --asr {whisper, moonshine, faster-whisper}
- -m, --model MODEL
- --moonshine-model MMODEL
- -s, --scene SCENE
- -d, --device ID
- --list-devices
- --local-asr
- --restart-server

### 翻譯與 LLM

- -e, --engine {llm, argos, nllb}
- --llm-model NAME
- --llm-host HOST
- --topic TOPIC

### 即時錄音/錄影

- --record
- --rec-device ID
- --record-video
- --video-device NAME
- --mic
- --mic-device ID
- --denoise

### HDMI

- --hdmi-output
- --hdmi-layout {grid, focus_left, focus_top}
- --hdmi-display ID
- --hdmi-resolution WxH
- --hdmi-fps FPS
- --hdmi-source SRC（可重複）

### 離線/摘要/字幕輸出

- --input FILE [FILE ...]
- --diarize
- --num-speakers N
- --summarize [FILE ...]
- --summary-model MODEL
- --summary-rounds N
- --post-summary-mode {correct_only, both}
- --post-summary-model MODEL
- --post-llm-host HOST
- --no-srt
- --no-vtt

### 其他

- --subtitle-overlay
- --webui（同時推送事件給 WebUI，需另啟 webui.py）

---

## 輸出檔案說明

預設輸出到 logs/<session>/：

- *.txt：逐字稿
- *.html：可視化逐字稿/摘要
- *.srt：字幕檔
- *.vtt：字幕檔
- 摘要_*.txt / 摘要_*.html：摘要與校正結果

錄音/錄影輸出到 recordings/。

---

## 系統需求

### macOS

- macOS（Apple Silicon / Intel）
- Python 3.12+
- Homebrew
- BlackHole 2ch（擷取系統音訊）

### Windows

- Windows 10+
- Python 3.12+
- PowerShell 5.1+

### 共通

- ffmpeg / ffplay（錄影、HDMI 需要）
- LLM 伺服器（做 LLM 翻譯與摘要時需要，例：Ollama）

---

## 常見問題

### Q1: 我只想快速看設定，該用哪種模式？

- 用 --webui，瀏覽器完成所有設定最直覺。

### Q2: HDMI 沒畫面怎麼辦？

先檢查：

- 是否有帶 --hdmi-output
- ffmpeg/ffplay 是否可執行
- --hdmi-display 是否對應正確螢幕
- 來源格式是否正確（例如 rtsp URL）

### Q3: 我要遠端另一台電腦操作設定

- 啟動 WebUI 後，從該機器 IP:port 開啟頁面。
- 可搭配 --no-browser，避免本機自動彈出瀏覽器。

---

## 相關文件

- 完整操作手冊：[SOP.md](SOP.md)
- 版本紀錄：[CHANGELOG.md](CHANGELOG.md)
- 授權：[LICENSE](LICENSE)
