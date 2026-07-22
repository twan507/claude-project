# O_invest_memo_03 — Portfolio Plan

Spec render báo cáo portfolio allocation + sequence entry + order list từ state tier 6. Đây là **deliverable actionable** — user dùng trực tiếp đặt lệnh qua broker.

Reference: `O_invest_memo_00` (master rules), `P_invest_memo_08` (tier 6 portfolio construction).

## 1. Input state & mapping

**1 state file chính:**

| State file | Role | Nội dung |
|---|---|---|
| `tier6_portfolio_<YYYYMMDD>_confirmed.md` | Portfolio plan base | Allocation bảng, sequence entry phase 1-3, limit orders, constraint check, cash buffer |

**State file tham khảo (để enrich memo summary phần 4):**

Cho mỗi mã trong portfolio, cần `tier5C_<ticker>_<YYYYMMDD>_confirmed.md` (memo 7 phần) để extract thesis 1 câu + target giá + catalyst cá thể cho memo summary trong portfolio plan.

**Nếu thiếu tier5C của mã nào:**
- Flag rõ ở memo summary mã đó: "Chưa có memo deep-dive, thông tin từ tier 3 chấm điểm"
- Vẫn render mã đó trong allocation bảng và order list

## 2. Structure báo cáo đầu ra

**Độ dài target:** 6-10 trang tuỳ số mã portfolio (thường 9-15 mã).

**Flex structure** theo master mục 8 (portfolio plan là one-off per cycle + rebalance).

Template structure:

```
# Portfolio Plan — <Ngày>

## 1. Executive summary
## 2. Portfolio allocation
## 3. Constraint check
## 4. Mã trong portfolio — thesis tóm tắt
## 5. Sequence entry plan
## 6. Limit orders — bảng đọc
## 7. Order list — block copy cho broker
## 8. Cảnh báo & lựa chọn sát nút
— Metadata
```

## 3. Compose từng phần chi tiết

### Phần 1 — Executive summary

Format cứng:

```
**Ngày plan:** [DD/MM/YYYY]  
**Portfolio size total:** [X] tỷ VND ([Y] triệu USD)  
**Regime (trạng thái thị trường):** [regime từ tier 0]  
**Phase 1 deployment:** [X]% portfolio — [N] positions  
**Cash buffer sau Phase 1:** [Y]%  
**Full potential sau Phase 2:** [Z]% — cash buffer long-term [W]%

**Phân bổ theo conviction tier:**
- High (cao): [N] positions, [X]% total
- Medium (trung bình): [N] positions, [X]% total
- Low (thấp): [N] positions, [X]% total
- Bucket 3 (watchlist, chưa vào): [N] positions, 0% hiện tại

**Phân bổ theo ngành:**
- [Ngành 1]: [X]% ([N] mã)
- [Ngành 2]: [X]% ([N] mã)
- ...

**Top concentration:**
- Mã max: [Ticker] [X]% portfolio
- Ngành max: [Ngành] [X]% portfolio
```

Độ dài 0.5-1 trang.

### Phần 2 — Portfolio allocation

Bảng đầy đủ:

```
| # | Ticker | Ngành | Conviction | Điểm | Bucket | Base % | × Regime | × Bucket P1 | ADV cap | Phase 1 final % | Phase 2 potential % | Full % |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 1 | [X] | [Ngành] | High (cao) | 16 | 1 (vào ngay) | 7% | 4.9% | 2.9% | — | 2.9% | +2.0% | 4.9% |
| 2 | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |
```

**Note:**
- Cột "ADV cap" chỉ điền giá trị nếu cap thực sự bind (constraint thanh khoản kích hoạt). Ngược lại để trống hoặc "—".
- Hàng cuối bảng tổng cộng: Phase 1 final total + Phase 2 potential total + Full total.

**Chart đề xuất phần 2:**

Chart 1 — Allocation distribution (pie):

````
```chart
type: pie
title: Phân bổ Portfolio — Phase 1
x_axis: [list ticker]
y_axis:
  - name: % portfolio
    data: [list giá trị %]
source: Tổng hợp
render_in_md: skip
render_in_docx: true
render_in_pptx: true
```
````

Chart 2 — Phân bổ theo ngành (stacked bar hoặc pie):

````
```chart
type: pie
title: Phân bổ theo ngành — Phase 1
x_axis: [list ngành]
y_axis:
  - name: % portfolio
    data: [list giá trị %]
source: Tổng hợp
render_in_md: skip
render_in_docx: true
render_in_pptx: true
```
````

### Phần 3 — Constraint check

Bảng kiểm tra constraint:

```
| Constraint | Ngưỡng rule | Thực tế | Status |
|---|---|---|---|
| ADV cap per stock | ≤ 5% ADV 20 phiên × 3 phiên accumulate | [list mã + ADV cap nếu bind] | Pass / Flag [mã X, Y cap thực tế] |
| Max per stock | ≤ 10% portfolio | Mã max [X]% | Pass / Flag |
| Max per industry | ≤ [X]% theo regime | Ngành max [Y]% | Pass / Flag |
| Min cash buffer | ≥ [X]% theo regime | [Y]% Phase 1 / [Z]% long-term | Pass / Fail |
| Catalyst play exposure | ≤ 15% portfolio | [X]% total | Pass / Flag |
| Min positions | ≥ 5-8 mã | [N] mã | Pass / Flag |
| Max positions | ≤ 12-15 mã | [N] mã | Pass / Flag |
| Min ngành | ≥ 3 ngành | [N] ngành | Pass / Flag |
```

**Với mỗi flag/fail:** 1 đoạn ngắn giải thích — vi phạm chỗ nào, impact gì, đã xử lý thế nào (giảm size mã X, skip mã Y, v.v.). Nếu có override từ user ở checkpoint → ghi rõ trong phần 8 cảnh báo.

### Phần 4 — Mã trong portfolio — thesis tóm tắt

**Đây là phần tạo giá trị của portfolio plan** — reader (có thể là sếp duyệt allocation) cần biết thesis mỗi mã mà không phải đọc 5 memo riêng biệt.

Cho mỗi mã (thứ tự giảm theo % portfolio):

```
### [Ticker] — [X]% portfolio ([Ngành])

- **Conviction:** [tier (dịch)] — [điểm]/18
- **Target giá:** Base [X1]k / Bull [X2]k / Bear [X3]k (current [Y]k, upside Base [Z]%)
- **Thesis:** [1-2 câu cốt lõi — differentiated view + evidence key]
- **Catalyst cá thể:** [1 câu — catalyst + timing]
- **Bucket:** [1/2/3] — [lý do ngắn về entry timing]
- **Risk chính:** [1 câu bear case mạnh nhất]
```

Độ dài: 5-8 dòng/mã. Với 9-15 mã → 2-4 trang phần này.

**Nếu thiếu memo tier 5C của mã:**
- Vẫn render block này, dùng data từ tier 3 chấm điểm (thesis ngắn từ phần bảng chấm)
- Flag: "Memo deep-dive chưa có, cần chạy `O_invest_memo_02` cho mã này trước khi vào position"

### Phần 5 — Sequence entry plan

Phân theo ngày T+0 đến T+4 + Phase 2/3 conditional:

```
### 5.1. Phase 1 — Ngày T+0 đến T+4

| Ngày | Mã vào | Size deploy |
|---|---|---|
| T+0 | Bucket 1 High ([ticker list]) | [X]% portfolio |
| T+1 | Bucket 2 High first tranche ([list]) + Bucket 1 Medium ([list]) | [Y]% |
| T+2 | Bucket 1 Low ([list]) + adjust unfilled T+0 orders | [Z]% |
| T+3 | Bucket 2 Medium first tranche ([list]) | [W]% |
| T+4 | Review, adjust unfilled orders | — |

### 5.2. Phase 2 — Chờ confirm Bucket 2

| Mã | Phase 1 đã vào | Phase 2 potential | Condition confirm |
|---|---|---|---|
| [X] | [X1]% | +[X2]% | Zone tuần bật A trở lên (đang B/C) + week_score dương 2-3 phiên + day_score dương 2/3 phiên gần nhất + volume cao hơn trung bình |
| ... | ... | ... | ... |

**Timeout:** 4 tuần kể từ Phase 1. Nếu không confirm sau timeout → review theo rule `P_invest_memo_09`.

### 5.3. Phase 3 — Bucket 3 watchlist

| Mã | Target size nếu upgrade | Condition upgrade sang Bucket 2 |
|---|---|---|
| [X] | [Y]% | Zone tuần chuyển B hoặc A (đang C) 2-3 phiên + week_score chuyển dương + day_score dương 3 phiên liên tiếp |
| ... | ... | ... |
```

Condition mô tả tự nhiên theo K hygiene, không dùng `w_trend`, `zone: A` raw.

### Phần 6 — Limit orders — bảng đọc

Format bảng dễ đọc cho user review trước khi đặt lệnh:

```
| # | Ticker | Current price | Limit max (Base × 0.85) | Limit min (Bear × 1.1) | Order 1 (40%) | Order 2 (30%) | Order 3 (30%) | Size total Phase 1 | VND estimate |
|---|---|---|---|---|---|---|---|---|---|
| 1 | [X] | 72.0k | 72.3k | 63.8k | 72.0k | 70.5k | 69.0k | 2.9% | [Y] triệu VND |
| ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |
```

**Ghi chú dưới bảng:**
- Mã có current > limit max: [list mã] — **chờ pullback, không vào ngay**
- Mã có current < limit min: [list mã] — **thesis có thể đang fail, rà soát memo**
- Mã có constraint đặc biệt (ADV cap, liquidity thấp): [list + note]

### Phần 7 — Order list — block copy cho broker

Block text plain user copy-paste vào platform broker. Format chuẩn hoá:

```
```text
PHASE 1 ORDERS — [DD/MM/YYYY]

T+0:
  [TICKER1] | BUY | LIMIT 72.0 | qty [X]cp | = 40% target
  [TICKER1] | BUY | LIMIT 70.5 | qty [Y]cp | = 30% target
  [TICKER1] | BUY | LIMIT 69.0 | qty [Z]cp | = 30% target
  [TICKER2] | BUY | LIMIT 80.0 | qty [X]cp | = 40% target
  ...

T+1:
  [TICKER3] | BUY | LIMIT 62.0 | qty [X]cp | = 40% target (Bucket 2 first tranche)
  ...

T+2:
  ...

Cancel condition: unfilled orders > 5 phiên → cancel + reassess
```
```

Quantity tính sẵn dựa trên: % portfolio × portfolio size / limit price. Round xuống đến lot (100 cp).

**Note cuối block:**
- Limit orders unfilled sau 3-5 phiên → cancel (Order 2-3 pullback thấp hơn, không chắc fill)
- Orders Phase 2 (Bucket 2 tranche kế tiếp) **chưa đặt** — chờ confirm condition trong `P_invest_memo_09` monitoring

### Phần 8 — Cảnh báo & lựa chọn sát nút

Structure:

```
### 8.1. Cảnh báo đặc biệt

[Bullet các cảnh báo từ constraint check + override từ user nếu có]

- Mã [X] conviction High 7% target nhưng ADV cap → thực tế [Y]%. Conviction không realize được qua size.
- Ngành [Y] đã đạt [Z]% portfolio, tiệm cận ngưỡng [W]%. Không thêm mã ngành Y nếu không giảm mã hiện tại.
- User override constraint [X] tại checkpoint CP6 ngày [date], lý do [lý do từ audit log]. Audit log: `audit_overrides.md` entry [date].
- Bucket 3 mã [X] — nếu không confirm upgrade trong 4 tuần → abort watchlist.

### 8.2. Lựa chọn sát nút (mã suýt vào shortlist)

[Bullet mã tier 3 không vào top 3/ngành. User có thể cân nhắc override thêm mã nào nếu thấy phù hợp.]

- [Ticker X] ngành [Y], [điểm]/18, [lý do không vào top 3 — ví dụ "Bucket 3 watchlist sâu, zone tuần và tháng đều yếu"]. Nếu user muốn thêm → thay mã nào trong top 3.
```

### Metadata cuối file

```
---

**Metadata**

- **Ngày plan:** [DD/MM/YYYY]
- **Regime confirmed:** [regime]
- **Portfolio size:** [X] tỷ VND
- **Số positions Phase 1:** [N]
- **Số watchlist Bucket 3:** [M]
- **State file nguồn:** `tier6_portfolio_<YYYYMMDD>_confirmed.md`
- **Memo deep-dive mã:** [list `tier5C_<ticker>_<YYYYMMDD>_confirmed.md` cho mỗi mã trong portfolio]
- **Audit log:** `audit_overrides.md` (nếu có override tại CP6)
- **Next step:** sau khi orders filled, chuyển sang `O_invest_memo_04/05/06` monitoring theo chu kỳ
```

## 4. Compose workflow step-by-step

Khi user yêu cầu render portfolio plan:

**Bước 1 — Xác định format + template** (theo master workflow): hỏi MD/docx/pptx + template.

**Bước 2 — Load state file:**
- `tier6_portfolio_*_confirmed.md` — bắt buộc
- `tier5C_<ticker>_*_confirmed.md` cho mỗi mã trong portfolio — tham khảo cho phần 4

**Bước 3 — Đọc state tier 6:** allocation bảng, constraint check, sequence entry, limit orders, cash buffer, override log nếu có.

**Bước 4 — Extract thesis cho phần 4** từ memo tier 5C của mỗi mã (phần 1 Recommendation + phần 5 Catalysts). Flag mã thiếu memo.

**Bước 5 — Compose executive summary phần 1** làm hook: regime + deployment + conviction mix + concentration.

**Bước 6 — Compose phần 2 allocation bảng + chart pie.**

**Bước 7 — Compose phần 3 constraint check** với status rõ Pass/Flag/Fail.

**Bước 8 — Compose phần 4 thesis tóm tắt** cho từng mã theo thứ tự giảm % portfolio.

**Bước 9 — Compose phần 5 sequence entry** theo ngày T+0 đến T+4 + Phase 2/3 condition.

**Bước 10 — Compose phần 6 limit orders bảng + phần 7 block copy.** Tính quantity cp theo portfolio size.

**Bước 11 — Compose phần 8 cảnh báo + sát nút.**

**Bước 12 — Self-check:**
- Portfolio size total matched giữa phần 1/2/6/7 (không lệch số)
- Sum % portfolio của các mã Phase 1 = Phase 1 deployment ở phần 1
- Ticker uppercase, số tỷ VND/k
- Thuật ngữ Conviction/Bucket/Phase/Regime lần đầu có dịch
- "(nguồn: Tổng hợp)" cho data từ agent_db
- Constraint check đủ 8 items
- Order block quantity round lot đúng (bội của 100 cp)

**Bước 13 — Render format cuối + present file.**

## 5. Guide render docx

> **Render khi user explicit yêu cầu + đã confirm style** (xem `system_prompt.md` mục 4 và `O_invest_memo_00.md` Nguyên tắc 1-2-3). **Body font: Roboto** (fallback Open Sans → Arial). MD final là source of truth — binary derive từ MD, sửa nội dung phải sửa MD trước rồi re-render.

**Layout:**
- Cover (trang 1): "Portfolio Plan — [ngày]" + regime badge + portfolio size
- Executive summary (trang 2) — phần 1 đầy đủ, bảng layout rộng
- Portfolio allocation (trang 3-4) — bảng lớn có thể cần landscape + 2 chart pie
- Constraint check (trang 5) — bảng status
- Thesis tóm tắt mã (trang 6-9) — 1 trang chứa 2-3 mã, format block
- Sequence entry + Limit orders (trang 10-11) — bảng chi tiết
- Order list block copy (trang 12) — monospace font cho copy dễ
- Cảnh báo + Metadata (trang 13) — cuối

**Template cần có:**
- Heading styles
- Table style (quan trọng — nhiều bảng lớn)
- Monospace style cho order block
- Landscape page option cho bảng lớn

## 6. Guide render pptx

> **Render khi user explicit yêu cầu + đã confirm style** (xem `system_prompt.md` mục 4 và `O_invest_memo_00.md` Nguyên tắc 1-2-3). **Body font: Roboto** (fallback Open Sans → Arial). MD final là source of truth — binary derive từ MD, sửa nội dung phải sửa MD trước rồi re-render.

**Layout 10-15 slide:**

| Slide | Nội dung |
|---|---|
| 1 | Cover — ngày + regime + portfolio size |
| 2 | Executive summary — % deployment + conviction mix + ngành concentration |
| 3 | Allocation chart pie + bảng rút gọn top 5 mã |
| 4 | Constraint check status |
| 5-7 | Thesis mỗi slide 2-3 mã |
| 8 | Sequence entry timeline T+0 đến T+4 |
| 9 | Phase 2 conditions table |
| 10 | Limit orders (bảng chính) |
| 11 | Order list block copy (có thể slide monospace) |
| 12 | Cảnh báo & lựa chọn sát nút |
| 13 | Next step — monitoring |

Pptx portfolio plan **ít dùng hơn** so với stock memo pitch — chủ yếu dành cho formal presentation cho sếp/committee duyệt allocation. MD và docx thường đủ cho use case đặt lệnh.

## 7. Xử lý rebalance mid-cycle

Portfolio plan có thể được render lại giữa cycle khi:
- Regime shift (cần giảm/tăng exposure)
- Concentration drift (1 mã vượt 10% do win)
- Sector rotation (1 ngành deteriorate, rebalance sang ngành khác)
- Phase 2 confirm (Bucket 2 vào thêm tranche)
- Phase 3 upgrade (Bucket 3 chuyển thành position mới)

Mỗi lần render rebalance:
- **Version:** increment (1.0 → 1.1, 2.0, v.v.) ghi trong metadata
- **Changelog block** ở phần 1 executive summary:

```
**Thay đổi so với version trước (v[X.Y], ngày [prev date]):**
- [Hành động 1: giảm/tăng/thêm/bỏ mã Z]
- [Hành động 2: ...]
- [Lý do: regime shift / drift / sector rotation / confirm Phase 2]
```

- **Order list** ở phần 7 chỉ chứa orders mới (delta), không repeat full portfolio

## 8. Ví dụ fragment

```markdown
# Portfolio Plan — 22/04/2026

## 1. Executive summary

**Ngày plan:** 22/04/2026  
**Portfolio size total:** 23 tỷ VND (~1 triệu USD)  
**Regime (trạng thái thị trường):** Risk-on selective (ưa rủi ro chọn lọc)  
**Phase 1 deployment:** 14% portfolio — 8 positions  
**Cash buffer sau Phase 1:** 86%  
**Full potential sau Phase 2:** 29% — cash buffer long-term 71%

**Phân bổ theo conviction tier:**
- High (cao): 4 positions, 8.8% total
- Medium (trung bình): 3 positions, 3.5% total
- Low (thấp): 1 position, 1.7% total
- Bucket 3 (watchlist, chưa vào): 1 position, 0% hiện tại

**Top concentration:**
- Mã max: VNM 2.9% portfolio
- Ngành max: Tiêu dùng thiết yếu 6.5% portfolio

...

## 7. Order list — block copy cho broker

```text
PHASE 1 ORDERS — 22/04/2026

T+0:
  VNM | BUY | LIMIT 72.0 | qty 11000cp | = 40% target (Bucket 1 High)
  VNM | BUY | LIMIT 70.5 | qty 8000cp  | = 30% target
  VNM | BUY | LIMIT 69.0 | qty 8000cp  | = 30% target
  HPG | BUY | LIMIT 28.5 | qty 20000cp | = 40% target (Bucket 1 High)
  ...

T+1:
  VCB | BUY | LIMIT 92.0 | qty 3000cp | = 40% target (Bucket 2 High first tranche)
  ...

Cancel condition: unfilled orders > 5 phiên → cancel + reassess
```
```

## 9. Checklist self-check cuối cùng

- [ ] State tier 6 đã có và confirmed
- [ ] Memo tier 5C đã có cho phần lớn mã (flag mã thiếu)
- [ ] Executive summary có: portfolio size + regime + deployment + conviction mix + top concentration
- [ ] Allocation bảng đầy đủ 13 cột, hàng cuối có tổng
- [ ] Constraint check 8 items đầy đủ với status
- [ ] Phần 4 mã thesis: mỗi mã có đủ Conviction + target + thesis + catalyst + bucket + risk
- [ ] Sequence entry phân rõ Phase 1 (T+0 đến T+4) + Phase 2 condition + Phase 3 upgrade condition
- [ ] Limit orders bảng và block copy **số liệu khớp nhau** (cùng limit price + quantity)
- [ ] Quantity block copy round lot 100 cp
- [ ] Sum % Phase 1 = Phase 1 deployment ở executive summary (không lệch)
- [ ] Portfolio size total matched mọi chỗ xuất hiện
- [ ] Thuật ngữ Conviction/Bucket/Phase/Regime có dịch lần đầu
- [ ] Zone/trend/condition mô tả tự nhiên, không DB raw
- [ ] Citation: agent_db "(nguồn: Tổng hợp)", tin finext link đầy đủ, BCTC PDF tên + trang, web external markdown link
- [ ] Metadata đầy đủ: ngày, regime, portfolio size, state file nguồn, memo link, audit log (nếu có)
- [ ] Nếu là rebalance: version increment + changelog block ở exec summary
- [ ] Format cuối đúng yêu cầu + template đúng
