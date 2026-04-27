# O_invest_memo_05 — Monthly Review

Spec render báo cáo review portfolio-level hàng tháng. **Rigid structure** — focus sâu hơn weekly: performance metrics, concentration drift, memo expiry, regime re-check.

Reference: `O_invest_memo_00` (master rules), `P_invest_memo_09` tier 7 monthly workflow (mục 3.3 + failure modes + portfolio-level actions mục 7).

## 1. Input state & mapping

**State file chính:**

| State file | Role |
|---|---|
| `tier7_monthly_<YYYYMM>.md` | Monthly review output — portfolio metrics, drift check, memo refresh, regime re-check |

**State file tham khảo:**

- `tier6_portfolio_<YYYYMMDD>_confirmed.md` version mới nhất — allocation baseline để so drift
- `tier5C_<ticker>_*_confirmed.md` cho mỗi mã active — để check memo expiry (30 ngày rule)
- `tier0_<YYYYMMDD>_confirmed.md` đầu cycle — regime baseline để so shift
- 4 weekly review trong tháng (`tier7_weekly_*` hoặc đã render thành `O_invest_memo_04`) — để tổng hợp trigger đã matched trong tháng

## 2. Structure báo cáo đầu ra

**Độ dài target:** 3-5 trang MD.

**Rigid structure bắt buộc:**

```
# Monthly Review — <Tháng/Năm>

## 1. Executive summary
## 2. Portfolio performance
## 3. Concentration & drift
## 4. Memo refresh status
## 5. Regime re-check
## 6. Missed opportunities
## 7. Rebalance proposal (nếu cần)
## 8. Actions needed
— Metadata
```

## 3. Compose từng phần chi tiết

### Phần 1 — Executive summary

Format cứng:

```
**Tháng:** [MM/YYYY]  
**Ngày review:** [DD/MM/YYYY]  
**Số positions active:** [N] | **Số Bucket 3 watchlist:** [M]

**Performance tháng:**
- Return tháng này (MoM): [+/-X]%
- Return YTD: [+/-Y]%
- Win rate: [Z]% ([N win] / [M total closed])

**Regime check:** [giữ nguyên / shift từ X sang Y]  
**Concentration:** [OK / có drift cần rebalance]  
**Memo refresh needed:** [N] memo quá 30 ngày

**Hành động key cần user quyết (tóm tắt):**
- [Bullet 2-4 action quan trọng nhất trong tháng, link đến phần 8]
```

Độ dài 0.5-1 trang.

### Phần 2 — Portfolio performance

Structure sub-section:

```
### 2.1. Returns

| Metric | Tháng này | YTD | Cycle này |
|---|---|---|---|
| Portfolio total return | [+/-X]% | [+/-Y]% | [+/-Z]% |
| VN-Index benchmark return | [+/-X]% | [+/-Y]% | [+/-Z]% |
| Alpha (portfolio - benchmark) | [+/-X]% | [+/-Y]% | [+/-Z]% |

### 2.2. Win rate & magnitude

- Positions closed trong tháng: [N] (winner [X] / loser [Y])
- Avg gain của winner: [+X]%
- Avg loss của loser: [-Y]%
- Win/loss ratio (magnitude): [X / Y = Z:1]

### 2.3. Max drawdown (mức sụt giảm sâu nhất từ đỉnh đến đáy)

- Max drawdown tháng: [X]%
- Max drawdown cycle này: [Y]%
- Current drawdown (nếu đang trong drawdown): [Z]%

### 2.4. Sharpe-like (return / volatility) — chỉ khi có đủ chuỗi

[Chỉ render nếu cycle đã đủ 3+ tháng. Ít hơn không đủ sample]

- Annualized return: [X]%
- Annualized volatility: [Y]%
- Sharpe-like: [X/Y = Z]
```

**Chart đề xuất phần 2:**

Chart 1 — Equity curve portfolio vs VN-Index:

````
```chart
type: line
title: Equity Curve — Portfolio vs VN-Index
x_axis: [list ngày hoặc tuần trong tháng/cycle]
y_axis:
  - name: Portfolio
    data: [...]
    y_label: '%'
  - name: VN-Index
    data: [...]
source: Tổng hợp
render_in_md: skip
render_in_docx: true
render_in_pptx: true
note: Normalize 100 tại điểm bắt đầu cycle
```
````

Chart 2 — Contribution analysis (winner/loser breakdown):

````
```chart
type: bar
title: Contribution Analysis Tháng [MM/YYYY]
x_axis: [list ticker đã closed + top 5 position active theo contribution]
y_axis:
  - name: Contribution to portfolio return (%)
    data: [+/- values]
y_label: '%'
source: Tổng hợp
render_in_md: skip
render_in_docx: true
render_in_pptx: true
note: Xanh = đóng góp dương, đỏ = đóng góp âm
```
````

### Phần 3 — Concentration & drift

Structure:

```
### 3.1. Concentration hiện tại vs plan

| Dimension | Rule cap | Baseline plan | Actual | Drift | Status |
|---|---|---|---|---|---|
| Max mã | 10% | [X]% (ban đầu) | [Y]% (hiện tại) | [+/-Z]% | [OK/Near cap/Over cap] |
| Max ngành | [X]% theo regime | [Y]% | [Z]% | [+/-W]% | [OK/Near/Over] |
| Cash buffer | [X]% min | [Y]% (plan) | [Z]% (actual) | [+/-W]% | [OK/Low/High] |
| Catalyst play | 15% max | [Y]% | [Z]% | [+/-W]% | [OK/Near/Over] |

### 3.2. Drift breakdown per stock

[Bảng mã có % portfolio khác baseline > 2%]

| Ticker | Baseline % (plan) | Current % | Drift | Lý do drift |
|---|---|---|---|---|
| [X] | 2.9% | 4.1% | +1.2% | Winner +42% từ entry, tỷ trọng tự tăng |
| [Y] | 1.7% | 1.0% | -0.7% | Loser -30% + đã trim 30% position |
| ... | ... | ... | ... | ... |

### 3.3. Rebalance recommendation (nếu có drift over cap)

[1 đoạn: mã nào cần trim bao nhiêu về mức target, mã nào cần tăng, hoặc không cần]

- **[Ticker X]** đã vượt cap 10% (current 13%), **proposal trim 30% position** về 9-10%.
- **[Ticker Y]** giảm còn 1%, thesis vẫn intact, **không trim thêm** — chờ Phase 2 confirm hoặc thoát nếu thesis fail.
```

**Chart đề xuất phần 3:**

Chart 3 — Concentration pie current vs plan (stacked bar so sánh hoặc 2 pie):

````
```chart
type: stacked_bar
title: Concentration Plan vs Actual — [MM/YYYY]
x_axis: [Plan, Actual]
y_axis:
  - name: [Mã 1]
    data: [plan%, actual%]
  - name: [Mã 2]
    data: [plan%, actual%]
  ...
  - name: Cash
    data: [plan%, actual%]
source: Tổng hợp
render_in_md: skip
render_in_docx: true
render_in_pptx: true
```
````

### Phần 4 — Memo refresh status

Memo 5C có expiry 30 ngày (rule `P_invest_memo_09` mục 6).

Structure:

```
| Ticker | Memo viết ngày | Tuổi memo (ngày) | Status | Action |
|---|---|---|---|---|
| [X] | DD/MM/YYYY | 15 | Fresh | Giữ nguyên |
| [Y] | DD/MM/YYYY | 28 | Cần refresh sớm | Refresh trong tuần tới |
| [Z] | DD/MM/YYYY | 35 | Expired | **Refresh ngay** — có thể có BCTC quý mới |
```

**Nếu có memo Expired:**

```
### 4.1. Memo cần refresh — [N]

Cho mỗi memo expired, chạy checklist 5 điểm (theo `P_invest_memo_09` mục 6.2):

**[Ticker X]** — memo viết [DD/MM], đã [N] ngày:
- Consensus update: [sell-side target có đổi không]
- BCTC quý mới: [nếu đã công bố, so số thực vs Base case assumption]
- Catalyst timeline: [catalyst nào đã đến date, realize hay fail]
- Gate 1 variant perception: [còn intact không]
- Gate 2 bear case: [bear arguments đã strengthen không]

**Kết luận refresh:**
- [Thesis intact → extend 30 ngày]
- [2+ điểm changed → full rewrite memo, có thể re-check gate]
- [Gate 1 không còn hold → đề xuất thoát position hoặc giảm size]
```

### Phần 5 — Regime re-check

```
### 5.1. Regime baseline (cycle start)

[Regime confirmed đầu cycle từ `tier0_*_confirmed.md`]: [Risk-on full / Risk-on selective / Defensive only / Đứng ngoài]

### 5.2. Regime hiện tại

[Re-assess với data current: VN-Index trend, breadth, vĩ mô Fed/DXY/commodity, flow khối ngoại]

**Kết luận:** [Giữ nguyên regime / Shift từ X sang Y]

### 5.3. Action nếu regime shift

[Nếu shift, apply rule `P_invest_memo_09` mục 7.2]

- **Risk-on full → Risk-on selective:** không urgent, review mã thesis yếu, giảm size 20-30%, tăng cash từ 10-20% lên 30-40%
- **Risk-on → Defensive only:** action urgent, giảm exposure aggressive, giữ mã High conviction defensive, thoát cyclical rủi ro cao, cash lên 50-60%
- **Risk-on → Đứng ngoài:** action urgent ngay, thoát toàn bộ position, start cycle mới từ tier 0
```

### Phần 6 — Missed opportunities

Structure:

```
### 6.1. Mã tier 3 sát nút (không vào shortlist đầu cycle)

| Ticker | Điểm tier 3 | Lý do không vào | Biến động tháng này | Learning |
|---|---|---|---|---|
| [X] | 12/18 | Bucket 3 watchlist sâu | +25% (outperform shortlist) | Có thể conviction scoring miss yếu tố Y |
| [Y] | 10/18 | Low conviction, không vào | -15% | Low conviction loại đúng |
| ... | ... | ... | ... | ... |

### 6.2. Ngành không vào tier 1 (có catalyst mới không ghi nhận)

[Bullet ngành có biến động catalyst trong tháng mà tier 1 loại]
- **Ngành [X]**: catalyst [Y] xuất hiện tháng này, ngành tăng [Z]%. Lý do tier 1 loại: [...]. Learning: [watch indicator nào cycle sau]
```

**Nếu không có missed opportunity đáng kể:** 1 câu "Không có mã/ngành nào missed đáng kể tháng này."

### Phần 7 — Rebalance proposal (nếu cần)

Chỉ render phần này nếu có rebalance action. Lý do trigger rebalance:
- Drift over cap (từ phần 3)
- Memo expired với thesis changed (từ phần 4)
- Regime shift (từ phần 5)
- Missed opportunity đáng kể có thể swap in (từ phần 6)

Structure:

```
**Rebalance summary:**

| Action | Ticker | Current % | Target % | Rationale |
|---|---|---|---|---|
| Trim | [X] | 13% | 9% | Drift over cap |
| Exit | [Y] | 2% | 0% | Thesis fail sau refresh memo |
| Increase | [Z] | 1% | 3% | Regime vẫn risk-on, thesis strengthen |
| Swap in | [W] | 0% | 2% | Missed opportunity, catalyst mới confirmed |

**Order list rebalance:**

[Block text format như `O_invest_memo_03` phần 7 — chỉ orders delta]

```text
REBALANCE ORDERS — [DD/MM/YYYY]

Trim:
  [X] | SELL | MARKET | qty [X]cp | from 13% → 9%

Exit:
  [Y] | SELL | MARKET | qty [Y]cp | position close

Increase / Swap in:
  [Z] | BUY | LIMIT [X]k | qty [Y]cp | to 3%
  [W] | BUY | LIMIT [X]k | qty [Y]cp | new position 2%
```

**Impact:**
- Cash change: [+/-X]% → new buffer [Y]%
- Concentration sau rebalance: mã max [X]%, ngành max [Y]%
```

Nếu không rebalance: render 1 dòng "Không cần rebalance tháng này, giữ portfolio hiện tại."

### Phần 8 — Actions needed

Format checkbox như weekly:

```
**Actions cần user quyết:**

- [ ] (a) Approve rebalance proposal? (nếu có phần 7)
- [ ] (b) Refresh memo [Ticker X/Y/Z] expired?
- [ ] (c) Quyết regime shift action nếu có (giảm exposure / thoát toàn bộ)?
- [ ] (d) Trim [Ticker W] over cap từ 13% xuống 9%?
- [ ] (e) Review missed opportunity [Ticker V] — có cần thêm vào watchlist cycle này không?
```

### Metadata

```
---

**Metadata**

- **Tháng:** [MM/YYYY]
- **Ngày review:** [DD/MM/YYYY]
- **Portfolio size current:** [X] tỷ VND
- **Regime status:** [giữ / shift]
- **State file nguồn:** `tier7_monthly_<YYYYMM>.md`
- **Portfolio plan version:** [X.Y] — link `tier6_portfolio_*_confirmed.md`
- **Next review:** đầu tháng [MM+1/YYYY]
```

## 4. Compose workflow step-by-step

**Bước 1 — Format + template.**

**Bước 2 — Load state:** tier 7 monthly + portfolio baseline + memo list + tier 0 regime baseline + 4 weekly gần nhất.

**Bước 3 — Compute metrics:**
- Return tháng, YTD, cycle
- Win rate, avg gain/loss
- Max drawdown
- Sharpe-like (nếu đủ chuỗi)

**Bước 4 — Concentration drift check:** so actual vs baseline plan, flag over cap.

**Bước 5 — Memo expiry scan:** duyệt list tier 5C memo, tính tuổi từng memo, flag expired.

**Bước 6 — Regime re-check:** query agent_db current regime indicator + so với baseline cycle.

**Bước 7 — Missed opportunities scan:** load mã tier 3 sát nút và ngành không vào tier 1, check biến động tháng.

**Bước 8 — Quyết định rebalance:** tổng hợp drift + memo + regime + opportunity → rebalance proposal nếu cần.

**Bước 9 — Compose executive summary phần 1** làm hook tổng.

**Bước 10 — Compose phần 2-8 theo thứ tự.**

**Bước 11 — Chart annotation:** equity curve + contribution + concentration stacked bar.

**Bước 12 — Self-check:**
- Return metric tháng/YTD/cycle khớp nhau logic (không âm/dương ngược)
- Drift values đúng (actual - baseline)
- Memo expiry tuổi tính đúng từ ngày viết
- Rebalance order list (nếu có) khớp với proposal table
- Thuật ngữ, citation, format theo master

**Bước 13 — Render + present.**

## 5. Guide render docx [LEGACY]

> Render binary out of scope. Section giữ làm reference cho tool render bên ngoài.

Monthly docx là **formal deliverable** quan trọng — user thường share sếp/committee.

**Layout:**
- Cover (trang 1): "Monthly Review — [MM/YYYY]" + logo + portfolio size + regime badge
- Executive summary (trang 2) — phần 1 đầy đủ, bảng key metric
- Performance (trang 3-4): bảng returns + equity curve chart + contribution chart
- Concentration & drift (trang 5): bảng + stacked bar chart
- Memo refresh (trang 6): bảng status + chi tiết expired
- Regime + Missed opportunities (trang 7)
- Rebalance proposal (trang 8, nếu có): bảng + order list monospace
- Actions needed + Metadata (trang 9)

**Template cần có:** đầy đủ heading + table + chart placeholder + monospace + footer.

## 6. Guide render pptx [LEGACY]

> Render binary out of scope. Section giữ làm reference cho tool render bên ngoài.

Monthly pptx dùng cho **review meeting** với sếp/portfolio committee.

**Layout 12-15 slide:**

| Slide | Nội dung |
|---|---|
| 1 | Cover — Monthly Review [MM/YYYY] |
| 2 | Executive summary — key metrics 1 slide |
| 3 | Performance — equity curve chart |
| 4 | Performance — contribution chart |
| 5 | Returns table (nếu cần detail) |
| 6 | Win rate + drawdown breakdown |
| 7 | Concentration current vs plan chart |
| 8 | Drift table detail |
| 9 | Memo refresh status |
| 10 | Regime re-check + action if shift |
| 11 | Missed opportunities |
| 12-13 | Rebalance proposal (nếu có) |
| 14 | Actions needed |
| 15 | Next month focus |

## 7. Ví dụ fragment

```markdown
# Monthly Review — 04/2026

## 1. Executive summary

**Tháng:** 04/2026  
**Ngày review:** 01/05/2026  
**Số positions active:** 7 | **Số Bucket 3 watchlist:** 2

**Performance tháng:**
- Return tháng này (MoM): +3.8%
- Return YTD: +12.4%
- Win rate: 71% (5 win / 7 closed)

**Regime check:** giữ nguyên Risk-on selective (ưa rủi ro chọn lọc)  
**Concentration:** VNM 11.2% đã vượt cap 10%, cần trim  
**Memo refresh needed:** 2 memo quá 30 ngày (VNM, VCB)

**Hành động key cần user quyết:**
- Trim VNM 11.2% → 9%
- Refresh memo VNM và VCB trong tuần đầu tháng 5
- Approve rebalance proposal phần 7
```

## 8. Checklist self-check cuối cùng

- [ ] State `tier7_monthly_*` đã có
- [ ] 8 section đầy đủ kể cả rỗng (với proposal "không cần" nếu không có action)
- [ ] Performance metrics tính đúng: MoM, YTD, cycle return
- [ ] Win rate = # winners / # closed (không phải / # total active)
- [ ] Max drawdown tính từ peak-to-trough trong period
- [ ] Concentration drift = actual - baseline (dấu đúng)
- [ ] Memo expiry tuổi tính từ ngày viết memo (trong metadata tier 5C)
- [ ] Regime check có so với baseline + action nếu shift
- [ ] Missed opportunity có learning concrete không chỉ list
- [ ] Rebalance proposal (nếu có): action table khớp với order block
- [ ] Order list quantity round lot 100 cp
- [ ] Equity curve chart normalize 100 tại start cycle
- [ ] Thuật ngữ Regime/Bucket/Phase/Conviction có dịch lần đầu
- [ ] Citation: "(nguồn: Tổng hợp)" cho agent_db, link finext cho tin, markdown link cho web external
- [ ] Metadata đầy đủ với portfolio plan version link
- [ ] Format cuối đúng yêu cầu + template đúng
