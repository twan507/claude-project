# O_weekly_overview_00 — Render Spec Báo cáo Tổng quan Thị trường Tuần

Spec render báo cáo `weekly_overview` — broadcast tuần độc lập chuẩn institutional buy-side. **Output: MD final.** Structure **rigid 12 phần** (khác `O_vbse_strategy_00` flex 6 trục).

Reference: `P_weekly_overview_00` (master philosophy + weight balance + nguyên tắc bất biến), `P_weekly_overview_01/02/03` (workflow chi tiết), `P_weekly_overview_04` (methodology + self-audit + output contract).

> **Render binary:** MD final là source of truth. Khi user yêu cầu pptx/docx, agent render theo style đã chọn (O pack spec + branding info + user explicit, body font Roboto) — chi tiết ở `system_prompt.md` mục 4.

## 1. Input từ P pack

P pack sinh structured content cho 12 phần. O pack chỉ render, không thêm/bớt nội dung.

**Trường hợp đặc biệt:**
- Phần 2 không có file W-1 → render 1 dòng "Tuần đầu cycle, chưa có dữ liệu review"
- Sub-section 5.4 không có biến động vĩ mô đáng kể → bỏ qua, không render bảng rỗng
- Sub-section 7.x cảnh báo trap không có mã thoả → bỏ qua
- Sub-section 6.x earnings beat candidate không có ngành thoả → ghi "Tuần này không có earnings beat candidate rõ ràng"
- Checkpoint regime override → ghi inline note phần 10
- Branding info user cung cấp → render branded header + disclaimer; không có → plain header

## 2. Structure báo cáo — 12 phần rigid

**Độ dài target:** 9-11 trang MD.

```
# Báo cáo Tổng quan Thị trường Tuần — [DD/MM-DD/MM/YYYY]

## 1. Tóm tắt điều hành              (Key calls / Watch / Risk)
## 2. Review tuần trước              (scorecard, skip nếu W-1 không có)
## 3. Bối cảnh quốc tế
## 4. Thị trường Việt Nam            (aggregate 18 ngành whitelist)
## 5. Vĩ mô & hàng hoá               (institutional table 5 cột sub-section 5.4)
## 6. Biến động 18 ngành whitelist   (+ sub-section earnings beat candidate)
## 7. Top dẫn dắt 2 góc nhìn         (+ sub-section cảnh báo trap)
## 8. Tin tức & catalyst             (+ conviction impact cột bảng mapping)
## 9. Định vị VNINDEX + 3 kịch bản + Risk map
## 10. Watchlist                     (10.1 bối cảnh / 10.2 cơ hội tăng / 10.3 cảnh báo áp lực)
## 11. Lịch sự kiện tuần tới         (+ conviction impact cột)
## 12. Tuyên bố miễn trừ trách nhiệm
— Metadata (cuối file)
```

**Quy tắc structure:**
- Đứng thứ tự cố định 12 phần
- Phần rỗng vẫn render với 1 dòng note (không bỏ phần ngoài Phần 2 nếu W-1 không có thì render note)
- Sub-section flex (vd 5.4 có thể bỏ qua nếu không có biến động)
- Mỗi phần H2 (##), sub-section H3 (###), sub-sub-section H4 (####)

## 3. Compose từng phần

### 3.1. Header

**Format plain (không branding):**

```
# Báo cáo Tổng quan Thị trường Tuần — [DD/MM-DD/MM/YYYY]

**Phát hành:** [DD/MM/YYYY] (Tối Chủ Nhật / Sáng thứ Hai)  
**Phạm vi:** Thị trường cổ phiếu Việt Nam (VNINDEX, HSX/HNX/UPCOM) — 18 ngành whitelist  
**Tuần báo cáo:** Thứ Hai DD/MM đến Chủ Nhật DD/MM/YYYY  
**Audience:** Phân tích nội bộ
```

**Format branded:**

```
# [TÊN CÔNG TY]
## BÁO CÁO TỔNG QUAN THỊ TRƯỜNG TUẦN
### Tuần [DD/MM-DD/MM/YYYY]

**Phát hành:** [DD/MM/YYYY]  
**Phòng ban biên soạn:** [user cung cấp]  
**Hotline:** [user cung cấp] | **Website:** [user cung cấp]

---

> [Disclaimer ngắn user cung cấp, hoặc default: "Tài liệu nhằm mục đích thông tin tham khảo, không phải khuyến nghị mua bán chứng khoán. Khách hàng cần tự cân nhắc trước khi quyết định đầu tư."]

---
```

### 3.2. Phần 1 — Tóm tắt điều hành (Key calls / Watch / Risk)

**Cấu trúc institutional weekly broadcast:**

```
## 1. Tóm tắt điều hành

### Key calls (HIGH conviction)
- **Regime tuần tới:** [risk-on full / selective / defensive / đứng ngoài] — Conviction HIGH — [1 dòng căn cứ chính từ 4 input]
- **Sector quan tâm dẫn đầu:** [Ngành A] (whitelist 18) — Conviction HIGH — [1 dòng cơ bản+catalyst]
- **Sector quan tâm thứ 2:** [Ngành B] — Conviction HIGH — [1 dòng]
- **Mã tiêu biểu:** [Ticker X] (Ngành A) — Conviction HIGH — Horizon 1-2 tuần — [1 dòng catalyst chính]

### Watch (MID conviction — đang theo dõi)
- **Theme nổi:** [theme tuần] — [1 dòng cơ chế + signal cần xác nhận]
- **Sector chuyển pha:** [Ngành C đang chuyển từ trung tính sang quan tâm] — [1 dòng signal đang cần xác nhận]
- **Mã đang gom kín** (nếu có từ phần 7): [Ticker Y] — [1 dòng]

### Risk (chính tuần tới)
- **Rủi ro #1:** [tên rủi ro từ Risk map phần 9.4] — Signal materialize: [chỉ báo cụ thể PREFER macro/fundamental] — Phản ứng định tính: [...]
- **Rủi ro #2** (optional): [tương tự]

### PM overlay note (optional)
- [Chỉ render khi có user view inject ở conviction HIGH hoặc Conflict chưa resolve]
```

**Quy tắc viết:**
- Đứng đầu báo cáo, đọc 30-60 giây hiểu toàn key calls
- Mỗi bullet 1 dòng, ngắn gọn
- Conviction marker bắt buộc cho mỗi Key call
- Horizon marker bắt buộc cho mã tiêu biểu
- Risk bullet bắt buộc kèm signal materialize cụ thể
- Không quá 8 bullet tổng

### 3.3. Phần 2 — Review tuần trước (scorecard)

**3 trường hợp render:**

**(a) Có file W-1 — full scorecard:**

```
## 2. Review tuần trước

### 2.1. Scorecard kịch bản match

| Kịch bản W-1 | Trigger gốc | Thực tế tuần | Match (✓/✗) |
|---|---|---|---|
| Cơ sở | [trigger] | [thực tế] | ✓ |
| Tích cực | [trigger] | [thực tế] | ✗ |
| Tiêu cực | [trigger] | [thực tế] | ✗ |

→ Kịch bản match: **[Cơ sở / Tích cực / Tiêu cực / Lệch khỏi cả 3]**

### 2.2. Scorecard sector bias

**Ngành quan tâm W-1:**
| Ngành | Conviction W-1 | m_pct thực tế | Hit/Miss |
|---|---|---|---|
| [...] | HIGH | +5.2% | Hit ✓ |

**Ngành cần thận trọng W-1:**
| Ngành | Conviction W-1 | m_pct thực tế | Hit/Miss |
|---|---|---|---|
| [...] | HIGH | -3.8% | Hit ✓ |

**Hit rate tổng:** [N/M] ngành quan tâm tăng giá, [N/M] ngành thận trọng giảm giá.

### 2.3. Scorecard watchlist

| Ticker | Conviction W-1 | Horizon W-1 | m_pct thực tế | Signal trigger? | Hit/Miss |
|---|---|---|---|---|---|
| [...] | HIGH | 1-2 tuần | +4.1% | Có (BCTC Q1 +22%) | Hit ✓ |

### 2.4. Best/Worst call + Carry-forward

- **Best call W-1:** [1-2 dòng — call nào đúng nhất, lý do calibration đúng]
- **Worst call W-1:** [1-2 dòng honest — call nào sai nhất, lý do calibration sai]
- **Carry-forward vào tuần mới:** [1-3 dòng learning chính, đặc biệt note nếu worst call gắn với pattern lặp lại]
```

**(b) Không có file W-1:** render 1 dòng:

```
## 2. Review tuần trước

Tuần đầu cycle / không có file W-1, chưa có dữ liệu review.
```

### 3.4. Phần 3 — Bối cảnh quốc tế

Sub-section flex. Cap technical = 0%.

```
## 3. Bối cảnh quốc tế

### 3.1. Chứng khoán quốc tế

| Chỉ số | Đóng cửa | % tuần | % tháng |
|---|---|---|---|
| S&P 500 | [...] | [...] | [...] |
| Dow Jones | [...] | [...] | [...] |
| Nasdaq | [...] | [...] | [...] |
| Nikkei 225 | [...] | [...] | [...] |
| Shanghai Composite | [...] | [...] | [...] |

### 3.2. Tỷ giá quốc tế

| Cặp | Tỷ giá | % tuần | % tháng |
|---|---|---|---|
| EUR/USD | [...] | [...] | [...] |
| GBP/USD | [...] | [...] | [...] |
| USD/JPY | [...] | [...] | [...] |
| USD/CNY | [...] | [...] | [...] |
| DXY | [...] | [...] | [...] |

### 3.3. Lãi suất quốc tế

| Chỉ tiêu | Mức | % tuần | Ghi chú |
|---|---|---|---|
| FFR (Fed) | [...] | [...] | [...] |
| ECB DFR | [...] | [...] | [...] |
| PBOC LPR | [...] | [...] | [...] |
| TPCP Mỹ 10Y | [...] | [...] | [...] |

### 3.4. Tổng kết

[Prose 3-4 dòng: tâm lý chung quốc tế, áp lực USD, kỳ vọng lãi suất, ngụ ý cho EM equity flow. Câu cuối kèm implied conviction (HIGH/MID/LOW) cho forward bias EM equity.]
```

### 3.5. Phần 4 — Thị trường Việt Nam (aggregate 18 ngành whitelist)

```
## 4. Thị trường Việt Nam

> *Aggregate proxy thị trường tính trên 18 ngành whitelist, không 24 ngành raw — theo Nguyên tắc 2 master.*

### 4.1. VNINDEX

| Chỉ tiêu | Giá trị |
|---|---|
| Giá đóng cửa phiên cuối tuần | [...] |
| % tuần | [...] |
| % tháng | [...] |
| % quý | [...] |
| % năm | [...] |

### 4.2. Thanh khoản

| Chỉ tiêu | Giá trị |
|---|---|
| GTGD phiên cuối tuần (tỷ đồng) | [...] |
| GTGD trung bình tuần (tỷ đồng) | [...] |
| So trung bình 1 tháng | [tăng/giảm X%] |

### 4.3. Dòng tiền nội thị trường (aggregate 18 ngành whitelist)

- **Điểm dòng tiền tuần (proxy aggregate 18 ngành):** [định tính + số trung bình mean/median]
- **Chuỗi 5 phiên (proxy aggregate 18 ngành):** [mô tả pattern — đồng đều dương / đồng đều âm / dao động / phục hồi cuối tuần]

### 4.4. Breadth phiên cuối tuần

- Số ngành whitelist 18 tăng/giảm: [N tăng / M giảm / K trung tính]
- Số mã toàn thị trường tăng/giảm: [N tăng / M giảm] (từ `data_briefing`)

### 4.5. Khối ngoại/Tự doanh tuần

| | Mua ròng/Bán ròng tuần | Top 5 mã |
|---|---|---|
| Khối ngoại | [tỷ đồng] | [list 5 ticker mua + 5 ticker bán] |
| Tự doanh | [tỷ đồng] | [list 5 ticker mua + 5 ticker bán] |

### 4.6. Lãi suất + tỷ giá VN

| Chỉ tiêu | Giá trị | Biến động tuần |
|---|---|---|
| Lãi suất tái cấp vốn | [...] | [...] |
| Lãi suất liên ngân hàng 2W/1M/3M | [...] | [...] |
| Tỷ giá USD/VND VCB | [...] | [...] |
```

### 3.6. Phần 5 — Vĩ mô & hàng hoá (institutional table 5 cột)

```
## 5. Vĩ mô & hàng hoá — Yếu tố dẫn dắt ngành

### 5.1. Lãi suất

[Bảng số liệu lãi suất điều hành / liên ngân hàng / huy động / cho vay + 2-3 dòng diễn giải biến động tuần qua]

### 5.2. Tỷ giá

[Bảng số liệu tỷ giá VN + quốc tế đáng chú ý + 2-3 dòng diễn giải]

### 5.3. Hàng hoá

[Bảng phân theo nhóm — kim loại / năng lượng / nông sản / hoá chất — số liệu thuần, không có cột "ngành nhạy" trong bảng này]

### 5.4. Tác động lên 18 ngành VN whitelist tuần này

**Bảng chỉ liệt kê chỉ số có biến động Material+ trở lên hoặc Persistence Trending+ trở lên:**

| Chỉ số biến động đáng kể (tuần) | Magnitude | Persistence | Ngành VN whitelist 18 bị tác động | Hướng + cơ chế (1 dòng) |
|---|---|---|---|---|
| Brent +12% tuần (đạt 92 USD) | Material | Trending | DAUKHI (tích cực — biên upstream) | + biên gộp dầu khí Q2 ước tăng 50bp; VANTAI (tiêu cực — chi phí nhiên liệu) |
| Quặng sắt -8% tuần | Material | Transitory | KIMLOAI (tích cực biên gộp) | - biên gộp thép tuần tới có thể cải thiện ~30bp |
| ... | ... | ... | ... | ... |

**Quy tắc Magnitude:**
- Small: trong phạm vi dao động bình thường
- Material: ngoài phạm vi bình thường, chưa shock
- Significant: shock — vượt ngưỡng tâm lý hoặc pattern đảo chiều rõ

**Quy tắc Persistence:**
- Transitory: 1 lần, có thể revert
- Trending: xu hướng đang hình thành 1-2 tháng
- Structural: shift cấu trúc dài hạn

Ngành cột 4 BẮT BUỘC thuộc whitelist 18. Cơ chế chạm ngành ngoài whitelist note "(ngoài scope)".

Nếu tuần không có biến động đáng kể, bỏ qua sub-section 5.4 — không render bảng rỗng.
```

### 3.7. Phần 6 — Biến động 18 ngành whitelist

```
## 6. Biến động 18 ngành whitelist

### 6.1. Bảng 18 ngành — Cross-section

Sort theo rank tự tổng hợp (theo `week_score` giảm dần) trong scope 18, cột:

| # | Ngành | w_pct | m_pct | P/E hiện tại | P/E phân vị 3Y | EPS Q YoY | P/B hiện tại | week_score | Rank (1..18 tự tổng hợp) |
|---|---|---|---|---|---|---|---|---|---|
| 1 | NGANHANG | +3.2% | +5.1% | 14x | 28% (rẻ) | +18% | 1.4x | +12.4 | 1/18 |
| 2 | BANLE | +2.8% | +4.6% | 16x | 25% (rẻ) | +20% | 2.1x | +11.2 | 2/18 |
| ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |
| 18 | BDS | -2.1% | -4.8% | 22x | 78% (đắt) | -8% | 1.8x | -8.7 | 18/18 |

**P/E phân vị 3Y:** rẻ tương đối (<40%) / trung tính (40-70%) / đắt tương đối (>70%) — re-rating opportunity hay risk.

### 6.2. Diễn giải

[Prose 4-6 dòng:
- Top ngành rank cao + cross-check sub-section 5.4 phần 5: ngành nào nằm giao của (rank cao + vĩ mô ủng hộ) = dẫn dắt thật candidate
- Phát hiện phân kỳ:
  - Ngành giá tuần dương rõ nhưng week_score thấp = nghi trụ kéo
  - Ngành giá đi ngang + week_score cao + vĩ mô ủng hộ = tích luỹ chuẩn bị bứt
- Ngành rank thấp + áp lực vĩ mô = cần thận trọng candidate]

### 6.3. Top 3 "Earnings beat candidate"

**Ngành đáp ứng cả 3 tiêu chí:**
- EPS Q YoY ≥ 15%
- P/E phân vị < 40%
- week_score dương cải thiện 2 tuần liên tiếp

| Ngành | EPS Q YoY | P/E phân vị | week_score | Diễn giải 1 dòng |
|---|---|---|---|---|
| NGANHANG | +18% | 28% | +12.4 (tăng từ +8.1 tuần trước) | NIM cải thiện + định giá đang rẻ, ứng viên surprise Q2 |

[Nếu không có ngành nào pass, ghi "Tuần này không có earnings beat candidate rõ ràng".]
```

### 3.8. Phần 7 — Top dẫn dắt 2 góc nhìn + cảnh báo trap

```
## 7. Top dẫn dắt — 2 góc nhìn

### 7.1. Top biến động giá tuần

**Top 5 tăng:**
| Ticker | Ngành | % tuần | GTGD TB tuần (tỷ) |
|---|---|---|---|

**Top 5 giảm:**
| Ticker | Ngành | % tuần | GTGD TB tuần (tỷ) |
|---|---|---|---|

### 7.2. Top 10 dòng tiền tuần

| Ticker | Ngành | week_score | % tuần | GTGD TB tuần (tỷ) |
|---|---|---|---|---|

[Edge case tuần thị trường yếu — nếu top 10 chứa week_score ≤ 0, ghi note dưới bảng: "Tuần thị trường yếu toàn diện, top 10 dòng tiền có mã week_score âm/bằng 0 — đây là 'ít yếu nhất' chứ không phải dẫn dắt thực sự."]

### 7.3. Cross-check 3 nhóm

- **Nhóm 1 (dẫn dắt thật, có cả lực giá + dòng tiền — cả 2 list):** [list ticker]
- **Nhóm 2 (gom kín, dòng tiền cao + giá chưa chạy):** [list ticker]
- **Nhóm 3 (chạy nhanh, biến động giá cao + dòng tiền không cao):** [list ticker]

### 7.4. Cảnh báo "Late money / Trap setup"

**Mã trong top dòng tiền (7.2) mà thuộc ngành cần thận trọng** (preview từ ngành rank thấp + áp lực vĩ mô):

| Ticker | Ngành (rank thấp) | week_score | Cảnh báo (1 dòng) |
|---|---|---|---|
| [...] | BDS (rank 18/18) | +8.2 | Late money có thể; tin xấu chính sách thắt tín dụng chưa lộ rõ |

[Nếu không có mã nào thoả, bỏ sub-section 7.4. Đây là warning sign chuẩn buy-side cho retail trap setup.]
```

### 3.9. Phần 8 — Tin tức & catalyst (+ conviction impact)

```
## 8. Tin tức & catalyst tuần

### 8.1. Tin trong nước impact ngành (3-5 tin)

- [Tin 1] — [ngành whitelist 18 ảnh hưởng] — [conviction impact] — [link finext.vn]
- ...

### 8.2. Tin quốc tế ảnh hưởng VN (2-3 tin)

- [Tin 1] — [ngành VN whitelist 18 / equity flow general] — [conviction impact] — [URL]
- ...

### 8.3. Bảng mapping tin → ngành — UPGRADED conviction impact

| Tin / sự kiện | Ngành VN ảnh hưởng (whitelist 18) | Hướng tác động | Conviction impact (LOW/MID/HIGH) | Cơ chế 1 dòng |
|---|---|---|---|---|
| [Nghị quyết X về Y thông qua DD/MM] | NGANHANG | Tích cực | HIGH | Mở dư địa tăng trưởng tín dụng Q2-Q3 |
| ... | ... | ... | ... | ... |

**Conviction impact:**
- **HIGH:** tin đã materialize + cơ chế tác động trực tiếp + magnitude lớn
- **MID:** tin cần verify thêm hoặc cơ chế tác động gián tiếp
- **LOW:** tin sớm, signal yếu

Mỗi tin có dẫn link `https://finext.vn/news/<slug>` hoặc URL gốc. Ngành cột 2 BẮT BUỘC whitelist 18; ngoài note "(ngoài scope)".
```

### 3.10. Phần 9 — Định vị VNINDEX + 3 kịch bản + Risk map

```
## 9. Định vị VNINDEX + Kịch bản tuần tới

### 9.1. Diễn biến giá tuần

[Prose 3-4 dòng: nến cuối tuần, vị thế MA, biên độ tuần/tháng/quý. Render technical pure — không tính cap.]

### 9.2. Vùng giá tham chiếu

Bảng kháng cự + hỗ trợ 4 khung:

| Khung | Kháng cự gần | Hỗ trợ gần | Breakout level (xác nhận tích cực) | Breakdown level (xác nhận tiêu cực) |
|---|---|---|---|---|
| Tuần | [...] | [...] | [...] | [...] |
| Tháng | [...] | [...] | [...] | [...] |
| Quý | [...] | [...] | [...] | [...] |
| Năm | [...] | [...] | [...] | [...] |

(Render technical pure cho user reference — không tính cap.)

### 9.3. Ba kịch bản tuần tới (fundamental-driven)

**Trigger primary BẮT BUỘC là macro/fundamental/policy/catalyst. Technical chỉ confirmation phụ ≤30%.**

**Kịch bản cơ sở:**
- **Trigger primary (macro/fundamental/policy):** [vd "Fed giữ rates + lãi suất liên ngân hàng VN ổn định + không có shock chính sách + tin trong nước trung tính"]
- **Confirmation phụ (technical/flow):** [vd "VNINDEX dao động trong band POC quý, breadth tổng thể duy trì >50%"]
- **Vùng VNINDEX dự kiến tuần tới:** [X] — [Y]
- **Hành vi kỳ vọng:** [định tính ngắn]

**Kịch bản tích cực:**
- **Trigger primary:** [vd "FOMC minutes dovish surprise / BCTC Q1 ngân hàng beat consensus ≥10% / Nghị quyết X thông qua / FII chuyển mua ròng ≥X nghìn tỷ tuần"]
- **Confirmation phụ:** [vd "VNINDEX đóng cửa trên POC quý + breadth > 60% + điểm dòng tiền phiên dương 3 phiên"]
- **Vùng VNINDEX kỳ vọng:** [Y]
- **Sector/theme củng cố:** [list]

**Kịch bản tiêu cực:**
- **Trigger primary:** [vd "Fed hawkish surprise / BCTC Q1 nhiều top sector miss / USD/VND vượt 26800 + NHNN can thiệp / Chính sách thắt tín dụng X ban hành"]
- **Confirmation phụ:** [vd "VNINDEX đóng cửa dưới hỗ trợ kép + breadth < 35% + FII bán ròng tăng cường"]
- **Vùng hỗ trợ kỳ vọng:** [Y]
- **Sector/theme bị impact:** [list]

> *Kịch bản là hệ thống điều kiện vĩ mô/cơ bản/chính sách + confirmation technical, không phải dự báo chắc chắn. Không gán % xác suất. Diễn biến thực tế có thể lệch khỏi cả 3 nếu xuất hiện sự kiện ngoài kỳ vọng.*

### 9.4. Risk map — 3-7 rủi ro

**Rủi ro 1 — [Tên ngắn]**
- **Bản chất + cơ chế cơ bản:** [2-3 dòng]
- **Signal materialize (PREFER macro/fundamental):** [chỉ báo / sự kiện / mức số cụ thể]
- **Phản ứng định tính:** [vd "giảm exposure ngành X" / "chuyển defensive" / "đứng ngoài chờ"]
- **Theme/sector bị invalidate (cross-link phần 10):** [list]

[3-7 rủi ro tổng — flex theo bối cảnh.]
```

### 3.11. Phần 10 — Watchlist (tách 2 hướng + conviction + horizon + disconfirming)

```
## 10. Watchlist tuần tới

### 10.1. Bối cảnh sector bias

- **Regime tuần tới:** [risk-on full / selective / defensive / đứng ngoài] — Conviction HIGH/MID/LOW
- **Ngành quan tâm:** [Ngành A — Conviction HIGH — 1 dòng lý do], [Ngành B — Conviction HIGH — 1 dòng], ...
- **Ngành cần thận trọng:** [Ngành X — Conviction HIGH — 1 dòng], [Ngành Y — Conviction MID — 1 dòng], ...
- **Risk-reward định tính tuần tới:** [rủi ro xuống lớn hơn tiềm năng tăng / cân bằng / tiềm năng tăng có ưu thế]

### 10.2. Mã cơ hội tăng (3-5 mã)

**[Ticker A]** (Ngành whitelist 18)
- **Luận điểm** (cơ bản-first): [1 dòng cơ chế cơ bản + catalyst]
- **Conviction:** HIGH   |   **Horizon:** 1-2 tuần
- **Signal theo dõi** (PREFER cơ bản/catalyst/chính sách):
  - "BCTC Q1 release tuần tới, EPS consensus +18%"
  - "Dòng tiền tuần dương 2 tuần liên tiếp"
  - "Ngày chốt cổ tức 7% — DD/MM"
- **Disconfirming signal:** "BCTC Q1 EPS miss consensus ≥ 10%"

**[Ticker B]** (Ngành whitelist 18)
- ... [same format]

[3-5 mã tổng. Mỗi mã 4 dòng đầy đủ.]

### 10.3. Mã cảnh báo áp lực (2-3 mã)

**[Ticker X]** (Ngành whitelist 18 — cần thận trọng)
- **Luận điểm** (hướng tiêu cực): [1 dòng "áp lực từ X → mã chịu nặng" / "cơ bản Q1 có thể yếu hơn kỳ vọng"]
- **Conviction (áp lực):** HIGH   |   **Horizon:** 1-2 tuần
- **Signal theo dõi** (PREFER cơ bản/macro/catalyst):
  - "BCTC Q1 EPS dự kiến miss"
  - "Dòng tiền tuần âm tiếp diễn"
  - "Chính sách thắt X có hiệu lực DD/MM"
- **Disconfirming signal** (điều kiện xoá áp lực): "Nghị quyết nới X được ban hành"

[2-3 mã tổng.]

### 10.4. Bucket entry (OPTIONAL — chỉ render khi user request)

[Nếu user yêu cầu "thêm bucket entry cho watchlist", thêm cột Bucket (1/2/3) cho mỗi mã 10.2 + 10.3 theo định nghĩa sau. Default OFF — không render trừ khi explicit request.]

| Bucket | Điều kiện | Observation |
|---|---|---|
| **1 — Vào ngay được** | zone w ∈ {A, AA, AAA} VÀ zone m ∈ {A, AA, AAA} VÀ week_score ≥ 6 | Momentum đa khung đồng thuận. |
| **2 — Chờ xác nhận (pullback)** | zone q ∈ {A, AA, AAA} HOẶC zone y ∈ {A, AA, AAA} NHƯNG zone w HOẶC m ∈ {B, C} | Uptrend dài hạn còn nguyên, ngắn hạn pullback. |
| **3 — Watchlist (chưa sẵn)** | zone q ∈ {A, AA, AAA} HOẶC zone y ∈ {A, AA, AAA} NHƯNG zone w VÀ m ĐỀU ∈ {C} | Pullback sâu, chờ phục hồi. |

**Wording rules toàn phần 10:**
- KHÔNG dùng từ command: "mua / bán / giảm tỷ trọng / stop loss / target"
- KHÔNG có entry zone / stop / target / size cụ thể
- Diễn đạt qua catalyst + flow + sự kiện sắp tới
- Mã user inject (overlay): render cùng format + badge `[PM input — Confirm/Partial/Conflict/Flag]`
```

### 3.12. Phần 11 — Lịch sự kiện tuần tới (+ conviction impact)

```
## 11. Lịch sự kiện tuần tới

### 11.1. Lịch macro

| Ngày | Sự kiện | Ngành VN whitelist 18 ảnh hưởng | Conviction impact (LOW/MID/HIGH) |
|---|---|---|---|
| Thứ Tư DD/MM | FOMC minutes | Toàn thị trường (qua DXY + FII) | HIGH |
| Thứ Năm DD/MM | CPI Mỹ tháng [N] | Toàn thị trường | MID |
| ... | ... | ... | ... |

### 11.2. Lịch corporate

| Ngày | Ticker | Sự kiện (BCTC/ĐHCĐ/M&A/etc.) | Conviction impact |
|---|---|---|---|
| Thứ Hai DD/MM | [Ticker] | BCTC Q1 công bố | HIGH |
| Thứ Tư DD/MM | [Ticker] | Ngày chốt cổ tức 7% | MID |
| ... | ... | ... | ... |

Lịch chỉ liệt kê sự kiện đáng chú ý. Ngành/mã thuộc whitelist 18 ưu tiên; sự kiện chạm ngành ngoài note "(ngoài scope)".
```

### 3.13. Phần 12 — Tuyên bố miễn trừ trách nhiệm

**(a) Custom disclaimer:** render full text user cung cấp + khối liên hệ.

**(b) Default branded:**

```
## 12. Tuyên bố miễn trừ trách nhiệm

**Phạm vi & nguồn dữ liệu**

Báo cáo này được [TÊN CÔNG TY] soạn thảo trên cơ sở các thông tin, dữ liệu được thu thập từ các nguồn được coi là đáng tin cậy vào thời điểm phát hành. Dữ liệu định lượng chính được trích từ hệ thống dữ liệu nội bộ (`agent_db`) trong phạm vi **18 ngành whitelist**, bổ sung bởi web search cho tin tức và sự kiện cập nhật. [TÊN CÔNG TY] không đảm bảo tuyệt đối về tính chính xác, đầy đủ của các thông tin này.

**Forward-looking statement**

Các nhận định, kịch bản, conviction level, horizon và disconfirming signals trong báo cáo này phản ánh **quan điểm độc lập tại thời điểm công bố** (horizon 1-4 tuần), **không phải dự báo chắc chắn** về diễn biến thị trường. Điều kiện thị trường có thể thay đổi do các sự kiện ngoài kỳ vọng, và các thesis có thể bị invalidate nếu disconfirming signals materialize.

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
## 12. Tuyên bố miễn trừ trách nhiệm

> ⚠️ **Lưu ý:** Báo cáo render bản plain do không có branding. Trước khi gửi khách hàng, cần bổ sung disclaimer phù hợp với pháp lý của tổ chức phát hành.

Báo cáo này phản ánh quan điểm phân tích nội bộ tại thời điểm soạn thảo, có tính horizon 1-4 tuần, trong phạm vi 18 ngành whitelist. Có thể thay đổi khi điều kiện thị trường shift hoặc disconfirming signals materialize.

Nhà đầu tư tự cân nhắc dựa trên tình hình tài chính cá nhân, mục tiêu và horizon đầu tư.

**Ngày phát hành:** [DD/MM/YYYY]
```

### 3.14. Metadata cuối file

```
---

**Metadata**

- **Tuần báo cáo:** Thứ Hai DD/MM đến Chủ Nhật DD/MM/YYYY
- **Ngày phát hành:** [DD/MM/YYYY]
- **Horizon:** 1-4 tuần forward
- **Whitelist ngành scope:** 18 ngành (tham khảo `K_agent_db_01` Section B)
- **Regime call:** [định tính] — Conviction HIGH/MID/LOW
- **Sector quan tâm:** [list ngành whitelist + conviction từng ngành]
- **Sector cần thận trọng:** [list + conviction]
- **Watchlist cơ hội tăng:** [list ticker + conviction + horizon]
- **Watchlist cảnh báo áp lực:** [list ticker + conviction + horizon]
- **Override checkpoint:** [có/không + lý do]
- **File W-1 đã tham chiếu:** [tên file / "không có"]

**User overlay log** (chỉ render khi có user view inject):

| # | Channel | View user nêu | Trục liên quan | Trạng thái xử lý | Vị trí integrate |
|---|---|---|---|---|---|
| 1 | Pre-flight câu 2 | [...] | Phần 8 + 10 | Confirm | Phần 8 tin + Phần 10.2 ticker A |
| 2 | Checkpoint override | [...] | Regime call | Override | Phần 10.1 + log |

(Nếu không có user view: "Không có user overlay trong cycle này — báo cáo build hoàn toàn từ data agent_db + web search.")

**Source & methodology disclosure:**
- **Source dữ liệu chính:** agent_db ([list collection chính đã query])
- **Web search bổ sung:** [có/không + chủ đề chính]
- **Methodology framework:** Weekly Overview 12 phần. Regime classification định tính 4 mức theo `P_weekly_overview_04` mục 1. Sector bias trong scope 18 whitelist. Conviction HIGH-MID-LOW + horizon 1-4 tuần + disconfirming signal bắt buộc mỗi call. Trigger 3 kịch bản fundamental-driven (Nguyên tắc 1 master). Không gán % xác suất kịch bản (tuân `K_agent_db_00` mục 4.3).
```

## 4. Convention chung

### 4.1. K hygiene + ngôn ngữ

- Ký hiệu DB raw đã dịch sang ngôn ngữ tự nhiên (`week_score: 18` → "điểm dòng tiền tuần 18 — dương rõ", rank tự tổng hợp "3" → "xếp hạng dòng tiền 3/18 ngành whitelist")
- KHÔNG dùng từ command
- Conviction: HIGH / MID / LOW
- Horizon: 1-2 tuần / 2-4 tuần
- Bias: Quan tâm / Trung tính / Cần thận trọng
- Magnitude: Small / Material / Significant
- Persistence: Transitory / Trending / Structural
- Trend indicator: CẤM dùng (`market_snapshot.trend`, `industry_snapshot.trend`, `group_snapshot.trend`, `*_recent.recent_trend`)

### 4.2. Source citation

- Mỗi claim định lượng có nguồn: collection + field hoặc URL web search
- Tin có dẫn link `https://finext.vn/news/<slug>` hoặc URL gốc

### 4.3. Whitelist 18 ngành scope

- Bảng 18 ngành phần 6: đủ 18
- Sector bias phần 10: chỉ trong 18 whitelist
- Watchlist mã: chỉ thuộc 18 whitelist
- Aggregate proxy thị trường phần 4: tính trên 18, không 24
- Ngành ngoài whitelist KHÔNG render trong báo cáo (chỉ note "(ngoài scope)" nếu cơ chế chạm tới)

### 4.4. Forward-looking

- Mọi kịch bản, conviction, horizon, disconfirming là **observation có cơ sở data**, không phải dự báo chắc chắn
- Không gán % xác suất
