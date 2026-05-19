# O_invest_strategy_00 — Render Spec Báo cáo chiến lược đầu tư

Spec render báo cáo chiến lược đầu tư VN theo 2 mode: **Monthly** (parent, đầu tháng) và **Weekly update** (child, tracking trong tháng). **Flex structure** (không rigid như `O_weekly_market_00`) — 6 trục cốt lõi cố định, độ sâu và sub-section flex theo phát hiện thực tế của tháng/tuần. **Output cuối: MD final.**

Reference: `P_invest_strategy_00` (workflow + khung tư duy + hướng tìm dữ liệu).

> **Render binary:** MD final là source of truth. Khi user yêu cầu pptx / docx, agent render theo style đã chọn (O pack spec + branding info + user explicit, body font Roboto) — chi tiết ở `system_prompt.md` mục 4 "Render binary — workflow". Pack này chưa có pptx/docx spec riêng — sẽ derive từ MD structure hiện tại + style chuẩn institutional khi user yêu cầu.

## 1. Input từ P pack

P pack sinh structured content theo 6 trục (mục 3 của P_invest_strategy_00) cộng status mode-specific. O pack chỉ render, không thêm/bớt nội dung.

**Trường hợp đặc biệt:**
- Monthly không có file N-1 → bỏ phần Review hit rate
- Weekly mode bắt buộc có file monthly upload — nếu thiếu, P pack không chạy, O pack không có gì render
- Checkpoint regime override → ghi inline note trong trục liên quan
- Branding info user cung cấp → render branded header + disclaimer; không có → plain header

## 2. Structure báo cáo — Mode Monthly

**Độ dài target:** 8-12 trang MD.

**Structure flex** (suggested, KHÔNG rigid):

```
# Báo cáo chiến lược đầu tư — Tháng [N/YYYY]

## Tóm tắt điều hành
## [Review tháng trước]            (skip nếu lần đầu)
## Trục 1 — Môi trường vĩ mô & tài chính
## Trục 2 — Định vị thị trường VN
## Trục 3 — Themes & narratives chính
## Trục 4 — Sector allocation
## Trục 5 — Kịch bản & risk map
## Trục 6 — Watchlist
## Tuyên bố miễn trừ trách nhiệm
— Metadata
```

**Quy tắc flex:**
- Mỗi trục có 1 H2 chính. Sub-section H3 tự agent quyết theo phát hiện thực tế (vd trục 1 có thể chỉ 1 prose 5-7 dòng nếu vĩ mô ổn định, hoặc chia 3-4 sub-section nếu có biến động lớn)
- Tên trục có thể adapt — vd "Trục 3 — Themes & narratives" → "Trục 3 — Sóng đầu tư công + Margin ngân hàng cải thiện" khi đã chốt themes cụ thể
- Trục Hold (không có gì đặc biệt) ghi 3-5 dòng ngắn, KHÔNG ép viết dài
- Trục có signal mạnh có thể chiếm 1-2 trang đào sâu
- Có thể merge 2 trục nếu ranh giới mờ (vd Trục 1 + 3 hợp nhất khi tháng dominated bởi 1 chính sách lớn) — báo trong tên H2: "Trục 1+3 — Môi trường vĩ mô & Theme chi phối"

## 3. Compose từng phần (Monthly)

### 3.1. Header

**Format plain (không branding):**

```
# Báo cáo chiến lược đầu tư — Tháng [N/YYYY]

**Phát hành:** [DD/MM/YYYY]  
**Phạm vi:** Thị trường cổ phiếu Việt Nam (VNINDEX, HSX/HNX/UPCOM)  
**Horizon ưu tiên:** [1 tháng / 1-3 tháng / 3-6 tháng]  
**Audience:** Phân tích nội bộ
```

**Format branded:**

```
# [TÊN CÔNG TY]
## BÁO CÁO CHIẾN LƯỢC ĐẦU TƯ
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
- **Định vị thị trường VN:** [định tính — vd "Tích luỹ sau giảm 6%, định giá ở phân vị 30% lịch sử"]. [1 dòng căn cứ.]
- **Top themes tháng [N]:** [Theme 1 (HIGH, 1-3m) + Theme 2 (MID, 3-6m) + Theme 3 (LOW, 1m) — tên ngắn + conviction + horizon]
- **Sector bias:** Quan tâm [3-5 ngành, kèm conviction marker từng ngành nếu khác nhau]. Thận trọng [1-3 ngành].
- **Risk chính:** [1-2 rủi ro nổi bật từ risk map, kèm signal materialize cụ thể nhất để theo dõi]
- **Watchlist tiêu biểu:** [1-3 mã đại diện theme chính, optional — kèm conviction + horizon nếu render]
- **PM overlay note:** [optional — chỉ render nếu có user view inject ở conviction HIGH hoặc trạng thái Conflict chưa resolve. Vd "PM nêu rủi ro thanh khoản TPCP DN tháng 5, agent partial confirm — xem mục Risk map và audit trail."]
```

### 3.3. Review tháng trước (Stage 0 Evaluation)

**3 trường hợp render:**

**(a) Full eval — user chạy Stage 0 + accept (pre-flight câu 2 = a):** render đầy đủ 6 phần eval (chuẩn institutional honest review)

```
## Review tháng trước — Stage 0 Evaluation

> *Đánh giá chiến lược tháng N-1 đã được user review và accept tại Checkpoint 0 (DD/MM/YYYY). Learning được carry-forward vào thesis tháng [N].*

### Regime evaluation

**Regime call tháng N-1:** [định tính] (Conviction tháng N-1: HIGH/MID/LOW) — Đánh giá: **đúng / lệch nhẹ / sai rõ**

[2-4 dòng so call cũ vs actual: vĩ mô tháng N-1 thực tế (lãi suất / tỷ giá / FII / commodities) khớp / lệch ra sao]

### Themes evaluation

| # | Theme N-1 | Conviction cũ | Catalyst trigger | Trạng thái thực tế | Hit/Miss |
|---|---|---|---|---|---|
| 1 | [Theme A] | HIGH | [tên catalyst] | Materialize / Partial / Fizzle / Disconfirming triggered | Hit ✓ |
| 2 | [Theme B] | MID | [tên catalyst] | [...] | Miss ✗ |
| ... | ... | ... | ... | ... | ... |

### Sector tilts evaluation

**Ngành quan tâm tháng N-1:**

| Ngành | Conviction cũ | m_pct thực tế | Hit/Miss |
|---|---|---|---|
| [Ngành A] | HIGH | +5.2% | Hit ✓ |
| ... | ... | ... | ... |

**Ngành cần thận trọng tháng N-1:**

| Ngành | Conviction cũ | m_pct thực tế | Hit/Miss |
|---|---|---|---|
| [Ngành X] | HIGH | -3.8% | Hit ✓ (giảm như dự đoán) |
| ... | ... | ... | ... |

**Hit rate tổng:** [N/M] ngành quan tâm tăng giá, [N/M] ngành thận trọng giảm giá.

### Watchlist evaluation

| Ticker | Theme cũ | Conviction cũ | Horizon cũ | m_pct thực tế | Signal trigger? | Disconfirming trigger? | Hit/Miss |
|---|---|---|---|---|---|---|---|
| [Ticker A] | [Theme A] | HIGH | 1-3m | +12% | Có (BCTC Q1 EPS +25%) | Không | Hit ✓ |
| [Ticker B] | [Theme B] | MID | 1m | -8% | Không | Có (dòng tiền âm 3 tuần) | Miss ✗ |
| ... | ... | ... | ... | ... | ... | ... | ... |

**Hit rate watchlist:** [N/M] mã chạy đúng luận điểm.

### Risk map evaluation

- **Rủi ro materialize tháng N-1:** [Liệt kê rủi ro nào trigger, signal nào materialize cụ thể, theme nào bị invalidate]
- **Rủi ro còn nguyên (carry-forward tháng [N]):** [Rủi ro nào chưa xảy ra nhưng còn relevant — đưa vào risk map tháng [N]]
- **Rủi ro mới agent đã miss (phát hiện sau):** [nếu có, ghi rõ — honesty review]

### Calibration learning

- **Best call tháng N-1:** [theme / sector / mã đúng nhất] — [1-2 dòng tại sao đúng, signals nào đã trigger, có gì replicable]
- **Worst call tháng N-1:** [theme / sector / mã sai nhất] — [1-2 dòng tại sao sai (honest), signals nào agent đã bỏ qua hoặc miscalibrated, có gắn với pitfall methodology nào không]
- **Carry-forward vào tháng [N]:** [2-3 dòng learning chính, đặc biệt note nếu worst call gắn với pitfall methodology — tháng [N] tránh lặp]
```

**(b) Short review — user skip Stage 0 nhưng có file N-1 (pre-flight câu 2 = b):** render ngắn không cần checkpoint

```
## Review tháng trước

**Regime call tháng N-1:** [đúng/lệch/sai]. [1 dòng]
**Hit rate themes:** [N/M] materialize, [N/M] còn nguyên, [N/M] fizzle
**Hit rate sector bias:** [N/M] ngành quan tâm tăng giá, [N/M] ngành thận trọng giảm giá
**Hit rate watchlist:** [N/M] mã chạy đúng luận điểm
**Best call:** [...] — [1 dòng]
**Worst call:** [...] — [1 dòng honest]
**Rủi ro materialize:** [list / không có]
**Learning chính cho tháng [N]:** [1-3 dòng]
```

**(c) Skip entirely — không có file N-1 (pre-flight câu 2 = c hoặc câu 1 = b):**

```
## Review tháng trước

Lần đầu chạy monthly cycle / không có file N-1, chưa có dữ liệu review tháng trước.
```

### 3.4. Trục 1 — Môi trường vĩ mô & tài chính

Sub-section flex. Suggestion (agent tự adapt):

```
## Trục 1 — Môi trường vĩ mô & tài chính

[Prose mở 3-5 dòng — kết luận regime vĩ mô tổng thể, lý do]

### 1.1. Chính sách tiền tệ & lãi suất

[Bảng lãi suất điều hành VN + Fed/ECB/PBOC + biến động tháng]
[Diễn giải 3-5 dòng]

### 1.2. Tỷ giá & dòng vốn

[Bảng tỷ giá USD/VND + EUR/USD + USD/CNY + DXY + FII net flow]
[Diễn giải 3-5 dòng]

### 1.3. Vĩ mô thực

[Bảng/list CPI, GDP, PMI, XNK, FDI, bán lẻ — chỉ số tháng gần nhất + xu hướng]
[Diễn giải tác động equity 2-4 dòng]

### 1.4. Hàng hoá cross-sector

[Bảng commodities biến động đáng kể tháng qua + ngành VN bị tác động]
[Bỏ sub-section này nếu tháng không có biến động commodity đáng kể]

### 1.5. Kết luận regime vĩ mô

[2-4 dòng — regime ở giai đoạn nào, kỳ vọng 1-3 tháng tới]
```

Trục Hold (vĩ mô ổn định cả tháng): rút gọn còn 1 prose 5-8 dòng, không cần sub-section.

### 3.5. Trục 2 — Định vị thị trường VN

```
## Trục 2 — Định vị thị trường VN trong chu kỳ

### 2.1. Định giá tổng thể

[Bảng/prose: P/E VNINDEX hiện tại vs median 3-5Y, phân vị lịch sử, so EM peers nếu có web search]

### 2.2. Dòng tiền cấp thị trường

[Xu hướng điểm dòng tiền tuần thị trường (proxy aggregate 24 ngành) 4-8 tuần gần nhất — pattern tăng/giảm/dao động]
[NN/TD aggregate tháng/quý — net flow + xu hướng]

### 2.3. Breadth & sentiment

[% ngành tăng giá tháng, % mã trên MA60/MA120, thanh khoản trung bình tháng vs 6 tháng]

### 2.4. Kết luận định vị

[2-4 dòng — thị trường ở giai đoạn nào trong chu kỳ, vùng giá VNINDEX 1-3 tháng tới]
```

### 3.6. Trục 3 — Themes & narratives chính

```
## Trục 3 — Themes & narratives chính

[2-5 themes — không ép số. Tên theme rõ ràng, có cơ chế cụ thể.]

### 3.1. Theme A — [Tên ngắn gọn]

**Conviction:** HIGH / MID / LOW   |   **Horizon:** 1m / 1-3m / 3-6m   |   **Ngành/mã liên quan:** [3-7 ngành + một số mã đại diện]

**Cơ chế:** [3-5 dòng — vì sao theme này, mạch logic nguyên nhân → hệ quả]

**Catalyst trigger:** [sự kiện / mức số / chính sách cụ thể; nếu có ngày dự kiến, ghi ngày]

**Disconfirming signals (kill criteria):**
- [Signal cụ thể 1 — vd "Industry rank ngành thép tụt khỏi top 8 trong 2 tuần liên tiếp"]
- [Signal cụ thể 2 — vd "USD/VND vượt 26500 + can thiệp NHNN >1 tỷ USD"]
- [Signal cụ thể 3, optional]

### 3.2. Theme B — [Tên]
...
```

**Sau danh sách themes, render bảng "Catalysts ahead" tổng hợp ngày từ các theme:**

```
### Catalysts ahead — Lịch tổng hợp

| Ngày dự kiến | Sự kiện | Theme tương ứng | Hướng tác động |
|---|---|---|---|
| Thứ Tư DD/MM | [FOMC minutes / CPI VN / BCTC mã X] | Theme A | Tích cực / Tiêu cực |
| Thứ Năm DD/MM | [...] | Theme B | ... |
| ... | ... | ... | ... |
```

[Bảng chỉ liệt kê catalyst có ngày tương đối rõ. Catalyst dạng "trong quý" không ngày cụ thể bỏ qua bảng này, giữ trong nội dung theme.]

Khi có override checkpoint, ghi inline note đầu trục:
> *Theme này chốt sau review checkpoint: [override note user nêu, vd "Bỏ theme 'M&A ngành thép' vì user phản hồi catalyst chưa rõ"].*

Khi theme có user overlay (PM input), render badge inline ngay sau tên theme:
> *Theme này có user overlay: `[Synthesized from PM input + data confirm]` — PM nêu Q4/2025: cải cách thuế bất động sản; data agent_db cross-check: industry_rank BĐS Dân dụng tăng từ rank 18 lên rank 9 trong 4 tuần qua + tin chính sách 3 tuần qua trên `news_history_feed` ủng hộ. Xem mục audit trail.*

### 3.7. Trục 4 — Sector allocation

```
## Trục 4 — Sector allocation

### 4.1. Bảng cross-section 24 ngành

[Bảng:
| Ngành | Biến động tháng | Dòng tiền tháng (proxy aggregate week_score 4 tuần) | P/E hiện tại vs median 3Y | % mã trong ngành tăng tháng | Xếp hạng dòng tiền | Bias |
]

(Bias: Quan tâm / Trung tính / Cần thận trọng)

### 4.2. Sector tilts table — bảng tổng hợp (chuẩn buy-side single-page scannable)

[Bảng tổng hợp tất cả ngành có bias rõ — không 24 ngành đầy đủ, chỉ những ngành quan tâm + cần thận trọng + 1-2 ngành trung tính đáng note. Mục đích: leadership scan 1 phút.]

| Ngành | Bias | Conviction | Driver chính | Signal hỗ trợ (số) | Disconfirming signal |
|---|---|---|---|---|---|
| Bán lẻ Tiêu dùng | Quan tâm | HIGH | Theme A — phục hồi tiêu dùng cuối năm | Dòng tiền rank 3 / 24, P/E 14x vs median 18x | Industry rank tụt khỏi top 8 trong 2 tuần liên tiếp |
| Thép | Quan tâm | MID | Theme B — chu kỳ thép upcycle theo TQ | HRC +8% tháng, dòng tiền rank 7 | Quặng sắt giảm > 10% trong 2 tuần |
| Ngân hàng | Trung tính | — | Margin cải thiện nhưng định giá cao | P/B 1.4x đã ở phân vị 70% 3Y | NIM Q4 release dưới 3.2% |
| Bất động sản Dân dụng | Cần thận trọng | HIGH | Theme tiêu cực — lãi suất cho vay tăng | Industry rank 21, breadth_in 5 / 35 | Lãi suất cho vay giảm > 0.5% trong tháng |
| ... | ... | ... | ... | ... | ... |

### 4.3. Phân tầng (chi tiết)

**Ngành quan tâm ([N] ngành):**
- **[Ngành A]** — [3-5 dòng: theme nào dẫn dắt, cơ chế vĩ mô hỗ trợ, dòng tiền tháng + xu hướng, định giá tương đối, dẫn dắt thật vs trụ kéo, signal cảnh báo nếu có]
- **[Ngành B]** — [...]

**Ngành trung tính ([N] ngành):**
- **[Ngành C]** — [2-3 dòng: signal hỗn hợp, không có theme]
- ...

**Ngành cần thận trọng ([N] ngành):**
- **[Ngành X]** — [3-5 dòng: vĩ mô áp lực / định vị xấu / định giá quá cao / theme tiêu cực]
- ...

### 4.4. Crowding & rotation note (optional)

[Crowding check: ngành nào đang "consensus crowded" (NN positioning rất nặng + breadth nội bộ rất rộng) → flag rủi ro thoái lui mạnh khi thesis lệch. 2-3 dòng nếu có.]
[Rotation note: nếu phát hiện rotation rõ giữa large/mid/small cap hoặc giữa nhóm ngành, ghi 3-5 dòng. Nếu không có rotation đáng kể, bỏ sub-section.]
```

### 3.8. Trục 5 — Kịch bản & risk map

```
## Trục 5 — Kịch bản & risk map

### 5.1. Ba kịch bản VNINDEX

**Kịch bản cơ sở:**
- **Trigger duy trì:** [điều kiện kỹ thuật + flow + sự kiện cụ thể]
- **Vùng VNINDEX dự kiến 1-3 tháng:** [X] — [Y]
- **Hành vi kỳ vọng:** [định tính]

**Kịch bản tích cực:**
- **Trigger break-out:** [điều kiện cụ thể — kháng cự, volume, flow, catalyst]
- **Mục tiêu kỹ thuật:** [Y] ([Fibonacci / POC khung lớn / đỉnh quý/năm])
- **Theme củng cố:** [theme nào được boost bởi kịch bản này]

**Kịch bản tiêu cực:**
- **Trigger break-down:** [điều kiện cụ thể]
- **Vùng hỗ trợ tiếp theo:** [Y]
- **Risk concentration:** [sector/theme nào chịu impact lớn nhất]

> *Kịch bản là hệ thống điều kiện kỹ thuật + cơ bản, không phải dự báo chắc chắn. Diễn biến thực tế có thể lệch khỏi cả 3 nếu xuất hiện sự kiện ngoài kỳ vọng.*

### 5.2. Risk map

**Rủi ro 1 — [Tên ngắn]**
- **Bản chất:** [2-3 dòng]
- **Signal materialize:** [chỉ báo / sự kiện / mức số cụ thể để biết đang xảy ra]
- **Phản ứng định tính:** [định tính — vd "giảm exposure ngành X" / "chuyển sang defensive" / "đứng ngoài chờ"]
- **Theme bị invalidate (nếu có):** [theme nào fall apart nếu rủi ro này xảy ra]

**Rủi ro 2 — [...]**
...

[3-7 rủi ro, flex theo bối cảnh.]
```

### 3.9. Trục 6 — Watchlist

```
## Trục 6 — High-conviction watchlist

[5-12 mã đại diện theme & sector bias. Gộp cả hướng tích cực và tiêu cực nếu có.]

- **[Ticker A]** (Ngành) — Theme: [tên theme]
  - **Conviction:** HIGH / MID / LOW   |   **Horizon:** 1m / 1-3m / 3-6m   |   **ADV tháng:** [X] tỷ
  - **Luận điểm:** [observation 1-2 dòng — cơ chế theme + lý do mã hưởng lợi]
  - **Signal theo dõi:** [2-3 chỉ báo cụ thể, vd: "BCTC Q1 EPS growth >20%", "dòng tiền tuần duy trì dương 4 tuần liên tiếp", "vùng kỹ thuật khung tuần giữ trên POC tháng [X]"]
  - **Disconfirming signal:** [1 dòng cụ thể — điều gì invalidate thesis mã này, vd "dòng tiền tuần âm 2 tuần liên tiếp", "BCTC Q1 EPS growth < 5%", "vùng kỹ thuật khung tháng tụt khỏi A"]

- **[Ticker B]** (Ngành) — Theme: [...]
  - **Conviction:** ... | **Horizon:** ... | **ADV tháng:** ...
  - **Luận điểm:** [...]
  - **Signal theo dõi:** [...]
  - **Disconfirming signal:** [...]

...
```

**Bảng tóm tắt watchlist** (optional, render khi watchlist > 7 mã):

| Ticker | Ngành | Theme | Conviction | Horizon | ADV tháng (tỷ) |
|---|---|---|---|---|---|
| [...] | [...] | [...] | HIGH | 1-3m | [X] |
| ... | ... | ... | ... | ... | ... |

**Wording rules:**
- KHÔNG dùng từ command: "mua / bán / giảm tỷ trọng / stop loss / target"
- KHÔNG có entry zone / stop / target / size cụ thể
- Diễn đạt qua catalyst + flow + technical observation
- Hướng tiêu cực (mã trong ngành thận trọng): vẫn cùng format, luận điểm rõ hướng "áp lực từ X → mã chịu nặng"

**User overlay cho mã (nếu user inject ticker idea):**

Render mã do user gợi ý cùng format 6 thành phần. Thêm badge sau ticker:
- `[PM input — Confirm by data]` nếu agent cross-check confirm
- `[PM input — Partial confirm]` nếu một phần data ủng hộ
- `[PM input — Data conflict]` nếu data hiện tại ngược chiều (vẫn render với cảnh báo, conviction = LOW)
- `[PM flag — chưa có data verify]` nếu không có data verify

Mã do user inject vẫn phải đáp ứng filter cơ bản (thanh khoản ≥ 5 tỷ/phiên ADV); nếu không, ghi note "ADV dưới ngưỡng filter, mã illiquid — render như reference only, không integrate vào watchlist chính".

### 3.10. Tuyên bố miễn trừ trách nhiệm

Render theo 3 trường hợp branding (giống `O_weekly_market_00` mục 3.13):

**(a) Custom disclaimer:** render full text user cung cấp + khối liên hệ.

**(b) Default branded:** render template default + khối liên hệ. Có forward-looking statement language chuẩn institutional.

```
## Tuyên bố miễn trừ trách nhiệm

**Phạm vi & nguồn dữ liệu**

Báo cáo này được [TÊN CÔNG TY] soạn thảo trên cơ sở các thông tin, dữ liệu được thu thập từ các nguồn được coi là đáng tin cậy vào thời điểm phát hành. Dữ liệu định lượng chính được trích từ hệ thống dữ liệu nội bộ (`agent_db` — bao gồm số liệu giá, dòng tiền, BCTC, vĩ mô, hàng hoá quốc tế) bổ sung bởi web search cho tin tức và sự kiện cập nhật. [TÊN CÔNG TY] không đảm bảo tuyệt đối về tính chính xác, đầy đủ của các thông tin này.

**Forward-looking statement**

Các nhận định, kịch bản, conviction level, time horizon và disconfirming signals trong báo cáo này phản ánh **quan điểm độc lập có tính chiến lược trung hạn** (1-6 tháng) tại thời điểm công bố, **không phải dự báo chắc chắn** về diễn biến thị trường. Điều kiện thị trường có thể thay đổi do các sự kiện ngoài kỳ vọng, và các thesis có thể bị invalidate nếu disconfirming signals materialize.

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

**(c) Plain (nội bộ, không branding):**

```
## Tuyên bố miễn trừ trách nhiệm

> ⚠️ **Lưu ý:** Báo cáo render bản plain do không có branding. Trước khi gửi khách hàng, cần bổ sung disclaimer phù hợp với pháp lý của tổ chức phát hành.

Báo cáo này phản ánh quan điểm phân tích nội bộ tại thời điểm soạn thảo, có tính chiến lược trung hạn (1-6 tháng) và có thể thay đổi khi điều kiện thị trường shift hoặc disconfirming signals materialize. Conviction level, time horizon và disconfirming signals trong báo cáo là **judgment có cơ sở data**, không phải dự báo chắc chắn.

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
- **Regime vĩ mô:** [định tính] (Conviction: HIGH/MID/LOW)
- **Top themes:** [list — mỗi theme kèm conviction + horizon, vd "Theme A (HIGH, 1-3m); Theme B (MID, 3-6m)"]
- **Sector quan tâm:** [list kèm conviction từng ngành]
- **Sector cần thận trọng:** [list kèm conviction]
- **Watchlist:** [list ticker — kèm conviction tóm tắt]
- **File N-1 đã tham chiếu:** [tên file / "không có"]
- **Override checkpoint:** [có / không + lý do nếu có]

**User overlay log** (chỉ render khi có user view inject):

| # | Channel | View user nêu (tóm tắt 1 dòng) | Trục liên quan | Trạng thái xử lý | Vị trí integrate trong báo cáo |
|---|---|---|---|---|---|
| 1 | Pre-flight câu 3 | [...] | Trục 3 | Confirm | Theme A — mục Trục 3 |
| 2 | Mid-flow (sau Stage 1) | [...] | Trục 5 | Partial confirm | Risk map item 2 |
| 3 | Checkpoint override | [...] | Trục 4 | Conflict — user override | Sector tilts table, dòng [X] |
| 4 | Mid-flow (Stage 3) | [...] | Trục 6 | Flag — no data | Watchlist, mã [Z] với badge |

(Nếu không có user view, ghi 1 dòng: "Không có user overlay trong cycle này — báo cáo build hoàn toàn từ data agent_db + web search.")

**Source & methodology disclosure:**
- **Source dữ liệu chính:** agent_db ([list collection chính đã query])
- **Web search bổ sung:** [có / không + chủ đề chính]
- **Methodology framework:** 6 trục (vĩ mô / định vị / themes / sector / risk / watchlist), conviction định tính HIGH-MID-LOW dựa cross-check ≥2 trục, horizon 1m/1-3m/3-6m theo timing catalyst materialize, disconfirming signals reference data field cụ thể. Không gán % xác suất kịch bản (tuân `K_agent_db_00` mục 4.3).
```

## 4. Structure báo cáo — Mode Weekly Update

**Độ dài target:** 3-5 trang MD. **Gọn hơn monthly nhiều** vì là tracking, không build từ đầu.

**Structure suggested:**

```
# Cập nhật chiến lược tuần — Tuần [N] của tháng [M/YYYY]

## Tóm tắt tuần
## [Review tuần W-1]                  (skip nếu Stage 0 skip hoặc tuần 1 tháng)
## Trục 1 — Vĩ mô  (status: Hold / Shift / Materialize)
## Trục 2 — Định vị thị trường  (status: ...)
## Trục 3 — Themes  (status: ...)
## Trục 4 — Sector  (status: ...)
## Trục 5 — Risk  (status: ...)
## Trục 6 — Watchlist refresh
## Action item tuần tới
## Tuyên bố miễn trừ trách nhiệm
— Metadata
```

### 4.1. Header weekly

```
# Cập nhật chiến lược tuần — Tuần [N] của tháng [M/YYYY]

**Cập nhật từ:** Báo cáo chiến lược tháng [M/YYYY] (`invest_strategy_monthly_<YYYYMM>.md`, phát hành DD/MM)  
**Tuần tham chiếu:** DD/MM đến DD/MM/YYYY (tuần [N] của tháng [M])  
**File W-1 đã eval:** [`invest_strategy_weekly_<YYYYMMDD>.md` / "không có"]  
**Phát hành:** DD/MM/YYYY
```

**Lưu ý format header weekly:**
- Bắt buộc ghi "Tuần [N] của tháng [M/YYYY]" trong heading title — không chỉ ngày
- Bắt buộc dẫn link file monthly tham chiếu rõ tên file
- Nếu user override (dùng monthly tháng khác làm parent), thêm dòng cảnh báo:
  > ⚠️ *Thesis carry-over từ tháng M-1 (đã decay [X] tuần). Conviction tổng thể giảm 1 bậc so monthly gốc. Đề xuất chạy monthly cycle cho tháng [M/YYYY] để có thesis tươi.*

Branded version giống monthly format (mục 3.1 branded), thay heading "BÁO CÁO CHIẾN LƯỢC ĐẦU TƯ" thành "CẬP NHẬT CHIẾN LƯỢC TUẦN — Tuần [N] tháng [M/YYYY]".

### 4.2. Tóm tắt tuần

3-6 bullet, mỗi bullet 1-2 dòng:

```
## Tóm tắt tuần

- **Trạng thái thesis tháng [N]:** [Hold đa số / Shift nhẹ / Có signal materialize cần lưu ý / Đề xuất chạy lại monthly cycle]
- **Trục có shift đáng kể:** [list 0-2 trục, hoặc "Không có"]
- **Risk materialize tuần qua:** [list 0-2 rủi ro, hoặc "Không có"]
- **Watchlist:** [N] mã Hold / [N] Watch closely / [N] Out / [N] Vào mới
- **PM overlay tuần này:** [optional — chỉ render nếu user inject view tuần này; vd "PM nêu BCTC mã X surprise; agent confirm, integrate vào Trục 6 — xem audit trail"]
- **Action item:** [1 câu tổng]
```

### 4.2b. Review tuần W-1 (Stage 0 Evaluation weekly)

**3 trường hợp render** (tương tự monthly mục 3.3):

**(a) Full eval — user chạy Stage 0 + accept:** render đầy đủ 4 phần

```
## Review tuần W-1 — Stage 0 Evaluation

> *Đánh giá tuần W-1 (DD/MM) đã được user review và accept tại Checkpoint 0. Learning carry-forward vào tracking tuần [N].*

### Status carry-over từ W-1

| Trục | Status W-1 | Shift/Materialize W-1 | Thực tế tuần [N] | Đánh giá |
|---|---|---|---|---|
| Trục 1 — Vĩ mô | Shift | Lãi suất liên ngân hàng tăng 20bp | Tiếp diễn, tăng thêm 15bp | Đúng — shift tiếp diễn ✓ |
| Trục 2 — Định vị | Hold | — | Vẫn Hold | Đúng ✓ |
| Trục 3 — Themes | Materialize | Theme A catalyst delay | Đã confirm, đẩy sang Q3 | Đúng — đã reverse — ✓ |
| ... | ... | ... | ... | ... |

### Watchlist W-1 tracking

| Ticker | Trạng thái W-1 | Biến động tuần [N] | Signal trigger? | Đánh giá |
|---|---|---|---|---|
| [Ticker A] | Hold | +2.1% | Hold confirm | ✓ |
| [Ticker B] | Watch closely | -3.8% + dòng tiền âm 2 tuần | Disconfirming trigger | → Out tuần [N] |
| [Ticker C] | Vào mới (W-1) | +5.2% | Theme A catalyst confirm | Hold tốt ✓ |
| ... | ... | ... | ... | ... |

### Action item W-1 — materialize?

- **Action item 1 W-1:** "Theo dõi FOMC minutes thứ Tư" → **Materialize:** Fed cho thấy tone hawkish hơn → ngụ ý: theme margin ngân hàng yếu hơn dự kiến
- **Action item 2 W-1:** [...] → [Materialize / chưa rõ / không relevant]

### Carry-forward vào tracking tuần [N]

- [2-3 dòng learning — vd "Trục 1 vĩ mô tiếp diễn shift sang hawkish, cần update Trục 3 theme A xuống MID conviction. Ticker B đã trigger disconfirming, loại Out khỏi watchlist tuần [N]."]
```

**(b) Short review — user skip Stage 0:** không render heading "Review tuần W-1", đi thẳng vào Trục 1.

**(c) Tuần 1 tháng (không có W-1):** không render heading "Review tuần W-1", đi thẳng vào Trục 1. Có thể ghi 1 dòng trong Tóm tắt tuần: "Tuần 1 của tháng [M] — vừa chạy monthly cycle, không có W-1 để eval."

### 4.3. Compose từng trục weekly

Mỗi trục heading kèm status badge `(status: Hold / Shift / Materialize)`:

**Trục status Hold (3-5 dòng):**

```
## Trục 1 — Vĩ mô  (Hold)

Không có shift đáng kể tuần qua. [1-2 dòng cập nhật ngắn — vd "Fed minute giữ tone tương tự FOMC trước, không có surprise. Tỷ giá USD/VND ổn định trong biên +/- 0.3%."]

Thesis monthly: [tóm tắt 1 dòng regime vĩ mô đã call] — vẫn valid.
```

**Trục status Shift (5-10 dòng):**

```
## Trục 3 — Themes  (Shift)

**Thay đổi:** [3-5 dòng mô tả cụ thể shift — vd "Catalyst chính của Theme A (nghị quyết về đầu tư công thông qua) đã được delay sang tháng sau. Theme A weaken nhưng chưa invalidate hoàn toàn — vẫn còn momentum từ giải ngân quý này."]

**Ngụ ý cho thesis:** [2-3 dòng — vd "Giảm priority Theme A từ top 2 xuống top 3-4 trong watchlist. Mã PVS, FCN trong watchlist hold nhưng monitor kỹ tuần tới."]
```

**Trục status Materialize (5-10 dòng):**

```
## Trục 5 — Risk  (Materialize: Rủi ro [tên])

**Rủi ro đã materialize:** [tên rủi ro từ risk map monthly]

**Signal đã xảy ra:** [chỉ báo / sự kiện / mức số cụ thể — dẫn nguồn]

**Phản ứng theo monthly:** [phản ứng định tính đã đặt trước trong monthly]

**Theme/sector bị invalidate:** [list nếu có]

**Cân nhắc:** [agent đề xuất 1-2 dòng — vd "Nếu rủi ro tiếp diễn 1-2 tuần nữa, đề xuất chạy lại monthly cycle giữa kỳ"]
```

### 4.4. Watchlist refresh

```
## Trục 6 — Watchlist refresh

**Hold ([N] mã):**
- **[Ticker A]** — [1 dòng confirm signals còn valid]
- **[Ticker B]** — [...]

**Watch closely ([N] mã):**
- **[Ticker C]** — [2-3 dòng — signal cảnh báo nào bắt đầu xuất hiện, chưa invalidate nhưng cần theo dõi sát]

**Out ([N] mã):**
- **[Ticker D]** — [2-3 dòng — disconfirming signal cụ thể đã materialize, vì sao loại khỏi watchlist]

**Vào mới ([N] mã):** [render đầy đủ 6 thành phần monthly format]

- **[Ticker E]** (Ngành) — Theme: [tên theme cũ trong monthly]
  - **Conviction:** HIGH / MID / LOW   |   **Horizon:** 1m / 1-3m / 3-6m   |   **ADV tháng:** [X] tỷ
  - **Luận điểm:** [observation 1-2 dòng]
  - **Signal theo dõi:** [2-3 chỉ báo cụ thể]
  - **Disconfirming signal:** [1 dòng cụ thể]
  - **Trigger vào tuần này:** [1 dòng — sự kiện gì tuần qua khiến mã đáng đưa vào, đây là điểm khác biệt mode weekly]
```

### 4.5. Action item tuần tới

```
## Action item tuần tới

1. [Định tính — vd "Theo dõi FOMC minute thứ Tư DD/MM, signal cho Theme 'Margin ngân hàng cải thiện'"]
2. [Vd "Quan sát phản ứng nhóm thép sau release CPI Trung Quốc thứ Năm DD/MM, có thể trigger shift sector bias"]
```

KHÔNG có entry/exit, level giá, kích thước vị thế, %.

### 4.6. Metadata cuối (weekly)

```
---

**Metadata**

- **Tuần báo cáo:** DD/MM đến DD/MM/YYYY
- **Ngày phát hành:** DD/MM/YYYY
- **Báo cáo tháng tham chiếu:** invest_strategy_monthly_<YYYYMM>.md (phát hành DD/MM/YYYY)
- **Trạng thái tổng thể:** [Hold đa số / Shift nhẹ / Materialize / Đề xuất chạy monthly cycle giữa kỳ]
- **Trục Shift:** [list]
- **Trục Materialize:** [list]
- **Watchlist change:** [N Hold / N Watch closely / N Out / N In]

**User overlay log** (chỉ render khi có user view inject tuần này):

| # | Channel | View user nêu | Trục liên quan | Trạng thái | Vị trí integrate |
|---|---|---|---|---|---|
| 1 | Pre-flight câu 3 | [...] | [...] | [Confirm/Partial/Conflict/Flag] | [...] |

(Nếu không có user view, ghi 1 dòng: "Không có user overlay trong update tuần này.")

- **Source dữ liệu chính:** agent_db
- **Web search:** [có / không + chủ đề]
```

## 5. Compose workflow step-by-step

### 5.1. Monthly mode

1. **Pre-flight:** P pack hỏi 6 câu (file N-1, Stage 0 eval, focus, user view, branding, horizon)
2. **Stage 0 (OPTIONAL):** nếu user chọn (a) câu 2 — P pack đọc file N-1, cross-check thesis vs actual data tháng N-1, compose eval block 6 phần
3. **Render Checkpoint 0 block:** O pack format eval block trong message (xem mục 6.1)
4. **Wait user:** dừng turn
5. **User phản hồi Checkpoint 0:** accept → carry-forward feed Stage 1; refine → revise eval, hỏi lại; skip carry-forward → giữ Best/Worst call cho Review section ngắn
6. **Compose Stage 1 (trục 1-3):** P pack chạy với context carry-forward, output structured
7. **Render Checkpoint 1 block:** O pack format block ngắn — regime + định vị + top themes + multi-choice (xem mục 6.2)
8. **Wait user:** dừng turn
9. **User phản hồi Checkpoint 1:** confirm → tiếp; override → ghi inline note + tiếp; đào thêm → P pack query bổ sung, hỏi lại
10. **Compose Stage 2 (trục 4-5):** sector allocation + risk
11. **Compose Stage 3 (trục 6 + executive summary):** watchlist + tóm tắt cuối
12. **Render full MD:** O pack ghép 6 trục + Review section (full eval / short / skip tuỳ Stage 0 path) + header + disclaimer + metadata
13. **Self-audit:** chạy checklist P pack mục 8 monthly + checklist O pack mục 7 dưới
14. **Present:** xuất nội dung MD trong message

### 5.2. Weekly update mode

1. **Pre-flight HARD GATE Bước 1:** P pack compute hôm nay → tuần [N] tháng [M/YYYY]
2. **Pre-flight HARD GATE Bước 2:** P pack hỏi user có monthly active đúng tháng không
   - (a) Có → upload → tiếp Bước 3
   - (b) Chưa có → O pack render HARD GATE block (mục 6.4), wait user chọn path (i/ii/iii). Nếu (i) chuyển sang monthly mode; nếu (ii) gợi ý `P_weekly_market`; nếu (iii) override với note rõ
   - (c) Monthly tháng khác → O pack render HARD GATE block case (c), wait user chọn (i)/(ii)
3. **Pre-flight Bước 3:** P pack hỏi file W-1, Stage 0 eval, context, user view
4. **Stage 0 weekly (OPTIONAL):** nếu user chọn (a) câu 3 — P pack đọc file W-1, cross-check thesis W-1 vs actual data tuần qua, compose eval block 4 phần
5. **Render Checkpoint 0 weekly block:** O pack format eval block (mục 6.3)
6. **Wait user:** accept / refine / skip carry-forward
7. **Stage 1 — Extract monthly:** P pack đọc file monthly user upload, extract 6 trục thesis
8. **Stage 1 — Quét 6 trục tuần qua:** P pack query data tuần, so thesis, status từng trục
9. **Compose 6 trục weekly + watchlist refresh + action item**
10. **Render MD ngắn:** O pack ghép theo structure mode weekly (header tuần [N] tháng [M] + Review tuần W-1 nếu có + 6 trục + watchlist + action item + disclaimer + metadata)
11. **Self-audit:** chạy checklist P pack mục 8 weekly + checklist O pack mục 7 dưới
12. **Present:** xuất nội dung MD trong message

## 6. Block render cho checkpoint (intermediate output)

Format intermediate user-facing trong session, KHÔNG phải file MD save xuống.

**Quy tắc chung:**
- Plain markdown trong message, KHÔNG header branding, KHÔNG metadata
- Độ dài 0.5-1 trang (eval blocks có thể tới 1.5 trang nếu nhiều theme/mã)
- Kết bằng multi-choice option (3-4 lựa chọn)

### 6.1. Checkpoint 0 — Eval block monthly (Stage 0)

Heading: `─── ĐÁNH GIÁ CHIẾN LƯỢC THÁNG [N-1] — Eval block ───`

Render đủ 6 phần eval (Regime / Themes / Sectors / Watchlist / Risk / Calibration), gọn hơn render trong báo cáo cuối (mỗi phần 3-5 dòng + 1 bảng compact thay vì full table). Kết bằng multi-choice (a) Accept / (b) Refine / (c) Skip carry-forward.

Spec block theo `P_invest_strategy_00` mục 5.2 Bước 4.

### 6.2. Checkpoint 1 — Regime + Themes monthly (Stage 1)

Heading: `─── REGIME VĨ MÔ + TOP THEMES — Call sơ bộ ───`

Render regime call + định vị + 2-5 themes (mỗi theme 2-3 dòng) + multi-choice 4 option (a/b/c/d).

Spec block theo `P_invest_strategy_00` mục 5.4.

### 6.3. Checkpoint 0 — Eval block weekly (Stage 0)

Heading: `─── ĐÁNH GIÁ TUẦN W-1 — Eval block ───`

Render đủ 4 phần (Status carry-over / Watchlist W-1 tracking / Action item W-1 / Carry-forward), bảng compact. Kết bằng multi-choice (a) Accept / (b) Refine / (c) Skip carry-forward.

Spec block theo `P_invest_strategy_00` mục 6.2 Bước 4.

### 6.4. HARD GATE pre-flight block weekly

Khi user chưa upload monthly active, agent xuất block:

```
─── PRE-FLIGHT HARD GATE — Weekly Update ───

**Ngày hôm nay:** DD/MM/YYYY  
**Vị trí trong chu kỳ:** Tuần [N] của tháng [M/YYYY]

**Trạng thái monthly:**
[Theo case user trả lời câu 1 — render message tương ứng]

[Nếu case (b) — chưa có monthly]:
**Để chạy weekly update, cần thesis monthly làm parent. Đề xuất 3 hướng:**

- (i) **Recommended:** Chạy monthly cycle cho tháng [M/YYYY] trước, sau đó weekly. Tôi có thể activate luôn workflow monthly nếu bạn confirm.
- (ii) Nếu chỉ cần tracking 1 tuần ad-hoc không gắn thesis tháng, dùng pack `P_weekly_market` (báo cáo tổng hợp thông tin tuần thị trường) thay thế.
- (iii) **Override** — vẫn muốn chạy weekly độc lập, không có parent thesis. Không recommended vì thiếu context để track shift/materialize, conviction tổng thể sẽ giảm.

Bạn chọn hướng nào?

[Nếu case (c) — monthly tháng khác]:
**Monthly bạn đang có là của tháng M-1, đã decay [X] tuần qua mốc tháng. Đề xuất 2 hướng:**

- (i) **Recommended:** Chạy monthly cycle cho tháng [M/YYYY] hiện tại trước.
- (ii) **Override** — dùng monthly tháng M-1 làm parent với note "Thesis carry-over decay [X] tuần, conviction giảm 1 bậc". Báo cáo cuối sẽ có cảnh báo header.

Bạn chọn hướng nào?
```

## 7. Self-check checklist O pack

### Monthly mode

- [ ] 6 trục đầy đủ — heading H2 cho mỗi trục (kể cả trục Hold rút gọn vẫn có H2)
- [ ] Tóm tắt điều hành đứng đầu, 3-7 bullet, có conviction marker regime + horizon cho themes, đọc 30 giây hiểu báo cáo
- [ ] Tóm tắt có dòng "PM overlay note" nếu có user view HIGH-conviction hoặc Conflict
- [ ] Review tháng trước render đúng case (Full eval / Short review / Skip) tuỳ Stage 0 path đã chọn ở pre-flight
- [ ] Nếu Full eval: 6 phần đầy đủ (Regime / Themes table / Sectors table / Watchlist table / Risk map / Calibration) + note "đã user accept tại Checkpoint 0"
- [ ] Trục 1-2-3-4-5-6 đúng thứ tự, KHÔNG đảo
- [ ] Sub-section trong trục flex theo phát hiện, không ép số sub
- [ ] **Trục 3 themes — mỗi theme đủ 5 thành phần: Conviction (HIGH/MID/LOW) + Horizon (1m / 1-3m / 3-6m) + Cơ chế + Catalyst trigger + Disconfirming signals (2-3 signals cụ thể)**
- [ ] Trục 3 có bảng "Catalysts ahead" tổng hợp ngày từ themes (chỉ catalyst có ngày tương đối rõ)
- [ ] **Trục 4 sector — có bảng cross-section 24 ngành + bảng sector tilts tổng hợp (Bias + Conviction + Driver + Signal hỗ trợ + Disconfirming signal) + phân tầng chi tiết**
- [ ] Trục 4 có crowding & rotation note nếu phát hiện
- [ ] Trục 5 ba kịch bản dùng if-then trigger, **không % xác suất**
- [ ] Trục 5 risk map mỗi rủi ro 3-4 thành phần (bản chất / signal / phản ứng / [theme invalidate])
- [ ] **Trục 6 watchlist 5-12 mã, mỗi mã đủ 6 thành phần: Ticker-Ngành-Theme + Conviction + Horizon + ADV tháng + Luận điểm + Signal theo dõi + Disconfirming signal**
- [ ] Watchlist có bảng tóm tắt nếu > 7 mã
- [ ] Watchlist **không có** entry/stop/target/size/% — chỉ observation
- [ ] Wording không command (không "mua/bán/giảm tỷ trọng/stop loss/target")
- [ ] **User overlay (nếu có)**: badge inline mỗi điểm tích hợp, log đầy đủ trong metadata
- [ ] Mã user inject phải có ADV ≥ 5 tỷ; nếu không có, render reference only kèm note
- [ ] Override checkpoint đã ghi inline note ở trục liên quan
- [ ] Header branded đúng (nếu user cung cấp) hoặc plain (nếu không)
- [ ] **Disclaimer có 4 mục: Phạm vi & nguồn + Forward-looking statement + Vai trò user overlay + Suitability & quyết định cuối** (branded version)
- [ ] K hygiene: ký hiệu DB raw đã dịch (`week_score`, `industry_rank`, `vsi`, `zone code` etc.) — không lộ ra trong output
- [ ] Số liệu đã quy đổi đơn vị (BCTC tỷ đồng, % thập phân nhân 100, NN/TD tỷ đồng giữ nguyên)
- [ ] Mỗi tin có dẫn link finext.vn hoặc URL gốc
- [ ] Mỗi claim định lượng có nguồn (collection + field hoặc URL)
- [ ] **Metadata có User overlay log table (hoặc note "không có user overlay") + Source & methodology disclosure**
- [ ] File save đúng tên `invest_strategy_monthly_<YYYYMM>.md`
- [ ] Xuất nội dung MD trong message

### Weekly update mode

- [ ] **HARD GATE pre-flight pass:** agent đã compute tuần [N] tháng [M/YYYY] chính xác? User đã confirm có monthly active đúng tháng (hoặc đã chọn override path với note rõ trong header)?
- [ ] Header weekly ghi rõ "Tuần [N] của tháng [M/YYYY]" trong heading title (không chỉ ngày)
- [ ] Nếu user override (monthly tháng khác): header có dòng cảnh báo "Thesis carry-over từ tháng M-1, đã decay [X] tuần, conviction giảm 1 bậc"
- [ ] Review tuần W-1 render đúng case (Full eval / Skip / Tuần 1 tháng) tuỳ Stage 0 path
- [ ] Nếu Full eval W-1: 4 phần đầy đủ (Status carry-over / Watchlist W-1 tracking / Action item W-1 / Carry-forward) + note user accept tại Checkpoint 0
- [ ] Header rõ link đến monthly active đang tham chiếu
- [ ] Tóm tắt tuần 3-6 bullet, có "Trạng thái thesis tháng" + "PM overlay tuần" (nếu có) rõ
- [ ] 6 trục mỗi trục có status badge (Hold / Shift / Materialize)
- [ ] Trục Hold rút gọn 3-5 dòng, không ép viết dài
- [ ] Trục Shift / Materialize có mô tả cụ thể + ngụ ý cho thesis
- [ ] Watchlist refresh 4 trạng thái rõ (Hold / Watch closely / Out / Vào mới)
- [ ] **Mã "Vào mới" có đủ 6 thành phần monthly format + Trigger vào tuần này (lý do tuần qua)**
- [ ] Mã "Out" có disconfirming signal cụ thể đã materialize
- [ ] Action item 1-2 item định tính, KHÔNG command/level giá
- [ ] **User overlay (nếu có) tuần này**: badge inline + User overlay log table trong metadata
- [ ] Độ dài 3-5 trang, không phình ra monthly mode
- [ ] K hygiene + nguồn đầy đủ
- [ ] Metadata cuối có link rõ đến monthly tham chiếu + User overlay log
- [ ] File save đúng tên `invest_strategy_weekly_<YYYYMMDD>.md`

## 8. Output contract

O pack render structured content P pack sinh thành file MD final theo structure flex (mode monthly hoặc weekly update). Không thêm/bớt nội dung, không tự ý insert section.

- 6 trục H2 cố định, sub-section H3 flex theo phát hiện
- Trục Hold không ép độ dài — rút gọn nếu không có gì đặc biệt
- Trục có signal mạnh có thể đào sâu, không ép giới hạn trên
- Override checkpoint render inline note trong trục liên quan
- Branding info render theo 3 trường hợp (custom / default branded / plain)
- Mode weekly bắt buộc tham chiếu monthly active — nếu thiếu, không render

User explicit yêu cầu format khác (docx / pptx) → từ chối theo system prompt mục 4 / 9. MD final là source of truth duy nhất.
