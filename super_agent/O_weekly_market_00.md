# O_weekly_market_00 — Render Spec Báo cáo tuần thị trường

Spec render báo cáo tuần thị trường VN cho họp nội bộ. **Rigid structure** — 12 phần thứ tự cố định, heading exact, dùng cho mỗi tuần để user scan nhanh và so sánh giữa các tuần.

Reference: `P_weekly_market_00` (workflow + methodology).

## 1. Input từ P pack

P pack sinh structured content cho 12 phần theo spec ở mục 5-7 của P_weekly_market_00. O pack chỉ render, không thêm/bớt nội dung.

Trường hợp đặc biệt:
- Phần 2 user không gửi file W-1 → render 1 dòng "Tuần đầu cycle, chưa có dữ liệu review tuần trước." Heading vẫn giữ.
- Checkpoint 1 user override regime/sector bias → render note inline trong phần 10 sub-section 10.1: "Regime chốt sau review: [X]. Override note: [lý do]."

## 2. Structure báo cáo đầu ra

**Độ dài target:** 9-11 trang MD.

**Rigid structure bắt buộc** (thứ tự + heading exact):

```
# Báo cáo thị trường tuần — <DD/MM/YYYY> đến <DD/MM/YYYY>

## 1. Executive summary
## 2. Review tuần trước
## 3. Bối cảnh quốc tế
## 4. Thị trường Việt Nam
## 5. Vĩ mô & hàng hoá — yếu tố dẫn dắt ngành
## 6. Biến động ngành
## 7. Top dẫn dắt
## 8. Tin tức & catalyst
## 9. Phân tích kỹ thuật VNINDEX + 3 kịch bản
## 10. Chiến lược tuần tới
## 11. Mã & sự kiện đáng chú ý
## 12. Calendar + Risk map
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

### 3.2. Phần 1 — Executive summary

Format bullet list, 3-5 bullet, mỗi bullet 1-2 dòng:

```
- **Regime tuần:** [risk-on full / risk-on selective / defensive only / đứng ngoài]. [1 dòng tóm tắt căn cứ chính.]
- **Catalyst lớn nhất:** [tin/sự kiện 1] [+ tin 2 nếu có].
- **Sector bias:** Quan tâm [danh sách ngành ngắn gọn, viết tên ngành đầy đủ]. Thận trọng [danh sách ngành].
- **Risk chính tuần tới:** [1 rủi ro nổi bật từ phần 12].
- **Mã đáng chú ý:** [1 mã từ watchlist + 1 dòng luận điểm, optional].
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
- **Chuỗi day_score 5 phiên:** [chuỗi 5 số mô tả pattern — đồng đều dương / dao động / đồng đều âm / phục hồi cuối tuần / đảo chiều]
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
- **Trigger duy trì:** [VNINDEX giữ trên POC quý X + day_score thị trường dao động giữa âm nhẹ và dương nhẹ]
- **Vùng dao động dự kiến:** [X] — [Y]
- **Hành vi kỳ vọng:** [tích luỹ trong vùng / sideway / điều chỉnh nhẹ]

**Kịch bản tích cực:**
- **Trigger break-out:** [đóng cửa trên kháng cự X + volume > 1.2x trung bình + week_score thị trường dương 3 phiên liên tiếp]
- **Mục tiêu kỹ thuật:** [Y] (đỉnh quý / Fibonacci kháng cự khung lớn)
- **Confirm bằng:** [tín hiệu phụ — dẫn dắt từ ngành A, B; NN mua ròng]

**Kịch bản tiêu cực:**
- **Trigger break-down:** [đóng cửa dưới hỗ trợ kép X (POC quý + MA60) + week_score thị trường âm 3 phiên + thanh khoản tăng phiên giảm]
- **Vùng hỗ trợ tiếp theo:** [Y] (POC tháng / Fibonacci 61.8% quý)
- **Confirm bằng:** [NN bán ròng tăng cường, breadth ngành tiếp tục thu hẹp]

> *Các kịch bản là hệ thống điều kiện kỹ thuật, không phải dự báo chắc chắn. Diễn biến thực tế có thể lệch khỏi cả 3 nếu xuất hiện sự kiện ngoài kỳ vọng.*
```

### 3.11. Phần 10 — Chiến lược tuần tới

```
## 10. Chiến lược tuần tới

### 10.1. Regime tuần

**Regime chốt:** [risk-on full / risk-on selective / defensive only / đứng ngoài]

[Nếu có override từ checkpoint 1:]
> Regime chốt sau review: [X]. Override note: [lý do user nêu].

[2-3 dòng ngụ ý chiến lược — universe rộng/hẹp, tỷ trọng tăng/giảm, ưu tiên phòng thủ hay tấn công, kỳ vọng turnover.]

### 10.2. Sector bias chi tiết

**Ngành quan tâm [N ngành]:**

- **[Ngành A]:** [2-3 dòng — lý do tích cực: vĩ mô + flow + catalyst. Điểm cần theo dõi tuần tới.]
- **[Ngành B]:** [...]
- ...

**Ngành cần thận trọng [M ngành]:**

- **[Ngành X]:** [1 dòng lý do thận trọng — rủi ro vĩ mô / catalyst tiêu cực / quá mua đa khung]
- ...

### 10.3. Vùng giá VNINDEX cần theo dõi

- **Kháng cự gần:** [X] (cản kỹ thuật khung tuần / Fibonacci)
- **Hỗ trợ gần:** [Y] (POC quý / MA20)
- **Breakout level:** [Z] — phá lên xác nhận kịch bản tích cực; [W] — phá xuống xác nhận kịch bản tiêu cực

### 10.4. Risk-reward định tính

[2-3 dòng — đánh giá định tính (không % không số): "Rủi ro xuống lớn hơn tiềm năng tăng trong tuần tới do [lý do]" / "Cân bằng" / "Tiềm năng tăng có ưu thế nhờ [lý do]". Nêu điều kiện cần để chuyển bias risk-reward sang chiều ngược.]
```

### 3.12. Phần 11 — Mã & sự kiện đáng chú ý

```
## 11. Mã & sự kiện đáng chú ý

### 11.1. Mã đáng chú ý ([N] mã, 5-8 mã, gộp cả hướng tích cực và tiêu cực)

- **[Ticker A]** (Ngành) — [Luận điểm đầu tư 1 câu, hướng tích cực: ví dụ "Catalyst Q1 dự kiến tích cực, dòng tiền cải thiện rõ tuần qua"]  
  [Dòng 2: catalyst + flow + technical, ngắn gọn observation]

- **[Ticker B]** (Ngành) — [Luận điểm 1 câu, hướng tiêu cực: ví dụ "Áp lực từ chi phí đầu vào tăng, kết quả Q1 có thể yếu hơn kỳ vọng"]  
  [Dòng 2: catalyst tiêu cực + dòng tiền tuần đang rút ra + technical đảo chiều]

- ...

**Lưu ý format:** không dùng từ command (mua/bán/giảm tỷ trọng/stop loss). Hướng cơ hội (tích cực) hay rủi ro (tiêu cực) thể hiện rõ trong wording luận điểm. Không kèm level giá vào/ra/stop.

### 11.2. Sự kiện đáng chú ý tuần tới ([N] sự kiện)

- **[Ticker Y]** (Ngành) — [Sự kiện: BCTC ngày DD/MM / ĐHCĐ ngày DD/MM / divestment / M&A / niêm yết công ty con]
- ...
```

### 3.13. Phần 12 — Calendar + Risk map

```
## 12. Calendar + Risk map

### 12.1. Calendar tuần tới

| Ngày | Sự kiện | Ngành VN ảnh hưởng |
|---|---|---|
| Thứ Hai DD/MM | [Sự kiện vĩ mô / BCTC / chính sách] | [Ngành A] |
| Thứ Ba DD/MM | [...] | [...] |
| ... | ... | ... |

### 12.2. Risk map

**Rủi ro 1 — [Tên ngắn gọn]**
- **Bản chất:** [2-3 dòng — rủi ro gì, ảnh hưởng kịch bản cơ sở thế nào]
- **Signal materialize:** [chỉ báo / sự kiện / mức giá nào để biết đang xảy ra]
- **Phản ứng đề xuất:** [giảm tỷ trọng X% / chuyển sector A→B / đứng ngoài / không action]

**Rủi ro 2 — [...]**
- ...

[3-7 rủi ro, flex theo bối cảnh thực tế tuần.]
```

### 3.14. Metadata cuối file

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

[Nếu user cung cấp branding ở pre-flight, render thêm block disclaimer footer dưới đây. Nếu không cung cấp, bỏ block này.]

---

**THÔNG BÁO MIỄN TRỪ TRÁCH NHIỆM**

[Nội dung disclaimer đầy đủ user cung cấp. Nếu user chỉ cung cấp branding cơ bản (tên + hotline + website) mà không cung cấp custom disclaimer, dùng template default sau:]

> *Tài liệu này do [TÊN CÔNG TY] biên soạn nhằm mục đích cung cấp thông tin tham khảo cho nhà đầu tư. Thông tin được tổng hợp từ các nguồn được cho là đáng tin cậy tại thời điểm công bố. [TÊN CÔNG TY] không cam kết hoặc bảo đảm tính chính xác, đầy đủ hay kịp thời của thông tin. Nội dung có thể thay đổi mà không cần thông báo. Tài liệu không phải khuyến nghị đầu tư và không phải lời mời chào mua bán hay nắm giữ bất kỳ chứng khoán nào. Khách hàng cần tự cân nhắc mục tiêu đầu tư và mức chịu đựng rủi ro của mình. [TÊN CÔNG TY] cùng các nhân sự liên quan không chịu trách nhiệm đối với bất kỳ tổn thất hay thiệt hại nào phát sinh từ việc sử dụng tài liệu.*

**[TÊN CÔNG TY]**  
Hotline: [user cung cấp] | Website: [user cung cấp]  
Phòng ban biên soạn: [user cung cấp]
```

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

**Bước 10 — Save & present:** save vào `/mnt/user-data/outputs/`, gọi `present_files`.

## 5. Guide render docx

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

## 6. Guide render pptx

Pptx weekly market hiếm dùng — chỉ khi user explicit yêu cầu cho meeting trình bày.

8-12 slide là đủ:

| Slide | Nội dung |
|---|---|
| 1 | Cover — Tuần + Regime badge |
| 2 | Executive summary (5 bullet) |
| 3 | Bối cảnh quốc tế (3 bảng compact) |
| 4 | Thị trường Việt Nam (VNINDEX chart + thanh khoản + dòng tiền) |
| 5 | Vĩ mô & hàng hoá + tác động ngành tuần này (bảng 5.4 dynamic, skip slide nếu tuần không có biến động đáng kể) |
| 6 | Biến động ngành (bảng 24 ngành — top 10 + bottom 5) |
| 7 | Top dẫn dắt (top 5 + cross-check) |
| 8 | Tin tức & catalyst (mapping bảng 8.3) |
| 9 | PTKT VNINDEX + 3 kịch bản |
| 10 | Chiến lược tuần tới (regime + sector bias) |
| 11 | Mã & sự kiện đáng chú ý |
| 12 | Calendar + Risk map |

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
- [ ] Phần 9 ba kịch bản dùng if-then trigger, **không có % xác suất**, có note disclaimer kỹ thuật cuối phần
- [ ] Phần 10 wording "ngành quan tâm / ngành cần thận trọng" (không "tập trung / tránh"), nếu có override checkpoint 1 đã ghi inline note
- [ ] Phần 11 watchlist 2 nhóm (mã đáng chú ý gộp + sự kiện), wording observation không command (không "mua/bán/giảm tỷ trọng/stop loss"), không có level giá vào/ra/stop
- [ ] Phần 12 risk map 3-7 rủi ro, mỗi rủi ro 3 dòng (bản chất / signal / phản ứng)
- [ ] **Không có chỉ báo trend ở bất kỳ phần nào** — không từ "xu hướng tuần/tháng/quý/năm" theo nghĩa breadth, không "đang rơi từ vùng quá mua / đang bật từ đáy"
- [ ] K hygiene: không lộ ký hiệu DB raw (week_score raw, day_score raw, vsi raw, technical_zone AAA/AA raw, rank_pct raw, period 2025_4...)
- [ ] Số liệu định lượng đã quy đổi đơn vị (BCTC tỷ đồng, % thập phân nhân 100, NN/TD đã ở tỷ đồng giữ nguyên)
- [ ] Mỗi tin có dẫn link, mỗi claim quan trọng có nguồn
- [ ] Metadata cuối đầy đủ
- [ ] Disclaimer footer: nếu user cung cấp branding đã render đầy đủ; nếu không, không render
- [ ] File save đúng tên `weekly_market_<YYYYMMDD>.md` với YYYYMMDD = ngày kết thúc tuần
- [ ] Present file qua present_files tool

## 8. Output contract

O pack render structured content P pack sinh thành 1 file MD final theo structure rigid 12 phần. Không thêm/bớt nội dung, không tự ý insert section. Khi P pack flag phần rỗng (như phần 2 không có W-1), O pack render 1 dòng note thay phần đó, heading vẫn giữ.

User explicit yêu cầu format khác (docx / pptx) → O pack render bổ sung theo guide mục 5-6. MD luôn là source of truth.
