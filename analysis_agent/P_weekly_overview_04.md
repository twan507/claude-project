# P_weekly_overview_04 — Methodology + Self-audit + Edge cases + Output contract

File meta của pack `P_weekly_overview` — methodology chi tiết cho regime classification + sector bias logic + mapping vĩ mô↔18 ngành whitelist + technical-as-noise rule, self-audit 12 item, edge cases cross-cutting, và output contract. Dependency: `P_weekly_overview_00` master.

## 1. Methodology — Regime classification

### 1.1. Bốn regime — định tính

| Regime | Đặc điểm tổng thể | Ngụ ý chiến lược tuần tới |
|---|---|---|
| **Risk-on full** | Dòng tiền tuần dương mạnh (mean week_score ≥ +6 qua 18 ngành whitelist); chuỗi day_score 5 phiên đa số dương rõ; đa số ngành whitelist 18 tăng giá tuần (≥ 65%); NN mua ròng đáng kể; vĩ mô ủng hộ phần lớn ngành dẫn đầu | Universe rộng, **4-5 ngành quan tâm**, mức thận trọng thấp |
| **Risk-on selective** | Dòng tiền tuần dương nhẹ hoặc dao động trong tuần (mean week_score 0 đến +5); ngành phân hoá rõ giữa tăng và giảm; NN trung tính hoặc một chiều nhẹ; vĩ mô hỗn hợp | Chọn lọc kỹ, **3 ngành quan tâm**, ưu tiên ngành có cả flow mạnh và vĩ mô ủng hộ rõ |
| **Defensive only** | Dòng tiền tuần âm hoặc dương rất nhẹ (mean week_score -5 đến 0); đa số ngành whitelist 18 giảm giá tuần (≥ 55%); NN bán ròng; hoặc rủi ro vĩ mô đang materialize | **2 ngành quan tâm** — chỉ ngành phòng thủ + ngành có catalyst riêng đủ mạnh, mức thận trọng cao |
| **Đứng ngoài** | Dòng tiền tuần âm sâu kéo dài nhiều tuần (mean week_score ≤ -6 trong ≥ 3/4 tuần); hầu hết ngành whitelist 18 giảm giá; NN bán ròng kéo dài 2+ tháng; hoặc shock vĩ mô lớn | Giữ tiền mặt, **không chọn ngành quan tâm**, chỉ watchlist sang tuần sau |

### 1.2. Cách dùng bảng

Agent đọc số liệu thực tế từ 4 input (xem `P_weekly_overview_03` mục 1.1), đối chiếu mô tả định tính từng regime, chọn regime gần nhất với bức tranh tổng thể tuần.

**Combo input ranh giới 2 regime:** agent dùng judgment có ngữ cảnh — xu hướng đang cải thiện hay xấu đi so tuần trước, tin tức quan trọng nào sắp có — để chọn. **Không cố ép vào quy tắc cứng.**

Checkpoint 1 user review judgment, không review "ngưỡng có đúng không" — bảng định tính ngay từ đầu, không có ngưỡng số cứng để debate.

### 1.3. Conviction logic cho regime call

**HIGH:** 4/4 input đồng thuận rõ ràng + vĩ mô ủng hộ + không có signal mâu thuẫn.

**MID:** 3/4 input đồng thuận + 1 input ranh giới hoặc mâu thuẫn nhẹ. Cần monitor signal cụ thể.

**LOW:** 2/4 hoặc combo borderline + signal mâu thuẫn rõ. Regime chưa stable — chấp nhận downgrade conviction, kèm disconfirming signal chặt.

## 2. Methodology — Sector bias selection

### 2.1. Logic chọn ngành quan tâm

Sau khi xác định regime, chọn sector bias từ bảng 18 ngành whitelist phần 6 cross-check sub-section 5.4 phần 5 (tác động vĩ mô tuần này).

**Tiêu chí ngành quan tâm:**
- Industry rank (re-rank trong scope 18) ở top 1/3 (rank ≤ 6/18)
- Week_score dương rõ (≥ +5)
- **VÀ ít nhất 1 trong:**
  - Có tác động vĩ mô tích cực ở sub-section 5.4 phần 5
  - Có catalyst chính sách / mùa BCTC / M&A trong 1-3 tháng
  - P/E phân vị < 40% (rẻ tương đối — re-rating opportunity)
- **VÀ:** Dẫn dắt thật (≥ 60% mã trong ngành tăng giá tuần) — không phải trụ kéo

Số lượng ngành quan tâm theo regime:
- Risk-on full: 4-5 ngành
- Risk-on selective: 3 ngành
- Defensive only: 2 ngành
- Đứng ngoài: 0 ngành

### 2.2. Phân biệt dẫn dắt thật vs trụ kéo

Query `stock_snapshot` filter `industry = X` (ngành whitelist 18), count `change.w_pct > 0` / total mã ngành:

| Tỷ lệ mã tăng giá tuần | Đánh giá | Implication |
|---|---|---|
| ≥ 60% | **Dẫn dắt thật** | Conviction sector bias không giảm |
| 40-60% | **Phân hoá / rotation nội bộ** | Cần thêm context — nếu vĩ mô + catalyst rõ → vẫn quan tâm MID. Nếu không → trung tính |
| ≤ 30% | **Trụ kéo** (vài mã vốn hoá lớn) | Cảnh giác — conviction sector bias giảm 1 bậc (HIGH → MID, MID → LOW) |

### 2.3. Logic chọn ngành cần thận trọng

**Tiêu chí:**
- Industry rank bottom 1/3 (rank ≥ 13/18)
- Week_score âm rõ (≤ -5) + biến động giá tuần âm
- **HOẶC** xuất hiện ở sub-section 5.4 phần 5 với hướng tác động tiêu cực (Magnitude Material+ và Persistence Trending+)
- **HOẶC** ngành quá mua đa khung (P/E phân vị > 80% + biến động tháng cao bất thường so median 3Y)
- **HOẶC** có catalyst tiêu cực mới phần 8 (Conviction impact HIGH/MID)

Số lượng ngành cần thận trọng: 2-3 (flex theo bối cảnh).

### 2.4. Conviction sector bias mỗi ngành

**HIGH:** 3/4 tiêu chí thuận lợi + cơ bản strong (EPS Q YoY positive) + không có signal mâu thuẫn rõ.

**MID:** 2/4 tiêu chí thuận lợi + cơ bản OK + có 1 signal đang theo dõi.

**LOW:** 1/4 tiêu chí + early signal hoặc đường catalyst override (cơ bản chưa pass nhưng catalyst rất mạnh).

## 3. Methodology — Mapping vĩ mô ↔ 18 ngành whitelist

Bảng mapping cơ chế chỉ số vĩ mô / commodity → ngành VN trong scope whitelist 18. Pack sử dụng bảng này khi compose phần 5 (sub-section 5.4) và phần 6 (cross-check ngành).

| Chỉ số vĩ mô / Commodity | Ngành whitelist 18 nhạy chính | Hướng tác động |
|---|---|---|
| Lãi suất điều hành NHNN | NGANHANG, BDS, CHUNGKHOAN, BANLE | Lãi suất giảm: tích cực ngân hàng (NIM), BĐS (cầu vay), tiêu dùng (sức mua), chứng khoán (margin) |
| Lãi suất liên ngân hàng (2W/1M/3M) | NGANHANG, CHUNGKHOAN | Tăng: áp lực NIM ngắn hạn, thanh khoản chứng khoán giảm |
| Tăng trưởng tín dụng | NGANHANG, BDS, KCN | Tăng: tích cực toàn bộ — đặc biệt BĐS phụ thuộc tín dụng |
| Tỷ giá USD/VND | DETMAY, THUYSAN, CONGNGHE, NGANHANG, KHOANGSAN | USD/VND tăng: tích cực xuất khẩu (DETMAY, THUYSAN, CONGNGHE); tiêu cực ngân hàng (FX risk) + nhập khẩu nguyên liệu (HOACHAT, NONGNGHIEP) |
| EUR/USD, USD/CNY | DETMAY, THUYSAN, NGANHANG | Indirect — qua dòng vốn FII và commodity globally |
| DXY | Tất cả (qua FII flow) | DXY tăng: tiêu cực EM equity flow — FII có xu hướng bán ròng |
| Dầu Brent, WTI | DAUKHI, VANTAI, HOACHAT, TIENICH | Dầu tăng: tích cực DAUKHI upstream; tiêu cực VANTAI (chi phí), HOACHAT (nguyên liệu), TIENICH (chi phí điện) |
| Quặng sắt, than cốc, HRC | KIMLOAI, XAYDUNG | HRC tăng + quặng sắt giảm: tích cực biên gộp KIMLOAI; XAYDUNG chịu chi phí |
| Urea, phốt pho, kali | HOACHAT, NONGNGHIEP | Urea tăng: tích cực HOACHAT (biên ure); tiêu cực NONGNGHIEP (chi phí phân bón) |
| Khí tự nhiên | HOACHAT, TIENICH | Khí tăng: tiêu cực HOACHAT (chi phí), TIENICH (điện khí) |
| Vàng, Bạc | KHOANGSAN | Vàng tăng: tích cực — nhưng VN không có pure gold producer lớn |
| Cà phê, hồ tiêu, cao su, gạo XK | NONGNGHIEP, THUCPHAM | Tăng: tích cực NONGNGHIEP (giá bán); ảnh hưởng THUCPHAM tuỳ chuỗi giá trị |
| Đường, ngô, đậu tương, heo hơi, tôm thẻ | THUCPHAM, NONGNGHIEP, THUYSAN | Phụ thuộc chuỗi giá trị — vd ngô tăng: tiêu cực chăn nuôi heo (chi phí thức ăn) |
| FII net flow tuần/tháng | Tất cả 18 (qua tâm lý thị trường) | FII mua ròng kéo dài: tích cực — đặc biệt large-cap NGANHANG, BDS, CHUNGKHOAN |
| Lãi suất Fed (FFR) | Tất cả (qua DXY + EM flow) | Fed cắt: tích cực EM, tích cực VN equity |
| ECB / PBOC | THUYSAN, DETMAY (qua EU/TQ demand) | Tuỳ context |
| China stimulus / property crisis | KIMLOAI, KHOANGSAN, HOACHAT, BDS | China stimulus: tích cực commodity cycle → tích cực KIMLOAI, KHOANGSAN |

**Cách dùng bảng:**
1. Khi compose phần 5 sub-section 5.4 (tác động vĩ mô lên ngành), agent đọc bảng để xác định ngành whitelist 18 nào bị tác động khi 1 chỉ số có Magnitude Material+ / Persistence Trending+
2. Khi cross-check ngành phần 6 (đánh giá ngành dẫn dắt thật vs trụ kéo), agent dùng bảng để hiểu tại sao 1 ngành đang up/down

**Bảng không exhaustive** — agent có thể bổ sung mapping cụ thể cho 1 ngành trong tuần báo cáo dựa context tin tức + cơ bản. Nhưng không thay đổi default mapping nếu không có lý do rõ.

## 4. Technical-as-noise rule (cho watchlist + sector bias)

Rule pack-internal cho weekly broadcast context — hệ quả của triết lý fundamental-driven supremacy (master mục 1.1):

**Rule:** trong báo cáo weekly_overview, KHI technical signal duy nhất (không kèm signal vĩ mô/cơ bản/chính sách) xuất hiện như lý do để:
- Upgrade/downgrade sector bias
- Đưa mã mới vào watchlist
- Loại mã khỏi watchlist
- Thay đổi conviction regime

**→ KHÔNG đủ.** Agent phải tìm thêm signal vĩ mô/cơ bản/chính sách kèm theo. Nếu không có → giữ status quo, ghi technical signal là "minh hoạ phụ" trong phần 9.1 hoặc 9.2.

**Ngoại lệ duy nhất:** sub-section 10.2.4 Bucket entry (nếu user request) — bucket entry hợp pháp PTKT-driven vì là entry timing observation, không thay đổi conviction.

## 5. Self-audit checklist — 12 item (trước khi xuất file)

Chạy 12 câu trước khi present báo cáo. Vi phạm câu nào sửa rồi mới render.

### 5.1. Whitelist 18 ngành (Nguyên tắc 2)

1. **Mọi query / aggregate ngành** dùng 18 ngành whitelist (`K_agent_db_01` Section B)?
2. **Bảng 18 ngành phần 6** render đủ 18, không có ngành ngoài whitelist?
3. **Aggregate proxy thị trường phần 4** (week_score, day_score chuỗi 5 phiên) tính trên 18 ngành, KHÔNG 24?
4. Rank ngành đã **tự tổng hợp theo `week_score`** trong scope 18 ngành whitelist (DB không lưu `industry_rank` ngành-vs-ngành — xem `K_agent_db_01` mục "Xếp hạng ngành")?

### 5.2. Weight balance & cap technical (Nguyên tắc 1)

5. **Cap technical toàn báo cáo (trừ 9.1+9.2) ≤ 15%?** Đếm thủ công nội dung technical trong từng phần.
6. **Phần 9.3 trigger 3 kịch bản** — trigger primary là macro/fundamental/policy/catalyst? Technical chỉ confirmation phụ (≤ 30% nội dung 9.3)?

### 5.3. Conviction + Horizon + Disconfirming (Nguyên tắc 3)

7. **Regime call** có conviction HIGH/MID/LOW + 1-2 disconfirming signal cụ thể?
8. **Mỗi ngành sector bias** có conviction + 1 disconfirming signal?
9. **Mỗi mã watchlist phần 10.2 + 10.3** có conviction + horizon + 3 signal theo dõi + 1 disconfirming signal?

### 5.4. K hygiene + nguồn (chuẩn institutional)

10. **K hygiene:** ký hiệu DB raw đã dịch hết, không còn `week_score: 18`, `industry_rank_pct: 0.9`, `vsi: 2.1` raw trong output? Số liệu định lượng đã quy đổi đơn vị (BCTC: tỷ đồng, % thập phân nhân 100)?
11. **Mỗi claim định lượng có nguồn** truy được: collection + field, hoặc URL web search? Tin có dẫn link `https://finext.vn/news/<slug>` hoặc URL gốc?

### 5.5. Checkpoint + structural

12. **Checkpoint 1** đã có user phản hồi (confirm/override) trước khi compose phần 10-12? Override (nếu có) đã ghi inline note + log trong metadata User overlay log? **Phần 1 viết cuối cùng** sau khi có đủ 11 phần còn lại, dùng cấu trúc Key calls / Watch / Risk?

## 6. Edge cases

### 6.1. Thiếu dữ liệu DB

- `market_recent` query rỗng tuần → báo user, hỏi có muốn dùng phiên cuối tuần trước đó không
- `industry_snapshot` thiếu 1-2 ngành whitelist 18 → ghi note "ngành X chưa có dữ liệu phiên cuối tuần", tiếp tục với 16-17 ngành còn lại
- `stock_nntd` rỗng → query `data_briefing` block NN/TD thay thế
- `news_history_feed` rỗng tuần → tăng tỷ trọng web search cho phần 8
- `industry_finstats` thiếu valuation_ratios 1 ngành → ghi note + skip cột P/E phân vị cho ngành đó

### 6.2. Conflict regime call

Combo 4 input cho ra 2 regime tương đương khả thi:
- Agent xuất 2 candidate ở checkpoint, không chọn 1
- Block format: "Call có 2 candidate khả thi: [A] vs [B]. Lý do A: ... Lý do B: ... Cần user quyết."

### 6.3. Tuần "không có gì đặc biệt"

- **Vĩ mô:** không có biến động vĩ mô đáng kể → bỏ sub-section 5.4 (không render bảng rỗng, không ghi "không có gì đặc biệt")
- **Themes:** không có theme dominant → ghi explicit "Tuần này không có theme chi phối rõ" trong phần 8 hoặc phần 9.4 risk map
- **Earnings beat candidate:** không có ngành nào pass 3 tiêu chí phần 6.x → ghi "Tuần này không có earnings beat candidate rõ ràng"

### 6.4. Tuần đầu cycle (không có file W-1)

Skip phần 2, ghi 1 dòng "Tuần đầu cycle, chưa có dữ liệu review". Workflow chạy bình thường từ phần 3.

### 6.5. User không phản hồi checkpoint

Sau khi xuất block Checkpoint 1, agent dừng. Không tự chuyển Stage 2. Đợi user phản hồi trong session sau cũng được — pack không có timeout.

### 6.6. Tuần thị trường yếu toàn diện

Nếu top 10 dòng tiền phần 7 chứa mã có điểm dòng tiền tuần âm hoặc bằng 0:
- Vẫn render bảng đầy đủ 10 mã
- Ghi note honest: "Tuần thị trường yếu toàn diện, danh sách top 10 dòng tiền có mã điểm dòng tiền âm hoặc bằng 0 — đây là 'ít yếu nhất' chứ không phải dẫn dắt thực sự."
- Phần 7.x cảnh báo trap setup vẫn render nếu detect được

### 6.7. Ngành whitelist 18 thay đổi giữa cycle

User thêm/bỏ ngành trong whitelist: áp dụng từ tuần kế tiếp, không retroactive trong báo cáo hiện tại. Ghi rõ trong metadata "whitelist update [DD/MM]".

## 7. Output contract

Pack sinh structured content cho `O_weekly_overview_00` render. Ràng buộc bắt buộc:

- **12 phần đầy đủ** theo thứ tự: 1 Tóm tắt điều hành / 2 Review tuần trước (scorecard) / 3 Bối cảnh quốc tế / 4 Thị trường Việt Nam (aggregate 18 whitelist) / 5 Vĩ mô & hàng hoá (institutional table 5 cột) / 6 Biến động 18 ngành whitelist / 7 Top dẫn dắt + cảnh báo trap / 8 Tin tức & catalyst + conviction impact / 9 Định vị VNINDEX + 3 kịch bản fundamental-driven + Risk map / 10 Watchlist (tách 2 hướng + conviction + horizon + disconfirming) / 11 Lịch sự kiện + conviction impact / 12 Tuyên bố miễn trừ trách nhiệm
- **Phần rỗng vẫn render** với 1 dòng note (vd phần 2 nếu không có W-1: "Tuần đầu cycle, chưa có dữ liệu review")
- **Whitelist 18 ngành:** mọi query/aggregate/bảng tuân Nguyên tắc 2 master
- **Weight balance:** trigger 3 kịch bản primary là macro/fundamental/policy/catalyst (Nguyên tắc 1); cap technical toàn báo cáo ≤ 15%
- **Conviction + Horizon + Disconfirming:** mỗi call có đầy đủ 3 thành phần (Nguyên tắc 3)
- **Số liệu đã quy đổi** đơn vị theo `K_agent_db_00` mục 6
- **Ký hiệu DB raw đã dịch** theo `K_agent_db_00` mục 5.2
- **Thuật ngữ tiếng Anh đã dịch** theo `K_agent_db_05` mục 9
- **Mỗi claim có nguồn** truy được
- **Tin có dẫn link** finext.vn hoặc URL gốc
- **Checkpoint 1** đã có user phản hồi trước khi vào Stage 2 (Nguyên tắc 4)
- **Phần 1 viết cuối** với cấu trúc Key calls / Watch / Risk
- **User overlay log** trong metadata (nếu có user inject view)
- **Disclaimer** render đúng theo branding info (3 mode: custom / default branded / plain)

Pack KHÔNG tự quyết heading style / xưng hô / tone / length cuối — `O_weekly_overview_00` quyết.
