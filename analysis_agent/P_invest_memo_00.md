# P_invest_memo_00 — Master Workflow

File này là điểm vào tổng thể của quy trình đầu tư. Đọc xong 5-7 trang này sẽ hiểu toàn bộ kiến trúc, biết giai đoạn nào cần mở file con nào. Chi tiết thực thi nằm ở 9 file con của pack — không trùng lặp với master.

**Pack dependency:** `P_invest_memo` phụ thuộc `K_agent_db` (schema, query patterns, methodology diễn giải chỉ báo và tin tức). Khi chạy workflow này, agent phải đọc trước file `_00` của cả hai pack.

## 1. Mục đích & triết lý

**Mục đích:** xây quy trình đầu tư cá nhân cho thị trường cổ phiếu niêm yết Việt Nam, horizon 1-6 tháng, chỉ long, portfolio dưới 1 triệu USD. Quy trình đi từ bối cảnh thị trường → chọn 3-5 ngành → shortlist 9-15 mã → memo deep-dive cho top 3-5 mã → quản lý danh mục và exit.

**Triết lý cốt lõi:** quy trình kết hợp 5 trục phân tích không tách rời:

1. **Top-down** — không bao giờ stock-pick (chọn mã cụ thể) khi chưa xác định regime thị trường và ngành ủng hộ
2. **Universe filter vs ranking filter tách biệt** — phân biệt rõ hai vai trò của các trục phân tích:
   - **Universe filter** (loại thẳng nếu không đạt): fundamental + catalyst. Đây là hai trục cơ bản trả lời câu hỏi "doanh nghiệp/ngành có xứng đáng đầu tư không".
   - **Ranking filter** (xếp hạng ưu tiên trong universe, không loại): flow + kỹ thuật (bao gồm dòng tiền, technical zone, trend đa khung). **Dòng tiền được phân loại là chỉ báo kỹ thuật**, không dùng để loại thẳng mã/ngành khỏi universe.
   
   Khi áp dụng: ngành/mã phải lọt fundamental + catalyst để vào universe. Sau đó dùng flow + kỹ thuật để chọn top ưu tiên trong universe đã lọt.
3. **Flow advantage trong ranking** — tận dụng tối đa dữ liệu dòng tiền + kỹ thuật mà `K_agent_db` có sẵn. Đây là lợi thế so sánh lớn nhất so với retail khi xếp hạng trong universe, không phải khi loại universe.
4. **Catalyst-driven** — mỗi vị thế phải gắn với catalyst cụ thể có timing và magnitude, không mua theo "câu chuyện dài hạn"
5. **Conviction memo** — không vào position nếu chưa viết xong memo 7 phần với variant perception, bear case, và exit trigger
6. **Universe vs entry timing tách biệt** — trong phạm vi các chỉ báo ranking, khung trung-dài hạn (quý, năm) dùng để xếp hạng ưu tiên universe. Khung ngắn hạn (tuần, tháng) chỉ dùng để phân bucket entry timing sau khi đã chọn mã, không dùng cho ranking. Áp dụng nguyên tắc này tránh mất các mã "pullback trong uptrend" (điều chỉnh ngắn hạn trong xu hướng tăng dài hạn) — cơ hội mua giá tốt nhất ở horizon 1-6 tháng

**Horizon 1-6 tháng dẫn đến 3 ưu tiên cao nhất** (ảnh hưởng cách phân bổ effort):

- **Variant perception** — alpha (lợi nhuận vượt benchmark thị trường) ở horizon ngắn đến từ "thấy trước thị trường", không phải "ôm compounding" (gộp lãi dài hạn). Không có variant perception rõ ràng = không có lý do để giá chạy về target trong 3-6 tháng.
- **Monitoring + exit trigger** — horizon ngắn nghĩa là thesis được kiểm chứng nhanh, sai là phải thoát nhanh. Exit rules viết trước khi vào position là sự khác biệt giữa pro và amateur.
- **Portfolio construction + risk budget** — turnover cao (tần suất ra/vào position lớn) ở horizon ngắn làm tích luỹ sai sizing nhanh. Framework sizing rõ ràng bảo vệ capital tốt hơn bất kỳ stock pick nào.

**Giới hạn tự thừa nhận:**

- Không làm primary research (điều tra gốc: không interview management, không channel check — kiểm chứng thông tin qua nhà cung cấp/khách hàng/đối thủ, không đến showroom)
- Không forensic accounting chuyên sâu (soi kỹ bất thường BCTC để phát hiện gian lận): Agent flag được red flag rõ ràng, không bắt được thủ thuật tinh vi kiểu Enron/Wirecard
- Model DCF (Discounted Cash Flow — chiết khấu dòng tiền tương lai để định giá) chỉ ở mức cơ bản, không thay model của analyst lâu năm — dùng cho thesis-level decision, không phải precise target

Chấp nhận những giới hạn này đổi lấy scalability và tốc độ. Quy trình đạt ~80% chuẩn memo chuyên nghiệp quốc tế — đủ vượt xa retail VN nhưng không bằng quỹ institutional full-stack (quỹ định chế có đầy đủ research team, channel check, modeling sâu).

## 2. Từ điển thuật ngữ cốt lõi

Các thuật ngữ nền tảng xuất hiện xuyên suốt pack này. Agent và user cần hiểu thống nhất trước khi vào workflow chi tiết. Thuật ngữ đầu tư chuyên nghiệp ít gặp khác được giải thích inline khi xuất hiện lần đầu.

| Thuật ngữ | Giải thích |
|---|---|
| **Horizon 1-6 tháng** | Khoảng thời gian kỳ vọng giữ một position trước khi thoát. Pack thiết kế cho trung hạn, không phải trading ngắn dưới 1 tháng, cũng không phải holding dài trên 1 năm. |
| **Chỉ long** | Chỉ mua cổ phiếu hưởng chênh lệch tăng giá + cổ tức. Không short (bán khống), không đòn bẩy margin cao. |
| **Top-down** | Phương pháp phân tích từ trên xuống: đánh giá bối cảnh vĩ mô/thị trường trước → chọn ngành phù hợp → chọn mã trong ngành. Ngược với bottom-up (chọn mã trước, không quan tâm vĩ mô). |
| **Universe filter** | Bộ tiêu chí loại thẳng. Ngành/mã không đạt thì không vào danh sách xét tiếp, dù chỉ báo khác có mạnh đến đâu. Pack này dùng `fundamental + catalyst` làm universe filter. |
| **Ranking filter** | Bộ tiêu chí xếp hạng trong danh sách đã lọt universe. Chỉ quyết định ưu tiên ai vào trước ai vào sau, không loại ai. Pack này dùng `flow + kỹ thuật` làm ranking filter. |
| **Funnel A/B/C/D** | Tên gọi 4 trục phân tích dùng xuyên suốt pack. Funnel A (Flow + kỹ thuật) / Funnel B (Fundamental) / Funnel C (Catalyst) / Funnel D (Thanh khoản). Vai trò loại thẳng hay xếp hạng tùy vào tier. |
| **Regime** | Trạng thái vĩ mô thị trường mà agent quyết định ở tier 0, có 4 giá trị: **risk-on full** (ưa rủi ro, universe rộng), **risk-on selective** (chọn lọc hơn), **defensive only** (chỉ ngành phòng thủ), **đứng ngoài** (giữ tiền mặt, dừng quy trình). |
| **Catalyst** | Sự kiện hoặc điều kiện cụ thể có thể đẩy giá mã/ngành trong 0-3 tháng: kết quả kinh doanh, chính sách mới, chu kỳ hàng hóa, M&A, niêm yết công ty con. Phải có timing và magnitude cụ thể, không phải "câu chuyện dài hạn". |
| **Catalyst score** | Điểm chấm mức độ mạnh và độ chắc của catalyst — thang 1-3 cho mỗi catalyst đơn lẻ; tổng cộng max 6 điểm/ngành (2 catalyst × 3 điểm) ở tier 0/1. Điểm ≥ 3 ở mức single catalyst được coi là catalyst active hợp lệ để vào universe filter. |
| **Variant perception** | Thesis đi ngược hoặc khác với consensus thị trường đang nghĩ. Ở horizon 1-6 tháng, alpha đến từ "thấy trước thị trường" về một yếu tố cụ thể. Không có variant perception = không có lý do giá phải chạy về target trong vài tháng tới. |
| **Conviction memo** | Tài liệu phân tích 7 phần viết trước khi vào position: Recommendation / Thesis / Variant perception / Business / Financial / Catalysts + Bear case / Monitoring + exit. Không có memo hoàn chỉnh = không vào position. |
| **Conviction tier** | 3 mức độ tự tin về một mã sau chấm điểm tier 3, gán bằng tổng điểm bảng chấm (thang 18): **high** (15-18 điểm), **medium** (11-14), **low** (8-10). Dùng cho quyết định position sizing ở tier 6. |
| **Bucket entry** | Phân loại thời điểm vào lệnh sau khi đã chọn mã. **Bucket 1** — vào ngay khi tuần+tháng zone A+. **Bucket 2** — chờ pullback khi tuần hoặc tháng đang B/C trong khi quý+năm vẫn tích cực. **Bucket 3** — watchlist, chưa vào, đợi tuần chuyển B+. Nguyên tắc: universe và ranking dùng khung trung-dài hạn (quý+năm), còn tuần+tháng chỉ để phân bucket. |
| **Checkpoint (CP)** | Điểm dừng bắt buộc sau mỗi giai đoạn. Agent xuất báo cáo 6 phần và chờ user confirm/override trước khi qua giai đoạn kế. Pack có 7 checkpoint (CP1, CP2, CP3, CP4, CP5A, CP5B, CP5C), chi tiết ở Phần 6. |

## 3. Kiến trúc tổng thể

### Sơ đồ 5 giai đoạn

```
GIAI ĐOẠN 1 — Gate vĩ mô              (P_invest_memo_01)
       │
       ▼
   [Regime + bảng catalyst]
       │
       ▼
GIAI ĐOẠN 2 — Chọn 3-5 ngành          (P_invest_memo_02)
       │
       ▼
   [3-5 ngành lọt giao thoa 3 funnel]
       │
       ▼
GIAI ĐOẠN 3 — Screen mã trong ngành   (P_invest_memo_03)
       │
       ▼
   [6-10 mã/ngành]
       │
       ▼
GIAI ĐOẠN 4 — Chấm điểm, top 3/ngành  (P_invest_memo_04)
       │
       ▼
   [9-15 mã shortlist final]
       │
       ▼
GIAI ĐOẠN 5 — Memo deep-dive          (P_invest_memo_05, 06, 07)
       │
       ├──► 5A: PDF BCTC extraction   (P_invest_memo_05)
       ├──► 5B: Valuation modeling     (P_invest_memo_06, chỉ top 3-5)
       └──► 5C: Viết memo 7 phần       (P_invest_memo_07)
       │
       ▼
   [Memo hoàn chỉnh cho mỗi mã]
       │
       ▼
QUYẾT ĐỊNH VÀO POSITION
       │
       ▼
GIAI ĐOẠN 6 (song song) — Quản lý danh mục
       ├──► Portfolio construction    (P_invest_memo_08)
       └──► Monitoring + exit         (P_invest_memo_09)
```

### Bảng tóm tắt giai đoạn

| Giai đoạn | Input | Output | File | Tần suất | Checkpoint |
|---|---|---|---|---|---|
| 1. Gate vĩ mô | Market data + tin vĩ mô + NN flow | Regime + bảng catalyst active | `P_invest_memo_01` | Hàng tuần, hoặc khi có macro shock | CP1 |
| 2. Chọn ngành | 24 ngành | Universe (B∩C) → top 3-5 ngành xếp hạng theo flow+kỹ thuật | `P_invest_memo_02` | Hàng tuần sau giai đoạn 1 | CP2 |
| 3. Screen mã | Full ticker list ngành | Universe → 6-10 mã/ngành xếp hạng + bucket entry | `P_invest_memo_03` | Theo ngành đã chọn | CP3 |
| 4. Chấm điểm | 6-10 mã/ngành | Top 3 mã/ngành (ranking tool, không filter) | `P_invest_memo_04` | Sau giai đoạn 3 | CP4 |
| 5A. PDF deep-dive | 3 PDF/mã | Red flag report | `P_invest_memo_05` | Hàng tháng/quý | CP5A (per-stock) |
| 5B. Modeling | DB + PDF + web | Target giá base/bull/bear | `P_invest_memo_06` | Chỉ top 3-5 high conviction | CP5B |
| 5C. Memo | Tất cả output trước | Memo 7 phần | `P_invest_memo_07` | Bắt buộc trước khi vào position | CP5C |
| 6A. Portfolio | Memo + shortlist | Sizing + rebalance plan | `P_invest_memo_08` | Hàng tháng + khi vào/ra position | Không có CP cứng |
| 6B. Monitoring | Position hiện tại | Trigger check + exit decision | `P_invest_memo_09` | Hàng tuần | Không có CP cứng |

Chi tiết từng checkpoint xem Phần 6. Giai đoạn 6 không có checkpoint cứng vì đây là hoạt động liên tục, không phải quyết định một lần.

**Ghi chú numbering Tier:** Tier mapping không liên tục — không có "Tier 4" do giai đoạn 5 (deep-dive) đã được tách thành 3 sub-tier 5A/5B/5C ở refactor lịch sử. File con vẫn dùng "Tier 5A/5B/5C" giữ nguyên để tránh phá hệ thống cross-reference. Khi đọc file con, "Tier" là reference cụ thể; "Giai đoạn" là grouping high-level dùng trong master để mô tả workflow tổng.

**Mapping Giai đoạn 6 → Tier:** "Giai đoạn 6 song song" trong master gồm 2 hoạt động chạy đồng thời sau khi có memo tier 5C — Tier 6 (`P_invest_memo_08` — Portfolio sizing + rebalance) và Tier 7 (`P_invest_memo_09` — Monitoring + Weekly Review Session). File 08 đặt tên "Tier 6", file 09 đặt tên "Tier 7", master gộp chung là "Giai đoạn 6". Khi cross-reference, dùng "Tier 6" hoặc "Tier 7" cho file cụ thể, "Giai đoạn 6" khi nói tổng quan portfolio + monitoring.

## 4. Flow chi tiết từng giai đoạn

Phần này tóm tắt mục tiêu, input/output, và logic cốt lõi của mỗi giai đoạn. Spec kỹ thuật chi tiết (query DB, tiêu chí numeric, prompt, edge case) nằm trong file tier tương ứng.

### Giai đoạn 1 — Gate vĩ mô (`P_invest_memo_01`)

**Mục tiêu:** quyết định 1 trong 4 regime — risk-on full / selective / defensive only / đứng ngoài. Lập bảng catalyst active trong 0-3 tháng tới.

**Input:** market trend đa khung + vĩ mô quốc tế + vĩ mô nội địa + dòng tiền khối ngoại toàn thị trường.

**Output:** regime + bảng catalyst + quota cho tier kế. Quota theo regime:
- Risk-on full: 5 ngành × 3 mã
- Risk-on selective: 3 ngành × 3 mã, tiêu chí khắt khe hơn
- Defensive only: 2 ngành × 3 mã, chỉ ngành phòng thủ
- Đứng ngoài: dừng quy trình, giữ tiền mặt

### Giai đoạn 2 — Chọn 3-5 ngành (`P_invest_memo_02`)

**Mục tiêu:** từ 24 ngành, chọn 3-5 ngành qua kiến trúc 2 tầng universe filter → ranking filter.

**Universe filter (loại thẳng):** Funnel B (Fundamental — growth, biên, P/E so median 3Y) ∩ Funnel C (Catalyst — điểm ≥ 3 từ tier 0).

**Ranking filter (xếp hạng, không loại):** Funnel A (Flow + kỹ thuật — rank ngành tự tổng hợp theo `week_score`, điểm dòng tiền tuần, trend quý+năm). Chọn top 3-5 trong universe.

**Exception:** catalyst override — 1 ngành/chu kỳ có thể lọt dù không đạt B nếu catalyst score = 6 và xu hướng 20 phiên bật từ đáy.

**Không có tiêu chí loại bắt buộc theo kỹ thuật** — y_trend > 0.8 chỉ là cảnh báo cho giai đoạn 3.

### Giai đoạn 3 — Screen mã trong ngành (`P_invest_memo_03`)

**Mục tiêu:** mỗi ngành thu còn 6-10 mã qua universe filter → ranking filter → phân bucket entry.

**Universe filter (loại thẳng):** (Vòng B Fundamental ∩ Vòng D Thanh khoản ≥ 5 tỷ/phiên trung bình 20 phiên) ∪ (Vòng C Catalyst mã ∩ Vòng D). C là đường riêng cho catalyst play có thể không đạt B. Chi tiết tiêu chí D ở `P_invest_memo_03` mục 4 (D1: trading value ≥ 5 tỷ, D2: volume ≥ 100k cp, D3: market_rank_pct > 0).

**Ranking filter (xếp hạng, không loại):** Vòng A (Flow + kỹ thuật trung-dài hạn — zone quý/năm, xếp hạng dòng tiền thị trường, điểm dòng tiền tuần). Top 6-10 trong universe.

**Phân bucket entry timing (sau khi có shortlist):**
- Bucket 1 — Vào ngay: tuần+tháng cùng zone A+, điểm dòng tiền tuần ≥ 6. Horizon 1-3 tháng, vào 50-70% size ngay
- Bucket 2 — Chờ xác nhận (pullback): quý+năm tích cực, tuần hoặc tháng B/C. Horizon 3-6 tháng, vào 30-50% trước, đợi tuần bật A thêm
- Bucket 3 — Watchlist: quý+năm tích cực nhưng tuần+tháng đều C. Chưa vào, đợi tuần chuyển B+

### Giai đoạn 4 — Chấm điểm, top 3/ngành (`P_invest_memo_04`)

**Mục tiêu:** từ 6-10 mã/ngành xuống 3 mã/ngành qua bảng chấm 6 tiêu chí (max 18).

**Vai trò bảng chấm:** chỉ là ranking tool, không filter. Tất cả mã đã lọt universe ở tier 2.

**6 tiêu chí:** vị thế ngành, quản trị & pháp lý, thanh khoản, catalyst cá thể, dòng tiền NN/TD 1 tháng, đồng đều điểm dòng tiền 5 phiên.

Hai tiêu chí cuối (dòng tiền) đóng vai tiebreaker khi 4 tiêu chí đầu bằng điểm nhau.

**Cân bằng bucket:** top 3 không nhất thiết 3 mã cao điểm nhất — cần phân bổ đa bucket để linh hoạt entry timing.

**Output:** 9-15 mã final phân vào 3 tier conviction (high 15-18 / medium 11-14 / low 8-10).

### Giai đoạn 5 — Memo deep-dive (`P_invest_memo_05, 06, 07`)

**Không vào position nếu chưa hoàn thành giai đoạn này.** Gate cuối loại mã "đẹp giấy nhưng không conviction".

- **5A. PDF extraction (`P_invest_memo_05`):** 6 tác vụ forensic (soi kỹ bất thường trong báo cáo tài chính), red flag report/mã
- **5B. Modeling (`P_invest_memo_06`):** chỉ top 3-5 high conviction, DCF + peer multiples (so sánh hệ số định giá với công ty cùng ngành như P/E, P/B, EV/EBITDA) + sensitivity (thử thay đổi giả định đầu vào để xem target giá nhạy đến đâu)
- **5C. Memo 7 phần (`P_invest_memo_07`):** Recommendation + Thesis + Variant perception + Business + Financial + Catalysts + Bear case + Monitoring/exit

**Gate cuối:** fail variant perception hoặc bear case mạnh hơn bull case → loại.

### Giai đoạn 6 (song song) — Quản lý danh mục

- **6A. Portfolio (`P_invest_memo_08`):** position sizing theo conviction tier, constraint thanh khoản/ngành/vĩ mô, cash buffer theo regime, entry tranching theo bucket
- **6B. Monitoring + exit (`P_invest_memo_09`):** 3 cấp trigger (quý/tháng/tuần), 4 exit trigger (target/thesis broken/better opportunity/stop loss), trigger riêng cho bucket 2

## 5. Sáu nguyên tắc Agent bất biến

Các nguyên tắc này áp dụng ở mọi giai đoạn. Agent không được skip hay tự ý nới lỏng, bất kể bối cảnh.

**Nguyên tắc 1 — Không bao giờ skip variant perception cho mã vào shortlist.**

Mỗi mã lọt shortlist final phải trả lời được 3 câu hỏi: consensus sell-side (quan điểm trung bình của analyst công ty chứng khoán) đang nghĩ gì về mã này, consensus retail đang nghĩ gì, và thesis khác consensus ở chỗ nào cụ thể. Đây là điều kiện tiên quyết cho alpha ở horizon 1-6 tháng. Nếu không viết được câu 3 (variant perception ≈ consensus, hoặc differentiation chỉ là wishful thinking không có evidence, hoặc undervalued không có catalyst để re-rate), Agent **flag cảnh báo + downgrade conviction 1 bậc** (High → Medium / Medium → Low / Low → Watch list), user quyết định proceed với size nhỏ + audit log hoặc loại mã. Agent không tự reject — discipline ở dạng force user explicit aware về rủi ro, không che giấu cảnh báo. Chi tiết Gate 1 ở `P_invest_memo_07`.

**Nguyên tắc 2 — Không vào position nếu chưa viết xong exit trigger.**

Memo phần 7 phải liệt kê 4 điều kiện thoát cụ thể, measurable: target giá đạt (từ modeling), thesis broken (điều kiện cụ thể ví dụ "biên gộp xuống dưới X% trong 2 quý"), better opportunity (ngưỡng risk-reward so với position mới), stop loss (-15% hoặc -20%). Exit trigger viết trước khi vào, không sửa trong drawdown. Đây là nguyên tắc bảo vệ capital quan trọng nhất ở horizon ngắn.

**Nguyên tắc 3 — Không bao giờ size mỗi phiên giải ngân vượt 5% ADV 20 phiên của mã.**

ADV 20 phiên (Average Daily Volume — khối lượng giao dịch trung bình 20 phiên gần nhất) là constraint thanh khoản tuyệt đối, áp **per-phiên giải ngân**, không phải absolute cap cho tổng vị thế. Cụ thể: max size mỗi phiên ≤ 5% × ADV; build full position phân 2-4 phiên → max tổng vị thế = 5% × ADV × N với N=2-4 (chi tiết công thức `P_invest_memo_08` Section 3.4). Portfolio < 1 triệu USD vẫn phải tuân thủ vì thói quen sẽ theo khi portfolio lớn lên. Slippage (chênh lệch giữa giá dự kiến và giá khớp thực tế do thanh khoản thấp) khi vào và ra ở mã thanh khoản thấp ăn trực tiếp vào alpha. Nếu đạt cả 3 tier conviction nhưng vi phạm constraint này → giảm size xuống mức hợp lệ, không bỏ qua ràng buộc.

**Nguyên tắc 4 — Bear case phải được steelman trước khi final long.**

Memo phần 6 phải viết 3 lập luận mạnh nhất để short mã từ góc độ một analyst Hindenburg Research (công ty nghiên cứu short-side nổi tiếng, chuyên phát hiện gian lận kế toán và overvalued stocks), mỗi lập luận có claim cụ thể với số, dẫn chứng từ BCTC hoặc tin tức, ước lượng downside nếu đúng. Đây là cách "steelman" — xây dựng luận điểm đối lập ở dạng mạnh nhất có thể trước khi phản biện. Sau đó mới viết phản biện. Nếu phản biện yếu so với bear case (dưới 1/3 rebuttal thuyết phục), Agent **flag cảnh báo + downgrade size 30-50%** so với conviction gốc; nếu probability-weighted bear target dưới giá hiện tại (downside > upside), **flag cảnh báo nghiêm trọng + downgrade size 50-70%** (coin-flip bet). User quyết định cuối — có thể proceed với size nhỏ và audit log rõ lý do override cảnh báo, hoặc loại mã. Đây là cơ chế chống confirmation bias (xu hướng chỉ tìm bằng chứng ủng hộ ý kiến của mình, bỏ qua bằng chứng ngược lại) — nguồn lỗi lớn nhất của retail. Chi tiết Gate 2 ở `P_invest_memo_07`.

**Nguyên tắc 5 — Dòng tiền dương + catalyst tiêu cực → loại, không bàn thêm.**

Khi mã có dòng tiền đang mạnh nhưng catalyst ngành/mã đã chuyển tiêu cực (tin xấu chính thức, chính sách siết chặt, hàng hoá đầu vào tăng cấu trúc) → dòng tiền đang priced-in (tin đã phản ánh vào giá) thứ gì đó chưa lộ, hoặc là late money (dòng tiền vào muộn, thường là retail đuổi theo khi đà tăng đã gần hết) sẽ kẹt. Đây là tình huống bẫy retail phổ biến nhất ở thị trường VN. Quy tắc: loại thẳng, không override bằng lý do "dòng tiền mạnh hẳn biết điều gì đó tốt".

**Nguyên tắc 6 — Mỗi giai đoạn kết thúc bằng checkpoint review, Agent không tự động chuyển tier.**

Agent xuất báo cáo checkpoint theo khung 6 phần chuẩn (xem Phần 6) sau mỗi giai đoạn, chờ user xác nhận hoặc điều chỉnh trước khi sang giai đoạn kế. **Tuyệt đối không chạy liên tục qua nhiều giai đoạn trong 1 session** — điều này ngăn lỗi ở giai đoạn sớm lây sang các giai đoạn sau. User có quyền override quyết định (thêm/bớt ngành/mã, thay đổi quota, yêu cầu deep-dive thêm) nhưng mỗi override phải được ghi vào audit log với lý do cụ thể, để sau này đánh giá chất lượng override theo thời gian.

**Lưu ý — convention nội bộ pack ngoài 6 nguyên tắc bất biến:**
- **Catalyst play exposure cap:** mã qua đường C tier 2 (đạt Catalyst, fail Fundamental) có cap riêng ≤ 15% portfolio, max 2-3 mã (chi tiết `P_invest_memo_08` Section 7.4).
- **Bucket 2 timeout:** sau 4 tuần kể từ Phase 1 entry, nếu Bucket 2 chưa confirm Phase 2 → agent flag user xem xét đóng Phase 1 hoặc giữ chờ thêm (chi tiết `P_invest_memo_08` Section 5.2 + `P_invest_memo_09` Section 5.2).

Đây là tactical convention ở tầng portfolio construction / monitoring, không phải nguyên tắc universal — user có thể override với audit log.

## 6. Cơ chế checkpoint review

Mỗi giai đoạn kết thúc bằng một checkpoint — Agent xuất báo cáo theo khung chuẩn, chờ user xác nhận trước khi sang tier kế. Chi tiết use case từng checkpoint + template báo cáo cụ thể nằm trong `P_invest_memo_01` đến `P_invest_memo_09` tương ứng. Phần này chỉ định nghĩa khung chung.

### Khung báo cáo 6 phần

Mọi checkpoint phải theo đúng khung này:

1. **Summary quyết định** (2-3 dòng) — một câu chốt về output của tier
2. **Bối cảnh đầu vào** — input dùng (regime, universe size, ngày snapshot, cảnh báo độ tươi)
3. **Quyết định + lý do** (phần chính) — mỗi lựa chọn nêu cụ thể: tiêu chí đạt bằng số nào, thiếu gì. Không lý do chung chung
4. **Số liệu kỹ thuật key** — 3-5 số quan trọng nhất cho mỗi quyết định để user đánh giá nhanh chất lượng
5. **Lựa chọn sát nút** — ngành/mã suýt đạt hoặc lọt universe nhưng ranking thấp, để user có thể override
6. **Câu hỏi chờ user** — cụ thể, có default + list điều chỉnh khả dĩ

Agent không tự chuyển sang tier kế. Chờ user trả lời Phần 6 (đặt cuối checkpoint) rõ ràng. Template chi tiết cho mỗi checkpoint xem trong file tier.

**Tier-specific data sections:** template file con (01-08) có thể insert thêm 1-3 phần data riêng (vd "Bảng catalyst active" ở CP1, "Mã bị loại bắt buộc" ở CP4, "Sequence entry" ở Tier 6) giữa Phần 5 (Lựa chọn sát nút) và "Câu hỏi user". 6 phần named anchors bên trên là yêu cầu tối thiểu — extra data sections cho phép, miễn là "Câu hỏi user" luôn ở cuối + 6 anchor đều hiện diện.

### Danh sách 7 checkpoint

| ID | Sau giai đoạn | Scope | Output review |
|---|---|---|---|
| CP1 | Gate vĩ mô (tier 0) | Batch | Regime + bảng catalyst + quota |
| CP2 | Chọn ngành (tier 1) | Batch | 3-5 ngành + lý do |
| CP3 | Screen mã (tier 2) | Batch | 6-10 mã/ngành + bucket entry |
| CP4 | Chấm điểm (tier 3) | Batch | Top 3 mã/ngành + tier conviction |
| CP5A | PDF deep-dive (tier 5A) | Per-stock | Red flag report mỗi mã |
| CP5B | Modeling (tier 5B) | Per-stock (top 3-5) | Giả định + target giá |
| CP5C | Memo hoàn chỉnh (tier 5C) | Per-stock | Gate cuối: variant perception + bear case + exit trigger |

CP5A làm riêng từng mã, không batch — vì red flag của mã này không liên quan mã khác. CP5B chỉ làm cho top 3-5 high conviction.

### Quyền override và audit log

User có quyền override quyết định Agent ở bất kỳ checkpoint nào (thêm/bớt ngành/mã, thay đổi quota, loại mã có issue qualitative). Mỗi override phải được ghi vào file `audit_overrides.md` với format:

```
[Ngày] - CP[X] - Loại: [thêm/bớt/thay đổi]
Quyết định Agent: [original]
User override: [thay đổi]
Lý do user: [nhập từ user]
```

Sau 3-6 tháng, review audit log để:
- Đánh giá override của user đúng/sai so với kết quả thực tế
- Phát hiện pattern override (user có xu hướng bias nhóm nào)
- Calibrate lại rule Agent nếu override đúng ≥ 80%

Chi tiết format audit log và cách Agent append entry xem phần "Setup project knowledge" dưới đây và trong từng file tier.

## 7. Hướng dẫn dùng trong Claude app

Pack `P_invest_memo` chạy trong Claude project với architecture 3 layer: system prompt (meta-rules cố định) + `KERNEL_SKELETON.md` (liệt kê pack available) + pack K/P/O. Chi tiết kiến trúc xem system prompt và `KERNEL_SKELETON.md`.

### Setup project knowledge

Claude project cần chứa các thành phần sau để agent chạy đúng:

**Layer meta (luôn active):**
- System prompt: đã load vào Claude project settings, không nằm trong knowledge
- `KERNEL_SKELETON.md`: file index liệt kê các pack available

**Pack dependency (bắt buộc load trước khi chạy workflow):**
- Pack `K_agent_db` — 6 file (`K_agent_db_00` đến `K_agent_db_05`) chứa schema, query patterns, anti-patterns, methodology diễn giải chỉ báo và tin tức. P_invest_memo gọi K_agent_db như thư viện cho mọi query định lượng.

**Pack này:**
- `P_invest_memo_00` (file master này) + 9 file tier con (`P_invest_memo_01` đến `P_invest_memo_09`).

**State artifacts (runtime, tạo và update theo thời gian):**
- Các memo đã viết — Agent reference khi đánh giá position mới tương tự hoặc khi re-review position cũ
- Danh sách catalyst active hiện tại (update hàng tuần sau CP1)
- File `audit_overrides.md` — append entry mỗi khi user override quyết định agent

Agent có thể search cross-file trong project knowledge khi cần. Không cần upload lại mỗi session.

### Cách start session mới

Agent tự động route qua router của Kernel khi user gọi giai đoạn cụ thể. Không cần user nói tên file — agent sẽ activate đúng pack dựa trên intent. Một số message mẫu:

**Session cho giai đoạn 1 — Gate vĩ mô:**

> "Chạy Gate vĩ mô cho tuần này. Output regime + bảng catalyst active."

Agent sẽ:
1. Activate `P_invest_memo` (router trong kernel skeleton match trigger "Gate vĩ mô")
2. Đọc `P_invest_memo_00` để nắm flow, rồi đọc `P_invest_memo_01` cho spec tier 0
3. Gọi `K_agent_db` cho query DB (market data, NN flow, other_data)
4. Web search tin vĩ mô gần nhất + lịch catalyst
5. Phân tích theo logic `P_invest_memo_01`
6. Output regime + bảng catalyst theo khung checkpoint 6 phần (CP1)

Bạn review. Nếu OK, lưu output vào 1 file markdown làm input cho session giai đoạn 2.

**Session cho giai đoạn 2-4:** tương tự, nói tên giai đoạn ("chọn ngành", "screen mã ngành X", "chấm điểm shortlist"), agent route đúng file tier. Input là output đã confirm của giai đoạn trước — paste vào đầu session hoặc upload file markdown.

**Session cho giai đoạn 5 (memo deep-dive):**

Upload 3 PDF (BCTC soát xét, BCTC năm, báo cáo thường niên) của 1 mã. Message đầu:
> "Chạy memo deep-dive cho mã X. Đã upload 3 PDF."

Agent làm tuần tự 5A → 5B → 5C theo `P_invest_memo_05/06/07`. Với mã low conviction, có thể skip 5B. Output là memo 7 phần.

**Session cho giai đoạn 6 (monitoring):**

Hàng tuần chạy 1 session:
> "Chạy monitoring tuần cho danh mục hiện tại gồm [list mã + entry price + entry date]."

Agent check các trigger theo `P_invest_memo_09`, flag mã cần review.

### Pattern 1 session/tier + checkpoint

Mỗi session tương ứng 1 giai đoạn + 1 checkpoint. Agent không được chạy qua nhiều giai đoạn trong 1 session (đây là ràng buộc từ Nguyên tắc 6 + rule checkpoint discipline ở system prompt mục 5.6).

Chia nhỏ session có 4 lợi ích:

1. **Context length không bị quá tải** — system prompt + kernel skeleton + `K_agent_db_00` + `P_invest_memo_00` + 1 file tier đủ cho 1 session
2. **Review dễ hơn** — output mỗi tier gọn, bạn catch lỗi sớm trước khi sang tier kế
3. **Lưu trữ có cấu trúc** — mỗi tier 1 file markdown output, build được audit trail rõ ràng
4. **Checkpoint bắt buộc** — end session khi Agent xuất báo cáo checkpoint, bạn review offline, start session mới cho tier kế với output đã confirm làm input

**Flow 1 session:**

1. User mở session mới với message yêu cầu giai đoạn cụ thể
2. Agent route, load pack dependencies, đọc file master trước khi đọc file tier (theo rule master-first reading system prompt mục 5.7)
3. Agent chạy logic của tier, query DB qua K_agent_db, web search khi cần
4. Agent xuất báo cáo checkpoint 6 phần
5. Agent dừng, chờ user confirm/override (Nguyên tắc 6 + system prompt mục 5.6)
6. User review, trả lời câu hỏi ở Phần 6 của báo cáo
7. Nếu confirm → end session, lưu output. Nếu có override → Agent ghi audit log, chạy lại với điều chỉnh, xuất báo cáo mới
8. Output confirmed trở thành input cho session tier kế

**Lưu trữ audit trail:**

- Mỗi session output lưu thành file markdown riêng, đặt tên theo format `tier[X]_YYYYMMDD_[status].md`
- Audit log override là file chung `audit_overrides.md`, append entry mỗi lần có override
- Sau 3-6 tháng, build được dataset để đánh giá chất lượng quyết định

### Khi nào cần bạn can thiệp manual

- **Variant perception draft** — Agent viết version đầu, bạn sharpen với góc nhìn ngành/doanh nghiệp bạn đã theo dõi lâu
- **Bear case phản biện** — Agent steelman bear case, bạn viết phản biện dựa trên insight riêng
- **Model giả định** — Agent dùng default (tăng trưởng theo historical trend, biên theo 3 năm gần), bạn challenge và điều chỉnh
- **Management quality** — Agent đọc được track record on paper, không đọc được "con người" — phần này bạn tự cảm nhận
- **Position sizing cuối cùng** — Agent đề xuất theo conviction tier, bạn có thể điều chỉnh ± 50% dựa trên portfolio context hiện tại

### Cross-reference giữa pack và file

Thứ tự ưu tiên khi Agent không chắc cách thực thi:

1. File tier tương ứng của `P_invest_memo` (01-09) — chi tiết thực thi workflow
2. `P_invest_memo_00` (file này) — nguyên tắc bất biến + flow tổng thể + thuật ngữ cốt lõi
3. Pack `K_agent_db` (00 đến 05) — schema, query patterns, methodology diễn giải chỉ báo và tin tức
4. System prompt + `KERNEL_SKELETON.md` — chỉ khi gặp vấn đề meta (clarification, rollback, layer activation, output style)

Không tự suy luận vượt spec khi spec đã có. Nếu spec chưa cover case cụ thể, hỏi user xác nhận trước khi tự quyết (theo rule clarification-before-analysis ở system prompt mục 5.4).
