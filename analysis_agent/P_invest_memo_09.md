# P_invest_memo_09 — Tier 7: Monitoring + Exit Execution

Giai đoạn 7 của quy trình, bắt đầu sau khi orders tier 6 đã filled. Đây là giai đoạn **vận hành liên tục** — monitor positions, execute exit triggers, manage Bucket 2/3 future entries, refresh thesis theo chu kỳ.

Reference: `P_invest_memo_00` phần Flow chi tiết (overview), `P_invest_memo_00` Nguyên tắc 2 (exit triggers trước position), `P_invest_memo_07` (tier 5C memo với 3 loại exit trigger), `P_invest_memo_08` (tier 6 portfolio với Bucket 2/3 pending).

---

## 1. Mục tiêu & output expected

**Mục tiêu:** duy trì discipline từ memo sang thực thi. Cụ thể:
- Monitor các exit trigger đã define trong tier 5C memo
- Execute hard triggers ngay khi match (không thảo luận)
- Cảnh báo user khi soft triggers match (user quyết định)
- Thực thi take profit theo plan khi giá đạt target
- Manage Phase 2 entry cho Bucket 2, Phase 1 entry cho Bucket 3 khi trigger chuyển
- Refresh thesis quá 30 ngày hoặc khi có data mới đáng kể
- Portfolio-level health check (concentration drift, regime shift)

**Input:**
1. Output tier 6 — portfolio với Phase 1 filled + Phase 2/3 pending conditions
2. Output tier 5C — memo per-stock với 3 loại exit trigger
3. DB state hiện tại — price, money_flow_score, technical_zone, news
4. User confirmation về positions filled thực tế (price fill, số cổ phiếu)

**Output chính:**
1. **Daily quick scan** (5-10 phút Agent work) — flag events need attention
2. **Weekly review** (30-45 phút) — full position health
3. **Monthly review** (1-2 giờ) — portfolio-level + memo refresh
4. **Quarterly review** (2-3 giờ) — BCTC mới, tier 5A/5B re-check
5. **Exit execution proposals** — khi trigger match, với action cụ thể
6. **End-of-cycle reconcile** — sau khi thoát hết positions, quay lại tier 0 cho cycle mới

**Tần suất session:** theo review cycle (daily/weekly/monthly/quarterly).

---

## 2. Triết lý

**Monitoring chống emotional trading.** Hard triggers định trong memo trước khi enter — lúc đó thesis rõ ràng, chưa dính emotion. Khi price giảm 8-10%, emotion kéo user muốn "chờ thêm, có thể bật lại". Discipline memo triggers ngăn holding qua deterioration.

**Exit đúng lúc quan trọng hơn entry đúng giá.** Nhiều case thesis correct nhưng miss timing exit → profit erode. Hard triggers là guarantee: khi match, action không negotiate.

**Agent không auto-execute lệnh.** Giống tier 6, Agent đề xuất action + lý do, user đặt lệnh qua broker. Nguyên tắc flex: Agent có thể đề xuất đợi 1-2 phiên trong case boundary, nhưng hard trigger match → proposal rõ ràng "bán ngay".

**Memo expiry 30 ngày là soft rule.** Nếu thesis vẫn intact + không có data mới → có thể extend 1-2 tuần. Nếu có BCTC quý mới hoặc catalyst xảy ra → rewrite memo ngay dù chưa đủ 30 ngày.

**Portfolio-level check định kỳ.** Position-level focus vào từng mã; portfolio-level focus vào:
- Concentration drift (1 mã lên quá mạnh → tỷ trọng vượt cap 10%)
- Regime change (tier 0 shift → adjust aggressiveness)
- Sector rotation (1 ngành quá mạnh trong portfolio nhưng ngành đang deteriorate)

---

## 3. 4 review cycles

### 3.1. Daily quick scan (5-10 phút)

**Mục tiêu:** check nhanh nếu có hard trigger match hoặc news quan trọng.

**Bước:**

1. **Price check** — các mã trong portfolio + watchlist (Bucket 3):
   - Giá current vs levels quan trọng (Bear × 0.9, Base, Bull)
   - Flag mã có pct_change hôm nay vượt ±5 (field đã là điểm %, đọc thẳng)

2. **Hard trigger auto-check** (per-stock):
   - Nếu price < Bear × 0.9 → **ALERT** proposal bán ngay
   - Query news_today_feed theo tickers portfolio → check có news CEO/CFO từ chức, vi phạm UBCK, scandal không

3. **Phase 2 confirm check** cho Bucket 2:
   - Query stock_snapshot.technical_zone.overall.w
   - Nếu zone w bật A và week_score dương → flag "Phase 2 confirm available"

4. **Phase 3 trigger check** cho Bucket 3:
   - Query zone w + week_score + day_score 3 phiên
   - Nếu condition match → flag "Bucket 3 → Bucket 2, xem xét vào position"

5. **Output:** report ngắn dạng bullet, 3-5 dòng. Nếu không có alert → "No action required today".

### 3.2. Weekly review (30-45 phút)

Mỗi thứ Hai đầu tuần hoặc cuối tuần trước (tuỳ user preference).

**Mục tiêu:** full position health, soft trigger check, Bucket 2/3 management.

**Bước:**

1. **Position health per-stock:**
   - P&L: unrealized gain/loss
   - Current price vs levels (entry, Base, Bull, Bear)
   - % toward target: price/Base × 100

2. **Soft trigger check:**
   - Xếp hạng dòng tiền thị trường trend 2 tuần gần (nội bộ `market_rank_pct`)
   - Vùng kỹ thuật khung quý — có chuyển từ A về B không? (nội bộ `technical_zone.overall.q`)
   - DSO trend (nếu có BCTC mới)
   - Flow NN 1 tháng

3. **Bucket 2 Phase 2 review:**
   - Mã nào đã confirm → đề xuất vào tranche 2
   - Mã nào gần timeout 4 tuần → cân nhắc abort
   - Mã nào vẫn trong pullback đúng thesis → giữ plan

4. **Bucket 3 trigger review:**
   - Mã nào đã chuyển sang Bucket 2 → apply rule Phase 2
   - Mã nào vẫn C vùng sâu → tiếp tục watchlist

5. **Take profit partial** (nếu price đạt Base):
   - Mã đạt Base → proposal bán 50%
   - Mã đạt Bull → proposal bán thêm 30%

6. **Output:** Weekly review session dạng bảng per-stock + portfolio summary.

### 3.3. Monthly review (1-2 giờ)

Đầu tháng hoặc tuần đầu mỗi tháng.

**Mục tiêu:** portfolio-level health + memo refresh + regime check.

**Bước:**

1. **Portfolio-level metrics:**
   - Total return YTD / MoM
   - Win rate: # positions profitable / total
   - Avg gain winners vs avg loss losers
   - Max drawdown (mức sụt giảm sâu nhất từ đỉnh đến đáy trong một khoảng thời gian, đo rủi ro peak-to-trough)
   - Sharpe-like (return / volatility) — nếu có chuỗi dài

2. **Concentration drift check:**
   - Mã nào % portfolio đã drift > 10% cap?
   - Ngành nào đã drift > 30-40% cap?
   - Nếu drift quá mức → đề xuất trim partial (bán 20-30%) về dưới cap

3. **Memo expiry check (30 ngày rule):**
   - Memo nào quá 30 ngày kể từ viết? → flag refresh
   - Refresh memo theo checklist: consensus update, BCTC quý mới nếu có, catalyst timeline update, thesis vẫn intact không
   - Nếu thesis changed → re-check gate 1 (variant perception) + gate 2 (bear case)

4. **Tier 0 regime re-check:**
   - Query market snapshot + breadth + trend
   - Compare với regime confirmed cuối tier 0
   - Nếu regime shift (Risk-on → Defensive chẳng hạn) → re-run tier 6 sizing, cân nhắc giảm exposure

5. **Missed opportunities:**
   - Mã tier 3 sát nút (không vào shortlist) có biến động lớn không?
   - Ngành không vào tier 1 có catalyst mới không?
   - Note learning để cải thiện tier 1-3 cycle sau

6. **Output:** Monthly review session với 6 mục trên + proposal rebalance nếu cần.

### 3.4. Quarterly review (2-3 giờ)

Sau khi BCTC quý của các mã trong portfolio công bố (thường cuối tháng kế tiếp quý).

**Mục tiêu:** re-verify thesis với data mới + tier 5A-5B partial re-check.

**Bước:**

1. **BCTC quý mới cho từng mã:**
   - Query stock_finstats để check period mới
   - Nếu có: extract Revenue Growth, NPAT Growth, ROE, key margins quý mới

2. **Thesis check theo BCTC thực vs memo assumption:**
   - Revenue Growth thực vs Base case assumption memo
   - Margin thực vs Base case
   - Có miss KPI nào catalyst memo liệt kê không?
   - Nếu miss major KPI → hard trigger có thể apply

3. **Re-run tier 5A forensic partial:**
   - Focus vào BCTC quý: CFO/LNST ratio, DSO trend, nhà cung cấp/khách hàng lớn có changed không
   - Không full re-run nhưng spot check 2-3 tác vụ quan trọng nhất

4. **Valuation update:**
   - Re-run quick DCF với BCTC quý mới → target có đổi không?
   - Nếu target thay đổi > 10% → update memo và monitoring levels

5. **Exit triggers update:**
   - Nếu thesis đã partial realize (ví dụ catalyst 1 realize, target re-rating đợt 1) → đổi hard trigger level (Bear level mới thay cũ)
   - Nếu thesis deteriorating → tighten exit (thay vì Bear × 0.9 → Bear × 0.95)

6. **Output:** Quarterly review session với status per-stock (thesis intact / partial realized / deteriorating / failed) + update exit levels + proposal rebalance.

---

## 4. Exit trigger execution — 3 loại

Tier 5C memo đã define 3 loại trigger. Tier 7 là nơi thực thi.

### 4.1. Hard triggers (bán ngay)

**Logic:** trigger match → proposal "bán ngay" rõ ràng, không thảo luận về "có nên chờ thêm".

**Ví dụ hard triggers từ memo:**
- Price < Bear × 0.9
- BCTC quý show major KPI fail (e.g., revenue decline mà consensus đã dự giảm nhẹ)
- CEO/CFO từ chức đột ngột không lý do rõ
- UBCK công bố điều tra vi phạm
- Audit opinion qualified kỳ tiếp theo

**Agent action khi trigger match:**

```
ALERT HARD TRIGGER — [Ticker] [ngày]

Trigger matched: [loại trigger cụ thể]
Source verified: [link news, BCTC page, price data]
Current position: [% portfolio, VND amount, unrealized P&L]

Proposal: BÁN NGAY toàn bộ position [X]% portfolio
Rationale: thesis [collapsed/major risk materialized]
Limit order: market sell hoặc limit sát bid để fill nhanh
Expected impact portfolio: [Y% loss realize]

User confirm để đặt lệnh?
```

**User flex:** có thể request "chờ 1 phiên xem bounce", nhưng:
- Agent ghi audit log "user delayed hard trigger execution bởi lý do X"
- Max delay 1 phiên, nếu phiên tiếp theo vẫn dưới trigger level → execute không negotiate

### 4.2. Soft triggers (cảnh báo + proposal)

**Logic:** trigger match → flag cảnh báo + đề xuất action, user quyết định.

**Ví dụ soft triggers từ memo (output user-facing dùng wording dịch; raw token chỉ trong audit log):**
- Xếp hạng dòng tiền thị trường rơi xuống top 70% (dưới rank_pct 30, percentile 0-100) trong 2 tuần
- Vùng kỹ thuật khung quý chuyển từ tích cực về trung tính (zone A → B)
- DSO quý mới tăng > 10 ngày so cùng kỳ
- Flow NN bán ròng > 2 tuần liên tiếp
- Key competitor công bố kết quả gây concern cho mã

**Agent action khi trigger match:**

```
SOFT TRIGGER CAUTION — [Ticker] [ngày]

Trigger matched: [loại trigger cụ thể]
Data: [số liệu cụ thể]
Thesis impact: [partial / moderate / major]

Proposals (user chọn):
(a) Giảm size 30-50% position, giữ phần còn lại, tiếp tục monitor
(b) Tighten hard trigger level (ví dụ Bear × 0.9 → Bear × 0.95)
(c) Giữ nguyên position nhưng review memo trong 1 tuần
(d) Bán toàn bộ nếu user thấy thesis đã fundamentally changed

Rationale for (a): [lý do tại sao giảm 30-50% là balanced]
Rationale for (b): [...]
...

User quyết định + ghi audit log?
```

**Flex nguyên tắc:** user có full info, chọn proposal phù hợp với niềm tin thesis. Agent không ép action nào.

### 4.3. Take profit triggers (theo plan)

**Logic:** price đạt target → thực thi take profit partial theo plan.

**Plan từ memo (tier 5C):**
- Price đạt Base target → bán 50% position
- Price đạt Bull target → bán thêm 30% (còn 20%)
- Price vượt Bull × 1.15 → re-examine thesis, có justify không? Nếu không → bán phần còn lại

**Agent action khi trigger match:**

```
TAKE PROFIT TRIGGER — [Ticker] [ngày]

Price đạt: Base target [X]k / Bull target [X]k / Bull +15% [X]k
Current position: [% portfolio, VND amount, unrealized gain]

Proposal theo plan memo:
- Bán [50%/30%/rest] position
- Limit order: current - 1-2 tick để fill nhanh
- Remaining position: [X]% (với exit triggers cập nhật)

Rationale: thesis [realized partial / realized full / overshoot suspected]

User confirm + optional adjustments:
- (a) Confirm bán theo plan
- (b) Adjust % bán (ví dụ 40% thay 50%) nếu user tin thesis còn upside
- (c) Delay 1 tuần nếu price action parabolic (có thể tiếp tục)
- (d) Bán nhiều hơn plan nếu user muốn de-risk
```

**Flex nguyên tắc:** plan là anchor nhưng user có thể adjust ± 10-20% dựa trên price action context (parabolic vs consolidation). Không adjust > 30% so plan vì mất discipline.

### 4.4. Priority khi nhiều triggers match đồng thời

Nếu 1 mã có nhiều triggers match cùng lúc:
1. Hard trigger priority cao nhất → execute trước
2. Take profit trigger thứ hai → sau khi xử lý hard
3. Soft trigger cuối → chỉ flag, không execute trong cùng session

Nếu multiple positions có hard trigger cùng ngày (ví dụ regime crash):
- Execute theo priority: mã có largest position trước, sau đó theo loss magnitude
- Agent flag rằng đây có thể là systemic event → gợi ý re-check tier 0 regime

---

## 5. Phase 2 & Phase 3 entry management

### 5.1. Phase 2 — Bucket 2 confirm vào tranche 2

Điều kiện confirm từ tier 6 (Section 5.2):
- Vùng kỹ thuật khung tuần bật lên tích cực (zone A) trở lên (nội bộ `technical_zone.overall.w`)
- Điểm dòng tiền tuần dương 2-3 phiên liên tiếp (nội bộ `week_score > 0`)
- Điểm dòng tiền ngày dương trong 2/3 phiên gần nhất (nội bộ `day_score > 0`)
- Cường độ thanh khoản phiên bật ≥ 1.2× trung bình gần đây (nội bộ `volume_strength_index ≥ 1.2`)

**Agent action khi match:**

```
PHASE 2 CONFIRM AVAILABLE — [Ticker] [ngày]

Bucket 2 mã [X]: Phase 1 đã vào [X%], Phase 2 potential [Y%] còn lại

Condition check:
- Zone w: [C → B → A] ✓
- week_score: [−5 → 3 → 12 → 18] ✓
- day_score 3 phiên: [+, +, +] ✓
- VSI: [1.4] ✓

Proposal: vào tranche 2 — [Y%] portfolio
Current price: [X]k (so Base [Y]k → upside còn [Z%])

Limit orders (split 40/30/30):
- Order 1: current price
- Order 2: current × 0.98
- Order 3: current × 0.97

User confirm + adjust nếu cần?
```

### 5.2. Timeout 4 tuần Phase 2

Nếu quá 4 tuần sau Phase 1 entry mà Bucket 2 không confirm:

```
PHASE 2 TIMEOUT — [Ticker] [ngày]

Bucket 2 mã [X]: Phase 1 đã vào [X%] cách đây [4 tuần]
Phase 2 condition chưa match: [chi tiết condition chưa match]

Thesis pullback assessment:
- Nếu zone q+y vẫn mạnh (A/AA): thesis structural intact, pullback kéo dài hơn dự kiến → có thể extend 2 tuần nữa
- Nếu zone q bắt đầu suy yếu (A → B): thesis pullback có thể fail → đề xuất thoát 50-70% Phase 1 position
- Nếu zone q đã yếu rõ (B → C): thesis fail, đề xuất thoát 100% Phase 1

Proposal: [theo assessment]
User quyết định?
```

### 5.3. Bucket 3 upgrade → Bucket 2

Điều kiện từ tier 6 (Section 5.3):
- Zone tuần chuyển B/A (từ C) 2-3 phiên
- week_score chuyển dương
- day_score dương 3 phiên liên tiếp

**Agent action khi match:**

```
BUCKET 3 → BUCKET 2 UPGRADE — [Ticker] [ngày]

Mã [X] watchlist đã chuyển sang Bucket 2:
- Zone w: [C → B → A] ✓
- week_score: [−15 → −5 → 3] ✓
- day_score 3 phiên: [+, +, +] ✓

Size plan từ tier 6: [Y%] full target
Phase 1 vào tranche 1: [Y × 0.4]% portfolio (40% của target)

Action:
- Apply rule Bucket 2 Phase 1 — vào 30-50% first tranche
- Set Phase 2 confirm condition để monitor tiếp

User confirm vào vị thế + adjust nếu cần?
```

---

## 6. Memo refresh (30 ngày rule)

### 6.1. Kiểm tra 30 ngày

Monthly review (Section 3.3 step 3) check các memo đã quá 30 ngày kể từ viết.

### 6.2. Refresh workflow

Thay vì rewrite full memo, check 5 điểm:

1. **Consensus update:** sell-side target có đổi không? (web search)
2. **BCTC quý mới (nếu có):** revenue/margin/ROE vs Base assumption
3. **Catalyst timeline:** các catalyst memo đã đến/quá date chưa? Realize hay fail?
4. **Thesis gate 1 (variant perception):** còn intact không? Consensus đã catch up chưa?
5. **Thesis gate 2 (bear case):** bear arguments đã strengthen không?

**Nếu tất cả 5 điểm thesis intact** → extend memo expiry thêm 30 ngày, cập nhật ngày refresh.

**Nếu ≥ 2 điểm đã changed** → full rewrite memo. Possibly re-check gate 1/2. Nếu gate 1 no longer holds → đề xuất thoát position hoặc giảm size.

### 6.3. Exit levels update

Mỗi refresh, kiểm tra levels Base / Bull / Bear:
- Nếu price đã lên gần Base → adjust Base up nếu thesis still valid (hoặc bán partial theo plan take profit)
- Nếu Bear target cần tighten do thesis partial fail → update hard trigger level

Mọi thay đổi levels ghi trong audit log + update tier 5C memo file.

---

## 7. Portfolio-level actions

### 7.1. Concentration drift

Sau period thời gian, position % portfolio drift do P&L:
- Mã win lớn: % portfolio tăng, có thể vượt 10% cap
- Mã loss: % portfolio giảm

**Rule:** nếu 1 mã > 10% portfolio do drift (winning) → đề xuất trim 20-30% position về khoảng 7-8%.

Exception: nếu conviction đã upgrade (thesis better than initial) + liquidity cho phép → có thể giữ 10-12%, ghi audit log.

### 7.2. Regime change

Monthly review check regime từ tier 0. Nếu shift:

**Risk-on full → Risk-on selective:**
- Không cần action urgent
- Review mã nào có thesis yếu nhất, cân nhắc giảm size 20-30%
- Tăng cash buffer từ 10-20% lên 30-40%

**Risk-on → Defensive only:**
- Action urgent: giảm exposure aggressive
- Giữ chỉ mã High conviction, thesis defensive (tiêu dùng, dịch vụ thiết yếu)
- Thoát mã cyclical, rủi ro cao
- Cash buffer tăng lên 50-60%

**Risk-on → Đứng ngoài:**
- Action urgent ngay: thoát toàn bộ position
- Chờ regime ổn định trở lại trước khi mở position mới
- Bắt đầu cycle mới từ tier 0

### 7.3. Sector rotation

Nếu 1 ngành trong portfolio có multiple mã và ngành deteriorate:
- Industry rank drop from top 5 → bottom 10
- Industry week_score từ dương sang âm kéo dài
- Tin ngành tiêu cực cấu trúc (regulation, cost pressure)

→ Đề xuất giảm ngành exposure 30-50%, rebalance sang ngành khác từ tier 3 sát nút hoặc tăng cash.

---

## 8. End-of-cycle reconcile

Khi:
- Tất cả positions đã thoát (take profit hoặc stop loss)
- Hoặc đầu quarter mới
- Hoặc user muốn re-evaluate portfolio

→ Chuyển sang tier 0 cho cycle mới.

**Bước reconcile:**

1. **Lessons learned:**
   - Positions win vs loss — pattern gì?
   - Thesis win: variant perception đúng chỗ nào?
   - Thesis fail: gate 1/2 miss gì?
   - Adjust cycle sau cho sharper

2. **Portfolio P&L summary:**
   - Total return cycle
   - Win rate
   - Avg gain winners vs avg loss losers
   - Max drawdown trong cycle
   - Compare vs VN-Index benchmark

3. **Available cash + potential new cycle:**
   - Cash 100% (đã thoát hết) hoặc partial (còn mã watchlist Bucket 3)
   - Sẵn sàng cho cycle tier 0 mới

4. **Carryover:**
   - Mã Bucket 3 watchlist từ cycle trước → có thể giữ observe cycle mới
   - Mã sát nút tier 3 → reconsider cycle mới với data update

---

## 9. Workflow per review session

### 9.1. Daily quick scan (5-10 phút)

1. Query snapshot + news của tất cả positions
2. Check hard trigger match (price, KPI, news)
3. Check Phase 2/3 condition nếu có Bucket 2/3 pending
4. Output alert report ngắn

### 9.2. Weekly review (30-45 phút)

1. Position health per-stock (P&L, price vs levels)
2. Soft trigger check
3. Phase 2/3 management
4. Take profit partial nếu price đạt target
5. Output Weekly review session

### 9.3. Monthly review (1-2 giờ)

1. Portfolio-level metrics
2. Concentration drift check
3. Memo refresh cho memo quá 30 ngày
4. Tier 0 regime re-check
5. Missed opportunities learning
6. Output Monthly review session + rebalance proposal nếu cần

### 9.4. Quarterly review (2-3 giờ)

1. BCTC quý mới review per-stock
2. Thesis check vs assumption memo
3. Re-run tier 5A forensic partial
4. Valuation update (quick DCF)
5. Exit levels update
6. Output Quarterly review session + major rebalance nếu cần

---

## 10. Template Weekly Review Session

Tier 7 không có checkpoint cứng (xem `P_invest_memo_00` Phần 6 — giai đoạn 6 là hoạt động liên tục, không phải quyết định một lần). Template dưới đây là format **review session** dùng cho output weekly, không phải checkpoint stage gate như tier 0-5C.

```
# Weekly Review Session [ngày]

## 1. Summary
Positions active: [N]
Phase 2 pending: [M]
Phase 3 watchlist: [K]

Alert level: [Green / Yellow / Red]
- Hard triggers matched: [số]
- Soft triggers matched: [số]
- Take profit available: [số]
- Phase 2 confirm available: [số]
- Phase 3 upgrade available: [số]

## 2. Position health per-stock

| Ticker | Size % | Entry | Current | % vs entry | % vs Base | % vs Bear | Status |
|---|---|---|---|---|---|---|---|
| A1 | 2.9% | 71k | 76k | +7% | 89% | +31% | Normal |
| A2 | 2.5% | 73k | 80k | +10% | 94% | +38% | Near Base |
| B1 | 1.7% | 62k | 58k | -6% | 77% | +16% | Pullback |
| D3 | 0% | - | 28k | - | - | - | Watchlist |
| ... | ... | ... | ... | ... | ... | ... | ... |

## 3. Triggers matched this week

### Hard triggers: [số]
- [Mã X]: trigger [Y] matched on [date]. Proposal: BÁN NGAY. [User action needed]

### Soft triggers: [số]
- [Mã Y]: trigger [Z] matched. Proposal: giảm size 30% hoặc tighten stop. [User decision]

### Take profit: [số]
- [Mã X]: price đạt Base. Proposal: bán 50% position.

## 4. Phase 2 / Phase 3 review

### Phase 2 confirm available: [số]
- [Mã A]: zone w bật A + week_score +18. Vào tranche 2 [Y%] portfolio?

### Phase 2 timeout gần: [số]
- [Mã B]: đã [3.5 tuần] Phase 1, chưa confirm. Zone q vẫn A → extend 2 tuần hay abort?

### Phase 3 upgrade: [số]
- [Mã C]: Bucket 3 → Bucket 2. Vào Phase 1 [Y%]?

## 5. Portfolio summary

Total P&L week: [+/- X%]
Total P&L vs entry: [+/- Y%]
Cash current: [Z%]
Concentration: mã max % portfolio = [W%], ngành max = [V%] — [within cap / near cap / over cap]

## 6. Câu hỏi chờ user

Actions needed:
- (a) Confirm execute hard trigger [Mã X]?
- (b) Decide soft trigger [Mã Y] — chọn proposal (a/b/c/d)?
- (c) Confirm take profit 50% [Mã Z]?
- (d) Confirm Phase 2 tranche 2 [Mã A]?
- (e) Decide Phase 2 timeout [Mã B]?

Nếu không có action → "No action required this week, next review Monday tuần sau".
```

**Độ dài target:** 1-2 trang weekly, 3-4 trang monthly, 4-5 trang quarterly.

---

## 11. Failure mode

### 11.1. Delay hard trigger execution quá 1 phiên

User hoặc Agent muốn "đợi bounce" khi hard trigger matched. Kéo dài 3-5 phiên → loss tăng.

**Xử lý:** hard trigger priority cao nhất. Max 1 phiên delay với audit log. Sau 1 phiên, không hỏi user nữa — notification mạnh "HARD TRIGGER — recommend execute immediately". Nếu user vẫn từ chối → audit log rõ rằng user override discipline.

### 11.2. Refresh memo qua loa không thực sự check

Agent check memo quá 30 ngày, ghi "thesis intact" mà không verify 5 điểm (consensus, BCTC, catalyst, gate 1, gate 2).

**Xử lý:** refresh memo có checklist 5 điểm cụ thể trong monthly review. Mỗi điểm phải có note "đã check, data: [số liệu cụ thể]". Refresh qua loa flag lại để Agent redo.

### 11.3. Concentration drift không action

Mã win tăng từ 7% lên 13% portfolio, Agent không flag. Position quá concentrated, 1 rủi ro gây loss lớn.

**Xử lý:** monthly review Section 3.3 step 2 bắt buộc check drift. Mã > 10% → proposal trim 20-30%. Không skip dù conviction cao.

### 11.4. Phase 2 timeout không abort

Bucket 2 quá 4 tuần không confirm, Agent vẫn monitor và đề xuất chờ thêm.

**Xử lý:** timeout là rule, không flex > 2 tuần extension. Nếu sau 4 + 2 = 6 tuần vẫn không confirm → auto proposal abort 50-100% Phase 1 position tuỳ assessment zone q/y. User quyết định cuối nhưng Agent không khuyến khích chờ lâu hơn.

### 11.5. Nhiều triggers match cùng lúc xử lý sai priority

Mã có hard trigger (Bear × 0.9) và take profit trigger (Base) match trong cùng ngày (giá fluctuate lớn). Agent confuse.

**Xử lý:** priority rõ:
1. Hard trigger > take profit > soft trigger
2. Trong trường hợp giá oscillate qua cả Bear và Base trong 1 phiên → xem close price cuối phiên. Nếu close < Bear × 0.9 → hard trigger. Nếu close > Base → take profit.

Không panic-react mid-session.

### 11.6. Regime change nhưng portfolio không rebalance

Tier 0 regime shift từ Risk-on full → Defensive only. Agent không propose thoát aggressive positions. Portfolio tiếp tục risk-on profile trong defensive market.

**Xử lý:** monthly regime check nếu detect shift → mandatory rebalance proposal. User có thể delay 1-2 tuần để xem regime confirm không, nhưng > 2 tuần là discipline violation.

### 11.7. Quên track Phase 3 Bucket 3 watchlist

Mã Bucket 3 với 0% position, Agent không include trong daily/weekly check, miss trigger upgrade sang Bucket 2.

**Xử lý:** daily quick scan bắt buộc include watchlist. Mỗi watchlist mã có condition explicit để trigger. Nếu condition match → flag ngay, không để drift.

### 11.8. Take profit partial rồi "gỡ" quyết định

Bán 50% tại Base, sau đó mã tiếp tục tăng 10% nữa. User regret, bảo Agent "nếu tiếp tục tăng cỡ 20% nữa thì mua lại". Đây là chasing, violate discipline.

**Xử lý:** take profit đã thực hiện là decision final. Không mua lại cùng mã cùng cycle. Nếu muốn re-enter phải qua full tier 3-5C mới (mã như mới, thesis mới).

### 11.9. End-of-cycle không reconcile properly

Cycle kết thúc, user muốn enter mã mới ngay. Agent skip reconcile, không assess lessons learned → cycle sau repeat same mistakes.

**Xử lý:** end-of-cycle mandatory reconcile (Section 8). Ghi lessons learned vào file chung của project. Trước khi start cycle mới từ tier 0 phải đọc lessons.

---

## 12. Đầu ra chuẩn

### 12.1. File per session

Mỗi review session lưu file:
- Daily: `tier7_daily_<YYYYMMDD>.md` (ngắn, chỉ khi có alert)
- Weekly: `tier7_weekly_<YYYYMMDD>.md`
- Monthly: `tier7_monthly_<YYYYMM>.md`
- Quarterly: `tier7_quarterly_<YYYY_Q>.md`

### 12.2. Audit log

Tất cả action (execute trigger, user override, refresh memo, rebalance) ghi trong file:
- `audit_log_<ticker>_<YYYYMMDD>.md` per-stock
- `audit_log_portfolio_<YYYYMMDD>.md` portfolio-level

### 12.3. Lessons learned (end-of-cycle)

File tổng kết cycle:
- `lessons_learned_cycle_<X>_<YYYYMM>.md`
- Nội dung: win/loss pattern, thesis correct/fail analysis, process improvement cho cycle sau
- Đọc trước khi start tier 0 cycle mới

---

## 13. Cycle hoàn chỉnh — từ tier 0 đến tier 7

Sau tier 7, khi positions đã thoát (via take profit hoặc hard trigger) và user ready cho cycle mới:

1. Reconcile lessons (Section 8 + 12.3)
2. Read lessons learned file
3. Start tier 0 cycle mới với:
   - Regime fresh assessment
   - Catalyst scan mới
   - Cash 100% available (hoặc reserved cho mã Bucket 3 watchlist)

Toàn bộ quy trình 7 tier là 1 cycle đầu tư từ khi bắt đầu đánh giá thị trường đến khi thoát hết positions. Mỗi cycle typical 3-6 tháng tuỳ horizon + regime. User có thể chạy nhiều cycles đồng thời nếu muốn diversify thesis (ví dụ cycle A focus value, cycle B focus growth) nhưng cần tách portfolio allocation rõ ràng cho mỗi cycle.
