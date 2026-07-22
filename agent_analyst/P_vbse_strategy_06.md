# P_vbse_strategy_06 — Trục 6: Watchlist 2-tier (20 mã: 10 Priority + 10 Standby)

File này detail Trục 6 của framework 6 trục. Dependency: chỉ `P_vbse_strategy_00` master (pack standalone, không cross-pack reference).

**Master weight balance áp dụng cho Trục 6:**
- **Tier 1 Phase 1 Screen:** PRIMARY (cơ bản + catalyst + thanh khoản). **Cap technical = 0%** ở phase này.
- **Tier 2 Phase 1 Screen:** cơ bản clean + KHÔNG catalyst tiêu cực + technical setup bottom-fishing (technical là criteria phụ trợ, không phải primary thesis).
- **Phase 2 Bucket entry:** TERTIARY (PTKT-driven). Áp đầy đủ cho Tier 1; restricted cho Tier 2 (không Bucket 1).

> ## ⚠️ TWO-TIER ARCHITECTURE — Watchlist 20 mã chia 2 tier
>
> **Tier 1 — Priority picks (10 mã, ưu tiên cao):**
> Cơ bản strong + catalyst rõ với ngày cụ thể + thanh khoản ≥5 tỷ ADV. Conviction HIGH/MID. Phase 2 Bucket entry 1/2/3 áp dụng đầy đủ.
>
> **Tier 2 — Standby (10 mã, chờ thời cơ):**
> Cơ bản clean (no red flag) + KHÔNG có catalyst tiêu cực + thanh khoản ≥3 tỷ ADV + technical setup bottom-fishing. Conviction MID/LOW (cap MID). Phase 2 Bucket entry: chỉ Bucket 2 hoặc 3 (KHÔNG Bucket 1 vì không có catalyst rõ để "vào ngay").
>
> Logic: Tier 1 là "active priority picks" — có catalyst kéo giá; Tier 2 là "patient bottom-fishing setup" — cơ bản không xấu, kỹ thuật đang đáy chờ bật. Tổng 20 mã, cả 2 tier đa dạng sector.

> ## ⚠️ TWO-PHASE ARCHITECTURE TRONG MỖI TIER — Screen vs Timing
>
> **Phase 1 — Screen universe (cơ bản-driven):**
> Tier 1: cơ bản strong + catalyst rõ. Tier 2: cơ bản clean + no negative catalyst + technical bottom-fishing setup. Tier 1 KHÔNG dùng technical_zone làm filter.
>
> **Phase 2 — Bucket entry timing (PTKT-driven):**
> Sau khi có watchlist 20 mã, phân Bucket 1/2/3 dựa technical_zone đa khung (định nghĩa inline ở mục 4.1). Tier 2 không vào Bucket 1.

## 1. Mục tiêu & câu hỏi cốt lõi

**Câu hỏi:** 20 mã đại diện themes & sector bias đã chốt — phân thành 10 Priority (Tier 1) + 10 Standby (Tier 2), mỗi mã có observation cơ bản, conviction, signal theo dõi, và bucket entry timing.

**Output:** watchlist 20 mã, mỗi mã có:
- Tier classification (1 hoặc 2)
- 6 thành phần fundamental (mục 3)
- Variant Perception (chỉ Tier 1, mandatory)
- ADV tháng (metadata)
- Bucket entry (1/2/3) — tách section riêng

### Phân bổ 2 tier theo Trục 2 định vị thị trường

| Trục 2 định vị | Tier 1 (Priority) | Tier 2 (Standby) | Total |
|---|---|---|---|
| Phục hồi sớm / bullish | 10 | 10 | 20 |
| Đỉnh / phân phối / suy yếu (bad market) | 8 | 12 | 20 |
| Quá mua (phân vị > 75%) | 6 | 14 | 20 |

Trigger **Bear regime mode** (mục 5) khi Trục 1 macro negative + Trục 2 định vị bad market — chỉ dựa vĩ mô, không dùng PTKT làm trigger.

## 2. Phase 1 — Screen universe (cơ bản-driven)

### 2.1. Universe scope

- **Ngành:** 18 ngành whitelist (xem `P_vbse_strategy_00` Nguyên tắc 3 + `K_agent_db_01` Section B)
- **Phân bổ sector Tier 1:** 60-70% từ ngành quan tâm Trục 4, 20-30% ngành trung tính có catalyst riêng, 0% ngành thận trọng
- **Phân bổ sector Tier 2:** đa dạng across ≥5 sectors (oversold quality từ nhiều ngành, không tập trung)
- **Cả 2 tier đa dạng sector** — không tập trung > 30% mã/sector ở mỗi tier

### 2.2. Filter Tier 1 — Priority picks (10 mã)

Bộ filter 5 lớp:

**(a) Thanh khoản — must-have:**
- ADV ≥ **5 tỷ/phiên** trung bình tháng (`stock_recent.series[0..19].price.trading_value` mean)

**(b) Tăng trưởng cơ bản — must-have ≥1:**
- EPS Q gần nhất YoY ≥ 15% (`stock_finstats.financial_statements.quarterly`)
- Doanh thu Q gần nhất YoY ≥ 10%
- Biên gộp / biên ròng Q cải thiện ≥ 50bp QoQ
- ROE TTM ≥ ngưỡng ngành (so với median ngành từ `industry_finstats`)

**(c) Catalyst — must-have ≥1 với ngày cụ thể trong horizon:**
- Catalyst cá thể (BCTC release, earnings call, ex-dividend, M&A close, capacity online)
- Catalyst ngành/chính sách (Nghị quyết / luật / QĐ bộ ngành mới)
- Catalyst commodity cycle (dầu/thép/USD chuyển pha trong horizon mã)

Cross-check `news_history_feed` filter ticker + web search.

**(d) Định giá — recommended:**
- P/E forward < median 5Y → upgrade conviction
- P/E forward > median 5Y × 1.3 → downgrade conviction (vẫn có thể giữ nếu (b) + (c) rất mạnh)

**(e) Negative catalyst gate — HARD REJECT (cùng áp cho Tier 2):**
Mã có ≥1 điều kiện sau → **loại hẳn khỏi Tier 1 và Tier 2**:
- Audit opinion qualified / adverse / disclaimer trong 4 quý gần
- Suspended / delisted warning
- Lãnh đạo (Chủ tịch / CEO / CFO) bị bắt / sai phạm pháp lý có công bố
- BCTC restate sang material direction (lợi nhuận thay đổi > 30% so với báo cáo cũ)
- Regulatory action ban hoạt động kinh doanh chính

### 2.3. Filter Tier 2 — Standby picks (10 mã)

Bộ filter 4 lớp (relaxed cơ bản, strict technical bottom-fishing):

**(a) Thanh khoản — must-have:**
- ADV ≥ **3 tỷ/phiên** trung bình tháng (nới hơn Tier 1)

**(b) Cơ bản clean — must-have ALL (không cần strong):**
- EPS Q gần nhất KHÔNG giảm > 30% YoY (không suy thoái mạnh)
- Doanh thu Q gần nhất KHÔNG giảm > 20% YoY
- Biên gộp KHÔNG thu hẹp > 200bp QoQ
- ROE TTM ≥ 5% (không yếu kém)

**(c) Technical bottom-fishing setup — must-have ≥1:**
- zone q HOẶC y ∈ {A, AA, AAA} + zone w VÀ m đang pullback ∈ {B, C}
- Hoặc: zone m ∈ {C, CC} ≥ 4 tuần (cạn đà ngắn hạn) + zone dài hạn không xấu
- Volume profile có vùng support rõ
- week_score đang zone âm nhưng có sign of stabilization (≥2 tuần không tệ hơn)

**(d) Negative catalyst gate — HARD REJECT cùng như Tier 1** (xem mục 2.2.e)

**(e) Soft negative catalyst — flag + cap conviction LOW (chỉ Tier 2, KHÔNG vào Tier 1):**
Mã có ≥1 điều kiện dưới → có thể vào Tier 2 với flag rõ + cap LOW:
- BCTC miss consensus 1 quý đơn lẻ (< 20%)
- Chính sách siết một phần ngành (hạn chế xuất khẩu, tăng thuế nhập khẩu)
- Regulatory observation (chưa phải action) — vd UBCK lưu ý
- Catalyst commodity tiêu cực ngắn hạn (giá dầu/thép giảm theo cycle)
- Sự kiện kế toán bất thường 1 lần (hợp nhất, divestiture) — không phải restate

### 2.4. Đường catalyst override (Tier 1 only)

Mã không đạt (b) Tăng trưởng cơ bản vẫn có thể vào Tier 1 nếu:
- (c) Catalyst rất mạnh (vd Nghị quyết lớn vừa ban hành tác động trực tiếp ngành mã)
- (a) Thanh khoản đạt
- (e) Negative catalyst gate clean
- Định giá không quá đắt (phân vị < 75%)

Khi đó **conviction CAP LOW** + disconfirming signal phải chặt + flag explicit "catalyst-driven, fundamental không support".

### 2.5. KHÔNG dùng trong Phase 1 Tier 1 Screen (LIỆT KÊ EXPLICIT)

Tier 1 Phase 1 cấm tuyệt đối:
- `stock_snapshot.technical_zone.overall.w/m/q/y` — KHÔNG làm filter
- `stock_snapshot.ma_zone` / `fibonacci_zone` / `volume_profile_zone` — KHÔNG làm filter
- `stock_snapshot.money_flow_score.day_score` đơn lẻ — KHÔNG làm filter

**Lý do:** technical filter ở Tier 1 sẽ loại mã có cơ bản tốt + định giá rẻ + catalyst rõ chỉ vì hiện tại technical yếu (vd đang pullback). Alpha leak lớn nhất.

**Lưu ý Tier 2 khác Tier 1:** Tier 2 dùng technical bottom-fishing setup làm criteria (mục 2.3.c) — đây là exception duy nhất, technical phụ trợ Tier 2 để identify bottom-fishing opportunity khi cơ bản clean nhưng chưa có catalyst rõ.

### 2.6. Số lượng mã sau Phase 1

Mặc định: Tier 1 = 10, Tier 2 = 10. Phân bổ điều chỉnh theo Trục 2 định vị (xem mục 1).

## 3. Output diễn giải mỗi mã — 6 thành phần (Phase 1)

Mỗi mã trong watchlist bắt buộc đủ 6 thành phần + tier marker:

1. **Ticker (ngành) — theme đại diện — Tier 1 hoặc Tier 2**

2. **Conviction:** HIGH / MID / LOW
   - **HIGH:** chỉ Tier 1, cross-check ≥2 trục đồng thuận + catalyst cơ bản/chính sách rõ ngày + định giá hợp lý (P/E forward ≤ median 5Y) + dòng tiền không ngược chiều
   - **MID:** Tier 1 (cơ bản pass, catalyst chưa có ngày cụ thể) hoặc Tier 2 (cơ bản clean + technical setup tốt)
   - **LOW:** đường catalyst override Tier 1, hoặc Tier 2 với soft negative catalyst flag

   **Conviction CAP rules — áp tuyệt đối:**
   - Theme/sector contradicting Trục 1 regime → cap LOW
   - Theme/sector không có catalyst với ngày cụ thể → cap MID
   - Sector chỉ "trung tính" Trục 4 → mã trong sector cap MID
   - Theme evidence từ < 2 trục → cap MID
   - Penny stock (vốn hoá < 1.000 tỷ) → cap LOW + chỉ được vào Tier 2
   - Newly listed (< 2 năm niêm yết) → cap MID
   - Mã có soft negative catalyst (mục 2.3.e) → cap LOW
   - Bear regime mode active (mục 5) → cap MID toàn pack

3. **Horizon:** 1m / 1-3m / 3-6m (theo timing catalyst materialize của theme)

4. **Luận điểm** (1-2 câu observation, không command): cơ chế theme → lý do mã hưởng lợi cụ thể về **cơ bản** (tăng trưởng / biên / chính sách hỗ trợ / catalyst), không chỉ technical.

5. **Signal theo dõi** (3-5 chỉ báo cụ thể — **PREFER cơ bản / catalyst / chính sách / định giá**, technical là phụ):
   - **Cơ bản (must-have ≥1):** vd "BCTC Q1 EPS growth ≥ 20% YoY", "Biên gộp Q1 cải thiện ≥ 50bp QoQ", "ROE TTM mở rộng từ 15% lên ≥18%"
   - **Catalyst / chính sách (must-have ≥1 cho Tier 1; optional Tier 2):** vd "Dự án X capacity Y MW online Q2", "Nghị quyết về sector Z thông qua tháng 5"
   - **Định giá (recommended):** vd "P/E forward về 9x vs median 5Y là 13x"
   - **Định vị/flow (secondary):** vd "Dòng tiền tuần duy trì dương ≥ 3/4 tuần", "FII mua ròng tháng"
   - **Technical (tertiary, chỉ ở Phase 2 Bucket entry, không ở section này)**

6. **Disconfirming signal** (1-2 dòng cụ thể — **PREFER cơ bản / catalyst / chính sách**): vd "BCTC Q1 EPS growth < 5% hoặc miss consensus ≥ 10%", "Dự án X delay sang Q4", "Chính sách Y bị huỷ bỏ"

**Variant Perception — MANDATORY chỉ cho Tier 1:**

Mỗi mã Tier 1 thêm 1 dòng:
- "Consensus đang nghĩ [X]"
- "Thesis này khác ở [Y với cơ sở định lượng]"

Nếu consensus đã trùng với thesis pack → flag "consensus crowded, alpha limited" + **cap MID** + demote sang Tier 2 nếu vẫn đáp ứng Tier 2 criteria.

Tier 2 KHÔNG bắt buộc Variant Perception.

**Metadata cuối mỗi entry — bắt buộc:**

- **Tier:** 1 hoặc 2
- **ADV tháng:** vd "ADV 28 tỷ" = mid-cap liquid; "ADV 4 tỷ" = small-cap Tier 2 setup
- **Bucket entry:** 1 / 2 / 3 (xem Phase 2 dưới)

## 4. Phase 2 — Bucket entry timing (PTKT-driven)

### 4.1. Định nghĩa Bucket (pack-internal)

Sau khi có universe watchlist từ Phase 1, phân mỗi mã vào 1 trong 3 bucket dựa trên `stock_snapshot.technical_zone.overall.w/m/q/y` + `money_flow_score.week_score`:

| Bucket | Điều kiện | Observation |
|---|---|---|
| **1 — Vào ngay được** | zone w ∈ {A, AA, AAA} VÀ zone m ∈ {A, AA, AAA} VÀ week_score ≥ 6 | Momentum đa khung đồng thuận. Mã sẵn sàng cho lệnh khi user quyết định vào, không cần đợi technical. |
| **2 — Chờ xác nhận (pullback trong uptrend)** | zone q ∈ {A, AA, AAA} HOẶC zone y ∈ {A, AA, AAA} NHƯNG zone w HOẶC zone m ∈ {B, C} | Uptrend dài hơi còn nguyên, ngắn hạn đang pullback. Cơ hội mua giá tốt khi tuần bật A. |
| **3 — Watchlist (chưa sẵn)** | zone q ∈ {A, AA, AAA} HOẶC zone y ∈ {A, AA, AAA} NHƯNG zone w VÀ zone m ĐỀU ∈ {C} | Pullback sâu — chờ technical phục hồi (zone tuần bật B+, week_score chuyển dương). |

### 4.2. Bucket entry áp dụng theo Tier

| Tier | Bucket 1 | Bucket 2 | Bucket 3 |
|---|---|---|---|
| Tier 1 (Priority) | Có thể default | Có thể | Có thể |
| Tier 2 (Standby) | **KHÔNG** (không có catalyst rõ để vào ngay) | Có thể (chờ pullback xác nhận) | Default (deep watch) |

### 4.3. Rule quan trọng cho Phase 2

- **Bucket KHÔNG nâng/giảm conviction theme.** Conviction đã chốt ở Phase 1 bằng cơ bản. Bucket chỉ là entry timing observation.
- **Mã catalyst override Tier 1 rơi vào Bucket 3:** flag rõ "catalyst-driven Bucket 3 — thị trường có thể priced-in tiêu cực hoặc chưa nhận ra. User review kỹ catalyst trước khi quyết định giữ/loại".
- **Quan hệ với Trục 2 định vị thị trường:**
  - Trục 2 "quá mua cảnh báo" (phân vị > 75%) → default downgrade Bucket 1 → Bucket 2 cho toàn bộ Tier 1. Tier 2 không thay đổi (đã không có Bucket 1). Flag explicit "downgrade do định vị thị trường quá mua".
  - Trục 2 "phân phối / suy yếu" + Trục 1 macro negative → trigger Bear regime mode (mục 5).

### 4.4. Cảnh báo Bucket 2 timeout

Mã Bucket 2 sau 4 tuần kể từ lúc vào watchlist mà zone w vẫn chưa bật A → flag inline "Bucket 2 timeout: thesis pullback chưa confirm sau 4 tuần, user xem xét chuyển Bucket 3 hoặc loại". Pack-internal tactical convention.

### 4.5. Rebucket trong weekly tracking

Trong workflow weekly (`P_vbse_strategy_08`), agent re-check technical_zone tuần cho từng mã watchlist → rebucket nếu có shift. Rebucket là **ngoại lệ duy nhất** được PTKT-driven trong weekly (xem `_08` mục 4). Rebucket KHÔNG là Shift thesis theme.

## 5. Bear regime mode

**Trigger (chỉ dựa vĩ mô + định vị, KHÔNG dùng PTKT):**
- Trục 1 macro negative (lãi tăng nhanh / GDP slowdown / chính sách thắt chặt) AND
- Trục 2 định vị "phân phối" hoặc "suy yếu"

Khi active, Trục 6 chuyển sang Bear regime mode với 5 điều chỉnh bắt buộc:

1. **Phân bổ tier:** Tier 1 = 8 mã, Tier 2 = 12 mã (theo bảng mục 1)
2. **Conviction CAP toàn pack:** MID là max (không HIGH) — bất kể cơ bản strong thế nào
3. **Defensive sectors floor:** ít nhất **40% Tier 1** thuộc defensive sectors (thực phẩm, điện, dược, nước)
4. **ADV threshold Tier 1 tăng lên 8 tỷ** (đòi thanh khoản cao để exit dễ khi market xấu)
5. **Mandatory bear case 1 đoạn cho mỗi Tier 1 mã** (không chỉ disconfirming signal — phải có scenario bear price/timeline + ngưỡng review)

Bear regime active sẽ flag ở Executive Summary monthly + render khác visual (badge "Bear regime mode active") trong O pack.

## 6. Hướng tìm dữ liệu

### Phase 1 Screen — Cơ bản + Catalyst + Thanh khoản + Negative catalyst gate

| Loại data | Collection | Field cụ thể |
|---|---|---|
| Universe ngành | `industry_info` filter whitelist 18 | `full_ticker_list` |
| Thanh khoản ADV 20 phiên | `stock_recent` | `series[0..19].price.trading_value` mean |
| EPS / doanh thu / biên / ROE | `stock_finstats` | `financial_statements.quarterly` |
| Định giá P/E forward / P/B | `stock_finstats` | `valuation_ratios.PE`, `PB` + cross `industry_finstats` median |
| Catalyst cá thể (positive + negative) | `news_history_feed` filter ticker + web search | `news_type`, body |
| Catalyst ngành | `news_history_feed` filter sector + web search | — |
| Audit opinion latest | `stock_finstats` | `audit_opinion` field (HARD gate) |
| Lãnh đạo sai phạm / suspended | `news_history_feed` + web | — |
| Vốn hoá (penny check) | `stock_info` | `market_cap` |
| Ngày niêm yết (newly listed check) | `stock_info` | `listing_date` |
| Flow NN/TD (cross-check, không filter) | `stock_nntd` | `month`, `quarter` aggregate |
| Cross-reference Trục 4 sector bias | Output Trục 4 | — |

### Phase 1 Tier 2 thêm — Technical bottom-fishing setup

| Loại data | Collection | Field cụ thể |
|---|---|---|
| Technical zone w/m/q/y mã | `stock_snapshot` | `technical_zone.overall.w/m/q/y` |
| Volume profile / support zone | `stock_snapshot` | `volume_profile_zone` |
| week_score stabilization | `stock_snapshot` | `money_flow_score.week_score` (chuỗi 4 tuần) |

### Phase 2 Bucket entry — PTKT đa khung

| Loại data | Collection | Field cụ thể |
|---|---|---|
| Technical zone w/m/q/y mã | `stock_snapshot` | `technical_zone.overall.w/m/q/y` |
| week_score mã | `stock_snapshot` | `money_flow_score.week_score` |
| day_score mã (cho rebucket weekly) | `stock_snapshot` | `money_flow_score.day_score` |
| Cross-check định vị thị trường | Output Trục 2 | — |

**Trọng số nguồn ước (toàn Trục 6):** ~85% DB + ~10% web + ~5% file user upload.

## 7. KHÔNG có trong watchlist

- Entry zone giá cụ thể
- Stop loss level
- Target giá
- Kích thước vị thế / sizing
- Ưu tiên thứ tự mua

Đây là **watchlist quan sát chiến lược**, KHÔNG phải lệnh giao dịch. Pack này dừng ở mức observation — không cover sizing/entry/stop/target.

## 8. Cross-reference đầu ra Trục 6

Output Trục 6 feed vào (nội bộ pack):
- **Executive summary monthly:** 3-5 mã tiêu biểu nhất (Tier 1 + Bucket 1 + HIGH conviction ưu tiên)
- **Workflow weekly tracking** (`_08`): mã Hold / Watch closely / Out / Vào mới + rebucket + Bear regime mode status

## 9. Edge cases

- **Theme có < 1 mã pass Phase 1 Tier 1:** flag "theme [X] chưa có mã đáp ứng Tier 1 — theme valid nhưng đại diện Tier 1 trống ở tháng N". Không bịa mã. Có thể vẫn có mã Tier 2 từ theme này.
- **Sector quan tâm có > 5 mã pass Phase 1 và ranking sát nhau:** cân nhắc phân bổ — không lấy > 30% mã/sector ở mỗi tier. Đa dạng sectors trong cả 2 tier.
- **Mã có cơ bản tốt nhưng catalyst chưa rõ thời điểm:** Tier 1 conviction LOW + ghi rõ "watching catalyst materialize" — hoặc demote sang Tier 2 nếu có technical setup tốt.
- **Mã ADV gần ngưỡng:** Tier 1 cần ≥5 tỷ, Tier 2 cần ≥3 tỷ — KHÔNG nới. Slippage thực tế ăn vào alpha nhiều hơn user nghĩ.
- **Mã có HARD negative catalyst (audit qualified/adverse/disclaimer, suspended, lãnh đạo sai phạm, BCTC restate material, regulatory action ban kinh doanh):** loại hẳn cả 2 tier. KHÔNG override.
- **Mã có SOFT negative catalyst (BCTC miss 1 quý < 20%, chính sách siết một phần, regulatory observation, commodity tiêu cực ngắn hạn, kế toán bất thường 1 lần):** flag + cap conviction LOW. Có thể vào Tier 2, KHÔNG vào Tier 1.
- **Mã thuộc ngành ngoài whitelist nhưng có catalyst rất rõ:** **KHÔNG vào watchlist** theo Nguyên tắc 3 master. User muốn deep-dive single ticker → activate pack chuyên biệt.
- **Mã Bucket 3 lâu (≥ 4 tuần) không bật:** trong weekly tracking, đề xuất user xem xét loại khỏi watchlist (chuyển "Out").
- **Bear regime mode trigger nhưng chưa có defensive sectors floor:** flag "Bear regime active nhưng Tier 1 chưa đủ 40% defensive — refresh universe ngành defensive trước khi finalize".
- **Tier 2 mã pass technical bottom-fishing setup nhưng cơ bản borderline:** ưu tiên defensive bias — nếu mã thuộc ngành cyclical (BĐS, vật liệu, chứng khoán) + setup chỉ marginal → loại; chỉ giữ nếu defensive sector hoặc setup rất rõ.

## 10. Self-audit Trục 6 (trước khi xuất)

### Phase 1 Screen — Tier 1
- [ ] 10 mã (hoặc 8 nếu Bear regime / 6 nếu Trục 2 quá mua)
- [ ] Universe chỉ từ 18 ngành whitelist
- [ ] Mỗi mã đạt thanh khoản ≥ 5 tỷ ADV (≥8 tỷ nếu Bear regime — không nới)
- [ ] Mỗi mã pass ≥1 tiêu chí tăng trưởng cơ bản (hoặc đường catalyst override với flag)
- [ ] Mỗi mã có ≥1 catalyst rõ với ngày cụ thể
- [ ] Mỗi mã có Variant Perception statement (consensus vs differentiated view)
- [ ] **0% technical filter Tier 1 Phase 1** — không mã nào bị loại vì technical_zone yếu
- [ ] Sector diversification: không > 30% mã/sector
- [ ] 60-70% mã từ ngành quan tâm Trục 4
- [ ] Signal theo dõi: must-have ≥1 cơ bản + ≥1 catalyst
- [ ] Disconfirming signal: PREFER cơ bản, technical phụ

### Phase 1 Screen — Tier 2
- [ ] 10 mã (hoặc 12 nếu Bear regime / 14 nếu Trục 2 quá mua)
- [ ] Universe chỉ từ 18 ngành whitelist
- [ ] Mỗi mã đạt thanh khoản ≥ 3 tỷ ADV
- [ ] Mỗi mã pass ALL tiêu chí cơ bản clean (không suy thoái mạnh)
- [ ] Mỗi mã có ≥1 technical bottom-fishing setup verified
- [ ] Mã có soft negative catalyst → flag rõ + cap LOW
- [ ] Sector diversification: ≥5 sectors, không > 30% mã/sector

### Negative catalyst gates
- [ ] HARD reject áp đúng: không có mã audit qualified/adverse/disclaimer, suspended, lãnh đạo sai phạm, BCTC restate material, regulatory action ban kinh doanh
- [ ] SOFT reject: mã có BCTC miss 1 quý / chính sách siết một phần / regulatory observation / commodity tiêu cực / kế toán bất thường → cap LOW + flag + chỉ Tier 2

### Conviction CAP rules
- [ ] Theme contradicting Trục 1 regime → cap LOW
- [ ] Theme không có catalyst với ngày → cap MID
- [ ] Sector "trung tính" Trục 4 → mã cap MID
- [ ] Theme evidence < 2 trục → cap MID
- [ ] Penny stock < 1.000 tỷ → cap LOW + chỉ Tier 2
- [ ] Newly listed < 2 năm → cap MID
- [ ] Bear regime active → cap MID toàn pack

### Phase 2 Bucket entry
- [ ] Mỗi mã có bucket 1 / 2 / 3 rõ ràng
- [ ] **Tier 2 KHÔNG có mã ở Bucket 1**
- [ ] Tiêu chí bucket theo định nghĩa mục 4.1 (pack-internal)
- [ ] Bucket KHÔNG nâng/giảm conviction (conviction đã chốt Phase 1)
- [ ] Mã catalyst override Tier 1 Bucket 3 có flag rõ
- [ ] Trục 2 "quá mua" → Tier 1 downgrade Bucket 1 → 2 với flag
- [ ] Bucket 2 mã ≥ 4 tuần chưa confirm → flag timeout (weekly tracking)

### Bear regime mode (khi trigger)
- [ ] Trigger đúng: Trục 1 macro negative AND Trục 2 định vị "phân phối/suy yếu" (KHÔNG dùng PTKT)
- [ ] Tier 1 = 8 mã, Tier 2 = 12 mã
- [ ] Conviction CAP MID toàn pack
- [ ] ≥40% Tier 1 thuộc defensive sectors (thực phẩm, điện, dược, nước)
- [ ] ADV Tier 1 ≥ 8 tỷ
- [ ] Mỗi Tier 1 mã có bear case 1 đoạn với scenario bear price/timeline

### Tổng
- [ ] 20 mã total, đúng phân bổ regime
- [ ] KHÔNG có entry/stop/target/size cụ thể
- [ ] % nội dung technical toàn Trục 6: Phase 1 Tier 1 = 0%, Phase 1 Tier 2 ≤ 25% (chỉ ở bottom-fishing setup), Phase 2 = 80-100%. Tỷ lệ Phase 2 / (Phase 1 + Phase 2) ≤ 30% nội dung trục
- [ ] Cross-reference Trục 4 sector quan tâm — phân bổ slot đúng

Vi phạm bất kỳ item nào → re-screen / re-bucket trước khi render.
