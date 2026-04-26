# O_invest_memo_06 — Quarterly Review

Spec render báo cáo review quarterly. **Rigid structure** — báo cáo sâu nhất trong chu kỳ monitoring: BCTC quý mới, thesis verification, tier 5A forensic partial, tier 5B valuation update, exit levels update.

Quarterly review quan trọng hơn monthly vì tích hợp dữ liệu BCTC chính thức — đây là điểm kiểm chứng cốt lõi của thesis.

Reference: `O_invest_memo_00` (master rules), `P_invest_memo_09` tier 7 quarterly workflow (mục 3.4).

## 1. Input state & mapping

**State file chính:**

| State file | Role |
|---|---|
| `tier7_quarterly_<YYYY_Q>.md` | Quarterly review output — BCTC quý mới per-stock, thesis check, forensic partial, valuation update, exit update |

**State file tham khảo (nhiều hơn monthly):**

- `tier6_portfolio_*_confirmed.md` version mới nhất
- `tier5A_<ticker>_*_confirmed.md` cho mỗi mã active — để so BCTC quý mới với baseline forensic
- `tier5B_<ticker>_*_confirmed.md` cho mỗi mã active — để update valuation với actual data
- `tier5C_<ticker>_*_confirmed.md` cho mỗi mã active — để check thesis vs assumption
- `tier0_<YYYYMMDD>_confirmed.md` — regime baseline
- 3 monthly review trong quý — tổng hợp performance

**Dữ liệu agent_db bổ sung:** `stock_finstats` cần được query cho period quý mới công bố (thường cuối tháng sau quý kết thúc).

## 2. Structure báo cáo đầu ra

**Độ dài target:** 5-8 trang MD cho portfolio 9-15 mã.

**Rigid structure bắt buộc:**

```
# Quarterly Review — <Q[N]/YYYY>

## 1. Executive summary
## 2. Performance quarterly
## 3. BCTC quý mới — per-stock
## 4. Thesis verification
## 5. Forensic partial re-check
## 6. Valuation update
## 7. Exit levels update
## 8. Rebalance proposal
## 9. Actions needed
— Metadata
```

## 3. Compose từng phần chi tiết

### Phần 1 — Executive summary

```
**Quý review:** Q[N]/YYYY ([tháng bắt đầu - tháng kết thúc]/YYYY)  
**Ngày review:** [DD/MM/YYYY]  
**BCTC quý [N-1]** đã công bố cho [X] mã / [Y] mã active ([chờ BCTC của Z mã]).

**Performance quý:**
- Return quý (QoQ): [+/-X]%
- Return YTD: [+/-Y]%
- Return cycle: [+/-Z]%
- Alpha vs VN-Index: [+/-W]%

**Thesis verification kết quả:**
- Thesis intact: [N] mã
- Thesis partial realize: [M] mã (catalyst đã realize, target re-rate)
- Thesis deteriorating: [K] mã (cần tighten stop hoặc giảm size)
- Thesis fail: [L] mã (đề xuất thoát)

**Major actions đề xuất:** 
- [Bullet 3-5 action quan trọng nhất quý này]
```

### Phần 2 — Performance quarterly

```
### 2.1. Returns by period

| Period | Portfolio | VN-Index | Alpha |
|---|---|---|---|
| Quý này (QoQ) | [+/-X]% | [+/-Y]% | [+/-Z]% |
| YTD | ... | ... | ... |
| Cycle từ start | ... | ... | ... |

### 2.2. Win rate cycle

- Positions closed từ cycle start: [N] (winner [X] / loser [Y])
- Win rate cycle: [Z]%
- Avg gain của winner: +[X]%
- Avg loss của loser: -[Y]%
- Win/loss magnitude ratio: [X/Y = Z:1]

### 2.3. Drawdown analysis

- Max drawdown cycle: [X]%
- Current drawdown (nếu đang trong): [Y]%
- Recovery time từ max drawdown: [Z] tuần (nếu đã recover)
```

**Chart đề xuất:**

Chart 1 — Equity curve 3 tháng + benchmark:

````
```chart
type: line
title: Equity Curve Q[N]/YYYY — Portfolio vs VN-Index
x_axis: [list tuần trong quý]
y_axis:
  - name: Portfolio
    data: [...]
  - name: VN-Index
    data: [...]
y_label: Normalized 100 tại đầu quý
source: Tổng hợp
render_in_md: skip
render_in_docx: true
render_in_pptx: true
```
````

Chart 2 — Monthly returns bar (3 tháng):

````
```chart
type: bar
title: Monthly Returns Q[N]/YYYY
x_axis: [Tháng 1, Tháng 2, Tháng 3]
y_axis:
  - name: Portfolio
    data: [+/-X1, +/-X2, +/-X3]
  - name: VN-Index
    data: [+/-Y1, +/-Y2, +/-Y3]
y_label: '%'
source: Tổng hợp
render_in_md: skip
render_in_docx: true
render_in_pptx: true
```
````

### Phần 3 — BCTC quý mới — per-stock

**Đây là phần cốt lõi quarterly review** — dữ liệu quý mới công bố là sự thật cuối cùng về thesis.

Structure: bảng tóm tắt + chi tiết per-stock cho thesis changes.

```
### 3.1. BCTC Q[N-1]/YYYY — tóm tắt

| Ticker | Revenue Q (tỷ) | YoY | QoQ | NPAT Q (tỷ) | YoY | ROE (%) | vs Base assumption |
|---|---|---|---|---|---|---|---|
| [X] | 3,450 | +12% | +8% | 480 | +15% | 24% | Đạt / Miss nhẹ / Miss lớn |
| [Y] | ... | ... | ... | ... | ... | ... | ... |
```

**Status values:**
- `Đạt` — số thực ≥ 95% Base assumption
- `Miss nhẹ` — số thực 85-95% Base (chưa critical)
- `Miss lớn` — số thực < 85% Base → thesis có vấn đề
- `Vượt` — số thực > 110% Base (Bull case đang realize)

```
### 3.2. Chi tiết per-stock (chỉ render nếu Miss lớn hoặc Vượt)

**[Ticker X]** — Status: [Miss lớn / Vượt]

- BCTC Q[N-1]: Revenue [X] tỷ ([+/-Y]% YoY), NPAT [Z] tỷ ([+/-W]% YoY), ROE [V]%
- Base case assumption từ memo tier 5C: Revenue [X'] tỷ ([+/-Y']% YoY)
- Gap vs assumption: Revenue -15%, NPAT -22%, ROE giảm 3 điểm %
- Driver gap: [1-2 câu cụ thể — segment nào miss, lý do]
- Catalyst memo đã list: [nào realize, nào fail]
- Thesis impact: [partial changed / fully broken / strengthen]
- Nguồn: BCTC [Ticker] Q[N-1]/YYYY soát xét (trang [X] nếu có)
```

**Nếu mã chưa có BCTC quý mới** (lùi công bố):

```
### 3.3. Chờ BCTC

[List mã chưa công bố + ngày dự kiến + rule review khi có: schedule session spot check sau khi BCTC publish]
```

### Phần 4 — Thesis verification

Tổng hợp thesis check với BCTC actual. Structure:

```
| Ticker | Thesis memo | BCTC actual | Kết luận | Action |
|---|---|---|---|---|
| [X] | BHX break-even Q1/2026 | Q1 vẫn loss nhẹ nhưng narrowing (-8 tỷ vs -25 tỷ Q4/2025) | Thesis partial realize, chậm 1 quý | Giữ position, update exit levels |
| [Y] | DSO ổn định 45 ngày | DSO tăng lên 58 ngày | Thesis deteriorating | Giảm size 30-50%, tighten stop |
| [Z] | Revenue CAGR 12% | Q1 revenue YoY +18% | Vượt, Bull case realize | Take profit partial, cân nhắc extend target |
| [W] | Margin ổn định 22% | Margin co hẹp xuống 17% 2 quý liên tiếp | Thesis fail | Đề xuất thoát |
```

**Cho mỗi mã thesis fail/deteriorating:**

```
**[Ticker W]** — Thesis fail deep-dive

Thesis gốc (từ memo tier 5C ngày [DD/MM]): [copy 1-2 câu thesis cốt lõi]

BCTC Q[N-1] reality:
- Margin: 17.2% (vs 22% assumption) — 2 quý liên tiếp co hẹp
- Revenue growth: +5% (vs +15% assumption) — dưới consensus
- Cause: [lý do cụ thể từ thuyết minh BCTC hoặc management commentary]

**Impact:**
- Target Base update xuống từ [X]k về [Y]k (-[Z]%)
- Hard trigger có thể match (price < Bear × 0.9 = [W]k) → cân nhắc thoát trước khi trigger

**Proposal:** bán 70-100% position. User quyết.
```

### Phần 5 — Forensic partial re-check

Không full re-run tier 5A (quá tốn), chỉ spot-check 2-3 tác vụ quan trọng theo rule `P_invest_memo_09` mục 3.4 step 3.

Structure:

```
### 5.1. Spot check per-stock

| Ticker | CFO/LNST ratio Q[N-1] | DSO trend | Bên liên quan | Red flag mới |
|---|---|---|---|---|
| [X] | 0.85 (vs 0.82 Q[N-2], healthy) | 45 → 48 (trend nhẹ, OK) | Stable | Không |
| [Y] | 0.42 (vs 0.78 Q[N-2], DROP) | 45 → 58 (tăng mạnh) | Stable | **Cảnh báo quality lợi nhuận** |
| ... | ... | ... | ... | ... |
```

**Cho mỗi mã có red flag mới:**

```
**[Ticker Y]** — Red flag mới phát hiện

- CFO/LNST ratio giảm từ 0.78 xuống 0.42 trong quý mới → lợi nhuận có thể chưa chuyển thành dòng tiền thực
- DSO tăng từ 45 lên 58 ngày → khách hàng trả chậm, có thể lead to bad debt
- Kết hợp 2 signal → cảnh báo quality lợi nhuận Q[N-1]

**Impact:** thesis memo dựa trên giả định margin stable + quality ổn định. Quality signal xấu → bear case cần update.

**Proposal:** tier 5A forensic partial full re-run cho mã Y trong tuần tới. Cho đến khi clear → giảm size 30%, tighten stop.
```

### Phần 6 — Valuation update

Quick DCF re-run với BCTC actual, không rebuild model full.

Structure:

```
| Ticker | Base cũ | Base update | Change | Lý do |
|---|---|---|---|---|
| [X] | 85k | 88k | +3.5% | Revenue CAGR assumption cao hơn 1%, margin strengthen |
| [Y] | 95k | 78k | -18% | Margin assumption giảm từ 22% → 18%, WACC tăng do risk |
| [Z] | 65k | 65k | 0% | Giữ nguyên, không có lý do change |
| ... | ... | ... | ... | ... |
```

**Ghi chú cho mỗi change > 10%:**

```
**[Ticker Y]** — Valuation update rationale

- Margin assumption cũ: 22% (từ model tier 5B ngày [DD/MM])
- Margin thực tế Q[N-2] + Q[N-1]: 18% trung bình, trend co hẹp
- Update assumption: margin năm 5 = 18% (thay vì 20%), WACC tăng từ 11% lên 12.5% do rủi ro tăng
- Terminal g giữ 3%
- Target Base update: 78k (giảm 18% từ 95k)
- Bull và Bear update tương ứng: [list]
```

### Phần 7 — Exit levels update

Dựa trên valuation update phần 6 + thesis verification phần 4:

Structure:

```
| Ticker | Hard trigger cũ | Hard trigger mới | Take profit cũ (Base/Bull) | Take profit mới (Base/Bull) |
|---|---|---|---|---|
| [X] | < 52k | < 52k (giữ) | 85/105k | 88/108k (raise) |
| [Y] | < 50k | < 62k (tighten) | 95/115k | 78/92k (lower) |
| [Z] | ... | ... | ... | ... |
```

**Logic update:**
- Target raise (thesis strengthen): hard trigger giữ hoặc loosen nhẹ, take profit tăng
- Target lower (thesis deteriorating): hard trigger tighten (đặt gần current price), take profit giảm
- Thesis fail: hard trigger = current price - 3% để thoát nhanh, take profit irrelevant

### Phần 8 — Rebalance proposal

Tổng hợp rebalance từ phần 4 (thesis action) + phần 6 (target change) + phần 7 (exit update).

Structure:

```
**Rebalance summary:**

| Action | Ticker | Current % | Target % | Rationale |
|---|---|---|---|---|
| Trim | [X] | 5% | 3.5% | Target Base vượt quá, take profit partial |
| Exit | [W] | 2% | 0% | Thesis fail sau BCTC Q |
| Tighten stop | [Y] | 3% | 3% (giữ size) | Thesis deteriorating, stop từ 50k → 62k |
| Add | [New] | 0% | 2% | Thesis [New] từ watchlist đã strengthen, upgrade vào |

**Order list rebalance:**

```text
REBALANCE Q[N] ORDERS — [DD/MM/YYYY]

Trim (take profit partial):
  [X] | SELL | LIMIT [price]k | qty [X]cp | from 5% → 3.5%

Exit (thesis fail):
  [W] | SELL | MARKET | qty [Y]cp | position close

Add (watchlist upgrade):
  [New] | BUY | LIMIT [price]k | qty [Z]cp | new position 2%
```

**Impact:**
- Cash change: [+/-X]% → new buffer [Y]%
- Number of positions: [old] → [new]
```

### Phần 9 — Actions needed

```
**Actions cần user quyết:**

- [ ] (a) Approve rebalance proposal phần 8?
- [ ] (b) Thoát [Ticker W] vì thesis fail — confirm?
- [ ] (c) Chạy tier 5A forensic partial full cho [Ticker Y] (red flag mới)?
- [ ] (d) Update exit levels cho toàn portfolio theo bảng phần 7?
- [ ] (e) [Mã Z vượt Base target nhiều] — extend target hay take profit?
- [ ] (f) Regime check — có cần re-run tier 0 không?
```

### Metadata

```
---

**Metadata**

- **Quý:** Q[N]/YYYY
- **Ngày review:** [DD/MM/YYYY]
- **Portfolio size current:** [X] tỷ VND
- **Cycle status:** [đang tiếp tục / gần kết thúc / chuyển cycle mới]
- **State file nguồn:** `tier7_quarterly_<YYYY_Q>.md`
- **BCTC công bố status:** [N]/[M] mã
- **Portfolio plan version hiện tại:** [X.Y]
- **Next review:** Q[N+1]/YYYY, sau khi BCTC Q[N] công bố (thường cuối tháng [N+1+1])
```

## 4. Compose workflow step-by-step

**Bước 1 — Format + template.**

**Bước 2 — Load state đầy đủ:** tier 7 quarterly + portfolio + tier 5A/5B/5C per mỗi mã active + tier 0 + 3 monthly của quý.

**Bước 3 — Query BCTC quý mới từ `stock_finstats`:** theo period mới nhất. List mã đã công bố vs chưa.

**Bước 4 — Compute performance metrics** (quý + YTD + cycle + win rate + drawdown).

**Bước 5 — Bảng BCTC tóm tắt phần 3.1:** status Đạt/Miss/Vượt per mã.

**Bước 6 — Thesis verification phần 4:** cross-check BCTC actual với memo assumption của mỗi mã. Phân loại 4 trạng thái thesis.

**Bước 7 — Forensic partial phần 5:** spot check CFO/LNST + DSO + bên liên quan từ BCTC quý mới.

**Bước 8 — Valuation update phần 6:** quick DCF với assumption update nếu thesis changed. Đây là tác vụ nặng nhất, agent có thể cần query thêm agent_db cho peer multiples update.

**Bước 9 — Exit levels update phần 7** dựa trên target mới.

**Bước 10 — Rebalance proposal phần 8** tổng hợp.

**Bước 11 — Compose executive summary phần 1** và actions phần 9.

**Bước 12 — Chart annotation** (equity curve + monthly returns).

**Bước 13 — Self-check:**
- Số BCTC announced khớp phần 1 và phần 3
- Thesis verification: mỗi mã có kết luận rõ (1 trong 4 trạng thái)
- Target update logic đúng (margin changed → target changed theo hướng tương ứng)
- Exit levels update khớp với target update (target raise → TP raise, thesis fail → stop tighten)
- Rebalance order list khớp với proposal table
- Order quantity round lot 100 cp
- Thuật ngữ Regime/Bucket/Phase/Conviction có dịch
- Citation: BCTC PDF cụ thể (Nhóm 3), "(nguồn: Tổng hợp)" cho agent_db data, link finext cho tin
- Metadata đầy đủ BCTC status + portfolio plan version

**Bước 14 — Render + present.**

## 5. Guide render docx

Quarterly docx là **deliverable formal nhất** trong 3 loại monitoring — báo cáo cycle-level cho sếp/committee.

**Layout:**
- Cover (trang 1): "Quarterly Review — Q[N]/YYYY" + logo + portfolio size + cycle status
- Executive summary (trang 2) — full version
- Performance (trang 3): bảng + 2 chart (equity curve + monthly bar)
- BCTC quý mới (trang 4-5): bảng tóm tắt + chi tiết per-stock
- Thesis verification (trang 6): bảng + chi tiết mã fail/deteriorating
- Forensic partial (trang 7): bảng spot check + red flag detail
- Valuation update (trang 8): bảng target change + rationale
- Exit levels update (trang 9): bảng
- Rebalance proposal (trang 10): bảng + order list monospace
- Actions + Metadata (trang 11)

**Template:** đầy đủ tất cả style + landscape cho bảng lớn (BCTC 7-8 cột).

## 6. Guide render pptx

Quarterly pptx cho **review meeting cycle** — có thể là thời điểm quan trọng nhất của quan hệ với portfolio committee.

**Layout 15-20 slide:**

| Slide | Nội dung |
|---|---|
| 1 | Cover Q[N]/YYYY |
| 2 | Executive summary — key metrics + thesis status |
| 3 | Performance — equity curve |
| 4 | Monthly returns bar |
| 5 | Win rate + drawdown breakdown |
| 6 | BCTC summary table top mã |
| 7-8 | BCTC detail cho mã Miss lớn / Vượt |
| 9 | Thesis verification matrix |
| 10 | Thesis fail deep-dive |
| 11 | Forensic spot check red flag |
| 12 | Valuation update table |
| 13 | Exit levels update |
| 14-15 | Rebalance proposal |
| 16 | Order list |
| 17 | Regime & next quarter outlook |
| 18 | Actions needed |
| 19-20 | Q&A / appendix |

## 7. Xử lý cycle end

Nếu quarterly review trùng với end-of-cycle (positions đã thoát hết hoặc gần hết, hoặc user muốn re-evaluate toàn bộ):

**Thêm phần 10 "Cycle recap"** (chỉ khi end-of-cycle):

```
## 10. Cycle recap

### 10.1. Final P&L

- Total return cycle: [+/-X]%
- Number cycles days: [N]
- Annualized return: [+/-Y]%
- Win rate cycle: [Z]% ([X win] / [M total])
- Max drawdown: [W]%

### 10.2. Lessons learned

**Thesis correct (winner):**
- [Mã W]: thesis [...] realize, driver chính [...]. Lesson: [learning cho cycle sau]
- ...

**Thesis fail (loser):**
- [Mã L]: thesis [...] fail do [...]. Gate 1/2 miss [...]. Lesson: [...]
- ...

### 10.3. Process improvement

[Bullet 2-4 recommendation cụ thể để cải thiện cycle sau]
- Chấm điểm tier 3 nên thêm tiêu chí [X] vì pattern [...]
- Tier 5A forensic nên spot check [Y] sớm hơn
- Bucket 2 confirm condition nên [...]

### 10.4. Next cycle readiness

- Cash available: [X]% portfolio ([Y] tỷ VND)
- Watchlist carry over: [list mã Bucket 3 còn observe]
- Ready to start tier 0 cycle mới: [Yes/No + lý do]
```

Phần này nối sang cycle mới qua `O_invest_memo_01` market scan cho tier 0/1/2/3 mới.

## 8. Ví dụ fragment

```markdown
# Quarterly Review — Q1/2026

## 1. Executive summary

**Quý review:** Q1/2026 (01/2026 - 03/2026)  
**Ngày review:** 25/04/2026  
**BCTC quý Q1/2026** đã công bố cho 7/9 mã active (chờ BCTC của MWG và FPT).

**Performance quý:**
- Return quý (QoQ): +8.2%
- Return YTD: +8.2%
- Return cycle: +22.5% (từ 01/11/2025)
- Alpha vs VN-Index: +3.1%

**Thesis verification kết quả:**
- Thesis intact: 4 mã (VNM, HPG, VCB, ACV)
- Thesis partial realize: 2 mã (catalyst đã realize sớm hơn dự kiến)
- Thesis deteriorating: 2 mã (margin co hẹp, DSO tăng)
- Thesis fail: 1 mã (NVL — target xuống 18k từ 35k)

**Major actions đề xuất:**
- Thoát NVL (thesis fail)
- Tier 5A forensic partial cho MSN (red flag CFO/LNST)
- Take profit partial VNM (vượt Base 92%)
- Chờ BCTC MWG và FPT cuối tháng 5
```

## 9. Checklist self-check cuối cùng

- [ ] State `tier7_quarterly_*` đã có
- [ ] 9 section đầy đủ (+ section 10 nếu end-of-cycle)
- [ ] Số BCTC announced khớp giữa phần 1 exec summary và phần 3
- [ ] Thesis verification: mỗi mã có 1 trong 4 trạng thái (intact / partial / deteriorating / fail)
- [ ] Mã thesis fail có deep-dive detail với BCTC reality vs thesis memo
- [ ] Forensic partial check CFO/LNST + DSO + bên liên quan per mã
- [ ] Valuation update logic: target change tương thích với assumption change
- [ ] Exit levels update tương thích: target raise → TP raise, thesis fail → stop tighten
- [ ] Rebalance proposal khớp với thesis verification + valuation + exit updates
- [ ] Order list quantity round lot 100 cp
- [ ] Nếu end-of-cycle: phần 10 có lessons learned concrete + next cycle readiness
- [ ] Citation: BCTC Nhóm 3 với tên tài liệu + trang, "(nguồn: Tổng hợp)" cho agent_db, link finext cho tin
- [ ] Thuật ngữ Regime/Bucket/Phase/Conviction có dịch
- [ ] Metadata đầy đủ: quý + ngày + portfolio size + BCTC status + portfolio plan version
- [ ] Format cuối đúng yêu cầu + template đúng
