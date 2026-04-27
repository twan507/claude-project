# P_weekly_market_00 — Master Workflow

Pack `P_weekly_market` build báo cáo thị trường tuần phục vụ họp nội bộ. Báo cáo độc lập với pack `P_invest_memo` — không đọc state file invest cycle, không liên quan portfolio đang cầm. Chạy 1 lần/tuần, output 1 file MD, có 1 checkpoint duy nhất sau khi quyết regime + sector bias.

Pack 1 file. Toàn bộ workflow + methodology nằm trong file này. Render spec ở `O_weekly_market_00`.

## 1. Mục đích & scope

**Mục đích:** sinh báo cáo thị trường tuần dạng deliverable MD, **dùng được cho cả họp nội bộ và gửi khách hàng**. Báo cáo gồm 12 phần, mục tiêu cao nhất là (a) thống kê đầy đủ dữ liệu thị trường tuần đã qua và (b) đưa ra chiến lược giao dịch cho tuần tới.

**Wording chung cho cả 2 audience:** dùng dạng observation/luận điểm phân tích, không dùng từ command (mua, bán, giảm tỷ trọng, stop loss). Sector bias diễn đạt "quan tâm / thận trọng" thay vì "tập trung / tránh". Watchlist phần 11 đề xuất mã có cơ hội đầu tư qua luận điểm, không kèm level giá vào/ra/stop. Cách này đảm bảo nội bộ vẫn diễn dịch đủ thành action, và khách hàng đọc được như góc nhìn phân tích để tự cân nhắc.

**Input kỳ vọng:**
- Trigger từ user: "viết báo cáo tuần", "weekly market report", hoặc tương đương
- File báo cáo tuần W-1 (optional) — user gửi đính kèm nếu muốn có phần Review
- Context bổ sung user cung cấp (sự kiện đặc biệt, focus ngành cụ thể, override default)
- Branding & disclaimer info (optional) — user cung cấp khi muốn báo cáo có cover branding + disclaimer footer cho audience khách hàng

**Output kỳ vọng:** file MD 9-11 trang theo structure rigid 12 phần `O_weekly_market_00` quy định. Có hoặc không có branding/disclaimer tuỳ user cung cấp.

**Tần suất phát hành:** tối Chủ Nhật / sáng thứ Hai trước phiên giao dịch đầu tuần. Tuần báo cáo = thứ Hai → Chủ Nhật của tuần đã kết thúc.

**Negative scope:**
- Không đọc state file của `P_invest_memo` (tier 6 portfolio, tier 7 weekly review)
- Không đưa khuyến nghị position cụ thể với level giá vào/ra/stop — watchlist phần 10 đề xuất mã qua luận điểm đầu tư, không kèm entry/stop/target
- Không dùng từ command trực tiếp (mua, bán, giảm tỷ trọng) — diễn đạt qua observation
- Không gán xác suất % cho kịch bản — dùng if-then trigger objective (xem K_agent_db_00 mục 4.3)
- **Không sử dụng chỉ báo trend** (`market_snapshot.trend`, `industry_snapshot.trend`, `group_snapshot.trend`, `*_recent.recent_trend`) — đây là phương pháp phân tích nội bộ, audience cuối (KH) không có nền methodology để hiểu. Chỉ `P_invest_memo` (audience analyst nội bộ) dùng trend.
- Không phân tích portfolio cá nhân, không suggest rebalance

**Dependency:** `K_agent_db`. Pack đọc trước `K_agent_db_00` master, các file con khi cần (chủ yếu `_01` schema, `_02` query patterns, `_05` news methodology).

## 2. Naming convention & lưu trữ

**Naming file output:** `weekly_market_<YYYYMMDD>.md` — ngày là ngày kết thúc tuần (Chủ Nhật).

**Lưu trữ:** agent KHÔNG lưu file qua các session. User tự archive sau khi nhận. Khi cần file W-1 cho phần Review, agent yêu cầu user upload.

## 3. Workflow tổng thể

Workflow chia 2 stage, ngăn cách bằng 1 checkpoint duy nhất:

```
─── Pre-flight ──────────────────────────────────────
  Hỏi user về file W-1
  Hỏi context bổ sung (optional)
  Hỏi branding & disclaimer info (optional)

─── Stage 1: Compose phần 2-9 ───────────────────────
  Phần 2  Review tuần trước (skip nếu không có file)
  Phần 3  Bối cảnh quốc tế
  Phần 4  Thị trường Việt Nam
  Phần 5  Vĩ mô & hàng hoá — yếu tố dẫn dắt ngành
  Phần 6  Biến động ngành
  Phần 7  Top dẫn dắt — 2 góc nhìn
  Phần 8  Tin tức & catalyst
  Phần 9  Phân tích kỹ thuật VNINDEX + 3 kịch bản + Risk map

─── CHECKPOINT 1: Regime + Sector bias ──────────────
  Agent xuất block call sơ bộ
  User confirm / override / yêu cầu phân tích thêm

─── Stage 2: Compose phần 10-12 + Phần 1 ────────────
  Phần 10  Watchlist — Mã đáng chú ý
  Phần 11  Lịch sự kiện tuần tới
  Phần 12  Tuyên bố miễn trừ trách nhiệm
  Phần 1   Tóm tắt điều hành (viết cuối)

─── Render & deliver ────────────────────────────────
  Compile full MD theo O_weekly_market_00 structure
  Save file weekly_market_<YYYYMMDD>.md
  Xuất nội dung MD trong message (Claude Desktop)
```

## 4. Pre-flight — hỏi user trước khi vào Stage 1

Agent hỏi 1 turn 3 câu, user trả lời rồi mới chạy:

```
Trước khi tôi bắt đầu compose báo cáo tuần [DD/MM-DD/MM/YYYY], xác nhận 3 điểm:

1. File báo cáo tuần W-1:
   (a) Có, tôi gửi đính kèm
   (b) Không có / tuần đầu cycle / skip phần Review

2. Có context đặc biệt cần lưu ý không?
   (a) Không, chạy default
   (b) Có — [user nêu]: focus ngành / sự kiện đặc biệt / override scope

3. Branding & disclaimer info (báo cáo có thể dùng cho cả nội bộ và gửi khách hàng):
   (a) Có — vui lòng cung cấp: tên công ty, logo (optional), hotline, website, phòng ban biên soạn, nội dung disclaimer mong muốn
   (b) Không cần — render bản plain
```

User trả lời → agent ghi nhận + chạy Stage 1. Branding info user gửi sẽ được insert vào header và footer file MD final ở bước render (xem `O_weekly_market_00`).

## 5. Stage 1 — Compose phần 2 đến phần 9

Mỗi phần dưới đây liệt kê: input cần query, methodology, output structured.

### 5.1. Phần 2 — Review tuần trước

**Skip nếu:** user chọn (b) ở pre-flight (không có file W-1).

**Compose nếu:** user upload file MD báo cáo W-1.

**Đọc từ file W-1:**
- 3 kịch bản VNINDEX phần 9 → so sánh với thực tế tuần vừa qua: kịch bản nào đã match
- Sector bias phần 10 → ngành quan tâm tuần trước có chạy đúng hướng không
- Watchlist phần 11 → mã đáng chú ý đã chạy đúng luận điểm chưa
- Risk map phần 12 → rủi ro nào đã materialize

**Output structured:**
- Kịch bản match: [cơ sở/tích cực/tiêu cực] hoặc lệch khỏi cả 3 → mô tả thực tế
- Hit rate sector bias: [N/M ngành quan tâm tăng giá]
- Hit rate watchlist: [N/M mã chạy đúng luận điểm — phân biệt mã hướng tích cực và mã hướng tiêu cực trong tổng]
- 1-2 learning chính cho tuần tới

### 5.2. Phần 3 — Bối cảnh quốc tế

**Query DB:**

`other_data` filter:
- group `international.global_index`: S&P 500, Dow Jones, Nasdaq, Nikkei 225, Shanghai Composite
- group `international.fx`: EUR/USD, GBP/USD, USD/JPY, USD/CNY, DXY
- group `international.bonds`: TPCP Mỹ 10Y
- group `macro.monetary` của các NHTW lớn (FED, ECB, PBOC) — nếu có trong DB; nếu không, web search

Lấy field: `name`, `value`, `unit`, `pct_change` (phiên gần nhất), `w_pct`, `m_pct`, `update_date`.

**Web search nếu cần:** lãi suất điều hành FED/ECB/PBOC nếu DB không có, hoặc cập nhật phát biểu Fed/ECB tuần qua.

**Output structured:**
- Bảng chứng khoán quốc tế: 5-6 chỉ số, biến động tuần + tháng
- Bảng tỷ giá quốc tế: 5 cặp + DXY
- Bảng lãi suất quốc tế: 4 mục
- Tổng kết prose 3-4 dòng: tâm lý chung quốc tế, áp lực USD, kỳ vọng lãi suất

### 5.3. Phần 4 — Thị trường Việt Nam

**Query DB:**

1. `market_snapshot` (1 doc duy nhất):
   - `price.close` (VNINDEX), `price.pct_change`, `price.trading_value` (GTGD phiên cuối tuần)
   - `change.w_pct`, `m_pct`, `q_pct`, `y_pct`
   - `breadth.breadth_in/out/neu` (rổ FNXINDEX, không phải toàn HOSE — xem K_agent_db_01 mục D)

2. `market_recent` slice 5 phiên (`recent_price[0..4]`):
   - Tính GTGD trung bình tuần, biến động tuần từ `price.close`, `price.trading_value`
   - **Lưu ý:** `market_recent` KHÔNG có `money_flow_score` (xem K_agent_db_01 mục D). Để có chuỗi 5 phiên dòng tiền cấp thị trường, dùng aggregate từ `industry_snapshot.money_flow_score.week_score` 24 ngành (cấp ngành) hoặc `group_snapshot.money_flow_score.day_score` 6 nhóm.

3. `industry_snapshot` (24 doc) — aggregate `money_flow_score.week_score` lấy mean/median làm proxy điểm dòng tiền thị trường tuần. Aggregate `money_flow_score.day_score` 24 ngành làm proxy điểm dòng tiền phiên cuối tuần.

4. `industry_recent` slice 5 phiên × 24 ngành — aggregate `series[i].money_flow_score.day_score` mean theo phiên để có chuỗi 5 phiên dòng tiền proxy thị trường.

5. `market_nntd` slice 5 phiên — net_value tuần, top mua/bán ròng tuần

6. `data_briefing` block market — breadth_in/out toàn thị trường phiên cuối tuần

7. `other_data` filter group `macro.exchange_rate` + `macro.monetary` (lãi suất liên ngân hàng, OMO, tỷ giá VCB)

**Methodology aggregate NN/TD tuần:**

Tổng net_value 5 phiên gần nhất = NN net tuần. Top 5 mã NN mua ròng tuần lấy từ `stock_nntd` aggregate net_value 5 phiên (sort desc), top 5 bán ròng tương tự (sort asc). Filter thanh khoản tối thiểu 5 tỷ/phiên trung bình tuần để loại nhiễu.

**Output structured:**
- VNINDEX: giá đóng, biến động tuần/tháng/quý
- Thanh khoản: GTGD phiên cuối tuần + GTGD trung bình tuần + biến động vs trung bình tháng
- Dòng tiền nội: điểm dòng tiền tuần thị trường (proxy aggregate 24 ngành), chuỗi điểm dòng tiền phiên 5 phiên (mô tả pattern: đồng đều dương / đồng đều âm / dao động / phục hồi cuối tuần)
- Breadth phiên cuối tuần: số ngành tăng/giảm, số mã tăng/giảm toàn thị trường
- NN/TD tuần: tổng mua/bán ròng + top 5 mỗi chiều
- Tỷ giá VN, lãi suất liên ngân hàng các kỳ hạn, OMO

### 5.4. Phần 5 — Vĩ mô & hàng hoá — yếu tố dẫn dắt ngành

**Phần quan trọng nhất để chuẩn bị regime call.** Phần này thiết lập context "ngành nào đang được vĩ mô tác động đáng kể tuần này" trước khi đọc bảng 24 ngành ở phần 6.

**Workflow logic (reverse — quét chỉ số trước, suy ngành sau):**

1. Quét toàn bộ chỉ số vĩ mô + commodity tuần qua, **detect chỉ số nào có biến động đáng kể** (định tính: tăng/giảm rõ rệt vượt ngoài biến động bình thường, có pattern đảo chiều, hoặc vượt ngưỡng tâm lý)
2. Với mỗi chỉ số đáng kể → suy ra ngành VN bị tác động qua mapping cơ chế
3. Output bảng kết luận chỉ liệt kê các ngành CÓ signal vĩ mô tuần này, không liệt kê 24 ngành cố định
4. Nếu tuần không có biến động vĩ mô nào đáng kể → bỏ qua phần kết luận, không ghi "không có gì đặc biệt"

**Query DB:**

`other_data` filter:
- **Lãi suất**: `macro.monetary` đầy đủ — lãi suất tái cấp vốn, chiết khấu, liên ngân hàng các kỳ hạn (2W/1M/3M), huy động, cho vay qua đêm
- **Tỷ giá**: `macro.exchange_rate` đầy đủ + `international.fx` (EUR/USD, USD/CNY)
- **Hàng hoá**:
  - `commodities.energy`: Dầu Brent, WTI, khí tự nhiên, than nhiệt, than cốc
  - `commodities.metals`: Quặng sắt, thép HRC, vàng, bạc, đồng
  - `commodities.chemical`: Urea Trung Đông, urea Trung Quốc, phốt pho vàng, nhựa PP/PVC/PET
  - `commodities.agriculture`: cà phê, hồ tiêu, cao su, gạo XK, đường, ngô, đậu tương, heo hơi, tôm thẻ

**Mapping cơ chế chỉ số → ngành VN:** tham chiếu trực tiếp `K_agent_db_01` mục F "Sử dụng trong phân tích" (line 830-839) cho danh sách 9 mapping core (Dầu khí, Thép, Phân bón, Ngân hàng, BĐS, Xuất khẩu, Vàng/Bạc, Nông nghiệp, Tâm lý toàn cầu). Bổ sung từ `K_agent_db_00` mục 8 "Lăng kính phân tích cốt lõi" cho mapping hướng tác động (tích cực / tiêu cực) chi tiết hơn theo cơ chế biên gộp / chi phí đầu vào / dòng tiền tiêu dùng. Pack KHÔNG redefine bảng mapping — đọc trực tiếp K khi cần.

**Định nghĩa "biến động đáng kể" (định tính):**

Agent dùng judgment dựa các yếu tố sau, không có ngưỡng % cứng:
- Biến động tuần / tháng vượt rõ phạm vi dao động bình thường của chỉ số đó
- Có pattern đảo chiều rõ (chuyển từ giảm sang tăng hoặc ngược lại sau giai đoạn ổn định)
- Vượt mức tâm lý (ví dụ dầu Brent vượt 100 USD, USD/VND vượt 26500, vàng vượt mốc tròn)
- Có liên hệ với tin tức / sự kiện vĩ mô tuần (bổ trợ phần 8)

Khi nghi ngờ một chỉ số có đáng kể không, tham chiếu thêm chuỗi `m_pct` và `q_pct` để xem biến động tuần có ngoài normal không.

**Output structured:**

Sub-section 5.1 Lãi suất: bảng số liệu + 2-3 dòng diễn giải biến động tuần qua  
Sub-section 5.2 Tỷ giá: bảng số liệu + 2-3 dòng diễn giải  
Sub-section 5.3 Hàng hoá: bảng phân theo nhóm (kim loại / năng lượng / nông sản / hoá chất) — số liệu thuần, không có cột "ngành nhạy" trong bảng này

Sub-section 5.4 **Tác động lên ngành VN tuần này** — bảng động:
- Cột 1: Chỉ số biến động đáng kể (tên + biến động tuần cụ thể)
- Cột 2: Ngành VN bị tác động
- Cột 3: Hướng tác động (tích cực / tiêu cực) + cơ chế (1 dòng)

Bảng chỉ liệt kê các chỉ số có biến động đáng kể tuần qua. Nếu tuần không có biến động vĩ mô / commodity nào đáng kể, bỏ qua sub-section 5.4 — không render bảng rỗng, không ghi "không có gì đặc biệt".

Mục đích sub-section 5.4: input trực tiếp cho phần 6 (đọc bảng ngành kèm context vĩ mô) và checkpoint 1 (regime + sector bias). Ngành không xuất hiện ở 5.4 = vĩ mô không tác động trực tiếp tuần này, đánh giá qua flow + tin tức ở phần 6, 8.

### 5.5. Phần 6 — Biến động ngành

**Query DB:**

`industry_snapshot` (24 doc) toàn bộ 24 ngành:
- `industry_name`
- `price.pct_change`, `change.w_pct`, `change.m_pct`
- `money_flow_score.week_score`, `money_flow_score.industry_rank`

`industry_finstats` (24 doc) cho định giá ngành — `valuation_ratios[]` lấy P/E và P/B median ngành (xem K_agent_db_01 mục B `industry_finstats`).

`industry_recent` (24 doc) slice `series[1..5]` lấy 5 phiên trước đó để tính delta `money_flow_score.week_score` tuần này vs tuần trước (`series[0]` = phiên hiện tại đã ở snapshot).

**Output structured:**

Bảng 24 ngành sort theo xếp hạng ngành giảm dần, cột:

| # | Ngành | Biến động tuần (%) | Biến động tháng (%) | P/E hiện tại | P/B hiện tại | Điểm dòng tiền tuần | Xếp hạng ngành |

Diễn giải prose 4-6 dòng:
- Top ngành xếp hạng cao + cross-check với sub-section 5.4 phần 5: ngành nào xếp hạng cao và đồng thời xuất hiện trong bảng tác động vĩ mô (hướng tích cực) = candidate dẫn dắt thật
- Phát hiện phân kỳ:
  - Ngành biến động giá tuần dương rõ rệt nhưng điểm dòng tiền tuần thấp hoặc xếp hạng tụt = nghi trụ kéo, cảnh giác (vài mã vốn hoá lớn kéo giá ngành, đa số mã không tham gia)
  - Ngành biến động giá đi ngang hoặc nhẹ nhưng điểm dòng tiền tuần cao và đồng thời được vĩ mô ủng hộ = đang tích luỹ chuẩn bị bứt
- Ngành xếp hạng thấp + biến động giá âm + xuất hiện ở 5.4 với hướng áp lực = ngành cần thận trọng

### 5.6. Phần 7 — Top dẫn dắt 2 góc nhìn

**Query DB:**

1. **Top 10 mã theo biến động giá tuần** (5 tăng + 5 giảm):
   - `stock_highlight` collection — đã pre-compute top tăng/giảm theo nhóm
   - Hoặc query `stock_snapshot` aggregate `change.w_pct` sort desc/asc, filter `price.trading_value` ≥ 5 tỷ/phiên trung bình tuần (loại nhiễu penny)
   - Limit 5 mỗi chiều

2. **Top 10 mã theo dòng tiền tuần**:
   - `stock_snapshot` sort `money_flow_score.week_score` desc
   - Filter thanh khoản trung bình tuần ≥ 5 tỷ/phiên (loại nhiễu penny, không phải analytic threshold)
   - Tiebreak bằng thanh khoản trung bình tuần khi điểm dòng tiền tuần bằng nhau
   - Limit 10

**Edge case tuần thị trường yếu toàn diện:**

Nếu top 10 dòng tiền chứa mã có điểm dòng tiền tuần âm hoặc bằng 0:
- Vẫn render bảng đầy đủ 10 mã
- Ghi note honest dưới bảng: "Tuần thị trường yếu toàn diện, danh sách top 10 dòng tiền có mã điểm dòng tiền âm hoặc bằng 0 — đây là 'ít yếu nhất' chứ không phải dẫn dắt thực sự. Cross-check phần 7.3 cần đọc với cảnh báo này."

**Methodology cross-check:**

- Mã ở **cả 2 list** = dẫn dắt thật, có cả lực giá và dòng tiền
- Mã chỉ ở list **dòng tiền cao + giá chưa chạy** = đang gom kín, watch sát tuần tới
- Mã chỉ ở list **biến động giá cao + dòng tiền không cao** = chạy nhanh, cảnh giác bền vững

**Output structured:**

Bảng top 5 tăng + top 5 giảm theo giá tuần: ticker, ngành, % tuần, GTGD trung bình tuần  
Bảng top 10 dòng tiền: ticker, ngành, điểm dòng tiền tuần, % tuần, GTGD  
Note edge case nếu áp dụng (tuần yếu)  
Note cross-check: liệt kê mã thuộc nhóm 1 (cả 2 list), nhóm 2 (gom kín), nhóm 3 (chạy nhanh không bền)

### 5.7. Phần 8 — Tin tức & catalyst

**Query DB:**

1. `news_history_feed` filter `created_at` trong tuần qua, `type: news_feed`:
   - Filter `news_type: doanh_nghiep` → tin doanh nghiệp tuần
   - Filter `news_type: trong_nuoc` → tin trong nước
   - Filter `news_type: quoc_te` → tin quốc tế
   - Filter `news_type: thong_cao` → thông cáo Chính phủ
   - Lấy `article_slug` để dẫn link finext.vn

2. `news_history_feed` filter `type: report_feed` — báo cáo tổng hợp trong tuần

3. `news_history_content` đọc nội dung 1 số tin chính sau khi screen feed

**Web search:** bổ sung tin quốc tế lớn không có trong DB (FOMC minutes, ECB statement, GDP TQ, geopolitics).

**Methodology chấm impact:**

Áp framework 5 cổng logic K_agent_db_05 mục 2.2 — **nội bộ, không lộ tên framework hay điểm số ra output**. Chấm xong chỉ giữ tin có impact MID/HIGH (>= 4 điểm), bỏ tin LOW.

**Output structured:**

Sub-section 8.1: 3-5 tin trong nước impact ngành (kết hợp doanh_nghiep + trong_nuoc + thong_cao)  
Sub-section 8.2: 2-3 tin quốc tế ảnh hưởng VN  
Sub-section 8.3: Bảng mapping 3 cột:
- Tin / sự kiện (1 dòng)
- Ngành VN ảnh hưởng
- Hướng tác động (tích cực / tiêu cực / trung tính, kèm 1 dòng cơ chế)

Mỗi tin có dẫn link `https://finext.vn/news/<slug>` hoặc URL web search.

### 5.8. Phần 9 — Phân tích kỹ thuật VNINDEX + 3 kịch bản + Risk map

**Query DB:**

`market_snapshot` (1 doc):
- `price.open, high, low, close, volume, trading_value, volume_strength_index, diff, pct_change`
- `change.w_pct, m_pct, q_pct, y_pct`
- `technical_indicator.ma`: ma5, ma20, ma60, ma120, ma240
- `technical_indicator.fibonacci.{w|m|q|y}`: `f382, f500, f618` (Fibonacci 38.2/50/61.8% — K_01 chỉ có 3 mức này, không có 23.6/78.6)
- `technical_indicator.volume_profile.{w|m|q|y}`: poc, val, vah
- `technical_indicator.pivot.{w|m|q|y}`: pivot, r1, s1 (Classical Pivot — K_01 chỉ có pivot/r1/s1, không có r2/r3/s2/s3)
- `technical_zone.overall.{w|m|q|y}`: zone đa khung (AAA/AA/A/B/C — chỉ dùng nội bộ, dịch khi output)

**Lưu ý:** `market_snapshot` không có field `range_position`. Vị thế giá trong biên độ tuần/tháng/quý tự tính từ `price.close` và `technical_indicator.ohl.{w|m|q|y}.prev_high/prev_low`.

`market_recent` slice 20 phiên (`recent_price[0..19]`) — xác nhận vận động giá + volume trend (KHÔNG có `money_flow_score`, có `recent_trend` riêng nhưng pack này cấm dùng trend).

**KHÔNG query `market_snapshot.trend` và `market_recent.recent_trend` — pack này cấm dùng trend.**

**Methodology:**

Compose theo K_agent_db_04 phần C (Technical Zone & chỉ báo kỹ thuật) — không dùng phần B (TREND).

3 kịch bản if-then trigger:

- **Kịch bản cơ sở**: điều kiện hiện tại tiếp tục → vùng dao động dự kiến. Trigger duy trì: VNINDEX giữ trên POC/Fibonacci hỗ trợ tuần + điểm dòng tiền phiên proxy thị trường dao động giữa âm nhẹ và dương nhẹ
- **Kịch bản tích cực**: trigger break-out cụ thể → mục tiêu kỹ thuật. Trigger ví dụ: đóng cửa trên kháng cự gần (high tuần / Fibonacci kháng cự) + volume > 1.2x trung bình + điểm dòng tiền phiên proxy thị trường dương 3 phiên
- **Kịch bản tiêu cực**: trigger break-down cụ thể → vùng hỗ trợ sâu. Trigger ví dụ: đóng cửa dưới hỗ trợ kép (POC quý + MA60) + điểm dòng tiền phiên proxy thị trường âm 3 phiên + thanh khoản tăng phiên giảm

**Mỗi kịch bản KHÔNG được gán % xác suất** (tuân K_agent_db_00 mục 4.3).

**Output structured:**

Sub-section 9.1 Diễn biến giá tuần: prose 3-4 dòng + nến cuối tuần, vị thế MA, biên độ tuần/tháng/quý  
Sub-section 9.2 Vùng kỹ thuật quan trọng: bảng kháng cự + hỗ trợ 4 khung (tuần / tháng / quý / năm), mỗi khung 2-3 mức (Fibonacci, POC, pivot, MA). Bao gồm vùng giá VNINDEX cần theo dõi tuần tới — kháng cự gần (1-2 mức), hỗ trợ gần (1-2 mức), breakout level (mức nếu phá lên/xuống thì confirm kịch bản tích cực/tiêu cực).  
Sub-section 9.3 Ba kịch bản: định nghĩa rõ trigger (đóng cửa trên/dưới mức X + điều kiện volume + điều kiện flow) và vùng mục tiêu

**Note kỹ thuật bắt buộc sau 9.3:** thêm 1 dòng "Các kịch bản là hệ thống điều kiện kỹ thuật, không phải dự báo chắc chắn. Diễn biến thực tế có thể lệch khỏi cả 3 nếu xuất hiện sự kiện ngoài kỳ vọng."

Sub-section 9.4 **Risk map** — 3-7 rủi ro chính tuần tới (flex theo bối cảnh, không ép số cứng). Mỗi rủi ro 3 dòng:
- Bản chất: rủi ro gì, ảnh hưởng đến kịch bản cơ sở thế nào
- Signal materialize: chỉ báo / sự kiện / mức giá nào để biết rủi ro đang xảy ra
- Phản ứng đề xuất (định tính, không command): nếu materialize thì hướng nào (giảm exposure / chuyển sector / đứng ngoài)

### 5.9. Tổng hợp Stage 1 — quyết regime + sector bias

Sau khi compose xong phần 2-9, agent **không in ra báo cáo** mà compose 1 block call sơ bộ và xuất ra checkpoint 1.

## 6. Checkpoint 1 — Regime + Sector bias

Đây là checkpoint duy nhất của workflow. Đặt giữa phần 9 và phần 10.

### 6.1. Method regime classification (4 input)

**Input 1 — Dòng tiền thị trường (proxy aggregate, vì `market_snapshot` không có `money_flow_score`):**
- Mean/median `industry_snapshot.money_flow_score.week_score` 24 ngành — proxy điểm dòng tiền tuần thị trường (dương/âm)
- Chuỗi 5 phiên: `industry_recent.series[0..4].money_flow_score.day_score` aggregate mean theo phiên qua 24 ngành (đồng đều dương / dao động / đồng đều âm / phục hồi cuối tuần)

**Input 2 — Breadth:**
- `data_briefing` block market: breadth_in/out (số mã tăng/giảm phiên cuối tuần)
- Tự tính: số ngành (trong 24) có biến động giá tuần dương / 24

**Input 3 — Dòng tiền khối ngoại:**
- `market_nntd` aggregate 5 phiên: net_value tuần (tỷ đồng)
- Aggregate 20 phiên: net_value tháng (để biết xu hướng)

**Input 4 — Vĩ mô context** (lấy từ phần 5):
- Lãi suất điều hành ổn định / có dấu hiệu siết / có dấu hiệu nới
- Tỷ giá USD/VND ổn định / áp lực mất giá / hỗ trợ
- Có / không có biến động commodity đáng kể tuần này (đếm từ sub-section 5.4)

### 6.2. Mapping 4 regime — định tính

| Regime | Đặc điểm tổng thể | Ngụ ý chiến lược |
|---|---|---|
| Risk-on full | Dòng tiền tuần dương mạnh, chuỗi day_score 5 phiên đa số dương rõ; đa số ngành tăng giá tuần; NN mua ròng đáng kể; vĩ mô ủng hộ phần lớn ngành dẫn đầu | Universe rộng, 5 ngành quan tâm, mức độ thận trọng thấp |
| Risk-on selective | Dòng tiền tuần dương nhẹ hoặc dao động trong tuần; ngành phân hoá rõ giữa tăng và giảm; NN trung tính hoặc một chiều nhẹ; vĩ mô hỗn hợp | Chọn lọc kỹ, 3-4 ngành quan tâm, ưu tiên ngành có cả flow mạnh và vĩ mô ủng hộ rõ |
| Defensive only | Dòng tiền tuần âm hoặc dương rất nhẹ; đa số ngành giảm giá tuần; NN bán ròng; hoặc có rủi ro vĩ mô đang materialize | 2 ngành quan tâm — chỉ ngành phòng thủ + ngành có catalyst riêng đủ mạnh, mức độ thận trọng cao |
| Đứng ngoài | Dòng tiền tuần âm sâu kéo dài nhiều tuần; hầu hết ngành giảm giá; NN bán ròng kéo dài; hoặc shock vĩ mô lớn | Giữ tiền mặt, không chọn ngành quan tâm, chỉ watchlist sang tuần sau |

**Cách dùng bảng:** Agent đọc số liệu thực tế từ 4 input, đối chiếu mô tả định tính từng regime, chọn regime gần nhất với bức tranh tổng thể tuần. Khi combo input nằm ranh giới giữa 2 regime, agent dùng judgment có ngữ cảnh (xu hướng đang cải thiện hay xấu đi so tuần trước, tin tức quan trọng nào sắp có) để chọn — không cố ép vào quy tắc cứng.

Checkpoint 1 user review judgment, không review "ngưỡng có đúng không" — bảng định tính ngay từ đầu, không có ngưỡng số để debate.

### 6.3. Sector bias

Sau khi xác định regime, chọn sector bias từ phần 6 (bảng 24 ngành) cross-check sub-section 5.4 phần 5 (tác động vĩ mô tuần này):

**Ngành quan tâm** (số lượng flex theo regime — full 5 / selective 3-4 / defensive 2):
- Ngành rank cao về dòng tiền + week_score dương rõ + (có thể có hoặc không có) tác động vĩ mô tích cực ở 5.4
- Phân biệt dẫn dắt thật vs trụ kéo bằng đếm tỷ lệ mã trong ngành tăng giá tuần:
  - Cách đếm: query `stock_snapshot` filter industry, count `change.w_pct > 0` / total mã ngành
  - **Đa số mã trong ngành cùng tăng giá tuần** = dẫn dắt thật
  - **Chỉ vài mã vốn hoá lớn tăng trong khi đa số đứng/giảm** = nghi trụ kéo, cảnh giác
  - **Phân hoá** (gần 50/50) = ngành đang trong giai đoạn rotation nội bộ, cần thêm context để phán đoán

**Ngành cần thận trọng** (2-3 ngành):
- Ngành rank thấp về dòng tiền + biến động giá tuần âm rõ + xuất hiện ở 5.4 với hướng áp lực
- Hoặc ngành quá mua đa khung (P/E + biến động tháng cao bất thường so median 3Y)
- Hoặc ngành có catalyst tiêu cực mới phần 8

### 6.4. Block xuất tại checkpoint

Agent xuất block sau khi compose Stage 1, trước khi chạy Stage 2:

```
─── REGIME + SECTOR BIAS — Call sơ bộ ───

**Regime call sơ bộ:** [risk-on full / risk-on selective / defensive only / đứng ngoài]

**Lý do (4 input):**
- Dòng tiền thị trường: điểm dòng tiền tuần [dương mạnh / dương nhẹ / trung tính / âm nhẹ / âm sâu], chuỗi 5 phiên [đồng đều dương / dao động / đồng đều âm / phục hồi cuối tuần]
- Breadth: [đa số / quá nửa / dưới nửa / phần lớn] ngành tăng giá tuần; [N] mã tăng / [M] mã giảm phiên cuối
- NN/TD tuần: net = U tỷ ([mua ròng / bán ròng / trung tính])
- Vĩ mô: [tóm tắt 1-2 dòng — biến động đáng kể nào tuần qua tác động ngành VN, hoặc "không có biến động vĩ mô đáng kể tuần này, đánh giá qua flow + tin tức"]

**Sector bias đề xuất:**
- Ngành quan tâm [N ngành]: [ngành 1 — 1 dòng lý do tích cực], [ngành 2 — ...], ...
- Ngành cần thận trọng [M ngành]: [ngành A — 1 dòng lý do thận trọng], [ngành B — ...], ...

Confirm hay override trước khi tiếp phần 10-12?
- (a) Confirm như trên
- (b) Override regime → [user nêu regime mới + lý do]
- (c) Override sector bias → [user nêu thay đổi]
- (d) Cần phân tích thêm số liệu cụ thể trước khi quyết
```

### 6.5. Xử lý phản hồi user

| User chọn | Action |
|---|---|
| (a) Confirm | Stage 2 chạy thẳng với regime + sector bias đã call |
| (b) Override regime | Ghi note inline trong báo cáo cuối phần 10: "Regime chốt sau review: X. Override note: [lý do user]". Stage 2 chạy với regime mới |
| (c) Override sector bias | Ghi note tương tự. Stage 2 chạy với sector bias mới |
| (d) Phân tích thêm | Agent query bổ sung theo yêu cầu user, refine call, hỏi lại cùng pattern |

## 7. Stage 2 — Compose phần 10, 11, 12, rồi quay về phần 1

### 7.1. Phần 10 — Watchlist (Mã đáng chú ý)

**Input:** regime đã chốt + sector bias đã chốt + dữ liệu mã từ phần 6, 7, 8.

**Wording đặc biệt:** dùng dạng observation/luận điểm đầu tư, KHÔNG dùng từ command như "mua", "bán", "giảm tỷ trọng", "stop loss". Diễn đạt qua catalyst + flow + sự kiện sắp tới. KHÔNG có level giá vào/ra/stop. Phù hợp cả audience nội bộ (analyst tự diễn dịch thành action) và audience khách hàng (đọc như góc nhìn phân tích, tự cân nhắc).

Ví dụ wording đúng:
- "Catalyst Q1 dự kiến tích cực, dòng tiền cải thiện rõ tuần qua, vùng kỹ thuật khung tuần tích cực"
- "Áp lực từ chi phí đầu vào tăng, kết quả Q1 có thể yếu hơn kỳ vọng, dòng tiền tuần đang rút ra"

Ví dụ wording cần tránh:
- "Mua khi giá pullback về vùng X" (command + level giá)
- "Bán giảm tỷ trọng" (command)
- "Stop loss tại Y" (level)

**Query DB:**

Cho mỗi ngành quan tâm (sector bias tích cực):
- `stock_snapshot` filter industry, sort theo combo `money_flow_score.week_score` + `money_flow_score.market_rank_pct`
- Filter điều kiện cơ bản: `technical_zone.overall.w` ∈ (A, AA, AAA), khối lượng phiên gần nhất tăng đáng kể so trung bình 5 phiên (`price.volume_strength_index` cao hơn rõ rệt so 1.0), thanh khoản trung bình tuần ≥ 5 tỷ/phiên (loại nhiễu penny)

Cho ngành cần thận trọng (sector bias tiêu cực):
- `stock_snapshot` filter industry, lấy mã có `money_flow_score.market_rank_pct` cao trong ngành nhưng ngành tổng thể yếu (mã vốn hoá lớn thường được nắm giữ rộng, có thể bị áp lực bán nếu ngành xấu hơn)

**Output structured:**

Sub-section 10.1 Bối cảnh sector bias (intro 4-6 dòng):
- 1-2 dòng nêu regime tuần + ngụ ý chiến lược (universe rộng/hẹp, mức độ thận trọng)
- **Ngành quan tâm**: list ngắn gọn tên ngành kèm 1 câu lý do mỗi ngành
- **Ngành cần thận trọng**: list ngắn gọn tên ngành kèm 1 câu lý do
- 1 dòng risk-reward định tính tuần tới ("rủi ro xuống lớn hơn tiềm năng tăng" / "cân bằng" / "tiềm năng tăng có ưu thế")

Sub-section 10.2 Mã đáng chú ý: 5-8 mã gộp cả cơ hội tăng và rủi ro giảm — không tách 2 nhóm. Mỗi mã 2 dòng:
- Dòng 1: ticker (ngành) — luận điểm đầu tư 1 câu (rõ hướng tích cực hay tiêu cực qua wording)
- Dòng 2: catalyst + flow + technical (ngắn gọn, observation)

**Lưu ý:** không ghi entry zone, stop, target, size cụ thể. Định tính qua observation.

### 7.2. Phần 11 — Lịch sự kiện tuần tới

**Input:** sự kiện corporate từ phần 8 + macro events từ web search.

**Web search bổ sung:**
- Macro release lịch tuần: FOMC minutes, CPI Mỹ, NFP, GDP, PMI Mỹ/EU/TQ/VN
- Họp chính sách tuần: FOMC, ECB meeting, NHNN VN
- Mùa BCTC: nếu đang trong mùa, lấy lịch công bố BCTC các mã top vốn hoá trong sector bias quan tâm

**Query DB:**
- `news_today_feed` + `news_history_feed` filter type báo cáo / thông cáo có ngày dự kiến tuần tới (chính sách, đấu giá TPCP, đáo hạn hợp đồng tương lai, ngày chốt cổ tức)
- Sự kiện corporate: BCTC dự kiến công bố, ĐHCĐ, divestment, M&A, niêm yết công ty con — chỉ ngày + sự kiện + ngành/mã liên quan, không kèm khuyến nghị action

**Output structured:**

Sub-section 11.1 Lịch macro: bảng ngày + sự kiện + ngành VN ảnh hưởng (nếu có)
Sub-section 11.2 Lịch corporate: bảng ngày + ticker + sự kiện (BCTC/ĐHCĐ/M&A/etc.)

Lịch chỉ liệt kê sự kiện đáng chú ý — bỏ qua sự kiện không impact materially.

### 7.3. Phần 12 — Tuyên bố miễn trừ trách nhiệm

**Input:** branding + disclaimer info từ pre-flight (Section 4 câu 3).

**3 trường hợp:**

**(a) User cung cấp custom disclaimer:**
Render đầy đủ disclaimer text user cung cấp + thông tin liên hệ (tên công ty, website, hotline, phòng ban).

**(b) User cung cấp branding nhưng không có disclaimer text:**
Dùng template default phù hợp báo cáo gửi KH:

> Báo cáo này được [TÊN CÔNG TY] soạn thảo trên cơ sở các thông tin, dữ liệu được thu thập từ các nguồn được coi là đáng tin cậy vào thời điểm phát hành. Tuy nhiên, [TÊN CÔNG TY] không đảm bảo tuyệt đối về tính chính xác, đầy đủ của các thông tin này.
>
> Các nhận định trong báo cáo phản ánh quan điểm độc lập tại thời điểm công bố và có thể thay đổi mà không cần thông báo trước. Nhà đầu tư cần tự đánh giá mức độ phù hợp với tình hình tài chính cá nhân, khả năng chịu rủi ro và mục tiêu đầu tư trước khi ra quyết định.
>
> Quyết định đầu tư cuối cùng hoàn toàn thuộc về Quý khách hàng. [TÊN CÔNG TY] không chịu trách nhiệm về bất kỳ tổn thất nào phát sinh từ việc sử dụng báo cáo này.

Kèm khối liên hệ: tên công ty, website, hotline, ngày phát hành.

**(c) User chọn không cung cấp branding (pre-flight 3b):**
Render note plain ngắn:

> Báo cáo này phản ánh quan điểm phân tích nội bộ tại thời điểm soạn thảo và có thể thay đổi mà không cần thông báo. Nhà đầu tư tự cân nhắc dựa trên tình hình tài chính cá nhân và mục tiêu đầu tư.
>
> Ngày phát hành: [DD/MM/YYYY]

### 7.4. Phần 1 — Tóm tắt điều hành (viết cuối)

**Input:** toàn bộ 11 phần đã compose.

**Output:** 3-5 bullet, mỗi bullet 1-2 dòng:
- Bullet 1: regime tuần + 1 dòng tóm tắt căn cứ
- Bullet 2: 1-2 catalyst lớn nhất tuần qua (tin trong nước hoặc quốc tế)
- Bullet 3: sector bias quan tâm (ngắn gọn liệt kê tên ngành) + thận trọng
- Bullet 4: 1 risk chính cần theo dõi tuần tới (từ Risk map phần 9.4)
- Bullet 5 (optional): 1 mã đáng chú ý nhất trong watchlist phần 10

Phần 1 phải đứng riêng, đọc 30 giây hiểu được toàn báo cáo.

## 8. Render & deliver

Sau khi có đủ 12 phần structured content, agent gọi `O_weekly_market_00` để render thành file MD final. Render rule + format chi tiết ở O pack.

File output: `weekly_market_<YYYYMMDD>.md`. Trên Claude Desktop, agent xuất nội dung MD trong message để user copy/save thủ công.

Nếu user yêu cầu format khác (docx / pptx), render thêm theo `O_weekly_market_00` mục tương ứng.

## 9. Edge cases & xử lý

### 9.1. Thiếu dữ liệu DB

- `market_recent` query rỗng tuần (DB chưa update) → báo user, hỏi có muốn dùng phiên cuối tuần trước đó không
- `industry_snapshot` thiếu 1-2 ngành → ghi note "ngành X chưa có dữ liệu phiên cuối tuần", tiếp tục với 22-23 ngành còn lại
- `stock_nntd` rỗng → query `data_briefing` block NN/TD thay thế
- `news_history_feed` rỗng tuần → tăng tỷ trọng web search cho phần 8

### 9.2. Conflict regime call

Combo 4 input cho ra 2 regime tương đương khả thi (ví dụ: dòng tiền + breadth báo selective, vĩ mô + NN báo defensive):
- Agent xuất 2 candidate ở checkpoint, không chọn 1
- Block format: "Call có 2 candidate khả thi: [A] vs [B]. Lý do A: ... Lý do B: ... Cần user quyết."

### 9.3. User không phản hồi checkpoint

Sau khi xuất block checkpoint, agent dừng. Không tự chuyển Stage 2. Đợi user phản hồi trong session sau cũng được — pack không có timeout.

### 9.4. Tuần đầu cycle (không có file W-1)

Skip phần 2, ghi 1 dòng "Tuần đầu cycle, chưa có dữ liệu review". Workflow chạy bình thường từ phần 3.

### 9.5. Trùng pack với P_invest_memo

Agent có thể đang trong session đã activate `P_invest_memo`. Khi user trigger weekly market report, activate thêm `P_weekly_market` song song. Hai pack độc lập, không share state, không conflict — chỉ chia sẻ K_agent_db.

## 10. Self-audit trước khi xuất file

Chạy 9 câu trước khi present file:

1. Phần 2 đã skip đúng nếu không có file W-1?
2. Phần 5 đã đặt mapping vĩ mô → ngành làm input cho phần 6 không?
3. **Đã không dùng bất kỳ chỉ báo `trend` nào** (market_snapshot.trend, industry_snapshot.trend, group_snapshot.trend, recent_trend)?
4. Phần 9 ba kịch bản dùng if-then trigger, không có % xác suất?
5. Phần 9.4 Risk map có 3-7 rủi ro với đủ 3 dòng (bản chất / signal / phản ứng)?
6. Checkpoint 1 đã được user confirm hoặc override trước khi compose phần 10-12?
7. Watchlist phần 10 không có level giá vào/ra/stop cụ thể (observation only)?
8. Phần 12 disclaimer render đúng theo branding info (custom / default branded / plain)?
9. K hygiene: ký hiệu DB raw đã dịch hết, không còn `week_score: 18`, `industry_rank: 3`, `vsi: 2.1` raw trong output? Số liệu định lượng đã quy đổi đơn vị (BCTC: tỷ đồng, % thập phân nhân 100)?

Vi phạm câu nào sửa rồi mới render file final.

## 11. Output contract

Pack sinh ra structured content cho `O_weekly_market_00` render. Ràng buộc:

- 12 phần đầy đủ theo thứ tự: 1 Tóm tắt điều hành / 2 Review tuần trước / 3 Bối cảnh quốc tế / 4 Thị trường Việt Nam / 5 Vĩ mô & hàng hoá / 6 Biến động ngành / 7 Top dẫn dắt / 8 Tin tức & catalyst / 9 PTKT VNINDEX + 3 kịch bản + Risk map / 10 Watchlist / 11 Lịch sự kiện tuần tới / 12 Tuyên bố miễn trừ trách nhiệm
- Phần rỗng vẫn render với 1 dòng note (vd phần 2 nếu không có W-1: "Tuần đầu cycle, chưa có dữ liệu review")
- Số liệu định lượng đã quy đổi đơn vị theo K_agent_db_00 mục 6
- Ký hiệu DB raw đã dịch theo K_agent_db_00 mục 5.2
- Thuật ngữ tiếng Anh đã dịch theo K_agent_db_05 mục 9
- Mỗi claim có nguồn truy được: collection + field, hoặc URL web search
- Tin có dẫn link finext.vn (`https://finext.vn/news/<slug>`) hoặc URL gốc
- Checkpoint 1 đã có user phản hồi (confirm/override) trước khi vào Stage 2
- Phần 1 viết cuối cùng, sau khi có đủ 11 phần còn lại

Pack KHÔNG tự quyết heading style / xưng hô / tone / length cuối cùng — `O_weekly_market_00` quyết.
