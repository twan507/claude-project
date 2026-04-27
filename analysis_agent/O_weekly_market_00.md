# O_weekly_market_00 — Render Spec Báo cáo tuần thị trường

Spec render báo cáo tuần thị trường VN. **Rigid structure** — 12 phần thứ tự cố định, heading exact. **Output cuối: MD final** (đã apply structure + K hygiene + citation + chart annotation YAML).

Reference: `P_weekly_market_00` (workflow + methodology).

> **Render binary out of scope:** Pack này dừng ở MD final. Render pptx/docx/xlsx là concern downstream, không thuộc scope. MD final đã đủ structured (heading + chart YAML + citation + locale) để consume bằng tool render bên ngoài. Mục pptx/docx ở dưới (nếu có) là legacy spec, sẽ được dọn ở audit pass tiếp theo.

## 1. Input từ P pack

P pack sinh structured content cho 12 phần theo spec ở mục 5-7 của P_weekly_market_00. O pack chỉ render, không thêm/bớt nội dung.

Trường hợp đặc biệt:
- Phần 2 user không gửi file W-1 → render 1 dòng "Tuần đầu cycle, chưa có dữ liệu review tuần trước." Heading vẫn giữ.
- Checkpoint 1 user override regime/sector bias → render note inline trong phần 10 sub-section 10.1: "Regime chốt sau review: [X]. Override note: [lý do]."
- Phần 12 disclaimer render theo branding info user cung cấp ở pre-flight (3 trường hợp: custom disclaimer / default branded / plain — xem `P_weekly_market_00` mục 7.3).

## 2. Structure báo cáo đầu ra

**Độ dài target:** 9-11 trang MD.

**Rigid structure bắt buộc** (thứ tự + heading exact):

```
# Báo cáo thị trường tuần — <DD/MM/YYYY> đến <DD/MM/YYYY>

## 1. Tóm tắt điều hành
## 2. Review tuần trước
## 3. Bối cảnh quốc tế
## 4. Thị trường Việt Nam
## 5. Vĩ mô & hàng hoá — yếu tố dẫn dắt ngành
## 6. Biến động ngành
## 7. Top dẫn dắt
## 8. Tin tức & catalyst
## 9. Phân tích kỹ thuật VNINDEX + 3 kịch bản + Risk map
## 10. Watchlist — Mã đáng chú ý
## 11. Lịch sự kiện tuần tới
## 12. Tuyên bố miễn trừ trách nhiệm
— Metadata
```

12 phần đầy đủ kể cả phần rỗng. Heading viết exact, không đổi từ ngữ.

## 3. Compose từng phần chi tiết

### 3.1. Header báo cáo

Format mặc định (không có branding):

```
# Báo cáo thị trường tuần — DD/MM/YYYY đến DD/MM/YYYY

**Phát hành:** [DD/MM/YYYY] (sáng thứ Hai trước phiên)  
**Phạm vi:** Thị trường cổ phiếu Việt Nam (VNINDEX, HSX/HNX/UPCOM)  
```

**Format có branding** (khi user cung cấp ở pre-flight câu 3):

```
# [TÊN CÔNG TY]
## BÁO CÁO THỊ TRƯỜNG TUẦN
### DD/MM/YYYY đến DD/MM/YYYY

**Phát hành:** [DD/MM/YYYY]  
**Phòng ban biên soạn:** [user cung cấp]  
**Phạm vi:** Thị trường cổ phiếu Việt Nam (VNINDEX, HSX/HNX/UPCOM)  
**Hotline:** [user cung cấp] | **Website:** [user cung cấp]

---

> [Nội dung disclaimer ngắn user cung cấp — thường 1-2 dòng kiểu "Tài liệu tham khảo, không phải khuyến nghị đầu tư." Nếu user không cung cấp custom thì dùng default: "Tài liệu này nhằm mục đích thông tin tham khảo, không phải khuyến nghị mua bán chứng khoán. Khách hàng cần tự cân nhắc trước khi quyết định đầu tư."]

---
```

Logo: nếu user cung cấp file logo, render theo cú pháp markdown image `![logo](path)` ở đầu header. Nếu không có thì bỏ.

### 3.2. Phần 1 — Tóm tắt điều hành

Format bullet list, 3-5 bullet, mỗi bullet 1-2 dòng:

```
- **Regime tuần:** [risk-on full / risk-on selective / defensive only / đứng ngoài]. [1 dòng tóm tắt căn cứ chính.]
- **Catalyst lớn nhất:** [tin/sự kiện 1] [+ tin 2 nếu có].
- **Sector bias:** Quan tâm [danh sách ngành ngắn gọn, viết tên ngành đầy đủ]. Thận trọng [danh sách ngành].
- **Risk chính tuần tới:** [1 rủi ro nổi bật từ Risk map phần 9.4].
- **Mã đáng chú ý:** [1 mã từ watchlist phần 10 + 1 dòng luận điểm, optional].
```

### 3.3. Phần 2 — Review tuần trước

**Nếu không có file W-1:**

```
## 2. Review tuần trước

Tuần đầu cycle, chưa có dữ liệu review tuần trước.
```

**Nếu có file W-1:**

```
## 2. Review tuần trước

**Kịch bản đã match:** [cơ sở / tích cực / tiêu cực / lệch khỏi cả 3]  
[1-2 dòng mô tả thực tế tuần qua so với 3 kịch bản đã đưa ra]

**Hit rate:**
- Sector bias quan tâm: [N/M] ngành tăng giá
- Watchlist: [N/M] mã chạy đúng luận điểm (cả mã hướng tích cực và hướng tiêu cực)

**Rủi ro materialize:** [Liệt kê rủi ro nào trong risk map W-1 đã xảy ra, hoặc "Không có"]

**Learning chính cho tuần tới:** [1-2 dòng]
```

### 3.4. Phần 3 — Bối cảnh quốc tế

3 sub-section + tổng kết prose.

```
## 3. Bối cảnh quốc tế

### 3.1. Chứng khoán quốc tế

| Chỉ số | Giá trị | Biến động tuần | Biến động tháng |
|---|---|---|---|
| Dow Jones | [X] | [+/-Y%] | [+/-Z%] |
| S&P 500 | [...] | [...] | [...] |
| Nasdaq | [...] | [...] | [...] |
| Nikkei 225 | [...] | [...] | [...] |
| Shanghai Composite | [...] | [...] | [...] |

### 3.2. Tỷ giá quốc tế

| Cặp | Giá trị | Biến động tuần | Biến động tháng |
|---|---|---|---|
| Chỉ số DXY | [X] | [+/-Y%] | [+/-Z%] |
| EUR/USD | [...] | [...] | [...] |
| GBP/USD | [...] | [...] | [...] |
| USD/JPY | [...] | [...] | [...] |
| USD/CNY | [...] | [...] | [...] |

### 3.3. Lãi suất quốc tế

| Lãi suất | Giá trị | Thay đổi tuần |
|---|---|---|
| Fed Funds Rate | [X%] | [...] |
| ECB Deposit Rate | [...] | [...] |
| PBOC LPR | [...] | [...] |
| TPCP Mỹ 10 năm | [...] | [...] |

**Tổng kết:** [3-4 dòng prose — tâm lý chung quốc tế tuần qua, áp lực USD, kỳ vọng lãi suất, sự kiện vĩ mô lớn nhất.]
```

### 3.5. Phần 4 — Thị trường Việt Nam

4 sub-section.

```
## 4. Thị trường Việt Nam

### 4.1. VNINDEX & thanh khoản

**Đóng cửa phiên cuối tuần:** [X] điểm (biến động phiên: [+/-Y%])

| Khung | Biến động |
|---|---|
| Tuần | [+/-X%] |
| Tháng | [+/-Y%] |
| Quý | [+/-Z%] |
| Năm | [+/-W%] |

**Thanh khoản:**
- GTGD phiên cuối tuần: [X] tỷ đồng
- GTGD trung bình tuần: [Y] tỷ đồng
- So với trung bình tháng: [+/-Z%]

### 4.2. Dòng tiền nội

- **Điểm dòng tiền tuần thị trường:** [X] ([dương mạnh / dương / trung tính / âm / âm mạnh])
- **Điểm dòng tiền 5 phiên:** [chuỗi 5 số mô tả pattern — đồng đều dương / dao động / đồng đều âm / phục hồi cuối tuần / đảo chiều]
- **Breadth phiên cuối tuần:** [X] mã tăng / [Y] mã giảm / [Z] mã đứng giá

### 4.3. Khối ngoại / Tự doanh tuần

**Tổng net tuần:**
- Khối ngoại: [+/-X] tỷ đồng ([mua ròng / bán ròng / trung tính])
- Tự doanh: [+/-Y] tỷ đồng

**Top 5 NN mua ròng tuần:**

| Ticker | Ngành | Net mua tuần (tỷ) |
|---|---|---|
| [...] | [...] | [...] |

**Top 5 NN bán ròng tuần:**

| Ticker | Ngành | Net bán tuần (tỷ) |
|---|---|---|
| [...] | [...] | [...] |

### 4.4. Tỷ giá & lãi suất Việt Nam

| Chỉ số | Giá trị | Biến động tuần | Biến động tháng |
|---|---|---|---|
| Tỷ giá USD/VND VCB bán | [X] | [...] | [...] |
| Tỷ giá phi chính thức | [...] | [...] | [...] |
| LS liên ngân hàng 1M | [X%] | [...] | [...] |
| LS liên ngân hàng 3M | [...] | [...] | [...] |
| LS reverse repo OMO | [...] | [...] | [...] |
| OMO lưu hành | [X] tỷ | [...] | [...] |
```

### 3.6. Phần 5 — Vĩ mô & hàng hoá

**Phần quan trọng nhất setup cho phần 6 và checkpoint 1.** 4 sub-section.

```
## 5. Vĩ mô & hàng hoá — yếu tố dẫn dắt ngành

### 5.1. Lãi suất

[Bảng lãi suất điều hành VN + liên ngân hàng các kỳ hạn + huy động + cho vay qua đêm]

**Diễn giải:** [2-3 dòng — tác động đến ngân hàng, BĐS]

### 5.2. Tỷ giá

[Bảng tỷ giá VND + EUR/USD + USD/CNY]

**Diễn giải:** [2-3 dòng — áp lực USD, tác động xuất khẩu, ngân hàng]

### 5.3. Hàng hoá

| Mặt hàng | Nhóm | Giá hiện tại | Biến động tuần | Biến động tháng |
|---|---|---|---|---|
| Dầu Brent | Năng lượng | [X] USD/thùng | [+/-Y%] | [+/-Z%] |
| Quặng sắt | Kim loại | [...] | [...] | [...] |
| Thép HRC | Kim loại | [...] | [...] | [...] |
| Than cốc | Năng lượng | [...] | [...] | [...] |
| Urea Trung Đông | Hoá chất | [...] | [...] | [...] |
| Heo hơi VN | Nông sản | [...] | [...] | [...] |
| Cao su Nhật | Hoá chất | [...] | [...] | [...] |
| Cà phê thế giới | Nông sản | [...] | [...] | [...] |
| Vàng thế giới | Kim loại quý | [...] | [...] | [...] |
| ... | ... | ... | ... | ... |

[Bảng full theo nhóm: kim loại / năng lượng / nông sản / hoá chất. Không có cột "ngành VN nhạy" trong bảng này — số liệu thuần.]

### 5.4. Tác động lên ngành VN tuần này

[Bảng động — chỉ render khi có biến động đáng kể tuần qua. Nếu tuần không có biến động vĩ mô / commodity nào đáng kể: bỏ qua sub-section này, KHÔNG render bảng rỗng, KHÔNG ghi note "không có gì đặc biệt". Heading 5.4 vẫn giữ nhưng có thể bỏ luôn nếu không có nội dung.]

| Chỉ số biến động | Ngành VN bị tác động | Hướng tác động + cơ chế |
|---|---|---|
| Dầu Brent +X% lên Y USD/thùng | Dầu khí | Tích cực — biên thượng nguồn cải thiện |
| Quặng sắt -X% + HRC -Y% trong khi than cốc +Z% | Thép | Tiêu cực — đầu vào tăng, đầu ra giảm, biên gộp thu hẹp |
| Urea Trung Đông +X% lên Y USD/tấn | Phân bón | Tích cực — giá bán cải thiện |
| Lãi suất huy động giảm X điểm % | Bất động sản | Tích cực — chi phí vốn giảm, kích cầu |
| USD/VND giảm X% xuống Y | Xuất khẩu (dệt may, thuỷ sản) | Tiêu cực — biên xuất khẩu thu hẹp |
| ... | ... | ... |

**Quy tắc render bảng 5.4:**
- Chỉ liệt kê chỉ số có biến động ĐÁNG KỂ tuần qua (judgment định tính: vượt biên độ bình thường, có pattern đảo chiều, vượt mốc tâm lý)
- Mỗi chỉ số đáng kể → 1 dòng, ngành bị tác động + cơ chế cụ thể
- 1 chỉ số có thể tác động > 1 ngành → tách thành nhiều dòng
- Tuần điều hoà không có biến động vĩ mô đáng kể → bỏ qua section này hoàn toàn
```

### 3.7. Phần 6 — Biến động ngành

```
## 6. Biến động ngành

Bảng 24 ngành sort theo xếp hạng dòng tiền giảm dần:

| # | Ngành | Biến động tuần | Biến động tháng | P/E | P/B | Điểm dòng tiền tuần | Xếp hạng dòng tiền |
|---|---|---|---|---|---|---|---|
| 1 | [Ngành A] | [+/-X%] | [+/-Y%] | [Z] | [W] | [V] | 1 |
| 2 | [Ngành B] | [...] | [...] | [...] | [...] | [...] | 2 |
| ... | ... | ... | ... | ... | ... | ... | ... |
| 24 | [Ngành X] | [...] | [...] | [...] | [...] | [...] | 24 |

**Diễn giải:**

[4-6 dòng prose:
- Top 5 ngành rank cao + cross-check với mapping vĩ mô phần 5.4 → ngành dẫn dắt thật candidate
- Phát hiện phân kỳ giá vs dòng tiền (trụ kéo / tích luỹ chuẩn bị bứt)
- Top 3 ngành rank thấp + biến động giá âm + vĩ mô áp lực → ngành cần thận trọng
]
```

### 3.8. Phần 7 — Top dẫn dắt

```
## 7. Top dẫn dắt — 2 góc nhìn

### 7.1. Top theo biến động giá tuần

**Top 5 tăng:**

| Ticker | Ngành | % tuần | GTGD trung bình tuần (tỷ) |
|---|---|---|---|
| [...] | [...] | [+X%] | [...] |

**Top 5 giảm:**

| Ticker | Ngành | % tuần | GTGD trung bình tuần (tỷ) |
|---|---|---|---|
| [...] | [...] | [-X%] | [...] |

### 7.2. Top theo dòng tiền

| Ticker | Ngành | Điểm dòng tiền tuần | Xếp hạng thị trường | % tuần | GTGD (tỷ) |
|---|---|---|---|---|---|
| [...] | [...] | [...] | top X% | [...] | [...] |

### 7.3. Cross-check

- **Dẫn dắt thật (cả 2 list):** [danh sách mã] — có cả lực giá và dòng tiền
- **Đang gom kín (chỉ dòng tiền cao, giá chưa chạy):** [danh sách mã] — watch sát tuần tới
- **Chạy nhanh không bền (chỉ biến động giá, dòng tiền không cao):** [danh sách mã] — cảnh giác bền vững
```

### 3.9. Phần 8 — Tin tức & catalyst

```
## 8. Tin tức & catalyst

### 8.1. Tin trong nước nổi bật ([N] tin)

- **[Tiêu đề tin 1]** ([nguồn / ngày]). [2-3 dòng nội dung chính + ngành VN ảnh hưởng]. [Link finext.vn](https://finext.vn/news/<slug>)
- **[Tiêu đề tin 2]** ...
- ...

### 8.2. Tin quốc tế ảnh hưởng VN ([N] tin)

- **[Tiêu đề tin 1]** ([nguồn / ngày]). [2-3 dòng]. [URL gốc]
- ...

### 8.3. Mapping tác động lên ngành VN

| Tin / sự kiện | Ngành VN ảnh hưởng | Hướng tác động |
|---|---|---|
| [Tin 1, 1 dòng] | [Ngành A, B] | Tích cực — [1 dòng cơ chế] |
| [Tin 2] | [Ngành C] | Tiêu cực — [...] |
| [Tin 3] | [Ngành D] | Trung tính / theo dõi — [...] |
```

### 3.10. Phần 9 — Phân tích kỹ thuật VNINDEX + 3 kịch bản

```
## 9. Phân tích kỹ thuật VNINDEX + 3 kịch bản

### 9.1. Diễn biến giá tuần

**Phiên cuối tuần:** mở [X] / cao [Y] / thấp [Z] / đóng [W] ([+/-V%])

[3-4 dòng prose — nến cuối tuần (xanh / đỏ / doji / shooting star), vị thế so với MA5/20/60 (trên / dưới + chênh lệch %), biên độ tuần / tháng / quý (vị thế % trong range), thanh khoản phiên cuối so trung bình.]

### 9.2. Vùng kỹ thuật quan trọng

| Khung | Kháng cự | Hỗ trợ |
|---|---|---|
| Tuần | [...] | [...] |
| Tháng | POC tháng [X] / Fibonacci 50% [Y] | POC [Z] / Fibonacci 38.2% [W] |
| Quý | [...] | POC quý [X] / MA60 [Y] |
| Năm | Đỉnh năm [X] | Fibonacci 61.8% năm [Y] |

### 9.3. Ba kịch bản tuần tới

**Kịch bản cơ sở:**
- **Trigger duy trì:** [VNINDEX giữ trên POC quý X + điểm dòng tiền phiên thị trường dao động giữa âm nhẹ và dương nhẹ]
- **Vùng dao động dự kiến:** [X] — [Y]
- **Hành vi kỳ vọng:** [tích luỹ trong vùng / sideway / điều chỉnh nhẹ]

**Kịch bản tích cực:**
- **Trigger break-out:** [đóng cửa trên kháng cự X + volume > 1.2x trung bình + điểm dòng tiền tuần thị trường dương 3 phiên liên tiếp]
- **Mục tiêu kỹ thuật:** [Y] (đỉnh quý / Fibonacci kháng cự khung lớn)
- **Confirm bằng:** [tín hiệu phụ — dẫn dắt từ ngành A, B; NN mua ròng]

**Kịch bản tiêu cực:**
- **Trigger break-down:** [đóng cửa dưới hỗ trợ kép X (POC quý + MA60) + điểm dòng tiền tuần thị trường âm 3 phiên + thanh khoản tăng phiên giảm]
- **Vùng hỗ trợ tiếp theo:** [Y] (POC tháng / Fibonacci 61.8% quý)
- **Confirm bằng:** [NN bán ròng tăng cường, breadth ngành tiếp tục thu hẹp]

> *Các kịch bản là hệ thống điều kiện kỹ thuật, không phải dự báo chắc chắn. Diễn biến thực tế có thể lệch khỏi cả 3 nếu xuất hiện sự kiện ngoài kỳ vọng.*

### 9.4. Risk map

**Rủi ro 1 — [Tên ngắn gọn]**
- **Bản chất:** [2-3 dòng — rủi ro gì, ảnh hưởng kịch bản cơ sở thế nào]
- **Signal materialize:** [chỉ báo / sự kiện / mức giá nào để biết đang xảy ra]
- **Phản ứng đề xuất:** [định tính, không command — vd "giảm exposure ngành X" / "chuyển sang sector defensive" / "đứng ngoài"]

**Rủi ro 2 — [...]**
- ...

[3-7 rủi ro, flex theo bối cảnh thực tế tuần.]
```

### 3.11. Phần 10 — Watchlist (Mã đáng chú ý)

```
## 10. Watchlist — Mã đáng chú ý

### 10.1. Bối cảnh sector bias

**Regime chốt:** [risk-on full / risk-on selective / defensive only / đứng ngoài]

[Nếu có override từ checkpoint 1:]
> Regime chốt sau review: [X]. Override note: [lý do user nêu].

[1-2 dòng ngụ ý chiến lược — universe rộng/hẹp, mức độ thận trọng cần thiết.]

**Ngành quan tâm:** [Ngành A] (1 câu lý do); [Ngành B] (1 câu); ...
**Ngành cần thận trọng:** [Ngành X] (1 câu lý do); ...

**Risk-reward định tính tuần tới:** [1 dòng — "Rủi ro xuống lớn hơn tiềm năng tăng" / "Cân bằng" / "Tiềm năng tăng có ưu thế"]

### 10.2. Mã đáng chú ý ([N] mã, 5-8 mã, gộp cả hướng tích cực và tiêu cực)

- **[Ticker A]** (Ngành) — [Luận điểm đầu tư 1 câu, hướng tích cực: ví dụ "Catalyst Q1 dự kiến tích cực, dòng tiền cải thiện rõ tuần qua"]  
  [Dòng 2: catalyst + flow + technical, ngắn gọn observation]

- **[Ticker B]** (Ngành) — [Luận điểm 1 câu, hướng tiêu cực: ví dụ "Áp lực từ chi phí đầu vào tăng, kết quả Q1 có thể yếu hơn kỳ vọng"]  
  [Dòng 2: catalyst tiêu cực + dòng tiền tuần đang rút ra + technical đảo chiều]

- ...

**Lưu ý format:** không dùng từ command (mua/bán/giảm tỷ trọng/stop loss). Hướng cơ hội (tích cực) hay rủi ro (tiêu cực) thể hiện rõ trong wording luận điểm. Không kèm level giá vào/ra/stop.
```

### 3.12. Phần 11 — Lịch sự kiện tuần tới

```
## 11. Lịch sự kiện tuần tới

### 11.1. Lịch macro

| Ngày | Sự kiện | Ngành VN ảnh hưởng |
|---|---|---|
| Thứ Hai DD/MM | [FOMC / CPI Mỹ / NFP / GDP / PMI / họp NHNN] | [Ngành A] |
| Thứ Ba DD/MM | [...] | [...] |
| ... | ... | ... |

### 11.2. Lịch corporate

| Ngày | Ticker | Sự kiện |
|---|---|---|
| Thứ Tư DD/MM | [Ticker] | [BCTC / ĐHCĐ / divestment / M&A / niêm yết công ty con / chốt cổ tức] |
| ... | ... | ... |

Lịch chỉ liệt kê sự kiện đáng chú ý — bỏ qua sự kiện không impact materially.
```

### 3.13. Phần 12 — Tuyên bố miễn trừ trách nhiệm

Render theo branding info user cung cấp ở pre-flight (Section 4 câu 3 của P_weekly_market_00). 3 trường hợp:

**(a) User cung cấp custom disclaimer text:**

```
## 12. Tuyên bố miễn trừ trách nhiệm

[Custom disclaimer text user cung cấp đầy đủ]

### LIÊN HỆ

**[TÊN CÔNG TY]**  
Website: [user cung cấp]  
Hotline: [user cung cấp]  
[Phòng ban biên soạn]

**Ngày phát hành:** [DD/MM/YYYY]
```

**(b) User cung cấp branding nhưng không có disclaimer text — dùng default:**

```
## 12. Tuyên bố miễn trừ trách nhiệm

Báo cáo này được [TÊN CÔNG TY] soạn thảo trên cơ sở các thông tin, dữ liệu được thu thập từ các nguồn được coi là đáng tin cậy vào thời điểm phát hành. Tuy nhiên, [TÊN CÔNG TY] không đảm bảo tuyệt đối về tính chính xác, đầy đủ của các thông tin này.

Các nhận định trong báo cáo phản ánh quan điểm độc lập tại thời điểm công bố và có thể thay đổi mà không cần thông báo trước. Nhà đầu tư cần tự đánh giá mức độ phù hợp với tình hình tài chính cá nhân, khả năng chịu rủi ro và mục tiêu đầu tư trước khi ra quyết định.

Quyết định đầu tư cuối cùng hoàn toàn thuộc về Quý khách hàng. [TÊN CÔNG TY] không chịu trách nhiệm về bất kỳ tổn thất nào phát sinh từ việc sử dụng báo cáo này.

### LIÊN HỆ

**[TÊN CÔNG TY]**  
Website: [user cung cấp]  
Hotline: [user cung cấp]

**Ngày phát hành:** [DD/MM/YYYY]
```

**(c) User chọn không cung cấp branding (pre-flight 3b) — render plain:**

```
## 12. Tuyên bố miễn trừ trách nhiệm

> ⚠️ **Lưu ý:** Báo cáo render bản plain do không có thông tin branding. Trước khi gửi khách hàng, cần bổ sung disclaimer phù hợp với pháp lý của tổ chức phát hành.

Báo cáo này phản ánh quan điểm phân tích nội bộ tại thời điểm soạn thảo và có thể thay đổi mà không cần thông báo. Nhà đầu tư tự cân nhắc dựa trên tình hình tài chính cá nhân và mục tiêu đầu tư.

**Ngày phát hành:** [DD/MM/YYYY]
```

### 3.14. Block render cho checkpoint 1 (intermediate output)

Khác với MD final, checkpoint block là output user-facing **trong session**, render ngắn gọn để user review regime + sector bias. KHÔNG phải file MD save xuống outputs.

**Format:**

- Plain markdown trong message, KHÔNG có frontmatter, KHÔNG có header branding
- Heading top: `─── REGIME + SECTOR BIAS — Call sơ bộ ───`
- Độ dài: 0.5-1 trang nội dung structured
- Kết bằng câu hỏi multi-choice 4 option (a/b/c/d)
- KHÔNG render full MD final — chỉ regime call + 4 input lý do + sector bias đề xuất

**Spec block:** theo template `P_weekly_market_00` mục 6.4 — gồm: regime call sơ bộ, lý do 4 input (dòng tiền / breadth / NN-TD / vĩ mô), sector bias đề xuất (ngành quan tâm + thận trọng), multi-choice 4 option (Confirm / Override regime / Override sector bias / Phân tích thêm).

Agent KHÔNG ghép checkpoint block vào MD final. Sau khi user confirm/override, MD final compose Stage 2 với regime + sector bias đã chốt; nếu có override, ghi inline note ở phần 10.1 (xem mục 3.11).

### 3.15. Metadata cuối file

```
---

**Metadata**

- **Tuần báo cáo:** DD/MM/YYYY đến DD/MM/YYYY
- **Ngày phát hành:** DD/MM/YYYY
- **Regime tuần:** [risk-on full / risk-on selective / defensive only / đứng ngoài]
- **Sector bias quan tâm:** [list]
- **Sector bias thận trọng:** [list]
- **File W-1 đã tham chiếu:** [tên file user gửi / "không có"]
- **Override checkpoint 1:** [có / không + lý do nếu có]
- **Source dữ liệu chính:** agent_db ([list các collection chính đã query])
- **Web search bổ sung:** [có / không + chủ đề chính]
```

Disclaimer chỉ render ở phần 12, không lặp ở metadata footer. Metadata thuần audit trail.

## 4. Compose workflow step-by-step

**Bước 1 — Pre-flight:** P pack hỏi user 3 câu (file W-1, context đặc biệt, branding info).

**Bước 2 — Compose Stage 1:** P pack chạy phần 2-9, output structured cho mỗi phần.

**Bước 3 — Render Stage 1 partial (intermediate):** O pack render phần 2-9 thành MD chunk để agent xem, không ship cho user. Mục đích: agent self-check phần 5 mapping vĩ mô có đầy đủ + phần 6 cross-check không bị thiếu.

**Bước 4 — Compose checkpoint 1 block:** P pack tổng hợp 4 input regime → call sơ bộ. O pack format thành block xuất ra cho user xem.

**Bước 5 — Wait user:** dừng turn, đợi user phản hồi.

**Bước 6 — User phản hồi:**
- Confirm → tiếp Bước 7
- Override regime → ghi inline note vào phần 10.1
- Override sector bias → cập nhật phần 10.2
- Cần phân tích thêm → P pack query bổ sung, refine, hỏi lại

**Bước 7 — Compose Stage 2:** P pack chạy phần 10, 11, 12, rồi quay lại phần 1.

**Bước 8 — Render full MD:** O pack ghép 12 phần + header + metadata thành file `weekly_market_<YYYYMMDD>.md`.

**Bước 9 — Self-audit:** chạy 8 câu self-check P pack mục 10 + checklist O pack mục 7 dưới đây.

**Bước 10 — Present:** xuất nội dung MD trong message (Claude Desktop), user copy/save thủ công.

## 5. Guide render docx [LEGACY]

> Render binary out of scope. Section giữ làm reference cho tool render bên ngoài.

Docx weekly market report ít dùng — chỉ khi user yêu cầu archive formal hoặc share qua email cho leadership.

**Layout:**
- Cover page compact (1 trang): logo (nếu có), title, tuần, regime tag, ngày phát hành
- Phần 1 Executive summary: 1 trang riêng, full-page treatment
- Phần 2-12: theo thứ tự, mỗi phần 1 heading
- Bảng giữ nguyên, format theo style table chuẩn
- Footer: số trang + "Báo cáo tuần [DD/MM]"

**Template cần có:**
- Heading 1, 2, 3 styles
- Body style (Arial / Times 11pt)
- Table style chuẩn
- Cover page template

## 6. Guide render pptx [LEGACY]

> Render binary out of scope. Section giữ làm reference cho tool render bên ngoài.

Pptx weekly market hiếm dùng — chỉ khi user explicit yêu cầu cho meeting trình bày.

8-12 slide là đủ:

| Slide | Nội dung |
|---|---|
| 1 | Cover — Tuần + Regime badge |
| 2 | Tóm tắt điều hành (5 bullet) |
| 3 | Bối cảnh quốc tế (3 bảng compact) |
| 4 | Thị trường Việt Nam (VNINDEX chart + thanh khoản + dòng tiền) |
| 5 | Vĩ mô & hàng hoá + tác động ngành tuần này (bảng 5.4 dynamic, skip slide nếu tuần không có biến động đáng kể) |
| 6 | Biến động ngành (bảng 24 ngành — top 10 + bottom 5) |
| 7 | Top dẫn dắt (top 5 + cross-check) |
| 8 | Tin tức & catalyst (mapping bảng 8.3) |
| 9 | PTKT VNINDEX + 3 kịch bản + Risk map |
| 10 | Watchlist (sector bias + mã đáng chú ý) |
| 11 | Lịch sự kiện tuần tới (macro + corporate) |
| 12 | Tuyên bố miễn trừ trách nhiệm |

## 7. Self-check checklist O pack

- [ ] 12 phần đầy đủ kể cả phần rỗng
- [ ] Heading exact theo mục 2 spec
- [ ] Header báo cáo: nếu user cung cấp branding ở pre-flight đã render branded version (logo + tên công ty + hotline + website + disclaimer ngắn); nếu không, render plain version
- [ ] Phần 2 skip đúng nếu không có file W-1, ghi note 1 dòng
- [ ] Phần 5 đặt trước phần 6 (vĩ mô là input cho bảng 24 ngành)
- [ ] Sub-section 5.4 chỉ render khi tuần có biến động vĩ mô / commodity đáng kể; tuần không có thì bỏ qua, không render bảng rỗng
- [ ] Phần 6 sort theo xếp hạng dòng tiền giảm dần (rank 1 ở trên)
- [ ] Phần 7 cross-check 3 nhóm rõ ràng
- [ ] Phần 8 mỗi tin có dẫn link finext.vn (slug raw không lộ trần)
- [ ] Phần 9 ba kịch bản dùng if-then trigger, **không có % xác suất**, có note disclaimer kỹ thuật sau 9.3
- [ ] Phần 9.4 Risk map 3-7 rủi ro, mỗi rủi ro 3 dòng (bản chất / signal / phản ứng định tính không command)
- [ ] Phần 10 wording "ngành quan tâm / ngành cần thận trọng" (không "tập trung / tránh"), nếu có override checkpoint 1 đã ghi inline note
- [ ] Phần 10 watchlist mã (5-8 mã gộp cả tích cực và tiêu cực), wording observation không command (không "mua/bán/giảm tỷ trọng/stop loss"), không có level giá vào/ra/stop
- [ ] Phần 11 lịch sự kiện 2 bảng (macro + corporate), chỉ liệt kê sự kiện đáng chú ý
- [ ] Phần 12 disclaimer render đúng theo branding info (custom / default branded / plain)
- [ ] **Không có chỉ báo trend ở bất kỳ phần nào** — không từ "xu hướng tuần/tháng/quý/năm" theo nghĩa breadth, không "đang rơi từ vùng quá mua / đang bật từ đáy"
- [ ] K hygiene: không lộ ký hiệu DB raw — dịch theo bảng `K_agent_db_00` mục 5.2 (field name, score raw, zone code, period code, money_flow_score raw)
- [ ] Số liệu định lượng đã quy đổi đơn vị (BCTC tỷ đồng, % thập phân nhân 100, NN/TD đã ở tỷ đồng giữ nguyên)
- [ ] Mỗi tin có dẫn link, mỗi claim quan trọng có nguồn
- [ ] Metadata cuối đầy đủ
- [ ] Disclaimer footer: nếu user cung cấp branding đã render đầy đủ; nếu không, không render
- [ ] File save đúng tên `weekly_market_<YYYYMMDD>.md` với YYYYMMDD = ngày kết thúc tuần
- [ ] Xuất nội dung MD trong message (Claude Desktop)

## 8. Output contract

O pack render structured content P pack sinh thành 1 file MD final theo structure rigid 12 phần. Không thêm/bớt nội dung, không tự ý insert section. Khi P pack flag phần rỗng (như phần 2 không có W-1), O pack render 1 dòng note thay phần đó, heading vẫn giữ.

User explicit yêu cầu format khác (docx / pptx) → O pack render bổ sung theo guide mục 5-6. MD luôn là source of truth.
