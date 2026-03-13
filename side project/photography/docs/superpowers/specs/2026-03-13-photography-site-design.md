# 個人攝影作品分享網站 — 完整規格文件

> 版本：1.1 | 日期：2026-03-13

---

## 專案背景

這是一個個人攝影作品展示網站，供攝影師（Admin）管理並公開分享照片給訪客（Viewer）。
Admin 可上傳、管理照片、相簿與標籤；Viewer 無需登入即可瀏覽、篩選與分享內容。
網站以低成本部署為優先，預計照片數量在 500 張以內。

---

## 角色與權限

| 操作 | Admin | Viewer |
|------|-------|--------|
| 登入 / 登出 | ✅ | ❌ |
| 上傳 / 編輯 / 刪除照片 | ✅ | ❌ |
| 隱藏照片 / 相簿 | ✅ | ❌ |
| 管理 Tag / Album | ✅ | ❌ |
| 瀏覽公開照片 | ✅（預覽模式）| ✅ |
| 依 Tag / 相簿篩選 | ✅ | ✅ |
| 瀏覽隱藏照片 / 相簿 | ✅（管理介面）/ 需開啟 filter（預覽模式）| ❌ |
| 分享連結 | ✅ | ✅ |

---

## 功能清單

### Admin — 照片管理

| 功能 | 目的 | 約束 / 備註 |
|------|------|------------|
| 上傳圖片 | 新增作品到系統 | 支援批次上傳；接受格式：JPEG、PNG、WebP；上傳時可同時指定 tag / album |
| 編輯圖片 | 修改圖片的 metadata | 可新增／刪除 tag、album、修改標題 / 描述 |
| 刪除圖片 | 移除不需要的照片 | 同步刪除儲存空間中的原圖與縮圖 |
| 隱藏圖片 | 暫時不公開但保留 | Viewer 不可見；預覽模式預設不顯示，可透過 filter 切換 |
| 查看圖片列表 | 管理所有照片 | filter: tag、date；預設依上傳日期降冪排序；支援 pagination |
| 查看單張圖片 | 確認圖片細節 | 依 config 設定顯示原圖或高品質壓縮版 |

### Admin — Tag 管理

| 功能 | 目的 | 約束 / 備註 |
|------|------|------------|
| 新增 Tag | 建立分類標籤 | |
| 編輯 Tag | 修改標籤名稱 | slug 不隨 name 改變（避免分享連結失效） |
| 刪除 Tag | 移除不需要的標籤 | 刪除 Tag 不刪除照片 |
| 查看 Tag 列表 | 管理所有標籤 | |
| 將圖片加入 Tag | 為照片貼標籤 | 一張圖片可有多個 tag |
| 將圖片從 Tag 移除 | 取消標籤 | |

### Admin — 相簿管理

| 功能 | 目的 | 約束 / 備註 |
|------|------|------------|
| 新增相簿 | 建立作品集 | |
| 編輯相簿 | 修改相簿名稱 / 描述 | slug 不隨 name 改變（避免分享連結失效） |
| 刪除相簿 | 移除不需要的相簿 | 刪除相簿不刪除照片 |
| 隱藏相簿 | 暫時不公開 | Viewer 不可見；預覽模式預設不顯示，可透過 filter 切換 |
| 查看相簿列表 | 管理所有相簿 | |
| 設定相簿封面圖 | 相簿列表的視覺呈現 | 從相簿內的照片中選擇；未設定時自動顯示最近加入的照片 |
| 將圖片加入相簿 | 整理作品集 | 一張圖片可屬於多個相簿 |
| 將圖片從相簿移除 | 調整相簿內容 | |

### Admin — 預覽模式

| 功能 | 目的 | 約束 / 備註 |
|------|------|------------|
| 首頁預覽 | 模擬 Viewer 視角 | 預設隱藏「已隱藏」的照片；頂部有持久性 toggle 可切換顯示隱藏照片，狀態存於 session |
| 單張圖片預覽 | 確認公開呈現效果 | |
| 相簿列表預覽 | 模擬 Viewer 視角 | 預設隱藏「已隱藏」的相簿；頂部有持久性 toggle 可切換；filter: album name；支援 pagination |
| 單一相簿預覽 | 確認相簿公開呈現效果 | filter: tag |

### Viewer — 瀏覽

| 功能 | 目的 | 約束 / 備註 |
|------|------|------------|
| 首頁瀏覽照片 | 展示作品 | Pinterest 式瀑布流；支援 tag filter |
| 隨機圖片 | 增加探索感 | 頁面 hydration 後，前端 client-side 隨機洗牌（見渲染策略說明） |
| 查看單張圖片 | 欣賞高畫質作品 | 依 config 設定顯示原圖或高品質壓縮版（見圖片品質 config） |
| 瀏覽相簿列表 | 依主題探索 | filter: album name；支援 pagination |
| 瀏覽單一相簿 | 看完整系列 | filter: tag；相簿內照片依加入時間降冪排序 |
| 分享連結 | 讓他人直接開啟特定頁面 | 支援：單張圖片、相簿、tag 篩選結果 |

---

## 資料模型

### Photo

| 欄位 | 類型 | 說明 |
|------|------|------|
| id | UUID | 主鍵 |
| title | String (optional) | 圖片標題 |
| description | String (optional) | 圖片描述 |
| r2_original_key | String | R2 原圖路徑（非公開 URL） |
| thumbnail_url | String | Cloudflare CDN 公開縮圖 URL |
| upload_date | Timestamp | 上傳時間 |
| taken_date | Date (optional) | 拍攝日期 |
| is_hidden | Boolean | 是否隱藏 |

> `r2_original_key` 儲存 R2 object key（如 `originals/uuid.jpg`），不對外直接暴露。前端存取原圖時需向 Go backend 請求 Presigned URL。
> `thumbnail_url` 是 Cloudflare CDN 公開 URL，可直接使用於 `<img src>`。

### Tag

| 欄位 | 類型 | 說明 |
|------|------|------|
| id | UUID | 主鍵 |
| name | String | 標籤名稱（唯一） |
| slug | String | URL 用識別碼（唯一；建立時由 name 自動產生；之後不隨 name 更新） |

### Album

| 欄位 | 類型 | 說明 |
|------|------|------|
| id | UUID | 主鍵 |
| name | String | 相簿名稱 |
| slug | String | URL 用識別碼（唯一；建立時由 name 自動產生；之後不隨 name 更新） |
| description | String (optional) | 相簿描述 |
| cover_photo_id | UUID (optional, FK → photos.id ON DELETE SET NULL) | 封面圖片；NULL 時 fallback 為最近加入的照片（依 photo_albums.added_at） |
| is_hidden | Boolean | 是否隱藏 |
| created_at | Timestamp | 建立時間 |

### photo_tags（Junction Table）

| 欄位 | 類型 | 說明 |
|------|------|------|
| photo_id | UUID (FK → photos.id ON DELETE CASCADE) | |
| tag_id | UUID (FK → tags.id ON DELETE CASCADE) | |
| PRIMARY KEY | (photo_id, tag_id) | |

### photo_albums（Junction Table）

| 欄位 | 類型 | 說明 |
|------|------|------|
| photo_id | UUID (FK → photos.id ON DELETE CASCADE) | |
| album_id | UUID (FK → albums.id ON DELETE CASCADE) | |
| added_at | Timestamp | 加入相簿的時間（用於封面 fallback 排序與相簿內排序） |
| PRIMARY KEY | (photo_id, album_id) | |

### Admin

| 欄位 | 類型 | 說明 |
|------|------|------|
| id | UUID | 主鍵 |
| username | String | 帳號名稱（唯一） |
| password_hash | String | bcrypt hash |
| created_at | Timestamp | 建立時間 |

---

## URL 路由結構

| 頁面 | URL |
|------|-----|
| 首頁 | `/` |
| tag 篩選 | `/?tag=[slug]` |
| 單張圖片 | `/photos/[id]` |
| 相簿列表 | `/albums` |
| 單一相簿 | `/albums/[slug]` |
| 單一相簿 + tag 篩選 | `/albums/[slug]?tag=[slug]` |
| Admin 管理介面 | `/admin` |

---

## 圖片品質 Config

單張圖片頁面的顯示品質透過 config 設定，支援兩種模式：

| 模式 | 說明 |
|------|------|
| `original` | 提供原始檔案（byte-for-byte），畫質最佳，流量較高 |
| `high_quality` | 統一轉換為 JPEG quality 90（含原為 PNG / WebP 的圖片），視覺接近原圖，流量大幅降低 |

> 攝影作品不含透明通道，PNG → JPEG 轉換不影響視覺呈現。

---

## 非功能需求

### 圖片顯示策略

| 情境 | 顯示方式 | URL 類型 |
|------|---------|---------|
| 列表 / 首頁 / 相簿縮圖 | 壓縮縮圖 | Cloudflare CDN 公開 URL（thumbnail_url） |
| 單張圖片頁面 | 依 config 設定 | Go backend 產生 Presigned URL（有效期 15 分鐘） |

### 版面配置

- 首頁與相簿採 Pinterest 瀑布流：固定欄寬，高度依圖片比例等比縮放
- 支援橫幅圖片依比例自然呈現
- 響應式佈局：手機 1 欄（< 640px）、平板 2 欄、桌面 3–4 欄

### 效能 / 儲存

- 上傳時自動產生壓縮縮圖，原圖另外儲存
- 預計照片 < 500 張，儲存規模小
- 圖片資源透過 CDN 提供，降低延遲與頻寬成本

### 安全性

- 僅一位 Admin，無需多帳號管理
- 需防止未授權存取管理功能
- 原圖透過 Presigned URL 存取，防止大量惡意請求；縮圖可公開

---

## 已決定的約束

- **部署成本**：優先選擇低成本 / 免費方案（Vercel + Railway + Supabase + Cloudflare R2）
- **照片規模**：< 500 張
- **使用者角色**：單一 Admin + 不限人數 Viewer（無需登入）
- **Admin 帳號**：僅一個，存於 DB（admins 表），密碼以 bcrypt hash 儲存
- **上傳格式**：JPEG、PNG、WebP
- **EXIF 資訊**：不顯示
- **Slug**：建立時由 name 自動產生；之後不隨 name 更新（避免分享連結失效）
- **Pagination**：offset / limit（< 500 張，無需 cursor-based）

---

## 系統架構

### 整體架構圖

```
Browser
  └── Next.js (Vercel)
        ├── App Router — SSR / ISR（公開頁面 + SEO + OG meta）
        ├── API Routes — 薄層轉發（proxy 到 Go backend）+ NextAuth session
        └── NextAuth.js — Cookie / session 管理

Go Backend (Railway)
  ├── REST API (Gin / Echo)
  ├── JWT 驗證 middleware
  ├── 業務邏輯（CRUD、隱藏邏輯、pagination）
  ├── 圖片壓縮（縮圖生成）
  ├── Presigned URL 生成（Cloudflare R2）
  └── ISR Revalidation Webhook（呼叫 Next.js）

Supabase PostgreSQL
  └── photos, tags, albums, photo_tags, photo_albums, admins

Cloudflare R2 + CDN
  ├── originals/{uuid}.{ext}    ← 原圖（非公開）
  └── thumbnails/{uuid}.webp   ← 縮圖（CDN 公開）
```

### 各層職責

| 層 | 技術 | 職責 |
|----|------|------|
| Frontend + BFF | Next.js (Vercel) | 頁面渲染、OG meta、session cookie、轉發 API 請求 |
| Backend | Go (Railway) | 業務邏輯、認證、圖片處理、Presigned URL、觸發 ISR revalidation |
| Database | Supabase PostgreSQL | 資料持久化 |
| 圖片儲存 | Cloudflare R2 | 原圖與縮圖儲存 |
| CDN | Cloudflare | 縮圖公開分發 |

### 圖片上傳流程

```
1. 前端向 Go backend 請求 Presigned Upload URL（R2 originals/）
2. 前端直傳圖片到 R2（繞過 Vercel 4.5MB 限制）
3. 上傳完成後，前端通知 Go backend（附 R2 object key）
4. Go backend 從 R2 讀取原圖，產生縮圖（WebP, 800px, quality 80），存至 R2 thumbnails/
5. Go backend 將 r2_original_key、thumbnail_url 寫入 PostgreSQL
6. Go backend 呼叫 Next.js revalidation webhook，觸發首頁 ISR 更新
```

---

## API Contract（Go Backend）

### 端點列表

| Method | Path | 說明 | Auth |
|--------|------|------|------|
| POST | `/auth/login` | Admin 登入，回傳 JWT | — |
| GET | `/photos` | 取得照片列表（含 pagination、filter） | — / Admin |
| POST | `/photos/upload-url` | 取得 R2 Presigned Upload URL | Admin |
| POST | `/photos` | 上傳完成後建立 photo 記錄 + 產生縮圖 | Admin |
| GET | `/photos/:id` | 取得單張照片 metadata | — / Admin |
| GET | `/photos/:id/view-url` | 取得原圖 Presigned View URL | —（公開，見說明） |
| PATCH | `/photos/:id` | 編輯 metadata（title, description, is_hidden） | Admin |
| DELETE | `/photos/:id` | 刪除照片（含 R2 清除） | Admin |
| GET | `/tags` | 取得 Tag 列表 | — |
| POST | `/tags` | 新增 Tag | Admin |
| PATCH | `/tags/:id` | 編輯 Tag name | Admin |
| DELETE | `/tags/:id` | 刪除 Tag | Admin |
| POST | `/photos/:id/tags/:tag_id` | 將圖片加入 Tag | Admin |
| DELETE | `/photos/:id/tags/:tag_id` | 將圖片從 Tag 移除 | Admin |
| GET | `/albums` | 取得相簿列表（含 pagination、filter） | — / Admin |
| POST | `/albums` | 新增相簿 | Admin |
| GET | `/albums/:slug` | 取得單一相簿（含照片列表）；以 slug 查詢方便 URL 對應 | — / Admin |
| PATCH | `/albums/:id` | 編輯相簿（name, description, cover_photo_id, is_hidden）；以 id 確保精確對應 | Admin |
| DELETE | `/albums/:id` | 刪除相簿 | Admin |
| POST | `/albums/:id/photos/:photo_id` | 將圖片加入相簿 | Admin |
| DELETE | `/albums/:id/photos/:photo_id` | 將圖片從相簿移除 | Admin |

> **Album path parameter 說明**：`GET /albums/:slug` 使用 slug（對應 URL 路由）；`PATCH` / `DELETE` / 照片管理操作使用 id（精確、不受 slug 影響）。前端需同時保存 `id`（供 mutations）與 `slug`（供導航）。

### Pagination 規格

Request query params：`?page=1&limit=20`（預設 limit 20）
Response 格式：

```json
{
  "data": [...],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 150,
    "total_pages": 8
  }
}
```

### Auth Header

Admin 請求需附 `Authorization: Bearer <JWT>`。JWT 有效期 24 小時，無 refresh token（重新登入即可）。

### Hidden Items 過濾規則

`GET /photos`、`GET /albums`、`GET /albums/:slug` 的 hidden 過濾邏輯：
- **無 JWT**（Viewer）：只回傳 `is_hidden = false` 的資料
- **有效 JWT**（Admin）：回傳所有資料（含 hidden）

### `GET /photos/:id/view-url` 公開存取說明

此端點不需要 JWT，因為單張圖片頁面對所有 Viewer 公開。安全性由以下機制保障：
- Presigned URL 有效期僅 **15 分鐘**，無法長期流通
- Go backend 對此端點加上 **rate limiting**（如每 IP 每分鐘 30 次），防止大量批次下載
- Presigned URL 只對單一 R2 object 有效，無法列舉其他檔案

### 縮圖生成逾時說明

縮圖在 `POST /photos` 的請求路徑上**同步生成**。對 < 500 張、個人使用的規模，這是可接受的取捨。若原圖較大，需確認 Railway 的 HTTP timeout 設定（預設 30s 通常足夠），Vercel proxy 端亦需設定對應 timeout。

---

## 前端渲染策略（SEO & Open Graph）

| 頁面 | 渲染策略 | OG Meta | 說明 |
|------|---------|---------|------|
| 首頁 `/` | ISR + client-side shuffle | 基本 site meta | ISR 產生靜態 HTML；hydration 後 JS 在 client 執行隨機洗牌 |
| 單張圖片 `/photos/[id]` | ISR | title、og:image（thumbnail_url）、description | 分享時需圖片預覽 |
| 相簿 `/albums/[slug]` | ISR | title、og:image（封面 thumbnail）、description | 分享需封面預覽 |
| 相簿列表 `/albums` | ISR | 基本 meta | |
| Tag 篩選 `/?tag=` | CSR | — | 動態 query，SEO 價值低 |
| Admin `/admin` | CSR | — | 不需 SEO，auth-protected |

### ISR Revalidation 機制

Go backend 在以下操作完成後，呼叫 `POST /api/revalidate`（Next.js API Route），觸發相關頁面 ISR 更新：

- 照片上傳、編輯、刪除、is_hidden 變更 → revalidate `/`
- 相簿建立、編輯、刪除、is_hidden 變更 → revalidate `/albums`、`/albums/[slug]`
- 圖片加入 / 移除相簿 → revalidate `/albums/[slug]`

Webhook 以共享 secret（`REVALIDATION_SECRET` env var）驗證請求合法性。

---

## 認證設計

- **機制**：NextAuth.js（Credentials provider）+ Go backend JWT
- **流程**：
  1. 前端送帳號密碼到 NextAuth API Route
  2. NextAuth 轉發到 Go backend `POST /auth/login`
  3. Go backend 查 DB 驗證 bcrypt，回傳 JWT（有效期 24 小時）
  4. NextAuth 將 JWT 存入 httpOnly session cookie
  5. 後續 Admin API 請求攜帶 JWT，Go backend middleware 驗證
- **帳號管理**：admins 表存唯一帳號，密碼以 bcrypt hash 儲存
- **Session 過期**：JWT 過期後自動登出，重新登入即可（無 refresh token）

---

## 圖片壓縮規格

| 類型 | 規格 |
|------|------|
| 縮圖（thumbnail） | 最大寬 800px，高等比，WebP，quality 80 |
| high_quality 模式 | 原始尺寸，統一輸出 JPEG quality 90 |
| original 模式 | byte-for-byte 原檔（透過 Presigned URL 直接存取 R2） |

縮圖在**上傳時**由 Go backend 產生（非請求時），避免重複運算。

---

## 潛在風險與對策

| 類型 | 風險 | 對策 |
|------|------|------|
| 安全性 | R2 originals 若意外公開，任何人可下載原圖 | R2 bucket 設為私有；前端存取原圖需向 Go backend 取 Presigned URL（有效期 15 分鐘） |
| 安全性 | Admin 登入暴力破解 | Go backend rate limiting on `POST /auth/login`；bcrypt 本身有計算成本 |
| 效能 | 大圖直傳 Vercel API Route 超過 4.5MB 限制 | 前端用 Presigned Upload URL 直傳 R2，完全繞過 Next.js |
| 效能 | 首頁載入大量縮圖 | lazy loading + offset pagination；ISR 快取頁面 HTML |
| 維運 | Supabase 免費方案閒置 7 天後暫停 | 設定 cron job 定期 ping，或升級至 $25/月方案 |
| 維運 | Railway 免費方案有月流量上限 | 監控用量；Go binary 極省資源 |
| 架構 | Next.js 與 Go 型別不同步 | Go backend 維護 OpenAPI spec；Next.js 依此手動或自動生成型別 |
| UX | Tag / Album rename 不更新 slug，Admin 需知道此限制 | 管理介面顯示 slug 欄位，提示「slug 建立後不可更改」 |
