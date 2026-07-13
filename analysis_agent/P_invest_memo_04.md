# P_invest_memo_04 — Tier 3: Chấm điểm + chọn top 3/ngành

Giai đoạn 4 của quy trình. Từ shortlist 6-10 mã/ngành ở tier 2, chấm điểm 6 tiêu chí (max 18 điểm) và chọn top 3 mã/ngành cho tier 5 (memo deep-dive). Đây là **gate quan trọng trước memo** — sau tier này mới đầu tư resource lớn vào PDF extraction và modeling.

Reference: `P_invest_memo_00` phần Flow chi tiết (overview), `P_invest_memo_00` phần Cơ chế checkpoint review, `P_invest_memo_03` (tier 2 output làm input), pack `K_agent_db` (`K_agent_db_01` schema, `K_agent_db_04` interpretation).

---

## 1. Mục tiêu & output expected

**Mục tiêu:** từ shortlist 6-10 mã/ngành (tier 2) xuống 3 mã/ngành qua:
- Chấm điểm 6 tiêu chí, mỗi tiêu chí 1-3 điểm, max 18 điểm
- Xếp hạng + chọn top 3/ngành
- Cân bằng bucket entry (không phải 3 mã cao điểm nhất — cần đa dạng bucket)
- Phân vào 3 tier conviction: high / medium / low

**Input:**

1. File output tier 2 (`tier2_YYYYMMDD_confirmed.md`) — shortlist 6-10 mã/ngành + bucket + flags kỹ thuật
2. Bảng benchmark ngành từ tier 2 (industry_finstats đã lọc) — dùng cho tiêu chí vị thế ngành
3. Thông tin catalyst mã đã quét ở tier 2

**Output chính:**

1. Cho mỗi ngành: top 3 mã với điểm tổng + tier conviction + bucket
2. Bảng chi tiết điểm từng tiêu chí cho 3 mã top/ngành
3. Các mã gần top nhưng không lọt (sát nút)
4. Mã bị loại bắt buộc (hard reject) và lý do
5. Report checkpoint 4 theo khung 6 phần

**Tần suất:** sau khi tier 2 được confirm.

**Thời gian session:** 30-45 phút cho 3-5 ngành. User review checkpoint 20-30 phút. Đây là checkpoint quan trọng nhất trước memo deep-dive.

---

## 2. Triết lý cơ bản: ranking tool, không phải filter

**Vai trò bảng chấm:** chỉ là công cụ **xếp hạng ưu tiên**. Tất cả mã trong shortlist tier 2 đã lọt universe filter (fundamental + liquidity + catalyst). Bảng chấm phân biệt mã "tốt" với mã "tốt hơn" để chọn top 3/ngành.

**Không dùng bảng chấm để loại mã:** nếu 1 mã có điểm thấp nhưng lọt shortlist tier 2, nó vẫn có chất lượng tối thiểu đã được verify. Loại mã khỏi shortlist phải dựa trên **tiêu chí loại bắt buộc** (Section 6) hoặc user override có audit.

**6 tiêu chí, 18 điểm max:** thiết kế để distribute vừa phải — mã trung bình 9-11đ, mã tốt 12-14đ, mã xuất sắc 15-18đ. Nếu phần lớn shortlist ≥ 15đ → tiêu chí đang quá rộng, cần siết lại. Nếu phần lớn < 10đ → tiêu chí quá khắt khe hoặc universe tier 2 chất lượng thấp.

---

## 3. Sáu tiêu chí chấm điểm

Mỗi tiêu chí 1-3 điểm. Thang điểm:
- **3 điểm** — vượt trội rõ ràng so với peer cùng ngành
- **2 điểm** — trên trung vị / đủ tốt
- **1 điểm** — đạt yêu cầu tối thiểu (vì đã pass universe filter)

### Tiêu chí 1 — Vị thế trong ngành (fundamental)

**Mục đích:** đo năng lực cạnh tranh thực sự của mã so với peer cùng ngành — ROE, biên, vị trí trong chuỗi giá trị.

**Chấm điểm:**

| Điểm | Tiêu chí SXKD | Tiêu chí NGANHANG | Tiêu chí CK | Tiêu chí BH |
|---|---|---|---|---|
| 3 | ROE > 150% median ngành + biên mở rộng 4 quý + top 20% market share | ROE > 150% median + CASA > 30% + NPL < 1% | Top 3 thị phần môi giới + ROE > 150% median | ROE > 150% median + Revenue Growth ≥ 15% |
| 2 | ROE ≥ median ngành + biên ổn định | ROE ≥ median + NPL ≤ 2% | Top 10 thị phần + ROE ≥ median | ROE ≥ median + Revenue Growth ≥ 0 |
| 1 | ROE dưới median nhưng đã pass universe | ROE dưới median | Ngoài top 10 thị phần | ROE dưới median nhưng pass |

**Giải thích viết tắt trong bảng:**
- CASA (Current Account Savings Account — tỷ lệ tiền gửi không kỳ hạn và tiết kiệm trên tổng huy động): bank có CASA cao = cost of fund thấp, NIM rộng, biên lợi nhuận bền vững
- Market share: thị phần doanh thu/sản lượng của mã trong ngành

**Data source:** stock_finstats (đã có từ tier 2) vs industry_finstats benchmark. Market share cho SXKD/CK lấy qua web search (báo cáo ngành FiinGroup, VinaCapital, hoặc báo cáo thường niên).

### Tiêu chí 2 — Quản trị & pháp lý (web search + fundamental)

**Mục đích:** phát hiện red flag quản trị — scandal gần, vi phạm UBCK (Ủy ban Chứng khoán Nhà nước), lãnh đạo bán ra bất thường, insider trading (giao dịch nội bộ dùng thông tin chưa công bố), audit qualified opinion (ý kiến kiểm toán ngoại trừ — có hạng mục kiểm toán không đồng ý với báo cáo).

**Chấm điểm:**

| Điểm | Tiêu chí |
|---|---|
| 3 | Quản trị sạch + track record minh bạch + lãnh đạo có credibility cao + báo cáo thường niên đạt chuẩn quốc tế |
| 2 | Không có red flag gần đây + báo cáo tuân thủ đầy đủ |
| 1 | Có red flag nhẹ (ĐHCĐ không đủ quorum, công bố chậm, 1 vụ phạt UBCK quy mô nhỏ) |
| Loại | Scandal lớn chưa resolve, qualified audit opinion, nghi ngờ gian lận BCTC |

**Quy trình check (bắt buộc):**

1. Web search: `<ticker> vi phạm UBCK 2025 2026`
2. Web search: `<ticker> bị phạt công bố thông tin`
3. Web search: `<ticker> lãnh đạo bán ra cổ phiếu 2026` (chỉ check 6 tháng gần)
4. Kiểm tra báo cáo soát xét (nếu có PDF upload) — ý kiến kiểm toán loại gì
5. DB: news_history_feed với ticker trong 90 ngày — quét tin tiêu cực

**Lưu ý:** tiêu chí này cần 15-20 phút/mã cho web search. Nếu ngành có 8-10 mã shortlist, có thể ưu tiên check top 5 điểm các tiêu chí khác trước — mã nào không vào top 5 thì check nhanh hơn.

### Tiêu chí 3 — Thanh khoản (fundamental-technical bridge)

**Mục đích:** mã có thanh khoản đủ lớn để vào/ra không slippage, phù hợp size position mục tiêu. Đây là bridge giữa fundamental (quy mô, free float — tỷ lệ cổ phiếu tự do giao dịch sau khi trừ các khoản nắm giữ dài hạn của cổ đông lớn, nội bộ, nhà nước) và technical (trading activity).

**Chấm điểm:**

| Điểm | Tiêu chí |
|---|---|
| 3 | Trading value trung bình 20 phiên ≥ 50 tỷ/phiên + LargeCaps + free float ≥ 30% |
| 2 | Trading value 20-50 tỷ/phiên + MidCaps + free float ≥ 20% |
| 1 | Trading value 5-20 tỷ/phiên (đã pass tier 2) + SmallCaps hoặc free float thấp |

**Phân loại quy mô vốn hoá (VN convention):** LargeCaps vốn hoá > 10,000 tỷ / MidCaps 3,000-10,000 tỷ / SmallCaps 1,000-3,000 tỷ / MicroCaps < 1,000 tỷ.

**Data source:**
- Trading value trung bình: tính từ stock_recent 20 phiên (đã có từ tier 2)
- marketcap, free_float_pct: từ stock_info (v2 đổi tên từ freeFloatRate, giá trị là điểm % — 45 = 45%)

**Tác động bucket xuống tier 5:**

Mã có Tiêu chí 3 = 1đ (thanh khoản thấp) sẽ vướng constraint 5% ADV khi sizing ở tier 6. Ví dụ mã 5-10 tỷ/phiên: per-phiên cap 5% ADV = 250-500 triệu; với N=3 phiên build position → max tổng vị thế = 750M-1.5 tỷ (~3-6% portfolio 1 triệu USD). Conviction High (target 6-8%) có thể không đủ size — flag user biết trước. Công thức đầy đủ ở `P_invest_memo_08` Section 3.4. Agent **flag rõ** điều này trong checkpoint để user biết trước.

### Tiêu chí 4 — Catalyst cá thể (catalyst)

**Mục đích:** đo cường độ và timing của catalyst cá thể (loại 4 từ `P_invest_memo_01`) — KQKD, M&A, thoái vốn, hợp đồng, tăng room.

**Chấm điểm:**

| Điểm | Tiêu chí |
|---|---|
| 3 | Catalyst mạnh đã xác nhận + timing trong 0-3 tháng + magnitude ước lượng ≥ 15% tác động EPS/định giá + chưa priced-in hoàn toàn |
| 2 | Catalyst khả năng cao + timing 3-6 tháng + magnitude trung bình (5-15%), hoặc có catalyst 3đ nhưng đã priced-in một phần |
| 1 | Catalyst yếu hoặc xa (> 6 tháng), hoặc magnitude nhỏ, hoặc chưa confirmed |

**Data source:**
- Bảng catalyst mã từ tier 2 (đã quét)
- Bảng catalyst vĩ mô/ngành từ tier 0 (ảnh hưởng cá thể)
- Web search bổ sung nếu cần: `<ticker> KQKD Q1 2026`, `<ticker> M&A`, `<ticker> ĐHCĐ 2026`

**Lưu ý về catalyst play (mã qua đường C tier 2):**

Mã vào shortlist qua đường C (fail B) chắc chắn có catalyst mạnh — mặc định Tiêu chí 4 ≥ 2đ, thường là 3đ. Nhưng Tiêu chí 1 (vị thế ngành) có thể thấp vì fundamental yếu. Kết quả thường: catalyst play có tổng điểm tập trung ở Tiêu chí 4-6, thấp ở Tiêu chí 1-3.

### Tiêu chí 5 — Dòng tiền NN/TD 1 tháng (kỹ thuật — flow)

**Mục đích:** xác nhận smart money (khối ngoại) và tự doanh đang tích luỹ mã. Đây là **tiebreaker** quan trọng, không phải điểm chính.

**Chấm điểm:**

| Điểm | Tiêu chí |
|---|---|
| 3 | NN mua ròng rõ rệt 1 tháng + có xu hướng tăng mua qua các tuần + còn room |
| 2 | NN mua ròng nhẹ hoặc trung tính với xu hướng cải thiện, HOẶC TD tích luỹ mạnh |
| 1 | NN trung tính không rõ xu hướng, hoặc bán ròng nhẹ |
| 0 (rare) | NN bán ròng mạnh kéo dài — xem xét loại mã, không chỉ chấm 0đ |

**Nguyên tắc đọc (khớp nguyên tắc tier 0 Input 4):**

- NN là **tham khảo**, không phải yếu tố quyết định — không mã nào NN cũng mua
- Mã không có NN giao dịch → chấm 1đ (không có signal), không loại
- Xu hướng quan trọng hơn giá trị tuyệt đối: tuần gần nhất tăng mua so với các tuần trước = tín hiệu tốt hơn là giá trị tháng cao

**Data source:** stock_nntd (đã có tier 2). Lưu ý v2: mã không có giao dịch NN/TD thì block `nn`/`td` bị omit hẳn khỏi doc — hiểu là "không có dữ liệu", chấm 1đ theo rule trên, KHÔNG phải "mua ròng 0". Check thêm `foreignerRoom` từ stock_info — mã cạn room thì NN mua ít ý nghĩa hơn.

### Tiêu chí 6 — Đồng đều điểm dòng tiền 5 phiên (kỹ thuật — flow)

**Mục đích:** xác nhận dòng tiền nội bộ đến mã **đồng đều**, không phải spike 1 phiên. Đây là **tiebreaker** thứ 2 cùng Tiêu chí 5.

**Chấm điểm:**

| Điểm | Tiêu chí |
|---|---|
| 3 | Day_score dương trong ≥ 4/5 phiên gần nhất + week_score ≥ 10 (top 25% thị trường) |
| 2 | Day_score dương trong 3/5 phiên + week_score ≥ 0 |
| 1 | Day_score dương trong ≤ 2/5 phiên hoặc tập trung vào 1-2 phiên spike |

**Data source:** stock_recent (đã có tier 2) — đọc field `series[].money_flow_score.day_score` cho 5 phiên mới nhất. week_score từ stock_snapshot.

**Vai trò tiebreaker:** khi 2 mã có tổng điểm từ Tiêu chí 1-4 bằng nhau (ví dụ cả 2 đều 10đ), dùng tổng Tiêu chí 5+6 để phân thứ tự. Mã có Tiêu chí 5+6 = 5đ xếp trên mã có 3đ. Không đặt trọng số riêng cho Tiêu chí 5-6 — chúng có điểm như các tiêu chí khác nhưng được ưu tiên khi có tranh chấp.

---

## 4. Cân bằng bucket khi chọn top 3

Top 3 mã/ngành **không nhất thiết là 3 mã cao điểm nhất**. Cần phân bổ đa bucket để có linh hoạt entry timing ở tier 6.

### Nguyên tắc cân bằng

**Target distribution** cho top 3 mã/ngành:

| Regime | Target distribution |
|---|---|
| Risk-on full | 1 Bucket 1 + 1-2 Bucket 2 + 0-1 Bucket 3 |
| Risk-on selective | 1 Bucket 1 + 2 Bucket 2 + 0 Bucket 3 |
| Defensive only | 0-1 Bucket 1 + 2 Bucket 2 + 1 Bucket 3 |

Nguyên tắc: ít nhất 1 mã Bucket 2 trong top 3 (pullback entry = phù hợp horizon 3-6 tháng nhất). Tránh all-in Bucket 1 — nếu thị trường rơi ngay sau khi vào, toàn bộ position dính downdraft.

### Khi bảng chấm cho 3 mã top toàn Bucket 1

Tình huống: 3 mã cao điểm nhất trong ngành đều ở Bucket 1 (tuần+tháng đồng thuận A+, week_score ≥ 6).

**Xử lý:** chọn 2 mã cao điểm Bucket 1, thay mã Bucket 1 thứ 3 bằng mã cao điểm nhất ở Bucket 2. Điểm của mã Bucket 2 có thể thấp hơn 1-3 điểm — vẫn chọn để có diversify bucket.

**Exception:** nếu mã cao điểm nhất Bucket 2 có điểm thấp hơn mã Bucket 1 thứ 3 quá 4 điểm → vẫn lấy mã Bucket 1 (chất lượng quá cách biệt). Flag rõ trong checkpoint: "Ngành này không có mã Bucket 2 chất lượng, shortlist top 3 đều Bucket 1 — tăng rủi ro timing".

### Khi bảng chấm cho 3 mã top toàn Bucket 2

Tình huống phổ biến hơn — Bucket 2 (pullback) thường có nhiều mã chất lượng tốt.

**Xử lý:** chọn 3 mã Bucket 2 chấp nhận được nếu:
- Điểm tổng các mã đều ≥ 11 (medium conviction trở lên)
- Ngành có flag "đang rơi từ đỉnh" hoặc "dao động biên độ lớn" từ tier 1 (không nên Bucket 1)

**Flag:** user biết shortlist ngành này toàn Bucket 2 = tier 6 sizing sẽ tiêu thụ ít cash ngay (30-50% size), giữ cash buffer cao hơn dự kiến.

### Khi bảng chấm có mã Bucket 3 cao điểm

Tình huống: mã Bucket 3 (zone w+m đều C) có điểm tổng ≥ 13 — mã chất lượng fundamental + catalyst cao, chỉ ngắn hạn rơi sâu.

**Xử lý:** cân nhắc đưa vào top 3 nếu:
- Mã pass B (không phải catalyst play fail B)
- Regime không phải Defensive only (Defensive thì hạn chế Bucket 3)
- Đã có ≥ 1 mã Bucket 1 hoặc Bucket 2 trong top 3

Mã Bucket 3 trong top 3 = mã chất lượng cao đang chờ tín hiệu bật tuần. Tier 5-6 sẽ **không vào ngay** — chỉ theo dõi, khi zone tuần chuyển B+ và week_score dương thì chuyển Bucket 2 và bắt đầu vào 30-50% size.

### Flag sếp cho tier 5

Trong output tier 3, ghi rõ distribution bucket của top 3 mỗi ngành để tier 5/6:
- Biết trước khối lượng cash sẽ tiêu thụ trong lần vào đầu tiên
- Chuẩn bị plan theo dõi mã Bucket 2 (đợi confirm) và Bucket 3 (watchlist)
- Điều chỉnh cash buffer nếu phần lớn shortlist là Bucket 2-3

---

## 5. Phân tier conviction

Sau khi chốt top 3/ngành, phân mỗi mã vào 1 trong 3 tier conviction dựa trên tổng điểm. Tier conviction quyết định size position ở tier 6.

| Tier | Điểm tổng | Size target (% portfolio) | Ý nghĩa |
|---|---|---|---|
| High | 15-18 | 6-8% | Mã có thesis mạnh đa chiều, conviction cao, đáng đầu tư lớn |
| Medium | 11-14 | 3-5% | Mã tốt với 1-2 điểm mạnh, conviction trung bình, size vừa |
| Low | 8-10 | 1-2% | Mã pass universe nhưng không xuất sắc, size nhỏ để có exposure |

**Nguyên tắc:**

- Size target là **guideline**; điều chỉnh cuối ở tier 5/6 dựa trên memo (tier 5C) + modeling (tier 5B) + ADV constraint tier 6 (xem `P_invest_memo_08` Section 3.4 cho công thức `5% × ADV × N` với N=2-4 phiên build position)
- Điểm 8-10 vẫn vào shortlist final vì đã pass universe — chỉ size nhỏ, không loại
- Điểm dưới 8 là hiếm (vì shortlist tier 2 đã filter chất lượng). Nếu có, flag bất thường trong checkpoint

### Điều kiện distribution conviction

**Khuyến nghị cho shortlist final** (3-5 ngành × 3 mã = 9-15 mã):

- Ít nhất 1-2 mã High (15-18) trong toàn portfolio
- Majority (5-10 mã) là Medium (11-14)
- Có thể có 1-3 mã Low (8-10) cho diversification

Nếu toàn portfolio đều Medium/Low (không có High) → conviction tổng thấp, cân nhắc tăng cash buffer để chờ cơ hội tốt hơn. Nếu > 50% là High → có thể đang overconfident, review lại bảng chấm có cho điểm quá rộng không.

---

## 6. Tiêu chí loại bắt buộc (hard reject)

Ngoài bảng chấm, một số điều kiện dẫn đến **loại thẳng** mã khỏi shortlist, dù điểm cao:

### R1. rank_pct = 0

Mã có market_rank_pct = 0 (thanh khoản không đủ để xếp hạng hệ thống) — đã loại ở tier 2 về nguyên tắc. Nếu do lỗi data tier 2 để lọt, loại ngay.

### R2. Sanity check trading value trung bình 20 phiên

Verify tier 2 D1 dùng đúng "trung bình 20 phiên ≥ 5 tỷ" (không phải snapshot 1-2 phiên). Nếu data tier 2 dùng phiên gần đây có spike, re-check với 20-phiên average mới — nếu < 5 tỷ → loại consistent với D1 spec.

### R3. Scandal lớn chưa resolve

Phát hiện ở Tiêu chí 2 (web search):
- Vi phạm nghiêm trọng UBCK (đình chỉ giao dịch, phạt lớn)
- Điều tra hình sự lãnh đạo
- Nghi ngờ gian lận BCTC được báo chí chính thống đưa tin
- Qualified audit opinion trong kỳ gần nhất

Agent **flag CẢNH BÁO MẠNH + downgrade conviction 2 bậc** (High → Low / Medium → Watch / Low → loại). User quyết định cuối: proceed với size rất nhỏ (≤ 1% portfolio) + audit log nêu lý do mạnh, hoặc loại mã. Triết lý flex+downgrade consistent với master mục 5 nguyên tắc 1, 4 — không strict reject vì governance issue có thể đã priced-in một phần, user là người evaluate cuối.

### R4. Cảnh báo dòng tiền + catalyst tiêu cực (Nguyên tắc 5 của `P_invest_memo_00`)

Mã có:
- Điểm dòng tiền tuần đang dương (week_score ≥ 10)
- Nhưng có catalyst **tiêu cực** mới xuất hiện (tin xấu chính thức, chính sách siết chặt, giá đầu vào tăng cấu trúc)

→ Dòng tiền đang priced-in điều gì chưa lộ, hoặc late money sẽ kẹt. Loại thẳng theo Nguyên tắc 5. Không override.

### R5. Chi tiết BCTC không khớp với tier 2

Rà lại BCTC kỳ gần nhất ở tier 3 (có thể tier 2 dùng data cũ), phát hiện số liệu thực tế xấu hơn nhiều (ví dụ CFO âm sâu, giao dịch bên liên quan bất thường lớn).

Agent **flag CẢNH BÁO + downgrade conviction 1-2 bậc** (mức độ tùy chênh lệch số liệu). User quyết định: chờ tier 5A deep-dive xác nhận trước khi proceed, hoặc loại mã. Triết lý flex+downgrade consistent — số liệu khác biệt có thể do lỗi data hoặc xấu thật, tier 5A confirm root cause trước khi quyết định.

---

## 7. Workflow đầy đủ — 7 bước

**Bước 1 — Load input tier 2**

Đọc file `tier2_YYYYMMDD_confirmed.md`. Extract:
- Shortlist 6-10 mã/ngành (3-5 ngành)
- Bucket đã phân
- Flags kỹ thuật từng mã
- Bảng benchmark ngành

**Bước 2 — Rà lại universe filter (quick check)**

Quick verify các mã trong shortlist có còn đạt tier 2 criteria không (data có thể cập nhật từ lúc tier 2 chạy):
- Check snapshot_date mới nhất
- Spot check 2-3 mã ngẫu nhiên/ngành

Nếu có mismatch → flag và report trong checkpoint.

**Bước 3 — Chấm Tiêu chí 1, 3, 4, 5, 6 (data đã có)**

5 tiêu chí này dùng data đã có từ tier 2:
- Tiêu chí 1: stock_finstats vs industry_finstats
- Tiêu chí 3: trading value trung bình + marketcap + free_float_pct
- Tiêu chí 4: catalyst map từ tier 2
- Tiêu chí 5: stock_nntd
- Tiêu chí 6: stock_recent day_score 5 phiên + week_score

Chạy được song song cho toàn bộ shortlist.

**Bước 4 — Chấm Tiêu chí 2 (web search)**

Tốn thời gian nhất. Với mỗi mã, chạy 3-5 web query + check news DB 90 ngày.

Tối ưu hoá: chấm Tiêu chí 1, 3, 4, 5, 6 trước, sort mã theo tổng 5 tiêu chí. Bắt đầu web search Tiêu chí 2 từ top xuống. Với mã top 5/ngành, check kỹ 10-15 phút. Với mã ngoài top 5, check nhanh 3-5 phút (chỉ quét scandal lớn).

**Bước 5 — Kiểm tra điều kiện loại bắt buộc**

Sau khi có điểm 6 tiêu chí, scan qua R1-R5. Mã nào rơi vào hard reject → loại, flag trong checkpoint.

**Bước 6 — Chọn top 3/ngành với cân bằng bucket**

Apply logic Section 4:
- Sort mã theo tổng điểm trong ngành
- Kiểm tra distribution bucket của top 3
- Điều chỉnh nếu toàn Bucket 1 hoặc thiếu diversify
- Phân tier conviction (High/Medium/Low)

**Bước 7 — Xuất checkpoint 4**

Theo template Section 8.

---

## 8. Template báo cáo Checkpoint 4

Theo khung 6 phần `P_invest_memo_00` phần Cơ chế checkpoint review.

```
# Checkpoint 4 — Top 3/ngành sau chấm điểm [ngày]

## 1. Summary quyết định
Shortlist final [tổng số] mã qua [N] ngành:
- Ngành A: [mã a1 High 16đ], [mã a2 Medium 13đ Bucket 2], [mã a3 Medium 12đ Bucket 2]
- Ngành B: [mã b1 High 15đ], [mã b2 Medium 12đ], [mã b3 Low 10đ Bucket 3]
...

Distribution conviction toàn portfolio: [X High, Y Medium, Z Low].
Distribution bucket: [X Bucket 1, Y Bucket 2, Z Bucket 3].
Mã bị loại hard reject: [số] — chi tiết Phần 5.

## 2. Bối cảnh đầu vào
- Tier 2 confirmed: [ngày]
- Tổng mã input: [X] từ [N] ngành
- Sau chấm điểm + loại bắt buộc: [Y] mã top 3 final
- Ngày snapshot DB kiểm tra lại: [snapshot_date]
- Cảnh báo độ tươi: [nếu có mismatch giữa tier 2 data và current]

## 3. Quyết định + lý do (từng ngành)

### Ngành [Tên] — 3 mã top

**Mã a1 — [tên] — Tổng 16đ — Tier High — Bucket 1**

| Tiêu chí | Điểm | Lý do |
|---|---|---|
| 1. Vị thế ngành | 3 | ROE 28% (175% median ngành), biên mở rộng 4Q, top 10% thị phần |
| 2. Quản trị | 3 | Không red flag, BCTC minh bạch, ĐHCĐ tuân thủ |
| 3. Thanh khoản | 3 | Trading value TB 120 tỷ/phiên, LargeCap, free float 35% |
| 4. Catalyst | 2 | KQKD Q1 công bố trong 3 tuần, consensus +12% LNST |
| 5. NN/TD 1T | 3 | NN mua ròng mạnh kéo dài 4 tuần, xu hướng tăng, còn room |
| 6. Dòng tiền 5P | 2 | Day_score dương 3/5 phiên, week_score 18 |

Tier High → size target 6-8% portfolio. Bucket 1 → vào ngay 50-70%.

**Mã a2 — [tên] — Tổng 13đ — Tier Medium — Bucket 2**

(bảng tương tự, ít chi tiết hơn)

**Mã a3 — [tên] — Tổng 12đ — Tier Medium — Bucket 2**

(bảng tương tự)

Lý do phân bổ bucket:
- Ngành này có 3 mã cao điểm nhất đều ở Bucket 1
- Chọn 1 Bucket 1 + 2 Bucket 2 thay vì 3 Bucket 1 để đa dạng entry timing
- Điểm chênh giữa mã Bucket 1 thứ 3 (14đ) và mã Bucket 2 top (13đ) chỉ 1đ → chấp nhận được

(Lặp cho từng ngành trong shortlist)

## 4. Số liệu kỹ thuật key

Bảng tổng hợp shortlist final:
| Ticker | Ngành | Tier | Tổng đ | Bucket | T1 | T2 | T3 | T4 | T5 | T6 |
|---|---|---|---|---|---|---|---|---|---|---|
| a1 | A | High | 16 | 1 | 3 | 3 | 3 | 2 | 3 | 2 |
| a2 | A | Medium | 13 | 2 | 2 | 3 | 2 | 2 | 2 | 2 |
| ... |

## 5. Mã bị loại bắt buộc

- Mã X (ngành A, tier 2 rank 3): loại R3 — phát hiện vi phạm UBCK công bố tuần trước về công bố chậm BCTC + đang bị xử phạt
- Mã Y (ngành B, tier 2 rank 5): loại R4 — dòng tiền tuần dương +15 nhưng tin ngành công bố Luật mới siết, priced-in tiêu cực → bẫy
- Mã Z (ngành D, tier 2 rank 2): loại R5 — rà lại BCTC Q4/2025 thấy CFO âm lớn khác với profile tier 2 dùng data cũ

## 6. Lựa chọn sát nút

### Mã điểm cao nhưng không vào top 3 do cân bằng bucket:
- Ngành A — mã a4 (14đ, Bucket 1): không lọt top 3 vì đã có 1 mã Bucket 1 và cần diversify Bucket 2
- Nếu user muốn tập trung Bucket 1 → có thể swap mã a3 (12đ Bucket 2) bằng a4

### Mã điểm sát biên tier conviction:
- Ngành B — mã b3 (Low 10đ): chỉ hơn ngưỡng hard reject 2đ. Nếu user thấy conviction thực tế thấp hơn → có thể bỏ, giảm shortlist ngành B xuống 2 mã

### Mã hard reject user có thể override với audit:
- Mã X (R3 scandal) — nếu user có thông tin nội bộ rằng vi phạm đã/đang được giải quyết → override với audit
- Các mã khác: R4, R5 khó override vì là cảnh báo system-level

## 7. Flags quan trọng chuyển sang tier 5

Cho mỗi mã top 3 final:

- Mã a1 (High): ưu tiên deep-dive PDF + modeling ở tier 5A-5B. Memo chất lượng cao.
- Mã a2, a3 (Medium Bucket 2): tier 5A check red flag cơ bản, tier 5B có thể đơn giản hơn (peer multiples nhanh thay vì DCF đầy đủ). Memo đầy đủ 7 phần.
- Mã b3 (Low Bucket 3): tier 5A check nhanh, tier 5B chỉ peer multiples. Memo gọn 3-4 phần — nếu memo không ra conviction đủ, loại khỏi shortlist final trước khi vào position.

Thanh khoản cảnh báo:
- Mã [X] trading value TB 7 tỷ/phiên — per-phiên cap 350 triệu (5% ADV); với N=3 phiên build → max tổng vị thế ~1 tỷ (≈4% portfolio 1M USD). Conviction High target 6-8% sẽ vướng constraint, size thực tế giảm ~4%. Công thức đầy đủ `P_invest_memo_08` Section 3.4.

## 8. Câu hỏi chờ user

Xác nhận shortlist final [tổng số] mã top 3 × [N] ngành sang tier 5 (memo deep-dive)?
Hoặc muốn:
- (a) Override cân bằng bucket — ví dụ ngành A chọn 3 Bucket 1 thay vì 1+2
- (b) Override hard reject [mã cụ thể] — ghi audit log
- (c) Swap mã top 3 bằng mã sát nút [cụ thể]
- (d) Loại 1 ngành hoàn toàn khỏi shortlist final (quota cash tăng) nếu chất lượng top 3 không đủ
- (e) Request re-score 1 mã — Agent chấm lại với thông tin user cung cấp thêm

Tier 5 yêu cầu resource lớn — sau khi confirm, Agent sẽ yêu cầu user upload 3 PDF/mã (BCTC soát xét, BCTC năm, BCTN) cho tier 5A.

Nếu confirm → end session, lưu output tier 3.
Nếu override → ghi audit log rồi revise.
```

**Độ dài target:** 4-5 trang tuỳ số ngành và chi tiết bảng chấm.

---

## 9. Ví dụ generic

Case: tier 2 đã đưa 19 mã qua 3 ngành (A: 8 mã, B: 6 mã, D: 5 mã).

### Ngành A — 8 mã shortlist

Chấm điểm 6 tiêu chí:

| Mã | T1 | T2 | T3 | T4 | T5 | T6 | Tổng | Bucket | Ghi chú |
|---|---|---|---|---|---|---|---|---|---|
| a1 | 3 | 3 | 3 | 2 | 3 | 2 | **16** | 1 | High |
| a2 | 3 | 2 | 2 | 2 | 3 | 3 | **15** | 1 | High |
| a3 | 2 | 3 | 3 | 3 | 2 | 2 | **15** | 1 | High |
| a4 | 2 | 2 | 2 | 2 | 2 | 3 | **13** | 2 | Medium |
| a5 | 2 | 2 | 2 | 2 | 2 | 2 | **12** | 2 | Medium |
| a6 | 2 | 2 | 1 | 3 | 1 | 2 | **11** | 2 | Medium (catalyst play qua C) |
| a7 | 1 | 2 | 1 | 1 | 1 | 2 | **8** | 3 | Low (rank chót) |
| a8 | - | - | - | - | - | - | Loại | - | R3 scandal phát hiện khi check T2 |

**Chọn top 3 với cân bằng bucket:**

Mã cao điểm nhất (a1-a3) đều Bucket 1. Áp logic: chọn 2 Bucket 1 cao nhất (a1 16đ + a2 15đ), thay mã Bucket 1 thứ 3 (a3 15đ) bằng mã Bucket 2 cao nhất (a4 13đ).

Top 3 ngành A: **a1 (High B1), a2 (High B1), a4 (Medium B2)**. Mã a3 — sát nút vì cân bằng bucket.

### Ngành B — 6 mã shortlist

| Mã | T1 | T2 | T3 | T4 | T5 | T6 | Tổng | Bucket |
|---|---|---|---|---|---|---|---|---|
| b1 | 3 | 2 | 2 | 3 | 2 | 2 | **14** | 2 |
| b2 | 2 | 3 | 2 | 2 | 2 | 2 | **13** | 2 |
| b3 | 2 | 2 | 3 | 2 | 1 | 2 | **12** | 1 |
| b4 | 1 | 2 | 2 | 2 | 2 | 3 | **12** | 2 |
| b5 | 1 | 1 | 1 | 2 | 2 | 2 | **9** | 2 |
| b6 | - | - | - | - | - | - | Loại | - | R4 |

Top 3 theo điểm: b1 (14 B2), b2 (13 B2), b3 (12 B1). Distribution 1 B1 + 2 B2 — đạt target cân bằng.

### Ngành D — 5 mã shortlist

| Mã | T1 | T2 | T3 | T4 | T5 | T6 | Tổng | Bucket |
|---|---|---|---|---|---|---|---|---|
| d1 | 3 | 3 | 2 | 2 | 3 | 2 | **15** | 2 |
| d2 | 2 | 2 | 2 | 3 | 2 | 2 | **13** | 1 |
| d3 | 2 | 2 | 1 | 2 | 2 | 2 | **11** | 3 |
| d4 | 1 | 2 | 1 | 1 | 2 | 2 | **9** | 2 |
| d5 | 1 | 1 | 1 | 3 | 1 | 1 | **8** | 2 | catalyst play qua C |

Top 3: d1 (High B2), d2 (Medium B1), d3 (Medium B3). Distribution 1 B1 + 1 B2 + 1 B3 — đa dạng. Mã d3 Bucket 3 sẽ không vào ngay, tier 5/6 theo dõi.

### Tổng shortlist final

- 9 mã qua 3 ngành (3 mã/ngành đúng quota)
- Distribution conviction: 3 High, 5 Medium, 1 Low
- Distribution bucket: 3 Bucket 1, 5 Bucket 2, 1 Bucket 3
- 2 mã loại bắt buộc (a8 R3, b6 R4)
- 4 mã sát nút (a3, b4, b5, d4) — user có thể swap

---

## 10. Failure mode

### 10.1. Bảng chấm quá rộng, điểm trung bình cao

Agent có xu hướng cho nhiều điểm 2-3, ít điểm 1 vì "mã đã pass universe = đủ tốt". Kết quả: phần lớn mã có điểm 13-16, không phân biệt được tốt với xuất sắc.

**Xử lý:** thiết kế thang điểm theo distribution:
- Mã trung bình ngành → chủ yếu 1-2 điểm
- Mã top 30% ngành → 2-3 điểm
- Mã top 10% ngành → chủ yếu 3 điểm

Nếu phân bố điểm tập trung trên 14/18, cần siết lại tiêu chí. Target distribution: median shortlist ~11-12đ, top 25% ≥ 14đ, top 10% ≥ 16đ.

### 10.2. Quên check Tiêu chí 2 (quản trị)

Tiêu chí 2 cần web search 15-20 phút/mã, dễ skip khi gấp. Nhưng bỏ qua có thể dẫn đến lọt mã scandal vào shortlist → tier 5 phát hiện quá muộn.

**Xử lý:** bắt buộc web search Tiêu chí 2 cho **mọi mã** trong shortlist, không skip vì tốn thời gian. Nếu cần tối ưu, ưu tiên check kỹ top 5 theo điểm 5 tiêu chí khác, check nhanh (3-5 phút scan news + 1 web query scandal) cho mã ngoài top 5.

### 10.3. Chọn top 3 chỉ theo điểm, bỏ qua bucket

Agent sort mã theo tổng điểm và lấy top 3 → có thể toàn Bucket 1 hoặc toàn Bucket 2. Mất lợi ích diversify entry timing.

**Xử lý:** sau khi sort, luôn check distribution bucket top 3. Nếu vi phạm target (Section 4), thay mã phù hợp. Điểm có thể giảm 1-3đ nhưng diversify bucket quan trọng hơn cho horizon 1-6 tháng.

### 10.4. Cho điểm catalyst play quá cao vì "có catalyst"

Mã catalyst play qua đường C tier 2 có Tiêu chí 4 ≥ 2đ (bắt buộc có catalyst mạnh). Agent có xu hướng cộng thêm điểm các tiêu chí khác để "bù" cho fundamental yếu, dẫn đến catalyst play có điểm tương đương mã pass B.

**Xử lý:** chấm từng tiêu chí độc lập. Catalyst play có Tiêu chí 1 thấp (fundamental yếu) thì không bù bằng cách nâng Tiêu chí 2-3. Kết quả thực tế: catalyst play thường tổng 10-13đ (Medium), không phải High.

### 10.5. Áp benchmark ngành sai type

Agent so ROE của mã ngân hàng với median SXKD, hoặc so P/E của CK với SXKD. Dẫn đến điểm Tiêu chí 1 sai lệch nghiêm trọng.

**Xử lý:** bước chấm Tiêu chí 1 bắt buộc đọc `type` của mã và lấy benchmark từ cùng type. Industry_finstats có field `type` — match với stock_finstats.type. 4 bộ benchmark riêng biệt.

### 10.6. Không phát hiện mã trong "dòng tiền dương + catalyst tiêu cực"

Mã có Tiêu chí 5+6 cao (dòng tiền vào) + Tiêu chí 4 trung tính → Agent cho điểm tổng trung bình 11-13. Nhưng có thể bỏ sót catalyst tiêu cực mới chưa index kỹ (ví dụ: tin xấu ngành vừa công bố).

**Xử lý:** Bước 5 workflow — kiểm tra R4 trước khi finalize. Web search: `<ngành> tin xấu`, `<ngành> chính sách siết` trong 2 tuần gần. Đối chiếu với mã có Tiêu chí 5+6 cao trong ngành đó. Nếu match → R4 apply, loại.

### 10.7. Skip kiểm tra lại universe filter ở Bước 2

Tier 2 có thể chạy cách đây vài ngày, data có thể đã thay đổi (BCTC quý mới công bố, tin mới). Agent skip Bước 2 → tier 3 chấm điểm trên data cũ.

**Xử lý:** Bước 2 bắt buộc spot check 2-3 mã/ngành — đối chiếu snapshot_date của stock_snapshot với ngày tier 2 chạy. Nếu khác > 2 phiên, re-run tier 2 filter cho toàn shortlist trước khi chấm điểm.

### 10.8. Không phân tier conviction rõ ràng

Agent có thể chỉ xuất tổng điểm mà không map sang tier High/Medium/Low, hoặc đặt ngưỡng ad-hoc khác nhau giữa các ngành. Dẫn đến tier 6 sizing không nhất quán.

**Xử lý:** áp ngưỡng cứng Section 5 cho toàn shortlist: High 15-18, Medium 11-14, Low 8-10. Cùng ngưỡng cho mọi ngành. Dưới 8 là bất thường, flag riêng.

---

## 11. Đầu ra chuẩn để tier 5 dùng

Output tier 3 lưu file `tier3_YYYYMMDD_confirmed.md` (sau khi user confirm), gồm:

1. **Header:** ngày chạy, tier 2 ref, tổng mã top 3 final
2. **Shortlist final:** bảng đầy đủ với cột Tier conviction, bucket, tổng điểm, 6 điểm tiêu chí
3. **Chi tiết từng mã top 3:**
   - Lý do điểm từng tiêu chí
   - Catalyst cá thể (tên, timing, magnitude ước lượng) — input cho memo phần 5
   - Thesis preliminary (1-2 câu) — input cho memo phần 1
   - Mức technical key (hỗ trợ, kháng cự từ fibonacci/pivot/volume profile) — input cho memo + tier 5B
4. **Flags chuyển cho tier 5:**
   - Mã nào ưu tiên deep-dive PDF + modeling đầy đủ (tier High)
   - Mã nào làm memo gọn (tier Low)
   - Cảnh báo thanh khoản ảnh hưởng sizing
5. **Bảng mã sát nút + hard reject** — để user có thể swap/override ở tier 5 nếu cần
6. **Danh sách PDF user cần upload** — cho từng mã top 3, list 3 file: BCTC soát xét kỳ gần nhất, BCTC năm mới nhất, BCTN mới nhất
7. **Link audit log** nếu có override

File này được enable vào project knowledge session tier 5A kế tiếp. Tier 5A chạy riêng từng mã (per-stock checkpoint CP5A) nên có thể tách nhiều session.
