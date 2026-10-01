# 私立中山紀念堂 — 後端 (Backend)

本專案為《私立中山紀念堂》的 Serverless 後端服務，基於 [Cloudflare Workers](https://workers.cloudflare.com/) 與 [Cloudflare Workers KV](https://developers.cloudflare.com/kv/) 構建，承載即時彈幕互動、違規詞過濾、白名單排程、AI 審核矩陣及管理後台配置同步。

---

## 路由與端點 (Endpoints)

| 方法 | 路徑 | 說明 | 鑑權要求 |
|---|---|---|---|
| `GET` | `/danmaku` | 取得最新彈幕留言陣列（支援 `?action=get_banned_words`） | 公開 |
| `POST` | `/danmaku` | 提交新彈幕留言（自動觸發單 IP 60秒限頻、垃圾過濾與 AI 審核） | 指紋限制 |
| `DELETE` | `/danmaku` | 撤回指定彈幕（支援發送者指紋自撤或管理員金鑰強制撤回） | 指紋或管理金鑰 |
| `GET` | `/admin/config` | 讀取後台儀表板完整配置數據 | 管理員 SHA-256 Key |
| `POST` | `/admin/config` | 更新敏感詞庫、白名單模式或 AI 矩陣設定 | 管理員 SHA-256 Key |
| `POST` | `/admin/learn` | AI 監督學習與誤判樣本覆核修正 | 管理員 SHA-256 Key |

---

## 部署指引

### 方法一：使用 Wrangler CLI 部署（推薦）

1. **安裝依賴**：
   ```bash
   npm install -g wrangler
   wrangler login
   ```
2. **建立 KV 命名空間**：
   ```bash
   wrangler kv:namespace create "MEMORIAL_KV"
   ```
   將輸出得到的 `id` 填寫至 `wrangler.toml` 中的 `[[kv_namespaces]]` 區塊。
3. **設定管理員金鑰（可選，預設為 SunYatSen1911）**：
   ```bash
   wrangler secret put ADMIN_SECRET
   ```
4. **發布部署**：
   ```bash
   wrangler deploy
   ```

### 方法二：Cloudflare Dashboard 網頁版手動部署

1. 登入 [Cloudflare 控制台](https://dash.cloudflare.com/)，進入 **Compute (Workers) > Workers & Pages**。
2. 建立新 Worker，命名為 `sys-memorial-backend`。
3. 進入 **Settings > Variables & Secrets**：
   - 於 **KV Namespace Bindings** 綁定變數名稱 `MEMORIAL_KV`。
   - 於 **Environment Variables** 新增 `ADMIN_SECRET`（型態為 Secret）。
4. 點擊 **Quick Edit**，將 `workers.js` 的程式碼貼上並點擊 **Save and Deploy**。

---

## 授權條款 (License)

本專案遵循 [GNU Affero General Public License v3.0 (AGPL-3.0)](LICENSE) 開源授權協議。
Copyright (C) 2026 Situla.
