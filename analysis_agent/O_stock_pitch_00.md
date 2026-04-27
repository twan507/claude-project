# O_stock_pitch_00 — Render Spec Báo cáo Khuyến nghị

Spec render báo cáo khuyến nghị mua mã đơn lẻ gửi khách hàng (stock pitch). **Rigid heading structure**, số mục flex **13-16 tuỳ số luận điểm 4-7** (4 LĐ=13 mục, 5=14, 6=15 default, 7=16). Heading exact, thứ tự cố định. **Output cuối: MD final** (đã apply structure + K hygiene + citation + chart annotation YAML).

Reference: `P_stock_pitch_00` (workflow + methodology).

> **Render binary out of scope:** Pack này dừng ở MD final. Render pptx/docx/xlsx là concern downstream, không thuộc scope. MD final đã đủ structured (heading + chart YAML + citation + locale) để consume bằng tool render bên ngoài. Mục pptx/docx ở dưới (nếu có) là legacy spec, sẽ được dọn ở audit pass tiếp theo.

## 1. Input từ P pack

P pack sinh structured content cho 15 mục theo spec ở mục 5-11 của P_stock_pitch_00. O pack chỉ render, không thêm/bớt nội dung.

Các trường hợp đặc biệt:
- User chọn 3(b) ở pre-flight (không cung cấp branding) → render bản plain, dùng disclaimer template default
- User chọn 2(a) ở pre-flight (có memo tier 5C) → P pack đã absorb nội dung memo vào structured content, O pack render bình thường
- Workflow abort ở checkpoint 1 hoặc 2 → KHÔNG render file. Agent thông báo cho user và dừng.

## 2. Structure báo cáo MD đầu ra

**Độ dài target:** 12-18 trang MD (dài hơn báo cáo tuần do depth phân tích từng luận điểm).

**Rigid heading structure, số mục flex 13-16** theo số luận điểm (4 LĐ=13 mục, 5=14, 6=15 default, 7=16). Thứ tự + heading exact theo template dưới đây (default 6 luận điểm; 4 LĐ skip section 8-9, 5 LĐ skip section 9-10, 7 LĐ shift cấu trúc mua → section 12, chốt lãi → section 13...):

```
[Header branding nếu có]

# BÁO CÁO PHÂN TÍCH [TICKER]
## [Tên đầy đủ công ty]

[Block khuyến nghị MUA + giá hiện tại + mục tiêu trung hạn + tiềm năng]

[Tiêu đề thesis chính 1 câu]

## 1. Tóm tắt khuyến nghị
## 2. Hồ sơ doanh nghiệp
## 3. Dữ liệu giao dịch phiên hiện tại
## 4. Luận điểm 01 — [Tiêu đề]
## 5. Luận điểm 02 — [Tiêu đề]
## 6. Luận điểm 03 — [Tiêu đề]
## 7. Luận điểm 04 — [Tiêu đề]
## 8. Luận điểm 05 — [Tiêu đề] [optional]
## 9. Luận điểm 06 — [Tiêu đề] [optional]
## 10. Luận điểm 07 — [Tiêu đề] [optional]
## 11. Chiến lược thực thi — Cấu trúc mua N lớp
## 12. Mục tiêu chốt lãi
## 13. Bear Case — Phản biện và lo ngại còn yếu
## 14. Tóm tắt hành động + 3 kịch bản
## 15. Tuyên bố miễn trừ trách nhiệm
```

**Lưu ý mapping với P pack:**

| P pack mục | O pack section trong MD |
|---|---|
| Mục 1 Cover | Header (trên cùng MD) |
| Mục 2 Tóm tắt khuyến nghị | Section 1 |
| Mục 3 Hồ sơ doanh nghiệp | Section 2 |
| Mục 4 Dữ liệu giao dịch + technical | Section 3 |
| Mục 5-10 Luận điểm 01-07 (4-7 luận điểm) | Section 4-10 (số section flex theo số luận điểm) |
| Mục 11 Cấu trúc mua | Section 11 |
| Mục 12 Mục tiêu chốt lãi | Section 12 |
| Mục 13 Bear case | Section 13 |
| Mục 14 Tóm tắt hành động + 3 kịch bản | Section 14 |
| Mục 15 Disclaimer | Section 15 |

**Rigid heading nhưng số section luận điểm flex 4-7:** nếu báo cáo có 4 luận điểm, MD có section 4-7 (4 luận điểm); nếu 7 luận điểm, MD có section 4-10. Tổng số mục đánh số: 13 (4 LĐ) → 16 (7 LĐ).

Để đơn giản, file này dùng convention 15 mục với assumption 6 luận điểm (default). Nếu pack ra 4 hoặc 7 luận điểm, adjust số.

## 3. Compose từng phần chi tiết

### 3.1. Header branding (trên cùng MD)

**Nếu user cung cấp branding info:**

```
[Logo nếu có — markdown image]

**[TÊN CÔNG TY]**  
Hotline: [user cung cấp] | Website: [user cung cấp]  
[Phòng ban biên soạn]

---

# BÁO CÁO PHÂN TÍCH [TICKER]
## [Tên đầy đủ công ty]
```

**Nếu user không cung cấp branding:**

```
# BÁO CÁO PHÂN TÍCH [TICKER]
## [Tên đầy đủ công ty]
```

### 3.2. Block khuyến nghị (ngay sau title)

Format cố định:

```
> **Khuyến nghị:** MUA  
> **Giá hiện tại:** [X] đồng  
> **Mục tiêu trung hạn:** [Y] đồng  
> **Tiềm năng tăng giá:** +[Z]%

**[Tiêu đề thesis chính — 1 câu captured insight chính]**

**Ngày phát hành:** [DD tháng MM năm YYYY]
```

Ví dụ tiêu đề thesis:
- "Cơ hội tích lũy hạ tầng Việt Nam"
- "Định giá hấp dẫn trước chu kỳ phục hồi xuất khẩu"
- "Tái cấu trúc tài sản tạo dư địa định giá lại"

### 3.3. Section 1 — Tóm tắt khuyến nghị

Format:

```
## 1. Tóm tắt khuyến nghị

Luận điểm MUA [TICKER] quanh vùng giá [X] đồng

| Chỉ số | Giá trị | Biến động |
|---|---|---|
| Giá mục tiêu ngắn hạn | [Y] đồng | +[A]% |
| Giá mục tiêu trung hạn | [Z] đồng | +[B]% |
| Điểm cắt lỗ | [W] đồng | -[C]% |
| Tỷ lệ R/R | 1:[N] | [Hấp dẫn / Tạm chấp nhận] |

### LUẬN ĐIỂM CỐT LÕI

**01 [Tiêu đề L1]**  
[2-3 dòng tóm tắt + headline data point chính]

**02 [Tiêu đề L2]**  
[...]

**03 [Tiêu đề L3]**  
[...]

**04 [Tiêu đề L4 — nếu báo cáo > 4 luận điểm, gộp 4 mạnh nhất vào tóm tắt]**  
[...]
```

**Lưu ý:** tóm tắt chỉ in 4 luận điểm mạnh nhất kể cả khi báo cáo đầy đủ có 5-7 luận điểm. Mục đích: KH scan nhanh thấy được điểm chính, không quá tải.

### 3.4. Section 2 — Hồ sơ doanh nghiệp

```
## 2. Hồ sơ doanh nghiệp

### 2.1. Thông tin cơ bản

| Thông tin | Giá trị |
|---|---|
| Mã cổ phiếu | [TICKER] ([sàn]) |
| Ngày niêm yết | [DD/MM/YYYY] |
| Vốn điều lệ | [X] tỷ đồng |
| Số cổ phiếu lưu hành | [X] triệu CP |
| Vốn hoá thị trường | [X] tỷ đồng |
| Tỷ lệ tự do chuyển nhượng | [X]% |
| Cổ đông lớn nhất | [Tên] ([X]%) |
| Tỷ lệ sở hữu nước ngoài | [X]% / [trần]% |
| Ngành | [tên ngành theo phân loại 24 ngành VBSE] |

### 2.2. Cơ cấu kinh doanh & tài sản cốt lõi

**[Mảng/công ty con 1]** ([tỷ lệ sở hữu nếu có])  
[2-3 dòng — đặc điểm + tình trạng hiện tại + dòng tiền]

**[Mảng/công ty con 2]**  
[...]

[3-6 mảng tuỳ độ phức tạp doanh nghiệp]
```

### 3.5. Section 3 — Dữ liệu giao dịch phiên hiện tại

```
## 3. Dữ liệu giao dịch phiên hiện tại

**Phiên giao dịch:** [DD/MM/YYYY]

### 3.1. Dữ liệu phiên

| Chỉ số | Giá trị |
|---|---|
| Đóng cửa | [X] đồng ([+/-Y%]) |
| Cao nhất | [X] đồng |
| Thấp nhất | [X] đồng |
| Giá trị giao dịch | [X] tỷ đồng |
| Cường độ thanh khoản | [X]x trung bình 5 phiên |

### 3.2. Dòng tiền tổ chức phiên

| Loại | Net phiên (tỷ đồng) | Net tuần (tỷ) | Net tháng (tỷ) |
|---|---|---|---|
| Khối ngoại | [+/-X] | [+/-Y] | [+/-Z] |
| Tự doanh | [+/-X] | [+/-Y] | [+/-Z] |

### 3.3. Các mức kỹ thuật quan trọng

| Mức | Giá (đồng) | +/- so giá hiện tại |
|---|---|---|
| Kháng cự 1 tuần (R1) | [X] | +[Y]% |
| Fibonacci 38.2% tuần | [X] | +[Y]% |
| Biên trên vùng giao dịch tuần | [X] | +[Y]% |
| Vùng tập trung giao dịch (POC tuần) | [X] | +/-[Y]% |
| **Giá hiện tại** | **[X]** | **—** |
| Biên dưới vùng giao dịch tuần | [X] | -[Y]% |
| Hỗ trợ 1 tuần (S1) | [X] | -[Y]% |
| Trung bình 60 phiên (MA60) | [X] | -[Y]% |
```

### 3.6. Section 4-10 — Luận điểm 01 đến 07 (flex 4-7)

**Format chung mỗi luận điểm:**

```
## [N]. Luận điểm [NN] — [Tiêu đề ngắn]

**[Sub-title 1 dòng — phát biểu thesis của luận điểm]**

> **[Headline data point — số/fact lớn nhất]**  
> [1 dòng diễn giải ý nghĩa]

### Ý chính

**[Sub-point 1]**  
[2-3 dòng — claim + data + nguồn]

**[Sub-point 2]**  
[2-3 dòng]

**[Sub-point 3]**  
[2-3 dòng]

[**[Sub-point 4 nếu có]**]
[...]

### [Bảng tham chiếu nếu có — peer comparison, lịch sử]

| Cột 1 | Cột 2 | Cột 3 |
|---|---|---|
| ... | ... | ... |
```

**Variant Perception lồng vào luận điểm phù hợp** (thường L1 hoặc L4):

```
> **Variant Perception**  
> 
> **Consensus sell-side / báo phổ thông:** [1-2 dòng]
> 
> **Consensus retail:** [1-2 dòng nếu khác consensus sell-side]
> 
> **Thesis khác consensus:** [2-3 dòng giải thích insight chính]
```

### 3.7. Section 11 — Chiến lược thực thi — Cấu trúc mua N lớp

```
## 11. Chiến lược thực thi — Cấu trúc mua [N] lớp

### LỚP 01 — MUA THĂM DÒ

| Thông số | Giá trị |
|---|---|
| Vùng giá mua | [X] - [Y] đồng |
| Tỷ trọng vị thế | [N]% |

[2-3 dòng — cơ sở kỹ thuật + trigger]

### LỚP 02 — MUA KHI XÁC NHẬN

| Thông số | Giá trị |
|---|---|
| Vùng giá mua | [X] - [Y] đồng |
| Tỷ trọng vị thế | [N]% |

[2-3 dòng]

### LỚP 03 — MUA GIA TĂNG

| Thông số | Giá trị |
|---|---|
| Vùng giá mua | [X] - [Y] đồng |
| Tỷ trọng vị thế | [N]% |

[2-3 dòng]

> *Tỷ trọng % phân bổ là gợi ý tham khảo, không bắt buộc. Khách hàng tự cân nhắc theo size portfolio, mức chịu đựng rủi ro và position sizing cá nhân.*

### QUẢN TRỊ RỦI RO

| Mức cắt lỗ | Giá | Rủi ro từ giá vào |
|---|---|---|
| Cắt lỗ cứng | [X] đồng | -[Y]% |
| Cắt lỗ dự phòng | [X] đồng | -[Y]% |

**Khung thời gian:** [N] tuần ngắn hạn / vị thế core [N]-[M] tháng cho catalyst lớn
```

### 3.8. Section 12 — Mục tiêu chốt lãi

```
## 12. Mục tiêu chốt lãi

### [N] mốc kỹ thuật

| Mốc | Giá chốt (đồng) | Lợi nhuận | Tỷ lệ chốt | Cơ sở kỹ thuật |
|---|---|---|---|---|
| M1 | [X] | +[A]% | [N1]% vị thế | [Vùng Fibonacci 38.2% khung tuần] |
| M2 | [X] | +[A]% | [N2]% vị thế | [Kháng cự 1 tuần] |
| M3 | [X] | +[A]% | [N3]% vị thế | [Biên trên vùng giao dịch tháng] |
| M4 | [X] | +[A]% | Phần còn lại | [Kháng cự tháng - mục tiêu trung hạn] |

### Profile lợi nhuận / rủi ro

| Chỉ số | Giá trị |
|---|---|
| Lợi nhuận bình quân các mốc đầu | +[X]% |
| Rủi ro tối đa đến điểm cắt lỗ | -[Y]% |
| Tỷ lệ rủi ro / lợi nhuận (R/R) | 1:[N] |
| Mục tiêu trung hạn cao nhất | +[Z]% |

[1-2 dòng đánh giá: Hấp dẫn / Tạm chấp nhận / R/R không lý tưởng nhưng [lý do exception nếu có]]
```

### 3.9. Section 13 — Bear Case

```
## 13. Bear Case — Phản biện và lo ngại còn yếu

### Đánh giá 5-7 lo ngại bear case

**Lo ngại 01:** [Phát biểu 1-2 dòng]

*Phản biện:* [2-3 dòng — số liệu cụ thể giải thích]

---

**Lo ngại 02:** [...]

*Phản biện:* [...]

---

[5-7 lo ngại + phản biện]

---

### Lo ngại CÒN YẾU sau phản biện

> **Đây là phần honest steelman bắt buộc** — báo cáo không "all win". 1-2 lo ngại sau phản biện chỉ "tạm chấp nhận" chứ không "hoàn toàn không phải vấn đề".

**[Tên lo ngại còn yếu 1]:**
- **Điểm yếu còn lại:** [1-2 dòng — phản biện chỉ giải quyết được gì, còn miss gì]
- **Tác động nếu materialize:** [1-2 dòng — ảnh hưởng đến thesis tổng thể như thế nào, target nào bị threaten]

[**[Tên lo ngại còn yếu 2 nếu có]:**]
[...]
```

### 3.10. Section 14 — Tóm tắt hành động + 3 kịch bản

```
## 14. Tóm tắt hành động + 3 kịch bản

### Hành động đề xuất

MUA [TICKER] vùng [X] - [Y] đồng. Mua thăm dò [N1]% vị thế phiên [DD-DD/MM] hoặc khi [trigger lớp 1], chờ test lại [Z]-[W] để mua tiếp [N2]%, giữ [N3]% cho bứt phá vượt [V] đồng.

> *Tỷ trọng % là gợi ý tham khảo, không bắt buộc. Khách hàng tự cân nhắc theo size portfolio và position sizing cá nhân.*

### Kịch bản cơ sở

| Thông số | Giá trị |
|---|---|
| Trigger duy trì | [điều kiện cụ thể] |
| Khung thời gian | [N tuần / tháng] |
| Giá mục tiêu | [vùng X-Y đồng] |
| Tỷ suất kỳ vọng | +[A-B]% |

[2-3 dòng — diễn biến kỳ vọng + action gợi ý]

### Kịch bản tích cực

| Thông số | Giá trị |
|---|---|
| Trigger break-out | [điều kiện cụ thể — ví dụ: tin chính thức + breakout vượt X với volume > 1.2x trung bình + NN mua ròng tăng cường] |
| Khung thời gian | [N tháng] |
| Giá mục tiêu | [vùng X-Y đồng — target trung hạn cao nhất] |
| Tỷ suất kỳ vọng | +[A-B]% |

[2-3 dòng]

### Kịch bản thận trọng

| Thông số | Giá trị |
|---|---|
| Trigger break-down | [điều kiện cụ thể — ví dụ: gãy hỗ trợ X + dòng tiền tổ chức đảo chiều bán ròng + lo ngại còn yếu materialize] |
| Khung thời gian | [N tuần] |
| Giá mục tiêu | [điểm cắt lỗ X đồng] |
| Tỷ suất kỳ vọng | -[Y]% |

[2-3 dòng — thực thi cắt lỗ kỷ luật, không ảnh hưởng vốn dài hạn]

> *Các kịch bản là hệ thống điều kiện kỹ thuật, không phải dự báo chắc chắn. Diễn biến thực tế có thể lệch khỏi cả 3 nếu xuất hiện sự kiện ngoài kỳ vọng.*
```

### 3.11. Section 15 — Tuyên bố miễn trừ trách nhiệm

**Nếu user cung cấp custom disclaimer ở pre-flight:**

```
## 15. Tuyên bố miễn trừ trách nhiệm

[Nội dung disclaimer user cung cấp đầy đủ]

### LIÊN HỆ

**[TÊN CÔNG TY]**  
Website: [user cung cấp]  
Hotline: [user cung cấp]  
[Phòng ban biên soạn]

**Ngày phát hành:** [DD/MM/YYYY]
```

**Nếu user không cung cấp custom disclaimer (chọn 3a ở pre-flight nhưng không nêu nội dung), dùng template default:**

```
## 15. Tuyên bố miễn trừ trách nhiệm

Báo cáo này được [TÊN CÔNG TY] soạn thảo trên cơ sở các thông tin, dữ liệu được thu thập từ các nguồn được coi là đáng tin cậy vào thời điểm phát hành. Tuy nhiên, [TÊN CÔNG TY] không đảm bảo tuyệt đối về tính chính xác, đầy đủ của các thông tin này.

Các khuyến nghị trong báo cáo phản ánh quan điểm độc lập của [TÊN CÔNG TY] tại thời điểm công bố và có thể thay đổi mà không cần thông báo trước. Nhà đầu tư cần tự đánh giá mức độ phù hợp với tình hình tài chính cá nhân, khả năng chịu rủi ro và mục tiêu đầu tư trước khi ra quyết định.

Quyết định đầu tư cuối cùng hoàn toàn thuộc về Quý khách hàng. [TÊN CÔNG TY] không chịu trách nhiệm về bất kỳ tổn thất nào phát sinh từ việc sử dụng báo cáo này.

### LIÊN HỆ

**[TÊN CÔNG TY]**  
Website: [user cung cấp]  
Hotline: [user cung cấp]

**Ngày phát hành:** [DD/MM/YYYY]
```

**Nếu user chọn 3b ở pre-flight (không cung cấp branding):**

```
## 15. Tuyên bố miễn trừ trách nhiệm

> ⚠️ **Cảnh báo:** Báo cáo này được render bản plain do không có thông tin branding/disclaimer. Trước khi gửi khách hàng, cần bổ sung disclaimer phù hợp với pháp lý của tổ chức phát hành.

Nội dung báo cáo phản ánh quan điểm độc lập của analyst tại thời điểm soạn thảo và có thể thay đổi mà không cần thông báo. Quyết định đầu tư cuối cùng hoàn toàn thuộc về Quý khách hàng.

**Ngày phát hành:** [DD/MM/YYYY]
```

### 3.12. Block render cho checkpoint 1 / 2 (intermediate output)

Khác với MD final, checkpoint block là output user-facing **trong session**, render ngắn gọn để user review. KHÔNG phải file MD save xuống outputs.

**Format chung (cả CP1 và CP2):**

- Plain markdown trong message, KHÔNG có frontmatter, KHÔNG có header branding
- Heading top: `─── THESIS REVIEW — Trước khi build execution ───` (CP1) hoặc `─── BEAR CASE REVIEW — Gate cuối trước recommend ───` (CP2)
- Độ dài: 0.5-1 trang nội dung structured
- Kết bằng câu hỏi multi-choice 4-5 option
- KHÔNG render full MD final — chỉ summary + variant perception (CP1) hoặc summary bear + lo ngại còn yếu (CP2)

**Spec block CP1:** theo template `P_stock_pitch_00` mục 7.1 — gồm: ticker, số luận điểm, tóm tắt từng L, variant perception 3 câu, bảng self-assessment 4 dấu hiệu, multi-choice 5 option (a-e).

**Spec block CP2:** theo template `P_stock_pitch_00` mục 10.1 — gồm: ticker, số lo ngại, tóm tắt phản biện, lo ngại còn yếu, self-assessment R/R + conviction, multi-choice 4 option (a-d).

Agent KHÔNG ghép checkpoint block vào MD final. Sau khi user confirm cả 2 CP, MD final compose từ đầu theo structure 15 mục (mục 2 spec).

## 4. Compose workflow step-by-step

**Bước 1 — Pre-flight:** P pack hỏi user 3 câu (ticker, memo tier 5C, branding info).

**Bước 2 — Compose Stage 1:** P pack chạy mục 3, 4 → output structured.

**Bước 3 — Compose Stage 2:** P pack chạy 4-7 luận điểm + variant perception → output structured.

**Bước 4 — Render Stage 2 partial cho checkpoint 1:** O pack render draft thesis + variant perception thành block ngắn cho user xem (KHÔNG render full MD).

**Bước 5 — Wait checkpoint 1:** dừng turn, đợi user.

**Bước 6 — User phản hồi checkpoint 1:**
- Confirm → Bước 7
- Thêm/bớt luận điểm → P pack refine, quay lại Bước 4
- Refine variant perception → P pack refine, quay lại Bước 4
- Abort → STOP, không render file

**Bước 7 — Compose Stage 3:** P pack chạy mục 11, 12.

**Bước 8 — Compose Stage 4:** P pack chạy mục 13 với honest steelman.

**Bước 9 — Render Stage 4 partial cho checkpoint 2:** O pack render bear case summary + lo ngại còn yếu thành block cho user xem.

**Bước 10 — Wait checkpoint 2:** dừng turn, đợi user.

**Bước 11 — User phản hồi checkpoint 2:**
- Confirm → Bước 12
- Phản biện thêm → P pack refine, quay lại Bước 9
- Bổ sung lo ngại → P pack refine, quay lại Bước 9
- **Abort → STOP, không render file. Ghi note "Conviction không đủ, abort recommendation."**

**Bước 12 — Compose Stage 5:** P pack chạy mục 1, 2, 14, 15.

**Bước 13 — Render full MD:** O pack ghép 15 mục + header + disclaimer thành file `stock_pitch_<TICKER>_<YYYYMMDD>.md`.

**Bước 14 — Self-audit:** chạy 12 câu self-check P pack mục 14.

**Bước 15 — Present MD:** xuất nội dung MD trong message (Claude Desktop), user copy/save thủ công.

**Bước 16 — Output cuối:** MD final đã đủ structured để consume bằng tool render bên ngoài. Pack này không tự render binary.

## 5. Guide render pptx — 15 slide [LEGACY]

> **Legacy spec — sẽ dọn ở audit pass tiếp theo.** Render pptx out of scope pack này. Giữ lại spec dưới đây làm reference cho việc build tool render bên ngoài.

Pptx là format gửi KH cuối cùng — render thực hiện downstream với MD final làm input.

**Layout chung:**

- Tỷ lệ 16:9
- Footer mỗi slide: "Báo cáo phân tích [TICKER] • [tên công ty/phòng ban] • [DD/MM/YYYY]" + số slide
- Color palette: ưu tiên branding công ty user. Default nếu chưa có: navy (#1E2761) + white + accent xanh dương (#3D5A80) hoặc đỏ tươi (#990011)
- Typography: title 32-40pt, sub-title 20-24pt, body 14-16pt, stat callout 60-72pt
- Mỗi slide có visual element (icon, chart, stat callout, bảng) — không text-only

**Mapping 15 slide:**

| Slide | Nội dung | Layout gợi ý |
|---|---|---|
| 1 | Cover (header branding + title + recommendation block + thesis chính + ngày) | Half-bleed background + logo + large title |
| 2 | Tóm tắt khuyến nghị (4 stat callout giá target + 4 luận điểm cốt lõi) | Top: 4 stat callout. Bottom: 4 numbered icon + bullet |
| 3 | Hồ sơ doanh nghiệp (thông tin cơ bản + cơ cấu kinh doanh) | Two-column: left bảng thông tin, right list mảng kinh doanh |
| 4 | Dữ liệu giao dịch phiên + mức kỹ thuật | Top: 4 stat callout phiên (open/high/low/value). Bottom: bảng mức kỹ thuật |
| 5 | Luận điểm 01 | Stat callout headline + 3-4 sub-point card |
| 6 | Luận điểm 02 | Tương tự, layout có thể đổi (variant: timeline, comparison) |
| 7 | Luận điểm 03 | Tương tự |
| 8 | Luận điểm 04 | Tương tự |
| 9 | Luận điểm 05 (nếu có) | Tương tự |
| 10 | Luận điểm 06-07 (nếu có, có thể gộp 2 luận điểm vào 1 slide) | Tương tự |
| 11 | Chiến lược thực thi 3 lớp + quản trị rủi ro | 3 column cards (Lớp 01/02/03) + bottom: bảng cắt lỗ + khung thời gian |
| 12 | Mục tiêu chốt lãi | Bảng N mốc + box R/R summary |
| 13 | Bear case | 6-7 cards: lo ngại + phản biện. Bottom: callout "Lo ngại còn yếu" highlighted |
| 14 | Tóm tắt hành động + 3 kịch bản | Top: hành động. Bottom: 3 column cards (cơ sở/tích cực/thận trọng) |
| 15 | Disclaimer + liên hệ | Plain text disclaimer + branding contact |

**Quy tắc visual quan trọng:**

- **Slide 5-10 luận điểm:** mỗi slide có 1 headline data point lớn (60-72pt). Đây là điểm KH nhớ nhất từ slide đó
- **Slide 13 bear case:** lo ngại còn yếu phải highlighted distinctly (background khác, border khác, hoặc icon cảnh báo). Mục đích: KH thấy rõ analyst honest, không "all bullish"
- **Slide 14 ba kịch bản:** không dùng từ "xác suất cao/vừa/thấp". Mỗi kịch bản dùng "Trigger" làm header con
- **Variant perception:** highlighted với box riêng, có thể dùng quote style hoặc background nhạt

**File output pptx:** `stock_pitch_<TICKER>_<YYYYMMDD>.pptx` (legacy reference).

## 6. Guide render docx (optional) [LEGACY]

> Render binary out of scope. Section giữ làm reference cho tool render bên ngoài.

Docx ít dùng — chủ yếu khi user muốn archive formal hoặc gửi qua email với file đính kèm.

Layout:
- Cover page (1 trang): logo + title + recommendation block + thesis chính + ngày
- Section 1-15 theo MD, mỗi section 1 heading
- Bảng giữ nguyên, format theo style chuẩn
- Footer: số trang + "Báo cáo phân tích [TICKER]"

File output: `stock_pitch_<TICKER>_<YYYYMMDD>.docx`.

## 7. Self-check checklist O pack

- [ ] 15 mục đầy đủ theo thứ tự, heading exact theo mục 2 spec
- [ ] Header branding đã render đúng (có branding user cung cấp / plain nếu không)
- [ ] Block khuyến nghị có đủ 4 dòng (khuyến nghị MUA / giá hiện tại / mục tiêu trung hạn / tiềm năng)
- [ ] Section 1 tóm tắt khuyến nghị in 4 luận điểm mạnh nhất (kể cả khi báo cáo có > 4 luận điểm)
- [ ] Section 2 hồ sơ có thông tin cơ bản + 3-6 mảng kinh doanh
- [ ] Section 3 có 3 sub-section (dữ liệu phiên / dòng tiền tổ chức / mức kỹ thuật)
- [ ] Số luận điểm trong 4-7 (section 4-10 flex)
- [ ] **Variant Perception explicit** trong 1 luận điểm với 3 câu chuẩn (consensus sell-side / retail / thesis khác)
- [ ] Section 11 có cấu trúc N lớp + disclaimer "tham khảo, không bắt buộc"
- [ ] Section 11 có 3 thông số quản trị rủi ro (cắt lỗ cứng / dự phòng / khung thời gian)
- [ ] Section 12 có bảng N mốc chốt lãi + summary R/R
- [ ] Section 13 có 5-7 lo ngại + phản biện
- [ ] **Section 13 có "Lo ngại còn yếu sau phản biện" highlighted** — không "all win"
- [ ] Section 14 ba kịch bản dùng "Trigger" header con, **không có % xác suất**
- [ ] Section 14 có note "kịch bản là hệ thống điều kiện kỹ thuật, không phải dự báo chắc chắn"
- [ ] Section 15 disclaimer đầy đủ theo branding info user cung cấp
- [ ] **Không có chỉ báo trend ở bất kỳ section nào** — không từ "xu hướng" theo nghĩa breadth, không "đang rơi từ vùng quá mua / đang bật từ đáy"
- [ ] K hygiene: không lộ ký hiệu DB raw — dịch theo bảng `K_agent_db_00` mục 5.2 (field name, score raw, zone code, period code, money_flow_score raw)
- [ ] Số liệu định lượng đã quy đổi đơn vị (giá đồng, BCTC tỷ đồng, % thập phân nhân 100, tỷ đồng cho NN/TD)
- [ ] Mỗi tin có dẫn link, mỗi claim quan trọng có nguồn
- [ ] File save đúng tên `stock_pitch_<TICKER>_<YYYYMMDD>.md`
- [ ] Đã hỏi user có cần render pptx không sau khi present MD
- [ ] Xuất nội dung MD trong message (Claude Desktop)

## 8. Output contract

O pack render structured content P pack sinh thành 1 file MD final theo structure rigid 15 mục. Không thêm/bớt nội dung, không tự ý insert section.

Khi P pack abort ở checkpoint 1 hoặc 2, **O pack KHÔNG render file** — agent thông báo abort cho user và dừng. Mục đích: tránh sản phẩm half-baked có thể bị gửi nhầm cho KH.

User explicit yêu cầu format khác (pptx / docx) → O pack render bổ sung theo guide mục 5-6. **MD luôn là source of truth** — pptx và docx render từ MD.
