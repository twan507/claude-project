# P_invest_memo_05 — Tier 5A: PDF Deep-dive Forensic

Giai đoạn 5A của quy trình. Với mỗi mã top 3/ngành từ tier 3, đọc BCTC năm kiểm toán + BCTC quý soát xét gần nhất và thực hiện 6 tác vụ forensic để phát hiện red flag chất lượng BCTC.

Reference: `P_invest_memo_00` phần Flow chi tiết (overview), `P_invest_memo_00` phần Cơ chế checkpoint review, `P_invest_memo_04` (tier 3 output làm input), `K_agent_db_04` D1-D4 (methodology theo 4 type).

---

## 1. Mục tiêu & output expected

**Mục tiêu:** verify chất lượng fundamental thực sự của mỗi mã qua đọc BCTC đầy đủ — phát hiện red flag mà DB không thể bắt được (chỉ có metric summarized, không có thuyết minh, cross-reference BCTC).

**Tier 5A là gate trước modeling (5B) + memo (5C).** Mã có red flag nghiêm trọng loại tại đây — tránh tốn resource cho mã không thể vào position.

**Input (user phải upload):**

Cho mỗi mã top 3:

**Bắt buộc — 2 PDF (default):**

1. **BCTC năm kiểm toán đầy đủ mới nhất** (forensic chính)
   - Phải đầy đủ: 4 loại BCTC (bảng cân đối kế toán, báo cáo kết quả hoạt động kinh doanh, báo cáo lưu chuyển tiền tệ, thuyết minh BCTC)
   - Phải có báo cáo của kiểm toán viên độc lập (ý kiến kiểm toán + KAM — Key Audit Matters, các vấn đề kiểm toán chính + Emphasis of Matter — các vấn đề cần nhấn mạnh như going concern, tranh chấp pháp lý)
   - Nhiều công ty VN lớn (LargeCaps, một số MidCaps) gộp cả BCTN (Báo cáo thường niên) vào file này — trường hợp này không cần upload BCTN riêng

2. **BCTC quý có soát xét gần nhất** (số liệu cập nhật)
   - Quý mới nhất đã công bố (thường trễ 20-45 ngày sau kết thúc quý)
   - Có soát xét (review) của kiểm toán, không phải BCTC tự lập
   - Đầy đủ 4 loại BCTC (thuyết minh có thể gọn hơn BCTC năm)

**Exception — chỉ 1 PDF:**

Khi BCTC quý gần nhất **chính là Q4** của năm kiểm toán mới nhất → BCTC năm đã bao gồm Q4, không cần upload BCTC Q4 riêng.

Ví dụ: tháng 5/2026, BCTC năm 2025 vừa kiểm toán xong (bao gồm cả Q4/2025), và chưa có BCTC Q1/2026 công bố → chỉ cần 1 PDF là BCTC năm 2025.

**Optional — BCTN riêng:** chỉ upload nếu mã có BCTN phát hành riêng (không gộp trong BCTC năm).

**Xử lý theo timing trong năm:**

| Thời điểm chạy tier 5A | BCTC năm mới nhất | BCTC quý soát xét bổ sung |
|---|---|---|
| Q1 (T1-T3) | BCTC năm N-1 (có thể chưa ra, phải dùng N-2) | BCTC Q3 hoặc Q4/N-1 (nếu có) |
| Q2 (T4-T6) | BCTC năm N-1 (thường đã ra tháng 4) | BCTC Q1/N (nếu đã công bố) |
| Q3 (T7-T9) | BCTC năm N-1 | BCTC Q2/N |
| Q4 (T10-T12) | BCTC năm N-1 | BCTC Q3/N |

**Rule độ cũ BCTC năm:**

- BCTC năm cũ ≤ 15 tháng so với thời điểm chạy tier 5A: **chấp nhận**, tiếp tục forensic bình thường với BCTC năm + BCTC quý
- BCTC năm cũ 15-20 tháng: **cảnh báo vàng** — data có thể lạc hậu. BCTC quý soát xét bổ sung là bắt buộc để cập nhật. Flag trong checkpoint: "Forensic dựa trên BCTC năm N-2, độ tin cậy giảm"
- BCTC năm cũ > 20 tháng: **red flag đỏ** — doanh nghiệp công bố chậm hơn quy định 90 ngày sau kết thúc năm. Đây là tín hiệu quản trị yếu hoặc có vấn đề kiểm toán. Loại mã khỏi shortlist trước khi bắt đầu tier 5A

**Ví dụ cụ thể:**

- Tháng 2/2026, mã X chỉ có BCTC năm 2024 (14 tháng cũ) và BCTC Q3/2025 soát xét: chấp nhận, tiếp tục forensic
- Tháng 5/2026, mã Y có BCTC năm 2025 kiểm toán vừa ra (bao gồm Q4/2025), chưa có BCTC Q1/2026: chỉ cần 1 PDF BCTC năm 2025
- Tháng 5/2026, mã Z chỉ có BCTC năm 2024 (17 tháng cũ, BCTC 2025 chưa ra sau 5 tháng) và BCTC Q1/2026 soát xét: cảnh báo vàng, bắt buộc forensic với BCTC Q1/2026 bổ sung
- Tháng 8/2026, mã W chỉ có BCTC năm 2024 (20 tháng cũ): đỏ, loại — công bố quá chậm

Thiếu file bắt buộc: Agent không thể làm tier 5A cho mã đó, flag báo user upload hoặc loại mã khỏi shortlist.

**Triết lý: ít file, đọc kỹ.** 2 PDF đủ coverage forensic (BCTC năm cho 6 tác vụ + BCTC quý cho cập nhật) — không cần thêm file làm phân tán chất lượng đọc. BCTC quý không thay thế BCTC năm (soát xét không sâu như kiểm toán, không có KAM), chỉ bổ sung góc nhìn cập nhật để phát hiện deterioration gần đây.

**Output chính:**

1. **Red flag report 3-4 trang/mã** với đánh giá đỏ/vàng/xanh cho 6 tác vụ
2. **Decision:** clear for tier 5B / watch list / loại khỏi shortlist
3. **Key findings** chuyển cho tier 5B (modeling) và tier 5C (memo bear case)
4. **Report Checkpoint 5A per-stock** — mỗi mã 1 checkpoint, không batch

**Tần suất:** per-stock. Mỗi mã 1 session con. User có thể upload PDF và review tier 5A theo thứ tự ưu tiên (top High conviction trước).

**Thời gian session:** 60-75 phút/mã Agent work (đọc 2 PDF: BCTC năm 100-250 trang + BCTC quý 30-80 trang, đọc selective theo từng tác vụ). User review checkpoint 25-35 phút/mã.

---

## 2. Triết lý: forensic, không phải confirm

**Tier 5A đọc PDF với tư duy forensic** — tìm điểm yếu và bất thường, không tìm lý do tin mã. Tương đương vai trò của short-seller analyst (Hindenburg Research, Muddy Waters) — stress-test fundamental thay vì confirm.

**Nguyên tắc 4 của `P_invest_memo_00`** — bear case được steelman TRƯỚC khi bull case được viết. Tier 5A là nơi thu thập material cho bear case.

**Ít file đọc kỹ, hơn nhiều file đọc vội.** 2 PDF (BCTC năm + BCTC quý soát xét) đủ coverage forensic cho 6 tác vụ. BCTC năm là nguồn chính (kiểm toán đầy đủ, có KAM + Emphasis of Matter). BCTC quý bổ sung để cập nhật số liệu và phát hiện deterioration gần đây. Nếu mã có BCTN riêng (không gộp trong BCTC năm), upload thêm để phục vụ Tác vụ 6. Ưu tiên chất lượng đọc thay vì khối lượng file.

**Không bỏ qua "red flag nhẹ":** nhiều vụ scandal lớn (Enron, Luckin, một số case VN) có dấu hiệu nhỏ trong BCTC từ 2-3 năm trước khi bung lộ. Tier 5A ghi nhận cả red flag nhẹ, không chỉ catastrophic.

**Red flag ≠ loại ngay:** red flag có 3 mức (xem Section 5). Chỉ mức đỏ nặng dẫn đến loại thẳng. Mức vàng là cảnh báo cần theo dõi, mức xanh là clear. Decision cuối do user quyết định sau khi có đủ thông tin.

---

## 3. Sáu tác vụ forensic

### Tác vụ 1 — Reconciliation LNST vs CFO

**Mục đích:** phát hiện chất lượng lợi nhuận. LNST trên báo cáo có thể đẹp nhưng nếu không quy đổi được thành tiền mặt thực tế → chất lượng thấp, có thể do ghi nhận doanh thu sớm, vốn hoá chi phí, hoặc thao túng công nợ.

**Công thức cơ bản:**

- LNST (Net Income) — lấy từ Báo cáo kết quả hoạt động kinh doanh
- CFO (Cash Flow from Operations) — lấy từ Báo cáo lưu chuyển tiền tệ
- Tỷ lệ CFO/LNST = chất lượng quy đổi

**Đọc xu hướng 3-5 năm:**

| CFO/LNST trung bình 3Y | Đánh giá |
|---|---|
| > 1.0 | Chất lượng cao — LNST trùng khớp tiền thực, có thể cao hơn do khấu hao lớn |
| 0.7 — 1.0 | Chất lượng OK — bình thường, chênh lệch do timing vốn lưu động |
| 0.4 — 0.7 | Cảnh báo — một phần LNST chưa thu được tiền, cần kiểm tra DSO |
| < 0.4 | Red flag đỏ — LNST không convert tiền, nghi ngờ accounting |
| Âm | Red flag cực đoan — kinh doanh không tạo tiền dù báo lãi |

**Điều chỉnh theo type:**

- **SXKD:** áp chuẩn trên
- **NGANHANG:** không dùng reconciliation này — bank có cấu trúc cash flow khác (tiền huy động dân cư vs cho vay, không phải revenue conversion). Bank dùng các chỉ tiêu khác như Provision adequacy + Credit Cost
- **CK:** tương tự bank, cash flow phụ thuộc giao dịch. Dùng P&L volatility thay thế
- **BH:** có cấu trúc đặc thù (premium income + claim outflow + investment income). Xem Non-life (bảo hiểm phi nhân thọ — xe cộ, tài sản, sức khoẻ ngắn hạn) vs Life insurance (bảo hiểm nhân thọ — hợp đồng dài hạn) riêng

**Chi tiết cần tìm trong PDF:**

1. Mở Báo cáo lưu chuyển tiền tệ — xem "Lưu chuyển tiền thuần từ hoạt động kinh doanh". BCTC năm VN thường có comparative 1-2 năm trước → có 2-3 năm data
2. Mở Báo cáo kết quả hoạt động kinh doanh — xem "Lợi nhuận sau thuế" cùng năm
3. Nhiều BCTC năm (đặc biệt công ty lớn) có bảng "Các chỉ tiêu tài chính chủ yếu 5 năm" ở đầu file hoặc phụ lục — đọc thêm để có trend dài
4. Tính tỷ lệ và xu hướng
5. Nếu CFO/LNST < 0.5, đọc thuyết minh "Các khoản điều chỉnh chủ yếu" để hiểu vì sao chênh
6. Nếu PDF không có đủ 3-5 năm để đánh giá trend: flag trong checkpoint ("chỉ có data 2 năm, trend đánh giá hạn chế"), yêu cầu user upload BCTC năm trước nếu muốn đánh giá dài hơn

**Bổ sung với BCTC quý soát xét (bắt buộc trừ exception Q4):**

Khi có BCTC quý soát xét gần nhất (không phải Q4 của năm kiểm toán):

- Tính CFO/LNST cho quý đó (từ BCTC quý)
- So sánh với xu hướng 3-5 năm từ BCTC năm
- **Red flag cảnh báo nếu:** CFO/LNST quý mới < 50% trung bình 3-5 năm trước, hoặc CFO quý âm trong khi 3-5 năm trước đều dương
- Nếu có dấu hiệu xấu trong quý mới, flag rõ trong checkpoint — có thể là early signal deterioration

BCTC quý soát xét **không thay thế** BCTC năm cho đánh giá chính (soát xét không sâu như kiểm toán), chỉ bổ sung góc nhìn cập nhật.

**Red flag cụ thể:**

- CFO âm trong khi LNST dương nhiều năm liền — đỉnh điểm rủi ro
- CFO/LNST giảm mạnh qua các năm (năm gần nhất thấp hơn nhiều năm trước) — trend xấu
- Phải thu khách hàng tăng nhanh hơn doanh thu — có thể ghi nhận doanh thu sớm

### Tác vụ 2 — Giao dịch với bên liên quan

**Mục đích:** phát hiện giao dịch không minh bạch giữa mã và công ty mẹ / công ty con / cá nhân lãnh đạo. Đây là kênh tunneling (rút tiền khỏi công ty niêm yết sang entity khác) — đặc biệt phổ biến ở thị trường VN.

**Chi tiết cần tìm trong PDF:**

1. Mở BCTC — mục **"Giao dịch với bên liên quan"** trong thuyết minh (thường section 30-40 trong BCTC)
2. Liệt kê tất cả giao dịch:
   - **Cho vay / đi vay** bên liên quan (quy mô vs vốn chủ)
   - **Bán hàng / mua hàng** bên liên quan (% so với doanh thu / COGS)
   - **Đầu tư vào / từ** bên liên quan
   - **Phải thu / phải trả** bên liên quan (quy mô vs doanh thu)
   - **Lương, thưởng HĐQT/BKS** — đọc riêng, xem có bất thường không

**Ngưỡng đánh giá:**

| Tỷ lệ giao dịch bên liên quan / Vốn chủ | Đánh giá |
|---|---|
| < 10% | Xanh — bình thường |
| 10% — 30% | Vàng — cần đọc kỹ thuyết minh, xem có rõ ràng không |
| 30% — 50% | Cảnh báo — tunneling risk cao, cần check giao dịch cụ thể |
| > 50% | Đỏ — tunneling nghiêm trọng, cân nhắc loại |

**Chi tiết cụ thể đánh giá mức đỏ:**

- Cho vay bên liên quan > 30% vốn chủ mà không có lãi suất rõ hoặc lãi suất < lãi suất thị trường
- Bán hàng bên liên quan > 30% doanh thu với giá thấp hơn thị trường
- Phải thu bên liên quan tăng > 50% YoY mà không giải thích
- Giao dịch bên liên quan không được kiểm toán xác nhận giá thị trường ("arm's length" — giá giao dịch công bằng như giữa 2 bên độc lập)

**Lưu ý đặc thù:**

- **Conglomerate** (VIC, HPG Group, VPB Group...): giao dịch bên liên quan lớn là bản chất mô hình — 30-40% có thể bình thường. Quan trọng là giá có chuẩn thị trường không, có minh bạch không
- **Công ty gia đình:** nguy cơ tunneling cao, cần đọc ĐHCĐ xem phê duyệt giao dịch có đa số cổ đông độc lập tán thành không
- **Mã type NGANHANG:** giao dịch bên liên quan chịu quy định NHNN khắt khe hơn. Red flag nếu có Cross-lending với VPS / CTG-trade / CIC lớn bất thường

### Tác vụ 3 — Ý kiến kiểm toán + note quan trọng

**Mục đích:** kiểm toán độc lập là lớp bảo vệ cuối của investor. Ý kiến không chuẩn = red flag cực nghiêm trọng.

**Chi tiết cần tìm trong PDF:**

1. Mở BCTC năm kiểm toán — tìm **"Báo cáo của kiểm toán viên độc lập"** ở đầu
2. Đọc **phần "Ý kiến kiểm toán"** — có 4 loại:

| Loại ý kiến | Nghĩa | Red flag |
|---|---|---|
| Unqualified (chấp nhận toàn bộ) | BCTC trình bày trung thực hợp lý | Xanh — bình thường |
| Qualified (chấp nhận có ngoại trừ) | Có 1 hạng mục kiểm toán không đồng ý | Đỏ — tùy mức độ ngoại trừ |
| Adverse (không chấp nhận) | BCTC không trung thực | Đỏ nghiêm trọng — đề xuất loại; user override yêu cầu audit log mạnh (xem Section 5) |
| Disclaimer (từ chối ý kiến) | Không đủ bằng chứng để đưa ý kiến | Đỏ nghiêm trọng — đề xuất loại; user override yêu cầu audit log mạnh (xem Section 5) |

3. Đọc **"Vấn đề kiểm toán chính" (KAM — Key Audit Matters)** — kiểm toán liệt kê các vấn đề phức tạp đã xử lý trong kỳ. Đây là manh mối rủi ro lớn nhất của doanh nghiệp theo góc nhìn kiểm toán
4. Đọc **"Các vấn đề cần lưu ý" (Emphasis of Matter)** — thường về going concern, biến cố sau kỳ báo cáo, tranh chấp pháp lý

**Going concern emphasis = cảnh báo doanh nghiệp có thể không tiếp tục hoạt động** trong 12 tháng tới. Red flag đỏ nghiêm trọng.

**Công ty kiểm toán:**

| Firm | Độ tin cậy |
|---|---|
| Big 4 (EY, PwC, KPMG, Deloitte) | Cao nhất — hiếm miss sai sót |
| Firm lớn VN (A&C, AASC, UHY, Grant Thornton VN...) | Trung bình — audit theo chuẩn nhưng ít kinh nghiệm đa quốc gia |
| Firm nhỏ không tên tuổi | Thấp — rủi ro bỏ sót tăng |

**Red flag bổ sung:**

- Mã đổi công ty kiểm toán nhiều lần trong 3-5 năm
- Đổi từ Big 4 sang firm nhỏ → nguy cơ kiểm toán dễ hơn
- Ý kiến kiểm toán kỳ soát xét (interim) khác kỳ kiểm toán năm → data lạ

### Tác vụ 4 — Trend DSO / DIO / DPO (cash cycle)

**Mục đích:** detect accounting engineering — nhận ghi doanh thu sớm, đẩy tồn kho, chậm thanh toán bên bán để đẹp số.

**Công thức:**

- **DSO (Days Sales Outstanding)** = Phải thu khách hàng / Doanh thu × 365 — số ngày tiền bán hàng chưa thu
- **DIO (Days Inventory Outstanding)** = Tồn kho / COGS × 365 — số ngày hàng tồn kho
- **DPO (Days Payable Outstanding)** = Phải trả người bán / COGS × 365 — số ngày nợ nhà cung cấp
- **CCC (Cash Conversion Cycle)** = DSO + DIO − DPO

**Đọc trend 3-5 năm** (từ BCTC năm):

| Pattern | Đánh giá |
|---|---|
| DSO ổn định hoặc giảm | Xanh — quản lý công nợ tốt |
| DSO tăng 10-20% qua 2-3 năm | Vàng — cần kiểm tra |
| DSO tăng > 30% qua 2-3 năm, vượt median ngành đáng kể | Đỏ — có thể ghi doanh thu sớm hoặc công nợ khó đòi |
| DSO bất thường tăng 1 năm rồi giảm | Vàng — có thể do thay đổi kênh khách hàng, đọc thuyết minh |
| DIO tăng nhanh hơn tăng trưởng doanh thu | Vàng đến đỏ — tồn kho xấu |
| DPO tăng lớn | Vàng — có thể căng thanh khoản |

**Điều chỉnh theo ngành:**

- Ngành có chu kỳ dài (BĐS, xây dựng): DSO, DIO cao bình thường (> 180 ngày). So với **ngành mình** không phải chuẩn chung
- Ngành retail (MWG, FRT, DGW): DSO thấp (< 30 ngày), DIO vừa (60-90 ngày). Nếu DSO > 60 → red flag
- Ngành SX (thép, hoá chất): DIO vừa (60-120 ngày), DSO vừa (45-90 ngày)
- Bank, CK, BH: không áp tác vụ này — cấu trúc cash cycle khác

**Bổ sung với BCTC quý soát xét (bắt buộc trừ exception Q4):**

Tính DSO, DIO, DPO cho quý soát xét (annualize doanh thu quý × 4 trước khi tính). So với xu hướng từ BCTC năm:

- **Red flag cảnh báo nếu:** DSO quý mới tăng > 15% so với DSO năm trước, hoặc DIO quý mới tăng > 20%
- Lưu ý seasonality: có ngành có pattern mùa vụ trong DSO/DIO (VD: retail có DIO cao trước Tết). So với cùng quý năm trước, không phải quý liền trước

**Cross-check với Tác vụ 1:**

Nếu CFO/LNST thấp (< 0.5) **VÀ** DSO tăng mạnh → tín hiệu rõ doanh thu bị đẩy (ghi sớm chưa thu). 2 tín hiệu đồng thuận = red flag đỏ.

### Tác vụ 5 — Off-balance sheet + cam kết nợ tiềm ẩn

**Mục đích:** tìm nợ/nghĩa vụ chưa thể hiện trên bảng cân đối kế toán — bảo lãnh, hợp đồng thuê, nghĩa vụ pháp lý tiềm tàng, SPV (Special Purpose Vehicle — pháp nhân chuyên biệt được lập để tách rủi ro khỏi công ty mẹ, thường dùng cho securitization hoặc tài trợ dự án).

**Chi tiết cần tìm trong PDF:**

1. **Mục "Các khoản nợ tiềm tàng và cam kết"** trong thuyết minh BCTC (thường cuối)
2. Liệt kê:
   - **Bảo lãnh** cho bên thứ ba (đặc biệt công ty con, liên kết)
   - **Hợp đồng thuê dài hạn** (off-balance sheet trước chuẩn mực mới, nay phải on-balance theo IFRS 16 — chuẩn mực kế toán quốc tế về thuê / VAS — Vietnamese Accounting Standards)
   - **Tranh chấp pháp lý đang diễn ra** với ước lượng phải trả
   - **Cam kết đầu tư / mua bán hàng hoá** (ví dụ mua urea, quặng sắt dài hạn với giá cố định)
   - **Thư tín dụng chưa sử dụng** (cho ngân hàng)
3. **Công ty liên kết, liên doanh** không consolidate — đặc biệt nếu equity method holding lớn

**Ngưỡng đánh giá:**

| Tỷ lệ cam kết tiềm ẩn / Vốn chủ | Đánh giá |
|---|---|
| < 20% | Xanh |
| 20% — 50% | Vàng — ghi nhận, đọc kỹ nội dung |
| 50% — 100% | Cảnh báo — leverage thực tế cao hơn on-balance |
| > 100% | Đỏ — rủi ro cực kỳ lớn nếu bị trigger |

**Red flag cụ thể:**

- **Bảo lãnh cho công ty liên kết** > 30% vốn chủ — nếu công ty đó phá sản, mã phải trả
- **SPV** không consolidate mà có nghĩa vụ tài chính lớn
- **Tranh chấp pháp lý** ước lượng phải trả > 10% vốn chủ
- **Lô đất / dự án BĐS** chưa có pháp lý rõ mà đã ghi nhận — tiềm ẩn phải hoàn trả
- **Đảm bảo tiền gửi khách hàng** (SPV cho sản phẩm tài chính) không consolidate

### Tác vụ 6 — Management discussion + chiến lược

**Mục đích:** đánh giá chất lượng quản trị qua văn phong + nội dung của ban lãnh đạo trong báo cáo thường niên. Văn phong tự tin thừa nhận khó khăn + kế hoạch cụ thể = quản trị mạnh. Văn phong chung chung / né tránh / đổ lỗi ngoại cảnh = quản trị yếu.

**Nguồn đọc (3 trường hợp):**

1. **BCTN riêng (nếu user upload):** đọc trực tiếp
2. **BCTN gộp trong BCTC năm** (phổ biến với LargeCaps): đọc phần "Báo cáo của HĐQT", "Báo cáo Ban TGĐ", "Định hướng chiến lược" trong file BCTC năm
3. **Không có BCTN (cả riêng lẫn gộp):** downgrade Tác vụ 6 sang đánh giá hạn chế — đọc thuyết minh BCTC phần "Thông tin chung về doanh nghiệp" + web search công bố ĐHCĐ + thông cáo HĐQT. Flag trong checkpoint rằng Tác vụ 6 đánh giá dựa trên nguồn gián tiếp

**Chi tiết cần tìm:**

1. **"Báo cáo của HĐQT / Ban Tổng Giám Đốc"**
2. **"Kết quả hoạt động năm qua"** — xem có tự phê bình khi miss target không
3. **"Định hướng chiến lược"** — xem có cụ thể không (số liệu, timeline) hay chung chung
4. **"Rủi ro trọng yếu"** — xem có liệt kê rủi ro thực sự hay chỉ generic

**Các điểm đánh giá (mỗi điểm 1 red/yellow/green):**

| Tín hiệu | Xanh | Vàng | Đỏ |
|---|---|---|---|
| Thừa nhận miss target | Có, giải thích nguyên nhân + plan cải thiện | Có đề cập nhưng né tránh | Không đề cập dù rõ ràng miss |
| Cụ thể kế hoạch | Số KPI + timeline + resource | Chỉ hướng chung chung | Chung chung, "tiếp tục phát triển" |
| Thảo luận rủi ro | Nêu 3-5 rủi ro cụ thể với biện pháp | Nêu chung chung như "rủi ro thị trường" | Không có phần rủi ro hoặc chỉ boilerplate |
| Disclose xung đột lợi ích | Công khai giao dịch bên liên quan + chuẩn | Nhắc qua không chi tiết | Không disclose |
| ESG / governance | Ban độc lập, tiểu ban kiểm toán, tuân thủ | Có ban nhưng không rõ vai trò | Không có, HĐQT tập trung gia đình |
| Định hướng M&A / Capex | Rõ ràng quy mô + đánh giá hiệu quả | Đề cập mơ hồ | Dấu hiệu empire-building không ROI |

**Phát hiện pattern "promotional":**

- Lạm dụng từ ngữ tự sướng ("tiên phong", "hàng đầu", "vượt trội") mà không có số liệu
- Khoe thứ hạng, giải thưởng không liên quan đến kinh doanh
- Tránh thảo luận khó khăn ngành
- Liên tục so sánh với quốc tế (Amazon, Tesla, Samsung...) để justify định giá cao

**Red flag cụ thể:**

- HĐQT thay đổi ≥ 2 thành viên trong 1 năm — cần đọc ĐHCĐ xem lý do
- CEO / CFO từ chức đột ngột — đặc biệt nếu không có thông cáo rõ
- Thay kiểm toán đồng thời với thay CFO → nguy cơ che giấu

---

## 4. Điều chỉnh 6 tác vụ theo 4 type

| Tác vụ | SXKD | NGANHANG | CK | BH |
|---|---|---|---|---|
| 1. CFO vs LNST | Áp đầy đủ | Thay bằng Provision adequacy + Credit Cost | Dùng P&L volatility vs ROE | Áp đầy đủ với adjust cho premium income |
| 2. Giao dịch bên liên quan | Áp đầy đủ | Check cross-lending với group ngân hàng + VPS/CIC | Check tự doanh trong mã liên quan + margin cross-lending | Áp đầy đủ |
| 3. Ý kiến kiểm toán | Áp đầy đủ | Áp đầy đủ + check NHNN disclosure | Áp đầy đủ + check SSC | Áp đầy đủ + check Bộ Tài chính |
| 4. Cash cycle | Áp đầy đủ | Không áp, thay bằng Loan concentration (top 10 borrowers vs tổng dư nợ) | Thay bằng Margin calls history + FVTPL volatility | Thay bằng Claims ratio trend + Reserves adequacy |
| 5. Off-balance sheet | Áp đầy đủ | Chú ý bảo lãnh + SPV sản phẩm + commitments | Chú ý margin commitment + underwriting commitment | Chú ý reinsurance arrangements |
| 6. Management discussion | Áp đầy đủ | Áp đầy đủ, thêm check CAR (Capital Adequacy Ratio — tỷ lệ an toàn vốn bank), tỷ lệ an toàn | Áp đầy đủ, thêm check thị phần margin | Áp đầy đủ, thêm check solvency ratio (tỷ lệ khả năng thanh toán của công ty bảo hiểm) |

---

## 5. Phân loại red flag — 3 mức

Sau khi chấm 6 tác vụ, tổng hợp thành 1 trong 3 mức cho mã:

### Xanh (Clear for tier 5B)

**Điều kiện:**
- Cả 6 tác vụ đều ở mức xanh, hoặc tối đa 1 tác vụ ở mức vàng nhẹ
- Không có red flag đỏ nào
- Ý kiến kiểm toán unqualified

**Action:** clear for tier 5B (modeling). Red flag nhẹ nếu có ghi nhận vào bear case của memo 5C.

### Vàng (Watch list — cần user quyết định)

**Điều kiện:**
- 1-2 tác vụ ở mức vàng rõ rệt, 0 tác vụ ở mức đỏ
- Hoặc 1 tác vụ ở mức đỏ nhẹ (ví dụ DSO tăng mạnh nhưng còn trong ngành, không kèm CFO/LNST thấp)

**Action:**
- Flag rõ trong checkpoint + mô tả cụ thể từng red flag vàng
- User quyết định:
  - (a) Chấp nhận rủi ro, clear for tier 5B
  - (b) Loại khỏi shortlist, quay lại tier 3 chọn mã khác
  - (c) Yêu cầu Agent deep-dive thêm 1 điểm cụ thể (ví dụ đọc kỹ thuyết minh giao dịch bên liên quan)

### Đỏ (Loại khỏi shortlist)

**Điều kiện (đủ ≥ 1):**
- Ý kiến kiểm toán: Qualified, Adverse, Disclaimer, hoặc Going Concern Emphasis
- Tác vụ 1 CFO/LNST < 0.4 trung bình 3 năm kèm trend xấu
- Tác vụ 2 giao dịch bên liên quan > 50% vốn chủ không rõ ràng
- Tác vụ 5 off-balance sheet commitments > 100% vốn chủ
- Cross-flag: CFO thấp + DSO tăng mạnh + off-balance cam kết lớn (đa tín hiệu tunneling/accounting)

**Action:** 
- Loại thẳng khỏi shortlist final
- Không đi tiếp tier 5B/5C cho mã này
- User có thể override với audit log (hiếm, cần lý do mạnh)
- Quay lại tier 3 chọn mã thay thế từ danh sách sát nút

---

## 6. Workflow per-stock — 8 bước

**Bước 1 — User upload PDF**

Agent confirm đã nhận:
- **Bắt buộc 1:** BCTC năm kiểm toán đầy đủ mới nhất
- **Bắt buộc 2 (trừ khi exception):** BCTC quý có soát xét gần nhất
  - **Exception:** nếu quý gần nhất chính là Q4 của năm kiểm toán → chỉ cần BCTC năm
- **Optional:** BCTN riêng (nếu mã có file riêng, không gộp trong BCTC năm)

Verify BCTC năm:
- File có phần "Báo cáo của kiểm toán viên độc lập" (không phải soát xét)
- Độ cũ: tính tháng giữa ngày phát hành BCTC năm và ngày hôm nay
  - ≤ 15 tháng → chấp nhận
  - 15-20 tháng → cảnh báo, BCTC quý soát xét bổ sung là bắt buộc
  - > 20 tháng → red flag đỏ, loại mã khỏi shortlist
- Có đủ 4 loại BCTC + thuyết minh

Verify BCTC quý soát xét:
- Có phần "Báo cáo soát xét" của kiểm toán
- Là quý mới nhất đã công bố (confirm với user đây có phải quý mới nhất không)
- Đầy đủ 4 loại BCTC
- Nếu BCTC quý cũng quá cũ (> 6 tháng): flag, có thể user upload file sai

Kiểm tra BCTN có gộp trong BCTC năm không: scan mục lục tìm phần "Báo cáo của HĐQT / Ban TGĐ", "Định hướng chiến lược", "Rủi ro trọng yếu". Nếu không có → yêu cầu user upload BCTN riêng.

Nếu thiếu file bắt buộc hoặc file không đủ chất lượng → flag yêu cầu user upload lại. KHÔNG fallback về DB để thay thế.

**Bước 2 — Agent đọc cấu trúc PDF**

Agent dùng skill pdf-reading để scan cấu trúc của 2 file (hoặc 1 file nếu exception):

**File 1 — BCTC năm kiểm toán:**
- Số trang, mục lục đầy đủ
- Locate các phần cần cho từng tác vụ:
  - Báo cáo kiểm toán (đầu file) → Tác vụ 3
  - Bảng cân đối kế toán → Tác vụ 1, 4
  - Báo cáo KQHĐKD → Tác vụ 1
  - Báo cáo lưu chuyển tiền tệ → Tác vụ 1
  - Thuyết minh "Giao dịch bên liên quan" → Tác vụ 2
  - Thuyết minh "Cam kết và nợ tiềm tàng" → Tác vụ 5
  - Báo cáo HĐQT / Ban TGĐ (nếu gộp BCTN) → Tác vụ 6
  - Định hướng chiến lược, Rủi ro → Tác vụ 6

**File 2 — BCTC quý soát xét (nếu có):**
- Số trang (thường mỏng hơn BCTC năm, 30-80 trang)
- Locate:
  - Báo cáo soát xét → verify ý kiến + ghi chú bất thường
  - 4 loại BCTC → Tác vụ 1, 4 bổ sung
  - Thuyết minh (có thể rút gọn) → check có giao dịch bên liên quan mới hay không

**Đọc selective, không đọc toàn bộ:** BCTC năm có thể 100-250 trang, phần lớn là bảng chi tiết. Agent đọc các section cụ thể theo tác vụ, không đọc tuần tự từ đầu.

**Bước 3 — Tác vụ 1 (CFO vs LNST)**

- Extract số liệu từ Báo cáo KQHĐKD + Báo cáo lưu chuyển tiền 3-5 năm
- Tính tỷ lệ, đọc trend
- Chấm red/yellow/green

**Bước 4 — Tác vụ 2 (Giao dịch bên liên quan)**

- Đọc thuyết minh section "Giao dịch bên liên quan"
- Liệt kê top 5 giao dịch lớn nhất + quy mô
- Tính tỷ lệ tổng / vốn chủ
- Chấm

**Bước 5 — Tác vụ 3 (Ý kiến kiểm toán)**

- Đọc Báo cáo kiểm toán đầu BCTC năm
- Ghi nhận loại ý kiến + KAM + Emphasis of Matter
- Kiểm tra công ty kiểm toán
- Chấm

**Bước 6 — Tác vụ 4 (Cash cycle DSO/DIO/DPO)** (SKIP nếu type NGANHANG/CK/BH, thay bằng tác vụ tương ứng)

- Tính DSO, DIO, DPO từ BCTC 3-5 năm
- So trend + so với median ngành (từ industry_finstats đã có tier 2)
- Cross-check với Tác vụ 1
- Chấm

**Bước 7 — Tác vụ 5 (Off-balance sheet) + Tác vụ 6 (Management discussion)**

- Đọc thuyết minh cam kết + công ty liên kết không consolidate
- Đọc management discussion trong BCTN
- Chấm

**Bước 8 — Tổng hợp + xuất checkpoint 5A**

- Phân loại red flag 3 mức (Section 5)
- Decision: clear / watch / loại
- Chuẩn bị key findings cho tier 5B và 5C
- Xuất checkpoint theo template Section 7

---

## 7. Template báo cáo Checkpoint 5A (per-stock)

Template này cho **1 mã**. Nếu có 9 mã top 3 final, chạy 9 checkpoint 5A riêng biệt.

```
# Checkpoint 5A — Forensic [Mã X] [ngày]

## 1. Summary quyết định
Mức red flag: [Xanh / Vàng / Đỏ]
Decision: [Clear for tier 5B / Watch list - cần user quyết / Loại khỏi shortlist]

Lý do chính (1-2 câu): [ví dụ: "CFO/LNST 0.85 trung bình 3Y ổn định, giao dịch bên liên quan 12% vốn chủ đều minh bạch, ý kiến kiểm toán unqualified — clear."]

## 2. Bối cảnh đầu vào
- Mã: [X] — ngành [Y] — type [SXKD/NGANHANG/CK/BH]
- Tier 3 tier conviction: [High/Medium/Low] — tổng [N]đ
- PDF đã đọc:
  - BCTC năm [YYYY] kiểm toán bởi [firm] — [số trang] — độ cũ [X tháng]
  - BCTC Q[N]/[YYYY] soát xét — [số trang] — quý mới nhất đã công bố
    (hoặc "Exception: quý gần nhất là Q4 đã gộp trong BCTC năm, không cần file riêng")
  - BCTN riêng [YYYY] (nếu có) — [số trang], hoặc "BCTN gộp trong BCTC năm" / "Không có BCTN riêng, đánh giá Tác vụ 6 dựa trên thuyết minh + web search"
- Ngày đọc: [ngày]
- Độ phủ data: [ví dụ "BCTC năm có comparative 2 năm = 2 năm data cho T1 và T4, BCTC Q1/2026 soát xét bổ sung Q1/2026"]
- PDF nào thiếu/không đầy đủ: [flag nếu có]
- Timing note (nếu relevant): [ví dụ "Đang Q2/2026, BCTC năm 2025 đã ra tháng 4, BCTC Q1/2026 soát xét vừa công bố tháng 5"]

## 3. Kết quả 6 tác vụ forensic

### Tác vụ 1 — CFO vs LNST: [Xanh/Vàng/Đỏ]

Số liệu từ BCTC năm 2025 (comparative 2023, 2024) + bảng 5 năm phụ lục (tỷ đồng):
| Năm | LNST | CFO | CFO/LNST |
|---|---|---|---|
| 2021 | 1,200 | 1,050 | 0.88 |
| 2022 | 1,450 | 1,380 | 0.95 |
| 2023 | 1,680 | 1,520 | 0.90 |
| 2024 | 1,890 | 1,620 | 0.86 |
| 2025 | 2,100 | 1,780 | 0.85 |

Nguồn: bảng chính BCTC 2025 (2023-2025) + bảng 5 năm phụ lục (2021-2022).

Đánh giá: trung bình 0.89 — chất lượng OK, ổn định. Không có trend xấu. **Xanh.**

### Tác vụ 2 — Giao dịch bên liên quan: [Xanh/Vàng/Đỏ]

Top giao dịch:
- Cho vay công ty con X: 450 tỷ (5% vốn chủ) — lãi suất thị trường, minh bạch
- Bán hàng công ty liên kết Y: 1,200 tỷ (15% doanh thu) — giá thị trường xác nhận bởi kiểm toán
- Phải thu công ty con Z: 180 tỷ (2% vốn chủ)

Tổng giao dịch bên liên quan: 1,830 tỷ = 20% vốn chủ. **Vàng — quy mô trung bình, cần theo dõi nhưng không bất thường.**

### Tác vụ 3 — Ý kiến kiểm toán: [Xanh/Vàng/Đỏ]

- Firm: KPMG (Big 4)
- Ý kiến 2025: Unqualified
- KAM: (1) Đánh giá giảm giá trị BĐS đầu tư dài hạn; (2) Ước tính công nợ bảo hành sản phẩm
- Emphasis of Matter: Không có

**Xanh** — ý kiến chuẩn, KAM chỉ là vấn đề ước tính thông thường.

### Tác vụ 4 — Cash cycle: [Xanh/Vàng/Đỏ] (hoặc NA nếu NGANHANG/CK/BH)

| Năm | DSO | DIO | DPO | CCC |
|---|---|---|---|---|
| 2023 | 45 | 78 | 32 | 91 |
| 2024 | 48 | 80 | 35 | 93 |
| 2025 | 52 | 85 | 38 | 99 |

Median ngành: DSO 50, DIO 82, DPO 40, CCC 92.

Đánh giá: DSO tăng 7 ngày trong 2Y, nằm trong ngành. DIO ổn định. Không có pattern alarming. **Xanh.**

### Tác vụ 5 — Off-balance sheet: [Xanh/Vàng/Đỏ]

Liệt kê:
- Bảo lãnh công ty con A: 250 tỷ (3% vốn chủ)
- Cam kết mua nguyên liệu dài hạn: 500 tỷ (6% vốn chủ) — giá floating
- Tranh chấp pháp lý: 30 tỷ ước lượng (0.3%)

Tổng: 780 tỷ = 9% vốn chủ. **Xanh — quy mô nhỏ, không bất thường.**

### Tác vụ 6 — Management discussion: [Xanh/Vàng/Đỏ]

- Thừa nhận miss target revenue 2024 (95% vs plan), giải thích chi tiết do biến động tỷ giá. Kế hoạch 2025 điều chỉnh guidance hợp lý. **Xanh.**
- Chiến lược rõ: Capex 800 tỷ mở rộng nhà máy X với ROI target 18% trong 3 năm. **Xanh.**
- Rủi ro thảo luận: 4 rủi ro cụ thể (giá nguyên liệu, tỷ giá, cạnh tranh, regulatory ESG). **Xanh.**
- Governance: HĐQT 7 người, 3 độc lập. Tiểu ban kiểm toán hoạt động đúng. **Xanh.**

Không có thay đổi HĐQT/CEO/CFO bất thường.

**Tổng thể xanh.**

## 4. Số liệu kỹ thuật key
- CFO/LNST trung bình 3Y: 0.89
- Giao dịch bên liên quan / vốn chủ: 20%
- Off-balance sheet / vốn chủ: 9%
- DSO trend 3Y: +7 ngày
- Ý kiến kiểm toán: Unqualified (KPMG)

## 5. Key findings cho tier 5B và 5C

Tier 5B (Modeling):
- Giả định tăng trưởng doanh thu: nếu giả định cao, cần đối chiếu DSO có thể tăng thêm (working capital lớn hơn)
- Giả định biên: trong BCTN có đề cập áp lực giá nguyên liệu 2025 — cân nhắc biên giảm 0.5-1%

Tier 5C (Memo — bear case):
- Giao dịch bên liên quan 20% vốn chủ — rủi ro tunneling mức trung bình, cần theo dõi
- DSO tăng từ 45 lên 52 trong 2Y — có thể là early signal công nợ xấu
- Cam kết mua nguyên liệu 500 tỷ — nếu giá nguyên liệu giảm mạnh, mã ôm giá cao

## 6. Lựa chọn sát nút

Nếu user không đồng ý đánh giá xanh:
- (a) Có thể downgrade sang vàng nếu user lo ngại DSO tăng
- (b) Có thể deep-dive thêm giao dịch bên liên quan với công ty liên kết Y (bán hàng 15% doanh thu) — review từng hợp đồng

## 7. Câu hỏi chờ user

Mã [X]: Decision Clear for tier 5B. Tiếp tục?

Hoặc muốn:
- (a) Deep-dive thêm 1 tác vụ cụ thể (ghi rõ tác vụ nào)
- (b) Downgrade mức đánh giá (ví dụ từ xanh sang vàng)
- (c) Loại mã khỏi shortlist dù đánh giá xanh (có lý do qualitative)

Nếu confirm → end session, tier 5B session kế tiếp cho mã này.
Nếu loại → Agent quay lại tier 3, chọn mã thay thế từ sát nút.
```

**Độ dài target:** 3-4 trang/mã.

---

## 8. Ví dụ generic — 3 case

### Case 1 — Mã clear for tier 5B (xanh)

Tất cả 6 tác vụ đều xanh hoặc tối đa 1 vàng nhẹ. Như ví dụ trong template Section 7.

**Decision:** Clear for tier 5B. Không cần user quyết định. Tier 5B proceed.

### Case 2 — Mã watch list (vàng)

**Tình huống:** mã có:
- Tác vụ 1: CFO/LNST 0.6 trung bình 3Y (vàng)
- Tác vụ 2: giao dịch bên liên quan 35% vốn chủ (vàng)
- Tác vụ 3: ý kiến kiểm toán unqualified nhưng có Emphasis of Matter về tranh chấp pháp lý (vàng nhẹ)
- Tác vụ 4-6: xanh

**Cross-flag analysis:** Tác vụ 1 + Tác vụ 2 kèm nhau có thể là tín hiệu tunneling — LNST không convert tiền + giao dịch bên liên quan lớn. Nhưng chưa đủ mức đỏ vì các chỉ số còn trong ngưỡng "vàng" không phải "đỏ nghiêm trọng".

**Decision:** Watch list. User quyết định:
- (a) Chấp nhận rủi ro, clear for tier 5B với ghi nhận bear case mạnh
- (b) Loại khỏi shortlist, quay lại tier 3 chọn mã thay
- (c) Deep-dive thêm giao dịch bên liên quan để xác định có tunneling không

### Case 3 — Mã loại (đỏ)

**Tình huống:** mã có:
- Tác vụ 1: CFO âm 2/3 năm gần, CFO/LNST trung bình 3Y = -0.2 (đỏ nghiêm trọng)
- Tác vụ 3: ý kiến kiểm toán 2025 **Qualified** về "không thu thập đủ bằng chứng giá trị hợp lý của khoản đầu tư dài hạn vào công ty liên kết X" (đỏ)
- Tác vụ 4: DSO tăng từ 60 → 110 ngày trong 2Y (đỏ)
- Các tác vụ còn lại vàng

**Cross-flag:** CFO âm + Qualified audit + DSO tăng dốc → 3 tín hiệu đồng pha của accounting engineering. Đây là pattern rõ ràng của Nguyên tắc 5 (dòng tiền dương + catalyst tiêu cực ≠ an toàn) — dù chưa có catalyst tiêu cực rõ, rủi ro accounting quá lớn.

**Decision:** Loại khỏi shortlist final. Không đi tiếp tier 5B/5C. Agent quay lại tier 3 chọn mã thay thế. User override cần lý do mạnh + audit log.

---

## 9. Failure mode

### 9.1. Agent skip tác vụ tốn thời gian

Tác vụ 2 (giao dịch bên liên quan) và tác vụ 6 (management discussion) tốn nhiều thời gian đọc. Agent có xu hướng lướt nhanh, bỏ sót chi tiết — dẫn đến đánh giá xanh oan.

**Xử lý:** quy tắc bắt buộc — mỗi tác vụ phải có ít nhất 1 đoạn trích dẫn cụ thể từ PDF (số trang + nội dung) để verify Agent đã đọc thực sự. Nếu checkpoint không có trích dẫn cho tác vụ nào → Agent chưa làm đủ, chạy lại.

### 9.2. Không điều chỉnh theo type

Agent áp tác vụ 4 (cash cycle DSO/DIO/DPO) cho mã ngân hàng → vô nghĩa, bank không có công nợ bán hàng truyền thống. Kết quả điểm xanh giả.

**Xử lý:** Bước 6 workflow bắt buộc skip tác vụ không áp dụng cho type, thay bằng tác vụ tương đương trong bảng Section 4. Với NGANHANG, tác vụ 4 thay bằng loan concentration (top 10 borrowers / tổng dư nợ). Với CK, thay bằng FVTPL volatility. Với BH, thay bằng claims ratio trend.

### 9.3. Bỏ sót emphasis of matter

Ý kiến kiểm toán Unqualified nhưng có emphasis of matter về going concern — Agent thấy "unqualified" là mặc định xanh, bỏ qua emphasis. Đây là red flag đỏ nghiêm trọng bị miss.

**Xử lý:** Bước 5 (tác vụ 3) bắt buộc đọc cả "Ý kiến" lẫn "Emphasis of Matter" lẫn "KAM" — mỗi phần có weight đánh giá riêng. Going concern emphasis → đỏ nghiêm trọng dù ý kiến là Unqualified.

### 9.4. Đánh giá cross-flag không đúng

Agent chấm từng tác vụ độc lập mà không đánh giá kết hợp. Ví dụ CFO/LNST = 0.6 (vàng) + DSO tăng 30% trong 2Y (vàng) — nếu đọc riêng là 2 vàng, nhưng đọc kết hợp là đỏ (2 tín hiệu đồng pha của accounting engineering).

**Xử lý:** Bước 8 (tổng hợp) bắt buộc check cross-flag:
- Tác vụ 1 vàng + Tác vụ 4 vàng → có thể là đỏ
- Tác vụ 2 vàng + Tác vụ 5 vàng → có thể là đỏ (tunneling)
- Tác vụ 3 Emphasis of Matter + bất kỳ tác vụ nào khác vàng → đỏ
- Tác vụ 6 thay đổi lãnh đạo + Tác vụ 3 thay kiểm toán → đỏ

### 9.5. Benchmark ngành không đúng

Tác vụ 4 so DSO mã với "chuẩn chung" thay vì median ngành. Ví dụ mã BĐS DSO 180 ngày — so với chung là cao, nhưng so với ngành BĐS là bình thường.

**Xử lý:** lấy benchmark từ industry_finstats của ngành mã (đã có từ tier 2). So DSO, DIO, DPO với median cùng ngành. Không áp chuẩn generic.

### 9.6. Không theo dõi thay đổi lãnh đạo

Tác vụ 6 yêu cầu check thay đổi HĐQT/CEO/CFO. Agent có thể bỏ qua nếu BCTN không nhấn mạnh — cần check cả Nghị quyết ĐHCĐ gần nhất.

**Xử lý:** nếu BCTN không có mục rõ về thay đổi lãnh đạo, web search thêm: `<ticker> thay CEO 2025`, `<ticker> thay HĐQT`. Đối chiếu danh sách HĐQT trong BCTC năm N vs năm N-1 nếu có cả 2 PDF.

### 9.7. User upload PDF sai hoặc thiếu

User có thể:
- Chỉ upload BCTC năm, thiếu BCTC quý soát xét gần nhất (khi không trong exception Q4)
- Upload BCTC quý không phải quý mới nhất (ví dụ upload Q2 khi đã có Q3 công bố)
- Upload BCTC năm quá cũ (> 20 tháng) nhưng không có BCTC năm mới hơn
- Upload BCTC không kiểm toán (chưa có báo cáo kiểm toán viên)
- Upload BCTN không kèm BCTC

Agent không check kỹ → phân tích trên data thiếu, lỡ mất early signal deterioration từ BCTC quý.

**Xử lý:** Bước 1 bắt buộc verify:
- **BCTC năm:** độ cũ theo rule Section 1 (≤ 15 tháng chấp nhận, 15-20 tháng bắt buộc có BCTC quý, > 20 tháng loại)
- **BCTC quý soát xét:** 
  - Bắt buộc có, trừ exception khi quý gần nhất = Q4 của năm kiểm toán
  - Verify là quý mới nhất đã công bố (hỏi user confirm nếu không chắc)
  - Độ cũ BCTC quý không quá 6 tháng
- **Kiểm toán vs soát xét:** BCTC năm phải có "Báo cáo kiểm toán", BCTC quý phải có "Báo cáo soát xét" (không phải BCTC tự lập)
- **Đầy đủ:** cả 2 file có đủ 4 loại BCTC + thuyết minh

**Case đầu năm (tháng 1-3) — BCTC năm mới chưa ra:**

Ví dụ: chạy tier 5A tháng 2/2026, BCTC năm 2025 đang kiểm toán chưa công bố. Xử lý:
- Chấp nhận BCTC năm 2024 (12-14 tháng cũ) làm file bắt buộc 1
- Bắt buộc upload BCTC Q4/2025 hoặc Q3/2025 soát xét (nếu có) làm file bắt buộc 2
- Nếu BCTC năm 2024 không có (phải dùng 2023 là 24+ tháng cũ) → red flag đỏ, doanh nghiệp công bố quá chậm, loại

**Case exception — Q4 vừa kiểm toán:**

Ví dụ: chạy tier 5A tháng 5/2026, BCTC năm 2025 kiểm toán vừa ra (bao gồm Q4/2025), chưa có BCTC Q1/2026. Xử lý:
- Chỉ cần 1 PDF BCTC năm 2025
- Ghi rõ trong checkpoint "Exception: quý gần nhất là Q4/2025 đã gộp trong BCTC năm, không cần BCTC quý riêng"

Nếu mismatch hoặc user không có file hợp lệ, flag yêu cầu user upload lại. **Không fallback về DB** — rule tier 5A là đảm bảo data toàn vẹn, đọc không được thì báo lại để user kiểm tra.

### 9.8. Dùng DB thay PDF khi user không upload

Agent có thể cố tìm BCTC trong news DB, web, hoặc dùng data summarized từ stock_finstats thay vì yêu cầu user upload PDF — data không đầy đủ bằng PDF kiểm toán gốc, không có thuyết minh chi tiết.

**Xử lý:** tier 5A **bắt buộc** dùng PDF user upload. DB không thay thế được:
- DB chỉ có metric summarized (ROE, P/E, Doanh thu...), không có thuyết minh
- DB không có ý kiến kiểm toán, KAM, Emphasis of Matter
- DB không có giao dịch bên liên quan chi tiết
- DB không có off-balance sheet commitments
- DB không có management discussion

Nếu user chưa upload PDF hoặc PDF không đầy đủ: Agent **không thể chạy tier 5A** — flag rõ cho user:
- Yêu cầu upload file đúng
- Hoặc user đồng ý skip tier 5A cho mã này (trường hợp đặc biệt, ghi audit log)

Skip tier 5A = rủi ro cao cho tier 5B/5C vì không có forensic validation. Nếu user muốn skip, memo phần bear case (5C) phải nêu rõ "không có forensic kiểm tra", và tier 6 sizing giảm 30-50%.

**Nguyên tắc:** chính xác và toàn vẹn dữ liệu > tốc độ. Đọc không được thì báo lại để kiểm tra, không tự thay thế bằng nguồn khác.

---

## 10. Đầu ra chuẩn để tier 5B và 5C dùng

Output tier 5A lưu file `tier5A_<ticker>_YYYYMMDD_confirmed.md` per-stock.

1. **Header:** mã, ngày, decision (xanh/vàng/đỏ)
2. **Bảng tổng hợp 6 tác vụ** với đánh giá từng tác vụ + lý do + trích dẫn page PDF
3. **Key findings cho tier 5B:**
   - Những giả định BCTC cần lưu ý khi modeling (DSO trend, off-balance commitments, cam kết mua nguyên liệu)
   - Các số liệu đã extract để tier 5B dùng (doanh thu, NPAT, CFO 3-5 năm gần nhất)
   - Flag về độ phủ data (2 năm / 3 năm / 5 năm comparative có được từ PDF)
4. **Key findings cho tier 5C (memo bear case):**
   - 3-5 điểm bear case cụ thể từ forensic
   - Mỗi điểm có số liệu + reference page PDF
5. **Red flag vàng/đỏ chi tiết** — để memo phần 6 (bear case) phân tích sâu
6. **Độ tin cậy forensic** — ghi rõ:
   - Full forensic: BCTC năm mới (≤ 15 tháng) + BCTC quý soát xét gần nhất + BCTN — độ tin cậy cao nhất
   - Medium forensic: BCTC năm mới + BCTC quý (không có BCTN riêng, gộp trong BCTC năm hoặc dùng web search bổ sung) — độ tin cậy cao
   - Limited forensic: BCTC năm cũ 15-20 tháng + BCTC quý soát xét — độ tin cậy trung bình, tier 5B/5C cần biết
   - Exception Q4: chỉ BCTC năm (quý gần nhất là Q4 đã gộp) — độ tin cậy cao, ghi rõ lý do
7. **Link audit log** nếu user override

File này enable vào project knowledge cho tier 5B session. Với mã đỏ (loại), không có tier 5B/5C — file chỉ để archive, ghi nhận đã loại + lý do.
