# Podcast/Audio Autopilot Kit — 含剪輯風格學習

*給 The Long Way Here（Olivia Pan + Nina Tseng）用*

*[English README](README.en.md)*

《The Long Way Here》（Olivia Pan + Nina Tseng 雙主持 podcast，雙週更新）的製作自動化。這個 repo 裡有兩塊東西：

1. **Podcast 發布流程**（`scripts/`）—— 直接照節目自己的
   [Production Playbook](docs/production_playbook.md) 蓋的：原始逐字稿 →
   標題、show notes、附章節的 YouTube 說明、podcast 說明、Buzzsprout 草稿、YouTube
   上傳 checklist。
2. **剪輯風格學習工具**（`src/`）—— 從
   [Hao0321/video-autopilot-kit](https://github.com/Hao0321/video-autopilot-kit)
   透過 Olivia 自己 fork 出來的
   [olieepop/video-autopilot-kit](https://github.com/olieepop/video-autopilot-kit)
   （本機叫 creator-voice-autopilot）搬過來的。那個 fork 真正的貢獻——從你自己的
   原始／發布逐字稿成對比對，學出剪輯時到底剪掉了什麼——比這個 repo 一開始從零手刻
   的版本更好用、做得更完整。框架架構全部歸功 Hao0321，`edit_style_model.py` /
   `rough_cut.py` 歸功 Olivia 的 fork；這裡搬過來的只有這個節目實際用得到的逐字稿／
   字幕相關部分，不是整套多格式 kit（Shorts、劇情片、靜音 vlog 等等——要那些的話去
   看原 fork）。MIT 授權，見 [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md)。

## 流程

```
Riverside 錄音
  -> 原始逐字稿（分主持人、.srt/.vtt/.txt）
  -> scripts/prep_transcript.py          （抓語言辨識錯誤的段落）
  -> src/edit_style_model.py learn       （累積你的剪輯風格檔——選用，用來輔助判斷剪點）
  -> src/rough_cut.py reconstruct        （已剪好的集數：算出精確剪點清單——訓練資料）
  -> scripts/script_01_text_outputs.py   （標題、show notes、YT 說明、podcast 說明、金句）
  -> CapCut 剪輯（手動——雙語燒錄字幕用 src/dual_subtitle.py）
  -> scripts/script_02_buzzsprout_upload.py  （podcast 草稿）
  -> scripts/script_03_youtube_prep.py       （YouTube 上傳 checklist）
```

Script 1–3 完全照 Playbook 的規格做。`prep_transcript.py` 存在的原因是 ep1 的原始
逐字稿暴露了 Playbook 沒預料到的問題——見
[docs/editing_learnings.md](docs/editing_learnings.md)。

## 結構

- `scripts/` —— podcast 發布流程，手動執行 `python scripts/<name>.py`
- `src/` —— 剪輯風格學習＋字幕工具，搬過來的（見上面的歸功說明）：
  `edit_style_model.py`（從成對的原始／發布逐字稿學出剪掉／留存風格檔）、
  `rough_cut.py`（推算／重建／合併／執行剪點清單）、`dual_subtitle.py`（繁中＋英文
  雙語燒錄字幕），還有它們依賴的 `delivery_media_ops.py` 與 `media_delivery_qa.py`
- `templates/edit_style_profile.template.md` —— 怎麼看 `src/edit_style_model.py` 的
  輸出；也是搬過來的
- `docs/production_playbook.md` —— Drive 上 production playbook 的鏡像，放進 repo
  是為了讓 script 的輸入輸出規格跟實作程式碼一起版控
- `docs/editing_learnings.md` —— 原始逐字稿跟發布版之間到底發生了什麼的持續記錄，
  每集剪完都更新。只能加，不要覆蓋掉之前的。
- `episodes/<epN>/` —— 各集的輸入輸出（逐字稿不進 git；產出的內容檔會進 git 留紀錄）
- `profiles/` —— 不進 git。`src/edit_style_model.py` 的輸出是直接從你自己未剪輯的
  原始語音算出來的——跟上游 kit 同一條規矩：個人資料不進 repo，只有從你自己本機檔案
  產生它的工具進 repo。

## 設定

```bash
pip install -r requirements.txt
cp .env.example .env   # 填 ANTHROPIC_API_KEY、BUZZSPROUT_API_KEY、BUZZSPROUT_PODCAST_ID
```

`src/` 需要 PATH 裡有 `ffmpeg`/`ffprobe`（`dual_subtitle.py burn` 跟 `rough_cut.py
propose`/`apply` 要用）；`edit_style_model.py learn` 跟 `rough_cut.py reconstruct`
是純 Python，不需要 `ffmpeg`。

## 現況

- `scripts/script_01_text_outputs.py` —— 已實作，手動對 ep1 逐字稿跑過一次，見
  `episodes/ep1/episode_ep1_content.md`。要真的跑需要有效的 `ANTHROPIC_API_KEY`。
- `scripts/script_02_buzzsprout_upload.py`、`scripts/script_03_youtube_prep.py` ——
  照 Playbook 規格搭好架子，`script_03` 已用真實輸出驗證過，`script_02` 還沒對真的
  Buzzsprout 帳號測試過。
- `src/edit_style_model.py`、`src/rough_cut.py` —— 搬過來後已對 ep1 真實逐字稿驗證
  （72.2 分鐘原始 → 191 段剪點、留下 36.1 分鐘——見 `docs/editing_learnings.md`）。
  `src/dual_subtitle.py` 已搬過來但還沒對真正剪好的 ep1 跑過。
