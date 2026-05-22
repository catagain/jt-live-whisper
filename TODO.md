# 開發待辦與實作規劃

這份文件先整理目前確認過的待辦，目的是把後續實作拆成可以逐項完成的工作。此階段只做規劃，不直接修改功能。

## 建議實作順序

1. 字幕產生邏輯檢查
2. Whisper 模型預載入
3. 及時字幕逐字顯示
4. 字幕位置 / 字型 / 字形設定
5. HDMI 子母畫面自由定位
6. YouTube 直播串流

排序理由：先處理會直接影響字幕正確性的核心流程，再做啟動體驗與顯示層功能，最後再擴充外部串流能力。

---

## 1. 字幕產生邏輯檢查

### 目標

- 解決句子重複顯示。
- 降低 prompt 洩漏、訓練語料外漏、模型自我解說等內容直接出現在字幕中的情況。
- 釐清問題是出在 ASR 切句、翻譯後處理、還是字幕顯示併段。

### 目前觀察

- [translate_meeting.py](c:/working_space/jt-live-whisper/translate_meeting.py#L2184) 已有 hallucination keyword 與 prompt 洩漏清理。
- [translate_meeting.py](c:/working_space/jt-live-whisper/translate_meeting.py#L2252) 已有多段 LLM 輸出清洗，但仍偏向字串後處理。
- [translate_meeting.py](c:/working_space/jt-live-whisper/translate_meeting.py#L5854) 與 [translate_meeting.py](c:/working_space/jt-live-whisper/translate_meeting.py#L6595) 有結果排序與顯示佇列邏輯，重複句問題可能也和 display queue 或 partial/final 邊界有關。

### 建議拆分

1. 建立重現案例：收集 3 到 5 組會重複字幕或外漏 prompt 的錄音 / log 範例。
2. 將問題分類：區分為 ASR 重複、翻譯重試重複、字幕顯示重複、摘要或校正內容誤入即時字幕。
3. 加診斷輸出：為每筆字幕標出來源階段、seq id、partial/final 狀態、清洗前後內容。
4. 定義過濾規則：先補 deterministic 規則，再決定是否需要更嚴格的 prompt 或重複抑制策略。
5. 建立回歸測試：至少保留一組重複句、一組 prompt 洩漏、一組正常句。

### 驗收標準

- 同一個 final 句子不會連續重複輸出。
- 明顯 instruction、think、翻譯說明、訓練資料片段不會直接出現在使用者字幕中。
- 修正後不明顯增加延遲或吃字。

---

## 2. 及時字幕加上逐字及時出現版本

### 目標

- 除了現有整句更新方式，再提供一種逐字或逐片段即時冒出的字幕模式。
- 使用者可以在穩定整句模式與高速逐字模式之間切換。

### 目前觀察

- 現有流程主要仍以句段為單位更新字幕，顯示面較偏向最終可讀性。
- 逐字模式會直接碰到 partial 結果抖動、改寫、回退等問題，必須先定義顯示策略。

### 建議拆分

1. 定義模式：`穩定整句`、`逐字即時`、`混合模式（partial 顯示 + final 覆蓋）`。
2. 決定資料來源：優先使用 whisper partial token / partial segment，而不是從 final 句硬切字。
3. 設計顯示規則：partial 用較淡樣式，final 到來後覆蓋並定稿。
4. 加設定入口：CLI、WebUI、懸浮字幕都要能選模式。
5. 做延遲與可讀性測試：確認不會因為頻繁刷新造成畫面閃爍。

### 驗收標準

- 使用者可切換至少兩種字幕更新模式。
- 逐字模式能持續更新且 final 到來後能正確收斂，不殘留舊字。
- CPU 與 UI 更新頻率維持可接受範圍。

---

## 3. 字幕位置、字型大小、字形設定功能

### 目標

- 讓字幕位置、字體大小、字體家族、粗細或樣式可由使用者設定。
- 設定需能保存，下次啟動可復用。

### 目前觀察

- [subtitle_overlay.py](c:/working_space/jt-live-whisper/subtitle_overlay.py#L97) 已有字體 preset、font family、opacity 與視窗位置恢復。
- [subtitle_overlay.py](c:/working_space/jt-live-whisper/subtitle_overlay.py#L223) 已有系統匣中的字體大小選單。
- 目前能力集中在 overlay 視窗，若要擴到 WebUI 內字幕或 HDMI 合成字幕，需要先統一設定模型。

### 建議拆分

1. 盤點字幕顯示面：terminal、WebUI、overlay、HDMI 合成字幕是否共用設定。
2. 統一設定欄位：位置、對齊、字型、字重、字級、行距、背景透明度。
3. WebUI 提供可視化設定頁，overlay 繼續保留快速調整入口。
4. 定義套用範圍：只套 overlay，或同步套到所有字幕輸出。
5. 增加設定儲存與匯出格式說明。

### 驗收標準

- 可從 UI 調整字幕位置與字型設定。
- 設定重啟後仍保留。
- 不同顯示面對同一份設定的行為一致，或文件中明確說明差異。

---

## 4. 開始錄影前先預載入 Whisper 模型

### 目標

- 將模型載入成本提前，避免按下開始錄影後還要等待 Whisper 啟動。
- 使用者看到的正式開始時間點要更可預期。

### 目前觀察

- repo memory 已記錄預熱策略，且 [translate_meeting.py](c:/working_space/jt-live-whisper/translate_meeting.py#L4513) 已有 whisper ready gate。
- [translate_meeting.py](c:/working_space/jt-live-whisper/translate_meeting.py#L4535) 與 [translate_meeting.py](c:/working_space/jt-live-whisper/translate_meeting.py#L4572) 已將 `prepare()` / `begin_recording()` 分開處理，代表這題已有基礎，但還沒有完整做成「使用者感知上的預載入」。

### 建議拆分

1. 明確定義預載入時機：程式啟動時、進 WebUI 時、切換模型時、按開始前。
2. UI 顯示預載狀態：未載入、載入中、已就緒。
3. 區分模型切換行為：切換模型後是否重新預熱，避免誤用舊狀態。
4. 加入 timeout 與失敗回退：預載入失敗時仍可手動開始。
5. 補啟動路徑測試：CLI、WebUI、錄音、錄影、HDMI 同開。

### 驗收標準

- 使用者按下開始後，不再因模型冷啟動而卡住數秒。
- WebUI 或 CLI 能清楚顯示模型是否已就緒。
- 切模型後的狀態一致且可理解。

---

## 5. 加上直播功能，透過 YouTube 金鑰輸出 HDMI 畫面到 YT

### 目標

- 在現有 HDMI 合成輸出基礎上，增加推流到 YouTube Live 的能力。
- 讓使用者輸入 stream key 後，直接將合成後的節目畫面送出。

### 主要範圍

- 串流目標先以 YouTube RTMP 為主。
- 優先支援將 HDMI 合成總畫面推流，不先處理多路獨立推流。

### 建議拆分

1. 決定推流架構：沿用 ffmpeg pipeline，直接從既有 HDMI 合成輸出分流。
2. 設計設定方式：stream key 儲存策略、遮罩顯示、是否允許寫入 config。
3. UI 控制：開始直播、停止直播、推流狀態、錯誤訊息。
4. 網路容錯：斷線重試、碼率預設、解析度 / FPS 限制。
5. 安全處理：避免把 stream key 寫進 log、前端回傳、或版本控制。

### 驗收標準

- 可從 CLI 或 WebUI 啟停直播。
- YouTube 能穩定收到 HDMI 合成畫面與音訊。
- stream key 不會出現在畫面、log 或匯出的設定檔中。

---

## 6. HDMI 子母畫面改成可在預覽畫面自由移動小視窗

### 目標

- 不再限制 PiP 小窗只能固定在左上、左下、右上、右下。
- 使用者可直接在預覽區拖曳小窗到任意位置並保存。

### 目前觀察

- [webui.html](c:/working_space/jt-live-whisper/webui.html#L684) 到 [webui.html](c:/working_space/jt-live-whisper/webui.html#L687) 目前 PiP 仍是四個固定角落版型。
- [webui.html](c:/working_space/jt-live-whisper/webui.html#L755) 到 [webui.html](c:/working_space/jt-live-whisper/webui.html#L782) 現在的拖曳是交換來源槽位，不是自由座標定位。

### 建議拆分

1. 新增自訂 PiP 模式：例如 `pip_custom`。
2. 定義資料格式：儲存小窗的 `x/y/width/height` 或相對比例座標。
3. 預覽區改成真正拖曳定位，必要時加上吸附邊界與安全區。
4. 套用到實際 HDMI 合成器，而不只停留在預覽畫面。
5. 補上重設位置與多解析度縮放對應。

### 驗收標準

- 使用者可在預覽畫面自由拖曳 PiP 小窗。
- 儲存後重新開啟仍維持位置。
- 預覽位置與實際 HDMI 輸出位置一致。

---

## 每一項實作前建議先補的共通工作

- 建立最小重現案例與對應測試資料。
- 明確區分 CLI、WebUI、overlay、HDMI 輸出的責任邊界。
- 先定義設定欄位與儲存格式，避免後面多次搬移 config schema。
- 每項功能實作後都補一段 README 或 SOP 說明，避免功能已存在但使用方式不明。