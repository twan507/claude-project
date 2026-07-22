# O_invest_memo_04 — Weekly Review

Spec render báo cáo review danh mục hàng tuần. **Rigid structure** (theo master mục 8) — user đọc 12-24 lần/cycle, cần format nhất quán để scan nhanh.

Reference: `O_invest_memo_00` (master rules), `P_invest_memo_09` tier 7 weekly workflow (mục 3.2 + mục 10 template Weekly Review Session).

## 1. Input state & mapping

**State file chính:**

| State file | Role |
|---|---|
| `tier7_weekly_<YYYYMMDD>.md` | Weekly review output — position health, soft trigger, Phase 2/3 management, take profit |

**State file tham khảo (cho context):**

- `tier6_portfolio_<YYYYMMDD>_confirmed.md` mới nhất — để biết allocation plan và Phase 2/3 conditions setup
- `tier5C_<ticker>_*_confirmed.md` của các mã active — để biết target Base/Bull/Bear và exit triggers

Tier 7 weekly state thường đã đủ data render, 2 state phụ dùng để resolve ambiguity nếu có.

## 2. Structure báo cáo đầu ra

**Độ dài target:** 1-2 trang MD. Nếu tuần không có alert nào → có thể ngắn 0.5-1 trang.

**Rigid structure bắt buộc** (thứ tự và heading exact):

```
# Weekly Review — <Ngày>

## 1. Alert level
## 2. Position health
## 3. Triggers matched tuần này
## 4. Phase 2 / Phase 3 review
## 5. Portfolio summary
## 6. Actions needed
— Metadata
```

Mọi weekly review phải có đầy đủ 6 section, kể cả khi section rỗng (viết "Không có" thay vì skip section).

## 3. Compose từng phần chi tiết

### Phần 1 — Alert level

Format cứng:

```
**Tuần:** [DD/MM/YYYY] đến [DD/MM/YYYY]  
**Ngày review:** [DD/MM/YYYY]  
**Alert level:** [Green / Yellow / Red]

**Trigger summary:**
- Hard triggers matched: [N]
- Soft triggers matched: [N]
- Take profit available: [N]
- Phase 2 confirm available: [N]
- Phase 3 upgrade available: [N]

**Positions active:** [N] | **Phase 2 pending:** [N] | **Phase 3 watchlist:** [N]
```

**Alert level rule:**
- **Red:** có ≥ 1 hard trigger matched → action khẩn
- **Yellow:** có soft trigger hoặc Phase 2/3 available nhưng không hard trigger
- **Green:** không trigger nào match, position bình thường

### Phần 2 — Position health

Bảng cố định cột:

```
| # | Ticker | Size % | Entry avg | Current | % vs entry | % vs Base | % vs Bear | Status |
|---|---|---|---|---|---|---|---|---|
| 1 | [X] | 2.9% | 71.2k | 76.5k | +7.4% | 90% | +32% | Normal |
| 2 | [Y] | 2.5% | 73.0k | 80.0k | +9.6% | 94% | +38% | Near Base |
| 3 | [Z] | 1.7% | 62.5k | 57.8k | -7.5% | 77% | +15% | Pullback |
| ... | ... | ... | ... | ... | ... | ... | ... | ... |
```

**Status values (standardized):**
- `Normal` — trong range bình thường, không gần trigger nào
- `Near Base` — price ≥ 90% Base target, gần take profit 1
- `Near Bull` — price ≥ 95% Bull target, gần take profit 2
- `Pullback` — price giảm 5-15% từ entry, vẫn trong range thesis
- `Warning` — price approach Bear × 0.95 (gần hard trigger)
- `Alert` — hard trigger đã match, cần action ngay

**Sort order:** theo Status (Alert → Warning → Near Base → Near Bull → Pullback → Normal), rồi theo Size % giảm dần.

**Bucket 3 watchlist:** render cuối bảng, Size = 0%, Status = `Watchlist`, các cột price giữ, cột "% vs entry" để trống.

### Phần 3 — Triggers matched tuần này

3 sub-section theo loại trigger:

```
### 3.1. Hard triggers matched — [N]

[Nếu N = 0: "Không có hard trigger nào match tuần này."]

[Nếu N ≥ 1: bullet từng trigger]
- **[Ticker X]** — Hard trigger [loại] matched ngày [DD/MM]. Source: [price/news/BCTC]. **Proposal: BÁN NGAY [X]% position.** Thesis [collapsed/major risk materialized]. Action needed ngay.

### 3.2. Soft triggers matched — [N]

[Nếu N = 0: "Không có soft trigger nào match tuần này."]

[Nếu N ≥ 1]
- **[Ticker Y]** — Soft trigger [loại, ví dụ "rank dòng tiền thị trường giảm sâu 2 tuần"]. Thesis impact: [partial/moderate/major]. Proposal (user chọn):
  - (a) Giảm size 30-50%
  - (b) Tighten hard trigger (Bear × 0.9 → Bear × 0.95)
  - (c) Giữ nguyên, review lại tuần sau

### 3.3. Take profit triggers — [N]

[Nếu N = 0: "Không có take profit nào match."]

[Nếu N ≥ 1]
- **[Ticker Z]** — Price đạt Base [X]k. Proposal bán 50% position ([Y]% portfolio = [Z] triệu VND).
- **[Ticker W]** — Price đạt Bull [X]k. Proposal bán thêm 30%, còn 20% position.
```

### Phần 4 — Phase 2 / Phase 3 review

```
### 4.1. Phase 2 confirm available — [N]

[Nếu N = 0: "Không có Bucket 2 nào confirm available tuần này."]

[Nếu N ≥ 1]
- **[Ticker X]** — Bucket 2 (chờ pullback xác nhận) đã confirm. Condition match: [chi tiết zone, flow, volume theo K hygiene]. **Proposal vào tranche 2 [Y]%.** Limit orders split:
  - Order 1 (50%): current price [Z]k
  - Order 2 (50%): current × 0.98 = [W]k

### 4.2. Phase 2 timeout gần — [N]

[Liệt kê mã Bucket 2 đã qua 3+ tuần chưa confirm]
- **[Ticker Y]** — Phase 1 vào ngày [DD/MM], đã [X] tuần. Chưa confirm condition: [chi tiết chưa match]. Zone quý/năm vẫn [mô tả]. Proposal: [extend 1-2 tuần / abort 50-70% / abort 100% theo rule P_invest_memo_09 mục 5.2].

### 4.3. Phase 3 upgrade available — [N]

[Nếu N = 0: "Không có Bucket 3 nào upgrade available."]

[Nếu N ≥ 1]
- **[Ticker Z]** — Bucket 3 (watchlist, chưa vào) đã chuyển sang Bucket 2. Condition match: [chi tiết]. Target size từ portfolio plan: [Y]%. Proposal: vào Phase 1 tranche đầu [Y × 0.4]% portfolio.
```

### Phần 5 — Portfolio summary

Format:

```
**Total P&L tuần này:** [+/-X]%  
**Total P&L vs entry toàn portfolio:** [+/-Y]%  
**Cash current:** [Z]% portfolio  
**Concentration check:**
- Mã max % portfolio: [Ticker] [X]% — [within cap 10% / near cap / over cap 10%]
- Ngành max % portfolio: [Ngành] [Y]% — [within cap / near cap]
```

**Nếu có concentration drift over cap:** flag rõ + proposal trim 20-30% xuống dưới cap (link sang phần 6).

**Chart đề xuất (chỉ khi Alert level Red/Yellow):**

Chart 1 — Position health visual (bar chart % vs Base target):

````
```chart
type: bar
title: Position Progress — Tuần [DD/MM]
x_axis: [list ticker active]
y_axis:
  - name: "% toward Base target"
    data: [list giá trị % vs Base]
y_label: '%'
source: Tổng hợp
render_in_md: skip
render_in_docx: true
render_in_pptx: false
note: 100% = đạt Base target, >100% = vượt Base, <0% = dưới entry
```
````

Alert level Green: không chart, weekly ngắn gọn.

### Phần 6 — Actions needed

**Đây là phần quan trọng nhất cho actionable.** User đọc xong phần này phải biết cần làm gì trong tuần sau.

Format bullet list câu hỏi rõ ràng, user trả lời từng cái:

```
**Actions cần user quyết:**

- [ ] (a) Confirm execute hard trigger [Ticker X]? [Lý do execute + impact]
- [ ] (b) Quyết soft trigger [Ticker Y] — chọn proposal (a/b/c)?
- [ ] (c) Confirm take profit 50% [Ticker Z] tại [X]k?
- [ ] (d) Confirm Phase 2 tranche 2 [Ticker A] — vào thêm [X]%?
- [ ] (e) Quyết Phase 2 timeout [Ticker B] — extend 2 tuần / abort 50% / abort 100%?
- [ ] (f) Confirm Phase 3 upgrade [Ticker C] — vào Phase 1 tranche đầu?
- [ ] (g) Trim concentration [Ticker D] từ [X]% xuống 7-8% (nếu over cap)?
```

**Nếu không có action nào:**

```
**Actions cần user quyết:**

Không có action bắt buộc tuần này. Review tiếp thứ Hai tuần sau.
```

### Metadata cuối file

```
---

**Metadata**

- **Tuần:** [DD/MM/YYYY] đến [DD/MM/YYYY]
- **Ngày review:** [DD/MM/YYYY]
- **Alert level:** [Green/Yellow/Red]
- **State file nguồn:** `tier7_weekly_<YYYYMMDD>.md`
- **Portfolio plan version hiện tại:** [X.Y từ tier6_portfolio_...md]
- **Next review:** Thứ Hai [DD/MM/YYYY]
```

## 4. Compose workflow step-by-step

**Bước 1 — Xác định format + template:** hỏi MD/docx/pptx (weekly thường chỉ MD, docx nếu cần formal share, pptx hiếm).

**Bước 2 — Load state:** `tier7_weekly_*` mới nhất + reference `tier6_portfolio_*` và các `tier5C_<ticker>_*` active positions.

**Bước 3 — Extract trigger count** từ state → xác định Alert level (Red/Yellow/Green).

**Bước 4 — Build phần 2 Position health bảng** từ state position health per-stock + cross-check target giá với tier 5C memo của mã.

**Bước 5 — Compose phần 3 Triggers matched** theo 3 loại (hard/soft/take profit). Với hard trigger → action rõ "BÁN NGAY", không soft-pedal.

**Bước 6 — Compose phần 4 Phase 2/3 review.** Cross-check với `tier6_portfolio_*` để biết condition setup gốc.

**Bước 7 — Tính Portfolio summary phần 5** (P&L tuần + total + cash + concentration).

**Bước 8 — Build phần 6 Actions needed** từ triggers + Phase confirm. Checkbox format cho user dễ trả lời.

**Bước 9 — Chart (nếu Red/Yellow).**

**Bước 10 — Self-check:**
- Bảng position health sort đúng (Alert/Warning trước)
- Trigger count ở phần 1 khớp số trigger list ở phần 3-4
- Mã trong Phase 2/3 đều có trong portfolio plan
- Ticker uppercase, % 1 decimal, VND tỷ/k
- Thuật ngữ Bucket/Phase/Regime lần đầu có dịch (weekly có thể skip dịch nếu đã review nhiều tuần, nhưng để an toàn, giữ dịch lần đầu xuất hiện trong mỗi report)
- Citation: data định lượng có "(nguồn: Tổng hợp)"

**Bước 11 — Render format cuối + present file.**

## 5. Guide render docx

> **Render khi user explicit yêu cầu + đã confirm style** (xem `system_prompt.md` mục 4 và `O_invest_memo_00.md` Nguyên tắc 1-2-3). **Body font: Roboto** (fallback Open Sans → Arial). MD final là source of truth — binary derive từ MD, sửa nội dung phải sửa MD trước rồi re-render.

Weekly docx ít dùng, chỉ khi cần archive formal hoặc share meeting thứ Hai.

**Layout:**
- Cover hoặc header compact (weekly không cần full cover page)
- Phần 1-6 theo thứ tự, mỗi phần 1 heading
- Bảng position health + Phase 2/3 render nguyên từ markdown
- Checkbox action list giữ nguyên format

**Template cần có:**
- Heading + body + table style (minimal)
- Footer compact: page + "Weekly [ngày]"

## 6. Guide render pptx

> **Render khi user explicit yêu cầu + đã confirm style** (xem `system_prompt.md` mục 4 và `O_invest_memo_00.md` Nguyên tắc 1-2-3). **Body font: Roboto** (fallback Open Sans → Arial). MD final là source of truth — binary derive từ MD, sửa nội dung phải sửa MD trước rồi re-render.

Weekly pptx **rất hiếm** — monitor operational không pitch được. Chỉ build pptx khi user explicit yêu cầu cho meeting tuần.

Nếu có: 4-6 slide là đủ.

| Slide | Nội dung |
|---|---|
| 1 | Cover — Tuần + Alert level badge |
| 2 | Position health summary (bảng rút gọn top 5 mã) |
| 3 | Triggers matched (nếu Red/Yellow) |
| 4 | Phase 2/3 actions |
| 5 | Portfolio summary + concentration |
| 6 | Actions needed cho user |

## 7. Ví dụ fragment

```markdown
# Weekly Review — 22/04/2026

## 1. Alert level

**Tuần:** 15/04/2026 đến 22/04/2026  
**Ngày review:** 22/04/2026  
**Alert level:** Yellow

**Trigger summary:**
- Hard triggers matched: 0
- Soft triggers matched: 1
- Take profit available: 1
- Phase 2 confirm available: 1
- Phase 3 upgrade available: 0

**Positions active:** 8 | **Phase 2 pending:** 3 | **Phase 3 watchlist:** 1

## 2. Position health

| # | Ticker | Size % | Entry avg | Current | % vs entry | % vs Base | % vs Bear | Status |
|---|---|---|---|---|---|---|---|---|
| 1 | VNM | 2.9% | 71.2k | 80.5k | +13.1% | 95% | +39% | Near Base |
| 2 | HPG | 2.5% | 28.5k | 26.8k | -6.0% | 84% | +18% | Pullback |
| 3 | VCB | 1.7% | 92.0k | 88.5k | -3.8% | 84% | +12% | Pullback |
| ... | ... | ... | ... | ... | ... | ... | ... | ... |

...

## 6. Actions needed

**Actions cần user quyết:**

- [ ] (a) Confirm take profit 50% VNM tại 85k? Current 80.5k, còn 5.6% tới Base.
- [ ] (b) Quyết soft trigger MWG — rank dòng tiền thị trường giảm 2 tuần liên tiếp. Proposal: (a) giảm 30% size / (b) tighten stop / (c) giữ review tuần sau?
- [ ] (c) Confirm Phase 2 tranche 2 HPG — vào thêm 1.3%, limit split 2 orders?
```

## 8. Checklist self-check cuối cùng

- [ ] State `tier7_weekly_*` đã có
- [ ] Alert level xác định đúng (Red nếu có hard trigger)
- [ ] 6 section đầy đủ kể cả section rỗng
- [ ] Bảng position health sort theo Status priority
- [ ] Trigger count ở phần 1 khớp phần 3
- [ ] Mỗi hard trigger có action "BÁN NGAY" rõ ràng
- [ ] Soft trigger có 3 proposal (a/b/c) cho user chọn
- [ ] Phase 2/3 mã match với portfolio plan setup
- [ ] Concentration check có flag nếu over cap
- [ ] Actions needed checkbox format
- [ ] Thuật ngữ Phase/Bucket lần đầu có dịch
- [ ] Zone/flow mô tả tự nhiên
- [ ] "(nguồn: Tổng hợp)" cho data agent_db
- [ ] Metadata đầy đủ
- [ ] Format cuối đúng yêu cầu
