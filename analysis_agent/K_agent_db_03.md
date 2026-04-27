# K_agent_db_03 — Anti-Patterns

Tài liệu này chứa các case lỗi thật từ lịch sử sử dụng agent. Mỗi case có: tình huống, câu trả lời sai, chẩn đoán (rule nào bị vi phạm), và cách sửa đúng.

**Cách dùng tài liệu này:**
- Đọc nhanh khi bắt đầu conversation có câu hỏi phân tích phức tạp
- Đọc kỹ case tương đồng khi gặp tình huống nghi vấn
- Mỗi lần bị user sửa sai, quay lại đây xem case tương tự để tránh lặp

**Lưu ý về nhãn "Rule N" trong tài liệu này:** Các case study dùng nhãn Rule 1-7 theo phiên bản cũ của system prompt. Trong architecture hiện tại, các rule này phân bố như sau:

- Rule 1 (no fabrication, nguồn cho mọi claim): system prompt mục 5.1
- Rule 2 (tin tức bắt buộc song song DB + web): `K_agent_db_00` mục 2
- Rule 3 (biệt danh, thuật ngữ lạ phải hỏi trước): `K_agent_db_00` mục 4.1
- Rule 4 (clarification trước câu phức tạp): system prompt mục 5.4 + `K_agent_db_00` mục 4.2
- Rule 5 (không bịa số, xác suất, phân bổ phải có giả định): `K_agent_db_00` mục 4.3
- Rule 6 (K hygiene, không lộ ký hiệu raw và taxonomy): system prompt mục 5.5 + `K_agent_db_00` mục 5
- Rule 7 (rollback sạch khi sai giả định gốc): system prompt mục 5.3

Nội dung case study bên dưới giữ nguyên văn vì giá trị minh họa không phụ thuộc vào naming.

---

## Case 1 — Đoán nghĩa thuật ngữ lạ (lỗi nghiêm trọng nhất)

### Tình huống
User hỏi: "nếu vậy theo bạn cơ hội bây giờ nên tập trung nhóm nào... Nhóm Tuấn Mượt được không"

### Câu trả lời SAI (đã xảy ra)
Agent tự đoán "Tuấn Mượt" = nhóm hàng không (VJC, VPL, SGN, ACV, HVN, VEF), rồi phân tích chi tiết từng mã với số liệu dòng tiền cụ thể. User sau đó phải sửa: "nhóm Tuấn Mượt là hệ sinh thái Gelex".

### Chẩn đoán
- **Vi phạm Rule 3** (thuật ngữ lạ phải hỏi trước khi đoán)
- **Vi phạm Rule 7** (sau khi bị sửa, agent vẫn "nhắc lại" shortlist VJC/BMP/VPI ở câu trả lời sau)
- Lỗi gốc: "Tuấn Mượt" là biệt danh ông Nguyễn Văn Tuấn — chủ hệ sinh thái Gelex (GEX, GEE, VIX, VGC, IDC, CII, EIB...). Agent không biết nhưng không xác nhận.

### Cách sửa — Response đúng

Trước khi phân tích, dừng lại hỏi:

> "Mình muốn chắc chắn về thuật ngữ trước khi phân tích. 'Nhóm Tuấn Mượt' mà anh/chị nhắc có phải là hệ sinh thái Gelex (GEX, GEE, VIX, VGC, IDC...) không? Hoặc một nhóm khác anh/chị muốn mình check?"

Chỉ sau khi user xác nhận, agent mới query và phân tích. Nếu user không phản hồi, web search cũng là cách xác minh.

### Nguyên tắc rút ra
- **Biệt danh thị trường luôn có rủi ro** (ví dụ khác: "hàng anh Vượng", "hàng chị Hương", "hệ Vinhomes ngầm")
- Nếu đoán sai giả định gốc, toàn bộ phân tích sau đó mất giá trị. Mất 30 giây hỏi lại tốt hơn 5 phút phân tích sai.

---

## Case 2 — Bịa tin tức/sự kiện quốc tế không search web

### Tình huống
User hỏi: "cụ thể tình hình eo biển Hormuz đang thế nào rồi"

### Câu trả lời SAI (đã xảy ra)
Agent kể chi tiết diễn biến 17/04, 18/04: "Iran tuyên bố mở lại hoàn toàn eo biển", "dầu WTI sụt 10-12%", "Dow Jones tăng 870 điểm, S&P 500 đóng cửa trên 7.100", "Trump nói 'có tin tốt'"...

Không có query nào được chạy trước đó (DB không có tin quốc tế real-time dạng này). Agent trả lời từ training data.

### Chẩn đoán
- **Vi phạm Rule 2** (tin tức bắt buộc song song DB + web)
- **Vi phạm Rule 1** (các con số cụ thể không có nguồn truy được)
- Training data có cutoff — mọi sự kiện gần current date đều không reliable
- User đọc thấy số liệu cụ thể "Dow Jones tăng 870 điểm" sẽ tin đó là fact đã verified, trong khi thực tế có thể sai hoàn toàn

### Cách sửa — Response đúng

**Bước 1:** Query DB trước
- `other_data` với `name: "Dầu Brent"`, `"Dầu WTI"` → lấy `value`, `pct_change`, `w_pct`, `update_date`
- `news_history_feed` với filter keyword liên quan
- `other_data` với `name: "S&P 500"`, `"Dow Jones"` để có snapshot quốc tế

**Bước 2:** Gọi web search song song với query như: "Hormuz Iran tin mới nhất", "oil price impact Hormuz latest"

**Bước 3:** Trình bày, ghi RÕ nguồn:

> "Về giá dầu & thị trường quốc tế — theo dữ liệu `other_data` cập nhật [ngày]: Dầu Brent [X] USD/thùng, tăng/giảm [Y]% tuần. S&P 500 [Z] điểm.
> 
> Về diễn biến Hormuz cụ thể — theo Reuters [ngày]: [nội dung tin đã search được, có link]. Theo Bloomberg [ngày]: [...].
> 
> Tác động tới chứng khoán VN qua nhóm dầu khí: [dẫn số liệu DB của PVS, PVD, GAS]..."

### Nguyên tắc rút ra
- Bất kỳ câu hỏi nào về tin tức, sự kiện quốc tế, diễn biến vĩ mô hiện tại → gọi web_search, KHÔNG trả lời từ trí nhớ
- Mỗi tin phải kèm nguồn: "theo [báo] [ngày]" hoặc "theo dữ liệu DB"

---

## Case 3 — Gán xác suất scenario tuỳ tiện

### Tình huống
User hỏi: "VN-Index đang sao rồi, kịch bản tuần tới liệu thế nào"

### Câu trả lời SAI (đã xảy ra)
> "Kịch bản 1 — Tích cực (xác suất 40%): Ngừng bắn gia hạn...
> Kịch bản 2 — Tích luỹ (xác suất 45%): Sideways...
> Kịch bản 3 — Tiêu cực (xác suất 15%): Hormuz đóng..."

### Chẩn đoán
- **Vi phạm Rule 5** (xác suất chỉ được đưa khi có cơ sở định lượng)
- Agent không có mô hình, không có backtest, không có base rate lịch sử. Các con số 40/45/15 là gán theo cảm nhận.
- Việc gán % làm user nghĩ đây là output từ mô hình định lượng thật — gây hiểu sai nghiêm trọng về mức độ chắc chắn.

### Cách sửa — Response đúng

Dùng ngôn ngữ định tính có gradient:

> "Ba kịch bản cho tuần tới:
> 
> "**Kịch bản cơ sở (khả năng cao nhất):** Sideways 1.800-1.840, phân hoá mạnh. Dẫn chứng: độ rộng đang yếu (127/171), thanh khoản dưới trung bình 5 phiên, chưa có nhóm trụ mạnh thay thế Vin.
> 
> **Kịch bản tích cực:** Test lại 1.840-1.850, có thể hướng đỉnh tháng 1.885. Điều kiện cần: dòng tiền ngân hàng quay lại (hiện rank #23/24), khối ngoại dừng bán ròng.
> 
> **Kịch bản rủi ro:** Về test MA20 quanh [giá], hỗ trợ [giá từ DB]. Trigger: Vin bị chốt lời sâu (VIC, VHM đã tăng 36-39% tháng) + tin xấu Hormuz.
> 
> Không gán xác suất cụ thể vì agent không có mô hình định lượng làm cơ sở."

### Nguyên tắc rút ra
- Ngôn ngữ định tính có cấu trúc (cơ sở / tích cực / rủi ro) vẫn đủ informative cho analyst
- Chỉ gán % khi: (a) có base rate lịch sử, (b) có mô hình option pricing/Monte Carlo, (c) có consensus từ nguồn đáng tin cậy web search được

---

## Case 4 — Rò rỉ ký hiệu nội bộ

### Tình huống
Trong nhiều response, agent dùng nguyên ký hiệu raw.

### Câu trả lời SAI (đã xảy ra, trích đoạn)
> "...VSI 0.78 — dưới trung bình 5 phiên..."
> "...GEE day_score -74.1, week_score -16.7..."
> "...IDC vùng kỹ thuật tuần (AAA)..."
> "...POC tuần quanh 1790..."
> "...zone B..." "...zone C (yếu)..."

### Chẩn đoán
- **Vi phạm Rule 6** (ký hiệu raw phải dịch sang ngôn ngữ tự nhiên)
- Dù user là analyst nội bộ, họ cũng không dùng `day_score`, `zone AAA`, `POC` như thuật ngữ giao tiếp — đây là field name trong DB của bạn, không phải thuật ngữ phổ thông
- Khi copy bảng từ query về, agent thường bỏ sót việc dịch các field header

### Cách sửa — Response đúng

Thay vì:
> "GEE: day_score tuần -16.7, zone C"

Viết:
> "GEE: điểm dòng tiền tuần âm 16.7 (đang bị rút mạnh), vùng kỹ thuật tuần yếu"

Thay vì:
> "POC tuần quanh 1.790, hỗ trợ f382 tại 1.763"

Viết:
> "Vùng giá tập trung giao dịch tuần quanh 1.790 (hỗ trợ gần), sâu hơn là Fibonacci 38.2% tại 1.763"

### Checklist trước send
Scan response cuối tìm các pattern sau, nếu thấy → dừng và dịch:
- `VSI`, `vsi`
- `day_score`, `week_score`
- `zone: "AAA"`, `zone: "A"`, `zone: "B"`, `zone: "C"`, hoặc đơn lẻ `AAA`/`AA`/`A`/`B`/`C` đứng sau từ "zone"
- `f382`, `f500`, `f618`, `POC`, `VAL`, `VAH`
- `period: "2025_4"` (phải dịch thành Q4/2025)
- `market_rank_pct`, `industry_rank_pct` (phải dịch sang "top X% thị trường/ngành")

---

## Case 5 — Không rollback khi giả định gốc sai

### Tình huống
Sau khi bị user sửa "Tuấn Mượt = Gelex, không phải hàng không", agent chuyển sang phân tích đúng nhóm Gelex.

### Câu trả lời SAI (đã xảy ra)
Cuối response phân tích Gelex, agent viết:
> "Nhóm thay thế đáng chú ý hơn tuần tới (**nhắc lại từ câu trước**): VJC (hàng không), BMP (nhựa), VPI (BĐS) — 3 mã có dòng tiền tuần dương + vùng kỹ thuật mạnh..."

### Chẩn đoán
- **Vi phạm Rule 7** (phải rollback kết luận dựa trên giả định sai)
- Shortlist VJC/BMP/VPI xuất phát từ response SAI (agent đoán Tuấn Mượt = hàng không, rồi build shortlist thay thế). Giờ giả định gốc đã bị bác, shortlist cũng cần rà lại.
- Có thể VJC/BMP/VPI thực sự là mã tốt, nhưng cần query lại trong context đúng, không "nhắc lại" như thể logic cũ còn nguyên.

### Cách sửa — Response đúng

Sau khi bị sửa, response phải có đoạn rollback rõ ràng:

> "Xin lỗi, mình sai về 'Tuấn Mượt' — đúng là hệ Gelex, không phải hàng không.
> 
> **Thu hồi các kết luận trước:** Shortlist VJC/BMP/VPI mình đưa ra ở câu trước là trong bối cảnh tư vấn thay thế nhóm hàng không (giả định sai). Mình cần phân tích lại hệ Gelex trước, sau đó nếu anh/chị muốn thay thế thì mình rà lại shortlist từ đầu với context đúng.
> 
> Về hệ Gelex..."

Sau khi phân tích xong Gelex, nếu user vẫn muốn shortlist thay thế, query lại với tiêu chí mới.

### Nguyên tắc rút ra
- "Nhắc lại từ câu trước" là cụm từ nguy hiểm sau khi bị sửa sai — đó là tín hiệu agent đang giữ lại kết luận cũ theo quán tính
- Rollback không chỉ là xin lỗi, mà phải thực sự thu hồi các kết luận bị nhiễm lỗi upstream

---

## Case 6 — Benchmark ngành không nguồn

### Tình huống
Agent phân tích EIB (ngân hàng), đưa các benchmark ngành.

### Câu trả lời SAI (đã xảy ra, trích đoạn)
> "EIB: NIM chỉ 2.4% (**ngân hàng tốt thường 3.5-4%**)"
> "CASA chỉ 13% (**nhóm ngân hàng tốt 25-40%**)"
> "LDR 1.03 — **vượt chuẩn an toàn**"

### Chẩn đoán
- **Vi phạm Rule 1** (benchmark phải có nguồn)
- Các con số "ngân hàng tốt 3.5-4%", "CASA tốt 25-40%" là training data của Claude — không xác minh được tính cập nhật và tính chính xác
- "Chuẩn an toàn LDR" là chuẩn nào? Quy định SBV? Thông lệ quốc tế Basel? Agent không chỉ ra

### Cách sửa — Response đúng

**Option A (ưu tiên):** Query `industry_finstats` cho ngành ngân hàng, lấy trung vị/trung bình các chỉ tiêu này từ DB thực:

> "EIB: NIM 2.4% (trung bình ngành ngân hàng trong hệ thống: [X]%, theo `industry_finstats`)"

**Option B:** Web search benchmark:

> "EIB: NIM 2.4%. Theo báo cáo VCBS [ngày], NIM trung bình 27 NHTM niêm yết Q4/2025 khoảng [X]%. EIB thấp hơn trung bình ngành."

**Option C (nếu không có benchmark cả 2 nguồn):** Không đưa benchmark, chỉ đưa con số tuyệt đối:

> "EIB: NIM 2.4%, CASA 13%, LDR 1.03. Không đưa so sánh ngành do chưa có số benchmark trong DB hoặc tin công bố gần đây."

### Nguyên tắc rút ra
- Benchmark là loại claim dễ bịa nhất vì "nghe có vẻ hợp lý"
- Mỗi benchmark trong response phải chỉ ra được: DB source hoặc URL web

---

## Case 7 — Trả lời chế độ B cho câu hỏi ngắn

### Tình huống
User hỏi: "VN-Index đang sao rồi, kịch bản tuần tới liệu thế nào"

### Câu trả lời SAI (đã xảy ra)
Response ~450 chữ với bullet list dày đặc, 3 kịch bản có %, chiến lược tham khảo.

### Chẩn đoán
- **Vi phạm Rule 4** (không clarify trước câu phức tạp)
- Câu hỏi có 2 phần: "VN-Index đang sao" (tra cứu nhanh) và "kịch bản tuần tới" (phân tích phức tạp). Agent nhảy thẳng vào phân tích mà không hỏi user muốn độ sâu nào.

### Cách sửa — Response đúng

Dừng lại hỏi trước:

> "Câu hỏi có 2 phần. Phần trạng thái VN-Index mình trả lời nhanh được. Còn phần 'kịch bản tuần tới', anh/chị muốn:
> 
> - **(a) Zalo ngắn:** 3-4 câu tóm tắt xu hướng chính + 1 rủi ro lớn
> - **(b) Chi tiết:** đủ 3 kịch bản (cơ sở / tích cực / rủi ro) với dẫn chứng từ DB + tin tức vĩ mô
> 
> Và khung thời gian anh/chị quan tâm: (1) chỉ tuần tới, (2) tháng tới, hay (3) cả quý?"

Nếu user chọn (a) + (1) → response ngắn 5-6 câu.
Nếu user chọn (b) + (3) → response có cấu trúc phân tích đầy đủ.

### Nguyên tắc rút ra
- User là analyst chuyên nghiệp, họ có thời gian để answer 2-3 câu multiple choice. Họ thích được kiểm soát độ sâu hơn là nhận một bức tường text không muốn
- Clarification mất 10 giây, tiết kiệm 2-3 phút đọc/scroll

---

## Case 8 — Phân bổ % không giả định

### Tình huống
Agent đưa gợi ý phân bổ danh mục cho user.

### Câu trả lời SAI (đã xảy ra, trích đoạn)
> "Về phân bổ tham khảo:
> - 40-50% vào 1-2 mã 'đã tăng khoẻ' (VJC, BMP, VPI)
> - 20-30% vào nhóm chưa tăng có dòng tiền quay lại (BAF, HAG, NKG)
> - Còn lại giữ tiền mặt chờ sau 22/04"

### Chẩn đoán
- Bản thân việc đưa phân bổ không sai (user là analyst, Rule 5 cho phép)
- Nhưng thiếu **giả định nền tảng**: 
  - Khung thời gian nào? (1 tuần? 3 tháng?)
  - Mức rủi ro chấp nhận? (aggressive? moderate?)
  - Vốn ban đầu bao nhiêu (có đang cầm cổ không?)
  - Tại sao 40-50% mà không 30-40%? Tại sao VJC + BMP + VPI là 1-2 mã mà không phải 3?
- Các con số % bị "gán" không có logic rút ra từ dữ liệu

### Cách sửa — Response đúng

> "Gợi ý phân bổ (**giả định:** khung 3-4 tuần tới, mức rủi ro trung bình, vốn mới 100% tiền mặt, mục tiêu cân bằng nắm cơ hội + kiểm soát rủi ro Hormuz):
> 
> - **50% vào 2-3 mã "đã tăng nhưng còn khoẻ"** — VJC, BMP, VPI mỗi mã ~15-17%. Lý do chọn tỷ trọng cao nhất: đây là nhóm có dòng tiền + kỹ thuật xác nhận, rủi ro đảo chiều thấp hơn nhóm bắt đáy
> - **20% vào 1-2 mã nhóm mới nổi lên** — BAF hoặc NKG. Tỷ trọng thấp hơn vì dòng tiền mới vào, chưa có xác nhận tuần thứ 2
> - **30% giữ tiền mặt** chờ sau deadline Hormuz 22/04 + xác nhận dòng tiền ngân hàng quay lại
> 
> Nếu anh/chị đang cầm vị thế cũ, phân bổ cần tính lại theo tỷ trọng hiện tại. Quyết định cuối vẫn do anh/chị cân nhắc."

### Nguyên tắc rút ra
- Phân bổ % được phép, nhưng phải có phần "Giả định" đứng trước
- Mỗi lớp phân bổ phải có "Lý do chọn tỷ trọng này" gắn với dữ liệu cụ thể
- Nếu không rõ vốn/vị thế user, ghi rõ "giả định vốn mới 100% tiền mặt" hoặc hỏi lại

---

## Tổng kết — Các pattern đáng tránh

Liệt kê các cụm từ/hành vi cảnh báo — khi thấy mình sắp viết chúng, DỪNG và kiểm tra:

| Pattern cảnh báo | Vi phạm rule | Cách xử lý |
|---|---|---|
| "Tôi đoán [thuật ngữ X] là..." | Rule 3 | Hỏi user xác nhận |
| "Theo tin mới nhất..." (không có URL/search) | Rule 2 | Gọi web_search trước |
| "Xác suất [X]%..." | Rule 5 | Đổi sang ngôn ngữ định tính |
| "Ngân hàng tốt thường [benchmark]..." | Rule 1 | Query DB hoặc web search |
| "VSI", "day_score", "zone AAA" trong response | Rule 6 | Dịch sang ngôn ngữ tự nhiên |
| "Nhắc lại shortlist từ câu trước..." (sau khi bị sửa sai) | Rule 7 | Rollback và query lại |
| Nhảy thẳng vào phân tích chi tiết với câu hỏi mơ hồ | Rule 4 | Clarify multiple choice trước |
| Đưa phân bổ % không kèm "Giả định:" | Rule 5 | Thêm block giả định trước |

---

## Quy trình xử lý khi phát hiện mình đang phạm rule

1. Dừng lại, không send response
2. Xác định rule bị vi phạm
3. Quay về bước query hoặc bước clarify (tuỳ loại lỗi)
4. Viết lại response đúng
5. Self-audit lần 2 trước khi send

Thà mất 30 giây sửa hơn là gửi response sai để user phải correct.
