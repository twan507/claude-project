# P_weekly_overview_02 — Stage 1 second half (Phần 6-9)

File này cover compose **phần 6-9** của báo cáo `weekly_overview`. Dependency: master `P_weekly_overview_00` cho philosophy (fundamental-driven supremacy) + weight balance + Nguyên tắc 1 (fundamental trigger bắt buộc). Phần 2-5 ở `_01`, checkpoint + phần 10-12 + phần 1 ở `_03`, methodology ở `_04`. Render qua `O_weekly_overview_00`.

> **CRITICAL CALLOUT — Phần 9 trigger:** 3 kịch bản VNINDEX **BẮT BUỘC trigger primary là vĩ mô / cơ bản / chính sách / catalyst**. Technical chỉ confirmation phụ (≤30% nội dung phần 9.3). Mirror `P_vbse_strategy_05`. Vi phạm = re-write trước khi render. Đây là khác biệt cốt lõi so với pack `P_weekly_market` cũ.

## 1. Phần 6 — Biến động 18 ngành whitelist

### 1.1. Query DB

`industry_snapshot` **filtered 18 ngành whitelist** (xem `K_agent_db_01` Section B + `K_agent_db_00` mục 4.5):
- `industry_name`
- `price.pct_change`, `change.w_pct`, `change.m_pct`
- `money_flow_score.week_score` (dòng tiền tuần — **field chuẩn để rank ngành**)

`industry_finstats` **filtered 18 ngành whitelist** cho định giá ngành — `valuation_ratios[]` lấy P/E và P/B median ngành. **Mới so với pack cũ**: tính thêm **P/E phân vị 3 năm** (so P/E hiện tại với phân phối median 3Y của ngành — rẻ tương đối <30%, trung tính 30-70%, đắt tương đối >70%) và **EPS Q YoY** (lấy `earnings.eps_q_yoy` median ngành).

`industry_recent` **filtered 18 ngành whitelist** slice `series[1..5]` lấy 5 phiên trước đó để tính delta `money_flow_score.week_score` tuần này vs tuần trước (`series[0]` = phiên hiện tại đã ở snapshot).

**Rank tự tổng hợp:** DB không lưu `industry_rank` ngành-vs-ngành (xem `K_agent_db_01` mục "Xếp hạng ngành"). Agent tự sort `week_score` giảm dần qua 18 ngành whitelist → gán rank 1..18 trong-flight.

### 1.2. Output bảng — 9 cột (upgrade thêm 2 cột)

Bảng 18 ngành whitelist sort theo rank tự tổng hợp giảm dần (theo `week_score`):

| # | Ngành | w_pct | m_pct | P/E hiện tại | **P/E phân vị 3Y** | **EPS Q YoY** | P/B hiện tại | week_score | Rank (1..18 tự tổng hợp) |

**2 cột mới — `P/E phân vị 3Y` + `EPS Q YoY` — phục vụ identify "earnings beat candidate"** (xem sub-section 1.4). Đây là upgrade cơ bản angle, đẩy phần 6 từ flow-centric (pack cũ) sang fundamental + flow blend.

### 1.3. Diễn giải prose (4-6 dòng)

Logic giữ nguyên từ pack cũ:
- Top ngành rank cao + cross-check sub-section 5.4 (vĩ mô tích cực, ở `_01`) = candidate dẫn dắt thật cho phần 7 và sector bias checkpoint
- Phát hiện phân kỳ ngành giá vs dòng tiền:
  - Ngành w_pct dương rõ rệt nhưng week_score thấp hoặc rank tự tổng hợp tụt = **nghi trụ kéo**, cảnh giác (vài mã vốn hoá lớn kéo giá ngành, đa số mã không tham gia)
  - Ngành w_pct đi ngang/nhẹ nhưng week_score cao + đồng thời được vĩ mô ủng hộ = **đang tích luỹ chuẩn bị bứt**
- Ngành rank thấp + w_pct âm + xuất hiện ở 5.4 với hướng áp lực = **cần thận trọng** (input cho checkpoint sector bias)

### 1.4. Sub-section 6.x — Top 3 "Earnings beat candidate"

Mới so với pack cũ. Ngành đáp ứng **tất cả 3 tiêu chí**:
- `EPS Q YoY` ≥ 15%
- `P/E phân vị 3Y` < 40%
- `week_score` dương và cải thiện 2 tuần liên tiếp (cross-check `industry_recent` series)

Liệt kê 1-3 ngành với 1 dòng diễn giải mỗi ngành (gắn 1 catalyst cụ thể mùa BCTC nếu có).

Nếu không có ngành nào pass: ghi đúng câu "**Tuần này không có earnings beat candidate rõ ràng.**" Không ép số.

### 1.5. Cap technical phần 6: ≤ 5%

Bảng + diễn giải prose tập trung cơ bản (định giá + EPS) + flow (week_score + rank). Không có cột `technical_zone` trong bảng quyết định bias. Vùng kỹ thuật ngành chỉ minh hoạ tối thiểu nếu thực sự cần.

### 1.6. Structural watch (exception, không default)

**Mặc định KHÔNG dùng `K_sector_framework`** ở phần này — broadcast tuần ưu tiên ngắn gọn 9-11 trang, không cần industry deep-dive mỗi tuần.

**Exception:** khi 1 ngành có biến động bất thường tuần này (vd dòng tiền tuần top 1 hoặc bottom 1, hoặc có catalyst lớn material) → có thể pull 1 dòng "Structural watch" từ `K_sector_framework` mục 5.X tương ứng (1 driver cấu trúc dominant 3-5 từ). Không quá 1 dòng/ngành. Không quá 2 ngành/tuần.

Format ví dụ trong bảng 18 ngành: thêm cột "Structural watch" chỉ render khi exception, để trống cho ngành không exception. Hoặc note rời 1-2 dòng dưới bảng.

## 2. Phần 7 — Top dẫn dắt 2 góc nhìn + Cảnh báo trap

### 2.1. Query DB

**(1) Top 10 mã theo biến động giá tuần** (5 tăng + 5 giảm):
- Query `stock_snapshot` aggregate `change.w_pct` sort desc/asc, filter `price.trading_value` ≥ 5 tỷ/phiên trung bình tuần (loại nhiễu penny)
- **Filter `industry ∈ 18 ngành whitelist`**
- Limit 5 mỗi chiều

**(2) Top 10 mã theo dòng tiền tuần**:
- `stock_snapshot` sort `money_flow_score.week_score` desc
- Filter thanh khoản trung bình tuần ≥ 5 tỷ/phiên (loại nhiễu penny, không phải analytic threshold)
- **Filter `industry ∈ 18 ngành whitelist`**
- Tiebreak bằng thanh khoản trung bình tuần khi điểm dòng tiền tuần bằng nhau
- Limit 10

### 2.2. Edge case — tuần thị trường yếu toàn diện

Nếu top 10 dòng tiền chứa mã có `week_score` âm hoặc bằng 0:
- Vẫn render bảng đầy đủ 10 mã
- Ghi note honest dưới bảng: "Tuần thị trường yếu toàn diện, danh sách top 10 dòng tiền có mã điểm dòng tiền âm hoặc bằng 0 — đây là 'ít yếu nhất' chứ không phải dẫn dắt thực sự. Cross-check 2.3 cần đọc với cảnh báo này."

### 2.3. Methodology cross-check — 3 nhóm

- Mã ở **cả 2 list** = dẫn dắt thật, có cả lực giá và dòng tiền
- Mã chỉ ở list **dòng tiền cao + giá chưa chạy** = đang gom kín, watch sát tuần tới
- Mã chỉ ở list **biến động giá cao + dòng tiền không cao** = chạy nhanh, cảnh giác bền vững

### 2.4. Output structured

- Bảng top 5 tăng + top 5 giảm theo giá tuần: ticker, ngành, % tuần, GTGD trung bình tuần
- Bảng top 10 dòng tiền: ticker, ngành, điểm dòng tiền tuần, % tuần, GTGD
- Note edge case nếu áp dụng (tuần yếu)
- Note cross-check: liệt kê mã thuộc nhóm 1 (cả 2 list), nhóm 2 (gom kín), nhóm 3 (chạy nhanh không bền)

### 2.5. Sub-section 7.x — Cảnh báo "Late money / Trap setup"

**Mới so với pack cũ.** Cross-check: mã trong **top 10 dòng tiền** mà thuộc **ngành cần thận trọng**.

Sector bias "cần thận trọng" sẽ chốt chính thức ở Checkpoint 1, nhưng có thể **preview từ phần 6**: ngành rank tự tổng hợp thấp (`week_score` âm/yếu) + đồng thời có áp lực vĩ mô từ 5.4.

**Filter:** `stock_snapshot` với `industry ∈ 18 whitelist` nhưng thuộc nhóm ngành dự kiến flag "cần thận trọng" tại checkpoint, đồng thời nằm trong top 10 dòng tiền tuần.

**Logic phân tích:** dòng tiền vào ngành xấu thường là 1 trong 3 pattern:
- **Late money** (retail đuổi theo, sắp kẹt)
- **Priced-in tiêu cực trước** (chưa lộ tin xấu nhưng smart money chuẩn bị thoát)
- **Mã catalyst đặc biệt cá thể** (cần verify catalyst riêng mã)

Render bảng nếu có ≥1 mã thoả:

| Ticker | Ngành (rank thấp) | week_score | Cảnh báo (1 dòng) |

Cảnh báo viết theo 1 trong 3 pattern trên. Nếu không có mã nào thoả → **bỏ sub-section 2.5 hoàn toàn** (không render bảng rỗng, không note placeholder).

Đây là warning sign chuẩn buy-side cho retail trap.

### 2.6. Cap technical phần 7: ≤ 10%

So sánh giá vs dòng tiền là PRIMARY signal. Technical chỉ làm observation phụ nếu cần.

## 3. Phần 8 — Tin tức & catalyst

### 3.1. Query DB

**(1)** `news_history_feed` filter `created_at` trong tuần qua, `type: news_feed`:
- Filter `news_type: doanh_nghiep` → tin doanh nghiệp tuần
- Filter `news_type: trong_nuoc` → tin trong nước
- Filter `news_type: quoc_te` → tin quốc tế
- Filter `news_type: thong_cao` → thông cáo Chính phủ
- Lấy `article_slug` để dẫn link finext.vn

**(2)** `news_history_feed` filter `type: report_feed` — báo cáo tổng hợp trong tuần

**(3)** `news_history_content` đọc nội dung 1 số tin chính sau khi screen feed

**Web search:** bổ sung tin quốc tế lớn không có trong DB (FOMC minutes, ECB statement, GDP TQ, geopolitics).

### 3.2. Methodology chấm impact

Áp framework **5 cổng logic K_agent_db_05 mục 2.2** — **nội bộ, không lộ tên framework hay điểm số ra output**. Chấm xong chỉ giữ tin có impact MID/HIGH (>= 4 điểm), bỏ tin LOW.

### 3.3. Output structured

- **Sub-section 8.1:** 3-5 tin trong nước impact ngành (kết hợp `doanh_nghiep` + `trong_nuoc` + `thong_cao`)
- **Sub-section 8.2:** 2-3 tin quốc tế ảnh hưởng VN
- **Sub-section 8.3 — Bảng mapping (upgrade thêm cột conviction impact):**

| Tin / sự kiện | Ngành VN ảnh hưởng (whitelist 18) | Hướng tác động | **Conviction impact (LOW/MID/HIGH)** | Cơ chế 1 dòng |

**Conviction impact** (cột mới):
- **HIGH**: tin đã materialize + cơ chế tác động trực tiếp + magnitude lớn
- **MID**: tin cần verify thêm hoặc cơ chế tác động gián tiếp
- **LOW**: tin sớm, signal yếu

Ngành cột 2 **BẮT BUỘC thuộc 18 whitelist**; ngành ngoài note "(ngoài scope)" và không tính vào tổng hợp checkpoint sector bias.

Mỗi tin có dẫn link `https://finext.vn/news/<slug>` hoặc URL web search.

### 3.4. Cap technical phần 8: 0%

Phần tin tức + catalyst là PRIMARY signal cơ bản, không có technical content.

## 4. Phần 9 — Định vị VNINDEX + 3 kịch bản fundamental-driven + Risk map (MAJOR REWRITE)

**Rename:** source pack cũ "Phân tích kỹ thuật VNINDEX" → "**Định vị VNINDEX + Kịch bản tuần tới**". Bỏ vocab "PTKT" làm focus. Technical chuyển từ driver (pack cũ) sang confirmation phụ (pack này).

### 4.1. Query DB

`market_snapshot` (1 doc):
- `price.open, high, low, close, volume, trading_value, volume_strength_index, diff, pct_change`
- `change.w_pct, m_pct, q_pct, y_pct`
- `technical_indicator.ma`: ma5, ma20, ma60, ma120, ma240
- `technical_indicator.fibonacci.{w|m|q|y}`: `f382, f500, f618`
- `technical_indicator.volume_profile.{w|m|q|y}`: poc, val, vah
- `technical_indicator.pivot.{w|m|q|y}`: pivot, r1, s1
- `technical_zone.overall.{w|m|q|y}`: zone đa khung (AAA/AA/A/B/C — chỉ dùng nội bộ, dịch khi output)

**Lưu ý:** `market_snapshot` không có field `range_position`. Vị thế giá trong biên độ tuần/tháng/quý tự tính từ `price.close` và `technical_indicator.ohl.{w|m|q|y}.prev_high/prev_low`.

`market_recent` slice 20 phiên (`recent_price[0..19]`) — xác nhận vận động giá + volume trend (KHÔNG có `money_flow_score`, có `recent_trend` riêng nhưng pack này **cấm dùng trend**).

**KHÔNG query `market_snapshot.trend` và `market_recent.recent_trend` — pack này cấm dùng trend (rule giữ nguyên từ pack cũ).**

### 4.2. Methodology

- Sub-section 4.4 (Diễn biến giá) + 4.5 (Vùng giá tham chiếu): compose theo `K_agent_db_04` phần C (Technical Zone & chỉ báo kỹ thuật) — không dùng phần B (TREND).
- Sub-section 4.6 (3 kịch bản): **trigger primary fundamental** (xem chi tiết bên dưới), technical chỉ confirmation phụ ≤30%.
- Sub-section 4.7 (Risk map): trigger materialize PREFER macro/fundamental/policy.

### 4.3. Note kỹ thuật bắt buộc

Mỗi kịch bản **KHÔNG được gán % xác suất** (tuân `K_agent_db_00` mục 4.3 + master Nguyên tắc 1).

### 4.4. Sub-section 9.1 — Diễn biến giá tuần (render technical pure)

Prose 3-4 dòng + nến cuối tuần, vị thế MA, biên độ tuần/tháng/quý. Đây là render technical pure cho user reference — **không tính cap technical**.

### 4.5. Sub-section 9.2 — Vùng giá tham chiếu (renamed)

**Đổi tên:** pack cũ "Vùng kỹ thuật quan trọng" → pack này "**Vùng giá tham chiếu**" (bớt vocab PTKT, phù hợp audience broad).

Bảng kháng cự + hỗ trợ 4 khung (tuần / tháng / quý / năm), mỗi khung 2-3 mức (Fibonacci, POC, pivot, MA). Bao gồm vùng giá VNINDEX cần theo dõi tuần tới:
- Kháng cự gần (1-2 mức)
- Hỗ trợ gần (1-2 mức)
- Breakout level (mức nếu phá lên/xuống thì confirm kịch bản tích cực/tiêu cực)

Đây là render technical pure cho user reference — **không tính cap technical**.

### 4.6. Sub-section 9.3 — Ba kịch bản tuần tới (REWRITE fundamental-driven)

**Trigger primary BẮT BUỘC là macro / fundamental / policy / catalyst** (theo Nguyên tắc 1 master + mirror `P_vbse_strategy_05`). Technical chỉ confirmation phụ ≤30% nội dung phần này.

#### Kịch bản cơ sở
- **Trigger primary (macro/fundamental/policy):** vd "Fed giữ rates + lãi suất liên ngân hàng VN ổn định 4.5-5% + không có shock chính sách + tin trong nước trung tính + flow FII đi ngang"
- **Confirmation phụ (technical/flow):** vd "VNINDEX dao động trong band POC quý, breadth tổng thể duy trì >50%, điểm dòng tiền phiên proxy thị trường dao động giữa âm nhẹ và dương nhẹ"
- **Vùng VNINDEX dự kiến tuần tới:** [X] — [Y]
- **Hành vi kỳ vọng:** [định tính ngắn, sector nào dẫn dắt nhẹ, sector nào tích luỹ]

#### Kịch bản tích cực
- **Trigger primary:** vd "FOMC minutes dovish-leaning surprise" / "BCTC Q1 ngành ngân hàng beat consensus ≥10%" / "Nghị quyết X về tín dụng/đầu tư công thông qua" / "FII chuyển mua ròng ≥X nghìn tỷ tuần"
- **Confirmation phụ (technical/flow):** vd "VNINDEX đóng cửa trên POC quý + breadth > 60% + điểm dòng tiền phiên proxy thị trường dương 3 phiên liên tiếp + volume > 1.2x trung bình"
- **Vùng VNINDEX kỳ vọng:** [Y]
- **Sector/theme củng cố:** [list — cross-link phần 6 ngành rank cao + phần 8 tin tích cực]

#### Kịch bản tiêu cực
- **Trigger primary:** vd "Fed hawkish surprise" / "BCTC Q1 nhiều top sector miss consensus" / "USD/VND vượt 26,800 + NHNN can thiệp" / "Chính sách thắt tín dụng X ban hành" / "Geopolitics shock tác động chuỗi cung ứng"
- **Confirmation phụ:** vd "VNINDEX đóng cửa dưới hỗ trợ kép (POC quý + MA60) + breadth < 35% + FII bán ròng tăng cường + điểm dòng tiền phiên proxy thị trường âm 3 phiên + thanh khoản tăng phiên giảm"
- **Vùng hỗ trợ kỳ vọng:** [Y]
- **Sector/theme bị impact:** [list — cross-link phần 6 ngành rank thấp + phần 8 tin tiêu cực]

> *Kịch bản là hệ thống điều kiện vĩ mô/cơ bản/chính sách + confirmation technical, không phải dự báo chắc chắn. Không gán % xác suất. Diễn biến thực tế có thể lệch khỏi cả 3 nếu xuất hiện sự kiện ngoài kỳ vọng.*

### 4.7. Sub-section 9.4 — Risk map (UPGRADE thêm cross-link)

3-7 rủi ro chính tuần tới (flex theo bối cảnh, không ép số cứng). Mỗi rủi ro **4 dòng** (upgrade từ 3 dòng pack cũ):

**Rủi ro N — [Tên ngắn]**
- **Bản chất + cơ chế cơ bản:** [2-3 dòng — rủi ro gì, ảnh hưởng đến kịch bản cơ sở thế nào, cơ chế truyền dẫn vĩ mô/cơ bản]
- **Signal materialize (PREFER macro/fundamental):** [chỉ báo / sự kiện / mức số cụ thể — vd "PMI tháng X dưới 49", "DXY vượt 108", "BCTC top 5 ngân hàng miss ≥5%"]
- **Phản ứng định tính:** [vd "giảm exposure ngành X" / "chuyển defensive" / "đứng ngoài chờ" — KHÔNG dùng từ command trực tiếp]
- **Theme/sector bị invalidate (cross-link Phần 10):** [list nếu có — đây là dòng MỚI, cross-link với phần 10 sector bias + watchlist]

3-7 rủi ro tổng. Mirror `P_vbse_strategy_05` mục 4.

### 4.8. Cap technical phần 9

- 9.1 + 9.2: render technical pure, **không tính cap**
- 9.3: **≤ 30%** technical (confirmation phụ; primary là macro/fundamental/policy)
- 9.4: **0%** technical (trigger materialize PREFER macro/fundamental/policy)

## 5. Rules cho file này

1. **Voice/tone:** Vietnamese terse, institutional buy-side voice, không emoji
2. **Cross-references:**
   - `K_agent_db_00`, `K_agent_db_01`, `K_agent_db_04`, `K_agent_db_05` references giữ nguyên
   - File pack: master = `P_weekly_overview_00`, phần 2-5 = `_01`, file này = `_02`, checkpoint+phần 10-12+phần 1 = `_03`, methodology = `_04`
   - Render spec: `O_weekly_overview_00`
3. **Fundamental-driven callout** đã đặt ở intro top file cho phần 9 — không lặp lại trong từng sub-section
4. **18 ngành whitelist** áp dụng cho mọi aggregate + filter mention; source "24 ngành" → "**18 ngành whitelist**" (Nguyên tắc 2 master)
5. **Trend cấm dùng** (rule giữ nguyên từ pack cũ): KHÔNG query `market_snapshot.trend`, `market_recent.recent_trend`, `industry_snapshot.trend`, `industry_recent.recent_trend` ở mọi sub-section của file này
6. **Conviction + Horizon + Disconfirming** (Nguyên tắc 3 master): áp dụng chính ở Checkpoint 1 (`_03`); file này chỉ chuẩn bị input (sector bias preview ở 6.3 + cảnh báo trap ở 2.5 + 3 kịch bản fundamental-driven ở 4.6)
7. **Cap technical tổng phần 6+7+8+9 (trừ 9.1+9.2)**: tuân master mục 4.1 — self-audit trước render
