# P_invest_memo_08 — Tier 6: Portfolio Construction

Giai đoạn 6 của quy trình. Từ các mã đã pass memo tier 5C, tính toán size allocation, constraint checks, sequence entry, và diversification. Đây là bước chuyển từ **phân tích per-stock** sang **portfolio-level decisions** — balance conviction với risk management.

Reference: `P_invest_memo_00` phần Flow chi tiết (overview), `P_invest_memo_00` phần Cơ chế checkpoint review, `P_invest_memo_00` phần Sáu nguyên tắc Agent bất biến (đặc biệt Nguyên tắc 3 size ≤ 5% ADV), `P_invest_memo_01` (regime + cash buffer), `P_invest_memo_04` (conviction tier), `P_invest_memo_07` (tier 5C memo + exit triggers).

---

## 1. Mục tiêu & output expected

**Mục tiêu:** chuyển shortlist các mã đã pass memo thành **portfolio cụ thể** với:
- Size % portfolio cho từng mã
- Sequence entry (mã nào vào trước, mã nào chờ)
- Cash buffer phù hợp với regime
- Diversification constraint (max per stock, per industry)
- Entry orders cụ thể để user đặt lệnh

**Input:**
1. Output tier 5C — các memo đã pass gate: `tier5C_<ticker>_YYYYMMDD_confirmed.md`
2. Tier 3 output — conviction tier, bucket, điểm 6 tiêu chí
3. Tier 0 output — regime hiện tại (Risk-on full / Risk-on selective / Defensive only / Đứng ngoài) + cash buffer target
4. Tier 5B output — target giá + entry levels cho mỗi mã
5. Size portfolio USD / VND hiện tại từ user (tổng vốn khả dụng)
6. Positions hiện tại (nếu đang có mã từ cycle trước) để tránh concentration

**Output chính:**
1. **Bảng size allocation** cho mỗi mã: % portfolio, VND absolute, số cổ phiếu
2. **Sequence entry plan** — phase 1 (vào ngay), phase 2 (chờ confirm), phase 3 (watchlist)
3. **Cash buffer plan** — theo regime + reserved cho Bucket 2/3 future entries
4. **Constraint check report** — ADV, diversification, concentration
5. **Order list** cụ thể cho user đặt lệnh (limit price range, ngày phân bổ)
6. **Report Checkpoint 6** để user approve trước khi đặt lệnh

**Tần suất:** sau khi toàn bộ memo tier 5C được confirm.

**Thời gian session:** 45-60 phút Agent work (tính toán + constraint check + sequence). User review checkpoint 30-45 phút (có thể adjust size per stock).

---

## 2. Triết lý

**Portfolio construction balance conviction với risk management.** Conviction tier từ tier 3 (High/Medium/Low) cho base size, nhưng constraint thực tế (liquidity, regime, diversification) có thể **cap size thấp hơn** conviction. Ngược lại, regime không cho phép tăng size trên base conviction — Agent không "ép" portfolio.

**Size decisions phải có lý do rõ ràng.** Không dùng "60%", "5%" random — mỗi con số phải trace được về:
- Conviction tier × Regime adjustment × Bucket multiplier × ADV cap

**Cash buffer là vị thế, không phải "tiền thừa".** Cash để:
- Ready cho Bucket 2 future tranches (sau pullback confirm)
- Opportunity cho mã tier 3 sát nút hoặc mới emerge
- Defense nếu regime xấu đi

**Rebalance chậm hơn quỹ institutional.** User là individual investor — rebalance không cần real-time. Sequence entry phân 2-4 phiên là bình thường, tránh market impact và tránh emotion vào lệnh lớn trong 1 ngày.

**Agent không tự đặt lệnh.** Output là **order list** với limit price range + ngày phân bổ. User đặt lệnh thực tế qua broker.

---

## 3. Sizing per stock — formula

### Công thức tổng quát

```
Target size = Base size (conviction) × Regime adjustment × Bucket multiplier × min(1, ADV cap / Base size)
```

4 thành phần chi tiết:

### 3.1. Base size (theo conviction tier)

Từ tier 3:

| Tier conviction | Điểm tổng | Base size (% portfolio) |
|---|---|---|
| High | 15-18 | 6-8% |
| Medium | 11-14 | 3-5% |
| Low | 8-10 | 1-2% |

**Chọn cụ thể trong range:**
- High 15-16đ: 6%, 17đ: 7%, 18đ: 8%
- Medium 11-12đ: 3%, 13đ: 4%, 14đ: 5%
- Low 8đ: 1%, 9-10đ: 1.5-2%

### 3.2. Regime adjustment (từ tier 0)

| Regime | Adjustment multiplier | Cash buffer target |
|---|---|---|
| Risk-on full | 1.0x (full base size) | 10-20% |
| Risk-on selective | 0.7x (giảm 30%) | 30-40% |
| Defensive only | 0.5x (giảm 50%, chỉ cho mã defensive type) | 50-60% |
| Đứng ngoài | 0x (không vào position mới) | 100% cash |

**Ví dụ:** High conviction 7% base × Risk-on selective 0.7x = **4.9% target** (≈ 5%)

### 3.3. Bucket multiplier (sequence entry)

Cho **immediate entry** (phase 1):

| Bucket | Vào ngay | Chờ confirm (phase 2) |
|---|---|---|
| 1 — Zone w+m đồng thuận A+ | 50-70% của target size | 30-50% còn lại tuỳ price action |
| 2 — Pullback trong uptrend quý/năm | 30-50% của target size | 30-50% sau khi confirm (zone w bật A + day_score dương 2-3 phiên) |
| 3 — Watchlist (zone w+m đều C) | 0% | Chuyển sang Bucket 2 khi zone w bật B/A, rồi apply rule Bucket 2 |

**Ví dụ tiếp:** mã High 5% target × Bucket 1 → vào ngay 2.5-3.5% portfolio phase 1.

### 3.4. ADV cap (Nguyên tắc 3 của `P_invest_memo_00`)

**Constraint:** size position ≤ 5% × trading_value trung bình 20 phiên × số phiên dự kiến accumulate.

Công thức:

```
Max size absolute = 5% × ADV_20_session × N_session
```

Trong đó N_session = 2-4 phiên để build full position (không ép vào 1 phiên).

**Ví dụ:**
- Mã A trading value TB 50 tỷ/phiên
- 5% ADV = 2.5 tỷ/phiên
- Build trong 3 phiên → max 7.5 tỷ position

Nếu portfolio user 1M USD = ~23 tỷ VND:
- Max size % portfolio = 7.5 tỷ / 23 tỷ = **32% portfolio** — không phải constraint cho hầu hết mã LargeCap

Nhưng với mã Small trading value 5 tỷ/phiên:
- 5% ADV = 250 triệu/phiên × 3 phiên = 750 triệu max
- % portfolio = 750M / 23B = **3.3%**

Nếu conviction High muốn 7% mà ADV cap chỉ 3.3% → **size bị cap xuống 3.3%**, flag rõ cho user biết "conviction cao nhưng liquidity hạn chế size thực tế".

### 3.5. Công thức tổng hợp

```python
def target_size(conviction_tier, regime, bucket, ADV_20):
    # 1. Base size từ conviction
    base = {'High': 0.07, 'Medium': 0.04, 'Low': 0.015}[conviction_tier]
    
    # 2. Regime adjustment
    regime_mult = {'Risk-on full': 1.0, 'Risk-on selective': 0.7, 
                   'Defensive only': 0.5, 'Đứng ngoài': 0}[regime]
    
    # 3. Bucket multiplier (immediate entry)
    bucket_mult = {1: 0.6, 2: 0.4, 3: 0}[bucket]  # trung bình
    
    # 4. ADV cap
    max_abs = 0.05 * ADV_20 * 3  # 3 phiên accumulate
    adv_cap_pct = max_abs / portfolio_size
    
    # Target size immediate
    target_pct = base * regime_mult * bucket_mult
    # Nhưng không vượt ADV cap
    target_final = min(target_pct, adv_cap_pct * bucket_mult)
    
    return target_final
```

Agent áp dụng cho từng mã, build bảng allocation.

---

## 4. Constraint checks

Sau khi tính target size per stock, kiểm tra 4 constraints:

### 4.1. ADV constraint (Nguyên tắc 3 của `P_invest_memo_00`)

Đã check trong công thức 3.5. Flag các mã bị cap:
- "Mã X: conviction High 7%, nhưng ADV cap → thực tế 3.3%"
- User biết trade-off conviction vs liquidity

### 4.2. Max per stock

**Rule:** không mã nào vượt **10% portfolio** dù conviction rất cao.

Lý do: concentration risk quá lớn — 1 mã sai thesis gây loss > 10% portfolio hiếm recover được trong 1-6 tháng.

Nếu High conviction đặc biệt muốn 10%: OK nhưng phải có exception reasoning (thesis rất chắc, liquidity cao, exit trigger rất rõ) + audit log.

### 4.3. Max per industry

**Rule:** không ngành nào vượt **30-40% portfolio**.

Ngưỡng cụ thể theo regime:
- Risk-on full: max 40% per industry
- Risk-on selective: max 30%
- Defensive only: max 25%

Lý do: ngành risk chung (regulatory, commodity price, cycle). 3 mã × 10% = 30% trong 1 ngành là ceiling.

Exception: ngành đa dạng sub-segment (ví dụ Bán lẻ: retail ICT + FMCG + nhu yếu phẩm) có thể nâng lên 35-40% vì sub-exposure khác nhau.

### 4.4. Min cash buffer (theo regime)

**Rule:** cash buffer sau phase 1 entry phải ≥ target regime:

| Regime | Min cash buffer sau phase 1 |
|---|---|
| Risk-on full | 10% |
| Risk-on selective | 30% |
| Defensive only | 50% |
| Đứng ngoài | 100% |

Nếu sau phase 1 deployment vượt buffer floor → giảm size hoặc trễ entry 1 số mã low conviction.

Cash buffer **không phải "tiền chết"** — để:
- Reserve cho Bucket 2/3 future tranches
- Opportunity new mã emerge từ tier 3 sát nút
- Defense nếu regime xấu đi, catalyst tiêu cực xuất hiện

### 4.5. Existing positions reconcile (nếu có)

Nếu user đã có positions từ cycle trước:
- Count existing position vào % portfolio hiện tại
- Check có mã nào overlap với shortlist mới không
- Adjust: nếu mã overlap, tăng size lên target mới thay vì mở position mới
- Check tổng portfolio concentration sau khi thêm positions mới có vi phạm 4.2, 4.3 không

---

## 5. Sequence entry — thứ tự vào lệnh

Phase 1 (immediate) và Phase 2 (chờ confirm) phải có priority rõ.

### 5.1. Priority order trong Phase 1

Từ cao xuống thấp:

1. **Bucket 1 High conviction** — vào 50-70% target, ngày T+0 (ngay phiên sau khi confirm checkpoint 6)
2. **Bucket 2 High conviction** — vào 30-50% first tranche, ngày T+0 hoặc T+1
3. **Bucket 1 Medium conviction** — vào 50-70% target, ngày T+1 hoặc T+2
4. **Bucket 2 Medium conviction** — vào 30-50% first tranche, ngày T+2
5. **Bucket 1 Low conviction** — vào 50-70% target, ngày T+2 hoặc T+3
6. **Bucket 2 Low conviction** — vào 30-50% first tranche, ngày T+3
7. **Bucket 3 (all tiers)** — watchlist, chưa vào

### 5.2. Phase 2 — chờ confirm Bucket 2

Điều kiện confirm cho Bucket 2 để vào thêm 30-50% còn lại:
- Zone tuần (technical_zone.overall.w) bật lên A trở lên (từ B/C)
- week_score dương 2-3 phiên liên tiếp
- day_score dương trong 2/3 phiên gần nhất
- Cường độ thanh khoản phiên bật ≥ 1.2× trung bình 5 phiên gần đây (volume xác nhận; nội bộ là `volume_strength_index ≥ 1.2`)

Agent monitor đều các mã Bucket 2 trong monitoring phase (tier 7). Khi có tín hiệu confirm → notify user + đề xuất vào thêm.

**Timeout:** sau **4 tuần** kể từ Phase 1 entry, nếu Bucket 2 vẫn không confirm:
- Thesis pullback đã fail
- Không vào tiếp — giữ phần đã vào hoặc thoát tùy price action
- Chi tiết rule trong tier 7 (`P_invest_memo_09` monitoring/exit)

### 5.3. Phase 3 — Bucket 3 watchlist

Điều kiện để Bucket 3 chuyển sang Bucket 2 (bắt đầu vào position):
- Zone tuần chuyển B hoặc A (từ C) trong 2-3 phiên liên tiếp
- week_score chuyển từ âm sang dương
- day_score dương 3 phiên liên tiếp

Khi chuyển Bucket 3 → Bucket 2, apply rule Phase 2 tiếp theo.

### 5.4. Phân bổ theo ngày thực tế

Không ép hết trong T+0 vì:
- Market impact (áp lực giá từ chính lệnh mua/bán của user khi size vượt thanh khoản phiên) — Nguyên tắc 3 ADV
- Emotion bias — vào lớn trong 1 ngày dễ tạo áp lực mental, panic ngay khi mã giảm 2-3%
- Price discovery tốt hơn khi entry phân 2-4 phiên

**Example kế hoạch 5 ngày:**

```
T+0: Bucket 1 High + Bucket 2 High first tranche (40% cash deploy)
T+1: Bucket 1 Medium + Bucket 2 Medium first tranche (20% cash deploy)
T+2: Bucket 1 Low + bổ sung Bucket 1 High nếu price pullback đẹp (10%)
T+3: Bucket 2 Low first tranche (5%)
T+4: Review, adjust limit orders nếu chưa fill
```

---

## 6. Entry limit orders

Mỗi mã có **limit price range** thay vì market order.

### 6.1. Rule entry price

Từ tier 5B, target Base đã có. Entry rule:
- **Limit max:** Base × 0.85 (margin of safety 15%)
- **Limit min:** Bear × 1.1 (10% trên Bear target — nếu rơi xuống đây thì thesis có thể đang fail)

Nếu current price > Limit max → **chờ pullback**, không vào
Nếu current price < Limit min → **thesis fail**, rà soát lại memo

### 6.2. Splitting limit orders

Cho mã mà current price trong range [Limit min, Limit max]:
- Limit order 1 (40% size): current price
- Limit order 2 (30% size): current × 0.98 (chờ pullback 2%)
- Limit order 3 (30% size): current × 0.96 (chờ pullback 4%)

Orders 2-3 nếu không fill trong 3-5 phiên → cancel + reassess.

### 6.3. Bucket 2 split khác

Bucket 2 mã đang pullback → limit orders tập trung vùng giá thấp hơn:
- Limit 1 (50% first tranche): current price hoặc current × 0.99
- Limit 2 (50% first tranche): current × 0.97-0.98

Chờ confirm xong (Phase 2) mới vào tranche 2 (50% còn lại).

---

## 7. Diversification rules tổng hợp

Sau khi tính allocation + constraint check, portfolio phải đạt:

### 7.1. Across stocks

- Max 1 mã: 10% (hard cap)
- Median position: 3-5%
- Min 5-8 positions total (diversify minimum)
- Max 12-15 positions total (tránh fragmentation, không focus đủ)

### 7.2. Across industries

- Max 1 ngành: 30-40% tuỳ regime (Section 4.3)
- Min 3 ngành trong portfolio (diversification)
- Ngành defensive (tiêu dùng, dịch vụ thiết yếu) nên có ≥ 1 mã trong mọi regime

### 7.3. Across marketcap

Không rule cứng, nhưng guideline:
- Nếu portfolio > 70% LargeCaps: trade-off liquidity tốt nhưng alpha thấp (LargeCap efficient hơn)
- Nếu portfolio > 50% SmallCaps: alpha tiềm năng cao nhưng liquidity risk — flag cho user aware
- Mix 40-50% LargeCap + 30-40% MidCap + 10-20% SmallCap là balanced cho portfolio < 1M USD

### 7.4. Catalyst play exposure

Các mã catalyst play (qua đường C tier 2, fail B nhưng có catalyst mạnh):
- Tổng ≤ 15% portfolio trong mọi regime
- Max 2-3 mã catalyst play
- Rule: catalyst play rủi ro cao hơn fundamental pass B, nên cap exposure riêng

**Lưu ý:** Đây là **convention nội bộ** (xem master `P_invest_memo_00` mục 5 Lưu ý), không phải nguyên tắc bất biến — user có thể override với audit log nêu lý do (vd nhiều catalyst đặc biệt cùng cycle).

---

## 8. Workflow 8 bước

**Bước 1 — Load input**

- Output tier 5C của tất cả memo passed gate
- Regime hiện tại từ tier 0
- Portfolio size USD/VND từ user
- Positions hiện tại (nếu có)

**Bước 2 — Tính target size per stock**

Công thức Section 3.5 — base × regime × bucket × ADV cap.

Build bảng đầy đủ với 4 cột: Base size, Regime-adjusted, Bucket-adjusted (phase 1), ADV-capped final.

**Bước 3 — Check 4 constraints**

- ADV per stock
- Max per stock (10%)
- Max per industry (30-40%)
- Min cash buffer per regime

Flag mã vi phạm + đề xuất adjustment.

**Bước 4 — Reconcile với existing positions**

Nếu user có positions từ cycle trước, check overlap + adjust size để tránh double-count.

**Bước 5 — Sequence entry plan**

Priority Phase 1 + Phase 2 + Phase 3 (Section 5).

Phân ngày T+0 đến T+4 cho phase 1.

**Bước 6 — Set limit orders**

Cho mỗi mã, tính 2-3 limit orders với split ratio (Section 6).

Verify: limit max < current price < limit min → OK vào; nếu không, flag mã cần chờ pullback.

**Bước 7 — Diversification check**

Portfolio tổng hợp sau allocation có đạt rules Section 7 không? Nếu không → adjust.

**Bước 8 — Xuất Checkpoint 6**

Theo template Section 10.

---

## 9. Ví dụ generic — portfolio construction

Case: Portfolio user 1M USD = 23 tỷ VND. Regime: Risk-on selective. 9 mã passed memo tier 5C.

### Input distribution

| Ticker | Ngành | Conviction | Điểm | Bucket | ADV (tỷ/phiên) |
|---|---|---|---|---|---|
| A1 | Ngành A | High | 16 | 1 | 120 |
| A2 | Ngành A | High | 15 | 1 | 80 |
| A4 | Ngành A | Medium | 13 | 2 | 45 |
| B1 | Ngành B | High | 15 | 2 | 60 |
| B2 | Ngành B | Medium | 13 | 2 | 40 |
| B3 | Ngành B | Medium | 12 | 1 | 25 |
| D1 | Ngành D | High | 15 | 2 | 55 |
| D2 | Ngành D | Medium | 13 | 1 | 30 |
| D3 | Ngành D | Low | 10 | 3 | 8 |

### Bước 2 — Tính target size

Regime Risk-on selective → multiplier 0.7x. Portfolio 23 tỷ.

| Ticker | Base % | × Regime | × Bucket phase 1 | ADV cap | Final Phase 1 % |
|---|---|---|---|---|---|
| A1 (High 16) | 7% | 4.9% | × 0.6 = 2.9% | 7.8%* | 2.9% |
| A2 (High 15) | 6% | 4.2% | × 0.6 = 2.5% | 5.2%* | 2.5% |
| A4 (Med 13) | 4% | 2.8% | × 0.4 = 1.1% | 2.9% | 1.1% |
| B1 (High 15) | 6% | 4.2% | × 0.4 = 1.7% | 3.9% | 1.7% |
| B2 (Med 13) | 4% | 2.8% | × 0.4 = 1.1% | 2.6% | 1.1% |
| B3 (Med 12) | 3% | 2.1% | × 0.6 = 1.3% | 1.6% | 1.3% |
| D1 (High 15) | 6% | 4.2% | × 0.4 = 1.7% | 3.6% | 1.7% |
| D2 (Med 13) | 4% | 2.8% | × 0.6 = 1.7% | 1.95% | 1.7% |
| D3 (Low 10) | 1.5% | 1.05% | × 0 (Bucket 3) = 0% | 0.5% | 0% (chờ) |

(* = ADV cap không binding, vì ADV lớn)

Tổng Phase 1: 14.0% portfolio deployed.

### Bước 3 — Constraint check

- **Max per stock:** không mã nào > 3% Phase 1 ✓
- **Max per industry:**
  - Ngành A: 2.9 + 2.5 + 1.1 = 6.5% ✓ (< 30%)
  - Ngành B: 1.7 + 1.1 + 1.3 = 4.1% ✓
  - Ngành D: 1.7 + 1.7 + 0 = 3.4% ✓
- **Cash buffer:** 86% sau Phase 1 >> 30% min regime ✓

Full potential size (Phase 1 + 2 toàn bộ):

| Ticker | Full target % | Phase 1 | Phase 2 (nếu confirm) |
|---|---|---|---|
| A1 | 4.9% | 2.9% | +2.0% |
| A2 | 4.2% | 2.5% | +1.7% |
| A4 | 2.8% | 1.1% | +1.7% |
| B1 | 4.2% | 1.7% | +2.5% |
| B2 | 2.8% | 1.1% | +1.7% |
| B3 | 2.1% | 1.3% | +0.8% |
| D1 | 4.2% | 1.7% | +2.5% |
| D2 | 2.8% | 1.7% | +1.1% |
| D3 | 1.05% | 0% | +1.05% |

Full potential: 29.0% deployed. Cash buffer long-term: 71%.

### Bước 5 — Sequence entry

**T+0:** A1 (2.9%), A2 (2.5%) = 5.4% deploy — Bucket 1 High priority

**T+1:** B1 phase 1 (1.7%), D1 phase 1 (1.7%) = 3.4% — Bucket 2 High first tranche

**T+2:** A4 (1.1%), B3 (1.3%) = 2.4% — Bucket 1 Medium

**T+3:** B2 phase 1 (1.1%), D2 (1.7%) = 2.8% — Bucket 2 Medium

**T+4:** Review, adjust unfilled orders

D3 Bucket 3: watchlist, chờ zone tuần bật A hoặc week_score dương

### Bước 6 — Limit orders (A1 ví dụ)

A1 Base target 85k, Bear target 58k, current 72k.
- Limit max: 85 × 0.85 = 72.3k → OK vào tại current price
- Limit min: 58 × 1.1 = 63.8k

Split orders:
- 40% position tại 72k (current)
- 30% position tại 70.5k (pullback 2%)
- 30% position tại 69k (pullback 4%)

Nếu orders 2-3 không fill trong 3-5 phiên → cancel, reassess.

### Summary output

- **Phase 1 deployment:** 14% portfolio trong 4 phiên
- **Cash buffer còn lại:** 86% (dư cho Phase 2 confirm + opportunistic)
- **Full deployment potential:** 29% nếu tất cả Bucket 2 confirm
- **Cash buffer min long-term:** 71% (cao hơn 30% target regime Risk-on selective)
- **Số positions:** 8 active + 1 watchlist

---

## 10. Template Checkpoint 6

```
# Checkpoint 6 — Portfolio Construction [ngày]

## 1. Summary quyết định

Phase 1 deployment: [X%] portfolio qua [N] positions
Cash buffer sau Phase 1: [Y%]
Full potential (sau Phase 2 confirm): [Z%]
Cash buffer min long-term: [W%]

Phân bổ:
- High conviction: [X positions, Y% total]
- Medium conviction: [X positions, Y% total]
- Low conviction: [X positions, Y% total]
- Bucket 3 watchlist: [X positions, 0% now]

## 2. Bối cảnh đầu vào
- Portfolio USD/VND total: [X]
- Regime hiện tại: [Risk-on full/selective/Defensive/Đứng ngoài]
- Number of memos passed tier 5C: [N]
- Existing positions (nếu có): [list + % portfolio]
- Ngày chạy tier 6: [date]

## 3. Bảng allocation đầy đủ

| Ticker | Ngành | Conviction | Bucket | Base % | × Regime | × Bucket P1 | ADV cap | Phase 1 final % | Phase 2 potential % | Full % |
|---|---|---|---|---|---|---|---|---|---|---|
| ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |

## 4. Constraint check

- ADV constraint: [list mã bị cap + lý do, nếu có]
- Max per stock (10% cap): [check, pass/flag]
- Max per industry (30-40% tuỳ regime): [breakdown per ngành]
- Min cash buffer: [% actual vs target, pass/fail]
- Catalyst play exposure: [% total, pass/flag nếu > 15%]

## 5. Sequence entry plan

### Phase 1 — ngày T+0 đến T+4
[Bảng phân bổ theo ngày]

### Phase 2 — chờ confirm Bucket 2
Mã A4, B1, B2, D1: vào thêm khi zone w bật A + week_score dương 2-3 phiên liên tiếp. Timeout 4 tuần nếu không confirm.

### Phase 3 — Bucket 3 watchlist
Mã D3: chờ zone w chuyển B/A, day_score dương 3 phiên. Chuyển sang Bucket 2 và apply rule Phase 2.

## 6. Limit orders per stock

| Ticker | Current price | Limit max (Base × 0.85) | Limit min (Bear × 1.1) | Order 1 (40%) | Order 2 (30%) | Order 3 (30%) |
|---|---|---|---|---|---|---|
| A1 | 72k | 72.3k | 63.8k | 72k | 70.5k | 69k |
| ... | ... | ... | ... | ... | ... | ... |

Các mã có current > limit max: [list, cần chờ pullback]
Các mã có current < limit min: [list, rà soát memo — có thể thesis đang fail]

## 7. Diversification check

- Max per stock: [check]
- Max per industry per regime: [list % per ngành]
- Min 3 ngành: [check]
- Marketcap mix: [% Large / Mid / Small]
- Catalyst play % total: [check]

## 8. Cảnh báo đặc biệt (nếu có)

- Mã [X] conviction High 7% nhưng ADV cap → chỉ 3.3% thực tế. Conviction high không realize được qua size
- Ngành [Y] tiệm cận 30% cap, nếu muốn thêm mã ngành Y phải giảm mã hiện tại
- Current portfolio value đã có mã [Z] từ cycle trước, overlap với shortlist → merge size
- Bucket 3 mã D3 nếu không confirm trong 4 tuần sau chuyển Bucket 2 → abort

## 9. Lựa chọn sát nút

Nếu user không agree bảng allocation:
- (a) Adjust size per stock cụ thể (user specify % thay Agent đề xuất)
- (b) Thay đổi sequence (ví dụ ưu tiên mã nào vào trước)
- (c) Skip 1-2 mã, vào portfolio nhỏ hơn
- (d) Hold sang phase sau — vào 1 phần (ví dụ 50% kế hoạch) rồi đánh giá 2 tuần

## 10. Câu hỏi chờ user

Confirm bảng allocation + sequence entry để đặt orders?
Hoặc muốn:
- (a) Adjust size specific stocks
- (b) Thay đổi sequence (ưu tiên khác)
- (c) Reduce deployment (vào 50% kế hoạch, chờ 2 tuần đánh giá)
- (d) Loại 1-2 mã nếu thấy position quá phân tán
- (e) Re-check 1 constraint cụ thể (ví dụ ADV cho mã liquidity thấp)

Sau khi confirm → Agent xuất order list cuối cùng (file sẵn để user đặt lệnh qua broker). User tự đặt lệnh thực tế — Agent không place orders.

Khi orders filled, sang tier 7 (monitoring + exit management, `P_invest_memo_09`).
```

**Độ dài target:** 4-5 trang.

---

## 11. Failure mode

### 11.1. Ép size conviction cao khi ADV không cho phép

Conviction High → 7% target, nhưng ADV cap chỉ 3%. Agent ignore cap, đưa size 7% → user đặt lệnh không đủ liquidity, slippage lớn hoặc không fill đủ.

**Xử lý:** ADV cap là hard constraint (Nguyên tắc 3 của `P_invest_memo_00`). Size cuối = min(target, ADV cap). Flag rõ cho user "conviction cao nhưng size thực tế bị giới hạn bởi thanh khoản". User có thể chọn: giảm size xuống cap, hoặc skip mã chuyển sang mã conviction thấp hơn nhưng ADV đủ.

### 11.2. Tính base size theo midpoint thay vì điểm cụ thể

Agent lấy midpoint của range (High 6-8% → 7% mặc định) cho mọi mã High, không phân biệt 15đ vs 18đ.

**Xử lý:** dùng cụ thể trong range theo điểm (Section 3.1). 15-16đ → 6%, 17đ → 7%, 18đ → 8%. Cho Medium/Low tương tự. Điểm tổng đã phản ánh cường độ conviction, size phải tương ứng.

### 11.3. Bỏ qua regime adjustment

Agent dùng base size 1.0x trong regime Risk-on selective (0.7x). Kết quả portfolio deploy quá nhiều khi regime không supportive.

**Xử lý:** Bước 2 workflow bắt buộc apply regime multiplier. Flag nếu Agent quên — total deployment > regime allow là red flag.

### 11.4. Cash buffer vượt floor không adjust

Sau Phase 1 + full Phase 2, cash buffer < 30% trong Risk-on selective (target min). Agent không adjust.

**Xử lý:** Bước 7 constraint check — nếu full deployment (Phase 1 + Phase 2 max) vi phạm min cash buffer → giảm size low conviction trước (Low → 0%, Medium 50% target). Giữ High conviction full size. Cash buffer là mandatory theo regime.

### 11.5. Không split limit orders

Agent đặt 100% size tại 1 limit price = current price. Nếu pullback xuống lại không có second order ready.

**Xử lý:** split 3 orders theo Section 6.2 (40% current, 30% -2%, 30% -4%). Second/third orders catch pullback opportunity. Nếu không fill trong 3-5 phiên → cancel + reassess.

### 11.6. Bucket 3 watchlist nhưng không set trigger theo dõi

Mã Bucket 3 để 0% position, nhưng không note rõ condition chuyển sang Bucket 2. Agent quên theo dõi.

**Xử lý:** mỗi mã Bucket 3 trong Checkpoint 6 Phần 5 phải có condition cụ thể (zone, week_score threshold). Tier 7 monitoring sẽ track các condition này weekly.

### 11.7. Tổng size < regime allow nhưng cash buffer vẫn cao

Ví dụ Risk-on full cho phép 80-90% deployment, nhưng Agent chỉ tính 30% → cash buffer 70%, thấp hơn optimal theo regime.

**Xử lý:** nếu shortlist tier 3/5 nhỏ (ít mã quality), không ép thêm mã kém. Cash buffer cao là OK khi regime "full" nhưng shortlist không đủ mã quality. Flag cho user: "regime full cho phép 80% deploy nhưng shortlist chỉ deliver 30% — đây là signal thị trường có ít cơ hội quality, cash buffer cao là lựa chọn hợp lý".

Không auto expand shortlist để ép deployment target.

### 11.8. Ngành ceiling cap nhưng vẫn muốn thêm mã

Agent đã có 3 mã ngành A = 28% portfolio. User memo thêm mã A4 (ngành A). Agent ignore cap 30% và thêm mã A4 → 32% ngành A.

**Xử lý:** Bước 7 constraint check — nếu thêm mã mới làm vi phạm max per industry, Agent đề xuất:
- (a) Giảm size mã hiện có trong ngành để fit mã mới
- (b) Skip mã mới, giữ 3 mã hiện có
- (c) User explicit override với audit log

Không tự thêm khi đã max per industry.

### 11.9. Bỏ qua existing positions

User đã có 5 mã từ cycle trước. Agent tính portfolio mới based on 0% current = fresh start → sai total exposure.

**Xử lý:** Bước 4 bắt buộc load existing positions và reconcile. Total portfolio sau tier 6 = existing (adjusted) + new positions từ tier 6. Constraint check applies to tổng này.

---

## 12. Đầu ra chuẩn để tier 7 dùng

Output tier 6 lưu file `tier6_portfolio_YYYYMMDD_confirmed.md`.

1. **Header:** ngày, regime, total portfolio size, số positions
2. **Bảng allocation đầy đủ** — từng mã với Phase 1, Phase 2 potential, Full target
3. **Sequence entry plan** Phase 1 (ngày T+0 đến T+4) với limit orders
4. **Bucket 2 confirm conditions** để tier 7 monitor
5. **Bucket 3 trigger conditions** để tier 7 theo dõi
6. **Constraint log** — flag nếu có mã bị ADV cap, industry cap, v.v.
7. **Order list final** — dạng bảng user copy-paste vào broker platform
8. **Link tier 5C memo files** (reference cho exit triggers)
9. **Audit log** nếu user override bất kỳ constraint nào

File này enable vào project knowledge session tier 7 (monitoring + exit management, `P_invest_memo_09`).

Sau tier 6 confirm, user đặt orders thực tế qua broker. Khi orders filled, Agent chuyển sang tier 7 để monitor positions + execute exit triggers khi condition match.
