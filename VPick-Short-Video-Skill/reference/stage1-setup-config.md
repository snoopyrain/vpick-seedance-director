# Stage 1 — 開始前設定確認

## 目的
在開始任何腳本拆解或圖片生成之前,先與使用者確認影片的基本參數。

## 操作流程

### Step 1.1 — 接收故事腳本
使用者通常會在第一輪貼上故事腳本或故事概念。先閱讀,**不要立刻回 JSON**。

若使用者沒貼腳本就要求開始,直接告訴他:

```
請貼一段故事腳本給我,任何長度都行,例如:

「下雨的夜晚,Anna 撐著紅色雨傘走在巷弄裡,
突然 Bryan 從陰影中出現,說:『我等妳很久了。』
她退後半步,握緊了傘柄...」

只要有人物、場景、動作或對話就可以,我會幫你拆成 4–7 個鏡頭並完成後製。
```

### Step 1.2 — 提問清單

```
在開始之前,我需要確認幾個基本設定:

1. 影片風格(擇一或自訂):
   - cinematic 電影感
   - anime 動畫風
   - photorealistic 寫實
   - commercial 廣告風
   - documentary 紀錄片風
   - 其他(請描述)

2. 對白/字幕語言:繁體中文 / 英文 / 其他

3. 目標平台與長寬比:
   - IG/TikTok/Reels 直式 9:16
   - YouTube 橫式 16:9
   - 方形 1:1

4. 影片總長:
   - 60 秒(4 個 Part,預設)
   - 30 秒(2 個 Part)
   - 自訂秒數(每 15 秒切一個 Part)

5. 角色設定:
   - 角色數量
   - 每個角色的性別、年齡、外觀關鍵字
   - 是否要提供**外型參照圖 URL**(可選,任一公開圖片 URL)?
     - 有 → 作為臉部 / 體型基準,Stage 3 仍會依劇本換裝
     - 無 → 純文字生成角色總圖
     - 想用內建範例 → 男主角可填 Andy:`https://storage.googleapis.com/aiheadphoto/andy-4-chatgpt.png`(**這只是範例頭像,建議你換成自己的**)
   - 是否要提供**語音參照 URL**(可選,任一公開 mp3 URL,建議 10–30 秒乾淨人聲)?
     - 有 → Stage 5 該角色開口時帶入該 voice
     - 無 → Seedance 自動生成語音
     - 想用內建範例 → 男主角可填 Andy 聲音:`https://storage.googleapis.com/aiheadphoto/andyvoice10min_first15s.mp3`(**這只是範例聲音,建議你換成自己錄的**)

6. 主要場景數量

7. 是否有對白:
   - 有 → Stage 5 影片會自動帶入該角色的 voice reference(若使用者於上一題提供)
   - 無 → Stage 5 影片只生成畫面 + 環境音

8. 重要道具:有沒有貫穿全片的關鍵道具?
```

### Step 1.3 — 輸出 setup_config JSON

```json
{
  "stage": 1,
  "setup_config": {
    "visual_style": "cinematic",
    "language": "zh-TW",
    "aspect_ratio": "9:16",
    "platform": "Instagram Reels",
    "total_duration_seconds": 60,
    "parts_count": 4,
    "part_duration_seconds": 15,
    "characters_count": 2,
    "characters_preview": [
      {
        "id": "character_a",
        "name": "Anna",
        "gender": "female",
        "reference_image_url": null,
        "voice_reference_url": null,
        "_note": "女主角無內建範例,使用者可貼自己的圖片 / mp3 URL"
      },
      {
        "id": "character_b",
        "name": "Bryan",
        "gender": "male",
        "reference_image_url": "https://storage.googleapis.com/aiheadphoto/andy-4-chatgpt.png",
        "voice_reference_url": "https://storage.googleapis.com/aiheadphoto/andyvoice10min_first15s.mp3",
        "_note": "↑ 預填 Andy 範例,使用者可換成自己的頭像 / 聲音 URL"
      }
    ],
    "environments_count": 2,
    "environments_preview": ["雨夜街道", "公寓臥室"],
    "has_dialogue": true,
    "key_props": ["紅色雨傘"]
  },
  "confirmation_required": true,
  "confirmation_message": "以上是基本設定,請確認是否正確?\n\n注意:character_b 我預填了 Andy 的範例頭像與聲音,**這只是讓你看看效果**。要做出像你自己的短劇,建議把 reference_image_url / voice_reference_url 換成你自己的頭像與聲音 URL(任何可公開存取的圖片/mp3 都可以)。\n\n確認後我會進入 Stage 2,將腳本拆解為 N 個 Part 的詳細分鏡。",
  "next_action": "wait_for_user_confirmation"
}
```

### Step 1.4 — 等待確認
使用者明確確認後,讀取 `stage2-script-json-schema.md`,進入 Stage 2。

## 注意事項

- 使用者沒提供故事腳本就要求開始,**先請使用者提供腳本**(用 Step 1.1 的範例提示)
- 影片總長不是 60 秒的倍數時,以 15 秒切一個 Part 為原則(45 秒 = 3 Parts,90 秒 = 6 Parts)
- `gender` 欄位很重要,影響角色描述用語與後續 reference 串接
- `reference_image_url` / `voice_reference_url`:**完全可選**
  - 若使用者沒提供 → 填 `null`,Stage 3/5 自動降級為純文字 / Seedance 自動生成
  - 男主角內建 Andy 範例 URL,可直接使用,但**必須主動告訴使用者「這只是範例,建議換成自己的」**以提升個人化效果
  - 女主角無內建範例,須由使用者提供 URL 或留 `null`
