# O_vbse_strategy_00 — Render Spec Báo cáo VBSE Strategy

Spec render báo cáo chiến lược đầu tư VN theo 2 mode: **Monthly** (parent, đầu tháng) và **Weekly update** (child, tracking trong tháng). **Flex structure** — 6 trục cốt lõi cố định, độ sâu và sub-section flex theo phát hiện thực tế. **Output cuối: MD final.**

Reference: `P_vbse_strategy_00` (master philosophy + weight balance + Nguyên tắc bất biến + file index).

> **Render binary:** MD final là source of truth. Khi user yêu cầu pptx/docx, agent render theo style đã chọn (O pack spec + branding info + user explicit, body font Roboto) — chi tiết ở `system_prompt.md` mục 4. Pack này chưa có pptx/docx spec riêng — sẽ derive từ MD structure hiện tại + style chuẩn institutional khi user yêu cầu.

## 1. Input từ P pack

P pack sinh structured content theo 6 trục (master `P_vbse_strategy_00` mục 5.3) cộng status mode-specific. O pack chỉ render, không thêm/bớt nội dung.

**Trường hợp đặc biệt:**
- Monthly không có file N-1 → bỏ phần Review hit rate
- Weekly mode bắt buộc có file monthly upload — nếu thiếu, P pack không chạy (HARD GATE), O pack không có gì render
- Checkpoint regime override → ghi inline note trong trục liên quan
- Branding info user cung cấp → render branded header + disclaimer; không có → plain header

## 2. Structure báo cáo — Mode Monthly

**Độ dài target:** 8-12 trang MD.

**Structure flex** (suggested, KHÔNG rigid):

```
# Báo cáo chiến lược đầu tư VBSE — Tháng [N/YYYY]

## Tóm tắt điều hành
## [Review tháng trước]            (skip nếu lần đầu)
## Trục 1 — Môi trường vĩ mô & tài chính
## Trục 2 — Định vị thị trường VN
## Trục 3 — Themes & narratives chính
## Trục 4 — Sector allocation (18 ngành whitelist)
## Trục 5 — Kịch bản & risk map
## Trục 6 — Watchlist (Phase 1 Screen + Phase 2 Bucket entry)
## Tuyên bố miễn trừ trách nhiệm
— Metadata
```

**Quy tắc flex:**
- Mỗi trục có 1 H2 chính. Sub-section H3 tự agent quyết theo phát hiện
- Trục Hold ghi 3-5 dòng, KHÔNG ép viết dài
- Trục có signal mạnh có thể chiếm 1-2 trang đào sâu
- Có thể merge 2 trục nếu ranh giới mờ — báo trong tên H2 (vd "Trục 1+3 — Vĩ mô & Theme chi phối")

## 3. Compose từng phần (Monthly)

### 3.1. Header

**Format plain (không branding):**

```
# Báo cáo chiến lược đầu tư VBSE — Tháng [N/YYYY]

**Phát hành:** [DD/MM/YYYY]  
**Phạm vi:** Thị trường cổ phiếu Việt Nam (VNINDEX, HSX/HNX/UPCOM) — 18 ngành whitelist  
**Horizon ưu tiên:** [1 tháng / 1-3 tháng / 3-6 tháng]  
**Audience:** Phân tích nội bộ
```

**Format branded:**

```
# [TÊN CÔNG TY]
## BÁO CÁO CHIẾN LƯỢC ĐẦU TƯ VBSE
### Tháng [N/YYYY]

**Phát hành:** [DD/MM/YYYY]  
**Phòng ban biên soạn:** [user cung cấp]  
**Horizon ưu tiên:** [1 tháng / 1-3 tháng / 3-6 tháng]  
**Hotline:** [user cung cấp] | **Website:** [user cung cấp]

---

> [Disclaimer ngắn user cung cấp, hoặc default: "Tài liệu nhằm mục đích thông tin tham khảo, không phải khuyến nghị mua bán chứng khoán. Khách hàng cần tự cân nhắc trước khi quyết định đầu tư."]

---
```

### 3.2. Tóm tắt điều hành

Format bullet, 3-7 bullet, mỗi bullet 1-2 dòng:

```
## Tóm tắt điều hành

- **Regime vĩ mô:** [định tính — vd "Đầu chu kỳ nới lỏng còn kéo dài 1-2 quý tới"]. **Conviction:** HIGH / MID / LOW. [1 dòng căn cứ chính.]
- **Định vị thị trường VN:** [**định tính fundamental-first** — vd "Định giá P/E 11.2x phân vị 32% (rẻ tương đối) + dòng tiền aggregate 18 ngành whitelist cải thiện 4 tuần + FII chuyển mua ròng → phục hồi sớm có cơ sở fundamental"]. [1 dòng căn cứ.]
- **Top themes tháng [N]:** [Theme 1 (HIGH, 1-3m) + Theme 2 (MID, 3-6m) + Theme 3 (LOW, 1m) — tên ngắn + conviction + horizon]
- **Sector bias (18 ngành whitelist):** Quan tâm [3-5 ngành, kèm conviction marker từng ngành nếu khác nhau]. Thận trọng [1-3 ngành].
- **Risk chính:** [1-2 rủi ro nổi bật từ risk map, kèm signal materialize cụ thể nhất]
- **Watchlist tiêu biểu:** [1-3 mã đại diện theme chính — kèm conviction + horizon + **Bucket entry (1/2/3)**]
- **PM overlay note:** [optional — chỉ render nếu có user view inject ở conviction HIGH hoặc trạng thái Conflict chưa resolve]
```

### 3.3. Review tháng trước (Stage 0 Evaluation)

**3 trường hợp render:**

**(a) Full eval — user chạy Stage 0 + accept:** render đầy đủ 6 phần eval

```
## Review tháng trước — Stage 0 Evaluation

> *Đánh giá chiến lược tháng N-1 đã được user review và accept tại Checkpoint 0 (DD/MM/YYYY). Learning được carry-forward vào thesis tháng [N].*

### Regime evaluation

**Regime call tháng N-1:** [định tính] (Conviction tháng N-1: HIGH/MID/LOW) — Đánh giá: **đúng / lệch nhẹ / sai rõ**

[2-4 dòng so call cũ vs actual: vĩ mô tháng N-1 thực tế khớp / lệch ra sao]

### Themes evaluation

| # | Theme N-1 | Conviction cũ | Catalyst trigger | Trạng thái thực tế | Hit/Miss |
|---|---|---|---|---|---|
| 1 | [Theme A] | HIGH | [tên catalyst] | Materialize / Partial / Fizzle / Disconfirming triggered | Hit ✓ |
| 2 | [Theme B] | MID | [tên catalyst] | [...] | Miss ✗ |

### Sector tilts evaluation (18 ngành whitelist)

**Ngành quan tâm tháng N-1:**

| Ngành | Conviction cũ | m_pct thực tế | Hit/Miss |
|---|---|---|---|
| [Ngành A] | HIGH | +5.2% | Hit ✓ |

**Ngành cần thận trọng tháng N-1:**

| Ngành | Conviction cũ | m_pct thực tế | Hit/Miss |
|---|---|---|---|
| [Ngành X] | HIGH | -3.8% | Hit ✓ (giảm như dự đoán) |

**Hit rate tổng:** [N/M] ngành quan tâm tăng giá, [N/M] ngành thận trọng giảm giá.

### Watchlist evaluation

| Ticker | Theme cũ | Conviction cũ | Bucket cũ | Horizon cũ | m_pct thực tế | Signal trigger? | Disconfirming? | Hit/Miss |
|---|---|---|---|---|---|---|---|---|
| [Ticker A] | [Theme A] | HIGH | 1 | 1-3m | +12% | Có (BCTC Q1 EPS +25%) | Không | Hit ✓ |
| [Ticker B] | [Theme B] | MID | 2 | 1m | -8% | Không | Có (BCTC miss 12%) | Miss ✗ |

**Hit rate watchlist:** [N/M] mã chạy đúng luận điểm. **Bucket accuracy:** [N/M] Bucket 1 vào nhanh được, [N/M] Bucket 2 confirm trong 4 tuần.

### Risk map evaluation

- **Rủi ro materialize tháng N-1:** [...]
- **Rủi ro còn nguyên (carry-forward tháng [N]):** [...]
- **Rủi ro mới agent đã miss (phát hiện sau):** [honesty review]

### Calibration learning

- **Best call tháng N-1:** [...] — [1-2 dòng]
- **Worst call tháng N-1:** [...] — [honest, gắn với pitfall methodology nếu có]
- **Carry-forward vào tháng [N]:** [2-3 dòng learning chính]
```

**(b) Short review — user skip Stage 0 nhưng có file N-1:** render ngắn không cần checkpoint

```
## Review tháng trước

**Regime call tháng N-1:** [đúng/lệch/sai]. [1 dòng]
**Hit rate themes:** [N/M] materialize, [N/M] còn nguyên, [N/M] fizzle
**Hit rate sector bias:** [N/M] quan tâm tăng giá, [N/M] thận trọng giảm giá
**Hit rate watchlist:** [N/M] mã chạy đúng luận điểm
**Best call:** [...] — [1 dòng]
**Worst call:** [...] — [1 dòng honest]
**Rủi ro materialize:** [list / không có]
**Learning chính cho tháng [N]:** [1-3 dòng]
```

**(c) Skip entirely — không có file N-1:**

```
## Review tháng trước

Lần đầu chạy monthly cycle / không có file N-1, chưa có dữ liệu review tháng trước.
```

### 3.4. Trục 1 — Môi trường vĩ mô & tài chính

Sub-section flex. Reference spec `P_vbse_strategy_01`.

```
## Trục 1 — Môi trường vĩ mô & tài chính

[Prose mở 3-5 dòng — kết luận regime vĩ mô tổng thể, lý do]

### 1.1. Chính sách tiền tệ & lãi suất
[Bảng lãi suất điều hành VN + Fed/ECB/PBOC + biến động tháng + diễn giải 3-5 dòng]

### 1.2. Tỷ giá & dòng vốn
[Bảng tỷ giá USD/VND + EUR/USD + USD/CNY + DXY + FII net flow + diễn giải]

### 1.3. Vĩ mô thực
[Bảng/list CPI, GDP, PMI, XNK, FDI, bán lẻ + diễn giải tác động equity]

### 1.4. Hàng hoá cross-sector
[Bỏ sub-section nếu tháng không có biến động commodity đáng kể]

### 1.5. Kết luận regime vĩ mô
[2-4 dòng — regime ở giai đoạn nào, kỳ vọng 1-3 tháng tới]
```

Trục Hold (vĩ mô ổn định): rút gọn còn 1 prose 5-8 dòng.

### 3.5. Trục 2 — Định vị thị trường VN (FUNDAMENTAL-FIRST)

Reference spec `P_vbse_strategy_02`. **Cap technical ≤ 20%** — câu chốt định vị mở đầu bằng PRIMARY (định giá + flow + FII + breadth), không bằng technical.

```
## Trục 2 — Định vị thị trường VN trong chu kỳ

### 2.1. Định giá phân vị lịch sử (PRIMARY)

[Bảng/prose: P/E VNINDEX hiện tại vs median 3-5Y, phân vị lịch sử cụ thể (vd "P/E 11.2x ở phân vị 28% so median 5Y"); P/B tương tự; P/E forward nếu có]

[Diễn giải 2-3 dòng: phân vị < 30% → rẻ tương đối → re-rating opportunity; phân vị > 70% → đắt → de-rating risk]

### 2.2. Dòng tiền cấp thị trường (PRIMARY — aggregate 18 ngành whitelist)

[Xu hướng mean/median week_score qua 18 ngành whitelist 4-8 tuần gần nhất — pattern]
[Trend chart hoặc bảng so sánh tuần qua tuần]
[**Lưu ý**: aggregate trên 18 ngành whitelist, KHÔNG 24]

### 2.3. Khối ngoại tháng/quý (PRIMARY)

[Bảng net flow tháng + xu hướng quý — có break trend không]

### 2.4. Breadth fundamental + giá (PRIMARY)

[% ngành whitelist 18 tăng giá tháng]
[% ngành whitelist 18 có EPS Q YoY > 0 — breadth fundamental]
[% mã universe tăng giá tháng/quý + % mã trên MA60/MA120]

### 2.5. Sentiment & thanh khoản (SECONDARY)

[Thanh khoản trung bình tháng vs 6 tháng + breadth phiên cross-check]

### 2.6. Kết luận định vị (Fundamental-first sentence)

[Câu chốt mở đầu bằng PRIMARY: "Định giá phân vị X% + dòng tiền aggregate ... + FII ... + breadth ... → định vị [phục hồi sớm / quá mua / phân phối / suy yếu] có cơ sở fundamental"]

[Optional ≤2 dòng: "Kèm minh hoạ kỹ thuật: technical_zone w/m/q/y = ... — đồng thuận / mâu thuẫn với định vị PRIMARY"]
```

### 3.6. Trục 3 — Themes & narratives chính

Reference spec `P_vbse_strategy_03`. 2-5 themes, mỗi theme 5 thành phần.

```
## Trục 3 — Themes & narratives chính

[2-5 themes — không ép số.]

### 3.1. Theme A — [Tên ngắn gọn]

**Conviction:** HIGH / MID / LOW   |   **Horizon:** 1m / 1-3m / 3-6m   |   **Ngành/mã liên quan:** [3-7 ngành whitelist + mã đại diện]

**Cơ chế:** [3-5 dòng — mạch logic nguyên nhân → hệ quả]

**Catalyst trigger:** [sự kiện / mức số / chính sách cụ thể; ngày dự kiến nếu có]

**Disconfirming signals (kill criteria):**
- [Signal cụ thể 1 PREFER cơ bản/chính sách/catalyst]
- [Signal cụ thể 2]
- [Signal cụ thể 3, optional]

### 3.2. Theme B — [Tên]
...
```

**Sau danh sách themes:**

```
### Catalysts ahead — Lịch tổng hợp

| Ngày dự kiến | Sự kiện | Theme tương ứng | Hướng tác động |
|---|---|---|---|
| Thứ Tư DD/MM | [FOMC minutes / CPI VN / BCTC mã X] | Theme A | Tích cực / Tiêu cực |
```

User overlay badge sau tên theme (nếu có):
> *Theme này có user overlay: `[Synthesized from PM input + data confirm]` — [chi tiết]. Xem mục audit trail.*

### 3.7. Trục 4 — Sector allocation (18 ngành whitelist)

Reference spec `P_vbse_strategy_04`. **Cap technical ≤ 5%** — bảng tilts KHÔNG có cột technical_zone làm decision factor.

```
## Trục 4 — Sector allocation (18 ngành whitelist)

### 4.1. Bảng cross-section 18 ngành whitelist

| Ngành | Biến động tháng | Dòng tiền tháng (week_score mean 4 tuần) | P/E hiện tại vs median 3Y | EPS Q YoY | % mã trong ngành tăng tháng | Industry rank (re-rank 18) | Bias |
|---|---|---|---|---|---|---|---|
| NGANHANG | +4.2% | +8.1 | 14x vs 18x (phân vị 28%) | +18% | 78% | 3/18 | Quan tâm |
| BANLE | ... | ... | ... | ... | ... | ... | ... |

[Render đủ 18 ngành whitelist — không có ngành ngoài whitelist]

(Bias: Quan tâm / Trung tính / Cần thận trọng)

### 4.2. Sector tilts table — bảng tổng hợp (chuẩn buy-side single-page scannable)

| Ngành | Bias | Conviction | Driver chính (cơ bản/chính sách/vĩ mô) | Signal hỗ trợ (số, prefer cơ bản) | Disconfirming signal (prefer cơ bản/macro/chính sách) |
|---|---|---|---|---|---|
| NGANHANG | Quan tâm | HIGH | Theme A — margin cải thiện cuối chu kỳ hạ lãi suất | EPS Q1 +18% YoY, NIM +20bp QoQ, P/B 1.4x phân vị 28% | NIM Q2 thu hẹp ≥20bp QoQ; chính sách thắt tín dụng mới |
| BANLE | Quan tâm | HIGH | Theme B — phục hồi tiêu dùng | Bán lẻ HH&DV YoY +12%, EPS Q1 +20%, P/E 14x phân vị 25% | Bán lẻ HH&DV YoY < 6% trong 2 tháng |
| BDS | Cần thận trọng | HIGH | Chính sách thắt tín dụng BĐS + pre-sales yếu | Pre-sales 3 tháng giảm >25% YoY, NPL BĐS tăng | NHNN nới tín dụng BĐS hoặc lãi suất cho vay BĐS giảm > 0.8% |
| ... (đủ 18 ngành) ... | ... | ... | ... | ... | ... |

**Lưu ý cột:** KHÔNG có cột "Technical zone" hoặc tương tự — sector bias không bị technical quyết. Technical timing để Trục 6 Phase 2 Bucket entry.

### 4.3. Phân tầng (chi tiết)

**Ngành quan tâm ([N] ngành):**
- **[Ngành A]** — [3-5 dòng: theme nào dẫn dắt, cơ chế vĩ mô hỗ trợ, dòng tiền tháng + xu hướng, định giá tương đối, dẫn dắt thật vs trụ kéo, signal cảnh báo nếu có]

**Ngành trung tính ([N] ngành):**
- **[Ngành C]** — [2-3 dòng: signal hỗn hợp, không có theme]

**Ngành cần thận trọng ([N] ngành):**
- **[Ngành X]** — [3-5 dòng: vĩ mô áp lực / định vị xấu / định giá quá cao / theme tiêu cực]

### 4.4. Crowding & rotation note (optional)

[Crowding check + rotation note nếu có]
```

### 3.8. Trục 5 — Kịch bản & risk map

Reference spec `P_vbse_strategy_05`. **Cap technical = 0%** — trigger primary là macro/fundamental/policy ONLY.

```
## Trục 5 — Kịch bản & risk map

### 5.1. Ba kịch bản VNINDEX

**Trigger primary BẮT BUỘC là macro / fundamental / chính sách / catalyst** (theo Weight balance `P_vbse_strategy_00` mục 4). Technical chỉ là **confirmation phụ optional**, không phải primary trigger.

**Kịch bản cơ sở:**
- **Trigger duy trì (macro/fundamental/policy primary):** [vd "Fed giữ rates + BCTC Q1 toàn thị trường tăng trưởng EPS YoY 8-12% như consensus + không có shock chính sách"]
- **Confirmation phụ (optional):** [vd "VNINDEX dao động trong band POC quý"]
- **Vùng VNINDEX dự kiến 1-3 tháng:** [X] — [Y]
- **Hành vi kỳ vọng:** [định tính — vd "tích luỹ + sector rotation chậm theo earnings"]

**Kịch bản tích cực:**
- **Trigger primary:** [vd "Fed cắt 25bp + ECB dovish tone" / "BCTC Q1 ngân hàng beat consensus ≥10% NIM +20bp" / "Nghị quyết gói hỗ trợ tài khoá X nghìn tỷ thông qua"]
- **Vùng VNINDEX kỳ vọng:** [Y] (re-rating khi P/E expand)
- **Theme củng cố:** [theme nào được boost]

**Kịch bản tiêu cực:**
- **Trigger primary:** [vd "Fed hawkish surprise" / "BCTC Q1 nhiều top sector miss consensus" / "USD/VND vượt 26800 + NHNN can thiệp"]
- **Vùng hỗ trợ kỳ vọng:** [Y] (de-rating)
- **Risk concentration:** [sector/theme nào chịu impact lớn nhất]

> *Kịch bản là hệ thống điều kiện vĩ mô/cơ bản/chính sách, không phải dự báo chắc chắn. Không gán % xác suất.*

### 5.2. Risk map

**Rủi ro 1 — [Tên ngắn]**
- **Bản chất:** [2-3 dòng cơ chế cơ bản/vĩ mô/chính sách]
- **Signal materialize:** [chỉ báo cụ thể PREFER macro/fundamental]
- **Phản ứng định tính:** [vd "giảm exposure ngành X" / "chuyển defensive" / "đứng ngoài chờ"]
- **Theme bị invalidate (nếu có):** [theme nào fall apart]

[3-7 rủi ro, flex theo bối cảnh.]
```

### 3.9. Trục 6 — Watchlist (Phase 1 Screen + Phase 2 Bucket entry)

Reference spec `P_vbse_strategy_06`. **CẤU TRÚC 2 PHASE — đặc trưng nhất của pack này.**

```
## Trục 6 — High-conviction watchlist

[5-12 mã đại diện theme & sector bias. Phase 1 Screen cơ bản-only, Phase 2 Bucket entry PTKT-driven.]

### 6.1. Watchlist universe (Phase 1 — Cơ bản screen)

Bộ filter Phase 1: thanh khoản ≥ 5 tỷ ADV + ≥1 tiêu chí tăng trưởng cơ bản + ≥1 catalyst + định giá hợp lý. **Cấm technical filter.**

#### [Ticker A] (Ngành) — Theme: [tên theme]

- **Conviction:** HIGH / MID / LOW   |   **Horizon:** 1m / 1-3m / 3-6m
- **ADV tháng:** [X] tỷ
- **Bucket entry:** **1** / 2 / 3 (xem mục 6.2 dưới)

- **Luận điểm:** [observation 1-2 dòng — cơ chế theme + lý do mã hưởng lợi cụ thể về **cơ bản** (tăng trưởng/biên/chính sách hỗ trợ/catalyst), KHÔNG technical]

- **Signal theo dõi** (3-5 chỉ báo, PREFER cơ bản/catalyst/chính sách/định giá; technical KHÔNG có ở section này):
  - *Cơ bản (must-have ≥1):* "BCTC Q1 EPS growth ≥ 20% YoY", "Biên gộp Q1 cải thiện ≥ 50bp QoQ", "ROE TTM mở rộng từ 15% lên ≥ 18%"
  - *Catalyst/chính sách (must-have ≥1):* "Dự án X capacity Y MW online Q2", "Ngày chốt cổ tức tiền mặt 7% — DD/MM"
  - *Định giá:* "P/E forward về 9x vs median 5Y là 13x"
  - *Định vị/flow (secondary):* "Dòng tiền tuần duy trì dương ≥ 3/4 tuần", "FII mua ròng tháng"

- **Disconfirming signal** (PREFER cơ bản/catalyst/chính sách): [vd "BCTC Q1 EPS growth < 5% hoặc miss consensus ≥ 10%", "Biên gộp Q1 thu hẹp ≥ 100bp QoQ", "Dự án X delay sang Q4"]

#### [Ticker B] (Ngành) — Theme: [...]
[Same format]

[5-12 mã total]

### 6.2. Phase 2 — Bucket entry (PTKT-driven, sau khi đã screen Phase 1)

**Định nghĩa Bucket (tham khảo `P_invest_memo_03` mục 5):**

| Bucket | Điều kiện technical_zone | Observation |
|---|---|---|
| **1 — Vào ngay được** | zone w VÀ zone m ∈ {A, AA, AAA} + week_score ≥ 6 | Sẵn sàng cho lệnh khi catalyst materialize |
| **2 — Chờ pullback** | zone q HOẶC zone y ∈ {A, AA, AAA} NHƯNG zone w/m ∈ {B, C} | Uptrend dài còn nguyên, ngắn hạn pullback. Đợi tuần bật A |
| **3 — Watchlist (chưa sẵn)** | zone q/y tích cực NHƯNG zone w VÀ m ĐỀU ∈ {C} | Pullback sâu, đợi technical phục hồi |

**Bảng phân bucket watchlist:**

| Ticker | Ngành | Conviction Phase 1 | Bucket Phase 2 | zone w | zone m | zone q | zone y | week_score | Note |
|---|---|---|---|---|---|---|---|---|---|
| Ticker A | NGANHANG | HIGH | 1 | A | AA | A | AAA | 12.4 | Sẵn sàng |
| Ticker B | BANLE | HIGH | 2 | B | A | AA | AAA | 3.1 | Pullback trong uptrend |
| Ticker C | KIMLOAI | MID | 3 | C | C | A | A | -2.8 | Đợi technical phục hồi |
| ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |

**Rule render Bucket:**
- Bucket KHÔNG nâng/giảm conviction Phase 1 — conviction đã chốt bằng cơ bản
- Mã catalyst override (fail cơ bản) Bucket 3 → flag "**Catalyst-driven Bucket 3**: thị trường có thể priced-in tiêu cực hoặc chưa nhận ra catalyst. User review kỹ catalyst"
- Nếu Trục 2 chốt "quá mua" (phân vị > 75%) → default downgrade Bucket 1 → 2 toàn watchlist + flag rõ
- Bucket 2 timeout cảnh báo: nếu mã Bucket 2 đã ≥ 4 tuần trong watchlist mà zone w chưa bật A → flag inline "Bucket 2 timeout — thesis pullback chưa confirm, user xem xét chuyển Bucket 3 hoặc loại"

### 6.3. Wording rules

- **KHÔNG dùng từ command:** "mua / bán / giảm tỷ trọng / stop loss / target"
- **KHÔNG có entry zone / stop / target / size cụ thể**
- Diễn đạt qua catalyst + observation
- Hướng tiêu cực (mã trong ngành thận trọng): vẫn cùng format, luận điểm rõ hướng "áp lực từ X → mã chịu nặng"

### 6.4. User overlay cho mã (nếu user inject ticker idea)

Render mã user gợi ý cùng format Phase 1 + Bucket Phase 2. Thêm badge sau ticker:
- `[PM input — Confirm by data]` nếu agent cross-check confirm
- `[PM input — Partial confirm]`
- `[PM input — Data conflict]` (render với cảnh báo, conviction = LOW)
- `[PM flag — chưa có data verify]`

Mã do user inject vẫn phải đáp ứng filter Phase 1 (thanh khoản ≥ 5 tỷ ADV, ngành thuộc whitelist 18); nếu không, ghi note "ngoài scope pack — render reference only, không integrate vào watchlist chính".
```

### 3.10. Tuyên bố miễn trừ trách nhiệm

Render theo 3 trường hợp branding (giống invest_strategy old format, không đổi cấu trúc):

**(a) Custom disclaimer:** render full text user cung cấp + khối liên hệ.

**(b) Default branded:**

```
## Tuyên bố miễn trừ trách nhiệm

**Phạm vi & nguồn dữ liệu**

Báo cáo này được [TÊN CÔNG TY] soạn thảo trên cơ sở các thông tin, dữ liệu được thu thập từ các nguồn được coi là đáng tin cậy vào thời điểm phát hành. Dữ liệu định lượng chính được trích từ hệ thống dữ liệu nội bộ (`agent_db`) trong phạm vi **18 ngành whitelist** của framework VBSE Strategy, bổ sung bởi web search cho tin tức và sự kiện cập nhật. [TÊN CÔNG TY] không đảm bảo tuyệt đối về tính chính xác, đầy đủ của các thông tin này.

**Forward-looking statement**

Các nhận định, kịch bản, conviction level, time horizon, Bucket entry và disconfirming signals trong báo cáo này phản ánh **quan điểm độc lập có tính chiến lược trung hạn** (1-6 tháng) tại thời điểm công bố, **không phải dự báo chắc chắn** về diễn biến thị trường. Điều kiện thị trường có thể thay đổi do các sự kiện ngoài kỳ vọng, và các thesis có thể bị invalidate nếu disconfirming signals materialize.

**Vai trò user overlay**

Báo cáo có thể bao gồm quan điểm (PM input / user overlay) được tích hợp từ phía người yêu cầu báo cáo, được phân biệt rõ qua badge `[PM input — ...]` trong nội dung. Phần này phản ánh judgment PM, không phải khuyến nghị độc lập từ [TÊN CÔNG TY].

**Suitability & quyết định cuối**

Nhà đầu tư cần tự đánh giá mức độ phù hợp với tình hình tài chính cá nhân, khả năng chịu rủi ro, mục tiêu đầu tư và horizon đầu tư trước khi ra quyết định. Báo cáo không phải khuyến nghị mua bán cho từng cá nhân cụ thể. Quyết định đầu tư cuối cùng hoàn toàn thuộc về Quý khách hàng. [TÊN CÔNG TY] không chịu trách nhiệm về bất kỳ tổn thất nào phát sinh từ việc sử dụng báo cáo này.

### LIÊN HỆ

**[TÊN CÔNG TY]**  
Website: [...]  
Hotline: [...]

**Ngày phát hành:** [DD/MM/YYYY]
```

**(c) Plain (nội bộ):**

```
## Tuyên bố miễn trừ trách nhiệm

> ⚠️ **Lưu ý:** Báo cáo render bản plain do không có branding. Trước khi gửi khách hàng, cần bổ sung disclaimer phù hợp với pháp lý của tổ chức phát hành.

Báo cáo này phản ánh quan điểm phân tích nội bộ tại thời điểm soạn thảo, có tính chiến lược trung hạn (1-6 tháng), trong phạm vi 18 ngành whitelist VBSE Strategy. Có thể thay đổi khi điều kiện thị trường shift hoặc disconfirming signals materialize.

Nhà đầu tư tự cân nhắc dựa trên tình hình tài chính cá nhân, mục tiêu và horizon đầu tư.

**Ngày phát hành:** [DD/MM/YYYY]
```

### 3.11. Metadata cuối file (monthly)

```
---

**Metadata**

- **Tháng báo cáo:** [N/YYYY]
- **Ngày phát hành:** [DD/MM/YYYY]
- **Horizon ưu tiên:** [1 tháng / 1-3 tháng / 3-6 tháng]
- **Whitelist ngành scope:** 18 ngành (tham khảo `K_agent_db_01` Section B)
- **Regime vĩ mô:** [định tính] (Conviction: HIGH/MID/LOW)
- **Top themes:** [list — mỗi theme kèm conviction + horizon]
- **Sector quan tâm:** [list 18 whitelist kèm conviction]
- **Sector cần thận trọng:** [list kèm conviction]
- **Watchlist:** [list ticker — kèm conviction + Bucket]
- **File N-1 đã tham chiếu:** [tên file / "không có"]
- **Override checkpoint:** [có / không + lý do nếu có]

**User overlay log** (chỉ render khi có user view inject):

| # | Channel | View user nêu | Trục liên quan | Trạng thái xử lý | Vị trí integrate |
|---|---|---|---|---|---|
| 1 | Pre-flight câu 4 | [...] | Trục 3 | Confirm | Theme A — mục Trục 3 |
| 2 | Mid-flow (sau Stage 1) | [...] | Trục 5 | Partial confirm | Risk map item 2 |
| 3 | Checkpoint override | [...] | Trục 4 | Conflict — user override | Sector tilts table, dòng [X] |
| 4 | Mid-flow (Stage 3) | [...] | Trục 6 | Flag — no data | Watchlist, mã [Z] với badge |

(Nếu không có user view: "Không có user overlay trong cycle này — báo cáo build hoàn toàn từ data agent_db + web search.")

**Source & methodology disclosure:**
- **Source dữ liệu chính:** agent_db ([list collection chính đã query])
- **Web search bổ sung:** [có / không + chủ đề chính]
- **Methodology framework:** VBSE Strategy 6 trục (vĩ mô / định vị / themes / sector 18 whitelist / risk / watchlist 2-phase). Conviction định tính HIGH-MID-LOW dựa cross-check ≥2 trục. Horizon 1m/1-3m/3-6m theo timing catalyst. Disconfirming signals reference data field cụ thể. Phase 1 Screen cơ bản-only, Phase 2 Bucket entry PTKT-driven (tham khảo `P_invest_memo_03` mục 5). Không gán % xác suất kịch bản (tuân `K_agent_db_00` mục 4.3).
```

## 4. Structure báo cáo — Mode Weekly Update

**Độ dài target:** 3-5 trang MD. **Gọn hơn monthly nhiều** vì là tracking.

**Structure suggested:**

```
# Cập nhật chiến lược VBSE — Tuần [N] của tháng [M/YYYY]

## Tóm tắt tuần
## [Review tuần W-1]                  (skip nếu Stage 0 skip hoặc tuần 1 tháng)
## Trục 1 — Vĩ mô  (status: Hold / Shift / Materialize)
## Trục 2 — Định vị thị trường  (status: ...)
## Trục 3 — Themes  (status: ...)
## Trục 4 — Sector  (status: ...)
## Trục 5 — Risk  (status: ...)
## Trục 6 — Watchlist refresh + Rebucket
## Action item tuần tới
## Tuyên bố miễn trừ trách nhiệm
— Metadata
```

### 4.1. Header weekly

```
# Cập nhật chiến lược VBSE — Tuần [N] của tháng [M/YYYY]

**Cập nhật từ:** Báo cáo chiến lược tháng [M/YYYY] (`vbse_strategy_monthly_<YYYYMM>.md`, phát hành DD/MM)  
**Tuần tham chiếu:** DD/MM đến DD/MM/YYYY (tuần [N] của tháng [M])  
**File W-1 đã eval:** [`vbse_strategy_weekly_<YYYYMMDD>.md` / "không có"]  
**Phát hành:** DD/MM/YYYY
```

**Lưu ý:**
- Bắt buộc ghi "Tuần [N] của tháng [M/YYYY]" trong heading title
- Bắt buộc dẫn link file monthly tham chiếu rõ tên file
- Nếu user override (dùng monthly tháng khác làm parent), thêm dòng cảnh báo:
  > ⚠️ *Thesis carry-over từ tháng M-1 (đã decay [X] tuần). Conviction tổng thể giảm 1 bậc so monthly gốc.*

### 4.2. Tóm tắt tuần

3-6 bullet, mỗi bullet 1-2 dòng:

```
## Tóm tắt tuần

- **Trạng thái thesis tháng [N]:** [Hold đa số / Shift nhẹ / Có signal materialize cần lưu ý / Đề xuất chạy lại monthly cycle]
- **Trục có shift đáng kể:** [list 0-2 trục, hoặc "Không có"]
  - **Lưu ý technical-as-noise rule:** chỉ liệt kê Shift kèm signal vĩ mô/cơ bản/chính sách. Technical shift đơn độc đã được classify "noise tạm thời, status Hold"
- **Risk materialize tuần qua:** [list 0-2 rủi ro, hoặc "Không có"]
- **Watchlist:** [N] mã Hold / [N] Watch closely / [N] Out / [N] Vào mới. **Rebucket:** [N] mã chuyển bucket (1↔2 hoặc 2↔3)
- **PM overlay tuần này:** [optional]
- **Action item:** [1 câu tổng]
```

### 4.3. Review tuần W-1 (Stage 0 Evaluation weekly)

**3 trường hợp render** (tương tự monthly 3.3):

**(a) Full eval — user chạy Stage 0 + accept:** render đầy đủ 4 phần

```
## Review tuần W-1 — Stage 0 Evaluation

> *Đánh giá tuần W-1 (DD/MM) đã được user review và accept tại Checkpoint 0. Learning carry-forward vào tracking tuần [N].*

### Status carry-over từ W-1

| Trục | Status W-1 | Shift/Materialize W-1 | Thực tế tuần [N] | Đánh giá |
|---|---|---|---|---|
| Trục 1 — Vĩ mô | Shift | Lãi suất liên ngân hàng tăng 20bp | Tiếp diễn, tăng thêm 15bp | Đúng — shift tiếp diễn ✓ |
| Trục 2 — Định vị | Hold | — | Vẫn Hold | Đúng ✓ |
| Trục 3 — Themes | Materialize | Theme A catalyst delay | Đã confirm, đẩy sang Q3 | Đúng — đã reverse ✓ |

### Watchlist W-1 tracking

| Ticker | Trạng thái W-1 | Bucket W-1 | Biến động tuần [N] | Signal trigger? | Bucket tuần [N] | Đánh giá |
|---|---|---|---|---|---|---|
| Ticker A | Hold | 1 | +2.1% | Hold confirm | 1 | ✓ |
| Ticker B | Watch closely | 2 | -3.8% + dòng tiền âm 2 tuần | Disconfirming trigger | Out | Out tuần [N] |
| Ticker C | Vào mới W-1 | 2 | +5.2% | Theme A catalyst confirm | **1 (rebucket)** | Hold tốt + bucket up ✓ |

### Action item W-1 — materialize?

- **Action item 1 W-1:** [...] → [Materialize / chưa rõ / không relevant]

### Carry-forward vào tracking tuần [N]

- [2-3 dòng learning]
```

**(b) Short review — user skip Stage 0:** không render heading "Review tuần W-1", đi thẳng vào Trục 1.

**(c) Tuần 1 tháng (không có W-1):** không render heading "Review tuần W-1". Có thể ghi 1 dòng trong Tóm tắt tuần: "Tuần 1 của tháng [M] — vừa chạy monthly cycle, không có W-1 để eval."

### 4.4. Compose từng trục weekly

Mỗi trục heading kèm status badge `(Hold / Shift / Materialize)`:

**Trục status Hold (3-5 dòng):**

```
## Trục 1 — Vĩ mô  (Hold)

Không có shift đáng kể tuần qua. [1-2 dòng cập nhật ngắn]

Thesis monthly: [tóm tắt 1 dòng regime vĩ mô đã call] — vẫn valid.
```

**Trục status Shift (5-10 dòng):**

```
## Trục 3 — Themes  (Shift)

**Signal shift (BẮT BUỘC vĩ mô/cơ bản/chính sách):** [3-5 dòng mô tả cụ thể shift — vd "Catalyst chính của Theme A (Nghị quyết về đầu tư công) đã được delay sang tháng sau theo thông cáo Văn phòng Chính phủ ngày DD/MM. Theme A weaken nhưng chưa invalidate hoàn toàn — vẫn còn momentum từ giải ngân quý này."]

**Ngụ ý cho thesis:** [2-3 dòng — vd "Giảm priority Theme A từ top 2 xuống top 3-4 trong watchlist."]
```

**Trục status Materialize (5-10 dòng):**

```
## Trục 5 — Risk  (Materialize: Rủi ro [tên])

**Rủi ro đã materialize:** [tên rủi ro từ risk map monthly]
**Signal đã xảy ra:** [chỉ báo cụ thể + nguồn]
**Phản ứng theo monthly:** [phản ứng định tính đã đặt trước]
**Theme/sector bị invalidate:** [list nếu có]
**Cân nhắc:** [agent đề xuất — vd "Nếu rủi ro tiếp diễn 1-2 tuần nữa, đề xuất chạy lại monthly cycle giữa kỳ"]
```

**Technical shift đơn độc (KHÔNG render là Shift):**

Nếu chỉ có technical_zone tụt, MA cắt xuống, industry rank tụt mà KHÔNG kèm signal vĩ mô/cơ bản/chính sách → render trục là **Hold** với note:

```
## Trục 2 — Định vị thị trường  (Hold)

[1-2 dòng PRIMARY check không shift]

**Technical shift đơn độc (noise):** technical_zone w VNINDEX tụt từ A xuống B trong tuần qua, MA20 cắt xuống MA60. **Phân loại noise tạm thời theo Technical-as-noise rule** (`P_vbse_strategy_00` mục 4.3 + `_08` mục 4) — chưa kèm signal vĩ mô/cơ bản/chính sách shift → giữ status Hold, không nâng lên Shift.
```

### 4.5. Watchlist refresh + Rebucket

```
## Trục 6 — Watchlist refresh + Rebucket

### 6.1. Hold ([N] mã)

| Ticker | Conviction | Bucket | Note |
|---|---|---|---|
| Ticker A | HIGH | 1 | Signals còn valid |
| ... | ... | ... | ... |

### 6.2. Watch closely ([N] mã)

- **Ticker C** (Bucket 2): [2-3 dòng — signal cảnh báo nào bắt đầu xuất hiện, chưa invalidate nhưng cần theo dõi sát]

### 6.3. Out ([N] mã)

- **Ticker D** (Bucket 2 → Out): [2-3 dòng — disconfirming signal cụ thể đã materialize, vì sao loại]

### 6.4. Vào mới ([N] mã)

[Render đầy đủ Phase 1 6 thành phần + ADV + Bucket Phase 2]

- **Ticker E** (Ngành) — Theme: [tên theme cũ trong monthly]
  - **Conviction:** HIGH / MID / LOW   |   **Horizon:** 1m / 1-3m / 3-6m   |   **ADV tháng:** [X] tỷ
  - **Bucket entry:** **1** / 2 / 3
  - **Luận điểm:** [observation 1-2 dòng]
  - **Signal theo dõi:** [2-3 chỉ báo cụ thể]
  - **Disconfirming signal:** [1 dòng cụ thể]
  - **Trigger vào tuần này:** [1 dòng — sự kiện gì tuần qua khiến mã đáng đưa vào]

### 6.5. Rebucket ([N] mã chuyển bucket)

**Ngoại lệ technical-as-noise rule** — rebucket là PTKT-driven hợp pháp trong weekly (không tính là Shift thesis).

| Ticker | Bucket W-1 | Bucket tuần [N] | Lý do rebucket | Conviction Phase 1 (KHÔNG đổi) |
|---|---|---|---|---|
| Ticker C | 2 | 1 | zone w bật từ B lên A + week_score chuyển +8.2 | HIGH (giữ nguyên) |
| Ticker F | 1 | 2 | zone w tụt từ A xuống B (pullback) | MID (giữ nguyên) |
| Ticker G | 3 | 2 | zone w bật từ C lên B, đang gần A | LOW (giữ nguyên) |

[Note: Conviction Phase 1 KHÔNG thay đổi do rebucket — chỉ entry timing thay đổi.]
```

### 4.6. Action item tuần tới

```
## Action item tuần tới

1-2 item định tính:
- "Theo dõi FOMC minutes thứ Tư, signal cho Theme A"
- "Quan sát phản ứng nhóm thép sau release CPI Trung Quốc tuần tới"

KHÔNG có entry/exit cụ thể, level giá, kích thước vị thế, %.
```

### 4.7. Disclaimer weekly + Metadata

Tương tự monthly 3.10 (3 trường hợp branding). Metadata gọn hơn:

```
---

**Metadata weekly**

- **Tuần báo cáo:** Tuần [N] của tháng [M/YYYY] (DD/MM-DD/MM)
- **Monthly active tham chiếu:** `vbse_strategy_monthly_<YYYYMM>.md`
- **File W-1 đã eval:** [tên file / "không có"]
- **Trục có shift:** [list / "không có"]
- **Risk materialize:** [list / "không có"]
- **Watchlist counts:** Hold [N] / Watch [N] / Out [N] / Vào mới [N] / Rebucket [N]

**User overlay log tuần** (chỉ render khi có view inject):
[table tương tự monthly]
```

## 5. Convention chung 2 mode

### 5.1. K hygiene + ngôn ngữ

- Ký hiệu DB raw đã dịch sang ngôn ngữ tự nhiên (`week_score` → "điểm dòng tiền tuần", `technical_zone.overall.w=A` → "vùng kỹ thuật khung tuần A — tích cực")
- KHÔNG dùng từ command (mua/bán/giảm tỷ trọng/stop loss)
- Conviction: HIGH / MID / LOW
- Horizon: 1m / 1-3m / 3-6m
- Bias: Quan tâm / Trung tính / Cần thận trọng (KHÔNG "overweight/underweight")
- Bucket: 1 / 2 / 3 (theo invest_memo_03 mục 5)
- Status weekly trục: Hold / Shift / Materialize

### 5.2. Source citation

- Mỗi claim định lượng có nguồn: collection + field (vd "`industry_finstats.financial_statements.quarterly` Q1/2026") hoặc URL web search
- Tin có dẫn link finext.vn hoặc URL gốc

### 5.3. Forward-looking statement

- Mọi kịch bản, conviction, horizon, disconfirming là **observation có cơ sở data**, không phải dự báo chắc chắn
- Không gán % xác suất

### 5.4. Render whitelist 18 ngành scope

- Bảng cross-section Trục 4: đủ 18 ngành whitelist (không thêm/bớt)
- Sector tilts: chỉ 18 ngành whitelist
- Watchlist: mã chỉ thuộc 18 ngành whitelist
- Aggregate proxy thị trường Trục 2: tính trên 18 ngành
- Ngành ngoài whitelist KHÔNG render
