# P_vbse_strategy_06 — Trục 6: Watchlist 2-phase (Screen + Bucket entry)

File này detail Trục 6 của framework 6 trục. Dependency: `P_vbse_strategy_00` master + `P_invest_memo_03` (reference cho bucket entry definition).

**Master weight balance áp dụng cho Trục 6:**
- **Phase 1 Screen:** PRIMARY (cơ bản + catalyst + thanh khoản). **Cap technical = 0% (CẤM TUYỆT ĐỐI)** ở phase này.
- **Phase 2 Bucket entry:** TERTIARY (PTKT-driven). Đây là **nơi duy nhất** PTKT có vai trò hợp pháp trong toàn pack (~80-100% nội dung Phase 2 là technical).

> ## ⚠️ TWO-PHASE ARCHITECTURE — Tách hoàn toàn screen vs timing
>
> **Phase 1 — Screen universe (cơ bản-only):**
> Chọn 5-12 mã đại diện theme + sector bias bằng **cơ bản + catalyst + thanh khoản**. KHÔNG dùng technical_zone, MA, Fibonacci, volume profile làm filter loại universe. Mã có cơ bản tốt + catalyst rõ nhưng technical xấu → **vẫn vào watchlist**.
>
> **Phase 2 — Bucket entry timing (PTKT-driven):**
> Sau khi đã có universe, dùng technical_zone đa khung (w/m/q/y) phân mã thành Bucket 1 / 2 / 3 (định nghĩa từ `P_invest_memo_03` mục 5). Bucket KHÔNG nâng/giảm conviction theme — chỉ là entry timing observation cho user.
>
> Logic: cơ bản quyết định **"có vào không"**, technical chỉ quyết định **"vào lúc nào"**.

## 1. Mục tiêu & câu hỏi cốt lõi

**Câu hỏi:** 5-12 mã đại diện cho themes & sector bias đã chốt, **observation** từng mã ra sao về cơ bản + catalyst, và **bucket entry timing** ra sao?

**Output:** watchlist 5-12 mã, mỗi mã có:
- 6 thành phần fundamental (mục 3 dưới)
- ADV tháng (metadata)
- Bucket entry (1 / 2 / 3) — tách section riêng

## 2. Phase 1 — Screen universe (cơ bản-only)

### 2.1. Universe scope

- **Ngành:** 18 ngành whitelist (xem `P_vbse_strategy_00` Nguyên tắc 3 + `K_agent_db_01` Section B)
- **Mã thuộc ngành "quan tâm"** từ Trục 4 — ưu tiên 60-70% slot watchlist
- **Mã thuộc ngành "trung tính"** có catalyst riêng đặc biệt — 20-30% slot
- **Mã thuộc ngành "cần thận trọng"** — 0% (loại khỏi watchlist)
- Mỗi theme/sector quan tâm: 1-3 mã đại diện

### 2.2. Filter — chỉ cơ bản + catalyst + thanh khoản

**Bộ filter 4 lớp (cấm PTKT trong cả 4):**

**(a) Thanh khoản — must-have:**
- `stock_recent.series[0..19].price.trading_value` mean ≥ **5 tỷ/phiên** trung bình tháng (ADV ≥ 5 tỷ)
- Loại nhiễu penny + mã không trade được size đủ

**(b) Tăng trưởng cơ bản — must-have ≥1 trong các tiêu chí:**
- EPS Q gần nhất YoY ≥ 15% (`stock_finstats.financial_statements.quarterly`)
- Doanh thu Q gần nhất YoY ≥ 10%
- Biên gộp / biên ròng Q cải thiện ≥ 50bp QoQ
- ROE TTM ≥ ngưỡng ngành (so với median ngành từ `industry_finstats`)

Mã không đạt ≥1 tiêu chí trên → **loại khỏi universe** (trừ đường catalyst override mục 2.3).

**(c) Catalyst — must-have ≥1 trong các loại:**
- Catalyst cá thể có ngày cụ thể (BCTC release, earnings call, ex-dividend, M&A close, capacity online)
- Catalyst ngành/chính sách (Nghị quyết / luật / QĐ bộ ngành mới)
- Catalyst commodity cycle (dầu/thép/USD chuyển pha trong horizon mã)

Cross-check `news_history_feed` filter ticker + web search.

**(d) Định giá — recommended (không loại nhưng giảm conviction):**
- P/E forward < median 5Y (rẻ tương đối) → upgrade conviction
- P/E forward > median 5Y × 1.3 (đắt rõ ràng) → downgrade conviction (vẫn có thể giữ nếu (b) + (c) rất mạnh)

### 2.3. Đường catalyst override

Mã không đạt **(b) Tăng trưởng cơ bản** vẫn có thể vào universe nếu:
- (c) Catalyst rất mạnh (vd Nghị quyết lớn vừa ban hành tác động trực tiếp ngành mã)
- (a) Thanh khoản đạt
- Định giá không quá đắt (phân vị < 75%)

Khi đó **conviction LOW** + disconfirming signal phải chặt + flag explicit "catalyst-driven, fundamental không support".

### 2.4. KHÔNG dùng trong Phase 1 (LIỆT KÊ EXPLICIT)

**Cấm tuyệt đối ở Phase 1 Screen:**
- `stock_snapshot.technical_zone.overall.w/m/q/y` — KHÔNG làm filter
- `stock_snapshot.ma_zone` / `fibonacci_zone` / `volume_profile_zone` — KHÔNG làm filter
- Trend đa khung (`stock_snapshot.trend.*` — vốn không có ở stock_snapshot theo `K_agent_db_01`)
- `stock_snapshot.money_flow_score.day_score` đơn lẻ — KHÔNG làm filter

**Lý do:** technical filter ở Phase 1 sẽ loại mã có cơ bản tốt + định giá rẻ + catalyst rõ chỉ vì hiện tại technical yếu (vd đang pullback). Đây là alpha leak lớn nhất — đặc biệt là Bucket 2 mã (pullback trong uptrend). Pack này không cho phép.

### 2.5. Số lượng mã sau Phase 1

Sau Phase 1 Screen, expect 8-20 mã/theme. Phase 2 sẽ thu về 5-12 mã final (tổng cộng).

## 3. Output diễn giải mỗi mã — 6 thành phần (Phase 1)

Mỗi mã trong watchlist bắt buộc đủ 6 thành phần (chuẩn buy-side):

1. **Ticker (ngành) — theme đại diện**

2. **Conviction:** HIGH / MID / LOW
   - **HIGH:** cross-check ≥2 trục đồng thuận + có catalyst cơ bản/chính sách rõ ngày + định giá hợp lý (P/E forward ≤ median 5Y) + dòng tiền không ngược chiều
   - **MID:** cơ bản pass + catalyst rõ nhưng chưa có ngày + định giá phân vị 40-60%
   - **LOW:** đường catalyst override, hoặc cơ bản pass nhưng catalyst chưa rõ thời điểm

3. **Horizon:** 1m / 1-3m / 3-6m (theo timing catalyst materialize của theme)

4. **Luận điểm** (1-2 câu observation, không command): cơ chế theme → lý do mã hưởng lợi cụ thể về **cơ bản** (tăng trưởng / biên / chính sách hỗ trợ / catalyst), không chỉ technical.

5. **Signal theo dõi** (3-5 chỉ báo cụ thể — **PREFER cơ bản / catalyst / chính sách / định giá**, technical là phụ):
   - **Cơ bản (must-have ≥1):** vd "BCTC Q1 EPS growth ≥ 20% YoY", "Biên gộp Q1 cải thiện ≥ 50bp QoQ", "ROE TTM mở rộng từ 15% lên ≥18%", "Doanh thu Q1 vượt consensus ≥ 8%"
   - **Catalyst / chính sách (must-have ≥1):** vd "Dự án X capacity Y MW online Q2", "Nghị quyết về sector Z thông qua tháng 5", "M&A close công ty con tháng 6", "Ngày chốt cổ tức tiền mặt 7%", "Earnings call ngày DD/MM"
   - **Định giá (recommended):** vd "P/E forward về 9x vs median 5Y là 13x", "P/B 1.1x vs ngành 1.6x"
   - **Định vị/flow (secondary):** vd "Dòng tiền tuần duy trì dương ≥ 3/4 tuần", "FII mua ròng tháng"
   - **Technical (tertiary, chỉ ở Phase 2 Bucket entry, không ở section signal theo dõi):** Section này KHÔNG nhắc technical làm signal theo dõi — technical đã có vai trò ở Phase 2 Bucket.

6. **Disconfirming signal** (1-2 dòng cụ thể — **PREFER cơ bản / catalyst / chính sách**): vd "BCTC Q1 EPS growth < 5% hoặc miss consensus ≥ 10%", "Biên gộp Q1 thu hẹp ≥ 100bp QoQ", "Dự án X delay sang Q4", "Chính sách Y bị huỷ bỏ".

**Metadata cuối mỗi entry — bắt buộc:**

- **ADV tháng:** `stock_recent.series[0..19].price.trading_value` mean — vd "ADV 28 tỷ" = mid-cap liquid; "ADV 6 tỷ" = small-cap sát ngưỡng filter
- **Bucket entry:** 1 / 2 / 3 (xem Phase 2 dưới)

## 4. Phase 2 — Bucket entry timing (PTKT-driven)

### 4.1. Định nghĩa Bucket (theo `P_invest_memo_03` mục 5)

Sau khi có universe watchlist từ Phase 1, phân mỗi mã vào 1 trong 3 bucket dựa trên `stock_snapshot.technical_zone.overall.w/m/q/y` + `money_flow_score.week_score`:

| Bucket | Điều kiện | Observation |
|---|---|---|
| **1 — Vào ngay được** | zone w ∈ {A, AA, AAA} VÀ zone m ∈ {A, AA, AAA} VÀ week_score ≥ 6 | Momentum đa khung đồng thuận. Mã sẵn sàng cho lệnh khi user quyết định vào, không cần đợi technical. |
| **2 — Chờ xác nhận (pullback trong uptrend)** | zone q ∈ {A, AA, AAA} HOẶC zone y ∈ {A, AA, AAA} NHƯNG zone w HOẶC zone m ∈ {B, C} | Uptrend dài hơi còn nguyên, ngắn hạn đang pullback. Cơ hội mua giá tốt khi tuần bật A. |
| **3 — Watchlist (chưa sẵn)** | zone q ∈ {A, AA, AAA} HOẶC zone y ∈ {A, AA, AAA} NHƯNG zone w VÀ zone m ĐỀU ∈ {C} | Pullback sâu — chờ technical phục hồi (zone tuần bật B+, week_score chuyển dương). |

### 4.2. Rule quan trọng cho Phase 2

- **Bucket KHÔNG nâng/giảm conviction theme.** Conviction đã chốt ở Phase 1 bằng cơ bản. Bucket chỉ là entry timing observation.
- **Mã catalyst override (đường C, fail cơ bản) rơi vào Bucket 3:** flag rõ "catalyst-driven Bucket 3 — thị trường có thể priced-in tiêu cực hoặc chưa nhận ra. User review kỹ catalyst trước khi quyết định giữ/loại".
- **Quan hệ với Trục 2 định vị thị trường:** nếu Trục 2 chốt "quá mua cảnh báo" (phân vị > 75%) → default downgrade Bucket 1 → Bucket 2 cho toàn bộ watchlist. Flag explicit "downgrade do định vị thị trường quá mua".

### 4.3. Cảnh báo Bucket 2 timeout

Mã Bucket 2 sau 4 tuần kể từ lúc vào watchlist mà zone w vẫn chưa bật A → flag inline "Bucket 2 timeout: thesis pullback chưa confirm sau 4 tuần, user xem xét chuyển Bucket 3 hoặc loại". Đây là tactical convention từ `P_invest_memo_08` mục 5.2 + `P_invest_memo_09` mục 5.2.

### 4.4. Rebucket trong weekly tracking

Trong workflow weekly (`P_vbse_strategy_08`), agent re-check technical_zone tuần cho từng mã watchlist → rebucket nếu có shift. Rebucket là **ngoại lệ duy nhất** được PTKT-driven trong weekly (xem `_08` mục 4). Rebucket KHÔNG là Shift thesis theme.

## 5. Hướng tìm dữ liệu

### Phase 1 Screen — Cơ bản + Catalyst + Thanh khoản

| Loại data | Collection | Field cụ thể |
|---|---|---|
| Universe ngành | `industry_info` filter whitelist 18 | `full_ticker_list` |
| Thanh khoản ADV 20 phiên | `stock_recent` | `series[0..19].price.trading_value` mean |
| EPS / doanh thu / biên / ROE | `stock_finstats` | `financial_statements.quarterly` |
| Định giá P/E forward / P/B | `stock_finstats` | `valuation_ratios.PE`, `PB` + cross `industry_finstats` median |
| Catalyst cá thể | `news_history_feed` filter ticker + web search | `news_type`, body |
| Catalyst ngành | `news_history_feed` filter sector + web search | — |
| Flow NN/TD (cross-check, không filter) | `stock_nntd` | `month`, `quarter` aggregate |
| Cross-reference Trục 4 sector bias | Output Trục 4 | — |

### Phase 2 Bucket entry — PTKT đa khung

| Loại data | Collection | Field cụ thể |
|---|---|---|
| Technical zone w/m/q/y mã | `stock_snapshot` | `technical_zone.overall.w/m/q/y` |
| week_score mã | `stock_snapshot` | `money_flow_score.week_score` |
| day_score mã (cho rebucket weekly) | `stock_snapshot` | `money_flow_score.day_score` |
| Cross-check định vị thị trường | Output Trục 2 | — |

**Trọng số nguồn ước (toàn Trục 6):** ~85% DB + ~10% web + ~5% file user upload.

## 6. KHÔNG có trong watchlist

- Entry zone giá cụ thể
- Stop loss level
- Target giá
- Kích thước vị thế / sizing
- Ưu tiên thứ tự mua

Đây là **watchlist quan sát chiến lược**, KHÔNG phải lệnh giao dịch. Sizing + entry/stop/target là job của `P_invest_memo` (memo deep-dive).

## 7. Cross-reference đầu ra Trục 6

Output Trục 6 feed vào:
- **Executive summary** monthly: 1-3 mã tiêu biểu nhất (Bucket 1 + HIGH conviction ưu tiên)
- **Workflow weekly tracking** (`_08`): mã Hold / Watch closely / Out / Vào mới + rebucket
- **Pack `P_invest_memo`** (nếu user chạy memo deep-dive): watchlist này là input candidate cho Tier 5C memo

## 8. Edge cases

- **Theme có < 1 mã pass Phase 1:** flag "theme [X] chưa có mã đáp ứng filter cơ bản — theme valid nhưng watchlist trống ở theme này tháng N". Không bịa mã.
- **Ngành quan tâm có > 3 mã pass Phase 1 và ranking sát nhau:** cân nhắc phân bổ bucket — không lấy 3 mã cùng Bucket 1, ưu tiên đa dạng bucket để linh hoạt entry timing (mirror `P_invest_memo_04` cân bằng bucket logic).
- **Mã có cơ bản tốt nhưng catalyst chưa rõ thời điểm:** Phase 1 conviction LOW + Bucket tùy technical, ghi rõ "watching catalyst materialize" trong signal theo dõi.
- **Mã ADV gần ngưỡng 5 tỷ (vd 4.8 tỷ):** loại khỏi watchlist. KHÔNG nới ngưỡng. Slippage thực tế ăn vào alpha nhiều hơn user nghĩ.
- **Mã có sự kiện kế toán bất thường (vd hợp nhất 1 lần, BCTC restate):** flag riêng + loại khỏi conviction HIGH cho đến khi 2 quý liên tiếp clean data.
- **Mã thuộc ngành ngoài whitelist nhưng có catalyst rất rõ:** **KHÔNG vào watchlist** theo Nguyên tắc 3 master. User muốn phân tích → dùng `P_invest_memo` đơn lẻ ticker.
- **Mã Bucket 3 lâu (≥ 4 tuần) không bật:** trong weekly tracking, đề xuất user xem xét loại khỏi watchlist (chuyển sang "Out" trạng thái weekly).

## 9. Self-audit Trục 6 (trước khi xuất)

### Phase 1 Screen
- [ ] Universe chỉ từ 18 ngành whitelist
- [ ] Mỗi mã đạt thanh khoản ≥ 5 tỷ ADV (không nới)
- [ ] Mỗi mã pass ≥1 tiêu chí tăng trưởng cơ bản (hoặc đường catalyst override với flag rõ)
- [ ] Mỗi mã có ≥1 catalyst rõ (cá thể / ngành / commodity)
- [ ] **0% technical filter ở Phase 1** — không có mã bị loại vì technical_zone yếu
- [ ] Mỗi mã có đủ 6 thành phần fundamental + ADV metadata
- [ ] Signal theo dõi: must-have ≥1 cơ bản + ≥1 catalyst
- [ ] Disconfirming signal: PREFER cơ bản, technical phụ

### Phase 2 Bucket entry
- [ ] Mỗi mã có bucket 1 / 2 / 3 rõ ràng
- [ ] Tiêu chí bucket theo `P_invest_memo_03` mục 5 (không tự sáng chế tiêu chí)
- [ ] Bucket KHÔNG nâng/giảm conviction (conviction đã chốt ở Phase 1)
- [ ] Mã catalyst override Bucket 3 có flag rõ
- [ ] Nếu Trục 2 chốt "quá mua" → downgrade Bucket 1 → 2 toàn watchlist với flag rõ
- [ ] Bucket 2 mã >= 4 tuần chưa confirm → flag timeout (trong weekly tracking)

### Tổng
- [ ] KHÔNG có entry/stop/target/size cụ thể
- [ ] Cross-reference Trục 4 sector quan tâm — phân bổ slot đúng 60-70% ngành quan tâm
- [ ] 5-12 mã total, không phình ra ngoài range
- [ ] % nội dung technical toàn Trục 6: Phase 1 = 0%, Phase 2 = 80-100%. Tỷ lệ Phase 2 / (Phase 1 + Phase 2) ≤ 30% nội dung trục.

Vi phạm bất kỳ item nào → re-screen / re-bucket trước khi render.
