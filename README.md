# 📋 HKO 工時表 - 員工工時與請假管理系統

一個基於 Firebase 的員工工時記錄與請假管理系統，支援半日/全日請假、公眾假期自動標記、特別安排備註等功能。

## ✨ 主要功能

- **工時記錄**: 記錄每日四個時間點 (時間1-4)
- **請假管理**: 
  - 全天請假 (勾選「全天請假」自動設定上午/下午請假)
- **自動計算**: 
  - 預計工時 (週一至週四 07:50，週五 07:40)
  - 預計下班時間 (根據時間1 + 預計工時 + 午休時間)
- **假日分類**:
  - 🟣 週末自動標記
  - 🔴 公眾假期 (從 Firebase 讀取)
  - 🟡 年假 (全天請假)
  - 🔴 特別安排 (含備註，不可編輯)
- **資料儲存**: 
  - 需先點擊「同意並啟用儲存」獲得授權
  - 支援未儲存變更暫存
  - 即時同步 Firebase 資料

## 🚀 線上使用

直接開啟 `index.html` 即可使用，首次使用需輸入 Firebase 設定。

## 🗄️ Firebase 資料結構

### Realtime Database 路徑

```
/
├── holiday/
│   └── {year}/
│       └── [日期陣列]          # 公眾假期列表
│
└── timesheet/
    └── {userId}/
        └── {year}/
            └── {month}/
                └── {day}/
                    ├── time1: "09:00"    # 上班時間
                    ├── time2: "12:30"    # 午休開始
                    ├── time3: "13:30"    # 午休結束
                    ├── time4: "18:00"    # 下班時間
                    ├── isAmOff: true     # 上午請假
                    ├── isPmOff: false    # 下午請假
                    └── remark: "特別安排" # 備註 (選填)
```

### 資料範例

```json
{
  "timesheet": {
    "userId123": {
      "2025": {
        "07": {
          "11": {
            "time1": "09:40",
            "time2": "12:40",
            "time3": "14:07",
            "time4": "19:01",
            "isAmOff": false,
            "isPmOff": false
          },
          "30": {
            "time1": null,
            "time2": null,
            "time3": null,
            "time4": null,
            "isAmOff": true,
            "isPmOff": true
          }
        },
        "08": {
          "05": {
            "time1": null,
            "time2": null,
            "time3": null,
            "time4": null,
            "isAmOff": true,
            "isPmOff": true,
            "remark": "Special Arrangement"
          }
        }
      }
    }
  },
  "holiday": {
    "2025": [
      "2025-01-01",
      "2025-01-29",
      "2025-04-04",
      "2025-05-01",
      "2025-07-01",
      "2025-10-01",
      "2025-12-25"
    ]
  }
}
```

## 🔐 Firebase 安全規則

```json
{
  "rules": {
    "holiday": {
      ".read": true,
      ".write": true
    },
    "timesheet": {
      ".read": true,
      ".write": true
    }
  }
}
```

**說明**: 
- 使用匿名驗證，每位使用者有獨立的 UID
- 能匿名讀寫 `timesheet/{UID}/*` 路徑
- `holiday` 為公開讀寫 (僅管理員可修改)

## 🛠️ 技術棧

- **前端**: HTML5, CSS3, JavaScript (ES6+)
- **UI 框架**: SweetAlert2
- **後端服務**: Firebase
  - Realtime Database (資料儲存)
  - Authentication (匿名登入)
- **部署**: 靜態 HTML 檔案，可直接在瀏覽器執行

## 📱 使用說明

### 1. 首次使用設定

1. 開啟 `index.html`
2. 輸入 Firebase 設定：
   - API Key (必填)
   - Database URL (必填)
   - 其他欄位可使用預設值
3. 點擊「連線」

### 2. 日常使用流程

1. **登入**: 系統自動匿名登入
2. **同意授權**: 點擊「同意並啟用儲存」按鈕
3. **填寫工時**: 
   - 可編輯日期 (綠色/橘色/黃色背景)
   - 輸入時間1-4 (24小時制)
4. **請假設定**: 
   - 勾選「全天請假」自動設為整天請假
   - 預計工時/下班時間自動計算
5. **儲存**: 修改後點擊右下角「儲存」按鈕

### 3. 資料狀態標示

| 背景顏色 | 說明 | 能否編輯 |
|---------|------|---------|
| 🟢 淺綠 | 已有資料 | ✅ 可編輯 |
| 🟠 淺橘 | 缺失資料 | ✅ 可編輯 |
| 🔵 淡藍 | 週末 | ❌ 不可編輯 |
| 🔴 淡紅 | 公眾假期 | ❌ 不可編輯 |
| 🟡 淡黃 | 年假/全天請假 | ✅ 可編輯 |
| 💗 淡粉 | 特別安排 (含備註) | ❌ 不可編輯 |
| ⚪ 灰色 | 歷史日期 | ❌ 不可編輯 |
| ⚪ 白色 | 未來日期 | ❌ 不可編輯 |

### 4. 預計下班時間計算公式

```
預計下班時間 = 時間1 + 預計工時 + (時間3 - 時間2)

其中:
- 預計工時: 
  - 週一至週四: 07:50
  - 週五: 07:40
  - 週末/請假: 00:00
```

## 📊 功能限制

- **可編輯日期**: 2025-07-11 至 今天 (UTC+8)
- **不可編輯**: 
  - 週末、公眾假期
  - 含「特別安排」備註的日期
  - 歷史/未來日期
- **可編輯但受限制**: 
  - 年假/全天請假日期 (僅能修改請假狀態，時間欄位自動清空)
- **需要同意**: 每次頁面載入需重新同意儲存授權

## 🔧 開發者資訊

### Firebase 專案設定

1. 建立 Firebase 專案
2. 啟用 Anonymous Authentication
3. 建立 Realtime Database
4. 複製 Firebase 設定到應用程式

### 本地開發

```bash
# 直接開啟 index.html 即可
open index.html

# 或使用簡單的 HTTP 伺服器
python -m http.server 8000
# 訪問 http://localhost:8000
```

### 環境變數

儲存在 `sessionStorage` 中，無需額外設定檔。

## 🎯 未來擴充建議

- [ ] 匯出報表功能 (CSV/Excel)
- [ ] 統計圖表 (每月工時趨勢)
- [ ] 多使用者管理介面
- [ ] 審核流程 (主管簽核)
- [ ] 郵件通知 (請假申請)
- [ ] 行動裝置優化 (PWA)

## 📝 版本記錄

- **v1.0.0** (2025-07-11)
  - 初始版本
  - 支援工時記錄與全天請假
  - Firebase 整合
  - 公眾假期自動標記

## 📄 授權

內部使用工具 - 版權所有 © HKO

## 🔗 相關連結

- [Firebase Console](https://console.firebase.google.com/u/0/project/hko-timesheet)
  - [Realtime Database](https://console.firebase.google.com/u/0/project/hko-timesheet/database/hko-timesheet-default-rtdb/data)
  - [Security Rules](https://console.firebase.google.com/u/0/project/hko-timesheet/database/hko-timesheet-default-rtdb/rules)
  - [Authentication](https://console.firebase.google.com/u/0/project/hko-timesheet/authentication/users)
- [API Key Settings](https://console.firebase.google.com/u/0/project/hko-timesheet/settings/general)

---

## ❓ 常見問題

**Q: 為什麼無法儲存資料？**  
A: 請先點擊「同意並啟用儲存」按鈕獲得授權。

**Q: 為什麼某些日期無法編輯？**  
A: 週末、公眾假期、特別安排、歷史日期 (早於2025-07-11) 或未來日期皆不可編輯。

**Q: 如何修改已儲存的資料？**  
A: 直接在表格中修改時間或請假狀態，會出現「未儲存的修改」提示，點擊儲存即可。

**Q: 年假/全天請假日期可以編輯嗎？**  
A: 可以。您可以在年假日期取消勾選「全天請假」，恢復為正常工作日以填寫工時。

**Q: 資料會遺失嗎？**  
A: 所有儲存資料都在 Firebase 雲端，除非手動刪除，否則不會遺失。
```

**主要更新**:
- 將「年假/全天請假」的編輯狀態改為 ✅ 可編輯
- 在功能限制中新增「可編輯但受限制」說明
- 新增常見問題關於年假編輯的說明